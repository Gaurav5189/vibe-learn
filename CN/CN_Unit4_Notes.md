# Computer Networks — Unit IV Study Notes (MCA)
> **Topics:** ISDN · X.25 · Frame Relay · Networking Devices · Routing · TCP/IP Suite · Application Layer Protocols

---

## 1. ISDN (Integrated Services Digital Network)

### 1.1 What is ISDN?
ISDN is a **digital telephone network** that carries voice, video, and data over **ordinary telephone lines** — all digitally. It replaced analog PSTN by providing end-to-end digital connectivity.

> 📝 Key idea: "Integrated" = one network for voice + data + video. "Digital" = no modems needed.

---

### 1.2 History
- Concept proposed by ITU-T in **1972**.
- First standards finalized: **1984** (Red Book), refined **1988** (Blue Book).
- Designed to digitize the entire telephone network.

---

### 1.3 ISDN Services

| Service Type | Description |
|--------------|-------------|
| **Bearer Services** | Transport of raw data between user endpoints (layer 1–3). E.g., 64 kbps digital channel. |
| **Teleservices** | End-to-end services including terminal functions. E.g., telephony, fax, videotex. |
| **Supplementary Services** | Add-ons to bearer/teleservices. E.g., call waiting, caller ID, call forwarding. |

---

### 1.4 ISDN Channels

| Channel | Bit Rate | Purpose |
|---------|----------|---------|
| **B channel** | 64 kbps | Bearer — carries voice/data |
| **D channel** | 16 or 64 kbps | Delta — carries signaling/control info |
| **H channel** | 384 kbps – 1.92 Mbps | High-speed data (video conferencing) |

---

### 1.5 Subscriber Access to ISDN

Two standard interfaces:

#### BRI — Basic Rate Interface
- **2B + D** = 2 × 64 kbps (B) + 1 × 16 kbps (D)
- Total = **144 kbps**
- For homes and small offices.

#### PRI — Primary Rate Interface
- **North America/Japan:** 23B + D = 23 × 64 kbps + 1 × 64 kbps = **1.544 Mbps (T1)**
- **Europe:** 30B + D = 30 × 64 kbps + 1 × 64 kbps = **2.048 Mbps (E1)**
- For large organizations, PBX systems.

> 📝 **Memory trick:** BRI = Basic (home use, 2B+D). PRI = Primary (business use, 23B+D or 30B+D).

---

### 1.6 ISDN Layers

| Layer | Protocol | Function |
|-------|----------|----------|
| **Layer 1** (Physical) | I.430 (BRI), I.431 (PRI) | Bit transmission over the local loop |
| **Layer 2** (Data Link) | LAPD (Link Access Protocol — D channel) | Framing, error/flow control on D channel |
| **Layer 3** (Network) | Q.931 | Call setup, teardown, signaling |

---

### 1.7 Broadband ISDN (B-ISDN)
- Successor to ISDN — supports very high speeds (155 Mbps and above).
- Uses **ATM (Asynchronous Transfer Mode)** as the transport technology.
- Transmitted over **fiber optic** cables.
- Supports integrated broadband services: HDTV, video-on-demand, high-speed internet.
- ITU-T standard: **I.121**.

> 📝 B-ISDN never fully deployed commercially — superseded by broadband internet (DSL, cable, fiber).

---

## 2. X.25

### 2.1 What is X.25?
X.25 is an ITU-T standard for **packet-switched WAN communication** over **unreliable analog telephone networks**. Defined in 1976, it was the dominant WAN protocol before Frame Relay.

> 📝 X.25 was designed for **noisy analog lines** → heavy error checking at every node (slow but reliable).

---

### 2.2 X.25 Layers

X.25 maps to the **bottom 3 layers** of the OSI model:

| X.25 Layer | OSI Layer | Protocol | Function |
|-----------|-----------|----------|----------|
| **Physical** | Layer 1 | X.21 (or X.21 bis) | Bit transmission over physical circuit |
| **Data Link** | Layer 2 | LAPB (Link Access Protocol — Balanced) | Error/flow control between DTE and DCE |
| **Packet** | Layer 3 | PLP (Packet Layer Protocol) | Virtual circuit setup, packet routing |

