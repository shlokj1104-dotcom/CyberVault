---
title: INTRODUCTION TO THE INTERNET
date: 2026-05-12
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 1.1 What Is the Internet?

> **One-Line Summary:** The Internet is a global system of billions of 
> interconnected devices communicating through shared rules called protocols 
> (TCP/IP), serving both as a hardware/software infrastructure and as a 
> platform for distributed applications.

---

# 1.1.1 A Nuts-and-Bolts Description

There are two ways to answer "What is the Internet?":
1. Describe its **physical components** (hardware + software)
2. Describe it as a **service infrastructure** for applications

This section covers approach #1.

---

## End Systems (Hosts)

The Internet connects billions of **computing devices** across the world.
Originally these were desktop PCs, Linux workstations, and servers.
Today the list includes:

- Laptops, smartphones, tablets
- TVs, gaming consoles, webcams
- Automobiles, environmental sensors
- Home electrical and security systems

> All of these devices are called **hosts** or **end systems** — the two 
> terms mean exactly the same thing and are used interchangeably throughout 
> networking literature.

The term *computer network* is starting to sound dated given how many 
non-traditional devices are connected — but the name stuck.

---

## Communication Links

End systems are connected to each other through **communication links**.

Links are made of different types of **physical media**:

| Media Type | Examples | Notes |
|---|---|---|
| Coaxial cable | Cable TV lines | Copper, shielded |
| Copper wire | Telephone lines, Ethernet | Most common wired |
| Optical fiber | Undersea cables, ISP backbone | Extremely fast, light-based |
| Radio spectrum | WiFi, 4G/5G, Bluetooth | Wireless, no physical wire |

- Each link can transmit data at different speeds.
- The speed of a link is called its **transmission rate**, measured in 
  **bits per second (bps)** — or Kbps, Mbps, Gbps.

---

## Packets — How Data Actually Travels

When one end system wants to send data to another, it doesn't send the 
whole thing at once. Here's what actually happens:

1. The sending end system **segments** the data into smaller chunks.
2. It adds **header bytes** to each chunk (containing addressing, ordering 
   info, etc.).
3. These chunks — called **packets** — are sent independently through the 
   network.
4. At the destination, the packets are **reassembled** back into the 
   original data.

> 💡 **Truck Analogy (from Kurose & Ross):**
> Imagine a factory needs to ship a huge load of cargo across the country.
> - The cargo is split and loaded into many trucks.
> - Each truck independently travels through highways and intersections.
> - At the destination warehouse, all cargo is unloaded and regrouped.

| Network Term | Analogy |
|---|---|
| Data | Cargo |
| Packet | Truck |
| Communication link | Highway / road |
| Packet switch | Intersection |
| End system | Building (factory/warehouse) |

---

## Packet Switches

A **packet switch** receives a packet on one of its incoming links and 
forwards it on one of its outgoing links — moving it closer to its 
destination.

Two most prominent types of packet switches:

### Routers
- Used in the **network core** (the middle of the Internet).
- Make intelligent forwarding decisions based on destination IP address.
- Forward packets toward their ultimate destination.

### Link-Layer Switches
- Used in **access networks** (closer to the edge — homes, offices).
- Operate at a lower level than routers.
- Forward frames within a local network.

> The sequence of communication links and packet switches that a packet 
> traverses from source to destination is called a **route** or **path** 
> through the network.

---

## ISPs — How You Actually Connect to the Internet

End systems don't connect to the Internet directly — they connect through 
**Internet Service Providers (ISPs)**.

### Types of ISPs

| ISP Type | Examples |
|---|---|
| Residential ISPs | Airtel, BSNL, JioFiber (at home) |
| Corporate ISPs | Company's own internet connection |
| University ISPs | IEM's campus network |
| Mobile ISPs | Jio, Vi (cellular data) |
| WiFi ISPs | Airport, hotel, coffee shop WiFi |

Each ISP is itself **a network of packet switches and communication links**.

### ISP Hierarchy

ISPs are organized in a tiered hierarchy:

- Lower-tier ISPs are **customers** of upper-tier ISPs.
- Upper-tier ISPs connect to each other directly (called **peering**).
- This creates the fully connected global Internet.
- Every ISP network — regardless of tier — independently runs the 
  **IP protocol** and conforms to naming/addressing conventions.

