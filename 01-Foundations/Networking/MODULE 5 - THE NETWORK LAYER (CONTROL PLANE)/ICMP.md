---
title: ICMP
date: 2026-07-08
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 5.6 ICMP: The Internet Control Message Protocol

> **One-Line Summary:** **ICMP (Internet Control Message Protocol)** is the network layer's built-in "error-reporting and diagnostics" channel — hosts and routers use it to tell each other about problems (unreachable destinations, expired TTLs, bad headers) and to run simple network-layer utilities like **ping** (echo request/reply) and **traceroute** (which cleverly abuses ICMP TTL-expired and port-unreachable messages to map the path to a destination), with every ICMP message itself riding inside an IP datagram as if it were just another transport-layer payload.

---

## Core Idea: The Network Layer's Feedback Mechanism

IP itself is a "best-effort," unreliable delivery service — it has no built-in way to tell a sender that something went wrong. **ICMP fills that gap.** It is used by hosts and routers to communicate network-layer information to each other, and its most typical use is **error reporting**. The classic example: during an HTTP session, a **"Destination network unreachable"** message traces its origin back to ICMP — at some point an IP router was unable to find a path to the host specified in the request, so that router created and sent an ICMP message back to the source indicating the error.

> **Analogy — The Postal Service's "Return to Sender" Slip:** IP is like a postal worker who just drops letters in mailboxes and moves on — it doesn't apologize or explain if a letter can't be delivered. ICMP is the "Return to Sender: Address Unknown" sticker the postal system attaches when something goes wrong. It's not a separate delivery service; it rides back through the same postal system (IP) that failed to deliver the original letter, carrying just enough information (a stamped, folded copy of the front of the original envelope) for the sender to know what failed and why.

### Where ICMP Lives: "Above IP," but Really a Network-Layer Helper

ICMP is often considered part of IP, but architecturally it sits **just above IP**, because ICMP messages are carried **inside IP datagrams** — carried as IP payload, exactly the same way TCP or UDP segments are carried as IP payload. When a host receives an IP datagram with ICMP specified as the upper-layer protocol (upper-layer protocol number **1**), it demultiplexes the datagram's contents to ICMP, just as it would demultiplex a datagram's contents to TCP or UDP.

```
Fig -- Where ICMP Sits in the Protocol Stack
────────────────────────────────────────────

  ┌────────────────────────────────────────┐
  │           APPLICATION LAYER             │
  │     (HTTP, DNS, SMTP, etc.)             │
  └───────────────────┬──────────────────────┘
  ┌───────────────────▼──────────────────────┐
  │  TRANSPORT LAYER   │  ICMP (logically     │
  │  (TCP / UDP)        │  "above" IP, but    │
  │                      │  really network-    │
  │                      │  layer helper)      │
  └───────────────────┬──────────────────────┘
  ┌───────────────────▼──────────────────────┐
  │           NETWORK LAYER (IP)              │
  │  ICMP messages are carried AS THE          │
  │  PAYLOAD of an IP datagram, exactly        │
  │  like a TCP segment or UDP segment          │
  └────────────────────────────────────────────┘

  IP datagram carrying ICMP:
  ┌───────────────┬───────────────────────┐
  │  IP header     │  ICMP message         │
  │  (proto = 1)   │  (type | code | ...)  │
  └───────────────┴───────────────────────┘
```

This is a subtle but important distinction: ICMP is _conceptually_ an upper-layer protocol from IP's point of view (it gets a protocol number and is demultiplexed like TCP/UDP), yet _functionally_ it exists purely to serve the network layer's own diagnostic and error-reporting needs — it isn't something an ordinary application talks to directly the way it talks to TCP or UDP.

---

## ICMP Message Structure

Every ICMP message contains three key components:

1. A **type** field
2. A **code** field
3. The **header and the first 8 bytes of the IP datagram that caused the ICMP message to be generated in the first place**

That third component is what lets the original sender figure out _which_ datagram triggered the error — the router or host generating the ICMP message essentially attaches a small "receipt" of the offending packet so the source can correlate the error back to a specific transmission.

