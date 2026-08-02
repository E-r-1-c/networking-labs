# Inter-VLAN Routing: Router-on-a-Stick & SVIs

## Overview

This lab compares two methods of routing traffic between VLANs:

- Router-on-a-Stick using router subinterfaces
- Switch Virtual Interfaces on a multilayer switch

Two separate networks were built in the same Packet Tracer file. Both use the same VLANs and addressing plan but operate independently.

---

## Topology

![Network Topology](./images/topology.png)

### Router-on-a-Stick Network

- R0 provides the VLAN gateways through subinterfaces
- SW0 connects the VLANs to R0 through an 802.1Q trunk
- PC0 belongs to VLAN 10
- PC1 belongs to VLAN 20

### SVI Network

- MLS0 provides the VLAN gateways through SVIs
- PC2 belongs to VLAN 10
- PC3 belongs to VLAN 20

---

## VLAN and Addressing Plan

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | Sales | `192.168.10.0/24` | `192.168.10.1` |
| 20 | Engineering | `192.168.20.0/24` | `192.168.20.1` |
| 99 | Management | `192.168.99.0/24` | `192.168.99.1` |
| 999 | Native | N/A | N/A |

| Device | IP Address | VLAN | Default Gateway |
|---|---|---|---|
| PC0 | `192.168.10.10/24` | 10 | `192.168.10.1` |
| PC1 | `192.168.20.10/24` | 20 | `192.168.20.1` |
| PC2 | `192.168.10.10/24` | 10 | `192.168.10.1` |
| PC3 | `192.168.20.10/24` | 20 | `192.168.20.1` |

The duplicate addresses are valid because the two networks are disconnected.

---

## Configuration

### Router-on-a-Stick

SW0 was configured with access ports for VLANs 10 and 20 and an 802.1Q trunk toward R0.

```cisco
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,99,999

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
```

R0 was configured with one subinterface for each routed VLAN.

```cisco
interface FastEthernet0/0
 no shutdown

interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

---

### Switch Virtual Interfaces

MLS0 was configured with access ports, VLAN interfaces, and Layer 3 routing.

```cisco
ip routing

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```

---

## Verification

### Router-on-a-Stick

The trunk and router subinterfaces were verified with:

```cisco
show interfaces trunk
show ip interface brief
show ip route
```

![ROAS Verification](./images/roas-verification.png)

The output confirmed:

- The switch-to-router link was trunking
- VLAN 999 was the native VLAN
- VLANs 10, 20, 99, and 999 were allowed
- The router subinterfaces were operational
- Both VLAN networks appeared as directly connected routes

---

### SVI Routing

The multilayer switch interfaces and routing table were verified with:

```cisco
show ip interface brief
show ip route
```

![SVI Verification](./images/svi-verification.png)

The output confirmed:

- VLAN 10 and VLAN 20 SVIs were operational
- Each SVI had the correct gateway address
- Both VLAN networks appeared as directly connected routes

---

### Connectivity Testing

Cross-VLAN communication was tested separately in each network.

```text
PC0> ping 192.168.20.10
PC2> ping 192.168.20.10
```

![Inter-VLAN Connectivity](./images/intervlan-connectivity.png)

Both tests succeeded, confirming that Router-on-a-Stick and SVI routing could forward traffic between VLAN 10 and VLAN 20.

---

## Key Takeaways

- VLANs require a Layer 3 gateway to communicate with other VLANs
- Router-on-a-Stick uses subinterfaces over a trunk link
- SVIs route traffic directly on a multilayer switch
- Router subinterfaces and SVIs must match the correct VLAN networks
- The routing table confirms that the VLAN networks are reachable
- Both designs solve the same problem using different network architectures

---

## Environment

- Cisco Packet Tracer

