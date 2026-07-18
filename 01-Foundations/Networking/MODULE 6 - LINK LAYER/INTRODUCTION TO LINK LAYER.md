---
title: INTRODUCTION TO LINK LAYER
date: 2026-07-18
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 6.1 Introduction to the Link Layer

> **One-Line Summary:** The **link layer** moves a network-layer **datagram** over one **individual link** in the end-to-end path by encapsulating it in a **link-layer frame**, and — depending on the specific link-layer protocol in use — may additionally provide **framing**, **link access (MAC)**, **reliable delivery**, and **error detection/correction**; this functionality is implemented as a **hardware/software hybrid**, centered on the **network adapter (NIC)**, which sits at the exact boundary where software meets hardware in the protocol stack.

---

## Core Idea: From "Any Two Hosts" to "One Single Link"

The network layer (Chapters 4–5) solved a big problem: how to deliver a datagram between **any two hosts**, anywhere in the network. But that end-to-end journey is actually built out of a **chain of individual hops**. Between the source host and destination host, a datagram travels over a **series of communication links** — some wired, some wireless — passing through a series of **packet switches** (switches and routers) along the way.

The link layer is the part of the protocol stack that answers a narrower, more local question: **how do you get a datagram across just one of those individual links?** This is a fundamentally different problem from network-layer routing — it's not about finding a path across the whole network, it's about successfully handing a frame from one node to the **immediately adjacent** node at the other end of a single link.

### Key Terminology

Before diving in, three terms anchor everything in this chapter:

|Term|Definition|
|---|---|
|**Node**|Any device that runs a link-layer (i.e., **Layer 2**) protocol. This includes hosts, routers, switches, and WiFi access points.|
|**Link**|The communication channel that connects **adjacent** nodes along the end-to-end communication path.|
|**Frame**|The link-layer encapsulation of a network-layer datagram, transmitted over a single link.|

> **Important distinction to hold onto:** A **router** operates at the network layer _and_ link layer (it must run a link-layer protocol on each of its interfaces to actually transmit frames onto its attached links), whereas a **link-layer switch** operates only at the link layer — it has no network-layer addressing role of its own. This distinction becomes central later in the chapter when comparing switches and routers directly.

### Six Hops, Six Separate Link-Layer Protocols

Consider sending a datagram from a wireless host to a server, passing through six links: a WiFi link between the sending host and a WiFi access point, an Ethernet link between the access point and a link-layer switch, a link between that switch and a router, a link between two routers, an Ethernet link between a router and another switch, and a final Ethernet link between that switch and the server.

```
Fig -- Six Link-Layer Hops: Host to Server
────────────────────────────────────────────────────
Host -1-> AP -2-> Switch -3-> Router -4-> Router
          -5-> Switch -6-> Server

 (1) Host <-> AP       : WiFi link
 (2) AP <-> Switch      : Ethernet link
 (3) Switch <-> Router  : link (switch-to-router)
 (4) Router <-> Router  : link (inter-router)
 (5) Router <-> Switch  : Ethernet link
 (6) Switch <-> Server  : Ethernet link

 Each numbered hop = one separate link-layer
 protocol run over one individual link.
```

![[Pasted image 20260718000001.png]]

The critical insight here: **different links along the same end-to-end path can — and often do — run entirely different link-layer protocols.** The WiFi hop uses one protocol; the Ethernet hops use another. Each link-layer protocol only has to worry about correctly moving a frame across _its own_ link — it has no responsibility for, or awareness of, the rest of the path.

