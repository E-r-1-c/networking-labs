# Spanning Tree Protocol (STP & RSTP)

## Overview

This lab demonstrates how Spanning Tree Protocol (STP) and Rapid Spanning Tree Protocol (RSTP) prevent Layer 2 switching loops in a redundant network topology.

A three-switch redundant topology was built to observe root bridge election, spanning-tree port roles, and port states. Bridge priorities were configured to control which switch became the primary and secondary root bridge. PortFast and BPDU Guard were also configured on access ports to improve edge network protection.

---

## Objectives

- Build a redundant Layer 2 topology
- Enable Rapid PVST+
- Observe the default root bridge election
- Configure a primary root bridge
- Configure a secondary root bridge
- Identify root, designated, and alternate ports
- Verify forwarding and discarding port states
- Configure PortFast on access ports
- Enable BPDU Guard protection
- Verify BPDU Guard err-disabled behavior
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

# Configuration

The following tasks were completed during this lab:

- Configured a redundant three-switch topology
- Enabled Rapid PVST+ on each switch
- Configured SW0 as the primary root bridge
- Configured SW1 as the secondary root bridge
- Enabled PortFast and BPDU Guard on SW2 access ports
- Verified the elected root bridge
- Identified each interface's spanning-tree role and state
- Confirmed that one redundant path was placed into a discarding state

---

## Rapid PVST+ Configuration

Enabled Rapid PVST+ on all switches:

```cisco
spanning-tree mode rapid-pvst
```

---

## Primary Root Bridge Configuration

SW0 was configured as the preferred root bridge for VLANs 10 and 20:

```cisco
spanning-tree vlan 10,20 root primary
```

This automatically lowers the bridge priority so SW0 becomes the root bridge for the specified VLANs.

---

## Secondary Root Bridge Configuration

SW1 was configured as the backup root bridge:

```cisco
spanning-tree vlan 10,20 root secondary
```

This configures SW1 with a higher priority than the primary root bridge but lower priority than default switches.

---

## Access Port Protection Configuration

PortFast and BPDU Guard were enabled on SW2 access ports:

```cisco
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

PortFast allows end devices connected to access ports to transition into the forwarding state immediately without waiting for normal spanning-tree convergence.

BPDU Guard protects edge ports by placing the interface into an err-disabled state if unexpected BPDUs are received.

---

# Verification

## Root Bridge Verification

The spanning-tree output was checked on the switches to verify the elected root bridge, bridge ID, priority, and root path information.

Command used:

```cisco
show spanning-tree
```

![Root Bridge Verification](./images/root-bridge.png)

---

## Port Role Verification

The spanning-tree output was used to identify root, designated, and alternate ports.

Command used:

```cisco
show spanning-tree
```

![Port Role Verification](./images/port-roles.png)

Expected results:

- The root bridge contains only designated ports.
- Each non-root switch has one root port toward the root bridge.
- Redundant links contain alternate ports placed into a discarding state.

---

## Root Bridge Summary Verification

The root bridge information for VLANs 10 and 20 was verified by reviewing the spanning-tree output.

Command used:

```cisco
show spanning-tree
```

![Spanning Tree Root Verification](./images/show-spanning-tree-root.png)

---

## BPDU Guard Verification

BPDU Guard was tested on the access switch (SW2) by introducing BPDUs on a PortFast-enabled access port.

When SW2 received an unexpected BPDU, BPDU Guard placed the affected interface into an err-disabled state to prevent an unauthorized switch from affecting the spanning-tree topology.

The interface status was verified using:

```cisco
show interfaces status
```

Example:

```cisco
show interfaces fa0/1
```

![BPDU Guard Err Disable](./images/bpduguard-errdisable.png)

---

# Expected Port Behavior

| Port Role | Purpose |
|-----------|---------|
| Root Port | Best path from a non-root switch toward the root bridge |
| Designated Port | Forwarding port selected for each Layer 2 segment |
| Alternate Port | Backup path placed into a discarding state to prevent loops |

The root bridge should only have designated forwarding ports.

Each non-root switch selects the best path toward the root bridge as its root port.

When redundant Layer 2 paths exist, STP places one path into an alternate/discarding state to prevent:

- Broadcast storms
- MAC address table instability
- Layer 2 switching loops

---

# Key Takeaways

- STP prevents Layer 2 loops by controlling which switch ports forward traffic.
- The switch with the lowest Bridge ID becomes the root bridge.
- Bridge priority can be configured to control root bridge placement.
- Every non-root switch selects one root port toward the root bridge.
- Each network segment selects one designated port.
- Alternate ports provide redundancy while remaining in a discarding state.
- Rapid PVST+ provides faster convergence than traditional STP.
- PortFast improves connectivity for end devices by allowing faster forwarding.
- BPDU Guard protects access ports by placing interfaces into an err-disabled state when unexpected BPDUs are received.
- Redundant links remain available without creating Layer 2 switching loops.

---

# Environment

- Cisco Packet Tracer
- Cisco IOS
