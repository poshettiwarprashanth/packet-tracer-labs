# NetSec Lab 14 – DHCP (Dynamic Host Configuration Protocol) Deep Dive (Same LAN Network)

## Objective
To understand how DHCP works inside a single LAN network using Cisco Packet Tracer, including:
- DHCP configuration on router
- DORA process (Discover, Offer, Request, ACK)
- Broadcast vs Unicast behavior
- Role of switch in DHCP traffic
- ARP interaction during DHCP process
- Real packet flow analysis in Simulation Mode

## Network Setup
- 1 Router (DHCP Server)
- 1 Switch (Layer 2 Device)
- 10 PCs (DHCP Clients)
- Single LAN Network: 192.168.10.0/24

Topology Flow: PCs → Switch → Router  
All devices are in same broadcast domain.

## DHCP Configuration (Router Side)

Step 1: Configure Interface
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

Role: Default Gateway + DHCP Server for LAN

Step 2: Exclude IPs
ip dhcp excluded-address 192.168.10.1 192.168.10.10

Purpose: Prevent gateway/reserved IP allocation

Step 3: DHCP Pool
ip dhcp pool LAN-POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

DHCP Provides:
- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

## DHCP Working (DORA Process)
D → Discover  
O → Offer  
R → Request  
A → ACK

## DHCP Packet Flow

Discover:
Client → Broadcast (0.0.0.0 → 255.255.255.255, MAC FF:FF:FF:FF:FF:FF)
Switch floods to all ports.

Offer:
Router offers IP (example 192.168.10.11)

Request:
Client accepts offered IP via broadcast

ACK:
Router confirms final IP assignment

## DHCP Rule Summary
Discover → Broadcast  
Offer → Broadcast/Unicast  
Request → Broadcast  
ACK → Broadcast/Unicast  

## Role of Switch
- Layer 2 device only
- Uses MAC address table
- Does NOT understand DHCP/IP

Behavior:
- Broadcast → Flood all ports
- Unicast → Forward to specific port

## ARP Interaction in DHCP

Server Side ARP:
Router checks if IP is free:
"Who has 192.168.10.11?"

Purpose:
- Prevent duplicate IPs

Client Side ARP:
Client sends Gratuitous ARP:
"I am 192.168.10.11"

Purpose:
- Detect IP conflict
- Update ARP tables

## Important Observations
- DHCP starts with Discover
- Client begins with 0.0.0.0
- Switch floods broadcast traffic
- Router assigns IP from pool
- ARP used only for validation

## Key Concepts Learned

DHCP Provides:
- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

Rules:
- Client has no IP initially
- DHCP is broadcast heavy
- Switch is Layer 2 only
- Router assigns IP
- ARP prevents conflicts

## Final Summary
DHCP works as a broadcast-based IP allocation system:

1. Client sends Discover
2. Server sends Offer
3. Client sends Request
4. Server sends ACK

After this:
- Client gets full network config
- ARP ensures no conflicts
- Normal communication starts