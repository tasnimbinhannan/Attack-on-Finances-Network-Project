# 🔧 CONSOLIDATED ROUTER & SWITCH CONFIGURATIONS
# Attack on Finances (AoF) - Enterprise Network Project
# All CLI commands in one reference file

# ==============================================================================
# DHAKA ROUTER CONFIGURATION
# ==============================================================================

Dhaka:
enable
configure terminal

# LAN Interface - Headquarters Network
interface FastEthernet0/0
 description HQ-LAN-Dhaka
 ip address 172.16.0.1 255.255.224.0
 no shutdown
exit

# WAN Interface to Bucharest
interface Serial6/0
 description WAN-to-Bucharest
 ip address 172.16.63.1 255.255.255.252
 no shutdown
exit

# WAN Interface to Paris
interface Serial3/0
 description WAN-to-Paris
 ip address 172.16.63.9 255.255.255.252
 no shutdown
exit

# WAN Interface to California (Primary)
interface Serial2/0
 description WAN-to-California-Primary
 ip address 172.16.63.17 255.255.255.252
 no shutdown
exit

# Dynamic Routing Configuration
router rip
 version 2
 network 172.16.0.0
 network 172.16.63.0
 network 172.16.63.8
 network 172.16.63.16
 no auto-summary
exit

# Static Routes (to Paris and Bucharest)
ip route 172.16.32.0 255.255.240.0 172.16.63.10
ip route 172.16.56.0 255.255.252.0 172.16.63.2

# ==============================================================================
# BUCHAREST ROUTER CONFIGURATION
# ==============================================================================

Bucharest:
enable
configure terminal

# LAN Interface - Bucharest Branch Network
interface FastEthernet0/0
 description Branch-LAN-Bucharest
 ip address 172.16.56.1 255.255.252.0
 no shutdown
exit

# WAN Interface to Dhaka
interface Serial2/0
 description WAN-to-Dhaka-HQ
 ip address 172.16.63.2 255.255.255.252
 no shutdown
exit

# Dynamic Routing Configuration
router rip
 version 2
 network 172.16.56.0
 network 172.16.63.0
 no auto-summary
exit

# Note: No DHCP configured - Static addressing required for security

# ==============================================================================
# PARIS ROUTER CONFIGURATION
# ==============================================================================

Paris:
enable
configure terminal

# LAN Interface - Paris Branch Network
interface FastEthernet0/0
 description Branch-LAN-Paris
 ip address 172.16.32.1 255.255.240.0
 no shutdown
exit

# WAN Interface to Dhaka
interface Serial2/0
 description WAN-to-Dhaka-HQ
 ip address 172.16.63.10 255.255.255.252
 no shutdown
exit

# DHCP Configuration for Paris
ip dhcp pool Paris
 network 172.16.32.0 255.255.240.0
 default-router 172.16.32.1
 dns-server 172.16.32.50
exit

# Dynamic Routing Configuration
router rip
 version 2
 network 172.16.32.0
 network 172.16.63.8
 no auto-summary
exit

# ==============================================================================
# CALIFORNIA ROUTER CONFIGURATION
# ==============================================================================

California:
enable
configure terminal

# LAN Interface - California Branch Network
interface FastEthernet0/0
 description Branch-LAN-California
 ip address 172.16.60.1 255.255.254.0
 no shutdown
exit

# WAN Interface to Dhaka (Primary)
interface Serial2/0
 description WAN-to-Dhaka-Primary
 ip address 172.16.63.18 255.255.255.252
 no shutdown
exit

# WAN Interface to Lisbon
interface Serial3/0
 description WAN-to-Lisbon
 ip address 172.16.63.33 255.255.255.252
 no shutdown
exit

# WAN Interface to Tokyo
interface Serial9/0
 description WAN-to-Tokyo
 ip address 172.16.63.25 255.255.255.252
 no shutdown
exit

# DHCP Configuration for California
ip dhcp pool California
 network 172.16.60.0 255.255.254.0
 default-router 172.16.60.1
 dns-server 172.16.60.50
