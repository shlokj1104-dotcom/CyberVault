---
title: TRANSPORT LAYER FUNCTIONALITY
date: 2026-06-27
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 3.8 Evolution of Transport-Layer Functionality

> **One-Line Summary:** Four decades of running everything through just two transport protocols — UDP and TCP — has exposed real gaps neither was built to fill, and the response has been two waves of evolution: first hybrid designs like DCCP that borrow UDP's lightness but plug in TCP-compatible congestion control, and then something far more dramatic — QUIC, which rebuilds TCP-like reliability, congestion control, and secure connection management _inside the application layer_, running over plain UDP, specifically so it can evolve at application speed instead of waiting on operating-system and standards-committee timescales.

---

## Core Idea: Two Workhorses, Neither Built for Everything

This chapter's deep dive into Internet transport protocols has centered on UDP and TCP — fairly described as the two "workhorses" of the Internet transport layer. But across four decades of real deployment experience, plenty of situations have surfaced where **neither protocol is ideally suited**: TCP's reliability and connection-orientation are sometimes more than an application wants to pay for, while UDP's complete absence of congestion control can make it an unsafe citizen on a shared network. The design and implementation of transport-layer functionality has kept evolving in response — and this chapter has already touched on some of that evolution directly: **TCP Fast Open** (Section 3.5.6) trimmed the handshake delay, and the newer congestion-control flavors — **CUBIC** and **BBR** (Section 3.7.2) — rethought how a sender should infer and react to congestion. This section looks at two further, larger steps in that same evolutionary story.

---

## DCCP: The Datagram Congestion Control Protocol

**DCCP** (RFC 4340) is best understood as a deliberate middle ground between UDP and TCP: it provides a **low-overhead, message-oriented, UDP-like unreliable service** — so, like UDP, no automatic retransmission or ordering guarantees — but pairs that with an **application-selected form of congestion control that's compatible with TCP's own.**

> **Analogy — Choosing Your Own Seatbelt, Not Your Own Engine:** UDP is like driving with no seatbelt at all — fast and unencumbered, but offering nothing if things go wrong for anyone else on the road. TCP is like a car that comes with one fixed, non-negotiable safety system built in. DCCP is closer to a car where you keep UDP's light, fast "engine" (no forced reliability), but you're required to pick _some_ recognized seatbelt system before you're allowed to merge onto the highway — the application gets to choose its congestion-control behavior, but it can't opt out of playing fair with everyone else's traffic.

### TFRC: One of DCCP's Selectable Congestion-Control Modes

