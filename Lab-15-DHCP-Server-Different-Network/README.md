# NetSec Lab 15 - DHCP Relay Agent (DHCP Server Outside Client Network)

## What We Are Going To Learn In This Lab

By the end of this lab, you will understand:

1. What is DHCP?
2. Why DHCP is needed?
3. Problems with Manual IP Address Configuration
4. DHCP Components
5. DORA Process
6. What is a Broadcast?
7. Why DHCP Discover uses Broadcast?
8. Why Routers Do Not Forward Broadcasts?
9. Why DHCP Fails Across Different Networks?
10. What is a DHCP Relay Agent?
11. How DHCP Relay Agent Solves the Problem?
12. What is `ip helper-address`?
13. Dedicated DHCP Server vs Router DHCP Server
14. DHCP Pool Configuration
15. Static Routing Requirement
16. Source and Destination IP Analysis
17. Source and Destination MAC Analysis
18. Real World Enterprise DHCP Design
19. DHCP Troubleshooting
20. Interview Questions

---

# Objective

Configure a DHCP Server located in a different subnet and use a DHCP Relay Agent (`ip helper-address`) to provide IP addresses to clients across routed networks.

---

# Introduction

In previous labs, the router itself acted as the DHCP Server:

```
PC → Switch → Router (DHCP Server)
```

Since both devices were inside the same broadcast domain, DHCP worked without any issues.

However, enterprise networks usually deploy a centralized DHCP server that serves multiple subnets and departments.

Benefits:

- Centralized management
- Easier IP tracking
- Better scalability
- Reduced configuration overhead

---

# Why DHCP Exists

Before DHCP, administrators manually configured:

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

## Problems With Manual Configuration

- Time consuming
- Human errors
- Duplicate IP addresses
- Difficult management
- Poor scalability

DHCP automates IP configuration dynamically.

---

# DHCP Components

## DHCP Client

Requests IP configuration.

Examples:

- PC
- Laptop
- Mobile Device
- Printer

---

## DHCP Server

Provides IP configuration.

Examples:

- Router
- Windows Server
- Linux Server
- Dedicated DHCP Appliance

---

## DHCP Relay Agent

A router feature that listens for DHCP broadcasts from clients and forwards them to a remote DHCP Server using unicast communication.

---

# DORA Process

DHCP follows the DORA process:

| Step | Meaning |
|---|---|
| D | Discover |
| O | Offer |
| R | Request |
| A | Acknowledgement |

---

## DHCP Discover

Client searches for available DHCP servers.

```
Source IP: 0.0.0.0
Destination IP: 255.255.255.255
```

---

## DHCP Offer

Server provides an available IP address.

---

## DHCP Request

Client requests the offered IP address.

---

## DHCP ACK

Server confirms the lease.

The client receives:

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

---

# What Is A Broadcast?

Broadcast means one device communicating with every device inside the local network.

Layer 2 Broadcast:

```
FF:FF:FF:FF:FF:FF
```

Layer 3 Broadcast:

```
255.255.255.255
```

> Switches forward broadcasts. Routers block broadcasts by default.

---

# Why DHCP Discover Uses Broadcast

At startup, the client has:

- No IP address
- No gateway information
- No DHCP server information

Therefore it sends:

```
Source IP:
0.0.0.0

Destination IP:
255.255.255.255
```

---

# Problem Statement

Client Network:

```
192.168.1.0/24
```

DHCP Server Network:

```
192.168.2.0/24
```

Will DHCP work automatically?

Answer:

```
No
```

---

# Why DHCP Fails Across Different Networks

DHCP Discover is a broadcast packet.

Flow:

```
PC → Switch → Router
```

The router receives the broadcast.

Routers do not forward Layer 3 broadcasts across networks.

Therefore:

```
Client ❌ DHCP Server
```

communication fails.

---

# Solution - DHCP Relay Agent

DHCP Relay Agent converts DHCP broadcasts into unicast packets.

Cisco command:

```bash
ip helper-address <DHCP_SERVER_IP>
```

Example:

```bash
ip helper-address 192.168.2.10
```

---

# Lab Topology

```
192.168.1.0/24

PC1
PC2
PC3
 |
Switch1
 |
G0/0
Router1
G0/1
 |
10.10.10.0/30
 |
G0/1
Router2
G0/0
 |
192.168.2.0/24
 |
DHCP Server
192.168.2.10
```

---

# Step-by-Step Configuration

## Router1 Configuration

