# Secure Remote Management

## Overview

This lab demonstrates how Cisco network devices can be managed remotely and how that management access can be secured.

The lab begins by using Telnet to verify basic remote CLI access with local authentication. SSH is then configured to provide encrypted remote access. Finally, the VTY lines are hardened to allow only SSH, and a VTY ACL is added so only the administrator PC can remotely manage the devices.

---

## Objectives

- Configure basic connectivity for remote management
- Configure VTY authentication
- Test remote access using Telnet
- Configure Privileged EXEC access
- Configure SSH using RSA keys and SSH version 2
- Test remote access using SSH
- Restrict VTY lines to SSH only
- Verify Telnet is blocked
- Restrict VTY access to an authorized management PC
- Verify unauthorized management access is blocked
- Test failed authentication
- Verify the final VTY configuration

---

## Topology

![Network Topology](./images/topology.png)

The topology uses one router, one switch, and two PCs on the same network.

PC-Admin is the authorized management workstation used for Telnet and SSH access. PC-Test is added to verify that the VTY ACL can block unauthorized remote management while normal network connectivity still works.

---

## Network Design

| Device | IP Address | Role |
|---|---|---|
| R0 | `192.168.10.1` | Router and remotely managed device |
| SW0 | `192.168.10.10` | Switch management address |
| PC-Admin | `192.168.10.11` | Authorized management workstation |
| PC-Test | `192.168.10.12` | Unauthorized workstation used for ACL testing |

---

## Baseline Connectivity

Before configuring remote access, connectivity was verified between the PCs and the network devices.

From PC-Admin:

```text
ping 192.168.10.1
ping 192.168.10.10
```

PC-Test was also able to reach R0 and SW0.

Successful pings confirmed that normal IP connectivity was working before remote-management configuration began.

This provides a baseline for later testing. If Telnet or SSH fails while ping still works, the problem is related to remote access rather than basic connectivity.

---

# Telnet Configuration

## Local User Authentication

A local user account was created on both R0 and SW0.

```text
username cisco secret cisco
```

For this lab:

```text
Username: cisco
Password: cisco
```

This creates an account in the device's local user database.

The account is not something Telnet specifically requires. Telnet requires the VTY lines to have an authentication method.

For example, VTY authentication could use a password configured directly on the VTY lines, or it can use the device's local username database.

This lab uses the local database because the same account can later also be used for SSH.

---

## VTY Authentication

Cisco IOS uses VTY lines for remote terminal sessions.

The VTY lines were selected and configured to authenticate against the local user database.

```text
line vty 0 15
 login local
```

`line vty 0 15` selects VTY lines 0 through 15.

`login local` tells those lines to use the usernames and passwords stored in the local user database.

The relationship is:

```text
username cisco secret cisco
        ↓
Creates local account

login local
        ↓
Tells VTY lines to use that account
```

At this point, an authentication method exists for remote CLI sessions.

---

## VTY Transport Behavior

The VTY lines also control which remote-management protocols are accepted.

This is controlled with:

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
transport input ssh telnet
```

allows both SSH and Telnet.

No explicit `transport input` command was configured during the initial part of this lab.

On the Packet Tracer devices used here, Telnet was already accepted by the existing VTY transport settings. Later, after SSH was configured, SSH was also accepted without changing the VTY transport setting.

The protocol is explicitly restricted later in the lab.

---

## Telnet Remote Access

Telnet was tested first to establish a working remote-management baseline.

On PC-Admin, the Packet Tracer Telnet/SSH application was opened and Telnet was selected.

Connections were tested to:

```text
R0
192.168.10.1
```

and:

```text
SW0
192.168.10.10
```

The local account was used to authenticate:

```text
Username: cisco
Password: cisco
```

![Telnet Access](./images/01-telnet-access.png)

Both Telnet connections succeeded.

This confirmed that:

- IP connectivity was working
- The VTY lines were accepting Telnet
- `login local` was using the local user database
- The local username and password were valid
- Remote CLI access was working

Telnet provides remote CLI access, but the traffic is not encrypted.

---

# Privileged EXEC Access

After logging in remotely, the session entered User EXEC mode.

```text
R0>
```

The local username and password allowed the remote login, but they did not automatically provide Privileged EXEC access.

The remote session was exited and the device console was used to configure an enable secret.

```text
enable secret cisco
```

After reconnecting remotely, the `enable` command could be used.

```text
R0> enable
Password:
R0#
```

The two credentials protect different stages of access:

| Access Stage | Credential Used |
|---|---|
| Remote login | Local username and password |
| User EXEC `>` → Privileged EXEC `#` | Enable secret |

The local account:

```text
username cisco secret cisco
```

allows the user to authenticate to the VTY session because the VTY lines use:

```text
login local
```

