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

> **One-Line Summary:** The Internet is a global system of billions of interconnected devices communicating through shared rules called protocols (TCP/IP), serving both as a hardware/software infrastructure and as a platform for distributed applications.

---

# 1.1.1 A Nuts-and-Bolts Description

There are two ways to answer "What is the Internet?":

1. Describe its **physical components** (hardware + software)
2. Describe it as a **service infrastructure** for applications

This section covers approach #1.

---

## End Systems (Hosts)

The Internet connects billions of **computing devices** across the world. Originally these were desktop PCs, Linux workstations, and servers. Today the list includes:

- Laptops, smartphones, tablets
- TVs, gaming consoles, webcams
- Automobiles, environmental sensors
- Home electrical and security systems
- Watches, eyeglasses, robots, and an ever-growing list of "smart" household objects

> All of these devices are called **hosts** or **end systems** — the two terms mean exactly the same thing and are used interchangeably throughout networking literature.

The term _computer network_ is starting to sound dated given how many non-traditional devices are connected — but the name stuck.

Roughly two-thirds of the world's population are active mobile Internet users. On the device-count side, there were about 19 billion devices connected to the Internet in 2025, with that number projected to reach 29 billion by 2030 — a scale that keeps climbing fast enough that any fixed number is stale within a couple of years.


---

## The Big Picture — How the Pieces Fit Together

It helps to see the whole Internet laid out as a set of named, interconnected network types before going further, since later sections build directly on this picture.

![[Pasted image 20260619122429.png]]
 _(NEW — add the network topology diagram here: a Home Network and Mobile Network each connect through a Local or Regional ISP, which connects up to a National or Global ISP; an Enterprise Network also connects through a Local or Regional ISP; Datacenter Networks and Content Provider Networks connect in directly at the upper-tier level alongside the National/Global ISP)_

|Network Type|What lives inside it|How it connects in|
|---|---|---|
|**Home Network**|Laptops, smartphones, smart fridges, thermostats, workstations — ordinary household devices|Connects up through a Local or Regional ISP|
|**Mobile Network**|Smartphones, cars, traffic lights, cell towers (base stations)|Connects up through a National or Global ISP|
|**Enterprise Network**|Company workstations, internal servers, office devices|Connects up through a Local or Regional ISP|
|**Local or Regional ISP**|A regional packet-switch network that aggregates traffic from homes, mobile users, and enterprises|Connects up to a National or Global ISP|
|**National or Global ISP**|A large-scale backbone network of high-speed routers|Peers directly with other top-tier ISPs, and connects down to regional ISPs, datacenters, and content providers|
|**Datacenter Network**|Racks of servers — the physical infrastructure behind cloud services|Connects directly into the upper-tier ISP layer|
|**Content Provider Network**|Infrastructure run by companies like streaming or search providers to serve their own content directly|Connects directly into the upper-tier ISP layer, bypassing part of the ISP hierarchy for their own traffic|

The icon key distinguishes hosts and servers from network equipment: a **host** icon represents an end-system device generically, a **server** icon represents a dedicated always-on machine, **mobile computer** and **smartphone/tablet** icons represent portable end systems, **router** and **link-layer switch** icons represent the packet-switching equipment described below, and a **base station** icon represents the cell tower a mobile device connects through. Non-traditional "things" get their own icons too — datacenter racks, workstations, traffic lights, thermostats, even a fridge.

The biggest structural point worth taking from this picture: **datacenters and large content providers sit in the top tier of the hierarchy**, alongside national and global ISPs, rather than being just another set of end systems hanging off the edge. A huge share of Internet traffic today is generated and consumed by large content and cloud providers rather than flowing purely between ordinary end users, and the topology reflects that directly.

---

## Communication Links

End systems are connected to each other through **communication links**.

Links are made of different types of **physical media**:

|Media Type|Examples|Notes|
|---|---|---|
|Coaxial cable|Cable TV lines|Copper, shielded|
|Copper wire|Telephone lines, Ethernet|Most common wired|
|Optical fiber|Undersea cables, ISP backbone|Extremely fast, light-based|
|Radio spectrum|WiFi, 4G/5G, Bluetooth|Wireless, no physical wire|

- Each link can transmit data at different speeds.
- The speed of a link is called its **transmission rate**, measured in **bits per second (bps)** — or Kbps, Mbps, Gbps.

