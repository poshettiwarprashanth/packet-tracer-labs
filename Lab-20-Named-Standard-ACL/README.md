# NetSec Lab 20 - Standard Named ACL

## Lab Overview

In this lab, we learned how to configure and verify a **Standard Named Access Control List (ACL)** on a Cisco router. The ACL was used to permit traffic from a specific source host while denying all other hosts through the implicit deny rule.

The primary objective of this lab was to understand:

* What an ACL is
* Why ACLs are used
* Difference between Numbered ACL and Named ACL
* Standard ACL operation
* ACL placement rules
* Implicit deny behavior
* Packet flow through a router
* ACL verification and troubleshooting
* Simulation Mode packet analysis

---

# Lab Objective

Configure a Standard Named ACL that:

* Allows PC1 to access the Server Network
* Blocks PC2 from accessing the Server Network
* Blocks PC3 from accessing the Server Network

---

# Theory

## What is an ACL?

An Access Control List (ACL) is a set of rules applied to a router interface that controls whether packets are permitted or denied.

ACLs are commonly used to:

* Restrict access to resources
* Improve network security
* Filter unwanted traffic
* Control communication between network segments

---

## Types of ACLs

### Standard ACL

Filters traffic using only:

* Source IP Address

Example:

```cisco
access-list 1 permit 192.168.1.10
```

A Standard ACL does not check:

* Destination IP Address
* Protocol
* Port Number

---

### Extended ACL

Filters traffic using:

* Source IP Address
* Destination IP Address
* Protocol
* Port Number

Example:

```cisco
permit tcp 192.168.1.0 0.0.0.255 any eq 80
```

---

## Numbered ACL vs Named ACL

### Numbered ACL

Example:

```cisco
access-list 1 permit 192.168.1.10
```

ACLs are identified using numbers.

---

### Named ACL

Example:

```cisco
ip access-list standard ALLOW_PC1
 permit 192.168.1.10
```

ACLs are identified using meaningful names.

Advantages:

* Easier management
* Easier troubleshooting
* Better readability
* Easier modification in production environments

---

## ACL Placement Rule

### Standard ACL

Place near the destination.

Reason:

A Standard ACL checks only the source IP address. Placing it near the source may unintentionally block access to multiple destinations.

### Extended ACL

Place near the source.

Reason:

An Extended ACL can precisely filter traffic using source, destination, protocol, and port information.

---

## Implicit Deny

Every ACL automatically ends with:

```cisco
deny any
```

even if it is not configured manually.

Example:

Configured ACL:

```cisco
permit 192.168.1.10
```

Router interpretation:

```cisco
permit 192.168.1.10
deny any
```

Therefore, all traffic that does not match a permit statement is denied.

---

# Topology

```text
PC1        PC2        PC3
 |          |          |
 +----------+----------+
            |
         Switch1
            |
          R1
            |
         Switch2
      /      |      \
 Server1  Server2  Server3
```

---

# Devices Used

* 3 PCs
* 2 Switches
* 1 Router
* 3 Servers

Total Devices: 9

---

# Cabling

| Device  | Interface | Connected To | Interface |
| ------- | --------- | ------------ | --------- |
| PC1     | Fa0       | Switch1      | Fa0/1     |
| PC2     | Fa0       | Switch1      | Fa0/2     |
| PC3     | Fa0       | Switch1      | Fa0/3     |
| Switch1 | Fa0/24    | R1           | G0/0      |
| R1      | G0/1      | Switch2      | Fa0/24    |
| Server1 | Fa0       | Switch2      | Fa0/1     |
| Server2 | Fa0       | Switch2      | Fa0/2     |
| Server3 | Fa0       | Switch2      | Fa0/3     |

Cable Type:

```text
Copper Straight Through
```

---

# IP Addressing Scheme

## User Network

Network:

```text
192.168.1.0/24
```

| Device  | IP Address   |
| ------- | ------------ |
| PC1     | 192.168.1.10 |
| PC2     | 192.168.1.20 |
| PC3     | 192.168.1.30 |
| R1 G0/0 | 192.168.1.1  |

Default Gateway for PCs:

```text
192.168.1.1
```

---

## Server Network

Network:

```text
192.168.2.0/24
```

| Device  | IP Address   |
| ------- | ------------ |
| Server1 | 192.168.2.10 |
| Server2 | 192.168.2.20 |
| Server3 | 192.168.2.30 |
| R1 G0/1 | 192.168.2.1  |

Default Gateway for Servers:

```text
192.168.2.1
```

---

# Router Configuration

```cisco
enable
configure terminal

interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

interface g0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

end
write
```

---

# Verification

## Verify Interface Status

Command:

```cisco
show ip interface brief
```

Expected Output:

```text
G0/0    192.168.1.1    up    up
G0/1    192.168.2.1    up    up
```

Meaning:

* First "up" = Layer 1 operational
* Second "up" = Layer 2 operational

---

# Connectivity Testing Before ACL

## PC1 to Gateway

```cisco
ping 192.168.1.1
```

Result:

```text
Success
```

---

## PC1 to Server1

```cisco
ping 192.168.2.10
```

Result:

```text
Success
```

---

## PC2 to Server1

```cisco
ping 192.168.2.10
```

