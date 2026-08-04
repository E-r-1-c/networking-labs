# Single-Area OSPFv2

## Overview

This lab shows how OSPF dynamically discovers neighboring routers, exchanges IPv4 routes, selects the best path, and updates the routing table when a link fails.

Four routers were configured in OSPF area 0. Three routers share an Ethernet segment for DR and BDR election, while two separate links provide a preferred path and a higher-cost backup path to the remote network.

R3 also advertises a default route so the internal routers can reach a simulated external destination without maintaining a specific route to it.

---

## Objectives

- Configure single-area OSPFv2
- Assign explicit OSPF router IDs
- Form OSPF neighbor relationships
- Demonstrate DR and BDR election
- Configure passive interfaces
- Control path selection using OSPF cost
- Advertise a default route through OSPF
- Verify OSPF-learned routes
- Test convergence after a link failure
- Confirm the preferred path returns after recovery

---

## Topology

![Network Topology](./images/topology.png)

R0, R1, and R2 connect to the same Ethernet segment through SW0.

This shared segment allows the routers to form OSPF neighbor relationships and elect a Designated Router and Backup Designated Router.

R1 and R2 each connect to R3. The R1–R3 link is the preferred path, while the R2–R3 link provides a higher-cost backup path.

R0 connects the Site 1 LAN. R3 connects the Site 2 LAN and represents the network edge.

---

## Network Design

### LAN Networks

| Network | Subnet | Router Address | End Device |
|---|---|---|---|
| Site 1 LAN | `192.168.10.0/24` | R0: `192.168.10.1` | PC0: `192.168.10.2` |
| Site 2 LAN | `192.168.30.0/24` | R3: `192.168.30.1` | PC1: `192.168.30.2` |
| Simulated External Destination | `203.0.113.1/32` | R3 Loopback0: `203.0.113.1` | None |

PC0 uses `192.168.10.1` as its default gateway.

PC1 uses `192.168.30.1` as its default gateway.

### Router Links

| Link | Subnet | Router Addresses |
|---|---|---|
| Shared OSPF Segment | `10.0.0.0/24` | R0: `10.0.0.1`, R1: `10.0.0.2`, R2: `10.0.0.3` |
| Preferred R1–R3 Link | `10.0.1.0/30` | R1: `10.0.1.1`, R3: `10.0.1.2` |
| Backup R2–R3 Link | `10.0.2.0/30` | R2: `10.0.2.1`, R3: `10.0.2.2` |

---

## OSPF Design

All internal networks use OSPF area 0.

| Router | Router ID | Shared-Segment Priority | Intended Role |
|---|---|---:|---|
| R0 | `1.1.1.1` | `0` | DROTHER |
| R1 | `2.2.2.2` | `100` | BDR |
| R2 | `3.3.3.3` | `200` | DR |
| R3 | `4.4.4.4` | Not connected | Internal Router |

R0 uses an OSPF priority of `0`, preventing it from becoming the DR or BDR.

R2 has the highest priority and becomes the DR. R1 has the second-highest priority and becomes the BDR.

The R2–R3 interface uses a higher OSPF cost, making the path through R1 preferred during normal operation.

---

## Configuration

### R0 OSPF Configuration

R0 advertises the Site 1 LAN and the shared OSPF segment.

The LAN interface is passive because no OSPF neighbor should form with the connected PC.

```cisco
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.0
 ip ospf priority 0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown

router ospf 1
 router-id 1.1.1.1
 passive-interface GigabitEthernet0/1
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
```

---

### R1 OSPF Configuration

R1 participates in the shared Ethernet segment and connects to R3 through the preferred path.

```cisco
interface GigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.0
 ip ospf priority 100
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.1.1 255.255.255.252
 no shutdown

router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.255 area 0
 network 10.0.1.0 0.0.0.3 area 0
```

---

### R2 OSPF Configuration

R2 has the highest OSPF priority on the shared network and becomes the DR.

A higher OSPF cost is configured on the R2–R3 link so this path remains available as a backup.

```cisco
interface GigabitEthernet0/0
 ip address 10.0.0.3 255.255.255.0
 ip ospf priority 200
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.2.1 255.255.255.252
 ip ospf cost 50
 no shutdown

router ospf 1
 router-id 3.3.3.3
 network 10.0.0.0 0.0.0.255 area 0
 network 10.0.2.0 0.0.0.3 area 0
```

---

### R3 OSPF Configuration

R3 connects to both available paths and advertises the Site 2 LAN.

The Site 2 LAN interface is passive because it connects to an end device instead of another OSPF router.

