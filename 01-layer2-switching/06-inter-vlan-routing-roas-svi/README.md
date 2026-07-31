# Inter-VLAN Routing: Router-on-a-Stick (ROAS) & Switch Virtual Interfaces (SVI)

## Overview

This lab demonstrates how to enable communication between isolated VLANs using two primary methods: Router-on-a-Stick (ROAS) using a dedicated router subinterface topology and Switch Virtual Interfaces (SVIs) on a Layer 3 Multilayer Switch.
By introducing Layer 3 routing, devices in different VLANs can securely route traffic between broadcast domains while maintaining Layer 2 isolation.

## Objectives

* Configure 802.1Q trunking on switch interfaces connected to routing devices
* Implement Router-on-a-Stick (ROAS) using router subinterfaces for multiple VLANs
* Configure subinterface IP address encapsulation matching 802.1Q VLAN IDs
* Implement Inter-VLAN routing using Layer 3 Switch Virtual Interfaces (SVIs)
* Enable IP routing on a Multilayer Switch
* Verify routing table entries for directly connected VLAN subnets
* Validate cross-VLAN ICMP connectivity

## Network Topology

![Inter-VLAN Routing Topology](topology.png)

## VLAN Design

| VLAN | Name | Purpose | Subnet | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| 10 | Sales | Sales Department | 192.168.10.0/24 | 192.168.10.1 |
| 20 | Engineering | Engineering Department | 192.168.20.0/24 | 192.168.20.1 |
| 99 | Management | Switch Management | 192.168.99.0/24 | 192.168.99.1 |
| 999 | Native | Unused Native VLAN | N/A | N/A |

## Device Addressing

| Device | Interface | IP Address | Subnet Mask | VLAN | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| PC0 | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 10 | 192.168.10.1 |
| PC1 | FastEthernet0 | 192.168.20.10 | 255.255.255.0 | 20 | 192.168.20.1 |
| R1 | GigabitEthernet0/0/0.10 | 192.168.10.1 | 255.255.255.0 | 10 | N/A |
| R1 | GigabitEthernet0/0/0.20 | 192.168.20.1 | 255.255.255.0 | 20 | N/A |
| MLS1 | VLAN 10 | 192.168.10.1 | 255.255.255.0 | 10 | N/A |
| MLS1 | VLAN 20 | 192.168.20.1 | 255.255.255.0 | 20 | N/A |

## Configuration

The following tasks were completed during this lab:

### 1. Switch Trunking & Access Port Setup
Configured access ports for end devices and established 802.1Q trunking to the router/core switch.

```cisco
enable
configure terminal

! Configure Access Ports
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 exit

! Configure Trunk Port to Router
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,99,999
 exit
```

### 2. Router-on-a-Stick (ROAS) Configuration
Configured subinterfaces on the physical router link, assigning 802.1Q encapsulation and default gateway IP addresses.

```cisco
enable
configure terminal

! Enable Physical Interface
interface GigabitEthernet0/0/0
 no shutdown
 exit

! Configure Subinterfaces
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
```

### 3. Layer 3 Switch (SVI) Configuration
Configured Switch Virtual Interfaces (SVIs) on a Multilayer Switch as a modern high-speed alternative to ROAS.

```cisco
enable
configure terminal

! Enable IP Routing globally
ip routing

! Create VLAN Interfaces (SVIs)
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit
```

## Verification

### Routing Table Verification

Verified that directly connected VLAN subnets populate the routing table on R1/MLS1.

```cisco
show ip route
```

### Trunking Verification

Verified active trunk interfaces, 802.1Q encapsulation, and allowed VLAN lists.

```cisco
show interfaces trunk
```

### Connectivity Testing

* Devices within the same VLAN successfully communicated directly across switch ports.
* Devices in different VLANs (PC0 in VLAN 10 to PC1 in VLAN 20) successfully pinged across their respective gateways.

## Key Takeaways

* Isolated VLANs require a Layer 3 routing mechanism (Router or Multilayer Switch) to forward packets between subnets.
* **ROAS** relies on a single physical link carrying multiple subinterfaces, creating potential bandwidth bottlenecks on high-traffic networks.
* **SVIs** process Inter-VLAN routing inside the switch ASIC hardware backplane at wire speed, making it the enterprise standard for campus networks.
* End devices in segmented VLANs must have their default gateway pointed to the corresponding subinterface or SVI IP address.

## Environment

* Cisco Packet Tracer
* Cisco IOS
