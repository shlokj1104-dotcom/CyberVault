---
title: Principles of Network Applications
date: 2026-05-19
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.1 Principles of Network Applications

> **One-Line Summary:** Network applications are programs running on different end systems that communicate over the network — structured as either client-server or P2P architectures, communicating via sockets, and choosing between TCP (reliable, no throughput/timing guarantee) or UDP (unreliable, fast) as their transport protocol.

---

## Chapter Context — Why Application Layer First

Network applications are the _raison d'être_ of a computer network — without useful applications, there would be no need for networking infrastructure and protocols at all.

**Evolution of Internet applications:**

|Era|Applications|
|---|---|
|1970s–1980s|Classic text-based apps: text e-mail, remote access to computers, file transfers, newsgroups|
|Mid-1990s|The **killer application**: the World Wide Web — Web surfing, search, e-commerce|
|2000s onward|Video conferencing (Zoom, FaceTime, Teams); user-generated video (YouTube); movies on demand (Netflix); multiplayer online games (Second Life, World of Warcraft)|
|Recent|Social networking apps building human networks atop the Internet (TikTok, Facebook, Instagram, X); location-based mobile apps — check-in, dating, road-traffic forecasting (Yelp, Tinder, Waze); mobile payment apps (WeChat, Apple Pay); messaging apps (WeChat, WhatsApp)|

> There has been no slowing down of new and exciting Internet applications — the next generation of "killer apps" is still being invented.

**What this chapter covers:**

- Application-layer concepts: network services required by applications, processes, clients/servers, transport-layer interfaces
- Several network applications in detail: the Web, e-mail, DNS, video streaming
- Network application development over both TCP and UDP
- The socket interface, with simple client-server applications walked through in Python
- Several fun socket programming assignments at the end of the chapter

> The application layer is a particularly good place to start the study of protocols — it's familiar ground. We're already acquainted with many of the applications that rely on the protocols we'll study, and it introduces many of the same issues we'll see again when studying transport, network, and link layer protocols.

---

## Core Idea

At the core of network application development is writing programs that:

- Run on **different end systems**
- **Communicate with each other** over the network

For example, in the Web application there are two distinct programs that communicate with each other: the browser program running on the user's host (desktop, laptop, tablet, smartphone, etc.) and the Web server program running on the Web server host. As another example, in a Video on Demand application such as Netflix, there is a Netflix-provided program running on the user's device and a Netflix server program running on the Netflix server host — servers are often (but not always) housed in a data center.

**You do NOT write software for network-core devices (routers, switches):**

- Network-core devices do not function at the application layer
- They function at lower layers — network layer and below
- Even if you wanted to write application software for routers, you couldn't — they don't expose that interface

![[Pasted image 20260619173653.png]] _(Figure 2.1 — Communication for a network application takes place between end systems at the application layer — the core just forwards)_

> **This design decision — confining application software to end systems — has facilitated the rapid development and deployment of a vast array of network applications.**

---

# 2.1.1 Network Application Architectures

Before writing any code, you need a broad **architectural plan**.

> **Important distinction:**
> 
> - **Network architecture** (5-layer stack from Chapter 1) — fixed, provides services to applications
> - **Application architecture** — designed by the application developer, dictates how the application is structured over end systems

In choosing an application architecture, a developer will likely draw on one of the two predominant architectural paradigms in modern network applications:

---

## Architecture 1 — Client-Server

![[Pasted image 20260619173910.png]] _(Figure 2.2 — (a) Client-server architecture; (b) P2P architecture)_

**Structure:**

- There is always an **always-on host** called the **server**
- The server takes requests from many other hosts called **clients**
- Clients do **not** directly communicate with each other
- The server has a **fixed, well-known IP address**
- Because the server is always on, a client can always contact it

A classic example is the Web application, for which an always-on Web server services requests from browsers running on client hosts.

**Key characteristics:**

|Property|Detail|
|---|---|
|Server availability|Always-on|
|Server address|Fixed, well-known IP|
|Client-client communication|Not direct — goes through server|
|Scalability|Single server can be overwhelmed|

**Data centers:**

