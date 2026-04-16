# Computer Networks — Unit III Study Notes (MCA)
> **Topics:** Data Link Control · Data Link Protocols · Local Area Networks · Switching

---

## 1. Data Link Control (DLC)

Data Link Control handles **reliable communication between two directly connected nodes**. It governs three key things:

### 1.1 Line Discipline
Controls **who transmits when** — prevents two stations from sending simultaneously.

| Method | Description |
|--------|-------------|
| **ENQ/ACK** | One primary, one secondary. Primary sends ENQ (enquiry), secondary replies ACK if ready. |
| **Poll/Select** | Used in multipoint links. Primary *polls* (asks) secondaries if they want to send; *selects* them to receive. |

> 📝 **Exam Tip:** ENQ/ACK = point-to-point. Poll/Select = multipoint (one primary, many secondaries).

---

### 1.2 Flow Control
Ensures the **sender doesn't overwhelm the receiver**.

#### Stop-and-Wait
- Send one frame → wait for ACK → send next.
- Simple but **inefficient** (wastes bandwidth during wait time).

```
Sender:  [Frame 0] ──────────────→
Receiver:           ←──────── [ACK 0]
Sender:  [Frame 1] ──────────────→
```

#### Sliding Window
- Sender can send **W frames** before needing an ACK.
- Much more efficient — keeps the channel busy.
- **Window Size (W)** = number of unacknowledged frames allowed in transit.

> 📝 **Example:** If propagation delay is high (satellite link), a large window size is needed.

---

### 1.3 Error Control
Handles lost or corrupted frames using **ARQ (Automatic Repeat reQuest)**.

| ARQ Type | How it works | Efficiency |
|----------|--------------|------------|
| **Stop-and-Wait ARQ** | Retransmit only if no ACK received before timeout | Low |
| **Go-Back-N ARQ** | On error, retransmit the errored frame **and all after it** | Medium |
| **Selective Repeat ARQ** | Retransmit **only** the errored frame | High |

> 📝 **Exam Q:** In Go-Back-N with window size W=4, if frame 2 errors, frames 2,3,4,5 are all resent. In Selective Repeat, only frame 2 is resent.

---

## 2. Data Link Protocols

### 2.1 Asynchronous vs Synchronous

| Feature | Asynchronous | Synchronous |
|---------|-------------|-------------|
| Synchronization | Per character (start/stop bits) | Block-level (sync characters or flags) |
| Overhead | High (2-3 bits per character) | Low |
| Speed | Low speed links | High speed links |
| Example | Early terminals, modems | HDLC, SDLC |

---

### 2.2 Character-Oriented Protocols
- Data treated as a sequence of **characters** (bytes).
- Use special characters for control.
- Example: **BSC (Binary Synchronous Communication)** by IBM.
- Problem: **Transparency issue** — control characters might appear in data → solved using **DLE (Data Link Escape)** stuffing.

**BSC Frame Structure:**
```
SYN | SYN | STX | Data | ETX | BCC
```
- `SYN` = Synchronization, `STX` = Start of Text, `ETX` = End of Text, `BCC` = Block Check Character

---

### 2.3 Bit-Oriented Protocols
- Data treated as a stream of **bits** (not characters).
- **HDLC (High-level Data Link Control)** — the most important bit-oriented protocol.

**HDLC Frame Structure:**
```
| Flag | Address | Control | Data | FCS | Flag |
| 01111110 | 8 bits | 8/16 bits | Variable | 16/32 bits | 01111110 |
```

- **Flag:** `01111110` — marks frame boundaries.
- **Bit Stuffing:** If five consecutive 1s appear in data, a 0 is inserted to prevent confusion with the flag. Receiver removes it.

**HDLC Frame Types:**

| Type | Purpose |
|------|---------|
| **I-frame (Information)** | Carries user data + piggybacked ACK |
| **S-frame (Supervisory)** | Flow/error control (ACK, NAK, RNR) |
| **U-frame (Unnumbered)** | Connection setup/teardown, management |

**HDLC Modes:**
- **NRM (Normal Response Mode):** Secondary only transmits when polled.
- **ARM (Asynchronous Response Mode):** Secondary can initiate transfer.
- **ABM (Asynchronous Balanced Mode):** Both are peers (used in PPP). ← Most common!

> 📝 Other bit-oriented protocols: **SDLC** (IBM), **LAPB** (X.25), **LLC** (IEEE 802).