**Key terms:**
- **DTE (Data Terminal Equipment):** Customer's device (computer, terminal).
- **DCE (Data Circuit-terminating Equipment):** ISP's switch/modem at the edge.
- **PSN (Packet Switching Network):** The carrier's network of switches.

---

### 2.3 Virtual Circuits in X.25

| Type | Description |
|------|-------------|
| **SVC (Switched Virtual Circuit)** | Temporary — set up on demand, torn down after use (like a phone call). |
| **PVC (Permanent Virtual Circuit)** | Always active — no setup/teardown needed (like a leased line). |

---

### 2.4 Protocols Related to X.25

| Protocol | Purpose |
|---------|---------|
| **LAPB** | Data link layer of X.25 — reliable link between DTE and local DCE |
| **X.21** | Physical interface standard |
| **PAD (X.3/X.28/X.29)** | Packet Assembler/Disassembler — connects non-X.25 devices to X.25 network |

> 📝 X.25 does **error checking at every node** (hop-by-hop) → very reliable but **slow**.

---

## 3. Frame Relay

### 3.1 Introduction
Frame Relay is a **streamlined, high-speed packet switching** WAN protocol. It evolved from X.25 but **removes the heavy error-checking** (assumes the network is reliable — modern digital lines).

**Key differences from X.25:**

| Feature | X.25 | Frame Relay |
|---------|------|-------------|
| Error correction | At every node | Only at endpoints |
| Speed | 64 kbps typical | 1.544 Mbps (T1) typical |
| Overhead | High | Low |
| Designed for | Noisy analog lines | Clean digital lines |
| Layer | 1–3 | 1–2 only |

---

### 3.2 Frame Relay Operation

- Uses **Virtual Circuits** (like X.25) — both SVC and PVC (PVCs are most common).
- Each virtual circuit identified by a **DLCI (Data Link Connection Identifier)** — a local address.
- Frame Relay is a **connection-oriented** service at layer 2.

**Frame Relay Frame:**
```
| Flag | Header (DLCI + control bits) | Data | FCS | Flag |
```

**Header bits:**
- **DLCI (10 bits):** Identifies the virtual circuit.
- **FECN:** Forward Explicit Congestion Notification — tells receiver congestion is ahead.
- **BECN:** Backward Explicit Congestion Notification — tells sender to slow down.
- **DE (Discard Eligibility):** Frame can be dropped if network is congested.

---

### 3.3 Frame Relay Layers

Frame Relay only defines **two layers:**
- **Physical Layer:** Any standard (T1, E1, etc.)
- **Data Link Layer:** Core + Control sublayers.

---

### 3.4 Congestion Control in Frame Relay

Frame Relay uses **notification-based congestion control** (not prevention):

1. **FECN:** Set by network on frames going toward the destination — "congestion ahead."
2. **BECN:** Set on frames going back toward the source — "slow down!"
3. **DE bit:** Frames marked DE=1 are dropped first during congestion.

**CIR (Committed Information Rate):** Guaranteed bandwidth the provider promises. Traffic above CIR gets DE=1 (best-effort).

---

### 3.5 Leaky Bucket Algorithm

Used for **traffic shaping** — controls the rate of data entering the network.

**How it works:**
- Imagine a bucket with a hole at the bottom.
- Data (packets) pour in from the top at **any rate** (bursty).
- Water drips out at a **fixed constant rate**.
- If bucket **overflows** → packets are **discarded**.

```
     Bursty input
         ↓↓↓
    ┌─────────┐
    │  BUCKET │ ← capacity = buffer size
    └────┬────┘
         │ constant rate output
         ↓
     Smooth output
```

**Effect:** Converts **bursty traffic** into **smooth, constant-rate** output.

**Token Bucket (variant):**
- Tokens are added at a constant rate to a bucket.
- Each packet needs a token to be sent.
- Allows **controlled bursting** (unlike leaky bucket which never bursts).

> 📝 **Leaky Bucket = smoothing.** **Token Bucket = smoothing + allows bursts up to bucket size.**

---

### 3.6 Traffic Control
- **Traffic Policing:** Drop or mark excess traffic (enforces limits strictly).
- **Traffic Shaping:** Buffer and delay excess traffic (smoother output).
- Frame Relay uses CIR, Bc (Committed Burst), and Be (Excess Burst) to define traffic contracts.

