# Network Security

Hands-on Cisco network security labs covering traffic filtering, access-layer attack prevention, secure remote management, and device access hardening.

This section shows how routers and switches restrict unauthorized traffic, protect access ports, validate address information, and secure administrative access to network devices.

---

## Focus Areas

- Standard and extended IPv4 access control lists
- Inbound and outbound traffic filtering
- ACL placement and processing order
- Permit and deny rule testing
- Implicit deny behavior
- Port Security and sticky MAC learning
- Maximum MAC address limits
- Port Security violation modes
- DHCP Snooping across multiple VLANs
- Trusted and untrusted switch interfaces
- Rogue DHCP server protection
- DHCP Snooping binding tables
- DHCP rate limiting on access ports
- Dynamic ARP Inspection
- ARP spoofing and poisoning prevention
- ARP validation using DHCP Snooping bindings
- IP Source Guard
- Source IP and MAC address validation
- Local user authentication
- RSA key generation and SSH version 2
- Secure VTY access
- Telnet removal
- Login restrictions and session timeouts
- Security failure, attack, and recovery testing

---

## Labs

| Lab | Description |
| :--- | :--- |
| **01 — ACL Traffic Filtering** | Configure standard and extended ACLs to control traffic between networks and verify permitted and denied connections. |
| **02 — Access-Layer Security Hardening** | Combine Port Security, DHCP Snooping, Dynamic ARP Inspection, and IP Source Guard across multiple VLANs and switches to block unauthorized devices, rogue DHCP responses, ARP spoofing, and source-address spoofing. |
| **03 — SSH & VTY Hardening** | Replace insecure Telnet access with SSH, configure local authentication, restrict remote access, and harden management sessions. |

---

## Lab Structure

Each lab includes an overview, objectives, network topology, security requirements, configuration, verification, attack or failure testing, recovery testing where appropriate, and key takeaways.

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
