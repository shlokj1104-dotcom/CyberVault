---
title: THE NETWORK CORE
date: 2026-05-15
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 1.3 The Network Core

> **One-Line Summary:** The network core is the mesh of packet switches and links interconnecting the Internet's end systems — it moves data using packet switching (store-and-forward, on-demand) rather than circuit switching (reserved, dedicated paths), and is structured as a multi-tier network of networks built from access ISPs up to global tier-1 ISPs.

Having examined the Internet's edge, we now delve more deeply into the network core — the mesh of packet switches and links that interconnects the Internet's end systems.

![[Pasted image 20260619130757.png]]
_(Figure 1.10 — The network core: mobile, home, and enterprise networks connect through a local/regional ISP up to a national/global ISP, which interconnects with datacenter networks and content provider networks. The shaded mesh of routers and links between these access points is the "network core.")_

---

# 1.3.1 Packet Switching

## What Are Messages and Packets?

In a network application, end systems exchange **messages** with each other. Messages can contain anything — control information (like a "Hi" handshake), data (an email, a JPEG image, an MP3 file).

To send a message from source to destination:

1. The source **breaks long messages into smaller chunks** called **packets**.
2. Each packet travels independently through communication links and packet switches — for which there are **two predominant types: routers and link-layer switches**.
3. At the destination, packets are **reassembled** into the original message.

**Key formula — Transmission Time:**

> If a source end system or packet switch sends a packet of **L bits** over a link with transmission rate **R bits/sec**, then:
> 
> **Time to transmit = L/R seconds**

Packets are transmitted at the **full transmission rate** of the link — not shared with other packets mid-transmission.

---

## Store-and-Forward Transmission

Most packet switches use **store-and-forward transmission**.

> **Definition:** The packet switch must receive the **entire packet** before it can begin to transmit the first bit of the packet onto the outbound link.

### Why Store-and-Forward?

The router needs to:

1. **Store** — buffer all incoming bits of the packet
2. **Process** — check the packet header, look up the forwarding table
3. **Forward** — transmit the entire packet onto the outgoing link

It cannot start retransmitting while still receiving — it must have the complete packet first.

### End-to-End Delay Calculation

![[Pasted image 20260619131156.png]] _(Figure 1.11 — Store-and-Forward Packet Switching: Source → Router → Destination)_

**Simple case: 1 packet, 1 router, 2 links**

|Time|Event|
|---|---|
|t = 0|Source begins transmitting packet (L bits at R bps)|
|t = L/R|Source finishes; router has received entire packet|
|t = L/R|Router begins forwarding onto outbound link|
|t = 2L/R|Destination receives entire packet|

**Total delay = 2L/R** (with one router, ignoring propagation delay — the time it takes bits to travel across the wire at near the speed of light, covered in Section 1.4)

> Compare: if the router forwarded bits as soon as they arrived (without store-and-forward), total delay would be only L/R. The store-and-forward mechanism adds L/R of delay at each router.

**General formula — N links, one packet:**

$$d_{\text{end-to-end}} = N \cdot \frac{L}{R} \tag{1.1}$$

Where:

- **N** = number of links (= number of routers + 1)
- **L** = packet size in bits
- **R** = transmission rate of each link in bps

**Three packets sent over 1 router:**

|Time|Event|
|---|---|
|t = L/R|Router begins forwarding packet 1; source starts packet 2|
|t = 2L/R|Destination gets packet 1; router gets packet 2; source starts packet 3|
|t = 3L/R|Destination gets packet 2; router gets packet 3|
|t = 4L/R|Destination gets packet 3 — all done!|

> 💡 Pipelining effect: source and router work simultaneously after the first packet — this is why packet switching is efficient.

---

## Queuing Delays and Packet Loss

Each packet switch has multiple links attached to it. For each link, the switch maintains an **output buffer** (also called **output queue**) — a storage area for packets waiting to be transmitted on that link.

