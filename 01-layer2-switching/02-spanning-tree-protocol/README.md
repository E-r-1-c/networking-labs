# Spanning Tree Protocol (STP & RSTP)

## Overview

This lab demonstrates how Spanning Tree Protocol (STP) prevents Layer 2 switching loops in a redundant switched network.

A three-switch topology with redundant links was created to observe root bridge election, spanning-tree port roles, and forwarding states. Rapid PVST+ was configured, root bridge selection was controlled through bridge priority, and BPDU Guard was enabled on access ports to provide edge protection.

---

## Objectives

- Build a redundant Layer 2 topology
- Configure Rapid PVST+
- Control root bridge selection
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

## Configuration

The topology was built with redundant Layer 2 connections between three switches to demonstrate STP path selection.

Rapid PVST+ was enabled to provide per-VLAN spanning-tree operation.

Root bridge priorities were adjusted so:

- SW0 became the preferred root bridge for VLANs 10 and 20
- SW1 provided secondary root bridge functionality
- SW2 operated as an access switch with edge-port protection enabled

PortFast and BPDU Guard were configured on SW2 access ports to protect end-device connections.

---

## Verification

### Root Bridge Verification

Verified the elected root bridge, bridge priority, and spanning-tree information.

```cisco
show spanning-tree vlan 10
```

![Root Bridge Verification](./images/01-primary-root-verification.png)

---

### Port Role Verification

Verified spanning-tree port roles and states.

```cisco
show spanning-tree vlan 10
```

![Port Role Verification](./images/02-blocking-port-verification.png)

The output confirmed:

- SW0 was elected as the root bridge.
- Non-root switches selected root ports toward SW0.
- Redundant paths were placed into an alternate/discarding state.

---

### BPDU Guard Verification

BPDU Guard was tested on SW2 by introducing BPDU traffic on a protected access port.

The interface entered an err-disabled state, preventing an unauthorized switch from affecting the spanning-tree topology.

Verification:

```cisco
show interfaces status
```

Interface details:

```cisco
show interfaces fa0/1
```

![BPDU Guard Verification](./images/03-bpduguard-errdisable.png)

---

## STP Port Roles

| Port Role | Purpose |
|-----------|---------|
| Root Port | Best path from a non-root switch toward the root bridge |
| Designated Port | Forwarding port responsible for a Layer 2 segment |
| Alternate Port | Backup path placed into a discarding state |

---

## Key Takeaways

- STP prevents Layer 2 loops by controlling which paths are allowed to forward traffic.
- Root bridge selection can be influenced using bridge priority.
- Root ports provide the best path toward the root bridge.
- Designated ports forward traffic for Layer 2 segments.
- Alternate ports maintain redundancy while preventing loops.
- Rapid PVST+ provides faster convergence through per-VLAN spanning-tree operation.
- PortFast and BPDU Guard improve access-layer protection.

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
