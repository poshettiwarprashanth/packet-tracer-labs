# Lab 03 – Switch MAC Address Learning and Forwarding

## Overview

In this lab, I explored how an Ethernet switch learns MAC addresses and makes forwarding decisions. Using Cisco Packet Tracer Simulation Mode, I observed how a switch builds its MAC (CAM) table, handles broadcasts and unicasts, and transitions from an empty table to intelligent frame forwarding.

This lab provided a practical understanding of Layer 2 switching behavior, which forms the foundation for advanced networking and cybersecurity concepts such as MAC flooding attacks and switch hardening.

---

## Objectives

By the end of this lab, I was able to:

* Understand what a MAC address is.
* Explain how a switch learns MAC addresses.
* Understand the purpose of the MAC (CAM) table.
* Differentiate between:

  * Broadcast traffic
  * Unknown Unicast traffic
  * Known Unicast traffic
* Observe how switches make forwarding decisions.
* Analyze switch behavior using Packet Tracer Simulation Mode.
* Understand why MAC learning is important from a cybersecurity perspective.

---

## Lab Topology

```
PC0 --------\
             \
              Switch0
             /
PC1 --------/
```

### Connections

| Device | Device Port   | Switch Port |
| ------ | ------------- | ----------- |
| PC0    | FastEthernet0 | Fa0/1       |
| PC1    | FastEthernet0 | Fa0/2       |

---

## IP Configuration

### PC0

* IP Address: `192.168.1.10`
* Subnet Mask: `255.255.255.0`
* Default Gateway: Not Configured

### PC1

* IP Address: `192.168.1.20`
* Subnet Mask: `255.255.255.0`
* Default Gateway: Not Configured

---

## Initial Switch State

Before any communication occurred:

### Questions

**Does the switch know where PC0 is?**

No.

**Does the switch know where PC1 is?**

No.

### MAC (CAM) Table

| MAC Address | Port  |
| ----------- | ----- |
| Empty       | Empty |

The switch starts with an empty CAM table.

This state is commonly referred to as a **cold switch**.

---

## Simulation Setup

Packet Tracer was switched to:

```
Simulation Mode
```

Protocol filters enabled:

* ARP
* ICMP

All other protocols were disabled.

Packet progression was observed using:

```
Capture / Forward
```

instead of Auto Capture.

---

## ARP Request Analysis

PC0 initiated communication by executing:

```bash
ping 192.168.1.20
```

Since PC0 did not know PC1's MAC address, it generated an ARP Request.

### ARP Request

Message:

> "Who has 192.168.1.20? Tell 192.168.1.10."

Destination MAC:

```
FF:FF:FF:FF:FF:FF
```

Type:

```
Broadcast
```

### Switch Behavior

The switch received the frame on:

```
Fa0/1
```

The switch learned:

```
PC0 MAC → Fa0/1
```

It then flooded the broadcast frame to all other ports.

### CAM Table After ARP Request

| MAC Address | Port  |
| ----------- | ----- |
| PC0 MAC     | Fa0/1 |

---

## ARP Reply Analysis

PC1 responded to the ARP Request.

Message:

> "I am 192.168.1.20."

Type:

```
Unicast
```

The switch received this frame on:

```
Fa0/2
```

The switch learned:

```
PC1 MAC → Fa0/2
```

### CAM Table After ARP Reply

| MAC Address | Port  |
| ----------- | ----- |
| PC0 MAC     | Fa0/1 |
| PC1 MAC     | Fa0/2 |

---

## Key Observation

The switch automatically learned both devices without any manual configuration.

### Important Principle

A switch learns only from:

```
Source MAC Addresses
```

It does not learn from destination MAC addresses.

---

## ICMP Echo Request Analysis

After ARP completed, the ping process continued.

PC0 sent an:

```
ICMP Echo Request
```

Destination:

```
PC1 MAC Address
```

The switch checked its CAM table.

Result:

```
PC1 MAC Found
```

Action:

```
Forward only to Fa0/2
```

No flooding occurred.

---

## ICMP Echo Reply Analysis

PC1 replied with an:

```
ICMP Echo Reply
```

Destination:

```
PC0 MAC Address
```

The switch checked its CAM table.

Result:

```
PC0 MAC Found
```

Action:

```
Forward only to Fa0/1
```

Again, no flooding occurred.

---

## Broadcast vs Unknown Unicast vs Known Unicast

### Broadcast

Destination MAC:

```
FF:FF:FF:FF:FF:FF
```

Switch Action:

* Flood all ports except the incoming port.

Example:

* ARP Request

---

### Unknown Unicast

Condition:

* Destination MAC not present in CAM table.

Switch Action:

* Flood all ports except the incoming port.

Example:

* First frame destined to an unseen MAC address.

---

### Known Unicast

Condition:

* Destination MAC exists in CAM table.

Switch Action:

* Forward only through the correct port.

Example:

* ICMP packets after MAC learning.

---

## Cybersecurity Perspective

Understanding switch behavior is important because attackers can exploit MAC learning mechanisms.

### MAC Flooding Attack

Attackers generate thousands of fake source MAC addresses.

Objective:

```
Overflow the CAM Table
↓
Switch cannot learn new MAC addresses
↓
Traffic begins flooding
↓
Attackers attempt packet sniffing
```

This attack abuses the switch's MAC learning process.

---

## Defensive Controls

Enterprise switches provide protections such as:

* Port Security
* Sticky MAC Address Learning
* Maximum MAC Address Limits
* Violation Actions (Protect, Restrict, Shutdown)

These mechanisms help prevent CAM table abuse and unauthorized devices.

---

## Key Learnings

* Switches initially know nothing about connected devices.
* MAC learning occurs automatically.
* Switches learn only from source MAC addresses.
* Broadcast traffic is flooded.
* Unknown unicasts are flooded.
* Known unicasts are forwarded intelligently.
* CAM tables enable efficient Layer 2 communication.
* Weaknesses in MAC learning can be exploited by attackers.
* Security features exist to defend against these attacks.

---

## Skills Practiced

* Cisco Packet Tracer Simulation Mode
* Packet-by-packet analysis
* ARP observation
* ICMP analysis
* CAM table interpretation
* Layer 2 switching concepts
* Cybersecurity attack awareness

---

## Conclusion

This lab strengthened my understanding of how Ethernet switches operate at Layer 2 of the OSI model. By observing MAC learning and frame forwarding in real time, I gained practical insight into both normal switching operations and the security implications associated with switch behavior. These concepts provide the foundation for advanced networking, traffic analysis, switch security, and cybersecurity investigations.
