---
title: MULTIPLE ACCESS LINK & PROTOCOL
date: 2026-07-20
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 6.3 Multiple Access Links and Protocols

> **One-Line Summary:** A **broadcast channel** — shared by multiple sending and receiving nodes — needs a **multiple access protocol** to decide _who_ gets to transmit _when_, since simultaneous transmissions **collide** and destroy each other; the dozens of real-world protocols in use all fall into one of three families — **channel partitioning** (TDM/FDM/CDMA — divide the channel statically), **random access** (ALOHA/CSMA/CSMA-CD — let nodes gamble on collisions and recover), and **taking-turns** (polling/token-passing — coordinate access explicitly) — each representing a different trade-off among efficiency, decentralization, delay, and simplicity.

---

## Core Idea: Two Kinds of Links, One New Problem

Recall from the introduction to this chapter that there are **two types of network links**: **point-to-point** links and **broadcast** links.

|Link Type|Description|Example Protocols|
|---|---|---|
|**Point-to-point**|A single sender at one end of the link, a single receiver at the other end|Point-to-Point Protocol (PPP), High-Level Data Link Control (HDLC)|
|**Broadcast**|Multiple sending **and** receiving nodes all connected to the **same, single, shared broadcast channel**|Ethernet, wireless LANs|

The term "broadcast" is used here because when **any one node** transmits a frame, the channel **broadcasts** the frame and **each of the other nodes receives a copy** — a fundamentally different situation from a private, one-to-one link.

```
Fig -- Point-to-Point vs Broadcast Link
────────────────────────────────────────
 Point-to-point:
   Sender ─────────────────▶ Receiver
        (1 sender, 1 receiver)

 Broadcast:
        Node A   Node B   Node C
          │        │        │
   ───────┴────────┴────────┴───────
          shared broadcast channel
   (all nodes send/receive on same
    channel; TV = 1-way, this = 2-way)
```

Broadcast channels are often used in **LANs** — networks that are geographically concentrated, such as within a single building or on a corporate/university campus. This section focuses on how multiple-access channels are used in LANs specifically.

### The Multiple Access Problem

This section steps back from specific link-layer protocols to first examine a problem of general importance to the link layer: **how do you coordinate the access of multiple sending and receiving nodes to a shared broadcast channel?** This is known as the **multiple access problem**.

We're all familiar with the notion of **broadcasting** — television has been using it since its invention. But traditional television is a **one-way broadcast** (one fixed node transmitting to many receiving nodes), whereas nodes on a computer-network broadcast channel can **both send and receive**.

> **Analogy — A Cocktail Party:** A more apt human analogy for a broadcast channel is a **cocktail party**, where many people gather in a large room (the air providing the broadcast medium) to talk and listen.

> **Analogy — A Classroom:** A second good analogy, familiar to most readers, is a **classroom** — where teacher(s) and student(s) share the same, single, broadcast medium.

A central problem in **both** scenarios is determining **who gets to talk (transmit into the channel) and when**. As humans, we've evolved an elaborate set of protocols for sharing the broadcast channel:

> "Give everyone a chance to speak." "Don't speak until you are spoken to." "Don't monopolize the conversation." "Raise your hand if you have a question." "Don't interrupt when someone is speaking." "Don't fall asleep when someone is talking."

Computer networks similarly have protocols — so-called **multiple access protocols** — by which nodes regulate their transmission into the shared broadcast channel. As shown in Figure 6.8, multiple access protocols are needed in a wide variety of network settings, including both **wired and wireless access networks**, and **satellite networks**.

```
Fig -- Real-World Broadcast Channel Settings
────────────────────────────────────────────
 Shared wire (e.g., cable access net):
   Head-end ══╦══╦══╦══
              house house house

 Shared wireless (e.g., WiFi):
   laptop <--)) AP ((--> laptop

 Satellite: ground stations <--> satellite
            (all share the uplink/downlink)

 Cocktail party: everyone shares "the air"
```

![[Pasted image 20260722204007.png]]

Although technically each node accesses the broadcast channel through its **adapter**, this section will simply refer to the "**node**" as the sending and receiving device. In practice, hundreds or even thousands of nodes can directly communicate over a single broadcast channel.

### Why Collisions Are So Costly

Because **all** nodes are capable of transmitting frames, **more than two nodes can transmit frames at the same time**. When this happens, **all** of the nodes receive **multiple** frames at the same time — that is, the transmitted frames **collide** at all of the receivers.

Typically, when there is a collision, **none** of the receiving nodes can make sense of _any_ of the frames that were transmitted — in a sense, the signals of the colliding frames become **inextricably tangled together**. Thus, **all** the frames involved in the collision are **lost**, and the broadcast channel is **wasted** during the collision interval. Clearly, if many nodes want to transmit frames frequently, many transmissions will result in collisions, and much of the bandwidth of the broadcast channel will be wasted.

In order to ensure that the broadcast channel performs useful work when multiple nodes are active, it's necessary to somehow **coordinate** the transmissions of the active nodes — this coordination job is the responsibility of the **multiple access protocol**. Over the past 50 years, thousands of papers and hundreds of PhD dissertations have been written on multiple access protocols; active research continues today, particularly for new types of wireless links.

### Three Categories, and Four Desirable Properties

Dozens of multiple access protocols have been implemented in a variety of link-layer technologies over the years. Nevertheless, just about **any** multiple access protocol can be classified as belonging to one of **three categories**:

