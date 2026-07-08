# Lab 18 - PAT (NAT Overload) using Cisco Packet Tracer

## Objective

Learn and implement Port Address Translation (PAT), also known as NAT Overload, and understand how multiple private hosts can share a single public IP address using port numbers (or ICMP identifiers).

---

## Why PAT?

Organizations often have many internal devices but only a limited number of public IPv4 addresses.

Example:

- PC1 - 192.168.1.10
- PC2 - 192.168.1.11
- PC3 - 192.168.1.12
- PC4 - 192.168.1.13
- PC5 - 192.168.1.14

Instead of assigning a separate public IP to every device, PAT allows all hosts to use a single public-facing IP address.

PAT is also called **NAT Overload** because many internal hosts overload a single public IP by using different port numbers/identifiers.

---

# Topology

```text
PC1
PC2
PC3
PC4
PC5
  |
Switch1
  |
R1
  |
R2  <-- PAT Configured Here
  |
R3
  |
Switch2
 |   |   |
S1  S2  S3
```

Devices Used:

- 5 PCs
- 2 Switches
- 3 Routers
- 3 Servers

---

# IP Addressing Scheme

## Client LAN

Network: 192.168.1.0/24

| Device | IP Address |
|----------|----------|
| PC1 | 192.168.1.10 |
| PC2 | 192.168.1.11 |
| PC3 | 192.168.1.12 |
| PC4 | 192.168.1.13 |
| PC5 | 192.168.1.14 |
| R1 G0/0 | 192.168.1.1 |

Default Gateway for all PCs:

```text
192.168.1.1
```

---

## R1 - R2 Link

Network: 10.10.10.0/30

| Device | IP Address |
|----------|----------|
| R1 G0/1 | 10.10.10.1 |
| R2 G0/0 | 10.10.10.2 |

---

## R2 - R3 Link

Network: 20.20.20.0/30

| Device | IP Address |
|----------|----------|
| R2 G0/1 | 20.20.20.1 |
| R3 G0/0 | 20.20.20.2 |

---

## Server Network

Network: 100.100.100.0/24

| Device | IP Address |
|----------|----------|
| R3 G0/1 | 100.100.100.1 |
| Server1 | 100.100.100.10 |
| Server2 | 100.100.100.20 |
| Server3 | 100.100.100.30 |

Default Gateway:

```text
100.100.100.1
```

---

# Cabling

## Straight Through

Used between:

- PC -> Switch
- Switch -> Router
- Server -> Switch

## Cross Over

Used between:

- Router -> Router

---

# Router Configuration

## R1

```cisco
enable
configure terminal

interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 10.10.10.1 255.255.255.252
 no shutdown

end
```

---

## R2

```cisco
enable
configure terminal

interface g0/0
 ip address 10.10.10.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 20.20.20.1 255.255.255.252
 no shutdown

end
```

---

## R3

```cisco
enable
configure terminal

interface g0/0
 ip address 20.20.20.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 100.100.100.1 255.255.255.0
 no shutdown

end
```

---

# Static Routing

## R1

```cisco
ip route 100.100.100.0 255.255.255.0 10.10.10.2
```

Reason:

R1 must know how to reach the server network.

---

## R2

```cisco
ip route 192.168.1.0 255.255.255.0 10.10.10.1

ip route 100.100.100.0 255.255.255.0 20.20.20.2
```

Reason:

R2 sits in the middle and must know both directions.

---

## R3

```cisco
ip route 192.168.1.0 255.255.255.0 20.20.20.1
```

Reason:

Return traffic must know how to reach the client LAN.

---

# Connectivity Verification

Commands:

```cisco
show ip route
show ip interface brief
```

Verification performed:

- PC1 -> Server1
- PC2 -> Server1
- PC3 -> Server2
- PC4 -> Server3
- PC5 -> Server1

Routing was verified before configuring PAT.

---

# PAT Configuration

PAT configured on R2.

## Step 1 - Create ACL

```cisco
access-list 1 permit 192.168.1.0 0.0.0.255
```

Purpose:

Identifies inside hosts that should be translated.

---

## Step 2 - Define Inside Interface

```cisco
interface g0/0
 ip nat inside
```

---

## Step 3 - Define Outside Interface

```cisco
interface g0/1
 ip nat outside
```

---

## Step 4 - Enable PAT

```cisco
ip nat inside source list 1 interface g0/1 overload
```

