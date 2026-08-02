# LACP EtherChannel & 802.1Q Trunking

## Overview

This lab demonstrates how Link Aggregation Control Protocol (LACP) combines multiple physical switch links into a single logical EtherChannel interface.

Two FastEthernet links were bundled into Port-Channel1 using LACP. The Port-Channel was configured as an 802.1Q trunk carrying VLANs 10 and 20, while Rapid PVST+ treated the EtherChannel as a single logical path.

---

## Objectives

- Configure an LACP EtherChannel between switches
- Bundle multiple physical interfaces into a logical Port-Channel
- Configure 802.1Q trunking on the Port-Channel
- Verify EtherChannel negotiation and operation
- Verify trunk operation and allowed VLANs
- Verify STP interaction with EtherChannel

---

## Topology

![Network Topology](./images/topology.png)

| Switch | Role | Physical Interfaces | Logical Interface | LACP Mode |
|---|---|---|---|---|
| SW0 | Distribution switch | Fa0/1–Fa0/2 | Port-Channel1 | Active |
| SW1 | Access switch | Fa0/1–Fa0/2 | Port-Channel1 | Active |

---

## Configuration

Fa0/1 and Fa0/2 were bundled into Port-Channel1 using LACP, and the Port-Channel was configured as a trunk carrying VLANs 10 and 20.

```cisco
interface range FastEthernet0/1 - 2
 channel-group 1 mode active

interface Port-Channel1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

---

## Verification

### EtherChannel Verification

The EtherChannel bundle was verified with:

```cisco
show etherchannel summary
```

![EtherChannel Summary Verification](./images/01-etherchannel-summary.png)

The output confirmed:

- Port-Channel1 was operational
- LACP was being used
- Fa0/1 and Fa0/2 were bundled successfully
- Both member interfaces displayed the `P` flag

---

### Trunk Verification

The Port-Channel trunk was verified with:

```cisco
show interfaces trunk
```

![Interface Trunk Verification](./images/02-interfaces-trunk.png)

The output confirmed:

- Port-Channel1 was operating as a trunk
- VLANs 10 and 20 were allowed across the trunk
- Multi-VLAN traffic was carried over the logical interface

---

### STP EtherChannel Verification

STP operation was verified with:

```cisco
show spanning-tree vlan 10
```

![Spanning Tree Port-Channel Verification](./images/03-stp-portchannel.png)

The output confirmed:

- Rapid PVST+ calculated the topology using Port-Channel1
- STP treated the EtherChannel as a single logical path
- STP did not calculate separate paths for each physical member interface

---

## LACP Modes & Flags

| Mode / Flag | Description |
|---|---|
| Active | Initiates LACP negotiation |
| Passive | Responds to LACP negotiation |
| S | Port-Channel operates as a Layer 2 interface |
| U | Port-Channel is currently in use |
| P | Physical interface is bundled in the Port-Channel |

---

## Key Takeaways

- EtherChannel combines multiple physical links into one logical interface
- LACP dynamically negotiates the EtherChannel between switches
- Trunk configuration is applied to the Port-Channel interface
- STP treats the Port-Channel as one logical path
- EtherChannel provides additional bandwidth and link redundancy

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