1. **Channel partitioning protocols**
2. **Random access protocols**
3. **Taking-turns protocols**

This section covers each of these three categories in the following three subsections. Ideally, a multiple access protocol for a broadcast channel of rate **R** bits per second should have these desirable characteristics:

|#|Desirable Property|
|---|---|
|1|**When only one node has data to send**, that node has a throughput of **R bps**.|
|2|**When M nodes have data to send**, each of these nodes has a throughput of **R/M bps** — on average, over some suitably defined interval of time (this need not imply each node's _instantaneous_ rate is always exactly R/M).|
|3|**The protocol is decentralized** — there is no centralized controller node that represents a single point of failure for the network.|
|4|**The protocol is simple**, so that it is inexpensive to implement.|

No single protocol perfectly satisfies all four properties at once — as will become clear, each category makes distinct trade-offs among them.

---

## 6.3.1 Channel Partitioning Protocols

Recall from the early discussion of **time-division multiplexing (TDM)** and **frequency-division multiplexing (FDM)** that these are two techniques that can be used to **partition** a broadcast channel's bandwidth among all the nodes sharing that channel.

### TDM (Time-Division Multiplexing)

Suppose the channel supports **N** nodes, and the transmission rate of the channel is **R bps**. TDM divides time into **time frames**, and further divides each time frame into **N time slots**. Each time slot is then assigned to one of the N nodes. Whenever a node has a packet to send, it transmits the packet's bits during its assigned time slot in the revolving TDM frame. Typically, slot sizes are chosen so that a single packet can be transmitted during a slot time.

> **Terminology alert:** The TDM _time frame_ should **not** be confused with the link-layer unit of data exchanged between sending and receiving adapters, which is also called a "frame." To reduce confusion, this subsection refers to the link-layer unit of data exchanged as a **packet**.

```
Fig -- TDM vs FDM (N=4 nodes, rate R)
────────────────────────────────────────
 FDM: R bps split into N frequency bands
  ┌────────┐
  │ Node 1 │  R/N bps (own frequency)
  ├────────┤
  │ Node 2 │  R/N bps
  ├────────┤
  │ Node 3 │  R/N bps
  ├────────┤
  │ Node 4 │  R/N bps
  └────────┘

 TDM: R bps split into repeating slots
  |1|2|3|4|1|2|3|4|1|2|3|4| ...
  <---- one TDM frame (4 slots) ---->
  Each node gets 1 slot per TDM frame.
```

![[Pasted image 20260722204220.png]]

Returning to the cocktail party analogy: a TDM-regulated cocktail party would allow one partygoer to speak for a fixed period of time, then let another partygoer speak for the same amount of time, and so on — once everyone had a chance to talk, the pattern would repeat.

**TDM is appealing** because it **eliminates collisions** and is **perfectly fair**: each node gets a dedicated transmission rate of R/N bps during each frame time. **However**, it has **two major drawbacks**:

1. A node is limited to an average rate of R/N bps **even when it is the only node with packets to send**.
2. A node **must always wait for its turn** in the transmission sequence — again, even when it's the only node with a frame to send. (Imagine being the only partygoer with anything to say, or the even rarer case where _everyone_ wants to speak but only one person actually has something to say — TDM would be a poor choice for this particular party.)

### FDM (Frequency-Division Multiplexing)

While TDM shares the broadcast channel in **time**, FDM divides the **R bps** channel into different **frequencies** (each with a bandwidth of R/N) and assigns each frequency to one of the N nodes. FDM thus creates **N smaller channels** of R/N bps out of the single, larger R channel.

FDM shares **both the benefits and drawbacks of TDM**: it avoids collisions and divides the bandwidth fairly among the N nodes, but a node is still limited to a bandwidth of R/N even when it's the **only** node with a frame to send.

TDM and FDM techniques are widely used in **cellular, WiFi, Bluetooth, and satellite systems**; both are also used in **cable access networks**.

### CDMA (Code Division Multiple Access)

A third channel partitioning protocol is **code division multiple access (CDMA)**. While TDM and FDM assign time slots and frequencies, respectively, to the nodes, **CDMA assigns a different code to each node**. Each node then uses its unique code to encode the data bits it sends.

If the codes are chosen carefully, CDMA networks have the wonderful property that different nodes can transmit **simultaneously** and yet have their respective receivers correctly receive a sender's encoded data bits (assuming the receiver knows the sender's code) **in spite of interfering transmissions by other nodes**.

CDMA was a **foundational technology for 3G cellular networks**, but has since been replaced by other technologies in **4G and 5G networks**.

---

## 6.3.2 Random Access Protocols

The second broad class of multiple access protocols is **random access protocols**. In a random access protocol, a transmitting node **always transmits at the full rate of the channel, namely, R bps**.

When there is a **collision**, each node involved in the collision **repeatedly retransmits its frame** (that is, packet) **until its frame gets through without a collision**. But when a node experiences a collision, it doesn't necessarily retransmit the frame right away — instead, it **waits a random delay before retransmitting the frame**. Each node involved in a collision chooses **independent random delays**. Because the random delays are independently chosen, it's possible that one of the nodes will pick a delay that is **sufficiently less than the delays of the other colliding nodes** and will therefore be able to sneak its frame into the channel without a collision.

