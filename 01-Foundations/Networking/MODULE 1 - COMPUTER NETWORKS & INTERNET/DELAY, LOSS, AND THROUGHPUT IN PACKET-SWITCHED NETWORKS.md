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

> **One-Line Summary:** Every packet suffers four types of nodal delay (processing, queuing, transmission, propagation) at each router along its path; queuing delay is the most variable and interesting; throughput is constrained by the bottleneck link — the slowest link on the path.

---

## Core Idea

Back in Section 1.1 we said the Internet can be viewed as an infrastructure that provides services to distributed applications running on end systems. Ideally, the Internet would move data between any two end systems **instantaneously** and **without loss**. In reality, this is a lofty, unachievable goal:

- **Throughput** is constrained — only so many bits/second can flow.
- **Delays** are introduced at every node along the path.
- **Packets can be lost** when buffers overflow.

These are not design failures — they are consequences of physical laws and finite resources. The flip side: because computer networks have these problems, there are endless fascinating issues around how to deal with them — more than enough to fill a course on computer networking, and to motivate thousands of PhD theses. Understanding delay, loss, and throughput is essential for designing applications and networks that work well despite these limitations.

---

# 1.4.1 Overview of Delay in Packet-Switched Networks

A packet starts at a source host, passes through a series of routers, and arrives at a destination host. At **each node** (host or router) along the path, the packet suffers from multiple types of delay. The performance of many Internet applications — search, Web browsing, email, maps, instant messaging, voice-over-IP — is greatly affected by these network delays.

![[Pasted image 20260619134241.png]] _(Figure 1.17 — The nodal delay at router A: processing → queuing → transmission → propagation to router B)_

The four delay components together form the **total nodal delay**:

$$d_{\text{nodal}} = d_{\text{proc}} + d_{\text{queue}} + d_{\text{trans}} + d_{\text{prop}}$$

---

## The Four Types of Delay

### 1. Processing Delay (d_proc)

**What it is:** The time required for a router to:

- Examine the packet's header.
- Determine which outbound link to direct the packet to.
- Check for bit-level errors that occurred during transmission from the upstream node.

**Magnitude:**

- Typically on the order of **microseconds or less** in high-speed routers.
- After processing, the router directs the packet to the queue preceding the outbound link.

**Key point:** Processing delay is generally the smallest of the four delays and is often considered negligible for analysis purposes — but it does influence the **maximum throughput** of a router (how many packets per second it can forward). The details of how a router actually operates internally are covered in Chapter 4.

---

### 2. Queuing Delay (d_queue)

**What it is:** The time a packet waits in the output buffer (queue) before it can be transmitted onto the outbound link.

**When it occurs:**

- If the outbound link is currently **busy** transmitting another packet, OR
- If there are **other packets ahead** in the queue,
- → the newly arriving packet joins the queue and waits.

**Characteristics:**

- **Most variable** of the four delays — differs from packet to packet.
- Depends on the **level of congestion** in the network.
- If queue is empty and link is idle → **zero** queuing delay.
- If traffic is heavy and many packets are queued → **large** queuing delay.
- Range in practice: **microseconds to milliseconds**.

**Key point:** This is why networking researchers and engineers spend enormous effort studying queuing delay — thousands of papers and many books are dedicated to it.

---

### 3. Transmission Delay (d_trans)

**What it is:** The time required to push (transmit) **all bits** of the packet onto the outbound link.

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

**Key point:** Transmission delay is about the **packet size and link speed** — not about distance. A large packet on a slow link has high transmission delay even if the destination is nearby.

---

### 4. Propagation Delay (d_prop)

**What it is:** The time required for a bit to propagate (travel physically) from the **beginning of the link** to the next router.

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
- **Negligible** (microseconds) for links connecting two routers on the same campus (say, 200 meters apart).
- **280 ms** for a geostationary satellite link (36,000 km altitude).

**Key point:** Propagation delay depends only on **distance** — not on packet size or link speed. A 1-bit packet and a 1 GB file take the same propagation time on the same link.

---

## Transmission Delay vs Propagation Delay — The Critical Distinction

This is one of the most commonly confused pairs in networking.

|Property|Transmission Delay (d_trans)|Propagation Delay (d_prop)|
|---|---|---|
|**What it measures**|Time to push ALL bits onto the link|Time for ONE bit to travel across the link|
|**Depends on**|Packet size (L) and link rate (R)|Distance (d) and propagation speed (s)|
|**Formula**|L/R|d/s|
|**Nothing to do with**|Distance between routers|Packet length or transmission rate|

