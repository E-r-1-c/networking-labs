# Secure Remote Management

## Overview

This lab demonstrates how Cisco network devices can be managed remotely and how that access can be secured.

The lab begins with basic Telnet access using a password configured directly on the VTY lines. It then transitions to SSH, which uses a local username and password and encrypts the remote session. Finally, the VTY lines are restricted to SSH only and an ACL is applied directly to the VTY lines so only the administrator PC can remotely manage the devices.

---

## Objectives

- Configure basic connectivity for remote management
- Configure basic Telnet access
- Understand VTY password authentication
- Configure Privileged EXEC access
- Create a local user account for SSH
- Configure SSH using RSA keys and SSH version 2
- Understand the difference between `login` and `login local`
- Restrict the VTY lines to SSH only
- Verify Telnet is blocked
- Restrict remote management by source IP using a VTY ACL
- Test authorized and unauthorized management access
- Verify failed authentication
- Verify the final remote-management configuration

---

## Topology

![Network Topology](./images/topology.png)

The topology uses one router, one switch, and two PCs on the same network.

PC-Admin is used for legitimate Telnet and SSH management. PC-Test is used later to verify that the VTY ACL can block unauthorized remote management without blocking normal network connectivity.

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

Before configuring remote access, basic connectivity was verified.

From PC-Admin:

```text
ping 192.168.10.1
ping 192.168.10.10
```

PC-Test was also able to reach R0 and SW0.

Successful pings confirmed that normal IP connectivity was working before Telnet or SSH was configured.

This gives the lab a baseline. If a remote-management connection fails later while ping still works, the failure is related to remote access rather than basic connectivity.

---

# Telnet

## VTY Lines

Cisco IOS uses VTY lines for remote CLI sessions.

The VTY lines were selected on both R0 and SW0:

```text
line vty 0 15
```

`line vty 0 15` selects VTY lines 0 through 15 so their remote-access settings can be configured.

---

## Telnet Authentication

For the first Telnet test, a password was configured directly on the VTY lines.

```text
line vty 0 15
 password cisco
 login
```

`password cisco` configures the password stored on the VTY lines.

`login` tells the VTY lines to require that password when someone connects remotely.

No username is required with this authentication method.

The relationship is:

```text
password cisco
      ↓
Creates the VTY password

login
      ↓
Tells the VTY lines to require it
```

Telnet does not specifically require a username and password.

It only requires the VTY lines to have an authentication method.

At this stage, the VTY password provides that authentication.

---

## VTY Transport Behavior

Authentication and transport are separate settings.

Authentication controls how a remote user proves they are allowed to log in.

Transport controls which remote-access protocols the VTY lines accept.

The transport setting is controlled with:

```text
transport input
```

For example:

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

No explicit `transport input` command was configured during the initial Telnet test.

On the Packet Tracer devices used in this lab, Telnet was already accepted by the existing VTY transport settings.

The transport policy is explicitly restricted later after SSH is working.

---

## Telnet Remote Access

PC-Admin was used to test Telnet through Packet Tracer's Telnet/SSH application.

Telnet connections were made to:

```text
R0
192.168.10.1
```

and:

```text
SW0
192.168.10.10
```

The VTY password was entered when prompted:

```text
Password: cisco
```

![Telnet Access](./images/01-telnet-access.png)

Both Telnet connections succeeded.

This confirmed that:

- IP connectivity was working
- The VTY lines were accepting Telnet
- The VTY password was configured correctly
- `login` was requiring that password
- Remote CLI access was working

Telnet provides remote CLI access, but its traffic is not encrypted.

---

# Privileged EXEC Access

After connecting through Telnet, the session entered User EXEC mode.

```text
R0>
```

The VTY password allowed access to the remote CLI, but it did not provide Privileged EXEC access.

The remote session was exited and the device console was used to configure an enable secret.

```text
enable secret cisco
```

After reconnecting through Telnet:

```text
R0> enable
Password:
R0#
```

These passwords protect different stages of access.

| Access Stage | Authentication |
|---|---|
| Remote Telnet login | VTY password |
| User EXEC `>` → Privileged EXEC `#` | Enable secret |

The VTY password controls entry into the remote session.

The enable secret separately controls access to Privileged EXEC mode.

---

# Transition from Telnet to SSH

Telnet proved that remote CLI access was working, but Telnet does not encrypt the session.

SSH is now configured to replace Telnet with encrypted remote management.

This also changes how users authenticate.

For Telnet, this lab used:

```text
password cisco
login
```

SSH will instead use a local username and password through:

```text
username cisco secret cisco
login local
```

The VTY lines remain the same.

What changes is their authentication method and the remote-access protocol being used.

---

# SSH

## Local User Account

A local user account was created on both R0 and SW0.

```text
username cisco secret cisco
```

For this lab:

