# 🔐 Linux Files & Permissions

A practical guide to Linux file permissions, ownership, special permissions, and security fundamentals.

## 📑 Contents

- [1. File Permissions](#1-file-permissions)
- [2. Understanding Permission Symbols](#2-understanding-permission-symbols)
- [3. The `chmod` Command](#3-the-chmod-command)
- [4. Numeric Permissions](#4-numeric-permissions)
- [5. The `chown` Command](#5-the-chown-command)
- [6. The `chgrp` Command](#6-the-chgrp-command)
- [7. Special Permissions](#7-special-permissions)
- [8. The `umask` Command](#8-the-umask-command)
- [9. Finding Files](#9-finding-files)
- [10. File Attributes](#10-file-attributes)
- [11. Linux Capabilities](#11-linux-capabilities)
- [12. Useful Permission Commands](#12-useful-permission-commands)
- [13. Practical Example](#13-practical-example)
- [14. Cybersecurity Notes](#14-cybersecurity-notes)

---

## 1. File Permissions

Linux uses permissions to control who can read, modify, or execute files and directories.

Check permissions with:

```bash
ls -l
```

Example output:

```text
-rwxr-xr-- 1 Tuncay developers 1200 Oct 20 18:30 script.sh
```

The permission string is:

```text
-rwxr-xr--
```

Linux permissions are divided into three main categories:

| Category | Meaning |
|---|---|
| User (`u`) | The file owner |
| Group (`g`) | Users belonging to the file's group |
| Others (`o`) | Everyone else |

Each category can have different permissions.

### Permission types

| Permission | Symbol | Meaning for files | Meaning for directories |
|---|---|---|---|
| Read | `r` | Read file contents | List directory entries |
| Write | `w` | Modify file contents | Create, delete, or rename entries, subject to other restrictions |
| Execute | `x` | Execute the file if it is executable | Traverse or access entries by name, subject to other permissions |

**Important:** Directory permissions work differently from file permissions. For example, reading a directory and traversing it are separate operations.

---

## 2. Understanding Permission Symbols

Consider this example:

```text
-rwxr-xr--
```

Break it into sections:

```text
- | rwx | r-x | r--
  | user|group|others
```

- `-` — Regular file.
- `rwx` — Owner can read, write, and execute.
- `r-x` — Group can read and execute.
- `r--` — Others can only read.

Another example:

```text
drwxr-xr-x
```

The first character `d` indicates that the entry is a directory.

Common file-type indicators:

| Symbol | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |

To inspect a particular file:

```bash
ls -l script.sh
```

For additional metadata:

```bash
stat script.sh
```

`stat` displays information such as file size, ownership, permissions, and timestamps.

---

## 3. The `chmod` Command

`chmod` changes file or directory permissions.

### Symbolic mode

```bash
chmod u+x script.sh
```

Adds execute permission for the owner.

Other examples:

```bash
chmod u-w file.txt
chmod g+r file.txt
chmod o-r file.txt
chmod a+x script.sh
```

| Symbol | Meaning |
|---|---|
| `u` | Owner |
| `g` | Group |
| `o` | Others |
| `a` | All categories |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set permissions explicitly |

For example:

```bash
chmod g=rx directory
```

Sets the group's permissions to read and execute, removing any group write permission.

### Recursive permissions

```bash
chmod -R 755 directory
```

The `-R` option applies changes recursively to the directory and its contents.

**Warning:** Recursive permission changes can unintentionally expose files or break applications. Inspect the target before using `-R`.

---

## 4. Numeric Permissions

Linux also represents permissions with numbers.

| Permission | Value |
|---|---:|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |

Add the values together to obtain each category's permission number.

| Number | Permissions |
|---:|---|
| `0` | `---` |
| `1` | `--x` |
| `2` | `-w-` |
| `3` | `-wx` |
| `4` | `r--` |
| `5` | `r-x` |
| `6` | `rw-` |
| `7` | `rwx` |

### Common examples

```bash
chmod 755 script.sh
chmod 644 notes.txt
chmod 700 private.sh
chmod 600 secret.txt
```

| Mode | Owner | Group | Others |
|---|---|---|---|
| `755` | `rwx` | `r-x` | `r-x` |
| `644` | `rw-` | `r--` | `r--` |
| `700` | `rwx` | `---` | `---` |
| `600` | `rw-` | `---` | `---` |

### How does `755` work?

- Owner: `4 + 2 + 1 = 7` → `rwx`
- Group: `4 + 0 + 1 = 5` → `r-x`
- Others: `4 + 0 + 1 = 5` → `r-x`

Therefore:

```bash
chmod 755 script.sh
```

gives the owner full permissions and gives the group and others read and execute permissions.

**Security tip:** Avoid granting write permissions to everyone unless they are genuinely required.

---

## 5. The `chown` Command

`chown` changes a file's owner and optionally its group.

Check current ownership:

```bash
ls -l notes.txt
```

Change the owner:

```bash
sudo chown Tuncay notes.txt
```

Change the owner and group:

```bash
sudo chown Tuncay:developers notes.txt
```

Change ownership recursively:

```bash
sudo chown -R Tuncay:developers directory
```

The user and group must exist, and changing ownership usually requires elevated privileges.

**Important:** Incorrect recursive ownership changes can break system services or applications. Avoid applying them to system directories without understanding the consequences.

---

## 6. The `chgrp` Command

`chgrp` changes a file's group ownership.

```bash
sudo chgrp developers notes.txt
```

Check the result:

```bash
ls -l notes.txt
```

For a directory and its contents:

```bash
sudo chgrp -R developers project/
```

As with `chown -R`, use recursive changes carefully.

---

## 7. Special Permissions

Linux has three special permission bits: SUID, SGID, and the sticky bit.

These can affect how programs execute or how directories behave.

### 7.1 SUID — Set User ID

SUID is represented by the numeric value `4000`.

On an executable file, it causes the program to run with the effective user ID of the file's owner, subject to operating-system and filesystem behavior.

Example:

```text
-rwsr-xr-x 1 root root 12345 Oct 20 18:30 example
```

The `s` in the owner's execute position indicates SUID.

Find SUID files:

```bash
find / -type f -perm -4000 2>/dev/null
```

Breakdown:

- `/` — Search from the filesystem root.
- `-type f` — Match regular files.
- `-perm -4000` — Match files with the SUID bit set.
- `2>/dev/null` — Suppress error messages sent to standard error.

SUID is important in Linux privilege-escalation labs because incorrectly configured or vulnerable privileged executables can create security risks.

**Remember:** An SUID bit alone does not mean a file is exploitable. The executable's behavior, ownership, permissions, and system configuration all matter.

### 7.2 SGID — Set Group ID

SGID is represented by the numeric value `2000`.

On an executable file, SGID can make the process run with the file's group as its effective group.

On a directory, SGID generally causes new entries created inside it to inherit the directory's group.

Example:

```text
drwxr-sr-x 2 root developers 4096 Oct 20 18:30 shared
```

The `s` in the group's execute position indicates SGID.

Find SGID files:

```bash
find / -type f -perm -2000 2>/dev/null
```

Set SGID on a directory:

```bash
chmod 2775 shared
```

This can be useful for shared project directories where files should inherit a common group.

### 7.3 Sticky Bit

The sticky bit is represented by the numeric value `1000`.

On a shared writable directory, it normally restricts deletion or renaming of entries to the entry's owner, the directory's owner, or a privileged process.

The `/tmp` directory commonly uses this permission.

Example:

```text
drwxrwxrwt 10 root root 4096 Oct 20 18:30 /tmp
```

The `t` in the others' execute position indicates the sticky bit.

Set the sticky bit:

```bash
chmod 1777 shared
```

**Note:** A lowercase `s` or `t` means the corresponding execute permission is also set. An uppercase `S` or `T` indicates that the special bit is set without the corresponding execute bit.

---

## 8. The `umask` Command

`umask` determines which permission bits are masked out when new files and directories are created.

Check the current value:

```bash
umask
```

Example:

```text
0022
```

With a typical `umask` of `022`:

- New files commonly start with permissions `644`.
- New directories commonly start with permissions `755`.

The defaults used for creation are normally `666` for regular files and `777` for directories, before applying the mask. Programs may choose different modes.

Set the mask for the current shell:

```bash
umask 027
```

With this mask, typical new-file permissions are `640`, and typical new-directory permissions are `750`.

**Security tip:** A restrictive `umask` can help prevent newly created files from being accessible to unintended users.

---

## 9. Finding Files

The `find` command searches for files and directories based on specified conditions.

Find a file by name:

```bash
find /home/Tuncay -name "notes.txt" 2>/dev/null
```

Find regular files:

```bash
find /home/Tuncay -type f
```

Find directories:

```bash
find /home/Tuncay -type d
```

Find files with particular permission bits:

```bash
find / -type f -perm -4000 2>/dev/null
find / -type f -perm -2000 2>/dev/null
```

Find files writable by the current user:

```bash
find /home/Tuncay -type f -writable
```

Find files modified in the last day:

```bash
find /home/Tuncay -type f -mtime -1
```

Useful options:

| Option | Meaning |
|---|---|
| `-name` | Match a name or pattern |
| `-type f` | Regular files |
| `-type d` | Directories |
| `-perm` | Match permission bits |
| `-writable` | Match files writable by the current user |
| `-mtime` | Filter by modification time in days |

**Note:** Search results depend on your permissions and the filesystem being searched. You may need elevated privileges to inspect certain locations.

---

## 10. File Attributes

The `lsattr` and `chattr` commands inspect and modify certain Linux filesystem attributes when supported by the filesystem.

Display attributes:

```bash
lsattr notes.txt
```

The immutable attribute, when supported, can prevent a file from being modified, deleted, or renamed until the attribute is removed.

Set immutable:

```bash
sudo chattr +i notes.txt
```

Remove immutable:

```bash
sudo chattr -i notes.txt
```

**Warning:** Use these commands only on files you own or are authorized to administer. Filesystem support and required privileges vary.

---

## 11. Linux Capabilities

Linux capabilities divide certain privileged operations into separate units rather than relying exclusively on the all-powerful root user.

Inspect a program's capabilities:

```bash
getcap -r /usr/bin /usr/local/bin 2>/dev/null
```

Example output:

```text
/usr/bin/example = cap_net_raw+ep
```

This is an illustrative example; the actual output depends on the system.

Some capability names:

| Capability | General purpose |
|---|---|
| `CAP_NET_RAW` | Permits certain raw and packet socket operations |
| `CAP_SETUID` | Permits changes to process user IDs, subject to capability rules |
| `CAP_NET_BIND_SERVICE` | Permits binding to privileged Internet-domain ports |
| `CAP_DAC_OVERRIDE` | Bypasses certain discretionary file permission checks |

Inspect the capabilities of a specific executable:

```bash
getcap /path/to/program
```

Capabilities can be useful for granting programs limited privileges, but an unnecessarily powerful capability may create a security weakness.

**Cybersecurity note:** During authorized labs, inspect unusual capabilities and understand what the associated executable actually does before drawing conclusions.

---

## 12. Useful Permission Commands

| Command | Purpose |
|---|---|
| `ls -l` | Inspect permissions and ownership |
| `stat file` | Display detailed file metadata |
| `chmod` | Change permissions |
| `chown` | Change owner and optionally group |
| `chgrp` | Change group ownership |
| `umask` | Inspect or set the permission mask |
| `find` | Search for files and directories |
| `getfacl` | Display access control lists when supported |
| `setfacl` | Modify access control lists when supported |
| `getcap` | Inspect file capabilities |
| `lsattr` | Display supported filesystem attributes |

### ACLs

Access Control Lists (ACLs) allow additional user or group permissions beyond the standard owner/group/others model.

Inspect ACLs:

```bash
getfacl notes.txt
```

Set an additional read permission for a user:

```bash
setfacl -m u:alice:r notes.txt
```

Remove that user's ACL entry:

```bash
setfacl -x u:alice notes.txt
```

ACL support depends on the filesystem and system configuration.

---

## 13. Practical Example

This exercise creates a file and changes its permissions.

```bash
mkdir -p ~/permissions-lab
cd ~/permissions-lab

touch notes.txt
ls -l notes.txt

chmod 600 notes.txt
ls -l notes.txt

chmod 644 notes.txt
ls -l notes.txt
```

Expected permission strings:

```text
-rw-------
-rw-r--r--
```

The owner retains read and write access. The second configuration also allows group members and others to read the file.

Now create a script:

```bash
printf '#!/bin/sh\necho "Hello, Tuncay!"\n' > hello.sh
chmod 700 hello.sh
./hello.sh
```

Expected output:

```text
Hello, Tuncay!
```

The script is executable by its owner. Its permissions do not grant access to other users.

Clean up the practice directory when finished:

```bash
cd ~
rm -r ~/permissions-lab
```

Only run the cleanup command after confirming that the directory contains nothing you need.

---

## 14. Cybersecurity Notes

When investigating permissions in an authorized Linux lab, consider these questions:

1. Who owns the file?
2. Which group owns it?
3. Can the current user read, modify, or execute it?
4. Is it an SUID or SGID executable?
5. Does it have unusual Linux capabilities?
6. Is the directory writable by unintended users?
7. Are ACLs granting additional permissions?
8. Does the executable's behavior introduce a real security issue?

Useful enumeration commands:

```bash
id
ls -la
find / -type f -perm -4000 2>/dev/null
find / -type f -perm -2000 2>/dev/null
getcap -r /usr/bin /usr/local/bin 2>/dev/null
```

These commands identify information for further investigation; they do not automatically prove that a privilege-escalation path exists.

---

## 🎯 Learning Goals

- Understand Linux ownership and permission models.
- Read and modify permissions using `chmod`.
- Change ownership using `chown` and `chgrp`.
- Understand SUID, SGID, and sticky bits.
- Find files and inspect their metadata.
- Recognize Linux capabilities and ACLs.
- Apply permission enumeration in authorized cybersecurity labs.

## 📖 Resources

- [Linux man-pages](https://man7.org/linux/man-pages/)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/)
- [HTB Academy](https://academy.hackthebox.com/)

---

[← Back to Linux Notes](README.md)

**Author:** Tuncay Sarxanlı
