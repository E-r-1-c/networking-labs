# Inter-VLAN Routing: Router-on-a-Stick (ROAS) & Switch Virtual Interfaces (SVI)

## Overview

This lab demonstrates how to enable communication between isolated VLANs using two primary methods: Router-on-a-Stick (ROAS) using a dedicated router subinterface topology and Switch Virtual Interfaces (SVIs) on a Layer 3 Multilayer Switch.
By introducing Layer 3 routing, devices in different VLANs can securely route traffic between broadcast domains while maintaining Layer 2 isolation.

---

## Objectives

- Configure 802.1Q trunking on switch interfaces connected to routing devices
- Implement Router-on-a-Stick (ROAS) using router subinterfaces for multiple VLANs
- Configure subinterface IP address encapsulation matching 802.1Q VLAN IDs
- Implement Inter-VLAN routing using Layer 3 Switch Virtual Interfaces (SVIs)
- Enable IP routing on a Multilayer Switch
- Verify routing table entries for directly connected VLAN subnets
- Validate cross-VLAN ICMP connectivity

---

## Network Topology & Switch Roles

![Network Topology](./images/topology.png)

| Device | Role | Interface / Connection | Routing Method |
|--------|------|------------------------|----------------|
| R0 | Edge Router | Fa0/0 (Subinterfaces) | Router-on-a-Stick (ROAS) |
| SW0 | Access / Trunk Switch | Fa0/1 (Trunk to R0) | Layer 2 Switching |
| MLS0 | Core Multilayer Switch | VLAN 10, 20 SVIs | Hardware Inter-VLAN Routing |

---

## VLAN Design

| VLAN | Name | Purpose | Subnet | Gateway |
|:---|:---|:---|:---|:---|
| 10 | Sales | Sales Department | 192.168.10.0/24 | 192.168.10.1 |
| 20 | Engineering | Engineering Department | 192.168.20.0/24 | 192.168.20.1 |
| 99 | Management | Switch Management | 192.168.99.0/24 | 192.168.99.1 |
| 999 | Native | Unused Native VLAN | N/A | N/A |

---

## Device Addressing

| Device | Interface | IP Address | Subnet Mask | VLAN | Default Gateway |
|:---|:---|:---|:---|:---|:---|
| PC0 | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 10 | 192.168.10.1 |
| PC1 | FastEthernet0 | 192.168.20.10 | 255.255.255.0 | 20 | 192.168.20.1 |
| R0 | FastEthernet0/0.10 | 192.168.10.1 | 255.255.255.0 | 10 | N/A |
| R0 | FastEthernet0/0.20 | 192.168.20.1 | 255.255.255.0 | 20 | N/A |
| MLS0 | VLAN 10 | 192.168.10.1 | 255.255.255.0 | 10 | N/A |
| MLS0 | VLAN 20 | 192.168.20.1 | 255.255.255.0 | 20 | N/A |

---

## Configuration

The following tasks were completed during this lab:

### 1. Switch Trunking & Access Port Setup
Configured access ports for end devices and established 802.1Q trunking on FastEthernet0/1 connected to the router.

```cisco
enable
configure terminal

! Configure Trunk Port to Router
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,99,999
 exit

! Configure Access Ports
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 exit

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 exit
```

### 2. Router-on-a-Stick (ROAS) Configuration
Configured subinterfaces on physical interface FastEthernet0/0, assigning 802.1Q encapsulation and default gateway IP addresses.

```cisco
enable
configure terminal

! Enable Physical Interface
interface FastEthernet0/0
 no shutdown
 exit

! Configure Subinterfaces
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface FastEthernet0/0.20
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

---

## Verification

### Routing Table Verification

Verified that directly connected VLAN subnets populate the routing table on R0/MLS0.

Command used:

```cisco
show ip route
```

![Routing Table Verification](./images/01-routing-table.png)

The output confirmed:
- Directly connected routes exist for subnets `192.168.10.0/24` and `192.168.20.0/24`.
- Subinterfaces / SVIs are actively routing Layer 3 traffic.

---

### Trunk Verification

Verified active trunk interfaces, 802.1Q encapsulation, and allowed VLAN lists.

Command used:

```cisco
show interfaces trunk
```

![Interface Trunk Verification](./images/02-interfaces-trunk.png)

The output confirmed:
- FastEthernet0/1 is actively trunking using 802.1Q.
- Native VLAN matches VLAN 999.
- VLANs 10, 20, 99, and 999 are permitted and active across the link.

---

### Connectivity Testing

Verified end-to-end connectivity across isolated VLANs via Layer 3 gateways.

![ICMP Connectivity Verification](./images/03-icmp-ping.png)

The output confirmed:
- `PC0` (`192.168.10.10`) successfully pinged `PC1` (`192.168.20.10`) across the default gateway.
- Inter-VLAN routing successfully forwarded ICMP traffic between distinct broadcast domains.

---

## Key Takeaways

- Isolated VLANs require a Layer 3 routing mechanism (Router or Multilayer Switch) to forward packets between subnets.
- **ROAS** relies on a single physical link carrying multiple subinterfaces, creating potential bandwidth bottlenecks on high-traffic networks.
- **SVIs** process Inter-VLAN routing inside the switch ASIC hardware backplane at wire speed, making it the enterprise standard for campus networks.
- End devices in segmented VLANs must have their default gateway pointed to the corresponding subinterface or SVI IP address.

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
