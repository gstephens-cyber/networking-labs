# Lab 01 – VLANs + Inter-VLAN Routing (Router-on-a-Stick)

## Objective

Configure multiple VLANs on a Layer 2 switch and use a router-on-a-stick design to route between three subnets.

## Topology

- Router: Cisco 1941 (R1)
- Switch: Cisco 2960 (SW1)
- PCs: PC-A, PC-B, PC-C

Links:
- PC-A → SW1 Fa0/1
- PC-B → SW1 Fa0/2
- PC-C → SW1 Fa0/3
- SW1 Fa0/24 → R1 G0/0

## IP Addressing

| VLAN | Name         | Subnet           | Gateway        | Host IP         |
|------|--------------|------------------|----------------|-----------------|
| 10   | Engineering  | 192.168.10.0/24  | 192.168.10.1   | PC-A: .10       |
| 20   | Finance      | 192.168.20.0/24  | 192.168.20.1   | PC-B: .10       |
| 30   | HR           | 192.168.30.0/24  | 192.168.30.1   | PC-C: .10       |

PCs:
- PC-A: 192.168.10.10 /24, GW 192.168.10.1  
- PC-B: 192.168.20.10 /24, GW 192.168.20.1  
- PC-C: 192.168.30.10 /24, GW 192.168.30.1  

## Config Summary

**Switch:**
- VLANs 10, 20, 30 created
- Fa0/1 → VLAN 10
- Fa0/2 → VLAN 20
- Fa0/3 → VLAN 30
- Fa0/24 → trunk to R1

**Router (R1):**
- G0/0.10 → 192.168.10.1 /24, dot1Q 10
- G0/0.20 → 192.168.20.1 /24, dot1Q 20
- G0/0.30 → 192.168.30.1 /24, dot1Q 30

## Verification

- All PCs can ping their default gateway.
- PCs in different VLANs can ping each other via the router.

Packet Tracer file for this lab: `vlan-intervlan-routing.pkt`.