> 💡 **Tollbooth Caravan Analogy (Figure 1.18):**
> 
> Imagine a highway with tollbooths every 100 km. A caravan of 10 cars travels together. Cars propagate at 100 km/hour. Tollbooth services cars at 1 car per 12 seconds.

|Network Concept|Highway Analogy|
|---|---|
|Bit|Car|
|Packet|Caravan (10 cars)|
|Router|Tollbooth|
|Transmission delay|Time for tollbooth to service all 10 cars: (10 cars)/(5 cars/min) = 2 min|
|Propagation delay|Time for one car to travel 100 km: 100/100 = 1 hour|
|Total|62 minutes (2 min transmission + 60 min propagation)|

> The tollbooth must finish servicing the whole caravan before it can begin — just like store-and-forward.

![[Pasted image 20260619134452.png]] _(Figure 1.18 — Caravan Analogy: 10 cars at tollbooth illustrating transmission delay vs propagation delay)_

**Important edge case — What if propagation < transmission?** If the caravan travels at 1,000 km/hour instead of 100 km/hour:

- Propagation between tollbooths: 100/1000 = **6 minutes**
- Transmission at tollbooth: **10 minutes**
- The first few cars arrive at the NEXT tollbooth before the last cars have left the first tollbooth.
- This is exactly what happens in packet-switched networks when propagation speed is fast relative to transmission time — bits arrive at the next router before all bits have been sent by the source.

> 🎬 The textbook website has an interactive animation that nicely illustrates and contrasts transmission delay and propagation delay — worth visiting [Smith 2009].

---

## Total Nodal Delay — Summary

$$d_{\text{nodal}} = d_{\text{proc}} + d_{\text{queue}} + d_{\text{trans}} + d_{\text{prop}}$$

Relative magnitudes depending on context:

|Delay|Typical Magnitude|Dominant when?|
|---|---|---|
|d_proc|Microseconds or less|Router is slow / overloaded|
|d_queue|Microseconds – milliseconds|Network is congested|
|d_trans|Microseconds – milliseconds|Low-speed link OR large packet|
|d_prop|Microseconds – milliseconds (up to hundreds of ms for satellite links)|Long-distance link (WAN, satellite) — can become the **dominant** term|

---

# 1.4.2 Queuing Delay and Packet Loss

## Why Queuing Delay Is Special

Unlike d_proc, d_trans, and d_prop — which are **fixed** for a given packet on a given path — **queuing delay varies from packet to packet**.

- 10 packets arriving simultaneously at an empty queue:
    - **First packet:** zero queuing delay.
    - **Last packet:** queues behind all 9 others → queuing delay ≈ 9 × (L/R).
    - In general, the nth packet transmitted has a queuing delay of (n − 1)L/R seconds.

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

![[Pasted image 20260619134604.png]] _(Figure 1.19 — Dependence of average queuing delay on traffic intensity: exponential increase as La/R approaches 1)_

The two examples of periodic arrivals (below) are a bit too academic — in practice, the arrival process to a queue is **random**: arrivals don't follow any pattern, and packets are spaced apart by random amounts of time. In this more realistic case, La/R alone isn't enough to fully characterize the queuing delay statistics, but it's still useful for building intuition.

### Three Regimes:

**Case 1: La/R > 1**

- Bits arrive faster than they can be transmitted.
- Queue grows without bound.
- Average queuing delay → **infinity**.
- ⚠️ **Golden Rule of Traffic Engineering:**
    
    > _Design your system so that the traffic intensity is no greater than 1._
    

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
- A small percentage increase in intensity causes a **much larger percentage-wise increase** in delay (exponential-like behavior near La/R = 1).
- This is why a highway that is "typically congested" becomes catastrophically worse with just slightly more traffic.

> **Real-world intuition:** A highway with La/R close to 1 is normally congested. One small incident (an accident, or just slightly more traffic than usual) causes traffic to back up for miles. The same incident on a highway with La/R = 0.5 would barely be noticed.

> 🎬 The textbook website also has an interactive animation for a queue — set the packet arrival rate high enough that traffic intensity exceeds 1, and you can watch the queue slowly build up over time.

---

## Packet Loss

The analysis above assumed **infinite queue capacity**. In reality:

- Every router has a **finite buffer** (finite output queue capacity) — the exact queuing capacity depends heavily on router design and cost.
- Because capacity is finite, packet delays don't actually approach infinity as traffic intensity approaches 1 — instead, a packet can arrive to find a **full queue**.
- With no place to store the packet, the router **drops** it — the packet is **lost**. (This overflow behavior can also be seen in the interactive queue animation once traffic intensity exceeds 1.)

