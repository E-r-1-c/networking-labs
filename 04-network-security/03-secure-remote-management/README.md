# Secure Remote Management

## Overview

This lab demonstrates how Cisco network devices can be managed remotely and how that access can be secured.

The lab begins by using Telnet to verify basic remote CLI access with VTY password authentication. SSH is then configured using local authentication to provide encrypted remote access. Finally, the VTY lines are restricted to SSH only, and a VTY ACL is added so only the administrator PC can remotely manage the devices.

---

## Objectives

- Configure remote CLI access using Telnet
- Configure VTY authentication
- Configure Privileged EXEC access
- Configure SSH version 2 using local authentication
- Restrict remote access to SSH only
- Restrict VTY access to an authorized management PC
- Test successful and failed remote-access attempts
- Verify the final remote-management configuration

---

## Topology

![Network Topology](./images/topology.png)

The topology uses one router, one switch, and two PCs on the same network.

PC-Admin is the authorized management workstation used for Telnet and SSH access. PC-Test is used later to verify that the VTY ACL blocks unauthorized remote management without blocking normal network connectivity.

---

## Network Design

| Device | IP Address | Role |
|---|---|---|
| R0 | `192.168.10.1` | Router and remotely managed device |
| SW0 | `192.168.10.10` | Switch management address |
| PC-Admin | `192.168.10.11` | Authorized management workstation |
| PC-Test | `192.168.10.12` | Workstation used for unauthorized-access testing |

---

## Baseline Connectivity

Before configuring remote access, basic connectivity was verified from both PCs.

From PC-Admin:

```text
ping 192.168.10.1
ping 192.168.10.10
```

PC-Test was also able to reach R0 and SW0.

Successful pings confirmed that normal IP connectivity was working before Telnet or SSH was configured. This provides a baseline so later remote-access failures can be separated from connectivity problems.

---

## Telnet Configuration

Cisco IOS uses VTY lines for remote CLI sessions.

For the initial Telnet test, a password was configured directly on the VTY lines.

```text
line vty 0 15
 password cisco
 login
```

`password cisco` configures the VTY password, while `login` tells the VTY lines to require that password for remote access.

No username is required with this method.

Telnet does not specifically require the `password` command. It only requires the VTY lines to have an authentication method. In this part of the lab, a VTY password with `login` is used. Later, SSH uses a local username and password with `login local`.

The VTY lines also control which remote-access protocols they accept through `transport input`.

```text
transport input telnet
```

allows only Telnet.

```text
transport input ssh
```

allows only SSH.

```text
transport input telnet ssh
```

allows both.

No explicit `transport input` command was configured at this point. On the Packet Tracer devices used in this lab, Telnet was already accepted by the existing VTY transport settings.

PC-Admin then used Packet Tracer's Telnet/SSH application to connect to:

```text
R0  - 192.168.10.1
SW0 - 192.168.10.10
```

The VTY password was entered when prompted:

```text
Password: cisco
```

![Telnet Access](./images/01-telnet-access.png)

Both connections succeeded, confirming that the devices were reachable, the VTY lines were accepting Telnet, and VTY authentication was working.

Telnet provides remote CLI access, but the session is not encrypted.

---

## Privileged EXEC Access

After connecting through Telnet, the session entered User EXEC mode.

```text
R0>
```

The VTY password allows the remote login, but it does not provide Privileged EXEC access.

When connected directly through the device console, `enable` can enter Privileged EXEC mode even if no enable secret has been configured.

A remote VTY session is different. A Telnet user cannot move from User EXEC mode to Privileged EXEC mode unless an enable password or enable secret has been configured on the device.

An enable secret was therefore configured locally:

```text
enable secret cisco
```

After reconnecting through Telnet:

```text
R0> enable
Password:
R0#
```

The two passwords protect different stages of access.

| Access Stage | Authentication |
|---|---|
| Remote Telnet login | VTY password |
| User EXEC `>` → Privileged EXEC `#` | Enable secret |

---

## SSH Configuration

Telnet confirmed that remote CLI access was working, but SSH is preferred because it encrypts the session.

For SSH, the VTY lines were changed from the simple password method to local username authentication.

A local account was created on both devices:

```text
username cisco secret cisco
```

This creates the local username and password on the device.

The VTY lines were then configured to use that local account:

```text
line vty 0 15
 login local
```

`login local` tells the VTY lines to authenticate remote users using the local username database.

The authentication changes from:

```text
Telnet

password cisco
login
```

to:

```text
SSH

username cisco secret cisco

line vty 0 15
 login local
```

Telnet could also use `login local`, but the simple VTY password was used first to demonstrate basic Telnet authentication before moving to username-based authentication for SSH.

SSH also requires additional configuration.

A hostname and domain name were configured:

```text
hostname R0
ip domain-name <domain-name>
```

SW0 was configured the same way using its own hostname.

RSA keys were then generated:

```text
crypto key generate rsa
```

A modulus of at least 768 bits was selected because the smaller 512-bit key was not sufficient for SSH version 2 in this lab.

SSH version 2 was then configured:

```text
ip ssh version 2
```

The SSH configuration sequence is:

```text
Hostname
   ↓
Domain name
   ↓
RSA keys
   ↓
SSH version 2
```

PC-Admin then used the Telnet/SSH application to connect to R0 and SW0 using SSH.

```text
Username: cisco
Password: cisco
```

![SSH Access](./images/02-ssh-access.png)

Both connections succeeded.

