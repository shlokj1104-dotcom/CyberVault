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

> **One-Line Summary:** Network applications are programs running on
> different end systems that communicate over the network — structured
> as either client-server or P2P architectures, communicating via
> sockets, and choosing between TCP (reliable, no throughput/timing
> guarantee) or UDP (unreliable, fast) as their transport protocol.

---

## Core Idea

At the core of network application development is writing programs that:
- Run on **different end systems**
- **Communicate with each other** over the network

**You do NOT write software for network-core devices (routers, switches):**
- Network-core devices do not function at the application layer
- They function at lower layers — network layer and below
- Even if you wanted to write application software for routers, you
  couldn't — they don't expose that interface

![[Pasted image 20260519190023.png]]
*(Figure 2.1 — Communication for a network application takes place
between end systems at the application layer — the core just forwards)*

> **This design decision — confining application software to end systems
> — has facilitated the rapid development and deployment of a vast array
> of network applications.**

---

# 2.1.1 Network Application Architectures

Before writing any code, you need a broad **architectural plan**.

> **Important distinction:**
> - **Network architecture** (5-layer stack from Chapter 1) — fixed,
>   provides services to applications
> - **Application architecture** — designed by the application developer,
>   dictates how the application is structured over end systems

Two predominant architectural paradigms in modern network applications:

---

## Architecture 1 — Client-Server

![[Pasted image 20260519190230.png]]
*(Figure 2.2 — (a) Client-server architecture; (b) P2P architecture)*

**Structure:**

- There is always an **always-on host** called the **server**
- The server services requests from many other hosts called **clients**
- Clients do **not** directly communicate with each other
- The server has a **fixed, well-known IP address**
- Because the server is always on, a client can always contact it

**Classic examples:** Web (HTTP), FTP, Telnet, email

**Key characteristics:**

| Property | Detail |
|---|---|
| Server availability | Always-on |
| Server address | Fixed, well-known IP |
| Client-client communication | Not direct — goes through server |
| Scalability | Single server can be overwhelmed |

**Data centers:**
- A single server host cannot keep up with all requests from a popular app
- Solution: **data center** — houses large numbers of hosts to create a
  powerful virtual server
- Google: 30–50 data centers distributed globally, collectively handling
  Search, YouTube, Gmail, and other services
- A data center can have **hundreds of thousands of servers**
- Service providers pay recurring interconnection and bandwidth costs

---

## Architecture 2 — Peer-to-Peer (P2P)

**Structure:**

- **Minimal (or no) reliance** on dedicated servers in data centers
- Application exploits **direct communication between pairs of
  intermittently connected hosts** called **peers**
- Peers are desktops and laptops controlled by users — residing in
  homes, universities, and offices
- Peers communicate **without passing through a dedicated server**

**Examples:**
- File sharing: **BitTorrent**
- Download acceleration: **Xunlei**
- Internet telephony: **Skype**
- IPTV: **Kankan, PPstream**

**P2P's most compelling feature — Self-Scalability:**

> In a P2P file-sharing application, although each peer generates
> workload by requesting files, each peer also **adds service capacity**
> to the system by distributing files to other peers.

**P2P is also cost effective** — doesn't require significant server
infrastructure or server bandwidth (unlike client-server with data centers).

**Three major challenges for future P2P applications:**

| Challenge | Detail |
|---|---|
| **ISP Friendly** | Most residential ISPs dimensioned for asymmetrical bandwidth (much more downstream than upstream). P2P video/file apps shift upstream traffic from servers to residential ISPs → puts significant stress on ISPs |
| **Security** | Highly distributed and open nature makes P2P apps a challenge to secure |
| **Incentives** | Success depends on convincing users to volunteer bandwidth, storage, and computation resources — the challenge of incentive design |

---

## Hybrid Architectures

Some applications combine both client-server and P2P elements.

**Example — Instant messaging:**
- Servers track the **IP addresses** of users (client-server component)
- User-to-user messages sent **directly between users** without
  intermediate servers (P2P component)

---

# 2.1.2 Processes Communicating

## Processes vs Programs

> In the jargon of operating systems, it is not actually **programs**
> but **processes** that communicate.

- A **process** is a program that is running within an **end system**
- Processes on the same end system communicate via **interprocess
  communication** — governed by the end system's OS
