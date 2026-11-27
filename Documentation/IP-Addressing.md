# STAR-TOP — IP Addressing Plan

## 1. Purpose

This document describes the IP addressing scheme used in the STAR-TOP GNS3 network topology. It covers the external/company connections, router-to-router transit links, routed VLANs, and the central NAT boundary.

> **Addressing note:** Values below are based on the recorded working configuration and design notes. Where an interface mapping or endpoint address has not been conclusively recorded, it is explicitly marked as **TBD** rather than guessed.

## 2. Addressing Overview

| Network / Segment | Subnet | Purpose |
|---|---|---|
| External / Cloud 1 | `10.3.0.0/16` | External network and upstream gateway |
| R2–C7200 | `10.255.0.0/30` | Point-to-point transit link |
| R2–C3600 | `10.255.0.4/30` | Point-to-point transit link |
| R2–R1 | `10.255.0.8/30` | Point-to-point transit link |
| C7200–C3600 | `10.255.0.12/30` | Point-to-point transit link |
| C7200–R3 | `10.255.0.16/30` | Point-to-point transit link |
| C7200–R1 | `10.255.0.20/30` | Point-to-point transit link |
| C3600–R3 | `10.255.0.24/30` | Point-to-point transit link |
| R1–R3 | `10.255.0.28/30` | Point-to-point transit link |
| R3–C3725 | `10.255.0.32/30` | Point-to-point transit link |
| VLAN 10 | `192.168.10.0/24` | Isolated guest VLAN |
| VLAN 20 | `192.168.20.0/24` | Internal user VLAN |
| VLAN 30 | `192.168.30.0/24` | Internal user VLAN |
| R2 inside segment | `192.168.40.0/24` | Internal routed segment |
| R1 LAN segment | `192.168.50.0/24` | Internal routed segment |
| DHCP-router transit | `192.168.100.0/30` | R1-to-DHCP-router connection |

## 3. Router Interface Addressing

### R2 — Central NAT Router

| Interface | IP address | Connected segment / role |
|---|---|---|
| FastEthernet0/0 | `10.3.99.17/16` | Cloud 1 / external network; NAT outside |
| FastEthernet1/0 | `10.255.0.5/30` | C3600 transit; NAT inside |
| FastEthernet1/1 | `10.255.0.9/30` | R1 transit; NAT inside |
| FastEthernet2/0 | `10.255.0.1/30` | C7200 transit; NAT inside |
| FastEthernet2/1 | `192.168.40.1/24` | Internal routed segment; NAT inside |

Upstream default gateway recorded for the external network: `10.3.0.1`.

### R1

| Interface | IP address | Connected segment |
|---|---|---|
| FastEthernet0/0 | `192.168.50.1/24` | Internal LAN segment |
| FastEthernet1/0 | `10.255.0.10/30` | R2 transit |
| FastEthernet1/1 | `10.255.0.22/30` | C7200 transit |
| FastEthernet2/0 | `10.255.0.30/30` | R3 transit |
| FastEthernet2/1 | `192.168.100.2/30` | DHCP-router transit |

### R3

| Interface | IP address | Connected segment |
|---|---|---|
| FastEthernet0/0 | `10.255.0.33/30` | C3725 Layer 3 switch |
| FastEthernet1/0 | `10.255.0.25/30` | C3600 transit |
| FastEthernet2/0 | `10.255.0.29/30` | R1 transit |
| FastEthernet3/0 | `10.255.0.17/30` | Recorded value; connection/interface mapping requires verification |

**Important validation item:** `10.255.0.17/30` belongs to subnet `10.255.0.16/30`, which is listed as the C7200–R3 transit network. Verify the R3 FastEthernet3/0 cable endpoint and the peer interface before treating this entry as final.

### C7200 and C3600

The transit networks recorded for these routers are:

| Link | Subnet | Usable host addresses |
|---|---|---|
| C7200–C3600 | `10.255.0.12/30` | `10.255.0.13` – `10.255.0.14` |
| C7200–R3 | `10.255.0.16/30` | `10.255.0.17` – `10.255.0.18` |
| C7200–R1 | `10.255.0.20/30` | `10.255.0.21` – `10.255.0.22` |
| C3600–R3 | `10.255.0.24/30` | `10.255.0.25` – `10.255.0.26` |

The exact C7200/C3600 interface-to-address assignments are not fully confirmed in the available configuration notes. Do not infer them from the subnet table; verify them with `show ip interface brief` on each router.

## 4. VLAN Addressing

The C3725 Layer 3 switch provides the default gateway interfaces for the VLANs.

| VLAN | Name | Network | Default gateway | Access ports |
|---|---|---|---|---|
| 10 | VLAN10 / Guest | `192.168.10.0/24` | `192.168.10.1` | Fa1/0–Fa1/4 |
| 20 | VLAN20 | `192.168.20.0/24` | `192.168.20.1` | Fa1/5–Fa1/9 |
| 30 | VLAN30 | `192.168.30.0/24` | `192.168.30.1` | Fa1/10–Fa1/14 |

VLAN 10 is designed as a guest network. It may access external networks, but access to the other internal VLANs is restricted by access-control policy.

## 5. DHCP Router Transit

| Device / interface | IP address | Role |
|---|---|---|
| R1 FastEthernet2/1 | `192.168.100.2/30` | Transit interface toward DHCP router |
| DHCP Router interface | `192.168.100.1/30` | DHCP-router side of the transit link |

The DHCP router provides address assignment for the lab-side client network as designed. DHCP pool network, default-router option, DNS option, and client lease ranges should be documented here once confirmed from the active DHCP configuration.

## 6. Addressing and Verification Commands

Run these commands on the relevant Cisco devices to validate the live configuration:

```text
show ip interface brief
show running-config
show ip route
show ip ospf neighbor
show vlan-switch brief
show access-lists
```

For a specific interface, verify its address and operational state with:

```text
show running-config interface FastEthernet0/0
```

## 7. Items to Confirm Before Treating This Table as Final

- Confirm the peer interface and cable endpoint for R3 FastEthernet3/0 (`10.255.0.17/30`).
- Record the exact C7200 and C3600 interface-to-IP assignments.
- Add the DHCP pool subnet, address range, DNS server, and default gateway.
- Add the individual PC addresses for the company-network clients and lab clients, if static addressing is used.
- Confirm the current R2 NAT inside/outside interface list against `show running-config`.