There are dozens if not hundreds of random access protocols described in the literature. This section covers a few of the most well-known: the **ALOHA protocols** and the **carrier sense multiple access (CSMA)** protocols. Ethernet is a popular and widely deployed CSMA protocol.

### Slotted ALOHA

Let's begin with one of the simplest random access protocols, **slotted ALOHA**. In our description, we assume:

- All frames consist of exactly **L bits**.
- Time is divided into **slots of size L/R seconds** (that is, a slot equals the time to transmit one frame).
- Nodes start to transmit frames **only at the beginnings of slots**.
- The nodes are **synchronized** so that each node knows when the slots begin.
- If two or more frames collide in a slot, then **all** the nodes detect the collision event **before the slot ends**.

Let **p** be a probability, a number between 0 and 1. The operation of slotted ALOHA in each node is simple:

1. **When the node has a fresh frame to send**, it waits until the beginning of the next slot and transmits the entire frame in the slot.
2. **If there isn't a collision**, the node has successfully transmitted its frame and thus needs not consider retransmitting the frame. (The node can prepare a new frame for transmission, if it has one.)
3. **If there is a collision**, the node detects the collision before the end of the slot. It retransmits its frame in each subsequent slot with probability **p** until the frame is transmitted without a collision.

By "retransmitting with probability p," we mean the node effectively tosses a biased coin: heads corresponds to "retransmit," which occurs with probability p; tails corresponds to "skip the slot and toss the coin again in the next slot," occurring with probability (1 − p). All nodes involved in a collision toss their coins independently.

```
Fig -- Slotted ALOHA: Slot Outcomes
────────────────────────────────────────
 Node 1: [1]-------[1]---------[1]--
 Node 2: [2][2]--[2]----------------
 Node 3: [3]------------[3]----[3]--

 Slot #:  1  2  3  4  5  6  7  8  9
 Result:  C  E  C  S  E  C  E  S  S

 C = Collision   E = Empty
 S = Successful transmission
 (schematic example; exact node/slot
  pairing illustrative only)
```

![[Pasted image 20260722204356.png]]

### Advantages of Slotted ALOHA

Slotted ALOHA would appear to have many advantages:

- **Unlike channel partitioning**, slotted ALOHA allows a node to transmit **continuously at the full rate, R**, when that node is the only active node.
- Slotted ALOHA is **highly decentralized**, because each node detects collisions and independently decides when to retransmit. (It does, however, require the slots to be synchronized in the nodes — the soon-to-be-discussed unslotted "pure" ALOHA, as well as CSMA protocols, require no such synchronization.)
- Slotted ALOHA is also an **extremely simple protocol**.

### Efficiency: How Good Is It, Really?

Slotted ALOHA works well when there is only **one** active node — but how efficient is it when there are **multiple** active nodes? Two efficiency concerns arise:

1. A certain fraction of slots will have **collisions** and will therefore be "wasted."
2. Another fraction of slots will be **empty**, because active nodes probabilistically refrain from transmitting.

The only "unwasted" slots will be those in which **exactly one** node transmits — a slot in which exactly one node transmits is said to be a **successful slot**. The **efficiency** of a slotted multiple access protocol is defined as the **long-run fraction of successful slots** in the case when there are a large number of active nodes, each always having a large number of frames to send. If no form of access control were used, and each node were to immediately retransmit after each collision, the efficiency would be **zero**.

### Deriving Maximum Efficiency

To derive the maximum efficiency, assume each node attempts to transmit a frame in **each slot** with probability **p** (i.e., each node always has a frame to send and transmits with probability p, whether the frame is fresh or previously collided). Suppose there are **N** nodes.

- The probability that a **given** slot is successful is the probability that **one** of the N nodes transmits and the remaining **N − 1** nodes do **not** transmit.
- The probability that a given node transmits is **p**; the probability that the remaining N − 1 nodes do not transmit is **(1 − p)^(N−1)**.
- The probability that a given node has a success is therefore **p(1 − p)^(N−1)**.
- Because there are N nodes, the probability that **any one** of the N nodes has a success is **Np(1 − p)^(N−1)**.

Thus, when there are N active nodes, **the efficiency of slotted ALOHA is Np(1 − p)^(N−1)**. To obtain the **maximum** efficiency for N active nodes, we'd have to find the p* that maximizes this expression. And to obtain the maximum efficiency for a _large number_ of active nodes, we take the limit of Np*(1 − p*)^(N−1) as N approaches infinity.

**The result:** the maximum efficiency of the protocol is given by **1/e ≈ 0.37**. That is, when a large number of nodes have many frames to transmit, then (at best) only **37 percent** of the slots do useful work! The effective transmission rate of the channel is therefore not R bps but only **0.37 R bps**. A similar analysis also shows that 37 percent of the slots go empty, and **26 percent** of slots have collisions.

> **A concrete example:** Imagine a poor network administrator who purchased a **100-Mbps slotted ALOHA system**, expecting to be able to use the network to transmit data among a large number of users at an aggregate rate of, say, **80 Mbps**. Although the channel is capable of transmitting a given frame at the full channel rate of 100 Mbps, in the long run, the successful throughput of this channel will be **less than 37 Mbps**.

### Pure (Unslotted) ALOHA

The first ALOHA protocol [Abramson 1970] was actually an **unslotted, fully decentralized** protocol — known today as **pure ALOHA**.

