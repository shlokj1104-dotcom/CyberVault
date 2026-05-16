---
title: DELAY, LOSS, AND THROUGHPUT IN PACKET-SWITCHED NETWORKS
date: 2026-05-16
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 1.4 Delay, Loss, and Throughput in Packet-Switched Networks

> **One-Line Summary:** Every packet suffers four types of nodal delay
> (processing, queuing, transmission, propagation) at each router along
> its path; queuing delay is the most variable and interesting; throughput
> is constrained by the bottleneck link — the slowest link on the path.

---

## Core Idea

Ideally, the Internet would move data between any two end systems
**instantaneously** and **without loss**. In reality:

- **Throughput** is constrained — only so many bits/second can flow.
- **Delays** are introduced at every node along the path.
- **Packets can be lost** when buffers overflow.

These are not design failures — they are consequences of physical laws
and finite resources. Understanding them is essential for designing
applications and networks that work well despite these limitations.

---

# 1.4.1 Overview of Delay in Packet-Switched Networks

A packet starts at a source host, passes through a series of routers,
and arrives at a destination host. At **each node** (host or router)
along the path, the packet suffers from multiple types of delay.

![[Pasted image 20260516121026.png]]
*(Figure 1.16 — The nodal delay at router A: processing → queuing →
transmission → propagation to router B)*

The four delay components together form the **total nodal delay**:

$$d_{\text{nodal}} = d_{\text{proc}} + d_{\text{queue}} + d_{\text{trans}} + d_{\text{prop}}$$

---

## The Four Types of Delay

### 1. Processing Delay (d_proc)

**What it is:**
The time required for a router to:
- Examine the packet's header.
- Determine which outbound link to direct the packet to.
- Check for bit-level errors that occurred during transmission from the
  upstream node.

**Magnitude:**
- Typically on the order of **microseconds or less** in high-speed routers.
- After processing, the router directs the packet to the queue preceding
  the outbound link.

**Key point:** Processing delay is generally the smallest of the four
delays and is often considered negligible for analysis purposes — but it
does influence the **maximum throughput** of a router (how many packets
per second it can forward).

---

### 2. Queuing Delay (d_queue)

**What it is:**
The time a packet waits in the output buffer (queue) before it can be
transmitted onto the outbound link.

**When it occurs:**
- If the outbound link is currently **busy** transmitting another packet,
  OR
- If there are **other packets ahead** in the queue,
- → the newly arriving packet joins the queue and waits.

**Characteristics:**
- **Most variable** of the four delays — differs from packet to packet.
- Depends on the **level of congestion** in the network.
- If queue is empty and link is idle → **zero** queuing delay.
- If traffic is heavy and many packets are queued → **large** queuing delay.
- Range in practice: **microseconds to milliseconds**.

**Key point:** This is why networking researchers and engineers spend
enormous effort studying queuing delay — thousands of papers and many
books are dedicated to it.

---

### 3. Transmission Delay (d_trans)

**What it is:**
The time required to push (transmit) **all bits** of the packet onto the
outbound link.

**Formula:**

$$d_{\text{trans}} = \frac{L}{R}$$

Where:
- **L** = length of packet in bits
- **R** = transmission rate of the link in bits/sec

**Examples:**
- 10 Mbps Ethernet: R = 10 × 10⁶ bps
- 100 Mbps Ethernet: R = 100 × 10⁶ bps

**Magnitude:**
- Typically **microseconds to milliseconds** in practice.
- Negligible for LANs (10+ Mbps links).
- Significant for low-speed links (e.g., dial-up at 56 kbps).

**Key point:** Transmission delay is about the **packet size and link
speed** — not about distance. A large packet on a slow link has high
transmission delay even if the destination is nearby.

---

### 4. Propagation Delay (d_prop)

**What it is:**
The time required for a bit to propagate (travel physically) from the
**beginning of the link** to the next router.

**Formula:**

$$d_{\text{prop}} = \frac{d}{s}$$

Where:
- **d** = distance between the two routers (meters)
- **s** = propagation speed of the link (meters/sec)

**Propagation speed depends on the physical medium:**

$$2 \times 10^8 \text{ m/s} \leq s \leq 3 \times 10^8 \text{ m/s}$$

This is approximately the speed of light (3 × 10⁸ m/s).

**Magnitude:**
- **Milliseconds** for wide-area networks (e.g., cross-country links).
- **Negligible** (microseconds) for links connecting two routers on the
  same campus (say, 200 meters apart).
- **280 ms** for a geostationary satellite link (36,000 km altitude).

