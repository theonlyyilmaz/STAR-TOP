# STAR-TOP — OSPF Configuration

## 1. Overview

STAR-TOP uses Open Shortest Path First (OSPF) as its dynamic routing protocol. OSPF allows participating routers and the Layer 3 switching device to exchange routing information automatically, reducing the need for manually maintained static routes.

The documented design uses OSPF process ID **1** and backbone **Area 0**.

> **Configuration accuracy:** This document describes the recorded design and verified project behavior. Exact interface-to-neighbor mappings and router-specific OSPF commands should be checked against the current GNS3 device configurations before being treated as a complete configuration backup.

## 2. OSPF Design

- Routing protocol: OSPF
- Process ID: `1`
- Area: `0` (backbone)
- Transit address space: `10.255.0.0`
- Point-to-point subnet size: `/30` (`255.255.255.252`)

The `/30` transit networks provide two usable host addresses for each point-to-point router link.

## 3. Recorded Transit Networks

| Transit link | Subnet |
|---|---|
| R2–C7200 | `10.255.0.0/30` |
| R2–C3600 | `10.255.0.4/30` |
| R2–R1 | `10.255.0.8/30` |
| C7200–C3600 | `10.255.0.12/30` |
| C7200–R3 | `10.255.0.16/30` |
| C7200–R1 | `10.255.0.20/30` |
| C3600–R3 | `10.255.0.24/30` |
| R1–R3 | `10.255.0.28/30` |
| R3–C3725 | `10.255.0.32/30` |

Confirm that each listed link is present and addressed as shown in the current topology. Do not add a `network` statement for a subnet unless an interface on that device actually has an address in that subnet.

## 4. Cisco IOS Configuration Pattern

The following is a configuration pattern, not a complete copy-and-paste configuration for every router. Apply only the `network` statements that match the IP addresses configured on the specific device's interfaces.

```text
configure terminal
router ospf 1
 network 10.255.0.0 0.0.0.3 area 0
end
```

For example, the wildcard mask `0.0.0.3` matches the four-address range covered by a `/30` subnet. The `network` command enables OSPF on interfaces whose IP addresses fall within the specified range and advertises the connected network into the stated OSPF area.

Additional `network` statements should be added for the other directly connected transit subnets and any LAN/VLAN networks that are intended to participate in OSPF.

## 5. Layer 3 Switch Participation

The C3725 Layer 3 switch provides the default gateways for the configured VLANs:

| VLAN | Network | SVI gateway |
|---|---|---|
| VLAN 10 | `192.168.10.0/24` | `192.168.10.1` |
| VLAN 20 | `192.168.20.0/24` | `192.168.20.1` |
| VLAN 30 | `192.168.30.0/24` | `192.168.30.1` |

If these networks are to be advertised through OSPF, the OSPF process on the Layer 3 switch must include the relevant connected interfaces/networks. Confirm the actual OSPF configuration and adjacency on the C3725 before assuming all VLAN routes are being advertised.

## 6. Verification

Run the following commands on each participating router or Layer 3 device.

### Check OSPF neighbors

```text
show ip ospf neighbor
```

### Check OSPF-enabled interfaces and process details

```text
show ip protocols
show ip ospf interface
```

### Check learned and installed routes

```text
show ip route
show ip route ospf
```

An OSPF-learned route is commonly marked with the route code `O` in the routing table.

## 7. Troubleshooting Checklist

If an expected OSPF neighbor does not form, check:

1. Both interfaces on the link have IP addresses in the same subnet.
2. The interfaces are administratively enabled with `no shutdown`.
3. The physical/virtual link is up.
4. OSPF is enabled on both interfaces.
5. Both sides use the intended OSPF area.
6. OSPF hello/dead timers and authentication settings are compatible, if customized.
7. The correct interfaces and networks are included in the OSPF configuration.

If a route is missing, verify that the destination network is connected to an OSPF-enabled interface and is being advertised by the correct device.

## 8. Project-Specific Validation Items

Before publishing a final configuration reference:

- Capture `show ip ospf neighbor` from each OSPF router.
- Capture `show ip route` from representative routers and the C3725.
- Confirm the exact OSPF `network` statements on each device.
- Confirm which VLAN interfaces are advertised into OSPF.
- Confirm whether any interfaces are configured as passive interfaces.
- Reconcile the transit-link table with the final GNS3 topology.