One of the congestion-control options an application can select under DCCP is the **TCP-Friendly Rate Control (TFRC) protocol** (RFC 5348). TFRC's specific job is to **smooth out the "saw tooth" behavior** that's characteristic of classic TCP congestion control (recall the AIMD sawtooth from Section 3.7.1's Figure 3.50), while still keeping a long-term sending rate that stays _reasonably close_ to what TCP itself would achieve under the same conditions.

|Property|Detail|
|---|---|
|**Smoother sending rate than TCP**|Avoids the sharp halve-then-climb pattern of classic AIMD, instead targeting a steadier rate over time|
|**"Equation-based" design**|TFRC measures the connection's actual **packet loss rate**, then feeds that measured rate as input to an equation that _estimates_ what TCP's throughput would be if a real TCP connection experienced that same loss rate [Padhye 2000]|
|**Target sending rate**|The output of that equation becomes TFRC's own target sending rate — in effect, "send at the rate TCP _would_ send, just without TCP's jagged ups and downs"|

TFRC is specifically envisioned for applications like **streaming media**, which can tolerate exploiting the tradeoff between strict reliability and timeliness (a late-arriving video frame is often worse than a slightly imperfect one), but which still need to remain genuinely **responsive to network congestion** rather than ignoring it the way raw UDP traffic does. If an application using DCCP needs reliable or semi-reliable data delivery on top of this, that has to be built by the application itself, using the same kinds of mechanisms studied back in Section 3.4 — DCCP doesn't provide that piece natively.

---

## The Bigger Shift: Transport Functionality Migrates Into the Application Layer

Beyond incremental protocols like DCCP, the transport layer has witnessed an even more dramatic change over roughly the past decade: the **migration of TCP's reliable data transfer, flow control, congestion control, and other services into the _application_ layer**, implemented as application-level modules that provide TCP-like services running on top of plain UDP.

The most well-known example of this trend is **QUIC** (Quick UDP Internet Connections) — already introduced earlier in the context of HTTP/3 (Section 2.2.7) — which this section now examines in more depth.

### A Genuinely New Design Choice for Application Builders

Today, an application designer choosing a transport service still has the two traditional options baked into the operating system — native TCP, or native UDP. But there's now a **real third choice**: an application designer can effectively **"roll their own" transport-layer service**, built in the application layer, running on top of UDP, using QUIC.

> The authors of an earlier version of QUIC have noted that implementing transport-layer functions _in the application layer_ means changes to QUIC can happen on "application-update timescales" — that is, with far greater feature velocity than waiting on the comparatively glacial pace of operating-system-level TCP or UDP updates. RFC 1263 made essentially the same observation, decades earlier, with a memorable bit of humor: that the real advantage of building protocols outside a formal standards process is being able to design and ship a new one in _less time_ than it would take a standards committee just to agree on a meeting time to discuss it.

This is, structurally, the same broader lesson Section 3.5.6 already hinted at with TCP Fast Open's adoption of 0-RTT handshaking, and the same lesson echoed by TLS and QUIC both independently developing their own 0-RTT techniques: **protocol innovation increasingly happens above the traditionally fixed transport layer**, precisely because that layer is otherwise so slow to change.

---

## QUIC: A Close-Up Look

QUIC (RFC 9000, RFC 9002) serves as a genuinely fitting capstone to this chapter's study of the transport layer, because it directly draws on — and recombines — nearly every major idea covered so far: reliable data transfer, congestion control, and connection management, all reimplemented together in a single modern protocol.

### Feature 1 — Connection-Oriented and Secure

Like TCP, QUIC is a **connection-oriented** protocol between two endpoints, requiring a handshake to establish connection state before data flows. Two specific pieces of that connection state are a **source connection ID** and a **destination connection ID**, both carried in every QUIC packet header.

The standout difference from TCP: **every single QUIC packet is encrypted.** QUIC achieves this by deliberately **combining** the handshake needed to establish connection state with the handshake needed for authentication and encryption (the transport-layer security, or **TLS**, concepts covered in depth later, in Chapter 8) — folding what would otherwise be _two separate_ handshakes into one.

> **Why combining handshakes actually matters for speed:** Establishing a secure HTTP connection the traditional way means first completing a TCP three-way handshake (Section 3.5.6), and _only then_ layering a separate TLS handshake on top of that already-established TCP connection — two sequential round trips of setup delay stacked on top of each other before any real application data moves. QUIC's combined handshake achieves the same end state — an authenticated, encrypted, connection-state-initialized channel — in meaningfully fewer round trips, precisely because it never treats "connection setup" and "security setup" as two separate problems to be solved one after the other.

#### Worked Example: Counting the RTTs

It's worth actually walking through both setups step by step, since "fewer round trips" is easy to state and easy to skim past.

**The traditional way — TCP, then TLS, as two separate, sequential handshakes:**

```
Step 1: TCP Handshake
  Client                          Server
  SYN          ───────────────►
               ◄───────────────   SYN + ACK
  ACK          ───────────────►

  TCP connection established. This already costs 1 RTT.

Step 2: TLS Handshake (only begins now, on top of the TCP connection)
  Client                          Server
  Hello        ───────────────►
               ◄───────────────   Certificate
  Key Exchange ───────────────►
               ◄───────────────   Finished

  During this handshake, the server authenticates itself and
  encryption keys get negotiated — but this is a SEPARATE
  round of back-and-forth, layered ON TOP of the already-
  completed TCP handshake above.

Total:  TCP Handshake  +  TLS Handshake  =  multiple RTTs
        Only after BOTH are fully complete can application
        data (the actual HTTP request) finally be sent.
```

**What QUIC does instead — refusing to treat these as two separate problems:**

QUIC's reasoning, in effect: _"Why perform two separate handshakes when one combined handshake can establish both connection state and security at the same time?"_

```
QUIC Handshake
  Client                                   Server
  Hello + TLS Info     ──────────────────►
                        ◄──────────────────   Certificate +
                                               Encryption Keys
  Encrypted Data        ──────────────────►

  Notice what happens SIMULTANEOUSLY in this one exchange:
    • The QUIC connection itself is established
    • The server authenticates itself
    • Encryption keys are negotiated
  Everything happens together, in one combined round of
  back-and-forth, rather than two stacked one after another.
```

The savings aren't about either handshake individually being "faster" — TLS still needs to verify a certificate, QUIC still needs to set up connection IDs. The savings come purely from **refusing to make the second handshake wait for the first one to fully finish** before it can even begin.

### Feature 2 — Streams

QUIC allows several different application-level **"streams"** to be multiplexed through a single QUIC connection. A **stream** is an abstraction representing reliable, in-order, bidirectional delivery of data between two QUIC endpoints — but it operates _underneath_ the level of the whole connection.

|Identifier|Scope|
|---|---|
|**Connection ID**|Identifies the QUIC connection as a whole|
|**Stream ID**|Identifies one particular stream _within_ a connection — a connection can carry many streams simultaneously|

Both IDs travel together inside the QUIC packet header (along with other header information). Data from _multiple different streams_ may be packed together within a single QUIC segment, which is itself carried inside one UDP packet.

> **A useful historical precedent:** The **Stream Control Transmission Protocol (SCTP)** — an earlier reliable, message-oriented protocol (RFC 4960, RFC 3286) — actually pioneered this exact idea of multiplexing several distinct application-level "streams" through one single connection, well before QUIC existed. QUIC's stream abstraction isn't a brand-new invention so much as a modernized, UDP-based revival of an idea that had already been worked out once before.

#### Worked Example: Loading a Webpage

Suppose you visit `www.example.com`. To render the page, the browser needs to download several distinct things:

```
HTML
CSS
JavaScript
Logo.png
Video.mp4
```

Under HTTP/3 (which uses QUIC), the browser might map each of these onto its own independent stream:

```
Stream 1 → HTML
Stream 2 → CSS
Stream 3 → JavaScript
Stream 4 → Logo
Stream 5 → Video
```

**All five of these streams travel over just one single QUIC connection** — one connection ID, one combined handshake already paid for — but each stream is tracked, acknowledged, and (if needed) retransmitted **completely independently** of the other four.

#### Worked Example: Can One QUIC Packet Hold Data From Multiple Streams?

**Yes.** This is exactly what's meant by the earlier statement that data from multiple streams may be contained within a single QUIC segment. Concretely:

```
   QUIC Packet
  ┌──────────────────────────────────────────┐
  │ Stream 1 : HTML  (300 bytes)              │
  │ Stream 2 : CSS   (150 bytes)               │
  │ Stream 4 : Image (500 bytes)               │
  └──────────────────────────────────────────┘
```

This **one UDP packet** is carrying data belonging to **three different streams** at once — notice:

- The HTML bytes belong to **Stream 1**
- The CSS bytes belong to **Stream 2**
- The image bytes belong to **Stream 4**

Each chunk still carries its own stream ID, so even though they all rode inside the very same packet, the receiving QUIC endpoint can sort them right back out to their three separate streams — which is exactly what makes the next feature (per-stream reliable delivery) possible in the first place.

### Feature 3 — Reliable, TCP-Friendly, Congestion-Controlled Data Transfer

This is where QUIC's streams payoff becomes concrete, and where the contrast with ordinary TCP-based HTTP is sharpest.

**Figure 3.57(a) — HTTP/1.1 over TLS over TCP:** Multiple HTTP requests are sent over a _single_ TCP connection. Because TCP guarantees reliable, **in-order** byte delivery, this means _all_ of the multiple HTTP requests/responses must be delivered to the application strictly in order — if bytes belonging to just one early HTTP exchange are lost, every _later_ HTTP request sharing that same TCP connection is stuck waiting, unable to be delivered to the destination HTTP server until those specific lost bytes are retransmitted and correctly received. This is exactly the **head-of-line (HOL) blocking problem** already encountered back in Section 2.2.5 — one lost segment effectively stalls everything queued behind it, even data that has nothing to do with the actual loss.

**Figure 3.57(b) — HTTP/3 using QUIC:** QUIC instead provides reliable, in-order delivery **separately, on a per-stream basis.** Because of this, a lost UDP segment only affects whichever _individual stream(s)_ had their data carried inside that specific segment — HTTP messages traveling on every _other_ stream can continue arriving and being delivered to the application completely undisturbed, even while the affected stream is still waiting on its own retransmission.

#### Worked Example: Watching HOL Blocking Actually Happen

Continuing the same webpage example from Feature 2 — same HTML, CSS, and Image data — here's what TCP's single byte stream looks like underneath, with everything concatenated together in one continuous sequence:

```
TCP Byte Stream
─────────────────────────────────────────────
| HTML bytes | CSS bytes | Image bytes |
─────────────────────────────────────────────
```

Suppose the packets carrying this stream go out like this:

```
Packet 1 → HTML (bytes 1–1000)        ✓ arrives fine
Packet 2 → HTML (bytes 1001–2000)     ✗ LOST
Packet 3 → CSS                         ✓ arrives fine
Packet 4 → Image                       ✓ arrives fine
```

Even though packets 3 and 4 physically arrived intact, here's what the **receiver** actually has on hand:

```
✓ HTML (Part 1)
✗ HTML (Part 2) — Missing
✓ CSS
✓ Image
```

**Can TCP deliver the CSS and Image data to the application yet? No.** TCP's whole guarantee is **reliable, in-order byte delivery** — it sees one single, continuous byte stream, not three separate files:

```
Byte Stream (as TCP sees it)
  1000 bytes          ✓
  next 1000 bytes     ✗ Missing
  CSS bytes           ✓ (physically arrived, but stuck behind the gap)
  Image bytes         ✓ (physically arrived, but stuck behind the gap)
```

Because bytes are missing in the _middle_ of that one stream, TCP **cannot deliver any of the later bytes up to the application** — not the CSS, not the Image — even though both arrived perfectly fine. TCP must wait until Packet 2 is retransmitted and successfully received; only then does it deliver the CSS and Image bytes that had been sitting there, fully intact, the entire time. This is head-of-line blocking made concrete: the CSS and Image data aren't broken, they're just **stuck in line** behind a gap that has nothing to do with them.

**Under QUIC, the same scenario plays out very differently:** because HTML, CSS, and the Image each ride on their _own_ stream (Streams 1, 2, and 4 from the earlier example), a lost packet carrying part of Stream 1's HTML data only blocks _Stream 1_. Stream 2 (CSS) and Stream 4 (Image) have no dependency on Stream 1's missing bytes, so QUIC delivers them to the application **immediately** — no waiting on someone else's retransmission.

![[Pasted image 20260627135902.png]]

> **Analogy — One Shared Checkout Line vs. Several Independent Ones:** TCP-over-one-connection is like every customer in a store being forced through a single checkout line — if the person at the front of the line has a problem with their order, _everyone_ behind them is stuck, no matter how unrelated their own purchase is. QUIC's per-stream delivery is like opening several independent checkout lines: a holdup at register 1 doesn't slow down anyone currently being served at registers 2 or 3.

Underneath this, QUIC provides its reliable data transfer using acknowledgment mechanisms genuinely similar to TCP's own (as specified in RFC 5681) — this isn't a wholly novel reliability scheme, just TCP's well-tested approach reapplied per-stream rather than connection-wide.

### QUIC's Congestion Control: A Direct Descendant of TCP's

QUIC's congestion control is based on **TCP NewReno** (RFC 6582) — a slight modification of the classic TCP Reno protocol studied earlier in this chapter. QUIC's own specification for loss detection and congestion control (RFC 9002) explicitly acknowledges this lineage: readers already comfortable with TCP's loss-detection and congestion-control mechanisms will recognize closely parallel algorithms in QUIC's own specification, deliberately mirroring well-established TCP approaches rather than reinventing them from scratch.

> This is, in a very real sense, the payoff for everything studied across Section 3.7: having carefully worked through classic TCP's slow start, congestion avoidance, fast recovery, and AIMD sawtooth behavior already means having a genuine, working head start on understanding exactly how QUIC's own congestion-control algorithm operates underneath HTTP/3.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**TCP's handshake and header fields travel in plaintext, even when TLS is layered on top**|A network observer or on-path middlebox can see and potentially interfere with raw TCP control information (sequence numbers, flags, connection state) even before any TLS-protected payload begins — this is exactly the kind of visibility that made RST injection and other on-path interference possible against classic TCP (Section 3.5.6)|QUIC's design, where essentially every packet is encrypted from the very first connection-establishing exchange onward, shrinks this visible attack surface dramatically compared to a separate, later-arriving TLS layer bolted onto an already-plaintext TCP handshake|
|**Separately sequenced TCP-then-TLS handshakes mean two distinct setup phases an attacker could try to interfere with**|More steps, more round trips, and more separately-observable setup phases can mean more individual opportunities for an on-path attacker to attempt interference or downgrade attempts|By combining connection-establishment and security-establishment into one handshake, QUIC reduces the number of distinct, individually-observable setup phases available for an attacker to target|
|**DCCP's TFRC is congestion-aware but says nothing about authenticity or confidentiality**|Choosing a congestion-control mode under DCCP doesn't, by itself, provide any protection against a malicious or spoofed peer — an application relying on DCCP for "TCP-friendliness" alone still needs to add its own security layer|Treat congestion-control compatibility and security as entirely separate design requirements; DCCP only solves the former, the same way raw TCP alone (without TLS) only solves reliability, not confidentiality|

---

## Questions I Still Have

- [ ] If QUIC's congestion control is based on TCP NewReno specifically, does that mean QUIC inherits the _same_ RTT-based unfairness against BBR-style flows discussed back in Section 3.7.4 — or does running over UDP at the application layer give QUIC implementations more freedom to swap in CUBIC- or BBR-style behavior without waiting on an OS update?
- [ ] With per-stream reliable delivery solving HOL blocking _within_ one QUIC connection, does congestion control still apply at the whole-connection level, or could one badly-behaved stream still indirectly throttle every other stream sharing the same connection's congestion window?
- [ ] DCCP and QUIC both exist as alternatives to plain TCP, but for very different audiences (streaming media vs. general secure web traffic) — has DCCP/TFRC actually seen meaningful real-world deployment the way QUIC clearly has, or did QUIC's broader feature set effectively supersede the use case DCCP was originally designed for?
- [ ] The text frames "application-update timescales" as a clear advantage for QUIC's evolution speed — but does that same speed advantage cut the other way too, in terms of how quickly _vulnerabilities_ in QUIC implementations might need patching across many independent application-layer codebases, compared to patching one shared OS-level TCP stack?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**DCCP (Datagram Congestion Control Protocol)**|A low-overhead, message-oriented, UDP-like unreliable protocol that adds application-selectable, TCP-compatible congestion control|
|**TFRC (TCP-Friendly Rate Control)**|A DCCP-selectable congestion-control mode that smooths out TCP's AIMD sawtooth while targeting a long-term rate similar to TCP's|
|**Equation-based congestion control**|TFRC's approach: measuring actual loss rate and feeding it into an equation that estimates the throughput a real TCP connection would achieve under that same loss rate|
|**QUIC (Quick UDP Internet Connections)**|An application-layer protocol, running over UDP, that reimplements TCP-like reliable transfer, congestion control, and secure connection management, used as the transport for HTTP/3|
|**Connection ID**|A QUIC identifier for the connection as a whole, carried in every packet header|
|**Stream ID**|A QUIC identifier for one individual stream within a connection, allowing many streams to be multiplexed over one connection|
|**Stream**|A QUIC abstraction providing reliable, in-order, bidirectional delivery of data between two endpoints, scoped to a single stream rather than the whole connection|
|**Head-of-line (HOL) blocking**|The problem where one lost/delayed unit of data stalls delivery of all unrelated data queued behind it on the same ordered channel|
|**SCTP (Stream Control Transmission Protocol)**|An earlier reliable, message-oriented protocol that pioneered multiplexing multiple application-level streams over a single connection|
|**TCP NewReno**|A slight modification of TCP Reno's congestion-control algorithm, and the basis for QUIC's own congestion control|
|**TLS (Transport Layer Security)**|The encryption/authentication handshake that QUIC combines directly with its connection-establishment handshake (covered in depth in Chapter 8)|

---

## Related Concepts

---

→ Next: [[INTRODUCTION - NETWORK LAYER]]