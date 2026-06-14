# 🛡️ Attack on Finances (AoF) — Enterprise Network Security Project

> A fully configured enterprise-grade Wide Area Network (WAN) designed and simulated in **Cisco Packet Tracer**, built as a cybersecurity response scenario for a global financial institution under active threat.

---

## 📖 Project Scenario

A large financial institution has been breached. Hackers based in **Bucharest, Romania** compromised the company's payment processing system, stealing millions of dollars and threatening to leak sensitive customer data.

As the cybersecurity analyst on the case, the objective was to **design and deploy a secure, globally distributed network topology** from scratch — covering the company's headquarters and all international branches — while enforcing strict access controls, network segmentation, routing policies, and service configurations to contain the threat and prevent further unauthorized access.

---

## 🌍 Network Topology Overview

The network spans **6 cities** across the globe, each represented by a dedicated router, with end devices, servers, and switches beneath them.

| City | Role | Devices | Connects To |
|---|---|---|---|
| **Dhaka** | Headquarters | 44AB* devices | California, Bucharest, Paris |
| **California** | Branch | 300 devices | Dhaka, Lisbon, Tokyo |
| **Lisbon** | Branch | 200 devices | California |
| **Paris** | Branch | 2450 devices | Dhaka |
| **Bucharest** | Branch (Threat Origin) | 756 devices | Dhaka |
| **Tokyo** | Branch | 10XY* devices | California |

> \* AB and XY are the last two digits of group members' student IDs.

### Inter-City Link Distances

| | Dhaka | California | Lisbon | Paris | Bucharest | Tokyo |
|---|---|---|---|---|---|---|
| **Dhaka** | — | 11 km | — | 56 km | 72 km | — |
| **California** | 11 km | — | 200 km | — | — | 152 km |
| **Lisbon** | — | 200 km | — | — | — | — |
| **Paris** | 56 km | — | — | — | — | — |
| **Bucharest** | 72 km | — | — | — | — | — |
| **Tokyo** | — | 152 km | — | — | — | — |

---

## 🗺️ Network Addressing & Subnetting

### Whole Network Block
```
Network Address : 172.16.0.0 / 16
Subnet Mask     : 255.255.0.0
```

All subnets are carved out of this block. Subnetting was done based on the **number of required hosts** per branch (largest-first allocation).

### Subnet Allocations

#### Paris (2450 hosts required)
```
Network Address : 172.16.32.0 / 20
Broadcast       : 172.16.47.255
Subnet Mask     : 255.255.240.0
Hosts Available : 4094
```

#### Bucharest (756 hosts required)
```
Network Address : 172.16.56.0 / 22
Broadcast       : 172.16.59.255
Subnet Mask     : 255.255.252.0
Hosts Available : 1022
```

#### Tokyo (1051 hosts required)
```
Network Address : 172.16.48.0 / 21
Broadcast       : 172.16.55.255
Subnet Mask     : 255.255.248.0
Hosts Available : 2046
```

#### Dhaka (4432 hosts required — includes 3 vendor VLANs)
```
Network Address : 172.16.0.0 / 19
Broadcast       : 172.16.31.255
Subnet Mask     : 255.255.224.0
Hosts Available : 8190
```

#### California (300 hosts required)
```
Network Address : 172.16.60.0 / 23
Broadcast       : 172.16.61.255
Subnet Mask     : 255.255.254.0
Hosts Available : 510
```

#### Lisbon (200 hosts required)
```
Network Address : 172.16.62.0 / 24
Broadcast       : 172.16.62.255
Subnet Mask     : 255.255.255.0
Hosts Available : 254
```

#### Point-to-Point Router Links (WAN Links)
Each router-to-router connection uses a `/30` subnet from the `172.16.63.0/24` block:

