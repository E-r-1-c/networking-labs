# Layer 2 Security Hardening: Port Security & DHCP Snooping

## Overview

This lab demonstrates Layer 2 switch hardening using Port Security and DHCP Snooping. Port Security was configured to restrict access based on MAC addresses, while DHCP Snooping was enabled to permit DHCP server messages only from trusted interfaces and build a binding table for legitimate clients.

---

## Objectives

- Configure Port Security on access ports
- Limit the number of learned MAC addresses
- Configure sticky MAC address learning
- Configure Port Security violation actions
- Enable DHCP Snooping for VLAN 10
- Configure trusted and untrusted interfaces
- Verify Port Security operation
- Verify DHCP Snooping bindings
- Validate security violations

---

## Network Topology & Switch Roles

![Network Topology](./images/topology.png)

| Device | Role | Interface | Description |
|--------|------|-----------|-------------|
| R1 | DHCP Server | Gig0/0/0 | Connected to trusted switch interface |
| SW1 | Access Switch | Multiple | Layer 2 security features enabled |
| PC-User | Authorized Client | Fa0/1 | Port Security enabled |
| Rogue-DHCP | Unauthorized Device | Fa0/2 | Used to test DHCP Snooping protection |

---

## Configuration

The switch was configured to improve Layer 2 security using Port Security and DHCP Snooping.

The following configurations were applied:

- Enabled Port Security on access interfaces
- Limited access ports to a single learned MAC address
- Enabled sticky MAC address learning
- Configured Port Security to shut down the interface upon a violation
- Enabled DHCP Snooping globally and for VLAN 10
- Configured the uplink toward the DHCP server as a trusted interface
- Applied DHCP rate limiting to untrusted access ports
- Enabled automatic recovery for Port Security violations

---

## Verification

### Port Security Verification

Verified that Port Security was enabled and operating on the protected access interface.

Command used:

```cisco
show port-security interface FastEthernet0/1
```

![Port Security Verification](./images/01-port-security.png)

The output confirmed:

- Port Security was enabled.
- The interface was operating in a secure state.
- Violation mode was configured for **shutdown**.
- Sticky MAC address learning successfully recorded the connected device.

---

### DHCP Snooping Verification

Verified that DHCP Snooping was tracking legitimate DHCP leases.

Command used:

```cisco
show ip dhcp snooping binding
```

![DHCP Snooping Binding Verification](./images/02-dhcp-snooping-binding.png)

The output confirmed:

- Client IP-to-MAC bindings were successfully learned.
- Lease information, VLAN membership, and interface assignments were recorded in the DHCP Snooping binding table.

---

### Port Security Violation

An unauthorized device was connected to the protected access port to trigger a Port Security violation.

The switch immediately placed the interface into an **err-disabled** state and generated a syslog message indicating the violation.

![Port Security Violation Verification](./images/03-violation-errdisable.png)

The output confirmed:

- An unauthorized MAC address was detected.
- The protected interface entered an **err-disabled** state.
- Traffic from the unauthorized device was blocked until the interface was manually or automatically recovered.

---

## Key Takeaways

- Port Security limits which MAC addresses are permitted on an access port.
- Sticky MAC learning automatically records authorized devices.
- Port Security violations can automatically disable compromised interfaces.
- DHCP Snooping permits DHCP server messages only from trusted interfaces.
- DHCP Snooping maintains a binding table of legitimate DHCP clients.
- Layer 2 security features help protect the access layer from unauthorized devices and rogue DHCP servers.

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
