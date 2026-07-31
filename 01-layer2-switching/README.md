# Layer 2 Switching

Hands-on Cisco switching labs covering VLANs, trunking, Inter-VLAN routing, redundancy, Layer 2 security, and troubleshooting. This section focuses on building reliable switched networks, validating network behavior, and diagnosing common Layer 2 issues found in enterprise environments.

---

## Focus Areas
* VLAN segmentation and 802.1Q trunking
* Inter-VLAN Routing (ROAS & SVIs)
* Native VLAN security & trunk pruning
* Spanning Tree Protocol (STP) and Rapid Spanning Tree Protocol (RSTP)
* Root bridge election, priority tuning, and path control
* EtherChannel link aggregation (LACP & PaGP)
* Port Security and sticky MAC learning
* DHCP Snooping & Dynamic ARP Inspection (DAI)
* PortFast and BPDU Guard
* Layer 2 troubleshooting & failure recovery

---

## Labs

| Lab | Description |
| :--- | :--- |
| **01 — VLANs & Trunking** | Configure VLANs, access ports, 802.1Q trunks, native VLAN security, and allowed VLAN lists. |
| **02 — Inter-VLAN Routing** | Implement Inter-VLAN routing using Router-on-a-Stick (ROAS) subinterfaces and Layer 3 Switch Virtual Interfaces (SVIs). |
| **03 — STP & RSTP** | Prevent Layer 2 loops, enforce root bridge placement via priority tuning, and verify RSTP convergence. |
| **04 — EtherChannel** | Configure and validate LACP EtherChannels across multi-switch topologies for bandwidth aggregation and link redundancy. |
| **05 — Layer 2 Security** | Secure access ports using Port Security, DHCP Snooping, Dynamic ARP Inspection (DAI), PortFast, and BPDU Guard. |
| **06 — Layer 2 Troubleshooting** | Diagnose and resolve common VLAN mismatches, trunk failures, STP root misconfigurations, and port violations. |

---

## Lab Structure
Each lab folder contains:
* **Project Overview:** Real-world design context and technical scope.
* **Network Topology:** Device naming, interface mapping, and IP addressing schemes.
* **Device Configurations:** Clean, production-ready Cisco IOS syntax.
* **Verification Commands:** CLI output demonstrating operational state and features.
* **Key Takeaways:** Engineering insights and protocol nuances observed during testing.

---

## Environment
* **Platform:** Cisco Packet Tracer / Real Hardware Lab
