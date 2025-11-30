# Enterprise Multi-Site Network - OSPF, Central DHCP & HTTP

**Technologies:** Cisco IOS, OSPF (single-area), DHCP relay, routed LANs, DNS, HTTP server, STP  
**Tools:** Cisco Packet Tracer

## Goal

Design and implement a small but realistic enterprise network with three routed sites and centralized services. The lab demonstrates IP addressing, dynamic routing and centralized DHCP/DNS/HTTP so that every host - regardless of location - receives configuration automatically, reaches other subnets and can open an internal web portal.

## Topology

### Sites and roles
- **Site A (Branch 1):** user LAN behind R2
- **Site B (HQ):** central routing core and voice access behind R1
- **Site C (Branch 2 + services):** user LAN and infrastructure services behind R3

### Devices
- 3× Cisco 2911 routers: R1 (core/HQ), R2 (Branch 1), R3 (Branch 2 + services)
- 3× switches: S2 (Branch 1 access), S3 (HQ access/voice), S1 (Branch 2 access)
- 1× server in Branch 2: HTTP + DHCP + DNS
- ~15× PCs + 3× IP Phones distributed across the LANs

### Logical links
- `R1 Gi0/1 ⇄ R2 Gi0/1` - WAN link `10.0.12.0/30`
- `R1 Gi0/2 ⇄ R3 Gi0/1` - WAN link `10.0.13.0/30`
- `R2 Gi0/0 ⇄ S2 Gi0/1` - Branch 1 LAN (LAN10)
- `R1 Gi0/0 ⇄ S3 Gi0/1` - HQ LAN (LAN30 - PCs + IP phones)
- `R3 Gi0/0 ⇄ S1 Gi0/1` - Branch 2 LAN (LAN20 - PCs + services server)

(Topology and test views can be exported from Packet Tracer screenshots and saved under `tests/`.)

## IP Plan

### WAN
- `R1–R2:` `10.0.12.0/30` (`R1 Gi0/1 = 10.0.12.1`, `R2 Gi0/1 = 10.0.12.2`)
- `R1–R3:` `10.0.13.0/30` (`R1 Gi0/2 = 10.0.13.1`, `R3 Gi0/1 = 10.0.13.2`)

### LANs
- **LAN10 (Branch 1 users):** `192.168.10.0/24` - gateway `192.168.10.1` (R2 Gi0/0)
- **LAN20 (Branch 2 users + services):** `192.168.20.0/24` - gateway `192.168.20.1` (R3 Gi0/0)
- **LAN30 (HQ users + voice):** `192.168.30.0/24` - gateway `192.168.30.1` (R1 Gi0/0)

### Services
- **Server (LAN20):** `192.168.20.10/24` - HTTP, DHCP, DNS

## Routing - OSPF Design

All routers run single-area OSPF (area 0).

- **R1 (core)**
  - OSPF process 1, router-id `1.1.1.1`
  - Advertises `192.168.30.0/24`, `10.0.12.0/30`, `10.0.13.0/30`
  - `Gi0/0` set as passive

- **R2 (Branch 1)**
  - OSPF process 1, router-id `2.2.2.2`
  - Advertises `192.168.10.0/24`, `10.0.12.0/30`
  - `Gi0/0` passive, `Gi0/1` forms adjacency with R1

- **R3 (Branch 2 + services)**
  - OSPF process 1, router-id `3.3.3.3`
  - Advertises `192.168.20.0/24`, `10.0.13.0/30`
  - `Gi0/0` passive, `Gi0/1` forms adjacency with R1

Result: OSPF distributes routes so each router learns all three /24 LANs and end-to-end connectivity is dynamic.

## DHCP Architecture

Centralized DHCP is hosted on the server in LAN20 and serves all LANs. Routers forward DHCP broadcasts using `ip helper-address` where required.

### Pools
- **LAN10** (`192.168.10.0/24`) - gateway `192.168.10.1`, DNS `192.168.20.10`, dynamic range `192.168.10.50–192.168.10.100`
- **LAN20** (`192.168.20.0/24`) - gateway `192.168.20.1`, DNS `192.168.20.10`, dynamic range `192.168.20.50–192.168.20.100`
- **LAN30** (`192.168.30.0/24`) - gateway `192.168.30.1`, DNS `192.168.20.10`, dynamic range `192.168.30.50–192.168.30.100`

### DHCP relay
- `R2 Gi0/0` → `ip helper-address 192.168.20.10` (for LAN10)
- `R1 Gi0/0` → `ip helper-address 192.168.20.10` (for LAN30)
- `R3` does not need a helper (server is local to LAN20)

All PCs and IP phones are DHCP clients and receive IP, mask, gateway and DNS automatically.

## DNS & HTTP Portal

- DNS: A record `jakuba23.local` → `192.168.20.10`. The server IP is distributed as DNS server via DHCP to all clients.
- HTTP: simple portal available at `http://192.168.20.10` and `http://jakuba23.local` (index.html contains ASCII-art “JaKuba23” and a brief lab description).

## Switching & Access Layer

Each site uses a single access switch connecting local hosts to the router.

- **S2 (Branch 1)**
  - Uplink `Gi0/1` → `R2 Gi0/0`
  - `Fa0/2–Fa0/8` - access ports for PCs
  - STP portfast on access ports

- **S1 (Branch 2)**
  - Uplink `Gi0/1` → `R3 Gi0/0`
  - `Fa0/2–Fa0/8` - access ports for PCs
  - `Fa0/10` - server port

- **S3 (HQ / voice)**
  - Uplink `Gi0/1` → `R1 Gi0/0`
  - Access ports for IP phones and PCs
  - STP portfast on user/phone ports

## Tests

Validation performed in Packet Tracer using `ping` and browser tests:

- **Addressing:** sample hosts in each LAN obtain addresses from the correct DHCP pool and use `192.168.20.10` as DNS.
- **Routing:** successful local and cross-site pings; `show ip route` on routers confirms OSPF-learned routes to all /24 networks.
- **HTTP & DNS:** HTTP accessible by IP and hostname from all sites, name resolution verified with `ping jakuba23.local`.

## Files

- Device configurations: `configs/` (exported Packet Tracer configs)
- Tests and screenshots: `tests/`

## Skills Demonstrated

- IP addressing and multi-site connectivity design
- Single-area OSPF configuration with passive interfaces
- Centralized DHCP with `ip helper-address` relays
- DNS and HTTP integration for internal services
- Simple access-layer switching with STP portfast

---