| Link | Network | Router A IP | Router B IP | Subnet Mask |
|---|---|---|---|---|
| Paris ↔ Dhaka (P2D) | 172.16.63.8 /30 | 172.16.63.9 | 172.16.63.10 | 255.255.255.252 |
| Dhaka ↔ Bucharest (D2B) | 172.16.63.0 /30 | 172.16.63.1 | 172.16.63.2 | 255.255.255.252 |
| Dhaka ↔ California (D2C) | 172.16.63.16 /30 | 172.16.63.17 | 172.16.63.18 | 255.255.255.252 |
| Dhaka ↔ California (D2C alt) | 172.16.63.12 /30 | 172.16.63.13 | 172.16.63.14 | 255.255.255.252 |
| California ↔ Lisbon (C2L) | 172.16.63.32 /30 | 172.16.63.33 | 172.16.63.34 | 255.255.255.252 |
| California ↔ Tokyo (C2T) | 172.16.63.24 /30 | 172.16.63.25 | 172.16.63.26 | 255.255.255.252 |

---

## 🔧 Router Configurations

### Router: Paris
```
Name            : Router-PT Paris
Default Gateway : 172.16.32.1
WAN Interface   : 172.16.63.9 (to Dhaka)
```

### Router: Dhaka (Central Hub)
```
Name            : Router-PT Dhaka
WAN Interfaces  : 172.16.63.17 (to California)
                  172.16.63.1  (to Bucharest)
                  172.16.63.9  (to Paris)
```

### Router: Bucharest
```
Name            : Router-PT Bucharest
Default Gateway : 172.16.56.1
WAN Interface   : 172.16.63.2 (to Dhaka)
```

### Router: California
```
Name            : Router-PT California
Default Gateway : 172.16.60.1
WAN Interfaces  : 172.16.63.18 (to Dhaka)
                  172.16.63.33 (to Lisbon)
                  172.16.63.25 (to Tokyo)
```

### Router: Lisbon
```
Name            : Router-PT Lisbon
Default Gateway : 172.16.62.1
WAN Interface   : 172.16.63.34 (to California)
```

### Router: Tokyo
```
Name            : Router-PT Tokyo
Default Gateway : 172.16.48.1
WAN Interface   : 172.16.63.26 (to California)
```

---

## 🖥️ End Device & Server Summary

### Paris
- **Devices:** Laptops and Printers (as per spec)
- **Represented by:** Laptop0 (user1@paris.com), Printer0, Vulnerable PC (user2@paris.com)
- **Server:** Server0 (MS: 172.16.32.50) — Mail Server
- **Addressing:** DHCP

### Bucharest
- **Devices:** PC1 (172.16.56.2, user1@bucharest.com), PC16 (172.16.56.3, user2@bucharest.com)
- **Server:** Server7 (MS: 172.16.56.50) — Mail Server
- **Addressing:** Static (security requirement)

### Dhaka (Headquarters)
- **Vendor Groups (VLANs):**
  - **Duber:** PC15 (172.16.0.2), PC16 (172.16.0.3)
  - **Uthao:** PC2 (172.16.0.4), PC17 (172.16.0.5)
  - **Kharaz:** PC18 (172.16.0.6), PC7 (172.16.0.7)
- **Addressing:** Static (security requirement)

### California
- **Devices:** PC4, PC5 (DHCP)
- **Servers:** Server0 (DNS Server, 172.16.60.50), Server1 (Web Server — "Welcome to Hotel California!")
- **Addressing:** DHCP

### Lisbon
- **Devices:** PC19, PC20 (DHCP)
- **Server:** Server8 (172.16.62.50) — Secure Server
- **Addressing:** DHCP

### Tokyo
- **Devices:** PC8, PC21 (DHCP)
- **Servers:** Server1 (Web Server — "I love Anime!"), DNS Server (172.16.48.50)
- **Addressing:** DHCP

---

## 🔀 Routing Configuration

### Strategy: Hybrid Routing (Static + Dynamic)

As per the project requirement, **half the network uses dynamic routing** and the other half uses **static routing**. No default routes are used anywhere — all packet delivery relies on explicitly configured routes.