Explanation:

- list 1 -> ACL containing internal hosts
- interface g0/1 -> Use interface IP for translation
- overload -> Enable PAT functionality

---

# PAT Verification

Commands:

```cisco
show ip nat translations
show ip nat statistics
```

Observed Translation Example:

```text
icmp 20.20.20.1:33  192.168.1.10:33  100.100.100.10:33  100.100.100.10:33
```

---

# NAT Terminology Analysis

## Inside Local

```text
192.168.1.10
```

Real private address of PC1.

---

## Inside Global

```text
20.20.20.1
```

Address seen by external devices after PAT.

---

## Outside Local

```text
100.100.100.10
```

Destination as viewed from the inside network.

---

## Outside Global

```text
100.100.100.10
```

Actual address of the destination server.

Since no NAT exists on the server side:

```text
Outside Local = Outside Global
```

---

# Simulation Mode Analysis

Filters Enabled:

- ARP
- ICMP

Filters Disabled:

- All Others

---

## Packet Flow

### Step 1

PC1 creates ICMP Echo Request.

```text
Source      : 192.168.1.10
Destination : 100.100.100.10
```

---

### Step 2

ARP Request generated for default gateway.

```text
Who has 192.168.1.1?
```

---

### Step 3

Packet reaches R1.

No NAT performed.

---

### Step 4

Packet reaches R2.

Inbound PDU:

```text
Source      : 192.168.1.10
Destination : 100.100.100.10
```

---

### Step 5

PAT Translation occurs.

Outbound PDU:

```text
Source      : 20.20.20.1
Destination : 100.100.100.10
```

---

### Step 6

R3 receives translated packet.

R3 only sees:

```text
20.20.20.1
```

It never sees:

```text
192.168.1.10
```

---

### Step 7

Server receives packet.

```text
Source      : 20.20.20.1
Destination : 100.100.100.10
```

---

### Step 8

Server replies.

```text
Source      : 100.100.100.10
Destination : 20.20.20.1
```

---

### Step 9

R2 performs reverse translation.

```text
20.20.20.1
      ↓
192.168.1.10
```

Packet delivered back to PC1.

---

# Key Learning

PAT translates:

```text
Private IP + Port
```

into

```text
Public IP + Different Port
```

Example:

```text
192.168.1.10:1025
      ↓
20.20.20.1:2001

192.168.1.11:1025
      ↓
20.20.20.1:2002
```

Same IP, different identifiers/ports.

---

# PAT vs Dynamic NAT

| Feature | Dynamic NAT | PAT |
|----------|----------|----------|
| Mapping | One-to-One | Many-to-One |
| Public IP Requirement | Multiple | Single |
| Port Translation | No | Yes |
| Real World Usage | Rare | Very Common |

---

# Troubleshooting Checklist

## Verify Interfaces

```cisco
show ip interface brief
```

Expected:

```text
up/up
```

---

## Verify Routes

```cisco
show ip route
```

---

## Verify ACL

```cisco
show access-lists
```

---

## Verify NAT Configuration

```cisco
show run | section nat
```

---

## Verify Translation Table

```cisco
show ip nat translations
```

---

## Verify Statistics

```cisco
show ip nat statistics
```

---

# Common PAT Problems

1. Missing ACL
2. Wrong inside interface
3. Wrong outside interface
4. Missing overload keyword
5. Missing routes
6. Interface shutdown

---

# Interview Questions

## What is PAT?

PAT allows multiple private IP addresses to share a single public IP address using port numbers.

---

## Why is PAT called NAT Overload?

Multiple internal hosts overload one public IP address.

---

## Which command enables PAT?

```cisco
ip nat inside source list 1 interface g0/1 overload
```

---

## Which keyword makes PAT possible?

```text
overload
```

---

## What is Inside Local?

Private address before translation.

Example:

```text
192.168.1.10
```

---

## What is Inside Global?

Translated address after PAT.

Example:

```text
20.20.20.1
```

---

# Lab Conclusion

Successfully implemented PAT (NAT Overload) using:

- 5 Client PCs
- 3 Routers
- 3 Servers
- Static Routing
- ACL-Based PAT
- Simulation Mode Verification
- NAT Translation Table Analysis

Result:

Multiple internal hosts successfully shared a single translated IP address (20.20.20.1) while communicating with the server network.

---

# Next Lab

Lab 19 - Standard and Extended Access Control Lists (ACLs)
