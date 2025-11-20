# Lab 03 – DHCP Server & DHCP Relay  
**Networking Fundamentals – Packet Tracer Lab**

This lab demonstrates how to configure a Cisco router as a DHCP Server and configure a second router as a DHCP Relay Agent. The goal is to dynamically assign IP addressing to two separate LANs while relaying DHCP broadcasts across routers.

---

## 🧱 Topology Overview

- **Router-1**  
  - DHCP Server  
  - LAN Network: `192.168.10.0/24`  
  - Interface g0/0: `192.168.10.1/24`  
  - Interface g0/1: `10.0.0.1/30`

- **Router-2**  
  - DHCP Relay Agent  
  - LAN Network: `192.168.20.0/24`  
  - Interface g0/0: `192.168.20.1/24`  
  - Interface g0/1: `10.0.0.2/30`  
  - DHCP Relay: `ip helper-address 192.168.10.1`

- **PC-A**  
  - DHCP Client on Router-1 LAN

- **PC-B**  
  - DHCP Client on Router-2 LAN (via relay)

---

## 📘 Lab Objective

1. Configure Router-1 as a DHCP server with two pools:
   - `LAN10` → 192.168.10.0/24
   - `LAN20` → 192.168.20.0/24

2. Configure Router-2 as a DHCP relay using:
ip helper-address 192.168.10.1

yaml
Copy code

3. Test DHCP address assignment on both PCs.

4. Verify routing between all subnets.

---

# A–Z SETUP INSTRUCTIONS

## A) Build the topology
1. Add:
- Router-1
- Router-2
- Switch-1
- Switch-2
- PC-A (connect to Switch-1)
- PC-B (connect to Switch-2)

2. Cable the devices:
- Router-1 g0/0 → Switch-1
- Router-2 g0/0 → Switch-2
- Router-1 g0/1 → Router-2 g0/1

---

## B) Configure Router-1 interfaces

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

yaml
Copy code

---

## C) Configure DHCP exclusions on Router-1

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10

yaml
Copy code

---

## D) Create DHCP Pools on Router-1

ip dhcp pool LAN10
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
exit

ip dhcp pool LAN20
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit

yaml
Copy code

---

## E) Add routing on Router-1

ip route 192.168.20.0 255.255.255.0 10.0.0.2

yaml
Copy code

---

## F) Configure Router-2 interfaces

enable
conf t

interface g0/0
ip address 192.168.20.1 255.255.255.0
ip helper-address 192.168.10.1
no shut
exit

interface g0/1
ip address 10.0.0.2 255.255.255.252
no shut
exit

yaml
Copy code

---

## G) Add routing on Router-2

ip route 192.168.10.0 255.255.255.0 10.0.0.1

yaml
Copy code

---

## H) Configure PCs

### PC-A
- Set to **DHCP**
- Should receive:  
  - IP: `192.168.10.x`  
  - Gateway: `192.168.10.1`

### PC-B
- Set to **DHCP**
- Should receive:  
  - IP: `192.168.20.x`  
  - Gateway: `192.168.20.1`

---

## I) Verification Commands

### Router-1
show ip dhcp pool
show ip dhcp binding
show ip route

shell
Copy code

### Router-2
show ip interface brief
show ip route

shell
Copy code

### PCs
ipconfig (Packet Tracer)

yaml
Copy code

---

## J) Expected Results

- PC-A → IP in 192.168.10.x  
- PC-B → IP in 192.168.20.x (via DHCP relay)  
- Full connectivity between all subnets  
- Router-1 shows **two** DHCP bindings

---

## License
Free to use, study, and modify.
