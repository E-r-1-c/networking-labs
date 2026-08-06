# ACL Traffic Filtering

## Overview

This lab demonstrates how standard and extended IPv4 access control lists control traffic between multiple client networks and a protected server network.

The topology contains HR, IT, Guest, and Server networks. Router-on-a-Stick provides routing for the client VLANs, while a separate router interface connects the server network.

A standard ACL prevents the Guest network from reaching the HR network without blocking Guest access to other destinations. An extended ACL protects the server by permitting only approved application traffic from each client network.

The lab also demonstrates ACL placement, interface direction, wildcard masks, top-down processing, first-match behavior, implicit deny behavior, match counters, configuration mistakes, and service recovery.

---

## Network Design

### VLAN and Addressing Plan

| VLAN | Name | Network | Gateway | Endpoint |
|---|---|---|---|---|
| 10 | HR | `192.168.10.0/24` | `192.168.10.1` | PC-HR: `192.168.10.10` |
| 20 | IT | `192.168.20.0/24` | `192.168.20.1` | PC-IT: `192.168.20.10` |
| 30 | Guest | `192.168.30.0/24` | `192.168.30.1` | PC-GUEST: `192.168.30.10` |
| 40 | Server | `192.168.40.0/24` | `192.168.40.1` | SERVER: `192.168.40.10` |
| 999 | Native | N/A | N/A | Unused native VLAN |

### Device Addressing

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| R1 | `G0/0.10` | `192.168.10.1/24` | HR gateway |
| R1 | `G0/0.20` | `192.168.20.1/24` | IT gateway |
| R1 | `G0/0.30` | `192.168.30.1/24` | Guest gateway |
| R1 | `G0/1` | `192.168.40.1/24` | Server gateway |
| PC-HR | NIC | `192.168.10.10/24` | HR client |
| PC-IT | NIC | `192.168.20.10/24` | IT client |
| PC-GUEST | NIC | `192.168.30.10/24` | Guest client |
| SERVER | NIC | `192.168.40.10/24` | Protected server |

---

## Security Requirements

### Standard ACL Requirement

The Guest network must not be allowed to communicate with the HR network.

Required behavior:

| Source | Destination | Expected Result |
|---|---|---|
| Guest | HR | Deny |
| Guest | IT | Permit |
| Guest | Server | Permit, subject to the server ACL |
| HR | IT | Permit |
| IT | HR | Permit |

Because a standard ACL matches only the source address, it must be placed close to the HR destination.

### Extended ACL Requirement

The server must allow only approved traffic from each client network.

| Source | HTTP | HTTPS | FTP | ICMP |
|---|---:|---:|---:|---:|
| HR | Permit | Permit | Deny | Deny |
| IT | Permit | Permit | Permit | Deny |
| Guest | Permit | Deny | Deny | Deny |

Traffic between the client networks must remain available unless restricted by the standard ACL.

---

## Base Network Configuration

### SW1 VLAN Configuration

SW1 was configured with VLANs for HR, IT, Guest, and the unused native VLAN.

```cisco
vlan 10
 name HR

vlan 20
 name IT

vlan 30
 name GUEST

vlan 999
 name NATIVE
```

The endpoint interfaces were configured as access ports.

```cisco
interface FastEthernet0/1
 description HR_CLIENT
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast

interface FastEthernet0/2
 description IT_CLIENT
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast

interface FastEthernet0/3
 description GUEST_CLIENT
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
```

The connection toward R1 was configured as an 802.1Q trunk.

```cisco
interface GigabitEthernet0/1
 description TRUNK_TO_R1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,999
```

---

### SW2 Configuration

SW2 connects the protected server to R1.

```cisco
vlan 40
 name SERVER

interface FastEthernet0/1
 description SERVER
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast

interface GigabitEthernet0/1
 description LINK_TO_R1
 switchport mode access
 switchport access vlan 40
```

---

### R1 Router-on-a-Stick Configuration

R1 provides the default gateways for the three client VLANs.

