---
title: MULTIPLEXING AND DEMULTIPLEXING
date: 2026-06-21
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 3.2 Multiplexing and Demultiplexing

> **One-Line Summary:** Multiplexing and demultiplexing are the two halves of one job — taking many application processes on a host and correctly funneling their data down into the network (multiplexing) on the way out, and correctly sorting incoming segments back up to the right process (demultiplexing) on the way in — and the entire mechanism that makes this possible is nothing more exotic than a pair of port numbers stamped on every segment.

---

## Core Idea: From "Which Host" to "Which Process"

Picture the destination host receiving segments from the network layer just below it. The transport layer's job at that moment is to deliver the data in each of those segments to the _correct application process_ running on that host — not just to the host in general, but to one specific process out of possibly many running at once.

Think about your own computer right now: you might be browsing the web while running one Zoom session and two web sessions. That's four separate network application processes active simultaneously (one Zoom process, two web-browser processes, plus whatever else is talking over the network). When a segment lands at the transport layer, the transport layer has to know _which one_ of those four processes the data inside it belongs to, and route it there — and nowhere else.

### Sockets: The Door Between Process and Network

To make this routing possible, every process that wants to send or receive data over the network needs one or more **sockets** — doors through which data passes from the network into the process, and from the process out into the network. A socket is the process's only interface to the transport layer; the process never touches the network layer directly.

The transport layer, sitting in the receiving host, does **not** deliver data directly to a process. It delivers data to a _socket_, and the operating system hands the data the rest of the way from socket to process. Because a host can have many sockets open at once, **each socket is given a unique identifier** — and the exact format of that identifier depends on whether the socket is a UDP socket or a TCP socket (more on this distinction shortly).

> **Why a socket, and not a direct process handoff?** Processes come and go, crash, and get created dynamically — the transport layer can't be expected to track process identities directly. A socket is a stable, OS-managed handle that a process opens once and which the transport layer can reliably target, regardless of what's happening inside the process itself.

### Two Jobs, One Mechanism: Multiplexing and Demultiplexing

Now we can name the two complementary jobs precisely:

|Job|Direction|What Happens|
|---|---|---|
|**Demultiplexing**|Incoming (network → process)|The transport layer examines fields in an arriving segment to identify the correct receiving socket, then directs the segment to that socket|
|**Multiplexing**|Outgoing (process → network)|The transport layer gathers data chunks from multiple sockets at the source host, encapsulates each chunk with header information, and passes the resulting segments down to the network layer|

These two jobs are mirror images of each other — demultiplexing sorts data _up_ toward the right process; multiplexing gathers data _down_ toward the shared network.

### Figure 3.2 — Seeing Multiplexing and Demultiplexing in Action

![[Pasted image 20260621172438.png]]
_(Figure 3.2 — Three end systems are shown, each running the full protocol stack down through Application, Transport, Network, Data Link, and Physical layers. The leftmost host runs application process P1, connected through its own socket to its Transport layer. The middle host runs two application processes, P3 and P4, each with its own socket. The rightmost host runs application process P2. Segments travel from P1's host, across the network, and arrive at the middle host — where the Transport layer must demultiplex each arriving segment to either P3's socket or P4's socket, based on which process the data is actually intended for. Simultaneously, the middle host's Transport layer must multiplex outgoing data gathered from both P3's and P4's sockets into segments, before handing them down to its own Network layer.)_

A simplified view of what's happening at that middle host:

```
                    MIDDLE HOST
                    ───────────
  Segments arrive ──► Transport layer examines
  from the network     header fields in each segment
        │                      │
        │              ┌───────┴───────┐
        │              ▼               ▼
        │         (demux to P3)   (demux to P4)
        │              │               │
        │           Socket P3       Socket P4
        │              │               │
        │              ▼               ▼
        │         Process P3       Process P4

  Meanwhile, going the other direction:

  Process P3 ──► Socket P3 ──┐
                              ├──► Transport layer (MUX) ──► segments ──► down to Network layer
  Process P4 ──► Socket P4 ──┘
```

It's worth pausing on one detail buried in that description: **a single protocol at one layer can be used by multiple protocols at the layer just above it.** Even though this chapter introduces multiplexing and demultiplexing specifically in the context of Internet transport protocols, the same general need — sorting and gathering data correctly between layers — exists at every layer of every protocol stack, not just here.

---

## Revisiting the Household Analogy for Mux/Demux