exit

# Dynamic Routing Configuration
router rip
 version 2
 network 172.16.60.0
 network 172.16.63.16
 network 172.16.63.32
 network 172.16.63.24
 no auto-summary
exit

# ==============================================================================
# LISBON ROUTER CONFIGURATION
# ==============================================================================

Lisbon:
enable
configure terminal

# LAN Interface - Lisbon Branch Network
interface FastEthernet0/0
 description Branch-LAN-Lisbon
 ip address 172.16.62.1 255.255.255.0
 no shutdown
exit

# WAN Interface to California
interface Serial2/0
 description WAN-to-California
 ip address 172.16.63.34 255.255.255.252
 no shutdown
exit

# DHCP Configuration for Lisbon
ip dhcp pool Lisbon
 network 172.16.62.0 255.255.255.0
 default-router 172.16.62.1
 dns-server 172.16.62.50
exit

# Access Control List - Block Vulnerable PC from Lisbon Server
access-list 101 deny ip 172.16.32.2 0.0.0.0 172.16.62.2 0.0.0.0
access-list 101 permit ip any any

# Apply ACL to LAN Interface
interface FastEthernet0/0
 ip access-group 101 in
exit

# Dynamic Routing Configuration
router rip
 version 2
 network 172.16.62.0
 network 172.16.63.32
 no auto-summary
exit

# ==============================================================================
# TOKYO ROUTER CONFIGURATION
# ==============================================================================

Tokyo:
enable
configure terminal

# LAN Interface - Tokyo Branch Network
interface FastEthernet0/0
 description Branch-LAN-Tokyo
 ip address 172.16.48.1 255.255.248.0
 no shutdown
exit

# WAN Interface to California
interface Serial2/0
 description WAN-to-California
 ip address 172.16.63.26 255.255.255.252
 no shutdown
exit

# DHCP Configuration for Tokyo
ip dhcp pool Tokyo
 network 172.16.48.0 255.255.248.0
 default-router 172.16.48.1
 dns-server 172.16.48.50
exit

# Dynamic Routing Configuration
router rip
 version 2
 network 172.16.48.0
 network 172.16.63.24
 no auto-summary
exit

# ==============================================================================
# DHAKA SWITCH CONFIGURATION
# ==============================================================================

Dhaka Switch:

# VLAN Creation
vlan 2
 name Duber
exit

vlan 3
 name Uthao
exit

vlan 4
 name Kharaz
exit

# Port Assignment - DUBER (VLAN 2)
interface FastEthernet1/1
 description Duber-PC15
 switchport mode access
 switchport access vlan 2
exit

interface FastEthernet2/1
 description Duber-PC16
 switchport mode access
 switchport access vlan 2
exit

# Port Assignment - UTHAO (VLAN 3)
interface FastEthernet3/1
 description Uthao-PC2
 switchport mode access
 switchport access vlan 3
exit

interface FastEthernet6/1
 description Uthao-PC17
 switchport mode access
 switchport access vlan 3
exit

# Port Assignment - KHARAZ (VLAN 4)
interface FastEthernet7/1
 description Kharaz-PC18
 switchport mode access
 switchport access vlan 4
exit

interface FastEthernet8/1
 description Kharaz-PC7
 switchport mode access
 switchport access vlan 4
exit

# ==============================================================================
# END OF ALL CONFIGURATIONS
# ==============================================================================
# Summary:
# - 6 Routers configured (Dhaka, Bucharest, Paris, California, Lisbon, Tokyo)
# - 1 Switch configured (Dhaka) with 3 VLANs
# - Hybrid routing: Static (Dhaka-Paris, Dhaka-Bucharest) + Dynamic RIP (others)
# - DHCP enabled on 4 branches (Paris, California, Lisbon, Tokyo)
# - Static addressing on 2 branches (Dhaka HQ, Bucharest)
# - ACL blocking Vulnerable PC from Lisbon Secure Server
# - VLAN isolation at Dhaka for 3 vendor groups
# ==============================================================================
