# STAR-TOP — NAT and DHCP Configuration

## 1. Overview

STAR-TOP uses Network Address Translation (NAT/PAT) to provide external network access for eligible internal clients. A dedicated DHCP router provides dynamic IP addressing to the designated lab-side client network.

The NAT and DHCP functions have separate roles:

- **NAT/PAT:** translates eligible internal source addresses when traffic exits through the external network connection.
- **DHCP:** assigns IP configuration to clients in the designated lab-side network.

## 2. NAT/PAT Design

R2 is the central NAT router in the recorded design.

| Item | Recorded configuration |
|---|---|
| NAT router | R2 |
| External interface | FastEthernet0/0 |
| External interface address | `10.3.99.17/16` |
| External network | Cloud 1 |
| Upstream gateway | `10.3.0.1` |
| NAT role of external interface | `ip nat outside` |

R2's internal-facing interfaces are intended to be configured as NAT inside interfaces. Verify the exact interface list against the current running configuration before making changes.

### Traffic flow

1. A lab client sends traffic to an external destination.
2. The traffic is routed through the internal network toward R2.
3. R2 applies NAT/PAT to eligible traffic.
4. The translated traffic exits through the external interface toward the upstream gateway.
5. Return traffic reaches R2 and is translated back to the original internal address.

## 3. NAT Verification

Use the following commands on R2:

```text
show running-config
show ip nat translations
show ip nat statistics
show ip route
```

A successful client ping to an external IP address, such as `8.8.8.8`, is useful evidence of routed external connectivity. NAT translation entries and counters provide additional confirmation that address translation is occurring.

## 4. DHCP Design

A dedicated DHCP router is connected directly to R1. The DHCP router is intended to serve the designated lab-side clients, while the company network remains separate.

The DHCP router's service should be scoped to the intended client subnet only. The exact DHCP pool network, address range, excluded addresses, default gateway, and DNS option must be documented from the active DHCP configuration.

### DHCP information to document

| DHCP parameter | Configured value |
|---|---|
| DHCP server device | Dedicated DHCP router |
| Client network | Verify from active DHCP pool |
| Address pool range | Verify from active DHCP pool |
| Default gateway option | Verify from active DHCP pool |
| DNS server option | Verify from active DHCP pool |
| Excluded/reserved addresses | Verify from active DHCP configuration |

These values are deliberately left for verification rather than guessed.

## 5. DHCP Verification

On the DHCP router, inspect the running configuration and active leases:

```text
show running-config
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
```

On a Cisco IOS client-facing router, verify interface status and DHCP relay configuration if relay is required:

```text
show ip interface brief
show running-config
```

On a VPCS client, renew or request an address using the VPCS DHCP command:

```text
ip dhcp
show ip
```

The client should receive an address from the intended pool, together with the correct gateway and DNS information.

## 6. DHCP and VLAN Considerations

The dedicated DHCP router serves the designated lab-side client network. The VLAN networks on the C3725 are a separate Layer 3 segment and may use an independent DHCP service if required.

If DHCP clients and their DHCP server are on different IP subnets, a DHCP relay configuration (for example, `ip helper-address`) may be required on the client VLAN gateway. Configure this only where the actual topology requires it.

## 7. End-to-End Validation

Use the following test sequence:

1. Confirm that the client receives an address from the intended DHCP pool.
2. Confirm that the client can reach its default gateway.
3. Confirm that internal routes are present on the routers.
4. Confirm that the client can reach R2 through the routed topology.
5. Confirm that R2 has an active NAT translation when the client sends external traffic.
6. Confirm that the client can reach a permitted external IP address.
7. Confirm that the VLAN 10 guest restrictions remain effective after NAT and routing changes.

Useful commands include:

```text
show ip route
show ip nat translations
show ip nat statistics
show ip dhcp binding
ping <destination-ip>
traceroute <destination-ip>
```

## 8. Items to Confirm Before Final Publication

- Record every R2 interface configured with `ip nat inside`.
- Confirm the NAT overload rule and its ACL or route-map, if used.
- Confirm the default route on R2 toward the upstream gateway.
- Record the exact DHCP client subnet and pool range.
- Record the DHCP default-router and DNS options.
- Confirm whether DHCP relay is needed on any VLAN interface.
- Capture successful DHCP lease and NAT translation outputs for the project screenshots.

## 9. Security Notes

- Do not publish real company credentials, shared secrets, or sensitive addressing information.
- Restrict DHCP pools to the intended client networks.
- Reserve or exclude gateway and infrastructure addresses from dynamic allocation.
- Review NAT rules and ACLs together so that translation does not unintentionally bypass the intended security policy.
