# 👥 Linux Users & Processes

A practical guide to Linux users, groups, privileges, processes, and system services.

## 📑 Contents

- [1. User Information](#1-user-information)
- [2. Users and Groups](#2-users-and-groups)
- [3. User Accounts](#3-user-accounts)
- [4. The `sudo` Command](#4-the-sudo-command)
- [5. Process Basics](#5-process-basics)
- [6. Managing Processes](#6-managing-processes)
- [7. Process Priorities](#7-process-priorities)
- [8. System Services](#8-system-services)
- [9. Environment Variables](#9-environment-variables)
- [10. Scheduled Tasks](#10-scheduled-tasks)
- [11. Useful Enumeration Commands](#11-useful-enumeration-commands)
- [12. Practical Example](#12-practical-example)

---

## 1. User Information

### `whoami`

Displays the current username.

```bash
whoami
```

Example output:

```text
Tuncay
```

### `id`

Displays the current user's UID, primary GID, and group memberships.

```bash
id
```

Example output:

```text
uid=1000(Tuncay) gid=1000(Tuncay) groups=1000(Tuncay),27(sudo)
```

Actual output depends on your system.

### `who`

Shows users with active login sessions.

```bash
who
```

### `w`

Displays logged-in users and information about their current activity.

```bash
w
```

### `logname`

Displays the login name associated with the current session.

```bash
logname
```

This may behave differently in containers, scripts, or sessions without a conventional login record.

---

## 2. Users and Groups

Linux uses users and groups to manage access to files, processes, and system resources.

### Important concepts

| Term | Meaning |
|---|---|
| UID | User ID |
| GID | Group ID |
| Root | The superuser, normally UID `0` |
| Primary group | The user's main group |
| Supplementary groups | Additional groups the user belongs to |

### Display user information

```bash
id
id root
```

### List groups

```bash
groups
groups Tuncay
```

### View local user accounts

```bash
cat /etc/passwd
```

Example entry:

```text
Tuncay:x:1000:1000:Tuncay:/home/Tuncay:/bin/bash
```

The fields are separated by colons:

1. Username
2. Password placeholder or marker
3. UID
4. Primary GID
5. Description or GECOS field
6. Home directory
7. Login shell

The `x` generally indicates that password hashes are stored separately, commonly in `/etc/shadow`.

**Important:** `/etc/passwd` is normally readable by regular users and does not ordinarily contain the actual password hashes.

### View group definitions

```bash
cat /etc/group
```

Example:

```text
developers:x:1001:Tuncay
```

The fields are the group name, password placeholder, GID, and optional member list.

A user's primary group may not be explicitly listed among the members in `/etc/group`, so use `id username` for a more complete view.

### View password-aging information

```bash
sudo chage -l Tuncay
```

Displays password-aging information for the specified account, subject to permissions and system configuration.

---

## 3. User Accounts

User management commands are generally intended for administrators and may require root privileges.

### `useradd`

Creates a user account.

```bash
sudo useradd -m student
```

The `-m` option creates a home directory when needed.

### `passwd`

Sets or changes an account password.

```bash
sudo passwd student
```

To change your own password:

```bash
passwd
```

### `usermod`

Modifies an existing user account.

For example, add a user to a supplementary group:

```bash
sudo usermod -aG developers student
```

- `-a` — Append to existing supplementary groups.
- `-G` — Specify supplementary groups.

**Important:** Omitting `-a` when using `-G` can replace the user's existing supplementary group memberships.

### `userdel`

Deletes a user account.

```bash
sudo userdel student
```

Delete the account and its home directory:

```bash
sudo userdel -r student
```

Use account-management commands carefully, especially on systems with real users or running services.

### `su`

Switches to another user.

```bash
su - student
```

The `-` option starts a login-style shell and loads the target user's login environment.

To return to the previous shell:

```bash
exit
```

---

## 4. The `sudo` Command

`sudo` allows authorized users to execute commands with elevated privileges, according to the system's sudoers configuration.

### Run a command as root

```bash
sudo whoami
```

If authorized, the output is typically:

```text
root
```

### Open a root shell

```bash
sudo -i
```

Exit the root shell:

```bash
exit
```

### Check your sudo permissions

```bash
sudo -l
```

This lists the commands the current user is allowed to run through `sudo`, subject to policy and authentication requirements.

### Why `sudo -l` matters

In authorized security labs, sudo rules can reveal whether a user has access to administrative commands.

However, permission to run a command as root does not automatically mean that the system is vulnerable. The allowed command, its arguments, and its behavior must be examined.

**Security tip:** Give users only the administrative permissions they actually need.

---

## 5. Process Basics

A process is a running instance of a program.

Linux assigns each process a Process ID (PID).

### `ps`

Displays information about processes.

```bash
ps
ps aux
```

Common columns in `ps aux` include:

| Column | Meaning |
|---|---|
| `USER` | Process owner |
| `PID` | Process ID |
| `%CPU` | CPU usage |
| `%MEM` | Memory usage |
| `STAT` | Process state |
| `COMMAND` | Command associated with the process |

### Search for a process

```bash
ps aux | grep ssh
```

This searches the displayed process list for text containing `ssh`. The output may include the `grep` command itself.

An alternative is:

```bash
pgrep -a ssh
```

### `top`

Displays running processes and resource usage interactively.

```bash
top
```

Press `q` to exit.

### `htop`

Provides an interactive process viewer, if installed.

```bash
htop
```

### Process hierarchy

Display processes in a tree:

```bash
pstree -p
```

Or use:

```bash
ps -ef --forest
```

A process tree can help you understand which process launched another process.

---

## 6. Managing Processes

### `kill`

Sends a signal to a process.

```bash
kill PID
```

Replace `PID` with the actual process ID.

By default, `kill PID` sends `SIGTERM`, which asks the process to terminate gracefully.

### `kill -9`

```bash
kill -9 PID
```

Sends `SIGKILL`, which cannot be caught or ignored by the target process.

Use it only when necessary because the process cannot clean up normally.

### `pkill`

Sends a signal to processes selected by name or other supported criteria.

```bash
pkill -x example
```

This targets processes whose name exactly matches `example`.

Always verify the target before terminating processes.

### `jobs`

Displays jobs started by the current shell.

```bash
jobs
```

### Background execution

Run a command in the background:

```bash
sleep 300 &
```

Check its job status:

```bash
jobs
```

Bring a job to the foreground:

```bash
fg
```

Suspend a foreground process with `Ctrl + Z`, then resume it in the background:

```bash
bg
```

These job-control commands apply to processes managed by the current shell.

---

## 7. Process Priorities

Linux schedules processes according to several factors, including scheduling policy and priority.

### `nice`

Starts a command with a specified nice value.

```bash
nice -n 10 sleep 300
```

Nice values normally range from `-20` to `19`:

- `-20` — Higher scheduling priority in the traditional nice-value model.
- `0` — Default nice value.
- `19` — Lower scheduling priority.

Increasing priority by using a lower nice value generally requires additional privileges.

### `renice`

Changes the nice value of a running process.

```bash
renice 10 -p PID
```

Replace `PID` with the actual process ID.

**Note:** Nice values influence scheduling priority; they do not directly determine a process's permissions or whether it runs as root.

---

## 8. System Services

A service is a background program that provides functionality such as SSH access, networking, or logging.

Many modern Linux distributions use `systemd` to manage services.

### Check service status

```bash
systemctl status ssh
```

The service name can differ between distributions. For example, the SSH server service is often named `ssh` on Kali and Debian-based systems.

### Start a service

```bash
sudo systemctl start ssh
```

### Stop a service

```bash
sudo systemctl stop ssh
```

### Restart a service

```bash
sudo systemctl restart ssh
```

### Enable a service at boot

```bash
sudo systemctl enable ssh
```

### Disable a service at boot

```bash
sudo systemctl disable ssh
```

### List active services

```bash
systemctl list-units --type=service --state=running
```

### Check listening network sockets

```bash
ss -lntup
```

This displays listening TCP and UDP sockets and, when permitted, associated process information.

**Cybersecurity note:** Running services can reveal the software available on a machine. In authorized labs, identify the service, its configuration, and whether it needs to be exposed.

---

## 9. Environment Variables

Environment variables provide information to processes and influence program behavior.

### Display environment variables

```bash
env
```

### Display a specific variable

```bash
echo "$HOME"
echo "$USER"
echo "$PATH"
```

Common variables:

| Variable | Meaning |
|---|---|
| `HOME` | User's home directory |
| `USER` | Commonly identifies the current user |
| `PATH` | Directories searched for executable commands |
| `SHELL` | User's configured shell, where set |

### Inspect the command search path

```bash
echo "$PATH"
```

Directories are separated by colons.

### Set a variable for the current shell

```bash
MY_VARIABLE="hello"
echo "$MY_VARIABLE"
```

To make it available to child processes:

```bash
export MY_VARIABLE="hello"
```

The variable usually disappears when the shell exits unless it is configured in a startup file or another persistent configuration.

**Security note:** Environment variables can affect how programs behave. Avoid placing sensitive credentials in environment variables unless the application's security requirements and exposure risks are understood.

---

## 10. Scheduled Tasks

Scheduled tasks run commands automatically at specified times or intervals.

### `cron`

User crontab entries can be inspected with:

```bash
crontab -l
```

Edit the current user's crontab:

```bash
crontab -e
```

A cron entry has five time fields followed by a command:

```text
minute hour day-of-month month day-of-week command
```

Example:

```text
0 3 * * * /home/Tuncay/backup.sh
```

This schedules the script to run daily at 03:00 according to the cron daemon's time configuration, assuming the script is executable and the environment is suitable.

### System-wide scheduled tasks

Common locations include:

```bash
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
```

Some systems also use systemd timers.

**Cybersecurity note:** In an authorized lab, inspect scheduled scripts, their ownership, permissions, and the privileges under which they run. A privileged scheduled task that executes an improperly writable script can introduce a security weakness.

---

## 11. Useful Enumeration Commands

These commands help collect basic information about users, processes, and services.

### User information

```bash
whoami
id
groups
cat /etc/passwd
cat /etc/group
sudo -l
```

### Process information

```bash
ps aux
pgrep -a ssh
pstree -p
top
```

### Services

```bash
systemctl --type=service --state=running
ss -lntup
```

### Scheduled tasks

```bash
crontab -l
ls -la /etc/cron.d/
```

### Environment

```bash
env
echo "$PATH"
```

Commands may show different results depending on your permissions, installed software, and Linux distribution.

---

## 12. Practical Example

This exercise creates a small script and runs it as the current user.

```bash
mkdir -p ~/process-lab
cd ~/process-lab

printf '#!/bin/sh\necho "Current user:"\nwhoami\n' > user-info.sh
chmod 700 user-info.sh
./user-info.sh
```

Example output:

```text
Current user:
Tuncay
```

Inspect the script process while it runs:

```bash
sleep 120 &
jobs
pgrep -a sleep
```

Stop the test process by bringing the background job to the foreground or using its verified PID:

```bash
kill PID
```

Replace `PID` with the process ID reported by your system.

### What did we learn?

1. Linux identifies users with UIDs and groups with GIDs.
2. `sudo` can run commands with elevated privileges when authorized.
3. Processes have PIDs and can be inspected with `ps` or `pgrep`.
4. `kill` sends signals to processes.
5. `systemctl` manages services on systemd-based systems.
6. Environment variables influence the shell and child processes.
7. Scheduled tasks can automate system operations.

---

## 🎯 Learning Goals

- Understand Linux users, groups, and account information.
- Inspect and manage processes.
- Understand root privileges and `sudo`.
- Manage services with `systemctl`.
- Recognize environment variables and scheduled tasks.
- Practice enumeration in authorized Linux labs.

## 📖 Resources

- [Linux man-pages](https://man7.org/linux/man-pages/)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [systemd Documentation](https://www.freedesktop.org/software/systemd/man/)
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/)
- [HTB Academy](https://academy.hackthebox.com/)

---

[← Back to Linux Notes](README.md)

**Author:** Tuncay Sarxanlı
