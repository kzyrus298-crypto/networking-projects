
# Dual-Stack Network with IPv6 Routing and ACL Security Policy

A Cisco Packet Tracer lab that builds a three-site, dual-stack (IPv4 + IPv6) network, routes it with OSPFv2 and OSPFv3, and enforces a role-based access policy with IPv4 and IPv6 ACLs.

**Tools:** Cisco Packet Tracer · 2911 routers · 2960 switches · OSPFv2 / OSPFv3 · SLAAC · Extended and standard ACLs · IPv6 ACLs

---

## Overview

Three sites (Staff, HQ/Core, Server) are connected in a line. Every LAN and every router-to-router link runs both IPv4 and IPv6. End devices get their IPv6 addresses through SLAAC. A security policy then limits what the Staff LAN and a Contractor PC can reach.

| Site | Router | LAN | Devices |
|---|---|---|---|
| Staff site | R1 (STAFF_SITE) | 192.168.10.0/24, 2001:db8:acad:10::/64 | STAFF PC, CONTRACTOR PC |
| HQ / Core | R2 (HQ/CORE) | 192.168.20.0/24, 2001:db8:acad:20::/64 | ADMIN PC |
| Server site | R3 (SERVER_SITE) | 192.168.30.0/24, 2001:db8:acad:30::/64 | WEB/FTP server |

## Topology

![Topology](images/topology.png)

## Addressing table

| Device | Interface | IPv4 | IPv6 |
|---|---|---|---|
| R1 | G0/0 (to SW1) | 192.168.10.1/24 | 2001:db8:acad:10::1/64 |
| R1 | G0/1 (to R2) | 10.0.12.1/30 | 2001:db8:acad:12::1/64 |
| R2 | G0/0 (to SW2) | 192.168.20.1/24 | 2001:db8:acad:20::1/64 |
| R2 | G0/1 (to R1) | 10.0.12.2/30 | 2001:db8:acad:12::2/64 |
| R2 | G0/2 (to R3) | 10.0.23.1/30 | 2001:db8:acad:23::2/64 |
| R3 | G0/1 (to R2) | 10.0.23.2/30 | 2001:db8:acad:23::1/64 |
| R3 | G0/0 (to SW3) | 192.168.30.1/24 | 2001:db8:acad:30::1/64 |
| STAFF PC | NIC | 192.168.10.10 | SLAAC |
| CONTRACTOR PC | NIC | 192.168.10.20 | SLAAC (2001:db8:acad:10:2e0:b0ff:fe36:e4db) |
| ADMIN PC | NIC | 192.168.20.10 | SLAAC (2001:db8:acad:20:20a:41ff:fe76:8766) |
| WEB/FTP server | NIC | 192.168.30.10 | SLAAC (2001:db8:acad:30:250:fff:fe50:a66b) |


```

There is no default route because the lab has no external network.


### R1: STAFF_POLICY (IPv4)

```
ip access-list extended STAFF_POLICY
 10 permit icmp 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
 20 permit tcp 192.168.10.0 0.0.0.255 host 192.168.30.10 eq www
 25 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
 30 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
 40 permit ip any any
!
interface GigabitEthernet0/0
 ip access-group STAFF_POLICY in
```

The order matters: the HTTP permit (20) must come before the deny to the Server LAN (25), otherwise HTTP would be blocked too.

## STAFF POLICY:
### Requirements


Standard ACL on R3 (BLOCK_CONTRACTOR, IPv6: BLOCK_CONTRACTOR-V6)**
- The Contractor PC (192.168.10.20) must never reach anything in the Server LAN.
- Everyone else must be unaffected by this ACL.
- Standard ACLs match the source address only, so the IPv4 ACL is placed as close to the destination as possible: **R3 G0/0, outbound**.
- IPv6 has no standard ACLs, so BLOCK_CONTRACTOR-V6 is written in the extended style. It denies the Contractor's SLAAC address (2001:db8:acad:10:2e0:b0ff:fe36:e4db) to the whole Server LAN (2001:db8:acad:30::/64) and permits everything else. It is placed on the same interface and direction: **R3 G0/0, outbound**.

Extended ACL on R1 (STAFF_POLICY, IPv6: STAFF_POLICY-V6)**
- The Staff LAN (192.168.10.0/24, 2001:db8:acad:10::/64) may reach the web server over HTTP only (192.168.30.10 on IPv4, 2001:db8:acad:30:250:fff:fe50:a66b on IPv6).
- The Staff LAN may ping the Admin LAN (192.168.20.0/24, 2001:db8:acad:20::/64), and the Admin LAN must be able to ping the Staff LAN and get replies back.
- All other traffic from the Staff LAN to the Server LAN or the Admin LAN is blocked, on both IPv4 and IPv6.
- Everything else (for example, pinging a router interface) is still allowed.
- Extended ACLs match source, destination and protocol, so both are placed as close to the source as possible: **R1 G0/0, inbound**.

### R1: STAFF_POLICY-V6 (IPv6)

```
ipv6 access-list STAFF_POLICY-V6
 permit tcp 2001:DB8:ACAD:10::/64 host 2001:DB8:ACAD:30:250:FFF:FE50:A66B eq www
 permit icmp 2001:DB8:ACAD:10::/64 2001:DB8:ACAD:20::/64
 deny ipv6 2001:DB8:ACAD:10::/64 2001:DB8:ACAD:30::/64
 deny ipv6 2001:DB8:ACAD:10::/64 2001:DB8:ACAD:20::/64
 permit ipv6 any any
