# NetSec Lab 19 - DNS (Domain Name System)

## Objective

The objective of this lab is to understand:

- What DNS is
- Why DNS is required
- How DNS resolves domain names into IP addresses
- DNS packet flow
- DNS configuration in Cisco Packet Tracer
- DNS troubleshooting
- DNS interview questions

---

# Lab Topology

```

PC1 --------\
\
PC2 --------- Switch1 -------- Router1 -------- Switch2 -------- DNS Server
/
PC3 --------/

|
+---- www.prashanthposhettiwar.com
|
+---- www.pramodposhettiwar.com

```

---

# Devices Used

| Device | Quantity |
|----------|----------|
| PC | 3 |
| Switch 2960 | 2 |
| Router 2911 | 1 |
| DNS Server | 1 |
| Web Servers | 2 |

---

# IP Addressing Scheme

## LAN Network

Network Address:

```

192.168.1.0/24

```

| Device | IP Address |
|----------|----------|
| Router1 G0/0 | 192.168.1.1 |
| PC1 | 192.168.1.10 |
| PC2 | 192.168.1.11 |
| PC3 | 192.168.1.12 |

---

## Server Network

Network Address:

```

192.168.2.0/24

```

| Device | IP Address |
|----------|----------|
| Router1 G0/1 | 192.168.2.1 |
| DNS Server | 192.168.2.10 |
| Web Server 1 | 192.168.2.20 |
| Web Server 2 | 192.168.2.30 |

---

# Router Configuration

```cisco
enable

configure terminal

interface g0/0
ip address 192.168.1.1 255.255.255.0
no shutdown

interface g0/1
ip address 192.168.2.1 255.255.255.0
no shutdown

end

write memory
```

---

# PC Configuration

## PC1

```
IP Address      : 192.168.1.10
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.1.1
DNS Server      : 192.168.2.10
```

## PC2

```
IP Address      : 192.168.1.11
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.1.1
DNS Server      : 192.168.2.10
```

## PC3

```
IP Address      : 192.168.1.12
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.1.1
DNS Server      : 192.168.2.10
```

---

# DNS Server Configuration

## IP Configuration

```
IP Address      : 192.168.2.10
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.2.1
```

---

## DNS Service

Services → DNS → ON

### DNS Records

| Domain Name | IP Address |
|------------|------------|
| www.prashanthposhettiwar.com | 192.168.2.20 |
| www.pramodposhettiwar.com | 192.168.2.30 |

---

# Web Server 1 Configuration

## IP Configuration

```
IP Address      : 192.168.2.20
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.2.1
DNS Server      : 192.168.2.10
```

### HTTP Service

```
Services → HTTP → ON
```

Website:

```
www.prashanthposhettiwar.com
```

---

# Web Server 2 Configuration

## IP Configuration

```
IP Address      : 192.168.2.30
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.2.1
DNS Server      : 192.168.2.10
```

### HTTP Service

```
Services → HTTP → ON
```

Website:

```
www.pramodposhettiwar.com
```

---

# Connectivity Testing

## Router Reachability

```bash
ping 192.168.1.1
```

Expected Result:

```
Success
```

---

## DNS Server Reachability

```bash
ping 192.168.2.10
```

Expected Result:

```
Success
```

---

## Web Server Reachability

```bash
ping 192.168.2.20
ping 192.168.2.30
```

Expected Result:

```
Success
```

---

# DNS Resolution Testing

## Test Website 1

```bash
ping www.prashanthposhettiwar.com
```

Expected Output:

```bash
Pinging 192.168.2.20
```

---

## Test Website 2

```bash
ping www.pramodposhettiwar.com
```

Expected Output:

```bash
Pinging 192.168.2.30
```

---

# Browser Testing

From PC1:

Desktop → Web Browser

Open:

```
www.prashanthposhettiwar.com
```

Expected:

```
Website loads successfully
```

---

Open:

```
www.pramodposhettiwar.com
```

Expected:

```
Website loads successfully
```

---

# Packet Flow Analysis

## Step 1 – ARP

PC1 does not know the MAC address of the default gateway.

ARP Request:

```
Who has 192.168.1.1?
```

ARP Reply:

```
192.168.1.1 is at xxxx.xxxx.xxxx
```

---

## Step 2 – DNS Query

PC1 sends a DNS request.

```
Source IP      : 192.168.1.10
Destination IP : 192.168.2.10
Protocol       : DNS
Port           : UDP 53
```

Question:

```
What is the IP address of
www.prashanthposhettiwar.com ?
```

---

## Step 3 – DNS Reply

DNS Server replies:

```
www.prashanthposhettiwar.com
=
192.168.2.20
```

---

## Step 4 – HTTP Connection

PC1 now knows the destination IP.

```
192.168.2.20
```

Browser starts communication with the web server.

---

## Step 5 – HTTP Response

Web Server returns webpage contents.

Browser displays the website.

---

# Important Learning

DNS does not transfer website data.

DNS only provides:

```
Domain Name → IP Address
```

After DNS resolution:

- HTTP
- HTTPS
- FTP
- SSH
- SMTP

use the returned IP address.

---

# Troubleshooting

## Problem 1

DNS Server IP not configured on PC.

Result:

```
ping www.prashanthposhettiwar.com
```

Fails.

But:

```
ping 192.168.2.20
```

Works.

---

## Problem 2

Wrong DNS Record

```
www.prashanthposhettiwar.com
→
192.168.2.99
```

DNS resolves successfully.

Website fails.

---

## Problem 3

DNS Service OFF

```
Services → DNS → OFF
```

Result:

```
Name resolution fails.
```

---

## Problem 4

Router Interface Shutdown

```
interface g0/1
shutdown
```

Result:

```
DNS Server unreachable.
```

---

# Interview Questions

### What is DNS?

DNS translates domain names into IP addresses.

### Which port does DNS use?

UDP Port 53

### Can DNS use TCP?

Yes.

TCP Port 53 is commonly used for zone transfers and large responses.

### What happens before opening a website?

1. ARP
2. DNS Query
3. DNS Reply
4. TCP Connection
5. HTTP/HTTPS Communication

### What if DNS fails?

Users can still access resources directly using IP addresses.

---

# Lab Completion Criteria

Successfully:

- Configured Router
- Configured DNS Server
- Created DNS Records
- Configured Two Web Servers
- Verified DNS Resolution
- Verified Browser Access
- Analyzed DNS Packet Flow

Lab Status:

✅ Completed