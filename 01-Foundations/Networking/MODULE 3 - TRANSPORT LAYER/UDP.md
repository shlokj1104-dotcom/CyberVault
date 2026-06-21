---
title: UDP
date: 2026-06-21
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 3.3 Connectionless Transport: UDP

> **One-Line Summary:** UDP is the Internet's deliberately minimal transport protocol — it does only the two things a transport protocol can't skip (multiplexing/demultiplexing, plus light error checking), adds no connection setup, no connection state, and no congestion control, and in exchange gives applications nearly direct, low-overhead access to IP's raw best-effort delivery.

---

## Core Idea: Designing the Bare-Minimum Transport Protocol

Imagine designing a no-frills transport protocol from scratch. The simplest possible version — a **vacuous transport protocol** — would just hand application messages straight to the network layer on the way out, and hand whatever arrives straight to the application on the way in.

That can't quite work, though: a host runs many processes at once, and at minimum the transport layer has to multiplex/demultiplex (Section 3.2), or there's no way to know which process a segment belongs to.

**UDP is essentially that vacuous protocol, plus the unavoidable minimum.** Defined in RFC 768, it adds nothing to IP beyond multiplexing/demultiplexing and a small amount of error checking. Choosing UDP over TCP means the application is, for practical purposes, talking almost directly to IP.

### What UDP Actually Does, Step by Step

1. UDP takes a message from the application process.
2. It attaches **source port** and **destination port** fields (for multiplexing/demultiplexing).
3. It adds two other small fields, covered in 3.3.1.
4. It passes the resulting segment to the network layer.
5. The network layer encapsulates it in an IP datagram and makes a **best-effort** delivery attempt (IP gives no guarantees — Section 3.1.2).
6. On arrival, UDP uses the destination port to deliver the data to the correct process.

```
Sending Side                              Receiving Side
─────────────                              ──────────────
Application message
     │
     ▼
UDP adds: [src port | dst port | length | checksum]
     │
     ▼
Network layer → IP datagram → best-effort delivery ──►  Network layer extracts segment
                                                                  │
                                                                  ▼
                                                  UDP reads dest port number
                                                                  │
                                                                  ▼
                                                  Delivers data to correct
                                                       application process
```

### Why UDP Is Called "Connectionless"

There's no handshaking between sender and receiver before a segment goes out — UDP doesn't confirm the other side is ready, it just sends. This is why UDP is called **connectionless**.

> **Analogy:** TCP is a phone call — dial, wait, confirm, then talk. UDP is a letter dropped in a mailbox: written, sealed, sent, with zero confirmation anyone's home to receive it.

### DNS Over UDP, Briefly

DNS is the classic example: the application builds a query, hands it to UDP, which adds its header and ships it off with no handshake. If no reply comes back, it's entirely up to the _application_ to retry, try another server, or give up — UDP itself has no opinion on lost replies.

---

## Why Would a Developer Ever Choose UDP Over TCP?

TCP isn't strictly "better" — some applications are genuinely better served by UDP, for four reasons.

### Reason 1 — Finer Application-Level Control

UDP packages and sends data the instant the application hands it over. TCP, by contrast, throttles the sender via congestion control and keeps retransmitting until a segment is acknowledged, no matter how long that takes. Real-time apps (Internet telephony, live video) often need a minimum sending rate and can tolerate some loss — TCP's model fights that. UDP lets them implement, at the application layer, only whatever extra functionality they actually need.

### Reason 2 — No Connection Establishment

TCP's three-way handshake (Section 3.5) costs a setup delay; UDP has none. This is why DNS typically runs over UDP — over TCP it'd be noticeably slower. It's also why **HTTP/3** uses **QUIC** (RFC 9000/9002) instead of TCP: QUIC rides on UDP but implements reliability itself, at the application layer, avoiding TCP's handshake delay entirely (more in Section 3.8).

### Reason 3 — No Connection State

TCP maintains per-connection state — buffers, congestion-control parameters, sequence/ack numbers (why this matters is covered in Section 3.5). UDP tracks none of it. Practical effect: a server can support far more active clients over UDP than over TCP, since there's no per-client bookkeeping to scale.

### Reason 4 — Small Header Overhead

|Protocol|Header Overhead Per Segment|
|---|---|
|**TCP**|20 bytes, every segment|
|**UDP**|8 bytes|

This adds up fast for apps sending many small packets.

### Figure 3.6 — Popular Internet Applications and Their Transport Protocols