![[Pasted image 20260619131342.png]] _(Figure 1.12 — Packet Switching: Hosts A and B sending to Host E via a congested 1.5 Mbps link, queue forming at router. Packets are drawn as three-dimensional slabs whose width represents the number of bits in the packet.)_

### Why Queuing Happens

If a packet arrives at a router but the outbound link is **busy** transmitting another packet, the arriving packet must **wait in the output buffer**.

This causes **queuing delay** — variable, depends on network congestion.

> **Analogy:** Like waiting in a bank queue. The more people ahead of you, the longer you wait. If everyone arrives at once (burst traffic), the queue grows.

### Packet Loss

Since buffer space is **finite**:

- If a packet arrives and the buffer is **completely full**, the router must **drop** a packet.
- Either the arriving packet is dropped, or one of the already-queued packets is dropped.
- This is called **packet loss**.

### Summary of Delay Types in Packet Switching

|Delay Type|Cause|Variable?|
|---|---|---|
|**Store-and-forward delay**|Must receive full packet before forwarding|No (fixed: L/R)|
|**Queuing delay**|Waiting in output buffer when link is busy|Yes (depends on traffic)|
|**Transmission delay**|Time to push all bits onto the link|No (fixed: L/R)|
|**Propagation delay**|Signal travel time across the physical link|No (fixed: d/s)|

_(Propagation and transmission delay details covered in Section 1.4)_

---

## Forwarding Tables and Routing Protocols

**Key question:** How does a router know which outgoing link to forward a packet onto?

### IP Addresses and Packet Headers

- Every end system has an **IP address**.
- When a source wants to send a packet, it includes the **destination's IP address** in the packet's header.
- IP addresses have a **hierarchical structure** (like a postal address — country → city → street → house number).

### Forwarding Tables

When a packet arrives at a router:

1. The router examines a **portion** of the packet's destination address.
2. It searches its **forwarding table** — a mapping of destination address ranges to outbound links.
3. It forwards the packet to the appropriate outbound link.

> 💡 **Road trip analogy:** Joe drives from Philadelphia to 156 Lakeside Drive in Orlando without a map. At each gas station (router), he asks for directions. The attendant (router) reads just the relevant portion of the address (state → city → street → house) and points him to the next step. Gas station attendants = routers. The address portion they read = the forwarding table lookup.

### Routing Protocols

**How do forwarding tables get set?**

The Internet uses special **routing protocols** that **automatically set the forwarding tables**. A routing protocol may:

- Determine the shortest path from each router to each destination.
- Use those shortest-path results to configure the forwarding tables.

> Routing protocols are covered in detail in Chapter 4. You can use **Traceroute** (www.traceroute.org) to see the actual end-to-end route packets take through the Internet right now.

---

# 1.3.2 Circuit Switching

There are **two fundamental approaches** to moving data through a network of links and switches:

|Approach|Resource Reservation|Guarantee|
|---|---|---|
|**Circuit Switching**|Resources **reserved** for the session duration|Guaranteed constant rate|
|**Packet Switching**|Resources used **on demand**, not reserved|Best effort, no guarantee|

---

## How Circuit Switching Works

In circuit-switched networks, resources (buffers, link transmission rate) along a path are **reserved** for the duration of the communication session.

**Traditional telephone networks** are the classic example:

1. Before sending information, the network **establishes a connection** between sender and receiver.
2. This is a **bona fide connection** — switches on the path maintain state for this connection.
3. In telephony jargon this is called a **circuit**.
4. A **constant transmission rate** is reserved in the network's links for this circuit — the sender can always transmit at the guaranteed rate.

![[Pasted image 20260619131517.png]] _(Figure 1.13 — A Simple Circuit-Switched Network: 4 switches, 4 links, each link supporting 4 simultaneous circuits)_

**Example calculation:**

- Each link has transmission rate 1 Mbps.
- Each link supports 4 circuits.
- Each circuit gets 1/4 of link capacity = **250 kbps** dedicated rate.