Recall the household analogy from Section 3.1.1: twelve kids in each house, with Ann managing mail on the West Coast and Bill managing it on the East Coast. That same analogy maps perfectly onto multiplexing and demultiplexing:

- **Demultiplexing** = what Bill does when a batch of mail arrives from the carrier. He looks at the name on each letter and delivers it to the correct sibling. He's taking one undifferentiated batch (addressed only to "the house") and sorting it down to individual recipients.
- **Multiplexing** = what Ann does when she collects letters from her brothers and sisters before the carrier arrives. She's taking many individually-addressed letters and gathering them into one outgoing batch, handed to the postal service as a single unit.

> **Analogy callout — Same Mechanism, Both Directions:** Notice Ann and Bill each do _both_ jobs, just for opposite directions of traffic — Ann multiplexes outgoing mail and demultiplexes incoming mail at her house; Bill does the same at his. This is exactly how a real host behaves: the same transport layer on the same machine multiplexes data going out to the network and demultiplexes data coming in from it, simultaneously, for many processes at once.

---

## How Sockets Actually Get Identified: Port Numbers

Now that the _purpose_ of multiplexing/demultiplexing is clear, the natural next question is: _how_ does the transport layer actually distinguish one socket from another? The answer is a pair of fields stuffed into every transport-layer segment's header.

Transport-layer multiplexing requires two ingredients:

1. **Sockets must have unique identifiers.**
2. **Each segment must carry special fields indicating which socket it's meant for.**

Those special fields are the **source port number field** and the **destination port number field**, found in the header of every transport-layer segment (both UDP and TCP carry other header fields too, covered in their own sections later — but these two port fields are common to both).

### Figure 3.3 — Where Port Numbers Live in a Segment

![[Pasted image 20260621172642.png]]
_(Figure 3.3 — A 32-bit-wide diagram of the start of a generic transport-layer segment header. The top row is split into two 16-bit fields side by side: "Source port #" on the left, "Dest. port #" on the right. Below that sits a row labeled "Other header fields" — the rest of whatever the specific protocol, UDP or TCP, needs. Below that sits the "Application data (message)" — the actual payload being carried.)_

```
 0                   16                  32   (bit position)
 ┌──────────────────┬──────────────────┐
 │   Source port #   │   Dest. port #   │
 ├──────────────────┴──────────────────┤
 │           Other header fields        │
 ├───────────────────────────────────────┤
 │     Application data (message)        │
 └───────────────────────────────────────┘
```

### Port Number Basics

|Property|Value|
|---|---|
|**Size**|16-bit number|
|**Range**|0 to 65535|
|**"Well-known" range**|0–1023 — reserved for well-known application protocols|
|**Examples of well-known ports**|HTTP → port 80, FTP → port 21|
|**Governing reference**|Originally listed in RFC 1700, now maintained and updated at iana.org (RFC 3232 — IANA's port registry supersedes the older RFC 1700 list)|

When you build a new network application (recall the simple socket-programming example from Section 2.6), you — the developer — must assign that application a port number.

> **Why are ports 0–1023 special?** They're _restricted_, meaning ordinary applications aren't supposed to claim them. They exist so that, no matter which machine you connect to anywhere on the Internet, "port 80" reliably means "talk to whatever's running the web server" without each server administrator having to publish a custom port number. It's a shared convention, not a technical limitation — nothing stops a misbehaving program from trying to use a low port number other than OS-level permission restrictions in practice.

### The Core Mechanism, Stated Plainly

With port numbers in hand, here's exactly how the transport layer _could_ implement demultiplexing (and, as you'll see, this is essentially what UDP actually does):

> Each socket on the host is assigned a port number. When a segment arrives at the host, the transport layer examines the segment's destination port number and directs the segment to the socket that has that port number attached. The segment's data then passes up through that socket into the attached process.

TCP's version of this mechanism, as you'll see in a moment, is a bit more subtle than UDP's — it uses more than just the destination port number alone.

---

## 3.2.1 Connectionless Multiplexing and Demultiplexing with UDP

### How a UDP Socket Gets Its Port Number

Recall from Section 2.6.1 that a Python program creates a UDP socket with:

```python
clientSocket = socket(AF_INET, SOCK_DGRAM)
```

The moment this line executes, **the transport layer automatically assigns the socket a port number** — specifically, a number somewhere in the range 1024 to 65535, chosen from whatever happens to currently be unused by any other UDP port on that host.

