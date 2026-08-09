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

### Device Roles

| Device | Purpose |
|---|---|
| R0 | Inter-VLAN routing and default gateways |
| Switch 0 | Connects R0 and the internal servers |
| Switch 1 | Connects the client network |
| DNS Server | Resolves internal hostnames |
| Web Server | Hosts the internal web page |
| PC | DNS client |

---

## DNS Design

The DNS server was used to provide name resolution for the internal web server.

An A record was created to map the hostname `testsite` to the web server's IPv4 address.

```text
testsite → <WEB-SERVER-IP>
```

The client was then configured to use the internal DNS server for name resolution.

---

## Baseline Connectivity

Before configuring DNS, basic IP connectivity was verified.

The client successfully reached both the DNS server and web server by IP address.

![Baseline Connectivity](./images/01-baseline-connectivity.png)

This confirmed that VLANs, Router-on-a-Stick, and routing between the user and services networks were working before DNS was added.

---

## DNS Server Configuration

DNS was enabled on the internal DNS server.

An A record was created for the web server:

```text
Name: testsite
Address: <WEB-SERVER-IP>
```

This allows the DNS server to return the web server's IP address when the client requests `testsite`.

---

## Client DNS Configuration

The client was configured to use the internal DNS server.

```text
DNS Server: <DNS-SERVER-IP>
```

This allows the client to send DNS requests to the server when a hostname needs to be resolved.

---

# Verification

## DNS Record Verification

The configured DNS record was verified on the DNS server.

![DNS Record](./images/02-dns-record.png)

The record confirmed that `testsite` was mapped to the web server's IP address.

---

## Hostname Resolution

The client tested the hostname instead of using the web server's IP address directly.

```text
ping testsite
```

![DNS Resolution](./images/03-dns-resolution.png)

The client resolved `testsite` to the correct web server IP address and received a successful response.

This confirmed that DNS name resolution was working.

---

## Web Access by Name

The internal web page was then accessed using the hostname.

```text
http://testsite
```

![Web Access](./images/04-web-access.png)

The page loaded successfully, confirming that the client could resolve the hostname and then connect to the web server.

---

## Failed Resolution Test

The DNS record for `testsite` was temporarily removed.

![Failed Resolution](./images/05-failed-resolution.png)

The client could no longer resolve the hostname, even though the network and web server were still available by IP address.

The DNS record was then restored and normal name resolution returned.

---

## Key Takeaways

- DNS translates hostnames into IP addresses
- An A record maps a hostname to an IPv4 address
- Clients must know which DNS server to query
- DNS depends on working IP connectivity between the client and DNS server
- DNS can operate across routed VLANs
- Name resolution and network connectivity are separate functions
- A server can still be reachable by IP even when DNS resolution fails
- Successful DNS resolution does not automatically prove that the application itself is working
- End-to-end testing should verify both name resolution and access to the actual service

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- VLANs
- Router-on-a-Stick
- DNS
- HTTP
