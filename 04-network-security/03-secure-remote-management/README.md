# Secure Remote Management

## Overview

This lab demonstrates how Cisco network devices can be managed remotely through the command line and how that access can be secured.

The lab first uses Telnet with local authentication to verify basic remote CLI access. SSH is then configured using a hostname, domain name, RSA keys, and SSH version 2. Finally, the VTY lines are hardened to accept SSH only, blocking Telnet while keeping secure remote management available.

---

## Objectives

- Configure IP connectivity for remote management
- Configure VTY lines for remote CLI access
- Create a local user account for authentication
- Verify Telnet access
- Configure privileged EXEC access
- Configure SSH version 2
- Generate RSA keys
- Verify SSH access
- Restrict VTY access to SSH only
- Verify Telnet is blocked after hardening
- Test failed authentication
- Verify active SSH sessions

---

## Topology

![Network Topology](./images/topology.png)

The topology uses one router, one switch, and one administrator PC on the same network.

PC-Admin is used to remotely manage both R0 and SW0.

---

## Network Design

| Device | IP Address | Role |
|---|---|---|
| R0 | `192.168.10.1` | Router and remotely managed device |
| SW0 | `192.168.10.10` | Switch management address |
| PC-Admin | `192.168.10.11` | Administrator workstation |

---

## Baseline Connectivity

Before configuring remote access, PC-Admin was used to verify that both devices were reachable.

```text
ping 192.168.10.1
ping 192.168.10.10
```

Successful responses confirmed that IP connectivity was working before Telnet or SSH was tested.

---

## Local User Authentication

A local user account was created on both devices.

```text
username cisco secret cisco
```

This creates a local username and password that can be used to authenticate remote users.

The username and password are:

```text
Username: cisco
Password: cisco
```

This account is used for remote login authentication. It does not automatically provide privileged EXEC access.

---

## VTY Configuration

Cisco IOS uses VTY lines for remote CLI sessions.

The existing VTY lines were selected and configured to use the local user database.

```text
line vty 0 15
 login local
```

`line vty 0 15` selects VTY lines 0 through 15.

`login local` tells those lines to authenticate remote users using the local username database.

At this point, the VTY lines have an authentication method configured and can accept supported remote-access protocols.

---

## Telnet Remote Access

Telnet was tested first as the baseline remote-management method.

No `transport input telnet` command was required because Telnet was already permitted by the default VTY transport settings on the Packet Tracer devices.

From PC-Admin:

```text
telnet 192.168.10.1
```

```text
telnet 192.168.10.10
```

![Telnet Access](./images/01-telnet-access.png)

Both connections succeeded.

This confirmed that:

- PC-Admin could reach both devices
- The VTY lines were accepting remote sessions
- Local authentication was working
- Remote CLI access was available

Telnet provides remote CLI access, but the session is not encrypted.

---

## Privileged EXEC Access

After logging in remotely, the session entered User EXEC mode.

```text
R0>
```

The local username and password authenticated the remote login, but they did not provide privileged EXEC access.

The remote session was exited, and the device console was used to configure an enable secret.

```text
enable secret cisco
```

This command configures the password used when moving from User EXEC mode to Privileged EXEC mode.

Afterward, the remote session could use:

```text
enable
```

and enter the enable secret to reach:

```text
R0#
```

The two authentication steps serve different purposes:

```text
| Access Stage | Credential Used |
|---|---|
| Remote login | Local username and password |
| User EXEC → Privileged EXEC | Enable secret |
```

Even though both were configured with the word `secret`, they are separate.

```text
username cisco secret cisco
```

creates the local account used to log in.

```text
enable secret cisco
```

creates the password used to enter Privileged EXEC mode.

---

## SSH Preparation

SSH requires additional configuration before it can be used.

A hostname was configured on each device.

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

The hostname and domain name are required before RSA keys can be generated.

---

## RSA Key Generation

RSA keys were generated on both devices.

```text
crypto key generate rsa
```

The default 512-bit key size was not large enough for SSH version 2 in Packet Tracer.

A key size of at least 768 bits was used instead.

The RSA keys are used by SSH to establish the encrypted connection.

---

## SSH Version 2

SSH version 2 was enabled on both devices.

```text
ip ssh version 2
```

