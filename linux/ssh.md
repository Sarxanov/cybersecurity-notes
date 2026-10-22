# SSH Notes — Secure Shell

> A practical guide to SSH for Linux, networking, system administration, and cybersecurity labs.

## Table of Contents

1. [Introduction to SSH](#1-introduction-to-ssh)
2. [How SSH Works](#2-how-ssh-works)
3. [SSH vs Other Protocols](#3-ssh-vs-other-protocols)
4. [Installing and Checking SSH](#4-installing-and-checking-ssh)
5. [Connecting to a Remote Machine](#5-connecting-to-a-remote-machine)
6. [SSH Command-Line Options](#6-ssh-command-line-options)
7. [SSH Authentication Methods](#7-ssh-authentication-methods)
8. [SSH Key Generation](#8-ssh-key-generation)
9. [ssh-agent](#9-ssh-agent)
10. [File Transfer with SCP](#10-file-transfer-with-scp)
11. [File Transfer with SFTP](#11-file-transfer-with-sftp)
12. [SSH Configuration](#12-ssh-configuration)
13. [Port Forwarding and Tunneling](#13-port-forwarding-and-tunneling)
14. [Host Keys and known_hosts](#14-host-keys-and-known_hosts)
15. [SSH Server Administration](#15-ssh-server-administration)
16. [SSH Security Hardening](#16-ssh-security-hardening)
17. [Troubleshooting](#17-troubleshooting)
18. [SSH in Penetration Testing Labs](#18-ssh-in-penetration-testing-labs)
19. [Common SSH Workflows](#19-common-ssh-workflows)
20. [Quick Reference Cheat Sheet](#20-quick-reference-cheat-sheet)
21. [Practice Exercises](#21-practice-exercises)
22. [Glossary](#22-glossary)

---

## 1. Introduction to SSH

**SSH (Secure Shell)** is a cryptographic network protocol used to securely connect to remote computers, execute commands, transfer files, and administer systems over a network.

SSH is widely used by Linux administrators, developers, network engineers, and cybersecurity professionals.

### Common use cases

- Accessing a remote Linux shell.
- Managing servers without physical access.
- Running commands on remote machines.
- Transferring files securely.
- Creating encrypted tunnels.
- Accessing internal services through an authorized SSH connection.
- Connecting to virtual machines and cybersecurity lab environments.
- Managing Git repositories using SSH authentication.

### Basic syntax

```bash
ssh username@hostname
```

Example:

```bash
ssh student@192.0.2.10
```

This attempts to connect to the host at `192.0.2.10` using the account `student`.

`192.0.2.10` is a documentation-only example address. Replace it with a system you are authorized to access.

### Why SSH matters

Traditional remote-login protocols such as Telnet do not provide modern encrypted transport by default. SSH protects session traffic against network eavesdropping and unauthorized modification when correctly configured.

However, SSH does not guarantee that the remote computer is trustworthy or uncompromised. Secure use also depends on host verification, account security, and server configuration.

---

## 2. How SSH Works

SSH normally uses **TCP port 22**, although an administrator can configure a different port.

A simplified SSH connection follows these stages:

1. **TCP connection:** The client connects to the server's SSH port.
2. **Protocol negotiation:** Client and server agree on supported SSH algorithms.
3. **Key exchange:** They establish shared session keys.
4. **Server authentication:** The client checks the server's host key.
5. **User authentication:** The client proves its identity using an accepted method.
6. **Encrypted session:** Commands and terminal traffic travel through the protected connection.

### Encryption

Encryption helps prevent someone observing the network from reading SSH session contents.

### Integrity protection

SSH protects transmitted data against unauthorized modification.

### Server authentication

The client checks the server's host key to help confirm that it is communicating with the expected server.

### User authentication

The server checks whether the connecting user is permitted to access the requested account.

### Two different types of SSH keys

SSH uses keys for different purposes:

- **Host keys:** Identify the SSH server to clients.
- **User authentication keys:** Allow a user to authenticate to a server.

A server's host key is not the same as your personal SSH key pair.

### Simplified connection diagram

```text
+------------------+                 +------------------+
|   SSH Client     |                 |   SSH Server     |
|                  |                 |                  |
| Private key      |                 | Public key       |
| SSH configuration|                 | sshd service     |
| known_hosts      |                 | authorized_keys  |
+--------+---------+                 +---------+--------+
         |                                     |
         |========== TCP connection ===========|
         |                                     |
         |------ Key exchange and checks ------|
         |                                     |
         |------ User authentication ----------|
         |                                     |
         |======= Encrypted session ===========|
```

This is a conceptual diagram. Client and server key material has distinct roles, and the exact protocol exchange is more sophisticated than the diagram suggests.

---

## 3. SSH vs Other Protocols

| Protocol | Main purpose | Security characteristics |
|---|---|---|
| SSH | Remote shell and secure administration | Encrypted transport |
| Telnet | Legacy remote terminal | No modern encryption by default |
| FTP | File transfer | Traditional FTP does not encrypt credentials or data |
| SFTP | Secure file transfer | Runs over SSH |
| SCP | File copying | Transfers files through SSH |
| HTTP | Web communication | Not encrypted by itself |
| HTTPS | Secure web communication | Uses TLS to protect HTTP traffic |

### SSH, SCP, and SFTP

These tools are related but serve different purposes:

- `ssh` opens a remote shell or executes a remote command.
- `scp` copies files between machines.
- `sftp` provides interactive file-transfer operations.

SFTP is not simply FTP with a different port. It is a separate file-transfer protocol that operates over SSH.

Modern OpenSSH `scp` uses the SFTP protocol internally by default, although compatibility modes and older implementations can differ.

---

## 4. Installing and Checking SSH

### Check the SSH client

```bash
ssh -V
```

This prints the installed OpenSSH client version.

Check whether common SSH utilities are available:

```bash
command -v ssh
command -v ssh-keygen
command -v ssh-add
command -v scp
command -v sftp
```

### Install the SSH client on Kali Linux or Debian

```bash
sudo apt update
sudo apt install openssh-client
```

The client is often already installed on Kali Linux.

### Install the SSH server

To allow authorized users to connect to your machine:

```bash
sudo apt install openssh-server
```

### Check the SSH service

On Kali and many Debian-based systems:

```bash
systemctl status ssh
```

Start the service:

```bash
sudo systemctl start ssh
```

Enable it at boot:

```bash
sudo systemctl enable ssh
```

Some distributions use `sshd` as the service name instead.

### Check listening ports

```bash
ss -tlnp
```

Inspect port 22 specifically:

```bash
sudo ss -tlnp | grep ':22'
```

A listening socket indicates that a service is accepting connections on the corresponding local address and port. It does not prove that the service is reachable through every firewall or network interface.

### Check network connectivity

```bash
nc -vz 192.0.2.10 22
```

This attempts a TCP connection to port 22.

A successful connection indicates that the TCP port is reachable from your current environment. It does not guarantee successful SSH authentication.

---

## 5. Connecting to a Remote Machine

### Basic connection

```bash
ssh username@hostname
```

Example:

```bash
ssh student@192.0.2.10
```

### Connect using a domain name

```bash
ssh username@example.org
```

The hostname must resolve to the intended machine.

### Connect on a custom port

```bash
ssh -p 2222 username@192.0.2.10
```

The lowercase `-p` specifies the SSH destination port.

### Connect using a specific private key

```bash
ssh -i ~/.ssh/id_ed25519 username@192.0.2.10
```

The identity file must correspond to a public key accepted by the server.

### Execute one remote command

```bash
ssh username@192.0.2.10 'whoami'
```

The command runs on the remote machine.

Other examples:

```bash
ssh username@192.0.2.10 'hostname'
ssh username@192.0.2.10 'pwd'
ssh username@192.0.2.10 'uptime'
```

### Confirm where you are connected

After connecting:

```bash
whoami
hostname
pwd
```

These commands help distinguish your local system from the remote system.

### Disconnect

```bash
exit
```

Alternatively, press `Ctrl+D` in a typical shell.

---

## 6. SSH Command-Line Options

### Verbose output

```bash
ssh -v username@host
ssh -vv username@host
ssh -vvv username@host
```

The additional `v` characters increase diagnostic detail.

Verbose output can show:

- Which configuration files are read.
- Which keys are considered.
- Which authentication methods are offered.
- Which algorithms are negotiated.
- Why an authentication attempt failed.

Be careful when sharing logs: verbose output may reveal usernames, hostnames, file paths, or other operational details.

### Connection timeout

```bash
ssh -o ConnectTimeout=5 username@host
```

This sets a five-second timeout for establishing the connection.

### Select a specific authentication method

```bash
ssh -o PreferredAuthentications=publickey username@host
```

This prioritizes public-key authentication for the connection attempt.

### Use only the specified identity

```bash
ssh -o IdentitiesOnly=yes \
    -i ~/.ssh/id_ed25519 \
    username@host
```

This helps prevent SSH from trying unrelated identities available through an agent or configuration.

### Disable password authentication for one attempt

```bash
ssh -o PreferredAuthentications=publickey \
    -o PasswordAuthentication=no \
    username@host
```

This is useful when testing whether key-based authentication works.

### Specify a known host-key file

```bash
ssh -o UserKnownHostsFile=~/.ssh/known_hosts username@host
```

This explicitly selects the file used to store known host keys. Avoid disabling host-key verification as a routine troubleshooting shortcut.

### Display effective configuration

```bash
ssh -G username@host
```

This prints the effective client configuration after processing applicable settings.

---

## 7. SSH Authentication Methods

SSH servers can support multiple authentication methods depending on their configuration.

### 7.1 Password authentication

The server asks for the account password.

```bash
ssh username@host
```

A password prompt may appear if password authentication is enabled.

When entering a password, terminal characters usually do not appear. This is normal.

Password security depends on password strength, account controls, rate limiting, and server configuration.

### 7.2 Public-key authentication

Public-key authentication uses a cryptographic key pair:

- The private key stays under the user's control.
- The public key is installed on the server.
- The client proves possession of the private key by signing authentication data.

The private key is not transmitted to the server during ordinary public-key authentication.

### 7.3 Keyboard-interactive authentication

The server may request interactive responses, such as an additional verification step. The exact behavior depends on server configuration and authentication providers.

### 7.4 Hardware-backed authentication

SSH can use supported hardware security keys and FIDO-based authentication methods. Availability depends on the OpenSSH version, key type, and server configuration.

### 7.5 SSH certificates

SSH certificates allow a trusted authority to sign user or host public keys. They can simplify access management in larger environments.

### Important distinction

Being able to reach TCP port 22 does not mean you have permission to log in. Authentication and authorization are separate steps.

---

## 8. SSH Key Generation

### Generate an Ed25519 key pair

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Ed25519 is a modern public-key algorithm supported by current OpenSSH implementations.

During generation, you may be asked to:

1. Choose the file path.
2. Set a passphrase.
3. Confirm the passphrase.

A passphrase protects the private-key file if someone obtains a copy of it.

### Default key files

```text
~/.ssh/id_ed25519       Private key
~/.ssh/id_ed25519.pub   Public key
```

**Never share your private key.** The `.pub` file contains the public key and is the file normally installed on a server.

### Generate a separate key for a lab

```bash
ssh-keygen -t ed25519 \
    -f ~/.ssh/lab_ed25519 \
    -C "authorized-lab-key"
```

Using a separate key makes it easier to manage access independently from your other keys.

Do not overwrite an existing key unless you intend to replace it.

### View your public key

```bash
cat ~/.ssh/id_ed25519.pub
```

### Display the key fingerprint

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

Fingerprints are useful for identifying keys.

### Install the public key

If you have legitimate access to the server and the server permits this method:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@host
```

For a custom port:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 username@host
```

This typically adds the public key to the account's `~/.ssh/authorized_keys` file.

### Test the key

```bash
ssh -i ~/.ssh/id_ed25519 username@host
```

If it fails, inspect the connection:

```bash
ssh -vvv -i ~/.ssh/id_ed25519 username@host
```

### Recommended permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/authorized_keys
```

Private-key files should not be readable by other ordinary users. The server may reject keys or configuration files with unsafe permissions.

### If a private key is compromised

1. Treat the key as compromised.
2. Remove its public key from systems where it is authorized.
3. Generate a replacement key pair.
4. Install the replacement public key through an authorized process.
5. Review relevant logs and access history when appropriate.

Deleting your local private-key file does not revoke a public key already authorized by a remote server.

---

## 9. ssh-agent

`ssh-agent` can hold unlocked key material in memory so you do not need to enter the private-key passphrase repeatedly during an agent session.

### Check the agent

```bash
echo "$SSH_AUTH_SOCK"
ssh-add -l
```

If no agent is available, `ssh-add` may report that it cannot connect to one.

### Add a key

```bash
ssh-add ~/.ssh/id_ed25519
```

### List loaded keys

```bash
ssh-add -l
```

### Remove one key

```bash
ssh-add -d ~/.ssh/id_ed25519
```

### Remove all keys from the current agent

```bash
ssh-add -D
```

An agent improves convenience but must be treated as a security-sensitive component. Do not expose its socket or forward it to untrusted hosts.

---

## 10. File Transfer with SCP

`scp` copies files over SSH.

### Upload a file

```bash
scp notes.txt username@host:/home/username/
```

### Download a file

```bash
scp username@host:/home/username/report.txt .
```

The final `.` means the current local directory.

### Copy a directory

```bash
scp -r project/ username@host:/home/username/
```

### Use a custom port

```bash
scp -P 2222 notes.txt username@host:/home/username/
```

Notice that `scp` uses uppercase `-P` for the port.

### Use a specific identity

```bash
scp -i ~/.ssh/lab_ed25519 notes.txt username@host:/home/username/
```

### Practical reminders

- Verify the destination before transferring files.
- Be careful about overwriting existing files.
- Use only authorized accounts and destinations.
- For complex transfers or interactive file management, consider SFTP.

---

## 11. File Transfer with SFTP

SFTP provides interactive file-transfer operations over SSH.

### Connect

```bash
sftp username@host
```

### Use a custom port

```bash
sftp -P 2222 username@host
```

### Common SFTP commands

| Command | Purpose |
|---|---|
| `pwd` | Show the remote working directory |
| `lpwd` | Show the local working directory |
| `ls` | List remote files |
| `lls` | List local files |
| `cd directory` | Change remote directory |
| `lcd directory` | Change local directory |
| `get file.txt` | Download a remote file |
| `put file.txt` | Upload a local file |
| `mkdir name` | Create a remote directory, if permitted |
| `help` | Show supported commands |
| `exit` | Close the SFTP session |

The exact commands available depend on the SFTP client.

---

## 12. SSH Configuration

The per-user client configuration file is normally:

```text
~/.ssh/config
```

Create the directory if needed:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

### Example configuration

```sshconfig
Host lab
    HostName 192.0.2.10
    User student
    Port 2222
    IdentityFile ~/.ssh/lab_ed25519
    IdentitiesOnly yes
```

Now connect using the alias:

```bash
ssh lab
```

Instead of typing the full hostname, username, port, and identity options each time, SSH reads the configuration.

### Configure multiple hosts

```sshconfig
Host lab
    HostName 192.0.2.10
    User student
    IdentityFile ~/.ssh/lab_ed25519
    IdentitiesOnly yes

Host dev-server
    HostName dev.example.org
    User developer
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

Example connections:

```bash
ssh lab
ssh dev-server
```

These hostnames and addresses are examples; replace them with your authorized systems.

### Check effective settings

```bash
ssh -G lab
```

### Configuration security

Protect the configuration file:

```bash
chmod 600 ~/.ssh/config
```

Do not put private-key contents, passwords, or other secrets in the configuration file.

---

## 13. Port Forwarding and Tunneling

SSH can forward network traffic through an encrypted connection. This is useful for securely reaching authorized services.

### 13.1 Local port forwarding

Local forwarding exposes a remote service through a port on your local machine.

General syntax:

```bash
ssh -L LOCAL_PORT:DESTINATION_HOST:DESTINATION_PORT username@ssh_server
```

Example:

```bash
ssh -L 8080:127.0.0.1:80 student@192.0.2.10
```

After the tunnel is established, opening this address locally:

```text
http://127.0.0.1:8080
```

forwards the connection through SSH to port 80 on the SSH server's own loopback interface.

This is useful when a web application is reachable from the SSH server but is not directly exposed to your local network.

### 13.2 Remote port forwarding

Remote forwarding makes a service on the client side accessible through a port on the SSH server side, subject to server configuration and network restrictions.

General syntax:

```bash
ssh -R REMOTE_PORT:DESTINATION_HOST:DESTINATION_PORT username@ssh_server
```

Example:

```bash
ssh -R 9000:127.0.0.1:3000 student@192.0.2.10
```

This asks the server to forward connections from the remote port to port 3000 on the SSH client's side. Exact reachability depends on the server's forwarding policy and bind-address settings.

### 13.3 Dynamic forwarding

Dynamic forwarding creates a local SOCKS proxy:

```bash
ssh -D 1080 username@192.0.2.10
```

A compatible application can use the local SOCKS proxy at `127.0.0.1:1080`.

Dynamic forwarding does not automatically route all system traffic. Applications must be configured to use the proxy.

### 13.4 Background tunnel

```bash
ssh -fN -L 8080:127.0.0.1:80 username@192.0.2.10
```

- `-f`: Requests that SSH go into the background after authentication.
- `-N`: Does not execute a remote command.
- `-L`: Sets up local forwarding.

Use forwarding only when authorized. Tunnels can expose internal services, so understand which interfaces and ports are accessible.

---

## 14. Host Keys and known_hosts

SSH host keys help the client recognize servers it has previously trusted.

Known host information is commonly stored in:

```text
~/.ssh/known_hosts
```

### Why host verification matters

If a server's host key changes unexpectedly, SSH may warn that the remote host identification has changed.

Possible explanations include:

- The server was rebuilt and received a new host key.
- The hostname now points to a different server.
- An administrator legitimately rotated the key.
- A network interception or impersonation attempt is occurring.

Do not dismiss this warning without investigation.

### Inspect a host key fingerprint

A server administrator can obtain the server's public host-key fingerprint through an approved method. Compare it with the fingerprint presented by the client.

For example, on a server you administer:

```bash
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

The actual host-key filename may differ by configuration.

### Inspect known hosts

```bash
ssh-keygen -F example.org
```

To check a non-default port:

```bash
ssh-keygen -F '[example.org]:2222'
```

### Remove a known-host entry

Only after independently verifying that the host-key change is legitimate:

```bash
ssh-keygen -R example.org
```

For a non-default port:

```bash
ssh-keygen -R '[example.org]:2222'
```

Reconnect and verify the new fingerprint through a trusted channel before accepting it.

Avoid using `StrictHostKeyChecking=no` as a general solution. Disabling host-key verification can make interception attacks easier.

---

## 15. SSH Server Administration

Server configuration is commonly stored in:

```text
/etc/ssh/sshd_config
```

Some systems also load configuration fragments from:

```text
/etc/ssh/sshd_config.d/
```

Always check your distribution's configuration structure before editing files.

### Validate configuration

```bash
sudo sshd -t
```

This checks the SSH daemon configuration for syntax errors.

### Reload the service

On systems using the `ssh` service name:

```bash
sudo systemctl reload ssh
```

If reload is unsupported, follow the distribution's documented procedure. Keep an existing administrative session open until the new configuration has been tested, to reduce the risk of locking yourself out.

### Check the daemon status

```bash
sudo systemctl status ssh
```

### Inspect authentication logs

On many systemd-based distributions:

```bash
sudo journalctl -u ssh
```

Some distributions use a different service name or record authentication events in files such as `/var/log/auth.log` or `/var/log/secure`.

Logs can help identify failed logins, configuration issues, and unexpected access attempts. Access to logs should be restricted appropriately.

---

## 16. SSH Security Hardening

SSH security should follow the principle of least privilege: users should receive only the access they need.

### 16.1 Use strong authentication

- Prefer public-key authentication where practical.
- Use a strong passphrase to protect private keys.
- Protect accounts with strong passwords where passwords remain enabled.
- Consider hardware-backed keys or SSH certificates for suitable environments.
- Remove keys that are no longer required.

### 16.2 Protect private keys

- Never commit private keys to Git.
- Never upload private keys to public repositories.
- Do not send private keys through chat or email.
- Restrict filesystem permissions.
- Rotate keys if compromise is suspected.

### 16.3 Disable root login where appropriate

An administrator may choose to configure:

```text
PermitRootLogin no
```

Before applying this setting, ensure that a separate authorized administrative account and an appropriate privilege-escalation method are available.

### 16.4 Restrict password authentication when appropriate

A server that has verified working public-key access may use:

```text
PasswordAuthentication no
```

Do not apply this blindly. Confirm that authorized users can still access the system and that recovery access is available.

### 16.5 Restrict users

Depending on the environment, administrators may use settings such as:

```text
AllowUsers student admin
```

Only list the accounts that should be permitted to log in through SSH. Validate the full configuration and test access before closing existing sessions.

### 16.6 Limit unnecessary forwarding

If port forwarding is not needed, an administrator can consider:

```text
AllowTcpForwarding no
```

This may break legitimate workflows, so choose a policy that matches the system's intended use.

### 16.7 Keep OpenSSH updated

Apply security updates through the operating system's normal package-management process.

On Kali or Debian-based systems:

```bash
sudo apt update
sudo apt upgrade
```

Review the changes and maintenance requirements for important systems before upgrading.

### 16.8 Use a firewall

Permit SSH only from networks that should have access whenever practical. Firewall syntax depends on the firewall software and network environment.

Changing the SSH port may reduce background noise from automated scans, but **port changes are not a substitute for authentication, patching, and access controls**.

### 16.9 Validate every configuration change

```bash
sudo sshd -t
```

Then reload the service using the correct service name and test a new connection before closing the existing administrative session.

---

## 17. Troubleshooting

### Error: `Connection refused`

Possible causes:

- The SSH server is not running.
- No service is listening on the requested port.
- The server is actively rejecting the connection.
- A firewall or intermediary is returning a rejection.

Checks on a server you administer:

```bash
systemctl status ssh
sudo ss -tlnp | grep ':22'
```

### Error: `Connection timed out`

Possible causes:

- Incorrect IP address or hostname.
- Network routing problems.
- Firewall silently dropping traffic.
- Wrong port.
- The target is offline.

Checks:

```bash
getent hosts example.org
ip route
nc -vz 192.0.2.10 22
```

A timeout and a refusal indicate different connection behaviors, although neither alone identifies the exact root cause.

### Error: `Permission denied (publickey)`

Possible causes:

- The wrong username was supplied.
- The wrong private key was selected.
- The matching public key is missing from `authorized_keys`.
- The server disallows the attempted authentication method.
- Permissions or ownership are incorrect.
- The account is restricted by server policy.

Debug:

```bash
ssh -vvv -i ~/.ssh/id_ed25519 username@host
```

On a server you administer, inspect the relevant account's home directory, `.ssh` directory, `authorized_keys`, and SSH authentication logs.

### Error: `Host key verification failed`

Possible causes:

- A stored host key does not match the key presented by the server.
- The host was rebuilt.
- The connection is reaching a different machine.
- A possible interception attempt is occurring.

Do not immediately disable host-key checking. Verify the server's fingerprint through a trusted source first.

### Error: `WARNING: UNPROTECTED PRIVATE KEY FILE!`

The private-key file permissions may be too permissive.

For a personal key:

```bash
chmod 600 ~/.ssh/id_ed25519
```

Check directory permissions:

```bash
chmod 700 ~/.ssh
```

### Error: `Could not resolve hostname`

Possible causes:

- Typographical error in the hostname.
- DNS problem.
- Incorrect local SSH configuration.
- Missing network connectivity.

Checks:

```bash
getent hosts example.org
```

For a lab that provides an IP address, confirm that you are using the correct address.

### SSH works locally but not remotely

Check:

- The daemon's listening address.
- Host firewall rules.
- Network firewall rules.
- Routing and network segmentation.
- Whether the service is listening on the expected port.
- Whether the destination machine's IP address is correct.

---

## 18. SSH in Penetration Testing Labs

SSH frequently appears in authorized cybersecurity labs and Capture The Flag environments.

A common learning workflow is:

1. Identify open ports.
2. Determine which service is listening.
3. Confirm the SSH service and port.
4. Identify authorized credentials or keys supplied by the lab.
5. Attempt a legitimate connection.
6. Enumerate the account's permitted environment.
7. Record findings and understand the security implications.

### Step 1: Discover the SSH port

On an authorized target:

```bash
nmap -p 22 -sV 192.0.2.10
```

To scan a specific alternative port:

```bash
nmap -p 2222 -sV 192.0.2.10
```

Service-version detection provides clues about the service, but a reported version may not fully identify patch status or vulnerabilities.

### Step 2: Connect with supplied credentials

```bash
ssh username@192.0.2.10
```

For a custom port:

```bash
ssh -p 2222 username@192.0.2.10
```

Use only credentials and systems authorized by the lab or system owner.

### Step 3: Check your identity and environment

After login:

```bash
whoami
id
hostname
pwd
```

Inspect basic system information:

```bash
uname -a
cat /etc/os-release
```

Access to particular files or commands depends on account permissions.

### Step 4: Transfer lab files

Download a file from the authorized lab machine:

```bash
scp username@192.0.2.10:/home/username/notes.txt .
```

Upload a local file if permitted:

```bash
scp ./notes.txt username@192.0.2.10:/home/username/
```

### SSH and privilege escalation

SSH itself does not automatically grant elevated privileges. The privileges available after login depend on the authenticated account, operating-system configuration, and assigned permissions.

In a lab, learn to distinguish:

- Network reachability.
- Successful authentication.
- Account identity.
- Account privileges.
- Permission to perform a particular action.

Only test privilege-escalation techniques in environments where you have explicit authorization.

---

## 19. Common SSH Workflows

### Workflow A: Connect to a lab machine

```bash
ssh student@192.0.2.10
```

Then:

```bash
whoami
hostname
pwd
```

### Workflow B: Use a dedicated lab key

Generate the key:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/lab_ed25519
```

Connect with it:

```bash
ssh -i ~/.ssh/lab_ed25519 student@192.0.2.10
```

This works only if the corresponding public key is authorized on the server.

### Workflow C: Use an SSH alias

Add the following to `~/.ssh/config`:

```sshconfig
Host my-lab
    HostName 192.0.2.10
    User student
    IdentityFile ~/.ssh/lab_ed25519
    IdentitiesOnly yes
```

Connect:

```bash
ssh my-lab
```

### Workflow D: Copy a file from the remote machine

```bash
scp student@192.0.2.10:/home/student/report.txt .
```

### Workflow E: Debug a connection

```bash
ssh -vvv student@192.0.2.10
```

Observe the output to determine whether the problem is name resolution, TCP connectivity, host-key verification, or user authentication.

### Workflow F: Create a local tunnel

```bash
ssh -L 8080:127.0.0.1:80 student@192.0.2.10
```

If the connection and forwarding succeed, access the forwarded service through `http://127.0.0.1:8080` on the local machine.

---

## 20. Quick Reference Cheat Sheet

| Task | Command |
|---|---|
| Show client version | `ssh -V` |
| Connect to host | `ssh user@host` |
| Connect to custom port | `ssh -p 2222 user@host` |
| Select a private key | `ssh -i ~/.ssh/id_ed25519 user@host` |
| Debug connection | `ssh -vvv user@host` |
| Generate Ed25519 key | `ssh-keygen -t ed25519` |
| View public key | `cat ~/.ssh/id_ed25519.pub` |
| Show key fingerprint | `ssh-keygen -lf ~/.ssh/id_ed25519.pub` |
| Add public key to server | `ssh-copy-id user@host` |
| List agent identities | `ssh-add -l` |
| Add key to agent | `ssh-add ~/.ssh/id_ed25519` |
| Copy file to server | `scp file.txt user@host:/path/` |
| Download file | `scp user@host:/path/file.txt .` |
| Start SFTP session | `sftp user@host` |
| Set up local forwarding | `ssh -L 8080:127.0.0.1:80 user@host` |
| Show effective client config | `ssh -G host-alias` |
| Find stored host key | `ssh-keygen -F hostname` |
| Remove verified old host key | `ssh-keygen -R hostname` |
| Check server config syntax | `sudo sshd -t` |
| Check service status | `systemctl status ssh` |
| Check listening sockets | `sudo ss -tlnp` |

---

## 21. Practice Exercises

Complete these exercises on your own machine, a virtual machine, or an authorized lab.

### Beginner

- [ ] Check the installed OpenSSH client version.
- [ ] Find the location of the `ssh` executable.
- [ ] Read the `ssh` manual page using `man ssh`.
- [ ] Explain the difference between an SSH client and an SSH server.
- [ ] Explain why SSH commonly uses TCP port 22.
- [ ] Identify the difference between a private key and a public key.

### Intermediate

- [ ] Generate a dedicated Ed25519 key pair for a test environment.
- [ ] Display the public key and fingerprint.
- [ ] Configure an SSH host alias.
- [ ] Connect to an authorized test machine with a specific key.
- [ ] Use verbose output to understand an authentication failure.
- [ ] Transfer a file using SCP.
- [ ] Transfer a file using SFTP.
- [ ] Explain the difference between a timeout and a connection refusal.

### Advanced

- [ ] Create a local port-forwarding tunnel to a test web service.
- [ ] Compare the effective configuration from `ssh -G` with the settings you wrote.
- [ ] Validate a modified SSH server configuration before reloading it.
- [ ] Review authentication logs on a server you administer.
- [ ] Design an SSH access policy based on least privilege.
- [ ] Explain how host-key verification helps defend against server impersonation.

### Reflection questions

1. Why should a private key never be committed to a public Git repository?
2. Why is changing port 22 not sufficient to secure SSH?
3. What is the difference between host authentication and user authentication?
4. Why should you investigate a changed host-key warning?
5. What does a successful TCP connection to port 22 actually prove?
6. Why should an administrator keep an existing SSH session open while testing a new server configuration?
7. How can port forwarding expose services that were not directly reachable from your local machine?

---

## 22. Glossary

| Term | Definition |
|---|---|
| SSH | Secure Shell, a protocol for secure remote access |
| OpenSSH | A widely used open-source SSH implementation |
| SSH client | Software that initiates an SSH connection |
| SSH server (`sshd`) | Software that accepts SSH connections |
| Host key | Key used to identify an SSH server |
| Public key | Public component of a cryptographic key pair |
| Private key | Secret component used to prove possession of a key |
| Passphrase | Secret used to protect a private-key file |
| `authorized_keys` | File commonly containing public keys allowed to authenticate to an account |
| `known_hosts` | File storing trusted SSH server host keys |
| `ssh-agent` | Program that manages unlocked SSH identities |
| SCP | Secure file-copy utility using SSH |
| SFTP | File-transfer protocol operating over SSH |
| Port forwarding | Redirecting connections through an SSH tunnel |
| Fingerprint | Short representation used to identify a cryptographic key |
| Authentication | Verifying an identity |
| Authorization | Determining what an authenticated identity is allowed to do |
| Least privilege | Giving users and services only the permissions they need |
| TCP | Transport protocol providing reliable, ordered byte streams |
| Tunnel | A connection that carries other traffic through another connection |

---

## Final Notes

SSH is one of the most important tools for Linux administration and cybersecurity. Understanding how connections, authentication, host keys, file transfers, and tunnels work will make remote administration and authorized lab work much easier.

**Core principle:** Verify the server, protect your private keys, use the least privilege necessary, and investigate unexpected authentication or host-key errors.

Practice only on systems you own or have explicit permission to access.