### Dynamic Routing
Configured on the **Dhaka–California–Tokyo–Lisbon** segment:
- Dhaka ↔ California
- California ↔ Lisbon
- California ↔ Tokyo

### Static Routing
Configured on the **Dhaka–Paris** and **Dhaka–Bucharest** segments:
- Dhaka ↔ Paris (static routes manually entered on both routers)
- Dhaka ↔ Bucharest (static routes manually entered on both routers)

> ⚠️ **No default routes (`0.0.0.0/0`) are used anywhere in this topology.** Every route is explicit.

---

## 🔒 Security Configurations

### 1. Lisbon Secure Server — Access Control List (ACL)
The Secure Server at Lisbon (`Server8`) is configured to **deny access specifically from the "Vulnerable PC"** in Paris.

- **Denied Host:** Vulnerable PC (`user2@paris.com`) in the Paris subnet (`172.16.32.0/20`)
- **All other hosts:** Permitted
- **Implementation:** ACL targeting the Vulnerable PC's specific IP address

### 2. Dhaka HQ — VLAN Segmentation (Inter-Vendor Isolation)
The three vendor groups at Dhaka headquarters are isolated from each other using **VLANs**, while each group can communicate freely within itself.

| VLAN | Vendor | Devices |
|---|---|---|
| VLAN 10 | Duber | PC15, PC16 |
| VLAN 20 | Uthao | PC2, PC17 |
| VLAN 30 | Kharaz | PC18, PC7 |

- **Inter-VLAN communication:** Disabled (no inter-VLAN routing configured)
- **Intra-VLAN communication:** Fully functional

### 3. Static Addressing for Critical Nodes
Dhaka and Bucharest — identified as the most security-sensitive branches — use **fully static IP addressing** rather than DHCP, reducing exposure to rogue DHCP attacks and ensuring address stability.

---

## 🌐 Web & DNS Services

### California — Web Server
| Property | Value |
|---|---|
| URL | `www.california..gov` |
| Response | `Welcome to Hotel California!` |
| Server IP | 172.16.60.x (California subnet) |
| DNS Record | A record configured on California DNS server |

### Tokyo — Web Server
| Property | Value |
|---|---|
| URL | `www.japan.com` |
| Response | `I love Anime!` |
| Server IP | 172.16.48.x (Tokyo subnet) |
| DNS Record | A record configured on DNS server |

### DNS Server — California
- Hosted at: `172.16.60.50`
- Resolves `www.california..gov` → California Web Server
- Resolves `www.japan.com` → Tokyo Web Server
- All DHCP-configured branches point to this DNS server

### DNS Server — Tokyo
- Hosted at: `172.16.48.50`
- Local resolution for Tokyo-side queries

---

## 📧 Email Service (Bonus Task)

Email servers are configured at **Paris** and **Bucharest** to enable full bidirectional mail exchange between the two branches.

### Paris Mail Server
```
Server    : Server0
MS IP     : 172.16.32.50
Domain    : paris.com
Users     : user1@paris.com  (Laptop0)
            user2@paris.com  (Vulnerable PC)
```

### Bucharest Mail Server
```
Server    : Server7
MS IP     : 172.16.56.50
Domain    : bucharest.com
Users     : user1@bucharest.com  (PC1)
            user2@bucharest.com  (PC16)
```

### Email Configuration Details
Each mail client (Laptop / PC) is configured with:
- **Incoming Mail Server (POP3/IMAP):** Points to its local mail server IP
- **Outgoing Mail Server (SMTP):** Points to its local mail server IP
- **Username / Password:** Configured per user account

### Verified Operations
- ✅ **Send mail** from Paris → Bucharest
- ✅ **Receive mail** at Bucharest from Paris
- ✅ **Reply to mail** from Bucharest → Paris

---

## 📡 IP Addressing — Complete Reference Table

### LAN Interfaces (Router Gateway IPs)