Alternatively, the program can request a _specific_ port number explicitly, by calling `bind()` right after creating the socket:

```python
clientSocket.bind(('', 19157))
```

### Who Picks the Port: Client vs. Server Convention

|Side of the Application|How the Port Gets Assigned|
|---|---|
|**Client side**|Typically left to the transport layer — it auto-assigns a port transparently, and the application developer doesn't need to think about it|
|**Server side**|Must assign a _specific_ port — because the server is implementing a well-known protocol, clients need to know in advance which port to connect to|

This is exactly why a developer implementing the server side of a well-known protocol must explicitly bind to that protocol's reserved well-known port number — there's no "auto-assign" option available there; the whole point of a well-known port is that it's predictable in advance.

### Walking Through UDP Demultiplexing, Precisely

Suppose a process on Host A, using UDP port 19157, wants to send a chunk of application data to a process on Host B, using UDP port 46428. Here's the exact sequence:

1. The transport layer at Host A creates a transport-layer segment containing: the application data, the source port number (19157), and the destination port number (46428) — plus a couple of other fields not relevant here.
2. That segment gets handed to the network layer at Host A, which encapsulates it inside an IP datagram and makes a **best-effort** attempt to deliver it to Host B (recall from Section 3.1.2: IP gives no guarantees).
3. If the segment successfully arrives, the transport layer at Host B examines the **destination port number** field (46428) in the segment.
4. Host B delivers the segment to whichever socket is identified by port 46428.

```
   Host A                                      Host B
   ──────                                      ──────
Process (UDP port 19157)                Process (UDP port 46428)
        │                                          ▲
        ▼                                          │
   Transport layer builds segment:                 │
   [src port: 19157 | dst port: 46428 | data]       │
        │                                          │
        ▼                                          │
   Network layer → IP datagram → best-effort  ──────┘
   delivery across the network          (Transport layer at B reads
                                          dst port 46428, delivers
                                          to that socket)
```

Note that Host B could be running **multiple** processes simultaneously, each with its own UDP socket and its own associated port number. As segments arrive from the network, Host B sorts (demultiplexes) each one to the appropriate socket purely by examining the segment's destination port number.

### The Critical Rule: UDP Sockets Are Identified by a Two-Tuple

> A UDP socket is fully identified by a **two-tuple**: (destination IP address, destination port number).

This has a direct, important consequence: **if two UDP segments have different source IP addresses and/or source port numbers, but the _same_ destination IP address and _same_ destination port number, both segments get directed to the same destination socket** — and therefore the same destination process. The source fields play no role at all in UDP demultiplexing.

### Figure 3.4 — UDP Multiplexing/Demultiplexing Example

![[Pasted image 20260621173440.png]]
_(Figure 3.4 — Host A on the left runs a client process attached to a socket; Server B on the right is shown as a separate machine. An A-to-B segment is drawn carrying source port 19157 and destination port 46428. A return, B-to-A segment is drawn below it carrying source port 46428 and destination port 19157 — i.e., the source and destination port values have swapped roles for the return trip.)_

### Then What's the Source Port _For_, If Demux Only Uses the Destination?

This is a fair question, since the rule above only mentioned the destination two-tuple. The source port number serves as part of a **"return address."**

