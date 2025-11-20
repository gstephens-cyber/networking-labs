# Extended ACL – HTTP Block for One Host, Allowed for Others  
**Packet Tracer 9.0.0.0810 – Beginner A–Z Lab**  
**Routers:** Cisco 1941 (with HWIC-2T serial module)  
**Switch:** Cisco 2960  
**End Devices:** PC-A, PC-B, Server-PT  

This lab demonstrates how to configure an **extended IPv4 access control list (ACL)** to control traffic based on **source**, **destination**, and **TCP port**.

### Lab Goal

- ❌ Block **PC-A (192.168.20.10)** from accessing the **web server (172.30.0.50)** using **HTTP (TCP/80)**  
- ✅ Allow PC-A to still **ping** the server  
- ✅ Allow **PC-B (192.168.20.20)** full access (ping + HTTP)

Extended ACL used: **ACL 101**, applied inbound on ROUTER1’s LAN interface (G0/0).

---

## Topology

```text
PC-A (192.168.20.10) \
                       \
PC-B (192.168.20.20) ---- 2960 Switch ---- ROUTER1 ---- Serial WAN ---- ROUTER2 ---- Server (172.30.0.50)
ROUTER1: Default gateway for the 192.168.20.0/24 LAN

ROUTER2: Default gateway for the 172.30.0.0/24 server network

Extended ACL 101 is applied on ROUTER1 G0/0 INBOUND, closest to the source

IP Addressing
LAN (left side)
Network: 192.168.20.0/24

ROUTER1 G0/0: 192.168.20.1

PC-A: 192.168.20.10 (GW: 192.168.20.1)

PC-B: 192.168.20.20 (GW: 192.168.20.1)

WAN / Serial link
Network: 10.10.20.0/30

ROUTER1 S0/0/0: 10.10.20.1

ROUTER2 S0/0/0: 10.10.20.2

Server side
Network: 172.30.0.0/24

ROUTER2 G0/0: 172.30.0.1

Server-PT: 172.30.0.50 (GW: 172.30.0.1)

Cabling
PC-A Fa0 → Switch Fa0/1 (Copper Straight-Through)

PC-B Fa0 → Switch Fa0/2 (Copper Straight-Through)

Switch Fa0/24 → ROUTER1 G0/0 (Copper Straight-Through)

ROUTER1 S0/0/0 → ROUTER2 S0/0/0 (Serial DCE/DTE cable)

ROUTER2 G0/0 → Server Fa0 (Copper Straight-Through)

Note: Cisco 1941 routers require the HWIC-2T module installed to expose serial interfaces (S0/0/0, S0/0/1).

ROUTER1 Configuration (Gateway + Route + Extended ACL)
bash
Copy code
enable
conf t

! LAN interface
interface g0/0
 ip address 192.168.20.1 255.255.255.0
 no shut
exit

! Serial WAN link to ROUTER2
interface s0/0/0
 ip address 10.10.20.1 255.255.255.252
 clock rate 64000
 no shut
exit

! Static route to server network
ip route 172.30.0.0 255.255.255.0 10.10.20.2

! Extended ACL 101:
! - Deny HTTP from PC-A to Server (TCP/80)
! - Permit all other IP traffic
no access-list 101
access-list 101 deny tcp host 192.168.20.10 host 172.30.0.50 eq 80
access-list 101 permit ip any any

! Apply ACL closest to the source – inbound on LAN interface
interface g0/0
 ip access-group 101 in
exit

end
If Packet Tracer requires wildcard masks instead of host, this version is equivalent:

bash
Copy code
access-list 101 deny tcp 192.168.20.10 0.0.0.0 172.30.0.50 0.0.0.0 eq 80
access-list 101 permit ip any any
ROUTER2 Configuration (WAN + Server LAN + Route Back)
bash
Copy code
enable
conf t

! Serial WAN link from ROUTER1
interface s0/0/0
 ip address 10.10.20.2 255.255.255.252
 no shut
exit

! Server-side LAN
interface g0/0
 ip address 172.30.0.1 255.255.255.0
 no shut
exit

! Route back to client LAN
ip route 192.168.20.0 255.255.255.0 10.10.20.1

end
PC Configuration
PC-A
IP: 192.168.20.10

Mask: 255.255.255.0

Gateway: 192.168.20.1

DNS: 8.8.8.8

PC-B
IP: 192.168.20.20

Mask: 255.255.255.0

Gateway: 192.168.20.1

DNS: 8.8.8.8

Server Configuration
IP: 172.30.0.50

Mask: 255.255.255.0

Gateway: 172.30.0.1

DNS: 8.8.8.8

On the server:

Go to Services tab

Ensure HTTP = ON (web server enabled)

Verification
Baseline connectivity
Before applying ACL 101, both PCs should be able to:

bash
Copy code
ping 172.30.0.50
and open:

text
Copy code
http://172.30.0.50
After ACL 101 is applied on ROUTER1 G0/0 inbound:

From PC-A:

Ping test:

bash
Copy code
ping 172.30.0.50
✅ Should succeed

HTTP test: open browser → http://172.30.0.50
❌ Should fail (no page load)

From PC-B:

Ping test:

bash
Copy code
ping 172.30.0.50
✅ Should succeed

HTTP test: open browser → http://172.30.0.50
✅ Should succeed

Check ACL hit counters on ROUTER1:

bash
Copy code
show access-lists 101
Expected:

deny tcp host 192.168.20.10 host 172.30.0.50 eq 80 → has matches

permit ip any any → has matches

This confirms extended ACL 101 is correctly blocking only HTTP from PC-A while allowing other traffic and all traffic from PC-B.
