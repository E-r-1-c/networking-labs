# Secure Remote Management

## Overview

This lab demonstrates how Cisco routers and switches can be managed remotely through the command line and how that access can be secured.

The lab first uses Telnet to verify basic remote CLI access, then configures SSH for encrypted remote management. The VTY lines are hardened to use local authentication and accept SSH only, allowing secure administrative access while blocking Telnet.

---

## Objectives

- Configure remote CLI access to a Cisco router and switch
- Use Telnet as a baseline remote-management method
- Configure local user authentication
- Configure SSH version 2
- Generate RSA keys for SSH
- Understand the purpose of VTY lines
- Restrict remote access to SSH only
- Verify successful SSH access
- Verify Telnet is blocked after hardening
- Test failed SSH authentication
- Verify active remote sessions

---

## Topology

![Network Topology](./images/topology.png)

The topology uses an administrator PC connected through SW0 to R0.

Both R0 and SW0 have reachable IP addresses so the administrator PC can remotely access their command-line interfaces.

---

## Network Design

| Device | Role |
|---|---|
| R0 | Router and remotely managed network device |
| SW0 | Switch and remotely managed network device |
| PC-Admin | Administrator workstation used for remote access |

Basic IP connectivity was configured and verified before Telnet or SSH was enabled.

---

## Baseline Connectivity

Before configuring remote management, the administrator PC was used to verify connectivity to both network devices.

```text
ping <R0-IP>
ping <SW0-management-IP>
```

Successful responses confirmed that the network path and management addressing were working before remote-access configuration began.

---

## Telnet Remote Access

Telnet was configured first to demonstrate basic remote CLI access.

The VTY lines are virtual terminal lines used by Cisco IOS for remote command-line sessions.

```text
line vty 0 15
```

A password was configured for the VTY lines and login authentication was enabled.

The administrator PC then connected remotely to both devices using Telnet.

```text
telnet <R0-IP>
telnet <SW0-management-IP>
```

![Telnet Access](./images/01-telnet-access.png)

Successful connections confirmed that both devices could be managed remotely.

Telnet provides remote CLI access, but the session is not encrypted, making it unsuitable for secure administrative access.

---

## Local User Authentication

A local administrator account was created on each device.

```text
username cisco secret <password>
```

The local username database is separate from the password used to enter privileged EXEC mode.

When `login local` is configured on the VTY lines, the device authenticates remote users using the locally configured username and password.

```text
login local
```

---

## Privileged EXEC Access

Remote login authentication and privileged EXEC access are separate.

A successful remote login can place the administrator in User EXEC mode:

```text
R0>
```

The `enable` command is then used to request Privileged EXEC mode.

```text
R0> enable
```

The configured enable secret controls access to:

```text
R0#
```

The local username authenticates the remote user, while the enable secret controls access to privileged EXEC mode.

---

## SSH Preparation

Before SSH could be enabled, each device was given a hostname and domain name.

```text
hostname <device-name>
ip domain-name <domain-name>
```

The hostname and domain information are used as part of the RSA key-generation process.

A local user account was also configured so SSH connections could use username and password authentication.

---

## RSA Key Generation

SSH requires cryptographic keys.

RSA keys were generated on both network devices.

```text
crypto key generate rsa
```

A key size large enough to support SSH version 2 was selected.

The default 512-bit key was not sufficient for SSH version 2 in this Packet Tracer environment, so a larger key was generated.

---

## SSH Version 2

SSH version 2 was enabled on both devices.

```text
ip ssh version 2
```

SSH provides encrypted remote management instead of the unencrypted connection used by Telnet.

---

## VTY Authentication

The VTY lines were configured to use the local username database.

```text
line vty 0 15
 login local
```

`login local` tells the device to authenticate remote users using the usernames and secrets configured locally on the device.

This is different from `login`, which uses a password configured directly on the VTY line.

---

## SSH Remote Access

After SSH was configured, the administrator PC connected remotely to both devices.

```text
ssh -l <username> <R0-IP>
```

```text
ssh -l <username> <SW0-management-IP>
```

![SSH Access](./images/02-ssh-access.png)

Successful authentication confirmed that:

- IP connectivity was working
- SSH was enabled
- RSA keys were available
- The VTY lines accepted the connection
- Local authentication was working

---

## VTY Hardening

After SSH was verified, the VTY lines were restricted so that SSH was the only permitted remote-access protocol.

```text
line vty 0 15
 transport input ssh
```

This keeps remote CLI management available while preventing Telnet connections.

---

# Verification

## SSH Status

SSH configuration was verified on each device.

```text
show ip ssh
```

![SSH Status](./images/03-ssh-status.png)

The output confirms that SSH is enabled and version 2 is being used.

---

## SSH Access Verification

The administrator PC successfully connected to R0 and SW0 using SSH.

```text
ssh -l <username> <device-IP>
```

![Successful SSH Login](./images/04-successful-ssh-login.png)

This confirms that secure remote CLI access is working.

---

## Telnet Blocking Test

After the VTY lines were restricted to SSH, another Telnet connection was attempted.

```text
telnet <device-IP>
```

![Telnet Blocked](./images/05-telnet-blocked.png)

The connection failed even though the device was still reachable by IP.

This confirms that the failure was caused by the remote-access policy rather than a network connectivity problem.

---

## Failed Authentication Test

An SSH connection was attempted using incorrect login information.

![Failed Authentication](./images/06-failed-authentication.png)

The device rejected the authentication attempt.

This shows that reaching the SSH service does not automatically provide access to the device.

---

## Active Session Verification

While connected through SSH, the active remote session was verified from the network device.

```text
show users
```

or:

```text
show ssh
```

![Active SSH Session](./images/07-active-ssh-session.png)

This confirms that the SSH session is active on the device.

---

## VTY Configuration Verification

The final VTY configuration was reviewed to verify the remote-access policy.

```text
show running-config | section line vty
```

The VTY lines should show:

```text
login local
transport input ssh
```

This confirms that local authentication is being used and only SSH is accepted for remote CLI access.

---

## Authentication Differences

Several different authentication settings are used in this lab, and each serves a different purpose.

| Configuration | Purpose |
|---|---|
| `login` | Uses the password configured directly on the VTY line |
| `login local` | Uses usernames and passwords stored locally on the device |
| Local username | Identifies and authenticates the remote user |
| Enable secret | Controls access from User EXEC mode to Privileged EXEC mode |

A remote user can successfully authenticate to the device but still require the enable secret before entering privileged EXEC mode.

---

## Key Takeaways

- Telnet and SSH both provide remote CLI access to network devices
- VTY lines are used to control remote terminal sessions
- A device must have working IP connectivity before remote management can work
- Telnet provides remote access but does not encrypt the session
- SSH provides encrypted remote administrative access
- SSH requires a hostname, domain information, RSA keys, and authentication
- SSH version 2 requires an appropriate RSA key size
- `login` and `login local` use different authentication methods
- A local username authenticates the remote user
- The enable secret controls access to privileged EXEC mode
- `transport input ssh` prevents Telnet while keeping SSH available
- A device can respond to ping while still rejecting Telnet or SSH
- Connectivity failures, protocol failures, and authentication failures are separate problems
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
