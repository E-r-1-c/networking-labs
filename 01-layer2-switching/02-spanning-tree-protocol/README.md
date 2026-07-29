# Spanning Tree Protocol (STP & RSTP)

## Overview

This lab demonstrates how Spanning Tree Protocol (STP) and Rapid Spanning Tree Protocol (RSTP) prevent Layer 2 switching loops in a redundant switched network.

A three-switch topology with redundant links was built to observe root bridge election, spanning-tree port roles, and forwarding states. Rapid PVST+ was enabled, bridge priorities were configured to control root bridge selection, and BPDU Guard was implemented on access ports for edge protection.

---

## Objectives

- Build a redundant Layer 2 topology
- Enable Rapid PVST+
- Configure primary and secondary root bridges
- Identify root, designated, and alternate ports
- Verify forwarding and blocking states
- Configure PortFast and BPDU Guard
- Validate STP loop prevention

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

The following tasks were completed:

- Enabled Rapid PVST+ on all switches
- Configured SW0 as the primary root bridge
- Configured SW1 as the secondary root bridge
- Enabled PortFast and BPDU Guard on SW2 access ports
- Created redundant Layer 2 paths to observe STP behavior

---

## Rapid PVST+ Configuration

```cisco
spanning-tree mode rapid-pvst
```

---

## Root Bridge Configuration

SW0 was configured as the primary root bridge:

```cisco
spanning-tree vlan 10,20 root primary
```

SW1 was configured as the secondary root bridge:

```cisco
spanning-tree vlan 10,20 root secondary
```

---

## Access Port Protection

Configured PortFast and BPDU Guard on SW2:

```cisco
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

---

# Verification

## Root Bridge Verification

Verified the elected root bridge and spanning-tree information:

```cisco
show spanning-tree vlan 10
```

![Root Bridge Verification](./images/01-primary-root-verification.png)

---

## Port Role Verification

Verified root ports, designated ports, and alternate ports:

```cisco
show spanning-tree vlan 10
```

![Port Role Verification](./images/02-blocking-port-verification.png)

Results:

- SW0 became the root bridge.
- SW1 and SW2 selected root ports toward SW0.
- A redundant path was placed into an alternate/discarding state to prevent Layer 2 loops.

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

![BPDU Guard Verification](./images/03-bpduguard-errdisable.png)

---

# Key Takeaways

- STP prevents Layer 2 loops by controlling which ports forward traffic.
- Root bridge selection can be influenced using bridge priority.
- Root ports provide the best path toward the root bridge.
- Designated ports forward traffic for Layer 2 segments.
- Alternate ports maintain redundancy while preventing loops.
- Rapid PVST+ provides faster convergence than traditional STP.
- PortFast and BPDU Guard improve access-layer protection.

---

# Environment

- Cisco Packet Tracer
- Cisco IOS
