# Access-Layer Security Hardening

## Overview

This lab will combine Port Security, DHCP Snooping, Dynamic ARP Inspection, and IP Source Guard to protect access-layer switch ports from several common local network attacks.

This lab builds on the earlier Port Security and DHCP Snooping labs by combining those protections with DAI and IP Source Guard in one access-layer security design.

---

## Objectives

- Configure Port Security on user-facing ports
- Configure DHCP Snooping and trusted interfaces
- Build and verify DHCP Snooping bindings
- Configure Dynamic ARP Inspection
- Configure IP Source Guard
- Verify legitimate client traffic
- Test unauthorized devices and rogue DHCP traffic
- Test invalid ARP traffic
- Test source-address spoofing
- Verify which security feature blocks each attack

---

## Security Features

| Feature | Purpose |
|---|---|
| Port Security | Restricts which MAC addresses can use an access port |
| DHCP Snooping | Blocks unauthorized DHCP server messages and builds trusted bindings |
| Dynamic ARP Inspection | Validates ARP traffic using trusted address information |
| IP Source Guard | Restricts source IP and MAC information on access ports |

DHCP Snooping is especially important because its binding table can be used by DAI and IP Source Guard for validation.

---

## Planned Topology

The topology will include:

- Multiple switches
- Legitimate client devices
- A legitimate DHCP server
- One or more devices used to test unauthorized traffic

User-facing ports will remain untrusted, while only the required infrastructure paths will be trusted.

---

## Planned Verification

The lab will verify each security feature using the appropriate IOS commands.

```cisco
show port-security interface <INTERFACE>
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip verify source
```

Verification will focus on confirming that the protections are active, valid client information is learned correctly, and legitimate traffic continues to work.

---

## Planned Security Testing

### Port Security

Introduce an unauthorized MAC address on a protected access port and verify the configured violation behavior.

### Rogue DHCP

Attempt to send DHCP server responses through an untrusted interface and verify that DHCP Snooping blocks them.

### ARP Spoofing

Generate invalid ARP information and verify that Dynamic ARP Inspection rejects it.

### Source Spoofing

Attempt to send traffic using source IP or MAC information that does not match the valid binding and verify that IP Source Guard blocks it.

### Final Connectivity

After the security tests, verify that authorized clients can still obtain valid addresses and communicate normally.

---

## Key Concepts to Prove

- Port Security controls which MAC addresses can use an access port
- DHCP Snooping separates trusted DHCP infrastructure from untrusted client ports
- DHCP Snooping builds trusted IP-to-MAC bindings
- DAI uses trusted information to validate ARP traffic
- IP Source Guard uses trusted bindings to validate source traffic
- Each control protects against a different problem
- One security feature does not replace the others
- Trusted interfaces must be chosen carefully
- Security controls should block unauthorized traffic without breaking legitimate traffic

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