- A single server host is often incapable of keeping up with all requests from clients — a popular social-networking site can quickly become overwhelmed if it has only one server handling its requests
- Solution: **data center** — houses a large number of hosts to create a powerful virtual server
- The most popular Internet services run in one or more data centers, including: generative AI (chatbots, text-to-image/video, etc.), search engines (Google, Bing, Baidu), Internet commerce (Amazon, eBay, Alibaba), Web-based e-mail (Gmail, Yahoo Mail), and social media (TikTok, Facebook, Instagram, X, WeChat)
- Google has **more than 20 data centers** distributed around the world, which collectively handle Search, YouTube, Gmail, and other services
- A data center can have **hundreds of thousands of servers**, which must be powered and maintained
- Service providers must also pay recurring interconnection and bandwidth costs for sending data from their data centers

Some better-known applications with a client-server architecture include the Web, e-mail, and video streaming.

---

## Architecture 2 — Peer-to-Peer (P2P)

**Structure:**

- **Minimal (or no) reliance** on dedicated servers in data centers
- Application exploits **direct communication between pairs of intermittently connected hosts** called **peers**
- Peers are not owned by a service provider, but instead are desktops and laptops controlled by users — residing in homes, universities, and offices
- Peers communicate **without passing through a dedicated server**

**Example:** A popular P2P application is the file-sharing application **BitTorrent**.

**P2P's most compelling feature — Self-Scalability:**

> In a P2P file-sharing application, although each peer generates workload by requesting files, each peer also **adds service capacity** to the system by distributing files to other peers.

**P2P is also cost effective** — it normally doesn't require significant server infrastructure or server bandwidth (in contrast with client-server designs that rely on data centers).

**Limitations:**

> However, P2P applications face challenges of **security**, **performance**, and **reliability** due to their highly decentralized structure. Because of these challenges, **most applications today have a client-server architecture.**

---

# 2.1.2 Processes Communicating

## Processes vs Programs

> In the jargon of operating systems, it is not actually **programs** but **processes** that communicate.

- A **process** is a program that is running within an **end system**
- Processes on the same end system communicate via **interprocess communication** — governed by the end system's OS
- In this book we care about processes running on **different hosts** (with potentially different operating systems) communicating

**How processes on different end systems communicate:**

> Processes on two different end systems communicate with each other by **exchanging messages** across the computer network.

- A **sending process** creates and sends messages into the network
- A **receiving process** receives messages and possibly responds

---

## Client and Server Processes

A network application consists of pairs of processes that send messages to each other over a network. For each pair of communicating processes, we typically label one process as the **client** and the other as the **server**.

**Formal definition:**

> _In the context of a communication session between a pair of processes, the process that **initiates the communication** (initially contacts the other process at the beginning of the session) is labeled as the **client**. The process that **waits to be contacted** to begin the session is the **server**._

With the Web, a browser process initiates contact with a Web server process; hence the browser is the **client** and the Web server is the **server**. When there's no confusion, the terms **"client side"** and **"server side"** of an application are also used.

---

## The Socket — Interface Between Process and Network

Any message sent from one process to another must go through the underlying network. A process sends messages into — and receives messages from — the network through a **software interface called a socket**.

![[Pasted image 20260619174208.png]] _(Figure 2.3 — Application processes, sockets, and underlying transport protocol: socket is the interface between application layer and transport layer within a host)_

**The house/door analogy:**

```
Process        =  a house
Socket         =  the door of that house
Network        =  transportation infrastructure outside the door

To send a message to another process on another host:
  → Process shoves message out its door (socket)
  → Transportation infrastructure carries it
  → Message arrives at destination host's door (socket)
  → Receiving process acts on the message
```

**What is a socket exactly?**

- A socket is the **interface between the application layer and the transport layer** within a host
- Also referred to as the **API (Application Programming Interface)** between the application and the network
- The socket is the **programming interface with which network applications are built**

**What the application developer controls vs does NOT control:**

```
CONTROLLED by app developer:          NOT controlled by app developer:
──────────────────────────────        ──────────────────────────────────
Everything on application-layer       Transport-layer side of the socket
side of the socket

Choice of transport protocol          TCP buffers, variables, internals
(TCP or UDP)

Some transport-layer parameters       Actual packet transmission
(max buffer size, max segment size)
```

