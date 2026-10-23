# Wireshark Notes

> Practical notes on network traffic analysis, packet inspection, protocol analysis, and PCAP investigation.

## Table of Contents

1. [Introduction](#1-introduction)
2. [What Is Wireshark?](#2-what-is-wireshark)
3. [Network Traffic Analysis Fundamentals](#3-network-traffic-analysis-fundamentals)
4. [Installing Wireshark](#4-installing-wireshark)
5. [Capturing Network Traffic](#5-capturing-network-traffic)
6. [Wireshark Interface](#6-wireshark-interface)
7. [Packet Analysis Basics](#7-packet-analysis-basics)
8. [Capture Filters vs. Display Filters](#8-capture-filters-vs-display-filters)
9. [Essential Display Filters](#9-essential-display-filters)
10. [TCP Analysis](#10-tcp-analysis)
11. [UDP Analysis](#11-udp-analysis)
12. [DNS Analysis](#12-dns-analysis)
13. [HTTP Analysis](#13-http-analysis)
14. [TLS and HTTPS](#14-tls-and-https)
15. [ARP, ICMP, and DHCP](#15-arp-icmp-and-dhcp)
16. [Following Network Conversations](#16-following-network-conversations)
17. [PCAP Investigation Workflow](#17-pcap-investigation-workflow)
18. [Cybersecurity Applications](#18-cybersecurity-applications)
19. [Useful Wireshark Features](#19-useful-wireshark-features)
20. [Command-Line Analysis with TShark](#20-command-line-analysis-with-tshark)
21. [Troubleshooting Common Issues](#21-troubleshooting-common-issues)
22. [Practical Exercises](#22-practical-exercises)
23. [Quick Reference](#23-quick-reference)
24. [Important Reminders](#24-important-reminders)

---

## 1. Introduction

Wireshark is one of the most widely used tools for network protocol analysis. It allows analysts to capture network packets, inspect protocol fields, reconstruct conversations, and investigate suspicious network activity.

These notes focus on practical usage, especially for:

- Network troubleshooting
- Cybersecurity fundamentals
- Digital forensics
- PCAP file investigation
- Network traffic analysis
- Blue team and SOC workflows
- Understanding network protocols
- Investigating suspicious connections in authorized labs

**Learning objective:** Understand how to inspect packets, identify network conversations, recognize important protocols, and extract useful evidence from packet captures.

## 2. What Is Wireshark?

Wireshark is an open-source network protocol analyzer.

It can capture packets from supported network interfaces and analyze previously saved packet capture files.

Common file extensions include:

- `.pcap`
- `.pcapng`

### What can Wireshark reveal?

Depending on the captured traffic and available decryption keys, Wireshark may reveal:

- Source and destination IP addresses
- Source and destination ports
- Protocol information
- TCP connection establishment and termination
- DNS queries and responses
- HTTP requests and responses
- Unencrypted credentials or other sensitive data
- Suspicious communication patterns
- File transfers and application payloads
- Signs of scanning, retransmissions, or network failures

### What Wireshark cannot automatically do

Wireshark is not an all-in-one security solution.

It does not automatically:

- Decrypt arbitrary HTTPS traffic
- Recover passwords that were never transmitted in observable form
- Identify every malicious packet
- See all traffic on every network interface
- Guarantee that a packet capture contains the complete conversation
- Prove that an IP address belongs to a specific person

Encrypted traffic may still expose metadata, such as IP addresses, ports, packet sizes, timing, and some handshake information.

## 3. Network Traffic Analysis Fundamentals

Before analyzing packets, understand the following concepts.

### 3.1 Packet

A packet is a unit of data transmitted across a network. The term is commonly used when discussing IP traffic.

### 3.2 Frame

A frame is a data unit at the data-link layer. In Wireshark, the captured frame includes the link-layer information available to the capture system.

A typical Ethernet packet capture can contain:

1. Ethernet header
2. IP header
3. TCP or UDP header
4. Application data, if present

The exact structure depends on the protocol and encapsulation.

### 3.3 IP Address

An IP address identifies a network-layer endpoint.

Examples:

- `192.168.1.10`
- `10.0.0.25`
- `8.8.8.8`

Private IPv4 address ranges include:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

### 3.4 MAC Address

A MAC address is used for link-layer communication on networks such as Ethernet and Wi-Fi.

Example:

`00:11:22:33:44:55`

A MAC address observed in a packet capture does not necessarily identify the ultimate destination across routed networks. Ethernet addresses normally identify the local link's sender and receiver.

### 3.5 Ports

Ports help identify application endpoints associated with transport protocols.

Common examples:

| Port | Common Service |
|---|---|
| 20/21 | FTP data/control, depending on mode |
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 67/68 | DHCP over UDP |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 445 | SMB |
| 3306 | MySQL |
| 3389 | RDP |
| 5432 | PostgreSQL |
| 8080 | Alternative HTTP service |

Ports are conventions, not proof of a service's identity. Services can use non-standard ports.

### 3.6 Protocols

A protocol defines rules for communication.

Examples:

- Ethernet: link-layer communication
- ARP: IPv4 address resolution on a local link
- IP: addressing and routing
- TCP: reliable, ordered byte-stream transport
- UDP: datagram transport without TCP-style reliability
- DNS: domain-name resolution
- HTTP: web communication
- TLS: cryptographic protection for many application protocols

### 3.7 The OSI Model

| Layer | Name | Examples |
|---|---|---|
| 7 | Application | HTTP, DNS, SMTP |
| 6 | Presentation | Data representation, encryption concepts |
| 5 | Session | Session management concepts |
| 4 | Transport | TCP, UDP |
| 3 | Network | IPv4, IPv6, ICMP |
| 2 | Data Link | Ethernet, Wi-Fi, ARP |
| 1 | Physical | Electrical, radio, optical signals |

The model is useful for organizing your investigation, although real protocol stacks do not always map perfectly to these layers.

## 4. Installing Wireshark

### Kali Linux / Debian-based systems

Install Wireshark using APT:

```bash
sudo apt update
sudo apt install wireshark
```

Verify the installation:

```bash
wireshark --version
```

Launch the graphical interface:

```bash
wireshark
```

For command-line analysis, install or verify TShark:

```bash
tshark --version
```

### Permissions

Packet capture may require elevated privileges or appropriate capture permissions.

Avoid running the entire graphical application as root unless necessary. Prefer the operating system's supported capture-permission configuration.

On Debian-based systems, package installation may offer a choice about whether non-root users can capture packets. Follow the distribution's security guidance and configure membership or capabilities only when appropriate.

### Windows

Download Wireshark from the official website:

https://www.wireshark.org/

The Windows installer normally offers to install Npcap, which provides packet-capture functionality.

Only download installers from trusted, official sources.

## 5. Capturing Network Traffic

### 5.1 Identify Network Interfaces

On Linux:

```bash
ip addr
```

Show routing information:

```bash
ip route
```

List interfaces with TShark:

```bash
tshark -D
```

Common interface names include:

- `eth0`: Ethernet interface
- `wlan0`: wireless interface
- `lo`: loopback interface

Actual names depend on the system.

### 5.2 Start a Capture

Open Wireshark and select the interface carrying the traffic you want to investigate.

Double-click the interface or start capturing with the capture button.

Useful interfaces may include:

- Ethernet
- Wi-Fi
- Loopback
- VPN or virtual-machine interfaces

Choose the correct interface before drawing conclusions from an empty capture.

### 5.3 Generate Test Traffic

In a separate terminal, generate some ordinary traffic:

```bash
ping -c 4 1.1.1.1
```

Perform a DNS lookup:

```bash
nslookup example.com
```

Send an HTTP request:

```bash
curl -I http://example.com
```

These commands can generate packets for you to inspect. The actual traffic depends on DNS configuration, routing, proxies, caching, and network access.

### 5.4 Save a Capture

Use **File → Save As** and save the capture as a `.pcapng` file.

Example filename:

`dns-http-lab.pcapng`

Capture files may contain sensitive information. Do not upload real credentials, private network traffic, or confidential organizational data to public repositories.

### 5.5 Stop Capturing

Stop the capture before beginning a detailed investigation when a stable snapshot is more convenient.

For a live incident, preserve the original capture and document when and where it was collected.

## 6. Wireshark Interface

The main interface contains several important areas.

### 6.1 Packet List

Displays packets in rows.

Common columns include:

- No.: packet number
- Time: timestamp relative to the selected time reference
- Source: source address
- Destination: destination address
- Protocol: detected protocol
- Length: packet or frame length
- Info: brief protocol summary

### 6.2 Packet Details

Displays the selected packet's protocol tree.

For an Ethernet/IPv4/TCP packet, expand:

- Frame
- Ethernet II
- Internet Protocol Version 4
- Transmission Control Protocol
- Application protocol, if recognized

This is where you inspect flags, ports, lengths, checksums, sequence numbers, and other fields.

### 6.3 Packet Bytes

Shows the captured packet data in hexadecimal and, where possible, ASCII.

This view is useful for understanding how protocol fields are represented in raw bytes.

However, binary data is not necessarily meaningful text, and encrypted payloads should not be expected to decode into readable application content.

### 6.4 Coloring Rules

Wireshark uses colors to help distinguish types of traffic and potential issues.

Colors depend on the active coloring rules. A red or dark-colored packet does not automatically mean an attack occurred.

Always inspect the protocol fields and surrounding conversation.

## 7. Packet Analysis Basics

A reliable investigation starts with a repeatable process.

### Step 1: Select an interesting packet

Start with a DNS query, TCP connection, HTTP request, or unfamiliar endpoint.

### Step 2: Inspect the protocol stack

Expand each protocol layer in Packet Details.

Ask:

- What is the source address?
- What is the destination address?
- Which transport protocol is used?
- Which ports are involved?
- Which application protocol is recognized?

### Step 3: Examine protocol fields

For TCP, inspect:

- Source port
- Destination port
- Sequence number
- Acknowledgment number
- SYN, ACK, FIN, and RST flags
- Window size
- TCP payload length

For DNS, inspect:

- Query name
- Query type
- Response code
- Answer records

For HTTP, inspect:

- Request method
- Host
- Request URI
- Response status code
- Relevant headers
- Visible payload, if unencrypted

### Step 4: Follow the conversation

Use **Analyze → Follow** and select the relevant protocol stream when supported.

This can provide a more readable view of a conversation than inspecting individual packets.

### Step 5: Build a timeline

Determine which event happened first, what followed, and whether the sequence is consistent with the suspected activity.

### Step 6: Record evidence

Document packet numbers, timestamps, endpoints, protocol fields, and your interpretation.

Separate observations from hypotheses.

## 8. Capture Filters vs. Display Filters

This distinction is extremely important.

### 8.1 Capture Filters

Capture filters determine which packets are collected in the first place.

They use the capture-filter syntax supported by libpcap.

Examples:

Capture traffic involving a host:

```text
host 192.168.1.10
```

Capture traffic involving a specific network:

```text
net 192.168.1.0/24
```

Capture TCP traffic:

```text
tcp
```

Capture traffic involving port 443:

```text
port 443
```

Capture only DNS traffic:

```text
udp port 53
```

Capture traffic to or from a specific destination port:

```text
dst port 443
```

Capture traffic to or from a host using TCP:

```text
tcp and host 192.168.1.10
```

Capture filters reduce the amount of traffic saved. If a filter excludes relevant packets, those packets will not be available for later analysis.

### 8.2 Display Filters

Display filters control which packets are shown in the current Wireshark view.

They do not remove the excluded packets from the underlying capture.

Examples:

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

Show packets involving an IP:

```text
ip.addr == 192.168.1.10
```

Show packets from an IP:

```text
ip.src == 192.168.1.10
```

Show packets to an IP:

```text
ip.dst == 192.168.1.10
```

Show traffic involving port 443:

```text
tcp.port == 443
```

### Important distinction

| Feature | Capture Filter | Display Filter |
|---|---|---|
| When it applies | During capture | During analysis |
| Syntax | Capture-filter syntax | Wireshark display-filter syntax |
| Removes packets from saved capture | Excluded packets are never captured by that capture session | No |
| Can be changed after capture | No, not to recover omitted packets | Yes |
| Typical use | Reduce captured traffic | Investigate existing traffic |

**Memory tip:** Capture filters decide what to collect. Display filters decide what to view.

## 9. Essential Display Filters

The following examples use Wireshark display-filter syntax.

### 9.1 IP Address Filters

```text
ip.addr == 192.168.1.10
```

```text
ip.src == 192.168.1.10
```

```text
ip.dst == 192.168.1.10
```

For IPv6, use fields such as:

```text
ipv6.addr == 2001:db8::10
```

The IPv6 address above is an example from the documentation address space.

### 9.2 Protocol Filters

```text
tcp
```

```text
udp
```

```text
dns
```

```text
http
```

```text
tls
```

```text
arp
```

```text
icmp
```

```text
dhcp
```

Protocol names and available fields can vary with Wireshark versions and protocol dissectors.

### 9.3 Port Filters

TCP source or destination port 80:

```text
tcp.port == 80
```

UDP port 53:

```text
udp.port == 53
```

Destination TCP port 22:

```text
tcp.dstport == 22
```

Source TCP port 443:

```text
tcp.srcport == 443
```

### 9.4 Combine Conditions

Use `and` to require both conditions:

```text
ip.addr == 192.168.1.10 and tcp
```

Use `or` to match either condition:

```text
dns or http
```

Use `not` to exclude a condition:

```text
not arp
```

Use parentheses to control logical grouping:

```text
(ip.addr == 192.168.1.10) and (tcp.port == 80 or tcp.port == 443)
```

### 9.5 HTTP Request Filters

Show HTTP GET requests:

```text
http.request.method == "GET"
```

Show HTTP POST requests:

```text
http.request.method == "POST"
```

Show HTTP responses with status code 404:

```text
http.response.code == 404
```

Show requests for a particular host:

```text
http.host == "example.com"
```

These filters only match HTTP fields that Wireshark can dissect. Ordinary HTTPS application data is encrypted and will not generally expose these HTTP fields without decryption.

### 9.6 DNS Filters

Show DNS queries:

```text
dns.flags.response == 0
```

Show DNS responses:

```text
dns.flags.response == 1
```

Show DNS queries for a specific name:

```text
dns.qry.name == "example.com"
```

Show DNS response codes other than NOERROR:

```text
dns.flags.rcode != 0
```

A nonzero DNS response code indicates a response condition that needs interpretation. It does not automatically indicate malicious activity.

### 9.7 TCP Flag Filters

Show SYN packets:

```text
tcp.flags.syn == 1
```

Show SYN packets without ACK:

```text
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

Show RST packets:

```text
tcp.flags.reset == 1
```

Show FIN packets:

```text
tcp.flags.fin == 1
```

Show packets marked as retransmissions:

```text
tcp.analysis.retransmission
```

Show duplicate acknowledgments:

```text
tcp.analysis.duplicate_ack
```

Wireshark's TCP analysis flags are diagnostic indicators, not definitive proof of an attack.

### 9.8 Packet Length

Show frames larger than 1,000 bytes:

```text
frame.len > 1000
```

Show small frames:

```text
frame.len < 100
```

Frame length includes the captured frame as represented in the capture, not just the application payload.

### 9.9 Expert Information

Open **Analyze → Expert Information** to review warnings, notes, and other protocol-analysis observations.

To inspect TCP resets:

```text
tcp.flags.reset == 1
```

To inspect malformed packets when the relevant field is available:

```text
_ws.malformed
```

Not every warning is a security issue. Capture loss, offloading, incomplete traffic, and unusual but valid protocol behavior can all affect the analysis.

## 10. TCP Analysis

TCP provides a reliable, ordered byte stream between endpoints.

### 10.1 The TCP Three-Way Handshake

A normal TCP connection usually begins with three packets:

1. SYN
2. SYN-ACK
3. ACK

Example:

```text
Client -> Server: SYN
Server -> Client: SYN, ACK
Client -> Server: ACK
```

The handshake establishes connection state and synchronizes sequence numbers.

### 10.2 Find Initial Connection Attempts

Use:

```text
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

This displays initial SYN packets, which can help identify connection attempts.

A large number of SYN packets directed toward many ports may be consistent with port scanning, but it can also result from monitoring, testing, or other legitimate activity.

Correlate the source, destination, timing, and surrounding packets before drawing a conclusion.

### 10.3 TCP Reset (RST)

A reset can terminate or reject a TCP connection.

Filter:

```text
tcp.flags.reset == 1
```

Possible explanations include:

- A closed port
- An application aborting a connection
- A firewall or middlebox resetting traffic
- Unexpected connection state

A reset alone does not prove an attack.

### 10.4 TCP Retransmissions

TCP may retransmit data when acknowledgments are missing or network conditions cause delivery problems.

Filter:

```text
tcp.analysis.retransmission
```

Possible causes include:

- Packet loss
- Congestion
- Wireless interference
- Delayed acknowledgments
- Capture artifacts

Repeated retransmissions may indicate connectivity problems, but the underlying cause requires further investigation.

### 10.5 TCP Sequence and Acknowledgment Numbers

Sequence numbers track byte positions in a TCP stream.

Acknowledgment numbers indicate the next byte expected from the other endpoint.

Wireshark usually uses relative sequence numbers to make analysis easier.

These fields help investigate:

- Missing data
- Out-of-order segments
- Retransmissions
- Stream reconstruction
- Connection behavior

### 10.6 TCP Stream Identification

Each TCP connection can be distinguished by its endpoint addresses and ports, along with the transport protocol.

A useful display filter is:

```text
tcp.stream eq 0
```

Replace `0` with the stream index you want to inspect.

Stream numbers are local identifiers within a capture, not universal connection IDs.

### 10.7 TCP Connection Timeline

When analyzing a connection, ask:

1. Was the handshake completed?
2. Which side sent the first application data?
3. Was the connection idle for a long period?
4. Were there retransmissions or resets?
5. Did the connection close cleanly?
6. Is the observed sequence consistent with the expected application behavior?

## 11. UDP Analysis

UDP is a connectionless transport protocol. It does not provide TCP-style handshakes, ordered delivery, or retransmission guarantees.

Common uses include:

- DNS
- DHCP
- Some streaming applications
- Voice and video traffic
- Certain VPN and tunneling protocols

### Basic filters

```text
udp
```

```text
udp.port == 53
```

```text
udp.dstport == 53
```

### UDP vs. TCP

| Property | TCP | UDP |
|---|---|---|
| Connection setup | Normally uses a handshake | No TCP-style handshake |
| Delivery guarantee | Reliable byte-stream transport | No built-in delivery guarantee |
| Ordering | Ordered byte stream | No built-in ordering guarantee |
| Retransmission | Managed by TCP | Application-dependent |
| Common examples | SSH, HTTP over TCP, SMB | DNS, DHCP, many real-time applications |

HTTP/3 uses QUIC over UDP, so seeing UDP traffic on port 443 does not automatically mean it is DNS or suspicious traffic.

## 12. DNS Analysis

DNS translates domain names into records used by networked systems.

A typical DNS lookup involves a query and a response, although caching, retries, encrypted DNS, and other resolver behavior can change what appears in a capture.

### 12.1 Inspect DNS Queries

Filter:

```text
dns.flags.response == 0
```

Look for:

- Query name
- Query type
- Query identifier
- Source and destination
- Query timing

Common record types include:

| Type | Purpose |
|---|---|
| A | IPv4 address |
| AAAA | IPv6 address |
| CNAME | Canonical name alias |
| MX | Mail exchanger |
| NS | Name server |
| TXT | Text record |
| PTR | Reverse DNS lookup |

### 12.2 Inspect DNS Responses

Filter:

```text
dns.flags.response == 1
```

Inspect:

- Response code
- Answer records
- TTL values
- Returned IP addresses
- Whether the answer matches the query

### 12.3 DNS Failures

A DNS response with a nonzero response code can indicate a failed lookup.

For example, NXDOMAIN means the queried name does not exist according to the responding DNS server.

Possible reasons include:

- A misspelled domain
- A removed domain
- An incorrect configuration
- A nonexistent internal hostname
- Suspicious or randomly generated domain names

A single NXDOMAIN response is not enough to establish malicious activity.

### 12.4 DNS Tunneling Indicators

DNS tunneling can encode information in DNS queries or responses.

Potential indicators include:

- Unusually long subdomain labels
- High volumes of queries to one domain
- Many unique subdomains
- High-entropy-looking names
- Unexpectedly frequent TXT queries
- Unusual periodic query patterns

Example of a suspicious-looking query name:

```text
a8f2k9x7p4m1.example.com
```

This is only an illustrative example. Random-looking labels can also occur in legitimate tracking, content delivery, and security systems.

Investigate query volume, domain reputation, timing, response sizes, and the host generating the traffic.

## 13. HTTP Analysis

HTTP is an application-layer protocol used for web communication.

Unencrypted HTTP traffic may expose request and response details directly in a packet capture.

### 13.1 HTTP Request Structure

A request commonly contains:

- Request method
- Request target
- HTTP version
- Headers
- Optional request body

Example:

```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: ExampleBrowser/1.0
Accept: text/html
```

### 13.2 HTTP Methods

| Method | Common Purpose |
|---|---|
| GET | Retrieve a resource |
| POST | Submit data |
| PUT | Create or replace a resource |
| PATCH | Partially modify a resource |
| DELETE | Request resource deletion |
| HEAD | Retrieve response headers without the usual response body |

The server's behavior and authorization rules determine what a method actually does.

### 13.3 HTTP Status Codes

| Code | Meaning |
|---|---|
| 200 | OK |
| 301 | Moved Permanently |
| 302 | Found / temporary redirect |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

### 13.4 Useful Filters

Show all recognized HTTP traffic:

```text
http
```

Show GET requests:

```text
http.request.method == "GET"
```

Show POST requests:

```text
http.request.method == "POST"
```

Show server errors:

```text
http.response.code >= 500
```

Show requests for a host:

```text
http.host == "example.com"
```

### 13.5 Inspecting HTTP Payloads

Select a packet and expand the HTTP section in Packet Details.

For a readable conversation, try:

**Right-click packet → Follow → TCP Stream**

This can reveal the sequence of HTTP requests and responses in the stream.

Depending on the traffic, a stream may contain:

- URL paths
- Headers
- Form parameters
- Cookies
- Session identifiers
- Application content

Only inspect sensitive data when you are authorized to do so.

### 13.6 Security Implications

Unencrypted HTTP can expose sensitive data to anyone with sufficient access to the traffic path or capture point.

HTTPS protects application content in transit when correctly configured, although it does not eliminate all metadata exposure or other security risks.

## 14. TLS and HTTPS

HTTPS generally means HTTP protected by TLS.

TLS provides encryption, integrity protection, and authentication mechanisms.

### 14.1 What Can Be Observed?

Without decryption, a capture may reveal:

- IP addresses
- Ports
- Connection timing
- Packet sizes
- TLS handshake messages
- Negotiated TLS version and cipher suite, when visible
- Certificates and related handshake details, depending on TLS version and configuration
- Some server-name information, depending on the handshake and privacy features

TLS 1.3 encrypts more of the handshake than earlier TLS versions. Server Name Indication (SNI) may be concealed when Encrypted Client Hello (ECH) is used successfully.

### 14.2 Useful Filters

```text
tls
```

```text
tcp.port == 443
```

```text
tls.handshake
```

```text
tls.alert
```

These filters show different aspects of TLS traffic, subject to protocol dissection and capture completeness.

### 14.3 Why HTTPS Payloads Look Unreadable

Encrypted application data is not normally readable as plain HTTP.

This is expected behavior, not evidence of malicious activity.

### 14.4 TLS Decryption

Wireshark can decrypt certain TLS sessions when suitable secrets are available and supported by the connection.

A common method is to configure a TLS key-log file:

1. Configure a supported client application to export TLS session secrets.
2. Capture the relevant connection.
3. Open Wireshark preferences.
4. Navigate to **Protocols → TLS**.
5. Configure the `(Pre)-Master-Secret log filename` field.
6. Reload or reopen the capture as needed.

Exact preference labels can vary by version.

A key-log file contains highly sensitive secrets. Protect it, avoid committing it to GitHub, and delete or securely store it according to your lab's requirements.

Possessing only the server certificate's public key is generally insufficient to decrypt modern TLS sessions using ephemeral key exchange.

## 15. ARP, ICMP, and DHCP

### 15.1 ARP

Address Resolution Protocol maps an IPv4 address to a link-layer address on a local network.

Filter:

```text
arp
```

An ARP request commonly asks:

"Who has this IPv4 address? Tell the requester."

ARP is a local-link protocol and is not routed across IP networks in the same way as ordinary IP packets.

#### ARP Spoofing Indicators

Potential indicators include:

- Unexpected changes in IP-to-MAC mappings
- Multiple MAC addresses claiming the same IP
- Unusual bursts of ARP replies
- A host unexpectedly claiming the gateway's address

These patterns can also have legitimate explanations, such as failover, virtualization, or network reconfiguration.

Correlate ARP observations with network configuration and other evidence.

### 15.2 ICMP

ICMP is used for IP network diagnostics and error reporting.

Filter:

```text
icmp
```

For IPv6, inspect ICMPv6:

```text
icmpv6
```

The `ping` utility commonly uses ICMP Echo Request and Echo Reply messages for IPv4.

A failed ping does not prove that a host is offline. Firewalls and network policies may block ICMP.

### 15.3 DHCP

DHCP helps clients obtain IP configuration automatically.

For IPv4, the common initial exchange is often described as DORA:

1. Discover
2. Offer
3. Request
4. Acknowledge

Filter:

```text
dhcp
```

Depending on the Wireshark version, DHCPv4 may also be identified with the `bootp` display-filter protocol name.

Useful information includes:

- Client MAC address
- Offered IP address
- DHCP server identifier
- Lease duration
- Router and DNS options

Unexpected DHCP servers or suspicious configuration offers may indicate a rogue DHCP server, but confirm the expected network architecture first.

## 16. Following Network Conversations

Following a stream is one of the most useful Wireshark techniques.

### Follow TCP Stream

1. Select a TCP packet.
2. Right-click it.
3. Choose **Follow → TCP Stream**.
4. Examine the reconstructed conversation.

Wireshark filters the display to the selected stream and provides a readable representation of the captured data.

### Other Follow Options

Depending on the selected packet and protocol, Wireshark may offer:

- UDP Stream
- TLS Stream
- HTTP/2 Stream
- QUIC Stream

Available options depend on protocol support and the selected packet.

### Why Follow Streams?

Individual packets show fragments of a conversation. Following a stream helps reconstruct the larger sequence.

This is useful for:

- HTTP request and response analysis
- Identifying application protocols
- Examining text-based protocols
- Understanding client-server exchanges
- Finding visible strings in an authorized PCAP

### Limitations

Stream reconstruction cannot recover packets that were never captured.

Missing data, encryption, compression, and application framing may also limit what can be reconstructed.

## 17. PCAP Investigation Workflow

A PCAP file is a saved packet capture. Investigating one is a common task in network forensics and cybersecurity labs.

### Phase 1: Preserve the Evidence

Before analysis:

- Keep an unchanged copy of the original file.
- Record the filename and source.
- Record the acquisition time and relevant context.
- Avoid editing or overwriting the original capture.
- Follow applicable evidence-handling procedures.

For formal investigations, hash the original file:

```bash
sha256sum capture.pcapng
```

Record the hash so you can later verify whether the file has changed.

### Phase 2: Understand the Capture

Open the file in Wireshark.

Check:

- Capture duration
- Number of packets
- Link-layer type
- Timestamp range
- Whether the capture appears truncated
- Whether relevant interfaces or traffic are missing

Open **Statistics → Capture File Properties** where available.

### Phase 3: Review Protocol Distribution

Open:

**Statistics → Protocol Hierarchy**

This helps identify the protocols present and their relative proportions.

Ask:

- Is the capture mostly TCP or UDP?
- Are there DNS queries?
- Is HTTP visible?
- Is there a large amount of TLS traffic?
- Are unexpected protocols present?

### Phase 4: Identify Conversations

Open:

**Statistics → Conversations**

Review Ethernet, IPv4, IPv6, TCP, and UDP conversations as applicable.

Look for:

- Frequent communicating hosts
- Unusual external destinations
- High-volume connections
- Unexpected ports
- Long-lived connections
- One-to-many communication patterns

### Phase 5: Inspect Endpoints

Open:

**Statistics → Endpoints**

Determine which hosts appear in the capture.

Use IP-address filters to isolate potentially relevant traffic.

Remember that a capture only represents the traffic visible at its collection point.

### Phase 6: Examine DNS

Use:

```text
dns
```

Identify domain names, query patterns, failures, and returned addresses.

Correlate DNS results with subsequent connections when possible.

### Phase 7: Examine TCP and UDP

Useful starting filters:

```text
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

```text
tcp.analysis.retransmission
```

```text
udp
```

Inspect unfamiliar destinations and determine whether the observed behavior has a reasonable explanation.

### Phase 8: Inspect Application Data

Use protocol filters and stream reconstruction.

For example:

```text
http
```

```text
ftp
```

```text
smtp
```

The actual content visible depends on the protocol, encryption, capture location, and available dissectors.

### Phase 9: Correlate Events

Build a timeline connecting:

1. DNS resolution
2. Initial connection
3. Application requests
4. Responses or failures
5. Connection termination

Not every application performs a fresh DNS lookup during the captured time window, so missing DNS packets do not automatically indicate an anomaly.

### Phase 10: Document Findings

A useful investigation report contains:

- Summary
- Scope and data source
- Relevant timestamps
- Source and destination addresses
- Protocols and ports
- Supporting packet numbers
- Observed behavior
- Possible explanations
- Confidence and limitations
- Recommended follow-up actions

Avoid declaring malicious activity solely because a packet looks unusual.

## 18. Cybersecurity Applications

### 18.1 Network Scanning Detection

Potential indicators include:

- Many initial SYN packets
- Connections to numerous ports on one host
- Connections to the same port across many hosts
- Repeated failed connection attempts

Useful filter:

```text
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

Use Statistics → Conversations and Endpoints to understand the traffic distribution.

A port scan hypothesis becomes stronger when the source, timing, destination-port pattern, and connection outcomes support it.

### 18.2 Suspicious DNS Activity

Investigate:

- Unusually long domain labels
- Many unique subdomains
- High query frequency
- Unexpected DNS record types
- Repeated failures
- DNS traffic from unusual hosts

No single DNS pattern proves malware or command-and-control activity.

### 18.3 Cleartext Credential Exposure

Some legacy or insecure protocols can transmit authentication data without adequate encryption.

Examples may include:

- FTP
- Telnet
- Some HTTP login forms
- Insecure mail configurations

In an authorized lab, inspect the relevant protocol fields and follow the stream to understand what is transmitted.

Do not attempt to collect or use other people's credentials.

### 18.4 Data Exfiltration Investigation

Potential indicators include:

- Large outbound transfers
- Unexpected destinations
- Unusual transfer timing
- Repeated communication with unfamiliar hosts
- DNS or other protocol traffic carrying unexpectedly large amounts of data

To investigate:

1. Identify the suspected source host.
2. Review its conversations.
3. Determine the destination and protocol.
4. Examine packet sizes and transfer direction.
5. Inspect application details where visible.
6. Correlate findings with endpoint and server logs.

Large transfers can be entirely legitimate. Establish context before concluding that data was exfiltrated.

### 18.5 Malware Traffic Analysis

A packet capture may help identify:

- Repeated beacon-like connections
- Suspicious DNS lookups
- Unusual destinations or ports
- Unexpected protocol usage
- Failed or repeated connection attempts
- Possible command-and-control communication

Encrypted traffic often limits visibility into application content. Use endpoint telemetry, DNS logs, proxy logs, and other evidence when available.

### 18.6 Network Troubleshooting

Wireshark can help diagnose:

- DNS resolution failures
- TCP connection failures
- Retransmissions
- Unexpected resets
- Slow application responses
- DHCP configuration issues
- MTU-related symptoms
- Protocol misconfiguration

Packet timing is useful, but distinguish network delay from server processing time and client-side delays.

## 19. Useful Wireshark Features

### 19.1 Statistics Menu

Important sections include:

- Protocol Hierarchy
- Conversations
- Endpoints
- I/O Graphs
- Flow Graph
- Capture File Properties
- Expert Information

These features help summarize traffic before you inspect individual packets.

### 19.2 I/O Graphs

Use I/O Graphs to visualize traffic volume over time.

This may help reveal:

- Bursts of traffic
- Periods of inactivity
- Repeated communication patterns
- Large transfers

Graph patterns are investigative leads, not proof of a particular event.

### 19.3 Flow Graph

The Flow Graph can illustrate packet exchanges between hosts.

It is especially useful for understanding:

- TCP handshakes
- Request-response sequences
- Retransmissions
- Connection termination
- Multi-host interactions

### 19.4 Export Objects

Wireshark may be able to extract supported application objects from certain unencrypted or decrypted protocols.

Depending on the protocol, options may appear under **File → Export Objects**.

Exported files are potentially dangerous. Treat them as untrusted artifacts and never execute suspicious files on your primary system.

### 19.5 Name Resolution

Wireshark can resolve some addresses to names when name resolution is enabled and suitable information is available.

Be careful when analyzing suspicious traffic: resolution may generate external lookups or produce names that are incomplete or misleading.

For repeatable forensic analysis, document whether name resolution was enabled.

### 19.6 Time Display

Wireshark supports different time display formats.

For incident timelines, consider absolute date and time when the capture timestamps are trustworthy.

Confirm timezone settings before comparing packet timestamps with system logs.

## 20. Command-Line Analysis with TShark

TShark is Wireshark's command-line protocol analyzer.

It is useful for automation, quick inspection, and headless environments.

### 20.1 Show Version

```bash
tshark --version
```

### 20.2 List Interfaces

```bash
tshark -D
```

### 20.3 Capture Packets

Example: capture traffic on interface `eth0`.

```bash
sudo tshark -i eth0
```

The interface must exist on your system. Replace it with the correct interface name.

Capture a limited number of packets:

```bash
sudo tshark -i eth0 -c 100
```

### 20.4 Read a PCAP File

```bash
tshark -r capture.pcapng
```

### 20.5 Apply a Display Filter

```bash
tshark -r capture.pcapng -Y "dns"
```

Show HTTP requests:

```bash
tshark -r capture.pcapng -Y "http.request"
```

Show initial TCP SYN packets:

```bash
tshark -r capture.pcapng -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0"
```

The `-Y` option accepts a Wireshark display filter.

### 20.6 Apply a Capture Filter

```bash
sudo tshark -i eth0 -f "tcp port 443"
```

The `-f` option uses capture-filter syntax, not display-filter syntax.

### 20.7 Extract Selected Fields

Display source IP, destination IP, and protocol:

```bash
tshark -r capture.pcapng \
  -T fields \
  -e ip.src \
  -e ip.dst \
  -e _ws.col.Protocol
```

Extract DNS query names:

```bash
tshark -r capture.pcapng \
  -Y "dns.flags.response == 0" \
  -T fields \
  -e frame.number \
  -e ip.src \
  -e dns.qry.name
```

Output field names depend on the dissected protocol and packet contents.

### 20.8 Export Results

Write selected packet details to a text file:

```bash
tshark -r capture.pcapng -Y "dns" > dns-analysis.txt
```

Be mindful of sensitive information when saving or sharing analysis output.

## 21. Troubleshooting Common Issues

### Problem: No Packets Appear

Possible causes:

- Incorrect interface selected
- Capture permissions missing
- No traffic generated
- Traffic routed through another interface
- Capture started after the relevant exchange
- Virtual machine or VPN networking differences

Check:

```bash
ip addr
ip route
tshark -D
```

Generate test traffic and confirm the active interface.

### Problem: HTTP Filter Shows Nothing

Possible causes:

- The application uses HTTPS
- The traffic uses HTTP/3 over QUIC
- The relevant packets are missing
- Wireshark cannot dissect the traffic as HTTP
- A display filter excludes the relevant packets

Try:

```text
tcp.port == 80
```

or inspect:

```text
tls
```

```text
quic
```

These are starting points, not guaranteed matches for every web connection.

### Problem: HTTPS Content Is Not Readable

This is expected when the application payload is encrypted and decryption secrets are unavailable.

Inspect visible metadata and handshake information instead.

### Problem: Display Filter Is Invalid

Check:

- Field spelling
- Correct filter syntax
- String quotation marks
- Comparison operators
- Wireshark version and protocol support

Use Wireshark's filter autocomplete to discover valid fields.

### Problem: Too Much Traffic

Start with narrow display filters:

```text
dns
```

```text
tcp
```

```text
ip.addr == 192.168.1.10
```

Then inspect Conversations and Endpoints to find relevant hosts.

### Problem: Packets Appear Malformed

Potential explanations include:

- Incomplete capture
- Truncated packets
- Incorrect or unexpected encapsulation
- Unsupported protocol decoding
- Corrupted traffic
- Capture artifacts

Do not automatically assume malformed traffic is malicious.

## 22. Practical Exercises

Perform these exercises using your own machine, authorized lab traffic, or publicly provided training PCAPs.

### Exercise 1: Basic Traffic Capture

**Goal:** Understand packet structure.

1. Start a Wireshark capture on the correct interface.
2. Run `ping -c 4 1.1.1.1`.
3. Filter for `icmp`.
4. Inspect an Echo Request and Echo Reply, if replies are received.
5. Expand the Ethernet, IP, and ICMP sections.

Questions:

- Which address sent the request?
- Which address received it?
- What is the ICMP message type?
- Did you receive a reply?

### Exercise 2: DNS Investigation

**Goal:** Learn DNS query and response analysis.

1. Start a capture.
2. Run `nslookup example.com`.
3. Apply the filter `dns`.
4. Inspect the query and corresponding response.

Questions:

- What was the queried name?
- What record type was requested?
- What response code was returned?
- Which addresses appeared in the answer?

Note: DNS caching or encrypted DNS may affect what appears in the capture.

### Exercise 3: HTTP Request Analysis

**Goal:** Inspect unencrypted HTTP in a controlled environment.

Use a lab web server or another HTTP endpoint you are authorized to access.

1. Capture the traffic.
2. Send an HTTP request with `curl`.
3. Filter for `http`.
4. Inspect the request method, host, URI, and response status.
5. Follow the relevant TCP stream.

Questions:

- Which method was used?
- What response status was returned?
- Which headers were visible?
- Could the application data be read without decryption?

### Exercise 4: TCP Handshake

**Goal:** Understand connection establishment.

1. Capture a new TCP connection.
2. Filter for:

   ```text
   tcp.flags.syn == 1 or tcp.flags.fin == 1 or tcp.flags.reset == 1
   ```

3. Find the initial SYN packet.
4. Inspect the corresponding SYN-ACK and ACK if present.
5. Examine how the connection terminates.

Questions:

- Was the handshake completed?
- Which endpoint initiated the connection?
- Was a FIN or RST observed?
- Are any expected packets missing?

### Exercise 5: Analyze a Provided PCAP

**Goal:** Practice a structured investigation.

1. Open a training PCAP.
2. Inspect Protocol Hierarchy.
3. Review Conversations and Endpoints.
4. Identify DNS queries.
5. Investigate unusual destinations.
6. Follow interesting streams.
7. Record evidence with packet numbers and timestamps.

Deliverable: Write a short report explaining the traffic, the most important observations, and any remaining uncertainty.

### Exercise 6: Detect Possible Scanning

**Goal:** Learn to identify patterns consistent with port scanning.

Use a lab capture containing known scanning activity, or generate a controlled scan against a system you own or are explicitly authorized to test.

1. Filter for initial SYN packets.
2. Identify the source address.
3. Determine the destination hosts and ports.
4. Look for repeated connection attempts.
5. Check the responses and surrounding traffic.

Explain why the pattern is or is not consistent with scanning.

### Exercise 7: Build a Timeline

**Goal:** Improve incident-investigation skills.

Choose a PCAP with a multi-step network exchange.

Record:

- First relevant DNS query
- First relevant connection attempt
- First application request
- Important responses
- Last observed packet in the conversation

Create a chronological summary and distinguish observed facts from assumptions.

## 23. Quick Reference

### Essential Display Filters

| Purpose | Filter |
|---|---|
| All TCP | `tcp` |
| All UDP | `udp` |
| All DNS | `dns` |
| All HTTP | `http` |
| TLS traffic | `tls` |
| ARP | `arp` |
| ICMP | `icmp` |
| Traffic involving an IP | `ip.addr == 192.168.1.10` |
| Traffic from an IP | `ip.src == 192.168.1.10` |
| Traffic to an IP | `ip.dst == 192.168.1.10` |
| TCP port 80 | `tcp.port == 80` |
| UDP port 53 | `udp.port == 53` |
| HTTP GET | `http.request.method == "GET"` |
| HTTP POST | `http.request.method == "POST"` |
| DNS queries | `dns.flags.response == 0` |
| DNS responses | `dns.flags.response == 1` |
| Initial TCP SYN | `tcp.flags.syn == 1 and tcp.flags.ack == 0` |
| TCP resets | `tcp.flags.reset == 1` |
| TCP retransmissions | `tcp.analysis.retransmission` |
| Frames over 1,000 bytes | `frame.len > 1000` |

### Essential Capture Filters

| Purpose | Filter |
|---|---|
| Specific host | `host 192.168.1.10` |
| Specific network | `net 192.168.1.0/24` |
| TCP traffic | `tcp` |
| UDP traffic | `udp` |
| Port 443 | `port 443` |
| DNS over UDP | `udp port 53` |
| TCP traffic involving a host | `tcp and host 192.168.1.10` |

### Essential TShark Commands

| Purpose | Command |
|---|---|
| List interfaces | `tshark -D` |
| Read a capture | `tshark -r capture.pcapng` |
| Filter DNS packets | `tshark -r capture.pcapng -Y "dns"` |
| Capture 100 packets | `sudo tshark -i eth0 -c 100` |
| Capture TCP port 443 | `sudo tshark -i eth0 -f "tcp port 443"` |
| Show version | `tshark --version` |

## 24. Important Reminders

- Always identify the correct capture interface.
- Understand the difference between capture filters and display filters.
- Start with a broad overview, then narrow the investigation.
- Use Conversations and Endpoints to identify important hosts.
- Follow streams to understand application-level communication.
- Do not assume that unusual traffic is automatically malicious.
- Remember that encrypted traffic limits payload visibility.
- Keep original PCAP files unchanged during investigations.
- Protect captures and exported artifacts because they may contain sensitive data.
- Record packet numbers, timestamps, and supporting evidence.
- Perform captures and security testing only on networks and systems you own or are authorized to investigate.

---

## Further Learning Resources

- [Wireshark Official Website](https://www.wireshark.org/)
- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Wireshark Wiki](https://wiki.wireshark.org/)
- [TShark Manual](https://www.wireshark.org/docs/man-pages/tshark.html)
- [RFC 9293 — Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293)
- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 8446 — TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446)

## Personal Progress Checklist

Use this checklist as you practice.

- [ ] Understand the Wireshark interface
- [ ] Capture traffic on the correct interface
- [ ] Distinguish capture filters from display filters
- [ ] Filter traffic by IP address and port
- [ ] Analyze a TCP three-way handshake
- [ ] Recognize TCP retransmissions and resets
- [ ] Inspect DNS queries and responses
- [ ] Analyze HTTP requests and responses
- [ ] Understand what HTTPS encryption hides
- [ ] Inspect ARP, ICMP, and DHCP traffic
- [ ] Follow TCP and UDP streams
- [ ] Use Protocol Hierarchy, Conversations, and Endpoints
- [ ] Read PCAP files with TShark
- [ ] Build a timeline from packet evidence
- [ ] Write a short PCAP investigation report

**Core principle:** Don't just look at packets. Understand the conversation, identify the evidence, and explain what the traffic actually proves.