> **Analogy — The Travel Agent and the Three-Leg Trip:** Imagine a tourist traveling from Princeton, New Jersey, to Lausanne, Switzerland. A travel agent books three separate segments: a limousine from Princeton to JFK airport, a plane from JFK to Geneva, and a train from Geneva to Lausanne. Each segment is "direct" between two _adjacent_ locations, and each is operated by a **completely different company** using a **completely different transportation mode** (limousine, plane, train) — yet each provides the same basic service: moving the passenger from one location to the next adjacent one. In this analogy, the **tourist is the datagram**, **each transportation segment is a link**, the **transportation mode is a link-layer protocol**, and the **travel agent is a routing protocol** (since it decided the sequence of segments). Just as the limousine company has zero involvement in how the airline handles its leg of the trip, the WiFi protocol on link (1) has zero involvement in how Ethernet handles link (2) — each link-layer protocol is self-contained and only responsible for its own hop.

---

## 6.1.1 The Services Provided by the Link Layer

Although the _basic_ service of any link layer is simply "move a datagram from one node to an adjacent node over a single link," the _details_ of the provided service can vary significantly from one link-layer protocol to the next. A given link-layer protocol may offer some or all of the following services.

### 1. Framing

Almost all link-layer protocols **encapsulate** each network-layer datagram within a **link-layer frame** before transmitting it over the link. A frame consists of a **data field** — into which the network-layer datagram is inserted — plus a number of **header fields** (and sometimes trailer fields). The exact structure of the frame is defined by the specific link-layer protocol in use, and varies from protocol to protocol.

```
Fig -- Link-Layer Framing
────────────────────────────────────────────
 Network-layer datagram (passed down from
 the network layer at the sending node)
          │
          ▼
 ┌────────┬────────────────────┬─────────┐
 │ Header │   Data field       │ Trailer │
 │ fields │ (datagram inserted │ (if any)│
 │        │  here, unchanged)  │         │
 └────────┴────────────────────┴─────────┘
          Link-layer FRAME
   (transmitted onto the link as bits)
```