```cisco
interface GigabitEthernet0/0
 description TRUNK_TO_SW1
 no shutdown

interface GigabitEthernet0/0.10
 description HR_GATEWAY
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 description IT_GATEWAY
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/0.30
 description GUEST_GATEWAY
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0

interface GigabitEthernet0/0.999
 description UNUSED_NATIVE_VLAN
 encapsulation dot1Q 999 native
 no ip address
```

The server network uses a separate physical interface.

```cisco
interface GigabitEthernet0/1
 description SERVER_NETWORK
 ip address 192.168.40.1 255.255.255.0
 no shutdown
```

---

## Server Services

The Packet Tracer server was configured with:

- HTTP enabled
- HTTPS enabled
- FTP enabled

The server uses:

```text
IP Address: 192.168.40.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.40.1
```

These services provide traffic targets for the extended ACL.

---

## Baseline Connectivity

Before applying any ACLs, unrestricted routing was verified.

The following tests succeeded:

- HR to IT
- HR to Guest
- HR to Server
- IT to HR
- IT to Guest
- IT to Server
- Guest to HR
- Guest to IT
- Guest to Server
- HTTP access to the server
- HTTPS access to the server
- FTP access to the server
- ICMP access to the server

![Baseline Connectivity](./images/01-baseline-connectivity.png)

This confirmed that routing, VLANs, gateways, trunking, and server services were working before traffic filtering was introduced.

---

## Standard ACL Configuration

A named standard ACL was created to block traffic sourced from the Guest network.

```cisco
ip access-list standard BLOCK_GUEST_TO_HR
 deny 192.168.30.0 0.0.0.255
 permit any
```

The wildcard mask `0.0.0.255` matches every address in the `192.168.30.0/24` network.

Because the ACL cannot identify the destination, it was applied outbound on the HR subinterface.

```cisco
interface GigabitEthernet0/0.10
 ip access-group BLOCK_GUEST_TO_HR out
```

This placement affects traffic leaving R1 toward the HR network.

Guest traffic heading toward IT or the server does not leave through this interface and is therefore not blocked by the standard ACL.

---

## Extended ACL Configuration

A named extended ACL was created to protect the server.

```cisco
ip access-list extended PROTECT_SERVER
 remark HR_WEB_ACCESS
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq 80
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq 443

 remark IT_WEB_AND_FTP_ACCESS
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq 80
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq 443
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq 21

 remark GUEST_HTTP_ONLY
 permit tcp 192.168.30.0 0.0.0.255 host 192.168.40.10 eq 80

 remark BLOCK_ICMP_TO_SERVER
 deny icmp 192.168.10.0 0.0.0.255 host 192.168.40.10
 deny icmp 192.168.20.0 0.0.0.255 host 192.168.40.10
 deny icmp 192.168.30.0 0.0.0.255 host 192.168.40.10

 remark BLOCK_OTHER_CLIENT_ACCESS_TO_SERVER
 deny ip 192.168.10.0 0.0.0.255 host 192.168.40.10
 deny ip 192.168.20.0 0.0.0.255 host 192.168.40.10
 deny ip 192.168.30.0 0.0.0.255 host 192.168.40.10

 remark ALLOW_OTHER_ROUTED_TRAFFIC
 permit ip any any
```

The ACL was applied outbound on the server-facing interface.

```cisco
interface GigabitEthernet0/1
 ip access-group PROTECT_SERVER out
```

This placement filters traffic immediately before it enters the server network.

The final `permit ip any any` preserves unrelated routed traffic that does not match the server-protection rules.

---

## Verification

### Standard ACL Configuration

The standard ACL was verified with:

```cisco
show access-lists BLOCK_GUEST_TO_HR
```

![Standard ACL Configuration](./images/02-standard-acl-configuration.png)

The output confirmed that:

- Guest traffic was denied
- Other source networks were permitted
- Match counters increased during testing

---

### Guest-to-HR Denial

PC-GUEST attempted to ping PC-HR.

```text
PC-GUEST> ping 192.168.10.10
```

![Standard ACL Deny](./images/03-standard-acl-deny.png)

