# Lab 08 – Static Routing (First Routing Lab)

## Objective

In this lab, we learned:

* Why routers need routing tables
* Why directly connected networks are not enough
* What Static Routing is
* What a Next-Hop Router is
* How routers forward packets between different networks
* How routing tables are used to make forwarding decisions
* How packets travel through multiple routers
* ARP and ICMP packet flow analysis using Simulation Mode

---

# Network Topology

```text
                    Network: 10.0.0.0/30

             10.0.0.1                    10.0.0.2
          +-----------+              +-----------+
          | Router0   |--------------| Router1   |
          +-----------+              +-----------+
                |                          |
                |                          |
         192.168.1.1                 192.168.2.1
                |                          |
             Switch0                    Switch1
            /       \                  /       \
         PC0       PC1              PC2       PC3

Network: 192.168.1.0/24       Network: 192.168.2.0/24
```

---

# IP Addressing Table

## LAN 1 – 192.168.1.0/24

| Device       | IP Address   | Subnet Mask   | Default Gateway |
| ------------ | ------------ | ------------- | --------------- |
| PC0          | 192.168.1.10 | 255.255.255.0 | 192.168.1.1     |
| PC1          | 192.168.1.20 | 255.255.255.0 | 192.168.1.1     |
| Router0 G0/0 | 192.168.1.1  | 255.255.255.0 | N/A             |

---

## Router-to-Router Link

| Device  | Interface | IP Address  |
| ------- | --------- | ----------- |
| Router0 | G0/1      | 10.0.0.1/30 |
| Router1 | G0/1      | 10.0.0.2/30 |

Network: **10.0.0.0/30**

---

## LAN 2 – 192.168.2.0/24

| Device       | IP Address   | Subnet Mask   | Default Gateway |
| ------------ | ------------ | ------------- | --------------- |
| PC2          | 192.168.2.10 | 255.255.255.0 | 192.168.2.1     |
| PC3          | 192.168.2.20 | 255.255.255.0 | 192.168.2.1     |
| Router1 G0/0 | 192.168.2.1  | 255.255.255.0 | N/A             |

---

# Router Configuration

## Router0

```bash
enable
configure terminal

interface g0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
exit

interface g0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
exit
```

---

## Router1

```bash
enable
configure terminal

interface g0/0
ip address 192.168.2.1 255.255.255.0
no shutdown
exit

interface g0/1
ip address 10.0.0.2 255.255.255.252
no shutdown
exit
```

---

# Verification Before Static Routing

## Test 1

```bash
PC0> ping 192.168.1.20
```

### Result

✅ Success

### Reason

Both devices belong to the same network.

---

## Test 2

```bash
PC0> ping 192.168.1.1
```

### Result

✅ Success

### Reason

Default gateway is reachable.

---

## Test 3

```bash
PC0> ping 192.168.2.10
```

### Result

❌ Destination Host Unreachable

### Reason

Router0 does not know how to reach network **192.168.2.0/24**.

---

# Important Learning

### Router0 Knows

* 192.168.1.0/24
* 10.0.0.0/30

### Router1 Knows

* 192.168.2.0/24
* 10.0.0.0/30

Even though Router0 knows Router1 exists:

```text
10.0.0.2
```

Router0 does **NOT** know:

```text
192.168.2.0/24 is behind Router1
```

### Key Concept

Routers never guess.

Routers forward packets only based on their Routing Table.

---

# Static Routing Configuration

## Router0

```bash
ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

### Meaning

To reach **192.168.2.0/24**, send packets to Router1 (**10.0.0.2**).

---

## Router1

```bash
ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

### Meaning

To reach **192.168.1.0/24**, send packets to Router0 (**10.0.0.1**).

---

# Routing Table Verification

## Router0

```bash
show ip route
```

New Entry:

```text
S 192.168.2.0/24 via 10.0.0.2
```

---

## Router1

```text
S 192.168.1.0/24 via 10.0.0.1
```

### Route Codes

| Code | Meaning         |
| ---- | --------------- |
| S    | Static Route    |
| C    | Connected Route |
| L    | Local Route     |

---

# Final Connectivity Test

```bash
PC0> ping 192.168.2.10
```

### Result

✅ Success

---

# Packet Flow

## Before Static Routing

```text
PC0
 ↓
Router0
 ↓
Packet Dropped
```

### Reason

No route found for:

```text
192.168.2.0/24
```

---

## After Static Routing

```text
PC0
 ↓
Switch0
 ↓
Router0
 ↓
Router1
 ↓
Switch1
 ↓
PC2
```

---

# Simulation Mode Analysis

## ARP Process

PC0 determines:

```text
192.168.2.10 belongs to a different network
```

Therefore it sends the packet to its Default Gateway:

```text
192.168.1.1
```

### ARP Request

```text
Who has 192.168.1.1?
```

### ARP Reply

```text
192.168.1.1 is me
```

---

## ICMP Process

PC0 generates:

```text
ICMP Echo Request
```

Router0 checks its Routing Table:

```text
192.168.2.0/24 via 10.0.0.2
```

Router0 forwards the packet to Router1.

Router1 forwards the packet to PC2.

PC2 responds with:

```text
ICMP Echo Reply
```

Return path:

```text
PC2 → Router1 → Router0 → PC0
```

---

# Cybersecurity / Packet Analysis Learning

## IP Header Remains the Same

```text
Source IP      = 192.168.1.10
Destination IP = 192.168.2.10
```

The Layer 3 (IP) information remains unchanged throughout the journey.

---

## MAC Address Changes at Every Hop

### PC0 → Router0

```text
Source MAC = PC0
Destination MAC = Router0
```

### Router0 → Router1

```text
Source MAC = Router0
Destination MAC = Router1
```

### Router1 → PC2

```text
Source MAC = Router1
Destination MAC = PC2
```

---

# Key Cybersecurity Concept

At every router:

1. Decapsulation
2. Routing Decision
3. Re-encapsulation

This is how routers move packets between different networks while preserving the original source and destination IP addresses.

---

# What We Learned

* Routing Table Fundamentals
* Connected Routes vs Static Routes
* Next-Hop Concept
* Static Route Configuration
* Packet Forwarding Logic
* ARP Before ICMP
* Router Decision Making Process
* End-to-End Packet Traversal
* IP Header vs MAC Header Behavior
* Decapsulation and Re-encapsulation
* Basic Packet Analysis for Cybersecurity
* Foundation for Dynamic Routing Protocols (RIP, OSPF, EIGRP)

---

## Lab Status

✅ Static Routing Successfully Configured

✅ End-to-End Connectivity Verified

✅ Packet Flow Analyzed Using Simulation Mode

✅ Foundation Ready for Advanced Routing Labs