> **Analogy — Packing a Letter Into an Envelope for Local Courier Delivery:** Think of the network-layer datagram as a **letter** already addressed for its final destination. Framing is like a local courier company slipping that letter into **its own branded envelope** before handing it to the next courier hub — the envelope (frame header) has local routing/handling info specific to _that courier's own network_, but the letter inside (the datagram) is never opened or altered. When the envelope reaches the next hub, it may get thrown away and the letter re-enveloped in a _different_ courier's packaging (a different link-layer protocol's frame) for the next leg — this is exactly why the WiFi frame gets stripped off and replaced with an Ethernet frame as the datagram moves from link (1) to link (2) in the six-hop example above.

### 2. Link Access

A **medium access control (MAC) protocol** specifies the rules by which a frame is transmitted onto the link. This service branches into two very different scenarios depending on the link's nature:

|Link Type|MAC Complexity|
|---|---|
|**Point-to-point link**|A single sender at one end, a single receiver at the other. The MAC protocol is simple — or effectively nonexistent — since there's no contention for the channel.|
|**Broadcast link**|Multiple nodes share a single broadcast channel. This is the more interesting and challenging case, known as the **multiple access problem** — the MAC protocol must coordinate frame transmissions among the many contending nodes.|

> **Analogy — A Private Driveway vs. a Shared Conference-Call Line:** A point-to-point link's MAC is like a private driveway connecting exactly two houses — there's no possibility of someone else pulling in at the same time, so no traffic-coordination rules are needed. A broadcast link's MAC is like a **shared conference-call line** where anyone who talks is heard by _everyone_ on the line — without a moderator or a "raise your hand" protocol, multiple people talking simultaneously turns into unintelligible noise (a **collision**). The MAC protocol is that moderator, deciding whose turn it is to speak.

### 3. Reliable Delivery

When a link-layer protocol provides **reliable delivery service**, it guarantees to move each network-layer datagram across the link **without error**. This should sound familiar — it's conceptually the same guarantee that certain transport-layer protocols (like TCP) provide end-to-end — and it's achieved the same way: via **acknowledgments and retransmissions**.

However, link-layer reliable delivery is a _targeted tool_, not a default:

- **Most useful for:** links that are prone to **high error rates**, such as wireless links — the goal being to correct an error **locally, right where it occurred**, rather than forcing an end-to-end retransmission all the way back from a transport- or application-layer protocol at the original source.
- **Often skipped entirely for:** low bit-error-rate links, including fiber, coax, and many twisted-pair copper links — where the overhead of per-link reliability mechanisms isn't worth paying, since errors are already rare. Many wired link-layer protocols therefore **do not** provide reliable delivery service.

> **Analogy — Proofreading at Every Desk vs. Only at the Final Destination:** Imagine a multi-stage document relay where the document passes through several offices before reaching its final reader. On a wireless-like "sloppy handwriting" link, it makes sense to have each office **proofread and correct errors immediately** before passing the document on — catching mistakes early and locally. On a clean, reliable link (like a pristine fiber-optic connection), proofreading at every single office is wasted effort — nobody's making mistakes there, so you only need one final check (if any) at the very end. This mirrors the classic **end-to-end argument**: put reliability where it's actually needed, not everywhere by default.

### 4. Error Detection and Correction

Bit errors are a physical reality on any link: receiving hardware can incorrectly interpret a transmitted 0 as a 1, or vice versa, due to **signal attenuation** and **electromagnetic noise**. Because there's no way to prevent every such error from occurring, many link-layer protocols provide a mechanism to at least **detect** — and sometimes **correct** — these errors.

- **Error detection** is achieved by having the **transmitting node** include error-detection bits in the frame, and having the **receiving node** perform an error check using those bits. (This should also sound familiar: the Internet's transport layer and network layer already provide a limited form of error detection via the **Internet checksum**.) Link-layer error detection is usually **more sophisticated** than the checksum and is typically **implemented in hardware**.
- **Error correction** goes a step further than detection: a receiver that performs error correction not only detects that bit errors occurred in the frame, but also determines **exactly where** in the frame those errors occurred — and then corrects them, all without needing a retransmission.

|Capability|What the Receiver Can Do|
|---|---|
|**Detection only**|Notice _that_ an error occurred (and typically request/trigger a retransmission)|
|**Detection + Correction**|Notice _that_ an error occurred, pinpoint _where_, and fix it directly|

> **Analogy — A Spell-Checker vs. Autocorrect:** Error **detection** is like a spell-checker that puts a red squiggly line under a misspelled word — it tells you _something_ is wrong but doesn't fix it for you (you might have to "retransmit" by retyping the word). Error **correction** is like **autocorrect** — it doesn't just flag the mistake, it identifies exactly which letters are wrong and fixes them on the spot, with no need to redo the whole sentence.

---

## 6.1.2 Where Is the Link Layer Implemented?

A natural question once you understand _what_ the link layer does: **where does this actually happen** inside a host? Is it hardware or software? A separate chip? How does it interface with the rest of a host's hardware and OS?

### The Network Adapter (NIC)

For the most part, the link layer is implemented on a chip called the **network adapter**, also known as a **network interface controller (NIC)**. Ethernet capabilities are either integrated directly into the motherboard chipset or implemented via a low-cost dedicated Ethernet chip. The network adapter implements **many** link-layer services — framing, link access, error detection, and so on — meaning **much of a link-layer controller's functionality is implemented in hardware**. (Real-world examples: Intel's 800-series adapters implement the Ethernet protocols; the Atheros Fastconnect 6800 subsystem implements the 802.11 / WiFi protocols.)

### How the Controller Operates

**On the sending side:**

1. The controller takes a datagram that has already been created and stored in **host memory** by the higher layers of the protocol stack (application → transport → network).
2. It **encapsulates** the datagram in a link-layer frame, filling in the frame's various header fields.
3. It **transmits** the frame into the communication link, following the link's access protocol (e.g., waiting its turn if it's a broadcast link).
4. If the link layer performs error detection, it is the **sending controller** that computes and sets the error-detection bits in the frame header.

**On the receiving side:**

