# Single-Area OSPFv2

## Overview

This lab shows how OSPF automatically discovers neighboring routers, learns routes to other networks, chooses the lowest-cost path, and updates the route when a link fails.

A four-router topology connects two LANs. R2 provides the preferred path, while R1 provides a higher-cost backup path. R0, R1, and R2 also share an Ethernet network to demonstrate OSPF neighbor formation and DR and BDR election.

---

## Objectives

- Configure single-area OSPFv2
- Assign OSPF router IDs
- Form OSPF neighbor relationships
- Demonstrate DR and BDR election
- Configure passive interfaces
- Use OSPF cost to control path selection
- Advertise a default route through OSPF
- Verify OSPF routes and traffic paths
- Test convergence after a link failure
- Confirm the preferred path returns after recovery

---

## Topology

![Network Topology](./images/topology.png)

R0 connects the Site 1 LAN, and R3 connects the Site 2 LAN.

Between them are two possible paths:

```text
Preferred Path: R0 → R2 → R3
Backup Path:    R0 → R1 → R3
```

R2 provides the preferred path because its link to R3 has the lower OSPF cost.

R1 provides the backup path because its link to R3 is configured with a higher cost.

R0, R1, and R2 connect to the same Ethernet network through SW0. OSPF uses this shared segment to form neighbor relationships and elect a DR and BDR.

R3 also advertises a default route that the other routers can use to reach a simulated external address.

---

## Network Design

### Site Networks

| Network | Subnet | Router Address | End Device |
|---|---|---|---|
| Site 1 LAN | `192.168.10.0/24` | R0: `192.168.10.1` | PC0: `192.168.10.2` |
| Site 2 LAN | `192.168.30.0/24` | R3: `192.168.30.1` | PC1: `192.168.30.2` |
| Simulated External Network | `203.0.113.1/32` | R3 Loopback0: `203.0.113.1` | None |

PC0 uses `192.168.10.1` as its default gateway.

PC1 uses `192.168.30.1` as its default gateway.

### Router Links

| Link | Subnet | Router Addresses | Purpose |
|---|---|---|---|
| Shared OSPF Network | `10.0.0.0/24` | R0: `10.0.0.1`, R1: `10.0.0.2`, R2: `10.0.0.3` | Neighbor formation and DR/BDR election |
| R1–R3 Link | `10.0.1.0/30` | R1: `10.0.1.1`, R3: `10.0.1.2` | Higher-cost backup path |
| R2–R3 Link | `10.0.2.0/30` | R2: `10.0.2.1`, R3: `10.0.2.2` | Lower-cost preferred path |

---

## OSPF Design

All internal networks are placed in OSPF area 0.

| Router | Router ID | Role on Shared Network |
|---|---|---|
| R0 | `0.0.0.1` | DROTHER |
| R1 | `1.1.1.1` | BDR |
| R2 | `2.2.2.2` | DR |
| R3 | `3.3.3.3` | Not connected to the shared network |

R2 uses the highest OSPF priority and becomes the DR.

R1 uses the second-highest priority and becomes the BDR.

R0 uses an OSPF priority of `0`, which prevents it from becoming the DR or BDR.

The R1–R3 link uses an OSPF cost of `10`. The R2–R3 link keeps its lower default cost. This causes OSPF to prefer the path through R2 while keeping the path through R1 available as a backup.

The OSPF `network` statements use each interface address with a wildcard mask of `0.0.0.0`. This matches only the interface using that exact IP address.

---

## Configuration

### R0 OSPF Configuration

R0 connects the Site 1 LAN to the shared OSPF network.

Its LAN-facing interface is passive because it connects to an end device rather than another OSPF router.

R0 also uses an OSPF priority of `0` on the shared network so it cannot become the DR or BDR.

```cisco
interface GigabitEthernet0/0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0/1
 ip address 10.0.0.1 255.255.255.0
 ip ospf priority 0
 no shutdown

router ospf 1
 router-id 0.0.0.1
 passive-interface GigabitEthernet0/0/0
 network 192.168.10.1 0.0.0.0 area 0
 network 10.0.0.1 0.0.0.0 area 0
```

---

### R1 OSPF Configuration

R1 connects the shared OSPF network to R3 and provides the backup path between the two sites.

Its interface toward R3 uses an OSPF cost of `10`, making this path less preferred.

```cisco
interface GigabitEthernet0/0/0
 ip address 10.0.0.2 255.255.255.0
 ip ospf priority 100
 no shutdown

interface GigabitEthernet0/0/1
 ip address 10.0.1.1 255.255.255.252
 ip ospf cost 10
 no shutdown

router ospf 1
 router-id 1.1.1.1
 network 10.0.0.2 0.0.0.0 area 0
 network 10.0.1.1 0.0.0.0 area 0
```

---

### R2 OSPF Configuration

R2 connects the shared OSPF network to R3 and provides the preferred path between the two sites.

R2 uses the highest OSPF priority on the shared network, causing it to become the DR.

Its link to R3 keeps the lower default OSPF cost.

```cisco
interface GigabitEthernet0/0/0
 ip address 10.0.0.3 255.255.255.0
 ip ospf priority 200
 no shutdown

interface GigabitEthernet0/0/1
 ip address 10.0.2.1 255.255.255.252
 no shutdown

router ospf 1
 router-id 2.2.2.2
 network 10.0.0.3 0.0.0.0 area 0
 network 10.0.2.1 0.0.0.0 area 0
```