At this point, SSH was working, but Telnet had not yet been explicitly blocked.

---

## VTY Hardening

After SSH was verified, the VTY lines were hardened in two ways:

1. Only SSH was allowed as the remote-management protocol.
2. Only PC-Admin was allowed to access the VTY lines.

### SSH-Only Access

The VTY lines were restricted to SSH.

```text
line vty 0 15
 transport input ssh
```

The important VTY settings are now:

```text
login local
transport input ssh
```

These commands control different parts of remote access.

| Setting | Purpose |
|---|---|
| `login local` | Controls how remote users authenticate |
| `transport input ssh` | Controls which remote-access protocol is allowed |

### VTY ACL

Allowing only SSH still does not control which devices are allowed to attempt remote management.

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

This differs from an ACL used to filter normal routed traffic.

An interface ACL uses `ip access-group` under an interface, while a VTY ACL uses `access-class` under the VTY lines because the goal is specifically to control remote-management access.

The final VTY controls are:

| Configuration | Purpose |
|---|---|
| `login local` | Authenticate using the local user database |
| `transport input ssh` | Allow only SSH |
| `access-class VTY-MANAGEMENT in` | Allow only approved source addresses |

---

# Verification

## SSH Status

SSH status was verified on both devices.

```text
show ip ssh
```

![SSH Status](./images/03-ssh-status.png)

The output confirms that SSH is active and version 2 is being used.

---

## SSH Access After Hardening

PC-Admin was used to connect to both devices through SSH again.

![Successful SSH Login](./images/04-successful-ssh-login.png)

Both connections succeeded.

This confirms that legitimate SSH management still works after the VTY lines are hardened.

---

## Telnet Blocking Test

PC-Admin attempted to connect to both devices using Telnet.

![Telnet Blocked](./images/05-telnet-blocked.png)

Telnet failed while ping and SSH continued to work.

```text
Ping    → Works
SSH     → Works
Telnet  → Fails
```

This confirms that `transport input ssh` is specifically preventing Telnet rather than a connectivity problem.

---

## Authorized Management Test

PC-Admin at `192.168.10.11` was used to connect through SSH after the VTY ACL was applied.

![Authorized SSH Access](./images/06-authorized-ssh-access.png)

SSH succeeded because PC-Admin is permitted by:

```text
permit host 192.168.10.11
```

---

## Unauthorized Management Test

PC-Test at `192.168.10.12` attempted to connect to R0 and SW0 using SSH.

![Unauthorized SSH Blocked](./images/07-unauthorized-ssh-blocked.png)

The SSH connections failed.

PC-Test could still ping both devices.

```text
PC-Test ping → Works
PC-Test SSH  → Fails
```

This confirms that the VTY ACL blocks remote management from PC-Test without blocking normal network connectivity.

---

## Failed Authentication Test

PC-Admin was used to attempt an SSH connection with incorrect credentials.

![Failed Authentication](./images/08-failed-authentication.png)

The connection was allowed because PC-Admin is an authorized source and SSH is permitted.

The login was then rejected because authentication failed.

This confirms that source authorization and user authentication are separate checks.

---

## Active SSH Session

While connected through SSH, the active session was verified.

```text
show users
```

or:

```text
show ssh
```

![Active SSH Session](./images/09-active-ssh-session.png)

The output confirms that an SSH management session is active.

---

## Final VTY Configuration

The final VTY configuration was reviewed.

```text
show running-config | section line vty
```

The important settings are:

```text
line vty 0 15
 login local
 transport input ssh
 access-class VTY-MANAGEMENT in
```

The completed remote-access process is:

```text
IP connectivity
      ↓
Source permitted by VTY ACL
      ↓
SSH permitted by transport policy
      ↓
Local username/password accepted
      ↓
User EXEC mode >
      ↓
enable + enable secret
      ↓
Privileged EXEC mode #
```

---

## Final Remote Management Policy

| Test | Expected Result | Reason |
|---|---|---|
| PC-Admin ping | Allowed | Normal connectivity |
| PC-Test ping | Allowed | VTY ACL does not filter normal traffic |
| PC-Admin SSH | Allowed | Authorized source using SSH |
| PC-Admin Telnet | Blocked | VTY lines accept SSH only |
| PC-Test SSH | Blocked | Source is denied by VTY ACL |
| PC-Admin SSH with wrong credentials | Blocked | Authentication fails |
| PC-Admin SSH with correct credentials | Allowed | All remote-access checks succeed |

---

## Key Takeaways

- Cisco IOS uses VTY lines for remote CLI sessions
- Telnet and SSH can use the same VTY lines
- Telnet requires the VTY lines to have an authentication method
- `password` with `login` provides simple VTY password authentication
- `username` creates a local user account
- `login local` tells the VTY lines to use the local user database
- Telnet can use either VTY password authentication or local authentication
- SSH uses local username authentication in this lab
- A remote VTY user needs an enable password or enable secret to enter Privileged EXEC mode
- A directly connected console session can enter Privileged EXEC with `enable` when no enable secret has been configured
- Telnet provides unencrypted remote access
- SSH provides encrypted remote access
- SSH requires a hostname, domain name, RSA keys, and SSH configuration
- `transport input` controls which remote-access protocols are accepted
- `transport input ssh` restricts the VTY lines to SSH only
- `access-class` applies an ACL directly to VTY access
- A VTY ACL can restrict remote management without blocking normal connectivity
- Connectivity, protocol restrictions, source restrictions, authentication, and privilege level are separate parts of remote management

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
