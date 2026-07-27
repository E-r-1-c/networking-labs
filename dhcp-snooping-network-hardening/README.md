# DHCP Snooping – Preventing Rogue DHCP Servers

Configured Cisco DHCP Snooping to protect the network from unauthorized DHCP servers by validating DHCP messages and establishing trusted interfaces for legitimate DHCP traffic.

---

## Overview

This lab demonstrates how DHCP Snooping strengthens Layer 2 security by preventing rogue DHCP servers from distributing IP addresses to clients.

The lab configures a router as the legitimate DHCP server, enables DHCP Snooping on multiple switches, establishes trusted uplinks, and verifies that clients successfully obtain IP addresses while unauthorized DHCP responses are blocked.

Topics covered:

- DHCP Server Configuration
- DHCP Snooping
- Trusted vs. Untrusted Interfaces
- DHCP Snooping Binding Table
- DHCP Relay Behavior
- Verification & Troubleshooting

---

## Topology

![Topology](images/topology.png)

---

# Part 1 — Configure the DHCP Server

## Configuration

Configured R1 as the authorized DHCP server.

### DHCP Policy

- Configured interface **G0/0** as the default gateway
- Created a DHCP pool for **192.168.1.0/24**
- Excluded addresses **192.168.1.1 – 192.168.1.9**
- Assigned the default gateway to clients

---

## Configuration Proof

![R1 DHCP Configuration](images/r1-dhcp-config.png)

Verified using:

- `show running-config`
- `show ip dhcp pool`
- `show ip dhcp binding`

---

# Part 2 — Configure DHCP Snooping

## Configuration

Configured DHCP Snooping on both switches.

### Security Policy

- Enabled DHCP Snooping globally
- Enabled DHCP Snooping for VLAN 1
- Configured uplink interfaces as trusted
- Left all client-facing access ports untrusted

---

## Configuration Proof

![DHCP Snooping Configuration](images/dhcp-snooping-config.png)

Verified using:

- `show ip dhcp snooping`
- `show running-config`

---

# Part 3 — DHCP Client Test

## Validation

Renewed the DHCP lease on **PC1** using:

```text
ipconfig /renew
```

### Result

The DHCP request failed.

The client was unable to obtain an IP address because DHCP Snooping was dropping DHCP messages received on an untrusted interface.

---

## Troubleshooting

Identified the missing trust configuration and corrected the appropriate uplink interface.

After applying the fix, renewed the DHCP lease again.

---

## Validation

Verified successful DHCP address assignment.

### Result

- PC1 successfully received an IP address.
- DHCP traffic was forwarded only through trusted interfaces.
- DHCP Snooping continued protecting all untrusted access ports.

---

## Verification

Verified operation using:

- `show ip dhcp snooping`
- `show ip dhcp snooping binding`

![DHCP Snooping Verification](images/dhcp-snooping-verification.png)

---

## Key Takeaways

- DHCP Snooping protects clients from rogue DHCP servers.
- Only trusted interfaces are permitted to forward DHCP server messages.
- Access ports remain untrusted by default.
- The DHCP Snooping binding table records legitimate IP-to-MAC mappings.
- Proper trust boundaries are required for DHCP traffic to function correctly.

---

## Environment

- Cisco Packet Tracer