When B wants to send a segment back to A, the destination port in that B-to-A segment takes its value directly from the **source** port of the original A-to-B segment. (The complete return address is actually A's IP address _plus_ the source port number — both are needed, since IP address alone only gets you back to the right host, not the right socket on that host.)

This is exactly what you've already seen in practice: in `UDPServer.py` from Section 2.7, the server calls `recvfrom()` specifically to extract the client's source port number out of the incoming segment — and then uses that extracted value as the destination port when sending its reply back to the client.

> **Analogy — The Return Address on an Envelope:** The source port is like the return address printed on an envelope. The postal service (network layer) doesn't need it to deliver the letter forward — that only needs the destination address. But the _recipient_ needs it if they ever want to write back. UDP's source port field exists purely so the receiving side has somewhere to send a reply.

---

## 3.2.2 Connection-Oriented Multiplexing and Demultiplexing with TCP

### The Key Difference: Four-Tuple, Not Two-Tuple

TCP demultiplexing requires a closer look at TCP sockets and TCP connection establishment, because TCP behaves differently from UDP in one important way:

> A **TCP socket** is identified by a **four-tuple**: (source IP address, source port number, destination IP address, destination port number).

Because all four values are used, **two arriving TCP segments with different source IP addresses or source port numbers will be directed to two different sockets** — even if both segments share the exact same destination IP address and destination port number. (The one exception is a TCP segment carrying the _original_ connection-establishment request, which is handled slightly differently — explained next.)

|Protocol|Socket Identified By|Consequence|
|---|---|---|
|**UDP**|Two-tuple: (dest IP, dest port)|Source fields are ignored for demux — many different senders sharing a destination all land on the same socket|
|**TCP**|Four-tuple: (src IP, src port, dest IP, dest port)|Every distinct client connection gets routed to its _own_ dedicated socket, even when all clients are talking to the same destination port|

### Walking Through TCP Connection Establishment and Demultiplexing

Reconsider the TCP client-server programming example from Section 2.6.2, step by step:

1. **The server has a "welcoming socket"** that sits and waits for connection-establishment requests from TCP clients, on a specific port — say, port 12000.
2. **The client creates a socket and sends a connection-establishment request:**

```python
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName, 12000))
```

3. **A connection-establishment request is, mechanically, nothing more than a TCP segment** with destination port number 12000 and a special connection-establishment bit set in the TCP header (the details of this bit are covered later, in Section 3.5). The segment also carries whatever source port number the client's transport layer chose.
4. **When the server's operating system receives this incoming connection-request segment**, it locates the server process that's waiting to accept connections on port 12000. That server process then creates a **brand-new socket**:

```python
connectionSocket, addr = serverSocket.accept()
```

3. **At the moment this new socket is created**, the transport layer at the server records four specific values from the connection-request segment: (1) the source port number in the segment, (2) the IP address of the source host, (3) the destination port number in the segment, and (4) its own (the server's) IP address.
4. **This newly created connection socket is now identified by exactly those four values.** Every subsequently arriving segment whose source port, source IP address, destination port, and destination IP address all match these four recorded values will be demultiplexed straight to this socket.
5. With the connection now established, the client and server can freely exchange data with each other through their respective sockets.

```
  CLIENT                                         SERVER
  ──────                                         ──────
clientSocket.connect()                  serverSocket (welcoming, port 12000)
       │                                            │
       ▼                                            ▼
 Connection-request segment:           OS sees request, hands it to the
 [src port: X, dst port: 12000,        server process waiting on port 12000
  conn-establishment bit SET]                       │
       │                                            ▼
       └──────────────────────────────►   connectionSocket, addr =
                                              serverSocket.accept()
                                                     │
                                          Server now records the 4-tuple:
                                          (client src port, client IP,
                                           12000, server's own IP)
                                                     │
                                          All future segments matching
                                          that 4-tuple → this new socket
```

### Why a Server Can Juggle Many Connections at Once

A server host may simultaneously support **many TCP connection sockets**, with each socket attached to its own process, and each socket identified by its own unique four-tuple. Whenever a TCP segment arrives at the host, **all four fields** — source IP address, source port, destination IP address, destination port — are used together to direct (demultiplex) that segment to the correct socket.

This is precisely why the same destination port number (say, port 80 for a web server) can be shared by thousands of simultaneous client connections without any confusion: each client brings a different source IP and/or source port, which makes its four-tuple unique even though the destination two values are identical for everyone.

---

## Focus on Security: Port Scanning

A server process waits patiently on an open port for contact from a remote client. Some ports are reserved by convention for well-known applications (Web, FTP, DNS, SMTP servers); others are used by convention for popular applications (for example, Microsoft's SQL Server traditionally listens on UDP port 1434).

Knowing which port is open on a host can reveal exactly which application is running there — which is genuinely useful information for two very different audiences:

|Who Wants This Information|Why|
|---|---|
|**System administrators**|To understand which network applications are actually running across the hosts on their own network|
|**Attackers**|To "case the joint" — scouting which ports are open on a _target_ host, so that if an application with a known security flaw is found running on an open port, that host becomes a candidate for attack|

Determining which applications are listening on which ports turns out to be a relatively easy task, thanks to a class of public-domain tools called **port scanners** — the most widely used of which is **nmap**, freely available and bundled with most Linux distributions.

|Protocol Being Scanned|How nmap Probes It|
|---|---|
|**TCP**|Sequentially scans ports, looking for ports that will accept TCP connections|
|**UDP**|Sequentially scans ports, looking for UDP ports that respond to transmitted UDP segments|

In both cases, a host running nmap returns a list of ports reported as open, closed, or unreachable — and crucially, a host running nmap can attempt to scan _any_ target host anywhere on the Internet.

> **Why this matters in context:** Port scanning is only possible _because_ of the exact mechanism this section just explained — port numbers are how the transport layer routes segments to processes, and that same addressability is what lets an external party probe, port by port, to learn what's running on a remote machine.

---

## Figure 3.5 and the Web-Server Case

![[Pasted image 20260621174140.png]]
_(Figure 3.5 — Two separate web client hosts, C and A, both initiate HTTP connections to the same web server, Host B. Host C opens two simultaneous HTTP sessions to B, using two different source ports, 7532 and 26145. Host A opens one HTTP session to B, independently choosing source port 26145 — coincidentally the same number Host C used for one of its connections. All three connections share the identical destination port, 80, and the identical destination IP, B. On the server side, B is shown spawning a separate per-connection HTTP process for each incoming connection, with the server's transport layer performing the demultiplexing that sorts each arriving segment to its correct process.)_

This scenario illustrates exactly why the four-tuple matters: Host C assigns its two source port numbers (26145 and 7532) independently of Host A — meaning Host A could easily, by coincidence, pick a source port (26145) that Host C is _also_ using. **This is not a problem.** Server B can still correctly demultiplex both connections that happen to share the same source port number, because they arrive from **different source IP addresses** — and the four-tuple (which includes source IP) is what actually distinguishes them, not the source port in isolation.

### Web Servers and TCP: A Closing Note

Consider a host running a web server (say, Apache) on port 80. When clients — browsers — send segments to this server, **all** of those segments will carry destination port 80, both the initial connection-establishment segments _and_ the segments carrying the actual HTTP request messages. The server tells different clients' traffic apart using source IP addresses and source port numbers, exactly as just described.

Figure 3.5 depicts a server that spawns a brand-new _process_ for each connection, with each such process owning its own connection socket through which HTTP requests arrive and HTTP responses are sent. In practice, though, there is **no strict one-to-one rule** requiring a connection socket to map to a full process:

|Implementation Choice|Description|
|---|---|
|**New process per connection**|Simple to reason about, but heavier-weight — what Figure 3.5 depicts|
|**New thread per connection**|What today's high-performing web servers typically do instead — a single process spins up a lightweight thread (essentially a "lightweight subprocess") with a new connection socket for each new connection|

At any given time, a single process on a busy server may have **many** connection sockets attached to it simultaneously, each with its own distinct identifier (its own four-tuple).

The lifetime of these sockets also depends on the kind of HTTP being used:

|HTTP Mode|Socket Behavior|
|---|---|
|**Persistent HTTP**|Client and server keep exchanging _all_ HTTP messages over the _same_ server socket for the entire duration of the connection|
|**Non-persistent HTTP**|A brand-new TCP connection is created and closed for _every single_ request/response pair — meaning a new socket is created and later closed for every request/response|

This frequent creating and closing of sockets under non-persistent HTTP can noticeably hurt the performance of a busy web server — though a number of operating-system-level tricks exist to mitigate that cost (readers interested in the deeper OS mechanics here are pointed to further reading: Nielsen 1997 and Nahum 2002 in the original text).

With multiplexing and demultiplexing now fully explained, the next section turns to the first of the Internet's two transport protocols — UDP — where you'll see that UDP adds remarkably little on top of the network layer beyond exactly this multiplexing/demultiplexing service.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Port numbers are the entire basis of demultiplexing**|Because every socket is identified by predictable, enumerable 16-bit values (0–65535), an attacker can systematically probe every port on a target host to discover which applications are listening — this is exactly what port scanning exploits|Restrict exposed ports to only what's needed; use firewalls to block scans on unused ports; monitor for the sequential, port-by-port probing pattern characteristic of tools like nmap|
|**UDP demultiplexes on destination two-tuple only, ignoring source**|Since UDP's socket identity never checks the source IP or source port, an attacker can trivially spoof a fake source address on a UDP segment with zero impact on whether it gets delivered — the destination socket has no way to detect the forgery from demultiplexing alone|Never rely on a UDP source port/IP as an authentication signal; any identity verification for UDP-based applications must happen at the application layer (e.g., via cryptographic protocols), not by trusting the transport layer's routing|
|**TCP's four-tuple gives stronger destination-side identity than UDP, but still no authentication**|An attacker who can guess or sniff a valid four-tuple (especially with weak/predictable initial sequence numbers, covered later in TCP security) can attempt to inject or hijack traffic into an existing TCP connection, since the four-tuple alone — not cryptographic proof — is what the connecting socket trusts|Use TLS on top of TCP for genuine endpoint authentication; don't treat "matches the four-tuple" as equivalent to "is who it claims to be"|
|**Port scanning reveals attack surface directly**|Identifying which application is listening on an open port lets an attacker target _known_ vulnerabilities in that specific application (the chapter's own example: a SQL server with a known buffer-overflow flaw, exploited historically by the Slammer worm)|Patch known vulnerabilities promptly in any service exposed on an open port; minimize the number of open, externally-reachable ports in the first place — the smaller the visible surface, the less a scan can reveal|
|**Well-known ports are predictable by design**|Because ports 0–1023 are standardized and public knowledge, an attacker doesn't even need to scan to guess that port 80 means HTTP or port 21 means FTP — the convention that makes the Internet interoperable also makes default attack targets obvious|Don't rely on "obscure" non-standard ports as a security measure (security through obscurity is weak); assume any well-known service is a known target and harden it accordingly|

---

## Questions I Still Have

- [ ] If TCP's four-tuple already uniquely identifies a connection without needing a dedicated thread or process, why does the textbook (and real servers) still typically spin up a new thread per connection — is that purely for application-level convenience, or does the OS actually need a separate execution context to manage each socket's buffers/state?
- [ ] The source port for UDP exists purely as a "return address" — but what happens if the same client process sends segments to two _different_ servers from the _same_ source port? Do both servers see the same return address and could that ever cause confusion on the client's own demultiplexing?
- [ ] For TCP's "exception" case — the very first connection-request segment, before the four-tuple-identified socket exists yet — which socket actually receives that initial segment, and how does the OS know to route it there instead of to an established connection socket?
- [ ] Nmap can scan "any target host anywhere in the Internet" — does this mean port scanning is, by default, not illegal or blocked anywhere, or is this purely a technical capability statement separate from legal/policy boundaries?
- [ ] If a busy server is suffering performance problems from non-persistent HTTP's constant socket creation/teardown, what are the actual OS-level mitigations referenced (Nielsen 1997, Nahum 2002) — connection pooling? SO_REUSEADDR-style socket reuse? Something else?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Multiplexing**|The transport-layer job of gathering data chunks from multiple sockets at the sending host, encapsulating each chunk into a segment, and passing the segments down to the network layer|
|**Demultiplexing**|The transport-layer job of examining fields in an arriving segment to identify the correct receiving socket, and delivering the segment's data to that socket|
|**Socket**|A process's door to the network — the interface through which a process sends data to, and receives data from, the transport layer; uniquely identified so the transport layer can target it correctly|
|**Port number**|A 16-bit number (0–65535) used to identify a specific socket on a host; carried in both the source port and destination port fields of a transport-layer segment header|
|**Well-known port number**|A port number in the range 0–1023, reserved for well-known application protocols (e.g., HTTP on port 80, FTP on port 21), as listed and maintained by IANA|
|**Source port number field**|A field in the segment header identifying the port the segment was sent _from_; primarily serves as part of a "return address" for replies, and (for TCP) as part of the socket-identifying four-tuple|
|**Destination port number field**|A field in the segment header identifying the port the segment is meant to be delivered _to_; the primary field UDP uses for demultiplexing|
|**UDP socket two-tuple**|(Destination IP address, destination port number) — the complete identifying information for a UDP socket; source fields play no role in UDP demultiplexing|
|**TCP socket four-tuple**|(Source IP address, source port number, destination IP address, destination port number) — the complete identifying information for a TCP connection socket|
|**Welcoming socket**|The socket a TCP server process uses to listen for and accept incoming connection-establishment requests, before any individual client connection has been established|
|**Connection socket**|The new, dedicated socket a TCP server creates (via `accept()`) for each individual client connection, identified by that connection's unique four-tuple|
|**Port scanning**|The technique — exemplified by tools like nmap — of probing a host port-by-port to determine which ports are open, closed, or unreachable, revealing which applications are running on that host|
|**Persistent HTTP**|An HTTP mode where client and server exchange all messages over a single, long-lived TCP connection socket|
|**Non-persistent HTTP**|An HTTP mode where a new TCP connection (and therefore a new socket) is created and closed for every single request/response pair|

---

## Related Concepts

---

→ Next: [[UDP]]