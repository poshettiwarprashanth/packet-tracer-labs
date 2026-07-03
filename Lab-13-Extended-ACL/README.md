# NetSec Lab 13 – Extended Access Control Lists (ACL)

## Objective

The objective of this lab was to understand and implement Extended Access Control Lists (ACLs) in Cisco Packet Tracer.

Unlike Standard ACLs, Extended ACLs can filter traffic based on:

* Source IP Address
* Destination IP Address
* Protocol Type
* TCP/UDP Port Numbers

This lab focused on learning ACL logic through multiple real-world security scenarios.

---

# Network Topology

The topology consisted of 4 Routers, 4 Switches, 6 PCs, and 3 Servers.

## Department Networks

### Sales Network

Network:

```text
192.168.10.0/24
```

Gateway:

```text
192.168.10.1
```

Devices:

```text
PC1 - 192.168.10.10
PC2 - 192.168.10.20
```

---

### HR Network

Network:

```text
192.168.20.0/24
```

Gateway:

```text
192.168.20.1
```

Devices:

```text
PC3 - 192.168.20.10
PC4 - 192.168.20.20
```

---

### Admin Network

Network:

```text
192.168.30.0/24
```

Gateway:

```text
192.168.30.1
```

Devices:

```text
PC5 - 192.168.30.10
PC6 - 192.168.30.20
```

---

### Server Farm Network

Network:

```text
192.168.100.0/24
```

Gateway:

```text
192.168.100.1
```

Devices:

```text
Web Server  - 192.168.100.10
DNS Server  - 192.168.100.20
FTP Server  - 192.168.100.30
```

---

# Router Interconnections

```text
R1 -------- R2 -------- R3
              |
              |
             R4
```

Serial Networks:

```text
R1-R2 : 10.0.12.0/30
R2-R3 : 10.0.23.0/30
R2-R4 : 10.0.24.0/30
```

---

# Router Configuration

## Router 1

```bash
interface g0/0
ip address 192.168.10.1 255.255.255.0
no shutdown

interface s0/0/0
ip address 10.0.12.1 255.255.255.252
clock rate 64000
no shutdown
```

---

## Router 2

```bash
interface g0/0
ip address 192.168.20.1 255.255.255.0
no shutdown

interface s0/0/0
ip address 10.0.12.2 255.255.255.252
no shutdown

interface s0/0/1
ip address 10.0.23.1 255.255.255.252
no shutdown

interface s0/1/0
ip address 10.0.24.1 255.255.255.252
no shutdown
```

---

## Router 3

```bash
interface g0/0
ip address 192.168.30.1 255.255.255.0
no shutdown

interface s0/0/0
ip address 10.0.23.2 255.255.255.252
no shutdown
```

---

## Router 4

```bash
interface g0/0
ip address 192.168.100.1 255.255.255.0
no shutdown

interface s0/0/1
ip address 10.0.24.2 255.255.255.252
no shutdown
```

---

# RIP Version 2 Configuration

## Router 1

```bash
router rip
version 2
no auto-summary
network 10.0.0.0
network 192.168.10.0
```

## Router 2

```bash
router rip
version 2
no auto-summary
network 10.0.0.0
network 192.168.20.0
```

## Router 3

```bash
router rip
version 2
no auto-summary
network 10.0.0.0
network 192.168.30.0
```

## Router 4

```bash
router rip
version 2
no auto-summary
network 10.0.0.0
network 192.168.100.0
```

---

# Concepts Learned

## Standard ACL vs Extended ACL

### Standard ACL

Filters traffic based only on:

* Source IP Address

Example:

```bash
access-list 10 deny 192.168.10.0 0.0.0.255
```

### Extended ACL

Filters traffic based on:

* Protocol
* Source IP
* Destination IP
* Port Number

Example:

```bash
access-list 100 deny tcp 192.168.20.0 0.0.0.255 host 192.168.100.10 eq 80
```

---

# Important ACL Rules Learned

## Rule 1 - First Match Wins

ACLs are processed from top to bottom.

Once a packet matches a rule:

* Action is taken
* ACL processing stops

Example:

```bash
access-list 100 deny icmp any any
access-list 100 permit ip any any
```

---

## Rule 2 - Implicit Deny

Every ACL contains an invisible final statement:

```bash
deny ip any any
```

Any packet not matching a configured rule is automatically dropped.

---

## Rule 3 - Extended ACL Placement

Extended ACLs should be placed:

```text
Close to the Source
```

to stop unwanted traffic as early as possible.

---

## Rule 4 - Wildcard Masks

Entire Network:

```bash
192.168.20.0 0.0.0.255
```

Single Host:

```bash
host 192.168.100.10
```

Equivalent:

```bash
192.168.100.10 0.0.0.0
```

---

# Scenario 1 - Block Access to FTP Server

## Requirement

Sales users should not access the FTP Server.

Source:

```text
192.168.10.0/24
```

Destination:

```text
192.168.100.30
```