---

### R3 OSPF Configuration

R3 connects both available OSPF paths to the Site 2 LAN.

Its interface toward R1 also uses an OSPF cost of `10`, making the R1 path less preferred in both directions.

The interface facing PC1 is passive because it does not connect to another OSPF router.

```cisco
interface GigabitEthernet0/0
 ip address 10.0.1.2 255.255.255.252
 ip ospf cost 10
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.2.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface Loopback0
 ip address 203.0.113.1 255.255.255.255
```

R3 uses a static default route to simulate an external connection.

```cisco
ip route 0.0.0.0 0.0.0.0 Null0
```

R3 then advertises the default route into OSPF.

```cisco
router ospf 1
 router-id 3.3.3.3
 passive-interface GigabitEthernet0/2
 network 10.0.1.2 0.0.0.0 area 0
 network 10.0.2.2 0.0.0.0 area 0
 network 192.168.30.1 0.0.0.0 area 0
 default-information originate
```

The `203.0.113.1/32` loopback is not advertised as a specific OSPF route.

The other routers reach it using the default route advertised by R3.

---

## Verification

### OSPF Neighbor Formation

The OSPF neighbors were checked on R0.

```cisco
show ip ospf neighbor
```

![OSPF Neighbor Formation](./images/01-ospf-neighbors.png)

The output confirmed that:

- R0 formed neighbor relationships with R1 and R2
- R2 was the DR
- R1 was the BDR
- Both neighbors reached the FULL state

---

### DR and BDR Election

The shared OSPF interface was checked on R0.

```cisco
show ip ospf interface GigabitEthernet0/0/1
```

![DR and BDR Election](./images/02-dr-bdr-election.png)

The output confirmed that:

- R2 became the DR
- R1 became the BDR
- R0 remained a DROTHER
- OSPF priority controlled the election

---

### OSPF Routing Table

The OSPF routes were checked on R0.

```cisco
show ip route ospf
```

![OSPF Routing Table](./images/03-ospf-routing-table.png)

The output confirmed that R0 learned:

- The Site 2 LAN
- The R1–R3 network
- The R2–R3 network
- A default route advertised by R3

The route to the Site 2 LAN used R2 as the next hop because the path through R2 had the lower total OSPF cost.

---

### Preferred Path Testing

PC0 traced the path to PC1 while all links were operational.

```text
PC0> tracert 192.168.30.2
```

![Primary Path Verification](./images/04-primary-path.png)

The trace confirmed that traffic followed the preferred path:

```text
PC0 → R0 → R2 → R3 → PC1
```

The path through R1 remained available but was not installed as the active route because it had a higher OSPF cost.

---

### Default Route Testing

PC0 traced the path to the simulated external address on R3.

```text
PC0> tracert 203.0.113.1
```

![Default Route Verification](./images/05-default-route.png)

R0 did not have a specific route to `203.0.113.1`, so it forwarded the traffic using the default route advertised by R3.

The trace confirmed that the traffic reached R3 through the preferred R2 path.

---

### Link Failure and OSPF Convergence

The R2–R3 link was disabled to simulate a failure on the preferred path.

```cisco
interface GigabitEthernet0/0/1
 shutdown
```

The OSPF routes were checked again on R0.

```cisco
show ip route ospf
```

![OSPF Convergence](./images/06-ospf-convergence.png)

OSPF removed the unavailable path through R2 and installed the remaining route through R1.

No manual routing changes were required.

---

### Backup Path Testing

PC0 traced the path to PC1 while the R2–R3 link was unavailable.

```text
PC0> tracert 192.168.30.2
```

![Backup Path Verification](./images/07-backup-path.png)

The trace confirmed that traffic followed the backup path:

```text
PC0 → R0 → R1 → R3 → PC1
```

The two LANs remained connected because OSPF recalculated the route and used the remaining path.

---

### Preferred Path Recovery

The R2–R3 link was restored.

```cisco
interface GigabitEthernet0/0/1
 no shutdown
```

After OSPF reconverged, PC0 traced the path to PC1 again.

```text
PC0> tracert 192.168.30.2
```

![Preferred Path Recovery](./images/08-preferred-path-recovery.png)

The trace confirmed that traffic returned to the lower-cost path:

```text
PC0 → R0 → R2 → R3 → PC1
```

This confirmed that OSPF automatically restored the preferred route after the failed link recovered.

---

## Key Takeaways

- OSPF forms neighbor relationships with other OSPF routers
- OSPF exchanges routing information and learns remote networks
- Router IDs uniquely identify routers within the OSPF process
- OSPF priority controls DR and BDR election on shared networks
- A priority of `0` prevents a router from becoming the DR or BDR
- Passive interfaces advertise connected networks without forming neighbors
- OSPF cost determines which available path is preferred
- The lower-cost path through R2 is used during normal operation
- The higher-cost path through R1 remains available as a backup
- OSPF automatically recalculates routes after a link failure
- OSPF returns to the preferred path after the failed link recovers
- `default-information originate` advertises a default route into OSPF
- Routing tables show the selected routes
- Traceroute shows the actual path used by traffic

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- OSPFv2
- Single OSPF area: Area 0