---

## 4. Networking and Internetworking Devices

### 4.1 Repeater (Layer 1)
- Regenerates and amplifies **weak signals**.
- Operates at the **Physical Layer**.
- Extends cable distance — no intelligence, just signal boosting.
- Cannot filter traffic or connect different network types.

---

### 4.2 Bridge (Layer 2)
- Connects two **similar LAN segments**.
- Operates at the **Data Link Layer** (uses MAC addresses).
- Maintains a **MAC address table** (forwarding table).
- Can **filter traffic** — only forwards frames to the segment where the destination MAC is located.
- Reduces collisions by dividing a network into segments.

> 📝 A **Switch** is essentially a multi-port bridge.

---

### 4.3 Router (Layer 3)
- Connects **different networks** (LANs, WANs).
- Operates at the **Network Layer** (uses IP addresses).
- Makes intelligent **routing decisions** using routing tables.
- Can connect networks with **different technologies** (Ethernet ↔ Frame Relay).
- Provides logical addressing and path determination.

---

### 4.4 Gateway (Layer 4–7)
- Connects networks with **completely different protocols** (e.g., TCP/IP to IBM SNA).
- Operates at **Transport Layer and above**.
- Performs **protocol translation**.
- Slowest device (most processing overhead).
- Example: Email gateway (converts between different email protocols).

---

### 4.5 Device Comparison

| Device | OSI Layer | Addresses Used | Function |
|--------|-----------|----------------|----------|
| Repeater | 1 – Physical | None | Signal regeneration |
| Bridge | 2 – Data Link | MAC address | Segment filtering |
| Router | 3 – Network | IP address | Path selection |
| Gateway | 4–7 | All | Protocol translation |

---

## 5. Routing Algorithms

### 5.1 Distance Vector Routing
- Each router maintains a table of **(destination, distance, next-hop)**.
- Routers **share their table with neighbors** periodically.
- Uses **Bellman-Ford algorithm**.
- Example: **RIP (Routing Information Protocol)** — metric = hop count, max 15 hops.

**Problem:** **Count-to-Infinity** — slow convergence when a link fails, routers keep incrementing distance.

---

### 5.2 Link State Routing
- Each router knows the **complete network topology**.
- Routers **flood** their link state info to all routers.
- Each router independently runs **Dijkstra's algorithm** to find shortest path.
- Example: **OSPF (Open Shortest Path First)**.

**Advantages:** Fast convergence, no count-to-infinity.
**Disadvantages:** High memory and processing overhead.

---

### 5.3 Comparison

| Feature | Distance Vector | Link State |
|---------|----------------|------------|
| Knowledge | Neighbors only | Entire topology |
| Algorithm | Bellman-Ford | Dijkstra |
| Convergence | Slow | Fast |
| Overhead | Low | High |
| Example | RIP | OSPF |
| Problem | Count-to-infinity | Memory/CPU intensive |

---

## 6. TCP/IP Protocol Suite

### 6.1 Overview

TCP/IP is the **protocol suite of the Internet** — a set of protocols organized in 4 layers:

```
┌─────────────────────────────┐
│     Application Layer       │  HTTP, FTP, SMTP, DNS, Telnet...
├─────────────────────────────┤
│      Transport Layer        │  TCP, UDP
├─────────────────────────────┤
│      Internet (Network)     │  IP, ICMP, ARP, RARP
├─────────────────────────────┤
│    Network Access Layer     │  Ethernet, Wi-Fi, Frame Relay...
└─────────────────────────────┘
```

---

### 6.2 Network Layer (Internet Layer)

#### IP (Internet Protocol)
- **Connectionless, unreliable** — best-effort delivery, no guarantee.
- Provides **logical addressing** (IP addresses) and **fragmentation**.
- Two versions: IPv4 (32-bit addresses) and IPv6 (128-bit).

**IPv4 Datagram Header (key fields):**

| Field | Size | Purpose |
|-------|------|---------|
| Version | 4 bits | IPv4 = 4 |
| Header Length | 4 bits | Length of header in 32-bit words |
| TTL | 8 bits | Prevents infinite looping — decremented at each hop |
| Protocol | 8 bits | Upper layer (6=TCP, 17=UDP, 1=ICMP) |
| Source IP | 32 bits | Sender's IP address |
| Destination IP | 32 bits | Receiver's IP address |

