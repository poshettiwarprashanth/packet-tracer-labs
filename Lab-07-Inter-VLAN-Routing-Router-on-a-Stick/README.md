# Lab 07 – Inter-VLAN Routing (Router-on-a-Stick)

## Objective

This lab demonstrates how devices in different VLANs communicate using **Inter-VLAN Routing**. It introduces the **Router-on-a-Stick** architecture, where a single router interface is divided into multiple subinterfaces to route traffic between VLANs.

---

## Concepts Covered

* VLANs and Broadcast Domains
* Access Ports vs Trunk Ports
* IEEE 802.1Q VLAN Tagging
* Default Gateway
* Router Subinterfaces
* Inter-VLAN Routing
* Router-on-a-Stick Architecture
* Packet Flow Between VLANs

---

## Network Topology

```
PC0 --------\
             \
              SW1 -------- Router0
             /
PC1 --------/
```

### Connections

* PC0 → SW1 Fa0/1
* PC1 → SW1 Fa0/2
* Router G0/0 → SW1 Fa0/24

---

## IP Addressing Scheme

| Device         | VLAN            | IP Address    |
| -------------- | --------------- | ------------- |
| PC0            | VLAN 10         | 192.168.10.10 |
| PC1            | VLAN 20         | 192.168.20.10 |
| Router G0/0.10 | VLAN 10 Gateway | 192.168.10.1  |
| Router G0/0.20 | VLAN 20 Gateway | 192.168.20.1  |

---

## VLAN Configuration

### Create VLANs

```bash
enable
configure terminal

vlan 10
name HR

vlan 20
name FINANCE
```

### Assign Access Ports

#### PC0 Port

```bash
interface fa0/1
switchport mode access
switchport access vlan 10
```

#### PC1 Port

```bash
interface fa0/2
switchport mode access
switchport access vlan 20
```

---

## Configure Trunk Port

```bash
interface fa0/24
switchport mode trunk
```

Verify:

```bash
show vlan brief
```

Expected:

* VLAN 10 → Fa0/1
* VLAN 20 → Fa0/2

---

## Configure PCs

### PC0

```text
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Gateway: 192.168.10.1
```

### PC1

```text
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Gateway: 192.168.20.1
```

---

## Verify Communication Before Router Configuration

From PC0:

```bash
ping 192.168.20.10
```

Result:

```text
Request Timed Out
```

### Why Does It Fail?

* VLAN 10 and VLAN 20 are separate broadcast domains.
* They belong to different IP networks.
* A Layer 2 switch cannot perform routing.
* No router is available to forward packets between VLANs.

---

## Router Configuration

### Enable Physical Interface

```bash
enable
configure terminal

interface g0/0
no shutdown
```

Verify:

```bash
show ip interface brief
```

---

## Create Router Subinterfaces

### VLAN 10 Gateway

```bash
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

### VLAN 20 Gateway

```bash
interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

---

## Understanding encapsulation dot1Q

```bash
encapsulation dot1Q 10
```

Meaning:

* Frames tagged with VLAN 10
* Processed by subinterface G0/0.10

```bash
encapsulation dot1Q 20
```

Meaning:

* Frames tagged with VLAN 20
* Processed by subinterface G0/0.20

---

## Why Use Subinterfaces?

One physical router interface:

```text
G0/0
```

Multiple logical interfaces:

```text
G0/0.10 → VLAN 10 Gateway
G0/0.20 → VLAN 20 Gateway
```

This allows a single router interface to route traffic for multiple VLANs.

This design is known as:

**Router-on-a-Stick**

---

## Verification

```bash
show ip interface brief
```

Expected:

```text
G0/0      up    up
G0/0.10   up    up
G0/0.20   up    up
```

---

## Test Inter-VLAN Routing

From PC0:

```bash
ping 192.168.20.10
```

Expected Result:

```text
Success
```

---

## Packet Flow

1. PC0 wants to communicate with PC1.
2. PC0 checks the destination IP.
3. Destination belongs to a different subnet.
4. PC0 forwards the packet to its default gateway (192.168.10.1).
5. PC0 sends an ARP request for the router MAC address.
6. Router replies with its MAC address.
7. PC0 sends the frame to the router.
8. Switch forwards the frame through the trunk link.
9. Frame contains VLAN 10 tag.
10. Router receives the frame on G0/0.10.
11. Router routes the packet to VLAN 20.
12. Router ARPs for PC1.
13. PC1 responds.
14. Router forwards the packet to VLAN 20.
15. PC1 receives the ICMP Echo Request.
16. PC1 sends an ICMP Echo Reply.
17. Ping succeeds.

---

## Key Learning Outcomes

### VLAN

Logical separation of devices into different broadcast domains.

### Access Port

Carries traffic for only one VLAN.

### Trunk Port

Carries traffic for multiple VLANs.

### IEEE 802.1Q

Adds VLAN tags to frames traversing trunk links.

### Subinterface

Logical interface created on a router's physical interface.

### Default Gateway

Used to reach destinations outside the local subnet.

### Inter-VLAN Routing

Communication between devices in different VLANs.

### Router-on-a-Stick

A routing technique using one router interface and multiple subinterfaces connected to a switch trunk.

---

## Lab Summary

### Before Router Configuration

```text
VLAN 10 ❌ VLAN 20
```

### After Router-on-a-Stick

```text
VLAN 10 → Router → VLAN 20 ✅
```

### Skills Practiced

* VLAN Creation
* Access Port Configuration
* Trunk Configuration
* Router Subinterface Configuration
* IEEE 802.1Q Tagging
* Default Gateway Configuration
* Inter-VLAN Routing Verification
* Packet Flow Analysis

---

## Conclusion

This lab demonstrates how a router enables communication between different VLANs. Using Router-on-a-Stick, multiple VLANs can share a single physical router interface while maintaining logical separation and controlled routing between networks.
