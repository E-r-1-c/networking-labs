# Inter-VLAN Routing: Router-on-a-Stick & Switch Virtual Interfaces

## Overview

This lab compares two methods of routing traffic between VLANs:

- **Router-on-a-Stick (ROAS)** using router subinterfaces
- **Switch Virtual Interfaces (SVIs)** on a multilayer switch

Each network uses the same VLANs and addressing plan, allowing both routing methods to be configured and tested independently.

The lab demonstrates how devices can remain separated into different Layer 2 broadcast domains while using a Layer 3 gateway to communicate between VLANs.

---

## Network Topology

![Network Topology](./images/topology.png)

The Packet Tracer file contains two independent network designs:

### Router-on-a-Stick Network

- **R0** provides the default gateways using router subinterfaces
- **SW0** provides Layer 2 switching and an 802.1Q trunk to R0
- **PC0** belongs to VLAN 10
- **PC1** belongs to VLAN 20

### Multilayer Switch Network

- **MLS0** provides the default gateways using SVIs
- **PC2** belongs to VLAN 10
- **PC3** belongs to VLAN 20

The two networks are not connected to each other. Each design was configured and verified separately.

---

## Objectives

- Create separate VLANs for different departments
- Assign access ports to the correct VLANs
- Configure an 802.1Q trunk for Router-on-a-Stick
- Configure router subinterfaces for multiple VLANs
- Configure SVIs on a multilayer switch
- Enable Layer 3 routing on the multilayer switch
- Verify directly connected VLAN routes
- Test communication between separate VLANs
- Compare external router-based routing with multilayer-switch routing

---

## VLAN Design

| VLAN | Name | Purpose | Network | Default Gateway |
|---|---|---|---|---|
| 10 | Sales | Sales department | `192.168.10.0/24` | `192.168.10.1` |
| 20 | Engineering | Engineering department | `192.168.20.0/24` | `192.168.20.1` |
| 99 | Management | Network device management | `192.168.99.0/24` | `192.168.99.1` |
| 999 | Native | Unused native VLAN | N/A | N/A |

---

## Device Addressing

### Router-on-a-Stick Network

| Device | Interface | IP Address | VLAN | Default Gateway |
|---|---|---|---|---|
| PC0 | FastEthernet0 | `192.168.10.10/24` | 10 | `192.168.10.1` |
| PC1 | FastEthernet0 | `192.168.20.10/24` | 20 | `192.168.20.1` |
| R0 | FastEthernet0/0.10 | `192.168.10.1/24` | 10 | N/A |
| R0 | FastEthernet0/0.20 | `192.168.20.1/24` | 20 | N/A |

### Multilayer Switch Network

| Device | Interface | IP Address | VLAN | Default Gateway |
|---|---|---|---|---|
| PC2 | FastEthernet0 | `192.168.10.10/24` | 10 | `192.168.10.1` |
| PC3 | FastEthernet0 | `192.168.20.10/24` | 20 | `192.168.20.1` |
| MLS0 | VLAN 10 SVI | `192.168.10.1/24` | 10 | N/A |
| MLS0 | VLAN 20 SVI | `192.168.20.1/24` | 20 | N/A |

The same IP networks are used in both designs because the two topologies are disconnected and operate independently.

---

## Configuration

## Router-on-a-Stick

### Switch Access Ports and Trunk

SW0 was configured with access ports for the Sales and Engineering devices. The connection between SW0 and R0 was configured as an 802.1Q trunk.

```cisco
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,99,999
 exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 exit

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 exit
```

The trunk carries traffic for the required VLANs over one physical link between the switch and router.

---

### Router Subinterfaces

R0 was configured with one subinterface for each routed VLAN.

```cisco
interface FastEthernet0/0
 no shutdown
 exit

interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
```

Each subinterface acts as the default gateway for its corresponding VLAN.

---

## Switch Virtual Interfaces

MLS0 was configured to route between VLANs internally using SVIs.

```cisco
ip routing

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit
```

The access ports connected to the end devices were assigned to their required VLANs.

```cisco
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 exit
```

The SVIs provide the default gateways for VLANs 10 and 20, while `ip routing` allows MLS0 to forward traffic between the VLAN networks.