- In this book we care about processes running on **different hosts**
  (with potentially different operating systems) communicating

**How processes on different end systems communicate:**

> Processes on two different end systems communicate with each other by
> **exchanging messages** across the computer network.

- A **sending process** creates and sends messages into the network
- A **receiving process** receives messages and possibly responds

---

## Client and Server Processes

A network application consists of pairs of processes that send messages
to each other over a network.

**Formal definition:**

> *In the context of a communication session between a pair of processes,
> the process that **initiates the communication** (initially contacts
> the other process at the beginning of the session) is labeled as the
> **client**. The process that **waits to be contacted** to begin the
> session is the **server**.*

**Examples:**

| Application | Client Process | Server Process |
|---|---|---|
| Web | Browser | Web server |
| P2P file sharing | Peer downloading the file | Peer uploading the file |
| Email | Mail client (Outlook) | Mail server |

> **Note:** In P2P file sharing, a process can be **both a client and a
> server** — it can simultaneously upload (server) and download (client).
> Nevertheless, in the context of any given communication session, we
> can still label one as client and one as server.

---

## The Socket — Interface Between Process and Network

Any message sent from one process to another must go through the
underlying network. A process sends messages into — and receives
messages from — the network through a **software interface called a
socket**.

![[Pasted image 20260519190412.png]]
*(Figure 2.3 — Application processes, sockets, and underlying transport
protocol: socket is the interface between application layer and transport
layer within a host)*

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

- A socket is the **interface between the application layer and the
  transport layer** within a host
- Also referred to as the **API (Application Programming Interface)**
  between the application and the network
- The socket is the **programming interface with which network
  applications are built**

**What the application developer controls vs does NOT control:**

```
CONTROLLED by app developer:          NOT controlled by app developer:
──────────────────────────────        ──────────────────────────────────
Everything on application-layer       Transport-layer side of the socket
side of the socket

Choice of transport protocol          TCP buffers, variables, internals
(TCP or UDP)

Some transport-layer parameters       Actual packet transmission
(max buffer size, segment size)
```

---

## Addressing Processes — IP Address + Port Number

To send postal mail, you need a destination address. Similarly, to send
packets to a process on another host, you need to identify it.

**Two pieces of information needed:**

### 1. IP Address — Identifies the Host

> An **IP address** is a 32-bit quantity that uniquely identifies the host.

- We will discuss IP addresses in detail in Chapter 4
- For now: IP address = unique identifier for a host on the Internet

### 2. Port Number — Identifies the Process on the Host

A host can be running many network applications simultaneously.

> A **destination port number** serves the purpose of identifying the
> specific receiving process (more specifically, the receiving socket)
> running on the destination host.

**Well-known port numbers for popular applications:**

| Application | Protocol | Port Number |
|---|---|---|
| Web server | HTTP | **80** |
| Mail server | SMTP | **25** |
| FTP | FTP | **21** |
| SSH | SSH | **22** |
| DNS | DNS | **53** |
| HTTPS | HTTPS | **443** |

> Full list of well-known port numbers: **http://www.iana.org**

**Complete address of a process:**

```
Process Address = IP Address + Port Number
Example:         128.119.245.12 : 80
                 ↑               ↑
                 Host (server)   Web server process
```

---

# 2.1.3 Transport Services Available to Applications

The socket is the interface between the application and the transport
protocol. The application pushes messages into the socket. On the other
side of the socket, the transport protocol delivers them to the
receiving socket.

**When developing an application, you must choose a transport protocol.**
How? Study the services provided and match them to your application's needs.

Transport-layer services can be classified along **four dimensions:**

---

## Dimension 1 — Reliable Data Transfer

> **The problem:** packets can get lost within a computer network (buffer
> overflow, corruption). For applications like email, file transfer,
> financial transactions — data loss has **devastating consequences**.

**Two types of applications:**

| Type                        | Description                                                                                         | Examples                                            |
| --------------------------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **Needs reliable transfer** | Data sent must arrive completely and correctly — app requires guaranteed delivery                   | Email, file transfer, web documents, financial apps |
| **Loss-tolerant**           | Can tolerate some amount of data loss — lost data results in a small glitch, not a critical failure | Multimedia audio/video, VoIP, streaming             |

> If a transport protocol provides guaranteed data delivery service,
> it is said to provide **reliable data transfer**.

