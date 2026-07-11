# Lab 21 – Named Extended ACL (Cisco Packet Tracer)

## Objective

The objective of this lab is to learn and implement Named Extended Access Control Lists (ACLs) in Cisco Packet Tracer. Unlike Standard ACLs, Extended ACLs can filter traffic based on:

- Source IP Address
- Destination IP Address
- Protocol Type (ICMP, TCP, UDP)
- Port Numbers

This lab demonstrates multiple real-world scenarios including server access restrictions, ICMP filtering, HTTP/Telnet control, and web service filtering.

---

# Topology

```text
                    Server1 (Web Server)
                    192.168.100.10
                           |
                    Server2 (DNS Server)
                    192.168.100.20
                           |
                    Server3 (FTP Server)
                    192.168.100.30
                           |
                        Switch4
                           |
                        Router4
                     192.168.100.1
                           |
                    10.0.24.2/30
                           |
                           |
Router1 -------- Router2 -------- Router3
10.0.12.1       10.0.12.2       10.0.23.2
192.168.10.1    10.0.23.1       192.168.30.1
     |               |                |
   Switch1         Switch2         Switch3
     |               |                |
 PC1,PC2         PC3,PC4         PC5,PC6

Network 1 : 192.168.10.0/24
Network 2 : 192.168.20.0/24
Network 3 : 192.168.30.0/24
Server Network : 192.168.100.0/24
```

---

# IP Addressing Table

## Router Interfaces

| Device | Interface | IP Address |
|----------|----------|----------|
| R1 | G0/0 | 192.168.10.1/24 |
| R1 | S0/0/0 | 10.0.12.1/30 |
| R2 | G0/0 | 192.168.20.1/24 |
| R2 | S0/0/0 | 10.0.12.2/30 |
| R2 | S0/1/0 | 10.0.23.1/30 |
| R2 | S0/0/1 | 10.0.24.1/30 |
| R3 | G0/0 | 192.168.30.1/24 |
| R3 | S0/0/0 | 10.0.23.2/30 |
| R4 | G0/0 | 192.168.100.1/24 |
| R4 | S0/0/1 | 10.0.24.2/30 |

---

## End Devices

| Device | IP Address | Gateway |
|----------|----------|----------|
| PC1 | 192.168.10.10 | 192.168.10.1 |
| PC2 | 192.168.10.20 | 192.168.10.1 |
| PC3 | 192.168.20.10 | 192.168.20.1 |
| PC4 | 192.168.20.20 | 192.168.20.1 |
| PC5 | 192.168.30.10 | 192.168.30.1 |
| PC6 | 192.168.30.20 | 192.168.30.1 |

---

## Servers

| Server | IP Address |
|----------|----------|
| Server1 | 192.168.100.10 |
| Server2 | 192.168.100.20 |
| Server3 | 192.168.100.30 |

---

# Routing Configuration

Static routes were configured to provide end-to-end connectivity.

## Router1

```bash
ip route 0.0.0.0 0.0.0.0 10.0.12.2
```

## Router3

```bash
ip route 0.0.0.0 0.0.0.0 10.0.23.1
```

## Router4

```bash
ip route 0.0.0.0 0.0.0.0 10.0.24.1
```

## Router2

```bash
ip route 192.168.10.0 255.255.255.0 10.0.12.1

ip route 192.168.30.0 255.255.255.0 10.0.23.2

ip route 192.168.100.0 255.255.255.0 10.0.24.2
```

---

# Pre-Lab Verification

Before implementing ACLs, connectivity was verified.

```bash
ping 192.168.100.10
ping 192.168.100.20
ping 192.168.100.30
```

All devices successfully communicated with the servers.

---

# Scenario 1
# Server Access Control

## Requirement

PC1 should:

- Access Server1
- Access Server2
- Not Access Server3

All other users should have full access.

---

## Configuration

```bash
ip access-list extended SERVER_CONTROL

permit ip host 192.168.10.10 host 192.168.100.10

permit ip host 192.168.10.10 host 192.168.100.20

deny ip host 192.168.10.10 host 192.168.100.30

permit ip any any

interface g0/0

ip access-group SERVER_CONTROL in
```

---

## Verification

### PC1

```bash
ping 192.168.100.10
```

Success

```bash
ping 192.168.100.20
```

Success

```bash
ping 192.168.100.30
```

Failed

### Other PCs

Successfully accessed all servers.

---

