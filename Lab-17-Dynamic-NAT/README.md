# Lab 17 – Dynamic NAT (Network Address Translation)

## Objective

The objective of this lab is to understand and implement Dynamic NAT using Cisco Packet Tracer. Dynamic NAT allows multiple private IP addresses to be translated to a pool of public IP addresses on a first-come, first-served basis.

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
R2 (NAT Router)
  |
R3
  |
Switch2
 |      |      |
Server1 Server2 Server3
```

---

# IP Addressing Scheme

## Internal LAN

| Device  | IP Address      |
| ------- | --------------- |
| PC1     | 192.168.1.10/24 |
| PC2     | 192.168.1.20/24 |
| PC3     | 192.168.1.30/24 |
| PC4     | 192.168.1.40/24 |
| PC5     | 192.168.1.50/24 |
| R1 G0/0 | 192.168.1.1/24  |

## R1 – R2 Link

| Device | Interface | IP Address    |
| ------ | --------- | ------------- |
| R1     | G0/1      | 10.10.10.1/30 |
| R2     | G0/0      | 10.10.10.2/30 |

## R2 – R3 Link

| Device | Interface | IP Address    |
| ------ | --------- | ------------- |
| R2     | G0/1      | 20.20.20.1/30 |
| R3     | G0/0      | 20.20.20.2/30 |

## Server Network

| Device  | IP Address    |
| ------- | ------------- |
| R3 G0/1 | 200.1.1.1/24  |
| Server1 | 200.1.1.10/24 |
| Server2 | 200.1.1.20/24 |
| Server3 | 200.1.1.30/24 |

---

# Routing Configuration

Routing was verified before configuring NAT.

End-to-end connectivity between PCs and servers was successfully tested.

This confirmed that routing was functioning correctly and any future issue would be related to NAT configuration rather than routing.

---

# Dynamic NAT Configuration

## Step 1 – Create Standard ACL

The ACL is used to identify which source addresses should be translated.

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
```

### Important Note

This ACL is NOT used for packet filtering.

The ACL acts only as a matching mechanism for NAT.

Permit = Translate

Deny = Do Not Translate

The ACL is not applied to any interface using `ip access-group`.

---

## Step 2 – Create NAT Pool

```bash
ip nat pool DYNAMIC_POOL 100.100.100.1 100.100.100.3 netmask 255.255.255.0
```

### NAT Pool

| Public IPs    |
| ------------- |
| 100.100.100.1 |
| 100.100.100.2 |
| 100.100.100.3 |

---

## Step 3 – Bind ACL to NAT Pool

```bash
ip nat inside source list 1 pool DYNAMIC_POOL
```

---

## Step 4 – Configure NAT Interfaces

### Inside Interface

```bash
interface g0/0
 ip nat inside
```

### Outside Interface

```bash
interface g0/1
 ip nat outside
```

---

# NAT Flow Analysis

## Before NAT

```text
Source IP      : 192.168.1.10
Destination IP : 200.1.1.10
```

## After NAT

```text
Source IP      : 100.100.100.1
Destination IP : 200.1.1.10
```

Only the source IP changes.

The destination IP remains unchanged.

---

# NAT Translation Table Example

```text
Inside Local      Inside Global
192.168.1.10  ->  100.100.100.1
192.168.1.20  ->  100.100.100.2
192.168.1.30  ->  100.100.100.3
```

---

# Troubleshooting Performed

## Issue Encountered

After configuring Dynamic NAT, ping requests timed out.

## Root Cause

R3 did not know how to reach the translated public IP addresses:

```text
100.100.100.0/24
```

Although R2 and R3 were directly connected, R3 only knew:

```text
20.20.20.0/30
```

R3 had no route for:

```text
100.100.100.0/24
```

When the server replied to a translated address, R3 dropped the packet because no route existed.

---

## Solution

Configure a route on R3:

```bash
ip route 100.100.100.0 255.255.255.0 20.20.20.1
```

This allowed return traffic to reach R2, which then translated the packets back to the original private addresses.

---

# Verification Commands

```bash
show ip nat translations
```

```bash
show ip nat statistics
```

```bash
show access-lists
```

```bash
show ip route
```

```bash
clear ip nat translation *
```

---

# Key Learning Points

* Dynamic NAT uses a pool of public IP addresses.
* Standard ACLs used by NAT are not filtering ACLs.
* NAT ACLs are used only for matching source networks.
* Dynamic NAT creates translations only when traffic exists.
* Return traffic must have a valid route back to the translated public IP addresses.
* Direct connectivity between routers does not automatically provide routes to NAT pools.
* Routers make forwarding decisions based on destination networks.
* Dynamic NAT can suffer from pool exhaustion.
* Dynamic NAT is less scalable than PAT (NAT Overload).

---

# Interview Questions

### What is Dynamic NAT?

Dynamic NAT translates private addresses to public addresses using a pool of available public IP addresses.

### Difference Between Static NAT and Dynamic NAT?

| Static NAT       | Dynamic NAT                  |
| ---------------- | ---------------------------- |
| Fixed mapping    | Pool-based mapping           |
| Permanent        | Temporary                    |
| One-to-one       | One-to-one from pool         |
| Always available | Depends on pool availability |

### Why was the ACL required?

The ACL identifies which source networks are eligible for translation.

### Does the ACL filter traffic?

No. It only determines whether traffic should be translated.

### Why did the ping fail initially?

R3 had no route to the NAT pool network.

### What happens when all pool addresses are used?

Additional hosts cannot obtain translations until an existing translation expires.

### Why is PAT preferred over Dynamic NAT?

PAT allows many hosts to share a single public IP address using port numbers, making it more efficient.
