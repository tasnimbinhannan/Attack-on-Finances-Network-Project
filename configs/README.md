# 🔧 Router & Switch Configurations

This directory contains the **complete CLI (Command Line Interface) configurations** for all routers and switches in the Attack on Finances network topology.

Each configuration file represents the exact commands entered into Cisco Packet Tracer to set up the network as designed.

---

## 📂 File Structure

```
configs/
├── README.md                    ← You are here
├── ROUTERS.md                   ← All router configs in one file (reference)
├── router-dhaka.cli
├── router-bucharest.cli
├── router-paris.cli
├── router-california.cli
├── router-lisbon.cli
├── router-tokyo.cli
├── switch-dhaka.cli
└── CONFIG_REFERENCE.md          ← Detailed explanation of each config
```

---

## 🔀 Configuration Overview

| Device | Type | Role | Key Feature |
|---|---|---|---|
| **Dhaka** | Router | HQ Hub | RIP dynamic routing + VLAN trunk |
| **Bucharest** | Router | Branch (Threat Origin) | RIP + Static addressing |
| **Paris** | Router | Branch | RIP + DHCP |
| **California** | Router | Branch Hub | RIP dynamic routing (3 WAN links) |
| **Lisbon** | Router | Branch | RIP + ACL (blocks Vulnerable PC) |
| **Tokyo** | Router | Branch | RIP + DHCP |
| **Dhaka Switch** | Switch | VLAN Manager | 3 VLANs (Duber, Uthao, Kharaz) |

---

## 📝 How to Use These Files

### Option 1: Copy-Paste Individual Router Configs
1. Open each `.cli` file
2. Copy the entire content
3. In Packet Tracer, double-click the router
4. Go to **CLI** tab
5. Paste the commands and press Enter

### Option 2: Reference from Running Network
1. Open `Final_Project_2_0.pkt` in Packet Tracer
2. Double-click any router
3. Run `show running-config` to see live configuration
4. Compare against the corresponding `.cli` file in this folder

### Option 3: Study & Learn
Each file is heavily commented to explain:
- What each interface does
- Why specific IP addresses were chosen
- How routing is configured
- Which security policies are applied

---

## 🔑 Key Configuration Features

### Dynamic Routing (RIP)
Used on these segments:
- **Dhaka ↔ California:** Primary backbone
- **California ↔ Lisbon:** Secondary backbone
- **California ↔ Tokyo:** Branch link

**Why RIP v2?**
- Classless routing support (VLSM)
- Automatic route propagation
- Faster convergence than RIPv1

### Static Routing
Used on these segments:
- **Dhaka ↔ Paris:** Manual routes only
- **Dhaka ↔ Bucharest:** Manual routes only

**Why static?**
- Meets the "50% static, 50% dynamic" requirement
- More control over routing decisions
- Simpler for smaller segments

### DHCP Pools
Configured for DHCP-enabled branches:
- **Paris:** 172.16.32.0/20 → DNS 172.16.32.50
- **California:** 172.16.60.0/23 → DNS 172.16.60.50
- **Lisbon:** 172.16.62.0/24 → DNS 172.16.62.50
- **Tokyo:** 172.16.48.0/21 → DNS 172.16.48.50

### Access Control List (ACL)
**Location:** Lisbon Router (fa0/0 inbound)

```
access-list 101 deny ip 172.16.62.2 0.0.0.0 172.16.32.2 0.0.0.0
access-list 101 permit ip any any
```

- **Denies:** Vulnerable PC (172.16.32.2) from reaching Lisbon (172.16.62.2)
- **Permits:** All other traffic
- **Applied:** Inbound on Lisbon's LAN interface

### VLAN Configuration
**Location:** Dhaka Switch

| VLAN | Name | Ports | Purpose |
|---|---|---|---|
| 2 | Duber | fa1/1, fa2/1 | Vendor group 1 |
| 3 | Uthao | fa3/1, fa6/1 | Vendor group 2 |
| 4 | Kharaz | fa7/1, fa8/1 | Vendor group 3 |

- **Isolation:** No inter-VLAN routing
- **Purpose:** Prevent unauthorized vendor-to-vendor communication

---

## 🚀 Quick Reference: Router Interfaces

### Dhaka (Central Hub)
| Interface | IP | Connected To | Purpose |
|---|---|---|---|
| fa0/0 | 172.16.0.1 | Internal LAN | Headquarters network |
| se6/0 | 172.16.63.1 | Bucharest | WAN link |
| se3/0 | 172.16.63.9 | Paris | WAN link |
| se2/0 | 172.16.63.17 | California | WAN link (primary) |

### California (Secondary Hub)
| Interface | IP | Connected To | Purpose |
|---|---|---|---|
| fa0/0 | 172.16.60.1 | Internal LAN | Branch network |
| se2/0 | 172.16.63.18 | Dhaka | WAN link |
| se3/0 | 172.16.63.33 | Lisbon | WAN link |
| se9/0 | 172.16.63.25 | Tokyo | WAN link |

---

## 📚 Related Documentation

- **README.md** (in repo root) → Full project overview
- **CONFIG_REFERENCE.md** (in this folder) → Detailed explanation of every command
- **Full_Project_Image.png** (in repo root) → Visual topology diagram
- **Final_Project_2_0.pkt** (in repo root) → The actual Packet Tracer simulation

---

## ✅ Verification Commands

To verify a router's configuration is correct, use these show commands in Packet Tracer:

```
show ip interface brief          # View all interfaces and their status
show running-config             # View complete configuration
show ip route                   # View routing table
show ip rip database            # View RIP routes (if RIP enabled)
show access-lists               # View ACLs (if configured)
show ip dhcp pool               # View DHCP pools
```

---

## 📝 Notes

- All configurations assume Cisco Packet Tracer 8.x or later
- No passwords or enable secrets are configured (as per simulation requirements)
- All IP addresses are from the `172.16.0.0/16` block
- No NAT or port forwarding is configured
