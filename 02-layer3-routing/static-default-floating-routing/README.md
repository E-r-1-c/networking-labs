# Static, Default & Floating Routing

## Overview

This lab shows how static routes connect different networks, how default routes handle unknown destinations, and how floating static routes provide a backup path if the main route fails.

A three-router topology was built with a direct path between two LANs. A third router connects both sites to the ISP and provides an alternate path if the direct connection fails.

---

## Objectives

- Configure static routes between IPv4 networks
- Configure default routes toward an ISP
- Configure floating static routes for backup connectivity
- Use administrative distance to choose the preferred route
- Demonstrate longest-prefix match
- Verify routing tables and traffic paths
- Test failover and recovery after a link failure

---

## Topology

![Network Topology](./images/topology.png)

R0 connects the Site 1 LAN, and R1 connects the Site 2 LAN.

The direct link between R0 and R1 is the preferred path between the sites. R2 connects both sites to the ISP and provides an alternate path if the direct link fails.

Cloud0 represents the ISP, while Server0 provides a reachable destination for testing the default routes.

---

## Network Design

### Site and ISP Networks

| Network | Subnet | Router Address | End Device |
|---|---|---|---|
| Site 1 LAN | `192.168.10.0/24` | R0: `192.168.10.1` | PC0: `192.168.10.10` |
| Site 2 LAN | `192.168.20.0/24` | R1: `192.168.20.1` | PC1: `192.168.20.10` |
| ISP Test Network | `203.0.113.0/24` | R2: `203.0.113.1` | Server0: `203.0.113.10` |

PC0 uses `192.168.10.1` as its default gateway. PC1 uses `192.168.20.1`, and Server0 uses `203.0.113.1`.

### Router Links

| Link | Subnet | Router Addresses |
|---|---|---|
| Primary Link — R0 to R1 | `10.0.0.0/30` | R0: `10.0.0.1`, R1: `10.0.0.2` |
| Site 1 Edge Link — R0 to R2 | `10.0.0.4/30` | R0: `10.0.0.5`, R2: `10.0.0.6` |
| Site 2 Edge Link — R1 to R2 | `10.0.0.8/30` | R1: `10.0.0.9`, R2: `10.0.0.10` |

---

## Configuration

### R0 Routing

R0 uses the direct link to R1 as the preferred path to the Site 2 LAN.

A floating static route through R2 provides a backup path. Its higher administrative distance keeps it inactive while the preferred route is available.

```cisco
ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 192.168.20.0 255.255.255.0 10.0.0.6 10
```

R0 uses a default route through R2 for unknown destinations.

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.6
```

The specific route to `192.168.20.0/24` is chosen instead of the default route because it is the longer and more-specific match.

---

### R1 Routing

R1 uses the direct link to R0 as the preferred path to the Site 1 LAN.

A floating static route through R2 provides a backup path.

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.10.0 255.255.255.0 10.0.0.10 10
```

R1 also uses a default route through R2 for unknown destinations.

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.10
```

---

### R2 Routing

R2 connects both sites to the ISP and provides the alternate path between them.

Static routes provide return paths to both site networks.

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.5
ip route 192.168.20.0 255.255.255.0 10.0.0.9
```

These routes allow R2 to return ISP traffic to each site and forward traffic between the sites if the direct R0–R1 link fails.

---

## Verification

### R0 Routing Table

The routing table was checked on R0 while all links were working.

```cisco
show ip route
```

![R0 Routing Table](./images/01-r0-routing-table.png)

The output confirmed that:

- The preferred route to Site 2 through R1 was active
- The floating route to Site 2 through R2 was inactive
- The default route through R2 was active

---

### R1 Routing Table

The routing table was checked on R1.

```cisco
show ip route
```

![R1 Routing Table](./images/02-r1-routing-table.png)

The output confirmed that:

- The preferred route to Site 1 through R0 was active
- The floating route to Site 1 through R2 was inactive
- The default route through R2 was active

---

### Primary Path and Longest-Prefix Match

PC0 traced the path to PC1 while all links were working.

```text
PC0> tracert 192.168.20.10
```

![Primary Path Verification](./images/03-primary-path.png)

The trace confirmed that traffic used the direct link between R0 and R1.

R0 had a specific route to `192.168.20.0/24` and a default route through R2. The specific route was selected because it was the longer and more-specific match.

---

### Default Route Testing

PC0 traced the path to Server0 on the ISP test network.

```text
PC0> tracert 203.0.113.10
```

![Default Route Verification](./images/04-default-route.png)

The trace confirmed that traffic to Server0 followed the default route from R0 to R2.

PC1 also reached Server0 through its default route to R2.

---

### Floating Route Activation

The direct link between R0 and R1 was disabled to simulate a failure.

The routing table was checked again on R0.

```cisco
show ip route
```

![R0 Floating Route](./images/05-r0-floating-route.png)

The output confirmed that the preferred route to Site 2 was removed and the floating static route through R2 became active.

The routing table was also checked on R1.

```cisco
show ip route
```

![R1 Floating Route](./images/06-r1-floating-route.png)

The output confirmed that the preferred route to Site 1 was removed and the floating static route through R2 became active.

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
- The default routes through R2 remained active

---

## Key Takeaways

- Static routes manually tell a router how to reach another network
- A default route handles destinations without a more-specific route
- Longest-prefix match chooses the most-specific route
- Administrative distance chooses between routes to the same destination
- A floating static route uses a higher administrative distance to act as a backup
- Floating routes become active when the preferred route fails
- R2 connects both sites to the ISP and provides an alternate path between them
- Both forward and return routes are required for communication
- Routing tables show which routes are active
- Traceroute shows the path traffic takes

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
