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

> **One-Line Summary:** TCP takes the unreliable, best-effort delivery service handed up from IP and, using nothing but sequence numbers, acknowledgments, timers, and a sliding receive window — all the principles built up in Section 3.4 — turns it into something an application can treat as a single, ordered, gap-free, congestion-and-receiver-aware stream of bytes flowing between two processes, complete with an explicit handshake to open the connection and a matching teardown to close it cleanly.

---

## Core Idea: Reliability Is Built, Not Given

The network layer's IP service is deliberately minimal and unreliable: it does not guarantee a datagram arrives, does not guarantee datagrams arrive in the order they were sent, and does not guarantee the bits inside a datagram are intact. Buffers overflow, routers drop packets, datagrams take different paths and arrive out of sequence, bits flip in transit. Because transport-layer segments ride inside IP datagrams, every one of these problems is inherited by TCP before it does anything else.

TCP's entire job in this section is to construct a **reliable data transfer service** on top of that shaky foundation — so that whatever byte stream a sending process writes into its socket is exactly the byte stream the receiving process reads out the other end: uncorrupted, gap-free, non-duplicated, and in order. It does this using the toolkit already developed in Section 3.4 (ACKs, timers, sequence numbers), but TCP is also **connection-oriented**: before any data flows, the two sides must explicitly handshake to agree on the parameters of the conversation.

TCP is formally defined in RFC 793, with a long trail of amendments (RFC 1122, RFC 2018, RFC 5681, RFC 7323) that have since been folded into a single consolidated specification, RFC 9293, which now officially supersedes RFC 793.

---

## 3.5.1 The TCP Connection

### Why "Connection-Oriented" Doesn't Mean a Physical Circuit

It's tempting to picture a TCP "connection" the way you'd picture a phone call routed through dedicated switched circuits (TDM/FDM) — a fixed path reserved end-to-end. That picture is wrong. A TCP connection is a **logical** construct: the only place any connection "state" actually lives is inside the operating systems of the two communicating end hosts. The routers and switches in between never look at TCP headers and never track connections — to them, every TCP segment is just payload inside an IP datagram, indistinguishable from any other traffic.

> **Analogy — A Connection Is a Shared Understanding, Not a Wire:** Think of two pen pals who agree, "from now on, we'll number our letters and confirm receipt of each one." The postal service in between doesn't know or care about this agreement — it just moves individual letters, the same as it always has. The "connection" exists only in the two notebooks the pen pals keep at their own desks, tracking what's been sent and confirmed. TCP's connection state is exactly that: bookkeeping kept only at the two endpoints.

### Two Defining Properties of a TCP Connection

|Property|What It Means|
|---|---|
|**Full-duplex**|If Process A and Process B are connected, data can flow from A to B _and_ from B to A at the same time — the connection isn't a one-way pipe that has to be "turned around."|
|**Point-to-point**|A TCP connection always involves exactly one sender and one receiver. Unlike multicast schemes that fan data out from one sender to many simultaneous receivers in a single send operation, TCP is strictly a two-party affair — "two hosts are company, three are a crowd."|

### The Three-Way Handshake

Before either side sends a single byte of real data, the process wanting to initiate contact (the **client process**) and the process waiting to be contacted (the **server process**) must first "handshake" — exchange preliminary segments to establish the parameters of the upcoming transfer and initialize the state variables each side will maintain for the life of the connection.

In code, the client side simply calls:

```python
clientSocket.connect((serverName, serverPort))
```

Underneath that single line, three segments are exchanged:

1. The client sends a special segment carrying no application-layer payload — just control information.
2. The server responds with its own special, payload-free segment.
3. The client responds a final time; this third segment _may_ carry the first chunk of actual application data.

