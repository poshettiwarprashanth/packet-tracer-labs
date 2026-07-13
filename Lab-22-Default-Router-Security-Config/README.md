# Cisco Router Basic Configuration and Security Lab

## Lab Objective

The objective of this lab is to learn how to perform the initial configuration of a Cisco router using the **Console Port**. This includes securing access to the router, understanding Cisco IOS modes, configuring passwords, creating login banners, encrypting passwords, and saving the configuration.

By the end of this lab, you will understand:

* How to connect a PC to a router using a console cable
* Cisco IOS command modes
* Router hostname configuration
* Privileged EXEC mode
* Global Configuration mode
* Console security
* VTY security
* Password encryption
* Login banners
* Saving router configuration

---

# Network Topology

```
PC -------- Console Cable -------- Router
```

The PC is connected directly to the router using a **Console Cable**.

Unlike Ethernet cables, a console cable is **only used for management**. It cannot transfer normal network traffic.

---

# What is the Console Port?

The Console Port is a dedicated management interface on Cisco devices.

It allows an administrator to configure the router even if:

* The router has no IP address
* The router has no network connectivity
* The router is brand new
* Remote access is unavailable

Think of the Console Port as sitting directly in front of the router.

---

# Why Do We Use a Console Cable?

A new router has:

* No IP address
* No SSH
* No Telnet
* No remote management

Since there is no network communication yet, we must physically connect using the Console Cable.

After the initial configuration, remote management (SSH) can be enabled.

---

# Difference Between Console and Ethernet Cable

| Console Cable                             | Ethernet Cable                 |
| ----------------------------------------- | ------------------------------ |
| Used for management                       | Used for network communication |
| Connects PC RS-232 to Router Console Port | Connects Ethernet interfaces   |
| No IP address required                    | Requires network configuration |
| Used for initial setup                    | Used after configuration       |

---

# Opening the Router CLI

After connecting the Console Cable:

1. Click the PC.
2. Open **Desktop**.
3. Click **Terminal**.
4. Keep the default settings:

   * 9600 Baud
   * 8 Data Bits
   * No Parity
   * 1 Stop Bit
   * No Flow Control
5. Click OK.
6. Press Enter.

You should now see:

```
Router>
```

---

# Cisco IOS Modes

Cisco IOS has different command modes.

## User EXEC Mode

```
Router>
```

Purpose:

* Basic monitoring
* Limited commands
* Cannot change configuration

Example:

```
show version
ping
```

---

## Privileged EXEC Mode

Enter:

```
enable
```

Prompt changes:

```
Router#
```

Purpose:

* Access advanced commands
* Save configuration
* Enter configuration mode

---

## Global Configuration Mode

Enter:

```
configure terminal
```

Prompt:

```
Router(config)#
```

Purpose:

Configure the router.

Examples:

* Hostname
* Passwords
* Interfaces
* Routing
* Security

---

# Step 1 - Configure Hostname

Command

```
hostname R1
```

Example

```
Router(config)# hostname R1
```

Result

```
R1(config)#
```

## Why?

A hostname uniquely identifies the router.

Instead of every router showing:

```
Router
```

it can display:

```
R1
BranchRouter
HQRouter
CoreRouter
```

This makes troubleshooting much easier.

---

# Step 2 - Configure Enable Secret

Command

```
enable secret Cisco123
```

Example

```
R1(config)# enable secret Cisco123
```

## Why?

The Enable Secret password protects Privileged EXEC Mode.

Without it, anyone connected to the router could access configuration commands.

When a user types:

```
enable
```

they must enter the secret password.

---

# Enable Password vs Enable Secret

| Enable Password | Enable Secret |
| --------------- | ------------- |
| Older method    | Modern method |
| Weak protection | Encrypted     |
| Less secure     | Highly secure |

Cisco recommends always using **Enable Secret**.

---

# Step 3 - Encrypt Passwords

Command

```
service password-encryption
```

## Why?

Without this command, passwords appear in plain text inside the configuration.

Example:

Before

```
password Cisco123
```

After encryption

```
password 0822455D0A16
```

