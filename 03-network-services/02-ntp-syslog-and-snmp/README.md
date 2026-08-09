# NTP, Syslog & SNMP

## Overview

This lab shows how NTP keeps device clocks synchronized, how Syslog sends network events to a central server, and how SNMP allows devices to be monitored and managed remotely.

The topology uses a router, two switches, and an internal management server. VLAN 20 is used for management, while VLAN 10 contains the user device.

---

## Objectives

- Configure a dedicated management VLAN
- Configure management IP addresses on Layer 2 switches
- Provide inter-VLAN routing using Router-on-a-Stick
- Configure centralized NTP time synchronization
- Configure Syslog forwarding to a management server
- Configure SNMP access on network devices
- Verify NTP synchronization
- Generate and verify centralized Syslog events
- Verify SNMP communication with network devices
- Test NTP failure and recovery

---

## Topology

![Network Topology](./images/topology.png)

VLAN 10 contains the end-user network, while VLAN 20 is used for network management. The management server and both switch management interfaces use VLAN 20, and R0 provides the default gateway for both VLANs through Router-on-a-Stick.

---

## Network Design

| VLAN | Name | Network | Gateway | Purpose |
|---|---|---|---|---|
| 10 | Users | `192.168.10.0/24` | `192.168.10.1` | End-user network |
| 20 | MANAGEMENT | `192.168.20.0/24` | `192.168.20.1` | Network management |

### Device Addressing

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| R0 | VLAN 10 Subinterface | `192.168.10.1/24` | VLAN 10 Default Gateway |
| R0 | VLAN 20 Subinterface | `192.168.20.1/24` | Management Default Gateway |
| Switch 0 | VLAN 20 SVI | `192.168.20.11/24` | Switch Management |
| Switch 1 | VLAN 20 SVI | `192.168.20.12/24` | Switch Management |
| Management Server | NIC | `192.168.20.10/24` | NTP, Syslog, and SNMP |
| PC | NIC | `192.168.10.10/24` | End-User Device |

The switches use VLAN 20 SVIs for IP-based management while continuing to operate as Layer 2 switches for normal traffic.

---

## Management Services

| Service | Purpose |
|---|---|
| NTP | Keeps network device clocks synchronized |
| Syslog | Sends device events and system messages to a central server |
| SNMP | Allows a management system to retrieve and modify device information |

Together, these services provide centralized time, logging, and network management.

---

## Baseline Connectivity

Before configuring NTP, Syslog, or SNMP, basic connectivity to the management server was verified.

The switches were given management IPs through VLAN 20 SVIs, while R0 used its VLAN 20 subinterface as the management gateway.

R0, both switches, and the VLAN 10 client successfully reached `192.168.20.10`.

![Baseline Connectivity](./images/01-baseline-connectivity.png)

This confirmed that the management network and inter-VLAN routing were working before the services were configured.

---

## NTP Configuration

The router and both switches were configured to use the management server as their NTP source.

```cisco
ntp server 192.168.20.10
```

Using the same time source keeps device clocks synchronized and provides consistent timestamps across the network.

---

## Syslog Configuration

The router and both switches were configured to send Syslog messages to the management server.

```cisco
logging host 192.168.20.10
service timestamps log datetime msec
```

The logging host command sends Syslog messages to the management server, while the timestamp command adds the device time to generated log messages.

Because the devices use the same NTP source, their Syslog timestamps can be compared more easily across the network.

---

## SNMP Configuration

SNMP was configured on the router and switches using a shared community string.

```cisco
snmp-server community cisco rw
```

The community string acts as a shared credential between the SNMP manager and the managed device.

The `rw` option gives the SNMP manager read-write access. This allows it to retrieve device information and also permits supported values to be changed through SNMP.

In this lab, the same community string was configured on R0, Switch 0, and Switch 1 so the management server could communicate with each device.

---

# Verification

## Management Address Verification

The switch management interfaces were verified before testing the management services.

```cisco
show ip interface brief
```

![Management Addresses](./images/02-management-addresses.png)

The output confirmed that the VLAN 20 management interfaces were configured and operational.

---

## NTP Verification

NTP was checked after the router and switches were configured to use `192.168.20.10`.

```cisco
show ntp associations
show ntp status
```

![NTP Verification](./images/03-ntp-verification.png)

The output confirmed that the devices formed an association with the NTP server and synchronized their clocks.

---

## Syslog Verification

The remote Syslog destination was verified on the network devices.

```cisco
show logging
```

![Syslog Configuration](./images/04-syslog-config.png)

The output confirmed that Syslog messages were being sent to `192.168.20.10` over UDP port 514.

---

## Syslog Event Testing

A network event was generated by temporarily changing the state of an interface.

```cisco
interface <INTERFACE>
 shutdown
 no shutdown
```

The Syslog service on the management server was then checked.

![Syslog Event](./images/05-syslog-event.png)

The server received the interface state-change messages from the network devices, confirming that centralized Syslog forwarding was working.

The messages also included timestamps generated from the synchronized device clocks.

---

## SNMP Verification

SNMP communication was tested between the management server and the network devices.

![SNMP Verification](./images/06-snmp-verification.png)

The management server successfully communicated with the configured devices using the shared SNMP community string.

This confirmed that SNMP was operational and that device information could be accessed remotely.

---

## Key Takeaways

- Layer 2 switches require management IP addresses to use IP-based management services
- An SVI provides a Layer 2 switch with an IP interface for management
- A dedicated management VLAN separates management addressing from the user network
- Router-on-a-Stick provides routing between the user and management VLANs
- NTP keeps device clocks synchronized
- Synchronized clocks provide consistent timestamps across network devices
- Syslog centralizes device events and system messages
- Syslog messages can include timestamps from the device clock
- SNMP allows a management system to access device and interface information remotely
- SNMP community strings control access between the manager and managed devices
- Read-write SNMP allows both monitoring and supported remote changes
- NTP, Syslog, and SNMP work together as part of centralized network management
- Management services depend on working IP connectivity to the management server
- Verification should prove that each service is operating, not just configured

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- VLANs
- Router-on-a-Stick
- Switch Virtual Interfaces
- NTP
- Syslog
- SNMP
