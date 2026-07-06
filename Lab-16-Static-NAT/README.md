# Lab 16: Implementing and Verifying Static NAT

This repository contains Cisco Packet Tracer configurations and verification data for **Lab 16: Static NAT**. This lab demonstrates how to configure a one-to-one mapping between a private, internal IP address and a public, external IP address to allow an internal server to communicate securely across a public WAN network.

---

# 1. Network Address Translation (NAT) Overview

## What is NAT?

**Network Address Translation (NAT)** is a method used in networking to map one IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device.

Primarily, NAT allows devices on a private network with non-routable private IP addresses to communicate with external networks like the Internet using public IP addresses.

---

# Why Use NAT?

## IPv4 Address Conservation

The primary driver for NAT deployment was the exhaustion of IPv4 addresses. NAT allows thousands of devices to access the internet using a limited pool of public IP addresses.

## Security & Privacy

NAT hides internal IP addresses from the outside world, preventing external entities from directly accessing internal hosts.

## Network Flexibility

NAT avoids the need to re-address internal hosts if the network changes Internet Service Providers (ISPs).

---

# Types of NAT

| NAT Type | Description | Key Characteristic |
|----------|-------------|--------------------|
| Static NAT | Maps a single private IP address to a single public IP address. | Fixed 1:1 bidirectional mapping |
| Dynamic NAT | Maps private IP addresses to a pool of available public IP addresses dynamically. | Many-to-many dynamic mapping |
| PAT (NAT Overload) | Maps multiple private IP addresses to a single public IP address using different source port numbers. | Many-to-one mapping using Layer 4 ports |

---

# 2. Lab Topology & Addressing Scheme

The network consists of an internal private network (LAN-1), an edge router (Router-1), a simulated Internet/ISP link (WAN-1), and a remote network (LAN-2) managed by Router-2.

---

# Interface IP Assignments

## Router-1 (NAT Edge Router)

Inside Interface:


GigabitEthernet0/0
IP Address: 192.168.10.1/24
Role: Inside NAT Interface


Outside Interface:


GigabitEthernet0/1
IP Address: 209.165.200.225/29
Role: Outside NAT Interface


---

## Router-2 (ISP Router)

WAN Interface:


GigabitEthernet0/1
IP Address: 209.165.200.226/29


External LAN Interface:


GigabitEthernet0/0
IP Address: 203.0.113.1/24


---

## Internal Web Server


IP Address: 192.168.10.40
Default Gateway: 192.168.10.1


---

# NAT Mapping Objective

The internal Web Server:


192.168.10.40


requires a permanent public IP address:


209.165.200.230


Static NAT Mapping:


192.168.10.40 <-----> 209.165.200.230


---

# 3. Router Configurations

## Router-1 (NAT Edge Router)

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 ip address 209.165.200.225 255.255.255.248
 ip nat outside
 duplex auto
 speed auto
!
ip nat inside source static 192.168.10.40 209.165.200.230
!
ip route 0.0.0.0 0.0.0.0 209.165.200.226


## Router-2 (ISP Router)

interface GigabitEthernet0/0
 ip address 203.0.113.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 ip address 209.165.200.226 255.255.255.248
 duplex auto
 speed auto
!
ip route 192.168.10.0 255.255.255.0 209.165.200.225


# Routing Note

In production scenarios, the ISP router would not normally have a static route pointing back to private address space like:

192.168.10.0/24

Instead, traffic would use the public NAT address:

209.165.200.230

This lab uses static routing to simulate ISP connectivity in Cisco Packet Tracer.


# 4. Verification and Statistics

Verification commands are executed on Router-1.


# Active NAT Translation Table

Command:

show ip nat translation


Output:

Pro  Inside global      Inside local       Outside local      Outside global
---  209.165.200.230    192.168.10.40      ---                ---


# NAT Terminology Breakdown


## Inside Local

The actual private IP address assigned to the host inside the network.

Example:

192.168.10.40


## Inside Global

The public IP address assigned to represent the internal host to outside networks.

Example:

209.165.200.230


## Outside Local

The address of an outside host as it appears to the inside network.


## Outside Global

The actual public IP address assigned to the outside host.


# NAT Statistics

Command:

show ip nat statistics


Output:

Total translations: 1 (1 static, 0 dynamic, 0 extended)

Outside Interfaces:

GigabitEthernet0/1

Inside Interfaces:

GigabitEthernet0/0

Hits: 0
Misses: 0

Expired translations: 0


# 5. Frequently Asked Interview Questions


## Q1: What is the main difference between Static NAT and Dynamic NAT?

Answer:

Static NAT creates a permanent fixed 1-to-1 mapping between an internal private IP address and an external public IP address.

Example:

192.168.10.40 = 209.165.200.230


Dynamic NAT maps private IP addresses to a pool of public IP addresses dynamically on a first-come-first-served basis.


## Q2: Why does show ip nat translation show an entry even when no traffic is passing?

Answer:

Because Static NAT creates a permanent translation entry.

The command:

ip nat inside source static

creates a fixed mapping that does not expire.


## Q3: What happens if ip nat inside or ip nat outside commands are missing?

Answer:

The NAT translation will fail.

Router-1 needs:

Inside Interface:

ip nat inside

Outside Interface:

ip nat outside

These commands define the NAT boundary.


# 6. Conclusion

This lab successfully demonstrates the concept and deployment of Static NAT.

A permanent one-to-one translation was created between the internal Web Server:

192.168.10.40

and the public IP address:

209.165.200.230


Static NAT allows internal services such as:

- Web Servers
- Mail Servers
- FTP Servers
- Application Servers

to be reachable from external networks while hiding the original private IP addressing structure.

This lab confirms NAT translation, interface configuration, routing, and verification commands.