# Computer Networks — Exam Preparation Notes
> **Exam Pattern:** Each topic carries **5 marks**. Answers should be precise, structured, and include diagrams/examples where applicable.

---

## Table of Contents
1. [Types of Networks (LAN, MAN, WAN)](#1-types-of-networks)
2. [Network Topologies](#2-network-topologies)
3. [OSI Model](#3-osi-model)
4. [TCP/IP Protocol Suite](#4-tcpip-protocol-suite)
5. [Error Detection and Correction](#5-error-detection-and-correction)
6. [Multiplexing](#6-multiplexing)

---

## 1. Types of Networks

Networks are classified based on their **geographical coverage**, **ownership**, and **data transfer rates**.

### 1.1 LAN — Local Area Network
- Covers a **small geographical area** (a room, building, or campus).
- **High speed:** typically 100 Mbps – 10 Gbps.
- **Privately owned** and managed.
- Uses technologies like **Ethernet (IEEE 802.3)** and **Wi-Fi (IEEE 802.11)**.

**Example:** All computers in a college lab connected via Ethernet switches form a LAN.

### 1.2 MAN — Metropolitan Area Network
- Spans a **city or large campus** (5 – 50 km range).
- Speed: typically **10 Mbps – 1 Gbps**.
- May be privately or publicly owned.
- Technologies: **DQDB (Distributed Queue Dual Bus)**, WiMAX.

**Example:** A cable TV network distributed across a city, or a university with multiple campuses in a city connected via fiber optic rings.

### 1.3 WAN — Wide Area Network
- Covers **countries or continents** — virtually unlimited geographical range.
- **Lower speed** relative to LAN/MAN, higher latency due to long-distance transmission.
- Uses **public/leased transmission lines** (telephone lines, satellite links, undersea cables).
- Technologies: **ATM, Frame Relay, MPLS, SDH**.

**Example:** The Internet is the largest WAN. A multinational company connecting its offices in India, USA, and UK over leased lines is a WAN.

### Comparison Table

| Feature       | LAN              | MAN               | WAN                    |
|---------------|------------------|-------------------|------------------------|
| Coverage      | Room/Building     | City              | Country/Continent       |
| Speed         | Very High         | High              | Moderate–Low            |
| Ownership     | Private           | Private/Public    | Public/Leased           |
| Error Rate    | Low               | Moderate          | Higher                  |
| Cost          | Low               | Moderate          | High                    |

---

## 2. Network Topologies

A **network topology** is the arrangement of nodes and links in a network. It can be **physical** (actual layout of cables) or **logical** (how data flows).

### 2.1 Mesh Topology
- Every device has a **dedicated point-to-point link to every other device**.
- For `n` nodes: **n(n-1)/2 physical links** and **n-1 I/O ports per device**.

```
A --- B
|\ /|
| X  |
|/ \|
C --- D
```

**Advantages:** Highly reliable; failure of one link doesn't affect others. Easy fault isolation.  
**Disadvantages:** Expensive due to large number of cables; difficult installation.  
**Used in:** Backbone/core networks, military communications.

### 2.2 Star Topology
- All devices connect to a **central hub or switch**.
- Communication goes through the central device.

```
    A
    |
B - HUB - C
    |
    D
```

**Advantages:** Easy to install/manage; failure of one node doesn't affect others; easy troubleshooting.  
**Disadvantages:** Central hub failure brings down the entire network.  
**Used in:** Most modern Ethernet LANs (using switches).

### 2.3 Tree (Hierarchical) Topology
- A **combination of star topologies** arranged in a hierarchy, like branches of a tree.
- Root node → secondary hubs → leaf nodes.

**Advantages:** Scalable; easy to manage sections independently.  
**Disadvantages:** If root or a backbone fails, entire subtree goes down.  
**Used in:** Corporate networks, cable TV networks.

### 2.4 Bus Topology
- All devices connect to a **single backbone cable** (the bus).
- Data travels in both directions along the bus; terminators at each end absorb signals.

```
A --- B --- C --- D --- E
|_________________________|
         Bus
```

**Advantages:** Easy and cheap to install; less cabling.  
**Disadvantages:** If the backbone cable fails, the entire network fails; difficult to troubleshoot; collisions possible.  
**Used in:** Early Ethernet (10Base2, 10Base5).

### 2.5 Ring Topology
- Each device connects to exactly **two neighbors**, forming a closed loop.
- Data travels in **one direction (unidirectional)** or both (bidirectional).

```
A → B → C → D → A
```

**Advantages:** Predictable performance; no collisions (token-based).  
**Disadvantages:** Failure of one node/cable can disrupt the entire ring (unless dual ring).  
**Used in:** Token Ring networks, SONET/SDH fiber rings.

### Quick Comparison

| Topology | Fault Tolerance | Cost     | Ease of Setup | Scalability |
|----------|----------------|----------|---------------|-------------|
| Mesh     | Excellent       | Very High | Difficult      | Poor        |
| Star     | Good            | Moderate | Easy           | Good        |
| Tree     | Moderate        | Moderate | Moderate       | Excellent   |
| Bus      | Poor            | Low       | Easy           | Poor        |
| Ring     | Poor–Moderate   | Low       | Moderate       | Moderate    |

---

## 3. OSI Model

The **Open Systems Interconnection (OSI)** model, developed by ISO, is a **conceptual framework** that standardizes network communication into **7 layers**. Each layer serves the layer above it and is served by the layer below it.

```
+---------------------------+
| 7. Application Layer      |  ← User-facing services (HTTP, FTP, SMTP)
+---------------------------+
| 6. Presentation Layer     |  ← Data formatting, encryption, compression
+---------------------------+
| 5. Session Layer          |  ← Session management, synchronization
+---------------------------+
| 4. Transport Layer        |  ← End-to-end delivery (TCP, UDP)
+---------------------------+
| 3. Network Layer          |  ← Routing, logical addressing (IP)
+---------------------------+
| 2. Data Link Layer        |  ← Framing, MAC addressing, error detection
+---------------------------+
| 1. Physical Layer         |  ← Bits over physical medium
+---------------------------+
```

**Mnemonic (top to bottom):** _All People Seem To Need Data Processing_  
**Mnemonic (bottom to top):** _Please Do Not Throw Sausage Pizza Away_

---

### Layer-by-Layer Functions

#### Layer 1 — Physical Layer
- Transmits **raw bits (0s and 1s)** over the physical medium.
- Deals with: voltage levels, cable types, connectors, data rates, bit synchronization.
- Defines: **simplex/half-duplex/full-duplex** transmission modes.
- **Examples:** Ethernet cables (RJ45), fiber optics, Wi-Fi radio signals, hubs, repeaters.

#### Layer 2 — Data Link Layer
- Provides **node-to-node (hop-to-hop)** reliable data transfer.
- **Framing:** Packages raw bits into frames with headers and trailers.
- **MAC Addressing:** Uses hardware addresses (e.g., `AA:BB:CC:DD:EE:FF`) for local delivery.
- **Error Detection:** Uses CRC, parity bits.
- **Flow Control:** Prevents fast senders from overwhelming slow receivers.
- Sub-layers: **LLC (Logical Link Control)** and **MAC (Media Access Control)**.
- **Examples:** Ethernet, Wi-Fi, PPP, switches, bridges.

#### Layer 3 — Network Layer
- Provides **host-to-host (source to destination)** logical addressing and routing.
- Assigns **IP addresses** and determines the best path for packet delivery.
- **Routing:** Uses routers and routing protocols (RIP, OSPF, BGP).
- **Fragmentation and Reassembly:** Splits large packets to fit MTU.
- **Examples:** IP (IPv4/IPv6), ICMP, routers.

#### Layer 4 — Transport Layer
- Provides **process-to-process (port-to-port)** communication.
- **Segmentation and Reassembly:** Breaks application data into segments.
- **Connection Control:** TCP (connection-oriented), UDP (connectionless).
- **Error Recovery and Flow Control** (in TCP).
- **Examples:** TCP (port 80 for HTTP, port 443 for HTTPS), UDP (port 53 for DNS).

#### Layer 5 — Session Layer
- **Establishes, manages, and terminates** communication sessions.
- Handles **synchronization** (checkpointing for long transfers so transmission can resume from a checkpoint on failure).
- **Dialog control:** Manages who can transmit at what time.
- **Examples:** NetBIOS, RPC (Remote Procedure Call).

#### Layer 6 — Presentation Layer
- Acts as a **translator** between the application and network formats.
- **Data Translation:** Converts EBCDIC to ASCII, etc.
- **Encryption/Decryption:** SSL/TLS operates here.
- **Compression:** Reduces data size before transmission.
- **Examples:** JPEG, MPEG, GIF formats; SSL/TLS.

#### Layer 7 — Application Layer
- Provides **network services directly to end-user applications**.
- Handles: file transfer, email, web browsing, remote login.
- **Examples:** HTTP, HTTPS, FTP, SMTP, POP3, DNS, Telnet.

---

### Data Encapsulation Across OSI Layers

| Layer       | PDU Name  | Header Added          |
|-------------|-----------|----------------------|
| Application | Data      | —                    |
| Transport   | Segment   | Port numbers, seq no  |
| Network     | Packet    | Source/Dest IP        |
| Data Link   | Frame     | MAC address, CRC      |
| Physical    | Bits      | —                    |

> **Key Point:** At the sender, each layer **adds a header** (encapsulation). At the receiver, each layer **removes its header** (decapsulation) — this is called **de-encapsulation**.

---

## 4. TCP/IP Protocol Suite

The **TCP/IP model** (also called the **Internet Model**) is the practical model used in the real Internet. It condenses OSI's 7 layers into **4 (or 5) layers**.

```
+-----------------------+     OSI Equivalent
| Application Layer     |  ←  Layers 5, 6, 7 (Session + Presentation + Application)
+-----------------------+
| Transport Layer       |  ←  Layer 4 (Transport)
+-----------------------+
| Internet Layer        |  ←  Layer 3 (Network)
+-----------------------+
| Network Access Layer  |  ←  Layers 1, 2 (Physical + Data Link)
+-----------------------+
```

### TCP/IP vs OSI

| Feature          | OSI Model              | TCP/IP Model             |
|------------------|------------------------|--------------------------|
| Layers           | 7                      | 4                        |
| Development      | Before protocols        | Based on existing protocols |
| Usage            | Reference model         | Implemented model (Internet) |
| Transport        | TCP, UDP               | TCP, UDP                  |
| Reliability      | Defined in each layer  | Defined in transport layer|

---

### Key Protocols at Each Layer

#### Application Layer
| Protocol | Port   | Purpose                        |
|----------|--------|-------------------------------|
| HTTP     | 80     | Web browsing                   |
| HTTPS    | 443    | Secure web browsing (with TLS) |
| FTP      | 20/21  | File transfer                  |
| SMTP     | 25     | Sending email                  |
| POP3     | 110    | Receiving email                |
| DNS      | 53     | Domain name → IP resolution    |
| Telnet   | 23     | Remote login                   |

#### Transport Layer

**TCP (Transmission Control Protocol)**
- **Connection-oriented:** Establishes a connection using a **3-way handshake** before data transfer.
  ```
  Client ──── SYN ────────► Server
  Client ◄─── SYN+ACK ──── Server
  Client ──── ACK ────────► Server
        [Connection Established]
  ```
- **Reliable:** Guarantees delivery using acknowledgments (ACKs) and retransmission.
- **Ordered delivery:** Sequence numbers ensure correct reassembly.
- **Flow control:** Sliding window mechanism.
- **Used for:** HTTP, FTP, Email.

**UDP (User Datagram Protocol)**
- **Connectionless:** No handshake; just sends datagrams.
- **Unreliable:** No guarantee of delivery or ordering.
- **Faster and lightweight:** Lower overhead than TCP.
- **Used for:** DNS, Video streaming, VoIP, Online gaming.

#### Internet Layer
- **IP (Internet Protocol):** Core protocol for addressing and routing packets. IPv4 uses 32-bit addresses; IPv6 uses 128-bit.
- **ICMP:** Error reporting (used by `ping` and `traceroute`).
- **ARP:** Resolves IP address → MAC address on local network.

#### Network Access Layer
- Handles physical transmission and MAC addressing.
- Protocols: Ethernet, Wi-Fi (802.11), PPP.

---

## 5. Error Detection and Correction

### 5.1 Types of Errors

**Single-Bit Error:** Only **one bit** in a data unit is changed (0→1 or 1→0).  
**Burst Error:** **Two or more consecutive bits** are corrupted. More common in real networks because noise affects a continuous period of time.

```
Sent:     1 0 1 1 0 0 1 0
Received: 1 0 0 0 1 0 1 0
                ↑↑↑
          Burst error (3 bits corrupted)
```

---

### 5.2 Error Detection — Redundancy Checks

The key idea: add **extra (redundant) bits** to the data so the receiver can detect or correct errors.

#### VRC — Vertical Redundancy Check (Parity Check)
- Adds **1 parity bit** to each character (7-bit ASCII → 8-bit with parity).
- **Even Parity:** Total number of 1s in the character (including parity bit) must be **even**.
- **Odd Parity:** Total number of 1s must be **odd**.

**Example (Even Parity):**
```
Data: 1 0 1 1 0 0 1   → Number of 1s = 4 (even) → Parity bit = 0
Data: 1 0 1 1 0 0 0   → Number of 1s = 3 (odd)  → Parity bit = 1
```

**Limitation:** VRC cannot detect errors if an **even number of bits** are flipped (errors cancel out).

---

#### LRC — Longitudinal Redundancy Check
- Applies parity check **across all characters in a block** (column-wise).
- A **redundant row** of bits (LRC byte) is appended to the entire block.

**Example:**
```
Characters:   ASCII bits
C1:           1 0 1 0 1 0 1
C2:           0 1 1 0 0 1 0
C3:           1 1 0 1 0 0 1
              ─────────────
LRC (even):   0 0 0 1 1 1 0
```
> Each column has even parity. The LRC row itself acts as a redundancy character.

**Advantage over VRC:** Can detect burst errors better.  
**Limitation:** Cannot detect errors where pairs of bits in the same position across rows are both flipped.

---

#### CRC — Cyclic Redundancy Check
- The most powerful and widely used error detection method.
- Based on **polynomial division** (binary division without carries — XOR operations).
- Sender and receiver agree on a **generator polynomial (divisor)**.

**Steps:**
1. Append `(n-1)` zeros to the original message (where `n` = number of bits in generator).
2. Divide the augmented message by the generator using **modulo-2 division (XOR)**.
3. The remainder is the **CRC (Frame Check Sequence — FCS)**.
4. Replace the appended zeros with the CRC and transmit.
5. Receiver divides received data by the same generator. If **remainder = 0**, no error. If non-zero, error detected.

**Example:**
```
Message (M):      1 1 0 1 0
Generator (G):    1 0 1 1   (degree 3, so append 3 zeros)
Augmented M:      1 1 0 1 0 0 0 0

XOR Division:
1 1 0 1 0 0 0 0
1 0 1 1
─────────────
  1 1 0 0 0 0 0
  1 0 1 1
  ───────────
    0 1 1 0 0 0
    (since leading bit is 0, bring down next bit)
      1 1 0 0 0
      1 0 1 1
      ─────────
        1 0 1 0
        1 0 1 1
        ───────
          0 0 1 (remainder/FCS = 001)

Transmitted frame: 1 1 0 1 0 0 0 1
```

**CRC Standards:**
- **CRC-16:** Used in HDLC, BISYNC — 16-bit remainder.
- **CRC-32:** Used in Ethernet, ZIP — 32-bit remainder (very robust).
- **CRC-CCITT:** Used in SDLC, X.25.

---

### 5.3 Error Correction — Hamming Code

Unlike error detection, **error correction** finds AND fixes the error — crucial when retransmission is impractical (e.g., deep space communication).

**Hamming Code** adds **redundant bits at positions that are powers of 2** (positions 1, 2, 4, 8, 16, …).

**Rules:**
- Position `p` (power of 2) is a **parity bit**.
- Each parity bit covers specific data bit positions (those that have the bit `p` set in their binary representation).

**Example: Encode data `1011`**
```
Positions:       1  2  3  4  5  6  7
Bits:            P1 P2 D1 P4 D2 D3 D4
Data bits:                  1     0  1  1
```

- P1 (pos 1) covers positions 1, 3, 5, 7 → covers D1=1, D2=0, D4=1 → P1 makes even parity → P1=0
- P2 (pos 2) covers positions 2, 3, 6, 7 → covers D1=1, D3=1, D4=1 → P2=1
- P4 (pos 4) covers positions 4, 5, 6, 7 → covers D2=0, D3=1, D4=1 → P4=0

```
Transmitted: 0 1 1 0 0 1 1
Positions:   1 2 3 4 5 6 7
```

**At receiver:** If error occurs at position 5 (bit flipped):
- Received: `0 1 1 0 1 1 1`
- Recalculate parity groups; identify incorrect positions.
- Sum of erroneous parity positions gives the **error position** → flip that bit to correct.

---

## 6. Multiplexing

**Multiplexing** is the technique of combining **multiple signals** over a **single shared transmission medium** to maximize channel utilization. A device called a **multiplexer (MUX)** combines signals at the sender, and a **demultiplexer (DEMUX)** separates them at the receiver.

```
Line 1 ─┐                    ┌─ Line 1
Line 2 ─┤ MUX ──[channel]── DEMUX ├─ Line 2
Line 3 ─┘                    └─ Line 3
```

---

### 6.1 FDM — Frequency Division Multiplexing

- Used with **analog signals**.
- The total available **bandwidth** of the medium is divided into **sub-bands (channels)**, each assigned to a different signal.
- Each signal is **modulated onto a different carrier frequency**, so all travel simultaneously without interfering.
- **Guard bands** (unused frequency gaps) are placed between channels to prevent interference.

```
Frequency →
|--Ch1--|--Guard--|--Ch2--|--Guard--|--Ch3--|
0 Hz                                       Max Hz
```

**Example:**
- **FM Radio:** Each station is assigned a different frequency band (e.g., 91.1 MHz, 93.5 MHz, 98.3 MHz) on the same broadcast spectrum.
- **Cable TV:** Each TV channel is assigned a frequency band on the coaxial cable.
- **ADSL (phone lines):** Voice and data use different frequency bands on the same wire.

**Advantages:** Simple; no need for synchronization between channels.  
**Disadvantages:** Bandwidth waste due to guard bands; susceptible to crosstalk.

---

### 6.2 WDM — Wavelength Division Multiplexing

- Conceptually **identical to FDM but used with optical fiber (light signals)**.
- Different signals are carried on **different wavelengths (colors) of light** simultaneously through the same fiber.

**Variants:**
- **Coarse WDM (CWDM):** Fewer channels, wider wavelength spacing, lower cost.
- **Dense WDM (DWDM):** Many channels (40, 80, 160+) with very narrow spacing — used in long-distance backbone networks.

**Example:** A single fiber carrying 8 different wavelengths of light, each carrying 10 Gbps → **80 Gbps total capacity** on one fiber.

**Advantages:** Enormous bandwidth capacity; transparent to data rate and format.  
**Disadvantages:** Expensive equipment (lasers, filters); complex to manage.

---

### 6.3 TDM — Time Division Multiplexing

- Used with **digital signals**.
- The total time on the channel is divided into **time slots**, and each sender gets a dedicated slot in a repeating cycle (**frame**).

#### Synchronous TDM
- Each input gets a **fixed time slot**, regardless of whether it has data to send or not.
- If a source has no data, its slot is **wasted (empty)**.

```
Frame:  | A | B | C | D | A | B | C | D | ...
        |<-- Frame 1 -->|<-- Frame 2 -->|
```

**Example:**
- **T1 line (DS1):** Combines 24 channels × 64 Kbps each = 1.544 Mbps.
- **E1 line:** 32 channels × 64 Kbps = 2.048 Mbps.
- Traditional telephony circuits use synchronous TDM.

#### Statistical (Asynchronous) TDM — STDM
- Slots are allocated **dynamically** — only to sources that actually have data.
- More efficient than synchronous TDM; no wasted slots.
- Requires overhead (addresses in each slot) to identify which source the slot belongs to.

**Comparison of TDM types:**

| Feature               | Synchronous TDM       | Statistical TDM       |
|-----------------------|-----------------------|-----------------------|
| Slot allocation       | Fixed                 | On-demand             |
| Efficiency            | Low (idle waste)      | High                  |
| Overhead              | Low                   | Higher (needs address) |
| Complexity            | Simple                | More complex          |
| Used in               | PSTN, T1/E1 lines     | Packet-switched nets   |

---

### 6.4 Multiplexing Comparison Summary

| Type | Signal   | Divided By    | Guard Separation | Key Use Case           |
|------|----------|---------------|------------------|------------------------|
| FDM  | Analog   | Frequency     | Guard bands       | Radio, Cable TV, ADSL   |
| WDM  | Optical  | Wavelength    | Wavelength spacing| Fiber optic backbones   |
| TDM  | Digital  | Time          | Guard times       | Telephony, T1/E1 lines  |

---

## Quick Revision Cheatsheet

| Topic              | Key Terms to Remember                                          |
|--------------------|----------------------------------------------------------------|
| Types of Networks  | LAN (building), MAN (city), WAN (global); speed ↓ as range ↑  |
| Topologies         | Mesh (n(n-1)/2 links), Star (central hub), Bus (backbone), Ring (loop), Tree (hierarchy) |
| OSI Model          | 7 layers; each adds header (encapsulation); PDU names differ per layer |
| TCP/IP             | 4 layers; TCP = reliable, 3-way handshake; UDP = fast, no guarantee |
| Error Detection    | VRC (1 parity/char), LRC (block parity), CRC (polynomial division, most powerful) |
| Error Correction   | Hamming code — redundant bits at power-of-2 positions; finds AND fixes error |
| Multiplexing       | FDM (frequency, analog), WDM (wavelength, optical), TDM (time, digital) |

---

> 📝 **Exam Tip:** For 5-mark questions, write a **definition (1 mark) + explanation (2 marks) + diagram or example (1 mark) + advantages/disadvantages or comparison (1 mark)**. This structure consistently scores full marks.
