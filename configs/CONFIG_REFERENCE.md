# 📚 Configuration Reference Guide

Complete breakdown of every command used in the AoF network topology.

---

## 📋 Table of Contents

1. [Interface Configuration Overview](#interface-configuration-overview)
2. [IP Addressing Scheme](#ip-addressing-scheme)
3. [Routing Configuration Details](#routing-configuration-details)
4. [DHCP Pool Setup](#dhcp-pool-setup)
5. [VLAN Configuration](#vlan-configuration)
6. [Access Control Lists (ACLs)](#access-control-lists)
7. [Verification Commands](#verification-commands)

---

## Interface Configuration Overview

### What is an Interface?

In Cisco routers, interfaces are the connection points to networks:
- **FastEthernet (fa0/0):** LAN interface connecting to local switch
- **Serial (se2/0, se3/0, etc.):** WAN interfaces connecting to other routers

### FastEthernet Interfaces (LAN)

Each router has one FastEthernet interface connecting to its local LAN:

```
interface FastEthernet0/0
 ip address [ROUTER_IP] [SUBNET_MASK]
 no shutdown          # Enable the interface (activate it)
exit
```

| Router | fa0/0 IP | Subnet | Role |
|---|---|---|---|
| Dhaka | 172.16.0.1 | 172.16.0.0/19 | HQ LAN gateway |
| Paris | 172.16.32.1 | 172.16.32.0/20 | Branch LAN gateway |
| Bucharest | 172.16.56.1 | 172.16.56.0/22 | Branch LAN gateway |
| California | 172.16.60.1 | 172.16.60.0/23 | Branch LAN gateway |
| Lisbon | 172.16.62.1 | 172.16.62.0/24 | Branch LAN gateway |
| Tokyo | 172.16.48.1 | 172.16.48.0/21 | Branch LAN gateway |

### Serial Interfaces (WAN)

Serial interfaces create point-to-point links between routers. Each link uses a `/30` subnet (4 addresses: 2 usable, 1 network, 1 broadcast):

```
interface Serial[NUMBER]/0
 ip address [THIS_ROUTER_IP] [SUBNET_MASK]
 no shutdown          # Enable the WAN link
exit
```

**Example: Dhaka to California link**
```
interface Serial2/0
 ip address 172.16.63.17 255.255.255.252  # Dhaka's end
 no shutdown
exit
```
California's end of the same link:
```
interface Serial2/0
 ip address 172.16.63.18 255.255.255.252  # California's end
 no shutdown
exit
```

**All WAN Links in the Network:**

| Link | Network | Router A (IP) | Router B (IP) | Router B Location |
|---|---|---|---|---|
| Dhaka→Bucharest | 172.16.63.0/30 | 172.16.63.1 | 172.16.63.2 | Bucharest (se2/0) |
| Dhaka→Paris | 172.16.63.8/30 | 172.16.63.9 | 172.16.63.10 | Paris (se2/0) |
| Dhaka→California | 172.16.63.16/30 | 172.16.63.17 | 172.16.63.18 | California (se2/0) |
| California→Lisbon | 172.16.63.32/30 | 172.16.63.33 | 172.16.63.34 | Lisbon (se2/0) |
| California→Tokyo | 172.16.63.24/30 | 172.16.63.25 | 172.16.63.26 | Tokyo (se2/0) |

> **Why /30?** A /30 subnet has only 4 addresses, perfect for point-to-point links where only 2 routers need an IP address.

---

## IP Addressing Scheme

### Class B Private Network

**Network Block:** `172.16.0.0/16`
- This is a Class B private network (reserved for internal use)
- Total addresses: 65,536
- Subnet mask: `255.255.0.0`

### VLSM (Variable Length Subnet Masking)

The network is divided into subnets of different sizes based on host count requirements:

```
Large Subnets (for branches with many hosts):
- Paris (/20): 172.16.32.0 — 4,094 hosts
- Tokyo (/21): 172.16.48.0 — 2,046 hosts
- Dhaka (/19): 172.16.0.0 — 8,190 hosts (includes 3 VLANs)

Medium Subnets:
- Bucharest (/22): 172.16.56.0 — 1,022 hosts

Small Subnets:
- California (/23): 172.16.60.0 — 510 hosts
- Lisbon (/24): 172.16.62.0 — 254 hosts

Point-to-Point WAN Links (/30):
- 172.16.63.0, 172.16.63.8, 172.16.63.16, 172.16.63.24, 172.16.63.32
- Each /30 has only 2 usable IPs (for router-to-router communication)
```

### Subnet Mask Quick Reference

| Notation | Dotted Decimal | Usable Hosts |
|---|---|---|
| /19 | 255.255.224.0 | 8,190 |
| /20 | 255.255.240.0 | 4,094 |
| /21 | 255.255.248.0 | 2,046 |
| /22 | 255.255.252.0 | 1,022 |
| /23 | 255.255.254.0 | 510 |
| /24 | 255.255.255.0 | 254 |
| /30 | 255.255.255.252 | 2 |

---

## Routing Configuration Details

### Static Routing

**Concept:** Manually tell a router how to reach specific networks.

```
ip route [DESTINATION_NETWORK] [SUBNET_MASK] [NEXT_HOP_IP]
```

**Example from Dhaka Router:**
```
ip route 172.16.32.0 255.255.240.0 172.16.63.10
```
Translation: "To reach Paris (172.16.32.0/20), send packets to 172.16.63.10 (Paris's WAN interface)"

**All Static Routes in AoF Network:**

```
Dhaka Router:
- Route to Paris: ip route 172.16.32.0 255.255.240.0 172.16.63.10
- Route to Bucharest: ip route 172.16.56.0 255.255.252.0 172.16.63.2
```

> **Why these 2?** The requirement was "50% static, 50% dynamic". Paris and Bucharest use static routes. The other 4 branches (California, Lisbon, Tokyo, and return traffic) use dynamic routing (RIP).

### Dynamic Routing with RIP

**Concept:** Routers automatically learn routes by exchanging information.

```
router rip
 version 2
 network [DIRECTLY_CONNECTED_NETWORK]
 network [ANOTHER_DIRECTLY_CONNECTED_NETWORK]
 no auto-summary
exit
```

**Why Version 2?**
- Supports **VLSM** (Variable Length Subnet Masks)
- Sends **subnet mask** in routing updates (v1 doesn't)
- Better for complex networks with different subnet sizes

**Example: California Router RIP Configuration**
```
router rip
 version 2
 network 172.16.60.0       # California's LAN
 network 172.16.63.16      # WAN to Dhaka
 network 172.16.63.32      # WAN to Lisbon
 network 172.16.63.24      # WAN to Tokyo
 no auto-summary
exit
```

**How RIP Works:**
1. California broadcasts "I have networks 172.16.60.0, 172.16.63.16, 172.16.63.32, 172.16.63.24"
2. Dhaka learns "California can reach these networks"
3. Dhaka adds them to its routing table
4. Dhaka advertises these networks (plus its own) to Paris, Bucharest, and back to California
5. All routers eventually learn about all networks

**RIP Limitations in This Network:**
- Maximum hop count: 15 (network can be maximum 15 hops away)
- Slower convergence than modern routing protocols (OSPF, EIGRP)
- Less efficient bandwidth usage
- Used here for educational purposes (simplicity)

**Networks Learned via RIP:**

Every RIP-enabled router learns all networks from other RIP routers:

| From Dhaka | From California | From Lisbon | From Tokyo |
|---|---|---|---|
| 172.16.0.0/19 | 172.16.60.0/23 | 172.16.62.0/24 | 172.16.48.0/21 |
| (static) | (learned) | (learned) | (learned) |

---

## DHCP Pool Setup

**Concept:** DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses to devices.

```
ip dhcp pool [POOL_NAME]
 network [NETWORK_ADDRESS] [SUBNET_MASK]
 default-router [GATEWAY_IP]
 dns-server [DNS_SERVER_IP]
exit
```

**Example: Paris Router DHCP**
```
ip dhcp pool Paris
 network 172.16.32.0 255.255.240.0
 default-router 172.16.32.1
 dns-server 172.16.32.50
exit
```

### How DHCP Works:

1. **Broadcast:** PC sends "Give me an IP address!" (DHCP Discover)
2. **Offer:** Router's DHCP server offers address from pool (DHCP Offer)
3. **Request:** PC requests that address (DHCP Request)
4. **Acknowledge:** Router assigns and confirms (DHCP Acknowledge)

### DHCP Configuration in AoF:

| Branch | Pool Network | Gateway | DNS Server |
|---|---|---|---|
| Paris | 172.16.32.0/20 | 172.16.32.1 | 172.16.32.50 (local) |
| California | 172.16.60.0/23 | 172.16.60.1 | 172.16.60.50 (local) |
| Lisbon | 172.16.62.0/24 | 172.16.62.1 | 172.16.62.50 (local) |
| Tokyo | 172.16.48.0/21 | 172.16.48.1 | 172.16.48.50 (local) |

### Branches WITHOUT DHCP (Static Addressing):

| Branch | Why Static? |
|---|---|
| Dhaka (HQ) | Security: control all IPs, prevent rogue DHCP |
| Bucharest | Security: breach-origin site, strict control |

---

## VLAN Configuration

**Concept:** VLANs are logical network segments that provide isolation within the same physical switch.

### VLAN Creation

```
vlan [VLAN_NUMBER]
 name [VLAN_NAME]
exit
```

### Assigning Ports to VLANs

```
interface FastEthernet[PORT_NUMBER]/[SUBPORT]
 switchport mode access      # This is a regular port (not a trunk)
 switchport access vlan [VLAN_NUMBER]
exit
```

### Dhaka Switch VLAN Configuration

**3 VLANs for 3 Vendor Groups:**

```
vlan 2
 name Duber                  # Vendor group 1
exit

vlan 3
 name Uthao                  # Vendor group 2
exit

vlan 4
 name Kharaz                 # Vendor group 3
exit
```

**Port Assignments:**

```
DUBER (VLAN 2):
  interface FastEthernet1/1   → PC15
  interface FastEthernet2/1   → PC16

UTHAO (VLAN 3):
  interface FastEthernet3/1   → PC2
  interface FastEthernet6/1   → PC17

KHARAZ (VLAN 4):
  interface FastEthernet7/1   → PC18
  interface FastEthernet8/1   → PC7
```

### VLAN Behavior in AoF

**What CAN happen:**
- PC15 ↔ PC16 (both Duber) = ✅ Can communicate
- PC2 ↔ PC17 (both Uthao) = ✅ Can communicate
- PC18 ↔ PC7 (both Kharaz) = ✅ Can communicate

**What CANNOT happen (without inter-VLAN routing):**
- PC15 (Duber) → PC2 (Uthao) = ❌ Blocked
- PC15 (Duber) → PC18 (Kharaz) = ❌ Blocked
- PC2 (Uthao) → PC18 (Kharaz) = ❌ Blocked

> **Why?** No router is configured with sub-interfaces to route between VLANs. This is intentional—vendors must be kept completely separate.

---

## Access Control Lists (ACLs)

**Concept:** ACLs filter network traffic by creating rules for what is allowed/denied.

### ACL Syntax

```
access-list [NUMBER] [permit|deny] [PROTOCOL] [SOURCE_IP] [WILDCARD] [DEST_IP] [WILDCARD]
```

### Wildcard Mask Explanation

Wildcard masks are the opposite of subnet masks:
- `0.0.0.0` = match exactly this one address
- `0.0.0.255` = match this range (entire /24)
- `255.255.255.255` = match any address

### Lisbon ACL Configuration

```
access-list 101 deny ip 172.16.32.2 0.0.0.0 172.16.62.2 0.0.0.0
```

Translation: "ACL 101: DENY IP traffic from **exactly** 172.16.32.2 (Vulnerable PC in Paris) to **exactly** 172.16.62.2 (Secure Server in Lisbon)"

```
access-list 101 permit ip any any
```

Translation: "ACL 101: PERMIT all other IP traffic from any source to any destination"

### Applying ACL to Interface

```
interface FastEthernet0/0
 ip access-group 101 in
exit
```

Translation: "Apply ACL 101 to inbound traffic on this interface"

### Result

| Packet | Source | Destination | Action |
|---|---|---|---|
| Regular traffic | Any host | Lisbon Secure Server | ✅ Pass |
| Vulnerable PC | 172.16.32.2 | Lisbon Secure Server | ❌ Drop |
| Vulnerable PC | 172.16.32.2 | Other hosts | ✅ Pass |

---

## Verification Commands

Test these commands in Cisco Packet Tracer to verify your configurations:

### Check Interface Status

```
show ip interface brief
```
Output shows all interfaces and their status (up/down).

```
Router-PT Dhaka#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
FastEthernet0/0        172.16.0.1      YES manual up                    up
Serial2/0              172.16.63.17    YES manual up                    up
Serial3/0              172.16.63.9     YES manual up                    up
Serial6/0              172.16.63.1     YES manual up                    up
```

### Check Routing Table

```
show ip route
```
Shows all routes the router knows about (static and learned via RIP).

```
Router-PT Dhaka#show ip route
Codes: C - connected, S - static, R - rip

C       172.16.0.0/19 is directly connected, FastEthernet0/0
S       172.16.32.0/20 [1/0] via 172.16.63.10
S       172.16.56.0/22 [1/0] via 172.16.63.2
R       172.16.60.0/23 [120/1] via 172.16.63.18
R       172.16.62.0/24 [120/2] via 172.16.63.18
R       172.16.48.0/21 [120/2] via 172.16.63.18
```

Key indicators:
- **C** = Connected directly to this router
- **S** = Static route (manually configured)
- **R** = RIP route (dynamically learned)
- **120** = RIP's administrative distance (how "trusted" it is)
- **[hop count]** = How many routers away the network is

### Check RIP Status

```
show ip rip database
```
Shows all RIP routes the router has learned.

### Check DHCP Pools

```
show ip dhcp pool
```
Shows active DHCP pools and how many addresses are being used.

### Check ACLs

```
show access-lists
```
Shows all configured ACLs and their hit counts (how many packets matched).

```
Router-PT Lisbon#show access-lists
Extended IP access list 101
    10 deny ip 172.16.32.2 0.0.0.0 172.16.62.2 0.0.0.0
    20 permit ip any any
```

### Check VLAN Configuration

```
show vlan
```
Shows all VLANs and their ports.

```
Switch#show vlan
VLAN Name                             Status    Ports
---- -------------------------------- --------- ---
1    default                          active    Fa1/2 Fa1/3...
2    Duber                            active    Fa1/1 Fa2/1
3    Uthao                            active    Fa3/1 Fa6/1
4    Kharaz                           active    Fa7/1 Fa8/1
```

---

## 🎓 Learning Path

**If you're new to networking, study in this order:**

1. **Interfaces** → Understand how routers connect
2. **IP Addressing** → Understand how devices get addresses
3. **Static Routing** → Understand how routers manually direct traffic
4. **Dynamic Routing (RIP)** → Understand how routers learn routes automatically
5. **DHCP** → Understand how devices get IPs automatically
6. **VLAN** → Understand how to segment networks
7. **ACL** → Understand how to filter traffic

---

## 📖 Additional Resources

- **Cisco Commands:** Run `?` at any prompt to see available commands
- **Interface Help:** Type `int fa0/0` then `?` to see interface-specific commands
- **RIP Help:** Type `router rip` then `?` to see RIP options

Happy networking! 🚀
