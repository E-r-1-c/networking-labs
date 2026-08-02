# Inter-VLAN Routing: Router-on-a-Stick & SVIs

## Overview

This lab compares two methods of routing traffic between VLANs:

- **Router-on-a-Stick (ROAS)** using router subinterfaces
- **Switch Virtual Interfaces (SVIs)** on a multilayer switch

Two disconnected networks were built in the same Packet Tracer file. Both use the same VLANs and IP addressing so each routing method can be configured and tested independently.

---

## Objectives

- Separate Sales and Engineering hosts into different VLANs
- Configure Router-on-a-Stick using an 802.1Q trunk and router subinterfaces
- Configure inter-VLAN routing using SVIs on a multilayer switch
- Verify Layer 3 interfaces and directly connected routes
- Test communication between VLAN 10 and VLAN 20
- Compare external router-based routing with multilayer-switch routing

---

## Topology

![Network Topology](./images/topology.png)

### Router-on-a-Stick Network

- **R0** provides the VLAN gateways through subinterfaces
- **SW0** connects the VLANs to R0 through an 802.1Q trunk
- **PC0** belongs to VLAN 10
- **PC1** belongs to VLAN 20

### SVI Network

- **MLS0** provides the VLAN gateways through SVIs
- **PC2** belongs to VLAN 10
- **PC3** belongs to VLAN 20

The two networks are not connected and were tested separately.

---

## Network Design

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | Sales | `192.168.10.0/24` | `192.168.10.1` |
| 20 | Engineering | `192.168.20.0/24` | `192.168.20.1` |
| 999 | Native | N/A | N/A |

PC0 and PC2 use `192.168.10.10/24`, while PC1 and PC3 use `192.168.20.10/24`. The duplicate addresses are valid because the two networks are disconnected.

---

## Configuration

### Router-on-a-Stick

SW0 was configured with VLAN access ports and an 802.1Q trunk toward R0.

```cisco
vlan 10
 name Sales

vlan 20
 name Engineering

vlan 999
 name Native

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,999

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

Each subinterface provides the default gateway for its corresponding VLAN.

---

### Switch Virtual Interfaces

MLS0 was configured with VLAN access ports, SVIs, and Layer 3 routing.

```cisco
vlan 10
 name Sales

vlan 20
 name Engineering

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

ip routing
```

The SVIs provide the VLAN gateways, while `ip routing` allows MLS0 to forward traffic between the two networks.

---

## Verification

### Router-on-a-Stick Verification

The trunk was verified on SW0, while the subinterfaces and routing table were verified on R0.

```cisco
show interfaces trunk
show ip interface brief
show ip route
```

![ROAS Verification](./images/roas-verification.png)

The output confirmed:

- The switch-to-router link was operating as a trunk
- VLAN 999 was configured as the native VLAN
- The router subinterfaces were operational
- VLAN 10 and VLAN 20 appeared as directly connected networks

---

### SVI Verification

The VLAN interfaces and routing table were verified on MLS0.

```cisco
show ip interface brief
show ip route
```

![SVI Verification](./images/svi-verification.png)

The output confirmed:

- The VLAN 10 and VLAN 20 SVIs were operational
- Each SVI used the correct gateway address
- Both VLAN networks appeared as directly connected routes

---

### Connectivity Testing

Cross-VLAN connectivity was tested independently in both networks.

```text
PC0> ping 192.168.20.10
PC2> ping 192.168.20.10
```

![Inter-VLAN Connectivity](./images/intervlan-connectivity.png)

Both tests succeeded, confirming that ROAS and SVI routing forwarded traffic between VLAN 10 and VLAN 20.

---

## Key Takeaways

- Devices in different VLANs require a Layer 3 gateway to communicate
- Router-on-a-Stick uses router subinterfaces over an 802.1Q trunk
- SVI routing performs inter-VLAN routing directly on a multilayer switch
- Both methods achieved the same result using different network designs
- Interface and routing-table verification confirms more than connectivity testing alone

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