In pure ALOHA, when a frame first arrives (that is, a network-layer datagram is passed down from the network layer at the sending node), the node **immediately** transmits the frame in its entirety into the broadcast channel. If a transmitted frame experiences a collision with one or more other transmissions, the node will then **immediately** (after completely transmitting its collided frame) retransmit the frame with probability **p**. Otherwise, the node waits for a frame transmission time. After this wait, it then transmits the frame with probability p, or waits (remaining idle) for another frame time with probability (1 − p).

```
Fig -- Pure ALOHA: Vulnerable Period
────────────────────────────────────────
      t0-1          t0          t0+1
 Node A:  ---[A's frame]---
 Node i:            [ i's frame ]
 Node B:                  ---[B's frame]--

 i's frame is vulnerable to any other
 transmission starting in (t0-1, t0+1):
 vulnerable window = 2 x frame time
 (double that of slotted ALOHA)
```

![[Pasted image 20260722204523.png]]

**Determining pure ALOHA's maximum efficiency:** using the same assumptions as in the slotted ALOHA analysis, take the frame transmission time as the unit of time. Suppose a node begins transmission at time t₀. For this frame to be successfully transmitted, **no other nodes can begin their transmission** in the interval [t₀ − 1, t₀] — such a transmission would overlap with the beginning of node i's frame. The probability that all other nodes do not begin a transmission in this interval is **(1 − p)^(N−1)**. Similarly, no other node can begin a transmission while node i is transmitting, as such a transmission would overlap with the _latter_ part of node i's transmission — the probability of this is also **(1 − p)^(N−1)**. Thus, the probability that a given node has a successful transmission is **p(1 − p)^(2(N−1))**. Taking limits as in the slotted ALOHA case, the maximum efficiency of pure ALOHA is only **1/(2e) — exactly half that of slotted ALOHA**. This is the price paid for a **fully decentralized** ALOHA protocol (no need for slot synchronization).

### Carrier Sense Multiple Access (CSMA)

In **both** slotted and pure ALOHA, a node's decision to transmit is made **independently** of the activity of the other nodes attached to the broadcast channel. In particular, a node neither pays attention to whether another node happens to be transmitting when it begins to transmit, nor stops transmitting if another node begins to interfere with its transmission. In the cocktail-party analogy, ALOHA protocols are quite like a **boorish partygoer** who continues to chatter away regardless of whether other people are talking.

As humans, we have human protocols that allow us not only to behave with more civility, but also to decrease the amount of time spent "colliding" with each other in conversation and, consequently, to increase the amount of data we exchange in our conversations. Specifically, there are **two important rules for polite human conversation**:

|Rule|Networking Equivalent|Description|
|---|---|---|
|**Listen before speaking**|**Carrier sensing**|A node listens to the channel before transmitting. If a frame from another node is currently being transmitted into the channel, a node waits until it detects no transmissions for a short amount of time and then begins transmission.|
|**If someone else begins talking at the same time, stop talking**|**Collision detection**|A transmitting node listens to the channel while it is transmitting. If it detects that another node is transmitting an interfering frame, it stops transmitting and waits a random amount of time before repeating the sense-and-transmit-when-idle cycle.|

These two rules are embodied in the family of **carrier sense multiple access (CSMA)** and **CSMA with collision detection (CSMA/CD)** protocols.

> **Case History — Norm Abramson and Alohanet:** Norm Abramson, a PhD engineer with a passion for surfing and an interest in packet switching, brought these interests together at the University of Hawaii in 1969. Hawaii's many mountainous islands made land-based networking difficult to install and operate, so Abramson designed a network that did packet switching over radio, with one central host and several secondary nodes scattered over the Hawaiian Islands, using two channels on different frequency bands. This observation about collisions on the upstream channel led Abramson to devise the **pure ALOHA protocol**. In 1970 he connected his ALOHAnet to the ARPAnet. Abramson's work is important not only because it was the first example of a radio packet network, but also because it inspired **Bob Metcalfe**, who a few years later modified the ALOHA protocol to create the **CSMA/CD protocol and the Ethernet LAN**.

### Why Do Collisions Still Occur Under CSMA?

A natural question: if all nodes perform carrier sensing, why do collisions occur in the first place? After all, a node will refrain from transmitting whenever it senses another node is transmitting. The answer is best illustrated using **space-time diagrams**.

```
Fig -- CSMA Space-Time Diagram
────────────────────────────────────────
 Space:    A ---- B ---- C ---- D

 t0: B senses channel idle, starts
     sending; B's bits propagate
     outward in both directions
 t1: D has a frame to send; D senses
     channel idle (B's bits haven't
     reached D yet) -> D transmits
     -> collision occurs near D
```

![[Pasted image 20260722204739.png]]

Consider four nodes (A, B, C, D) attached to a linear broadcast bus; the horizontal axis shows the position of each node in space, the vertical axis represents time. **At time t₀**, node B senses the channel idle (no other nodes are currently transmitting) and begins transmitting, with its bits propagating in both directions along the broadcast medium. The downward propagation of B's bits with increasing time indicates that a **nonzero amount of time is needed** for B's bits to actually propagate (albeit near the speed of light) along the broadcast medium.

**At time t₁ (t₁ > t₀)**, node D has a frame to send. Although B is currently transmitting, the bits being transmitted by B have **not yet reached D**, so D senses the channel **idle** at t₁. In accordance with the CSMA protocol, D thus begins transmitting its frame. A short time later, B's transmission begins to **interfere with D's transmission at D**.