The ping failed because the packet was sourced from the Guest network and left R1 through the HR subinterface.

The deny counter increased on the standard ACL.

---

### Guest-to-IT Permit

PC-GUEST tested connectivity to PC-IT.

```text
PC-GUEST> ping 192.168.20.10
```

![Standard ACL Permit](./images/04-standard-acl-permit.png)

The ping succeeded because the standard ACL was applied only on traffic leaving toward the HR network.

This confirmed that the ACL placement prevented the rule from blocking Guest access to unrelated destinations.

---

### Extended ACL Configuration

The server-protection ACL was verified with:

```cisco
show access-lists PROTECT_SERVER
```

![Extended ACL Configuration](./images/05-extended-acl-configuration.png)

The output displayed each permit and deny entry along with its match counter.

---

### HTTP Access

HR, IT, and Guest clients opened:

```text
http://192.168.40.10
```

![HTTP Permitted](./images/06-http-permitted.png)

HTTP access succeeded from all three client networks because TCP port 80 was explicitly permitted.

---

### HTTPS Access

HR and IT opened:

```text
https://192.168.40.10
```

HTTPS access succeeded from HR and IT.

Guest HTTPS access failed because no TCP port 443 permit entry exists for the Guest network.

---

### FTP Access

PC-IT connected to the server FTP service.

```text
PC-IT> ftp 192.168.40.10
```

![FTP Permitted](./images/07-ftp-permitted.png)

The connection succeeded because TCP port 21 was explicitly permitted from the IT network.

FTP attempts from HR and Guest failed.

![Unauthorized Service Denied](./images/08-unauthorized-service-denied.png)

These attempts matched the broader deny entries protecting the server.

---

### ICMP Filtering

Each client attempted to ping the server.

```text
ping 192.168.40.10
```

![ICMP Denied](./images/09-icmp-denied.png)

All client-to-server pings failed because ICMP traffic from the three client networks was explicitly denied.

The inability to ping the server did not mean every service was unavailable. Approved TCP services still worked because the ACL handled each protocol separately.

---

### Traffic Test Results

| Source | Destination | Traffic | Result |
|---|---|---|---|
| HR | Server | HTTP | Permit |
| HR | Server | HTTPS | Permit |
| HR | Server | FTP | Deny |
| HR | Server | ICMP | Deny |
| IT | Server | HTTP | Permit |
| IT | Server | HTTPS | Permit |
| IT | Server | FTP | Permit |
| IT | Server | ICMP | Deny |
| Guest | Server | HTTP | Permit |
| Guest | Server | HTTPS | Deny |
| Guest | Server | FTP | Deny |
| Guest | Server | ICMP | Deny |
| Guest | HR | ICMP | Deny |
| Guest | IT | ICMP | Permit |
| HR | IT | ICMP | Permit |
| IT | HR | ICMP | Permit |

---

### ACL Match Counters

The ACL counters were checked after running the traffic tests.

```cisco
show access-lists
```

![ACL Match Counters](./images/10-acl-match-counters.png)

The counters confirmed that:

- Guest-to-HR traffic matched the standard ACL deny entry
- HTTP traffic matched TCP port 80 permit entries
- HTTPS traffic matched TCP port 443 permit entries
- IT FTP traffic matched the TCP port 21 permit entry
- ICMP traffic matched the ICMP deny entries
- Unauthorized service attempts matched the broader deny entries
- Unrelated routed traffic matched the final permit entry

---

### ACL Interface Placement

The interfaces were checked with:

```cisco
show ip interface GigabitEthernet0/0.10
```

```cisco
show ip interface GigabitEthernet0/1
```

The output confirmed that:

- `BLOCK_GUEST_TO_HR` was applied outbound on `G0/0.10`
- `PROTECT_SERVER` was applied outbound on `G0/1`

---

## Failure and Recovery Testing

### Incorrect Standard ACL Placement

The standard ACL was temporarily removed from the HR subinterface.

```cisco
interface GigabitEthernet0/0.10
 no ip access-group BLOCK_GUEST_TO_HR out
```