```text
Username: cisco
Password: cisco
```

This command only creates the local account on the device.

It does not automatically tell the VTY lines to use it.

That happens with `login local`.

---

## Change VTY Authentication

The VTY lines were changed from the simple VTY password method to local username authentication.

```text
line vty 0 15
 login local
```

`login local` tells the VTY lines to authenticate users against the device's local username database.

The important change is:

```text
Telnet stage

password cisco
login
```

becomes:

```text
SSH stage

username cisco secret cisco

line vty 0 15
 login local
```

The original VTY password is no longer the authentication method once `login local` is used.

The local username and password are now used instead.

This authentication method also works with Telnet, but it is being introduced here because SSH needs username-based authentication in this lab.

---

## Telnet vs SSH Authentication

The difference is easier to see side by side.

| Telnet Stage | SSH Stage |
|---|---|
| VTY password | Local username and password |
| `password cisco` | `username cisco secret cisco` |
| `login` | `login local` |
| No username required | Username required |
| Unencrypted session | Encrypted session |

Telnet could also use `login local`.

The purpose of using `login` first is to demonstrate the simpler VTY password method before moving to the local-account authentication used with SSH.

---

## SSH Preparation

SSH requires additional configuration before it can operate.

A hostname was configured on each device.

R0:

```text
hostname R0
```

SW0:

```text
hostname SW0
```

A domain name was also configured.

```text
ip domain-name <domain-name>
```

The hostname and domain name provide information IOS uses during RSA key generation.

---

## RSA Key Generation

RSA keys were generated on both devices.

```text
crypto key generate rsa
```

A modulus of at least 768 bits was selected.

The smaller 512-bit value was not sufficient for SSH version 2 in this lab.

The RSA key pair provides the cryptographic keys used by the SSH server.

---

## SSH Version 2

After the RSA keys were generated, SSH version 2 was explicitly configured.

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

The RSA key pair is generated first.

`ip ssh version 2` then tells the device to use SSH version 2.

---

## SSH Remote Access

PC-Admin was used to test SSH through Packet Tracer's Telnet/SSH application.

SSH connections were made to:

```text
R0
192.168.10.1
```

and:

```text
SW0
192.168.10.10
```

The local account was used:

```text
Username: cisco
Password: cisco
```

![SSH Access](./images/02-ssh-access.png)

Both SSH connections succeeded.

This confirmed that:

- The devices were reachable
- The local username existed
- `login local` was using the local user database
- RSA keys were available
- SSH version 2 was active
- The VTY lines were accepting SSH

At this point, SSH works.

Telnet has not yet been explicitly blocked.

---

# VTY Transport Hardening

Now that SSH is confirmed to work, Telnet can be removed from the VTY lines.

```text
line vty 0 15
 transport input ssh
```

`transport input` controls which incoming remote-access protocols can use the VTY lines.

Before this command, the Packet Tracer devices used in this lab accepted Telnet and later accepted SSH once SSH was configured.

After:

```text
transport input ssh
```

only SSH is accepted.

The important VTY configuration is now:

```text
line vty 0 15
 login local
 transport input ssh
```

These commands perform different jobs.

| Command | Purpose |
|---|---|
| `login local` | Use the local username database for authentication |
| `transport input ssh` | Allow only SSH as the remote-access protocol |

One controls authentication.

The other controls transport.

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

PC-Admin was used to test SSH again through the Telnet/SSH application.

SSH access to both R0 and SW0 remained successful.

![Successful SSH Login](./images/04-successful-ssh-login.png)

This confirms that restricting the VTY lines to SSH did not break legitimate remote management.

---

## Telnet Blocking Test

PC-Admin was then switched back to Telnet in the Telnet/SSH application.

Connections were attempted to R0 and SW0.

![Telnet Blocked](./images/05-telnet-blocked.png)

Both Telnet connections failed.

Ping still succeeded.

The result was:

```text
Ping    → Works
SSH     → Works
Telnet  → Fails
```

This confirms that the devices are still reachable.

Telnet is specifically being rejected because the VTY lines contain:

```text
transport input ssh
```

---

# VTY ACL Hardening

Restricting the VTY lines to SSH controls which protocol may be used.

It still does not control which source devices are allowed to attempt remote management.

A VTY ACL was added for that purpose.

PC-Admin:

```text
192.168.10.11
```

is the authorized management workstation.

PC-Test:

```text
192.168.10.12
```

is used to test unauthorized access.

---

## Create the Management ACL

A standard ACL was created on both R0 and SW0.

```text
ip access-list standard VTY-MANAGEMENT
 permit host 192.168.10.11
```

Only PC-Admin is explicitly permitted.

The ACL was then applied directly to the VTY lines.

```text
line vty 0 15
 access-class VTY-MANAGEMENT in
```