![[Pasted image 20260621214755.png]] _(Figure 3.6 — A two-column table pairing common applications with their application-layer protocol and underlying transport protocol.)_

|Application|Application-Layer Protocol|Underlying Transport Protocol|
|---|---|---|
|Electronic mail|SMTP|TCP|
|Remote terminal access|Telnet|TCP|
|Secure remote terminal access|SSH|TCP|
|Web|HTTP, HTTP/3|TCP (for HTTP), UDP (for HTTP/3)|
|File transfer|FTP|TCP|
|Remote file server|NFS|Typically UDP|
|Streaming multimedia|DASH|TCP|
|Internet telephony|Typically proprietary|UDP or TCP|
|Network management|SNMP|Typically UDP|
|Name translation|DNS|Typically UDP|

Patterns worth noting:

- **Email, remote terminal access, file transfer → TCP.** Losing or garbling a byte is unacceptable.
- **HTTP/3 → UDP (via QUIC),** providing its own error/congestion control at the application layer instead of relying on TCP.
- **SNMP → UDP**, since network management traffic often has to run precisely when the network is already stressed — exactly when TCP's reliable, congestion-controlled transfer is hardest to achieve.
- **DNS → usually UDP**, to dodge TCP's connection-establishment delay (though it can run over TCP too).
- **Multimedia → UDP or TCP.** These apps tolerate small packet loss and react poorly to TCP's congestion control, favoring UDP — but as packet loss drops and more networks block UDP for security reasons (Chapter 8), TCP is becoming more attractive for streaming media too.

---

## Multimedia Over UDP: A Double-Edged Sword

Running multimedia over UDP is common, but needs care — UDP has no congestion control, and congestion control exists to stop the network from reaching a state where almost no useful work gets done. If everyone streamed high-bitrate video with zero congestion control, routers would overflow and few packets would survive — and the resulting loss would force well-behaved **TCP** senders (which _do_ back off under congestion) to throttle dramatically. **Lack of congestion control in UDP can produce high loss rates and crowd out TCP sessions sharing the same links.** Researchers have proposed mechanisms to force adaptive congestion control onto UDP sources too (Mahdavi 1997; Floyd 2000; Kohler 2006/RFC 4340).

### Reliable Transfer Over UDP Is Still Possible

It's possible to have reliable data transfer while still using UDP — by building reliability (acknowledgments, retransmission) directly into the application itself, the same principles studied later for TCP. QUIC does exactly this, implementing reliability at the application layer on top of UDP — giving applications reliability _without_ being bound by TCP's rate-throttling congestion control. "Have your cake and eat it too."

---

## 3.3.1 UDP Segment Structure

Defined in RFC 768 and shown in Figure 3.7:

- **Application data** fills the data field — a query/response message for DNS, audio samples for streaming audio.
- **The header has only four fields**, 2 bytes each.

### Figure 3.7 — UDP Segment Structure

![[Pasted image 20260621215525.png]] _(Figure 3.7 — A 32-bit-wide header diagram. Top row: "Source port #" and "Dest. port #," each 16 bits. Next row: "Length" and "Checksum." Below both: the "Application data (message)" field.)_

```
            32 bits
 ┌──────────────────┬──────────────────┐
 │   Source port #   │   Dest. port #   │
 ├──────────────────┼──────────────────┤
 │      Length       │     Checksum     │
 ├───────────────────┴──────────────────┤
 │       Application data (message)       │
 └─────────────────────────────────────────┘
```

|Field|Purpose|
|---|---|
|**Source port #**|Sending socket's "return address" (Section 3.2.1)|
|**Dest. port #**|Performs demultiplexing — gets data to the correct process|
|**Length**|Total UDP segment size in bytes (header + data); needed since data size varies segment to segment|
|**Checksum**|Lets the receiver check for errors|

> **Note:** the checksum is actually also calculated over a few IP-header fields, not just the UDP segment — a detail set aside here for clarity (error detection basics are covered separately in Section 6.2).

---

## 3.3.2 UDP Checksum

