# Secure Remote Management

## Overview

This lab demonstrates how Cisco network devices can be managed remotely through the command line and how that access can be secured.

The lab first configures the VTY lines and uses Telnet to verify basic remote CLI access. SSH is then configured using local authentication and RSA keys, and the VTY lines are hardened to allow SSH only so Telnet is blocked.

---

## Objectives

- Configure IP connectivity for remote management
- Configure VTY lines for remote CLI access
- Use Telnet as a baseline remote-management method
- Create a local user account for authentication
- Configure SSH version 2
- Generate RSA keys for SSH
- Restrict VTY access to SSH only
- Verify successful SSH access
- Verify Telnet is blocked after hardening
- Test failed SSH authentication
- Verify active remote sessions

---

## Topology

![Network Topology](./images/topology.png)

The topology uses a router, switch, and administrator PC on the same network.

The administrator PC is used to remotely access both R0 and SW0 through their IP addresses.

---

## Network Design

| Device | IP Address | Role |
|---|---|---|
| R0 | `192.168.10.1` | Router and remotely managed device |
| SW0 | `192.168.10.10` | Switch management address |
| PC-Admin | `192.168.10.11` | Administrator workstation |

---

## Baseline Connectivity

Before configuring remote access, IP connectivity was verified from PC-Admin.

```text
ping 192.168.10.1
ping 192.168.10.10
```

Successful responses confirmed that both network devices were reachable before Telnet or SSH was configured.

---

## VTY Configuration for Remote Access

Cisco IOS already provides VTY lines for remote terminal sessions.

The existing VTY lines were selected so their authentication and remote-access settings could be configured.

```text
line vty 0 15
```

This command does not create the VTY lines. It selects VTY lines 0 through 15 for configuration.

Authentication was then configured on the VTY lines before remote access was tested.

---

## Telnet Remote Access

Telnet was used first to verify that remote CLI access was working.

The VTY lines were configured with the required login settings, and Telnet was allowed as a remote-access method.

From PC-Admin, connections were made to both devices:

```text
telnet 192.168.10.1
telnet 192.168.10.10
```

![Telnet Access](./images/01-telnet-access.png)

Successful connections confirmed that:

- IP connectivity was working
- The VTY lines were available for remote sessions
- Authentication was working
- The router and switch could be managed remotely

Telnet provides remote CLI access, but the session is not encrypted.

---

## Local User Authentication

A local user account was created on each device.

```text
username cisco secret cisco
```

The local user account is used when the VTY lines are configured with:

```text
login local
```

`login local` tells the device to authenticate remote users using usernames and passwords stored in the local user database.

This is different from:

```text
login
```

which uses a password configured directly on the VTY line.

---

## Privileged EXEC Access

Remote login authentication and privileged EXEC access are separate.

After successfully logging in remotely, the session can begin in User EXEC mode:

```text
R0>
```

The `enable` command is then used to request privileged EXEC access.

```text
R0> enable
```

The configured enable secret is used to enter:

```text
R0#
```

The local username and password authenticate the remote user, while the enable secret controls access to privileged EXEC mode.

---

## SSH Preparation

SSH requires additional configuration that Telnet does not.

Each device was configured with a hostname and domain name.

```text
hostname <device-name>
ip domain-name <domain-name>
```

A local user account was already available for SSH authentication.

The hostname and domain name are also required before RSA keys can be generated.

---

## RSA Key Generation

RSA keys were generated on both devices.

```text
crypto key generate rsa
```

The initial default key size of 512 bits was not large enough for SSH version 2 in Packet Tracer.

A larger RSA key size was generated so SSH version 2 could be used.

---

## SSH Version 2

SSH version 2 was enabled on both devices.

```text
ip ssh version 2
```

SSH provides encrypted remote CLI access, unlike Telnet.

---

## SSH Authentication

The VTY lines were configured to use the local username database.

```text
line vty 0 15
 login local
```

This requires remote users to authenticate with the locally configured username and password.

---

## SSH Remote Access

SSH was tested from PC-Admin against both devices.

```text
ssh -l cisco 192.168.10.1
```

```text
ssh -l cisco 192.168.10.10
```

![SSH Access](./images/02-ssh-access.png)

Successful connections confirmed that:

- The devices were reachable
- SSH was enabled
- RSA keys were available
- The VTY lines accepted SSH
- Local authentication was working

---

## VTY Hardening

After SSH was verified, the VTY lines were restricted to accept SSH only.

```text
line vty 0 15
 transport input ssh
```

This removes Telnet as an accepted remote-access method while keeping SSH available.

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

## Successful SSH Access

SSH connections were tested again after the VTY lines were hardened.

```text
ssh -l cisco 192.168.10.1
```

```text
ssh -l cisco 192.168.10.10
```

![Successful SSH Login](./images/04-successful-ssh-login.png)

Both devices remained remotely accessible through SSH.

---

## Telnet Blocking Test

After the VTY lines were restricted to SSH, Telnet was tested again.

```text
telnet 192.168.10.1
```

```text
telnet 192.168.10.10
```

![Telnet Blocked](./images/05-telnet-blocked.png)

The Telnet connections failed even though both devices were still reachable by IP.

This confirms that the failure was caused by the VTY remote-access policy rather than a network connectivity problem.

---

## Failed Authentication Test

An SSH connection was attempted using incorrect login information.

![Failed Authentication](./images/06-failed-authentication.png)

The device rejected the login attempt.

This confirms that reaching the SSH service does not automatically provide administrative access.

---

## Active Session Verification

While connected through SSH, the remote session was verified from the network device.

```text
show users
```

or:

```text
show ssh
```

![Active SSH Session](./images/07-active-ssh-session.png)

The output confirms that an SSH session is active.

---

## VTY Configuration Verification

The final VTY configuration was reviewed.

```text
show running-config | section line vty
```

The important settings are:

```text
login local
transport input ssh
```

This confirms that remote users authenticate through the local user database and that SSH is the only accepted remote-access protocol.

---

## Authentication Differences

The authentication settings used in this lab serve different purposes.

| Configuration | Purpose |
|---|---|
| `login` | Uses the password configured directly on the VTY line |
| `login local` | Uses usernames and passwords stored locally on the device |
| Local username | Authenticates the remote user |
| Enable secret | Controls access from User EXEC mode to Privileged EXEC mode |

Remote login and privileged EXEC access are separate authentication steps.

---

## Key Takeaways

- Telnet and SSH both provide remote CLI access to network devices
- Cisco IOS already contains VTY lines for remote terminal sessions
- `line vty 0 15` selects existing VTY lines for configuration
- A device must have working IP connectivity before remote management can work
- VTY authentication must be configured before remote access can be used
- Telnet provides remote access but does not encrypt the session
- SSH provides encrypted remote administrative access
- SSH requires a hostname, domain name, RSA keys, and authentication
- SSH version 2 requires an RSA key large enough to support it
- `login` and `login local` use different authentication methods
- A local username authenticates the remote user
- The enable secret controls access to privileged EXEC mode
- `transport input ssh` prevents Telnet while keeping SSH available
- A device can still respond to ping while rejecting Telnet or SSH
- Connectivity, protocol configuration, and authentication are separate parts of remote access
- Remote management should be verified from both the client and network-device sides

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
