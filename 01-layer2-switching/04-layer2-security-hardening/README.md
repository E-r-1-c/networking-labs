# Layer 2 Security Hardening: Port Security & DHCP Snooping

## Scenario

A small office network uses a centralized DHCP server to provide IP addresses to client devices.

An unauthorized device has been connected to the same user VLAN and is attempting to provide its own DHCP responses. This creates a security risk because clients could receive incorrect network settings from an untrusted source.

The access switch must be hardened to protect authorized users while preventing unauthorized devices and services from affecting the network.

---

## Network Topology

Build the following topology:

- R0 — Legitimate DHCP Server
- SW0 — Access Switch
- PC-User — Authorized Client
- Rogue-DHCP — Unauthorized DHCP Server

Network design:

- R1 provides valid DHCP services.
- PC-User receives its IP configuration from R1.
- Rogue-DHCP represents an unauthorized DHCP server connected to the same user network.

---

## Objectives

- Build a Layer 2 access network with DHCP services
- Configure access-layer security features
- Restrict unauthorized devices using Port Security
- Protect DHCP services using DHCP Snooping
- Verify legitimate DHCP operation
- Test security controls against unauthorized activity

---

## Tasks

### Part 1 — Build and Verify the Network

- Configure basic device settings.
- Connect all devices according to the topology.
- Configure the client network.
- Verify that the authorized client can successfully obtain an IP address from the legitimate DHCP server.

---

### Part 2 — Configure Port Security

Secure the access port connected to the authorized client.

Requirements:

- Restrict the number of devices allowed on the port.
- Learn the authorized device automatically.
- Define the switch behavior when an unauthorized device is connected.

Verify that the authorized client continues to communicate normally.

---

### Part 3 — Configure DHCP Protection

Protect the DHCP infrastructure from unauthorized DHCP servers.

Requirements:

- Allow DHCP messages from the legitimate DHCP server.
- Prevent DHCP server responses from unauthorized devices.
- Ensure clients can still receive valid IP addresses.

Consider:

- Which interface connects to the legitimate DHCP server?
- Which interfaces should be considered untrusted?
- How does the switch identify legitimate versus unauthorized DHCP traffic?

---

### Part 4 — Verify Security Features

Collect verification evidence showing:

- Port Security status.
- Learned authorized MAC address.
- DHCP client binding information.
- Correct operation of trusted and untrusted interfaces.

---

### Part 5 — Test Port Security

Connect an unauthorized device to the protected access port.

Observe:

- How the switch responds.
- Whether the device is allowed network access.
- What security event occurs.

Capture evidence of the violation.

---

### Part 6 — Test DHCP Snooping

Use Packet Tracer Simulation Mode to observe DHCP traffic.

Verify:

- The legitimate DHCP server can provide IP addresses.
- The rogue DHCP server cannot provide DHCP information to clients.
- Unauthorized DHCP messages are blocked by the switch.

Capture evidence showing the security feature working.

---

## Deliverables

Your completed lab should include:

- Network topology screenshot
- Port Security verification
- DHCP Snooping verification
- Rogue DHCP protection test
- Port Security violation test

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