---

## Packets — How Data Actually Travels

When one end system wants to send data to another, it doesn't send the whole thing at once. Here's what actually happens:

1. The sending end system **segments** the data into smaller chunks.
2. It adds **header bytes** to each chunk (containing addressing, ordering info, etc.).
3. These chunks — called **packets** — are sent independently through the network.
4. At the destination, the packets are **reassembled** back into the original data.

> **Truck Analogy:** Imagine a factory needs to ship a huge load of cargo across the country.
> 
> - The cargo is split and loaded into many trucks.
> - Each truck independently travels through highways and intersections.
> - At the destination warehouse, all cargo is unloaded and regrouped.

|Network Term|Analogy|
|---|---|
|Data|Cargo|
|Packet|Truck|
|Communication link|Highway / road|
|Packet switch|Intersection|
|End system|Building (factory/warehouse)|

---

## Packet Switches

A **packet switch** receives a packet on one of its incoming links and forwards it on one of its outgoing links — moving it closer to its destination.

Two most prominent types of packet switches:

### Routers

- Used in the **network core** (the middle of the Internet).
- Make intelligent forwarding decisions based on destination IP address.
- Forward packets toward their ultimate destination.
- **Analogy: simply connects different networks to transfer data.**

### Link-Layer Switches

- Used in **access networks** (closer to the edge — homes, offices).
- Operate at a lower level than routers.
- Forward frames within a local network.
- **Analogy: connects hosts/end systems within the same local network.**

> The sequence of communication links and packet switches that a packet traverses from source to destination is called a **route** or **path** through the network.

![[Pasted image 20260619123700.png]]

---

## ISPs — How You Actually Connect to the Internet

End systems don't connect to the Internet directly — they connect through **Internet Service Providers (ISPs)**.

### Types of ISPs

|ISP Type|Examples|
|---|---|
|Residential ISPs|Airtel, BSNL, JioFiber (at home)|
|Corporate ISPs|Company's own internet connection|
|University ISPs|IEM's campus network|
|Mobile ISPs|Jio, Vi (cellular data)|
|WiFi ISPs|Airport, hotel, coffee shop WiFi|

Each ISP is itself **a network of packet switches and communication links**.

### ISP Hierarchy

ISPs are organized in a tiered hierarchy:

- Lower-tier ISPs are **customers** of upper-tier ISPs.
- Upper-tier ISPs connect to each other directly (called **peering**).
- This creates the fully connected global Internet.
- Every ISP network — regardless of tier — independently runs the **IP protocol** and conforms to naming/addressing conventions.

Datacenters and large content providers fit into this hierarchy too, but not as ordinary edge customers — they connect directly into the upper-tier ISP layer, alongside national and global ISPs (see the topology table above). This matters because so much of today's traffic — video streaming, cloud services, social media — originates from a relatively small number of massive content and datacenter operators rather than flowing peer-to-peer between ordinary residential connections.

![[Pasted image 20260619123711.png]]

---

## Protocols — The Rules of the Internet

End systems, packet switches, and every other Internet component run **protocols** that control how information is sent and received.

The two most important protocols on the Internet:

|Protocol|Full Name|Role|
|---|---|---|
|**TCP**|Transmission Control Protocol|Reliable delivery, ordering, flow control|
|**IP**|Internet Protocol|Specifies packet format, addressing|

Together they're called **TCP/IP** — the foundation of the Internet.

> The **IP protocol** specifically defines the format of packets that are sent and received among routers and end systems. This is why the Internet is sometimes called an **IP network**.

### Internet Standards — Who Actually Defines These Protocols?

For two independently-built systems to interoperate, everyone has to agree on exactly what a given protocol does — this is the entire reason standards exist. The body responsible for developing most core Internet standards is the **Internet Engineering Task Force (IETF)**. IETF standards documents are called **Requests for Comments (RFCs)** — a name that's a historical leftover from when these documents started out as informal requests for feedback on protocol and network design problems during the Internet's early days, rather than the formal specifications they've since become. RFCs are typically dense and technical, and they're what define protocols like TCP, IP, HTTP (the Web), and SMTP (e-mail) in full detail. There are currently close to 9,000 RFCs.

