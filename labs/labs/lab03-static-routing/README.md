✅ STEP 1 — Create the Lab 03 Folder

Go to your GitHub repo:
https://github.com/gstephens-cyber/networking-labs

On the left side folder tree, click the folder:

labs


Click the “Add file” button →
“Create new file”

At the top where it asks for a file name, type:

labs/lab03-static-routing/README.md


This automatically creates the folder and the README.md.

✅ STEP 2 — Paste This Lab 03 README.md Content

Copy-paste the exact text below into the GitHub editor:

Lab 03 – Static Routing Between Two Routers
Objective

Build a simple network with two LANs connected by two routers, and configure static routing to allow full end-to-end communication.

This lab introduces:

Basic router interface configuration

Point-to-point WAN links

Static routing

DHCP for end devices

Verifying routing & connectivity

Topology
PC-A → Switch-1 → Router-1 ↔ Router-2 → Switch-2 → PC-B


Devices Used:

2 × Cisco 1941 Routers

2 × Cisco 2960 Switches

2 × PCs

IP Addressing Plan
LAN A (Left Side)
Device	Interface	IP Address	Mask	Notes
PC-A	Fa0	192.168.10.10	/24	Static
Router-1	G0/0	192.168.10.1	/24	Default gateway
Router-to-Router WAN Link
Device	Interface	IP Address	Mask
Router-1	G0/1	10.0.0.1	/30
Router-2	G0/0	10.0.0.2	/30
LAN B (Right Side)
Device	Interface	IP Address	Mask	Notes
Router-2	G0/1	192.168.20.1	/24	DHCP server
PC-B	Fa0	DHCP	/24	Receives IP dynamically
Step-by-Step Configuration
1. Router-1 Configuration
enable
conf t

interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shut
exit

interface g0/1
 ip address 10.0.0.1 255.255.255.252
 no shut
exit

ip route 192.168.20.0 255.255.255.0 10.0.0.2
end
write

2. Router-2 Configuration
enable
conf t

interface g0/0
 ip address 10.0.0.2 255.255.255.252
 no shut
exit

interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shut
exit

! DHCP for LAN B
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool LAN_B
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
exit

ip route 192.168.10.0 255.255.255.0 10.0.0.1
end
write

3. PC Configuration
PC-A (Static)

Desktop → IP Configuration

IP: 192.168.10.10

Mask: 255.255.255.0

Gateway: 192.168.10.1

PC-B (DHCP)

Desktop → IP Configuration

Select DHCP

Should get 192.168.20.x

4. Verification Commands
On Router-1:
show ip interface brief
show ip route
ping 192.168.20.1

On Router-2:
show ip interface brief
show ip route
ping 192.168.10.1

From PC-A:
ping 192.168.10.1
ping 10.0.0.1
ping 10.0.0.2
ping 192.168.20.1
ping <PC-B IP>

Result

Full end-to-end connectivity between PC-A and PC-B using static routing.
