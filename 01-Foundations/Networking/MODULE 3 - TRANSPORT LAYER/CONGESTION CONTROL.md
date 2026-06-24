---
title: CONGESTION CONTROL
date: 2026-06-24
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 3.6 Principles of Congestion Control

> **One-Line Summary:** Congestion control exists because retransmission treats only the _symptom_ of network congestion (a lost packet) but not the _cause_ (too many sources sending too fast); this section builds intuition for what congestion costs the network through three increasingly realistic scenarios, then identifies two fundamental approaches — end-to-end and network-assisted — for how senders can be throttled before the network collapses entirely.

---

## Core Idea: Retransmission Is a Bandage, Not a Cure

Section 3.5 showed how TCP recovers from packet loss — but it never asked _why_ those packets were lost in the first place. In practice, the overwhelming cause of packet loss is **router buffer overflow**: a router's outgoing link buffer fills up when arriving traffic exceeds the link's output capacity, and subsequent arriving packets are simply dropped.

TCP's retransmission mechanism responds to this by re-sending the dropped segment — but that response is, at best, treating the symptom. If many sources are all simultaneously sending at high rates, the _root cause_ — too much aggregate traffic for the network to carry — remains untouched. More retransmissions from those same sources may actually make the situation _worse_. To address the cause, there must be a mechanism that **throttles senders** when the network is congested.

> **Analogy — A Hospital That Only Treats Injuries, Never Prevents Them:** If a dangerous intersection keeps causing car accidents, patching up the injured drivers (retransmission) does nothing about the broken traffic light (congestion). You need to either fix the light or reduce the number of cars using the intersection. Congestion control is the traffic light fix.

This section studies congestion in a **general context** before Section 3.7 narrows to TCP's specific algorithm. The goals here are to understand:

- Why congestion is bad (costs to performance)
- How congestion manifests in what upper-layer applications actually experience
- What general approaches exist to avoid or react to congestion

---

## 3.6.1 The Causes and the Costs of Congestion

To build precise intuition, the textbook walks through three progressively realistic scenarios, each adding one more real-world complication. In all scenarios, the focus is on _what happens as hosts increase their transmission rate_ and the network becomes congested — not yet on _how to fix it_.

---

### Scenario 1: Two Senders, a Router with Infinite Buffers

![[Pasted image 20260624145910.png]]

**Setup:**

- Two hosts (A and B), each with a connection sharing a single router hop between source and destination (Figure 3.41).
- The shared outgoing link has capacity **R** bytes/sec.
- The router has **infinite buffer space** — it never drops packets, it just queues them.
- Both hosts send original data only: **no retransmission, no flow control, no congestion control**.
- Each connection's application sends at rate **λ_in** bytes/sec. Because there are no retransmissions, the rate offered to the router is also λ_in.

```
Host A ──┐
         ├──[Router: ∞ buffer, link capacity R]──► Destinations
Host B ──┘
         (two flows share one outgoing link of capacity R)
```

**What Figure 3.42 reveals:**

![[Pasted image 20260624150104.png]]

_Left graph — Throughput (λ_out) vs. Sending Rate (λ_in):_

|Sending Rate Range|Throughput Behavior|
|---|---|
|λ_in between 0 and R/2|Throughput = λ_in (everything sent is received, with finite delay)|
|λ_in approaches R/2|Throughput asymptotically approaches R/2 — the connection's fair share|
|λ_in > R/2|Throughput stays capped at R/2, regardless of how much more the sender pushes|

The cap at R/2 is simply the consequence of **sharing link capacity** between two connections. No matter how high either sender pushes λ_in, each connection can only ever receive R/2 — the link simply cannot deliver packets at a steady-state rate exceeding R/2 per connection.

_Right graph — Delay vs. Sending Rate (λ_in):_

|Sending Rate Range|Delay Behavior|
|---|---|
|λ_in well below R/2|Delay is finite and relatively low|
|λ_in approaches R/2 from the left|Average delay grows larger and larger without bound|
|λ_in exceeds R/2|Average delay becomes **infinite** (packets queue forever in the infinite buffer)|

**Why delay blows up:** As λ_in approaches R/2, the aggregate arrival rate approaches the full link capacity R. With an infinite buffer available, packets are never dropped — they just wait longer and longer. The queue grows unboundedly, and so does queuing delay.

> **Cost #1 Discovered:** _Large queuing delays are experienced as packet-arrival rate approaches link capacity._ Even in this idealized, loss-free scenario, the delay cost of congestion is already severe.

