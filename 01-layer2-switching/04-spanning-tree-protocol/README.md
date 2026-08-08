# Spanning Tree Protocol: STP & RSTP

## Overview

This lab demonstrates how Spanning Tree Protocol prevents Layer 2 loops in a switched network with redundant links.

Rapid PVST+ was configured across a three-switch topology. Root bridge selection was controlled for VLANs 10 and 20, while PortFast and BPDU Guard were applied to access ports to protect the spanning-tree topology.

---

## Objectives

- Build a redundant Layer 2 topology
- Configure Rapid PVST+
- Control root bridge selection
- Identify root, designated, and alternate ports
- Verify forwarding and discarding states
- Configure PortFast and BPDU Guard
- Test protection against unexpected BPDU traffic

---

## Topology

![Network Topology](./images/topology.png)

| Switch | Role |
|---|---|
| SW0 | Primary root bridge |
| SW1 | Secondary root bridge |
| SW2 | Access switch |

---

## Configuration

Rapid PVST+ was enabled across the switched topology.

```cisco
spanning-tree mode rapid-pvst
```

SW0 was configured as the preferred root bridge for VLANs 10 and 20.

```cisco
spanning-tree vlan 10,20 root primary
```

SW1 was configured as the secondary root bridge.

```cisco
spanning-tree vlan 10,20 root secondary
```

PortFast and BPDU Guard were enabled on the SW2 access ports connected to end devices.

PortFast allowed the access ports to transition directly to forwarding, while BPDU Guard protected the topology by disabling a protected port if unexpected BPDU traffic was received.

---

## Verification

### Root Bridge Verification

The spanning-tree state was verified on SW0.

```cisco
show spanning-tree
```

![Root Bridge Verification](./images/01-primary-root-verification.png)

The output confirmed that SW0 was elected as the root bridge for the configured VLANs.

---

### Port Role Verification

The spanning-tree port roles and states were verified on SW2.

```cisco
show spanning-tree
```

![Port Role Verification](./images/02-blocking-port-verification.png)

The output confirmed that:

- SW2 selected its root port toward SW0
- Designated ports remained in the forwarding state
- The redundant path was placed into an alternate discarding state
- The topology remained loop-free while maintaining redundancy

---

### BPDU Guard Test

BPDU Guard was tested by introducing BPDU traffic on a protected SW2 access port.

![BPDU Guard Verification](./images/03-bpduguard-errdisable.png)

The switch generated a syslog message and placed the affected interface into an err-disabled state.

This prevented an unauthorized switch from influencing the spanning-tree topology.

---

## Key Takeaways

- STP prevents Layer 2 loops by blocking redundant forwarding paths
- Root bridge selection can be controlled through bridge priority
- Root ports provide the best path toward the root bridge
- Alternate ports preserve redundancy without creating a loop
- Rapid PVST+ provides faster convergence for each VLAN
- PortFast and BPDU Guard protect end-device access ports

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