Because exactly three segments cross the wire to get the connection going, this is called the **three-way handshake**. (Section 3.5.6 covers the exact bits and state-machine mechanics; for now what matters is that the handshake exists and that it's how each side learns the other is ready and agrees on starting parameters.)

```
   CLIENT                                          SERVER
   ──────                                          ──────
clientSocket.connect()
        │
        ├──── Segment 1: connection request ─────────────►
        │            (no payload)
        │
        │◄──── Segment 2: server's response ───────────────┤
        │            (no payload)
        │
        ├──── Segment 3: client's ack / first data ───────►
        │       (may carry payload)
        │
        ▼                                                  ▼
   Connection established — full-duplex byte stream now open
```

### From Socket to Send Buffer to Network

Once the connection exists, sending data is conceptually simple from the application's point of view, but several things happen underneath:

1. The client process writes a stream of bytes through its **socket** — recall from Section 3.2 that the socket is the only door between a process and the transport layer.
2. The moment data passes through that door, it is in TCP's hands, and TCP places it into the connection's **send buffer** — one of several buffers set aside during the three-way handshake.
3. From time to time, TCP grabs a chunk of data out of the send buffer, wraps it in a header, and hands the resulting segment down to the network layer.

![[Pasted image 20260623221757.png]]

The textbook's "laid-back" original wording (RFC 793) leaves the _exact_ moment TCP decides to send buffered data deliberately unspecified — TCP is free to send "at its own convenience."

### How Big a Chunk? The Maximum Segment Size (MSS)

TCP can't just grab an arbitrarily large chunk of the send buffer and ship it — the chunk has to fit inside a single link-layer frame. This limit is called the **Maximum Segment Size (MSS)**.

|Term|Definition|
|---|---|
|**MTU**|Maximum Transmission Unit — the largest link-layer frame the local sending host can put on the wire.|
|**MSS**|The largest chunk of _application-layer_ data TCP will put into one segment, set so that MSS plus the TCP/IP header (typically 40 bytes) still fits inside one MTU-sized frame.|

A typical Ethernet/PPP MTU of 1,500 bytes therefore yields an MSS around 1,460 bytes. (Confusingly, MSS measures the data field only — _not_ the total segment size including headers — but this terminology is too entrenched to fix now.) More sophisticated approaches exist to discover the smallest MTU along an entire source-to-destination path (RFC 1191) and size MSS accordingly.

When TCP needs to send something larger than one MSS-worth of data — a large file, for instance — it simply breaks the data into a sequence of MSS-sized **TCP segments** (with the final leftover chunk typically smaller than MSS). Interactive applications like Telnet and SSH often send segments far smaller than MSS, sometimes carrying just a single byte of payload.

---

## 3.5.2 TCP Segment Structure

A TCP segment is built from a **header** followed by a **data field**, where the data field holds a chunk of application bytes (up to MSS in size).

### Anatomy of the TCP Header

![[Pasted image 20260623222005.png]]

|Field|Size / Purpose|
|---|---|
|**Source / Dest port #**|The same two fields explained in Section 3.2 — used for multiplexing/demultiplexing to the right socket.|
|**Sequence number**|32-bit; identifies where in the byte stream this segment's data begins. Used in providing reliable data transfer.|
|**Acknowledgment number**|32-bit; identifies the next byte the _sender of this segment_ is expecting from the other side. Also central to reliable data transfer.|
|**Header length**|4-bit field, giving the header's length in 32-bit words — needed because the options field makes the header variable-length (typical header is 20 bytes, meaning options is usually empty).|
|**Flags (6 bits)**|ACK, RST, SYN, FIN, CWR, ECE — see table below.|
|**Receive window**|16-bit; used for **flow control** (Section 3.5.5) — tells the other side how many more bytes the receiver is currently willing to accept.|
|**Internet checksum**|Same role as in UDP — error detection over the segment.|
|**Urgent data pointer**|Marks the end of "urgent" data flagged by the URG bit (rarely used in practice; kept mostly for completeness).|
|**Options**|Variable-length; used to negotiate MSS, or to carry a window-scaling factor for high-speed networks, among other things (RFC 854, RFC 1323).|

### The Six Flag Bits

|Flag|Meaning|
|---|---|
|**ACK**|Set when the acknowledgment number field actually contains a valid acknowledgment.|
|**RST, SYN, FIN**|Used for connection setup and teardown (covered fully in 3.5.6).|
|**CWR, ECE**|Used for explicit congestion notification (Section 3.7.2).|
|**PSH**|(Rarely used in practice) signals the receiver should pass data up to the application immediately rather than buffering it.|
|**URG**|(Rarely used in practice) signals the sender has marked some data as "urgent"; the urgent data pointer field marks where that urgent data ends.|

> **Why so many fields feel "unused" in practice:** TCP was designed to be general enough for use cases beyond ordinary file/web transfer. PSH and URG exist for applications that want fine control over buffering timing, but almost no modern software actually relies on them — they're vestigial generality, not dead weight removed from the spec.

---

## Sequence Numbers and Acknowledgment Numbers: TCP's Core Bookkeeping

These two fields are arguably the most important part of the entire TCP header, since reliable data transfer is built almost entirely on top of them.

### TCP Sees Data as a Byte Stream, Not a Series of Segments

This is the single most important mental shift coming from UDP: **TCP numbers individual bytes**, not segments. The sequence number written into a segment's header is simply _the byte-stream number of that segment's first data byte._

**Worked example:** Suppose Host A wants to send a 500,000-byte file to Host B, with MSS = 1,000 bytes. TCP implicitly numbers every byte in the stream starting from some initial value (say 0 for simplicity), and breaks the stream into 500 segments:

|Segment|Sequence Number|
|---|---|
|1st|0|
|2nd|1,000|
|3rd|2,000|
|...|...|

![[Pasted image 20260623223949.png]]

> **Analogy — Numbering Pages, Not Chapters:** Imagine mailing a 500-page manuscript one envelope at a time, ten pages per envelope. You don't label each envelope "Envelope #1, #2, #3…" — you label it with the page number of the first sheet inside ("Pages 1–10," "Pages 11–20"). That way, if envelope #3 goes missing, the recipient knows precisely which pages are gone, not just "the third batch" — useful information regardless of how the other envelopes were split or reordered.

### Acknowledgment Numbers: A Little Trickier

Because a TCP connection is full-duplex, Host A may simultaneously be _sending_ data to B and _receiving_ data from B as part of the same connection. The acknowledgment number A puts in _its own_ outgoing segments is **the sequence number of the next byte A is expecting from B** — i.e., it acknowledges everything received so far by naming the very next thing still wanted.

**Worked example:** Suppose Host A has correctly received bytes 0 through 535 from B, and is now waiting for byte 536 onward. When A sends its next segment to B, the acknowledgment number field will contain **536** — not 535. TCP acknowledges "what I want next," not "what I last got."

### Cumulative Acknowledgments and Out-of-Order Arrival

Now suppose Host A has received two segments from B: one with bytes 0–535, and a _second_ one with bytes 900–1,000 — meaning bytes 536–899 are still missing, and the second segment arrived **out of order** (ahead of where it belongs in the stream).

Because A is still waiting to fill the gap left by the missing bytes, A's next segment to B will still carry acknowledgment number **536** — the sequence number of the first byte still missing from the contiguous stream. This is exactly what it means for TCP to provide **cumulative acknowledgments**: an ACK number always names the first byte still missing, regardless of what's arrived out of order beyond that gap.

### The Receiver's Dilemma: What to Do With Out-of-Order Data

The TCP specification deliberately leaves this choice up to the implementer. There are two options:

|Choice|Behavior|Trade-off|
|---|---|---|
|**Discard out-of-order segments**|Simpler receiver design|Wastes network bandwidth — the sender will have to retransmit data that already physically arrived|
|**Buffer out-of-order bytes, wait for gaps to fill**|More efficient use of bandwidth|More complex receiver bookkeeping|

In practice, every real TCP implementation takes the second approach.

### Why Initial Sequence Numbers Are Randomized

The worked examples above assumed sequence numbers start at 0 for simplicity, but **in reality both sides of a TCP connection pick a random initial sequence number (ISN)** during the handshake. This isn't an arbitrary design flourish — it exists specifically to minimize the chance that a stray segment, still wandering the network from an earlier, already-closed connection between the _same two hosts on the same port numbers_, gets mistaken for a valid segment belonging to the new connection.

> **Security relevance:** Because the four-tuple identifying a TCP socket (Section 3.2) doesn't change just because a connection closed and a new one opened on the same ports, the sequence number space is one of the only things separating "old, dead traffic" from "new, live traffic." Predictable ISNs have historically been exploited in real attacks (sequence-number prediction, used to hijack or inject into TCP streams) — which is exactly why modern TCP stacks generate ISNs cryptographically rather than incrementally.

---

## Telnet: A Worked Case Study in Sequence/Ack Numbers

Telnet (RFC 854) is a remote-login application protocol that runs over TCP. It's useful here specifically _because_ it's interactive rather than bulk-transfer: each character a user types is sent in its own tiny segment, and the remote host **echoes back** a copy of every character so it can be displayed locally, confirming the keystroke was received and processed at the far end.

> **Why echo-back matters:** Without it, a user would have no way of knowing whether their keystroke actually reached the remote machine — they'd be typing blind. The price is that every character crosses the network _twice_: once on the way in, once echoed back on the way out, before it ever appears on the user's own screen.

> **A security footnote:** Telnet sends everything — including passwords — completely unencrypted, making it trivially vulnerable to eavesdropping. This is precisely why SSH has replaced Telnet as the standard remote-login tool in practice; Telnet survives mainly as a clean _teaching example_ for sequence numbers, not as something anyone should actually deploy.

### Walking the Three-Segment Exchange

Suppose Host A (client) has initial sequence number 42, and Host B (server) has initial sequence number 79. The user types a single character, `'C'`, then walks away for coffee.

![[Pasted image 20260623222203.png]]

|Segment|Sequence #|Ack #|Data|Purpose|
|---|---|---|---|---|
|**1 (A→B)**|42|79|`'C'`|First byte of client's stream (ISN=42); A hasn't received anything from B yet, so it ACKs B's _initial_ sequence number, 79|
|**2 (B→A)**|79|43|`'C'` (echo)|First byte of server's stream (ISN=79); the ACK=43 confirms B successfully got A's byte 42 and is now waiting for byte 43. The echoed character is **piggybacked** onto this same acknowledgment segment — one segment does double duty|
|**3 (A→B)**|43|80|_(empty)_|Pure acknowledgment — confirms receipt of the echoed `'C'` (server's byte 79), so ACK=80. Even with zero data bytes, the segment still needs _some_ sequence number, since TCP always has one|

> **Analogy — Piggybacking as Combining Errands:** Segment 2 is doing two jobs at once: it's confirming "got your letter" _and_ delivering its own reply, all in a single trip. This is exactly like a courier who, having driven out to deliver your package, also picks up your neighbor's outgoing mail on the same trip rather than making a separate run — TCP combines an acknowledgment with outbound data whenever it conveniently can, rather than always sending bare ACKs.

---

## 3.5.3 Round-Trip Time Estimation and Timeout

TCP recovers lost segments using the same timeout/retransmit mechanism introduced generically in Section 3.4 — but turning that idea into a working real-world protocol raises subtle questions: how long should the timeout actually be? Too short, and perfectly fine segments get needlessly retransmitted; too long, and a genuinely lost segment sits unretransmitted for ages, inflating delay. The timeout has to be calibrated against the connection's actual **round-trip time (RTT)** — the time from sending a segment to receiving its acknowledgment.

### Measuring SampleRTT

TCP defines **SampleRTT** as the measured time between sending a segment and receiving an acknowledgment for it. Two important restrictions on how this is actually measured:

1. **Only one SampleRTT measurement is in flight at a time**, even though many segments may be unacknowledged simultaneously (because of pipelining) — TCP doesn't time every single outstanding segment, just one at a time, yielding roughly one new SampleRTT value per RTT.
2. **A retransmitted segment is never used to compute SampleRTT** (Karn's algorithm). The reasoning: if a segment had to be retransmitted, you can no longer tell whether the ACK you eventually got was answering the _original_ transmission or the _retransmission_ — using it would corrupt the RTT estimate with an ambiguous measurement.

### From SampleRTT to a Stable Estimate: EstimatedRTT

Raw SampleRTT values bounce around a lot — congestion in routers and shifting load on the end systems both cause natural fluctuation. So rather than reacting to any single sample, TCP maintains a smoothed running estimate, **EstimatedRTT**, updated on every new sample using:

```
EstimatedRTT = (1 − α) · EstimatedRTT + α · SampleRTT
```

The recommended value is **α = 0.125** (i.e., 1/8), per RFC 6298, giving:

```
EstimatedRTT = 0.875 · EstimatedRTT + 0.125 · SampleRTT
```

This kind of formula is called an **exponential weighted moving average (EWMA)** in statistics — "exponential" because the influence of any individual old sample decays exponentially fast as newer samples arrive. The effect is that recent network conditions are weighted more heavily than stale ones, which makes sense: a sample from ten round trips ago says very little about _current_ congestion.

### Measuring the Wobble: DevRTT

Knowing the _average_ RTT isn't enough — a timeout also needs to account for how much RTT samples typically vary, so the timer isn't tripped by ordinary jitter. RFC 6298 defines **DevRTT** as an estimate of how far SampleRTT typically deviates from EstimatedRTT:

```
DevRTT = (1 − β) · DevRTT + β · | SampleRTT − EstimatedRTT |
```

with the recommended **β = 0.25**. DevRTT is itself an EWMA — this time, of the _absolute difference_ between each new sample and the current estimate, rather than of the samples themselves.

|If SampleRTT fluctuates...|Then DevRTT will be...|
|---|---|
|Very little|Small|
|A lot|Large|

### Setting the Actual Timeout Interval

Putting EstimatedRTT and DevRTT together, two competing pressures shape the right value for TCP's `TimeoutInterval`:

|Pressure|Consequence if Violated|
|---|---|
|Interval should be **≥ EstimatedRTT**|Otherwise the timer fires before a reply could possibly have arrived, triggering pointless, unnecessary retransmissions|
|Interval **shouldn't be much larger than EstimatedRTT**|Otherwise a genuinely lost segment sits around far too long before TCP notices and retransmits, inflating end-to-end delay|

The resolution is to set the timeout to EstimatedRTT _plus a margin_, where the margin itself should grow when SampleRTT is fluctuating a lot, and shrink when it's stable — which is exactly what DevRTT measures. TCP's actual formula:

```
TimeoutInterval = EstimatedRTT + 4 · DevRTT
```

An initial `TimeoutInterval` value of **1 second** is recommended (RFC 6298). Whenever a timeout actually fires, the value of `TimeoutInterval` is **doubled** before the segment is retransmitted — a safeguard against a premature second timeout on a segment that's already about to be acknowledged. As soon as any _new_ segment is received and EstimatedRTT updates, `TimeoutInterval` is immediately recomputed from the formula above, discarding the doubled value.

> **Analogy — Why Double the Timeout After a Miss:** If you text a friend and get no reply in the time you normally expect, you might wait a bit longer before texting again rather than immediately assuming something's wrong and re-sending — because maybe they're just a little slower than usual right now, not gone entirely. Doubling the timeout is TCP giving the network the same benefit of the doubt, just once, before reverting to its normal expectation on the next successful round trip.

![[Pasted image 20260623222401.png]]

### Principles in Practice: How the Pieces Fit Together

TCP provides reliable transfer using positive acknowledgments and timers, in the spirit of Section 3.4 — ACK what's correctly received, retransmit what's presumed lost or corrupted. Some implementations layer an _implicit NAK_ mechanism on top: as covered shortly under Fast Retransmit, three duplicate ACKs for the same segment act as an indirect signal that the _following_ segment was lost, triggering retransmission before the timer even expires.

Just as in rdt3.0, TCP itself can never be _certain_ whether a segment or its ACK was lost, corrupted, or simply delayed — the sender's response is identical regardless: retransmit. TCP also uses **pipelining**, allowing multiple transmitted-but-not-yet-acknowledged segments to be in flight simultaneously (recall from Section 3.4 how much this helps throughput when the segment-size-to-RTT ratio is small). The exact number of outstanding unacknowledged segments a sender is allowed at once is governed jointly by flow control (this section, 3.5.5) and congestion control (Section 3.7) — for now, it's enough to know pipelining is happening under the hood.

---

## 3.5.4 Reliable Data Transfer

### Why a Single Timer, Not One Per Segment

Conceptually, the cleanest design would associate a separate timer with _every_ unacknowledged segment. In practice, that's expensive — tracking and firing many concurrent timers is real overhead. The recommended approach (RFC 6298), and the one TCP actually follows, is to use just a **single retransmission timer**, even while multiple segments are simultaneously in flight unacknowledged.

### The Three Events a TCP Sender Must Handle

A highly simplified TCP sender — one not yet constrained by flow or congestion control, sending data in only one direction — reduces to reacting to exactly three kinds of events:

|Event|Sender's Response|
|---|---|
|**Data arrives from the application above**|TCP wraps it into a segment carrying sequence number `NextSeqNum`, starts the timer _if it isn't already running_, passes the segment to IP, and advances `NextSeqNum` by the length of the data sent|
|**The timer expires (timeout)**|TCP retransmits the _not-yet-acknowledged segment with the smallest sequence number_ — i.e., the oldest outstanding segment — and restarts the timer|
|**An ACK arrives with value `y`**|If `y > SendBase`, the ACK is acknowledging at least one previously-unacknowledged segment: TCP updates `SendBase = y`, and restarts the timer if there are still any unacknowledged segments outstanding|

Two state variables drive this logic:

|Variable|Meaning|
|---|---|
|**SendBase**|The sequence number of the oldest byte that is still unacknowledged. (`SendBase − 1` is therefore the last byte known to have been received in order by the receiver.)|
|**NextSeqNum**|The sequence number to assign to the _next_ new segment of data sent|

```
loop forever:
    on data from application:
        segment = build(seq = NextSeqNum, data)
        if timer not running: start timer
        send segment to IP
        NextSeqNum += len(data)

    on timer expiry:
        retransmit oldest unacked segment (smallest seq #)
        restart timer

    on ACK received (ack field = y):
        if y > SendBase:
            SendBase = y
            if any segments still unacked:
                restart timer
```

It's worth noting explicitly: TCP starts the timer when a segment is handed to IP **only if no timer is already running** for some other unacknowledged segment — it's conceptually helpful to think of the single timer as always being associated with the _oldest_ unacknowledged segment, since that's the one whose loss would be detected first.

Because TCP uses cumulative ACKs, an ACK value `y > SendBase` may be confirming **multiple** previously-sent segments at once, not just one — which is exactly the mechanism that lets cumulative ACKs absorb the loss of an earlier ACK without forcing a retransmission, as the next scenarios show.

### A Few Worked Scenarios

**Scenario 1 — A Lost ACK Triggers an Unnecessary-Looking but Correct Retransmission**

Host A sends one 8-byte segment (seq=92) to Host B. B receives it fine and replies with ACK=100 — but that ACK is lost in the network. A's timer eventually fires, and A retransmits the _same_ segment (seq=92). B receives the retransmission, recognizes from the sequence number that these bytes were already received, and simply discards the duplicate data (while presumably re-ACKing).

![[Pasted image 20260623224355.png]]

**Scenario 2 — Cumulative ACKs Absorb a Lost ACK Without a Wasted Retransmission**

A sends two segments back-to-back: seq=92 (8 bytes) and seq=100 (20 bytes). Both arrive at B intact, and B sends two separate ACKs — ACK=100 for the first, ACK=120 for the second. Suppose **neither** ACK reaches A before A's timeout fires. A retransmits only the _first_ segment (seq=92) and restarts the timer. As long as the ACK for the _second_ segment (ACK=120) arrives before this new timeout expires, the second segment is **not** retransmitted — A's pipelined design only ever resends the single oldest unacknowledged segment, not everything outstanding.

![[Pasted image 20260623224648.png]]

**Scenario 3 — A Cumulative ACK Arriving Just Before Timeout Suppresses _Both_ Retransmissions**

Same setup as Scenario 2, but this time the ACK for the _first_ segment is lost — yet, just before the timeout event fires, A receives an ACK with value **120**. Because TCP's ACKs are cumulative, ACK=120 by itself confirms that B has received _everything_ through byte 119 — both segments — even though A never separately learned that the first one made it. A therefore retransmits **neither** segment.

> **The single biggest lesson across all three scenarios:** because TCP's acknowledgments are cumulative, a _later_ ACK can silently make an _earlier_, lost ACK irrelevant. The sender's state (`SendBase`) only ever cares about the highest cumulative ACK value seen, never about which individual ACK packets happened to survive the trip.

### Fast Retransmit: Beating the Timeout to the Punch

A genuine weakness of relying purely on timeouts is that the timeout period can be relatively long — which means a lost segment might sit unacknowledged (and unretransmitted) for a noticeably long stretch, directly inflating end-to-end delay. Fortunately, the sender can often detect loss _well before_ the timer ever fires, by watching for **duplicate ACKs**.

A **duplicate ACK** is an ACK that re-acknowledges a byte the sender has already received an acknowledgment for previously. Here's why the receiver generates one in the first place: when a TCP receiver gets a segment whose sequence number is _larger_ than the next expected, in-order byte, it has detected a gap — likely a lost or reordered segment. Since TCP only ever ACKs cumulatively, the receiver's response to this gap is simply to **re-send an ACK for the last in-order byte it has** — i.e., a duplicate of the ACK it already sent previously.

Because the sender pipelines multiple segments, there are typically several segments in flight at once. If just one of those in-flight segments is lost, the receiver will generate a _string_ of back-to-back duplicate ACKs — one for every subsequent (correctly-arriving, but out-of-order) segment that lands after the gap. TCP's rule of thumb: if the sender sees **three duplicate ACKs** for the same data (the original ACK, plus three more identical ones), it treats this as strong evidence that the segment immediately following the acknowledged data has been lost.

> Three is a threshold deliberately tuned to filter out simple network reordering — a single duplicate ACK might just mean one segment briefly arrived out of order and self-corrected, but three in a row is a much stronger and more specific loss signal.

When this **triple duplicate ACK** condition is detected, TCP performs a **fast retransmit** (RFC 5681): it retransmits the missing segment _immediately_, without waiting for that segment's timer to expire at all.

![[Pasted image 20260623224842.png]]

### Is TCP's Error-Recovery Mechanism GBN, or Selective Repeat?

This is a genuinely interesting design question, since TCP shares real features with both protocols from Section 3.4:

|Feature|Looks Like GBN|Looks Like SR|
|---|---|---|
|Tracks only the smallest unacked sequence number (`SendBase`) and the next sequence number to send (`NextSeqNum`)|✔||
|Acknowledgments are cumulative; out-of-order segments are not individually ACKed|✔||
|On a lost segment, **GBN-style behavior** would be to retransmit _that_ segment **and every subsequent segment already sent**, even ones that arrived fine|✔ (this is what GBN would do)||
|TCP's _actual_ behavior: retransmits **at most one segment** (the lost one), and won't even retransmit it if an ACK for the segment _after_ it arrives before timeout||✔ (this is much closer to SR's selective behavior)|
|The optional **selective acknowledgment** extension (RFC 2018) lets a receiver explicitly ACK out-of-order segments rather than just cumulatively||✔|

The conclusion the textbook reaches: TCP's error-recovery mechanism is best described as a **hybrid of GBN and Selective Repeat** — structurally organized like GBN (cumulative ACKs, simple sender state), but behaviorally avoiding GBN's wasteful blanket retransmission, landing much closer to SR in practice. With the selective-acknowledgment extension enabled, it leans even further toward SR.

---

## 3.5.5 Flow Control

### The Problem Flow Control Solves

Each side of a TCP connection sets aside a **receive buffer**. When correctly-sequenced bytes arrive, TCP places them into this buffer — but the associated application process doesn't necessarily read that data the instant it arrives; the application might be busy, slow, or simply not yet have gotten around to reading. If the sender keeps shipping data faster than the receiving application reads it, the receive buffer can overflow.

> **Analogy — A Sink Faster Than the Drain:** Picture water (data) pouring into a sink (the receive buffer) while the drain (the application reading from the buffer) only lets water out at its own pace. If the tap (the sender) runs faster than the drain can keep up, the sink overflows — regardless of how perfectly clean and intact the water itself is. Flow control is what makes the tap match its flow rate to the drain's, rather than to its own maximum capacity.

TCP's **flow-control service** is precisely this speed-matching mechanism: it lets the sender throttle its rate to match how fast the _receiving application_ is actually reading, eliminating the possibility of overflow purely from a speed mismatch.

> **A crucial distinction worth not blurring:** Flow control and congestion control (Section 3.7) both result in the sender being throttled, and many people use the terms loosely interchangeably — but they exist for entirely different reasons. Flow control protects the **receiver's buffer** from being overwhelmed by a fast sender. Congestion control protects the **network itself** from being overwhelmed by too much aggregate traffic. The throttling looks similar from the sender's point of view; the _cause_ being defended against is completely different.

### The Receive Window (rwnd): The Mechanism Behind Flow Control

TCP implements flow control by having the **sender** maintain a variable called the **receive window**, denoted `rwnd` — essentially, the sender's running estimate of how much free buffer space currently exists at the receiver.

Setting up the variables needed (in the context of Host A sending a large file to Host B over one TCP connection, where Host B allocates a receive buffer of size `RcvBuffer`):

|Variable|Meaning|
|---|---|
|**LastByteRead**|The number of the last byte in the data stream that B's application process has actually read out of the buffer|
|**LastByteRcvd**|The number of the last byte that has arrived from the network and been placed into B's receive buffer|

Because TCP can never be allowed to overflow the allocated buffer, the following invariant must always hold:

```
LastByteRcvd − LastByteRead ≤ RcvBuffer
```

The receive window itself is simply defined as the _spare room_ currently available in that buffer:

```
rwnd = RcvBuffer − [LastByteRcvd − LastByteRead]
```

Since the gap between bytes received and bytes read changes constantly as data arrives and the application reads it, **`rwnd` is dynamic** — it shrinks as unread data piles up, and grows back as the application catches up on reading.

![[Pasted image 20260623225149.png]]

### How the Two Sides Use rwnd Together

Host B tells Host A how much spare room it currently has by stamping the **current value of `rwnd`** into the receive-window field of _every single segment_ it sends to A — not just occasionally, but continuously, on every outgoing segment. Initially, B sets `rwnd = RcvBuffer` (the buffer starts completely empty, so it's entirely "spare").

Host A, meanwhile, tracks two of its own variables:

|Variable|Meaning|
|---|---|
|**LastByteSent**|The number of the last byte A has sent into the connection so far|
|**LastByteAcked**|The number of the last byte A has received an acknowledgment for|

The difference `LastByteSent − LastByteAcked` is exactly the amount of data A has sent but not yet had confirmed — A's currently "unacknowledged, in-flight" data. By keeping this quantity always **less than or equal to** the most recently advertised `rwnd`, A guarantees it never overflows B's receive buffer:

```
LastByteSent − LastByteAcked ≤ rwnd
```

### The Zero-Window Deadlock — and TCP's Fix

There's one subtle trap baked into this scheme. Suppose B's receive buffer fills up completely, so `rwnd` drops to **0**, and B advertises `rwnd = 0` to A. Now suppose B also currently has nothing of its own to send to A. As B's application process gradually empties the buffer over time, B's transport layer has no _new outgoing segment_ to attach an updated, nonzero `rwnd` value to — and TCP only sends a segment to A when it has data to send or an acknowledgment to give. The result: **A is never informed that space has opened back up**, and is permanently stuck, blocked from sending anything more, even though room genuinely exists again at B.

> This is a real deadlock risk hiding inside an otherwise sensible design — both sides are technically behaving correctly by their own local rules, yet the connection stalls indefinitely.

TCP's specified fix: whenever Host A is told `rwnd = 0`, it must continue sending segments carrying **just one byte of data** at a time, indefinitely, even though it's technically "blocked." B's TCP will acknowledge these tiny probing segments — and as the receive buffer empties over time, eventually one of those acknowledgments will carry a fresh, **nonzero** `rwnd` value, unsticking A and letting normal-sized transmission resume.

> **Analogy — Knocking Periodically on a Closed Door:** If you're told "the office is full right now, don't come in," you don't necessarily find out the _moment_ a seat frees up — unless you keep knocking periodically to check. TCP's one-byte probe segments are exactly that periodic knock: small, low-cost check-ins that let A discover the instant B has room again, rather than waiting for B to proactively volunteer the news (which, as shown above, B has no built-in trigger to do).

### A Brief Contrast: UDP Has No Flow Control At All

Having now fully described TCP's flow-control service, it's worth noting by contrast that **UDP provides no flow control whatsoever** — and as a direct consequence, UDP segments absolutely can be (and are) lost purely due to **receiver-side buffer overflow**, completely independent of anything happening in the network.

A typical UDP implementation appends arriving segments into a finite-sized buffer that sits in front of the corresponding socket — the process reads one whole segment at a time out of that buffer. If the receiving process doesn't read fast enough, the buffer fills up, and any further arriving segments are simply dropped, with no mechanism on either side to detect or prevent it. This is the direct cost UDP applications pay for skipping all of TCP's bookkeeping overhead: speed and simplicity for the sender, but zero protection against a slow reader on the receiving end.

---

## 3.5.6 TCP Connection Management

This closing piece of the TCP picture looks at how a connection actually gets **established and torn down** — easy to gloss over as bureaucratic plumbing, but it matters for two concrete reasons: connection setup adds directly to perceived delay (every time you load a page over a fresh connection, you're paying this cost), and the setup procedure is the exact mechanism exploited by one of the most common denial-of-service attacks on the Internet, the SYN flood.

### Establishing a Connection: Three Steps, Three Segments

Suppose a client process wants to connect to a server process. The client's TCP and the server's TCP carry out the following:

|Step|What Happens|
|---|---|
|**Step 1 — Client sends SYN**|The client-side TCP sends a special segment with **no application-layer data**, but with the **SYN bit** set to 1 in the header (hence "SYN segment"). The client also randomly chooses an **initial sequence number, `client_isn`**, and places it in the segment's sequence number field. This segment is encapsulated in an IP datagram and sent to the server.|
|**Step 2 — Server sends SYNACK**|Once the datagram arrives, the server extracts the SYN segment, **allocates the TCP buffers and variables for the connection**, and sends back a connection-granted segment — also carrying no application data, but containing three pieces of information: the SYN bit set to 1, the acknowledgment field set to `client_isn + 1`, and the server's own randomly-chosen initial sequence number, `server_isn`, in the sequence number field. This is the **SYNACK segment**.|
|**Step 3 — Client sends ACK**|Upon receiving the SYNACK, the client **also allocates its own buffers and variables** for the connection, then sends one final segment acknowledging the server's SYNACK — putting `server_isn + 1` in the acknowledgment field. The SYN bit is now set to **0**, since the connection is officially established. This third segment **may carry the client's first chunk of application payload.**|

> **Why randomize `client_isn` and `server_isn`?** Exactly the same reasoning given earlier for ISN randomization in general (Section 3.5.2) applies here with extra force: there has been considerable dedicated interest in properly randomizing the client's ISN specifically to defend against certain security attacks (RFC 4987) — predictable ISNs are a foothold for connection-hijacking and injection attempts.

```
   CLIENT                                          SERVER
   ──────                                          ──────
        │── SYN=1, seq=client_isn ──────────────────►│  (Step 1: connection request)
        │                                            │
        │◄── SYN=1, seq=server_isn, ───────────────────┤  (Step 2: connection granted —
        │       ack=client_isn+1                      │   server allocates buffers/vars
        │                                            │   *before* this 3rd step completes)
        │── SYN=0, seq=client_isn+1, ────────────────►│  (Step 3: ACK — client allocates
        │       ack=server_isn+1                      │   buffers/vars; may carry payload)
        ▼                                            ▼
                  Connection ESTABLISHED
```

Because exactly three segments cross the wire — SYN, SYNACK, ACK — this is the same **three-way handshake** introduced informally back in 3.5.1, now fully specified bit-by-bit.

> **Analogy — The Climber and the Belayer:** A rock climber and their belayer (the person managing the safety rope from below) use a verbal three-way exchange before the climb begins — "On belay?" / "Belay on." / "Climbing." — structurally identical to TCP's handshake: each side confirms readiness _and_ receives confirmation back before committing to the activity. Neither party wants to act on an assumption the other side never actually confirmed.

### The Cost of the Handshake: One RTT Before Any Data

Because the client has to wait to receive the SYNACK before it knows the server has accepted the connection and initialized its state, **a full RTT delay is structurally built into ordinary connection setup** — unavoidable with this protocol, no matter how fast either host is individually.

### TCP Fast Open: Skipping the Handshake When Possible

If a client and server have communicated before, a technique called **fast open** (also called **0-RTT handshaking**) can reduce the handshake delay to _zero_. During an earlier, ordinary three-way handshake, the client can request a **fast-open cookie** from the server — an encoding of all the connection information needed for a future connection, saved at both client and server. The next time that client wants to reconnect, it sends the cookie _together with its application data_ in its very first message. If the server finds the cookie acceptable, it establishes the connection and responds with application-layer data immediately — completely skipping the RTT that a traditional handshake would have spent before any data could move.

|Detail|Note|
|---|---|
|**Client-side benefit**|Fast open is purely a potential optimization for the client — it costs the client nothing to offer the cookie|
|**Server discretion**|The server may decline to accept the cookie for policy reasons — e.g., it no longer wants to talk to that client, or wants to force re-authentication (as in TLS handshaking)|
|**Cookie integrity**|The cookie must be generated using cryptographically secure techniques — otherwise it becomes a forgeable shortcut around the handshake's protections|
|**Where else this shows up**|TLS (RFC 8446) and QUIC (RFC 9000, RFC 9002) also support their own zero-RTT fast-open mechanisms — this isn't a TCP-only idea, just introduced here first|

### Tearing Down a Connection

All good things come to an end, and either side of a TCP connection can independently initiate closing it. Suppose the **client** decides to close:

1. The client application issues a close command, causing the client TCP to send a special segment with the **FIN bit** set to 1.
2. The server, on receiving this FIN segment, sends back an acknowledgment.
3. The server then sends its **own** shutdown segment, also with the FIN bit set to 1.
4. The client acknowledges the server's FIN.
5. At this point, all resources (buffers and variables) on **both** hosts are deallocated, and the connection ceases to exist.

```
   CLIENT                                          SERVER
   ──────                                          ──────
  Close
        │── FIN ─────────────────────────────────────►│
        │◄── ACK ───────────────────────────────────────┤
        │                                              │ Close
        │◄── FIN ───────────────────────────────────────┤
        │── ACK ──────────────────────────────────────►│
        │  (Timed wait)                                │
   Closed                                          Closed
```

### Watching the State Machine: How TCP Actually Tracks This

Throughout a connection's life, the TCP protocol running on each host moves through a defined sequence of **TCP states**. Two separate state diagrams describe this — one for the side that initiates the connection (typically the client), one for the side that listens for it (typically the server).

**Client-side states:**

```
   CLOSED ──Send SYN──► SYN_SENT ──Recv SYN & ACK, send ACK──► ESTABLISHED
                                                                     │
                                                              Send FIN (app
                                                              decides to close)
                                                                     ▼
                                                              FIN_WAIT_1
                                                                     │
                                                      Recv ACK, send nothing
                                                                     ▼
                                                              FIN_WAIT_2
                                                                     │
                                                          Recv FIN, send ACK
                                                                     ▼
   CLOSED ◄── Wait 30 seconds ── TIME_WAIT
```

|State|What's Happening|
|---|---|
|**CLOSED**|Starting point — no connection exists yet|
|**SYN_SENT**|Client has sent its SYN and is waiting for the server's SYNACK|
|**ESTABLISHED**|Handshake complete; the client can freely send and receive payload-carrying segments|
|**FIN_WAIT_1**|Client has sent its own FIN and is waiting for an ack/FIN from the server|
|**FIN_WAIT_2**|Client's FIN has been acknowledged; now waiting for the server's own FIN|
|**TIME_WAIT**|Client has ACKed the server's FIN and resends that final ACK if it gets lost, before fully closing|

> **Why linger in TIME_WAIT instead of closing immediately?** The whole point is insurance: if the client's very last ACK never reaches the server, the server will retransmit its FIN, and the client needs to still be around — holding just enough state — to resend the ACK in response. Closing instantly would leave the server's retransmitted FIN with nowhere valid to land. The exact wait duration is **implementation-dependent**, but typical values are 30 seconds, 1 minute, or 2 minutes. Only after the wait does the connection formally close and release all client-side resources, including port numbers.

**Server-side states** follow a mirrored but distinct path:

```
   CLOSED ──Create listen socket──► LISTEN ──Recv SYN, send SYN & ACK──► SYN_RCVD
                                                                              │
                                                                  Recv ACK, send nothing
                                                                              ▼
                                                                       ESTABLISHED
                                                                              │
                                                                        Recv FIN, send ACK
                                                                              ▼
                                                                       CLOSE_WAIT
                                                                              │
                                                                          Send FIN
                                                                              ▼
   CLOSED ◄── Receive ACK, send nothing ── LAST_ACK
```

> Both diagrams "only show how a TCP connection is normally established and shut down" — pathological edge cases, like both sides trying to initiate or close _simultaneously_, exist but go beyond this introductory pass (see Stevens 1994 for the full depth).

### Reset Segments: When a Segment Has No Home

The discussion so far has assumed both sides are prepared to communicate — the server is actually listening on the port the client is reaching for. What happens when that's _not_ true?

Suppose a host receives a TCP SYN packet addressed to a port it isn't accepting connections on (no web server running on port 80, for instance, yet a SYN arrives for port 80 anyway). The host responds with a special **reset segment** — a TCP segment with the **RST flag bit** set to 1. Sending a reset segment is the host's way of explicitly telling the source, _"I don't have a socket for that segment — please don't resend it."_

> The UDP equivalent of this situation produces a different response entirely: when a host receives a UDP packet whose destination port doesn't match any ongoing UDP socket, it sends a special **ICMP datagram** instead (the mechanics of ICMP are covered later, in Chapter 5) — TCP and UDP each have their own distinct way of saying "nobody's home" at a given port.

---

## Focus on Security: The SYN Flood Attack

The three-way handshake has an exploitable asymmetry baked into it: **the server allocates and initializes connection buffers and variables the moment it receives a SYN — before it has any confirmation the client is legitimate or will ever complete the handshake.** If the client never sends the final ACK, the server eventually (often after a minute or more) times out the **half-open connection** and reclaims those resources — but in the meantime, real capacity sat reserved for a connection that was never going to complete.

The **SYN flood attack** weaponizes exactly this asymmetry: an attacker sends a large volume of SYN segments without ever completing the third handshake step. The server's pool of connection resources gets exhausted allocating (but never using) half-open connections, and legitimate clients attempting to connect get denied service — a classic **Denial-of-Service (DoS)** attack. SYN flooding was among the very first documented DoS attack types, and even decades later it still accounts for a meaningful share of DoS activity in the wild (cited at 14% of DoS attacks in recent measurement, per Fastly 2024).

### The Defense: SYN Cookies

The standard, now widely deployed defense is **SYN cookies** (RFC 4987). The core trick is to make the server stop trusting — and stop _remembering_ — anything about a SYN until the client proves it's real by completing the handshake:

|Step|What the Server Does|
|---|---|
|**On receiving a SYN**|Instead of creating real connection state, the server computes an initial sequence number as a **hash function** of the source/destination IP addresses, source/destination ports, _and_ a secret value known only to the server. This specially-crafted value is the **"cookie."** The server sends a SYNACK carrying this cookie as its ISN — and **deliberately keeps no memory** of the SYN or any related state.|
|**On receiving a legitimate client's ACK**|The server must verify the ACK corresponds to a real, previously-sent SYNACK — without having stored anything about it. Since a legitimate ACK's acknowledgment field equals (SYNACK's ISN) + 1, the server simply **re-runs the same hash function** using the IP addresses, ports, and secret (all of which are still available, since the source/destination fields are right there in the incoming segment) and checks whether the result, plus one, matches the ACK's acknowledgment value. A match confirms validity; **only then** does the server allocate a real, fully open connection and socket.|
|**If no ACK ever arrives**|No harm done — because the server never allocated any resources for the original SYN in the first place, an attacker's flood of never-completed SYNs costs the server essentially nothing beyond computing cheap hash functions.|

> **Analogy — A Claim Ticket Instead of a Reserved Table:** Imagine a restaurant that, instead of immediately reserving and setting a table the moment someone calls to book, simply hands out a verifiable claim ticket (encoding who called and when) without committing any actual table or staff time. Only when the person shows up _with a valid ticket_ does the restaurant commit real resources. A prankster who calls repeatedly and never shows up costs the restaurant nothing — there was never anything to reclaim, because nothing was ever set aside.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Sequence numbers identify byte position, and are randomized at connection start**|If initial sequence numbers were predictable (e.g., simply incrementing), an attacker who can guess or observe them could inject forged segments into an active connection, or replay a stray segment from an already-dead connection as if it belonged to a new one|Modern TCP stacks generate ISNs using strong randomization specifically to make this kind of injection/replay infeasible; never assume a four-tuple match alone proves segment authenticity|
|**TCP trusts ACKs and sequence numbers, not cryptographic identity**|An attacker positioned to sniff or guess the four-tuple plus current sequence numbers of an active connection can attempt session hijacking, since TCP's own mechanism has no concept of "proving who sent this"|Run TLS on top of TCP for genuine endpoint authentication and payload confidentiality; treat raw TCP framing as providing _delivery_, never _trust_|
|**Telnet sends everything — including credentials — in plaintext**|Anyone able to observe traffic on the path can read login credentials and session content directly out of Telnet segments|Use SSH instead of Telnet for any real remote-login use case; this is precisely why SSH displaced Telnet in practice|
|**Single shared retransmission timer, predictable backoff (doubling)**|A well-resourced attacker who can selectively drop ACKs might attempt to manipulate a connection's effective throughput by forcing repeated retransmission/backoff cycles|Not generally a high-value attack surface on its own, but illustrates why TCP's timing behavior (RTT estimation, backoff) is occasionally relevant in denial-of-service and traffic-analysis discussions|
|**The handshake allocates server resources before the client is verified**|A SYN flood exploits exactly this gap — flooding half-open connection requests to exhaust server resources without ever completing a single real connection|Deploy SYN cookies so the server commits zero state until a verified ACK arrives, making half-open-connection floods essentially costless to absorb|
|**Connection-establishment behavior reveals whether a port is listening at all**|An RST response to an unsolicited SYN, or an ICMP message to a stray UDP packet, both leak information about which ports are actually active — useful raw material for the port scanning described in Section 3.2|Don't assume silence is the only safe response; understand that explicit rejection (RST/ICMP) is itself a small information leak, and is part of why minimizing exposed ports matters at the network-design level, not just the firewall-rule level|

---

## Questions I Still Have

- [ ] The textbook describes TCP retransmitting "at most one segment" on a triple-duplicate-ACK fast retransmit — does this mean fast retransmit and a subsequent timeout for a _different_ segment can never both fire for the same loss event, or can they interact in edge cases?
- [ ] With selective acknowledgments (RFC 2018) enabled, how much does TCP's behavior actually converge toward "true" Selective Repeat versus staying a GBN-style hybrid in terms of sender-side state complexity?
- [ ] The one-byte probe-segment fix for the zero-window deadlock — is there a defined interval for how often A sends these probes, or is it left to implementation, similar to how the spec leaves the timing of _when_ TCP sends buffered data unspecified?
- [ ] How does window scaling (mentioned briefly under the Options field) interact with the 16-bit `rwnd` field, given that 16 bits alone would cap advertisable window size well below what high-speed/high-RTT links need?
- [ ] What actually happens in the "pathological" cases the textbook waves off — both sides trying to initiate, or both sides trying to close, at the same time? Does the state machine have defined transitions for simultaneous SYN or simultaneous FIN, or does it just rely on one side "winning" by timing?
- [ ] With TCP Fast Open's cookie skipping the handshake, does that also mean the usual one-RTT delay _and_ the usual ISN-randomization protection are both being traded away for speed — or does the cookie itself still encode enough randomness to preserve that protection?
- [ ] SYN cookies replace the server's normal ISN choice with a hash-derived value — does this interact at all with the security rationale for randomized ISNs (RFC 4987), or are the two mechanisms solving genuinely separate problems that happen to both live in the sequence number field?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Three-way handshake**|The three-segment exchange (request → response → ack/first-data) by which a TCP client and server establish a connection before any bulk data transfer begins|
|**Full-duplex**|A connection property where data flows in both directions simultaneously|
|**Point-to-point**|A connection property restricting TCP to exactly one sender and one receiver per connection (no multicast)|
|**MTU**|Maximum Transmission Unit — the largest link-layer frame a host's local network can carry|
|**MSS**|Maximum Segment Size — the largest chunk of application data TCP places in one segment, sized so MSS + header fits inside one MTU|
|**Sequence number**|The byte-stream position of the first data byte in a given TCP segment|
|**Acknowledgment number**|The sequence number of the next byte the sender of that segment is expecting to receive|
|**Cumulative acknowledgment**|TCP's ACK behavior of always naming the next byte still needed to make the stream contiguous, regardless of any later, out-of-order bytes already received|
|**SampleRTT**|A single measured round-trip time for one (non-retransmitted) segment-ACK pair|
|**EstimatedRTT**|An exponentially-weighted moving average of SampleRTT values, smoothing out short-term fluctuation|
|**DevRTT**|An exponentially-weighted moving average of how far SampleRTT typically deviates from EstimatedRTT — TCP's measure of RTT variability|
|**TimeoutInterval**|TCP's actual retransmission timeout, computed as `EstimatedRTT + 4 · DevRTT`|
|**SendBase**|Sender-side state variable: the sequence number of the oldest byte sent but not yet acknowledged|
|**NextSeqNum**|Sender-side state variable: the sequence number to be assigned to the next new segment of data|
|**Duplicate ACK**|An ACK that re-acknowledges a byte already confirmed by an earlier ACK; generated by a receiver detecting a gap in the byte stream|
|**Fast retransmit**|Retransmitting a presumed-lost segment immediately upon receiving three duplicate ACKs for it, without waiting for the timeout|
|**Flow control**|TCP's mechanism for preventing a fast sender from overflowing a slow receiver's buffer, by having the sender respect the receiver's advertised window|
|**Receive window (rwnd)**|The amount of free space currently available in the receiver's buffer, advertised by the receiver and respected by the sender|
|**RcvBuffer**|The total size of the buffer a host allocates for an incoming TCP connection's data|
|**LastByteRead / LastByteRcvd**|Receiver-side variables tracking, respectively, the last byte read by the application and the last byte placed in the buffer by the network|
|**LastByteSent / LastByteAcked**|Sender-side variables tracking, respectively, the last byte sent into the connection and the last byte acknowledged by the receiver|
|**Zero-window probe**|A one-byte segment TCP keeps sending when told `rwnd = 0`, specifically to detect when receiver buffer space reopens|
|**SYN segment**|The first handshake segment — no payload, SYN bit set to 1, carries the client's randomly-chosen `client_isn`|
|**SYNACK segment**|The server's handshake reply — SYN bit set to 1, acknowledges `client_isn + 1`, carries the server's own randomly-chosen `server_isn`|
|**TCP Fast Open / 0-RTT**|A cookie-based technique letting a returning client skip the handshake delay entirely by sending application data with its first message|
|**FIN segment**|A segment with the FIN bit set to 1, signaling one side's intent to close its half of the connection|
|**TIME_WAIT state**|A client-side state held briefly after closing, to allow resending a final ACK if the server's FIN gets retransmitted due to ACK loss|
|**RST segment**|A segment with the RST bit set to 1, sent when a host receives a segment for a port with no matching socket — "please don't resend this"|
|**Half-open connection**|A connection for which the server has allocated resources after receiving a SYN, but for which the third handshake step (ACK) has not yet arrived|
|**SYN flood attack**|A DoS attack that floods a server with SYNs and never completes the handshake, exhausting resources reserved for half-open connections|
|**SYN cookie**|A defense against SYN flooding: the server encodes connection info into a hash-derived ISN and verifies it on ACK receipt, without storing any state up front|

---

## Related Concepts

---

→ Next: [[3.6 Principles of Congestion Control]]