> 🍽️ **Restaurant analogy:**
> 
> - **Circuit switching** = restaurant requiring reservations. You call ahead, your table is held even if you're late. Guaranteed seat, but wasted if you don't show.
> - **Packet switching** = no-reservations restaurant. Walk in and wait if full, but no resources are wasted holding tables for no-shows.

---

## Multiplexing in Circuit-Switched Networks

A circuit in a link is implemented using one of two multiplexing approaches:

### FDM — Frequency-Division Multiplexing

- The **frequency spectrum** of the link is divided among connections.
- Each connection gets a **dedicated frequency band** for the duration.
- The width of the band is called the **bandwidth**.
- Telephone networks: each band is typically **4 kHz** wide.
- FM radio also uses FDM — each station gets its own frequency band between 88 MHz and 108 MHz.

### TDM — Time-Division Multiplexing

- **Time** is divided into **frames** of fixed duration.
- Each frame is divided into a fixed number of **time slots**.
- Each circuit is assigned **one slot per frame** — exclusively dedicated to that connection.
- Slot is available in every revolving frame.

**TDM Example:**

- Link transmits 8,000 frames/second.
- Each slot = 8 bits.
- Transmission rate of one circuit = 8,000 × 8 = **64 kbps**.

![[Pasted image 20260619131657.png]] _(Figure 1.14 — FDM vs TDM: FDM gives each circuit a frequency band; TDM gives each circuit a time slot in every frame)_

### Key Weakness of Circuit Switching — Silent Periods

> Proponents of packet switching argue that circuit switching is **wasteful** because dedicated circuits are **idle during silent periods**.

- During a phone call, when you stop talking → frequency band or time slot is idle → **cannot be used by other connections**.
- A radiologist viewing X-rays over circuit-switched network wastes resources during contemplation time.

---

## Circuit Switching: Worked Example

**Problem:**

- Send a file of 640,000 bits from Host A to Host B.
- All links use TDM with 24 slots, bit rate = 1.536 Mbps.
- Circuit establishment takes 500 ms.

**Solution:**

- Each circuit's transmission rate = 1.536 Mbps / 24 = **64 kbps**
- Time to transmit file = 640,000 / 64,000 = **10 seconds**
- Add circuit establishment time: 10 + 0.5 = **10.5 seconds total**

> Note: Transmission time is independent of the number of links in the path — because the full rate is reserved end-to-end.

---

## Packet Switching vs. Circuit Switching

|Feature|Packet Switching|Circuit Switching|
|---|---|---|
|Resource reservation|None — on demand|Yes — reserved upfront|
|Delay|Variable queuing delays|Fixed, predictable delay|
|Efficiency|High — resources shared|Low — idle circuits waste bandwidth|
|Complexity|Simpler|Complex (signaling to set up circuit)|
|Good for|Bursty data (web, email, files)|Real-time (voice calls)|
|Congestion risk|Yes — queuing, packet loss|No — guaranteed rate|
|Supports more users|Yes|No|

> 📚 For a deeper comparison of the two approaches, see [Molinero-Fernandez 2002].

### Why Packet Switching Is More Efficient — Example

**Setup:**

- 1 Mbps shared link.
- Each user: active 10% of the time at 100 kbps; idle 90% of time.

**Circuit switching:**

- Must reserve 100 kbps per user at all times.
- Maximum users = 1 Mbps / 100 kbps = **10 users**.

**Packet switching:**

- P(user active) = 0.1
- With 35 users, P(11 or more simultaneously active) = **0.0004** (very small).
- So 35 users can share the link, and 99.96% of the time no congestion occurs.
- **Packet switching supports 3.5× more users** with essentially the same performance.

> When ≤10 users are active simultaneously (which happens with probability 0.9996), aggregate arrival rate ≤ 1 Mbps → no queuing. Packet switching achieves circuit-switching-like performance while serving far more users.

### Second Example — Burst Traffic

- 10 users, one suddenly sends 1,000 × 1,000-bit packets back-to-back.
- Other 9 users inactive.

**TDM circuit switching (10 slots):**

- Active user gets 1 slot → 1/10 of link rate.
- Time to transmit 1 million bits = **10 seconds**.

