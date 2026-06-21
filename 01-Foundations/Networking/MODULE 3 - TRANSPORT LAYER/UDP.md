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

> **One-Line Summary:** UDP is the Internet's deliberately minimal transport protocol — it does only the two things a transport protocol absolutely cannot avoid doing (multiplexing/demultiplexing, plus a small amount of error checking), adds no connection setup, no connection state, and no congestion control, and in exchange gives applications nearly direct, low-overhead access to IP's raw best-effort delivery service.

---

## Core Idea: Designing the Bare-Minimum Transport Protocol

Here's a useful thought experiment for understanding _why_ UDP looks the way it does. Suppose you were asked to design a no-frills, bare-bones transport protocol from scratch. Your first instinct might be to build a **vacuous transport protocol** — one that does almost nothing at all: on the sending side, just take the message straight from the application process and hand it directly to the network layer; on the receiving side, just take whatever arrives from the network layer and hand it straight to the application process.

But as you already know from Section 3.2, this can't quite work as stated. At an absolute minimum, the transport layer _has_ to provide a multiplexing/demultiplexing service — without it, there's no way to get data to the correct process on a host running more than one network application at once.

**UDP is essentially that vacuous protocol, plus the unavoidable minimum.** Defined in RFC 768, UDP does just about as little as a transport protocol possibly can. Beyond the multiplexing/demultiplexing function and a small amount of error checking, **it adds nothing to IP.** In fact, if an application developer chooses UDP instead of TCP, the application is, for most practical purposes, talking almost directly to IP itself.

### What UDP Actually Does, Step by Step

1. UDP takes a message from the application process.
2. It attaches **source port** and **destination port** fields, which exist purely to support the multiplexing/demultiplexing service (Section 3.2).
3. It adds two other small fields (covered shortly, in 3.3.1).
4. It passes the resulting segment down to the network layer.
5. The network layer encapsulates the segment inside an IP datagram and makes a **best-effort** attempt to deliver it to the receiving host (recall: IP gives no guarantees at all — Section 3.1.2).
6. If the segment arrives, UDP uses the destination port number to deliver the segment's data to the correct application process.

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

Note one crucial detail in that sequence: **there is no handshaking between the sending and receiving transport-layer entities before a segment gets sent.** UDP doesn't check in with the other side, doesn't negotiate anything, doesn't confirm the receiver is even ready. It simply builds a segment and sends it. For exactly this reason, UDP is described as **connectionless**.

> **Analogy — Dropping a Letter in a Mailbox vs. Placing a Phone Call:** TCP, as you'll see later, is like making a phone call — you dial, wait for the other person to pick up, confirm you're both ready, and only then start talking. UDP is like dropping a letter in a mailbox: you write it, seal it, and send it, with zero confirmation that anyone is even home to receive it.

### A Worked Example: DNS Over UDP

DNS is a textbook example of an application-layer protocol that typically runs over UDP. Here's the full flow:

1. The DNS application on a host wants to make a query.
2. It constructs a DNS query message.
3. It passes that message to UDP.
4. **No handshaking happens with the UDP entity on the destination end system** — UDP just adds its header fields to the message and passes the resulting segment to the network layer.
5. The network layer encapsulates the UDP segment into a datagram and sends it toward a name server.
6. The querying host's DNS application then **waits** for a reply.
7. If no reply arrives (possibly because the underlying network lost the query, or lost the reply), the application might retry by resending the query, try a different name server, or simply give up and inform whatever invoked it that it couldn't get a reply.

Notice that _none_ of that retry logic lives inside UDP itself — UDP doesn't know or care whether a reply ever comes back. Any resilience has to be built by the application sitting on top of it.

---

## Why Would a Developer Ever Choose UDP Over TCP?

It's natural to wonder why anyone would deliberately pick UDP, given that TCP offers a reliable data transfer service and UDP does not. The answer is that **it's genuinely not always preferable** — some applications are better served by UDP, for four concrete reasons.

### Reason 1 — Finer Application-Level Control Over What Data Is Sent, and When