---

## Dimension 2 — Throughput

Available throughput between two processes can fluctuate over time
because other sessions share the network path bandwidth.

**Two types of applications by throughput need:**

| Type | Description | Examples |
|---|---|---|
| **Bandwidth-sensitive** | Require a minimum guaranteed throughput of r bits/sec to be effective — if throughput drops below r, app must lower quality or give up | Internet telephony (32 kbps voice), video conferencing |
| **Elastic** | Can make use of as much or as little throughput as happens to be available — more is always better | Email, file transfer, web transfers |

> *"There's an adage that says that one cannot be too rich, too thin,
> or have too much throughput!"*

---

## Dimension 3 — Timing

> A transport protocol can provide **timing guarantees**.

**Example guarantee:** every bit that the sender pumps into the socket
arrives at the receiver's socket **no more than 100 msec later**.

**Applications that need tight timing:**

- Internet telephony
- Virtual environments
- Teleconferencing
- Multiplayer games

**Why timing matters:**
- Long delays in telephony → unnatural pauses in conversation
- Long delays in multiplayer games → seeing the response from another
  player at the end of an end-to-end connection makes the game feel
  unrealistic

**Non-real-time applications:** lower delay is always preferable but
no tight timing constraint is placed on end-to-end delays.

---

## Dimension 4 — Security

> A transport protocol can provide an application with one or more
> **security services**.

**Example services:**

- **Encryption** — transport protocol encrypts all data transmitted by
  the sending process; transport layer at receiving host decrypts before
  delivering to receiving process → **confidentiality** even if packets
  are sniffed in between
- **Data integrity** — ensure data has not been tampered with in transit
- **End-point authentication** — verify that the other party is who they
  claim to be

These topics are covered in detail in Chapter 8.

---

## Application Requirements — Summary Table

*(From Figure 2.4)*

| Application | Data Loss | Throughput | Time-Sensitive |
|---|---|---|---|
| File transfer / download | No loss | Elastic | No |
| Email | No loss | Elastic | No |
| Web documents | No loss | Elastic (few kbps) | No |
| Internet telephony / video conf. | Loss-tolerant | Audio: few kbps–1 Mbps / Video: 10–5 Mbps | Yes: 100s of msec |
| Streaming stored audio/video | Loss-tolerant | Same as above | Yes: few seconds |
| Interactive games | Loss-tolerant | Few kbps–10 kbps | Yes: 100s of msec |
| Instant messaging | No loss | Elastic | Yes and no |

---

# 2.1.4 Transport Services Provided by the Internet

The Internet (TCP/IP networks) makes **two transport protocols**
available to applications: **TCP** and **UDP**.

When you develop a new network application for the Internet, one of the
first decisions is: **TCP or UDP?**

---

## TCP Services

> The TCP service model includes a **connection-oriented service** and a
> **reliable data transfer service**.

### Connection-Oriented Service

- TCP has the client and server **exchange transport-layer control
  information** with each other **before** application-level messages begin
  to flow → called the **handshaking procedure**
- After handshaking, a **TCP connection** is said to exist between the
  sockets of the two processes
- The connection is **full-duplex** — both processes can send messages
  to each other over the connection at the same time
- When the application finishes, it must **tear down** the connection

### Reliable Data Transfer Service

> The communicating processes can rely on TCP to deliver all data sent
> **without error and in the proper order** — no missing or duplicate bytes.

### Congestion Control

- TCP also includes a **congestion-control mechanism**
- This is a service for the **general welfare of the Internet** rather
  than for the direct benefit of the communicating processes
- TCP congestion control **throttles a sending process** (client or
  server) when the network is congested between sender and receiver
- Also attempts to limit each TCP connection to its **fair share of
  network bandwidth**

### Securing TCP — SSL

> Neither TCP nor UDP provides any encryption — data passed into the
> socket travels over the network in **cleartext**.

**SSL (Secure Sockets Layer):**

- An enhancement of TCP developed by the Internet community
- **SSL is NOT a third Internet transport protocol** on the same level
  as TCP and UDP
- SSL is an **enhancement of TCP** implemented in the **application layer**
- To use SSL, an application includes SSL code (libraries/classes) in
  both client and server sides
- SSL has its own socket API similar to the traditional TCP socket API

**How SSL works:**

