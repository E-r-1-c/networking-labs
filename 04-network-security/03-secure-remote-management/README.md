# Secure Remote Management

## Overview

This lab demonstrates how Cisco network devices can be managed remotely through the command line and how that access can be secured.

The lab first uses Telnet with local authentication to verify basic remote CLI access. SSH is then configured using a hostname, domain name, RSA keys, and SSH version 2. The VTY lines are hardened to accept SSH only, and an ACL is applied directly to the VTY lines so only the administrator PC can establish remote management sessions.

---

## Objectives

- Configure IP connectivity for remote management
- Configure VTY lines for remote CLI access
- Create a local user account for authentication
- Verify Telnet access
- Configure privileged EXEC access
- Generate RSA keys for SSH
- Configure SSH version 2
- Verify SSH access
- Restrict VTY access to SSH only
- Verify Telnet is blocked after hardening
- Restrict remote management to the administrator PC using an ACL
- Verify an unauthorized source cannot establish an SSH session
- Test failed authentication
- Verify active SSH sessions

---

## Topology

![Network Topology](./images/topology.png)

The topology uses one router, one switch, an administrator PC, and a second PC used to test unauthorized remote access.

PC-Admin is the authorized management workstation. PC-Test is used later to verify that the VTY ACL prevents other devices from remotely managing R0 and SW0.

---

## Network Design

| Device | IP Address | Role |
|---|---|---|
| R0 | `192.168.10.1` | Router and remotely managed device |
| SW0 | `192.168.10.10` | Switch management address |
| PC-Admin | `192.168.10.11` | Authorized administrator workstation |
| PC-Test | `192.168.10.12` | Unauthorized workstation used for ACL testing |

---

## Baseline Connectivity

Before configuring remote access, connectivity was verified from PC-Admin.

```text
ping 192.168.10.1
ping 192.168.10.10
```

Successful responses confirmed that both network devices were reachable before Telnet or SSH was tested.

PC-Test was also given normal IP connectivity so it could later be used to prove that the VTY ACL blocks remote management without blocking normal network access.

---

## Local User Authentication

A local user account was created on both devices.

```text
username cisco secret cisco
```

This creates a local account that can be used to authenticate remote users.

For this lab:

```text
Username: cisco
Password: cisco
```

The account controls remote login authentication. It does not automatically provide Privileged EXEC access.

---

## VTY Configuration

Cisco IOS uses VTY lines for remote CLI sessions.

The existing VTY lines were selected and configured to use the local user database.

```text
line vty 0 15
 login local
```

`line vty 0 15` selects VTY lines 0 through 15.

`login local` tells those lines to authenticate remote users against the local username database.

At this point, authentication was configured for remote CLI sessions.

---

## Telnet Remote Access

Telnet was tested first as the baseline remote-management method.

No `transport input telnet` command was configured during this stage because Telnet was already accepted by the existing VTY transport settings on the Packet Tracer devices.

From PC-Admin:

```text
telnet 192.168.10.1
telnet 192.168.10.10
```

![Telnet Access](./images/01-telnet-access.png)

Both connections succeeded.

This confirmed that:

- IP connectivity was working
- The VTY lines were accepting remote sessions
- Local authentication was working
- R0 and SW0 could be managed remotely

Telnet provides remote CLI access, but the session is not encrypted.

---

## Privileged EXEC Access

After logging in remotely, the session entered User EXEC mode.

```text
R0>
```

The local username and password authenticated the remote login, but they did not provide Privileged EXEC access.

The remote session was exited and the device console was used to configure an enable secret.

```text
enable secret cisco
```

The remote connection was then tested again.

After logging in, the `enable` command could be used:

```text
R0> enable
Password:
R0#
```

The remote login and Privileged EXEC password protect different stages of access.

| Access Stage | Credential Used |
|---|---|
| Remote login | Local username and password |
| User EXEC → Privileged EXEC | Enable secret |

`username cisco secret cisco` creates the local account used to log in.

`enable secret cisco` separately configures the password used to enter Privileged EXEC mode.

---

## SSH Preparation

SSH requires additional configuration that Telnet does not.

