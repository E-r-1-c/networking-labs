# Layer 2 Switching

Hands-on Cisco switching labs covering VLANs, trunking, redundancy, Layer 2 security, Inter-VLAN routing, and troubleshooting. This section focuses on building reliable switched networks, validating network behavior, and diagnosing common Layer 2 issues found in enterprise environments.

---

## Focus Areas
* VLAN segmentation and 802.1Q trunking
* Native VLAN configuration and allowed VLAN lists
* Spanning Tree Protocol (STP) and Rapid Spanning Tree Protocol (RSTP)
* Root bridge selection and path control
* EtherChannel link aggregation (LACP)
* Port Security and sticky MAC learning
* DHCP Snooping and Option 82 untrusted port behavior
* Dynamic ARP Inspection (DAI) and ARP spoofing mitigation
* Inter-VLAN routing via Router-on-a-Stick (ROAS) and Switch Virtual Interfaces (SVIs)
* Layer 2 troubleshooting and link failure recovery

---

## Labs

| Lab | Description |
| :--- | :--- |
| **01 — VLANs & Trunking** | Configure VLANs, access ports, 802.1Q trunks, native VLANs, and allowed VLANs. |
| **02 — STP & RSTP** | Prevent Layer 2 loops, control root bridge placement, and verify convergence after failures. |
| **03 — EtherChannel** | Configure and validate LACP EtherChannels for redundancy and increased bandwidth. |
| **04 — Port Security & DHCP Snooping** | Secure access switch ports using Port Security and mitigate rogue DHCP servers with DHCP Snooping. |
| **05 — Dynamic ARP Inspection** | Prevent ARP poisoning and man-in-the-middle attacks using DAI and trusted switch interfaces. |
| **06 — Inter-VLAN Routing** | Implement Inter-VLAN routing using Router-on-a-Stick (ROAS) subinterfaces and Layer 3 Switch Virtual Interfaces (SVIs). |
| **07 — Layer 2 Troubleshooting** | Diagnose and resolve common VLAN, trunk, STP, EtherChannel, and Layer 2 security issues. |

---

## Lab Structure
Each lab includes:
* Project overview
* Network topology
* Device configurations
* Verification commands
* Connectivity testing
* Key takeaways

---

## Environment
* Cisco Packet Tracer