```
Fig -- ICMP Header Layout
────────────────────────────────────────────

   0        8        16              31
  ┌────────┬────────┬──────────────────┐
  │  Type  │  Code  │    Checksum      │
  ├────────┴────────┴──────────────────┤
  │      (type/code-specific data)      │
  ├──────────────────────────────────────┤
  │  Header + first 8 bytes of the       │
  │  IP datagram that caused the error   │
  └──────────────────────────────────────┘
```

Note that ICMP messages are not used **only** for signaling error conditions — some types (like echo request/reply, used by ping) are purely informational/diagnostic rather than error reports.

### ICMP Type/Code Reference Table

|ICMP Type|Code|Description|
|---|---|---|
|**0**|0|Echo reply (to ping)|
|**3**|0|Destination network unreachable|
|**3**|1|Destination host unreachable|
|**3**|2|Destination protocol unreachable|
|**3**|3|Destination port unreachable|
|**3**|6|Destination network unknown|
|**3**|7|Destination host unknown|
|**4**|0|Source quench (congestion control)|
|**8**|0|Echo request|
|**9**|0|Router advertisement|
|**10**|0|Router discovery|
|**11**|0|TTL expired|
|**12**|0|IP header bad|

> **Analogy — Type = Category, Code = Subcategory:** Think of _type_ as a broad department (e.g., type 3 = "delivery failed") and _code_ as the specific reason within that department (e.g., code 3 = "the package arrived at the right building, but no one at that specific mailbox slot was expecting it" — i.e., port unreachable). This two-level scheme lets ICMP be precise about _why_ something failed without needing a unique type for every possible failure reason.

---

## Ping: ICMP Echo Request / Echo Reply

The well-known **ping** program sends an **ICMP type 8, code 0** message ("echo request") to the specified host. The destination host, seeing the echo request, sends back a **type 0, code 0 ICMP echo reply**. Most TCP/IP implementations support the ping server directly in the operating system — that is, the server is not a separate user-level process, but built into the OS's networking stack itself.

```
Fig -- Ping: ICMP Echo Request / Echo Reply
────────────────────────────────────────────

   HOST A                          HOST B
  (source)                     (destination)

     │   ICMP type 8, code 0        │
     │   "Echo Request"              │
     ├──────────────────────────────▶│
     │                                │
     │   ICMP type 0, code 0         │
     │   "Echo Reply"                 │
     │◀──────────────────────────────┤
     │                                │

  Round-trip time (RTT) = time from
  sending Echo Request to receiving
  matching Echo Reply.
```

The client program itself needs to be able to instruct the operating system to generate an ICMP echo-request message of type 8, code 0 — the actual echo-reply generation on the far end, however, is typically handled transparently by the destination's OS kernel, with no user-level "ping server" process needed at all.

---

## Traceroute: Mapping a Path by Deliberately Failing

**Traceroute** is implemented entirely using ICMP messages, but in a clever indirect way — it doesn't ask routers "who are you?" directly. Instead, it exploits the router's _own error-reporting behavior_ to reveal the path hop by hop.

### How It Works, Step by Step

To determine the names and addresses of the routers between a source and a destination, Traceroute sends a **series of ordinary IP datagrams** to the destination. Each of these datagrams carries a **UDP segment with an unlikely (deliberately unused) UDP port number**.

- The **first** datagram has a **TTL of 1**
- The **second** has a **TTL of 2**
- The **third** has a **TTL of 3**
- ...and so on

The source also starts a timer for each of the datagrams it sends. When the _n_-th datagram arrives at the _n_-th router along the path, that router observes that the datagram's TTL has just expired. According to the rules of the IP protocol, a router **must discard** a datagram whose TTL has expired. In addition, the router typically sends an **ICMP warning message (type 11, code 0)** back to the source. This warning message includes the **name of the router and its IP address**. When this ICMP message arrives back at the source, the source obtains the **round-trip time** (from its timer) and the **name and IP address of the n-th router** (from the ICMP message).