The IETF isn't the only standards body relevant to networking — other organizations specify standards for specific kinds of network components, most notably for the physical network links themselves. The **IEEE 802 LAN Standards Committee**, for instance, is the body responsible for specifying Ethernet and WiFi standards.

|Standards Body|What it specifies|Document type|
|---|---|---|
|**IETF**|Core Internet protocols: TCP, IP, HTTP, SMTP, and roughly 9,000 others|RFC (Request for Comments)|
|**IEEE 802 LAN Standards Committee**|Link-layer/physical standards: Ethernet, WiFi|IEEE 802 standards|

---

# 1.1.2 A Services Description

The second way to describe the Internet: as an **infrastructure that provides services to applications**.

---

## Distributed Applications

The Internet exists to support **applications**. Examples:

- Electronic mail (SMTP)
- Web surfing (HTTP/HTTPS)
- Internet messaging
- Mapping with real-time road-traffic information
- Music, movie, and television streaming
- Online social media
- Video conferencing — Google Meet, Zoom
- Voice-over-IP (VoIP) — WhatsApp calls
- Multi-person online games
- Location-based recommendation systems
- Peer-to-peer (P2P) file sharing
- Remote login (SSH)

These are called **distributed applications** because they involve **multiple end systems that exchange data with each other**.

> Critical point: Internet applications run on **end systems** — NOT on packet switches in the network core. Packet switches facilitate the exchange of data between end systems, but they don't care about what the application is or what data it's exchanging.

---

## The Internet Socket Interface

Here's a key question: how does a program running on one end system instruct the Internet to deliver data to a program on another end system?

End systems attached to the Internet provide a **socket interface** (sometimes called the Internet API) that specifies how a program on one end system asks the Internet infrastructure to deliver data to a specific destination program on another end system. This socket interface is a set of rules that the sending program must follow so that the Internet can deliver the data to the destination program. This is the same interface that gets implemented concretely in Chapter 2 with actual socket code — `socket()`, `bind()`, `connect()`, and so on.

> **Postal Service Analogy:** Alice wants to send a letter to Bob via postal service. She can't just drop the letter out her window. The postal service has its own "interface," a set of rules she has to follow:
> 
> - Put the letter in an envelope.
> - Write Bob's full name, address, and zip code in the center.
> - Seal the envelope.
> - Put a stamp in the top-right corner.
> - Drop it in an official mailbox.
> 
> Only then will the postal service deliver it. The Internet's socket interface works the same way — rules must be followed before delivery happens, and the postal service itself doesn't care what's written inside the envelope, just as packet switches don't care what application data they're carrying.

The Internet provides **multiple services** to applications, just as the postal service offers express delivery alongside ordinary mail. When building an Internet application, the developer chooses which Internet service fits the need — covered in full in Chapter 2.

![[Pasted image 20260619123722.png]]

---

# 1.1.3 What Is a Protocol?

Now the most important concept in all of computer networking: **protocols**.

---

## Human Protocol Analogy

People follow protocols in everyday life without realizing it.

**Scenario: Asking someone for the time**

What if they responded with "Don't bother me!" or said nothing? The protocol **breaks down** — no useful communication happens.

**Key observations from the human analogy:**

