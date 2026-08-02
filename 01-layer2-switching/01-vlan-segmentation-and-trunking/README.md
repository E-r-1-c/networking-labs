# VLAN Segmentation & 802.1Q Trunking

## Overview

This lab demonstrates how VLANs divide a switched network into separate Layer 2 broadcast domains and how 802.1Q trunks carry traffic for multiple VLANs between switches.

Devices assigned to the same VLAN can communicate across the trunk links. Devices in different VLANs cannot communicate because no Layer 3 routing is configured.

---

## Objectives

- Create VLANs for separate network groups
- Assign switch access ports to the correct VLANs
- Configure 802.1Q trunk links between switches
- Configure a dedicated native VLAN
- Restrict trunks to the required VLANs
- Verify VLAN membership and trunk operation
- Test same-VLAN and cross-VLAN connectivity

---

## Topology

![Network Topology](./images/topology.png)

---

## VLAN Design

| VLAN | Name | Purpose |
|---|---|---|
| 10 | HR | Human Resources |
| 20 | IT | Information Technology |
| 99 | Management | Switch management |
| 999 | Native | Unused native VLAN |

### Endpoint Addressing

| Device | IP Address | VLAN |
|---|---|---|
| PC0 | `192.168.10.1/24` | 10 |
| PC1 | `192.168.20.2/24` | 20 |
| PC2 | `192.168.10.3/24` | 10 |
| PC3 | `192.168.20.4/24` | 20 |

No default gateways were required because this lab tested Layer 2 connectivity only.

---

## Configuration

The switches were configured with VLANs 10, 20, 99, and 999.

```cisco
vlan 10
 name HR

vlan 20
 name IT

vlan 99
 name Management

vlan 999
 name Native
```

The end-device interfaces were configured as access ports and assigned to either VLAN 10 or VLAN 20.

```cisco
switchport mode access
switchport access vlan 10
```

```cisco
switchport mode access
switchport access vlan 20
```

The switch-to-switch links were configured as 802.1Q trunks.

```cisco
switchport mode trunk
switchport trunk native vlan 999
switchport trunk allowed vlan 10,20,99,999
```

VLAN 999 was used as the dedicated native VLAN, and the allowed VLAN list restricted the trunks to only the VLANs required by the design.

---

## Verification

### VLAN Configuration

```cisco
show vlan brief
```

![VLAN Configuration](./images/show-vlan-brief.png)

The output confirmed that VLANs 10, 20, 99, and 999 existed and that the access ports were assigned to the correct VLANs.

---

### Trunk Configuration

```cisco
show interfaces trunk
```

![Trunk Configuration](./images/show-interfaces-trunk.png)

The output confirmed that:

- The switch-to-switch links were operating as trunks
- VLAN 999 was configured as the native VLAN
- VLANs 10, 20, 99, and 999 were allowed across the trunks

---

### Connectivity Testing

Devices assigned to the same VLAN successfully communicated across the trunk links.

![Same-VLAN Communication](./images/same-vlan-ping.png)

Devices assigned to different VLANs could not communicate because no Layer 3 routing method was configured.

![Different-VLAN Communication](./images/different-vlan-ping.png)

These tests confirmed that the trunks extended each VLAN between switches while preserving isolation between separate VLANs.

---

## Key Takeaways

- VLANs create separate Layer 2 broadcast domains
- Access ports determine which VLAN a connected device belongs to
- 802.1Q trunks carry multiple VLANs between switches
- The native VLAN must match on both ends of a trunk
- Allowed VLAN lists restrict which VLANs can cross a trunk
- Communication between different VLANs requires Layer 3 routing

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
