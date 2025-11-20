# Standard ACL – Block PC-A, Allow PC-B  
**Packet Tracer 9.0.0.0810 – Beginner A–Z Lab**  
**Routers:** Cisco 1941 (with HWIC-2T for serial ports)  
**Switch:** Cisco 2960  
**End Devices:** PC-A, PC-B, Server-PT  

This lab demonstrates how to configure a **standard IPv4 access control list (ACL)** to block traffic from a single host while allowing all other hosts. The goal is:

- **Deny:** PC-A (192.168.10.10) from reaching the server (172.16.0.50)  
- **Allow:** PC-B (192.168.10.20) to reach the server normally  

Standard ACLs filter only on **source IP address**, so proper placement is critical.

---

## Topology

```text
PC-A (192.168.10.10) \
                       \
PC-B (192.168.10.20) ---- 2960 Switch ---- ROUTER1 ---- Serial WAN ---- ROUTER2 ---- Server (172.16.0.50)
ROUTER1: Default gateway for the 192.168.10.0/24 LAN

ROUTER2: Default gateway for the 172.16.0.0/24 server network

Standard ACL is applied on ROUTER1, outbound on the serial interface towards ROUTER2

IP Addressing
LAN (left side)
Network: 192.168.10.0/24

ROUTER1 G0/0: 192.168.10.1

PC-A: 192.168.10.10 (GW: 192.168.10.1)

PC-B: 192.168.10.20 (GW: 192.168.10.1)

WAN / Serial link
Network: 10.0.0.0/30

ROUTER1 S0/0/0: 10.0.0.1

ROUTER2 S0/0/0: 10.0.0.2

Server side
Network: 172.16.0.0/24

ROUTER2 G0/0: 172.16.0.1

Server-PT: 172.16.0.50 (GW: 172.16.0.1)

Cabling
PC-A Fa0 → Switch Fa0/1 (Copper Straight-Through)

PC-B Fa0 → Switch Fa0/2 (Copper Straight-Through)

Switch Fa0/24 → ROUTER1 G0/0 (Copper Straight-Through)

ROUTER1 S0/0/0 → ROUTER2 S0/0/0 (Serial DCE/DTE)

ROUTER2 G0/0 → Server Fa0 (Copper Straight-Through)

Note: 1941 routers require the HWIC-2T module installed to expose serial interfaces (S0/0/0, S0/1/0).

ROUTER1 Configuration (Gateway + Route + ACL)
bash
Copy code
enable
conf t

! LAN interface
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shut
exit

! Serial WAN link to ROUTER2
interface s0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shut
exit

! Static route to server network
ip route 172.16.0.0 255.255.255.0 10.0.0.2

! Standard ACL: block PC-A, allow everyone else
no access-list 10
access-list 10 deny host 192.168.10.10
access-list 10 permit any

! Apply ACL closest to the source (outbound on serial link)
interface s0/0/0
 ip access-group 10 out
exit

end
ROUTER2 Configuration (WAN + Server Network)
bash
Copy code
enable
conf t

! Serial WAN link from ROUTER1
interface s0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shut
exit

! Server-facing LAN
interface g0/0
 ip address 172.16.0.1 255.255.255.0
 no shut
exit

! Route back to client LAN
ip route 192.168.10.0 255.255.255.0 10.0.0.1

end
PC and Server Configuration
PC-A
IP: 192.168.10.10

Mask: 255.255.255.0

Gateway: 192.168.10.1

DNS: 8.8.8.8

PC-B
IP: 192.168.10.20

Mask: 255.255.255.0

Gateway: 192.168.10.1

DNS: 8.8.8.8

Server-PT
IP: 172.16.0.50

Mask: 255.255.255.0

Gateway: 172.16.0.1

DNS: 8.8.8.8

Verification
Baseline (before ACL)

Both PC-A and PC-B should successfully ping the server:

bash
Copy code
ping 172.16.0.50
After ACL is applied on ROUTER1 S0/0/0 outbound:

From PC-A (should FAIL):

bash
Copy code
ping 172.16.0.50
From PC-B (should SUCCEED):

bash
Copy code
ping 172.16.0.50
Check ACL hit counters on ROUTER1:

bash
Copy code
show access-lists 10
Expected:

deny host 192.168.10.10 → has matches

permit any → has matches

This confirms that the standard ACL is correctly blocking PC-A while allowing PC-B, and that it has been placed in the correct direction and on the correct interface.



