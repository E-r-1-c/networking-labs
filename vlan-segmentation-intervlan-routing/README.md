# Network Segmentation with VLANs and Inter-VLAN Routing

Designed and implemented a segmented network using VLANs, trunking, and Layer 3 routing to control communication between departments and enable connectivity across networks.

---

## Overview

This project demonstrates how a network evolves from basic VLAN segmentation to full inter-VLAN communication and optimized Layer 3 switching.

The goal is to separate departments into isolated networks and then progressively enable controlled communication between them.

---

## Project Structure

This project is built in three stages:

- **Part 1 — VLAN Segmentation**  
  Devices are separated into different VLANs on a single switch.

- **Part 2 — Inter-VLAN Routing (Router-on-a-Stick)**  
  A router is introduced to allow communication between VLANs.

- **Part 3 — Layer 3 Switching**  
  Routing is moved to a multilayer switch for better performance.

---

## Part 1 — VLAN Segmentation

![VLAN Segmentation](images/vlan-segmentation.png)

- Created VLAN 10 (Engineering), VLAN 20 (HR), VLAN 30 (Sales)  
- Assigned switch ports to the correct VLANs  
- Verified that devices in different VLANs cannot communicate  

---

## Part 2 — Inter-VLAN Routing (Router-on-a-Stick)

![Inter-VLAN Routing](images/inter-vlan-routing.png)

- Configured trunk link between switches  
- Configured router subinterfaces for each VLAN  
- Assigned gateway IP addresses  
- Enabled communication between VLANs  

---

## Part 3 — Layer 3 Switching

![Layer 3 Switching](images/layer3-switching.png)

- Replaced router-on-a-stick with a multilayer switch  
- Configured Switch Virtual Interfaces (SVIs)  
- Enabled routing directly on the switch  
- Improved efficiency by removing router bottleneck  

---

## Network Design

The network is divided into multiple VLANs, each representing a department:

- VLAN 10 — Engineering → `10.0.0.0/26`  
- VLAN 20 — HR → `10.0.0.64/26`  
- VLAN 30 — Sales → `10.0.0.128/26`  

Each VLAN operates as its own broadcast domain with a separate subnet.

---

## Key Configurations

### VLAN Segmentation
- Created VLANs on switches  
- Assigned access ports to VLANs  
- Verified isolation between VLANs  

### Trunking
- Configured trunk links between switches  
- Allowed VLAN traffic across switches  

### Inter-VLAN Routing
- Configured router subinterfaces  
- Assigned gateway IPs per VLAN  
- Enabled communication between networks  

### Layer 3 Switching
- Configured SVIs on multilayer switch  
- Enabled routing on the switch  

---

## Validation

The network was tested to verify:

- Devices in the same VLAN can communicate  
- Devices in different VLANs are isolated before routing  
- Inter-VLAN communication works after routing is enabled  
- Routing functions correctly across all VLANs  

---

## Key Takeaways

- VLANs provide network segmentation and isolate traffic  
- Trunking allows VLANs to span multiple switches  
- Inter-VLAN routing enables controlled communication  
- Layer 3 switching improves performance and scalability  
- Network segmentation is critical for security and efficiency  

---

## Environment

- Cisco Packet Tracer