Result:

```text
Success
```

---

## PC3 to Server1

```cisco
ping 192.168.2.10
```

Result:

```text
Success
```

This confirmed routing was functioning correctly before ACL implementation.

---

# Packet Flow Analysis Before ACL

## Step 1

PC1 wants to reach:

```text
192.168.2.10
```

PC1 determines the destination is on a remote network.

---

## Step 2

PC1 performs ARP for the default gateway:

```text
Who has 192.168.1.1?
```

---

## Step 3

R1 replies with its MAC address.

---

## Step 4

PC1 sends ICMP Echo Request.

Layer 3:

```text
Source IP      = 192.168.1.10
Destination IP = 192.168.2.10
```

Layer 2:

```text
Source MAC      = PC1 MAC
Destination MAC = R1 G0/0 MAC
```

---

## Step 5

Router receives the packet.

Router checks routing table.

Destination network:

```text
192.168.2.0/24
```

is directly connected to G0/1.

---

## Step 6

Router performs ARP for Server1 if MAC address is unknown.

---

## Step 7

Router forwards packet.

Layer 3:

```text
Source IP      = 192.168.1.10
Destination IP = 192.168.2.10
```

Layer 2:

```text
Source MAC      = R1 G0/1 MAC
Destination MAC = Server1 MAC
```

---

# Important Learning Point

At every router:

```text
Layer 2 addresses change.
Layer 3 addresses remain unchanged.
```

Exception:

```text
NAT or PAT configurations.
```

---

# Standard Named ACL Configuration

## ACL Requirement

Allow:

```text
PC1 (192.168.1.10)
```

Block:

```text
PC2 (192.168.1.20)
PC3 (192.168.1.30)
```

---

## Create Named ACL

```cisco
enable
configure terminal

ip access-list standard ALLOW_PC1
 permit 192.168.1.10

exit
```

Router automatically adds:

```cisco
deny any
```

through the implicit deny rule.

---

# ACL Placement

Applied on:

```text
R1 G0/1
```

Direction:

```text
Outbound
```

Reason:

```text
Standard ACLs should be placed near the destination.
```

---

# Apply ACL

```cisco
interface g0/1
 ip access-group ALLOW_PC1 out

end
write
```

---

# ACL Verification

## Verify ACL

```cisco
show access-lists
```

Observed Output:

```text
Standard IP access list ALLOW_PC1
    10 permit host 192.168.1.10 (8 match(es))
```

Meaning:

Traffic from PC1 successfully matched the ACL eight times.

---

## Verify Interface Association

```cisco
show ip interface g0/1
```

Expected:

```text
Outbound access list is ALLOW_PC1
```

---

# Connectivity Testing After ACL

## PC1

```cisco
ping 192.168.2.10
```

Result:

```text
Success
```

Reason:

```text
Matched permit statement.
```

---

## PC2

```cisco
ping 192.168.2.10
```

Result:

```text
Request Timed Out
```

Reason:

```text
Implicit deny any.
```

---

## PC3

```cisco
ping 192.168.2.10
```

Result:

```text
Request Timed Out
```

Reason:

```text
Implicit deny any.
```

---

# Simulation Mode Analysis

## Packet Path

```text
PC2
 ↓
Switch1
 ↓
R1 G0/0
 ↓
Routing Decision
 ↓
ACL Check on G0/1 Outbound
 ↓
Packet Dropped
```

---

## Important Observation

The ACL does not stop the packet at G0/0.

The router:

1. Receives the packet
2. Performs routing lookup
3. Determines outgoing interface
4. Checks ACL on G0/1 outbound
5. Drops the packet

---

# Device Responsible for Dropping the Packet

Answer:

```text
R1
```

Reason:

The ACL is configured on R1.

---

# Troubleshooting Commands

```cisco
show access-lists

show running-config

show ip interface g0/1

show ip interface brief
```

---

# Interview Questions

1. What is an ACL?
2. What is a Standard ACL?
3. What is an Extended ACL?
4. What is a Named ACL?
5. Difference between Numbered ACL and Named ACL?
6. Why are Named ACLs preferred?
7. What is the implicit deny rule?
8. Why should Standard ACLs be placed near the destination?
9. Which device drops the packet when ACL denies traffic?
10. What does up/up mean?
11. What information does a Standard ACL examine?
12. What happens when traffic does not match any ACL entry?
13. How do you verify ACL matches?
14. Which addresses change when a packet crosses a router?
15. How do you apply a Named ACL to an interface?
16. What is the purpose of show access-lists?
17. Why did PC1 succeed while PC2 and PC3 failed?
18. Does the router check the ACL before or after routing?
19. Why does a host use a default gateway?
20. What is the difference between inbound and outbound ACLs?

---

# Lab Conclusion

In this lab, a Standard Named ACL named ALLOW_PC1 was created and applied outbound on R1 G0/1. The ACL permitted only PC1 (192.168.1.10) to access the server network while all other hosts were denied through the implicit deny rule. The lab demonstrated ACL creation, placement, verification, packet flow analysis, Simulation Mode behavior, and real-world troubleshooting techniques. This lab reinforced the fundamental concept that Standard ACLs filter traffic using only the source IP address and should be placed as close to the destination as possible.