**The takeaway:** the end-to-end **channel propagation delay** of a broadcast channel — the time it takes for a signal to propagate from one of the nodes to another — plays a crucial role in determining CSMA's performance. **The longer this propagation delay, the greater the chance that a carrier-sensing node is not yet able to sense a transmission that has already begun at another node in the network.**

### Carrier Sense Multiple Access with Collision Detection (CSMA/CD)

In the plain CSMA scenario above, nodes B and D do **not** perform collision detection — both B and D continue to transmit their frames in their entirety even though a collision has occurred. **When a node performs collision detection, it ceases transmission as soon as it detects a collision** — rather than continuing to transmit a useless, damaged (interfered-with) frame in its entirety.

```
Fig -- CSMA/CD: Detect and Abort Early
────────────────────────────────────────
 Space:    A ---- B ---- C ---- D

 t0: B starts sending (channel was
     idle at B)
 t1: D starts sending (channel still
     looked idle to D)
 Both B and D detect the other's
 signal energy mid-transmission and
 ABORT immediately -- unlike plain
 CSMA, which finishes sending the
 doomed frame anyway.
```

![[Pasted image 20260722205242.png]]

**Adding collision detection to a multiple access protocol helps protocol performance** by not wasting channel capacity transmitting a frame that's already doomed to be unusable.

### CSMA/CD from the Adapter's Perspective

Before analyzing CSMA/CD further, here is its operation summarized from the perspective of an **adapter** (in a node) attached to a broadcast channel:

1. The adapter **obtains a datagram** from the network layer, prepares a link-layer frame, and puts the frame in the adapter buffer.
2. **If the adapter senses the channel is idle** (no signal energy entering the adapter), it starts to transmit the frame. If the adapter senses the channel is **busy**, it waits until it senses no signal energy and then starts to transmit the frame.
3. **While transmitting**, the adapter monitors for the presence of signal energy coming from other adapters using the broadcast channel.
4. If the adapter transmits the entire frame **without** detecting signal energy from other adapters, the adapter is **finished** with the frame. If, on the other hand, the adapter **detects** signal energy from other adapters while transmitting, it **aborts** the transmission (that is, it stops transmitting its frame).
5. **After aborting**, the adapter waits a **random** amount of time and then returns to step 2.

### Why the Wait Time Must Be Random

The need to wait a **random** (rather than fixed) amount of time before retrying is clear: if two nodes transmitted frames at the same time and then both waited the **same fixed** amount of time, they'd collide **forever**, over and over.

But what's a good interval from which to choose the random backoff time?

- If the interval is **large** and the number of colliding nodes is **small**, nodes are likely to wait a large amount of time (with the channel remaining idle) before repeating the sense-and-transmit-when-idle step.
- If the interval is **small** and the number of colliding nodes is **large**, it's likely the chosen random values will be nearly the same, and the transmitting nodes will collide again.

**What we'd like** is an interval that is short when the number of colliding nodes is small, and long when the number of colliding nodes is large.

### Binary Exponential Backoff

The **binary exponential backoff** algorithm — used in **Ethernet** as well as in **DOCSIS cable network** multiple access protocols — elegantly solves this problem. Specifically, when transmitting a frame that has already experienced **n** collisions, a node chooses the value of **K** at random from **{0, 1, 2, ..., 2ⁿ − 1}**. Thus, the more collisions experienced by a frame, the larger the interval from which K is chosen.

For Ethernet, the actual amount of time a node waits is **K × 512 bit times** (i.e., K times the amount of time needed to send 512 bits into the Ethernet), and the maximum value that n can take is **capped at 10**.

**Worked example:** Suppose a node attempts to transmit a frame for the first time and, while transmitting, detects a collision. The node then chooses **K = 0 with probability 0.5**, or **K = 1 with probability 0.5**.

- If the node chooses K = 0, it immediately begins sensing the channel.
- If the node chooses K = 1, it waits 512 bit times (e.g., 5.12 microseconds for a 100 Mbps Ethernet) before beginning the sense-and-transmit-when-idle cycle.

|# of Collisions (n)|K chosen equally likely from...|
|---|---|
|1|{0, 1}|
|2|{0, 1, 2, 3}|
|3|{0, 1, 2, 3, 4, 5, 6, 7}|
|10 or more|{0, 1, 2, ..., 1023} (capped — set size stops growing beyond n=10)|

Thus, the size of the sets from which K is chosen grows **exponentially** with the number of collisions — hence the name "**binary exponential backoff**."

**One more subtlety:** each time a node prepares a _new_ frame for transmission, it runs the CSMA/CD algorithm without taking into account any collisions that may have occurred in the recent past. So it's possible that a node with a brand-new frame will immediately be able to sneak in a successful transmission while several other nodes are stuck in the exponential backoff state.

> **Analogy — Progressively Longer Waits After Awkward Interruptions:** Imagine two people who both start speaking at the exact same moment at a party. If, after every interruption, they both waited the _same_ fixed pause before trying again, they'd keep starting at the same instant forever — an endless loop of stepping on each other's sentences. Binary exponential backoff is like each person, after each successive awkward double-start, choosing to wait a pause drawn from an **increasingly wide range of possible lengths** — the first time, maybe "immediately" or "wait a beat"; after several repeated collisions, the possible wait times spread out much further, making it statistically far more likely that one person's chosen pause is meaningfully shorter than the other's, letting exactly one of them go first.

