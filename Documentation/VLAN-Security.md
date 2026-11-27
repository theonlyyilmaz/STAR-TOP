# STAR-TOP — VLAN and Security Design

## 1. Overview

STAR-TOP uses VLAN segmentation on a Cisco C3725 Layer 3 switch to separate endpoint groups into distinct Layer 2 broadcast domains. The switch also provides Layer 3 gateway interfaces for inter-VLAN routing.

Access Control Lists (ACLs) are used to enforce the guest-network policy and restrict unwanted traffic between VLANs.

## 2. VLAN Plan

| VLAN ID | VLAN name | Subnet | Default gateway | Assigned access ports |
|---|---|---|---|---|
| 10 | VLAN10 | `192.168.10.0/24` | `192.168.10.1` | Fa1/0–Fa1/4 |
| 20 | VLAN20 | `192.168.20.0/24` | `192.168.20.1` | Fa1/5–Fa1/9 |
| 30 | VLAN30 | `192.168.30.0/24` | `192.168.30.1` | Fa1/10–Fa1/14 |

The port assignments above reflect the recorded C3725 configuration. Confirm the live switch output if the topology has subsequently changed.

## 3. Layer 3 Switching

The C3725 provides the VLAN gateway addresses through switched virtual interfaces (SVIs):

- VLAN 10: `192.168.10.1/24`
- VLAN 20: `192.168.20.1/24`
- VLAN 30: `192.168.30.1/24`

Inter-VLAN routing is enabled on the Layer 3 switch. This allows routing between VLANs at Layer 3, subject to the configured ACL policies.

## 4. Guest Network Policy — VLAN 10

VLAN 10 is designated as an isolated guest network.

**Policy requirements:**

- Hosts in VLAN 10 may access permitted external networks, including the internet.
- Hosts in VLAN 10 must not communicate with hosts in VLAN 20.
- Hosts in VLAN 10 must not communicate with hosts in VLAN 30.
- Access to internal resources should be denied unless a specific exception is explicitly authorized.

This policy provides network segmentation and helps prevent guest devices from accessing internal network resources.

## 5. ACL Enforcement

ACLs on the Layer 3 switch enforce the guest VLAN restrictions. The exact ACL name, sequence numbers, direction, and command syntax must be taken from the active switch configuration; they are intentionally not fabricated in this document.

When reviewing or changing the ACL, verify that:

1. The deny rules cover traffic from VLAN 10 to the VLAN 20 and VLAN 30 networks.
2. Required external-bound traffic remains permitted.
3. Any required exceptions are placed before broader deny rules.
4. The ACL is applied to the correct SVI and in the intended direction.
5. The implicit deny at the end of an ACL is accounted for.

**Important:** An ACL that blocks access to VLAN 20 and VLAN 30 should not unintentionally block the guest network's required DNS, DHCP, gateway, or internet access. Test each required service after changing the policy.

## 6. Security Verification

Use the following commands on the C3725:

```text
show vlan-switch brief
show ip interface brief
show running-config
show access-lists
```

Check the ACL attachment and interface configuration in the running configuration:

```text
show running-config interface vlan 10
show running-config interface vlan 20
show running-config interface vlan 30
```

## 7. Recommended Test Matrix

| Test source | Destination | Expected result |
|---|---|---|
| VLAN 10 host | VLAN 10 gateway (`192.168.10.1`) | Allowed |
| VLAN 10 host | VLAN 20 host | Denied |
| VLAN 10 host | VLAN 30 host | Denied |
| VLAN 10 host | Permitted external destination | Allowed |
| VLAN 20 host | VLAN 30 host | Depends on the configured internal policy |
| VLAN 30 host | Permitted external destination | Depends on routing and NAT policy |

Run tests from actual VPCS/host endpoints and record the results. Do not infer a successful policy test solely from the presence of an ACL in the configuration.

## 8. Security Notes

- VLANs provide segmentation, but VLAN separation alone is not a complete security policy.
- ACLs should be reviewed whenever a VLAN, subnet, or permitted service changes.
- Keep management access restricted to authorized devices.
- Do not publish real credentials, shared secrets, or sensitive company network details in the public repository.
- Document any ACL exceptions with their source, destination, protocol, and business purpose.