The separate command:

```text
enable secret cisco
```

protects access to Privileged EXEC mode.

---

# SSH Configuration

## SSH Preparation

SSH requires additional configuration that Telnet does not.

A hostname was configured on each network device.

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

The hostname and domain name provide the information IOS uses when generating the RSA key pair.

---

## RSA Key Generation

RSA keys were generated on both devices.

```text
crypto key generate rsa
```

A key size of at least 768 bits was selected.

The smaller 512-bit value was not sufficient for SSH version 2 in this lab.

The RSA key pair provides the cryptographic keys used by the SSH server.

Generating the RSA keys is also what makes the SSH service available on the device.

---

## SSH Version 2

After generating the RSA keys, SSH version 2 was explicitly selected.

```text
ip ssh version 2
```

The configuration order used in this lab is:

```text
Hostname
   ↓
Domain name
   ↓
RSA keys
   ↓
SSH version 2
```

The RSA keys are created first so the device has the cryptographic keys required by SSH.

`ip ssh version 2` then tells the device to use SSH version 2.

---

## SSH Authentication

No new user account was needed for SSH.

The VTY lines were already configured with:

```text
login local
```

and the device already had:

```text
username cisco secret cisco
```

SSH therefore uses the same local account that was previously used during the Telnet test.

The important difference is not the account.

The difference is the transport protocol:

| Telnet | SSH |
|---|---|
| Remote CLI access | Remote CLI access |
| Uses VTY lines | Uses VTY lines |
| Uses local authentication in this lab | Uses local authentication in this lab |
| Unencrypted | Encrypted |
| Did not require RSA keys | Requires RSA keys |
| No SSH version configuration | SSH version 2 configured |

The local username/password handles authentication for both protocols in this lab.

SSH adds encryption and its own SSH configuration.

---

## SSH Remote Access

SSH was tested using the Telnet/SSH application on PC-Admin.

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

The same local account was used:

```text
Username: cisco
Password: cisco
```

![SSH Access](./images/02-ssh-access.png)

Both connections succeeded.

This confirmed that:

- IP connectivity was working
- The local account was working
- VTY authentication was working
- RSA keys were available
- SSH version 2 was active
- The VTY lines were accepting SSH

At this point, SSH worked, but Telnet had not yet been explicitly blocked.

---

# VTY Transport Hardening

After SSH was confirmed to work, the VTY lines were restricted to SSH.

```text
line vty 0 15
 transport input ssh
```

`transport input` controls which incoming remote-management protocols can use the VTY lines.

Before this command, the Packet Tracer devices used in this lab accepted both Telnet and SSH once each protocol was available.

After:

```text
transport input ssh
```

the VTY lines accept only SSH.

The VTY lines are entered again because a different part of their configuration is now being changed.

Earlier:

```text
login local
```

configured authentication.

Now:

```text
transport input ssh
```

configures the allowed transport protocol.

The important VTY configuration is now:

```text
line vty 0 15
 login local
 transport input ssh
```

This means:

```text
login local
        ↓
Use local username/password

transport input ssh
        ↓
Allow SSH only
```

These commands solve different problems.

---

# Verification

## SSH Status

SSH status was verified on both devices.

```text
show ip ssh
```

![SSH Status](./images/03-ssh-status.png)

The output confirms that SSH is active and SSH version 2 is being used.

---

## SSH Access After Hardening

PC-Admin was used to test SSH again through the Packet Tracer Telnet/SSH application.

Connections to both devices remained successful.

![Successful SSH Login](./images/04-successful-ssh-login.png)

This confirms that restricting the VTY lines to SSH did not break legitimate SSH management.

---

## Telnet Blocking Test

Telnet was then selected in the PC-Admin Telnet/SSH application and connections were attempted to R0 and SW0.

![Telnet Blocked](./images/05-telnet-blocked.png)

The Telnet connections failed.

Ping was tested again and still succeeded.

This produced:

```text
Ping       → Works
SSH        → Works
Telnet     → Fails
```

The devices are still reachable.

Telnet is specifically being rejected because the VTY lines now contain:

```text
transport input ssh
```

---

# VTY ACL Hardening

Restricting the VTY lines to SSH controls **how** remote management connections can be made.

It does not control **which source devices** are allowed to make those connections.

A VTY ACL was added to restrict management access to PC-Admin.

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

The ACL is then applied to the VTY lines.

```text
line vty 0 15
 access-class VTY-MANAGEMENT in
```

This is different from an ACL used to filter normal routed traffic.

For normal interface traffic, an ACL might be applied using:

```text
ip access-group
```

under an interface.

For remote-management access, this lab uses:

```text
access-class
```

under the VTY lines.

The ACL therefore controls access to the remote-management lines themselves.

---

