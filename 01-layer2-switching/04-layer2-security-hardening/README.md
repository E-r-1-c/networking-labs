# Layer 2 Security Hardening: Port Security & DHCP Snooping

## Overview

This lab demonstrates how Layer 2 security features protect an access switch from unauthorized devices and rogue DHCP servers.

Port Security was configured to restrict access to authorized devices by limiting MAC addresses on access ports. DHCP Snooping was implemented to allow DHCP services only from trusted interfaces while blocking unauthorized DHCP server responses from untrusted ports.

A rogue DHCP server was introduced on the same VLAN as legitimate clients to simulate a real-world access-layer attack scenario.

---

## Objectives

- Configure Port Security on access ports
- Enable sticky MAC address learning
- Configure Port Security violation handling
- Enable DHCP Snooping
- Define trusted and untrusted switch interfaces
- Verify DHCP Snooping bindings
- Test protection against rogue DHCP servers
- Validate access-layer security controls

---

## Network Topology

![Network Topology](./images/topology.png)

---

## Device Roles

| Device | Role | Interface | Description |
|--------|------|-----------|-------------|
| R1 | Legitimate DHCP Server | Gi0/1 | Trusted DHCP source |
| SW1 | Access Switch | Multiple | Layer 2 security controls configured |
| PC-User | Authorized Client | Fa0/2 | Protected access port |
| Rogue-DHCP | Unauthorized DHCP Server | Fa0/3 | Used to test DHCP Snooping |

---

## Configuration

The access switch was hardened using Port Security and DHCP Snooping.

The following configurations were implemented:

- Created the user VLAN for client communication.
- Connected the legitimate DHCP server to a trusted switch interface.
- Enabled Port Security on the authorized client port.
- Limited the client port to a single MAC address.
- Enabled sticky MAC learning.
- Configured violation handling for unauthorized devices.
- Enabled DHCP Snooping globally.
- Enabled DHCP Snooping for the user VLAN.
- Configured the DHCP server-facing interface as trusted.
- Maintained client-facing interfaces as untrusted.

---

# Verification

## Port Security Verification

Port Security status was verified on the authorized client interface.

Command used:

```cisco
show port-security interface fa0/2