**What packet loss looks like from an end-system perspective:**

- A packet is transmitted into the network core but **never emerges** at the destination.
- From the application's perspective, the packet simply vanished.

**Implication:**

- Performance at a node is measured not only in terms of **delay** but also in terms of the **probability of packet loss**.
- As traffic intensity increases, both delay AND packet loss probability increase.
- Lost packets may need to be **retransmitted** on an end-to-end basis (handled by TCP at the transport layer) to ensure all data eventually gets through.

---

# 1.4.3 End-to-End Delay

## From Nodal to End-to-End

So far we've focused on **nodal delay** — delay at a single router.

Now we consider the **total delay from source host to destination host** across a path with N-1 routers (N links).

**Assumptions for the formula:**

- Network is **uncongested** → queuing delays negligible.
- Processing delay at each router = d_proc (same everywhere).
- Transmission rate out of each router = R bits/sec (same everywhere).
- Propagation on each link = d_prop (same everywhere).

$$d_{\text{end-end}} = N(d_{\text{proc}} + d_{\text{trans}} + d_{\text{prop}})$$

Where d_trans = L/R.

> **Note:** This is a generalization of Equation 1.1 (which only counted transmission delay). Equation 1.2 adds processing and propagation delays. In reality, delays at each node may differ — the formula can be generalized to heterogeneous delays and to the presence of an average queuing delay too.

---

## Traceroute — Measuring End-to-End Delay in Practice

**What is Traceroute?** A simple program that runs on any Internet host (described in detail in RFC 1393). When the user specifies a destination hostname, the source sends a **series of special packets** toward the destination, each marked with a sequence number — the first packet marked 1, the second marked 2, and so on (this marking is the IP **TTL — Time To Live** field).

How the marking is actually used along the path:

1. Each router that forwards one of these packets **decrements the marking by one** before passing it along.
2. When a router receives a packet whose marking has been **decremented to zero**, that router does **not** forward the packet toward its destination — instead, it sends a short message back to the source.
3. When the destination host itself receives a packet marked 1, it also returns a message back to the source. (After receiving this message, the source stops sending further special packets.)
4. The source records the time elapsed between sending each packet and receiving its corresponding return message, plus the name and address of whichever router (or the destination) sent the message.

This is exactly how Traceroute reconstructs the route and measures the **round-trip time (RTT)** to every router along the way. Traceroute actually repeats the whole experiment **3 times**, sending 3·N packets in total, where N is the number of routers between source and destination.

```
Sample Traceroute Output
(source: gaia.cs.umass.edu, UMass CS dept — destination: a host in the
computer science department at the University of Sorbonne, Paris,
formerly known as UPMC)

1  gw-vlan-2451.cs.umass.edu (128.119.245.1)        1.899 ms  3.266 ms  3.280 ms
2  j-cs-gw-int-10-240.cs.umass.edu (10.119.240.254) 1.296 ms  1.276 ms  1.245 ms
3  n5-rt-1-1-ne-2-1-0.gw.umass.edu (128.119.3.33)   2.237 ms  2.217 ms  2.187 ms
4  core1-rt-et-5-2-0.gw.umass.edu (128.119.0.9)     0.351 ms  0.392 ms  0.380 ms
5  border1-rt-et-5-0-0.gw.umass.edu (192.80.83.102) 0.345 ms  0.345 ms  0.344 ms
6  nox300gw1-umass-eos.nox.org (192.5.89.101)       3.260 ms  0.416 ms  3.127 ms
7  nox300gw1-umass-eos.nox.org (192.5.89.101)       3.165 ms  7.326 ms  7.311 ms
8  198.71.45.237                                    7.826 ms  77.246 ms 77.744 ms
9  renater-lbl-gw1.mpls.par.fr.geant.net (62.40.124.70) 79.357 ms 77.729 ms 79.152 ms
10 193.51.180.109                                   78.379 ms 79.936 ms 80.042 ms
11 *  193.51.180.109                                 80.640 ms *
12 *  195.221.127.182                                78.408 ms *
13 195.221.127.182                                  80.696 ms 79.796 ms 78.434 ms
14 r-upmc1.reseau.jussieu.fr (134.157.254.10)        78.399 ms 80.840 ms  81.353 ms
```

**Reading the output:**

