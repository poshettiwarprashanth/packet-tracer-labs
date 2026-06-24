# 🧪 Lab 04: CAM Table Aging and Port Security Basics

## 🎯 Lab Objective

By the end of this lab, you will understand:

* How a switch stores MAC addresses
* What Dynamic MAC entries are
* Why switches remove old MAC entries
* What CAM Table Aging is
* What Port Security is
* Why Port Security is important in cybersecurity

---

# 📚 Lab Structure

This lab is divided into two parts:

## Part A – CAM Table Aging

Learn how switches automatically remove inactive MAC addresses from their CAM table.

## Part B – Port Security Basics

Learn how switches restrict unauthorized devices from connecting to the network.

---

# Part A – CAM Table Aging

## 🛠️ Step 1: Create a New Workspace

Open Cisco Packet Tracer and save the file as:

```text
Lab-04-CAM-Table-Aging-and-Port-Security-Basics.pkt
```

---

## 🖥️ Step 2: Build the Topology

Add the following devices:

* 1 × Cisco 2960 Switch
* PC0
* PC1

### Topology Diagram

```text
PC0 -------- Switch0 -------- PC1
```

---

## 🔌 Step 3: Connect the Devices

Use **Copper Straight-Through** cables.

| Device | Interface     | Connected To | Interface |
| ------ | ------------- | ------------ | --------- |
| PC0    | FastEthernet0 | Switch0      | Fa0/1     |
| PC1    | FastEthernet0 | Switch0      | Fa0/2     |

Wait until all link lights turn green.

---

## 🌐 Step 4: Configure IP Addresses

### PC0

```text
IP Address:     192.168.1.10
Subnet Mask:    255.255.255.0
Default Gateway: Leave Blank
```

### PC1

```text
IP Address:     192.168.1.20
Subnet Mask:    255.255.255.0
Default Gateway: Leave Blank
```

---

## 📋 Step 5: Check the Switch MAC Address Table

Open the Switch CLI and execute:

```bash
enable
show mac-address-table
```

### Expected Result

Ideally, the MAC table should be empty.

However, Packet Tracer may already display **DYNAMIC** MAC entries because it sometimes generates Layer 2 traffic automatically.

### Screenshot 1

**Title:** Initial MAC Table State

Capture the output of:

```bash
show mac-address-table
```

---

## 🔍 Step 6: Observe Dynamic Entries

If you see:

```text
Type = DYNAMIC
```

it means the switch learned the MAC addresses automatically from incoming frames.

### Observation

Dynamic MAC entries are learned automatically and stored in the switch's CAM table.

---

## ⏳ Step 7: Perform the CAM Aging Experiment

Do not generate any traffic.

* Do NOT ping.
* Do NOT send packets.
* Simply wait for approximately 5 minutes.

---

## 📋 Step 8: Check the MAC Table Again

Run:

```bash
show mac-address-table
```

### Possible Outcomes

#### Case 1

MAC entries disappear.

#### Case 2

MAC entries remain.

### Lab Result

✅ The dynamic MAC entries disappeared after waiting.

### Screenshot 2

**Title:** CAM Table After Aging

Capture the MAC table showing that the entries have been removed.

---

## 🧠 What Did We Learn?

Switches do not store MAC addresses forever.

Inactive MAC addresses are automatically removed from the CAM table after a certain aging period.

This process is called:

### CAM Table Aging

---

## 🔐 Cybersecurity Importance of CAM Aging

Without CAM aging:

* CAM tables could become full
* Switch memory would be wasted
* Network performance could degrade

CAM aging helps maintain efficient switching operations.

---

# Part B – Port Security Basics

---

## 🔒 Step 9: Why Do We Need Port Security?

Imagine the following situation:

Initially:

```text
PC0 -------- Fa0/1
```

An attacker disconnects PC0 and connects their own device:

```text
Attacker Laptop -------- Fa0/1
```

Without Port Security:

❌ The switch accepts the attacker.

This creates a significant security risk.

---

## 🛡️ Step 10: What Is Port Security?

Port Security allows only approved MAC addresses to communicate through a switch port.

### Example

```text
Port: Fa0/1

Allowed MAC:
00E0.F9E0.0151
```

Any other MAC address causes a security violation.

---

## 🚨 Step 11: Port Security Violation Modes

### Protect

* Unauthorized traffic is dropped
* No alerts are generated

### Restrict

* Unauthorized traffic is dropped
* Logs and counters are generated

### Shutdown (Default)

* Unauthorized MAC address detected
* Port enters Err-Disabled state
* Communication stops immediately

> This is the most commonly asked Port Security interview question.

---

## ⚙️ Step 12: Check Packet Tracer Support

Open the Switch CLI:

```bash
enable
configure terminal
interface fa0/1
switchport port-security ?
```

### Result

If supported:

* Configure and test Port Security practically.

If not supported:

* Learn the concept and document the behavior.

---

## 🎯 Step 13: Cybersecurity Relevance

Port Security helps protect networks against:

### 1. Unauthorized Device Access

Prevents attackers from plugging devices into unused switch ports.

### 2. MAC Flooding Attacks

Attackers generate thousands of fake MAC addresses to overflow the CAM table.

### Attacker Goal

Force the switch to behave like a hub so that traffic can be intercepted.

### Common Defenses

* Port Security
* Sticky MAC Addresses
* Maximum MAC Address Limits

---

# 📖 Key Concepts Learned

✅ MAC Address Learning

✅ Dynamic MAC Entries

✅ CAM Table Aging

✅ Layer 2 Switching Behavior

✅ Port Security Fundamentals

✅ Violation Modes

✅ Unauthorized Device Prevention

✅ MAC Flooding Attack Mitigation

---

# 🎓 Conclusion

In this lab, we explored how switches dynamically learn and age out MAC addresses from the CAM table. We also studied Port Security, one of the most fundamental Layer 2 security mechanisms used to prevent unauthorized devices and defend against MAC flooding attacks.

Understanding CAM Table Aging and Port Security is essential for Network Engineers, SOC Analysts, Security Engineers, and Cybersecurity Professionals because these concepts form the foundation of secure Layer 2 network operations.
