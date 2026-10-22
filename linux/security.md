# Linux Security Notes

> A practical guide to Linux security, system hardening, access control, permissions, networking, and defensive cybersecurity.

## Table of Contents

1. [Introduction to Linux Security](#1-introduction-to-linux-security)
2. [The Linux Security Model](#2-the-linux-security-model)
3. [Users and Groups](#3-users-and-groups)
4. [File Permissions](#4-file-permissions)
5. [Ownership and Special Permissions](#5-ownership-and-special-permissions)
6. [sudo and Privilege Management](#6-sudo-and-privilege-management)
7. [Processes and Services](#7-processes-and-services)
8. [Package Management and Updates](#8-package-management-and-updates)
9. [Network Security](#9-network-security)
10. [Firewall Basics](#10-firewall-basics)
11. [SSH Security](#11-ssh-security)
12. [Logs and Monitoring](#12-logs-and-monitoring)
13. [File Integrity and Hashing](#13-file-integrity-and-hashing)
14. [Sensitive Files and Secrets](#14-sensitive-files-and-secrets)
15. [Linux Security Hardening](#15-linux-security-hardening)
16. [Common Security Tools](#16-common-security-tools)
17. [Incident Investigation Basics](#17-incident-investigation-basics)
18. [Common Misconfigurations](#18-common-misconfigurations)
19. [Security Checklist](#19-security-checklist)
20. [Practical Exercises](#20-practical-exercises)
21. [Quick Reference Cheat Sheet](#21-quick-reference-cheat-sheet)
22. [Glossary](#22-glossary)

---

## 1. Introduction to Linux Security

Linux security is the practice of protecting Linux systems, user accounts, files, services, applications, and network connections against unauthorized access, misuse, and compromise.

Linux is widely used in servers, cloud infrastructure, embedded systems, development environments, and cybersecurity laboratories.

Security is not provided by a single command or tool. It depends on several layers working together:

- Authentication: verifying who is connecting.
- Authorization: determining what an account can do.
- Access control: restricting access to files and resources.
- Updates: fixing known software vulnerabilities.
- Network protection: controlling incoming and outgoing traffic.
- Monitoring: detecting suspicious behavior.
- Hardening: reducing unnecessary functionality and attack surface.
- Recovery: restoring systems after failures or incidents.

### Core principle

**Use the least privilege necessary.**

Users, services, and applications should receive only the permissions required to perform their intended tasks.

For example, a program that only needs to read a configuration file should not automatically run as root.

---

## 2. The Linux Security Model

Linux is a multiuser operating system. Different users and processes can have different identities and permissions.

### Important concepts

| Concept | Meaning |
|---|---|
| User | An identity associated with processes and files |
| Group | A collection of users used for access management |
| UID | Numeric user identifier |
| GID | Numeric group identifier |
| Root | The superuser, normally UID 0 |
| Permission | An access rule for a resource |
| Capability | A specific privileged operation that may be granted to a process |
| Process | A running program |
| Service | A program or process that provides a system function |
| Authentication | Verifying identity |
| Authorization | Deciding which actions are permitted |

### Root and ordinary users

The root account has extensive administrative privileges. A compromised root process can potentially affect the entire system.

Ordinary users generally operate under more restricted permissions.

Check your current identity:

```bash
whoami
id
```

Display your numeric user and group identifiers:

```bash
id -u
id -g
```

Check whether your current shell has root privileges:

```bash
id
```

If the output contains `uid=0(root)`, the current process is running as root.

### Defense in depth

Defense in depth means using multiple security controls instead of depending on one.

For example, a secure SSH server may use:

- Strong authentication.
- Restricted account permissions.
- Firewall rules.
- Timely software updates.
- Authentication logging.
- Monitoring and alerting.

If one control fails, other controls may still reduce the impact.

---

## 3. Users and Groups

Linux uses user and group identities to control access to system resources.

### List logged-in users

```bash
who
w
```

### Display the current user

```bash
whoami
id
```

### List local user accounts

```bash
cat /etc/passwd
```

The `/etc/passwd` file contains account metadata. On modern Linux systems, password hashes are normally stored separately in `/etc/shadow`, not directly in `/etc/passwd`.

A typical entry looks like:

```text
username:x:1000:1000:User Name:/home/username:/bin/bash
```

The fields represent:

1. Username.
2. Password placeholder.
3. UID.
4. Primary GID.
5. Account description.
6. Home directory.
7. Login shell.

### Inspect groups

```bash
cat /etc/group
groups
id username
```

### Create a user

On Debian-based distributions:

```bash
sudo adduser newuser
```

### Add a user to a group

```bash
sudo usermod -aG groupname username
```

The `-a` option appends the group membership. Omitting `-a` while using `-G` can replace supplementary group memberships.

Use elevated group membership carefully, especially for groups that grant administrative or device access.

### Remove a user from a group

```bash
sudo gpasswd -d username groupname
```

### Security considerations

- Remove accounts that are no longer needed.
- Avoid sharing personal accounts.
- Review privileged group memberships.
- Use strong authentication.
- Restrict service accounts to the resources they need.
- Avoid assigning administrative privileges without a clear reason.

---

## 4. File Permissions

Linux permissions determine who can read, modify, or execute a file.

There are three main permission categories:

- `u`: owner.
- `g`: group.
- `o`: others.

The three basic permissions are:

| Permission | Symbol | Regular file | Directory |
|---|---|---|---|
| Read | `r` | Read file contents | List directory entries |
| Write | `w` | Modify file contents | Create, delete, or rename entries, subject to other controls |
| Execute | `x` | Execute the file if applicable | Traverse the directory |

### Inspect permissions

```bash
ls -l
```

Example:

```text
-rwxr-x--- 1 alice developers 2048 Oct 20 10:00 script.sh
```

Interpretation:

- `-`: Regular file.
- `rwx`: Owner can read, write, and execute.
- `r-x`: Group can read and execute.
- `---`: Others have no permissions.

### Change permissions with chmod

Symbolic mode:

```bash
chmod u+x script.sh
chmod g-w report.txt
chmod o-r private.txt
```

Numeric mode:

```bash
chmod 644 report.txt
chmod 600 secret.txt
chmod 700 private_directory
```

Common permission values:

| Mode | Meaning |
|---|---|
| `600` | Owner can read and write |
| `640` | Owner can read/write; group can read |
| `644` | Owner can read/write; others can read |
| `700` | Owner has full access |
| `750` | Owner has full access; group can read/execute |
| `755` | Owner has full access; others can read/execute |
| `777` | Everyone has read/write/execute permissions, subject to other controls |

### Why `777` is dangerous

Giving everyone write permission can allow unintended users or processes to modify a file or directory.

Avoid using `chmod 777` as a generic fix for permission errors. Identify the intended owner and access requirements instead.

### Check access to a file

```bash
ls -l filename
namei -l /path/to/file
```

`namei -l` can help identify permissions along the directory path.

Remember that directory permissions also affect access to files within them.

---

## 5. Ownership and Special Permissions

### Change ownership

```bash
sudo chown username filename
```

Change owner and group:

```bash
sudo chown username:groupname filename
```

Change ownership recursively when appropriate:

```bash
sudo chown -R username:groupname directory/
```

Recursive ownership changes can have significant effects. Verify the target path before running them.

### Change group ownership

```bash
sudo chgrp groupname filename
```

### Special permission: SUID

Set-user-ID can cause an executable to run with the effective user identity associated with its owner, subject to operating-system rules and mount options.

Inspect permissions:

```bash
find /usr/bin -type f -perm -4000 -ls 2>/dev/null
```

This lists SUID files under `/usr/bin`. It is an inventory command, not proof that any listed file is vulnerable.

SUID is sometimes required for legitimate system functionality. Do not remove it without understanding the program's purpose.

### Special permission: SGID

Inspect SGID files:

```bash
find /usr/bin -type f -perm -2000 -ls 2>/dev/null
```

On directories, SGID commonly causes newly created entries to inherit the directory's group.

### Special permission: sticky bit

The sticky bit on a shared directory restricts who can remove or rename entries, subject to ownership and privileges.

Inspect `/tmp`:

```bash
ls -ld /tmp
```

A typical permission string includes a `t` at the end, such as:

```text
drwxrwxrwt
```

This is a common configuration for a shared temporary directory.

### Linux capabilities

Capabilities divide certain privileged operations into smaller units.

Inspect capabilities on executables:

```bash
getcap -r /usr/bin /usr/sbin 2>/dev/null
```

A capability may be legitimate, but unexpected or unnecessary capabilities can increase risk.

Investigate the executable, its owner, its purpose, and the reason for the capability before changing it.

---

## 6. sudo and Privilege Management

`sudo` allows authorized users to run commands with another user's privileges, commonly root.

### Run a command with elevated privileges

```bash
sudo apt update
```

### Inspect sudo permissions

```bash
sudo -l
```

This displays the commands the current user is permitted to execute through sudo, subject to policy.

### Switch to a root shell

```bash
sudo -i
```

Exit the root shell when finished:

```bash
exit
```

Prefer running a single command with `sudo` when a full privileged shell is unnecessary.

### Why sudo matters

Excessive administrative access increases the impact of mistakes, malware, and compromised accounts.

Good practices:

- Grant only necessary privileges.
- Review sudoers rules.
- Avoid unnecessary direct root login.
- Do not run untrusted scripts with `sudo`.
- Verify commands before executing them as root.

### Edit sudoers safely

Use:

```bash
sudo visudo
```

`visudo` validates the sudoers configuration to help prevent syntax errors that could break administrative access.

Do not edit `/etc/sudoers` casually with a normal text editor.

---

## 7. Processes and Services

A process is a running program. Services often run in the background and provide network or system functionality.

### List processes

```bash
ps aux
```

Display a live process view:

```bash
top
```

If installed:

```bash
htop
```

### Find a process

```bash
pgrep -a sshd
```

Search the process listing:

```bash
ps aux | grep '[s]shd'
```

### Inspect services

```bash
systemctl --type=service --state=running
systemctl status ssh
```

### Start, stop, or restart a service

```bash
sudo systemctl start service-name
sudo systemctl stop service-name
sudo systemctl restart service-name
```

Replace `service-name` with the correct service.

Restarting production services can interrupt users. Understand the impact before taking action.

### Enable or disable startup

```bash
sudo systemctl enable service-name
sudo systemctl disable service-name
```

### Security principles for services

- Disable services that are not needed.
- Keep required services patched.
- Avoid running network services with unnecessary privileges.
- Review service configuration and exposed ports.
- Monitor unexpected service changes.
- Investigate processes with unusual paths, identities, or behavior.

A process using high CPU or an unfamiliar name is not automatically malicious. Investigate before drawing conclusions.

---

## 8. Package Management and Updates

Unpatched software may contain known vulnerabilities.

On Kali Linux and Debian-based systems, APT manages packages.

### Refresh package metadata

```bash
sudo apt update
```

### Upgrade installed packages

```bash
sudo apt upgrade
```

### Inspect package details

```bash
apt show package-name
```

### Search for packages

```bash
apt search package-name
```

### Check installed package versions

```bash
dpkg -l package-name
```

### Why updates matter

Updates may provide:

- Security fixes.
- Bug fixes.
- Stability improvements.
- Compatibility changes.

### Update safely

- Read important package-manager messages.
- Back up important data.
- Review major changes on critical systems.
- Reboot when required by the update or system maintenance plan.
- Avoid installing packages from untrusted repositories.
- Remove obsolete software when it is no longer needed.

A newer version number alone does not prove that every vulnerability is fixed. Consider the distribution's security advisories and backported patches.

---

## 9. Network Security

Network security involves understanding which services are listening, which connections are active, and which hosts are reachable.

### Display IP addresses

```bash
ip addr
```

### Inspect routing

```bash
ip route
```

### List listening TCP and UDP sockets

```bash
ss -tuln
```

Useful flags:

- `-t`: TCP sockets.
- `-u`: UDP sockets.
- `-l`: Listening sockets.
- `-n`: Numeric addresses and ports.

Include process information where permissions allow:

```bash
sudo ss -tulpn
```

### Test a TCP port

```bash
nc -vz 192.0.2.10 22
```

### Inspect DNS resolution

```bash
getent hosts example.org
```

### Scan an authorized host with Nmap

```bash
nmap -sV -p 22,80,443 192.0.2.10
```

This checks selected ports and attempts service-version detection.

Use network scanning only on systems you own or are authorized to assess.

### Important distinction

- A **listening port** indicates a local service is waiting for connections.
- An **open port** reported by a remote scan indicates the scanner received a response consistent with a listening service.
- A **reachable service** may still reject authentication or application requests.
- A **filtered port** may be blocked or obscured by network controls.

Network exposure should be reviewed against the machine's intended role.

---

## 10. Firewall Basics

A firewall controls network traffic according to configured rules.

Linux systems may use tools such as UFW, nftables, or firewalld. Choose the tool appropriate for your distribution and existing configuration.

### UFW status

```bash
sudo ufw status verbose
```

### Allow SSH with UFW

```bash
sudo ufw allow OpenSSH
```

Alternatively, if the profile is unavailable:

```bash
sudo ufw allow 22/tcp
```

### Allow a specific TCP port

```bash
sudo ufw allow 443/tcp
```

### Remove a rule

```bash
sudo ufw status numbered
sudo ufw delete RULE_NUMBER
```

Confirm the rule number and its effect before deleting it.

### Enable UFW

```bash
sudo ufw enable
```

**Warning:** Before enabling or modifying a firewall on a remote system, confirm that the required management connection is permitted. Incorrect rules can lock you out.

### Firewall best practices

- Permit only required inbound traffic.
- Restrict administrative ports to trusted networks where practical.
- Remove obsolete rules.
- Review both host and network firewall controls.
- Test changes carefully.
- Do not assume a running firewall is correctly configured.

A firewall is one security layer, not a replacement for secure applications, authentication, or patching.

---

## 11. SSH Security

SSH provides encrypted remote administration and secure file transfer.

### Check the client version

```bash
ssh -V
```

### Inspect SSH server status

```bash
systemctl status ssh
```

### Validate server configuration

```bash
sudo sshd -t
```

### Important SSH configuration settings

The server configuration is commonly located at:

```text
/etc/ssh/sshd_config
```

Some systems also use files in `/etc/ssh/sshd_config.d/`.

Examples of security-related settings include:

```text
PermitRootLogin no
PasswordAuthentication no
```

These settings must be chosen according to the environment. Disabling password authentication before confirming working alternative access may lock out legitimate users.

### SSH security checklist

- Verify server host keys.
- Protect private keys with appropriate permissions and passphrases.
- Remove unused public keys from authorized accounts.
- Restrict SSH access to necessary users.
- Use least privilege.
- Keep OpenSSH patched.
- Review authentication logs.
- Restrict port forwarding when it is unnecessary.
- Test configuration changes before closing existing sessions.

For more detail, see the project's [SSH Notes](sshnotes.md), if that file exists in the same repository.

---

## 12. Logs and Monitoring

Logs record events from the operating system, services, and applications.

They can help investigate failed logins, service errors, configuration changes, and unexpected activity.

### View systemd journal entries

```bash
journalctl
```

### Show recent logs

```bash
journalctl -n 100
```

### Follow logs in real time

```bash
journalctl -f
```

### Inspect SSH service logs

```bash
sudo journalctl -u ssh
```

The service may be named `sshd` on some systems.

### Inspect authentication logs

On many Debian-based systems:

```bash
sudo less /var/log/auth.log
```

On some Red Hat-based systems:

```bash
sudo less /var/log/secure
```

Log locations vary by distribution and configuration.

### Filter for failed authentication messages

```bash
sudo grep -i "failed" /var/log/auth.log
```

This is a simple text search, not a complete security detector. The wording and file location can vary.

### What to investigate

Potentially relevant events include:

- Repeated failed authentication attempts.
- Unexpected successful logins.
- New or modified privileged accounts.
- Unusual service restarts.
- Changes to critical configuration files.
- Unexpected network listeners.
- Repeated application errors.

One event rarely proves a compromise. Correlate timestamps, users, processes, network activity, and system context.

### Protect logs

- Restrict access to sensitive logs.
- Configure retention appropriate to operational needs.
- Preserve relevant logs during an investigation.
- Consider centralized logging for important systems.
- Protect logs from unauthorized modification.

---

## 13. File Integrity and Hashing

A cryptographic hash produces a fixed-length digest of input data. Hashes can help compare files and detect changes.

### Calculate a SHA-256 hash

```bash
sha256sum filename
```

### Verify a file against a trusted checksum

If a trusted checksum file is available:

```bash
sha256sum -c checksums.txt
```

The checksum itself must come from a trustworthy source. An attacker who can replace both the file and its checksum may defeat this check.

### Compare two files

```bash
sha256sum file1 file2
```

Matching SHA-256 digests provide strong evidence that the file contents are identical, assuming no practical hash collision has been engineered.

### File integrity monitoring

For important files, compare current hashes against a securely stored baseline.

Examples of files that may deserve protection include:

- SSH server configuration.
- Critical application configuration.
- Administrative scripts.
- System service definitions.
- Important executable files.

Hashing detects differences; it does not automatically explain whether a change is malicious or legitimate.

---

## 14. Sensitive Files and Secrets

Linux systems contain configuration files, credentials, keys, and account information that require appropriate protection.

### Important paths

| Path | Purpose |
|---|---|
| `/etc/passwd` | User account metadata |
| `/etc/shadow` | Password hash and account-aging information on typical Linux systems |
| `/etc/group` | Group metadata |
| `/etc/sudoers` | Sudo policy |
| `/etc/ssh/sshd_config` | Common SSH server configuration |
| `~/.ssh/` | Per-user SSH configuration and keys |
| `~/.bash_history` | Bash history, if enabled and saved |
| `/var/log/` | Common location for system and application logs |
| `/etc/` | System-wide configuration directory |

### Protect secrets

- Never commit passwords, private keys, API tokens, or database credentials to public repositories.
- Do not include real credentials in screenshots or example output.
- Use environment variables or dedicated secret-management systems where appropriate.
- Restrict access to configuration files containing secrets.
- Rotate exposed credentials promptly.
- Avoid placing secrets in command-line arguments when they may be visible to other processes or recorded in logs.

### Check for accidentally committed secrets

Before publishing a repository, inspect files and commit history for credentials.

A secret removed from the latest version of a file may still exist in earlier Git commits. If a real credential was exposed, revoke or rotate it; deleting the visible file is not sufficient.

For public learning notes, use placeholders such as:

```text
API_KEY=replace_with_your_own_key
```

Never use a real secret as an example.

---

## 15. Linux Security Hardening

Hardening reduces unnecessary risk by making the system more restrictive and easier to monitor.

### 15.1 Keep the system updated

```bash
sudo apt update
sudo apt upgrade
```

Review changes and reboot when required.

### 15.2 Review users and groups

```bash
getent passwd
getent group
```

Check privileged access and remove obsolete accounts through the appropriate administrative process.

### 15.3 Review listening services

```bash
sudo ss -tulpn
```

Identify services that should not be exposed and disable them only after confirming their purpose.

### 15.4 Check file permissions

```bash
ls -l
namei -l /path/to/file
```

Pay particular attention to sensitive configuration, private keys, scripts, and writable directories.

### 15.5 Protect SSH

Use strong authentication, verified host keys, least privilege, and appropriate access restrictions.

### 15.6 Configure a firewall

Allow required traffic and deny unnecessary exposure according to the system's role.

### 15.7 Review logs

```bash
journalctl -p warning
```

This shows journal entries at warning priority and above, subject to journal configuration and access.

### 15.8 Back up important data

A backup should be protected from unauthorized modification and periodically tested.

Consider keeping a backup that cannot be modified by the same account or system that could be compromised.

### 15.9 Reduce attack surface

Remove or disable software and services that are not needed. Avoid unnecessary packages, exposed management interfaces, and excessive privileges.

### 15.10 Use mandatory access control where appropriate

Systems may use security frameworks such as AppArmor or SELinux to enforce additional restrictions on applications.

These controls require correct policies and maintenance; their mere presence does not guarantee that a system is secure.

### 15.11 Maintain secure defaults

- Avoid world-writable sensitive files.
- Avoid running applications as root unnecessarily.
- Keep secrets out of source control.
- Review third-party scripts before executing them.
- Use trusted package sources.
- Document changes to important system configuration.

---

## 16. Common Security Tools

### `ss`

Inspect network sockets:

```bash
ss -tulpn
```

### `ps`

Inspect processes:

```bash
ps aux
```

### `journalctl`

Review systemd journal entries:

```bash
journalctl -u ssh
```

### `find`

Locate files matching a condition:

```bash
find /etc -type f -name "*.conf"
```

Searches across restricted directories may produce permission errors. Do not suppress errors automatically if you need a complete audit.

### `grep`

Search text:

```bash
grep -Rni "password" ./config/
```

This can help review your own configuration directory, but matching the word `password` does not prove that a secret exists.

### `sha256sum`

Calculate file hashes:

```bash
sha256sum filename
```

### `getcap`

Inspect file capabilities:

```bash
getcap -r /usr/bin /usr/sbin 2>/dev/null
```

### `find` and permission review

List world-writable files in a directory you administer:

```bash
find /path/to/directory -type f -perm -0002 -ls
```

The result requires interpretation. Some writable files are intentional, and access may also depend on ACLs, mount options, and other controls.

### Nmap

Inspect authorized network services:

```bash
nmap -sV -p 22,80,443 192.0.2.10
```

Use Nmap only within an authorized scope.

---

## 17. Incident Investigation Basics

If you suspect that a Linux machine has been compromised, avoid immediately deleting suspicious files or wiping logs. Evidence can be important for understanding the incident.

### Initial checks

Record the time and scope of the observation.

Check logged-in users:

```bash
who
w
```

Check processes:

```bash
ps aux
```

Check listening ports:

```bash
sudo ss -tulpn
```

Review recent journal events:

```bash
journalctl -n 200
```

Inspect relevant authentication logs, where available:

```bash
sudo journalctl -u ssh
```

### Investigative principles

- Preserve relevant evidence.
- Record commands and observations.
- Avoid unnecessary changes to the affected system.
- Restrict access if ongoing harm is occurring.
- Follow the organization's incident-response process.
- Rotate credentials from a trusted device when compromise is suspected.
- Seek qualified assistance for critical systems.

If an incident is active, containment and evidence preservation may require different priorities depending on the environment. Do not assume one generic command sequence is suitable for every incident.

---

## 18. Common Misconfigurations

### Using `chmod 777` to fix everything

This can give unintended users write access. Set permissions according to the actual access requirements.

### Running everything as root

This increases the potential impact of mistakes and compromised applications. Use an ordinary account and elevate only when needed.

### Leaving unnecessary services enabled

Every exposed service can add maintenance and security responsibilities. Disable services only after confirming that they are not required.

### Ignoring updates

Known vulnerabilities may remain exploitable on outdated systems. Apply appropriate security updates.

### Trusting every SSH host key

Unexpected host-key changes require investigation. Blindly accepting new keys can expose users to impersonation.

### Publishing credentials to GitHub

Private keys, tokens, and passwords should never be committed. Revoke exposed secrets immediately and check whether they remain in history.

### Treating logs as definitive proof

Logs can be incomplete, rotated, altered, or unavailable. Correlate multiple sources and consider system context.

### Assuming a firewall guarantees security

A firewall cannot fix vulnerable applications, weak credentials, excessive permissions, or compromised accounts by itself.

### Copying commands without understanding them

Commands run with `sudo`, recursive operations, firewall changes, and permission modifications can cause significant damage. Understand their purpose and target paths first.

---

## 19. Security Checklist

Use this checklist when reviewing a Linux machine you own or administer.

### Accounts and permissions

- [ ] Review user accounts and group memberships.
- [ ] Remove or disable obsolete accounts appropriately.
- [ ] Review sudo privileges.
- [ ] Check sensitive file and directory permissions.
- [ ] Protect private keys and credentials.
- [ ] Avoid unnecessary root access.

### Software and services

- [ ] Apply appropriate security updates.
- [ ] Review installed software.
- [ ] Inspect active services.
- [ ] Identify listening network ports.
- [ ] Disable unnecessary services after assessing their purpose.

### Network and SSH

- [ ] Review firewall rules.
- [ ] Verify SSH host keys.
- [ ] Use strong authentication.
- [ ] Restrict administrative access.
- [ ] Review SSH logs.
- [ ] Restrict forwarding features when unnecessary.

### Monitoring and recovery

- [ ] Review important logs.
- [ ] Protect logs from unauthorized access.
- [ ] Maintain backups.
- [ ] Test backup restoration.
- [ ] Document important configuration changes.
- [ ] Have a recovery plan for compromised credentials or systems.

---

## 20. Practical Exercises

Complete these exercises on a personal Linux machine or an authorized virtual lab.

### Beginner

- [ ] Run `whoami` and `id`; explain the output.
- [ ] Inspect `/etc/passwd` and explain its fields.
- [ ] Use `ls -l` to interpret file permissions.
- [ ] Create a test file and change its permissions using `chmod`.
- [ ] Calculate its SHA-256 hash.
- [ ] Inspect running processes with `ps aux`.

### Intermediate

- [ ] Create a test user in a virtual machine.
- [ ] Inspect the user's group memberships.
- [ ] Review your own `sudo -l` output.
- [ ] List listening ports using `ss`.
- [ ] Inspect SSH authentication logs.
- [ ] Check the permissions of your `~/.ssh` directory.
- [ ] Review which services start automatically.
- [ ] Write a short script that records selected system information without changing system configuration.

### Advanced

- [ ] Create a security checklist for a fresh Linux virtual machine.
- [ ] Review firewall rules and document their purpose.
- [ ] Compare expected and actual file permissions in a test directory.
- [ ] Identify files with special permission bits and research why they exist.
- [ ] Review the SSH configuration of a machine you administer.
- [ ] Develop a small log-analysis script that counts failed login events in a sample log file.
- [ ] Write an incident-investigation report using a simulated scenario.

### Reflection questions

1. What is the difference between authentication and authorization?
2. Why is running an application as root risky?
3. What is the difference between file permissions and ownership?
4. Why is `chmod 777` usually a poor general solution?
5. What does a listening port tell you, and what does it not tell you?
6. Why are security updates important?
7. Why is a firewall only one layer of defense?
8. What should you do if a private SSH key is exposed?
9. Why can a hash comparison help detect file changes?
10. Why should suspicious events be investigated rather than automatically treated as proof of compromise?

---

## 21. Quick Reference Cheat Sheet

| Task | Command |
|---|---|
| Current user | `whoami` |
| User and group IDs | `id` |
| Logged-in users | `who` |
| User sessions and activity | `w` |
| File permissions | `ls -l` |
| Path permission inspection | `namei -l /path/to/file` |
| Change permissions | `chmod 640 filename` |
| Change ownership | `sudo chown user:group filename` |
| Inspect sudo privileges | `sudo -l` |
| List processes | `ps aux` |
| Show listening ports | `sudo ss -tulpn` |
| Show IP addresses | `ip addr` |
| Show routing table | `ip route` |
| Check service status | `systemctl status service-name` |
| View system logs | `journalctl -n 100` |
| View SSH logs | `sudo journalctl -u ssh` |
| Refresh package metadata | `sudo apt update` |
| Upgrade packages | `sudo apt upgrade` |
| Check UFW status | `sudo ufw status verbose` |
| Calculate SHA-256 hash | `sha256sum filename` |
| Inspect file capabilities | `getcap -r /usr/bin /usr/sbin 2>/dev/null` |
| Validate SSH server config | `sudo sshd -t` |

---

## 22. Glossary

| Term | Definition |
|---|---|
| Authentication | Verifying an identity |
| Authorization | Determining what an identity can do |
| Attack surface | The set of components or interfaces that may be exposed to attack |
| Capability | A specific privileged operation available to a process |
| Defense in depth | Using multiple complementary security controls |
| Firewall | A control that filters network traffic |
| GID | Numeric group identifier |
| Hardening | Reducing unnecessary risk through secure configuration |
| Hash | Fixed-length digest derived from data |
| Least privilege | Granting only the permissions necessary |
| Log | A record of system or application events |
| Root | The Linux superuser, normally UID 0 |
| SUID | A permission bit that can affect the effective user identity of an executable |
| SGID | A permission bit that affects group execution identity or directory inheritance |
| SSH | Secure Shell, a protocol for protected remote access |
| UID | Numeric user identifier |
| Vulnerability | A weakness that can affect confidentiality, integrity, or availability |

---

## Final Notes

Linux security is an ongoing process rather than a one-time configuration task. Good security depends on understanding permissions, managing accounts carefully, keeping software updated, limiting exposed services, protecting secrets, monitoring logs, and maintaining recoverable backups.

**Remember:** Understand a command before running it with elevated privileges, investigate unexpected changes, and test security controls in an authorized environment.

These notes are educational and should be adapted to the distribution, operational requirements, and threat model of the system being protected.