|Column|Meaning|
|---|---|
|1st|Router number (n) along the route|
|2nd|Router hostname|
|3rd|Router IP address|
|4th–6th|RTT for 3 repeated experiments (ms)|

There are 14 routers between source and destination in this trace. Most have a hostname, and all have an IP address — e.g., router 4's name is `core1-rt-et-5-2-0.gw.umass.edu` at `128.119.0.9`, with round-trip delays of 0.351, 0.392, and 0.380 ms across the three trials. These round-trip delays bundle together transmission, propagation, router processing, and queuing delay — all of it.

**Key observations from the trace:**

- Because queuing delay varies with time, the RTT to router _n_ can sometimes be _longer_ than the RTT to router _n+1_ — exactly what happens here, where the delay to router 11 is briefly larger than the delay to router 12.
- There's a **big jump** in round-trip delay going from router 7 to router 8 — this is due to a **transatlantic fiber-optic link** between the two, giving rise to a relatively large propagation delay.
- Asterisks (`*`) appear when a probe doesn't get a response within the timeout window for that trial.

**Try it yourself:**

- Visit www.traceroute.org — web interface to trace from many countries.
- Also available as a command-line tool on Linux/macOS (`traceroute`) and Windows (`tracert`).
- **PingPlotter** [PingPlotter 2025] is a popular free graphical front-end for Traceroute.
- **Whois** is another handy tool — it queries the distributed Whois database to look up registration information about IP addresses.

---

## End System, Application, and Other Delays

Beyond the four nodal delays, end systems can introduce additional delays:

|Additional Delay|Cause|
|---|---|
|**Media access delay**|End system sharing a medium (WiFi, cable modem) must purposefully delay its own transmission as part of the protocol for sharing the medium with other end systems — covered in detail in Chapter 6|
|**Packetization delay**|Video conferencing apps (e.g., Zoom, Google Hangouts) must fill a packet with encoded digitized video/speech before sending — the time to fill this packet is the packetization delay and directly impacts perceived call quality|

> **Note:** This packetization delay is a meaningful fraction of the end-to-end delay for real-time voice/video, and is explored further in a homework problem at the end of Chapter 1.

---

# 1.4.4 Throughput in Computer Networks

## Definitions

In addition to delay and packet loss, another critical performance measure is **end-to-end throughput** — for example, transferring a large video clip from one peer to another.

**Instantaneous throughput:** The rate (in bits/sec) at which Host B is receiving the file **at any instant of time**. (Many download UIs display this live — and you can try measuring your own end-to-end download throughput with a tool like **Speedtest** [Speedtest 2025].)

**Average throughput:** If a file of **F bits** is transferred in **T seconds**:

$$\text{Average throughput} = \frac{F}{T} \text{ bits/sec}$$

For some applications, a **low but consistent** throughput above a threshold matters more than a high peak rate — e.g., **over 24 kbps** for some Internet telephony applications, and **over 256 kbps** for some real-time video applications.

---

## Bottleneck Link — The Key Concept

![[Pasted image 20260619134650.png]] _(Figure 1.20 — Throughput for a file transfer: (a) two-link network; (b) N-link network — throughput = rate of bottleneck link)_

**Simple two-link network:**

- Server → Router: link rate R_s (bits/sec)
- Router → Client: link rate R_c (bits/sec)

**What is the throughput?**

Think of bits as **fluid** and communication links as **pipes**:

- Server pumps bits into its link at rate R_s.
- Router forwards bits to client at rate R_c.

**Two cases:**

1. **R_s < R_c:** Bits flow through router and arrive at client at rate R_s. Throughput = R_s.
2. **R_c < R_s:** Bits pile up at router (backlog grows — undesirable!). Client only receives at rate R_c. Throughput = R_c.

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

![[Pasted image 20260619134831.png]] _(Figure 1.21 — End-to-end throughput: (a) single server-client pair; (b) 10 servers and 10 clients sharing a common core link)_

**Today's Internet is over-provisioned in the core:**

- The Internet core has **very high transmission rates** and experiences little congestion.
- Core links are much faster than R_s and R_c.
- Therefore, the constraining factor (bottleneck) is **almost always the access network** — your home DSL, cable, or cellular link.

For a single server-client pair:

$$\text{Throughput} = \min(R_s, R_c)$$

The core links don't matter — R_s and R_c dominate.

---

## Shared Bottleneck — Multiple Flows

**New scenario (Figure 1.21b):**

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

