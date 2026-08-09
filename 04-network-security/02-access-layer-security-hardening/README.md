# Access-Layer Security Hardening

## Overview

This lab will combine Port Security, DHCP Snooping, Dynamic ARP Inspection, and IP Source Guard to protect access-layer switch ports from common local network attacks.

It builds on the earlier Port Security and DHCP Snooping labs by using multiple protections together in one network instead of testing each feature by itself.

---

## Objectives

- Protect user-facing switch ports
- Block unauthorized DHCP activity
- Validate ARP traffic
- Restrict invalid source addressing
- Verify legitimate clients still work normally
- Test how each protection responds to different attacks

---

## Security Features

| Feature | Purpose |
|---|---|
| Port Security | Restricts which devices can use an access port |
| DHCP Snooping | Blocks unauthorized DHCP responses and learns valid client information |
| Dynamic ARP Inspection | Checks ARP traffic for invalid address information |
| IP Source Guard | Restricts traffic that uses invalid source addressing |

These protections cover different problems and are designed to work together at the access layer.

---

## Planned Topology

The lab will include:

- Multiple switches
- Legitimate client devices
- A legitimate DHCP server
- Devices used to test unauthorized traffic

The network will be designed so normal client traffic works before any attacks are introduced.

---

## Planned Testing

The lab will test:

- Unauthorized device access
- Rogue DHCP responses
- Invalid ARP information
- Source-address spoofing
- Normal client connectivity after the protections are enabled

Each test will identify which security feature is responsible for allowing or blocking the traffic.

---

## Planned Verification

Verification will confirm that:

- The security features are active
- Legitimate clients are learned correctly
- Unauthorized traffic is blocked
- Normal network traffic continues to work
- The different protections are performing separate roles

---

## Key Concepts to Prove

- Each security feature protects against a different problem
- Port Security and DHCP Snooping alone do not provide complete access-layer protection
- DHCP Snooping provides information that other protections can use
- Security controls should block invalid traffic without breaking legitimate users
- Multiple access-layer protections can work together in the same network

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- VLANs
- Port Security
- DHCP Snooping
- Dynamic ARP Inspection
- IP Source Guard