```cisco
interface GigabitEthernet0/0
 ip address 10.0.1.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.2.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface Loopback0
 ip address 203.0.113.1 255.255.255.255

ip route 0.0.0.0 0.0.0.0 Null0

router ospf 1
 router-id 4.4.4.4
 passive-interface GigabitEthernet0/2
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0
 default-information originate
```

The Loopback0 network is not advertised directly through OSPF.

Internal routers reach `203.0.113.1` through the OSPF default route advertised by R3.

---

## Verification

### OSPF Neighbor Formation

The OSPF neighbors were checked from R0.

```cisco
show ip ospf neighbor
```

![OSPF Neighbors](./images/01-ospf-neighbors.png)

The output confirmed that:

- R0 formed neighbor relationships with R1 and R2
- R2 was identified as the DR
- R1 was identified as the BDR
- Both neighbors reached the FULL state

---

### DR and BDR Election

The OSPF interface information was checked on the shared Ethernet segment.

```cisco
show ip ospf interface GigabitEthernet0/0
```

![DR and BDR Election](./images/02-dr-bdr-election.png)

The output confirmed that:

- R2 became the DR
- R1 became the BDR
- R0 remained a DROTHER
- The configured OSPF priorities controlled the election

---

### OSPF Routing Table

The OSPF-learned routes were checked on R0.

```cisco
show ip route ospf
```

![OSPF Routing Table](./images/03-ospf-routing-table.png)

The output confirmed that R0 learned:

- The Site 2 LAN through OSPF
- The router links behind R1 and R2
- An external default route advertised by R3

The preferred route to the Site 2 LAN used the lower-cost path through R1.

---

### Primary Path Testing

PC0 traced the path to PC1 during normal operation.

```text
PC0> tracert 192.168.30.2
```

![Primary Path](./images/04-primary-path.png)

The trace confirmed that traffic followed the preferred path:

```text
PC0 → R0 → R1 → R3 → PC1
```

The path through R1 was selected because it had a lower total OSPF cost than the path through R2.

---

### Default Route Advertisement

PC0 traced the path to the simulated external destination on R3.

```text
PC0> tracert 203.0.113.1
```

![Default Route](./images/05-default-route.png)

The trace confirmed that R0 used the OSPF-learned default route to reach the external destination through R3.

The default route appears in the routing table as an OSPF external route.

---

### Link Failure and Convergence

The preferred R1–R3 link was disabled to simulate a failure.

```cisco
interface GigabitEthernet0/1
 shutdown
```

The routing table was checked again on R0.

```cisco
show ip route ospf
```

![OSPF Convergence](./images/06-ospf-convergence.png)

The output confirmed that OSPF removed the failed path and installed the available route through R2.

No manual routing changes were required.

---

### Backup Path Testing

PC0 traced the path to PC1 while the R1–R3 link was down.

```text
PC0> tracert 192.168.30.2
```

![Backup Path](./images/07-backup-path.png)

The trace confirmed that traffic followed the backup path:

```text
PC0 → R0 → R2 → R3 → PC1
```

Connectivity continued through R2 despite the preferred-link failure.

---

### Preferred Path Recovery

The R1–R3 link was restored.

```cisco
interface GigabitEthernet0/1
 no shutdown
```

PC0 traced the path to PC1 again after the OSPF routes reconverged.

```text
PC0> tracert 192.168.30.2
```

![Preferred Path Recovery](./images/08-preferred-path-recovery.png)

The trace confirmed that traffic returned to the lower-cost path through R1.

---

## Screenshot Files

```text
images/
├── topology.png
├── 01-ospf-neighbors.png
├── 02-dr-bdr-election.png
├── 03-ospf-routing-table.png
├── 04-primary-path.png
├── 05-default-route.png
├── 06-ospf-convergence.png
├── 07-backup-path.png
└── 08-preferred-path-recovery.png
```

---

## Key Takeaways

- OSPF automatically discovers neighbors and exchanges routing information
- Router IDs uniquely identify OSPF routers
- OSPF neighbors must reach the FULL state before complete route information is exchanged
- Broadcast networks use a DR and BDR to reduce unnecessary OSPF adjacencies
- OSPF priority can control DR and BDR election
- Passive interfaces advertise networks without sending OSPF hello messages
- OSPF cost determines the preferred path
- A higher-cost path can remain available as a backup
- OSPF automatically updates the routing table after a link failure
- `default-information originate` advertises an existing default route
- Routing tables show the selected route
- Traceroute shows the actual path taken by traffic

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- Single OSPF area: Area 0