> Throughput depends not only on the transmission rates of links along the path but also on **intervening traffic** sharing those links. A link with a high transmission rate may nonetheless be the bottleneck if many other data flows are also passing through it.

---

## Summary — Throughput Rules

|Scenario|Throughput|
|---|---|
|2-link, no competing traffic|min(R_s, R_c)|
|N-link, no competing traffic|min(R_1, R_2, ..., R_N)|
|N-link, competing traffic on link i|Each flow gets R_i / (number of flows on link i)|
|Today's Internet (core over-provisioned)|min(R_s, R_c) — access link is almost always bottleneck|

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Queuing delay**|Flood a link → La/R > 1 → queue explodes → DoS for all users|Rate limiting, traffic policing — keep La/R well below 1|
|**Packet loss**|Cause packet loss deliberately → force TCP retransmissions → slow down victim|Buffer management, ECN (Explicit Congestion Notification)|
|**Traffic intensity**|Sustained attack traffic near La/R = 1 → collapse entire ISP segment with small increase|Over-provision critical links; deploy DDoS scrubbing|
|**Traceroute (TTL-based)**|Attackers use Traceroute to **map the network** — learn router names, IPs, topology, and even spot a transatlantic/long-haul hop|Firewalls can block ICMP TTL-exceeded messages to hide internal topology|
|**Bottleneck link**|Attacker identifies bottleneck link (e.g., ISP's access link or a shared core link) and saturates it|Bandwidth management, burst protection at bottleneck|
|**Propagation delay**|Exploit known propagation delay in timing attacks (e.g., locating anonymous users by RTT)|VPN, Tor — add unpredictable hop latency|

---

## Questions I Still Have

- [ ] How does TCP react when packets are lost — does it retransmit immediately or wait?
- [ ] In the queuing delay formula: if packets arrive randomly (Poisson process), what is the exact average queuing delay formula?
- [ ] Why does Traceroute sometimes show asterisks (*) instead of router names — is the router not responding or blocking ICMP?
- [x] ~~How does Traceroute actually work — what packets does it send to discover the path?~~ **Answered:** it sends packets with an increasing TTL marking (1, 2, 3, ...); each router decrements the TTL by one, and the router where it hits zero replies to the source instead of forwarding — that's how each hop gets identified.
- [ ] In the 10-server 10-client shared bottleneck example — how does each flow "know" to share the link equally? What protocol enforces this?
- [ ] How does ECN (Explicit Congestion Notification) work to prevent packet loss before buffers overflow?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Nodal delay**|Total delay at a single node: d_proc + d_queue + d_trans + d_prop|
|**Processing delay**|Time to examine header and determine outbound link|
|**Queuing delay**|Time waiting in output buffer before transmission|
|**Transmission delay**|Time to push all L bits onto link: L/R seconds|
|**Propagation delay**|Time for bit to travel across link: d/s seconds|
|**Traffic intensity**|La/R — ratio of bit arrival rate to link rate|
|**Packet loss**|Dropped packet when output buffer is completely full|
|**End-to-end delay**|Total accumulated delay from source to destination|
|**Traceroute**|Program that measures RTT to each router along the path, using the IP TTL field|
|**TTL (Time To Live)**|IP header field decremented by each router; reaching zero triggers a reply instead of forwarding — the mechanism Traceroute exploits|
|**RTT**|Round-Trip Time — time for packet to travel source→router→source|
|**Throughput**|Rate (bits/sec) at which destination receives data|
|**Instantaneous throughput**|Throughput at a specific instant in time|
|**Average throughput**|F/T — total bits transferred divided by total time|
|**Bottleneck link**|Link with lowest transmission rate along the path|
|**Packetization delay**|Video/voice conferencing: time to fill a packet with encoded media|

---

## Formula Sheet — 1.4 At a Glance

|Formula|Meaning|
|---|---|
|d_trans = L/R|Transmission delay: packet size / link rate|
|d_prop = d/s|Propagation delay: distance / propagation speed|
|d_nodal = d_proc + d_queue + d_trans + d_prop|Total delay at one node|
|d_end-end = N(d_proc + d_trans + d_prop)|End-to-end delay (N links, no queuing)|
|Traffic intensity = La/R|Ratio of arrival rate to service rate|
|Throughput = min(R_1, R_2, ..., R_N)|Bottleneck link determines throughput|
|T ≈ F / min(R_s, R_c)|File transfer time|

---

## Related Concepts

---

## Sources Cited (New Edition)

- 

---

→ Next: [[PROTOCOL LAYERS AND THEIR SERVICE MODELS]]