**Key point:** Propagation delay depends only on **distance** — not on
packet size or link speed. A 1-bit packet and a 1 GB file take the
same propagation time on the same link.

---

## Transmission Delay vs Propagation Delay — The Critical Distinction

This is one of the most commonly confused pairs in networking.

| Property | Transmission Delay (d_trans) | Propagation Delay (d_prop) |
|---|---|---|
| **What it measures** | Time to push ALL bits onto the link | Time for ONE bit to travel across the link |
| **Depends on** | Packet size (L) and link rate (R) | Distance (d) and propagation speed (s) |
| **Formula** | L/R | d/s |
| **Nothing to do with** | Distance between routers | Packet length or transmission rate |

> 💡 **Tollbooth Caravan Analogy (Figure 1.17):**
>
> Imagine a highway with tollbooths every 100 km. A caravan of 10 cars
> travels together. Cars propagate at 100 km/hour. Tollbooth services
> cars at 1 car per 12 seconds.
>

| Network Concept    | Highway Analogy                                                           |
| ------------------ | ------------------------------------------------------------------------- |
| Bit                | Car                                                                       |
| Packet             | Caravan (10 cars)                                                         |
| Router             | Tollbooth                                                                 |
| Transmission delay | Time for tollbooth to service all 10 cars: (10 cars)/(5 cars/min) = 2 min |
| Propagation delay  | Time for one car to travel 100 km: 100/100 = 1 hour                       |
| Total              | 62 minutes (2 min transmission + 60 min propagation)                      |

> The tollbooth must finish servicing the whole caravan before it can
> begin — just like store-and-forward.

![[Pasted image 20260516121209.png]]
*(Figure 1.17 — Caravan Analogy: 10 cars at tollbooth illustrating
transmission delay vs propagation delay)*

**Important edge case — What if propagation < transmission?**
If the caravan travels at 1,000 km/hour instead of 100 km/hour:
- Propagation between tollbooths: 100/1000 = **6 minutes**
- Transmission at tollbooth: **10 minutes**
- The first few cars arrive at the NEXT tollbooth before the last cars
  have left the first tollbooth.
- This is exactly what happens in packet-switched networks when propagation
  speed is fast relative to transmission time — bits arrive at the next
  router before all bits have been sent by the source.

---

## Total Nodal Delay — Summary

$$d_{\text{nodal}} = d_{\text{proc}} + d_{\text{queue}} + d_{\text{trans}} + d_{\text{prop}}$$

Relative magnitudes depending on context:

| Delay | Typical Magnitude | Dominant when? |
|---|---|---|
| d_proc | Microseconds or less | Router is slow / overloaded |
| d_queue | Microseconds – milliseconds | Network is congested |
| d_trans | Microseconds – milliseconds | Low-speed link OR large packet |
| d_prop | Microseconds – milliseconds | Long-distance link (WAN, satellite) |

---

# 1.4.2 Queuing Delay and Packet Loss

## Why Queuing Delay Is Special

Unlike d_proc, d_trans, and d_prop — which are **fixed** for a given
packet on a given path — **queuing delay varies from packet to packet**.

- 10 packets arriving simultaneously at an empty queue:
  - **First packet:** zero queuing delay.
  - **Last packet:** queues behind all 9 others → queuing delay ≈ 9 × (L/R).

This variability means queuing delay is characterized statistically:
- Average queuing delay
- Variance of queuing delay
- Probability that queuing delay exceeds a threshold

---

## Traffic Intensity — The Key Metric

**Definitions:**
- **a** = average rate at which packets arrive at the queue (packets/sec)
- **R** = transmission rate (bits/sec)
- **L** = packet size (bits, assumed constant)
- **La** = average rate at which bits arrive at the queue (bits/sec)

The ratio **La/R** is called **traffic intensity**:

$$\text{Traffic Intensity} = \frac{La}{R}$$

![[Pasted image 20260516121335.png]]
*(Figure 1.18 — Dependence of average queuing delay on traffic intensity:
exponential increase as La/R approaches 1)*

### Three Regimes:

**Case 1: La/R > 1**
- Bits arrive faster than they can be transmitted.
- Queue grows without bound.
- Average queuing delay → **infinity**.
- ⚠️ **Golden Rule of Traffic Engineering:**
  > *Design your system so that the traffic intensity is no greater than 1.*

**Case 2: La/R ≤ 1, periodic arrivals**
- If packets arrive exactly every L/R seconds (perfectly spaced):
  - Every packet arrives at an empty queue → **zero** queuing delay.