### CSMA/CD Efficiency

When only **one** node has a frame to send, the node can transmit at the **full channel rate** (e.g., for Ethernet, typical rates are 10 Mbps, 100 Mbps, or 1 Gbps). However, if **many** nodes have frames to transmit, the **effective transmission rate** of the channel can be much less.

The **efficiency of CSMA/CD** is defined to be the long-run fraction of time during which frames are being transmitted on the channel **without collisions** when there is a large number of active nodes, with each node having a large number of frames to send.

To present a closed-form approximation of the efficiency of Ethernet, let **d_prop** denote the **maximum time** it takes signal energy to propagate between any two adapters, and let **d_trans** be the time to transmit a **maximum-size frame** (approximately 1.2 msecs for a 10 Mbps Ethernet). A full derivation is beyond the scope of the text [Lam 1980; Bertsekas 1991]; the resulting approximation is simply stated as:

```
Efficiency = 1 / (1 + 5 · d_prop / d_trans)
```

**Interpreting this formula:**

- As **d_prop approaches 0**, efficiency **approaches 1**. This matches intuition — if propagation delay is zero, colliding nodes will abort immediately without wasting the channel.
- As **d_trans becomes very large**, efficiency also **approaches 1**. This too is intuitive: when a frame grabs the channel, it will hold onto that channel for a very long time, so the channel will be doing productive work most of the time.

---

## 6.3.3 Taking-Turns Protocols

Recall the two desirable properties of a multiple access protocol mentioned earlier: (1) when only **one** node is active, that node has a throughput of R bps, and (2) when **M** nodes are active, then each node has a throughput of nearly R/M bps. The ALOHA and CSMA protocols have this **first** property but **not** the second — this has motivated researchers to create another class of protocols, the **taking-turns protocols**. As with random access, there are dozens of taking-turns protocols with many variations; two of the more important ones are discussed here.

### The Polling Protocol

The **polling protocol** requires one of the nodes to be designated as a **controller node**. The controller node **polls** each of the nodes in a round-robin fashion. Specifically, the controller node first sends a message to node 1, saying that it (node 1) can transmit up to some **maximum number** of frames. After node 1 transmits some frames, the controller node tells node 2 it can transmit up to the maximum number of frames. (The controller node can determine when a node has finished sending its frames by observing the lack of a signal on the channel.) The procedure continues in this manner, with the controller node polling each of the nodes in a cyclic manner.

**Advantage:** the polling protocol eliminates the collisions and empty slots that plague random access protocols, allowing it to achieve much higher efficiency.

**Drawbacks:**

|Drawback|Description|
|---|---|
|**Polling delay**|The protocol introduces a polling delay — the amount of time required to notify a node that it can transmit. If, for example, only one node is active, then the node will transmit at a rate less than R bps, as the controller must poll each of the inactive nodes in turn each time the active node has sent its maximum number of frames.|
|**Single point of failure**|If the controller node **fails**, the entire channel becomes inoperable. (The Bluetooth protocol, covered in Section 7.5.1, is an example of a polling protocol.)|

### The Token-Passing Protocol

The second taking-turns protocol is the **token-passing protocol**. In this protocol there is **no controller node**. A small, special-purpose frame known as a **token** is exchanged among the nodes in some fixed order. For example, node 1 might always send the token to node 2, node 2 might always send the token to node 3, and node N might always send the token to node 1.

When a node receives a token, it holds onto the token **only if it has frames to transmit**; otherwise, it **immediately forwards** the token to the next node. If a node does have frames to transmit when it receives the token, it sends up to a **maximum number of frames** and then forwards the token to the next node.

**Advantage:** token passing is **decentralized and highly efficient**.

**Drawbacks:**

- If **one node crashes**, it can crash the **entire channel** (since the token can no longer be forwarded).
- If a node accidentally **neglects to release** the token, some **recovery procedure** must be invoked to get the token back in circulation.

Over the years, many token-passing protocols have been developed, including the **fiber distributed data interface (FDDI)** protocol [Jain 1994] and the **IEEE 802.5 token ring protocol** [IEEE 802.5 2012], each having to address these as well as other sticky issues.

---

## 6.3.4 DOCSIS: The Link-Layer Protocol for Cable Internet Access

In the previous three subsections, three broad classes of multiple access protocols were introduced: channel partitioning, random access, and taking-turns protocols. A cable access network makes for an excellent case study here, since it draws on aspects of **each** of these three classes within the _same_ network!

Recall that a cable access network typically connects several thousand residential cable modems to a **cable modem termination system (CMTS)** at the cable network headend. The **Data-Over-Cable Service Interface Specifications (DOCSIS)** [DOCSIS 3.1 2014; Hamzeh 2015] specifies the cable data network architecture and its protocols.

### FDM Splits Downstream and Upstream

DOCSIS uses **FDM** to divide the downstream (CMTS to modem) and upstream (modem to CMTS) network segments into **multiple frequency channels**.

|Direction|Channel Width|Max Throughput per Channel|
|---|---|---|
|**Downstream** (each channel)|24 MHz to 192 MHz wide|Approximately **1.6 Gbps**|
|**Upstream** (each channel)|6.4 MHz to 96 MHz wide|Approximately **1 Gbps**|

**Each upstream and downstream channel is itself a broadcast channel.** Frames transmitted on the downstream channel by the CMTS are received by **all** cable modems receiving that channel; since there is just a **single CMTS** transmitting into the downstream channel, however, there is **no multiple access problem** on the downstream direction!