1. A controller **receives the entire frame**.
2. It **extracts** the network-layer datagram from the frame.
3. If error detection is in use, it is the **receiving controller** that performs the actual error check.
4. Link-layer software then responds to **controller interrupts** (e.g., triggered by the receipt of one or more frames) — handling error conditions and **passing the extracted datagram up to the network layer**.

```
Fig -- Network Adapter (NIC) in a Host
────────────────────────────────────────────
            HOST
 ┌──────────────────────────────────┐
 │ Application                       │
 │ Transport                         │
 │ Network                           │
 │ Link   ◀── part SOFTWARE (driver) │
 │  ⋮                                 │
 └───────────────┬────────────────────┘
                  │ Motherboard bus
      ┌───────────┴───────────┐
      │                       │
 ┌────▼─────┐          ┌──────▼──────┐
 │   CPU /   │          │  Controller │
 │  Memory   │          │   (NIC)     │◀─ part
 └───────────┘          └──────┬──────┘  HARDWARE
                                │
                         ┌──────▼──────┐
                         │  Physical   │
                         │transmission │
                         └─────────────┘
```

![[Pasted image 20260718000002.png]]

### The Hardware/Software Split

This is the single most important takeaway of Section 6.1.2: **the link layer is a hybrid — part hardware, part software.**

|Portion|What It Does|Where It Lives|
|---|---|---|
|**Hardware portion**|Framing, link access, error detection, activating the controller hardware for transmission|The **controller** chip on the network adapter/NIC|
|**Software portion**|Higher-level link-layer functionality: assembling link-layer addressing information, activating the controller hardware, and — on the receive side — **responding to controller interrupts** (e.g., on frame receipt), handling error conditions, and passing the extracted datagram up to the network layer|Runs on the **host's CPU**, typically as part of the OS's device **driver** for the network adapter|

> **Analogy — A Restaurant's Kitchen Line Cook vs. the Front-of-House Manager:** Think of the **controller (hardware)** as a specialized **line cook** who handles the fast, repetitive, mechanical work — plating dishes in a standard format (framing), waiting for their turn at a shared grill (link access), and doing a quick taste-check for spoiled ingredients (error detection) — all without needing to think or make judgment calls. The **software/driver portion**, running on the host's CPU, is like the **front-of-house manager**: they don't do the mechanical cooking themselves, but they respond when the kitchen "rings the bell" (a controller interrupt signaling a dish/frame is ready), decide what to do if something came out wrong (handling error conditions), and are the ones who actually **carry the finished dish out to the customer** (passing the datagram up to the network layer). Neither role alone is "the kitchen" — it's the _combination_ of the fast hardware line cook and the coordinating software manager that gets a dish from raw ingredients to the customer's table.

> **Why this design makes sense:** Framing, MAC timing, and error-detection computation are all **repetitive, latency-sensitive, bit-level operations** — exactly the kind of work that's fast and efficient when hardwired into a chip, but would be painfully slow (and CPU-hogging) if handled entirely by general-purpose software running on the CPU for every single frame. Meanwhile, tasks like deciding how to respond to an error condition, or handing a datagram up to the next layer of the stack, benefit from the **flexibility** of software — this is precisely the kind of logic that might need to change (via a driver update) without requiring new hardware. This hardware/software boundary is, in the book's own words, **"the place in the protocol stack where software meets hardware."**

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Framing exposes a new attack surface per link**|Since each link along a path can use a _different_ link-layer protocol, an attacker with access to a specific link (e.g., a shared WiFi network) can craft malicious frames using **that link's** framing rules — a technique that wouldn't work against a different link type further along the path|Link-layer security (e.g., WPA on WiFi, port security on Ethernet switches) must be applied **per-link**, since there's no single, uniform link-layer defense that protects the entire end-to-end path|
|**Broadcast links (multiple access) allow eavesdropping and injection**|On a shared broadcast channel, any node "on the line" can potentially **hear all traffic** meant for other nodes, and — absent access control — inject frames pretending to be from another node|Broadcast-link environments motivate encryption above the link layer and access-control mechanisms (e.g., MAC address filtering, WPA authentication) at the link layer itself|
|**The controller performs error checks in hardware, not the software stack**|An attacker who can tamper with signal integrity (e.g., signal jamming) is contending with hardware-level error detection first — but a sufficiently crafted frame that still passes the checksum could slip malicious content through to the software layer above|Because error detection catches _accidental_ bit errors, not malicious tampering, link-layer error-detection bits (like the Internet checksum) should never be relied upon as a **security** mechanism — they protect against noise, not adversaries|
|**The driver (software portion) is a common target for privilege escalation**|Since the software portion of the link layer runs on the host CPU and directly handles interrupts and raw frame data, a vulnerability in a NIC driver can be a path to **kernel-level compromise** from something as simple as a malformed received frame|Keeping NIC drivers patched and minimizing the amount of untrusted frame data parsed in privileged driver code reduces this attack surface|