- If packets arrive in **bursts** of N every (L/R)N seconds:
  - First packet: zero delay
  - Second packet: L/R delay
  - nth packet: (n-1) × L/R delay
  - **Significant average queuing delay** even though La/R ≤ 1.

**Case 3: La/R → 0 (light traffic)**
- Packets arrive rarely → almost always find empty queue → **near-zero** delay.

**Case 4: La/R → 1 (heavy traffic, realistic random arrivals)**
- Traffic intensity close to 1 → queue builds during bursts.
- Even small increases in traffic intensity cause **disproportionately large**
  increases in delay (exponential-like behavior near La/R = 1).
- This is why a highway that is "typically congested" becomes catastrophically
  worse with just slightly more traffic.

> **Real-world intuition:** A highway with La/R close to 1 is normally
> congested. One small incident (an accident) causes traffic to back up
> for miles. The same incident on a highway with La/R = 0.5 would barely
> be noticed.

---

## Packet Loss

The analysis above assumed **infinite queue capacity**. In reality:
- Every router has a **finite buffer** (finite output queue capacity).
- When the buffer is **completely full** and a new packet arrives:
  - The router has **no place** to store the packet.
  - The router **drops** the packet → the packet is **lost**.

**What packet loss looks like from an end-system perspective:**
- A packet is transmitted into the network core but **never emerges** at
  the destination.
- From the application's perspective, the packet simply vanished.

**Implication:**
- Performance at a node is measured not only in terms of **delay** but also
  in terms of the **probability of packet loss**.
- As traffic intensity increases, both delay AND packet loss probability
  increase.
- Lost packets may need to be **retransmitted** (handled by TCP at the
  transport layer).

---

# 1.4.3 End-to-End Delay

## From Nodal to End-to-End

So far we've focused on **nodal delay** — delay at a single router.

Now we consider the **total delay from source host to destination host**
across a path with N-1 routers (N links).

**Assumptions for the formula:**
- Network is **uncongested** → queuing delays negligible.
- Processing delay at each router = d_proc (same everywhere).
- Transmission rate out of each router = R bits/sec (same everywhere).
- Propagation on each link = d_prop (same everywhere).

$$d_{\text{end-end}} = N(d_{\text{proc}} + d_{\text{trans}} + d_{\text{prop}})$$

Where d_trans = L/R.

> **Note:** This is a generalization of Equation 1.1 (which only counted
> transmission delay). Equation 1.2 adds processing and propagation delays.
> In reality, delays at each node may differ — the formula can be generalized
> to heterogeneous delays.

---

## Traceroute — Measuring End-to-End Delay in Practice

**What is Traceroute?**
A program that runs on any Internet host. It:
1. Sends N special packets toward the destination.
2. When router n receives the nth packet, it sends a short reply message
   back to the source containing the **router's name and address**.
3. When the destination receives the Nth packet, it also returns a message.
4. The source records the **round-trip time (RTT)** for each packet.

**Result:** Traceroute reconstructs the entire path and measures RTT
to each router along the way.

```
Sample Traceroute Output:

Source: cs-gw (128.119.240.254)
Destination: cis.poly.edu (128.238.32.126)

Hop 1 - cs-gw (128.119.240.254)
RTT: 1.009 ms / 0.899 ms / 0.993 ms
Avg: 0.967 ms

Hop 2 - 128.119.3.154
RTT: 0.931 ms / 0.441 ms / 0.651 ms
Avg: 0.674 ms

Hop 3 - border4-rt-gi-1-3.gw.umass.edu
RTT: 1.032 ms / 0.484 ms / 0.451 ms
Avg: 0.656 ms

Hop 4 - acr1-ge-2-1-0.Boston.cw.net
RTT: 10.006 ms / 8.150 ms / 8.460 ms
Avg: 8.872 ms

Hop 5 - agr4-loopback.NewYork.cw.net
RTT: 12.272 ms / 14.344 ms / 13.267 ms
Avg: 13.294 ms

Hop 6 - acr2-loopback.NewYork.cw.net
RTT: 13.225 ms / 12.292 ms / 12.148 ms
Avg: 12.555 ms

Hop 7 - pos10-2.core2.NewYork.Level3
RTT: 12.218 ms / 11.823 ms / 11.793 ms
Avg: 11.945 ms

Hop 8 - gige9-1-52.hsipaccess1.NewYork
RTT: 13.081 ms / 11.556 ms / 13.297 ms
Avg: 12.645 ms

Hop 9 - p0-0.polyju.bbnplanet.net
RTT: 12.169 ms / 13.052 ms / 12.786 ms
Avg: 12.669 ms

Hop 10 - cis.poly.edu (128.238.32.126)
RTT: 14.080 ms / 13.035 ms / 12.802 ms
Avg: 13.306 ms
```

