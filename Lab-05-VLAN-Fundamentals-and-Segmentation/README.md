# 🧪 Lab 05: VLAN Fundamentals and Network Segmentation

## 📌 Lab Objective

By the end of this lab, you will understand:

* What a VLAN (Virtual Local Area Network) is
* Why VLANs are used in enterprise networks
* The concept of broadcast domains
* How to create VLANs on a switch
* How to assign switch ports to VLANs
* Why devices in different VLANs cannot communicate without routing
* How VLANs contribute to network security and segmentation

---

# 🌐 Why This Lab Matters

In real-world organizations, all devices are not placed in a single network.

Different departments such as:

* Human Resources (HR)
* Finance
* Information Technology (IT)
* Management

are commonly separated using VLANs.

VLANs help organizations:

✅ Reduce unnecessary broadcast traffic

✅ Improve network performance

✅ Increase security through segmentation

✅ Enforce access control policies

✅ Limit attacker movement across the network

As a Cybersecurity Professional, VLANs are one of the first and most important layers of network defense.

---

# 🖥️ Network Topology

```text
PC0 --------\
             \
              Switch0
             /
PC1 --------/

PC2 --------\
             \
              Switch0
             /
PC3 --------/
```

---

# 📋 VLAN Assignment

## VLAN 10 – HR Department

| Device | VLAN    |
| ------ | ------- |
| PC0    | VLAN 10 |
| PC1    | VLAN 10 |

---

## VLAN 20 – IT Department

| Device | VLAN    |
| ------ | ------- |
| PC2    | VLAN 20 |
| PC3    | VLAN 20 |

---

# 🌍 IP Addressing Scheme

## VLAN 10 (HR)

| Device | IP Address       |
| ------ | ---------------- |
| PC0    | 192.168.10.10/24 |
| PC1    | 192.168.10.20/24 |

---

## VLAN 20 (IT)

| Device | IP Address       |
| ------ | ---------------- |
| PC2    | 192.168.20.10/24 |
| PC3    | 192.168.20.20/24 |

> **Note:** Default Gateway can be left blank for this lab because inter-VLAN routing is not configured.

---

# ⚙️ Task 1: Create VLANs

Enter configuration mode and create VLAN 10 and VLAN 20.

```bash
enable
configure terminal

vlan 10
name HR

vlan 20
name IT
```

---

# ⚙️ Task 2: Assign Switch Ports to VLANs

### Port Mapping

| Device | Switch Port |
| ------ | ----------- |
| PC0    | Fa0/1       |
| PC1    | Fa0/2       |
| PC2    | Fa0/3       |
| PC3    | Fa0/4       |

Configure VLAN assignments:

```bash
interface range fa0/1-2
switchport mode access
switchport access vlan 10

interface range fa0/3-4
switchport mode access
switchport access vlan 20
```

---

# ⚙️ Task 3: Verify VLAN Configuration

Use:

```bash
show vlan brief
```

Expected output:

* VLAN 10 contains Fa0/1 and Fa0/2
* VLAN 20 contains Fa0/3 and Fa0/4

---

# ⚙️ Task 4: Test Connectivity

## Same VLAN Communication

### From PC0

```bash
ping 192.168.10.20
```

Expected Result:

```text
✅ Ping Successful
```

---

### From PC2

```bash
ping 192.168.20.20
```

Expected Result:

```text
✅ Ping Successful
```

---

## Different VLAN Communication

### From PC0

```bash
ping 192.168.20.10
```

Expected Result:

```text
❌ Request Timed Out
```

Devices located in different VLANs cannot communicate because no Layer 3 routing device is present.

---

# 🧠 Key Concept: Broadcast Domains

### Before VLANs

```text
One Switch
=
One Broadcast Domain
```

Every device receives broadcast traffic.

---

### After VLANs

```text
One Switch
=
Multiple Broadcast Domains
```

Each VLAN becomes its own logical network.

Broadcast traffic generated inside one VLAN remains within that VLAN.

---

# 🔒 Cybersecurity Perspective

VLANs are a fundamental network security control.

They help organizations:

* Isolate sensitive departments
* Reduce attack surfaces
* Limit lateral movement after compromise
* Protect critical systems from unauthorized access
* Improve network management and monitoring
* Support Zero Trust and Defense-in-Depth strategies

### Example

If an attacker compromises a device in the HR VLAN, VLAN segmentation can prevent direct access to systems located in the IT VLAN.

This significantly reduces the impact of a security breach.

---

# 📚 Key Takeaways

✔ VLANs logically divide a physical switch into multiple networks

✔ Each VLAN represents a separate broadcast domain

✔ Devices in the same VLAN can communicate directly

✔ Devices in different VLANs require routing to communicate

✔ VLANs improve both security and network performance

✔ Network segmentation is a core cybersecurity principle

---

## 🛠️ Tools Used

* Cisco Packet Tracer
* Layer 2 Switch
* End Devices (PCs)
* VLAN Configuration Commands

---

## 🎯 Skills Practiced

* VLAN Creation
* VLAN Port Assignment
* Network Segmentation
* Broadcast Domain Analysis
* Basic Switch Configuration
* Cybersecurity Network Design Fundamentals

---

### Author

**Prashanth Poshettiwar**

Cybersecurity & Networking Enthusiast

Building hands-on networking and cybersecurity labs using Cisco Packet Tracer while learning network security fundamentals from the ground up.