# Scenario 2
# ICMP Filtering

## Requirement

PC1 should:

- Ping Server1
- Not Ping Server2
- Not Ping Server3

Everyone else should be allowed.

---

## Configuration

```bash
ip access-list extended ICMP_CONTROL

permit icmp host 192.168.10.10 host 192.168.100.10

deny icmp host 192.168.10.10 host 192.168.100.20

deny icmp host 192.168.10.10 host 192.168.100.30

permit ip any any

interface g0/0

ip access-group ICMP_CONTROL in
```

---

## Verification

### PC1

```bash
ping 192.168.100.10
```

Success

```bash
ping 192.168.100.20
```

Failed

```bash
ping 192.168.100.30
```

Failed

---

# Scenario 3
# Allow HTTP and Block Telnet

## Requirement

PC1 should:

- Access Server1 Web Service
- Not Access Server1 via Telnet

---

## Server Configuration

### Enable HTTP

```text
Server1
Services
HTTP
ON
```

### Enable Telnet

```text
Server1
Services
Telnet
ON
Username : admin
Password : cisco
```

---

## ACL Configuration

```bash
ip access-list extended WEB_POLICY

permit tcp host 192.168.10.10 host 192.168.100.10 eq 80

deny tcp host 192.168.10.10 host 192.168.100.10 eq 23

permit ip any any

interface g0/0

ip access-group WEB_POLICY in
```

---

## Verification

### HTTP Test

```text
http://192.168.100.10
```

Web page opened successfully.

### Telnet Test

```bash
telnet 192.168.100.10
```

Connection failed.

---

# Scenario 4
# Block HTTP and Allow HTTPS & SSH

## Requirement

PC1 should:

- Not Access HTTP
- Access HTTPS
- Access SSH

---

## ACL Configuration

```bash
ip access-list extended WEB_POLICY

deny tcp host 192.168.10.10 host 192.168.100.10 eq 80

permit tcp host 192.168.10.10 host 192.168.100.10 eq 443

permit tcp host 192.168.10.10 host 192.168.100.10 eq 22

permit ip any any

interface g0/0

ip access-group WEB_POLICY in
```

---

## Verification

### HTTP

```text
http://192.168.100.10
```

Blocked

### HTTPS

```text
https://192.168.100.10
```

Allowed

### SSH

```bash
ssh -l admin 192.168.100.10
```

Allowed

---

# ACL Verification Commands

## Display ACL Entries

```bash
show access-lists
```

## Display Interface ACL Assignment

```bash
show ip interface g0/0
```

## Display Running Configuration

```bash
show running-config
```

---

# Important Concepts Learned

## Standard ACL

Filters only:

```text
Source IP Address
```

---

## Extended ACL

Filters using:

```text
Source IP Address
Destination IP Address
Protocol
Port Number
```

---

## ACL Placement Rule

### Standard ACL

```text
Near Destination
```

### Extended ACL

```text
Near Source
```

In this lab, ACLs were applied on:

```text
Router1
Interface G0/0
Inbound Direction
```

because traffic originated from the 192.168.10.0/24 network.

---

# Common Port Numbers

| Service | Port |
|----------|----------|
| HTTP | 80 |
| HTTPS | 443 |
| FTP | 21 |
| SSH | 22 |
| Telnet | 23 |
| DNS | 53 |
| SMTP | 25 |

---

# Key Interview Questions

### Why use Extended ACL instead of Standard ACL?

Extended ACL allows filtering based on source IP, destination IP, protocol type, and port numbers.

---

### Why place Extended ACL near the source?

To stop unwanted traffic as early as possible and avoid unnecessary routing.

---

### What is the difference between these commands?

```bash
permit ip any any
```

Allows all IP traffic.

```bash
permit icmp any any
```

Allows only ping-related traffic.

```bash
permit tcp any any
```

Allows only TCP traffic.

---

### What is an Implicit Deny?

Every ACL automatically ends with:

```bash
deny ip any any
```

even if it is not visible.

Any traffic that does not match a permit statement is automatically dropped.

---

# Lab Conclusion

In this lab, Named Extended ACLs were configured and tested using multiple real-world scenarios. The lab covered source and destination filtering, ICMP filtering, service-specific access control, HTTP/HTTPS filtering, SSH access, Telnet blocking, ACL placement strategies, and verification techniques. By the end of the lab, a complete understanding of Extended ACL operation, configuration, troubleshooting, and best practices was achieved.