---

## 3. Local Area Networks (LANs)

### 3.1 IEEE 802 Standards

| Standard | Description |
|----------|-------------|
| **802.1** | Bridging and management |
| **802.2** | LLC (Logical Link Control) |
| **802.3** | CSMA/CD — Ethernet |
| **802.4** | Token Bus |
| **802.5** | Token Ring |
| **802.11** | Wireless LAN (Wi-Fi) |

**IEEE 802 splits the Data Link Layer into two sublayers:**
- **LLC (Logical Link Control):** Upper sublayer — flow/error control, framing.
- **MAC (Medium Access Control):** Lower sublayer — controls access to the shared medium.

---

### 3.2 Ethernet (IEEE 802.3)

**Access Method: CSMA/CD** (Carrier Sense Multiple Access with Collision Detection)

**How CSMA/CD works:**
1. Station **listens** before transmitting (Carrier Sense).
2. If channel is idle → transmit.
3. While transmitting, **monitor** for collision (Collision Detection).
4. If collision detected → send **jam signal**, wait a **random backoff time** (Binary Exponential Backoff), then retry.

**Ethernet Frame:**
```
| Preamble | Dest. MAC | Src. MAC | Type/Length | Data | FCS |
| 8 bytes  | 6 bytes   | 6 bytes  | 2 bytes     | 46–1500 bytes | 4 bytes |
```

- Minimum frame size: **64 bytes** (to ensure collision detection works).
- Maximum frame size: **1518 bytes**.

**Ethernet Versions:**

| Version | Speed | Medium |
|---------|-------|--------|
| 10Base2 | 10 Mbps | Thin coaxial |
| 10BaseT | 10 Mbps | Twisted pair (UTP) |
| 100BaseTX | 100 Mbps | Fast Ethernet |
| 1000BaseT | 1 Gbps | Gigabit Ethernet |

> 📝 **Format:** `<Speed><Signaling><Medium>` — e.g., 10BaseT = 10 Mbps, Baseband, Twisted pair.

---

### 3.3 Token Bus (IEEE 802.4)

- Physical topology: **Bus**.
- Logical topology: **Ring** (virtual ring among stations).
- A **token** (special frame) is passed around the logical ring.
- Only the station **holding the token** can transmit.
- Used in **industrial/manufacturing** environments (MAP — Manufacturing Automation Protocol).

**Key points:**
- No collisions (token-controlled access).
- Token passed in a **predefined order** (not physically).
- If a station doesn't need to transmit, it passes the token immediately.

---

### 3.4 Token Ring (IEEE 802.5)

- Physical topology: **Ring** (stations connected in a physical ring via MAU — Multistation Access Unit).
- A **token** circulates on the ring.
- Station grabs the **free token**, transmits its frame, then releases the token.

**How it works:**
1. Free token circulates.
2. Station with data captures token, changes it to a **busy token**, appends data.
3. Frame travels around the ring — destination copies the data.
4. Original sender receives frame back, checks for errors, removes it.
5. Sender releases a **new free token**.

**Advantages over Ethernet:**
- Deterministic (guaranteed access time) — good for real-time applications.
- No collisions.

**Disadvantage:** A broken cable or failed station can bring down the whole ring (though MAU has bypass switches).

> 📝 **Comparison:**

| Feature | Ethernet | Token Bus | Token Ring |
|---------|---------|-----------|------------|
| Access | CSMA/CD | Token | Token |
| Topology | Bus/Star | Bus | Ring |
| Collisions | Yes | No | No |
| Deterministic | No | Yes | Yes |
| Standard | 802.3 | 802.4 | 802.5 |

---

### 3.5 FDDI (Fiber Distributed Data Interface)

- **100 Mbps** LAN using **optical fiber**.
- Dual ring topology — **primary ring** (normal) + **secondary ring** (backup).
- Uses **token passing** (like Token Ring) but more sophisticated.
- If primary ring fails → **ring wrapping** — traffic rerouted onto secondary ring automatically.
- Range: up to **200 km**; supports up to **500 stations**.

**FDDI vs Token Ring:**

| Feature | FDDI | Token Ring |
|---------|------|------------|
| Speed | 100 Mbps | 4/16 Mbps |
| Medium | Fiber optic | Twisted pair |
| Rings | Dual | Single |
| Fault Tolerance | High (dual ring) | Low |