```
Sending side:
  App passes cleartext → SSL socket
  SSL encrypts the data
  SSL passes encrypted data → TCP socket
  Encrypted data travels over Internet

Receiving side:
  TCP socket receives encrypted data
  Passes to SSL
  SSL decrypts
  SSL passes cleartext → receiving process
```

Covered in detail in **Chapter 8**.

---

## UDP Services

> UDP is a **no-frills, lightweight transport protocol** providing
> minimal services.

**What UDP provides:**

- **Connectionless** — no handshaking before processes start to communicate
- **Unreliable data transfer** — no guarantee that message will ever
  reach the receiving process
- Messages that do arrive **may arrive out of order**
- **No congestion control** — sending side can pump data into the layer
  below at any rate it pleases

**Why use UDP if it provides so little?**

- **Speed** — no connection setup overhead, no state maintained
- **No congestion throttling** — useful for real-time apps that need
  to send at a specific rate regardless of congestion
- **Lower overhead** — simpler protocol → less processing

---

## What TCP and UDP Do NOT Provide

> **Conspicuously missing from both TCP and UDP:**
> **throughput guarantees** and **timing guarantees**

| Service | TCP | UDP |
|---|---|---|
| Reliable data transfer | ✅ Yes | ❌ No |
| Congestion control | ✅ Yes | ❌ No |
| Connection setup | ✅ Yes (handshake) | ❌ No |
| Throughput guarantee | ❌ No | ❌ No |
| Timing guarantee | ❌ No | ❌ No |
| Security (basic) | ❌ No (need SSL) | ❌ No |

**Does this mean time-sensitive apps cannot run on the Internet?**

No — the Internet has been hosting time-sensitive applications for many
years. These applications are **designed to cope** with the lack of
guarantee to the greatest extent possible. Clever design has its
limitations when delay is excessive, but today's Internet can often
provide satisfactory service. It **cannot** provide any timing or
throughput guarantees.

---

## Popular Applications — Protocol Summary

*(From Figure 2.5)*

| Application | App-Layer Protocol | Transport Protocol |
|---|---|---|
| Electronic mail | SMTP (RFC 5321) | TCP |
| Remote terminal access | Telnet (RFC 854) | TCP |
| Web | HTTP (RFC 2616) | TCP |
| File transfer | FTP (RFC 959) | TCP |
| Streaming multimedia | HTTP (e.g., YouTube) | TCP |
| Internet telephony | SIP, RTP, or proprietary (Skype) | UDP or TCP |

**Why do email, web, and file transfer all use TCP?**

TCP provides reliable data transfer, guaranteeing all data eventually
reaches its destination. These applications **cannot tolerate data loss**.

**Why does Internet telephony prefer UDP?**

- Can tolerate some loss but requires a minimum rate to be effective
- Developers prefer UDP to circumvent TCP's congestion control and
  packet overheads
- **However:** many firewalls block UDP traffic → Internet telephony
  apps are often designed to use **TCP as a backup** if UDP fails

---

# 2.1.5 Application-Layer Protocols

Processes send messages into sockets. But how are these messages
structured? When do processes send messages? What do the fields mean?

> An **application-layer protocol** defines how an application's
> processes, running on different end systems, **pass messages to each
> other**.

**An application-layer protocol defines:**

- **Types of messages** exchanged — request messages and response messages
- **Syntax** of the various message types — fields in the message and
  how the fields are delineated
- **Semantics** of the fields — the meaning of the information in the fields
- **Rules** for determining when and how a process sends messages and
  responds to messages

---

## Public Domain vs Proprietary Protocols

| Type | Description | Example |
|---|---|---|
| **Public domain** | Specified in RFCs, available to everyone — if a browser follows the HTTP RFC, it can retrieve pages from any server that also follows it | HTTP (RFC 2616), SMTP (RFC 5321) |
| **Proprietary** | Intentionally not available in the public domain | Skype's proprietary protocols |

---

## Application-Layer Protocol ≠ Network Application

> **Critical distinction:**
> An application-layer protocol is only **one piece** of a network
> application — albeit a very important piece.

**Example — The Web:**

The Web application consists of many components:

```
Web Application
├── Standard for document formats (HTML)
├── Web browsers (Firefox, Chrome, Internet Explorer)
├── Web servers (Apache, Microsoft IIS)
└── Application-layer protocol: HTTP
    └── Defines format and sequence of messages exchanged
        between browser and Web server
```

