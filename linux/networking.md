# 🌐 Linux Networking

A practical guide to Linux networking commands, IP addresses, ports, DNS, routing, and network troubleshooting.

## 📑 Contents

- [1. Network Interfaces](#1-network-interfaces)
- [2. IP Addresses](#2-ip-addresses)
- [3. Routing](#3-routing)
- [4. Connectivity Testing](#4-connectivity-testing)
- [5. Ports and Listening Services](#5-ports-and-listening-services)
- [6. DNS](#6-dns)
- [7. Network Connections with `curl`](#7-network-connections-with-curl)
- [8. Downloading Files](#8-downloading-files)
- [9. Network Configuration](#9-network-configuration)
- [10. TCP and UDP](#10-tcp-and-udp)
- [11. Network Troubleshooting](#11-network-troubleshooting)
- [12. Useful Enumeration Commands](#12-useful-enumeration-commands)
- [13. Practical Example](#13-practical-example)

---

## 1. Network Interfaces

A network interface allows a computer to communicate over a network.

Common interface names include:

- `eth0` — Ethernet interface on some systems.
- `wlan0` — Wireless interface on some systems.
- `lo` — Loopback interface.
- `tun0` — Often a VPN or tunnel interface.

Interface names vary by operating system and configuration.

### `ip link`

Lists network interfaces.

```bash
ip link
```

### `ip addr`

Displays interfaces and their IP addresses.

```bash
ip addr
```

Short form:

```bash
ip a
```

Show one interface:

```bash
ip addr show wlan0
```

Replace `wlan0` with your actual interface name.

### `ip -br addr`

Displays a shorter summary.

```bash
ip -br addr
```

Example output:

```text
lo       UNKNOWN  127.0.0.1/8 ::1/128
eth0     UP       192.168.1.20/24
```

The addresses above are examples; your output will differ.

### `ip link show`

Check whether an interface is enabled and its current state.

```bash
ip link show
```

**Tip:** Before troubleshooting connectivity, identify the correct interface and check whether it has an IP address.

---

## 2. IP Addresses

An IP address identifies a network interface at the IP layer.

### IPv4

Example:

```text
192.168.1.20
```

An IPv4 address contains 32 bits.

### IPv6

Example:

```text
2001:db8::10
```

IPv6 uses 128-bit addresses. The `2001:db8::/32` range is reserved for documentation examples.

### Private IPv4 ranges

| Range | CIDR |
|---|---|
| `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` |
| `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` |

Private addresses are commonly used in home networks, offices, virtual machines, and labs.

### Loopback address

```text
127.0.0.1
```

The loopback address refers to the local machine.

Test it:

```bash
ping -c 4 127.0.0.1
```

### CIDR notation

Consider:

```text
192.168.1.20/24
```

- `192.168.1.20` — Interface IP address.
- `/24` — Prefix length, indicating that 24 bits belong to the network prefix.
- The corresponding subnet mask is `255.255.255.0`.

A `/24` IPv4 subnet contains 256 addresses in total, traditionally including a network address and a broadcast address.

**Cybersecurity note:** Understanding CIDR helps you identify the scope of a network during authorized enumeration.

---

## 3. Routing

Routing determines where network packets should be sent next.

### `ip route`

Displays the routing table.

```bash
ip route
```

Example output:

```text
default via 192.168.1.1 dev wlan0
192.168.1.0/24 dev wlan0 proto kernel scope link
```

In this example:

- `default` — Route used when no more specific route matches.
- `via 192.168.1.1` — The default gateway.
- `dev wlan0` — The interface used by the route.

Your actual gateway and interface may differ.

### Display IPv6 routes

```bash
ip -6 route
```

### Check the route to a destination

```bash
ip route get 8.8.8.8
```

This shows the route the kernel would select for that destination.

**Tip:** If a device can reach its local network but not the internet, inspect its IP address, routing table, and default gateway.

---

## 4. Connectivity Testing

### `ping`

Tests whether a destination responds to ICMP Echo Requests.

```bash
ping -c 4 8.8.8.8
```

- `-c 4` — Sends four requests and stops.

Test a domain:

```bash
ping -c 4 example.com
```

If the IP test works but the domain test fails, DNS may be a problem. However, firewall rules and other network conditions can also explain failures.

**Important:** A failed ping does not prove that a host is offline. ICMP traffic may be filtered.

### `traceroute`

Shows the network hops toward a destination.

```bash
traceroute example.com
```

If not installed, your distribution may offer it through its package manager.

Some networks filter or deprioritize traceroute probes, so not every hop will necessarily respond.

### `tracepath`

An alternative for examining a route and path MTU.

```bash
tracepath example.com
```

Availability depends on the system.

---

## 5. Ports and Listening Services

A port identifies a communication endpoint associated with a transport protocol, such as TCP or UDP.

### `ss`

Displays socket information.

List listening TCP sockets:

```bash
ss -ltn
```

List listening TCP and UDP sockets:

```bash
ss -lntup
```

Useful options:

| Option | Meaning |
|---|---|
| `-l` | Listening sockets |
| `-n` | Numeric addresses and ports |
| `-t` | TCP |
| `-u` | UDP |
| `-p` | Show associated processes when permitted |
| `-a` | All sockets |

Example:

```bash
ss -lnt
```

Example output:

```text
State  Recv-Q Send-Q Local Address:Port  Peer Address:Port
LISTEN 0      128    0.0.0.0:22          0.0.0.0:*
```

This example indicates a TCP service listening on port `22` on all local IPv4 interfaces. The actual output depends on your machine.

### Localhost versus all interfaces

- `127.0.0.1:8080` — Listening only on the IPv4 loopback interface.
- `0.0.0.0:8080` — Listening on all local IPv4 interfaces.
- `[::]:8080` — Listening on the IPv6 wildcard address; IPv4 behavior depends on configuration.

A service listening on all interfaces may be reachable from other systems if routing and firewall rules allow it.

### Common ports

| Port | Common service |
|---:|---|
| 20/21 TCP | FTP data/control in traditional active FTP |
| 22 TCP | SSH |
| 25 TCP | SMTP |
| 53 TCP/UDP | DNS |
| 80 TCP | HTTP |
| 110 TCP | POP3 |
| 139 TCP | NetBIOS Session Service |
| 143 TCP | IMAP |
| 443 TCP | HTTPS |
| 445 TCP | SMB |
| 3306 TCP | MySQL |
| 3389 TCP | RDP |
| 5432 TCP | PostgreSQL |
| 6379 TCP | Redis |

These are conventional ports, not guarantees. Services can run on non-standard ports, and some protocols use additional ports.

**Cybersecurity connection:** When Nmap discovers a port, identify the service, inspect its behavior, and enumerate it further. An open port is a starting point, not proof of a vulnerability.

---

## 6. DNS

The Domain Name System (DNS) translates domain names into records such as IP addresses.

For example:

```text
example.com → IP address
```

### `getent`

Uses the system's configured name-service mechanisms to resolve a hostname.

```bash
getent hosts example.com
```

### `dig`

Queries DNS.

```bash
dig example.com
```

Show only the answer section:

```bash
dig +short example.com
```

Query a specific record type:

```bash
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com NS
dig example.com TXT
```

| Record | Purpose |
|---|---|
| `A` | IPv4 address |
| `AAAA` | IPv6 address |
| `MX` | Mail servers |
| `NS` | Authoritative name servers |
| `TXT` | Text records, often used for domain verification and email security policies |
| `CNAME` | Alias to another domain name |

### `nslookup`

Another DNS lookup utility.

```bash
nslookup example.com
```

### `resolvectl`

On systems using `systemd-resolved`, this can display DNS configuration and query information.

```bash
resolvectl status
resolvectl query example.com
```

The command may be unavailable on systems that do not use `systemd-resolved`.

**Tip:** DNS resolution can fail even when basic IP connectivity works. Compare hostname lookups with direct IP connectivity to narrow down the problem.

---

## 7. Network Connections with `curl`

`curl` transfers data to or from URLs and supports protocols including HTTP and HTTPS.

### Make an HTTP request

```bash
curl http://example.com
```

Displays the response body.

### Display response headers

```bash
curl -I https://example.com
```

The `-I` option requests headers only for HTTP.

### Display headers and body

```bash
curl -i https://example.com
```

### Follow redirects

```bash
curl -L https://example.com
```

### Display verbose connection details

```bash
curl -v https://example.com
```

Verbose output can help you inspect DNS resolution, connection attempts, TLS negotiation, and HTTP details.

### Send a GET request

```bash
curl -X GET https://example.com
```

GET is the default HTTP method for ordinary `curl` requests.

### Send a POST request

```bash
curl -X POST -d "username=test" https://example.com/login
```

This sends form-style data to the URL. The server must actually support the endpoint and expected parameters for the request to succeed.

**Security note:** Use POST examples only with systems you own or are authorized to test. Never put real passwords or tokens in shell history or shared notes.

---

## 8. Downloading Files

### `wget`

Downloads a file from a URL.

```bash
wget https://example.com/file.txt
```

Save it under a chosen name:

```bash
wget -O downloaded.txt https://example.com/file.txt
```

### `curl -o`

Download a response to a file:

```bash
curl -o downloaded.txt https://example.com/file.txt
```

Use `-L` when the URL may redirect:

```bash
curl -L -o downloaded.txt https://example.com/file.txt
```

**Important:** A successful download does not guarantee that the file is trustworthy. Verify the source and, when appropriate, check its checksum before executing or installing it.

---

## 9. Network Configuration

### View the current DNS configuration

```bash
cat /etc/resolv.conf
```

This file may be generated or managed by another network service, so editing it directly may not persist.

### View the system hostname

```bash
hostname
```

### View network interface details

```bash
ip -br addr
ip link
```

### View the routing table

```bash
ip route
```

### Restart networking

Network management differs between distributions and configurations. On a system using NetworkManager, inspect its state with:

```bash
nmcli device status
nmcli connection show
```

You can also inspect active connections:

```bash
nmcli connection show --active
```

Avoid restarting network services over a remote session unless you understand the impact; doing so can disconnect you.

---

## 10. TCP and UDP

TCP and UDP are transport-layer protocols.

### TCP

TCP provides a connection-oriented byte stream with mechanisms for reliable delivery, ordering, and retransmission.

Common applications include SSH, HTTP, and HTTPS.

A simplified TCP connection setup is called the three-way handshake:

1. Client sends `SYN`.
2. Server responds with `SYN-ACK`.
3. Client sends `ACK`.

### UDP

UDP sends individual datagrams without TCP's connection setup or built-in reliable ordered delivery.

It is used by protocols and applications such as many DNS queries, DHCP, voice/video traffic, and real-time applications. DNS can also use TCP.

### Key differences

| Feature | TCP | UDP |
|---|---|---|
| Connection model | Connection-oriented | Connectionless |
| Built-in reliable ordered stream | Yes | No |
| Protocol overhead | Generally higher | Generally lower |
| Examples | SSH, HTTP, HTTPS | DHCP, many DNS queries, streaming and real-time traffic |

**Remember:** TCP and UDP use separate port spaces. TCP port 53 and UDP port 53 are distinct endpoints.

---

## 11. Network Troubleshooting

When a connection fails, investigate systematically rather than guessing.

### Step 1: Check the interface

```bash
ip -br addr
```

Does the interface have the expected address?

### Step 2: Check the routing table

```bash
ip route
```

Is there a suitable route to the destination?

### Step 3: Test IP connectivity

```bash
ping -c 4 1.1.1.1
```

A failed ping may indicate filtering rather than a complete connectivity failure.

### Step 4: Test DNS

```bash
getent hosts example.com
dig +short example.com
```

Does the name resolve?

### Step 5: Test the application

```bash
curl -I --connect-timeout 5 https://example.com
```

This tests an HTTPS request with a connection timeout.

### Step 6: Check listening sockets

For a service running on your own machine:

```bash
ss -lntup
```

Check the expected port, address, and protocol.

### Common problems

| Symptom | Possible causes |
|---|---|
| No IP address | DHCP issue, disabled interface, or configuration problem |
| Local network works, internet fails | Gateway, routing, firewall, or upstream issue |
| IP works, hostname fails | DNS configuration or resolution issue |
| Connection refused | Host reachable but no service listening on that endpoint, or an active rejection |
| Connection times out | Packet filtering, routing issue, offline host, or service not responding |
| Service works locally but not remotely | Binding address, firewall, routing, or access-control issue |

These symptoms suggest possible causes; none is conclusive on its own.

---

## 12. Useful Enumeration Commands

These commands are useful when examining your own Linux system or an authorized lab target.

### Interfaces and routes

```bash
ip addr
ip route
ip -br addr
```

### Connectivity and DNS

```bash
ping -c 4 127.0.0.1
getent hosts example.com
dig example.com
```

### Listening sockets

```bash
ss -lntup
```

### HTTP and HTTPS

```bash
curl -I https://example.com
curl -v https://example.com
```

### Network configuration

```bash
cat /etc/resolv.conf
nmcli device status
```

### Nmap connection

From a machine with Nmap installed, scan only systems you own or have permission to test:

```bash
nmap -sV 127.0.0.1
```

This checks the local machine for accessible TCP services and attempts service/version detection. Results depend on which services are running and how the system is configured.

---

## 13. Practical Example

This exercise checks the local network configuration and makes a basic HTTPS request.

```bash
whoami
ip -br addr
ip route
getent hosts example.com
curl -I --connect-timeout 5 https://example.com
ss -lnt
```

### What did we learn?

1. `ip -br addr` summarizes network interfaces and addresses.
2. `ip route` displays routing information.
3. `getent hosts` checks hostname resolution.
4. `curl -I` inspects HTTP response headers.
5. `ss -lnt` displays listening TCP sockets.

The output will vary depending on your network, firewall, DNS resolver, and running services.

---

## 🎯 Learning Goals

- Understand network interfaces and IP addresses.
- Read routing tables and identify the default gateway.
- Test basic network connectivity.
- Understand ports and listening services.
- Perform basic DNS lookups.
- Use `curl` to inspect HTTP and HTTPS.
- Recognize the differences between TCP and UDP.
- Troubleshoot common network issues.
- Build a foundation for Nmap and service enumeration.

## 📖 Resources

- [Linux man-pages](https://man7.org/linux/man-pages/)
- [iproute2 Documentation](https://wiki.linuxfoundation.org/networking/iproute2)
- [curl Documentation](https://curl.se/docs/)
- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/)
- [HTB Academy](https://academy.hackthebox.com/)

---

[← Back to Linux Notes](README.md)

**Author:** Tuncay Sarxanlı