#### ICMP (Internet Control Message Protocol)
- Used for **error reporting and diagnostics**.
- Examples: Destination unreachable, Time exceeded, Echo request/reply.
- Used by **ping** (Echo request/reply) and **traceroute** (Time exceeded).

#### ARP (Address Resolution Protocol)
- Maps **IP address → MAC address**.
- Broadcasts "Who has IP x.x.x.x?" — owner replies with its MAC.

#### RARP (Reverse ARP)
- Maps **MAC address → IP address** (used by diskless workstations at boot).

---

### 6.3 IP Addressing

**IPv4 Address:** 32-bit, written in **dotted decimal** (e.g., 192.168.1.10).

**Address Classes:**

| Class | Range | Default Subnet Mask | Use |
|-------|-------|---------------------|-----|
| A | 1.0.0.0 – 126.255.255.255 | 255.0.0.0 /8 | Large networks |
| B | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 /16 | Medium networks |
| C | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 /24 | Small networks |
| D | 224.0.0.0 – 239.255.255.255 | — | Multicast |
| E | 240.0.0.0 – 255.255.255.255 | — | Reserved/Experimental |

**Special addresses:**
- `127.0.0.1` = Loopback (localhost).
- `255.255.255.255` = Limited broadcast.

---

### 6.4 Subnetting

**Why subnet?** To divide a large network into smaller, manageable sub-networks and reduce broadcast traffic.

**Subnet Mask:** Determines which part is Network ID and which is Host ID.

**Example:**
```
IP Address:   192.168.1.0    Class C, /24
Subnet Mask:  255.255.255.0
Host bits: 8 → 2⁸ - 2 = 254 usable hosts
(subtract 2: network address + broadcast address)
```

**Subnetting Example — divide 192.168.1.0/24 into 4 subnets:**
```
Borrow 2 bits → subnet mask becomes /26 (255.255.255.192)
Each subnet: 2⁶ - 2 = 62 usable hosts

Subnet 1: 192.168.1.0  – 192.168.1.63   (Network: .0,  Broadcast: .63)
Subnet 2: 192.168.1.64 – 192.168.1.127  (Network: .64, Broadcast: .127)
Subnet 3: 192.168.1.128– 192.168.1.191  (Network: .128,Broadcast: .191)
Subnet 4: 192.168.1.192– 192.168.1.255  (Network: .192,Broadcast: .255)
```

> 📝 **Formula:** N subnets → borrow log₂(N) bits. Hosts per subnet = 2^(remaining host bits) − 2.

---

### 6.5 Transport Layer

#### TCP (Transmission Control Protocol)
- **Connection-oriented, reliable, ordered** delivery.
- Uses **3-way handshake** to establish connection.
- Features: Flow control (sliding window), error control, congestion control.

**3-Way Handshake:**
```
Client → Server:  SYN (seq=x)
Server → Client:  SYN-ACK (seq=y, ack=x+1)
Client → Server:  ACK (ack=y+1)
[Connection Established]
```

**4-Way Termination:**
```
Client: FIN → Server: ACK → Server: FIN → Client: ACK
```

#### UDP (User Datagram Protocol)
- **Connectionless, unreliable** — no handshake, no guarantee.
- Low overhead — fast.
- Used for: DNS, DHCP, video streaming, gaming, VoIP.

**TCP vs UDP:**

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable | Unreliable |
| Order | Ordered | No ordering |
| Speed | Slower | Faster |
| Header size | 20 bytes | 8 bytes |
| Use case | HTTP, FTP, Email | DNS, Video, VoIP |

---

## 7. Application Layer Protocols

### 7.1 Client-Server Model
- **Client** initiates requests; **Server** responds.
- Server runs continuously, listening on a **well-known port**.
- Multiple clients can connect to one server.

---

### 7.2 BOOTP (Bootstrap Protocol)
- Allows a **diskless workstation** to discover its IP address at boot time.
- Client broadcasts → Server responds with IP, subnet mask, default gateway, server IP.
- Static — IP–MAC mappings must be configured manually on server.
- **Port:** UDP 67 (server), UDP 68 (client).