Although this is not strong encryption, it prevents someone from easily reading passwords.

---

# Step 4 - Configure Console Password

Enter Console Line

```
line console 0
```

Configure

```
password Console123
login
```

Complete Example

```
R1(config)# line console 0
R1(config-line)# password Console123
R1(config-line)# login
```

---

# What is Console Line?

The Console Line controls **physical access** to the router.

Anyone connecting with a Console Cable must authenticate using this password.

---

# Why is it Called "Console 0"?

Cisco routers normally have **one physical console port**.

Therefore the numbering starts at:

```
console 0
```

There is no:

```
console 1
console 2
```

because only one console port exists.

---

# Step 5 - Configure VTY Password

Command

```
line vty 0 4
```

Configure

```
password Cisco123
login
```

Complete Example

```
R1(config)# line vty 0 4
R1(config-line)# password Cisco123
R1(config-line)# login
```

---

# What is VTY?

VTY stands for:

**Virtual Teletype**

It represents **virtual remote login sessions**.

Unlike the Console Port, VTY connections happen over the network.

Examples:

* Telnet
* SSH

---

# What Does "0 4" Mean?

```
line vty 0 4
```

means:

Configure VTY lines:

* VTY 0
* VTY 1
* VTY 2
* VTY 3
* VTY 4

This gives **five simultaneous remote login sessions**.

```
0
1
2
3
4
```

Five users can remotely log into the router at the same time.

Some Cisco devices support more VTY lines, such as `0 15`, allowing up to 16 remote sessions.

---

# Console vs VTY

| Console                | VTY                      |
| ---------------------- | ------------------------ |
| Physical connection    | Remote connection        |
| Console cable          | Network                  |
| Works without IP       | Requires IP connectivity |
| Used for initial setup | Used after configuration |

---

# Step 6 - Configure Login Banner

Command

```
banner motd #Unauthorized Access Prohibited#
```

## Why?

The Message of the Day (MOTD) banner displays before login.

Example:

```
****************************************
Unauthorized Access Prohibited
****************************************
```

Organizations use banners for:

* Legal notice
* Security warning
* Unauthorized access notification

---

# Step 7 - Save Configuration

Command

```
copy running-config startup-config
```

Router asks:

```
Destination filename [startup-config]?
```

Simply press:

```
Enter
```

Result

```
Building configuration...
[OK]
```

---

# Running Configuration vs Startup Configuration

## Running Configuration

Stored in RAM.

* Lost after reboot
* Active configuration

Command:

```
show running-config
```

---

## Startup Configuration

Stored in NVRAM.

* Survives reboot
* Loaded during startup

Command:

```
show startup-config
```

Always save your configuration after making changes.

---

# Complete Configuration

```
enable
configure terminal

hostname R1

enable secret Cisco123

service password-encryption

line console 0
password Console123
login
exit

line vty 0 4
password Cisco123
login
exit

banner motd #Unauthorized Access Prohibited#

end

copy running-config startup-config
```

---

# Commands to Verify Configuration

```
show running-config
show startup-config
show version
show line
show users
```

---

# Key Interview Questions

1. What is the purpose of a console port?
2. Why is a console cable required for initial router setup?
3. What is the difference between User EXEC and Privileged EXEC modes?
4. Why is `enable secret` preferred over `enable password`?
5. What does `service password-encryption` do?
6. What is the purpose of `line console 0`?
7. What is VTY and why is it used?
8. What does `line vty 0 4` mean?
9. What is the difference between the Console Line and VTY Lines?
10. Why is an MOTD banner important?
11. What is the difference between the running configuration and the startup configuration?
12. What happens if you forget to save the configuration before rebooting the router?

---

# Lab Summary

In this lab, you performed the initial configuration of a Cisco router using the Console Port. You learned how Cisco IOS command modes work, secured local and remote administrative access using console and VTY passwords, protected privileged access with an encrypted enable secret, enabled password encryption, configured a login banner, and saved the running configuration to NVRAM. These foundational tasks are essential for preparing a router for secure deployment and are among the first steps performed by network and cybersecurity professionals when provisioning Cisco devices.
