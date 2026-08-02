# Layer 2 Hardening: Port Security & DHCP Snooping

## Overview

This lab demonstrates Layer 2 security hardening on a Cisco IOS switch. Port Security restricts unauthorized devices on an access port, while DHCP Snooping blocks DHCP server responses received from untrusted interfaces.

---

## Objectives

- Restrict the authorized user port to one MAC address
- Enable sticky MAC address learning
- Configure shutdown violation mode
- Enable DHCP Snooping for VLAN 10
- Trust only the legitimate DHCP server interface
- Test Port Security and rogue DHCP protection

---

## Topology

![Network Topology](./images/topology.png)

- **SW0**: Central Layer 2 switch
- **R0**: Legitimate DHCP server connected to `Fa0/1`
- **PC0**: Authorized host connected to `Fa0/2`
- **Rogue-DHCP**: Rogue DHCP server connected to `Fa0/3`

---

## Configuration

### Port Security

Port Security was configured on `Fa0/2`, which connects to the authorized user.

```cisco
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

The port permits one learned MAC address. If another MAC address is detected, the interface is placed into an err-disabled state.

---

### DHCP Snooping

DHCP Snooping was enabled globally and applied to VLAN 10.

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10
no ip dhcp snooping information option
```

Option 82 insertion was disabled because it caused DHCP issues in Packet Tracer.

The interface connected to the legitimate DHCP server was configured as trusted.

```cisco
interface FastEthernet0/1
 ip dhcp snooping trust
```

The authorized user port and rogue DHCP server port remained untrusted by default.

---

## Verification

### Port Security

```cisco
show port-security interface FastEthernet0/2
```

![Port Security Verification](./images/01-port-security.png)

The output confirmed that Port Security was enabled, the port was limited to one MAC address, sticky learning was active, and shutdown violation mode was configured.

---

### Port Security Violation

An unauthorized device was introduced after the authorized MAC address had already been learned.

![Port Security Violation](./images/02-port-security-violation.png)

The switch detected the unauthorized MAC address and placed `Fa0/2` into an err-disabled state, preventing the device from accessing the network.

---

### DHCP Snooping

```cisco
show ip dhcp snooping
```

![DHCP Snooping Verification](./images/04-dhcp-snooping.png)

The output confirmed that DHCP Snooping was active for VLAN 10, `Fa0/1` was trusted, and the access ports remained untrusted.

---

### DHCP Snooping Binding Table

```cisco
show ip dhcp snooping binding
```

![DHCP Snooping Binding](./images/05-dhcp-binding.png)

The binding table recorded the legitimate client’s MAC address, assigned IP address, lease time, VLAN, and switch interface.

---

### Rogue DHCP Server Test

Packet Tracer simulation mode was used to observe DHCP traffic from the legitimate and rogue DHCP servers.

![Rogue DHCP Drop](./images/06-rogue-dhcp-drop.png)

The legitimate DHCP response entering through trusted interface `Fa0/1` was allowed. The rogue DHCP response entering through untrusted interface `Fa0/3` was dropped.

---

## Key Takeaways

- Port Security limits which devices can use an access port
- Sticky learning automatically records the authorized MAC address
- Shutdown violation mode places the port into an err-disabled state
- DHCP Snooping permits DHCP server responses only through trusted interfaces
- Access ports remain untrusted by default
- DHCP Snooping drops rogue responses without shutting down the rogue server port

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