## VTY Transport vs VTY ACL

The two controls perform different jobs.

| Configuration | Controls |
|---|---|
| `transport input ssh` | Which remote-management protocol is allowed |
| `access-class VTY-MANAGEMENT in` | Which source IP addresses may access the VTY lines |

Together:

```text
Source IP
   ↓
VTY ACL
   ↓
Allowed source?
   ↓
SSH-only transport
   ↓
Local authentication
   ↓
Remote CLI
```

---

## Authorized Management Test

PC-Admin was tested after the VTY ACL was applied.

Using the Telnet/SSH application, SSH connections were attempted to:

```text
192.168.10.1
192.168.10.10
```

![Authorized SSH Access](./images/06-authorized-ssh-access.png)

Both SSH connections succeeded.

PC-Admin is permitted because its source address is:

```text
192.168.10.11
```

which matches:

```text
permit host 192.168.10.11
```

---

## Unauthorized Management Test

PC-Test was then used to attempt SSH connections to both devices.

PC-Test uses:

```text
192.168.10.12
```

![Unauthorized SSH Blocked](./images/07-unauthorized-ssh-blocked.png)

The SSH connections failed.

PC-Test was then used to ping R0 and SW0.

The pings still succeeded.

The result is:

```text
PC-Test ping → Works
PC-Test SSH  → Fails
```

This proves that PC-Test still has normal network connectivity.

Only its access to the VTY management lines is being denied.

---

## Failed Authentication Test

PC-Admin is an authorized source, so it was used for the authentication failure test.

An SSH connection was attempted using incorrect credentials.

![Failed Authentication](./images/08-failed-authentication.png)

The device accepted the connection attempt from PC-Admin because its source IP is allowed by the VTY ACL.

The login then failed because the username or password was incorrect.

This demonstrates that source authorization and user authentication are separate controls.

```text
Source allowed by VTY ACL
        ↓
SSH allowed by VTY transport
        ↓
Username/password checked
        ↓
Login accepted or rejected
```

---

## Active SSH Session

While a valid SSH connection was active, the session was checked from the network device.

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

Each command controls a different part of remote management.

| Setting | Purpose |
|---|---|
| `login local` | Authenticate users against the local user database |
| `transport input ssh` | Allow only SSH connections |
| `access-class VTY-MANAGEMENT in` | Allow VTY access only from approved source addresses |

The local account and enable secret are separate from these VTY controls:

| Configuration | Purpose |
|---|---|
| `username cisco secret cisco` | Creates the local remote-login account |
| `enable secret cisco` | Protects access to Privileged EXEC mode |

---

## Final Remote Management Policy

The finished configuration creates several separate checks.

| Test | Expected Result | Reason |
|---|---|---|
| PC-Admin ping | Allowed | Normal IP connectivity |
| PC-Test ping | Allowed | VTY ACL does not filter normal traffic |
| PC-Admin SSH | Allowed | Authorized source using allowed protocol |
| PC-Admin Telnet | Blocked | VTY lines accept SSH only |
| PC-Test SSH | Blocked | Source is not permitted by VTY ACL |
| PC-Admin SSH with wrong credentials | Blocked | Local authentication fails |
| PC-Admin SSH with correct credentials | Allowed | Source, protocol, and authentication all pass |
| Valid remote user + correct enable secret | Privileged EXEC allowed | Enable authentication succeeds |

The complete access process is:

```text
IP connectivity
      ↓
Source allowed by VTY ACL
      ↓
Protocol allowed by VTY transport
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

- Telnet and SSH both provide remote CLI access through VTY lines
- VTY lines require an authentication method before remote users can log in
- Telnet does not specifically require a local username database
- This lab uses `login local` so Telnet and SSH can use the same local account
- `username cisco secret cisco` creates the account
- `login local` tells the VTY lines to use that account
- The local account authenticates the remote login but does not automatically provide Privileged EXEC access
- `enable secret` separately protects access from User EXEC mode to Privileged EXEC mode
- Telnet provides unencrypted remote access
- SSH provides encrypted remote access
- SSH requires RSA keys before it can operate
- SSH version 2 requires an appropriate RSA key size
- RSA keys were generated before explicitly selecting SSH version 2
- `transport input` controls which remote-management protocols may use the VTY lines
- `transport input ssh` allows only SSH
- `transport input telnet` allows only Telnet
- `transport input ssh telnet` allows both
- Before an explicit transport restriction was configured, the Packet Tracer devices in this lab accepted Telnet and later SSH
- `access-class` applies an ACL directly to the VTY lines
- A VTY ACL can restrict management access without blocking normal IP connectivity
- VTY transport restrictions and VTY ACLs solve different problems
- Connectivity, source authorization, protocol selection, user authentication, and privilege level are separate parts of remote management

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