![[isp-hierarchy.png]]
*(Figure 1.1 — Mobile Network, Home Network, Enterprise Network all 
connecting upward through Local ISP → National ISP → Internet)*

---

## Protocols — The Rules of the Internet

End systems, packet switches, and every other Internet component run 
**protocols** that control how information is sent and received.

The two most important protocols on the Internet:

| Protocol | Full Name | Role |
|---|---|---|
| **TCP** | Transmission Control Protocol | Reliable delivery, ordering, flow control |
| **IP** | Internet Protocol | Specifies packet format, addressing |

Together they're called **TCP/IP** — the foundation of the Internet.

> The **IP protocol** specifically defines the format of packets that 
> are sent and received among routers and end systems. This is why 
> the Internet is sometimes called an **IP network**.

---

## Internet Standards — Who Decides the Rules?

For the Internet to work, everyone must agree on what each protocol does 
so systems from different manufacturers can interoperate.

This is handled by **standards bodies**:

### IETF — Internet Engineering Task Force
- Develops and maintains Internet standards.
- Documents are called **RFCs** — **Requests for Comments**.
- RFCs started as informal requests to solve network problems — the name stuck.
- RFCs are highly technical and define protocols precisely.
- Examples: TCP (RFC 793), IP (RFC 791), HTTP (RFC 2616), SMTP (RFC 5321).
- There are currently **6,000+ RFCs**.

### IEEE 802 Committee
- Handles standards for **network hardware links**.
- Covers **Ethernet** (wired LAN) and **WiFi** (wireless LAN) standards.

> 🔑 Standards ensure that a Jio router in Kolkata and a Comcast router 
> in New York can understand each other's packets — because they both 
> follow the same RFCs.

---

# 1.1.2 A Services Description

The second way to describe the Internet: as an **infrastructure that 
provides services to applications**.

---

## Distributed Applications

The Internet exists to support **applications**. Examples:

- Electronic mail (SMTP)
- Web surfing (HTTP/HTTPS)
- Social networks
- Instant messaging
- Voice-over-IP (VoIP) — WhatsApp calls, Google Meet
- Video streaming — YouTube, Netflix
- Peer-to-peer (P2P) file sharing
- Online/multiplayer games
- Remote login (SSH)

These are called **distributed applications** because they involve 
**multiple end systems that exchange data with each other**.

> ⚠️ Critical point: Internet applications run on **end systems** — 
> NOT on packet switches in the network core.
> 
> Packet switches facilitate the exchange of data between end systems, 
> but they don't care about what the application is or what data it's 
> exchanging.

---

## The Internet API

Here's a key question: *How does a program running on one end system 
instruct the Internet to deliver data to a program on another end system?*

Answer: through the **Internet API** (Application Programming Interface).

- End systems attached to the Internet provide an **API**.
- The API specifies how a program on one end system **asks** the Internet 
  infrastructure to deliver data to a destination program on another 
  end system.
- It is a **set of rules** that the sending program must follow.

> 💡 **Postal Service Analogy:**
> Alice wants to send a letter to Bob via postal service. She can't just 
> drop the letter out her window. The postal service has its own "API":
> - Put the letter in an envelope.
> - Write Bob's full name, address, and zip code in the center.
> - Seal the envelope.
> - Put a stamp in the top-right corner.
> - Drop it in an official mailbox.
>
> Only then will the postal service deliver it.
> The Internet API works the same way — rules must be followed for 
> delivery to happen.

The Internet provides **multiple services** to applications (like express 
delivery vs. ordinary mail). When you develop an Internet application, 
you choose which Internet service fits your needs. More on this in 
Chapter 2.

---

# 1.1.3 What Is a Protocol?

Now the most important concept in all of computer networking: 
**protocols**.

---

## Human Protocol Analogy

You follow protocols in everyday life without realizing it.

**Scenario: Asking someone for the time**

What if they responded with "Don't bother me!" or said nothing?
→ The protocol **breaks down** — no useful communication happens.

