# Lab 09 – Multi-Hop Static Routing (3 Routers)

## 📌 Objective

The goal of this lab is to understand how **static routing** enables communication between remote networks connected through multiple routers.

By completing this lab, you will learn:

* Why routers cannot communicate with remote networks by default
* What a routing table is
* Difference between Connected, Local, and Static routes
* Why static routes are required
* How to configure static routes
* What a next-hop IP address is
* How packets travel through multiple routers
* How to verify routing tables
* How to troubleshoot static routing problems
* End-to-end communication across multiple networks

---

# 🖥️ Network Topology

```
LAN 1                     WAN 1               WAN 2                    LAN 2

PC0 ---- SW1 ---- R1 -------- R2 -------- R3 ---- SW2 ---- PC1
```

---

# 🛠 Devices Used

* 2 PCs
* 2 Cisco 2960 Switches
* 3 Cisco 2911 Routers

---

# 🔌 Cabling

| From       | To         | Cable                   |
| ---------- | ---------- | ----------------------- |
| PC0 Fa0    | SW1 Fa0/1  | Copper Straight Through |
| SW1 Fa0/24 | R1 Gi0/0   | Copper Straight Through |
| R1 Gi0/1   | R2 Gi0/0   | Copper Cross-Over       |
| R2 Gi0/1   | R3 Gi0/0   | Copper Cross-Over       |
| R3 Gi0/1   | SW2 Fa0/24 | Copper Straight Through |
| SW2 Fa0/1  | PC1 Fa0    | Copper Straight Through |

---

# 🌐 IP Addressing

| Device | Interface | IP Address    | Subnet Mask     |
| ------ | --------- | ------------- | --------------- |
| PC0    | Fa0       | 192.168.10.10 | 255.255.255.0   |
| R1     | Gi0/0     | 192.168.10.1  | 255.255.255.0   |
| R1     | Gi0/1     | 10.0.12.1     | 255.255.255.252 |
| R2     | Gi0/0     | 10.0.12.2     | 255.255.255.252 |
| R2     | Gi0/1     | 10.0.23.1     | 255.255.255.252 |
| R3     | Gi0/0     | 10.0.23.2     | 255.255.255.252 |
| R3     | Gi0/1     | 192.168.20.1  | 255.255.255.0   |
| PC1    | Fa0       | 192.168.20.10 | 255.255.255.0   |

---

# 🚀 Lab Procedure

## Step 1 – Configure Router Interfaces

Configured IP addresses on all router interfaces and enabled them using:

* `ip address`
* `no shutdown`

Verified interface status using:

```
show ip interface brief
```

---

## Step 2 – Configure PCs

Configured:

* IP Address
* Subnet Mask
* Default Gateway

Verified local connectivity using ping.

---

## Step 3 – Verify Interface Status

Executed:

```
show ip interface brief
```

Expected:

```
Status : up
Protocol : up
```

---

## Step 4 – Verify Directly Connected Networks

Performed connectivity tests between directly connected devices.

Examples:

```
PC0 → R1
R1 → R2
R2 → R3
R3 → PC1
```

The first ping may fail due to ARP resolution. Subsequent pings should succeed.

---

## Step 5 – Examine Routing Tables

Executed:

```
show ip route
```

Observed only:

* C (Connected)
* L (Local)

No routes existed for remote networks.

Example on R1:

```
C 192.168.10.0/24
C 10.0.12.0/30

L 192.168.10.1/32
L 10.0.12.1/32
```

Notice:

```
192.168.20.0
```

was missing.

This means R1 had no knowledge of LAN2.

---

# Why Static Routing?

Routers only know:

* Directly connected networks
* Their own interfaces

They do **not** automatically learn remote networks.

Static routes manually teach routers where remote networks are located.

---

# Step 6 – Configure Static Routes

Configured static routes on:

* R1
* R2
* R3

using:

```
ip route
```

Example:

