# Spanning Tree Protocol (STP & RSTP)

## Overview

This lab demonstrates how Spanning Tree Protocol (STP) prevents Layer 2 switching loops in a redundant network topology.

A three-switch topology was built using Rapid PVST+ to observe root bridge election, spanning-tree port roles, and forwarding states. Bridge priorities were modified to control root bridge placement, and PortFast with BPDU Guard was configured on the access switch to protect edge ports.

---

## Objectives

- Build a redundant Layer 2 topology
- Configure Rapid PVST+
- Control root bridge election
- Verify spanning-tree port roles and states
- Configure PortFast and BPDU Guard
- Validate loop prevention behavior

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

## Rapid PVST+

Enabled Rapid PVST+ on all switches:

```cisco
spanning-tree mode rapid-pvst
```

Rapid PVST+ creates a spanning-tree instance per VLAN and provides faster convergence than traditional STP.

---

## Root Bridge Selection

SW0 was configured as the primary root bridge for VLANs 10 and 20:

```cisco
spanning-tree vlan 10,20 root primary
```

SW1 was configured as the secondary root bridge:

```cisco
spanning-tree vlan 10,20 root secondary
```

These commands adjust bridge priority to control which switches are selected during the root bridge election.

---

## Access Port Protection

PortFast and BPDU Guard were enabled on SW2 access ports:

```cisco
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

PortFast allows end devices to enter the forwarding state faster. BPDU Guard places protected ports into an err-disabled state if unexpected BPDUs are received.

---

# Verification

## Root Bridge Verification

The spanning-tree output was used to verify the elected root bridge, bridge priority, and root path information.

Command:

```cisco
show spanning-tree vlan 10
```

![Root Bridge Verification](./images/01-primary-root-verification.png)

---

## Port Role Verification

Spanning-tree roles and states were verified to confirm forwarding and blocking behavior.

Command:

```cisco
show spanning-tree vlan 10
```

![Port Role Verification](./images/02-blocking-port-verification.png)

Observed results:

- SW0 became the root bridge for VLANs 10 and 20.
- SW0 ports operated as designated ports.
- SW1 and SW2 selected root ports toward SW0.
- A redundant path was placed into an alternate/discarding state.

---

## Root Bridge Summary

Root bridge information was verified using:

```cisco
show spanning-tree root
```

![Spanning Tree Root Verification](./images/show-spanning-tree-root.png)

---

## BPDU Guard Verification

BPDU Guard was tested on SW2 by introducing BPDU traffic on a PortFast-enabled access port.

The interface entered an err-disabled state, preventing an unauthorized switch from affecting the spanning-tree topology.

Verification:

```cisco
show interfaces status
```

Example:

```cisco
show interfaces fa0/1
```

![BPDU Guard Err Disable](./images/03-bpduguard-errdisable.png)

---

# STP Port Roles

| Port Role | Function |
|-----------|----------|
| Root Port | Best path from a non-root switch toward the root bridge |
| Designated Port | Forwarding port for a Layer 2 segment |
| Alternate Port | Backup path placed into a discarding state |

---

# Results

The topology successfully demonstrated STP loop prevention.

- SW0 was elected as the root bridge.
- SW1 provided secondary root bridge functionality.
- SW2 operated as an access switch with edge protection enabled.
- STP blocked redundant paths to prevent Layer 2 loops.
- BPDU Guard successfully protected access ports from unexpected BPDU traffic.

---

# Key Takeaways

- STP prevents Layer 2 loops by controlling forwarding paths.
- Bridge priority can be used to influence root bridge placement.
- Root ports provide the best path toward the root bridge.
- Designated ports forward traffic on Layer 2 segments.
- Alternate ports maintain redundancy without creating loops.
- Rapid PVST+ improves convergence speed.
- PortFast and BPDU Guard improve access-layer protection.

---

# Environment

- Cisco Packet Tracer
- Cisco IOS