> The only control the application developer has on the transport-layer side of the socket is (1) the choice of transport protocol and (2) perhaps the ability to fix a few transport-layer parameters such as maximum buffer and maximum segment sizes (covered in Chapter 3).

---

## Addressing Processes — IP Address + Port Number

To send postal mail, you need a destination address. Similarly, to send packets to a process on another host, you need to identify it.

**Two pieces of information needed:**

### 1. IP Address — Identifies the Host

> An **IP address** is a 32-bit quantity that uniquely identifies the host.

- We will discuss IP addresses in detail in Chapter 4
- For now: IP address = unique identifier for a host on the Internet

### 2. Port Number — Identifies the Process on the Host

A host can be running many network applications simultaneously, so the sending process must also identify the receiving process (more specifically, the receiving socket) running on the destination host. This is the purpose of a **destination port number**.

**Well-known port numbers:**

|Application|Protocol|Port Number|
|---|---|---|
|Web server|HTTP|**80**|
|Mail server|SMTP|**25**|

> A full list of well-known port numbers for all Internet standard protocols can be found at **www.iana.org**. Port numbers are covered in detail in Chapter 3.

**Complete address of a process:**

```
Process Address = IP Address + Port Number
Example:         128.119.245.12 : 80
                 ↑               ↑
                 Host (server)   Web server process
```

---

# 2.1.3 Transport Services Available to Applications

The socket is the interface between the application and the transport protocol. The application pushes messages into the socket. On the other side of the socket, the transport protocol delivers them to the receiving socket.

**When developing an application, you must choose a transport protocol.** How? Study the services provided and match them to your application's needs.

Transport-layer services can be classified along **four dimensions:**

---

## Dimension 1 — Reliable Data Transfer

> **The problem:** packets can get lost within a computer network (buffer overflow, corruption). For applications like email, file transfer, financial transactions — data loss has **devastating consequences**.

**Two types of applications:**

|Type|Description|Examples|
|---|---|---|
|**Needs reliable transfer**|Data sent must arrive completely and correctly — app requires guaranteed delivery|Email, file transfer, web documents, financial apps|
|**Loss-tolerant**|Can tolerate some amount of data loss — lost data results in a small glitch, not a critical failure|Multimedia audio/video, VoIP, streaming|

> If a transport protocol provides guaranteed data delivery service, it is said to provide **reliable data transfer**.

---

## Dimension 2 — Throughput

Available throughput between two processes can fluctuate over time because other sessions share the network path bandwidth.

**Two types of applications by throughput need:**

|Type|Description|Examples|
|---|---|---|
|**Bandwidth-sensitive**|Require a minimum guaranteed throughput of r bits/sec to be effective — if throughput drops below r, app must lower quality or give up|Internet telephony (32 kbps voice), video conferencing|
|**Elastic**|Can make use of as much or as little throughput as happens to be available — more is always better|Email, file transfer, web transfers|

> _"There's an adage that says that one cannot be too rich, too thin, or have too much throughput!"_

---

## Dimension 3 — Timing

> A transport protocol can provide **timing guarantees**.

**Example guarantee:** every bit that the sender pumps into the socket arrives at the receiver's socket **no more than 100 msec later**.

**Applications that need tight timing:**

- Internet telephony
- Video conferencing
- Metaverses / virtual interactive environments
- Multiplayer games

**Why timing matters:**

- Long delays in Internet telephony → unnatural pauses in the conversation
- In a multiplayer game or virtual interactive metaverse, a long delay between taking an action and seeing the response from the environment (e.g., from another player at the end of an end-to-end connection) makes the application feel less realistic

**Non-real-time applications:** lower delay is always preferable but no tight timing constraint is placed on end-to-end delays.

---

## Dimension 4 — Security

> A transport protocol can provide an application with one or more **security services**.

**Example services:**

- **Encryption** — transport protocol encrypts all data transmitted by the sending process; transport layer at receiving host decrypts before delivering to receiving process → **confidentiality** even if packets are sniffed in between
- **Data integrity** — ensure data has not been tampered with in transit
- **End-point authentication** — verify that the other party is who they claim to be