> **Analogy — A One-Lane Bridge at Rush Hour:** Two streams of cars (A and B) share one narrow bridge with capacity R cars/min. If both streams together send more than R cars/min, the bridge can still forward cars — no car is "lost" (infinite queue on the road), but waiting time grows without limit. Each stream, no matter how many cars it queues, gets at most R/2 cars/min across. Sending more cars doesn't help throughput; it only lengthens the queue.

---

### Scenario 2: Two Senders and a Router with Finite Buffers

![[Pasted image 20260624150325.png]]

**The Two Key Changes from Scenario 1:**

1. **Finite router buffers** — packets _are_ dropped when the buffer is full.
2. **Reliable connections** — since packets can be dropped, senders _retransmit_ lost segments.

**New Terminology:** Because retransmissions exist, we now need to distinguish between two different rates:

|Symbol|Meaning|
|---|---|
|**λ_in**|Rate at which the application sends _original_ data into the socket (bytes/sec)|
|**λ'_in**|Rate at which the transport layer sends _both original and retransmitted_ data into the network — the **offered load**|

λ'_in ≥ λ_in always, since retransmissions add to what the transport layer injects. The difference represents wasted bandwidth spent re-sending data the receiver already has, or already lost.

```
Host A ──┐  λ'_in (original + retransmitted)
         ├──[Router: FINITE buffer, capacity R]──► Destinations
Host B ──┘
         (packets dropped when buffer full → retransmissions needed)
```


![[Pasted image 20260624150621.png]]

**Three sub-cases (Figure 3.44):**

**Case (a): Perfect knowledge — sender retransmits only when certain of loss**

- The (unrealistic) assumption: the sender somehow knows exactly when a buffer slot is free, and sends only then.
- Result: no losses ever occur → λ'_in = λ_in → throughput equals λ_in → ideal performance, capped at R/2.
- This is the theoretical ceiling — actual performance is always worse.

**Case (b): Retransmission only on confirmed loss (realistic-ish)**

- The sender sets its timeout large enough to be virtually certain an un-ACKed segment is genuinely lost before retransmitting.
- With offered load λ'_in = R/2: the delivered throughput is only **R/3**.
- Why? Out of every 0.5R units of data the transport layer injects per second, 0.333R are original data and 0.166R are retransmitted data — a 2:1 useful-to-wasted ratio.

> **Cost #2 Discovered:** _The sender must perform retransmissions to compensate for dropped packets due to buffer overflow. This consumes real link bandwidth that could have carried fresh data._

**Case (c): Premature retransmission (timeout fires too early)**

- The sender times out and retransmits a segment that was merely _delayed_ in the queue, not actually lost.
- Now _both_ the original (delayed) packet and the retransmission arrive at the receiver — the receiver discards the duplicate copy.
- The router spent bandwidth forwarding a copy the receiver didn't need.
- Throughput asymptotes to **R/4** as offered load approaches R/2, since on average each packet is forwarded twice.

> **Cost #3 Discovered:** _Unneeded retransmissions by the sender in the face of large delays cause a router to use its link bandwidth to forward unneeded copies of a packet._

> **Analogy — A Printer That Re-Prints Because You Clicked Twice:** You send a print job, the printer is busy and the job is queued, but after a few seconds you assume it didn't receive the job and send it again. Now the printer prints two copies. The paper used for the second copy was wasted — the original was always going to print. TCP's premature retransmission is exactly this: re-sending something that would have arrived fine, wasting the network's equivalent of "paper" (link bandwidth).

---

### Scenario 3: Four Senders, Routers with Finite Buffers, and Multihop Paths

![[Pasted image 20260624151100.png]]

**Setup (Figure 3.45):**

- Four hosts transmit packets, each over overlapping two-hop paths.
- All router links have capacity **R** bytes/sec.
- All hosts use timeout/retransmit for reliability.
- The A→C path passes through routers R1 then R2. The B→D path also passes through R2. So **R2 is shared** between A–C and B–D traffic.

```
Host A ──[R1]──[R2]──► Host C        (A–C connection, 2 hops)
Host B ──[R4]──[R2]──► Host D        (B–D connection, 2 hops)
                ↑
         R2 is the shared bottleneck
```

**The multihop insight:**

_At low traffic:_ Buffer overflows at R2 are rare. Throughput ≈ offered load. An increase in λ_in increases λ_out. No surprises.