**Reading the output:**

| Column | Meaning |
|---|---|
| 1st | Router number (n) along the route |
| 2nd | Router hostname |
| 3rd | Router IP address |
| 4th–6th | RTT for 3 repeated experiments (ms) |

**Key observation from the trace:**
- Router 6 RTTs are **larger** than Router 7 RTTs.
- This is possible because queuing delays vary with time — the 3 experiments
  are not sent simultaneously, so the network state differs.
- Traceroute repeats each measurement 3 times for this reason.

**Try it yourself:**
- Visit www.traceroute.org — web interface to trace from many countries.
- Also available as a command-line tool on Linux/macOS (`traceroute`) and
  Windows (`tracert`).

---

## End-System and Other Delays

Beyond the four nodal delays, end systems can introduce additional delays:

| Additional Delay | Cause |
|---|---|
| **Media access delay** | End system sharing a medium (WiFi, cable modem) must wait for its turn to transmit — part of media access protocols (Chapter 5) |
| **Packetization delay** | VoIP applications must fill a packet with encoded digitized speech before sending — the time to fill this packet is the packetization delay and directly impacts perceived call quality |

> **VoIP note:** A VoIP sender can't send a packet until it has enough
> encoded speech to fill it. This packetization delay is a meaningful
> fraction of the end-to-end delay for real-time voice and is a homework
> problem at the end of Chapter 1.

---

# 1.4.4 Throughput in Computer Networks

## Definitions

**Instantaneous throughput:**
The rate (in bits/sec) at which Host B is receiving the file **at any
instant of time**.

**Average throughput:**
If a file of **F bits** is transferred in **T seconds**:

$$\text{Average throughput} = \frac{F}{T} \text{ bits/sec}$$

---

## Bottleneck Link — The Key Concept

![[Pasted image 20260516121513.png]]
*(Figure 1.19 — Throughput for a file transfer: (a) two-link network;
(b) N-link network — throughput = rate of bottleneck link)*

**Simple two-link network:**
- Server → Router: link rate R_s (bits/sec)
- Router → Client: link rate R_c (bits/sec)

**What is the throughput?**

Think of bits as **fluid** and communication links as **pipes**:
- Server pumps bits into its link at rate R_s.
- Router forwards bits to client at rate R_c.

**Two cases:**
1. **R_s < R_c:** Bits flow through router and arrive at client at rate R_s.
   Throughput = R_s.
2. **R_c < R_s:** Bits pile up at router (backlog grows — undesirable!).
   Client only receives at rate R_c.
   Throughput = R_c.

**In both cases:**

$$\text{Throughput} = \min(R_s, R_c)$$

The constraining link is the **bottleneck link**.

**File transfer time approximation:**

$$T \approx \frac{F}{\min(R_s, R_c)}$$

**Worked example:**
- F = 32 million bits (MP3 file)
- R_s = 2 Mbps
- R_c = 1 Mbps
- Bottleneck = R_c = 1 Mbps
- Transfer time = 32 × 10⁶ / 1 × 10⁶ = **32 seconds**

**General N-link network:**

$$\text{Throughput} = \min(R_1, R_2, \ldots, R_N)$$

Always the **minimum transmission rate** along the entire path.

---

## Where Is the Bottleneck in Today's Internet?

![[Pasted image 20260516121657.png]]
*(Figure 1.20 — End-to-end throughput: (a) single server-client pair;
(b) 10 servers and 10 clients sharing a common core link)*

**Today's Internet is over-provisioned in the core:**
- The Internet core has **very high transmission rates**.
- Core links are much faster than R_s and R_c.
- Therefore, the constraining factor (bottleneck) is **almost always the
  access network** — your home DSL, cable, or cellular link.

For a single server-client pair:

$$\text{Throughput} = \min(R_s, R_c)$$

The core links don't matter — R_s and R_c dominate.

---

## Shared Bottleneck — Multiple Flows

**New scenario (Figure 1.20b):**
- 10 servers, 10 clients.
- All 10 server-client pairs share a **common link of rate R** in the core.

**Case 1: R is large (100× larger than R_s and R_c)**
- Core link is not the bottleneck.
- Throughput for each download = min(R_s, R_c).