The **upstream direction**, however, is more interesting and technically challenging, since **multiple** cable modems share the **same** upstream channel (frequency) to the CMTS, and thus collisions can potentially occur.

```
Fig -- DOCSIS Upstream / Downstream
────────────────────────────────────────
        CMTS (cable headend)
            │
   downstream channel i (broadcast)
            ▼
   ───────────────────────────────▶
      Residences w/ cable modems
   ───────────────────────────────
            ▲
     upstream channel j (shared)
            │
        CMTS (cable headend)

 Upstream ch. j split into minislots
 (TDM-like). CMTS sends a MAP frame
 downstream granting specific mini-
 slots to specific cable modems --
 eliminating upstream collisions.
```

![[Pasted image 20260722205438.png]]

### How the Upstream Channel Is Managed: TDM + Explicit Grants

Each **upstream channel** is divided into **intervals of time (TDM-like)**, each containing a sequence of **mini-slots**, during which cable modems can transmit to the CMTS. **The CMTS explicitly grants permission** to individual cable modems to transmit during specific mini-slots. It accomplishes this by sending a control message known as a **MAP frame** on a downstream channel — this specifies which cable modem (with data to send) can transmit during which mini-slot for the interval of time specified in the control message.

Since mini-slots are **explicitly allocated** to cable modems, **the CMTS can ensure there are no colliding transmissions during a mini-slot** — this is fundamentally a **channel-partitioning** approach (TDM) for actual data transmission.

### But How Does the CMTS Know Who Has Data to Send?

This is accomplished by having cable modems send **mini-slot-request frames** to the CMTS during a special set of interval mini-slots dedicated for this purpose. These mini-slot-request frames are transmitted in a **random access manner** and so **may collide with each other** — this is the **random-access** component of DOCSIS.

- A cable modem can **neither sense** whether the upstream channel is busy, **nor detect collisions** directly.
- Instead, the cable modem **infers** that its mini-slot-request frame experienced a collision if it does **not** receive a response in the next downstream control message.
- When a collision is inferred, a cable modem uses **binary exponential backoff** to defer the retransmission of its mini-slot-request frame to a future time slot.
- When there is little traffic on the upstream channel, a cable modem may actually transmit data frames directly during slots nominally assigned for mini-slot-request frames — avoiding having to wait for a mini-slot assignment in the common case.

### DOCSIS as a Composite Case Study

A cable access network thus serves as a **terrific example of multiple access protocols in action** — combining **FDM**, **TDM**, **random access**, and **centrally allocated time slots**, all within one single network! This makes DOCSIS a uniquely useful case study for seeing how the three theoretical categories of multiple access protocols aren't mutually exclusive in real systems — they're frequently **layered together**, each handling the part of the coordination problem it's best suited for.

> **Analogy — A Restaurant's Reservation-and-Walk-in System:** Think of the CMTS like a restaurant host stand. The **downstream** direction is like the host announcing table assignments over a loudspeaker that every waiter can hear — there's only one host, so no coordination problem there (no multiple access issue). The **upstream** request phase is like walk-in customers all shouting their party size to the host at once during a brief "check-in window" — if two shout at the same time, the host can't understand either (a collision), so a customer whose shout wasn't acknowledged simply waits a random amount of time and tries shouting again (binary exponential backoff). Once the host _does_ hear a party clearly, they explicitly assign that party a specific table and time slot (a TDM-like mini-slot grant) — guaranteeing no further collisions for the actual seating (transmission) itself.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Broadcast channels mean everyone hears everything**|Since a broadcast channel delivers a copy of every transmitted frame to every attached node, any node — including a malicious one — can potentially eavesdrop on all traffic on that channel, or attempt to identify traffic patterns from legitimate nodes|Encryption above the link layer (or WPA-style link-layer encryption on wireless) is essential on broadcast media, since the multiple-access protocol itself provides zero confidentiality — it only arbitrates who gets to transmit, not who can listen|
|**Random access protocols are vulnerable to deliberate jamming/flooding**|An attacker can deliberately transmit continuously (or with a very aggressive retransmission policy) on a shared broadcast channel to induce repeated collisions, effectively performing a **denial-of-service attack** against legitimate nodes trying to use ALOHA/CSMA-style access|Detecting anomalous collision rates, or falling back to explicitly scheduled channel-partitioning/taking-turns access (which is far more resistant to greedy or malicious nodes) can mitigate this, though it trades away random access's simplicity and decentralization|
|**Taking-turns protocols have single points of control**|In the polling protocol, compromising or disabling the controller node takes down the entire channel — a highly attractive, concentrated target. In token-passing, a malicious node that captures the token and never releases it can stall the entire network|Controller redundancy/failover for polling protocols, and token-recovery/timeout procedures for token-passing protocols, are necessary defenses against both accidental failure and deliberate misbehavior by a compromised node|
|**CSMA/CD's collision-detection assumption can be exploited**|A malicious adapter that ignores the CSMA/CD "listen before speaking" and "stop on collision" rules (i.e., behaves like a boorish, non-compliant node) can dominate a shared channel, since well-behaved nodes will keep backing off politely while the misbehaving node keeps transmitting|Because CSMA/CD's fairness properties depend entirely on **voluntary compliance** by all adapters, physical/administrative control over which devices are permitted on a shared Ethernet segment matters — this is one reason switched (point-to-point) Ethernet has become far more common than shared broadcast Ethernet segments|
|**DOCSIS's request-phase collisions are inferable by an attacker**|Since a cable modem infers a collision merely from the _absence_ of a MAP grant, an attacker able to jam or spoof the downstream control channel could cause legitimate modems to falsely believe collisions occurred, forcing needless exponential backoff and degrading service (a denial-of-service vector)|Authentication and integrity protection of the downstream MAP control messages (as specified within DOCSIS's own security extensions) helps ensure cable modems aren't misled by forged or corrupted control messages|

