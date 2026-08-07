# ACL Traffic Filtering

## Overview

This lab demonstrates how standard and extended IPv4 access control lists control traffic between multiple client networks and a protected server network.

HR, IT, and Guest devices connect through separate VLANs. R1 provides inter-VLAN routing and connects the client networks to a server running HTTP, HTTPS, and FTP services.

A standard ACL prevents Guest devices from reaching the HR network. An extended ACL allows each client network to access only the approved services on the server.

---

## Objectives

- Configure standard and extended IPv4 ACLs
- Match traffic using source and destination addresses
- Filter traffic by protocol and TCP port
- Apply ACLs in the correct direction
- Demonstrate standard ACL placement
- Verify ACL configuration and interface attachment
- Verify permitted and denied traffic
- Inspect ACL match counters
- Demonstrate first-match and implicit deny behavior
- Test failure and recovery after ACL mistakes

---

## Topology

![Network Topology](./images/topology.png)

R1 uses Router-on-a-Stick for the HR, IT, and Guest VLANs. A separate physical interface connects the server network.

---

## Network Design

| VLAN | Name | Network | Gateway | Endpoint |
|---|---|---|---|---|
| 10 | HR | `192.168.10.0/24` | `192.168.10.1` | PC-HR: `192.168.10.10` |
| 20 | IT | `192.168.20.0/24` | `192.168.20.1` | PC-IT: `192.168.20.10` |
| 30 | Guest | `192.168.30.0/24` | `192.168.30.1` | PC-GUEST: `192.168.30.10` |
| 40 | Server | `192.168.40.0/24` | `192.168.40.1` | SERVER: `192.168.40.10` |
| 999 | Native | N/A | N/A | Unused native VLAN |

The server provides HTTP, HTTPS, and FTP services for ACL testing.

---

## Security Requirements

### Client Network Filtering

The Guest network must not be allowed to reach the HR network.

| Source | Destination | Result |
|---|---|---|
| Guest | HR | Deny |
| Guest | IT | Permit |
| HR | IT | Permit |
| IT | HR | Permit |

### Server Access

| Source | HTTP | HTTPS | FTP | ICMP |
|---|---:|---:|---:|---:|
| HR | Permit | Permit | Deny | Deny |
| IT | Permit | Permit | Permit | Deny |
| Guest | Permit | Deny | Deny | Deny |

---

## Baseline Connectivity

Before applying ACLs, routing and server connectivity were tested without restrictions.

The client networks could communicate with each other, ping the server, and access its enabled services.

![Baseline Connectivity](./images/01-baseline-connectivity.png)

This confirmed that any later failures were caused by ACL filtering rather than an addressing, VLAN, trunking, routing, or server problem.

---

## Standard ACL Configuration

A named standard ACL was created to block traffic sourced from the Guest network.

```cisco
ip access-list standard BLOCK_GUEST_TO_HR
 deny 192.168.30.0 0.0.0.255
 permit any
```

Because a standard ACL matches only the source address, it was placed close to the HR destination.

```cisco
interface GigabitEthernet0/0.10
 ip access-group BLOCK_GUEST_TO_HR out
```

Applying the ACL outbound on the HR subinterface blocks Guest traffic only when it is leaving R1 toward the HR network.

Guest traffic toward IT and the server leaves through different interfaces and is not affected by this ACL.

---

## Standard ACL Verification

### ACL Rules

The standard ACL was checked on R1.

```cisco
show access-lists BLOCK_GUEST_TO_HR
```

![Standard ACL Configuration](./images/02-standard-acl-config.png)

The output confirmed that the Guest subnet was denied and all other source addresses were permitted.

---

### ACL Interface Attachment

The HR subinterface was checked to verify that the ACL was attached in the correct direction.

```cisco
show ip interface GigabitEthernet0/0.10
```

![Standard ACL Interface](./images/03-standard-acl-interface.png)

The output confirmed that `BLOCK_GUEST_TO_HR` was applied outbound.

---

### Traffic Testing

PC-GUEST attempted to reach PC-HR.

```text
PC-GUEST> ping 192.168.10.10
```

![Guest-to-HR Denied](./images/04-standard-acl-deny.png)

The ping failed because the packet matched the Guest deny entry.

PC-GUEST then tested connectivity to PC-IT.

```text
PC-GUEST> ping 192.168.20.10
```

![Guest-to-IT Permitted](./images/05-standard-acl-permit.png)

The ping succeeded, confirming that the standard ACL blocked only traffic leaving toward the HR network.

---

## Extended ACL Configuration

A named extended ACL was created to protect the server.