**Case 2: R_s = 2 Mbps, R_c = 1 Mbps, R = 5 Mbps shared equally**
- Core link rate per download = 5 Mbps / 10 = **500 kbps**.
- 500 kbps < R_c = 1 Mbps → core link IS the bottleneck.
- Throughput for each download = **500 kbps** (not R_c = 1 Mbps).

**Key insight:**
> Throughput depends not only on the transmission rates of links along
> the path but also on **intervening traffic** sharing those links. A
> link with a high transmission rate may nonetheless be the bottleneck
> if many other data flows are also passing through it.

---

## Summary — Throughput Rules

| Scenario | Throughput |
|---|---|
| 2-link, no competing traffic | min(R_s, R_c) |
| N-link, no competing traffic | min(R_1, R_2, ..., R_N) |
| N-link, competing traffic on link i | Each flow gets R_i / (number of flows on link i) |
| Today's Internet (core over-provisioned) | min(R_s, R_c) — access link is almost always bottleneck |

---

# Why It Matters for Security

| Concept | Attacker's Perspective | Defender's Perspective |
|---|---|---|
| **Queuing delay** | Flood a link → La/R > 1 → queue explodes → DoS for all users | Rate limiting, traffic policing — keep La/R well below 1 |
| **Packet loss** | Cause packet loss deliberately → force TCP retransmissions → slow down victim | Buffer management, ECN (Explicit Congestion Notification) |
| **Traffic intensity** | Sustained attack traffic near La/R = 1 → collapse entire ISP segment with small increase | Over-provision critical links; deploy DDoS scrubbing |
| **Traceroute** | Attackers use Traceroute to **map the network** — learn router names, IPs, topology | Firewalls can block ICMP TTL-exceeded messages to hide internal topology |
| **Bottleneck link** | Attacker identifies bottleneck link (e.g., ISP's access link) and saturates it | Bandwidth management, burst protection at bottleneck |
| **Propagation delay** | Exploit known propagation delay in timing attacks (e.g., locating anonymous users by RTT) | VPN, Tor — add unpredictable hop latency |

---

## Questions I Still Have

- [ ] How does TCP react when packets are lost — does it retransmit
      immediately or wait?
- [ ] In the queuing delay formula: if packets arrive randomly (Poisson
      process), what is the exact average queuing delay formula?
- [ ] Why does Traceroute sometimes show asterisks (*) instead of router
      names — is the router not responding or blocking ICMP?
- [ ] In the 10-server 10-client shared bottleneck example — how does each
      flow "know" to share the link equally? What protocol enforces this?
- [ ] How does ECN (Explicit Congestion Notification) work to prevent
      packet loss before buffers overflow?

---

## Key Terms — Quick Reference

| Term | Definition |
|---|---|
| **Nodal delay** | Total delay at a single node: d_proc + d_queue + d_trans + d_prop |
| **Processing delay** | Time to examine header and determine outbound link |
| **Queuing delay** | Time waiting in output buffer before transmission |
| **Transmission delay** | Time to push all L bits onto link: L/R seconds |
| **Propagation delay** | Time for bit to travel across link: d/s seconds |
| **Traffic intensity** | La/R — ratio of bit arrival rate to link rate |
| **Packet loss** | Dropped packet when output buffer is completely full |
| **End-to-end delay** | Total accumulated delay from source to destination |
| **Traceroute** | Program that measures RTT to each router along the path |
| **RTT** | Round-Trip Time — time for packet to travel source→router→source |
| **Throughput** | Rate (bits/sec) at which destination receives data |
| **Instantaneous throughput** | Throughput at a specific instant in time |
| **Average throughput** | F/T — total bits transferred divided by total time |
| **Bottleneck link** | Link with lowest transmission rate along the path |
| **Packetization delay** | VoIP: time to fill a packet with encoded speech |

---

## Formula Sheet — 1.4 At a Glance

| Formula | Meaning |
|---|---|
| d_trans = L/R | Transmission delay: packet size / link rate |
| d_prop = d/s | Propagation delay: distance / propagation speed |
| d_nodal = d_proc + d_queue + d_trans + d_prop | Total delay at one node |
| d_end-end = N(d_proc + d_trans + d_prop) | End-to-end delay (N links, no queuing) |
| Traffic intensity = La/R | Ratio of arrival rate to service rate |
| Throughput = min(R_1, R_2, ..., R_N) | Bottleneck link determines throughput |
| T ≈ F / min(R_s, R_c) | File transfer time |

---

## Related Concepts

- 
---

→ Next: [[1.5 - Protocol Layers and Their Service Models]]