These topics are covered in detail in Chapter 8.

---

## Application Requirements — Summary Table

![[Pasted image 20260619174400.png]]
_(From Figure 2.4)_

|Application|Data Loss|Throughput|Time-Sensitive|
|---|---|---|---|
|File transfer / download|No loss|Elastic|No|
|Email|No loss|Elastic|No|
|Web documents|No loss|Elastic (few kbps)|No|
|Internet telephony / video conferencing|Loss-tolerant|Audio: few kbps–1 Mbps / Video: 10–5 Mbps|Yes: 100s of msec|
|Streaming stored audio/video|Loss-tolerant|Same as above|Yes: few seconds|
|Interactive games|Loss-tolerant|Few kbps–10 kbps|Yes: 100s of msec|
|Smartphone messaging|No loss|Elastic|Yes and no|

---

# 2.1.4 Transport Services Provided by the Internet

The Internet (TCP/IP networks) makes **two transport protocols** available to applications: **TCP** and **UDP**.

When you develop a new network application for the Internet, one of the first decisions is: **TCP or UDP?**

---

## TCP Services

> The TCP service model includes a **connection-oriented service** and a **reliable data transfer service**.

### Connection-Oriented Service

- TCP has the client and server **exchange transport-layer control information** with each other **before** application-level messages begin to flow → called the **handshaking procedure**
- This handshaking procedure alerts the client and server, allowing them to prepare for an onslaught of packets
- After handshaking, a **TCP connection** is said to exist between the sockets of the two processes
- The connection is **full-duplex** — both processes can send messages to each other over the connection at the same time
- When the application finishes, it must **tear down** the connection

### Reliable Data Transfer Service

> The communicating processes can rely on TCP to deliver all data sent **without error and in the proper order** — no missing or duplicate bytes.

### Congestion Control

- TCP also includes a **congestion-control mechanism**
- This is a service for the **general welfare of the Internet** rather than for the direct benefit of the communicating processes
- TCP congestion control **throttles a sending process** (client or server) when the network is congested between sender and receiver
- Also attempts to limit each TCP connection to its **fair share of network bandwidth**

### Securing TCP — TLS

> Neither TCP nor UDP provides any encryption — data passed into the socket travels over the network as the same data that the sending process passed in. If the sending process sends a password in cleartext (unencrypted), the password will travel over all the links between sender and receiver, potentially getting sniffed and discovered at any of the intervening links.

Because privacy and other security issues have become critical for many applications, the Internet community has developed an enhancement for TCP, called **Transport Layer Security (TLS)** [RFC 5246].

> **TLS is NOT a third Internet transport protocol** on the same level as TCP and UDP. It is an **enhancement of TCP**, with the enhancements being implemented in the **application layer**.

- TCP-enhanced-with-TLS does everything traditional TCP does, but also provides critical process-to-process security services, including **encryption**, **data integrity**, and **end-point authentication**
- To use TLS, an application includes TLS code (existing, highly optimized libraries and classes) in both the client and server sides
- TLS has its own socket API, similar to the traditional TCP socket API

**How TLS works:**

```
Sending side:
  App passes cleartext → TLS socket
  TLS in the sending host encrypts the data
  TLS passes encrypted data → TCP socket
  Encrypted data travels over the Internet

Receiving side:
  TCP socket receives encrypted data
  Passes it to TLS
  TLS decrypts the data
  TLS passes cleartext data → receiving process (through its TLS socket)
```

Covered in detail in **Chapter 8**.

---

## UDP Services

> UDP is a **no-frills, lightweight transport protocol** providing minimal services.

**What UDP provides:**

- **Connectionless** — no handshaking before processes start to communicate
- **Unreliable data transfer** — no guarantee that message will ever reach the receiving process
- Messages that do arrive **may arrive out of order**
- **No congestion control** — sending side can pump data into the layer below at any rate it pleases (the actual end-to-end throughput may still be less than this rate due to limited transmission capacity of intervening links or due to congestion)

**Why use UDP if it provides so little?**