_At high traffic:_ Consider what happens when λ_in (and hence λ'_in) becomes very large for all connections. Router R2's incoming link from R1 can carry at most R bytes/sec of A–C traffic — R1's own link is the bottleneck on what feeds R2. Meanwhile, B–D traffic also arrives at R2 directly from R4, potentially at a very high rate. Because A–C and B–D traffic **compete for R2's finite buffer space**, the amount of A–C traffic that successfully gets through R2 becomes smaller and smaller as B–D's offered load grows.

In the extreme limit, as B–D's offered load approaches infinity, every available buffer slot at R2 is immediately filled by B–D traffic. The A–C connection's throughput at R2 goes to **zero**.

**The crucial inefficiency revealed:** Whenever a packet is dropped at R2, the work done by R1 to forward that packet from A to R2 was _completely wasted_. R1 consumed its full link bandwidth for a packet that never made it to Host C. If R1 had simply left that packet in R4's queue (or never forwarded it), R2's capacity could have been used for something that actually delivered value.

> **Cost #4 Discovered (and the most structurally important one):** _When a packet is dropped due to congestion, the transmission capacity that was used at each of the upstream links to forward that packet to the point at which it is dropped ends up having been wasted._

![[Pasted image 20260624151240.png]]
Figure 3.46 shows the result: as offered load increases beyond a moderate level, throughput actually **decreases** and eventually **collapses to zero**. This phenomenon — _congestion collapse_ — means that at very high load levels, sending more only yields less, because so many upstream resources are consumed carrying packets that will be dropped downstream.

> **Analogy — Carrying Groceries Upstairs Only to Find a Locked Door:** Imagine carrying heavy bags up five flights of stairs, only to find the apartment you're delivering to is locked. All that effort was wasted. Now imagine hundreds of delivery people doing this simultaneously, each clogging the stairwells carrying bags that will never be accepted — so even the few deliveries that _could_ succeed can't get through because the stairwells are full of people carrying bags to locked doors. That's congestion collapse in a multihop network.

**Summary of All Costs:**

|Scenario|New Cost of Congestion|
|---|---|
|Scenario 1 (∞ buffers)|Large queuing delays as arrival rate approaches link capacity|
|Scenario 2 (finite buffers, certain retransmit)|Wasted bandwidth on retransmissions for actual losses|
|Scenario 2 (finite buffers, premature retransmit)|Further wasted bandwidth on unneeded duplicate transmissions|
|Scenario 3 (multihop, finite buffers)|All upstream bandwidth spent on a packet that gets dropped downstream is _entirely wasted_; potential for congestion collapse|

---

## The Bottleneck Link: The Fundamental Problem

![[Pasted image 20260624151456.png]]

Before developing solutions, it helps to characterize the fundamental problem precisely. 
Figure 3.47 shows N transport-layer connections, each passing through a shared **bottleneck link** of capacity R bps.

**What is a bottleneck link?** A connection's bottleneck link is the link along its end-to-end path that, if senders slowly increase their transmission rate, would be the _first_ link on that path to experience congestion loss. It's the narrowest point in the pipe.

Key facts about bottleneck links:

- We usually refer to a link as a bottleneck only when it is in a congested state.
- A common and empirically well-supported assumption is that **a connection has at most one bottleneck link** at any given time (Cardwell 2017; Kleinrock 2018; Hu 2024).
- Connections with different source-to-destination paths may have different bottleneck links.

**Why the bottleneck link determines aggregate throughput:**

If the bottleneck link's arrival rate is near its capacity R, and the N connections sharing it try to increase their sending rates, the bottleneck simply cannot provide any additional throughput — it's already always busy. Therefore, _the aggregate end-to-end throughput achieved by all N connections passing through that bottleneck link is bounded by R_. If the link's capacity is fairly shared (Section 7.3.4), each connection gets approximately R/N.

> **Analogy — N Garden Hoses Sharing One Main Supply:** Think of N hoses connected to a single garden main supply pipe whose maximum flow rate is R liters/sec. No matter how wide each individual hose is opened, the combined output cannot exceed R liters/sec. If there are N hoses and the water is fairly shared, each hose gets about R/N liters/sec. Opening your hose wider doesn't get you more water — it just starves the others and wastes pressure upstream.

**The goal of congestion control**, stated precisely: _The N connections in Figure 3.47 should set their transport layer's sending rates so that their aggregate sending rate is close to, but does not exceed, the bottleneck link capacity._

Two fundamental questions immediately arise from this goal:

1. How does a connection _know_ that a link on its source-to-destination path is congested (i.e., that there is a bottleneck link)?
2. How do connections _individually regulate_ their sending rates so that their aggregate rate stays near but below their path's bottleneck capacity?

Question 1 is addressed in Section 7.6.2. Question 2 is addressed in Section 7.3.

---

## 3.6.2 End-to-End and Network-Assisted Approaches to Congestion Control

The two fundamental questions above lead to the two fundamental _approaches_ to congestion control, which differ in one key architectural dimension: **does the network layer (routers) provide explicit assistance to the transport layer for congestion-control purposes, or not?**

### Approach 1: End-to-End Congestion Control

**Core idea:** The network layer provides **no explicit support** to the transport layer for congestion control. Routers never tell senders "I am congested." The presence of congestion must be _inferred_ entirely by the end systems, based only on what they can observe about the network's behavior.

**What signals do end systems use?**

- **Packet loss** (detected by timeout or by receipt of three duplicate ACKs) — an indication that a buffer somewhere overflowed.
- **Delay** — increasing round-trip time is an early warning sign of building queues before actual loss occurs.
- **Throughput** — a measured drop in delivery rate can also signal network trouble.

**TCP's approach:** TCP takes the end-to-end approach. Since the IP layer is not required to provide feedback to hosts regarding network congestion, TCP must do all the work itself. In the classic formulation (Sections 3.7.1 and 3.7.2):

- A timeout or three duplicate ACKs is treated as a congestion signal.
- TCP responds by _decreasing its congestion window_ (and thus its sending rate).

More recent TCP "flavors" (covered in Section 3.7) additionally use the connection's measured RTT and/or throughput to infer and respond to congestion earlier, without waiting for loss events.

> **Analogy — Driving Without Dashboard Warning Lights:** Imagine driving a car where the engine sends no warning lights to the dashboard — no temperature gauge, no oil pressure indicator. You have to infer engine trouble from what you can _feel and hear_: the car slowing down (lost throughput), the engine running hot and sluggish (increasing delay), or finally a breakdown (packet loss). End-to-end congestion control is exactly this: TCP listens to what it experiences, not to what the routers explicitly say.

### Approach 2: Network-Assisted Congestion Control

**Core idea:** Routers provide **explicit feedback** to the sender and/or receiver regarding the network's congestion state. The network actively participates in congestion control rather than staying silent.

**What forms can this feedback take?**

_Simple feedback (1-bit signal):_

- A single bit set by a router in a passing packet to indicate congestion — used in early IBM SNA (1982), DEC DECnet (1989), Ramakrishnan 1990 architectures, and ATM (1995).
- Modern TCP/IP's **Explicit Congestion Notification (ECN)** mechanism (Section 3.7.2) also uses this approach: routers set bits in IP packet headers; the receiver notifies the sender.

_Rich feedback (explicit rate):_

- In **ATM Available Bit Rate (ABR)** congestion control, a router explicitly informs the sender of the _maximum host sending rate_ it can support on the outgoing link — the sender then uses exactly that rate, no guessing required.

**Two paths for delivering network feedback back to the sender (Figure 3.48):**

![[Pasted image 20260624151815.png]]

The second path (via receiver) is the more common form and is what TCP's ECN mechanism uses. Because the notification travels in the data packet to the receiver, and then back to the sender in the acknowledgment, this round-trip takes a full RTT — meaning congestion signals always arrive at the sender with at least one RTT of delay.

> **Analogy — Direct vs. Relay Fire Alarm:** Direct feedback is like a fire alarm in your office that immediately triggers a siren on your desk. Network-feedback-via-receiver is like the fire alarm triggering an alert to the building manager, who then calls your phone to tell you to evacuate — you still get the message, but there's a delay while the information travels the roundabout route.

**Comparison of the two approaches:**

|Dimension|End-to-End|Network-Assisted|
|---|---|---|
|Router complexity|Low — routers stay dumb, just forward|Higher — routers must detect and signal congestion|
|Feedback speed|Slow — sender infers from loss/delay, ≥1 RTT|Can be faster (direct) or ≥1 RTT (via receiver)|
|Accuracy of signal|Indirect; loss can have causes other than congestion|Direct; router _knows_ it is congested|
|Internet deployment|Default approach (TCP/IP)|Optional / specialized (ECN, ABR, QUIC)|
|Deployment cost|No changes to routers needed|Requires network-layer changes|

**What the Internet actually does:** The default Internet approach is end-to-end. IP provides no congestion feedback to hosts. However, as Section 3.7.3 will show, IP and TCP _may optionally_ implement network-assisted congestion control via ECN — the infrastructure is there, but it isn't mandatory.

---

## Why Congestion Control Matters (Putting It All Together)

The three scenarios in Section 3.6.1 collectively establish that an uncontrolled network where every host sends at whatever rate it pleases will produce:

1. Unbounded queuing delay as loads approach link capacity (even with infinite buffers).
2. Wasted bandwidth on retransmissions for losses (finite buffers, certain retransmit).
3. Further wasted bandwidth on premature, unnecessary retransmissions (finite buffers, early timeout).
4. Congestion collapse — throughput going to _zero_ — in multihop networks when upstream bandwidth is squandered forwarding packets that will be dropped downstream.

The need for some form of congestion control is clear. The question is only which approach — end-to-end inference, network-assisted signaling, or some hybrid — best fits the deployment constraints and performance requirements of the network in question.

> **The Core Takeaway:** Congestion control is not just a performance optimization — it's a _stability mechanism_. Without it, networks can and do collapse: adding more load yields less throughput, not more, and the entire system enters a death spiral. Congestion control is what keeps the Internet from being its own worst enemy.

---

## Questions I Still Have

- [ ] Scenario 3 shows that throughput collapses to zero in the limit as offered load → ∞ for multihop paths. Does this mean a single aggressive sender on a shared path can, in principle, starve _all other connections_ passing through the same bottleneck router?
- [ ] ECN requires both routers and end systems to support it — what fraction of the Internet's routers actually support ECN marking today, and does the lack of ubiquitous deployment mean end-to-end inference is still the dominant signal in practice?
- [ ] The textbook says "a connection has at most one bottleneck link" is "usually good in both theory and practice." Are there known counterexamples where a connection has two links that are simultaneously at capacity, and what happens to congestion control behavior in those cases?
- [ ] In the "via receiver" feedback path for network-assisted control, the congestion notification takes one full RTT to reach the sender. For long-RTT links (e.g., satellite), is this delay large enough to make ECN signals arrive too late to be useful, and how do modern protocols compensate?
- [ ] ATM ABR explicitly tells senders the maximum rate the network can support — this seems strictly better than guessing. Why did ATM largely lose to IP/TCP, and is something like ABR's explicit rate feedback seeing a revival in modern datacenter congestion control (e.g., DCQCN, SWIFT)?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Congestion**|A network condition in which too many sources are sending data at too high a rate, causing router buffers to overflow and packets to be dropped|
|**Congestion control**|Mechanisms that throttle senders in the face of network congestion, treating the _cause_ (too much traffic) rather than just the symptom (a dropped packet)|
|**Offered load (λ'_in)**|The rate at which the transport layer sends data (both original and retransmitted) into the network; always ≥ the application's original data rate λ_in|
|**Per-connection throughput (λ_out)**|The number of bytes per second actually delivered to the receiver application for a given connection|
|**Link capacity (R)**|The maximum transmission rate of an outgoing link, in bytes/sec|
|**Congestion collapse**|The scenario (especially in multihop networks) where increasing offered load actually decreases end-to-end throughput, eventually toward zero, because upstream bandwidth is wasted forwarding packets that get dropped downstream|
|**Bottleneck link**|The link along a connection's end-to-end path that would be the first to experience congestion loss as senders increase their transmission rates; the link that limits aggregate throughput|
|**End-to-end congestion control**|An approach where the network layer provides no explicit congestion feedback; end systems infer congestion from observed network behavior (loss, delay, throughput)|
|**Network-assisted congestion control**|An approach where routers actively provide feedback to senders and/or receivers about the congestion state of the network|
|**Choke packet**|A direct feedback message sent by a congested router to a sender, explicitly indicating congestion (one form of network-assisted feedback)|
|**ECN (Explicit Congestion Notification)**|A network-assisted mechanism where a router marks a bit in a passing IP packet to indicate congestion; the receiver then notifies the sender in a subsequent ACK|
|**ATM ABR (Available Bit Rate)**|An ATM congestion-control scheme where routers explicitly inform senders of the maximum sending rate they can support on an outgoing link|
|**Premature retransmission**|A retransmission triggered by a timeout that fired before the original packet was actually lost — the original packet eventually arrives anyway, making the retransmission a wasted copy|
|**Wasted upstream capacity**|The bandwidth consumed by upstream routers to forward a packet that is ultimately dropped at a downstream router; a key cost unique to multihop congestion|

---

## Related Concepts

→ Next: [[3.6 Principles of Congestion Control]]