**Packet switching:**

- No other users → active user uses full 1 Mbps link rate.
- Time to transmit 1 million bits = **1 second**.

> Packet switching allocates link use **on demand** — idle capacity isn't wasted. Circuit switching pre-allocates regardless of demand.

### The Trend

Although both exist today, the trend is **toward packet switching**:

- Even telephone networks increasingly use packet switching for expensive overseas calls.
- Circuit-switched telephone networks are slowly migrating to packet switching.

---

# 1.3.3 A Network of Networks

## Why ISPs Must Interconnect

End systems connect to the Internet via **access ISPs** (DSL, cable, FTTH, WiFi, cellular). An access ISP doesn't have to be a telco or cable company — it can be a university (giving students, staff, and faculty access) or a company (giving employees access). But connecting end users to their access ISP solves only part of the problem.

> The access ISPs themselves must be **interconnected** so that any end system can send packets to any other end system — anywhere in the world. This interconnection creates a **network of networks**.

Over the years, this network of networks has evolved into a very complex structure — much of it driven by **economics and national policy**, rather than by performance considerations alone.

---

## Evolution of Network Structure

The Internet's ISP interconnection structure evolved through five conceptual "network structures":

### Network Structure 1 — Single Global Transit ISP

- One imaginary global ISP connects all access ISPs.
- The global ISP spans the globe with routers near every access ISP.
- Access ISPs are **customers**; the global ISP is the **provider**.
- Very expensive to build — not realistic.

![[Pasted image 20260619131901.png]]

### Network Structure 2 — Multiple Global Transit ISPs

- Multiple competing global transit ISPs emerge.
- Access ISPs can choose among competing global providers based on price/service.
- Global transit ISPs must **interconnect** so their customers can reach each other.
- Two-tier hierarchy: global ISPs at top, access ISPs at bottom.

![[Pasted image 20260619132044.png]]

### Network Structure 3 — Regional ISPs Added

- In reality, no global ISP has presence in every city.
- In each region, there is a **regional ISP** to which local access ISPs connect.
- Each regional ISP connects to a **tier-1 ISP**.
- Tier-1 ISPs actually exist: ~12 of them, including **Level 3 Communications, AT&T, Sprint, NTT**.
- No official body grants tier-1 status — if you have to ask, you're probably not.
- Multi-tier hierarchy: tier-1 → regional → access.

> In China: access ISPs → provincial ISPs → national ISPs → tier-1 ISPs. The hierarchy can be as deep as the country's network warrants [Tian 2012].

![[Pasted image 20260515143142.png]]

### Network Structure 4 — PoPs, Multi-Homing, Peering, IXPs Added

Building on Structure 3, we add four more elements:

**PoP (Point of Presence):**

- A group of one or more routers at the **provider's location** where customer ISPs can connect into the provider.
- Exists at all levels of the hierarchy except the access ISP level.
- A customer ISP connects to a PoP by leasing a high-speed link from a third-party telecom provider.

**Multi-Homing:**

- Any ISP (except tier-1) may connect to **two or more provider ISPs**.
- If one provider fails, traffic can still flow through the other.
- Access ISP may multi-home with two regional ISPs.
- Regional ISP may multi-home with multiple tier-1 ISPs.

**Peering:**

- A pair of **nearby ISPs at the same level** can directly connect their networks.
- All traffic between them passes over this **direct link** rather than through upstream intermediaries → reduces cost.
- Typically **settlement-free** — neither ISP pays the other.
- Tier-1 ISPs also peer with one another, settlement-free.
- See [Van der Berg 2008] for a readable discussion of peering vs. customer-provider relationships.

**IXP (Internet Exchange Point):**

- A third-party facility (typically a standalone building with its own switches [Ager 2012]) where **multiple ISPs can peer together**.
- There are **over 600 IXPs** in the Internet today [PeeringDB 2025].
- Reduces cost for all parties — traffic stays local rather than transiting expensive upstream links.

![[Pasted image 20260515144248.png]]

### Network Structure 5 — Content Provider Networks Added