```
ip route 192.168.20.0 255.255.255.0 10.0.12.2
```

Meaning:

To reach LAN2,

forward packets to the next-hop router.

---

# Step 7 – Verify Static Routes

Executed:

```
show ip route
```

Observed new entries:

```
S = Static
```

Example:

```
S 10.0.23.0/30
S 192.168.20.0/24
```

---

# Step 8 – End-to-End Connectivity

Performed:

```
PC0

↓

Ping

↓

PC1
```

Result:

```
Reply
Reply
Reply
Reply
```

Communication across all three routers was successful.

---

# Step 9 – Packet Flow

Request Path

```
PC0
 ↓
SW1
 ↓
R1
 ↓
R2
 ↓
R3
 ↓
SW2
 ↓
PC1
```

Reply Path

```
PC1
 ↓
SW2
 ↓
R3
 ↓
R2
 ↓
R1
 ↓
SW1
 ↓
PC0
```

---

# Step 10 – Simulation Mode

Packet Tracer Simulation Mode was used to visualize packet forwarding.

Observed:

* ARP Request
* ARP Reply
* ICMP Echo Request
* Packet forwarding at R1
* Packet forwarding at R2
* Packet forwarding at R3
* ICMP Echo Reply

This clearly demonstrates how each router forwards packets based on its routing table.

---

# 🛠 Troubleshooting Scenario

## Problem

```
show running-config
```

showed static routes,

but

```
show ip route
```

did not display any **S** entries.

---

## Root Cause

The **Gi0/1 interface on R1** had an incorrect IP address.

As a result,

the configured next-hop **10.0.12.2** was unreachable.

Cisco IOS keeps the static route in the configuration,

but does **not** install it into the routing table until the next-hop becomes reachable.

---

## Solution

Corrected the IP address on R1 Gi0/1.

Immediately afterwards,

```
show ip route
```

displayed all static routes successfully.

---

## Troubleshooting Checklist

Whenever static routes do not appear:

* Verify interface IP addresses
* Verify interfaces are Up/Up
* Ping the next-hop router
* Verify static route configuration
* Check routing table

---

# 💻 Commands Used

```
enable

configure terminal

hostname

interface g0/x

ip address

no shutdown

show ip interface brief

show ip route

show running-config

show running-config | include ip route

ping

ip route

write memory
```

---

# 📚 Key Concepts Learned

* Static Routing
* Routing Tables
* Connected Routes
* Local Routes
* Static Routes
* Next-Hop Address
* Multi-Hop Routing
* Packet Forwarding
* Route Verification
* Static Route Troubleshooting
* End-to-End Communication
* Three-Router Routing Architecture

---

# 📸 GitHub Screenshots

Include the following screenshots:

* Network Topology
* `show ip interface brief` (all routers)
* `show ip route` before static routing
* Static route configuration (`show running-config | include ip route`)
* `show ip route` after static routing (showing **S** entries)
* Successful ping from PC0 to PC1
* Simulation Mode packet traversal (optional but recommended)

---

# ✅ Lab Outcome

Successfully configured **Multi-Hop Static Routing** across three Cisco routers.

Verified end-to-end communication between two remote LANs using manually configured static routes.

Gained hands-on experience with routing tables, next-hop forwarding, packet traversal, route verification, and troubleshooting static routing issues in Cisco Packet Tracer.
# NetSec Lab 10 – Dynamic Routing using RIP (Routing Information Protocol)

## Objective

In this lab, I learned how routers can automatically exchange routing information using the Routing Information Protocol (RIP). Unlike Static Routing, where routes are manually configured on every router, RIP enables routers to discover remote networks automatically by sharing routing updates with neighboring routers.

By the end of this lab, I understood:

