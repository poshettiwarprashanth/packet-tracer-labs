# NetSec Lab 12 – Standard ACL (Access Control List)

## Objective

The objective of this lab was to understand how Standard Access Control Lists (ACLs) work in Cisco networks and how they can be used to control network traffic based on the source IP address.

By the end of this lab, I was able to:

* Understand what an ACL is.
* Understand why ACLs are used in networking and cybersecurity.
* Configure a Standard ACL on a Cisco router.
* Permit and deny traffic based on source IP addresses.
* Understand wildcard masks.
* Understand the concepts of `host` and `any`.
* Learn how ACL rules are processed.
* Understand the "First Match Wins" principle.
* Verify ACL functionality using ping tests and router commands.

---
# Understanding Access Control Lists (ACLs)

## What is an ACL?

**ACL = Access Control List**

Think of an ACL as a security guard standing at a router interface.

When a packet arrives, the router checks:

* Should I allow this packet?
* Should I block this packet?

The ACL contains rules that answer these questions.

---

## Real-Life Example

Imagine a company building with a security guard.

Security Rules:

* Employee → Allow
* Visitor → Allow
* Blacklisted Person → Deny

ACLs work in exactly the same way for network traffic.

---

## Why is ACL Needed?

Without an ACL:

```text
PC1 → Server ✓
PC2 → Server ✓
PC3 → Server ✓
```

Any device can communicate with any other device.

This can be dangerous.

Examples:

* HR Server should only be accessed by HR PCs.
* Student PCs should not access Admin PCs.
* Guest Network should only access the Internet.

ACLs solve these problems by controlling who can access what.

---

## What Can ACL Do?

ACLs can:

* Allow traffic
* Deny traffic
* Restrict network access
* Improve security
* Control routing traffic
* Filter packets

---

## Where is ACL Applied?

ACLs are typically applied on router interfaces.

### Example

```text
PC ---- Router ---- Server
```

The router checks packets against ACL rules before forwarding them.

---

# ACL Types

There are two major types of ACLs:

## 1. Standard ACL

A Standard ACL filters traffic using only the **Source IP Address**.

### Example

```text
Allow 192.168.1.10
Deny 192.168.1.20
```

A Standard ACL does **not** care about:

* Destination IP Address
* Port Number
* Protocol

It only checks the source IP.

---

## 2. Extended ACL

An Extended ACL filters traffic using:

* Source IP Address
* Destination IP Address
* Protocol
* Port Number

### Example

```text
Allow HTTP
Deny FTP
Allow DNS
```

Extended ACLs provide much more granular control than Standard ACLs.

---

# How ACL Thinks

Suppose an ACL contains the following rules:

```text
Allow PC1
Deny PC2
```

### Packet from PC1

```text
Matched
Allow
```

Result:

```text
Packet Forwarded
```

### Packet from PC2

```text
Matched
Deny
```

Result:

```text
Packet Dropped
```

---

# ACL Rule Processing

ACL rules are processed from:

```text
TOP → DOWN
```

### Example

```text
1 Permit 192.168.1.10
2 Deny 192.168.1.20
3 Permit Any
```

The router checks:

1. Rule 1
2. Rule 2
3. Rule 3

Once a match is found:

```text
STOP
```

No further rules are checked.

---

## First Match Wins

This is one of the most important ACL concepts.

Once a packet matches a rule:

```text
Permit → Stop
Deny → Stop
```

The router does not check any remaining ACL entries.

---

# Implicit Deny

Every ACL automatically contains:

```text
deny any
```

at the end.

Even if you do not configure it.

### Example

Configured ACL:

```bash
access-list 10 permit 192.168.1.10
```

Actual Behavior:

```bash
access-list 10 permit 192.168.1.10
deny any
```

Any traffic that does not match a permit statement is automatically denied.

---

# Standard ACL Number Range

Cisco reserves the following ranges for Standard ACLs:

```text
1 - 99
1300 - 1999
```

### Example

```bash
access-list 10 permit any
```

ACL Number:

```text
10
```

Therefore, it is a Standard ACL.

---

# Standard ACL Syntax

### Permit

```bash
access-list <ACL_NUMBER> permit <SOURCE>
```

Example:

```bash
access-list 10 permit 192.168.1.10
```

### Deny

```bash
access-list 10 deny 192.168.1.20
```

---

# Wildcard Masks

ACLs use **Wildcard Masks**, not subnet masks.

---

## Wildcard Logic

```text
0 = Must Match
1 = Ignore
```

Another way to remember:

```text
0 = Check
1 = Don't Care
```

---

## Match a Single Host

```bash
192.168.1.10 0.0.0.0
```

Meaning:

```text
Match exactly 192.168.1.10
```

Only that host will match.

---

## Match an Entire Network

```bash
192.168.1.0 0.0.0.255
```

Meaning:

```text
Match all addresses in 192.168.1.0/24
```

Examples:

```text
192.168.1.1
192.168.1.10
192.168.1.20
192.168.1.100
```

All will match.

---

# Shortcut Keywords

## Host

Instead of:

```bash
192.168.1.10 0.0.0.0
```

Cisco allows:

```bash
host 192.168.1.10
```

Both are identical.

---

## Any

Instead of:

```bash
0.0.0.0 255.255.255.255
```

Cisco allows:

```bash
any
```

Both are identical.

---

# ACL Placement Rule

For Standard ACLs:

> Place the ACL as close to the destination as possible.

### Why?

Because Standard ACLs only know the source IP address.