### How Does Traceroute Know When to Stop?

Recall that the source increments the TTL field for each successive datagram it sends. Thus, **one of the datagrams will eventually make it all the way to the destination host**. Because that final datagram contains a UDP segment with a deliberately unlikely (unused) port number, the destination host responds with an **ICMP type 3, code 3 "port unreachable"** message back to the source. When the source host receives this particular ICMP message, **it knows it does not need to send any additional probes** — the path has been fully traced.

```
Fig -- Traceroute: TTL-Escalation Mechanism
────────────────────────────────────────────

  SOURCE            R1        R2        DEST
    │                │          │          │
    │ UDP datagram   │          │          │
    │ TTL=1, odd port│          │          │
    ├───────────────▶│          │          │
    │                │ TTL→0    │          │
    │  ICMP type 11  │ discard  │          │
    │  code 0 (TTL   │ + notify │          │
    │  expired)      │          │          │
    │◀───────────────┤          │          │
    │  (has R1 name  │          │          │
    │   + IP, RTT)   │          │          │
    │                │          │          │
    │ UDP datagram   │          │          │
    │ TTL=2          │          │          │
    ├───────────────▶├─────────▶│          │
    │                │          │ TTL→0    │
    │        ICMP type 11 code 0│ discard  │
    │◀───────────────────────────┤          │
    │                │          │          │
    │ UDP datagram   │          │          │
    │ TTL=3          │          │          │
    ├───────────────▶├─────────▶├─────────▶│
    │                │          │          │ port
    │                │          │          │ unreachable
    │      ICMP type 3 code 3 (port unreach)│
    │◀────────────────────────────────────────┤
    │                                        │
    │  Source sees port-unreachable msg →   │
    │  STOP sending further probes           │
```

In this manner, the source host learns both the **number and identities of the routers** that lie between it and the destination, as well as the **round-trip time** between the two hosts. Note that the Traceroute client program must be able to instruct the operating system to generate UDP datagrams with **specific TTL values**, and must also be able to be notified by its operating system when ICMP messages arrive. In practice, the standard Traceroute program actually sends **sets of three packets at each TTL value**; thus, the Traceroute output provides **three results for each TTL** (giving three RTT samples per hop rather than just one).

> **Analogy — The "Return to Sender, Ran Out of Postage Stamps" Trick:** Imagine you want to discover every post office a letter passes through on its way across the country, but post offices won't just tell you their names if you ask directly. So instead, you deliberately under-stamp a series of letters just enough that the _first_ letter runs out of postage exactly at the first post office, the _second_ runs out at the second post office, and so on — each post office that "runs out of postage" on your letter stamps a rejection slip with its own name and mails it back to you. Once a letter has _enough_ postage to reach the true destination, but you addressed it to a made-up department that doesn't exist there, the final destination sends back its own "no such department here" slip — telling you the letter has fully arrived and you can stop the experiment. TTL is the "postage," ICMP TTL-expired is the "ran out of postage" slip, and ICMP port-unreachable is the "arrived, but no such department" slip.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**ICMP echo request/reply (ping) reveals host liveness**|Attackers routinely use ping sweeps to discover which hosts on a network are alive before launching a targeted attack — it's cheap, fast reconnaissance|Many networks block or rate-limit inbound ICMP echo requests at the perimeter to reduce the ease of host discovery, at the cost of making legitimate diagnostics harder|
|**Traceroute reveals internal network topology and router identities/addresses**|An attacker can map an organization's internal routing path, learn router IP addresses, and infer network structure — useful for planning further attacks or identifying choke points|Some organizations suppress or filter ICMP TTL-expired responses from internal routers to obscure topology from external probes, though this can break legitimate path diagnostics for partners/customers|
|**ICMP error messages echo back the header and first 8 bytes of the original datagram**|This "receipt" mechanism can be abused for information leakage or crafted to trigger malformed/oversized ICMP responses in poorly implemented stacks (historically a source of buffer-overflow-style vulnerabilities, e.g., the "ping of death")|Modern stacks strictly validate and bound ICMP payload sizes; firewalls often inspect ICMP payloads to ensure they correspond to genuinely recent, legitimate outbound traffic before allowing the error back in|
|**ICMP can be used as a covert channel or DoS vector**|Because ICMP payloads can carry arbitrary data and are often permitted through firewalls for diagnostic reasons, attackers have used ICMP tunneling for covert data exfiltration, and ICMP floods (e.g., "ping flood," "smurf attacks") for denial of service|Rate-limiting ICMP traffic, disabling directed broadcast forwarding (which prevents smurf-style amplification), and monitoring for anomalous ICMP payload sizes/volumes are standard mitigations|
|**Source quench (type 4) was originally intended as a congestion signal**|Because source quench messages could be trivially spoofed by an attacker to force a victim to needlessly throttle its own sending rate, this created a denial-of-service opportunity|Source quench is now largely deprecated/ignored by modern TCP/IP stacks in favor of transport-layer congestion control (Section 3.6), precisely because of this spoofing weakness|