* What Dynamic Routing is.
* Why Static Routing is difficult to manage in large networks.
* What a Routing Protocol is.
* What RIP (Routing Information Protocol) is.
* Difference between Static Routing and Dynamic Routing.
* How routers exchange routing information automatically.
* What Routing Updates are.
* What Hop Count is and how RIP uses it.
* Difference between RIP Version 1 and RIP Version 2.
* Why RIP Version 2 is preferred.
* Why the `no auto-summary` command is used.
* How to configure RIP on Cisco routers.
* How routers automatically learn remote networks.
* How to verify dynamically learned routes.
* How to troubleshoot RIP configuration issues.
* How packets travel across multiple routers after RIP learns the routes.

---

# Prerequisites

Before performing this lab, I completed the following networking labs:

* Lab 07 – Inter-VLAN Routing (Router-on-a-Stick)
* Lab 08 – Static Routing
* Lab 09 – Multi-Hop Static Routing

These labs helped me understand IP addressing, routing tables, static routes, and communication between multiple routers before moving to Dynamic Routing.

---

# Problem Statement

In the previous lab, every router required manually configured static routes.

For example, on Router1, I had to configure:

```bash
ip route 192.168.3.0 255.255.255.0 10.0.12.2
```

This command manually tells Router1:

> "To reach the 192.168.3.0/24 network, forward packets to Router2."

While this works for small networks, it becomes very difficult to maintain in large organizations.

Imagine a company with:

* 10 Routers
* 50 Routers
* 100 Routers
* 500 Routers

Configuring static routes manually on every router would be time-consuming and difficult to manage.

This problem is solved using **Dynamic Routing Protocols**.

---

# What is Dynamic Routing?

Dynamic Routing is a process where routers automatically exchange routing information and learn about remote networks without requiring manual static route configuration.

Instead of configuring every route manually, routers communicate with each other using Routing Protocols.

Whenever a new network is added or an existing route changes, routers automatically update their routing tables.

---

# Real-Life Example

Imagine three friends:

* Friend A knows Hyderabad.
* Friend B knows Delhi.
* Friend C knows Mumbai.

If Friend A wants to reach Mumbai, Friend B replies:

> "Go through me. I know the way."

Friend A doesn't need a map because Friend B already knows the route.

Dynamic Routing works in the same way. Routers continuously share information with their neighboring routers so every router eventually learns how to reach every network.

---

# What is a Routing Protocol?

A Routing Protocol is a communication language used by routers to exchange routing information.

Without a routing protocol, routers know only their directly connected networks.

With a routing protocol, routers automatically learn about remote networks.

Some commonly used routing protocols are:

* RIP (Routing Information Protocol)
* OSPF (Open Shortest Path First)
* EIGRP (Enhanced Interior Gateway Routing Protocol)
* IS-IS (Intermediate System to Intermediate System)
* BGP (Border Gateway Protocol)

In this lab, I learned the simplest routing protocol:

**RIP (Routing Information Protocol)**

---

# Topology

```text
                 10.0.12.0/30                10.0.23.0/30

 PC1           PC3                           PC4
  |             |                             |
 S1            S3                            S4
  |             |                             |
R1-------------R2----------------------------R3
 |
S2
|
PC2
```

---

# Devices Used

* 3 × Cisco 2911 Routers
* 4 × Cisco 2960 Switches
* 4 × PCs

---

# Cable Types Used

### Straight-Through Cable

Used between:

* PC ↔ Switch
* Switch ↔ Router

Reason:

Different devices require Straight-Through cables for communication.

---

### Serial DCE/DTE Cable

Used between:

* Router1 ↔ Router2
* Router2 ↔ Router3

The DCE side provides the clock signal using:

```bash
clock rate 64000
```

The DTE side does not require a clock rate configuration.

---

# IP Addressing Plan

## LAN Networks

| Network       | Address         |
| ------------- | --------------- |
| R1 LAN        | 192.168.1.0/24  |
| R1 Second LAN | 192.168.10.0/24 |
| R2 LAN        | 192.168.2.0/24  |
| R3 LAN        | 192.168.3.0/24  |