> 📝 FDDI is used as a **backbone** (connecting multiple LANs) rather than desktop connections.

---

## 4. Switching

Switching determines **how data travels from source to destination** across a network.

### 4.1 Circuit Switching

- A **dedicated physical path** is established between sender and receiver **before** data transfer.
- Path remains reserved for the **entire duration** of communication.
- Example: **Traditional telephone network (PSTN)**.

**Three phases:**
1. **Connection Establishment** — path is set up end-to-end.
2. **Data Transfer** — data flows through the dedicated path.
3. **Connection Teardown** — path is released.

**Advantages:**
- Guaranteed bandwidth.
- Low latency once connected.
- Suitable for real-time voice/video.

**Disadvantages:**
- Wasteful — channel is idle when no data is being sent.
- Setup delay before communication begins.
- Not flexible for bursty data.

---

### 4.2 Packet Switching

- Data is broken into **packets**; each packet travels **independently** through the network.
- No dedicated path — packets may take **different routes** and are **reassembled** at the destination.
- Example: **The Internet**.

**Two sub-types:**

#### Datagram Packet Switching
- Each packet has full destination address.
- Packets routed **independently** — may arrive **out of order**.
- Receiver must reorder packets.
- Example: IP protocol.

#### Virtual Circuit Packet Switching
- A **logical path** (virtual circuit) is established before sending (but no dedicated physical resources).
- All packets follow the **same path** and arrive **in order**.
- Has setup/teardown phases like circuit switching.
- Example: ATM, Frame Relay, X.25.

| Feature | Datagram | Virtual Circuit |
|---------|---------|-----------------|
| Path | Different per packet | Same for all packets |
| Order | Out of order possible | In order |
| Setup | None | Required |
| Example | IP | ATM, X.25 |

---

### 4.3 Message Switching

- The **entire message** is sent as a single unit — no breaking into packets.
- Each intermediate node **stores the message** completely, then **forwards** it (Store-and-Forward).
- Example: Old telegraph/telex systems, early email.

**Disadvantages:**
- Large messages can block links for a long time.
- High storage requirement at nodes.
- High latency — not suitable for real-time communication.

> 📝 **Why packet switching won:** It's the best of both worlds — no dedicated circuit needed (efficient), messages broken into small packets (no blocking), and reliable delivery.

---

### 4.4 Comparison: All Three Switching Methods

| Feature | Circuit Switching | Message Switching | Packet Switching |
|---------|-----------------|-------------------|-----------------|
| Path | Dedicated | None (store-forward) | None (or virtual) |
| Bandwidth waste | High (idle channel) | Medium | Low |
| Delay | Low (after setup) | High | Low-Medium |
| Order | Always in order | In order | May be out of order |
| Real-time use | ✅ Yes | ❌ No | ✅ (with QoS) |
| Example | Phone calls | Telegram | Internet |

---

## 🔁 Quick Revision Summary

| Topic | Key Point |
|-------|-----------|
| Line Discipline | ENQ/ACK (P2P), Poll/Select (multipoint) |
| Flow Control | Stop-and-Wait, Sliding Window |
| Error Control | Stop-and-Wait ARQ, Go-Back-N, Selective Repeat |
| Character Protocol | BSC — uses DLE stuffing for transparency |
| Bit Protocol | HDLC — bit stuffing, I/S/U frames, Flag = 01111110 |
| IEEE 802.3 | Ethernet — CSMA/CD, min frame 64 bytes |
| IEEE 802.4 | Token Bus — logical ring on physical bus |
| IEEE 802.5 | Token Ring — physical ring, token passing |
| FDDI | 100 Mbps, dual ring, fiber, ring wrapping |
| Circuit Switching | Dedicated path, phone network |
| Packet Switching | Independent packets, Internet |
| Message Switching | Store-and-forward, no packet fragmentation |

---

## 📌 Likely Exam Questions

1. Explain Go-Back-N vs Selective Repeat ARQ with examples.
2. Describe the HDLC frame structure and its frame types.
3. Compare CSMA/CD (Ethernet) with Token Ring.
4. What is bit stuffing? How does HDLC use it?
5. Differentiate Circuit Switching, Packet Switching, and Message Switching.
6. What is FDDI? How does dual ring provide fault tolerance?
7. Explain the IEEE 802 standard layers (LLC and MAC).
8. What are the three modes of HDLC operation?

---
*Notes prepared for MCA — Computer Networks (Unit III)*
