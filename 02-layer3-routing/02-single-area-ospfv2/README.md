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

Two paths are available between the sites:

```text
Preferred Path: R0 → R2 → R3
Backup Path:    R0 → R1 → R3
```

R2 provides the preferred path because its link to R3 has the lower OSPF cost. R1 provides the backup path if the R2–R3 link fails.

R0, R1, and R2 connect to the same Ethernet network through SW0. This shared network is used for OSPF neighbor formation and DR and BDR election.

R3 also acts as the network edge and advertises a default route to the other routers.

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

| Router | Router ID | Shared-Network Role |
|---|---|---|
| R0 | `0.0.0.1` | DROTHER |
| R1 | `1.1.1.1` | BDR |
| R2 | `2.2.2.2` | DR |
| R3 | `3.3.3.3` | Not connected to the shared network |

R2 uses the highest OSPF priority and becomes the DR.

R1 uses the second-highest priority and becomes the BDR.

R0 uses an OSPF priority of `0`, preventing it from becoming the DR or BDR.

The R1–R3 link uses an OSPF cost of `10`. The R2–R3 link keeps its lower default cost, making R2 the preferred path and R1 the backup path.

The OSPF `network` statements use each interface’s exact IP address with a wildcard mask of `0.0.0.0`. This enables OSPF only on the interface using that address.

---

## Configuration

Each router was configured by first assigning interface addresses, then setting any required OSPF interface behavior, and finally enabling OSPF and advertising the connected networks.

---

### R0 Configuration

R0 provides the gateway for the Site 1 LAN and connects it to the shared OSPF network.

The Site 1 interface was configured first.

```cisco
interface GigabitEthernet0/0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

The shared-network interface was configured with an OSPF priority of `0`, preventing R0 from becoming the DR or BDR.

```cisco
interface GigabitEthernet0/0/1
 ip address 10.0.0.1 255.255.255.0
 ip ospf priority 0
 no shutdown
```

OSPF process 1 was then configured with router ID `0.0.0.1`.

The Site 1 interface was made passive because it connects to PC0 rather than another OSPF router. The network is still advertised, but OSPF Hello packets are not sent onto the LAN.

```cisco
router ospf 1
 router-id 0.0.0.1
 passive-interface GigabitEthernet0/0/0
 network 192.168.10.1 0.0.0.0 area 0
 network 10.0.0.1 0.0.0.0 area 0
```

---

### R1 Configuration

R1 connects the shared OSPF network to R3 and provides the backup path.

The shared-network interface uses an OSPF priority of `100`, allowing R1 to become the BDR.

```cisco
interface GigabitEthernet0/0/0
 ip address 10.0.0.2 255.255.255.0
 ip ospf priority 100
 no shutdown
```

The link toward R3 was configured with an OSPF cost of `10`, making it less preferred than the path through R2.

```cisco
interface GigabitEthernet0/0/1
 ip address 10.0.1.1 255.255.255.252
 ip ospf cost 10
 no shutdown
```

OSPF process 1 was configured with router ID `1.1.1.1`, and both connected links were added to area 0.

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.2 0.0.0.0 area 0
 network 10.0.1.1 0.0.0.0 area 0
```

---

### R2 Configuration

R2 connects the shared OSPF network to R3 and provides the preferred path.

The shared-network interface uses an OSPF priority of `200`, causing R2 to become the DR.

```cisco
interface GigabitEthernet0/0/0
 ip address 10.0.0.3 255.255.255.0
 ip ospf priority 200
 no shutdown
```

The link toward R3 keeps its lower default OSPF cost.

```cisco
interface GigabitEthernet0/0/1
 ip address 10.0.2.1 255.255.255.252
 no shutdown
```

OSPF process 1 was configured with router ID `2.2.2.2`, and both connected links were added to area 0.

```cisco
router ospf 1
 router-id 2.2.2.2
 network 10.0.0.3 0.0.0.0 area 0
 network 10.0.2.1 0.0.0.0 area 0
```

Because this path has the lower total OSPF cost, it is selected during normal operation.

---

### R3 Configuration

R3 connects both available paths to the Site 2 LAN and advertises the default route.

The link toward R1 was configured with an OSPF cost of `10`, keeping the R1 path less preferred in both directions.

```cisco
interface GigabitEthernet0/0
 ip address 10.0.1.2 255.255.255.252
 ip ospf cost 10
 no shutdown
```

The link toward R2 keeps its lower default cost.

```cisco
interface GigabitEthernet0/1
 ip address 10.0.2.2 255.255.255.252
 no shutdown
```

The Site 2 interface provides the default gateway for PC1.

```cisco
interface GigabitEthernet0/2
 ip address 192.168.30.1 255.255.255.0
 no shutdown
```

A loopback interface provides an address used to test the default route.

```cisco
interface Loopback0
 ip address 203.0.113.1 255.255.255.255
```

R3 was given a static default route pointing to `Null0`.

```cisco
ip route 0.0.0.0 0.0.0.0 Null0
```

OSPF process 1 was configured with router ID `3.3.3.3`.

The Site 2 interface was made passive, and the default route was advertised with `default-information originate`.

```cisco
router ospf 1
 router-id 3.3.3.3
 passive-interface GigabitEthernet0/2
 network 10.0.1.2 0.0.0.0 area 0
 network 10.0.2.2 0.0.0.0 area 0
 network 192.168.30.1 0.0.0.0 area 0
 default-information originate
```

The `203.0.113.1/32` loopback is not advertised as a specific OSPF route. The other routers reach it using the default route advertised by R3.

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

The route to the Site 2 LAN used R2 as the next hop because the path through R2 had the lower OSPF cost.

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

---

### Default Route Testing

PC0 traced the path to the simulated external address on R3.

```text
PC0> tracert 203.0.113.1
```

![Default Route Verification](./images/05-default-route.png)

R0 did not have a specific route to `203.0.113.1`, so it used the default route advertised by R3.

---

### OSPF Convergence

The preferred R2–R3 link was disabled to simulate a failure.

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

No manual route changes were required.

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

---

## Key Takeaways

- OSPF forms neighbor relationships with other OSPF routers
- OSPF exchanges routes and learns remote networks
- Router IDs identify routers within the OSPF process
- OSPF priority controls DR and BDR election
- A priority of `0` prevents a router from becoming the DR or BDR
- Passive interfaces advertise networks without forming neighbors
- OSPF cost determines the preferred path
- The lower-cost path through R2 is used during normal operation
- The higher-cost path through R1 remains available as a backup
- OSPF automatically recalculates routes after a link failure
- OSPF returns to the preferred path after the failed link recovers
- `default-information originate` advertises a default route into OSPF
- Routing tables show the selected routes
- Traceroute shows the path used by traffic

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- OSPFv2
- Single OSPF area: Area 0
