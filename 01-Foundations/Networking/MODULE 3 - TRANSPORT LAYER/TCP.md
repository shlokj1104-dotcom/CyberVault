---
title: TCP
date: 2026-06-23
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 3.5 Connection-Oriented Transport: TCP

> **One-Line Summary:** TCP is the Internet's workhorse reliable transport protocol — a connection-oriented, full-duplex, point-to-point service that stitches together every tool from Section 3.4 (sequence numbers, ACKs, timers, pipelining) and adds three new layers of sophistication on top: dynamic RTT estimation to set smart timeouts, flow control so a fast sender can't drown a slow receiver, and a handshake-based connection lifecycle that establishes shared state before any data ever moves.

---

## Core Idea: A Reliable Byte Stream On Top of Unreliable IP

TCP's job is exactly the same abstraction built in [[PRINCIPLES OF RELIABLE DATA TRANSFER]] — but now implemented for real, running on IP, which makes **zero guarantees** (no ordering, no delivery, no corruption protection). TCP hides all that messiness and presents each application with a clean, ordered, error-free **byte stream**, as if the two endpoints were connected by a private, lossless pipe.

Three properties define TCP at a glance:

1. **Connection-oriented** — before a single byte of application data is exchanged, the two endpoints must _shake hands_: exchange special control segments to establish shared state (buffers, sequence numbers, window sizes) at both ends. This is not merely a convention — the connection state lives **only inside the two endpoints** (not in any router between them).
2. **Full-duplex** — data flows in both directions on the same connection simultaneously. Host A can be sending a file to Host B at the same moment Host B is sending a response back — over the same TCP connection.
3. **Point-to-point** — exactly **one sender, one receiver.** TCP does not support multicasting (one sender → many receivers simultaneously). If you want that, you use UDP and handle reliability yourself.

**Analogy:** Setting up a TCP connection is like two people picking up landline phones at the same time. Before either says anything useful, they both say "Hello?" and confirm the other is present and listening — _then_ the conversation starts. Both can talk and listen simultaneously (full-duplex), but it's strictly a one-on-one call, not a conference.

---

## 3.5.1 The TCP Connection

### How a Connection is Born: The Three-Way Handshake

The process of establishing a TCP connection is called the **three-way handshake** — three segment exchanges, not two, not one. Here's why each step matters:

![[Pasted image 20260623221757.png]]

- **Step 1 — SYN segment:** The client picks a random **initial sequence number (ISN)**, call it `x`, and sends a special TCP segment with the **SYN flag = 1** and no data. Why random? To prevent old, stale connection segments from a crashed previous session being mistaken for new ones.
- **Step 2 — SYNACK segment:** The server acknowledges (`ACK = x+1`, meaning "send me byte `x+1` next") and announces its own random ISN `y`. It also **allocates send and receive buffers** for this connection at this point.
- **Step 3 — ACK segment:** The client acknowledges the server's ISN (`ACK = y+1`) and may already include application data. The client **allocates its own buffers** only after receiving the SYNACK.

The three-way handshake is required — a two-way handshake would fail (explained in detail in §3.5.6).

### MSS and the TCP Send Buffer

Once connected, both sides have a **send buffer** and a **receive buffer**. Application data written into the send buffer gets packetized by TCP into segments for transmission; received segments get reassembled in the receive buffer before being read by the application.

**MSS (Maximum Segment Size):** The maximum amount of _application-layer data_ TCP will put in a single segment. Crucially, MSS does **not** include TCP and IP header sizes — it refers to data payload only.

> **Typical value:** 1460 bytes on Ethernet. Why? Ethernet's **MTU (Maximum Transmission Unit)** is 1500 bytes. Subtract 20 bytes for the IP header and 20 bytes for the TCP header = **1460 bytes of application data per segment.** If a segment crossed a link with a smaller MTU, IP-level fragmentation would be needed (expensive) — so TCP tries to size segments to avoid this.

---