---

## Questions I Still Have

- [ ] The chapter says many wired link-layer protocols skip reliable delivery because their bit-error rates are already low — but what specific error-rate threshold (if any) determines when a protocol designer decides reliable delivery _is_ worth the overhead?
- [ ] How exactly does the "hardware portion" of the controller decide when to hand off to the "software" driver — is it purely interrupt-driven for every single frame, or do modern high-throughput NICs batch multiple frames before interrupting the CPU (e.g., interrupt coalescing)?
- [ ] Since a router runs the link layer on multiple interfaces (potentially with different link-layer protocols on each), does each interface get its own separate physical NIC/controller, or can one chip handle multiple different link-layer protocols?
- [ ] The travel-agent analogy maps the routing protocol to the travel agent that plans the whole path — but within a single link, who or what plays the role of "deciding" to use WiFi vs. Ethernet on a given hop? Is that purely a matter of physical infrastructure, or is there any dynamic negotiation involved?
- [ ] For broadcast links using MAC protocols, how does the "moderator" (MAC protocol) actually get chosen or configured — is it always statically defined by the link-layer standard (e.g., Ethernet's own MAC protocol), or can different broadcast links on the same network use different MAC protocols?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Node**|Any device running a link-layer (Layer 2) protocol — hosts, routers, switches, and WiFi access points|
|**Link**|The communication channel connecting adjacent nodes along an end-to-end path|
|**Frame**|The link-layer encapsulation of a network-layer datagram, transmitted over a single link|
|**Framing**|The link-layer service of encapsulating a datagram within a frame's data field, bounded by header (and sometimes trailer) fields|
|**MAC (Medium Access Control) protocol**|The set of rules specifying how a frame is transmitted onto a link, especially critical for coordinating access on shared broadcast links|
|**Point-to-point link**|A link with exactly one sender and one receiver; MAC coordination is simple or unnecessary|
|**Broadcast link**|A link shared by multiple nodes, requiring a MAC protocol to coordinate/arbitrate transmissions and avoid collisions|
|**Multiple access problem**|The general challenge of coordinating frame transmissions among many nodes sharing one broadcast channel|
|**Reliable delivery service**|A link-layer guarantee (via acknowledgments and retransmissions) that a datagram crosses a link without error; common on error-prone links like wireless, often omitted on low-error wired links|
|**Error detection**|A link-layer mechanism (via error-detection bits set by the sender and checked by the receiver) for noticing that a bit error occurred in a frame|
|**Error correction**|An extension of error detection that also pinpoints and fixes the exact location of bit errors, without requiring retransmission|
|**Network adapter / Network Interface Controller (NIC)**|The hardware chip that implements most link-layer functionality (framing, link access, error detection) for a host or router interface|
|**Controller**|The hardware component of the network adapter that performs the fast, repetitive link-layer operations|
|**Driver**|The software component (running on the host CPU) that manages the controller — activating hardware, responding to interrupts, handling errors, and passing datagrams up to the network layer|

---

## Related Concepts

---

→ Next: 6.1.3 / 6.2 Error-Detection and Error-Correction Techniques