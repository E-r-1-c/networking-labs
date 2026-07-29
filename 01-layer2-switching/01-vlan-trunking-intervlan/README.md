# Lab 01 — VLANs & 802.1Q Trunking

## Overview

This lab demonstrates how VLANs segment a switched network into separate broadcast domains and how 802.1Q trunks carry traffic for multiple VLANs between switches.

The network uses a dedicated native VLAN and restricted allowed VLAN lists to reduce unnecessary traffic and improve switch-to-switch security.

---

## Objectives

- Create and configure VLANs
- Assign access ports to the correct VLAN
- Configure 802.1Q trunk links
- Configure a dedicated native VLAN
- Restrict allowed VLANs on trunk links
- Verify VLAN segmentation
- Verify trunk operation
- Validate same-VLAN connectivity across multiple switches

---

## Network Topology

![Lab Topology](./images/topology.png)

### Addressing

| Device | Interface | IP Address | VLAN |
|---------|-----------|------------|------|
| HR-PC1 | NIC | 10.10.10.10/24 | 10 |
| HR-PC2 | NIC | 10.10.10.11/24 | 10 |
| IT-PC1 | NIC | 10.10.20.10/24 | 20 |
| IT-PC2 | NIC | 10.10.20.11/24 | 20 |

### VLAN Design

| VLAN | Name | Purpose |
|------|------|---------|
| 10 | HR | Human Resources |
| 20 | IT | Information Technology |
| 99 | Native | Native VLAN |
