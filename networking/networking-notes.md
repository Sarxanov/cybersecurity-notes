# Networking Notes

> Practical networking fundamentals for cybersecurity, penetration testing labs, network troubleshooting, and packet analysis.

## Table of Contents

1. [Introduction to Networking](#1-introduction-to-networking)
2. [The OSI Model](#2-the-osi-model)
3. [The TCP/IP Model](#3-the-tcpip-model)
4. [IP Addressing](#4-ip-addressing)
5. [Subnetting and CIDR](#5-subnetting-and-cidr)
6. [MAC Addresses and ARP](#6-mac-addresses-and-arp)
7. [TCP and UDP](#7-tcp-and-udp)
8. [TCP Three-Way Handshake](#8-tcp-three-way-handshake)
9. [Ports and Common Services](#9-ports-and-common-services)
10. [DNS](#10-dns)
11. [DHCP](#11-dhcp)
12. [HTTP and HTTPS](#12-http-and-https)
13. [Routing and Default Gateways](#13-routing-and-default-gateways)
14. [NAT](#14-nat)
15. [ICMP](#15-icmp)
16. [Firewalls and Network Segmentation](#16-firewalls-and-network-segmentation)
17. [VLANs](#17-vlans)
18. [VPNs and Tunneling](#18-vpns-and-tunneling)
19. [Proxies](#19-proxies)
20. [Common Network Services](#20-common-network-services)
21. [Essential Linux Networking Commands](#21-essential-linux-networking-commands)
22. [Networking with Nmap](#22-networking-with-nmap)
23. [Networking with Netcat](#23-networking-with-netcat)
24. [Packet Analysis with Wireshark](#24-packet-analysis-with-wireshark)
25. [Common Networking Troubleshooting](#25-common-networking-troubleshooting)
26. [Networking in Cybersecurity](#26-networking-in-cybersecurity)
27. [Practical Labs](#27-practical-labs)
28. [Quick Reference](#28-quick-reference)
29. [Further Learning](#29-further-learning)

---

## 1. Introduction to Networking

A computer network is a collection of devices that communicate and exchange data.

Devices may include:

- Computers and laptops
- Servers
- Routers
- Switches
- Firewalls
- Wireless access points
- Printers
- Virtual machines
- Internet of Things (IoT) devices

Networking is fundamental to cybersecurity because most systems communicate over networks.

Understanding networking helps security professionals identify exposed services, troubleshoot connectivity problems, analyze network traffic, and understand how attacks and defenses work.

### Essential networking concepts

| Concept | Description |
|---|---|
| IP address | Identifies a network-layer endpoint |
| MAC address | Identifies a link-layer interface address |
| Port | Identifies a transport-layer endpoint |
| Protocol | Defines communication rules |
| Packet | A unit of network-layer data |
| Frame | A link-layer data unit |
| Router | Forwards packets between networks |
| Switch | Forwards frames within a Layer 2 network |
| DNS | Resolves domain names into records |
| DHCP | Provides network configuration automatically |
| Firewall | Applies rules to network traffic |

### How does communication work?

Consider a computer connecting to a web server.

1. The computer determines where the server is located.
2. DNS may resolve the server's domain name to an IP address.
3. The operating system decides how to route the traffic.
4. The computer uses a transport protocol, such as TCP, when required.
5. The application exchanges data using HTTP or HTTPS.
6. Routers forward packets toward the destination.
7. The server processes the request and returns a response.

The exact process depends on the protocol, network configuration, and whether connections or DNS results are already available.

---

## 2. The OSI Model

The Open Systems Interconnection (OSI) model divides network communication into seven conceptual layers.

| Layer | Name | Examples |
|---|---|---|
| 7 | Application | HTTP, DNS, SMTP, SSH |
| 6 | Presentation | Data representation, encryption concepts |
| 5 | Session | Session management concepts |
| 4 | Transport | TCP, UDP |
| 3 | Network | IPv4, IPv6, ICMP |
| 2 | Data Link | Ethernet, Wi-Fi, ARP |
| 1 | Physical | Cables, radio signals, electrical signals |

### Layer 1 — Physical

Responsible for transmitting raw signals over a physical medium.

Examples:

- Ethernet cables
- Fiber-optic cables
- Radio signals
- Electrical signaling

### Layer 2 — Data Link

Handles communication over a local link.

Important concepts:

- MAC addresses
- Ethernet frames
- Switches
- VLANs
- ARP in IPv4 Ethernet networks

### Layer 3 — Network

Responsible for logical addressing and routing between networks.

Important concepts:

- IPv4
- IPv6
- Routers
- Routing tables
- ICMP

### Layer 4 — Transport

Provides communication between application endpoints.

**TCP** provides a reliable, ordered byte stream.

**UDP** provides datagrams without TCP's built-in reliability and ordering guarantees.

Ports are associated with transport-layer endpoints.

### Layers 5–7 — Session, Presentation, and Application

These layers describe session management, data representation, and application-level communication.

Examples of application protocols include:

- HTTP
- DNS
- SSH
- SMTP
- SMB

Real-world protocol stacks do not always map cleanly to individual OSI layers. Use the OSI model as a learning and troubleshooting framework.

### Why is the OSI model useful in cybersecurity?

It helps organize investigations.

For example:

- Layer 2: investigate unexpected MAC addresses or ARP behavior.
- Layer 3: investigate IP connectivity and routing.
- Layer 4: investigate ports, TCP connections, and UDP traffic.
- Layer 7: investigate HTTP requests, DNS queries, and application behavior.

---

## 3. The TCP/IP Model

The TCP/IP model describes networking in a way that closely reflects Internet protocol stacks.

| TCP/IP Layer | Examples | Approximate OSI Mapping |
|---|---|---|
| Application | HTTP, DNS, SSH, SMTP | 5–7 |
| Transport | TCP, UDP | 4 |
| Internet | IPv4, IPv6, ICMP | 3 |
| Network Access | Ethernet, Wi-Fi | 1–2 |

### Encapsulation

As application data travels through the network stack, each layer adds the information required for its operation.

A typical Ethernet transmission may look like:

```text
Ethernet Frame
└── IP Packet
    └── TCP Segment
        └── HTTP Data
```

For UDP:

```text
Ethernet Frame
└── IP Packet
    └── UDP Datagram
        └── Application Data
```

At the receiving endpoint, the layers process the corresponding headers and deliver the data to the appropriate application.

This process is called **encapsulation** on transmission and **decapsulation** on reception.

---

## 4. IP Addressing

An IP address identifies a network-layer endpoint.

There are two primary versions:

- IPv4
- IPv6

### 4.1 IPv4

IPv4 addresses contain 32 bits, normally written as four decimal octets.

Example:

```text
192.168.1.10
```

Each octet ranges from 0 to 255.

### 4.2 IPv6

IPv6 addresses contain 128 bits and are written in hexadecimal groups.

Example:

```text
2001:db8::10
```

The `2001:db8::/32` prefix is reserved for documentation.

IPv6 supports a much larger address space than IPv4.

### 4.3 Public and Private IPv4 Addresses

Private IPv4 ranges are reserved for internal networks.

| Range | CIDR |
|---|---|
| `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` |
| `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` |

These addresses are not globally routable on the public Internet by default.

Public IPv4 addresses are globally allocated address space, subject to routing and network policies.

### 4.4 Special IPv4 Addresses

| Address or Range | Purpose |
|---|---|
| `127.0.0.1` | IPv4 loopback |
| `127.0.0.0/8` | IPv4 loopback range |
| `0.0.0.0` | Unspecified address; also used in some binding and routing contexts |
| `0.0.0.0/0` | All IPv4 destinations in routing notation |
| `255.255.255.255` | Limited broadcast |
| `169.254.0.0/16` | IPv4 link-local |
| `224.0.0.0/4` | IPv4 multicast |

### 4.5 IPv4 vs. IPv6

| Feature | IPv4 | IPv6 |
|---|---|---|
| Address size | 32 bits | 128 bits |
| Notation | Decimal octets | Hexadecimal groups |
| Example | `192.168.1.10` | `2001:db8::10` |
| Broadcast | Supported | No broadcast; multicast is used instead |
| Neighbor discovery | ARP on Ethernet networks | Neighbor Discovery using ICMPv6 |

### 4.6 Check Your IP Address

On Linux:

```bash
ip addr
```

Show only IPv4 addresses:

```bash
ip -4 addr
```

Show only IPv6 addresses:

```bash
ip -6 addr
```

On Windows:

```powershell
ipconfig
```

### 4.7 Loopback

Loopback addresses allow a system to communicate with itself.

Common examples:

```text
127.0.0.1
::1
```

A service bound only to `127.0.0.1` is normally accessible only through the local IPv4 loopback interface, rather than directly through the machine's LAN address.

This distinction is important when troubleshooting local web servers and development environments.

---

## 5. Subnetting and CIDR

Subnetting divides an IP network into smaller logical networks.

CIDR stands for Classless Inter-Domain Routing.

A CIDR prefix indicates how many leading bits belong to the network prefix.

### 5.1 Understanding CIDR

Example:

```text
192.168.1.0/24
```

The `/24` means that 24 of the 32 IPv4 bits are part of the network prefix.

That leaves:

```text
32 - 24 = 8 host bits
```

The total number of addresses is:

```text
2^8 = 256
```

For a conventional IPv4 `/24` subnet:

- Network address: `192.168.1.0`
- First conventional host address: `192.168.1.1`
- Last conventional host address: `192.168.1.254`
- Broadcast address: `192.168.1.255`

There are normally 254 usable host addresses in this example.

Point-to-point links, `/31` networks, and `/32` host routes have special considerations.

### 5.2 Common IPv4 Prefixes

| CIDR | Subnet Mask | Total Addresses | Conventional Usable Hosts |
|---|---|---:|---:|
| `/8` | `255.0.0.0` | 16,777,216 | 16,777,214 |
| `/16` | `255.255.0.0` | 65,536 | 65,534 |
| `/24` | `255.255.255.0` | 256 | 254 |
| `/25` | `255.255.255.128` | 128 | 126 |
| `/26` | `255.255.255.192` | 64 | 62 |
| `/27` | `255.255.255.224` | 32 | 30 |
| `/28` | `255.255.255.240` | 16 | 14 |
| `/29` | `255.255.255.248` | 8 | 6 |
| `/30` | `255.255.255.252` | 4 | 2 |
| `/31` | `255.255.255.254` | 2 | Special point-to-point usage |
| `/32` | `255.255.255.255` | 1 | Single address/host route |

### 5.3 Subnet Mask

A subnet mask identifies the network prefix in IPv4.

Example:

```text
IP Address:  192.168.1.10
Subnet Mask: 255.255.255.0
CIDR:        /24
```

The network is:

```text
192.168.1.0/24
```

### 5.4 Why Subnetting Matters

Subnetting helps you understand:

- Whether two addresses belong to the same subnet
- Whether a packet should be sent directly or through a router
- The size of a network
- How internal networks are divided
- Which IP ranges may be relevant during authorized host discovery

For example, an address in `192.168.1.0/24` belongs to a different subnet from `192.168.2.0/24` when those prefixes are configured as shown.

### 5.5 Host Discovery and CIDR

An Nmap target can be specified using CIDR notation.

Example against an authorized lab:

```bash
nmap -sn 192.168.1.0/24
```

This requests host discovery without a normal port scan.

Discovery results depend on routing, firewalls, host behavior, and the discovery methods used. A host that does not respond is not necessarily offline.

---

## 6. MAC Addresses and ARP

### 6.1 MAC Addresses

A MAC address is a link-layer address commonly used by Ethernet and Wi-Fi interfaces.

Example:

```text
00:11:22:33:44:55
```

MAC addresses are used for delivery over the local link.

Routers normally replace the link-layer header when forwarding an IP packet onto another network.

Therefore, the destination MAC address in an Ethernet frame is often the next-hop router's MAC address rather than the remote server's MAC address.

### 6.2 ARP

Address Resolution Protocol maps an IPv4 address to a link-layer address on the local network.

Suppose a computer needs to communicate with `192.168.1.20`.

If the corresponding MAC address is not already known, the computer may broadcast an ARP request.

```text
Who has 192.168.1.20?
Tell 192.168.1.10.
```

The device with that IPv4 address can respond with its MAC address.

### 6.3 View Neighbor Information

On Linux:

```bash
ip neigh
```

On Windows:

```powershell
arp -a
```

These commands display cached neighbor or ARP information, depending on the operating system.

### 6.4 ARP Spoofing

ARP does not provide strong built-in authentication for IP-to-MAC mappings.

An attacker on a local network may attempt to send forged ARP messages to redirect traffic.

Potential defenses include:

- Dynamic ARP Inspection on supported switches
- DHCP snooping as part of a suitable switch configuration
- Network segmentation
- Monitoring unexpected address-to-MAC changes
- Encryption such as HTTPS and SSH to protect application traffic

Encryption reduces the impact of traffic interception but does not prevent every network-layer attack.

---

## 7. TCP and UDP

TCP and UDP are major transport-layer protocols.

### 7.1 TCP

Transmission Control Protocol provides:

- Connection-oriented communication
- Reliable delivery mechanisms
- Ordered byte-stream delivery
- Retransmission when necessary
- Flow control
- Congestion control

Common TCP-based applications include:

- SSH
- HTTPS over TCP
- SMB
- SMTP
- Many database services

### 7.2 UDP

User Datagram Protocol provides datagram transport with minimal protocol overhead.

UDP does not itself guarantee delivery, ordering, or retransmission.

Applications can implement their own reliability mechanisms when needed.

Common uses include:

- DNS
- DHCP
- Voice and video applications
- QUIC and HTTP/3
- Various VPN protocols

### 7.3 TCP vs. UDP

| Property | TCP | UDP |
|---|---|---|
| Connection model | Connection-oriented | Connectionless |
| Built-in reliable delivery | Yes | No |
| Built-in ordered byte stream | Yes | No |
| Handshake | Normally three-way | No TCP-style handshake |
| Retransmission | Managed by TCP | Application-dependent |
| Common example | SSH | DHCP |

Neither protocol is universally faster or more secure. The right choice depends on the application's requirements and implementation.

### 7.4 TCP Flags

| Flag | Meaning |
|---|---|
| SYN | Synchronizes sequence numbers and initiates a connection |
| ACK | Acknowledgment field is valid |
| FIN | Sender has finished sending data |
| RST | Resets a connection |
| PSH | Requests prompt delivery of data to the receiving application |
| URG | Indicates urgent-pointer semantics |

### 7.5 TCP Three-Way Handshake

A typical TCP connection begins with:

```text
Client                         Server
  |                               |
  | -------- SYN ----------------> |
  | <----- SYN + ACK ------------- |
  | -------- ACK ----------------> |
  |                               |
  |       Connection ready        |
```

The handshake establishes TCP connection state and synchronizes initial sequence numbers.

Useful Wireshark filter for initial SYN packets:

```text
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

### 7.6 TCP Connection Termination

A normal TCP connection can use FIN and ACK exchanges to close the connection gracefully.

RST may be used when a connection is reset.

A reset does not automatically mean an attack occurred. Applications, operating systems, and network devices can all generate resets for legitimate reasons.

---

## 8. TCP Three-Way Handshake

This section summarizes the handshake from a practical troubleshooting perspective.

### Step 1: SYN

The client requests to establish a TCP connection.

### Step 2: SYN-ACK

The server acknowledges the client's SYN and sends its own SYN.

### Step 3: ACK

The client acknowledges the server's SYN.

### What if the handshake fails?

**SYN with no visible response**

Possible explanations:

- The server is unreachable.
- A firewall drops the packet.
- The destination port is filtered.
- The response is lost.
- The capture does not observe both directions.

**SYN followed by RST**

Possible explanations:

- The port is closed.
- The host or an intermediary actively rejects the connection.
- The service is unavailable.

**SYN, SYN-ACK, ACK**

The TCP handshake completed, but this does not guarantee that the application will work correctly.

Application-level failures may still occur afterward.

### Networking and Nmap

Nmap's SYN scan uses TCP SYN packets to infer port states without completing a full TCP connection in the usual way.

Example in an authorized lab:

```bash
sudo nmap -sS -p 22,80,443 192.168.1.20
```

The result can help identify which services need further investigation.

---

## 9. Ports and Common Services

A port identifies a transport-layer endpoint associated with an application.

TCP and UDP maintain separate port spaces.

### 9.1 Common Ports

| Port | Protocol/Service | Typical Use |
|---:|---|---|
| 20/21 | FTP | File transfer |
| 22 | SSH | Secure remote shell |
| 23 | Telnet | Unencrypted remote terminal |
| 25 | SMTP | Mail transfer |
| 53 | DNS | Name resolution |
| 67/68 | DHCP | IPv4 configuration |
| 69 | TFTP | Simple file transfer |
| 80 | HTTP | Web traffic |
| 110 | POP3 | Mail retrieval |
| 123 | NTP | Time synchronization |
| 135 | MS RPC Endpoint Mapper | Windows RPC services |
| 137–139 | NetBIOS services | Legacy Windows networking |
| 143 | IMAP | Mail retrieval |
| 161/162 | SNMP | Network management |
| 389 | LDAP | Directory services |
| 443 | HTTPS | HTTP over TLS |
| 445 | SMB | File sharing and Windows networking |
| 465 | SMTP over implicit TLS | Secure mail submission |
| 500 | IKE | IPsec negotiation |
| 514 | Syslog | System logging, commonly UDP |
| 587 | SMTP submission | Mail submission |
| 636 | LDAPS | LDAP over TLS |
| 993 | IMAPS | IMAP over TLS |
| 995 | POP3S | POP3 over TLS |
| 1433 | Microsoft SQL Server | Database service |
| 1521 | Oracle Database | Database service |
| 2049 | NFS | Network file system |
| 3306 | MySQL | Database service |
| 3389 | RDP | Remote Desktop Protocol |
| 5432 | PostgreSQL | Database service |
| 5900 | VNC | Remote desktop |
| 5985 | WinRM over HTTP | Windows remote management |
| 5986 | WinRM over HTTPS | Windows remote management |
| 6379 | Redis | In-memory data store |
| 8080 | Alternative HTTP | Web applications and proxies |
| 8443 | Alternative HTTPS | Web applications |

These are common defaults, not guarantees. Services can use other ports, and a port number alone does not identify the running application with certainty.

### 9.2 Port Ranges

| Range | Name |
|---|---|
| 0–1023 | System / well-known ports |
| 1024–49151 | User / registered ports |
| 49152–65535 | Dynamic / private ports |

Operating systems and applications can use ephemeral source ports for client connections.

Example:

```text
Client: 192.168.1.10:51524
Server: 192.168.1.20:443
```

The client uses a temporary source port while connecting to the server's destination port.

### 9.3 Why Ports Matter in Cybersecurity

An exposed port may reveal a service that needs to be assessed.

For example:

- Port 22: inspect SSH configuration and authentication.
- Port 445: investigate SMB exposure and permissions.
- Port 3389: review RDP exposure and access controls.
- Port 3306: determine whether a database should be reachable from that network.
- Port 5985/5986: review WinRM configuration and authorized remote management.

An open port is not automatically a vulnerability. Assess the service, version, configuration, exposure, and access controls.

---

## 10. DNS

Domain Name System resolves domain names and provides other types of records.

Example:

```text
example.com -> IP address
```

### 10.1 DNS Query Process

A simplified lookup may involve:

1. The application requests a hostname lookup.
2. The operating system or application checks available caches.
3. A configured DNS resolver is queried if necessary.
4. The resolver obtains the required information from its cache or through DNS resolution.
5. The result is returned to the client.

Not every lookup requires a fresh external DNS request.

### 10.2 Common DNS Record Types

| Record | Purpose |
|---|---|
| A | Maps a name to an IPv4 address |
| AAAA | Maps a name to an IPv6 address |
| CNAME | Alias to another canonical name |
| MX | Identifies mail exchangers |
| NS | Identifies authoritative name servers |
| TXT | Stores text data, often used for verification and email security |
| PTR | Supports reverse DNS lookup |
| SOA | Contains zone authority and administrative information |
| SRV | Advertises services and their connection details |

### 10.3 DNS Commands

Linux:

```bash
dig example.com
```

Query an A record:

```bash
dig example.com A
```

Query an MX record:

```bash
dig example.com MX
```

Query a specific resolver:

```bash
dig @1.1.1.1 example.com
```

Another common utility:

```bash
nslookup example.com
```

### 10.4 DNS over HTTPS and DNS over TLS

Traditional DNS commonly uses UDP port 53 and can also use TCP port 53.

Encrypted DNS options include:

- DNS over TLS (DoT)
- DNS over HTTPS (DoH)

Encrypted DNS can make DNS query contents less visible to passive network observers.

This means a packet capture may not reveal every hostname requested by an application.

### 10.5 DNS in Cybersecurity

DNS is useful for:

- Domain reconnaissance in authorized assessments
- Investigating suspicious hostnames
- Troubleshooting name-resolution failures
- Correlating hostnames with network connections
- Detecting potential tunneling or beaconing activity

DNS records are useful evidence, but their interpretation requires context.

---

## 11. DHCP

Dynamic Host Configuration Protocol provides network configuration to clients.

DHCP can supply:

- IP address
- Subnet mask or prefix
- Default gateway
- DNS resolver addresses
- Lease duration
- Other configuration options

### 11.1 DHCP DORA

A common IPv4 DHCP exchange follows four steps:

1. Discover
2. Offer
3. Request
4. Acknowledge

```text
Client                         DHCP Server
  |                                  |
  | -------- Discover ------------> |
  | <------- Offer ---------------- |
  | -------- Request -------------> |
  | <------- Acknowledge ---------- |
```

The initial exchange often uses UDP ports 68 and 67. Relays and other DHCP configurations can affect the observed traffic.

### 11.2 Check Network Configuration

Linux:

```bash
ip addr
ip route
cat /etc/resolv.conf
```

Windows:

```powershell
ipconfig /all
```

### 11.3 Common DHCP Problems

- No DHCP server is available.
- The DHCP address pool is exhausted.
- A client cannot reach the server.
- A rogue DHCP server provides incorrect settings.
- A client receives the wrong gateway or DNS configuration.

An IPv4 address in the `169.254.0.0/16` range may indicate link-local configuration, including a failure to obtain a DHCP lease in networks that normally use DHCP.

---

## 12. HTTP and HTTPS

HTTP is an application protocol used for web communication.

HTTPS protects HTTP communication using TLS.

### 12.1 HTTP Request

Example:

```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: ExampleBrowser/1.0
Accept: text/html
```

Common request methods:

- GET
- POST
- PUT
- PATCH
- DELETE
- HEAD
- OPTIONS

### 12.2 HTTP Status Codes

| Status | Meaning |
|---|---|
| 200 | OK |
| 301 | Moved Permanently |
| 302 | Found / temporary redirect |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

### 12.3 HTTPS

HTTPS protects application data in transit when TLS is correctly configured and validated.

It helps protect:

- Credentials
- Session cookies
- Request bodies
- Response content
- Other application data

HTTPS does not automatically protect a vulnerable server or prevent phishing, insecure application logic, or endpoint compromise.

### 12.4 Inspect HTTP with curl

Show response headers:

```bash
curl -I https://example.com
```

Show verbose connection information:

```bash
curl -v https://example.com
```

Retrieve a page:

```bash
curl https://example.com
```

Verbose output can contain sensitive connection information. Avoid sharing it without reviewing the contents.

---

## 13. Routing and Default Gateways

Routing determines where packets should be sent next.

A host uses its routing table to select an appropriate route based on the destination address.

### 13.1 Local vs. Remote Destination

Suppose a host has:

```text
IP Address:  192.168.1.10
Prefix:      /24
Gateway:     192.168.1.1
```

A destination of `192.168.1.20` belongs to the same `/24` network, so the host can normally communicate directly over the local link.

A destination of `8.8.8.8` is outside that subnet, so the host normally sends the packet to its configured next-hop gateway.

The gateway forwards packets toward other networks when routing is available.

### 13.2 Default Route

A default route is used when no more-specific matching route is selected.

For IPv4, it is commonly written as:

```text
0.0.0.0/0
```

For IPv6:

```text
::/0
```

### 13.3 View Routing Tables

Linux:

```bash
ip route
```

IPv6:

```bash
ip -6 route
```

Windows:

```powershell
route print
```

### 13.4 Example

A Linux routing table may contain:

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link
```

Interpretation:

- The default route sends other destinations through `192.168.1.1`.
- The local subnet is directly connected through `eth0`.

Actual routing output depends on the host's configuration.

---

## 14. NAT

Network Address Translation changes IP address information as packets pass through a network device.

NAT is commonly used to allow multiple private IPv4 hosts to share a public IPv4 address.

### 14.1 Source NAT

Source NAT modifies the source address of outgoing traffic.

### 14.2 Destination NAT

Destination NAT modifies the destination address of traffic.

Port forwarding is a common use of destination NAT.

### 14.3 PAT

Port Address Translation allows multiple connections to share an address while distinguishing flows through transport-layer port mappings.

PAT is commonly used in home routers.

### 14.4 NAT and Security

NAT can reduce direct inbound reachability when combined with appropriate routing and firewall behavior.

However, **NAT is not a substitute for a firewall**.

Security depends on filtering rules, service exposure, host configuration, and the overall network architecture.

### 14.5 Example

An internal device:

```text
192.168.1.10:51524
```

may communicate with an external server using a public source address and a translated source port.

The router tracks the translation so that returning traffic can be associated with the correct internal flow.

---

## 15. ICMP

Internet Control Message Protocol supports network diagnostics and error reporting.

Common uses include:

- Echo requests and replies
- Destination unreachable messages
- Time exceeded messages
- Path MTU Discovery

### 15.1 Ping

Linux:

```bash
ping -c 4 1.1.1.1
```

Windows:

```powershell
ping 1.1.1.1
```

Ping commonly uses ICMP Echo Request and Echo Reply messages.

### 15.2 Traceroute

Linux:

```bash
traceroute example.com
```

Alternative:

```bash
tracepath example.com
```

Windows:

```powershell
tracert example.com
```

Traceroute tools use probe packets and responses to infer the network path.

Depending on the implementation, probes may use UDP, ICMP, or TCP.

### 15.3 Important Limitation

A failed ping does not prove that a host is offline.

Possible explanations include:

- ICMP is filtered.
- The destination does not respond to Echo Requests.
- A firewall blocks the probes.
- Routing is broken.
- Replies are lost.

Use multiple methods to investigate connectivity.

---

## 16. Firewalls and Network Segmentation

A firewall enforces rules governing network traffic.

Depending on the firewall, rules may consider:

- Source IP address
- Destination IP address
- Protocol
- Source and destination ports
- Connection state
- Application identity
- User identity
- Other contextual information

### 16.1 Firewall Types

**Stateless firewall:** Evaluates packets individually against rules.

**Stateful firewall:** Tracks connection state and uses it when making decisions.

**Application-layer firewall:** Can inspect application-layer information.

**Host firewall:** Runs on an endpoint or server.

**Network firewall:** Protects traffic between network segments or at network boundaries.

### 16.2 Default-Deny Principle

A default-deny policy blocks traffic unless it is explicitly permitted by the configured rules.

This reduces unnecessary exposure but requires careful rule design to avoid breaking legitimate services.

### 16.3 Network Segmentation

Segmentation divides a network into separate security zones.

Examples:

- User workstations
- Servers
- Database systems
- Guest Wi-Fi
- Management interfaces
- Security monitoring systems

Segmentation can reduce unnecessary communication and limit the impact of a compromised host.

### 16.4 Linux Firewall Tools

Depending on the distribution and configuration, Linux systems may use:

- nftables
- iptables
- UFW
- firewalld

Inspect the system's active firewall configuration before changing rules.

For example:

```bash
sudo nft list ruleset
```

On systems using UFW:

```bash
sudo ufw status verbose
```

The absence of UFW rules does not prove that no other firewall mechanism is active.

---

## 17. VLANs

A Virtual Local Area Network (VLAN) divides a switched network into separate Layer 2 broadcast domains.

VLANs can help separate groups of devices without requiring a separate physical switch for every group.

### Example

A company may use:

- VLAN 10: Employees
- VLAN 20: Servers
- VLAN 30: Guest devices
- VLAN 40: Management

Devices in different VLANs normally need Layer 3 routing to communicate.

### Access Ports and Trunk Ports

**Access port:** Usually carries traffic for one configured VLAN.

**Trunk port:** Can carry traffic for multiple VLANs using VLAN tagging, commonly IEEE 802.1Q.

### Security Considerations

VLANs help organize networks, but VLAN separation alone is not a complete security boundary.

Use suitable routing controls, access-control lists, firewalls, and switch configuration to enforce communication policies.

---

## 18. VPNs and Tunneling

A Virtual Private Network (VPN) creates a logical communication path over another network.

VPNs are commonly used for:

- Remote access
- Site-to-site connectivity
- Access to private services
- Connecting lab environments
- Protecting traffic across untrusted networks

### 18.1 Tunneling

Tunneling encapsulates one type of traffic inside another protocol.

For example, an IP packet may be encapsulated within a VPN transport protocol.

### 18.2 Common VPN Technologies

Examples include:

- WireGuard
- OpenVPN
- IPsec
- IKEv2-based VPNs

Each technology has different design, authentication, and configuration requirements.

### 18.3 VPNs in Cybersecurity Labs

VPNs are commonly used to connect to isolated training networks.

Before scanning a lab:

1. Confirm that the VPN connection is active.
2. Check the assigned interface and IP address.
3. Inspect the routing table.
4. Verify that the target belongs to the authorized lab scope.
5. Run discovery and enumeration against permitted targets.

Useful commands:

```bash
ip addr
ip route
```

```bash
ping -c 4 TARGET_IP
```

A failed ping does not necessarily mean that the VPN is broken or the target is unavailable.

---

## 19. Proxies

A proxy forwards requests between a client and another endpoint.

### 19.1 Forward Proxy

A forward proxy acts on behalf of clients accessing other systems.

Common uses include:

- Web filtering
- Access control
- Logging
- Privacy and network policy enforcement

### 19.2 Reverse Proxy

A reverse proxy sits in front of backend servers.

Common uses include:

- Load balancing
- TLS termination
- Request routing
- Web application protection
- Caching

### 19.3 Proxy vs. VPN

| Feature | Proxy | VPN |
|---|---|---|
| Typical function | Forwards selected application requests | Creates a virtual network path |
| Scope | Often application-specific | Often routes traffic at the network level |
| Traffic protection | Depends on the proxy protocol and configuration | Depends on the VPN protocol and configuration |
| Routing changes | Not necessarily | Commonly changes routing behavior |

Neither automatically guarantees anonymity or security.

---

## 20. Common Network Services

### 20.1 SSH

Secure Shell provides encrypted remote access and other secure communication features.

Default port: TCP 22.

Connect to a remote host:

```bash
ssh username@192.168.1.20
```

Specify a port:

```bash
ssh -p 2222 username@192.168.1.20
```

Security considerations:

- Use strong authentication.
- Protect private keys.
- Restrict unnecessary access.
- Monitor authentication logs.
- Verify host keys.

### 20.2 SMB

Server Message Block is commonly used for file sharing and Windows networking.

Common port: TCP 445.

Inspect shares on an authorized system:

```bash
smbclient -L //192.168.1.20 -N
```

The `-N` option suppresses the password prompt. It does not bypass server-side access controls.

SMB security depends on configuration, authentication, authorization, patching, and network exposure.

### 20.3 FTP

File Transfer Protocol commonly uses TCP port 21 for its control connection.

Traditional FTP may transmit credentials and data without encryption.

Secure alternatives include SFTP over SSH and FTPS using TLS. These are different protocols or protocol configurations.

### 20.4 RDP

Remote Desktop Protocol provides remote graphical access to Windows systems.

Common port: TCP 3389.

Security considerations:

- Restrict network exposure.
- Require strong authentication.
- Use appropriate access controls.
- Apply security updates.
- Monitor authentication and session activity.

### 20.5 WinRM

Windows Remote Management supports remote administration.

Common ports:

- TCP 5985: HTTP transport
- TCP 5986: HTTPS transport

The security of WinRM depends on authentication, transport protection, authorization, and configuration.

### 20.6 Database Services

Common database ports include:

- MySQL: TCP 3306
- Microsoft SQL Server: TCP 1433
- PostgreSQL: TCP 5432
- Oracle Database: TCP 1521
- Redis: TCP 6379

Database ports should generally be reachable only by systems that require access.

An exposed database service is not automatically vulnerable, but unnecessary exposure increases risk.

---

## 21. Essential Linux Networking Commands

### 21.1 View Interfaces and IP Addresses

```bash
ip addr
```

Short form:

```bash
ip a
```

Show IPv4 addresses:

```bash
ip -4 addr
```

### 21.2 View Routes

```bash
ip route
```

### 21.3 View Neighbor Cache

```bash
ip neigh
```

### 21.4 View Listening Ports and Connections

```bash
ss -tuln
```

Options:

- `-t`: TCP sockets
- `-u`: UDP sockets
- `-l`: listening sockets
- `-n`: numeric addresses and ports

Show processes associated with sockets when permitted:

```bash
sudo ss -tulpn
```

### 21.5 Test DNS

```bash
dig example.com
```

```bash
nslookup example.com
```

### 21.6 Test Connectivity

```bash
ping -c 4 1.1.1.1
```

### 21.7 Trace the Network Path

```bash
traceroute example.com
```

### 21.8 Inspect Local Network Configuration

```bash
ip addr
ip route
ip neigh
cat /etc/resolv.conf
```

These commands help identify the local address, route, neighboring devices, and configured DNS resolvers.

### 21.9 Inspect Network Traffic

List interfaces:

```bash
sudo tcpdump -D
```

Capture DNS traffic on an interface:

```bash
sudo tcpdump -i eth0 port 53
```

Capture traffic to or from a specific host:

```bash
sudo tcpdump -i eth0 host 192.168.1.20
```

Save a capture:

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

Read a capture:

```bash
tcpdump -r capture.pcap
```

Replace interface names and addresses with values appropriate to your environment.

---

## 22. Networking with Nmap

Nmap is a network discovery and security auditing tool.

It can identify reachable hosts, scan ports, detect services, and perform additional enumeration.

Use Nmap only against systems you own or are explicitly authorized to test.

### 22.1 Host Discovery

```bash
nmap -sn 192.168.1.0/24
```

This performs host discovery without a normal port scan.

### 22.2 Scan Common Ports

```bash
nmap 192.168.1.20
```

### 22.3 Scan Specific Ports

```bash
nmap -p 22,80,443 192.168.1.20
```

### 22.4 Scan All TCP Ports

```bash
nmap -p- 192.168.1.20
```

### 22.5 Detect Services and Versions

```bash
nmap -sV 192.168.1.20
```

### 22.6 Run Default Scripts

```bash
nmap -sC 192.168.1.20
```

### 22.7 SYN Scan

```bash
sudo nmap -sS 192.168.1.20
```

### 22.8 Combine Options

```bash
sudo nmap -sS -sV -sC -p- 192.168.1.20
```

This combination can be useful in an authorized lab, but scanning every port and running scripts may generate more traffic and take longer.

### 22.9 Understanding Port States

| State | Meaning |
|---|---|
| Open | An application is accepting connections on the port |
| Closed | The host is reachable, but no application is listening on that port |
| Filtered | Nmap cannot determine whether the port is open because packet filtering or another network condition prevents a conclusive result |
| Open\|Filtered | Nmap cannot distinguish between open and filtered |
| Closed\|Filtered | Nmap cannot distinguish between closed and filtered in applicable scan types |

Port states are conclusions drawn from probe responses, not absolute descriptions of every possible network condition.

### 22.10 A Useful Enumeration Workflow

1. Identify the authorized target.
2. Confirm the target IP and network route.
3. Discover reachable hosts.
4. Scan relevant ports.
5. Identify services and versions.
6. Enumerate each service with appropriate tools.
7. Record evidence and findings.
8. Validate security issues without exceeding the permitted scope.

**Remember:** A port number is a starting point, not a complete service identification.

---

## 23. Networking with Netcat

Netcat (`nc`) is a network utility for interacting with TCP and UDP endpoints.

It is useful for basic connectivity tests and controlled lab exercises.

### 23.1 Test a TCP Port

```bash
nc -vz 192.168.1.20 22
```

This attempts a TCP connection to port 22.

The result depends on connectivity, filtering, and the remote service.

### 23.2 Connect to a Service

```bash
nc 192.168.1.20 80
```

On a service accepting plain HTTP, you can manually send a simple request:

```http
GET / HTTP/1.1
Host: example.com
Connection: close

```

A blank line terminates the HTTP request headers.

### 23.3 Start a Listener in a Lab

On a system you control:

```bash
nc -lv 4444
```

In a second terminal on the same system:

```bash
nc 127.0.0.1 4444
```

Type a message in one terminal and observe it in the other.

Netcat implementations differ in their command-line options. Use `nc -h` or the local manual to check your version.

### 23.4 Limitations

A successful TCP connection only proves that a connection could be established. It does not prove that the service is secure, correctly configured, or functioning at the application level.

---

## 24. Packet Analysis with Wireshark

Wireshark captures and analyzes network packets.

It is useful for troubleshooting, protocol analysis, and cybersecurity investigations.

### 24.1 Useful Display Filters

Show TCP:

```text
tcp
```

Show DNS:

```text
dns
```

Show HTTP:

```text
http
```

Show TLS:

```text
tls
```

Show traffic involving a host:

```text
ip.addr == 192.168.1.20
```

Show traffic involving TCP port 445:

```text
tcp.port == 445
```

Show initial TCP SYN packets:

```text
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

Show TCP retransmissions:

```text
tcp.analysis.retransmission
```

### 24.2 Follow a Stream

Right-click a relevant packet and select:

**Follow → TCP Stream**

This helps reconstruct the captured TCP conversation.

### 24.3 Useful Statistics

- Protocol Hierarchy
- Conversations
- Endpoints
- I/O Graphs
- Flow Graph
- Expert Information

### 24.4 PCAP Investigation Method

1. Preserve the original capture.
2. Review protocol distribution.
3. Identify communicating hosts.
4. Inspect DNS and connection attempts.
5. Follow interesting streams.
6. Build a timeline.
7. Record packet numbers and timestamps.
8. Explain what the evidence supports and what remains uncertain.

Encrypted traffic may hide application content. The amount of visible metadata depends on the protocol and its configuration.

For more detailed filters and exercises, see `wireshark-notes.md`.

---

## 25. Common Networking Troubleshooting

When a connection fails, investigate systematically instead of immediately blaming DNS, the firewall, or the server.

### 25.1 Layer-by-Layer Workflow

**Step 1: Check the local interface**

```bash
ip addr
```

Confirm that the interface is active and has the expected address.

**Step 2: Check routing**

```bash
ip route
```

Confirm that a suitable route exists for the destination.

**Step 3: Check local neighbor information**

```bash
ip neigh
```

Review relevant local IPv4 neighbor entries.

**Step 4: Test basic reachability**

```bash
ping -c 4 TARGET_IP
```

Remember that ICMP may be filtered.

**Step 5: Test DNS**

```bash
dig example.com
```

If an IP-based connection works but a hostname-based connection fails, DNS is one possible cause.

**Step 6: Test the service port**

```bash
nc -vz TARGET_IP 443
```

Use the correct address and port for the service being tested.

**Step 7: Inspect the application**

For an HTTP-based service:

```bash
curl -v https://example.com
```

**Step 8: Capture traffic**

Use Wireshark or tcpdump to identify where the expected exchange stops.

### 25.2 Common Symptoms

| Symptom | Possible Causes |
|---|---|
| No IP address | DHCP failure, interface configuration |
| Cannot reach local gateway | Local link, VLAN, ARP, or gateway issue |
| IP works but hostname fails | DNS or name-resolution issue |
| Ping fails but TCP works | ICMP filtering or destination behavior |
| TCP connection times out | Packet dropping, routing, host failure |
| Connection is refused | No listener or active rejection |
| TCP connects but application fails | Application configuration, TLS, authentication, protocol error |
| Intermittent connections | Packet loss, congestion, unstable links, server overload |
| Wrong network interface | Routing or interface-selection issue |

### 25.3 Timeout vs. Connection Refused

**Timeout:** No suitable response arrives before the operation times out. Several network or host conditions may cause this.

**Connection refused:** A rejection response is received, often because no service is listening on the destination port.

These are useful clues, not absolute proof of a specific cause.

---

## 26. Networking in Cybersecurity

Networking knowledge supports both offensive security testing and defensive monitoring.

### 26.1 Reconnaissance

During an authorized assessment, network reconnaissance may involve:

- Discovering hosts
- Identifying reachable ports
- Detecting services
- Understanding network ranges
- Mapping reachable systems

Tools may include:

- Nmap
- Wireshark
- dig
- traceroute
- Netcat

### 26.2 Service Enumeration

After discovering an open port, identify the actual service and gather relevant information.

Examples:

- TCP 22: SSH
- TCP 445: SMB
- TCP 3389: RDP
- TCP 5985/5986: WinRM
- TCP 1433: Microsoft SQL Server

Do not assume the service solely from the port number.

### 26.3 Network Defense

Defenders use networking knowledge to:

- Restrict unnecessary services
- Segment networks
- Monitor traffic
- Detect suspicious communication
- Investigate incidents
- Enforce least privilege
- Reduce lateral movement opportunities

### 26.4 Lateral Movement

Lateral movement describes an attacker's movement between systems inside an environment after gaining an initial foothold.

Understanding routing, segmentation, authentication, and service exposure helps defenders identify and limit these paths.

### 26.5 Command-and-Control Traffic

Compromised systems may communicate with external infrastructure.

Possible indicators include:

- Repeated outbound connections
- Unusual DNS queries
- Unexpected destinations
- Periodic communication patterns
- Suspicious protocol usage

No single indicator proves that command-and-control activity is occurring. Correlate network observations with endpoint telemetry and other evidence.

### 26.6 The Core Enumeration Method

A useful approach in security labs is:

```text
Target
  |
  v
Reachability
  |
  v
Port Discovery
  |
  v
Service Identification
  |
  v
Service Enumeration
  |
  v
Vulnerability Assessment
  |
  v
Evidence and Documentation
```

The goal is to understand what is reachable, why it is reachable, and whether that exposure introduces a security risk.

---

## 27. Practical Labs

Perform these exercises on your own equipment, local virtual machines, or explicitly authorized training targets.

### Lab 1: Explore Local Network Configuration

Commands:

```bash
ip addr
ip route
ip neigh
```

Tasks:

- Identify your active interface.
- Record its IPv4 and IPv6 addresses, if present.
- Identify the default gateway.
- Inspect neighbor entries.
- Explain which route is used for a remote IPv4 destination.

### Lab 2: Understand DNS

Commands:

```bash
dig example.com
dig example.com A
dig example.com AAAA
dig example.com MX
```

Tasks:

- Identify the queried record type.
- Inspect the answer section.
- Compare the responses.
- Explain what happens when a cached DNS result is used.

### Lab 3: Compare TCP and UDP

Use Wireshark to observe DNS traffic and a TCP connection to a service you control.

Tasks:

- Identify the transport protocol.
- Locate source and destination ports.
- Find the TCP handshake.
- Explain why UDP does not use the TCP three-way handshake.

### Lab 4: Inspect Listening Ports

Run:

```bash
ss -tuln
```

Then, where permitted:

```bash
sudo ss -tulpn
```

Tasks:

- Identify listening TCP sockets.
- Identify listening UDP sockets.
- Determine which processes own the sockets, if shown.
- Explain the difference between `127.0.0.1` and `0.0.0.0` as bind addresses.

### Lab 5: Scan a Local Lab Host

Choose a virtual machine that you own or are authorized to assess.

Run:

```bash
nmap -sn TARGET_NETWORK
```

Then scan the intended lab host:

```bash
nmap -sV TARGET_IP
```

Tasks:

- Record discovered hosts.
- Record open ports.
- Identify detected services.
- Explain the limitations of service-version detection.

### Lab 6: Test a Local TCP Connection

In a controlled environment, start a listener:

```bash
nc -lv 4444
```

Connect from a second terminal:

```bash
nc 127.0.0.1 4444
```

Tasks:

- Exchange a test message.
- Observe the traffic in Wireshark.
- Identify the TCP handshake.
- Explain what the connection proves and what it does not prove.

### Lab 7: Troubleshoot a Failed Connection

Choose a lab service that is intentionally unavailable or use a controlled test environment.

Tasks:

- Check the local interface.
- Check the routing table.
- Test DNS if a hostname is involved.
- Test the destination port.
- Capture the connection attempt.
- Explain whether the evidence suggests a timeout, rejection, or application-level failure.

### Lab 8: Build a Mini Network Investigation Report

Investigate a training PCAP or a network exchange you generated yourself.

Your report should include:

- Objective
- Capture source
- Relevant hosts and ports
- Protocols observed
- Timeline of important events
- Supporting packet numbers or command output
- Findings
- Limitations
- Recommended follow-up steps

---

## 28. Quick Reference

### Important Linux Commands

| Purpose | Command |
|---|---|
| View interfaces | `ip addr` |
| View IPv4 addresses | `ip -4 addr` |
| View routes | `ip route` |
| View neighbor cache | `ip neigh` |
| View listening ports | `ss -tuln` |
| View sockets and processes | `sudo ss -tulpn` |
| DNS lookup | `dig example.com` |
| Alternative DNS lookup | `nslookup example.com` |
| Test connectivity | `ping -c 4 TARGET_IP` |
| Trace network path | `traceroute example.com` |
| Capture traffic | `sudo tcpdump -i eth0` |
| Read PCAP | `tcpdump -r capture.pcap` |

### Important Nmap Commands

| Purpose | Command |
|---|---|
| Host discovery | `nmap -sn TARGET_NETWORK` |
| Default port scan | `nmap TARGET_IP` |
| Specific ports | `nmap -p 22,80,443 TARGET_IP` |
| All TCP ports | `nmap -p- TARGET_IP` |
| Service detection | `nmap -sV TARGET_IP` |
| Default scripts | `nmap -sC TARGET_IP` |
| SYN scan | `sudo nmap -sS TARGET_IP` |

### Networking Concepts to Memorize

- IP address: identifies a network-layer endpoint.
- MAC address: used for link-layer delivery.
- Port: identifies a transport-layer endpoint.
- DNS: resolves names to DNS records.
- DHCP: provides network configuration.
- ARP: resolves IPv4 neighbors on a local link.
- TCP: reliable, ordered byte-stream transport.
- UDP: datagram transport without TCP-style reliability.
- Router: forwards packets between networks.
- Switch: forwards frames within a Layer 2 network.
- Firewall: filters traffic according to configured policy.
- NAT: translates address information.
- VPN: creates a virtual communication path.
- Subnet: a logical division of an IP address space.

### Troubleshooting Checklist

- [ ] Is the correct network interface active?
- [ ] Does the host have the expected IP address?
- [ ] Is the destination covered by a valid route?
- [ ] Is the default gateway configured correctly?
- [ ] Is local neighbor resolution working where applicable?
- [ ] Does DNS resolve the intended hostname?
- [ ] Is the destination port reachable?
- [ ] Is the application responding correctly?
- [ ] Could a firewall or network policy be blocking traffic?
- [ ] Have I captured enough evidence to support the conclusion?

---

## 29. Further Learning

### Official Documentation

- [Wireshark](https://www.wireshark.org/)
- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [Linux `ip` manual](https://man7.org/linux/man-pages/man8/ip.8.html)
- [Linux `ss` manual](https://man7.org/linux/man-pages/man8/ss.8.html)
- [tcpdump manual](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [RFC 9293 — TCP](https://www.rfc-editor.org/rfc/rfc9293)
- [RFC 768 — UDP](https://www.rfc-editor.org/rfc/rfc768)
- [RFC 791 — IPv4](https://www.rfc-editor.org/rfc/rfc791)
- [RFC 8200 — IPv6](https://www.rfc-editor.org/rfc/rfc8200)
- [RFC 1035 — DNS](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 2131 — DHCP](https://www.rfc-editor.org/rfc/rfc2131)
- [RFC 8446 — TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446)

### Personal Progress Checklist

- [ ] Understand the OSI and TCP/IP models.
- [ ] Explain encapsulation.
- [ ] Identify private, public, loopback, and link-local addresses.
- [ ] Understand CIDR and basic subnetting.
- [ ] Explain MAC addresses and ARP.
- [ ] Understand TCP and UDP.
- [ ] Explain the TCP three-way handshake.
- [ ] Recognize common ports and services.
- [ ] Perform DNS lookups.
- [ ] Explain DHCP DORA.
- [ ] Understand routing tables and default gateways.
- [ ] Explain NAT and its limitations.
- [ ] Understand firewalls and network segmentation.
- [ ] Inspect listening sockets on Linux.
- [ ] Troubleshoot DNS and TCP connectivity.
- [ ] Use Nmap in an authorized lab.
- [ ] Capture and inspect traffic with Wireshark.
- [ ] Analyze a PCAP and document findings.

---

**Core principle:** Understand how data moves from one endpoint to another. When a connection fails, follow the path from the local interface through routing and transport to the application, and use evidence to identify where the problem occurs.