---

## Questions I Still Have

- [ ] The text derives slotted ALOHA's maximum efficiency as exactly 1/e ≈ 0.37 — what's the intuitive (not just algebraic) reason this specific constant, e, falls out of the limit as N approaches infinity?
- [ ] For CSMA/CD's binary exponential backoff, why is the cap specifically set at n=10 collisions (rather than some other number) before the backoff range stops growing — is there a specific trade-off being balanced at that threshold?
- [ ] The efficiency formula for CSMA/CD (1 / (1 + 5·d_prop/d_trans)) has a "5" constant in it — where does that specific factor of 5 come from in the underlying derivation?
- [ ] Since token-passing is described as "decentralized and highly efficient," why isn't it more dominant today compared to CSMA/CD-based Ethernet or switched Ethernet, given its efficiency advantages and lack of collisions?
- [ ] For DOCSIS, when a cable modem infers a collision purely from not receiving a MAP grant (since it can't directly sense the channel or detect collisions), how does it distinguish between "my request collided" versus "my request was simply lost due to noise/an unrelated failure" — does the recovery procedure treat these cases the same way?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Broadcast link**|A link on which multiple sending and receiving nodes all connect to the same, shared channel; any one node's transmission is received as a copy by all other nodes|
|**Multiple access problem**|The general challenge of coordinating which node gets to transmit into a shared broadcast channel, and when|
|**Multiple access protocol**|A protocol by which nodes regulate their transmission into a shared broadcast channel; falls into channel partitioning, random access, or taking-turns categories|
|**Collision**|The event where two or more nodes transmit into a broadcast channel at the same time, causing all involved frames to become unusable/lost at the receivers|
|**Channel partitioning protocols**|Multiple access protocols (TDM, FDM, CDMA) that statically divide a channel's resources (time, frequency, or code) among nodes, avoiding collisions but limiting a lone active node's rate|
|**TDM (Time-Division Multiplexing)**|Divides channel time into repeating frames of N time slots, one dedicated per node|
|**FDM (Frequency-Division Multiplexing)**|Divides the channel's bandwidth into N frequency bands, one dedicated per node|
|**CDMA (Code Division Multiple Access)**|Assigns each node a unique code, allowing multiple nodes to transmit simultaneously and be decoded separately by their receivers|
|**Random access protocols**|Multiple access protocols (ALOHA, CSMA, CSMA/CD) in which nodes transmit at the full channel rate and use random retransmission delays to resolve collisions|
|**Slotted ALOHA**|A random access protocol with synchronized time slots; nodes transmit fresh frames at the next slot boundary and retransmit collided frames with probability p per slot|
|**Pure (unslotted) ALOHA**|A fully decentralized, unsynchronized version of ALOHA in which nodes transmit immediately upon a frame's arrival, with maximum efficiency exactly half that of slotted ALOHA|
|**Efficiency (of a multiple access protocol)**|The long-run fraction of successful/collision-free transmission time under heavy, sustained load from many active nodes|
|**Carrier sensing**|Listening to the channel before transmitting, and waiting if the channel is currently busy|
|**Collision detection**|Monitoring the channel while transmitting, and aborting transmission immediately if another node's signal is detected|
|**CSMA (Carrier Sense Multiple Access)**|A random access protocol family in which nodes sense the channel before transmitting|
|**CSMA/CD**|CSMA extended with collision detection, allowing nodes to abort a doomed transmission early rather than sending the entire frame|
|**Channel propagation delay**|The time it takes a signal to travel from one node to another across a broadcast channel; the key reason collisions can still occur even under CSMA|
|**Binary exponential backoff**|An algorithm (used in Ethernet and DOCSIS) where the range of possible random wait times after a collision grows exponentially, 2ⁿ, with the number n of consecutive collisions experienced, capped at n=10 for Ethernet|
|**Taking-turns protocols**|Multiple access protocols (polling, token-passing) that explicitly coordinate node access in turn, achieving near-R/M throughput per active node without collisions|
|**Polling protocol**|A taking-turns protocol where a designated controller node polls each other node in round-robin fashion, granting transmission permission|
|**Token-passing protocol**|A decentralized taking-turns protocol where a special token frame circulates among nodes in fixed order; a node may only transmit while holding the token|
|**DOCSIS**|The link-layer protocol standard for cable Internet access, combining FDM (splitting upstream/downstream), TDM (mini-slot grants for data), and random access (mini-slot-request collisions) within a single network|
|**CMTS (Cable Modem Termination System)**|The headend device in a cable access network that communicates with residential cable modems over downstream and upstream channels|
|**MAP frame**|A DOCSIS downstream control message from the CMTS specifying which cable modem may transmit during which upstream mini-slot|

---

## Related Concepts

---

→ Next: [[SWITCHED LANS]]