| Branch | Gateway IP | Subnet | Mask |
|---|---|---|---|
| Paris | 172.16.32.1 | 172.16.32.0/20 | 255.255.240.0 |
| Bucharest | 172.16.56.1 | 172.16.56.0/22 | 255.255.252.0 |
| Tokyo | 172.16.48.1 | 172.16.48.0/21 | 255.255.248.0 |
| Dhaka | 172.16.0.1 | 172.16.0.0/19 | 255.255.224.0 |
| California | 172.16.60.1 | 172.16.60.0/23 | 255.255.254.0 |
| Lisbon | 172.16.62.1 | 172.16.62.0/24 | 255.255.255.0 |

---

## 🧪 Testing & Verification

All of the following were verified to be working in the simulation:

| Test | Result |
|---|---|
| Ping between all end-devices (non-restricted) | ✅ Pass |
| Vulnerable PC cannot reach Lisbon Secure Server | ✅ Blocked |
| Duber ↔ Uthao cross-VLAN ping | ✅ Blocked |
| Duber ↔ Duber intra-VLAN ping | ✅ Pass |
| Browser → `www.california..gov` shows "Welcome to Hotel California!" | ✅ Pass |
| Browser → `www.japan.com` shows "I love Anime!" | ✅ Pass |
| DHCP IP assignment at Paris, California, Lisbon, Tokyo | ✅ Pass |
| Static IPs confirmed at Dhaka and Bucharest | ✅ Pass |
| Email: Paris → Bucharest send & receive | ✅ Pass |
| Email: Bucharest → Paris reply | ✅ Pass |
| No default routes present anywhere | ✅ Confirmed |
| Dynamic routing on Dhaka-California-Lisbon-Tokyo segment | ✅ Active |
| Static routing on Dhaka-Paris and Dhaka-Bucharest segments | ✅ Active |

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| **Cisco Packet Tracer** | Network simulation and configuration |
| **RIP / OSPF** | Dynamic routing protocol |
| **Static Routing** | Manual route configuration |
| **DHCP** | Automatic IP assignment (Paris, Lisbon, California, Tokyo) |
| **DNS** | Domain name resolution |
| **HTTP / Web Server** | Hosting branch web pages |
| **SMTP / POP3** | Email exchange (Paris ↔ Bucharest) |
| **ACL** | Access control — blocking Vulnerable PC from Lisbon |
| **VLAN** | Vendor isolation at Dhaka HQ |
| **Subnetting (VLSM)** | Variable Length Subnet Masking for efficient IP allocation |

---

## 📁 Repository Structure

```
📦 Attack-on-Finances-Network-Project
 ┣ 📄 README.md                                      ← You are here
 ┣ 📄 Computer_Network_Project_Specifications.pdf    ← Original project brief
 ┣ 🖼️  Full_Project_Image.png                        ← Full topology screenshot
 ┗ 📦 Final_Project_2_0.pkt                          ← Cisco Packet Tracer project file
```

---

## 🚀 How to Open the Project

1. Download and install **Cisco Packet Tracer** (version 8.x or later recommended).
   - Available free at: [https://www.netacad.com/courses/packet-tracer](https://www.netacad.com/courses/packet-tracer)
2. Clone or download this repository.
3. Open `Final_Project_2_0.pkt` in Cisco Packet Tracer.
4. Use **Simulation Mode** to trace packets and verify routing behavior.
5. Use the **Web Browser** tool on any PC to visit `www.california..gov` or `www.japan.com`.
6. Use the **Email client** on Paris/Bucharest PCs to test mail exchange.

---

## 👥 Team

This project was developed as a group assignment for a Computer Networks course. Group member student IDs determined the variable parameters AB and XY used in device count calculations (Dhaka: 44AB devices, Tokyo: 10XY devices).

---

## 📜 License

This project is submitted for academic purposes. The topology, configurations, and documentation were designed and implemented by the project group members.