```bash
enable

configure terminal

interface gigabitEthernet0/0

ip address 192.168.1.1 255.255.255.0

no shutdown


interface gigabitEthernet0/1

ip address 10.10.10.1 255.255.255.252

no shutdown

exit
```

---

## Router2 Configuration

```bash
enable

configure terminal

interface gigabitEthernet0/1

ip address 10.10.10.2 255.255.255.252

no shutdown


interface gigabitEthernet0/0

ip address 192.168.2.1 255.255.255.0

no shutdown


end
```

---

# Static Routing

Router1:

```bash
ip route 192.168.2.0 255.255.255.0 10.10.10.2
```

Router2:

```bash
ip route 192.168.1.0 255.255.255.0 10.10.10.1
```

---

# DHCP Server Configuration

Server IP:

```
IP Address:
192.168.2.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.2.1

DNS:
8.8.8.8
```

---

# DHCP Pool

Enable DHCP Service.

Pool:

```
Pool Name:
CLIENT_LAN

Default Gateway:
192.168.1.1

DNS:
8.8.8.8

Start IP:
192.168.1.100

Subnet Mask:
255.255.255.0

Maximum Users:
100
```

---

# Verify Routing

Router1:

```bash
ping 10.10.10.2

ping 192.168.2.10
```

---

# Test Without Relay Agent

PC1:

```
Desktop → IP Configuration → DHCP
```

Result:

```
Failure
```

Client gets:

```
169.254.x.x
```

Reason:

Router drops DHCP broadcast.

---

# Configure DHCP Relay Agent

Router1:

```bash
configure terminal

interface g0/0

ip helper-address 192.168.2.10

end

write memory
```

---

# Verify DHCP Allocation

PC1 receives:

```
IP Address:
192.168.1.100

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.1.1

DNS:
8.8.8.8
```

---

# Packet Analysis

## Before Relay

```
Source IP:
0.0.0.0

Destination IP:
255.255.255.255

Destination MAC:
FF:FF:FF:FF:FF:FF
```

Router drops the packet.

---

## After Relay

Router converts broadcast into unicast:

```
Source IP:
192.168.1.1

Destination IP:
192.168.2.10
```

---

# IP Address Transition

| Segment | Type | Source IP | Destination IP |
|---|---|---|---|
| PC → Router | Broadcast | 0.0.0.0 | 255.255.255.255 |
| Router → DHCP Server | Unicast | 192.168.1.1 | 192.168.2.10 |

---

# MAC Address Transition

| Segment | Source MAC | Destination MAC |
|---|---|---|
| Client LAN | PC MAC | FF:FF:FF:FF:FF:FF |
| Router Link | Router1 MAC | Router2 MAC |
| Server LAN | Router2 MAC | Server MAC |

---

# Dedicated DHCP Server vs Router DHCP

## Router DHCP

Used for:

- Small networks
- Home networks
- Branch offices

Advantages:

- Simple
- Low cost

Disadvantages:

- Limited scalability

---

## Dedicated DHCP Server

Used in:

- Enterprise networks
- Data centers

Advantages:

- Central management
- Multiple scopes
- Better tracking

---

# Real World Enterprise Design

```
HR LAN
 |
Router
 |
ip helper-address
 |
Central DHCP Server


Sales LAN
 |
Router
 |
ip helper-address
 |
Central DHCP Server


IT LAN
 |
Router
 |
ip helper-address
 |
Central DHCP Server
```

---

# Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| 169.254.x.x address | DHCP OFF | Enable DHCP |
| No IP received | Missing helper | Configure helper |
| DORA failure | Routing issue | Check routes |
| Wrong gateway | DHCP pool issue | Correct pool |

---

# Interview Questions

## Why DHCP Discover uses 0.0.0.0?

Because the client does not have an IP address initially.

## Why DHCP uses 255.255.255.255?

Because the client does not know the DHCP server location.

## Why DHCP fails across routers?

Because routers block broadcasts.

## DHCP Relay command?

```bash
ip helper-address <server-ip>
```

## Where is helper-address configured?

On the client-facing router interface.

---

# Key Concepts Learned

- DHCP operation
- DORA process
- Broadcast behavior
- DHCP Relay Agent
- ip helper-address
- Static routing
- Packet flow analysis
- Enterprise DHCP design
- Troubleshooting

---

# Conclusion

In this lab, we configured a centralized DHCP Server located in another subnet and used a DHCP Relay Agent to forward DHCP requests across routers.

We learned:

- Why DHCP broadcasts fail across routed networks
- How routers block broadcasts
- How `ip helper-address` solves the problem
- How enterprise networks use centralized DHCP servers
```