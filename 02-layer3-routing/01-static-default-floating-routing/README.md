# Single-Area OSPFv2

## Overview

This lab shows how OSPF routers discover neighbors, exchange routes, choose the best path using cost, and automatically use another path when a link fails.

A four-router topology was built between two LANs. R1 provides the preferred path between the sites, while R2 provides a higher-cost backup path. R0, R1, and R2 also share an Ethernet network to demonstrate OSPF neighbor formation and DR and BDR election.

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

R1 provides the preferred path between R0 and R3. R2 provides a higher-cost backup path if the R1–R3 link fails.

R0, R1, and R2 connect to the same Ethernet network through SW0. This shared network is used to demonstrate OSPF neighbor formation and DR and BDR election.

R3 acts as the network edge and advertises a default route to the other routers. A loopback interface on R3 provides a reachable address for testing the default route.

---

## Network Design

### Site Networks

| Network | Subnet | Router Address | End Device |
|---|---|---|---|
| Site 1 LAN | `192.168.10.0/24` | R0: `192.168.10.1` | PC0: `192.168.10.2` |
| Site 2 LAN | `192.168.30.0/24` | R3: `192.168.30.1` | PC1: `192.168.30.2` |
| Simulated External Network | `203.0.113.1/32` | R3 Loopback0: `203.0.113.1` | None |

PC0 uses `192.168.10.1` as its default gateway. PC1 uses `192.168.30.1`.

### Router Links

| Link | Subnet | Router Addresses |
|---|---|---|
| Shared OSPF Network | `10.0.0.0/24` | R0: `10.0.0.1`, R1: `10.0.0.2`, R2: `10.0.0.3` |
| Preferred Link — R1 to R3 | `10.0.1.0/30` | R1: `10.0.1.1`, R3: `10.0.1.2` |
| Backup Link — R2 to R3 | `10.0.2.0/30` | R2: `10.0.2.1`, R3: `10.0.2.2` |

---

## OSPF Design

All internal networks are placed in OSPF area 0.

| Router | Router ID | OSPF Priority | Shared-Network Role |
|---|---|---:|---|
| R0 | `1.1.1.1` | `0` | DROTHER |
| R1 | `2.2.2.2` | `100` | BDR |
| R2 | `3.3.3.3` | `200` | DR |
| R3 | `4.4.4.4` | Not connected | Not on shared network |

R2 has the highest priority and becomes the DR.

R1 has the second-highest priority and becomes the BDR.

R0 uses a priority of `0`, preventing it from becoming the DR or BDR.

The R2–R3 link uses a higher OSPF cost. This keeps the path through R1 preferred while leaving the path through R2 available as a backup.

---

## Configuration

### R0 OSPF Configuration

R0 advertises the Site 1 LAN and the shared OSPF network.

The Site 1 LAN interface is passive because it connects to a PC instead of another OSPF router.

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

R1 participates in the shared OSPF network and provides the preferred path to R3.

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

R2 has the highest priority on the shared network and becomes the DR.

The link from R2 to R3 uses a higher OSPF cost so it acts as the backup path.

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

R3 connects to the preferred and backup paths and advertises the Site 2 LAN.

The Site 2 LAN interface is passive because it connects to a PC instead of another OSPF router.

The R3 interface toward R2 also uses a higher cost so the R1 path is preferred in both directions.

```cisco
interface GigabitEthernet0/0
 ip address 10.0.1.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 ip address 10.0.2.2 255.255.255.252
 ip ospf cost 50
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface Loopback0
 ip address 203.0.113.1 255.255.255.255
```

R3 uses a static default route to simulate an Internet connection.

```cisco
ip route 0.0.0.0 0.0.0.0 Null0
```

R3 then advertises that default route to the other OSPF routers.

```cisco
router ospf 1
 router-id 4.4.4.4
 passive-interface GigabitEthernet0/2
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0
 default-information originate
```

The `203.0.113.1` loopback network is not advertised through OSPF.

The internal routers reach it using the default route advertised by R3.

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
show ip ospf interface GigabitEthernet0/0
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

- The Site 2 LAN through OSPF
- The networks between R1, R2, and R3
- A default route advertised by R3

The route to the Site 2 LAN used the lower-cost path through R1.

---

### Primary Path Testing

PC0 traced the path to PC1 while all links were working.

```text
PC0> tracert 192.168.30.2
```

![Primary Path Verification](./images/04-primary-path.png)

The trace confirmed that traffic used the preferred path:

```text
PC0 → R0 → R1 → R3 → PC1
```

OSPF selected this path because it had a lower total cost than the path through R2.

---

### Default Route Testing

PC0 traced the path to the simulated external address on R3.

```text
PC0> tracert 203.0.113.1
```

![Default Route Verification](./images/05-default-route.png)

The trace confirmed that traffic used the OSPF default route advertised by R3.

R0 did not have a specific route to `203.0.113.1`, so it used the default route.

---

### OSPF Convergence

The preferred R1–R3 link was disabled to simulate a failure.

```cisco
interface GigabitEthernet0/1
 shutdown
```

The OSPF routing table was checked again on R0.

```cisco
show ip route ospf
```

![OSPF Convergence](./images/06-ospf-convergence.png)

The output confirmed that OSPF removed the failed path and installed the available route through R2.

No manual route changes were required.

---

### Backup Path Testing

PC0 traced the path to PC1 while the R1–R3 link was down.

```text
PC0> tracert 192.168.30.2
```

![Backup Path Verification](./images/07-backup-path.png)

The trace confirmed that traffic used the backup path:

```text
PC0 → R0 → R2 → R3 → PC1
```

PC0 and PC1 remained connected even though the preferred link was unavailable.

---

### Preferred Path Recovery

The R1–R3 link was restored.

```cisco
interface GigabitEthernet0/1
 no shutdown
```

PC0 traced the path to PC1 again after OSPF reconverged.

```text
PC0> tracert 192.168.30.2
```

![Preferred Path Recovery](./images/08-preferred-path-recovery.png)

The trace confirmed that traffic returned to the lower-cost path through R1.

---

## Key Takeaways

- OSPF automatically discovers neighboring routers
- OSPF exchanges routes instead of requiring a static route for every network
- Router IDs uniquely identify OSPF routers
- OSPF priority controls DR and BDR election
- The DR and BDR are elected on shared Ethernet networks
- Passive interfaces advertise networks without forming unnecessary neighbors
- OSPF cost determines the preferred path
- A higher-cost path can remain available as a backup
- OSPF automatically changes routes when a link fails
- `default-information originate` advertises a default route
- Routing tables show which OSPF routes are active
- Traceroute shows the path traffic takes

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- OSPFv2
- Single OSPF area: Area 0