Under UDP, the moment an application process hands data to UDP, UDP packages it into a segment and immediately passes it to the network layer. TCP, by contrast, has a **congestion-control mechanism** that actively throttles the sender whenever one or more links between source and destination become excessively congested. TCP will also keep re-sending a segment until its receipt is acknowledged by the destination — no matter how long reliable delivery actually takes.

Real-time applications (think Internet telephony or live video) often need a **minimum sending rate**, are willing to tolerate **some data loss**, and simply can't accept arbitrarily long delays waiting for guaranteed delivery. TCP's service model is a poor match for these needs. By using UDP instead, these applications stay free to implement, at the application layer, only whatever additional functionality they actually need beyond UDP's bare-bones, no-frills segment delivery.

### Reason 2 — No Connection Establishment

TCP uses a **three-way handshake** before it transfers any data (covered later, in Section 3.5). UDP just blasts data out without any formal preliminaries — meaning UDP introduces **no additional setup delay** to establish a connection.

This is the principal reason DNS typically runs over UDP rather than TCP: DNS would be noticeably slower if it had to run over TCP. Recall from Section 2.2 that HTTP/1 and HTTP/2 use TCP rather than UDP, since reliability is critical for web pages containing text. But that same TCP connection-establishment delay is itself a real contributor to the delays users experience downloading web documents.

This is precisely why **HTTP/3** instead uses the **QUIC protocol** (Quick UDP Internet Connection, RFC 9000 and RFC 9002) — QUIC uses UDP as its underlying transport protocol, and implements reliability _itself_, in the application layer, sitting on top of UDP. (A closer look at QUIC follows later, in Section 3.8.)

### Reason 3 — No Connection State

