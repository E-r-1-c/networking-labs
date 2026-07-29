# Spanning Tree Protocol (STP & RSTP)

## Overview

This lab demonstrates how Spanning Tree Protocol (STP) and Rapid Spanning Tree Protocol (RSTP) prevent Layer 2 switching loops in a redundant network topology.

A three-switch topology with redundant links was created to observe root bridge election, spanning-tree port roles, and forwarding states. Bridge priorities were configured to control root bridge placement, and BPDU Guard was enabled on access ports to protect the spanning-tree topology.

---

## Objectives

- Build a redundant Layer 2 switching topology
- Enable Rapid PVST+
- Control root bridge election using bridge priority
- Identify root, designated, and alternate ports
- Verify forwarding and discarding states
- Configure PortFast and BPDU Guard on edge ports
- Confirm loop prevention behavior

---

## Network Topology

![Network Topology](./images/topology.png)

---

## Switch Roles

| Switch | Role |
|--------|------|
| SW0 | Primary Root Bridge |
| SW1 | Secondary Root Bridge |
| SW2 | Access Switch |

---

# Configuration

## Enable Rapid PVST+

Rapid PVST+ was enabled on all switches:

```cisco
spanning-tree mode rapid-pvst