```cisco
ip access-list extended PROTECT_SERVER
 remark HR_HTTP_AND_HTTPS
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq 80
 permit tcp 192.168.10.0 0.0.0.255 host 192.168.40.10 eq 443

 remark IT_HTTP_HTTPS_AND_FTP
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq 80
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq 443
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.10 eq 21

 remark GUEST_HTTP_ONLY
 permit tcp 192.168.30.0 0.0.0.255 host 192.168.40.10 eq 80

 remark DENY_OTHER_CLIENT_ACCESS_TO_SERVER
 deny ip 192.168.10.0 0.0.0.255 host 192.168.40.10
 deny ip 192.168.20.0 0.0.0.255 host 192.168.40.10
 deny ip 192.168.30.0 0.0.0.255 host 192.168.40.10

 remark ALLOW_UNRELATED_TRAFFIC
 permit ip any any
```

The ACL was applied outbound on the server-facing interface.

```cisco
interface GigabitEthernet0/1
 ip access-group PROTECT_SERVER out
```

This filters traffic immediately before it enters the server network.

The specific permit entries appear before the broader deny entries because ACLs use top-down, first-match processing.

---

## Extended ACL Verification

### ACL Rules

The extended ACL was checked on R1.

```cisco
show access-lists PROTECT_SERVER
```

![Extended ACL Configuration](./images/06-extended-acl-config.png)

The output confirmed that the service-specific permit entries and broader deny entries were installed in the intended order.

---

### ACL Interface Attachment

The server-facing interface was checked to verify where the ACL was applied.

```cisco
show ip interface GigabitEthernet0/1
```

![Extended ACL Interface](./images/07-extended-acl-interface.png)

The output confirmed that `PROTECT_SERVER` was applied outbound toward the server network.

---

### Server Access Testing

| Source | Traffic | Result |
|---|---|---|
| HR | HTTP | Permit |
| HR | HTTPS | Permit |
| HR | FTP | Deny |
| HR | ICMP | Deny |
| IT | HTTP | Permit |
| IT | HTTPS | Permit |
| IT | FTP | Permit |
| IT | ICMP | Deny |
| Guest | HTTP | Permit |
| Guest | HTTPS | Deny |
| Guest | FTP | Deny |
| Guest | ICMP | Deny |

![HTTP Access Permitted](./images/08-http-permitted.png)

HTTP access succeeded from HR, IT, and Guest because TCP port 80 was explicitly permitted.

![FTP Access Testing](./images/09-ftp-testing.png)

FTP succeeded from IT but failed from HR and Guest.

![Unauthorized Traffic Denied](./images/10-unauthorized-traffic-denied.png)

HTTPS from Guest and ICMP from all client networks were denied by the broader server-protection entries.

---

### ACL Match Counters

The ACL entries and match counters were checked after testing.

```cisco
show access-lists
```

![ACL Match Counters](./images/11-acl-match-counters.png)

The output confirmed that:

- Guest-to-HR traffic matched the standard ACL deny entry
- HTTP and HTTPS traffic matched the correct TCP permit entries
- IT FTP traffic matched the TCP port 21 permit entry
- Unauthorized traffic matched the broader deny entries
- Unrelated traffic matched the final permit entry

The counters provide direct evidence that the tested traffic was processed by the expected ACL rules.

---

## Failure and Recovery Testing

### Incorrect Standard ACL Placement

The standard ACL was temporarily applied inbound on the Guest subinterface.

```cisco
interface GigabitEthernet0/0.30
 ip access-group BLOCK_GUEST_TO_HR in
```

![Incorrect ACL Placement](./images/12-wrong-placement.png)

Because the ACL matched only the Guest source network, this placement blocked Guest traffic toward HR, IT, and the server.

The ACL was removed from the Guest subinterface and restored outbound on the HR subinterface.

Guest access to IT and permitted server services recovered while Guest access to HR remained blocked.

---

### First-Match Processing

A broad Guest permit was temporarily placed above the more specific restrictions.

```cisco
ip access-list extended PROTECT_SERVER
 5 permit ip 192.168.30.0 0.0.0.255 host 192.168.40.10
```

Guest HTTPS and FTP traffic became permitted because the packet matched this entry before reaching the later deny rules.

The broad permit was removed, restoring the intended HTTP-only Guest access.

---

### Implicit Deny

The final permit entry was temporarily removed.

```cisco
ip access-list extended PROTECT_SERVER
 no permit ip any any
```

Traffic that did not match an earlier permit was blocked by the implicit deny at the end of the ACL.

The final permit was restored, and unrelated routed traffic recovered.

![ACL Recovery](./images/13-acl-recovery.png)

---

## Key Takeaways

- Standard ACLs match only the source IPv4 address
- Extended ACLs can match source, destination, protocol, and port
- Standard ACLs are normally placed close to the destination
- ACL direction is determined from the router’s perspective
- `show access-lists` verifies ACL rules and match counters
- `show ip interface` verifies where an ACL is attached and in which direction
- ACL entries are processed from top to bottom
- The first matching entry determines the result
- Specific entries must appear before broader entries
- Every ACL ends with an implicit deny
- A final permit may be required to preserve unrelated traffic
- Incorrect ACL placement can block more traffic than intended
- Baseline testing helps separate ACL problems from routing problems

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- Standard and extended ACLs
- Router-on-a-Stick
- HTTP
- HTTPS
- FTP
- ICMP
