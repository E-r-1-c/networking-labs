# Spanning Tree Protocol (STP & RSTP)

## Overview

Configured a 3-switch redundant Layer 2 topology to observe Rapid PVST+ convergence, enforce primary/secondary root bridge placement, and secure access ports using PortFast and BPDU Guard.

---

## Network Topology & Switch Roles

![Network Topology](./images/topology.png)

| Switch | Role | Configuration |
| :--- | :--- | :--- |
| **SW0** | Primary Root Bridge | `spanning-tree vlan 10,20 root primary` |
| **SW1** | Secondary Root Bridge | `spanning-tree vlan 10,20 root secondary` |
| **SW2** | Access Switch | Default Priority + PortFast / BPDU Guard |

---

## Configuration & Verification

### 1. Rapid PVST+ & Root Tuning
Configured SW0 as the primary root bridge and SW1 as the secondary backup root bridge for VLANs 10 and 20.

```cisco
! SW0 - Primary Root Bridge
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 root primary

! SW1 - Secondary Root Bridge
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20 root secondary
```

**Verification:** Confirmed SW0 won the election and all active interfaces transitioned to **Designated Forwarding (`FWD`)** ports.

```cisco
show spanning-tree vlan 10
```

![Root Bridge Verification](./images/01-primary-root-verification.png)

---

### 2. Loop Prevention & Port Roles
Verified that non-root switches selected an optimal path to SW0 while placing redundant links into a non-forwarding state.

```cisco
show spanning-tree vlan 10
```

![Port Role Verification](./images/02-blocking-port-verification.png)

* **SW0:** All active ports operate as **Designated (`FWD`)**.
* **SW1 / SW2:** Selected `Fa0/1` as their **Root Port (`FWD`)**.
* **SW2:** Placed redundant link `Fa0/2` into **Alternate Discarding (`BLK`)** to break the switching loop.

```cisco
show spanning-tree root
```

![Spanning Tree Root Verification](./images/show-spanning-tree-root.png)

---

### 3. Edge Security (PortFast & BPDU Guard)
Enabled PortFast on SW2 access ports for immediate end-device connectivity, paired with BPDU Guard to shut down ports if an unauthorized switch is connected.

```cisco
! SW2 - Access Switch Protection
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

**Verification:** Injected BPDU traffic into an access port. BPDU Guard immediately placed the interface into an `err-disabled` state to protect the topology.

```cisco
show interfaces status err-disabled
```

![BPDU Guard Err Disable](./images/03-bpduguard-errdisable.png)

---

## Quick Reference: Port Roles & States

| Port Role | State | Function |
| :--- | :--- | :--- |
| **Root Port** | Forwarding (`FWD`) | Best path from a non-root switch back to the root bridge. |
| **Designated Port** | Forwarding (`FWD`) | Active forwarding port assigned to a network segment. |
| **Alternate Port** | Discarding (`BLK`) | Standby path blocked to prevent Layer 2 loops. |

---

## Key Takeaways

* **Deterministic Elections:** Manually defining bridge priorities prevents random access switches from becoming the root bridge.
* **Fast Convergence:** Rapid PVST+ reduces convergence times compared to legacy 802.1D STP by using explicit handshakes instead of fixed timers.
* **Proactive Security:** Combining PortFast with BPDU Guard protects access ports from rogue switches attempting to manipulate the spanning-tree topology.

---

## Environment

* **Simulation Tool:** Cisco Packet Tracer
* **Operating System:** Cisco IOS