## 3.5.2 TCP Segment Structure

Every TCP segment has a fixed 20-byte header (sometimes more if the Options field is used) followed by the data payload. The header fields are:

![[Pasted image 20260623222005.png]]

### Field-by-Field Breakdown

|Field|Size|Purpose|
|---|---|---|
|**Source Port / Dest Port**|16 bits each|Multiplexing/demultiplexing — identifies the sending/receiving socket (see §3.2)|
|**Sequence Number**|32 bits|Byte-stream position of the _first byte of data_ in this segment|
|**Acknowledgment Number**|32 bits|The sequence number of the _next byte the sender expects to receive_ — confirms receipt of everything before this number|
|**Header Length**|4 bits|TCP header length in 32-bit words (needed because Options field is variable-length)|
|**Flag bits (6 total)**|1 bit each|Control bits — see table below|
|**Receive Window**|16 bits|Number of bytes the _sender of this segment_ is willing to accept — the core of flow control (§3.5.5)|
|**Checksum**|16 bits|Error detection, computed over header + data (same mechanism as UDP)|
|**Urgent Data Pointer**|16 bits|Used only when URG flag is set; points to where urgent data ends|
|**Options**|Variable|Most common: MSS option (negotiated during handshake), timestamp option (used for RTT measurement per RFC 7323)|

### The Six Flag Bits

|Flag|Meaning|
|---|---|
|**URG**|Urgent data present — the Urgent Data Pointer field is valid. Rarely used in practice.|
|**ACK**|The Acknowledgment Number field is valid. Set in almost every segment after the initial SYN.|
|**PSH**|Push — receiver should deliver data to the application immediately, don't buffer.|
|**RST**|Reset — connection must be torn down immediately (used for error conditions).|
|**SYN**|Synchronize — used only during the three-way handshake to initiate a connection.|
|**FIN**|Finish — the sender has no more data to send; initiates connection teardown.|

---

## 3.5.3 Sequence Numbers and Acknowledgment Numbers

These two fields are the heart of TCP's reliability mechanism. Together they define TCP as a **byte-stream protocol** — unlike UDP, which thinks in discrete datagrams, TCP thinks of data as a continuous numbered stream of bytes.

### Sequence Numbers

> **Definition:** The sequence number of a TCP segment is the **byte-stream number of the first byte of data** carried in that segment.

Example: If Host A is sending a 500,000-byte file and the MSS is 1,000 bytes, TCP will create 500 segments:

- Segment 1: sequence number = 0 (bytes 0–999)
- Segment 2: sequence number = 1,000 (bytes 1,000–1,999)
- Segment 3: sequence number = 2,000 (bytes 2,000–2,999)
- …and so on.

The initial sequence number (ISN) is **chosen randomly** at connection setup — not necessarily 0. Why random? If it always started at 0, a stale segment from a recently-closed connection could be accepted by a new connection with the same port pair if it happened to carry a "valid" sequence number.

### Acknowledgment Numbers

> **Definition:** The acknowledgment number in a segment from Host B to Host A is the **sequence number of the next byte Host B is expecting to receive from Host A.** It implicitly confirms receipt of everything _before_ that byte.

This is TCP's **cumulative acknowledgment** model — exactly like GBN's cumulative ACKs from [[PRINCIPLES OF RELIABLE DATA TRANSFER]]. A single ACK saying "I've received everything up through byte 999; send me byte 1000 next" confirms 1,000 bytes in one shot, no matter how many segments they arrived in.

**What if a segment arrives out of order?** The TCP spec doesn't dictate — it says "the implementor should do whatever is best." In practice, virtually all implementations **buffer** out-of-order segments (SR-style), though the acknowledgment number only moves forward when in-order bytes are delivered.

### The Telnet Example — Seeing Both Fields Live

Consider a Telnet session where the client sends a single character `'C'` to the server, and the server echoes it back (this is how classic Telnet works — every keystroke travels round-trip).

