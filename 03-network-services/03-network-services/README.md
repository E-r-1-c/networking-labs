# Network Services

Hands-on Cisco networking labs covering address assignment, time synchronization, monitoring, configuration backups, file transfer, and default gateway redundancy.

This section shows how network services support connected devices, provide centralized management, and keep important network functions available during a failure.

---

## Focus Areas

- DHCP address assignment
- DHCP relay
- Network Time Protocol
- Centralized Syslog
- SNMP monitoring
- Configuration backups
- File transfer
- Default gateway redundancy
- Service verification and failure testing

---

## Labs

### [01 — DHCP Server & Relay](./01-dhcp-server-and-relay/)

Configure a centralized DHCP server to provide IP addresses to clients on local and remote networks.

DHCP relay allows requests to cross a router when the server and client are on different networks.

---

### [02 — NTP, Syslog & SNMP](./02-ntp-syslog-and-snmp/)

Configure centralized services for time synchronization, log collection, and network monitoring.

The lab shows how accurate time supports useful logging and how devices provide operational information to a monitoring server.

---

### [03 — Configuration Backup & File Transfer](./03-configuration-backup-and-file-transfer/)

Back up and restore Cisco device configurations using a centralized server.

The lab demonstrates how configuration files can be transferred between network devices and a server for recovery and management.

---

### [04 — HSRP Default Gateway Redundancy](./04-hsrp-default-gateway-redundancy/)

Configure two routers to provide one shared default gateway for connected devices.

One router handles traffic during normal operation, while the second router takes over if the active gateway fails.

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
