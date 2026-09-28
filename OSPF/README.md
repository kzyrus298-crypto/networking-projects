# Multi-VLAN Enterprise Network with Firewall & Internet Simulation

## Overview
This project simulates a small enterprise network built in Cisco Packet Tracer, featuring three departmental VLANs, inter-router connectivity via OSPF, and a Cisco ASA 5505 firewall providing NAT and perimeter security toward a simulated ISP/internet connection.



## Topology Summary
The network consists of three edge routers (R1, R2, R3), each serving a dedicated department through an access-layer switch:

| Router | VLAN | Department | Network |
|--------|------|------------|---------|
| R1 | VLAN 10 | IT | 192.168.1.0/24 |
| R2 | VLAN 20 | HR | 192.168.2.0/24 |
| R3 | VLAN 30 | Sales | 192.168.3.0/24 |

The routers are interconnected via point-to-point /30 links (192.168.100.0/30, 192.168.100.4/30, 192.168.100.8/30), with **OSPF** handling dynamic routing between R1, R2, and R3.

R3 connects to a **Cisco ASA 5505** firewall, which separates the internal network (inside) from a simulated external network (outside) represented by a router acting as an ISP, using a public-style addressing scheme (200.0.113.0/30).

## Firewall Configuration Highlights
- **Inside interface (VLAN1):** 192.168.100.10/30 — security-level 100
- **Outside interface (VLAN2):** 200.0.113.1/30 — security-level 0
- **NAT:** Dynamic PAT (interface overload) configured per-department, allowing all three internal networks (IT, HR, Sales) to reach the outside network
- **Static routing:** Explicit routes toward each internal VLAN and each transit /30 subnet, since the ASA has no dynamic routing protocol configured
- **Default route:** Points to the simulated ISP for all unmatched traffic
- **Inspection policy:** ICMP inspection enabled via Modular Policy Framework (MPF) to allow return traffic for ping tests to traverse the firewall correctly

## Key Learning Points
- Physical switch ports on the ASA 5505 must be explicitly assigned to a VLAN (`switchport access vlan`) before the corresponding VLAN interface becomes usable — even VLAN 1, though this assignment does not always appear in `show running-config` since it matches the default.
- Static routes are required on the ASA for **every** subnet it doesn't directly own, including transit links between routers — NAT alone does not provide reachability.
- Stale ARP cache entries can cause intermittent or misleading connectivity issues after a routing change; clearing the affected interface (or reloading the device) resolves this in Packet Tracer.
- ICMP to a router or firewall's own interface is not required to validate a working network — the meaningful test is end-to-end connectivity between hosts and their destination.

## Tools Used
- Cisco Packet Tracer
- OSPF (inter-router routing)
- Cisco ASA 5505 (NAT, static routing, MPF/ICMP inspection)