TCP maintains **connection state** inside the end systems for every connection it manages. This state includes receive and send buffers, congestion-control parameters, and sequence/acknowledgment number parameters (you'll see in Section 3.5 exactly why this state is needed — it's what makes TCP's reliable data transfer service and its congestion control possible in the first place).

**UDP does not maintain any connection state at all**, and doesn't track any of those parameters. A direct consequence: a server devoted to a particular application can typically support **many more active clients** when that application runs over UDP instead of TCP, simply because the server isn't burning memory and bookkeeping on per-client connection state.

### Reason 4 — Small Packet Header Overhead

|Protocol|Header Overhead Per Segment|
|---|---|
|**TCP**|20 bytes, in _every_ segment|
|**UDP**|Only 8 bytes|

For applications sending many small packets, that overhead difference compounds quickly.

### Figure 3.6 — Popular Internet Applications and Their Transport Protocols

![[Pasted image 20260621214755.png]]
_(Figure 3.6 — A two-column table pairing common applications with their application-layer protocol and underlying transport protocol.)_

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

A few patterns are worth internalizing from this table:

- **E-mail, remote terminal access, and file transfer all run over TCP** — every one of these applications fundamentally needs TCP's reliable data transfer service; losing or garbling a byte of a file transfer or an email is unacceptable.
- **Early versions of HTTP ran over TCP**, but more recent versions run over UDP instead, providing their _own_ error control and congestion control (among other services) up at the application layer — this is the QUIC story from Reason 2 above.
- **UDP is preferred for network management (SNMP)** specifically _because_ network-management traffic often needs to run precisely when the network is already in a stressed state — which is exactly when reliable, congestion-controlled data transfer (TCP's specialty) is hardest to achieve.
- **DNS usually runs over UDP**, avoiding TCP's connection-establishment delay — though DNS _can_ run over TCP when needed.
- **Multimedia applications** — Internet phone, real-time video conferencing, streaming of stored audio and video — sometimes use UDP and sometimes use TCP. All of these can tolerate a small amount of packet loss, so reliable data transfer isn't absolutely critical to their success. Real-time applications in particular react very poorly to TCP's congestion control, which is why developers often choose UDP for them instead — though when packet loss rates are low, and as more organizations block UDP traffic for security reasons (see Chapter 8), TCP has become an increasingly attractive option for streaming media transport too.

---

## Multimedia Over UDP: A Double-Edged Sword

Although running multimedia applications over UDP is common today, **it needs to be done with care.** As already noted, UDP has no congestion control — but congestion control exists for an important reason: to prevent the network from entering a congested state in which very little useful work gets done at all.

If everyone started streaming high-bit-rate video without any congestion control whatsoever, routers along the source-to-destination path would overflow, and very few UDP packets would actually survive the trip successfully. Worse, the high loss rates induced by these uncontrolled UDP senders would cause **TCP senders** (which, as you'll see, _do_ decrease their sending rates in the face of congestion) to dramatically throttle back their own rates. The net effect: **a lack of congestion control in UDP can result in high loss rates between a UDP sender and receiver, and can crowd out well-behaved TCP sessions sharing the same congested links.**

This is a real enough problem that researchers have proposed new mechanisms to force _all_ sources — including UDP sources — to perform some form of adaptive congestion control (see, for example, Mahdavi 1997; Floyd 2000; Kohler 2006 / RFC 4340).

### Having Your Cake and Eating It Too: Reliable Transfer Over UDP

Before moving on, it's worth flagging something that might seem contradictory at first: **it is entirely possible for an application to have reliable data transfer while still using UDP.** This works if reliability is built directly into the _application itself_ — for example, by the application adding its own acknowledgment and retransmission mechanisms (the same principles you'll study in the next section, applied at the application layer instead of inside the transport layer).

QUIC is the prime real-world example: it implements reliability directly in the application layer, on top of UDP. By doing this, application processes get to communicate reliably **without** being subjected to the transmission-rate constraints that TCP's congestion-control mechanism would otherwise impose on them — they get reliability _and_ freedom from TCP's rate throttling, simultaneously.

---

## 3.3.1 UDP Segment Structure

The UDP segment structure, shown in Figure 3.7, is defined in RFC 768.

### What's Inside a UDP Segment

- **The application data occupies the data field** of the UDP segment. For DNS, that data field holds either a query message or a response message. For a streaming-audio application, it's filled with audio samples instead.
- **The UDP header has only four fields**, each consisting of just two bytes (16 bits).

### Figure 3.7 — UDP Segment Structure


_(Figure 3.7 — A 32-bit-wide header diagram. The top row holds two 16-bit fields side by side: "Source port #" and "Dest. port #." The next row holds two more 16-bit fields side by side: "Length" and "Checksum." Below both rows sits the "Application data (message)" field.)_

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

### What Each Field Is For

|Field|Purpose|
|---|---|
|**Source port #**|Identifies the sending socket — primarily useful as a "return address" (Section 3.2.1)|
|**Dest. port #**|Lets the destination host pass the application data to the correct process running on the destination end system — i.e., this is the field that actually performs the demultiplexing function|
|**Length**|Specifies the number of bytes in the entire UDP segment (header **plus** data)|
|**Checksum**|Used by the receiving host to check whether errors have been introduced into the segment|

**Why does UDP even need an explicit length field?** Because the size of the data field can differ from one UDP segment to the next — there's no fixed payload size, so an explicit length value is necessary so the receiver knows exactly where the segment ends.

> **A subtle technical note:** the checksum is, in truth, also calculated over a few fields drawn from the IP header, in addition to the UDP segment itself — but that detail is set aside here so the bigger picture isn't lost in technicalities. (The basic principles behind error detection generally are covered separately, in Section 6.2.)

---

## 3.3.2 UDP Checksum

### What the Checksum Is For

**The UDP checksum provides error detection.** That is, the checksum exists to determine whether bits within a UDP segment have been altered in transit — for example, due to noise on a link, or corruption while the segment sat in a router's memory — as it moved from source to destination.

### How the Sender Computes It

UDP, on the sending side, performs the **1's complement of the sum of all the 16-bit words** in the segment, wrapping around any overflow encountered during that sum. The resulting value is placed into the checksum field of the UDP segment.

### A Worked Example

Suppose a segment contains the following three 16-bit words:

```
0110011001100000
0101010101010101
1000111100001100
```

**Step 1 — Add the first two words:**

```
  0110011001100000
+ 0101010101010101
  ─────────────────
  1011101110110101
```

**Step 2 — Add the third word to that running sum:**

```
  1011101110110101
+ 1000111100001100
  ─────────────────
  0100101011000010   (with the overflow bit wrapped around)
```

Notice that this last addition produced an overflow, which got wrapped around as part of the standard procedure.

**Step 3 — Take the 1's complement of the final sum.** The 1's complement is obtained simply by flipping every 0 to a 1 and every 1 to a 0. Applying that to the sum `0100101011000010` gives:

```
1011010100111101   ← this becomes the checksum
```

**At the receiver:** all four 16-bit words are added together — including the checksum value itself this time. If no errors were introduced anywhere along the path, that sum comes out to all 1's: `1111111111111111`. If even a single bit in that result is a 0, the receiver knows errors were introduced somewhere into the segment.

### Why Bother, When Link Layers Already Check for Errors?

A fair question: many link-layer protocols — including the popular Ethernet protocol — already perform their own error checking. So why does UDP duplicate that effort at the transport layer?

Two reasons:

1. **There's no guarantee every link along the path performs error checking.** One of the links between source and destination might use a link-layer protocol that simply doesn't check for errors at all.
2. **Even if every link does check correctly, bit errors can still be introduced while a segment sits in a router's memory** — a problem no link-layer check can catch, since it isn't a transmission error at all.

### The End-to-End Principle

Given that **neither** link-by-link reliability **nor** in-memory error detection is guaranteed anywhere along the path, UDP has to provide error detection itself, at the transport layer, **on an end-to-end basis** — if the end-to-end data transfer service is going to provide error detection at all, someone has to do it from one true endpoint to the other.

This is a textbook example of the celebrated **end-to-end principle** in system design (Saltzer 1984), which states that since certain functionality — error detection, in this case — must ultimately be implemented on an end-to-end basis anyway, the same functionality placed at lower layers may end up being redundant, or of comparatively little value, when weighed against the cost of providing it there too.

> **Analogy — Checking Your Own Package, Even After the Courier Inspected It:** Imagine a courier company that promises to inspect every package for damage along its route — but you have no way of verifying that _every single_ courier on _every single_ leg of the journey, including warehouse handlers between trucks, actually did that inspection properly. The only way to be truly sure your package arrived intact is to check it yourself, once, at the very end. That's the end-to-end principle: don't fully trust intermediate checks you can't verify — do the check yourself at the endpoints, where it actually matters.

Because IP is meant to be able to run over just about any layer-2 protocol, it's genuinely useful for the transport layer to provide error checking as a **safety net** — a backstop that holds even when whatever's underneath doesn't.

### What UDP Does (and Doesn't Do) After Detecting an Error

Although UDP provides error checking, **it does nothing to actually recover from an error it detects.** Some implementations of UDP simply **discard** the damaged segment outright; others **pass the damaged segment up to the application anyway, along with a warning.**

That wraps up the core discussion of UDP. As you'll see next, **TCP offers reliable data transfer to its applications, along with other services UDP simply doesn't provide** — and naturally, TCP is also considerably more complex than UDP as a result. Before diving into TCP itself, it's worth stepping back first to study the underlying _principles_ of reliable data transfer in their own right — independent of any one specific protocol.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**UDP is connectionless — no handshake to verify identity**|Since UDP never confirms the receiver is ready or who the sender claims to be before data flows, an attacker can trivially forge ("spoof") the source IP address on a UDP segment, since there's no handshake step that would expose the forgery|Never treat the presence of UDP traffic from a given source as proof of that source's identity; pair UDP-based applications with application-layer authentication where identity actually matters|
|**No connection state means no built-in resource exhaustion check**|Because UDP tracks zero per-client state, an attacker can blast huge volumes of UDP traffic at a target with very little cost to themselves, since the protocol itself places no inherent ceiling on how many "connections" can be attempted — this is the core mechanic behind UDP flood attacks|Rate-limit and filter UDP traffic at the network edge; since UDP itself won't self-regulate or track abuse, that job has to be done by firewalls, load balancers, or upstream infrastructure|
|**No congestion control is a standing amplification risk**|Combined with spoofed source addresses, an attacker can send a small UDP request to a service with a much larger reply (DNS, NTP, and others have historically been abused this way), forging the source address so the large reply floods a victim instead of the attacker — a UDP amplification attack|Disable or rate-limit open UDP services that produce large replies to small requests; validate source addresses at the network boundary (BCP 38 / RFC 2827) so spoofed packets don't leave the network in the first place|
|**UDP checksum detects corruption, not malice**|The checksum only verifies _accidental_ bit errors; it provides no cryptographic guarantee, so an attacker who modifies a segment in transit can simply recompute a valid checksum for their tampered version — the checksum offers zero protection against deliberate tampering|Use cryptographic integrity protection (e.g., TLS/DTLS or application-layer MACs) wherever tamper-resistance actually matters; never rely on the UDP checksum as a security control|
|**Organizations increasingly block UDP for security reasons**|Attackers who rely on UDP-based exploits (amplification, flooding, or evasion of stateful inspection) find their traffic increasingly filtered as more networks restrict UDP at the perimeter|This trend is exactly why TCP is becoming a more attractive transport even for streaming media — it's easier to apply stateful security controls to a protocol that already maintains connection state|

---

## Questions I Still Have

- [ ] If reliability can be built entirely at the application layer on top of UDP (as QUIC does), is there any real scenario where TCP's _built-in_ reliability is still strictly necessary rather than just convenient?
- [ ] The chapter says some UDP implementations discard a damaged segment while others pass it up with a warning — who actually decides this behavior: the OS, the UDP implementation itself, or the application? Is this configurable?
- [ ] If UDP amplification attacks rely on a small request producing a large reply, are there design guidelines for new UDP-based protocols to avoid being usable this way, or is this purely something defenders have to patch after the fact?
- [ ] The end-to-end principle argues lower-layer error checking may be "redundant or of little value" — does that mean link-layer error checking (like Ethernet's) is actually pointless given UDP's end-to-end checksum, or does it still add value in some other way (e.g., catching errors before they waste further bandwidth)?
- [ ] QUIC implements reliability and (presumably) something like congestion control at the application layer over UDP — does that mean QUIC traffic effectively behaves like a "third" transport option in practice, even though it's technically still riding on UDP underneath?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**UDP (User Datagram Protocol)**|RFC 768; the Internet's minimal, connectionless transport protocol — provides only multiplexing/demultiplexing and light error checking, nothing more|
|**Connectionless**|A property of a protocol where no handshaking occurs between sender and receiver before data is sent — UDP sends without ever confirming the other side is ready|
|**Three-way handshake**|The connection-setup procedure TCP uses before transferring data (covered fully in Section 3.5); UDP has no equivalent, which is exactly why it introduces no setup delay|
|**Connection state**|Per-connection bookkeeping (buffers, congestion-control parameters, sequence/ack numbers) that TCP maintains in the end systems; UDP maintains none of this|
|**QUIC**|"Quick UDP Internet Connection" (RFC 9000, RFC 9002); the transport protocol underlying HTTP/3, which runs over UDP but implements its own reliability at the application layer|
|**End-to-end principle**|A system-design principle (Saltzer 1984) stating that functionality which must ultimately be guaranteed end-to-end may be redundant or low-value if also implemented at lower layers|
|**UDP segment structure**|Four 16-bit fields — source port, destination port, length, checksum — followed by the application data|
|**Length field (UDP)**|Specifies the total size, in bytes, of the UDP segment (header plus data); needed because data field size varies segment to segment|
|**Checksum (UDP)**|A 16-bit field computed via 1's-complement addition of the segment's 16-bit words, used by the receiver to detect (not correct) bit errors introduced in transit|
|**1's complement**|The error-detection technique UDP's checksum is built on: sum all 16-bit words (wrapping overflow), then flip every bit of the result to produce the checksum|
|**UDP amplification attack**|An attack exploiting UDP's lack of handshaking and source-address verification: a small spoofed request triggers a large reply directed at a forged (victim) address|
|**Adaptive congestion control (for UDP sources)**|Proposed mechanisms (Mahdavi 1997; Floyd 2000; Kohler 2006/RFC 4340) to force UDP-based applications to behave more cooperatively with network congestion, despite UDP itself having none built in|

---

## Related Concepts

---

→ Next: [[PRINCIPLES OF RELIABLE DATA TRANSFER]]