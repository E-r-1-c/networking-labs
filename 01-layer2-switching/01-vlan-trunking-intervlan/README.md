# VLAN Segmentation & 802.1Q Trunking

## Overview

This lab demonstrates how VLANs segment a switched network into separate broadcast domains and how 802.1Q trunk links carry traffic for multiple VLANs between switches.

The network uses a dedicated native VLAN and restricted allowed VLAN lists to reduce unnecessary VLAN traffic across trunk links.

---

## Objectives

- Create VLANs
- Configure access ports
- Configure 802.1Q trunk links
- Configure a dedicated native VLAN
- Restrict allowed VLANs
- Verify VLAN segmentation
- Verify trunk operation
- Validate end-to-end connectivity

---

## Network Topology

![Topology](images/topology.png)

### VLAN Design

| VLAN | Name | Purpose |
|------|------|---------|
| 10 | HR | Human Resources |
| 20 | IT | Information Technology |
| 99 | Management | Switch Management |
| 999 | Native | Native VLAN |

---

## Configuration

Explain what you configured here.

---

## Verification

### VLAN Verification

`show vlan brief`

![VLAN Verification](images/show-vlan-brief.png)

---

### Trunk Verification

`show interfaces trunk`

![Trunk Verification](images/show-interfaces-trunk.png)

---

### Connectivity Tests

Successful communication between devices in the same VLAN.

![Same VLAN](images/same-vlan-ping.png)

Communication between different VLANs fails as expected because no Layer 3 routing is configured.

![Different VLAN](images/different-vlan-ping.png)

---

## Key Takeaways

- VLANs create separate Layer 2 broadcast domains.
- Access ports determine VLAN membership.
- Trunk links carry traffic for multiple VLANs between switches.
- Native VLANs should be configured consistently across both ends of a trunk.
- Restricting allowed VLANs limits unnecessary VLAN traffic across trunk links.
- Devices in different VLANs cannot communicate without Layer 3 routing.

---

## Environment

- Cisco Packet Tracer