![[Pasted image 20260623222203.png]]

Notice the **piggybacking** in step 2: the server's echo data segment and its ACK for the client's keystroke travel together in one TCP segment — no separate ACK segment needed. This is how TCP achieves efficiency: control information rides along with data wherever possible.

> **Analogy:** Imagine writing a letter to a pen pal, and every letter you send also includes "P.S. — got your last letter, thanks." That P.S. is the piggybacked ACK.

---

## 3.5.4 Round-Trip Time Estimation and Timeout

TCP must set a retransmission timer (equivalent to `rdt3.0`'s countdown timer), but with one critical difference: unlike the abstract protocol of Section 3.4, TCP is running on the **real Internet**, where RTT can vary wildly and unpredictably — it might be 2 ms on a local network or 200 ms to a server on the other side of the world.

Setting the timeout too short → **spurious retransmissions** (wasted bandwidth, congestion amplified). Setting the timeout too long → **sluggish recovery** from actual loss (poor performance).

TCP solves this with a **running estimate of RTT that adapts in real time.**

### Step 1 — Measuring SampleRTT

`SampleRTT` = time from when a segment is transmitted to when its ACK arrives. TCP measures this for **one outstanding segment at a time** (whichever it chooses), and **never measures SampleRTT for retransmitted segments** (since you can't tell if the ACK is for the original or the retransmitted copy — this rule is called **Karn's algorithm**).

### Step 2 — Smoothing with EWMA

Raw `SampleRTT` fluctuates too much to use directly as a timeout. TCP smooths it using an **Exponential Weighted Moving Average (EWMA)**:

$$\text{EstimatedRTT} = (1 - \alpha) \cdot \text{EstimatedRTT} + \alpha \cdot \text{SampleRTT}$$

Where **α = 0.125** (i.e., 1/8) by default (RFC 6298).

The key insight of EWMA: the **most recent sample gets the most weight**, but old samples don't disappear instantly — they decay exponentially. Expanding the formula reveals this:

$$\text{EstimatedRTT}_n = (1-\alpha)^n \cdot \text{EstimatedRTT}_0 + \alpha(1-\alpha)^{n-1} \cdot \text{SampleRTT}_1 + \ldots + \alpha \cdot \text{SampleRTT}_n$$

Each past sample contributes, but its influence diminishes geometrically.

**Analogy:** `EstimatedRTT` is like a car's speedometer that reacts quickly to current speed but doesn't thrash violently on every tiny bump — it's a damped reading.

### Step 3 — Measuring Variability

A good timeout needs to account not just for the _average_ RTT but for how much it _varies_. A network with steady 50 ms RTT is very different from one that swings between 10 ms and 90 ms, even if the average is the same.

TCP tracks RTT variability via `DevRTT`:

$$\text{DevRTT} = (1 - \beta) \cdot \text{DevRTT} + \beta \cdot |\text{SampleRTT} - \text{EstimatedRTT}|$$

Where **β = 0.25** (i.e., 1/4) by default. This is essentially a running estimate of the mean absolute deviation of RTT.

### Step 4 — Computing TimeoutInterval

$$\text{TimeoutInterval} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT}$$

The `4 × DevRTT` term provides a **safety margin**: on a stable network where `DevRTT ≈ 0`, the timeout is barely above `EstimatedRTT`. On a jittery network, `DevRTT` grows and the timeout widens automatically to avoid spurious retransmissions.

The **initial TimeoutInterval** is set to **1 second** at connection start, before any samples are available. Once a sample arrives, the formula takes over.

![[Pasted image 20260623222401.png]]
### Timer Doubling (Exponential Backoff)

There's one more rule on top of the formula above: **each time a timeout fires and a retransmission is triggered, the timeout interval is doubled** — regardless of what the RTT formula would otherwise say. This is reset to the formula value as soon as a valid ACK is received.