- **Speed** — no connection setup overhead, no state maintained
- **No congestion throttling** — useful for real-time apps that need to send at a specific rate regardless of congestion
- **Lower overhead** — simpler protocol → less processing

---

## What TCP and UDP Do NOT Provide

> **Conspicuously missing from both TCP and UDP:** **throughput guarantees** and **timing guarantees**

|Service|TCP|UDP|
|---|---|---|
|Reliable data transfer|✅ Yes|❌ No|
|Congestion control|✅ Yes|❌ No|
|Connection setup|✅ Yes (handshake)|❌ No|
|Throughput guarantee|❌ No|❌ No|
|Timing guarantee|❌ No|❌ No|
|Security (basic)|❌ No (need TLS)|❌ No|

**Does this mean time-sensitive apps cannot run on the Internet?**

No — the Internet has been hosting time-sensitive applications for many years. These applications are **designed to cope**, to the greatest extent possible, with this lack of guarantee. Clever design has its limitations when delay is excessive, but today's Internet can often provide satisfactory service to time-sensitive applications. It **cannot**, however, provide any timing or throughput guarantees.

---

## Popular Applications — Protocol Summary

_(From Figure 2.5)_

|Application|Application-Layer Protocol|Underlying Transport Protocol|
|---|---|---|
|Electronic mail|SMTP [RFC 5321]|TCP|
|Web|HTTP/2 [RFC 7540]|TCP|
|File transfer|FTP [RFC 959]|TCP|
|Streaming multimedia|HTTP (e.g., YouTube), DASH|TCP|
|Internet telephony|SIP [RFC 3261], RTP [RFC 3550], or proprietary (e.g., Zoom)|UDP or TCP|

**Why do e-mail, remote terminal access, the Web, and file transfer all use TCP?**

These applications have chosen TCP primarily because TCP provides reliable data transfer, guaranteeing that all data will eventually get to its destination.

**Why do video conferencing applications (such as Zoom) prefer UDP?**

- They can often tolerate some loss but require a minimal rate to be effective
- Developers usually prefer to run them over UDP, circumventing TCP's congestion-control mechanism and packet overheads
- **However:** many firewalls are configured to block (most types of) UDP traffic → Internet telephony apps are often designed to use **TCP as a backup** if UDP communication fails

---

# 2.1.5 Application-Layer Protocols

Processes send messages into sockets. But how are these messages structured? When do processes send messages? What do the fields mean?

> An **application-layer protocol** defines how an application's processes, running on different end systems, **pass messages to each other**.

**An application-layer protocol defines:**

- **Types of messages** exchanged — request messages and response messages
- **Syntax** of the various message types — fields in the message and how the fields are delineated
- **Semantics** of the fields — the meaning of the information in the fields
- **Rules** for determining when and how a process sends messages and responds to messages

---

## Public Domain vs Proprietary Protocols

|Type|Description|Example|
|---|---|---|
|**Public domain**|Specified in RFCs, available to everyone — if a browser follows the HTTP RFC, it can retrieve pages from any server that also follows it|HTTP (RFC 7230), SMTP (RFC 5321)|
|**Proprietary**|Intentionally not available in the public domain|Zoom's proprietary application-layer protocols|

---

## Application-Layer Protocol ≠ Network Application

> **Critical distinction:** An application-layer protocol is only **one piece** of a network application — albeit a very important piece.

**Example — The Web:**

The Web application consists of many components:

```
Web Application
├── Standard for document formats (HTML)
├── Web browsers (e.g., Google Chrome, Microsoft Edge)
├── Web servers (e.g., Apache, Microsoft servers)
└── Application-layer protocol: HTTP
    └── Defines format and sequence of messages exchanged
        between browser and Web server
```

> HTTP is only one piece (albeit a critical piece) of the Web application.

**Example — Netflix Video Streaming:**

```
Netflix Application
├── Servers that store and transmit videos
├── Other servers that manage billing and client functions
├── Clients (e.g., the Netflix app on smartphone, tablet, or computer)
└── Application-level protocol: DASH
    └── Defines the format and sequence of messages exchanged
        between a Netflix server and client
```

> DASH is only one piece (albeit a critical piece) of the Netflix application.