**Purpose:** error detection — determining whether bits in a UDP segment were altered in transit (link noise, or corruption while sitting in a router's memory).

**How the sender computes it:** take the **1's complement of the sum of all 16-bit words** in the segment, wrapping any overflow, and place the result in the checksum field.

### Worked Example

Three 16-bit words:

```
0110011001100000
0101010101010101
1000111100001100
```

Sum of first two:

```
  0110011001100000
+ 0101010101010101
  1011101110110101
```

Add the third (note the overflow, wrapped around):

```
  1011101110110101
+ 1000111100001100
  0100101011000010
```

1's complement (flip every bit) → **`1011010100111101`** — this is the checksum.

**At the receiver:** all four words (including the checksum) are added. All-1s (`1111...1`) → no detected errors. Any 0 in the result → corruption detected.

### Why Bother, When Link Layers Already Check for Errors?

Two gaps a link-layer check (like Ethernet's) can't close:

1. **Not every link is guaranteed to check for errors** — some may use a link-layer protocol that doesn't.
2. **Bit errors can be introduced while a segment sits in a router's memory**, which isn't a transmission error a link check would ever catch.

### The End-to-End Principle

Since neither link-by-link reliability nor in-memory error detection is guaranteed, UDP must provide error detection itself, **end-to-end**. This is a textbook case of the **end-to-end principle** (Saltzer 1984): functionality that must ultimately be guaranteed end-to-end may make a partial lower-layer version of it redundant or low-value. Because IP is meant to run over virtually any link-layer technology, UDP's checksum acts as a safety net that holds even when the layer underneath doesn't.

> **Analogy:** A courier promises to inspect every package along the route — but you can't verify every handler did it properly. The only way to be sure is to check it yourself, once, at the very end.

**Detection ≠ correction.** UDP does nothing to recover from a detected error — some implementations discard the damaged segment, others pass it to the application with a warning.

That's UDP. Next: TCP offers reliable data transfer and more — and is correspondingly more complex. Before TCP itself, it's worth studying the underlying _principles_ of reliable data transfer on their own terms.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**No handshake → no identity check**|Source IP spoofing is trivial; no handshake step would expose the forgery|Never treat UDP source info as proof of identity; pair with application-layer authentication|
|**No connection state**|Attacker can flood huge UDP volumes cheaply — no per-connection cost on their end (UDP flood)|Rate-limit/filter at the network edge, since UDP won't self-regulate|
|**No congestion control + spoofable source**|Small spoofed request → large reply directed at a forged victim address (amplification attacks, e.g. DNS/NTP)|Rate-limit open UDP services with large replies; validate source addresses (BCP 38/RFC 2827)|
|**Checksum ≠ authentication**|Tamper with payload, recompute a valid checksum — no protection against deliberate tampering|Use TLS/DTLS or app-layer MACs where tamper-resistance matters|
|**Orgs increasingly block UDP**|UDP-based exploits (amplification, flooding) face more filtering at network perimeters|Easier to apply stateful security controls to a protocol (TCP) that already tracks state|

---

## Questions I Still Have

- [ ] If reliability can be fully built at the application layer over UDP (QUIC), is there any case where TCP's _built-in_ reliability is strictly necessary rather than just convenient?
- [ ] Who decides whether a damaged UDP segment gets discarded vs. passed up with a warning — OS, UDP implementation, or application? Configurable?
- [ ] Does the end-to-end principle imply link-layer error checking (Ethernet's) is pointless given UDP's end-to-end checksum, or does it still add value (e.g., catching errors before wasting further bandwidth)?

---

## Key Terms — Quick Reference

| Term                                      | Definition                                                                                                                          |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **UDP**                                   | RFC 768; minimal, connectionless transport — only multiplexing/demultiplexing and light error checking                              |
| **Connectionless**                        | No handshaking before data is sent                                                                                                  |
| **Connection state**                      | Per-connection bookkeeping (buffers, congestion params, seq/ack numbers); TCP has it, UDP doesn't                                   |
| **QUIC**                                  | Reliable transport built at the application layer over UDP; underlies HTTP/3 (RFC 9000/9002)                                        |
| **End-to-end principle**                  | Functionality that must hold end-to-end may make a partial lower-layer version redundant (Saltzer 1984)                             |
| **UDP segment structure**                 | Four 16-bit fields — source port, dest port, length, checksum — plus application data                                               |
| **Checksum (UDP)**                        | 1's-complement sum of 16-bit words; detects but never corrects errors                                                               |
| **UDP amplification attack**              | Spoofed small request → large reply → flood directed at a forged (victim) address                                                   |
| **Adaptive congestion control (for UDP)** | Proposed mechanisms (Mahdavi 1997; Floyd 2000; Kohler 2006/RFC 4340) to make UDP sources behave more cooperatively under congestion |

---

## Related Concepts

---

→ Next: [[PRINCIPLES OF RELIABLE DATA TRANSFER]]