!
interface GigabitEthernet0/0
 ipv6 traffic-filter STAFF_POLICY-V6 in
```

### R3: BLOCK_CONTRACTOR (IPv4, standard)

```
ip access-list standard BLOCK_CONTRACTOR
 10 deny host 192.168.10.20
 20 permit any
!
interface GigabitEthernet0/0
 ip access-group BLOCK_CONTRACTOR out
```

### R3: BLOCK_CONTRACTOR-V6 (IPv6)

IPv6 has no standard ACLs, so the source host and the destination (the whole Server LAN) are both written out.

```
ipv6 access-list BLOCK_CONTRACTOR-V6
 deny ipv6 host 2001:DB8:ACAD:10:2E0:B0FF:FE36:E4DB 2001:DB8:ACAD:30::/64
 permit ipv6 any any
!
interface GigabitEthernet0/0
 ipv6 traffic-filter BLOCK_CONTRACTOR-V6 out
```

## Verification

Each result below matched the policy and has a screenshot.

| From | To | Test | Protocol | Expected | Result |
|---|---|---|---|---|---|
| STAFF | Web server | HTTP | IPv4 | Allowed | Page loaded |
| STAFF | Web server | HTTP | IPv6 | Allowed | Page loaded |
| STAFF | Web server | Ping | IPv4 | Blocked | Destination host unreachable |
| STAFF | Web server | Ping | IPv6 | Blocked | Destination host unreachable |
| STAFF | ADMIN | Ping | IPv6 | Allowed | 4/4 replies |
| CONTRACTOR | Web server | HTTP | IPv4 | Blocked | Request timeout |
| CONTRACTOR | Web server | HTTP | IPv6 | Blocked | Request timeout |

### Evidence

| STAFF to web server over HTTP (IPv4) | STAFF to web server over HTTP (IPv6) |
<img width="1917" height="1011" alt="Screenshot 2026-11-10 134529" src="https://github.com/user-attachments/assets/2435595f-bae3-47c5-ab4d-02d8bdc142b6" />

<img width="1917" height="1011" alt="Screenshot 2026-11-10 134529" src="https://github.com/user-attachments/assets/d52b526c-e6e8-49e7-8ad3-08b1ad83e6bc" />




| CONTRACTOR to web server (IPv4) | CONTRACTOR to web server (IPv6) |
<img width="1917" height="1016" alt="Screenshot 2026-11-10 134404" src="https://github.com/user-attachments/assets/accf5bfb-154a-404c-9eb4-b5d1bc9e85a2" />
<img width="1917" height="961" alt="Screenshot 2026-11-10 134551" src="https://github.com/user-attachments/assets/9276a8d6-05d4-40db-9331-4312866034b2" />



STAFF pinging the web server is rejected by R1 itself (the reply comes from R1's LAN address), and pinging the ADMIN PC over IPv6 succeeds:
<img width="702" height="956" alt="Screenshot 2026-11-10 134259" src="https://github.com/user-attachments/assets/c9491afa-0ee3-4124-9eed-04f3be3afb4f" />
<img width="773" height="938" alt="Screenshot 2026-11-10 140339" src="https://github.com/user-attachments/assets/5a9f0c34-9810-47b1-a7f0-92763ef83ef3" />