---

### 7.3 DHCP (Dynamic Host Configuration Protocol)
- **Automatic IP address assignment** — evolved from BOOTP.
- Server maintains a **pool** of IP addresses and leases them dynamically.

**DHCP Process (DORA):**
```
1. DISCOVER  — Client broadcasts "I need an IP"
2. OFFER     — Server offers an IP address
3. REQUEST   — Client requests the offered IP
4. ACK       — Server confirms the lease
```

- **Lease time:** IP is assigned for a limited period; client must renew.
- **Port:** UDP 67/68.

> 📝 DHCP vs BOOTP: DHCP is dynamic (pool of IPs, leases). BOOTP is static (fixed mapping).

---

### 7.4 DNS (Domain Name System)
- Translates **human-readable domain names → IP addresses**.
- Example: `www.google.com → 142.250.195.36`
- **Distributed hierarchical database** — not one central server.
- **Port:** UDP/TCP 53.

**DNS Hierarchy:**
```
Root (.)
  └── Top-Level Domains (TLDs): .com, .org, .in, .edu
        └── Second-Level Domains: google.com, amazon.com
              └── Subdomains: www.google.com, mail.google.com
```

**DNS Resolution (Recursive):**
1. Browser asks **Local DNS Resolver**.
2. Local asks **Root Server** → gets TLD server address.
3. Local asks **TLD Server** → gets authoritative server address.
4. Local asks **Authoritative Server** → gets final IP.
5. IP returned to browser and cached.

**DNS Record Types:**

| Record | Purpose |
|--------|---------|
| A | Domain → IPv4 address |
| AAAA | Domain → IPv6 address |
| MX | Mail exchange server |
| CNAME | Alias to another domain |
| NS | Name server for domain |
| PTR | Reverse lookup (IP → name) |

---

### 7.5 Telnet
- Provides **remote login** to another computer over the network.
- **Port: TCP 23**.
- Transmits data (including passwords) in **plaintext** — insecure.
- Replaced by **SSH (Secure Shell)** which encrypts traffic.

---

### 7.6 FTP (File Transfer Protocol)
- Transfers files between client and server.
- **Port: TCP 21** (control), **TCP 20** (data).
- Uses **two separate connections:**
  - **Control connection (port 21):** Commands and responses (persists throughout session).
  - **Data connection (port 20):** Actual file transfer (opened for each transfer).
- Supports authentication (username/password) but sends in **plaintext**.
- **SFTP/FTPS** = secure alternatives.

**FTP Modes:**
- **Active mode:** Server initiates data connection back to client.
- **Passive mode:** Client initiates both connections (firewall-friendly).

---

### 7.7 SMTP (Simple Mail Transfer Protocol)
- Used to **send emails** from client to server, and between mail servers.
- **Port: TCP 25** (server-to-server), TCP 587 (client-to-server submission).
- Push protocol — only for sending.
- **POP3 (Port 110) / IMAP (Port 143)** are used to **retrieve** emails.

**Email Flow:**
```
Sender → [SMTP] → Sender's Mail Server → [SMTP] → Receiver's Mail Server
                                                          ↓
                                             Receiver ← [POP3/IMAP]
```

---

### 7.8 SNMP (Simple Network Management Protocol)
- Used for **monitoring and managing network devices** (routers, switches, printers).
- **Port: UDP 161** (agent), UDP 162 (trap).
- **Components:**
  - **Manager:** Central system that monitors (NMS — Network Management System).
  - **Agent:** Software on each managed device.
  - **MIB (Management Information Base):** Database of manageable objects on device.

**SNMP Operations:**

| Operation | Description |
|-----------|-------------|
| GET | Manager requests a value from agent |
| SET | Manager changes a value on agent |
| TRAP | Agent sends unsolicited alert to manager |
| GETNEXT | Get next variable in MIB |

---

### 7.9 HTTP (HyperText Transfer Protocol)
- Foundation of the **World Wide Web** — transfers web pages (HTML, images, etc.).
- **Port: TCP 80** (HTTP), **TCP 443** (HTTPS — secure).
- **Stateless** — each request is independent (no memory of past requests).
- Cookies used to maintain state.