**Network Structure 5 = Structure 4 + Content Provider Networks**

**Example: Google**

- Over 20 major data centers across North America, Europe, Asia, and South America [Google datacenters 2024], each with tens or hundreds of thousands of servers.
- Additionally, smaller data centers with just a few hundred servers each — these are often located **within IXPs**.
- All data centers are interconnected via Google's **private TCP/IP network** spanning the globe — separate from the public Internet, and carrying **only Google's own traffic**.
- Google's network attempts to **bypass upper tiers** by:
    - Peering directly with lower-tier ISPs (settlement-free) [Labovitz 2010].
    - Connecting with ISPs at IXPs.
- However, many access ISPs are still only reachable through tier-1 networks → Google also connects to tier-1 ISPs and **pays** for that traffic.

_(Figure 1.15 — Google cloud locations and network, as of 2024 [Google Cloud 2025]: a world map showing Google's data center sites — e.g. Havfrue (US, IE, DK), Grace Hopper (US, UK, ES) 2022, Dunant (US, FR), Equiano (PT, NG, ZA) 2021, Curie (CL, US, PA), Junior, Monet, Tannat, Raman (SA, JO, DJ, OM, IN) 2024, Indigo-West/Central (SG, AU), JGA-S (GU, AU), Echo (US, SG, ID) 2023, Unity/FASTER/PLCN (US, JP, TW) — connected by both current network links and submarine cable investments spanning the Americas, Europe, Africa, and Asia-Pacific.)_

![[Pasted image 20260515144646.png]] _(Figure 1.16 — Interconnection of ISPs and a content provider: Tier-1 ISPs, a content provider such as Google, and regional/access ISPs all interconnecting both directly and via IXPs)_

**Why content providers build their own networks:**

- Reduces payments to upper-tier ISPs.
- Greater control over how services are delivered to end users.

> 📊 **Industry-wide trend:** As of 2020, Amazon, Google, IBM, and Microsoft collectively were able to reach **76% of the Internet without passing through Tier-1 ISPs** [Arnold 2020] — content providers building their own networks is no longer a Google-only strategy.

---

## ISP Hierarchy — Customer-Provider Relationships

|ISP Level|Pays|Receives payment from|
|---|---|---|
|**Tier-1 ISPs**|Nobody (at the top)|Regional ISPs and content providers|
|**Regional ISPs**|Tier-1 ISPs|Access ISPs|
|**Access ISPs**|Regional (or tier-1 directly)|End users/businesses|

> Tier-1 ISPs **do not pay anyone** — they are at the top of the hierarchy. Everyone below pays someone above for connectivity.

---

## Internet Today — Final Picture

Today's Internet:

- **~12 tier-1 ISPs** with global coverage.
- **Hundreds of thousands of lower-tier ISPs** — diverse geographic reach.
- Lower-tier ISPs connect to higher-tier ISPs.
- Higher-tier ISPs interconnect with one another.
- Users and content providers are customers of lower-tier ISPs.
- Lower-tier ISPs are customers of higher-tier ISPs.
- Major content providers (Google, Meta, Netflix) have created their own networks and connect directly into lower-tier ISPs where possible.
- As of 2020, the largest cloud providers (Amazon, Google, IBM, Microsoft) combined could already reach **76%** of the Internet without ever transiting a tier-1 ISP [Arnold 2020].

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Packet switching — no reservation**|Flood the network with packets → fill output buffers → cause packet loss (DoS/DDoS)|Rate limiting, traffic shaping, ingress filtering|
|**Store-and-forward**|Inject malicious packets — routers must process entire packet before forwarding (slow path attacks)|Deep Packet Inspection (DPI) at routers|
|**Forwarding tables**|BGP hijacking — poison routing tables to redirect traffic|RPKI (Resource Public Key Infrastructure), BGP monitoring|
|**Peering/IXPs**|Man-in-the-middle at IXP — traffic passing through shared switches can be monitored|Encrypt traffic end-to-end (TLS, IPSec)|
|**Content provider networks**|Attacker that compromises Google's private network can affect millions|Google uses its own security operations, separate from public Internet|
|**Queuing delays**|Exploit variable queuing delays to infer network state (timing side-channels)|Randomize packet scheduling, add jitter|

---

## Questions I Still Have

- [ ] How exactly does a router build its forwarding table — what does a routing protocol message look like?
- [ ] If two ISPs peer at an IXP, how do they agree on what traffic to exchange settlement-free vs. paid?
- [ ] In packet switching, if the output buffer is full, which packet gets dropped — the new arrival or an existing one?
- [ ] How does Traceroute actually work — what packets does it send to discover the path?
- [ ] What happens during BGP hijacking in detail — how does an attacker advertise fake routes?
- [ ] With cloud providers now reaching 76%+ of the Internet outside tier-1 transit, does this change how a tier-1 BGP hijack or outage would actually impact end users today?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Message**|Application-layer data unit exchanged between end systems|
|**Packet**|Chunk of message data with header bytes added by the source|
|**Router / Link-layer switch**|The two predominant types of packet switches in a network|
|**Store-and-forward**|Router must receive entire packet before forwarding|
|**Output buffer / queue**|Storage at router for packets waiting to be transmitted|
|**Queuing delay**|Waiting time in output buffer due to link congestion|
|**Packet loss**|Dropped packet when output buffer is completely full|
|**Forwarding table**|Router's map of destination addresses to outbound links|
|**Routing protocol**|Automated protocol that sets forwarding tables|
|**Circuit switching**|Reserves resources end-to-end for duration of session|
|**Packet switching**|Uses resources on demand, no reservation|
|**Circuit**|A dedicated end-to-end connection in circuit-switched network|
|**FDM**|Frequency-Division Multiplexing — divide frequency spectrum per circuit|
|**TDM**|Time-Division Multiplexing — divide time into slots per circuit|
|**Bandwidth**|Width of frequency band allocated to a circuit in FDM|
|**Silent period**|Idle time in circuit where reserved resources go unused|
|**Access ISP**|ISP providing end-user Internet access (DSL, cable, etc.)|
|**Tier-1 ISP**|Top-level ISP with global coverage; pays nobody for transit|
|**Regional ISP**|Mid-tier ISP connecting access ISPs to tier-1 ISPs|
|**PoP**|Point of Presence — provider's connection point for customer ISPs|
|**Multi-homing**|ISP connecting to two or more provider ISPs for redundancy|
|**Peering**|Two ISPs at same level directly connecting, settlement-free|
|**IXP**|Internet Exchange Point — facility where multiple ISPs peer (600+ today)|
|**Content provider network**|Private network built by a content company (e.g., Google)|

---

## Summary — Packet Switching vs Circuit Switching

||Packet Switching|Circuit Switching|
|---|---|---|
|**Resource use**|On demand|Reserved|
|**Delay**|Variable (queuing)|Fixed|
|**Efficiency**|High|Low (idle waste)|
|**Users supported**|Many more|Limited|
|**Complexity**|Low|High (signaling)|
|**Best for**|Bursty data|Real-time voice|
|**Example**|Internet|Traditional PSTN|

---

## Related Concepts

---

## Sources Cited (New Edition)

- [Tian 2012] — China's multi-tier ISP hierarchy (provincial → national → tier-1)
- [Van der Berg 2008] — Peering and customer-provider relationships
- [Ager 2012] — IXP facility structure
- [PeeringDB 2025] — Current IXP count (600+)
- [Google datacenters 2024] — Google data center count/distribution
- [Google Cloud 2025] — Google's worldwide cloud network map (Figure 1.15)
- [Labovitz 2010] — Content providers bypassing upper-tier ISPs
- [Arnold 2020] — Cloud providers reaching 76% of the Internet without tier-1 transit
- [Molinero-Fernandez 2002] — Packet vs. circuit switching discussion

---

→ Next: [[DELAY, LOSS, AND THROUGHPUT IN PACKET-SWITCHED NETWORKS]]