# Lab 02 - IP Configuration and First Ping

## Objective

The objective of this lab is to manually configure IPv4 addresses, understand subnet masks, perform the first network ping, and observe ARP and ICMP packet behavior using Cisco Packet Tracer Simulation Mode.

---

## Topology

PC0 -------- Switch -------- PC1

### Devices Used

* 2 PCs
* 1 Cisco 2960 Switch
* Copper Straight-Through Cables

---

## Physical Connections

| Device | Interface     | Connected To |
| ------ | ------------- | ------------ |
| PC0    | FastEthernet0 | Switch Fa0/1 |
| PC1    | FastEthernet0 | Switch Fa0/2 |

---

## IP Address Configuration

### PC0

| Parameter       | Value          |
| --------------- | -------------- |
| IP Address      | 192.168.1.10   |
| Subnet Mask     | 255.255.255.0  |
| Default Gateway | Not Configured |

### PC1

| Parameter       | Value          |
| --------------- | -------------- |
| IP Address      | 192.168.1.20   |
| Subnet Mask     | 255.255.255.0  |
| Default Gateway | Not Configured |

---

## Network Analysis

Subnet Mask:

255.255.255.0

Network ID:

192.168.1.0/24

Both hosts belong to the same network:

* PC0 → 192.168.1.10
* PC1 → 192.168.1.20

Therefore, communication can occur directly through the switch without requiring a router.

---

## Ping Test

Command executed on PC0:

ping 192.168.1.20

Result:

Successful communication between PC0 and PC1.

---

## Simulation Mode Analysis

### Enabled Protocols

* ARP
* ICMP

### Observations

1. PC0 first generated an ARP Request.
2. The switch flooded the ARP Request to all ports.
3. PC1 responded with an ARP Reply containing its MAC address.
4. PC0 updated its ARP table.
5. ICMP Echo Request was sent.
6. PC1 responded with an ICMP Echo Reply.
7. Communication was successfully established.

---

## Key Concepts Learned

* IPv4 Address Configuration
* Subnet Masks
* Same-Network Communication
* ARP Request and Reply
* ICMP Echo Request and Echo Reply
* Simulation Mode Packet Analysis
* Layer 2 Switching Fundamentals

---

## Security Perspective

Before devices can communicate using IP addresses, they must first discover the destination MAC address using ARP.

This process is important for cybersecurity professionals because ARP is commonly targeted in attacks such as:

* ARP Spoofing
* ARP Poisoning
* Man-in-the-Middle (MITM) Attacks

Understanding normal ARP behavior is the first step toward identifying malicious activity in enterprise networks.

---

## Outcome

Successfully configured IPv4 addresses, verified connectivity using ping, and analyzed ARP and ICMP packet exchanges using Simulation Mode in Cisco Packet Tracer.
