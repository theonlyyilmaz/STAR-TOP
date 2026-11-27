# STAR-TOP — Network Design

## 1. Project Overview

STAR-TOP is a segmented enterprise-style network topology implemented in GNS3. The design combines multiple routers, a Layer 3 switch, virtual PCs, separate Cloud connections, dynamic routing, DHCP, NAT/PAT, and access-control policies.

The topology is intended to demonstrate routed connectivity between network segments while maintaining separation between the company network, the lab environment, and guest users.

## 2. Design Objectives

- Build a multi-router topology with redundant or alternative routed paths where supported by the physical design.
- Use OSPF Area 0 for dynamic route exchange between participating routers and Layer 3 devices.
- Separate endpoint groups with VLANs and Layer 3 switching.
- Provide DHCP-based addressing for the lab-side clients.
- Centralize internet-bound NAT/PAT on R2.
- Enforce guest-network isolation so VLAN 10 cannot initiate communication with the other internal VLANs.
- Validate end-to-end connectivity with routing, reachability, and access-control tests.

## 3. High-Level Network Components

| Component | Responsibility |
|---|---|
| Cloud 1 | Connects R2 to the external/upstream network |
| Cloud 2 | Represents the separate company-network connection |
| R2 | Central NAT/PAT router and external-network gateway |
| R1 and R3 | Internal routing between network segments |
| C7200 and C3600 | Additional routers forming the routed transit topology |
| C3725 Layer 3 switch | VLAN gateway interfaces and inter-VLAN routing |
| DHCP Router | Provides DHCP service for the designated lab-side client network |
| VPCS nodes | Simulated end-user devices used for connectivity testing |

## 4. Cloud Connectivity and Network Separation

The topology uses two separate Cloud connections for different purposes:

### Cloud 1 — External Network

Cloud 1 connects to R2. R2's FastEthernet0/0 interface is configured with `10.3.99.17/16` and acts as the NAT outside interface. The upstream gateway recorded for this external network is `10.3.0.1`.

Internet-bound traffic from internal networks is routed toward R2, where NAT/PAT translates the source addresses before forwarding traffic to the external network.

### Cloud 2 — Company Network

Cloud 2 represents the company network and is kept separate from the lab-side Cloud connection. Company-network endpoints use the company-provided addressing scheme. The company endpoint address assignments should be maintained according to the actual network allocation and are not assumed in this document.

## 5. Routing Architecture

The routers are interconnected using point-to-point transit networks from the `10.255.0.0` address space. These links use `/30` subnets to provide two usable IP addresses per point-to-point connection.

OSPF process 1 is used for dynamic routing, with the routed topology operating in Area 0. OSPF advertises the connected routed networks so that participating devices can learn paths dynamically instead of relying entirely on manually configured static routes.

The transit links documented in the IP addressing plan include:

- R2–C7200
- R2–C3600
- R2–R1
- C7200–C3600
- C7200–R3
- C7200–R1
- C3600–R3
- R1–R3
- R3–C3725

The exact interface assignments should be checked against the live GNS3 topology and each device's `show ip interface brief` output.

## 6. VLAN and Layer 3 Switching Design

The C3725 is used as the Layer 3 switch. It hosts the VLAN gateway interfaces and performs inter-VLAN routing.

| VLAN | Network | Gateway | Port allocation | Policy |
|---|---|---|---|---|
| VLAN 10 | `192.168.10.0/24` | `192.168.10.1` | Fa1/0–Fa1/4 | Guest isolation |
| VLAN 20 | `192.168.20.0/24` | `192.168.20.1` | Fa1/5–Fa1/9 | Internal network |
| VLAN 30 | `192.168.30.0/24` | `192.168.30.1` | Fa1/10–Fa1/14 | Internal network |

### Guest VLAN — VLAN 10

VLAN 10 is designed as an isolated guest network. Hosts in this VLAN are allowed to access external networks, including the internet, but are not allowed to communicate with hosts in VLAN 20 or VLAN 30. This policy is enforced using ACLs.

This approach provides network segmentation and reduces the risk of guest devices accessing internal resources.

## 7. DHCP Design

A dedicated DHCP router is connected to R1 through the `192.168.100.0/30` transit network:

- DHCP Router: `192.168.100.1/30`
- R1 FastEthernet2/1: `192.168.100.2/30`

The DHCP router serves the designated lab-side clients. The DHCP service is kept separate from the company-network addressing. The Layer 3 switch VLAN networks can use their own DHCP design independently if required.

The exact DHCP pool ranges and options should be taken from the active DHCP configuration rather than inferred.

## 8. NAT/PAT and Internet Access

R2 is the centralized NAT/PAT device.

- **NAT outside:** R2 FastEthernet0/0 (`10.3.99.17/16`), connected to Cloud 1.
- **NAT inside:** R2's internal-facing interfaces, including the recorded transit and internal-segment interfaces.
- **Upstream gateway:** `10.3.0.1`.

Internal traffic is routed toward R2. R2 translates eligible source addresses using NAT/PAT and forwards the traffic to the external network. Return traffic is translated back and routed to the originating internal network.

The current working project has demonstrated successful reachability from a lab client to `8.8.8.8`.

## 9. Security and Access Control

The design applies segmentation at both Layer 2 and Layer 3:

- VLANs separate endpoint groups into distinct broadcast domains.
- The C3725 provides the VLAN gateway interfaces.
- ACLs restrict VLAN 10 from reaching VLAN 20 and VLAN 30.
- VLAN 10 retains access to external networks as required by the guest-network policy.
- NAT/PAT provides address translation at the external boundary; it should not be treated as a replacement for internal ACL security.

Any required exceptions should be implemented as explicit ACL entries and documented with the permitted source, destination, and protocol.

## 10. Verification and Testing

The following commands are useful for validating the design.

### Interface and IP status

```text
show ip interface brief
show running-config
```

### Routing and OSPF

```text
show ip route
show ip ospf neighbor
show ip protocols
```

### VLAN and access-control status

```text
show vlan-switch brief
show access-lists
```

### End-to-end connectivity

```text
ping <destination-ip>
traceroute <destination-ip>
```

Testing should confirm that:

1. OSPF neighbors form on the intended routed links.
2. Internal routes are learned dynamically.
3. VLAN hosts can reach their configured default gateways.
4. VLAN 10 cannot reach VLAN 20 or VLAN 30.
5. VLAN 10 can reach permitted external destinations.
6. Lab clients can obtain DHCP leases from the designated DHCP router.
7. Lab clients can reach external destinations through R2 and NAT/PAT.

## 11. Design Notes

This document describes the logical design and the configuration facts currently recorded for STAR-TOP. The live GNS3 project and device running configurations remain the authoritative sources for exact interface mappings and any values marked for confirmation.

Before publishing a final revision, reconcile the interface tables with the current device outputs and update any DHCP pool, endpoint, or OSPF details that have changed.