> HTTP is only one piece of the Web application.

**Example — Email:**

```
Email Application
├── Mail servers (house user mailboxes)
├── Mail clients (Outlook, Thunderbird)
├── Standard for email message structure
├── Application-layer protocols:
│   ├── SMTP — how messages are passed between servers
│   ├── SMTP — how messages are passed from mail client to server
│   └── POP3/IMAP — how messages are downloaded to client
```

> SMTP is only one piece of the email application.

---

# Why It Matters for Security

| Concept | Attacker's Perspective | Defender's Perspective |
|---|---|---|
| **Sockets** | Binding to a well-known port → attackers scan for open ports to find vulnerable services | Close unused ports; firewall rules to restrict port access |
| **TCP connection-oriented** | SYN flood — exhaust TCP connection table | SYN cookies, rate limiting, firewalls |
| **UDP connectionless** | UDP flooding — no handshake overhead → very fast flood generation | Rate limiting, block UDP where not needed |
| **No SSL by default** | Sniff cleartext data from TCP streams (passwords, session tokens) | Always use SSL/TLS; HTTPS only |
| **P2P open nature** | P2P networks are difficult to secure; malware can spread peer-to-peer | Application-level security, certificate pinning |
| **Port numbers are known** | Attackers target well-known ports (80, 25, 22) | Non-default ports (security through obscurity — weak but adds friction) |
| **Application-layer protocols in public RFCs** | Study the RFC to find vulnerabilities in protocol design | Security-focused RFC extensions (e.g., HTTPS, SMTPS, STARTTLS) |

---

## Questions I Still Have

- [ ] How exactly does the TCP handshake work at the byte level —
      what is exchanged in SYN, SYN-ACK, ACK?
- [ ] Why can't TCP provide throughput guarantees — is it a fundamental
      limitation or a design choice?
- [ ] How does SSL/TLS sit "inside" the application layer — does it have
      its own separate socket from the TCP socket?
- [ ] In P2P, how does a peer find another peer to connect to without
      a central server — what is the discovery mechanism?
- [ ] Why does HTTP (streaming multimedia on YouTube) use TCP instead of
      UDP — isn't video a loss-tolerant, timing-sensitive application?

---

## Key Terms — Quick Reference

| Term | Definition |
|---|---|
| **Process** | A program running within an end system |
| **Socket** | Software interface between application layer and transport layer; the network API |
| **Client process** | Process that initiates the communication session |
| **Server process** | Process that waits to be contacted to begin the session |
| **IP address** | 32-bit quantity uniquely identifying a host on the Internet |
| **Port number** | Identifier specifying which process (socket) on the host to deliver to |
| **Client-server architecture** | Always-on server services requests from clients; clients don't talk directly |
| **P2P architecture** | Peers communicate directly without dedicated servers |
| **Self-scalability** | P2P property: each new peer adds both workload and service capacity |
| **Data center** | Housing for large numbers of hosts creating a powerful virtual server |
| **Reliable data transfer** | Guarantee that all data arrives correctly and completely |
| **Loss-tolerant application** | App that can tolerate some data loss (multimedia) |
| **Bandwidth-sensitive app** | Requires minimum guaranteed throughput to function |
| **Elastic application** | Can use as much or as little throughput as available |
| **TCP** | Connection-oriented, reliable, congestion-controlled transport protocol |
| **UDP** | Connectionless, unreliable, no-frills transport protocol |
| **SSL/TLS** | Application-layer enhancement of TCP providing encryption and authentication |
| **Application-layer protocol** | Defines how processes on different end systems pass messages |
| **Full-duplex** | Both processes can send messages simultaneously over a TCP connection |
| **Congestion control** | TCP mechanism that throttles sender when network is congested |

---

## Related Concepts

- [[1.5 - Protocol Layers and Their Service Models]]
- [[1.6 - Networks Under Attack]]
- [[TCP Three-Way Handshake — Chapter 3 Preview]]
- [[HTTP and the Web — Section 2.2]]
- [[DNS — Section 2.4]]
- [[Socket Programming — Section 2.7]]
- [[SSL and TLS — Chapter 8 Preview]]

---

→ Next: [[2.2 - The Web and HTTP]]