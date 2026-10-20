# 🐧 Linux Basics

A practical guide to essential Linux commands, terminal usage, filesystem navigation, and basic file management.

## 📑 Contents

- [1. Current User](#1-current-user)
- [2. Current Directory](#2-current-directory)
- [3. Listing Files](#3-listing-files)
- [4. Directory Navigation](#4-directory-navigation)
- [5. Absolute and Relative Paths](#5-absolute-and-relative-paths)
- [6. Creating Files and Directories](#6-creating-files-and-directories)
- [7. Reading Files](#7-reading-files)
- [8. Copying, Moving, and Renaming](#8-copying-moving-and-renaming)
- [9. Removing Files and Directories](#9-removing-files-and-directories)
- [10. Command Help](#10-command-help)
- [11. Useful Terminal Shortcuts](#11-useful-terminal-shortcuts)
- [12. Basic System Information](#12-basic-system-information)
- [13. Pipes and Redirection](#13-pipes-and-redirection)
- [14. Important Filesystem Directories](#14-important-filesystem-directories)
- [15. Practical Example](#15-practical-example)

---

## 1. Current User

### `whoami`

Displays the username of the currently logged-in user.

```bash
whoami
```

Example output:

```text
Tuncay
```

### `id`

Displays the user's UID, GID, and group memberships.

```bash
id
```

Example output:

```text
uid=1000(Tuncay) gid=1000(Tuncay) groups=1000(Tuncay),27(sudo)
```

*Example output may differ depending on your system.*

- `uid` — User ID.
- `gid` — Primary group ID.
- `groups` — Groups the user belongs to.

### `users`

Displays usernames currently logged in.

```bash
users
```

### `who`

Shows information about logged-in sessions.

```bash
who
```

**Cybersecurity note:** Knowing which user you are and which groups you belong to is important when investigating permissions and privilege escalation in authorized labs.

---

## 2. Current Directory

### `pwd`

Prints the absolute path of your current working directory.

```bash
pwd
```

Example output:

```text
/home/Tuncay
```

### `hostname`

Displays the system's hostname.

```bash
hostname
```

### `uname`

Displays system information.

```bash
uname
uname -a
```

- `uname` — Shows the kernel name.
- `uname -a` — Displays additional kernel and system information.

---

## 3. Listing Files

### `ls`

Lists files and directories in the current directory.

```bash
ls
```

### Useful options

```bash
ls -l
ls -a
ls -lh
ls -lah
```

| Option | Meaning |
|---|---|
| `-l` | Long format with permissions, owner, size, and date |
| `-a` | Includes hidden files |
| `-h` | Human-readable file sizes |
| `-lah` | Combines all three options |

Example:

```bash
ls -lah
```

Example output:

```text
drwxr-xr-x  5 Tuncay Tuncay 4.0K Oct 20 19:00 .
drwxr-xr-x  3 root   root   4.0K Oct 20 10:00 ..
-rw-r--r--  1 Tuncay Tuncay  120 Oct 20 19:00 notes.txt
```

Important details:

- `d` at the beginning indicates a directory.
- `-` at the beginning indicates a regular file.
- `.` represents the current directory.
- `..` represents the parent directory.
- Names beginning with `.` are hidden by default.

**Tip:** Hidden files often contain configuration settings, so `ls -la` is especially useful when enumerating a Linux system.

---

## 4. Directory Navigation

### `cd`

Changes the current working directory.

```bash
cd /home
cd ..
cd ~
cd -
```

| Command | Meaning |
|---|---|
| `cd /home` | Moves to `/home` |
| `cd ..` | Moves to the parent directory |
| `cd ~` | Moves to your home directory |
| `cd -` | Returns to the previous working directory |
| `cd /` | Moves to the filesystem root |

Example workflow:

```bash
pwd
cd /tmp
pwd
cd -
pwd
```

### Tab completion

Start typing a file or directory name and press `Tab` to complete it.

```bash
cd Doc<Tab>
```

If the name is unambiguous, the terminal can complete it automatically.

---

## 5. Absolute and Relative Paths

A path identifies the location of a file or directory.

### Absolute path

Starts from the filesystem root `/`.

```bash
/home/Tuncay/notes.txt
```

### Relative path

Is interpreted relative to the current working directory.

```bash
notes.txt
../notes.txt
./script.sh
```

Special path symbols:

| Symbol | Meaning |
|---|---|
| `/` | Filesystem root |
| `~` | Current user's home directory |
| `.` | Current directory |
| `..` | Parent directory |

Example:

```bash
cd /home/Tuncay
cat notes.txt
```

Here, `notes.txt` is a relative path, while `/home/Tuncay/notes.txt` is an absolute path.

---

## 6. Creating Files and Directories

### `touch`

Creates an empty file if it does not exist. If the file already exists, it normally updates its modification timestamp.

```bash
touch notes.txt
touch file1.txt file2.txt
```

### `mkdir`

Creates a directory.

```bash
mkdir practice
```

Create nested directories:

```bash
mkdir -p projects/linux/basics
```

The `-p` option creates missing parent directories as needed.

### `file`

Identifies the type of a file.

```bash
file notes.txt
file /bin/ls
```

The result depends on the file's contents and format, not just its extension.

---

## 7. Reading Files

### `cat`

Displays file contents.

```bash
cat notes.txt
```

Display multiple files:

```bash
cat file1.txt file2.txt
```

### `less`

Reads a file page by page.

```bash
less /etc/passwd
```

Useful keys inside `less`:

- `Space` — Next page.
- `b` — Previous page.
- `/word` — Search for a word.
- `q` — Quit.

### `head`

Displays the first lines of a file.

```bash
head notes.txt
head -n 5 notes.txt
```

### `tail`

Displays the last lines of a file.

```bash
tail notes.txt
tail -n 20 notes.txt
```

Follow a log file as new lines are added:

```bash
tail -f /var/log/syslog
```

The log path and availability depend on the Linux distribution and logging configuration.

### `wc`

Counts lines, words, and bytes.

```bash
wc notes.txt
wc -l notes.txt
wc -w notes.txt
```

- `-l` — Lines.
- `-w` — Words.
- `-c` — Bytes.

---

## 8. Copying, Moving, and Renaming

### `cp`

Copies files.

```bash
cp notes.txt backup.txt
```

Copy a file to another directory:

```bash
cp notes.txt /tmp/
```

Copy directories recursively:

```bash
cp -r projects projects_backup
```

### `mv`

Moves or renames files and directories.

Rename a file:

```bash
mv oldname.txt newname.txt
```

Move a file:

```bash
mv notes.txt /tmp/
```

**Important:** `mv` can overwrite an existing destination in some circumstances. Check the destination before running commands that modify important files.

---

## 9. Removing Files and Directories

### `rm`

Removes files.

```bash
rm notes.txt
```

### `rmdir`

Removes an empty directory.

```bash
rmdir practice
```

### Recursive removal

```bash
rm -r old_project
```

- `-r` — Recursively removes a directory and its contents.

**⚠️ Warning:** `rm` generally does not move files to a recycle bin. Be especially careful with `rm -r`, `rm -f`, wildcards such as `*`, and commands involving elevated privileges.

Always verify the target path before deleting anything.

---

## 10. Command Help

Linux provides built-in documentation for commands.

### `man`

Opens a command's manual page.

```bash
man ls
man chmod
man grep
```

Press `q` to exit.

### `--help`

Many commands support a quick help option.

```bash
ls --help
cp --help
ip --help
```

### `whatis`

Displays a short description of a command when its manual database is available.

```bash
whatis ls
```

### `which` and `type`

```bash
which python3
type cd
type ls
```

- `which` — Locates an executable through the command search path.
- `type` — Explains how the shell interprets a command; it can identify aliases, built-ins, and executables.

**Learning tip:** When you encounter an unfamiliar command in a CTF, check its documentation instead of blindly copying it.

---

## 11. Useful Terminal Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + C` | Interrupts the foreground command |
| `Ctrl + L` | Clears the terminal display |
| `Ctrl + D` | Sends EOF; may exit the shell |
| `Ctrl + A` | Moves to the beginning of the command line |
| `Ctrl + E` | Moves to the end of the command line |
| `Ctrl + R` | Searches command history |
| `Tab` | Completes commands and paths |
| `↑` / `↓` | Navigates command history |

These shortcuts make terminal work faster and more efficient.

---

## 12. Basic System Information

### `date`

Displays the current date and time.

```bash
date
```

### `uptime`

Displays how long the system has been running and system load averages.

```bash
uptime
```

### `free`

Displays memory usage.

```bash
free -h
```

### `df`

Displays filesystem disk space usage.

```bash
df -h
```

### `du`

Estimates disk space used by files and directories.

```bash
du -sh ~/Downloads
```

### `ps`

Displays information about running processes.

```bash
ps
ps aux
```

### `top`

Shows running processes and resource usage interactively.

```bash
top
```

Press `q` to quit.

---

## 13. Pipes and Redirection

Linux allows commands to pass data between each other.

### Output redirection: `>`

Writes command output to a file, replacing its previous contents.

```bash
whoami > user.txt
```

Read the result:

```bash
cat user.txt
```

Expected output:

```text
Tuncay
```

### Append output: `>>`

Adds output to the end of a file.

```bash
date >> log.txt
```

### Input redirection: `<`

Uses a file as standard input.

```bash
wc -l < notes.txt
```

### Pipes: `|`

Sends one command's output to another command.

```bash
ls -lah | less
```

Example: search for a specific entry in a file.

```bash
cat /etc/passwd | grep root
```

A simpler alternative is:

```bash
grep root /etc/passwd
```

**Remember:** A pipe passes output between commands; it does not automatically save the output to a file.

---

## 14. Important Filesystem Directories

| Directory | Purpose |
|---|---|
| `/` | Root of the filesystem |
| `/home` | Home directories of regular users |
| `/root` | Home directory of the root user |
| `/etc` | System-wide configuration files |
| `/tmp` | Temporary files |
| `/var` | Variable data, including many logs |
| `/var/log` | Common location for system and application logs |
| `/usr/bin` | Many user commands and executables |
| `/bin` | Essential commands; may be linked to `/usr/bin` |
| `/opt` | Optional or third-party software |
| `/dev` | Device files |
| `/proc` | Virtual filesystem exposing process and kernel information |
| `/sys` | Kernel and device information |
| `/mnt` | Common mount point for temporary mounts |
| `/media` | Common mount point for removable media |

### Useful examples

```bash
ls /
ls /etc
ls /var/log
ls /home
```

**Cybersecurity note:** During authorized Linux enumeration, configuration files, logs, user directories, and process information can reveal useful information about the system.

Access restrictions still apply: not every file is readable by every user.

---

## 15. Practical Example

This workflow creates a small practice directory, writes a note, reads it, and checks the current user.

```bash
mkdir -p ~/linux-practice
cd ~/linux-practice
whoami
pwd
touch notes.txt
echo "Learning Linux fundamentals" > notes.txt
cat notes.txt
ls -lah
```

Example output from `whoami`:

```text
Tuncay
```

The other output will depend on your system and the files already present.

### What did we learn?

1. `mkdir -p` creates the practice directory.
2. `cd` enters that directory.
3. `whoami` identifies the current user.
4. `pwd` shows the current location.
5. `touch` creates a file.
6. `echo` writes text using output redirection.
7. `cat` displays the file contents.
8. `ls -lah` lists files, including hidden ones, with details.

---

## 🎯 Learning Goals

- Navigate the Linux filesystem confidently.
- Understand users, paths, files, and directories.
- Read command documentation independently.
- Manage files safely using terminal commands.
- Understand basic pipes and redirection.
- Build a foundation for Linux enumeration and privilege escalation labs.

## 📖 Resources

- [Linux man-pages](https://man7.org/linux/man-pages/)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/)
- [HTB Academy](https://academy.hackthebox.com/)

---

[← Back to Linux Notes](README.md)

**Author:** Tuncay Sarxanlı
