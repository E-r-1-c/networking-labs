# ACL Traffic Filtering

> **Status: Work in Progress**

This folder is reserved for a network security lab covering standard and extended IPv4 access control lists.

The completed lab will use ACLs to control communication between multiple networks, restrict access to selected services, and verify how Cisco IOS processes permitted and denied traffic.

---

## Planned Focus Areas

- Standard IPv4 ACLs
- Extended IPv4 ACLs
- Numbered and named ACLs
- Source and destination address matching
- Protocol and port filtering
- Wildcard masks
- Inbound and outbound ACL application
- ACL placement
- Top-down rule processing
- First-match behavior
- Implicit deny behavior
- Permit and deny verification
- ACL match counters
- Configuration mistakes and recovery

---

## Planned Traffic Requirements

The completed topology will include multiple client networks and a server network.

ACL requirements will be used to demonstrate rules such as:

- Permit one client network to reach another network
- Deny a selected source network
- Permit specific application traffic to a server
- Deny unauthorized access to protected services
- Allow required traffic without permitting unnecessary access
- Confirm that unmatched traffic is blocked by the implicit deny

---

## Planned Testing

The completed lab will verify:

- Traffic permitted by an ACL
- Traffic denied by an ACL
- Correct standard ACL placement
- Correct extended ACL placement
- Protocol and port-specific filtering
- ACL processing order
- ACL match counters
- The effect of applying an ACL in the wrong direction
- Service recovery after correcting an ACL rule

---

## Planned Lab Structure

The completed lab will include:

- Project overview
- Objectives
- Network topology
- Addressing design
- Security requirements
- Standard ACL configuration
- Extended ACL configuration
- ACL placement and direction
- Connectivity and service testing
- Failure and recovery testing
- Screenshots
- Key takeaways

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
