Simple Network with Redundancy, EtherChannel, and Firewall Protection
Overview

This lab demonstrates a small enterprise-style network design that focuses on redundancy, link resiliency, and perimeter security. IP addressing was planned using VLSM, and the topology was built to survive a switch or link failure while filtering traffic entering from outside the network.

Topology
3 switches configured for redundant switching paths (backup switch/port in case one path fails)
EtherChannel between switches to bundle multiple physical links into one logical link
Firewall placed at the network edge to inspect inbound traffic from outside

<img width="722" height="645" alt="Screenshot 2026-09-22 085625" src="https://github.com/user-attachments/assets/ab341071-c0be-48c5-8211-92b07de3e870" />




Concepts Covered
VLSM (Variable Length Subnet Masking) — used to plan and assign IP addressing efficiently across the network
Switch Redundancy — a third switch was added to provide a backup path, so if one link or switch goes down, traffic can still reach its destination through the alternate path (this is the kind of scenario STP/RSTP normally manages so the redundant path doesn't cause a loop)
EtherChannel — multiple physical links between switches were bundled into a single logical link, so if one link fails, the connection stays up through the remaining link(s) instead of going down completely
Firewall / Perimeter Security — a firewall was placed between the internal network and the outside connection to inspect incoming traffic (e.g. ping/ICMP requests) and control what is allowed in, protecting the internal network from unsolicited external traffic
Configuration Highlights
IP addressing assigned via VLSM based on host requirements per segment
Switch-to-switch links configured with EtherChannel (Layer 2 link aggregation)
Redundant switch/port paths configured across the 3 switches
Firewall rules configured to inspect and control inbound traffic from outside the network
Key Learnings
How redundant paths protect against a single point of failure at the switch/link level
How EtherChannel prevents a single failed link from taking down connectivity between switches
How a firewall acts as a checkpoint for traffic entering the network, only allowing legitimate/expected requests through