Each device was configured with a hostname.

```text
hostname R0
```

```text
hostname SW0
```

A domain name was also configured.

```text
ip domain-name <domain-name>
```

On the IOS implementation used in this lab, the hostname and domain name are required before the RSA keys can be generated.

---

## RSA Key Generation

RSA keys were generated on both devices.

```text
crypto key generate rsa
```

A key size large enough for SSH version 2 was selected.

SSH version 2 requires an RSA modulus of at least 768 bits, so the smaller 512-bit value was not used.

The RSA key pair provides the cryptographic keys required by the SSH server.

---

## SSH Version 2

After the RSA keys were generated, SSH version 2 was explicitly configured.

```text
ip ssh version 2
```

The RSA keys are generated first because the SSH server requires a usable key pair. The SSH version command then specifies that the device should use SSH version 2.

SSH version 2 provides encrypted remote CLI access instead of the clear-text remote access provided by Telnet.

The VTY lines were already configured with `login local`, so the existing local account could also be used for SSH authentication.

There was no need to configure `login local` again.

---

## SSH Remote Access

SSH was tested from PC-Admin.

```text
ssh -l cisco 192.168.10.1
```

```text
ssh -l cisco 192.168.10.10
```

![SSH Access](./images/02-ssh-access.png)

Both connections succeeded.

This confirmed that:

- The devices were reachable
- Local authentication was working
- RSA keys were available
- SSH version 2 was active
- The VTY lines were accepting SSH sessions

At this stage, SSH worked, but Telnet had not yet been explicitly removed from the VTY lines.

---

## VTY Transport Hardening

After SSH was verified, the VTY lines were restricted so that SSH was the only accepted remote-management protocol.

```text
line vty 0 15
 transport input ssh
```

`transport input` controls which incoming remote-access protocols can use the VTY lines.

The VTY lines are entered again here because their transport policy is now being changed.

The existing authentication configuration remains in place.

The important VTY settings are now:

```text
login local
transport input ssh
```

This means:

- Remote users authenticate with the local username database
- SSH connections are accepted
- Telnet connections are rejected

---

# Verification

## SSH Status

SSH status was verified on both devices.

```text
show ip ssh
```

![SSH Status](./images/03-ssh-status.png)

The output confirms that SSH is enabled and version 2 is active.

---

## SSH Access After Hardening

SSH was tested again after the VTY lines were restricted to SSH only.

```text
ssh -l cisco 192.168.10.1
```

```text
ssh -l cisco 192.168.10.10
```

![Successful SSH Login](./images/04-successful-ssh-login.png)

Both connections remained successful.

This confirms that secure remote management still works after the VTY transport policy is hardened.

---

## Telnet Blocking Test

Telnet was tested again from PC-Admin.

```text
telnet 192.168.10.1
```

```text
telnet 192.168.10.10
```

![Telnet Blocked](./images/05-telnet-blocked.png)

The devices remained reachable by IP, but Telnet connections failed.

This confirms that the failure was caused by the VTY transport policy rather than a loss of network connectivity.

---

## VTY ACL Hardening

SSH-only access controls the protocol used for remote management, but it does not by itself restrict which devices are allowed to attempt an SSH connection.

A standard ACL was created to permit only PC-Admin:

```text
ip access-list standard VTY-MANAGEMENT
 permit host 192.168.10.11
```

The ACL was then applied directly to the VTY lines:

```text
line vty 0 15
 access-class VTY-MANAGEMENT in
```

Unlike the ACLs used for normal routed traffic, this ACL is not applied to a physical or logical interface.

`access-class` applies the ACL directly to incoming VTY access.

The two controls now protect different parts of remote management:

| Control | Purpose |
|---|---|
| `transport input ssh` | Allows only SSH as the remote-management protocol |
| `access-class VTY-MANAGEMENT in` | Allows remote management only from approved source addresses |

---

## Authorized Management Test

PC-Admin was tested after the VTY ACL was applied.

```text
ssh -l cisco 192.168.10.1
```

```text
ssh -l cisco 192.168.10.10
```