It was then applied inbound on the Guest subinterface.

```cisco
interface GigabitEthernet0/0.30
 ip access-group BLOCK_GUEST_TO_HR in
```

![Wrong Placement Failure](./images/11-wrong-placement-failure.png)

This blocked all routed traffic sourced from the Guest network, including:

- Guest to HR
- Guest to IT
- Guest to Server

The ACL blocked too much because a standard ACL cannot identify the destination.

The incorrect application was removed.

```cisco
interface GigabitEthernet0/0.30
 no ip access-group BLOCK_GUEST_TO_HR in
```

The ACL was restored outbound on the HR subinterface.

```cisco
interface GigabitEthernet0/0.10
 ip access-group BLOCK_GUEST_TO_HR out
```

Guest access to IT and permitted server services recovered, while Guest access to HR remained blocked.

---

### Incorrect ACL Direction

The extended ACL was temporarily removed from the outbound direction.

```cisco
interface GigabitEthernet0/1
 no ip access-group PROTECT_SERVER out
```

It was then applied inbound on the server-facing interface.

```cisco
interface GigabitEthernet0/1
 ip access-group PROTECT_SERVER in
```

The intended client-to-server traffic was no longer filtered because client packets enter R1 through the client subinterfaces and leave through `G0/1`.

The ACL was inspecting traffic entering R1 from the server network instead.

The incorrect application was removed.

```cisco
interface GigabitEthernet0/1
 no ip access-group PROTECT_SERVER in
```

The ACL was restored in the outbound direction.

```cisco
interface GigabitEthernet0/1
 ip access-group PROTECT_SERVER out
```

The required server filtering returned.

---

### First-Match Processing

A broad permit entry was temporarily placed before the Guest HTTPS restriction.

```cisco
ip access-list extended PROTECT_SERVER
 5 permit ip 192.168.30.0 0.0.0.255 host 192.168.40.10
```

![Rule Order Failure](./images/12-rule-order-failure.png)

Guest HTTPS and FTP traffic was permitted because the broad entry matched before the packet reached the later restrictions.

The packet was not evaluated against any remaining entries after the first match.

The incorrect rule was removed.

```cisco
ip access-list extended PROTECT_SERVER
 no 5
```

Guest access was tested again, and only HTTP remained available.

---

### Implicit Deny

The final broader permit entry was temporarily removed.

```cisco
ip access-list extended PROTECT_SERVER
 no permit ip any any
```

Traffic that did not match an earlier permit entry was blocked by the implicit deny at the end of the ACL.

This affected unrelated traffic that was not intended to be part of the server restriction.

The final permit was restored.

```cisco
ip access-list extended PROTECT_SERVER
 permit ip any any
```

![Service Recovery](./images/13-service-recovery.png)

Approved services and unrelated routed traffic recovered after the rule was restored.

---

## Key Takeaways

- Standard ACLs match only the source IPv4 address
- Extended ACLs can match source, destination, protocol, and port
- Standard ACLs are normally placed close to the destination
- Extended ACLs are normally placed close to the source when practical
- Interface direction is evaluated from the router’s perspective
- Inbound ACLs filter packets as they enter an interface
- Outbound ACLs filter packets before they leave an interface
- Wildcard masks determine which address bits must match
- A wildcard mask of `0.0.0.255` matches an entire `/24` network
- ACLs are processed from top to bottom
- The first matching entry determines the result
- Later entries are not evaluated after a match
- Narrow rules must be placed before broad rules
- Every ACL ends with an implicit deny
- A final permit may be required to preserve unrelated traffic
- ACL match counters help prove which entries processed traffic
- Successful ping tests do not prove that application services are allowed
- Failed ping tests do not prove that every application service is blocked
- Baseline testing should be completed before ACLs are applied
- Incorrect ACL placement can block more traffic than intended
- Removing or correcting an ACL entry can restore service without rebuilding the network

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- Standard IPv4 ACLs
- Extended IPv4 ACLs
- Router-on-a-Stick
- HTTP
- HTTPS
- FTP
- ICMP
