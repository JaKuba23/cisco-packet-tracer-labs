# Enterprise LAN – VLAN, Inter-VLAN Routing & DHCP

Technologies: Cisco IOS, VLAN, 802.1Q, Router-on-a-Stick, OSPF (single-area), DHCP, STP, Port-Security  
Tools: Cisco Packet Tracer

## Goal

Design and implement a small enterprise LAN with VLAN segmentation (HR, IT, Users, Guests), inter-VLAN routing on a router-on-a-stick, centralized DHCP per VLAN and basic Layer 2 hardening.  

The lab is built as a practical, CCNA-level example for junior network / security roles.

## Topology

Devices:
- 1× Router R1 (Cisco 1941)
- 3× Switch 2960: S1 (core), S2 (access), S3 (access)
- 4× PCs: PC1–PC4 in separate VLANs

Links (logical):
- R1 Gi0/0 ⇄ S1 Gi0/1 (802.1Q trunk)
- S1 Gi0/2 ⇄ S2 Gi0/1 (trunk)
- S1 Fa0/24 ⇄ S3 Gi0/2 (trunk)
- PC1 → S1 Fa0/1 (VLAN 10 – HR)
- PC2 → S1 Fa0/2 (VLAN 20 – IT)
- PC3 → S2 Fa0/1 (VLAN 30 – USERS)
- PC4 → S3 Fa0/1 (VLAN 40 – GUESTS)

Topology and test view: see `tests/01-connectivity-overview.png`.

## IP Plan

- VLAN 10 (HR):    10.10.10.0/24, gateway 10.10.10.1 (R1 Gi0/0.10)  
- VLAN 20 (IT):    10.10.20.0/24, gateway 10.10.20.1 (R1 Gi0/0.20)  
- VLAN 30 (USERS): 10.10.30.0/24, gateway 10.10.30.1 (R1 Gi0/0.30)  
- VLAN 40 (GUESTS):10.10.40.0/24, gateway 10.10.40.1 (R1 Gi0/0.40)  

DHCP pools on R1 allocate addresses dynamically in each VLAN with the first 10 addresses excluded for infrastructure.

## Key Configuration Highlights

R1:
- Router-on-a-stick on Gi0/0 with subinterfaces 10/20/30/40 (802.1Q encapsulation).  
- DHCP server with separate pools per VLAN and excluded infrastructure addresses.  
- OSPF process 1 advertising all 10.10.x.0/24 VLAN networks (single-area 0).

S1 (core):
- VLAN 10/20/30/40 defined with names HR, IT, USERS, GUESTS.  
- Trunk links to R1, S2 and S3 carrying VLANs 10,20,30,40.  
- Access ports Fa0/1 (PC1) in VLAN10 and Fa0/2 (PC2) in VLAN20.  
- Port-Security on access ports (max 2 MAC, sticky, restrict).

S2 / S3 (access):
- S2: VLAN30, trunk to S1, Fa0/1 as access port for PC3 with port-security.  
- S3: VLAN40, trunk to S1, Fa0/1 as access port for PC4 with port-security.

Full device configurations are available in:
- `configs/R1.cfg`
- `configs/S1.cfg`
- `configs/S2.cfg`
- `configs/S3.cfg`

## Tests

Connectivity was verified using ICMP between VLANs and to the default gateways:

- All PCs obtain addresses via DHCP:
  - PC1 → 10.10.10.x / gw 10.10.10.1  
  - PC2 → 10.10.20.x / gw 10.10.20.1  
  - PC3 → 10.10.30.x / gw 10.10.30.1  
  - PC4 → 10.10.40.x / gw 10.10.40.1  

- Successful pings:
  - PC1 → 10.10.10.1, 10.10.40.1 and PC4 IP  
  - PC3 → 10.10.30.1, 10.10.40.1 and PC4 IP  

Screenshots of the topology and ping tests are stored in `tests/`.

## Skills Demonstrated

- VLAN design and access/trunk port configuration on Cisco switches  
- Inter-VLAN routing using router-on-a-stick  
- DHCP server configuration for multiple VLANs  
- Single-area OSPF configuration and verification  
- Layer 2 hardening with Port-Security