SSH provides encrypted remote CLI access, unlike Telnet.

The VTY lines were already configured with `login local`, so the same local account could be used for SSH authentication.

There was no need to configure the VTY authentication again.

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

- IP connectivity was working
- Local authentication was working
- RSA keys had been generated
- SSH version 2 was active
- The VTY lines were accepting SSH sessions

---

## VTY Hardening

After SSH was verified, the VTY lines were restricted so that only SSH could be used for remote access.

```text
line vty 0 15
 transport input ssh
```

`transport input` controls which remote-access protocols are allowed to use the VTY lines.

Earlier in the lab, no explicit Telnet transport command was needed because Telnet was already allowed.

Now:

```text
transport input ssh
```

changes the VTY transport policy so only SSH is accepted.

The authentication method remains unchanged.

The final VTY behavior is:

```text
login local
transport input ssh
```

This means:

- Remote users authenticate with the local username database
- SSH is allowed
- Telnet is blocked

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

SSH was tested again after the VTY lines were restricted to SSH only.

```text
ssh -l cisco 192.168.10.1
```

```text
ssh -l cisco 192.168.10.10
```

![Successful SSH Login](./images/04-successful-ssh-login.png)

Both connections remained successful.

This confirms that secure remote management still works after Telnet is removed.

---

## Telnet Blocking Test

Telnet was tested again after the VTY lines were restricted to SSH.

```text
telnet 192.168.10.1
```

```text
telnet 192.168.10.10
```

![Telnet Blocked](./images/05-telnet-blocked.png)

The devices remained reachable by IP, but Telnet connections failed.

This confirms that Telnet was being blocked by the VTY transport policy rather than by a connectivity problem.

---

## Failed Authentication Test

An SSH connection was attempted using incorrect login credentials.

![Failed Authentication](./images/06-failed-authentication.png)

The connection reached the device, but the login was rejected.

This confirms that connectivity and authentication are separate parts of remote access.

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

This confirms that the VTY lines use local authentication and accept SSH only.

---

## Authentication and Access Controls

| Configuration | Purpose |
|---|---|
| `username cisco secret cisco` | Creates the local user used for remote login |
| `login local` | Tells the VTY lines to authenticate against the local user database |
| `enable secret cisco` | Configures the password used to enter Privileged EXEC mode |
| `transport input ssh` | Restricts the VTY lines so only SSH is accepted |

These settings control different parts of remote access.

The local account controls who can log in.

The enable secret controls whether that user can move from User EXEC mode to Privileged EXEC mode.

The transport setting controls which remote-access protocol is allowed.

---

## ACL Hardening

An ACL is not required to make SSH work or to disable Telnet.

`transport input ssh` controls the protocol allowed on the VTY lines.

An ACL can be added separately if remote management should also be limited to specific source addresses.

For example, the final policy could be designed so that only the administrator PC is allowed to remotely manage the devices.

That would provide two separate controls:

```text
SSH-only transport
        +
Allowed management source
```

The transport setting controls how the connection is made.

The ACL controls who is allowed to make the connection.

---

## Key Takeaways

- Telnet and SSH both provide remote CLI access
- Cisco IOS uses VTY lines for remote terminal sessions
- `line vty 0 15` selects the VTY lines for configuration
- `login local` tells the VTY lines to authenticate against the local username database
- Telnet was already permitted by the default VTY transport settings in this lab
- `transport input telnet` was therefore not required for the initial Telnet test
- SSH requires additional configuration before it can be used
- A hostname and domain name are required before RSA keys can be generated
- RSA keys provide the cryptographic information used by SSH
- SSH version 2 requires an appropriate RSA key size
- The local username and password authenticate the remote user
- The local account does not automatically provide Privileged EXEC access
- `enable secret` controls access from User EXEC mode to Privileged EXEC mode
- SSH uses the same VTY lines and local authentication already configured earlier
- The VTY lines only need to be entered again when their configuration is being changed
- `transport input ssh` restricts remote access to SSH and blocks Telnet
- ACLs and VTY transport restrictions solve different problems
- A device can remain reachable by IP while rejecting Telnet
- Connectivity, authentication, privilege level, transport protocol, and source restrictions are separate parts of remote management

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
