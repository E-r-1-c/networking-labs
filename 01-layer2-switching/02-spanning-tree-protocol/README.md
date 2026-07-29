# Spanning Tree Protocol (STP & RSTP)

## Overview

This lab demonstrates how Spanning Tree Protocol (STP) and Rapid Spanning Tree Protocol (RSTP) prevent Layer 2 switching loops in a redundant network topology.

A three-switch redundant topology was built to observe root bridge election, port roles, and port states. Bridge priorities were then configured to control which switch became the primary and secondary root bridge.

---

## Objectives

- Build a redundant Layer 2 topology
- Enable Rapid PVST+
- Observe the default root bridge election
- Configure a primary root bridge
- Configure a secondary root bridge
- Identify root, designated, and alternate ports
- Verify forwarding and discarding port states
- Confirm that the topology remains loop-free

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

## Configuration

The following tasks were completed during this lab:

- Configured a redundant three-switch topology
- Enabled Rapid PVST+ on each switch
- Configured SW0 as the primary root bridge
- Configured SW1 as the secondary root bridge
- Verified the elected root bridge
- Identified each interface's spanning-tree role and state
- Confirmed that one redundant path was placed into a discarding state

### Primary Root Bridge

```cisco
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 root primary
```

### Secondary Root Bridge

```cisco
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 root secondary
```

---

## Verification

### Root Bridge Verification

The spanning-tree output was checked to confirm the elected root bridge and its bridge priority.

```cisco
show spanning-tree
```

![Root Bridge Verification](./images/root-bridge.png)

---

### Port Role Verification

The spanning-tree output was used to identify root, designated, and alternate ports.

```cisco
show spanning-tree
```

![Port Role Verification](./images/port-roles.png)

---

### Root Summary

The root bridge for each VLAN was verified using the root summary command.

```cisco
show spanning-tree root
```

![Spanning Tree Root Summary](./images/show-spanning-tree-root.png)

---

## Expected Port Behavior

| Port Role | Purpose |
|-----------|---------|
| Root Port | Best path from a non-root switch toward the root bridge |
| Designated Port | Forwarding port selected for a network segment |
| Alternate Port | Backup path that remains in the discarding state until needed |

The root bridge should have only designated ports. Each non-root switch should have one root port, while one redundant link in the topology should contain an alternate port to prevent a Layer 2 loop.

---

## Key Takeaways

- STP prevents Layer 2 loops by placing redundant links into a non-forwarding state.
- The switch with the lowest bridge ID becomes the root bridge.
- Bridge priority can be configured to control root bridge placement.
- Every non-root switch selects one root port toward the root bridge.
- Each network segment selects one designated port.
- RSTP uses alternate ports and converges faster than traditional STP.
- A redundant link remains available without creating a switching loop.

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