**Key observations from the human analogy:**
- There are **specific messages** we send.
- There are **specific actions** we take in response to received messages.
- If one party runs a **different protocol** (e.g., doesn't speak English), 
  no useful work can be accomplished.

**Second analogy: Asking a question in class**

Again — specific messages, specific actions, specific order.

---

## Network Protocol

A network protocol is the same idea, except the communicating entities 
are **hardware or software components** of devices (computers, routers, 
smartphones, etc.).

**Example — Requesting a Web Page (Figure 1.2):**

![[protocol-diagram.png]]
*(Figure 1.2 — Human protocol (left) vs. network protocol (right))*

Every step — the format of the message, the order they happen in, the 
actions taken — is defined by the protocol.

---

## The Formal Definition of a Protocol

> **"A protocol defines the format and the order of messages exchanged 
> between two or more communicating entities, as well as the actions 
> taken on the transmission and/or receipt of a message or other event."**
>
> — Kurose & Ross, *Computer Networking: A Top-Down Approach*

Breaking this definition down:

| Component | Meaning | Example |
|---|---|---|
| **Format** | What a message looks like | IP packet has a header with source/dest address |
| **Order** | The sequence messages must follow | SYN before SYN-ACK before ACK |
| **Actions** | What happens when a message is sent/received | Server opens a connection when it gets SYN |

---

## Protocols Are Everywhere on the Internet

| Where | Protocol | What it controls |
|---|---|---|
| Between two NICs | Hardware protocol | Flow of bits on the wire |
| End systems | Congestion-control protocol | Rate of packet transmission |
| Routers | Routing protocol | Path a packet takes |
| Web browsing | HTTP | Format of web requests/responses |
| Email | SMTP | Format and delivery of email |
| Reliable delivery | TCP | Ordering, retransmission, flow control |
| Addressing | IP | Packet format, addressing |

> 🔑 **The big idea:** All activity on the Internet involving two or more 
> communicating remote entities is governed by a protocol. Understanding 
> computer networking = understanding the **what, why, and how** of 
> networking protocols.

---

# Security Perspective on 1.1

| Concept | How Attackers Abuse It | How Defenders Use It |
|---|---|---|
| **Packets** | Inject malicious packets, packet sniffing | Deep packet inspection (DPI), firewalls |
| **IP Protocol** | IP spoofing (faking source address) | Ingress filtering, IPSec |
| **TCP Protocol** | SYN flood (exhaust server connections) | SYN cookies, rate limiting |
| **ISP hierarchy** | BGP hijacking (redirect traffic) | BGP monitoring, RPKI |
| **Protocols are predictable** | Exploit known message formats | Anomaly detection (deviation from RFC = suspicious) |
| **Internet API** | Abuse API endpoints, injection attacks | Input validation, authentication |

---

## Questions I Still Have

- [ ] How exactly does a router decide which outgoing link to forward a 
      packet on?
- [ ] What happens when two ISPs don't have a peering agreement — does 
      the packet get dropped?
- [ ] How is the Internet API different for TCP vs UDP services?
- [ ] What does a real RFC document look like — how technical is it?
- [ ] How does IP spoofing actually work at the packet level?

---

## Key Terms — Quick Reference

| Term | Definition |
|---|---|
| **Host / End System** | Any device connected to the Internet |
| **Packet** | A chunk of data with header bytes, sent through the network |
| **Transmission rate** | Speed of a link in bits/second |
| **Packet switch** | Device that forwards packets toward their destination |
| **Router** | Packet switch used in the network core |
| **Link-layer switch** | Packet switch used in access networks |
| **Route / Path** | Sequence of links and switches a packet traverses |
| **ISP** | Internet Service Provider — how end systems access the Internet |
| **TCP** | Transmission Control Protocol — reliable delivery |
| **IP** | Internet Protocol — packet format and addressing |
| **RFC** | Request for Comments — IETF standard documents |
| **IETF** | Internet Engineering Task Force — develops Internet standards |
| **Distributed application** | App running across multiple end systems |
| **Internet API** | Rules a program follows to use Internet delivery services |
| **Protocol** | Rules defining format, order, and actions for communication |

---

## Related Concepts

- [[TCP - Three Way Handshake]]
- [[IP Addressing and Subnetting]]
- [[OSI Model vs TCP-IP Model]]
- [[ISP Hierarchy and BGP]]
- [[HTTP and the Web]]

---

→ Next: [[1.2 - The Network Edge]]