This differs from an ACL used to filter normal routed traffic.

An interface ACL is applied using:

```text
ip access-group
```

under an interface.

This management ACL is applied using:

```text
access-class
```

under the VTY lines.

It therefore controls access to the remote-management lines rather than normal traffic passing through the device.

---

## Transport Restriction vs VTY ACL

These controls solve different problems.

| Configuration | Controls |
|---|---|
| `transport input ssh` | Which remote-management protocol is accepted |
| `access-class VTY-MANAGEMENT in` | Which source IP addresses may access the VTY lines |

The remote-management process now looks like:

```text
Source device
      ↓
VTY ACL
      ↓
Source allowed?
      ↓
SSH-only transport
      ↓
Local username/password
      ↓
Remote CLI
```

---

## Authorized Management Test

PC-Admin was tested after the VTY ACL was applied.

SSH connections were made to:

```text
192.168.10.1
192.168.10.10
```

![Authorized SSH Access](./images/06-authorized-ssh-access.png)

Both connections succeeded.

PC-Admin is permitted because its source address:

```text
192.168.10.11
```

matches:

```text
permit host 192.168.10.11
```

---

## Unauthorized Management Test

PC-Test was then used to attempt SSH access.

Its address is:

```text
192.168.10.12
```

SSH connections were attempted to both R0 and SW0.

![Unauthorized SSH Blocked](./images/07-unauthorized-ssh-blocked.png)

The SSH connections failed.

PC-Test was then used to ping both devices.

The pings still succeeded.

The result was:

```text
PC-Test ping → Works
PC-Test SSH  → Fails
```

This proves that PC-Test still has normal IP connectivity.

Only its access to the VTY management lines is being denied.

---

## Failed Authentication Test

PC-Admin is an authorized source, so it was used to test failed authentication.

An SSH connection was attempted using incorrect credentials.

![Failed Authentication](./images/08-failed-authentication.png)

The connection was allowed to reach the device because PC-Admin is permitted by the VTY ACL and SSH is an allowed protocol.

The login was then rejected because the username or password was incorrect.

This shows that source authorization and user authentication are separate checks.

```text
Source allowed
      ↓
SSH allowed
      ↓
Username/password checked
      ↓
Login accepted or rejected
```

---

## Active SSH Session

While a valid SSH session was active, the connection was verified from the network device.

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

Each setting controls a different part of remote management.

| Setting | Purpose |
|---|---|
| `login local` | Authenticate users using the local username database |
| `transport input ssh` | Allow only SSH |
| `access-class VTY-MANAGEMENT in` | Allow VTY access only from approved source addresses |

The local account itself is created separately:

```text
username cisco secret cisco
```

Privileged EXEC access is also protected separately:

```text
enable secret cisco
```

---

## Final Remote Management Policy

| Test | Expected Result | Reason |
|---|---|---|
| PC-Admin ping | Allowed | Normal IP connectivity |
| PC-Test ping | Allowed | VTY ACL does not filter normal traffic |
| PC-Admin SSH | Allowed | Authorized source using allowed protocol |
| PC-Admin Telnet | Blocked | VTY lines accept SSH only |
| PC-Test SSH | Blocked | Source is not permitted by VTY ACL |
| PC-Admin SSH with wrong credentials | Blocked | Local authentication fails |
| PC-Admin SSH with correct credentials | Allowed | Source, protocol, and authentication succeed |
| Valid remote user + correct enable secret | Privileged EXEC allowed | Enable authentication succeeds |

The completed access process is:

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

## Key Takeaways

- Cisco IOS uses VTY lines for remote CLI sessions
- Telnet and SSH can use the same VTY lines
- Telnet only needs the VTY lines to have an authentication method
- `password` plus `login` provides simple VTY password authentication
- This method does not require a username
- A local account is created separately with `username`
- `login local` tells the VTY lines to use the local username database
- In this lab, local authentication is introduced when moving from Telnet to SSH
- Telnet could also use `login local`, but it is not required to use the local database
- SSH uses username-based authentication in this lab
- The same VTY lines are reused when moving from Telnet to SSH
- Telnet provides unencrypted remote access
- SSH provides encrypted remote access
- SSH requires RSA keys
- SSH version 2 was selected after generating an appropriate RSA key pair
- `transport input` controls which remote-management protocols can use the VTY lines
- `transport input telnet` permits only Telnet
- `transport input ssh` permits only SSH
- `transport input telnet ssh` permits both
- Before an explicit transport restriction was configured, the Packet Tracer devices in this lab accepted the available remote-access protocols
- `enable secret` separately protects Privileged EXEC access
- `access-class` applies an ACL directly to the VTY lines
- A VTY ACL can block remote management from a source without blocking its normal network connectivity
- Authentication, transport protocol, source authorization, and privilege level are separate controls

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