If a Standard ACL is placed too close to the source, it may unintentionally block traffic destined for multiple networks.

Placing it close to the destination allows more precise traffic filtering.

---

# Key Takeaways

* ACL = Access Control List
* ACLs act like security guards for network traffic.
* Standard ACLs filter using only the Source IP Address.
* ACL rules are processed from top to bottom.
* First Match Wins.
* Every ACL has an implicit `deny any`.
* Wildcard masks use:

  * `0 = Must Match`
  * `1 = Ignore`
* `host` = Single IP Address
* `any` = Any IP Address
* Standard ACLs should be placed close to the destination.



## Network Topology

```text
PC1 (192.168.1.10)
        \
         \
          Switch1 ---- Router1 ---- Switch2 ---- Server1 (192.168.2.10)
         /
        /
PC2 (192.168.1.20)
```

### IP Addressing

| Device      | IP Address   |
| ----------- | ------------ |
| PC1         | 192.168.1.10 |
| PC2         | 192.168.1.20 |
| Router G0/0 | 192.168.1.1  |
| Router G0/1 | 192.168.2.1  |
| Server1     | 192.168.2.10 |

---

## Initial Connectivity Test

Before configuring the ACL:

* PC1 could ping Server1 successfully.
* PC2 could ping Server1 successfully.

This confirmed that routing and IP addressing were configured correctly.

---

## ACL Goal

Allow:

```text
PC1 → Server1
```

Deny:

```text
PC2 → Server1
```

---

## ACL Configuration

### Create Standard ACL

```bash
access-list 10 deny host 192.168.1.20
access-list 10 permit any
```

### Apply ACL to Router Interface

```bash
interface g0/1
ip access-group 10 out
```

### Save Configuration

```bash
end
write memory
```

---

## Verification Commands

### View ACL

```bash
show access-lists
```

### Verify Interface ACL

```bash
show ip interface g0/1
```

### View Running Configuration

```bash
show running-config
```

---

## ACL Processing Logic

ACL rules are processed from top to bottom.

```text
Rule 1 -> deny host 192.168.1.20
Rule 2 -> permit any
```

### Packet from PC1

```text
Source IP = 192.168.1.10
```

Rule 1:

```text
No Match
```

Rule 2:

```text
Match
```

Result:

```text
PERMIT
```

---

### Packet from PC2

```text
Source IP = 192.168.1.20
```

Rule 1:

```text
Match
```

Result:

```text
DENY
```

The router stops processing further rules.

---

## Key Concepts Learned

### 1. Standard ACL

A Standard ACL filters traffic using only the source IP address.

It does not inspect:

* Destination IP
* Protocol
* Port Number

---

### 2. First Match Wins

ACLs are processed from top to bottom.

Once a packet matches a rule:

```text
Permit -> Stop
Deny -> Stop
```

No further ACL entries are checked.

---

### 3. Implicit Deny

Every ACL automatically ends with:

```text
deny any
```

even if it is not visible.

Example:

```bash
access-list 10 deny host 192.168.1.20
```

Actually behaves as:

```bash
access-list 10 deny host 192.168.1.20
deny any
```

---

### 4. Wildcard Masks

ACLs use wildcard masks instead of subnet masks.

Rule:

```text
0 = Must Match
1 = Ignore
```

Examples:

#### Match One Host

```bash
192.168.1.20 0.0.0.0
```

Equivalent:

```bash
host 192.168.1.20
```

---

#### Match Entire Network

```bash
192.168.1.0 0.0.0.255
```

Matches:

```text
192.168.1.1
192.168.1.10
192.168.1.20
192.168.1.100
```

---

#### Match Any IP

```bash
0.0.0.0 255.255.255.255
```

Equivalent:

```bash
any
```

---

### 5. ACL Placement Rule

Standard ACLs should be placed as close to the destination as possible.

In this lab:

```bash
interface g0/1
ip access-group 10 out
```

was applied on the interface closest to the server network.

---

## Challenges Faced

### Challenge 1: Understanding Wildcard Masks

Initially, I was confused between:

```bash
host 192.168.1.20
```

and

```bash
192.168.1.0 0.0.0.255
```

I learned that:

* `host` matches a single IP address.
* `0.0.0.255` matches an entire /24 network.

---

### Challenge 2: Additional ACL Entry

I accidentally created:

```bash
access-list 10 deny 192.168.1.0 0.0.0.255
```

which denied the entire network instead of a single host.

This helped me understand how ACL matching works.

---

### Challenge 3: Understanding Rule Order

I learned that ACLs are processed sequentially.

The order of rules directly affects network behavior.

A wrongly placed rule can block more traffic than intended.

---

### Challenge 4: Understanding 'any'

I learned that:

```bash
permit any
```

is equivalent to:

```bash
permit 0.0.0.0 255.255.255.255
```

which means permit all source IP addresses.

---

## Final Results

### Before ACL

```text
PC1 -> Server1 = Success
PC2 -> Server1 = Success
```

### After ACL

```text
PC1 -> Server1 = Success
PC2 -> Server1 = Failed
```

The Standard ACL successfully blocked PC2 while allowing PC1 to access the server.

---

## Conclusion

This lab provided hands-on experience with Cisco Standard ACLs and demonstrated how routers can be used to filter traffic based on source IP addresses. I learned ACL creation, wildcard masks, ACL placement, rule processing, implicit deny behavior, and troubleshooting techniques. This lab forms the foundation for more advanced traffic filtering concepts that will be explored later using Extended ACLs.
