# Static, Default & Floating Routing

## Overview

This lab shows how static routes connect different networks, how default routes handle unknown destinations, and how floating static routes provide a backup path if the main route fails.

A three-router topology was built with a direct path between two LANs and an alternate path through a third router. The alternate path provides backup connectivity if the direct connection fails.

---

## Objectives

- Configure static routes between IPv4 networks
- Configure a default route for unknown destinations
- Configure floating static routes for backup connectivity
- Use administrative distance to choose the preferred route
- Demonstrate longest-prefix match
- Verify routing tables and traffic paths
- Test failover and recovery after a link failure

---

## Topology

![Network Topology](./images/topology.png)

R0 connects the Site 1 LAN, and R1 connects the Site 2 LAN.

The direct link between R0 and R1 is the preferred path. R2 provides an alternate path if the direct link fails.

R2 also has a loopback network used to test longest-prefix match.

---

## Network Design

### Site Networks

| Network | Subnet | Router Address | End Device |
|---|---|---|---|
| Site 1 LAN | `192.168.10.0/24` | R0: `192.168.10.1` | PC0: `192.168.10.10` |
| Site 2 LAN | `192.168.20.0/24` | R1: `192.168.20.1` | PC1: `192.168.20.10` |
| Test Network | `198.51.100.0/24` | R2 Loopback0: `198.51.100.1` | None |

PC0 uses `192.168.10.1` as its default gateway. PC1 uses `192.168.20.1`.

### Router Links

| Link | Subnet | Router Addresses |
|---|---|---|
| Primary Link — R0 to R1 | `10.0.0.0/30` | R0: `10.0.0.1`, R1: `10.0.0.2` |
| Alternate Link — R0 to R2 | `10.0.0.4/30` | R0: `10.0.0.5`, R2: `10.0.0.6` |
| Alternate Link — R2 to R1 | `10.0.0.8/30` | R2: `10.0.0.10`, R1: `10.0.0.9` |

---

## Configuration

### R0 Routing

R0 uses a default route through R1 as the preferred path.

A second default route goes through R2. It uses a higher administrative distance, making it a floating static route. It becomes active only if the preferred route fails.

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.2
ip route 0.0.0.0 0.0.0.0 10.0.0.6 10
```

R0 also has a specific route to the test network on R2.

```cisco
ip route 198.51.100.0 255.255.255.0 10.0.0.6
```

The specific route is chosen instead of the default route because it is the longer match.

---

### R1 Routing

R1 uses the direct link to R0 as the preferred path to the Site 1 LAN.

A floating static route through R2 provides a backup path.

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.10.0 255.255.255.0 10.0.0.10 10
```

R1 also uses R2 to reach the test network.

```cisco
ip route 198.51.100.0 255.255.255.0 10.0.0.10
```

---

### R2 Routing

R2 provides the alternate path between the two LANs.

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.5
ip route 192.168.20.0 255.255.255.0 10.0.0.9
```

These routes allow R2 to forward traffic between the sites if the direct R0–R1 link fails.

The loopback interface creates a separate network for longest-prefix match testing.

```cisco
interface Loopback0
 ip address 198.51.100.1 255.255.255.0
```

---

## Verification

### Interface Status

The router interfaces were checked before testing the routes.

```cisco
show ip interface brief
```

![Interface Status](./images/01-interface-status.png)

The output confirmed that the required interfaces were up and using the correct IP addresses.

---

### R0 Routing Table

The routing table was checked on R0 while all links were working.

```cisco
show ip route
```

![R0 Normal Routing Table](./images/02-r0-normal-routing-table.png)

The output confirmed that:

- The preferred default route through R1 was active
- The floating default route through R2 was inactive
- The specific route to the test network through R2 was active

---

### R1 Routing Table

The routing table was checked on R1.

```cisco
show ip route
```

![R1 Normal Routing Table](./images/03-r1-normal-routing-table.png)

The output confirmed that R1 used the direct link to R0 as the preferred route to the Site 1 LAN.

The floating static route through R2 remained inactive.

---

### Primary Path Testing

PC0 traced the path to PC1 while all links were working.

```text
PC0> tracert 192.168.20.10
```

![Primary Path Verification](./images/04-primary-path.png)

The trace confirmed that traffic used the direct link between R0 and R1.

---

### Longest-Prefix Match Testing

PC0 traced the path to the test network on R2.

```text
PC0> tracert 198.51.100.1
```

![Longest-Prefix Match Verification](./images/05-longest-prefix-match.png)

R0 had two routes that matched the destination:

- The default route `0.0.0.0/0` through R1
- The specific route `198.51.100.0/24` through R2

The `/24` route was selected because it was the longer and more-specific match.

---

### Floating Route Activation

The direct link between R0 and R1 was disabled to simulate a failure.

The routing tables were checked again.

```cisco
show ip route
```

![Floating Route Activation](./images/06-floating-route-activation.png)

The output confirmed that the preferred routes were removed and the floating static routes through R2 became active.

---

### Backup Path Testing

PC0 traced the path to PC1 while the direct R0–R1 link was down.

```text
PC0> tracert 192.168.20.10
```

![Backup Path Verification](./images/07-backup-path.png)

The trace confirmed that traffic used the alternate path through R2.

PC0 and PC1 remained connected even though the preferred path was unavailable.

---

### Primary Path Recovery

The direct link between R0 and R1 was restored.

The routing tables were checked again.

```cisco
show ip route
```

![Primary Path Recovery](./images/08-primary-path-recovery.png)

The output confirmed that:

- The preferred routes returned
- The floating static routes became inactive
- Traffic returned to the direct path between R0 and R1

---

## Key Takeaways

- Static routes manually tell a router how to reach another network
- A default route handles destinations without a more-specific route
- Longest-prefix match chooses the most-specific route
- Administrative distance chooses between routes to the same destination
- A floating static route uses a higher administrative distance to act as a backup
- Floating routes become active when the preferred route fails
- Both forward and return routes are required for communication
- Routing tables show which routes are active
- Traceroute shows the path traffic takes

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