![Authorized SSH Access](./images/06-authorized-ssh-access.png)

SSH remained successful because PC-Admin at `192.168.10.11` is permitted by the VTY ACL.

---

## Unauthorized Management Test

PC-Test at `192.168.10.12` was then used to attempt an SSH connection.

```text
ssh -l cisco 192.168.10.1
```

```text
ssh -l cisco 192.168.10.10
```

![Unauthorized SSH Blocked](./images/07-unauthorized-ssh-blocked.png)

The SSH attempts failed because PC-Test is not permitted by the VTY ACL.

PC-Test can still have normal IP connectivity to the devices.

This proves that the ACL is restricting remote management access rather than simply removing network connectivity.

---

## Failed Authentication Test

From the authorized PC, an SSH connection was attempted using incorrect login credentials.

![Failed Authentication](./images/08-failed-authentication.png)

The connection was allowed to reach the SSH service because PC-Admin is an authorized source, but the login was rejected because authentication failed.

This demonstrates another separate control:

```text
Source allowed by VTY ACL
        ↓
SSH protocol allowed
        ↓
Username/password still required
```

---

## Active SSH Session

While connected through SSH, the active session was verified from the network device.

```text
show users
```

or:

```text
show ssh
```

![Active SSH Session](./images/09-active-ssh-session.png)

The output confirms that an SSH session is active.

---

## Final VTY Configuration

The final VTY configuration was reviewed.

```text
show running-config | section line vty
```

The important settings are:

```text
login local
transport input ssh
access-class VTY-MANAGEMENT in
```

Together, these settings control three different parts of remote access:

| Setting | Controls |
|---|---|
| `login local` | How the user authenticates |
| `transport input ssh` | Which remote protocol is accepted |
| `access-class VTY-MANAGEMENT in` | Which source devices may attempt remote access |

---

## Final Remote Management Policy

The completed configuration creates multiple layers of control.

| Test | Expected Result | Reason |
|---|---|---|
| PC-Admin ping | Allowed | Basic IP connectivity |
| PC-Test ping | Allowed | VTY ACL does not block normal IP traffic |
| PC-Admin SSH | Allowed | Authorized source using permitted protocol |
| PC-Admin Telnet | Blocked | VTY lines accept SSH only |
| PC-Test SSH | Blocked | Source is not permitted by VTY ACL |
| PC-Admin SSH with wrong password | Blocked | Local authentication fails |
| PC-Admin SSH with correct password | Allowed | Source, protocol, and authentication all succeed |

---

## Authentication and Access Controls

| Configuration | Purpose |
|---|---|
| `username cisco secret cisco` | Creates the local user used for remote login |
| `login local` | Uses the local user database for VTY authentication |
| `enable secret cisco` | Protects access to Privileged EXEC mode |
| `transport input ssh` | Restricts the VTY lines to SSH |
| `access-class VTY-MANAGEMENT in` | Restricts VTY access by source IP address |

These controls operate at different stages and are not replacements for one another.

---

## Key Takeaways

- Telnet and SSH both provide remote CLI access
- Cisco IOS uses VTY lines for remote terminal sessions
- `login local` tells the VTY lines to authenticate against the local username database
- Telnet was already accepted by the initial VTY transport settings in this lab
- SSH requires additional configuration before it can be used
- A hostname and domain name are used before generating the RSA keys in this lab
- RSA keys are required for the SSH server
- SSH version 2 requires an RSA key of at least 768 bits
- The RSA key can be generated before explicitly selecting SSH version 2
- The local account controls remote login authentication
- The enable secret separately protects Privileged EXEC access
- `transport input ssh` restricts the VTY lines to SSH and blocks Telnet
- A VTY ACL can restrict remote management without blocking normal network connectivity
- `access-class` applies an ACL directly to VTY access rather than to a routed interface
- Protocol restrictions and source restrictions solve different problems
- An authorized source can still be rejected if authentication fails
- Connectivity, source authorization, transport protocol, authentication, and privilege level are separate parts of remote management

---

## Environment

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- Telnet
- SSH Version 2
- RSA
- VTY lines
- Local authentication
- Standard ACL
