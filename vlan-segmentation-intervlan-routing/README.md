# Network Segmentation with VLANs and Inter-VLAN Routing

Designed and implemented a segmented network using VLANs, trunking, and Layer 3 routing to control communication between departments and enable connectivity across networks.

---

## Overview

This project simulates a multi-department network environment where systems are separated into different VLANs and communication is controlled through routing.

The network includes:

- VLAN-based segmentation (Engineering, HR, Sales)
- Trunk links between switches
- Inter-VLAN routing
- Transition from router-based routing to Layer 3 switching

---

## Network Design

The network is divided into multiple VLANs, each representing a different department:

- VLAN 10 — Engineering  
- VLAN 20 — HR  
- VLAN 30 — Sales  

Each VLAN operates as its own broadcast domain with a separate subnet.

---

## Key Configurations

### VLAN Segmentation
- Created VLANs on switches
- Assigned access ports to the correct VLANs
- Verified that devices in different VLANs cannot communicate by default

---

### Trunking
- Configured trunk links between switches
- Allowed VLAN traffic to pass between switches
- Verified VLAN propagation across the network

---

### Inter-VLAN Routing (Router-on-a-Stick)
- Configured subinterfaces on the router for each VLAN
- Assigned gateway IP addresses for each subnet
- Enabled communication between VLANs

---

### Layer 3 Switching (Multilayer Switch)
- Replaced router-on-a-stick with a multilayer switch
- Configured Switch Virtual Interfaces (SVIs)
- Enabled routing directly on the switch

---

## Validation

The network was tested to verify:

- Devices in the same VLAN can communicate
- Devices in different VLANs are isolated before routing
- Inter-VLAN communication works after routing is enabled
- Routing operates correctly across all VLANs

---

## Key Takeaways

- VLANs provide segmentation and isolate traffic
- Trunking allows VLANs to span multiple switches
- Inter-VLAN routing enables controlled communication
- Layer 3 switching improves efficiency over router-based routing
- Proper segmentation is a foundational concept for secure network design

---

## Environment

Built and tested in a controlled network simulation environment using:

- Cisco Packet Tracer
