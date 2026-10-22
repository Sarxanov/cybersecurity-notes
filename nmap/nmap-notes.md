# Nmap Notes 🛰️

> Practical Nmap reference for network discovery, port scanning, service enumeration, and security labs.

## Table of Contents

- [1. What Is Nmap?](#1-what-is-nmap)
- [2. Basic Syntax](#2-basic-syntax)
- [3. Host Discovery](#3-host-discovery)
- [4. Port Scanning](#4-port-scanning)
- [5. Service and OS Detection](#5-service-and-os-detection)
- [6. Scan Speed and Timing](#6-scan-speed-and-timing)
- [7. NSE Scripts](#7-nse-scripts)
- [8. Output Formats](#8-output-formats)
- [9. Firewall and Evasion Concepts](#9-firewall-and-evasion-concepts)
- [10. Practical Enumeration Workflow](#10-practical-enumeration-workflow)
- [11. Useful Lab Examples](#11-useful-lab-examples)
- [12. Common Ports](#12-common-ports)
- [13. Common Mistakes](#13-common-mistakes)
- [14. Quick Cheat Sheet](#14-quick-cheat-sheet)
- [15. Practice Checklist](#15-practice-checklist)

---

## 1. What Is Nmap?

**Nmap (Network Mapper)** is an open-source tool used to discover hosts, identify open ports, detect services, and gather network information.

Common use cases:

- Host discovery
- TCP and UDP port scanning
- Service and version detection
- Operating system detection
- NSE-based enumeration
- Security auditing

**Important:** Only scan systems you own or have explicit permission to test.

## 2. Basic Syntax

General syntax:

```bash
nmap [options] <target>
```

Examples:

```bash
nmap 10.10.10.10
nmap scanme.nmap.org
nmap 192.168.1.0/24
nmap 10.10.10.10 10.10.10.20
```

Targets can be:

- A single IP address
- A hostname
- Multiple IP addresses
- A CIDR network range

### Target options

| Option | Meaning |
|---|---|
| `-iL targets.txt` | Read targets from a file |
| `-iR 10` | Select 10 random targets |
| `--exclude 10.10.10.5` | Exclude a target |
| `-n` | Disable DNS resolution |
| `-R` | Force DNS resolution |

## 3. Host Discovery

Host discovery helps determine which hosts appear to be online.

```bash
nmap -sn 192.168.1.0/24
```

- `-sn` — host discovery without a port scan.
- `-Pn` — skip host discovery and treat targets as online.
- `-PR` — ARP discovery on a local Ethernet network.
- `-PE` — ICMP echo discovery.
- `-PS22,80,443` — TCP SYN discovery probes to selected ports.
- `-PA80,443` — TCP ACK discovery probes.

### Example: scan a host that blocks ping

```bash
nmap -Pn 10.10.10.10
```

A host may be online even if discovery probes receive no response. `-Pn` can help in that situation, but it does not guarantee the host is reachable.

## 4. Port Scanning

### Default scan

```bash
nmap 10.10.10.10
```

By default, Nmap scans the 1,000 most common TCP ports.

### Scan specific ports

```bash
nmap -p 22 10.10.10.10
nmap -p 22,80,443 10.10.10.10
nmap -p 1-1000 10.10.10.10
```

### Scan all TCP ports

```bash
nmap -p- 10.10.10.10
```

`-p-` scans TCP ports 1–65535.

### TCP SYN scan

```bash
sudo nmap -sS 10.10.10.10
```

- Often called a half-open scan.
- Sends SYN probes and analyzes the responses.
- Usually requires elevated privileges for raw packet access.

### TCP connect scan

```bash
nmap -sT 10.10.10.10
```

- Uses the operating system's `connect()` call.
- Useful when raw packet scanning is unavailable.
- Completes the TCP connection when the port is open.

### UDP scan

```bash
sudo nmap -sU 10.10.10.10
```

UDP scans can be slower because UDP services often do not respond to probes.

### Scan type comparison

| Scan | Flag | Purpose |
|---|---|---|
| TCP SYN | `-sS` | SYN-based TCP scan |
| TCP connect | `-sT` | Full TCP connection scan |
| UDP | `-sU` | Check UDP ports |
| TCP ACK | `-sA` | Study firewall filtering behavior |
| TCP FIN | `-sF` | FIN-based TCP probing |

### Understanding port states

| State | Meaning |
|---|---|
| `open` | An application is accepting connections |
| `closed` | The port is reachable, but no application is listening |
| `filtered` | Filtering prevents Nmap from determining the state |
| `unfiltered` | The port is reachable, but open/closed status is undetermined |
| `open|filtered` | Nmap cannot distinguish between open and filtered |
| `closed|filtered` | Nmap cannot distinguish between closed and filtered |

## 5. Service and OS Detection

### Detect service versions

```bash
nmap -sV 10.10.10.10
```

Attempts to identify the applications and versions listening on discovered ports.

### Detect operating system

```bash
sudo nmap -O 10.10.10.10
```

OS detection works best when Nmap receives suitable responses from the target.

### Default scripts and version detection

```bash
nmap -sC -sV 10.10.10.10
```

- `-sC` runs Nmap's default NSE script set.
- `-sV` attempts service/version detection.

### Combine common options

```bash
sudo nmap -sC -sV -O -p- 10.10.10.10
```

This combines default scripts, version detection, OS detection, and a full TCP port scan. It may take longer than a targeted scan.

**Tip:** A practical two-stage workflow is to find open ports first, then run detailed detection against those ports.

```bash
nmap -p- --open 10.10.10.10
nmap -sC -sV -p 22,80,445 10.10.10.10
```

Replace the example ports with those actually discovered.

## 6. Scan Speed and Timing

Nmap provides timing templates from `-T0` to `-T5`.

| Option | General behavior |
|---|---|
| `-T0` | Extremely slow |
| `-T1` | Very slow |
| `-T2` | Slower |
| `-T3` | Default timing |
| `-T4` | Faster; commonly used in reliable lab networks |
| `-T5` | Very aggressive; may reduce accuracy |

Example:

```bash
nmap -T4 -p- 10.10.10.10
```

Other useful options:

```bash
nmap --open 10.10.10.10
nmap -v 10.10.10.10
nmap -vv 10.10.10.10
nmap --reason 10.10.10.10
nmap --top-ports 100 10.10.10.10
```

- `--open` — show open or potentially open ports.
- `-v` / `-vv` — increase verbosity.
- `--reason` — show why Nmap assigned a state.
- `--top-ports 100` — scan the 100 most common ports.

## 7. NSE Scripts

Nmap Scripting Engine (NSE) scripts extend Nmap's capabilities for service enumeration, information gathering, and security checks.

### Useful commands

```bash
nmap -sC 10.10.10.10
nmap --script-help http-title
nmap --script http-title -p 80,443 10.10.10.10
nmap --script "ssh-*" -p 22 10.10.10.10
```

Examples of script categories include:

- `default` — commonly useful scripts
- `discovery` — additional information gathering
- `safe` — scripts not designed to be intrusive
- `vuln` — vulnerability checks
- `auth` — authentication-related checks

Inspect a script's documentation before running it. Some scripts can be intrusive, affect services, or generate substantial traffic.

### Web enumeration example

```bash
nmap --script http-title,http-headers -p 80,443 10.10.10.10
```

### SMB enumeration example

```bash
nmap --script smb-os-discovery -p 445 10.10.10.10
```

Run these only against authorized targets.

## 8. Output Formats

Saving scan results makes it easier to compare scans and document findings.

### Normal output

```bash
nmap -oN scan.txt 10.10.10.10
```

### XML output

```bash
nmap -oX scan.xml 10.10.10.10
```

### Grepable output

```bash
nmap -oG scan.gnmap 10.10.10.10
```

### Save all major formats

```bash
nmap -oA scans/initial_scan 10.10.10.10
```

This creates normal, XML, and grepable output files using the same base name. Create the `scans` directory first if it does not exist.

## 9. Firewall and Evasion Concepts

These options can help diagnose network behavior in authorized testing environments.

```bash
nmap -sA 10.10.10.10
nmap -f 10.10.10.10
nmap --mtu 24 10.10.10.10
```

- `-sA` — TCP ACK scan, useful for examining certain firewall filtering behaviors.
- `-f` — fragment packets.
- `--mtu` — specify a packet fragmentation MTU.

These techniques do **not** guarantee firewall bypass or anonymity. Fragmentation and packet manipulation can cause scans to fail or trigger security controls. Use them only when you have permission and a clear testing objective.

## 10. Practical Enumeration Workflow

A useful methodology for CTFs and authorized assessments:

### Step 1 — Discover open ports

```bash
nmap -p- --open -T4 10.10.10.10
```

Record every open port.

### Step 2 — Identify services and versions

```bash
nmap -sC -sV -p 22,80,445 10.10.10.10
```

Replace the sample ports with your findings.

### Step 3 — Investigate each service

Examples:

- **22/SSH:** identify the version and authentication options.
- **21/FTP:** check the service banner and whether anonymous access is permitted.
- **80/443/HTTP(S):** inspect the website and investigate directories.
- **445/SMB:** enumerate shares and available host information.
- **3306/MySQL:** identify the database service and review exposure.
- **6379/Redis:** investigate whether the service is exposed and requires authentication.

Nmap findings are starting points, not proof that a service is vulnerable.

### Step 4 — Save evidence

```bash
mkdir -p scans
nmap -sC -sV -oA scans/target_initial 10.10.10.10
```

### Step 5 — Document your results

Record:

- Target IP
- Scan date
- Command used
- Open ports
- Detected services and versions
- Interesting findings
- Follow-up actions

## 11. Useful Lab Examples

Use these examples in your own lab or an authorized training environment.

### Quick scan

```bash
nmap -T4 --top-ports 100 10.10.10.10
```

### Full TCP port scan

```bash
nmap -p- -T4 --open 10.10.10.10
```

### Detailed scan

```bash
sudo nmap -sC -sV -O -p 22,80,445 10.10.10.10
```

### UDP scan of common ports

```bash
sudo nmap -sU --top-ports 20 10.10.10.10
```

### Scan an authorized local network

```bash
nmap -sn 192.168.1.0/24
```

Change the network range to match your own network.

## 12. Common Ports

| Port | Service | What to investigate |
|---:|---|---|
| 21 | FTP | Anonymous access, banners |
| 22 | SSH | Version, authentication |
| 23 | Telnet | Cleartext protocol exposure |
| 25 | SMTP | Mail service configuration |
| 53 | DNS | DNS records and configuration |
| 80 | HTTP | Web application |
| 110 | POP3 | Mail access configuration |
| 139 | NetBIOS | Legacy Windows file-sharing services |
| 143 | IMAP | Mail access configuration |
| 443 | HTTPS | TLS and web application |
| 445 | SMB | Shares, host information |
| 3306 | MySQL | Database exposure |
| 3389 | RDP | Remote desktop exposure |
| 5432 | PostgreSQL | Database exposure |
| 5900 | VNC | Remote desktop exposure |
| 6379 | Redis | Authentication and network exposure |
| 8080 | HTTP alternate | Web applications and proxies |

Port numbers are conventions, not guarantees. Always verify the detected service rather than assuming a port tells the whole story.

## 13. Common Mistakes

- **Scanning only the default ports:** important services may run on high-numbered ports.
- **Skipping `-sV`:** the port number alone does not identify the actual application.
- **Ignoring UDP:** some relevant services use UDP rather than TCP.
- **Assuming `filtered` means closed:** filtering prevents a definitive conclusion.
- **Using aggressive timing everywhere:** packet loss and network conditions can reduce reliability.
- **Treating a version as proof of a vulnerability:** verify the configuration and applicable security advisories.
- **Failing to save output:** results become difficult to compare or reproduce.
- **Running scans without authorization:** always establish permission and scope first.

## 14. Quick Cheat Sheet

| Goal | Command |
|---|---|
| Basic scan | `nmap TARGET` |
| Discover live hosts | `nmap -sn NETWORK` |
| Skip host discovery | `nmap -Pn TARGET` |
| Scan specific ports | `nmap -p 22,80,443 TARGET` |
| Scan all TCP ports | `nmap -p- TARGET` |
| SYN scan | `sudo nmap -sS TARGET` |
| TCP connect scan | `nmap -sT TARGET` |
| UDP scan | `sudo nmap -sU TARGET` |
| Version detection | `nmap -sV TARGET` |
| Default scripts | `nmap -sC TARGET` |
| OS detection | `sudo nmap -O TARGET` |
| Detailed scan | `sudo nmap -sC -sV -O TARGET` |
| Faster timing | `nmap -T4 TARGET` |
| Show open ports | `nmap --open TARGET` |
| Explain port states | `nmap --reason TARGET` |
| Save all formats | `nmap -oA output TARGET` |

Replace `TARGET` with the authorized target IP or hostname.

## 15. Practice Checklist

- [ ] Perform a basic TCP scan.
- [ ] Scan a specific range of ports.
- [ ] Scan all TCP ports.
- [ ] Explain the difference between `-sS` and `-sT`.
- [ ] Identify service versions using `-sV`.
- [ ] Run default NSE scripts using `-sC`.
- [ ] Explain `open`, `closed`, and `filtered`.
- [ ] Save results using `-oA`.
- [ ] Enumerate a web service and an SMB service.
- [ ] Compare a quick scan with a detailed scan.

---

## Key Takeaway

**Nmap is the beginning of enumeration, not the end.**

A solid workflow is:

`Discover → Scan Ports → Identify Services → Enumerate → Investigate Findings → Document`

The goal is not just to find open ports. The goal is to understand what is running, what information it exposes, and what to investigate next.