This exponential backoff behavior provides a crude but effective form of congestion control: if the network is so congested that ACKs aren't coming back, doubling the timeout backs off exponentially rather than flooding the network with retransmissions. (RFC 6298 mandates this.)

---

## 3.5.5 Reliable Data Transfer

TCP builds reliable data transfer on top of IP's unreliable service by combining everything from [[PRINCIPLES OF RELIABLE DATA TRANSFER]] — checksums, sequence numbers, ACKs, retransmission timers — into a real working protocol.

**Key simplification:** TCP uses only **a single retransmission timer** for the entire connection (not one per segment like pure SR). Conceptually, this timer is associated with the _oldest unacknowledged segment_ in the sender's window, just like GBN.

### The TCP Sender — Three Events

The TCP sender's logic can be summarized by how it responds to three distinct events:

**Event 1 — Application calls `send()`:**

- Packetize data into one or more TCP segments (respecting MSS).
- Assign sequence numbers.
- If the timer isn't already running, start it (it tracks the oldest unACK'd byte).
- Pass segments to IP for transmission.

**Event 2 — Timer timeout:**

- **Retransmit the one segment** that caused the timeout (the segment containing the oldest unACK'd byte).
- **Restart the timer** for that retransmission.
- **Double the TimeoutInterval** (exponential backoff — see above).

**Event 3 — ACK arrives:**

- If the ACK covers previously unACK'd segments (i.e., it advances `SendBase`):
    - Update `SendBase` to the new ACK value.
    - **If there are still unACK'd segments in flight**, restart the timer for the _new_ oldest unACK'd segment.
    - If all outstanding data is now ACK'd, stop the timer entirely.

### Retransmission Scenarios — Three Cases in Detail

**Scenario (a) — Lost ACK:**

```
SENDER          RECEIVER
send Seq=92  ──────────────▶  receives, delivers, sends ACK=100
             ◀── (ACK=100 LOST)
[Timer expires for Seq=92]
resend Seq=92 ─────────────▶  receives duplicate → discards → resends ACK=100
             ◀── ACK=100 ────
```

The receiver sees the duplicate (via sequence number), discards the data but still re-ACKs. The sender proceeds.

**Scenario (b) — Premature Timeout:**

```
SENDER          RECEIVER
send Seq=92  ──────────────▶  ACK=100 sent (slow in transit)
send Seq=100 ──────────────▶  ACK=120 sent (faster, arrives first)
[Seq=92's timer fires early — ACK=120 covers it!]
resend Seq=92 ──────────────▶  receiver discards duplicate, re-ACKs with 120
```

TCP's **cumulative ACK** saves the day: `ACK=120` already confirmed `Seq=92` and `Seq=100`, so even the "spurious" retransmission of Seq=92 produces no correctness problem — just slightly wasted bandwidth.

**Scenario (c) — Cumulative ACK Rescues Two Segments:**

```
SENDER                       RECEIVER
send Seq=92  ──────────────▶  receives
send Seq=100 ──────────────▶  receives
             ◀── ACK=92 LOST (but ACK=120 arrives)
             ◀──── ACK=120 ────
```

Because `ACK=120` covers _everything through byte 119_, both `Seq=92` and `Seq=100` are confirmed by a single ACK. This is cumulative acknowledgment efficiency.

### Fast Retransmit — Don't Wait for the Timer

The timer-based mechanism works, but can be slow: if a segment is lost, you might wait a full timeout interval (potentially hundreds of milliseconds) before retransmitting it. TCP has an optimization: **fast retransmit**.

The insight: if a segment is lost but subsequent segments keep arriving, the receiver keeps sending ACKs for the most recently in-order byte it received — **duplicate ACKs**. Each out-of-order arrival generates another duplicate ACK.

> **Rule:** If the sender receives **3 duplicate ACKs** for the same sequence number (meaning 4 total ACKs for that same byte), it **immediately retransmits** the missing segment — without waiting for the timer to expire.

```
SENDER                       RECEIVER
send pkt0 ──────────────────▶  ACK0
send pkt1 ──────────────────▶  ACK1
send pkt2 ──X (lost)
send pkt3 ──────────────────▶  out-of-order → duplicate ACK1
send pkt4 ──────────────────▶  out-of-order → duplicate ACK1
send pkt5 ──────────────────▶  out-of-order → duplicate ACK1
[3 dup ACKs received → FAST RETRANSMIT pkt2 immediately]
resend pkt2 ────────────────▶  ACK5 (covers pkt2 through pkt5)
```

Why 3 duplicate ACKs specifically, not 1 or 2? One or two duplicate ACKs might just be reordering (packets taking slightly different paths and arriving slightly out of order). Three is a strong enough signal to indicate a genuine loss — while still being much faster than waiting for a timer.

**Analogy:** Fast retransmit is like a package tracking system that, instead of waiting for the delivery deadline to pass, alerts the sender the moment three separate "where is my package?" notifications arrive from the same customer — clearly something went wrong.

---

## 3.5.6 Flow Control

TCP's **flow control** mechanism prevents a fast sender from overwhelming a slow receiver — specifically, from filling the receiver's buffer and forcing it to discard data that then has to be retransmitted.

This is distinct from _congestion control_ (§3.7), which throttles the sender to protect the _network_. Flow control protects the _receiver_.

### The Receive Window (`rwnd`)

The receiver advertises its current free buffer space in the **Receive Window** field of every TCP segment it sends. The sender must honor this: at no point should the total amount of unACK'd data the sender has in flight exceed `rwnd`.

**Key variables at the receiver:**

|Variable|Meaning|
|---|---|
|`RcvBuffer`|Total allocated receive buffer size (set at connection time, e.g., 4096 bytes to 4 MB)|
|`LastByteRcvd`|Sequence number of the last byte that arrived from the network|
|`LastByteRead`|Sequence number of the last byte the _application_ has actually read out of the buffer|

The **amount of free space in the receive buffer** at any instant is:

$$\text{rwnd} = \text{RcvBuffer} - [\text{LastByteRcvd} - \text{LastByteRead}]$$

The bracketed term is how much of the buffer is currently _occupied_ by data waiting for the application to read. `rwnd` is the slack — the remaining capacity.

The receiver puts this value in the Receive Window field of every ACK it sends back.

**Key constraint on the sender:**

$$\text{LastByteSent} - \text{LastByteAcked} \leq \text{rwnd}$$

The sender tracks both its most recent byte sent and the most recent byte acknowledged, and ensures the difference — the amount of unACK'd, in-flight data — never exceeds the receiver's advertised `rwnd`.

**Analogy:** `rwnd` is like a water tower level gauge that the city (receiver) broadcasts to everyone pumping water into it (sender). If the gauge reads "only 500 gallons of space left," the pumping station is obligated to slow its flow to avoid an overflow — regardless of how fast its pumps can physically run.

### The Zero-Window Edge Case

What happens when `rwnd = 0`? The receiver's buffer is completely full — the application hasn't read fast enough. The sender's constraint means it **must stop sending**. But now, if the receiver never sends any data of its own (no reason to ACK anything since nothing is being sent), the sender gets no updates that `rwnd` has grown — a **deadlock**.

TCP's fix: the sender continues to send **1-byte probe segments** even when `rwnd = 0`. These tiny segments elicit ACK responses from the receiver, which carry the updated `rwnd` value. Once `rwnd > 0` again, the sender resumes normal operation.

---

## 3.5.7 TCP Connection Management

### Why Not a Two-Way Handshake?

Before understanding the three-way handshake in depth, it's worth understanding why two exchanges would fail. Imagine:

1. Client sends "I want to connect, my ISN = 43."
2. Server replies "OK, connection established."

Problem: the server has no guarantee the client's message wasn't a **delayed duplicate** from a previous, already-closed connection. If a very old SYN segment from a crashed client session was still floating around the network and arrived now, the server would allocate resources for a "connection" whose client has long since moved on. A two-way handshake cannot distinguish a new SYN from a delayed old one.

The three-way handshake solves this: the third leg (client's ACK of the server's SYNACK) confirms the client is _actually present and responding right now_ — a ghost message from the past cannot do that.

### Three-Way Handshake — Full Detail

```
CLIENT                          SERVER
(state: CLOSED)                 (state: LISTEN)
        │                             │
        │─ SYN ─────────────────────▶ │  [seq=x, SYN=1, no data]
(SYN_SENT)                            │  Server: alloc rcv/snd buffers & vars
        │                             │
        │ ◀── SYNACK ─────────────────│  [seq=y, ack=x+1, SYN=1, ACK=1]
        │                        (SYN_RCVD)
(ESTABLISHED)                         │
Client: alloc rcv/snd buffers         │
        │                             │
        │─ ACK ─────────────────────▶ │  [ack=y+1, ACK=1, may carry data]
        │                        (ESTABLISHED)
        │                             │
        ◀══════════ DATA ═════════════▶
```

- The SYN segment costs one sequence number (even though it carries no data) — `x+1` is the next byte expected.
- Similarly, the SYNACK costs one sequence number — `y+1` is the next byte expected from the server.
- The third ACK does **not** consume a sequence number (unless it also carries data).

### Connection Teardown — The Four-Way Close

Either side can initiate closing. Suppose the client decides it has no more data to send:

```
CLIENT                          SERVER
(ESTABLISHED)                   (ESTABLISHED)
        │                             │
        │─ FIN ─────────────────────▶ │  [FIN=1, seq=x+2]
(FIN_WAIT_1)                          │  (server can still send data!)
        │                        (CLOSE_WAIT)
        │ ◀── ACK ────────────────────│  [ack=x+3] ← server ACKs the FIN
(FIN_WAIT_2)                          │  ... server finishes sending remaining data ...
        │                             │
        │ ◀── FIN ────────────────────│  [FIN=1, seq=y+2]
        │                        (LAST_ACK)
(TIME_WAIT)                           │
        │─ ACK ─────────────────────▶ │  [ack=y+3]
        │                        (CLOSED)
[wait 2 × max segment lifetime]
        │
(CLOSED)
```

This is a **half-close**: after the client sends FIN, it can no longer send data but can still _receive_. The server can continue sending until it's also done, then sends its own FIN. Total: 4 segments (FIN, ACK, FIN, ACK).

**Why TIME_WAIT?** The client waits ~30 seconds to 2 minutes after sending the final ACK before truly closing. This serves two purposes:

1. If the final ACK was lost, the server will retransmit its FIN, and the client needs to be alive to respond.
2. Ensures all old, in-flight segments from this connection's port pair have expired before those port numbers are reused for a new connection — preventing "ghost" segments from being mistaken for new traffic.

If a connection is abruptly reset (e.g., the server sends an unexpected segment to a port with no active TCP connection), TCP responds with a **RST segment** to signal the error immediately.

### TCP State Diagrams

**Client-side state machine (simplified):**

```
CLOSED
  │ (active open / send SYN)
  ▼
SYN_SENT
  │ (rcv SYNACK / send ACK)
  ▼
ESTABLISHED ◀─────────────────────────────────────────────────────────▶ (data transfer)
  │ (close / send FIN)
  ▼
FIN_WAIT_1
  │ (rcv ACK)
  ▼
FIN_WAIT_2
  │ (rcv FIN / send ACK)
  ▼
TIME_WAIT
  │ (timeout after 2 × max segment lifetime)
  ▼
CLOSED
```

**Server-side state machine (simplified):**

```
CLOSED
  │ (socket created / passive open)
  ▼
LISTEN
  │ (rcv SYN / send SYNACK)
  ▼
SYN_RCVD
  │ (rcv ACK)
  ▼
ESTABLISHED ◀─────────────────────────────────────────────────────────▶ (data transfer)
  │ (rcv FIN / send ACK)
  ▼
CLOSE_WAIT
  │ (close / send FIN)
  ▼
LAST_ACK
  │ (rcv ACK)
  ▼
CLOSED
```

These states are directly observable: on Linux, `ss -tn` or `netstat -tn` shows every TCP connection and its current state — you'll see `SYN_SENT`, `ESTABLISHED`, `TIME_WAIT` live on any active machine.

---

## Putting TCP Together — How All the Pieces Interlock

The five mechanisms of TCP are not independent — they form a tightly interdependent system:

```
Three-way handshake ──────▶ Establishes ISNs for sequence/ack numbering
                                        │
Sequence + ACK numbers ───▶ Enable reliable, ordered byte-stream delivery
                                        │
RTT estimation (EWMA) ────▶ Sets TimeoutInterval for the retransmission timer
                                        │
Retransmission timer ─────▶ Detects loss; triggers retransmit
Fast retransmit (3 dup ACKs)▶ Detects loss faster; skips the timer wait
                                        │
Flow control (rwnd) ──────▶ Caps sender rate at receiver's advertised capacity
                                        │
Connection teardown ──────▶ Gracefully closes both half-connections, then waits
```

TCP is, in essence, a **GBN-style** protocol (cumulative ACKs, single timer) with **SR-style** optimizations (receiver buffering of out-of-order segments, SACK option for explicit selective ACK information) layered on top. The textbook presents a simplified view; real TCP implementations include dozens of additional refinements (SACK, FACK, ECN, TFO, BBR congestion control, and so on).

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Random ISNs**|Predictable ISNs enabled classic **TCP session hijacking**: forge a RST or data segment with the right sequence number and you can terminate or inject into someone else's session|Use cryptographically unpredictable ISNs (RFC 6528 mandates this); apply TLS to protect data integrity on top of TCP|
|**SYN flood attack**|Send a flood of SYN segments from spoofed source IPs → server allocates buffers and waits for Step 3 that never comes → memory exhausted|**SYN cookies**: defer buffer allocation until the full handshake completes; the server encodes state in the SYNACK's sequence number rather than allocating it|
|**Three-way handshake amplification**|Spoofed SYN with victim's IP as source → server sends SYNACK to victim → victim sends RST → server gets RST, no harm done. But at scale (DDoS): the server wastes resources on millions of half-open connections|SYN rate limiting, SYN cookies, firewall scrubbing|
|**ACK spoofing and window manipulation**|Inject forged ACKs with large sequence numbers to trick sender into believing data was received; or forge zero-window ACKs to force sender to pause|TLS provides cryptographic integrity over the entire TCP payload including ACK numbers (DTLS does the same for UDP)|
|**RST injection**|Forge a RST segment with a valid sequence number → silently terminate a TCP connection (basis of "TCP Reset attacks", used by some firewalls and censorship systems)|Use TCP Authentication Option (RFC 5925) to authenticate TCP segments; or protect with TLS|
|**TIME_WAIT exploitation**|On systems with predictable port reuse, a packet from a "dead" connection might be accepted as valid by a new connection on the same ports|Randomize ephemeral port selection; OS-enforced TIME_WAIT duration prevents reuse|
|**Flow control abuse**|Announce `rwnd = 0` constantly → force sender to pause indefinitely; a form of low-rate DoS|Rate-limit zero-window persist probes; detect stalled connections with keepalives|

---

## Questions I Still Have

- [ ] TCP's retransmission uses a single timer for the oldest unACK'd byte — but real Linux TCP (`tcp_retransmit_skb`) triggers retransmission per-socket based on a `jiffies`-based timer wheel. What's the actual implementation difference between "one logical timer" and how the kernel efficiently handles thousands of simultaneous TCP connections?
- [ ] The SACK (Selective Acknowledgment) TCP option (RFC 2018) extends the ACK field to explicitly report received out-of-order blocks. How does a TCP sender with SACK enabled decide which segment to retransmit — does it behave exactly like textbook SR, or are there corner cases (e.g., SACK scoreboard + FACK + D-SACK)?
- [ ] During SYN floods, SYN cookies work by encoding the ISN as `hash(src_ip, src_port, dst_ip, dst_port, timestamp)` — what does the server lose by deferring buffer allocation this way? (E.g., can it still negotiate MSS and window scale options if it encoded them in the ISN hash?)
- [ ] The TIME_WAIT timer is set to "2 × maximum segment lifetime" — RFC 793 says MSL = 2 minutes, giving a 4-minute TIME_WAIT. Modern Linux defaults to 60 seconds. In practice, high-traffic servers (millions of connections/day) can accumulate thousands of TIME_WAIT sockets. What are the real-world mitigation strategies beyond `SO_REUSEADDR`?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Connection-oriented**|A protocol that establishes shared state between endpoints via a handshake before data flows|
|**Full-duplex**|Data can flow in both directions simultaneously over the same connection|
|**Three-way handshake**|The SYN → SYNACK → ACK exchange that initializes a TCP connection|
|**MSS (Maximum Segment Size)**|Maximum application-data bytes per TCP segment; typically 1460 bytes on Ethernet|
|**ISN (Initial Sequence Number)**|The randomly chosen starting sequence number for a new TCP connection|
|**Sequence number**|Byte-stream offset of the first data byte in a segment|
|**Acknowledgment number**|The next byte the receiver expects; implicitly confirms all prior bytes|
|**Cumulative ACK**|A single ACK that confirms all bytes up to and including a given sequence number|
|**Piggybacking**|Bundling an ACK for received data together with new outgoing data in the same segment|
|**SampleRTT**|A single measured round-trip time for one non-retransmitted segment|
|**EstimatedRTT**|Exponential weighted moving average of SampleRTT values; α = 0.125|
|**DevRTT**|EWMA of RTT variability (mean absolute deviation); β = 0.25|
|**TimeoutInterval**|`EstimatedRTT + 4 × DevRTT`; the actual timer value TCP sets|
|**Karn's algorithm**|Rule: never use a retransmitted segment's ACK arrival for SampleRTT measurement|
|**Exponential backoff**|Doubling TimeoutInterval on each consecutive timeout, to back off under congestion|
|**Fast retransmit**|Retransmitting a segment immediately upon receiving 3 duplicate ACKs, before the timer fires|
|**Flow control**|Mechanism preventing the sender from overrunning the receiver's buffer|
|**Receive window (rwnd)**|Advertised free buffer space at the receiver; constrains sender's in-flight data volume|
|**Zero-window probe**|1-byte segment sent when rwnd = 0 to solicit an updated window advertisement|
|**FIN**|TCP flag signaling "I have no more data to send"; initiates half-close|
|**RST**|TCP flag signaling immediate, abnormal connection teardown|
|**TIME_WAIT**|Post-FIN delay (≈ 2 × MSL) before fully closing, to ensure clean teardown|
|**SYN flood**|DoS attack exploiting the half-open connection state created by unanswered SYNs|
|**SYN cookies**|Defense against SYN floods: encode connection state in ISN, deferring buffer allocation|

---

## Related Concepts

- [[3.4 Principles of Reliable Data Transfer]] — the abstract toolkit (rdt1.0 through SR) that TCP instantiates
- [[3.6 Principles of Congestion Control]] — the complement to flow control: protecting the _network_, not the receiver
- [[3.7 TCP Congestion Control]] — TCP's actual congestion control algorithm (slow start, congestion avoidance, fast recovery)
- [[2.2 The Web and HTTP]] — HTTP/1.1 runs over TCP; the three-way handshake cost is part of why HTTP/2 and QUIC were invented

---

→ Next: [[3.6 Principles of Congestion Control]]