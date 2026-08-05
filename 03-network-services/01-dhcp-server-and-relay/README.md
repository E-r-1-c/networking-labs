# DHCP Server & Relay

## Overview

This lab shows how a centralized DHCP server automatically assigns IP addresses to clients on different networks and how DHCP relay forwards requests across a router.

A three-LAN topology was built with the DHCP server on one network and two client networks on separate router interfaces. R0 forwards DHCP requests from both client networks to the server, which assigns addresses from the correct DHCP pool.

---

## Objectives

- Configure a centralized DHCP server
- Create separate DHCP pools for different networks
- Configure DHCP relay on router interfaces
- Automatically assign client IP addresses
- Provide the correct subnet mask and default gateway
- Verify DHCP leases on both client networks
- Test connectivity between the clients and server
- Test DHCP failure and recovery when relay is removed

---

## Topology

![Network Topology](./images/topology.png)

Server0 connects to the Server LAN through SW0.

PC0 connects to Client LAN 1 through SW1, and PC1 connects to Client LAN 2 through SW2.

R0 connects all three networks. The DHCP server is not directly connected to either client network, so R0 must relay the DHCP requests to Server0.

---

## Network Design

### Networks

| Network | Subnet | Router Address | Connected Device |
| :--- | :--- | :--- | :--- |
| Server LAN | `192.168.100.0/24` | R0: `192.168.100.1` | Server0: `192.168.100.10` |
| Client LAN 1 | `192.168.10.0/24` | R0: `192.168.10.1` | PC0: DHCP |
| Client LAN 2 | `192.168.20.0/24` | R0: `192.168.20.1` | PC1: DHCP |

Server0 uses `192.168.100.1` as its default gateway.

PC0 and PC1 receive their IP address, subnet mask, and default gateway automatically from Server0.

### DHCP Pools

| Pool | Network | Starting Address | Default Gateway |
| :--- | :--- | :--- | :--- |
| CLIENT-LAN-1 | `192.168.10.0/24` | `192.168.10.10` | `192.168.10.1` |
| CLIENT-LAN-2 | `192.168.20.0/24` | `192.168.20.10` | `192.168.20.1` |

The pools begin at `.10` so the lower addresses remain available for routers and other network devices.

---

## Configuration

### R0 Interface Configuration

R0 connects the server network and both client networks.

```cisco
interface GigabitEthernet0/0/0
 ip address 192.168.100.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0/1
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown

interface GigabitEthernet0/0/2
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
```

The `ip helper-address` command forwards DHCP requests from each client network to Server0.

The Server LAN interface does not need DHCP relay because Server0 is directly connected to that network.

---

### Server0 Addressing

Server0 uses a static address so R0 always knows where to forward DHCP requests.

```text
IP Address:      192.168.100.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.100.1
```

---

### Client LAN 1 DHCP Pool

The first DHCP pool provides addresses to devices on `192.168.10.0/24`.

```text
Pool Name:        CLIENT-LAN-1
Default Gateway:  192.168.10.1
Starting Address: 192.168.10.10
Subnet Mask:      255.255.255.0
```

---

### Client LAN 2 DHCP Pool

The second DHCP pool provides addresses to devices on `192.168.20.0/24`.

```text
Pool Name:        CLIENT-LAN-2
Default Gateway:  192.168.20.1
Starting Address: 192.168.20.10
Subnet Mask:      255.255.255.0
```

When R0 relays a DHCP request, it identifies the network where the request was received. Server0 uses that information to choose the correct DHCP pool.

---

## Verification

### DHCP Server Pools

The DHCP service was checked on Server0.

![DHCP Server Pools](./images/01-dhcp-server-pools.png)

The server contained separate pools for Client LAN 1 and Client LAN 2.

Each pool used the correct network, subnet mask, starting address, and default gateway.

---

### Client LAN 1 Address Assignment

PC0 was configured to obtain its addressing through DHCP.

```text
PC0> ipconfig /all
```

![Client LAN 1 DHCP Lease](./images/02-client-lan1-dhcp-lease.png)

The output confirmed that PC0 received:

- An address from `192.168.10.0/24`
- A `/24` subnet mask
- `192.168.10.1` as its default gateway

---

### Client LAN 2 Address Assignment

PC1 was configured to obtain its addressing through DHCP.

```text
PC1> ipconfig /all
```

![Client LAN 2 DHCP Lease](./images/03-client-lan2-dhcp-lease.png)

The output confirmed that PC1 received:

- An address from `192.168.20.0/24`
- A `/24` subnet mask
- `192.168.20.1` as its default gateway

---

### DHCP Relay Configuration

The R0 configuration was checked to verify that both client interfaces forwarded DHCP requests to Server0.

```cisco
show running-config
```

![DHCP Relay Configuration](./images/04-dhcp-relay-config.png)

The output confirmed that both client-facing interfaces used:

```cisco
ip helper-address 192.168.100.10
```

---

### Connectivity Testing

PC0 tested connectivity to PC1 and Server0.

```text
PC0> ping 192.168.20.10
PC0> ping 192.168.100.10
```

![Connectivity Verification](./images/05-connectivity-verification.png)

The successful replies confirmed that:

- Both clients received usable DHCP addresses
- Each client used the correct default gateway
- R0 routed traffic between the three networks
- Both clients could reach the centralized server

---

### DHCP Relay Failure

The DHCP relay command was temporarily removed from the Client LAN 2 interface.

PC1 released its existing address and attempted to obtain a new lease.

```text
PC1> ipconfig /release
PC1> ipconfig /renew
```

![DHCP Relay Failure](./images/06-dhcp-relay-failure.png)

PC1 could not obtain a new DHCP lease because DHCP broadcasts could not cross R0 without the relay configuration.

---

### DHCP Relay Recovery

The DHCP relay configuration was restored on the Client LAN 2 interface.

PC1 requested an address again.

```text
PC1> ipconfig /renew
```

![DHCP Relay Recovery](./images/07-dhcp-relay-recovery.png)

PC1 received an address from the correct pool after the relay configuration was restored.

---

## Key Takeaways

- DHCP automatically provides IP configuration to clients
- Separate networks require separate DHCP pools
- DHCP clients begin by sending broadcast messages
- Routers do not normally forward DHCP broadcasts
- DHCP relay forwards requests to a server on another network
- `ip helper-address` identifies the destination DHCP server
- The relay identifies which client network the request came from
- The server uses that information to select the correct pool
- Each client receives the correct address and default gateway
- Removing DHCP relay prevents remote clients from obtaining new leases
- Restoring DHCP relay restores automatic address assignment

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