## Serial Networks

| Link              | Network      |
| ----------------- | ------------ |
| Router1 ↔ Router2 | 10.0.12.0/30 |
| Router2 ↔ Router3 | 10.0.23.0/30 |

---

# Steps Performed

## Step 1 – Build the Network Topology

Created the complete network using:

* 3 Cisco 2911 Routers
* 4 Cisco 2960 Switches
* 4 PCs

Connected all devices according to the topology.

---

## Step 2 – Configure PC IP Addresses

Configured:

* IP Address
* Subnet Mask
* Default Gateway

for all four PCs.

---

## Step 3 – Configure Router Interfaces

Configured:

* Hostname
* Gigabit Ethernet Interfaces
* Serial Interfaces
* IP Addresses
* Clock Rate on DCE Interfaces
* `no shutdown`

Verified interface status using:

```bash
show ip interface brief
```

---

## Step 4 – Verify Directly Connected Networks

Before configuring RIP, verified that routers could communicate only with their directly connected networks.

Testing:

* Router1 ↔ Router2 ✅
* Router2 ↔ Router3 ✅
* PC1 ↔ PC3 ❌

Reason:

Routers had no knowledge of remote networks.

---

## Step 5 – Configure RIP Version 2

Configured RIP on all three routers.

Commands used:

```bash
router rip
version 2
no auto-summary
```

Advertised directly connected networks using:

```bash
network <network-address>
```

Router1:

* 192.168.1.0
* 192.168.10.0
* 10.0.12.0

Router2:

* 192.168.2.0
* 10.0.12.0
* 10.0.23.0

Router3:

* 192.168.3.0
* 10.0.23.0

---

## Step 6 – Verify RIP Operation

Verified RIP using:

```bash
show ip route
show ip protocols
```

Observed:

* Connected Routes (C)
* Local Routes (L)
* RIP Learned Routes (R)

Verified that routers automatically learned remote networks.

---

## Step 7 – End-to-End Connectivity Testing

Performed ping tests between all LANs.

Examples:

* PC1 → PC3
* PC1 → PC4
* PC2 → PC4

All pings were successful after RIP exchanged routing information.

---

## Step 8 – Simulation Mode

Used Packet Tracer Simulation Mode to observe:

* RIP Routing Updates
* Route Advertisement
* ICMP Echo Request
* ICMP Echo Reply
* Packet forwarding through multiple routers

This helped visualize how routers automatically exchange routing information.

---

# Troubleshooting

During this lab, I encountered and resolved several issues:

* Incorrect Serial IP Address
* Wrong DCE/DTE Interface
* Missing `clock rate`
* Missing `no shutdown`
* Interfaces showing `administratively down`
* Incorrect subnet mask
* RIP not advertising a network
* Verified routing tables using `show ip route`

---

# Commands Used

```bash
hostname
interface
ip address
clock rate 64000
no shutdown
show ip interface brief
ping
router rip
version 2
no auto-summary
network
show ip route
show ip protocols
```

---

# What I Learned

* Dynamic Routing removes the need for manually configuring every route.
* RIP allows routers to exchange routing information automatically.
* RIP Version 2 supports VLSM and subnet masks.
* The `network` command enables RIP on directly connected networks and advertises them.
* Routers automatically update their routing tables through RIP updates.
* `show ip route` displays both directly connected and dynamically learned routes.
* Simulation Mode provides a clear visualization of routing updates and packet flow.
* Dynamic Routing is more scalable and manageable than Static Routing for larger networks.

---

# Conclusion

In this lab, I successfully implemented Dynamic Routing using RIP Version 2 on a three-router network. I configured router interfaces, enabled RIP, verified automatic route learning, tested end-to-end communication between multiple LANs, analyzed routing tables, and observed RIP updates and ICMP packets using Cisco Packet Tracer Simulation Mode. This lab strengthened my understanding of how routing protocols automate route discovery and improve network scalability.