- There are **specific messages** that get sent.
- There are **specific actions** taken in response to received messages.
- If one party runs a **different protocol** (e.g., doesn't speak English), no useful work can be accomplished.

**Second analogy: asking a question in class**

Again — specific messages, specific actions, specific order: raising a hand (an implicit message to the teacher), getting acknowledged, asking the question, receiving an answer.

---

## Network Protocol

A network protocol is the same idea, except the communicating entities are **hardware or software components** of devices — computers, routers, smartphones, or any other network-capable device.

**Example — Requesting a Web Page:**

![[Pasted image 20260619123732.png]]

When a browser fetches a Web page, the computer first sends a connection request message to the Web server and waits for a reply; the server returns a connection reply message; once the connection is confirmed, the computer sends a `GET` message naming the specific page it wants — for example, `GET http://www.pearsonhighered.com/cs-resources/`; finally, the server returns the requested file. Every step — the format of each message, the order they happen in, the actions taken on sending or receiving — is defined by the protocol, here HTTP running on top of a TCP connection.

---

## The Formal Definition of a Protocol

> **"A protocol defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event."** — Kurose & Ross, _Computer Networking: A Top-Down Approach_

Breaking this definition down:

|Component|Meaning|Example|
|---|---|---|
|**Format**|What a message looks like|IP packet has a header with source/dest address|
|**Order**|The sequence messages must follow|SYN before SYN-ACK before ACK|
|**Actions**|What happens when a message is sent/received|Server opens a connection when it gets SYN|

---

## Protocols Are Everywhere on the Internet

|Where|Protocol|What it controls|
|---|---|---|
|Between two NICs|Hardware protocol|Flow of bits on the wire|
|End systems|Congestion-control protocol|Rate of packet transmission|
|Routers|Routing protocol|Path a packet takes|
|Web browsing|HTTP|Format of web requests/responses|
|Email|SMTP|Format and delivery of email|
|Reliable delivery|TCP|Ordering, retransmission, flow control|
|Addressing|IP|Packet format, addressing|

> The big idea: all activity on the Internet involving two or more communicating remote entities is governed by a protocol. Understanding computer networking essentially means understanding the **what, why, and how** of networking protocols.

---

# Security Perspective on 1.1

|Concept|How Attackers Abuse It|How Defenders Use It|
|---|---|---|
|**Packets**|Inject malicious packets, packet sniffing|Deep packet inspection (DPI), firewalls|
|**IP Protocol**|IP spoofing (faking source address)|Ingress filtering, IPSec|
|**TCP Protocol**|SYN flood (exhaust server connections)|SYN cookies, rate limiting|
|**ISP hierarchy**|BGP hijacking (redirect traffic)|BGP monitoring, RPKI|
|**Protocols are predictable**|Exploit known message formats|Anomaly detection (deviation from RFC = suspicious)|
|**Socket interface**|Abuse exposed sockets/endpoints, injection attacks|Input validation, authentication, TLS-wrapped sockets|
|**Datacenter/content-provider tier**|Compromise a major content provider to attack a huge downstream user base at once (supply-chain-style blast radius)|Hardened datacenter network segmentation, strict peering controls at the upper-tier ISP layer|

---

## Questions I Still Have

- [ ] How exactly does a router decide which outgoing link to forward a packet on?
- [ ] What happens when two ISPs don't have a peering agreement — does the packet get dropped, or rerouted through a paid transit path?
- [ ] How is the socket interface different in practice for TCP vs UDP services? (Section 2.7 covers this directly — worth cross-referencing.)
- [ ] How does IP spoofing actually work at the packet level?
- [ ] Now that content providers and datacenters connect directly into the upper-tier ISP layer instead of sitting at the edge, does that change anything about how routing or peering disputes between ISPs and content providers actually play out?
- [ ] Are RFCs ever formally retired or superseded, or do all ~9,000 of them stay permanently valid as historical record even after a protocol changes?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Host / End System**|Any device connected to the Internet|
|**Packet**|A chunk of data with header bytes, sent through the network|
|**Transmission rate**|Speed of a link in bits/second|
|**Packet switch**|Device that forwards packets toward their destination|
|**Router**|Packet switch used in the network core|
|**Link-layer switch**|Packet switch used in access networks|
|**Route / Path**|Sequence of links and switches a packet traverses|
|**ISP**|Internet Service Provider — how end systems access the Internet|
|**Datacenter Network**|A network of servers run by a cloud/service provider, connecting directly into the upper-tier ISP hierarchy|
|**Content Provider Network**|Infrastructure run directly by a content company (streaming, social, etc.) to serve its own traffic, also connecting at the upper-tier level|
|**TCP**|Transmission Control Protocol — reliable delivery|
|**IP**|Internet Protocol — packet format and addressing|
|**IETF**|Internet Engineering Task Force — the body that develops most core Internet standards|
|**RFC**|Request for Comments — the document format IETF standards are published as; nearly 9,000 exist|
|**IEEE 802**|The standards committee responsible for Ethernet and WiFi specifications|
|**Distributed application**|App running across multiple end systems|
|**Socket interface**|The set of rules a program follows to ask the Internet infrastructure to deliver data to another program on another end system (also called the Internet API)|
|**Protocol**|Rules defining format, order, and actions for communication|

---

## Related Concepts

- 

---

→ Next: [[THE NETWORK EDGE]]