**HTTP Methods:**

| Method | Purpose |
|--------|---------|
| GET | Retrieve a resource |
| POST | Submit data to server |
| PUT | Update a resource |
| DELETE | Delete a resource |
| HEAD | Get headers only (no body) |

**HTTP Response Codes:**

| Code | Meaning |
|------|---------|
| 200 | OK — success |
| 301 | Moved Permanently |
| 400 | Bad Request |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |

**HTTP vs HTTPS:** HTTPS uses **SSL/TLS** for encryption — protects data in transit.

---

### 7.10 WWW (World Wide Web)
- A system of interlinked **hypertext documents** accessed via the Internet using HTTP.
- **Key components:**
  - **URL (Uniform Resource Locator):** Address of a resource. `https://www.example.com/page.html`
  - **HTML (HyperText Markup Language):** Language for creating web pages.
  - **Web Browser:** Client that renders HTML and communicates via HTTP.
  - **Web Server:** Hosts and serves web content (Apache, Nginx).
- Invented by **Tim Berners-Lee** at CERN in 1989.

> 📝 The Internet ≠ WWW. The Internet is the network infrastructure; WWW is one service running on it (alongside email, FTP, etc.).

---

## 🔁 Quick Revision Summary

| Topic | Key Point |
|-------|-----------|
| ISDN BRI | 2B + D = 144 kbps (home use) |
| ISDN PRI | 23B+D = 1.544 Mbps (US) or 30B+D = 2.048 Mbps (EU) |
| ISDN Layers | L1: I.430/431, L2: LAPD, L3: Q.931 |
| B-ISDN | High-speed, uses ATM, fiber |
| X.25 | Error check every node, maps OSI L1–L3, uses LAPB |
| Frame Relay | Fast, error check endpoints only, uses DLCI, FECN/BECN |
| Leaky Bucket | Smooth bursty traffic to constant rate |
| Repeater | Layer 1 — signal regeneration |
| Bridge | Layer 2 — MAC-based filtering |
| Router | Layer 3 — IP-based routing |
| Gateway | Layer 4–7 — protocol translation |
| Distance Vector | Bellman-Ford, shares tables with neighbors, RIP |
| Link State | Dijkstra, full topology, OSPF |
| IP | Connectionless, best-effort, 32-bit address |
| ARP | IP → MAC mapping |
| TCP 3-way | SYN → SYN-ACK → ACK |
| DHCP | DORA process, dynamic IP, UDP 67/68 |
| DNS | Name → IP, port 53, hierarchical |
| FTP | Port 20 (data) + 21 (control), two connections |
| SMTP | Send email, port 25 |
| HTTP | Web pages, stateless, port 80 |
| SNMP | Network management, GET/SET/TRAP, UDP 161 |

---

## 📌 Well-Known Port Numbers (Must Know!)

| Protocol | Port |
|----------|------|
| FTP Data | 20 |
| FTP Control | 21 |
| Telnet | 23 |
| SMTP | 25 |
| DNS | 53 |
| HTTP | 80 |
| POP3 | 110 |
| SNMP | 161 |
| IMAP | 143 |
| HTTPS | 443 |
| DHCP Server | 67 |
| DHCP Client | 68 |

---

## 📌 Likely Exam Questions

1. Explain ISDN channels (B, D, H) and the difference between BRI and PRI.
2. What is X.25? Explain its three layers.
3. Compare X.25 and Frame Relay. What is a DLCI?
4. Explain the Leaky Bucket algorithm with a diagram. How is it different from Token Bucket?
5. Differentiate Repeater, Bridge, Router, and Gateway with OSI layers.
6. Compare Distance Vector and Link State routing algorithms.
7. Explain the TCP 3-way handshake.
8. Explain the DHCP DORA process.
9. How does DNS work? Explain with the hierarchy.
10. Compare TCP and UDP with use cases.
11. What is FTP? Explain its two connections (control and data).
12. What is SNMP? Explain its components (Manager, Agent, MIB) and operations.
13. What is subnetting? Divide a Class C network into 4 subnets.
14. What is the difference between HTTP and HTTPS?

---
*Notes prepared for MCA — Computer Networks (Unit IV)*
