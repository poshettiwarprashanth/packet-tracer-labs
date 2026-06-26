# Lab 06 – VLAN Trunking

## Objective

This lab demonstrates how VLAN traffic can be extended across multiple switches using trunk links. It covers:

* Why VLAN Trunking is required
* Access Port vs Trunk Port
* IEEE 802.1Q VLAN Tagging
* Communication between same VLANs across different switches
* Why different VLANs still cannot communicate without routing

---

## Network Topology

```text
PC0 ---- SW1 ---- SW2 ---- PC2
PC1                  PC3
```

### Devices Used

* 2 × Cisco 2960 Switches
* 4 × PCs

### Connections

| Device | Interface | Connected To |
| ------ | --------- | ------------ |
| PC0    | Fa0       | SW1 Fa0/1    |
| PC1    | Fa0       | SW1 Fa0/2    |
| SW1    | Fa0/3     | SW2 Fa0/3    |
| PC2    | Fa0       | SW2 Fa0/1    |
| PC3    | Fa0       | SW2 Fa0/2    |

---

## IP Address Configuration

| Device | IP Address    | Subnet Mask   |
| ------ | ------------- | ------------- |
| PC0    | 192.168.10.10 | 255.255.255.0 |
| PC1    | 192.168.20.10 | 255.255.255.0 |
| PC2    | 192.168.10.20 | 255.255.255.0 |
| PC3    | 192.168.20.20 | 255.255.255.0 |

**Default Gateway:** Not configured

---

## VLAN Configuration

### VLANs Created

| VLAN ID | Name  |
| ------- | ----- |
| 10      | SALES |
| 20      | HR    |

### Commands

```bash
enable
configure terminal

vlan 10
name SALES

vlan 20
name HR

end
```

---

## Access Port Configuration

### SW1

#### PC0 → VLAN 10

```bash
interface fa0/1
switchport mode access
switchport access vlan 10
```

#### PC1 → VLAN 20

```bash
interface fa0/2
switchport mode access
switchport access vlan 20
```

### SW2

#### PC2 → VLAN 10

```bash
interface fa0/1
switchport mode access
switchport access vlan 10
```

#### PC3 → VLAN 20

```bash
interface fa0/2
switchport mode access
switchport access vlan 20
```

---

## Verification

### Verify VLAN Creation

```bash
show vlan brief
```

Expected Output:

```text
VLAN 10  SALES
VLAN 20  HR
```

### Verify VLAN Assignment

Expected:

```text
VLAN 10 -> Fa0/1
VLAN 20 -> Fa0/2
```

---

## Testing Before Trunking

### Ping Test

From PC0:

```bash
ping 192.168.10.20
```

### Result

❌ Failed

### Reason

Although VLAN 10 exists on both switches, the switch-to-switch link is operating as an access link and does not carry VLAN information between switches.

---

## Trunk Configuration

### SW1

```bash
configure terminal

interface fa0/3
switchport mode trunk

end
```

### SW2

```bash
configure terminal

interface fa0/3
switchport mode trunk

end
```

---

## Verify Trunk

```bash
show interfaces trunk
```

Expected Output:

```text
Port      Mode      Status
Fa0/3     trunk     trunking
```

---

## Testing After Trunking

### VLAN 10 Communication

From PC0:

```bash
ping 192.168.10.20
```

Result:

✅ Success

---

### VLAN 20 Communication

From PC1:

```bash
ping 192.168.20.20
```

Result:

✅ Success

---

### Inter-VLAN Communication

From PC0:

```bash
ping 192.168.20.20
```

Result:

❌ Failed

### Reason

Trunk links only transport VLAN-tagged frames between switches.

They do **not** perform routing.

Communication between different VLANs requires:

* Router-on-a-Stick
* Layer 3 Switch
* Inter-VLAN Routing

---

## Access Port vs Trunk Port

| Access Port                     | Trunk Port                   |
| ------------------------------- | ---------------------------- |
| Carries one VLAN                | Carries multiple VLANs       |
| Connects end devices            | Connects switches            |
| No VLAN tagging                 | Uses IEEE 802.1Q tagging     |
| Used for PCs, Printers, Servers | Used between network devices |

---

## IEEE 802.1Q Tagging

When a frame enters a trunk port:

1. Switch adds a VLAN ID tag.
2. Frame travels across the trunk link.
3. Receiving switch reads the VLAN tag.
4. Frame is forwarded to the correct VLAN.

This allows multiple VLANs to share a single physical cable between switches.

---

## Key Learning Outcomes

* VLANs create separate Layer 2 broadcast domains.
* VLAN information does not automatically cross switches.
* Trunking allows multiple VLANs to traverse a single link.
* IEEE 802.1Q tagging identifies VLAN membership.
* Devices in the same VLAN can communicate across switches after trunking.
* Different VLANs still require routing for communication.

---

## Commands Used

### Create VLANs

```bash
vlan 10
vlan 20
```

### Configure Access Ports

```bash
switchport mode access
switchport access vlan 10
```

### Configure Trunk Ports

```bash
switchport mode trunk
```

### Verification Commands

```bash
show vlan brief
show interfaces trunk
```

---

## Conclusion

### Before Trunking

```text
PC0 (VLAN10) -> PC2 (VLAN10)
FAILED
```

### After Trunking

```text
PC0 (VLAN10) -> PC2 (VLAN10)
SUCCESS
```

### Inter-VLAN Communication

```text
PC0 (VLAN10) -> PC3 (VLAN20)
FAILED
```

Trunking extends VLANs across switches but does not provide routing between VLANs. Inter-VLAN communication requires a Layer 3 device such as a router or Layer 3 switch.
