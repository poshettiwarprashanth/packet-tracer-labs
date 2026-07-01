# NetSec Lab 11 - Default Routing (Gateway of Last Resort)

## Objective

In this lab, I learned:

* What a Default Route is
* Why Static Routing becomes difficult in larger networks
* What the Gateway of Last Resort means
* Difference between Static Routing and Default Routing
* How routers forward packets when the destination network is unknown
* How to configure a Default Route
* How to verify Default Routes using routing tables
* Why successful communication requires both forward and return paths

---

# Theory

A Default Route is used when a router does not have a specific route for a destination network.

Instead of creating individual static routes for every remote network, a router can be configured to send all unknown traffic to a specific next-hop router.

Default Route Syntax:

```bash
ip route 0.0.0.0 0.0.0.0 <next-hop-IP>
```

This route is also called:

* Quad Zero Route
* Default Route
* Gateway of Last Resort

Meaning:

```text
0.0.0.0 0.0.0.0
```

matches any destination network that is not already present in the routing table.

---

# Topology

Devices Used:

* PC1
* PC2
* Router R1
* Router R2

Connections:

```text
PC1 ---- R1 ---- R2 ---- PC2
```

Cables Used:

* PC to Router → Copper Straight-Through
* Router to Router → Copper Cross-Over

---

# IP Addressing Scheme

## LAN 1

Network:

```text
10.1.1.0/24
```

PC1:

```text
IP Address: 10.1.1.10
Subnet Mask: 255.255.255.0
Default Gateway: 10.1.1.1
```

R1 G0/0:

```text
10.1.1.1/24
```

---

## Router-to-Router Link

Network:

```text
192.168.12.0/24
```

R1 G0/1:

```text
192.168.12.1/24
```

R2 G0/1:

```text
192.168.12.2/24
```

---

## LAN 2

Network:

```text
10.2.2.0/24
```

PC2:

```text
IP Address: 10.2.2.10
Subnet Mask: 255.255.255.0
Default Gateway: 10.2.2.1
```

R2 G0/0:

```text
10.2.2.1/24
```

---

# Router Configuration

## R1 Configuration

```bash
enable
configure terminal

interface g0/0
ip address 10.1.1.1 255.255.255.0
no shutdown

interface g0/1
ip address 192.168.12.1 255.255.255.0
no shutdown
```

---

## R2 Configuration

```bash
enable
configure terminal

interface g0/0
ip address 10.2.2.1 255.255.255.0
no shutdown

interface g0/1
ip address 192.168.12.2 255.255.255.0
no shutdown
```

---

# Verification Before Routing

Verified interface status:

```bash
show ip interface brief
```

Verified routing table:

```bash
show ip route
```

Observation:

R1 knew only:

```text
10.1.1.0/24
192.168.12.0/24
```

R2 knew only:

```text
10.2.2.0/24
192.168.12.0/24
```

Neither router knew the remote LAN.

---

# Connectivity Testing

Router-to-Router Ping:

```bash
R1# ping 192.168.12.2
R2# ping 192.168.12.1
```

Result:

```text
Success
```

---

# End-to-End Test Before Routing

From PC1:

```bash
ping 10.2.2.10
```

Result:

```text
Failed
```

Reason:

R1 had no route for:

```text
10.2.2.0/24
```

and therefore dropped the packet.

---

# Default Route Configuration

## Configure Default Route on R1

```bash
enable
configure terminal

ip route 0.0.0.0 0.0.0.0 192.168.12.2
```

Meaning:

```text
If destination is unknown,
send traffic to R2.
```

---

## Configure Default Route on R2

```bash
enable
configure terminal

ip route 0.0.0.0 0.0.0.0 192.168.12.1
```

Meaning:

```text
If destination is unknown,
send traffic to R1.
```

---

# Verification

Verify routing table:

```bash
show ip route
```

Observed:

```text
Gateway of last resort is 192.168.12.2

S* 0.0.0.0/0 [1/0] via 192.168.12.2
```

Explanation:

```text
S = Static Route
* = Default Route
```

This indicates a Static Default Route.

---

# Final Connectivity Test

From PC1:

```bash
ping 10.2.2.10
```

Result:

```text
Success
```

Packet Flow:

```text
PC1
↓
R1
↓
R2
↓
PC2
↓
R2
↓
R1
↓
PC1
```

Both forward and return paths were successfully established.

---

# Key Concepts Learned

* A Default Route is used when no specific route exists.
* Default Route is configured using 0.0.0.0/0.
* Gateway of Last Resort is used for unknown destinations.
* Routing decisions are made using the routing table.
* Communication requires both forward and return paths.
* Default Routing reduces the need for multiple static routes.
* The routing table displays Default Routes as S* entries.

---

# Commands Used

```bash
show ip interface brief

show ip route

ping 192.168.12.1

ping 192.168.12.2

ping 10.2.2.10

ip route 0.0.0.0 0.0.0.0 192.168.12.2

ip route 0.0.0.0 0.0.0.0 192.168.12.1
```

---

# Lab Outcome

Successfully configured and verified Default Routing between two routers.

Learned how routers use the Gateway of Last Resort to forward packets toward unknown destinations and how Default Routing simplifies route management compared to maintaining multiple static routes.