---

# 2.1.6 Network Applications Covered in This Book

New applications are being developed every day. Rather than covering a large number of Internet applications in an encyclopedic manner, this book focuses on a small number of applications that are both pervasive and important. Four applications are discussed in this chapter:

|Application|Why it's covered|
|---|---|
|**The Web**|Enormously popular, and its application-layer protocol (HTTP) is relatively straightforward and easy to understand — the Internet's first killer application|
|**Electronic mail**|More complex than the Web, since it makes use of not one but several application-layer protocols|
|**DNS (directory service)**|Most users don't interact with DNS directly — instead, it's invoked indirectly through other applications (the Web, file transfer, e-mail). DNS illustrates how a piece of core network functionality (name-to-address translation) can be implemented at the application layer|
|**Video streaming on demand**|Includes distributing stored video over content distribution networks|

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Sockets**|Binding to a well-known port → attackers scan for open ports to find vulnerable services|Close unused ports; firewall rules to restrict port access|
|**TCP connection-oriented**|SYN flood — exhaust TCP connection table|SYN cookies, rate limiting, firewalls|
|**UDP connectionless**|UDP flooding — no handshake overhead → very fast flood generation|Rate limiting, block UDP where not needed|
|**No TLS by default**|Sniff cleartext data from TCP streams (passwords, session tokens)|Always use TLS; HTTPS only|
|**P2P open nature**|P2P networks are difficult to secure; malware can spread peer-to-peer|Application-level security, certificate pinning|
|**Port numbers are known**|Attackers target well-known ports (80, 25)|Non-default ports (security through obscurity — weak but adds friction)|
|**Application-layer protocols in public RFCs**|Study the RFC to find vulnerabilities in protocol design|Security-focused RFC extensions (e.g., HTTPS, SMTPS, STARTTLS)|

---

## Questions I Still Have

- [ ] How exactly does the TCP handshake work at the byte level — what is exchanged in SYN, SYN-ACK, ACK?
- [ ] Why can't TCP provide throughput guarantees — is it a fundamental limitation or a design choice?
- [ ] How does TLS sit "inside" the application layer — does it have its own separate socket from the TCP socket?
- [ ] In P2P, how does a peer find another peer to connect to without a central server — what is the discovery mechanism?
- [ ] Why does HTTP (streaming multimedia on YouTube) use TCP instead of UDP — isn't video a loss-tolerant, timing-sensitive application? How does DASH fit into this picture?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Process**|A program running within an end system|
|**Socket**|Software interface between application layer and transport layer; the network API|
|**Client process**|Process that initiates the communication session|
|**Server process**|Process that waits to be contacted to begin the session|
|**IP address**|32-bit quantity uniquely identifying a host on the Internet|
|**Port number**|Identifier specifying which process (socket) on the host to deliver to|
|**Client-server architecture**|Always-on server services requests from clients; clients don't talk directly|
|**P2P architecture**|Peers communicate directly without dedicated servers|
|**Self-scalability**|P2P property: each new peer adds both workload and service capacity|
|**Data center**|Housing for large numbers of hosts creating a powerful virtual server|
|**Reliable data transfer**|Guarantee that all data arrives correctly and completely|
|**Loss-tolerant application**|App that can tolerate some data loss (multimedia)|
|**Bandwidth-sensitive app**|Requires minimum guaranteed throughput to function|
|**Elastic application**|Can use as much or as little throughput as available|
|**TCP**|Connection-oriented, reliable, congestion-controlled transport protocol|
|**UDP**|Connectionless, unreliable, no-frills transport protocol|
|**TLS (Transport Layer Security)**|Application-layer enhancement of TCP (RFC 5246) providing encryption, data integrity, and end-point authentication|
|**DASH**|Application-level protocol defining message format/sequence between a video-streaming server (e.g., Netflix) and client|
|**Application-layer protocol**|Defines how processes on different end systems pass messages|
|**Full-duplex**|Both processes can send messages simultaneously over a TCP connection|
|**Congestion control**|TCP mechanism that throttles sender when network is congested|

---

## Related Concepts

---

→ Next: [[THE WEB & HTTP]]