---

## Verification

### VLAN and Port Assignment

VLAN creation and access-port membership were verified on SW0 and MLS0.

```cisco
show vlan brief
```

The output confirmed:

- VLANs 10, 20, 99, and 999 existed
- Sales devices were assigned to VLAN 10
- Engineering devices were assigned to VLAN 20
- Access ports belonged to the correct VLANs

---

### ROAS Trunk Verification

The trunk between SW0 and R0 was verified.

```cisco
show interfaces trunk
```

![Trunk Verification](./images/trunk-verification.png)

The output confirmed:

- The switch-to-router interface was operating as a trunk
- VLAN 999 was configured as the native VLAN
- VLANs 10, 20, 99, and 999 were allowed across the trunk

---

### Router Subinterface Verification

The status and addressing of the router subinterfaces were verified.

```cisco
show ip interface brief
```

The output confirmed:

- `FastEthernet0/0.10` was operational
- `FastEthernet0/0.20` was operational
- Each subinterface had the correct gateway address

---

### SVI Verification

The VLAN interfaces on MLS0 were verified.

```cisco
show ip interface brief
```

![SVI Verification](./images/svi-verification.png)

The output confirmed:

- VLAN 10 SVI was operational
- VLAN 20 SVI was operational
- Each SVI had the correct gateway address

---

### Routing Table Verification

The routing tables were checked on R0 and MLS0.

```cisco
show ip route
```

![Routing Table Verification](./images/routing-table.png)

The output confirmed directly connected routes for:

- `192.168.10.0/24`
- `192.168.20.0/24`

On R0, the routes were connected through router subinterfaces.

On MLS0, the routes were connected through VLAN interfaces.

---

### Router-on-a-Stick Connectivity Test

PC0 tested connectivity to PC1 through R0.

```text
PC0> ping 192.168.20.10
```

The successful response confirmed that R0 routed traffic between VLAN 10 and VLAN 20 through its subinterfaces.

---

### SVI Connectivity Test

PC2 tested connectivity to PC3 through MLS0.

```text
PC2> ping 192.168.20.10
```

![Inter-VLAN Connectivity](./images/intervlan-connectivity.png)

The successful response confirmed that MLS0 routed traffic between VLAN 10 and VLAN 20 using its SVIs.

---

## Traffic Flow

### Router-on-a-Stick

When a device in VLAN 10 sends traffic to VLAN 20:

1. The device sends the packet to its default gateway.
2. SW0 forwards the frame across the trunk with the VLAN 10 tag.
3. R0 receives the frame through the VLAN 10 subinterface.
4. R0 routes the packet toward the VLAN 20 network.
5. The packet returns across the same physical link using the VLAN 20 subinterface.
6. SW0 forwards the frame to the destination device in VLAN 20.

### Switch Virtual Interfaces

When a device in VLAN 10 sends traffic to VLAN 20:

1. The device sends the packet to the VLAN 10 SVI.
2. MLS0 performs the Layer 3 route lookup.
3. MLS0 forwards the packet through the VLAN 20 SVI.
4. The frame is delivered to the destination access port in VLAN 20.

---

## Results

Both inter-VLAN routing methods successfully allowed communication between VLAN 10 and VLAN 20.

Router-on-a-Stick used router subinterfaces and a single trunk connection between SW0 and R0.

The multilayer-switch design used SVIs to perform routing directly on MLS0 without sending the traffic through an external router.

The two designs achieved the same routing goal using different network architectures.

---

## Key Takeaways

- VLANs create separate Layer 2 broadcast domains
- Devices in different VLANs require a Layer 3 gateway to communicate
- Router-on-a-Stick uses router subinterfaces over an 802.1Q trunk
- Each router subinterface must match the correct VLAN ID
- SVIs provide Layer 3 gateways directly on a multilayer switch
- Multilayer-switch routing requires `ip routing`
- The ROAS and SVI networks must be tested independently when using identical addressing
- Routing tables verify that the VLAN networks are directly connected
- Successful pings confirm connectivity but should be supported by interface, VLAN, trunk, and routing verification
- Both methods solve the same problem using different designs

---

## Environment

- Cisco Packet Tracer
````