---

## Questions I Still Have

- [ ] The text notes traceroute sends UDP segments with an "unlikely" port number — how is that port chosen in practice (fixed default, or does it increment per probe as some implementations do), and does that choice matter for correctness?
- [ ] Modern traceroute implementations on some OSes (e.g., Windows' `tracert`) actually use ICMP echo requests instead of UDP datagrams for the probes themselves — how does the TTL-expiration logic differ (if at all) when ICMP is used both as the probe _and_ the response mechanism?
- [ ] Why was source quench (type 4) effectively abandoned in favor of transport-layer mechanisms like TCP congestion control (Section 3.6) rather than being fixed/secured at the network layer?
- [ ] Given that ICMP TTL-expired messages reveal router identity/address, how commonly do real-world ISPs actually suppress these in practice, and does that meaningfully break end-user path diagnostics?
- [ ] How does IPv6's ICMPv6 differ structurally from ICMPv4 in terms of type/code numbering and added responsibilities (e.g., Neighbor Discovery folded into ICMPv6)?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**ICMP (Internet Control Message Protocol)**|A network-layer protocol, specified in RFC 792, used by hosts and routers to communicate network-layer information — primarily error reporting — to each other|
|**ICMP message**|Consists of a type field, a code field, and the header plus first 8 bytes of the IP datagram that caused the message to be generated|
|**Type field**|Broad category of an ICMP message (e.g., type 3 = destination unreachable, type 11 = time exceeded)|
|**Code field**|Sub-category within a type, giving a more specific reason (e.g., type 3 code 3 = port unreachable)|
|**Upper-layer protocol number 1**|The IP protocol number used to indicate that a datagram's payload is an ICMP message, allowing correct demultiplexing at the receiving host|
|**Echo request (type 8, code 0)**|The ICMP message sent by the `ping` program to a target host|
|**Echo reply (type 0, code 0)**|The ICMP message a host sends back in response to an echo request|
|**Ping**|A diagnostic program that sends ICMP echo requests and measures round-trip time based on received echo replies; typically implemented directly in the OS kernel rather than as a user-level server|
|**TTL expired (type 11, code 0)**|The ICMP message a router sends back to a datagram's source when that datagram's Time-To-Live field reaches zero and the datagram is discarded|
|**Port unreachable (type 3, code 3)**|The ICMP message a destination host sends when it receives a UDP datagram addressed to a port with no listening process — used by traceroute to detect it has reached the final destination|
|**Traceroute**|A diagnostic program that discovers the routers along a path to a destination by sending UDP datagrams with successively incrementing TTL values and interpreting the resulting ICMP TTL-expired and port-unreachable messages|
|**Round-trip time (RTT)**|The time measured from sending a probe (echo request or TTL-limited datagram) to receiving the corresponding ICMP response|
|**Source quench (type 4, code 0)**|A largely deprecated ICMP message originally intended as a network-layer congestion-control signal|

---

## Related Concepts

---

→ Next: [[NETWORK MANAGEMENT]]