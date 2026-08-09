# DNS Name Resolution

## Overview

This lab shows how DNS translates hostnames into IP addresses so users can access network resources by name instead of remembering addresses.

The topology uses a router, two switches, a client, and two internal servers. One server provides DNS, while the other hosts a web service. The client reaches both servers across VLANs through Router-on-a-Stick and uses DNS to resolve the web server’s hostname before connecting to it.

---

## Objectives

- Configure separate user and services VLANs
- Provide inter-VLAN routing using Router-on-a-Stick
- Configure an internal DNS server
- Create a DNS record for a web server
- Configure a client to use the DNS server
- Verify connectivity before DNS is configured
- Verify hostname resolution
- Access a web service using a hostname
- Test what happens when the DNS record is removed

---

## Topology

![Network Topology](./images/topology.png)

The client is placed in the user VLAN, while the DNS and web servers are placed in the services VLAN.

R0 routes traffic between both networks using Router-on-a-Stick.

---

## Network Design

| VLAN | Name | Purpose |
|---|---|---|
| 10 | USERS | End-user network |
| 20 | SERVICES | Internal server network |

### Device Addressing

| Device | IP Address | Purpose |
|---|---|---|
| R0 | VLAN gateway addresses | Inter-VLAN routing |
| Web Server | `192.168.20.10` | Hosts the internal web page |
| DNS Server | `192.168.20.11` | Provides DNS name resolution |
| PC | User VLAN address | DNS client |

---

## DNS Design

The DNS server provides name resolution for the internal web server.

An A record was created to map the hostname `testsite` to the web server.

```text
testsite → 192.168.20.10
```

The client was configured to use the DNS server at `192.168.20.11`.

---

## Baseline Connectivity

Before configuring DNS, basic IP connectivity was verified.

The client successfully reached both the web server at `192.168.20.10` and the DNS server at `192.168.20.11`.

![Baseline Connectivity](./images/01-baseline-connectivity.png)

This confirmed that the VLANs and inter-VLAN routing were working before DNS was added.

---

## DNS Server Configuration

DNS was enabled on the internal DNS server at `192.168.20.11`.

An A record was created for the web server:

```text
Name: testsite
Address: 192.168.20.10
```

This allows the DNS server to return the web server's address when the client requests `testsite`.

---

## Client DNS Configuration

The client was configured to use the internal DNS server.

```text
DNS Server: 192.168.20.11
```

When the client needs to resolve a hostname, it sends the DNS request to this server.

---

# Verification

## DNS Record Verification

The configured A record was verified on the DNS server.

![DNS Record](./images/02-dns-record.png)

The record confirmed that `testsite` was mapped to `192.168.20.10`.

---

## Hostname Resolution

The client tested the hostname instead of using the web server's IP address directly.

```text
ping testsite
```

![DNS Resolution](./images/03-dns-resolution.png)

The client resolved `testsite` to `192.168.20.10` and received a successful response.

This confirmed that DNS name resolution was working.

---

## Web Access by Name

The internal web page was then accessed using the hostname.

```text
http://testsite
```

![Web Access](./images/04-web-access.png)

The page loaded successfully, confirming that the client could resolve `testsite` and connect to the web server.

---

## Failed Resolution Test

The DNS record for `testsite` was temporarily removed.

![Failed Resolution](./images/05-failed-resolution.png)

The client could no longer resolve the hostname, even though the web server was still reachable directly at `192.168.20.10`.

The DNS record was restored after the test and normal name resolution returned.

---

## Key Takeaways

- DNS translates hostnames into IP addresses
- An A record maps a hostname to an IPv4 address
- Clients must be configured with a DNS server to resolve names
- DNS requires working IP connectivity between the client and DNS server
- DNS can operate across routed VLANs
- Name resolution and basic network connectivity are separate functions
- A web server can remain reachable by IP even when DNS resolution fails
- Successful name resolution does not automatically prove that the web service itself is working
- Testing by both IP address and hostname helps separate DNS problems from connectivity problems

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- VLANs
- Router-on-a-Stick
- DNS
- HTTP