Protocol:

```text
IP
```

ACL:

```bash
access-list 100 deny ip 192.168.10.0 0.0.0.255 host 192.168.100.30
access-list 100 permit ip any any
```

Applied On:

```text
Router1
G0/0
Inbound
```

Result:

* FTP Server Blocked
* Other Traffic Allowed

---

# Scenario 2 - Block Ping to Web Server

## Requirement

HR users should not ping the Web Server.

Source:

```text
192.168.20.0/24
```

Destination:

```text
192.168.100.10
```

Protocol:

```text
ICMP
```

ACL:

```bash
access-list 100 deny icmp 192.168.20.0 0.0.0.255 host 192.168.100.10
access-list 100 permit ip any any
```

Applied On:

```text
Router2
G0/0
Inbound
```

Result:

* Ping Failed
* HTTP Successful
* HTTPS Successful

---

# Scenario 3 - Block HTTP Access

## Requirement

HR users should not access the Web Server using HTTP.

Source:

```text
192.168.20.0/24
```

Destination:

```text
192.168.100.10
```

Protocol:

```text
TCP
```

Port:

```text
80
```

ACL:

```bash
access-list 101 deny tcp 192.168.20.0 0.0.0.255 host 192.168.100.10 eq 80
access-list 101 permit ip any any
```

Applied On:

```text
Router2
G0/0
Inbound
```

Testing:

```text
ping 192.168.100.10
```

Result:

```text
Success
```

Testing:

```text
http://192.168.100.10
```

Result:

```text
Blocked
```

Testing:

```text
https://192.168.100.10
```

Result:

```text
Success
```

Learning Outcome:

Learned how Extended ACLs filter traffic using TCP Port Numbers.

---

# Scenario 4 - Allow SSH and Block Telnet

## Requirement

HR users should:

* Access SSH
* Be blocked from Telnet

Source:

```text
192.168.20.0/24
```

Destination:

```text
192.168.100.10
```

Protocol:

```text
TCP
```

SSH Port:

```text
22
```

Telnet Port:

```text
23
```

ACL:

```bash
access-list 100 deny tcp 192.168.20.0 0.0.0.255 host 192.168.100.10 eq 23

access-list 100 permit tcp 192.168.20.0 0.0.0.255 host 192.168.100.10 eq 22

access-list 100 permit ip any any
```

Learning Outcome:

* Learned Port 22 (SSH)
* Learned Port 23 (Telnet)
* Learned ACL ordering
* Learned First Match Wins concept

Note:

My Cisco Packet Tracer version does not provide SSH and Telnet services on Server-PT devices.

Therefore:

```text
Scenario 4 was completed conceptually.
Practical testing will be performed later in a dedicated SSH and Telnet Lab.
```

---

# Challenges Faced

## Challenge 1 - Wrong Wildcard Mask

Configured:

```bash
0.0.0.0
```

Instead of:

```bash
0.0.0.255
```

Result:

ACL matched only one host instead of the entire network.

Fix:

```bash
192.168.20.0 0.0.0.255
```

---

## Challenge 2 - Incorrect Destination IP

Mistakenly used:

```bash
192.168.100.1
```

Instead of:

```bash
192.168.100.10
```

Result:

ACL never matched the intended traffic.

Fix:

Corrected destination address.

---

## Challenge 3 - Existing ACL Interference

An old ICMP ACL was still active.

Result:

Testing became confusing because traffic was already being filtered.

Fix:

Created separate Packet Tracer files for individual ACL scenarios.

---

## Challenge 4 - RIP Route Learning Issue

Router2 initially learned only one route due to incorrect addressing and subnetting mistakes.

Fix:

Verified interface IP addresses and corrected network configuration.

---

## Challenge 5 - Understanding First Match Wins

Initially assumed every ACL statement was checked.

Learned:

```text
ACL Processing

Top → Bottom

First Match → Stop
```

This became one of the most important ACL concepts learned during this lab.

---

# Verification Commands

View ACLs:

```bash
show access-lists
```

Check ACL Applied To Interface:

```bash
show ip interface g0/0
```

Check Routing Table:

```bash
show ip route
```

Check RIP Routes:

```bash
show ip protocols
```

View Running Configuration:

```bash
show running-config
```

---

# Key Takeaways

By the end of this lab I learned:

* Difference between Standard ACL and Extended ACL
* Source IP Filtering
* Destination IP Filtering
* Protocol Filtering
* ICMP Filtering
* TCP Filtering
* Port-Based Filtering
* HTTP Filtering (Port 80)
* SSH Filtering (Port 22)
* Telnet Filtering (Port 23)
* Wildcard Masks
* ACL Placement Strategy
* First Match Wins Rule
* Implicit Deny Rule
* ACL Troubleshooting
* Real-world Security Policy Implementation

This lab provided hands-on experience with Extended ACLs and demonstrated how enterprise networks control traffic based on IP addresses, protocols, and application ports.
