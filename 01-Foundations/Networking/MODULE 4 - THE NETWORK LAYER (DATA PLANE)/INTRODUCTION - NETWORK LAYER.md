---
title: INTRODUCTION - NETWORK LAYER
date: 2026-06-28
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 4.1 Overview of the Network Layer

> **One-Line Summary:** The network layer is the one piece of the protocol stack that lives inside _every single device_ on the path — not just the end-system hosts, like the transport and application layers — and its entire job splits cleanly into two pieces: a fast, local **data plane** that just forwards each arriving packet out the correct link, and a slower, network-wide **control plane** that figures out what those forwarding decisions should actually be — a split that modern Software-Defined Networking makes architecturally explicit by physically relocating the control plane off the router entirely.

---

## Core Idea: A Layer That Lives Everywhere

Chapter 3 established that the transport layer provides process-to-process communication by leaning on the network layer's **host-to-host communication service** — without the transport layer ever needing to know _how_ that service is actually implemented underneath. This chapter (and the next) lift the hood on exactly that question.

Here's the structural fact that makes the network layer fundamentally different from everything studied in Chapters 2 and 3: **there is a piece of the network layer inside every host _and_ every router in the network** — not just at the two communicating endpoints. The transport and application layers only ever run at end systems; routers in the middle of the network don't run a web browser or a TCP stack. But every router absolutely must run network-layer logic, since routing packets between hosts is the network layer's entire reason for existing. This is precisely why network-layer protocols are widely considered among the most challenging — and most interesting — in the whole protocol stack.

Because there's so much ground to cover, the network layer's discussion is split across two full chapters, organized around a single foundational distinction: the network layer decomposes into two interacting parts, the **data plane** and the **control plane**.

|Plane|Scope of This Discussion|
|---|---|
|**Data plane**|Covered in **this chapter (4)** — the _per-router_ functions: how a single arriving datagram on one input link gets forwarded out one particular output link. Covers both traditional IP forwarding (based on destination address) and generalized forwarding (based on values from several different header fields). Also where IPv4 and IPv6 addressing get studied in depth.|
|**Control plane**|Covered in **Chapter 5** — the _network-wide_ logic that determines the end-to-end path a datagram actually takes from source to destination, across many routers. Covers routing algorithms and real, widely-deployed routing protocols like OSPF and BGP.|

Traditionally, a single router has implemented _both_ of these functions monolithically — the routing logic and the forwarding logic, bundled together inside the same physical box. **Software-Defined Networking (SDN)** explicitly pulls these two functions apart, implementing the control-plane logic as a separate service — typically running on a remote "controller" — while the router itself is left to handle nothing but forwarding. SDN controllers get their own deep dive in Chapter 5, but this data-plane/control-plane distinction is worth holding onto throughout everything that follows in _this_ chapter too — it's the single idea that structures how the rest of the network layer's role makes sense.

### Worked Example: Tracing a Packet From H1 to H2

Picture a simple network: two hosts, H1 and H2, with a handful of routers — including R1 (nearest H1) and R2 (nearest H2) — somewhere on the path between them.

![[Figure 4.1.png]]

Suppose H1's transport layer hands the network layer a 1,000-byte HTTP response segment, destined for H2. Concretely, here's what happens at each hop:

```
  H1 (full stack)                                          H2 (full stack)
  ───────────────                                          ───────────────
  Application                                               Application
  Transport     ── 1,000-byte segment ──┐                   Transport     ▲
  Network       ◄── encapsulates segment │                  Network       │
       │            into a DATAGRAM      │                       │  extracts segment
  Link/Physical       (adds header)      │                  Link/Physical │ from datagram,
       │                                  ▼                       ▲       │ hands up to
       ▼                                                          │       │ transport layer
   sent to R1                                                 received from R2
       │                                                          ▲
       ▼                                                          │
   ┌─────────────────────┐                              ┌─────────────────────┐
   │   Router R1          │                              │   Router R2          │
   │   (truncated stack)  │   ── many hops, many ──►     │   (truncated stack)  │
   │   Network            │      routers in between      │   Network            │
   │   Link / Physical    │                              │   Link / Physical    │
   └─────────────────────┘                              └─────────────────────┘
       no Application or Transport layer running here — routers only run
       up through the Network layer; they never run a browser or a TCP stack
```

|Step|What Happens|
|---|---|
|**At H1**|The network layer takes the segment handed down from the transport layer, **encapsulates** it inside a datagram (adding a network-layer header), and sends the resulting datagram to nearby router R1|
|**At R1, and every intervening router**|Each router's **data plane** examines the arriving datagram and forwards it out the correct link toward the next hop — this is the _only_ network-layer job a router performs|
|**At H2**|The network layer receives the datagrams arriving from nearby router R2, **extracts** the original transport-layer segment, and delivers it up to H2's transport layer|

The **data plane's** job, at every single router along that path, is just: take what arrived on this input link, send it out the right output link. The **control plane's** job is the much bigger-picture one: making sure all of those individual, local forwarding decisions — taken together, across every router on the path — actually add up to a coherent end-to-end route from H1 all the way to H2.

---

## 4.1.1 Forwarding and Routing: The Data and Control Planes

The network layer's primary role sounds deceptively simple — move packets from a sending host to a receiving host. Achieving that requires two distinct network-layer functions.

### Forwarding

**Forwarding** is the action a router takes the instant a packet arrives at one of its input links: the router must move that packet to the appropriate output link. For example, a packet arriving at Router R1 (sent from H1) must be forwarded onward, toward whichever next router lies on the path to H2.

Forwarding is the most common — and most important — function implemented in the data plane, but it isn't the _only_ possible one. In the more general case (covered later, in Section 4.4), a packet might instead be **blocked** from exiting a router entirely (say, if it originated at a known malicious sending host, or is destined for a forbidden destination), or could even be **duplicated** and sent out over _multiple_ outgoing links at once.

### Routing

**Routing** is the network layer's job of determining the actual route, or path, that packets take as they travel all the way from a sender to a receiver. The algorithms that compute these paths are called **routing algorithms** — a routing algorithm is exactly what would determine, say, the specific sequence of routers a packet travels through getting from H1 to H2. Routing is implemented in the **control plane** of the network layer.

### Why the Book Insists on Distinguishing These Two Terms

"Forwarding" and "routing" are frequently used interchangeably by other authors discussing the network layer — but it's worth being precise, because the two operate on genuinely different timescales and in genuinely different places:

||Forwarding|Routing|
|---|---|---|
|**Scope**|Router-local — the action of moving a packet from one specific input interface to the appropriate output interface|Network-wide — the process that determines the end-to-end path packets take from source all the way to destination|
|**Timescale**|Very short — typically a few **nanoseconds**|Much longer — typically **seconds**|
|**Typical implementation**|Hardware|Software|

### The Driving Analogy

Recall the earlier driving analogy (Section 1.3.1): a road trip from Pennsylvania to Florida.

> **Worked Example — One Interchange, Many Interchanges:** Suppose the driver's route takes them through an interchange near Allentown, PA, then later through one outside Richmond, VA, then again near Savannah, GA, before finally reaching Orlando. **Forwarding** is the process of getting through any _one_ of those single interchanges — the car arrives at the Richmond interchange and, in that moment, must decide which single road to take to leave that interchange (a quick, entirely local decision made right there at the interchange). **Routing**, in contrast, is the process of _planning the entire trip_ before ever leaving Pennsylvania: consulting a map, comparing several possible full routes from PA to Orlando, and choosing one — where each chosen route is itself just a sequence of individual road segments, connected together at a whole series of interchanges. The driver does the "routing" exactly once, at the very start, on a long timescale; the car does "forwarding" repeatedly, once at each interchange, on a short timescale.

### The Forwarding Table: The Key Element Inside Every Router

A key element present inside every network router is its **forwarding table**. A router forwards an arriving packet by examining the value of one or more fields in that packet's header, and then using those header values to **index into** its forwarding table. Whatever value is stored in the forwarding-table entry matching those header values tells the router exactly which outgoing link interface that packet should be forwarded out of.

### Worked Example: A Packet With Header Value 0110

Suppose a router's forwarding table looks like this:

|Header Field Value|Output Link Interface|
|---|---|
|`0100`|3|
|`0110`|2|
|`0111`|2|
|`1001`|1|

A packet carrying header field value **`0110`** arrives at the router. The router indexes into its forwarding table using that exact value, finds the matching row, and determines that the correct output link interface for this particular packet is **interface 2**. The router then internally forwards the packet to interface 2 — and that's the entire forwarding decision, made in a few nanoseconds, with nothing more than a table lookup.

![[Figure 4.2.png]]

```
                       ┌─────────────────────┐
   Routing Algorithm   │   ROUTING ALGORITHM   │   ← Control plane
   (computes table) ──►│  computes the table   │     (longer timescale,
                       └──────────┬────────────┘      typically software)
   ─────────────────────────────────────────────────────────────────────
                       ┌──────────▼────────────┐
                       │  Local forwarding table │  ← Data plane
                       │  header │ output         │     (nanosecond
                       │  0100   │  3              │      timescale,
                       │  0110   │  2              │      typically hardware)
                       │  0111   │  2              │
                       │  1001   │  1              │
                       └──────────┬────────────┘
                                  │
   Header value of arriving         ┌──┐
   packet:  [0110]  ───────────────►│  │── interface 2 ──► out the door
                                     └──┘
                          (3 other interfaces, unused
                           by this particular packet)
```

---

## Control Plane: The Traditional Approach

A natural question follows immediately: how does a router's forwarding table actually get configured _in the first place_? Answering this is exactly where the interplay between forwarding (data plane) and routing (control plane) becomes visible.

In the traditional approach, a **routing algorithm runs inside every single router**, and both the forwarding function _and_ the routing function live together inside one router. The routing function in any one router needs to communicate with the routing functions running in _other_ routers, specifically to compute the correct values for its own forwarding table. This communication happens by exchanging **routing messages**, formatted according to a shared **routing protocol** (covered in depth in Sections 5.2 through 5.4).

> **A useful, if unrealistic, thought experiment:** Imagine a network where every forwarding table is configured _directly_ by human network operators, physically present at each router — in this hypothetical case, **no routing protocol would be needed at all.** Of course, those human operators would still need to coordinate with each other carefully, to make sure every forwarding table was set up so packets actually reach their intended destinations. This kind of manual configuration would also be far more error-prone, and far slower to adapt whenever the network's topology changed, than letting an automated routing protocol handle it. It's genuinely fortunate that real networks have _both_ a forwarding function and a routing function working together, rather than relying purely on either one alone.

---

## Control Plane: The SDN Approach

The approach just described — each router running its own routing component, which communicates with the routing components of _other_ routers — has historically been the standard approach taken by networking-equipment vendors, at least until fairly recently. Routers in this traditional setup keep both forwarding and routing functionality physically bundled inside the same box.

The fact that this _could_ instead be configured by humans directly suggests a broader insight: there might be other, fundamentally different ways to determine what goes into a data-plane forwarding table — and that's exactly what SDN does.

### How SDN Restructures the Same Picture

In the SDN approach, a **physically separate, remote controller** computes and distributes the forwarding tables used by every single router across the network. Crucially, the _data-plane_ components are completely identical to the traditional approach above — what's different is that the control-plane routing functionality has been pulled entirely off of the physical router. The routing device itself now performs **forwarding only**; a remote controller computes and distributes the forwarding tables that every router then simply looks values up in.

```
                       ┌───────────────────────────────────┐
                       │          REMOTE CONTROLLER          │  ← Control plane
                       │   (computes & distributes tables)   │     (can run in a
                       └─────┬──────┬──────┬──────┬─────────┘      remote data center)
       ─────────────────────│──────│──────│──────│──────────────────────
                             ▼      ▼      ▼      ▼
                       ┌─────────────────────────────────────┐
                       │  Local forwarding table (per router)  │  ← Data plane
                       │  header │ output                       │
                       │  0100   │  3                            │
                       │  0110   │  2                            │
                       │  0111   │  2                            │
                       │  1001   │  1                            │
                       └─────────────────────────────────────┘
                                  (same as before — only WHERE
                                   the table came from differs)
```

![[Figure 4.3.png]]

This remote controller might run in a remote data center with built-in reliability and redundancy, and could be managed by the ISP itself or by some third party entirely. The routers and the controller communicate by exchanging messages that contain forwarding tables and other routing-related information — structurally the same _kind_ of communication as before, just rerouted to a centralized point rather than peer-to-peer between routers.

This SDN approach to the control plane is precisely what's meant by **software-defined networking** — the network is "software-defined" because a controller, implemented in software, computes the forwarding tables and interacts directly with every router. Increasingly, these software implementations are also **open** — similar to open-source Linux kernel code — meaning the controlling code is publicly available, letting ISPs, networking researchers, and even students innovate and propose changes to the software governing network-layer functionality. SDN's control plane gets a full treatment later, in Section 5.5.

> **Analogy — A Building Manager vs. Every Office Managing Its Own Thermostat:** The traditional approach is like every office in a building independently deciding and adjusting its own heating settings, occasionally calling neighboring offices to coordinate. SDN is like a single building manager, sitting in a central office, who monitors the whole building and remotely sets every office's thermostat from one place — the physical heating equipment in each office (the "data plane") doesn't change at all, but _who decides the setting_ has moved to one centralized, more easily updated location.

---

## 4.1.2 Network Service Model

Before diving into the data plane's actual mechanics, it's worth stepping back and asking a broader question: what kind of _service_ might the network layer actually offer to whatever sits above it?

When the transport layer at a sending host hands a packet down to the network layer, several real questions arise: Can the transport layer rely on the network layer to actually deliver that packet to its destination? If several packets are sent, will they be delivered to the receiving host's transport layer in the same order they were sent? Will the time between two sequential packet transmissions equal the time between their eventual receptions? Will the network give any feedback at all about congestion along the way? The answers to these questions — and others like them — are determined entirely by the **network service model**, which defines the characteristics of end-to-end packet delivery between a sending host and a receiving host.

### A Menu of Possible Services (That the Internet Mostly Doesn't Offer)

|Possible Service|What It Would Guarantee|
|---|---|
|**Guaranteed delivery**|A packet sent by a source host will _eventually_ arrive at the destination host|
|**Guaranteed delivery with bounded delay**|Not just delivery, but delivery within a specified host-to-host delay bound — for example, within 100 msec|
|**In-order packet delivery**|Packets arrive at the destination in the exact order they were sent|
|**Guaranteed minimal bandwidth**|Emulates the behavior of a transmission link of a specified bit rate (e.g., 1 Mbps); as long as the sending host transmits below that rate, all packets are guaranteed to eventually be delivered|
|**Security**|The network layer could encrypt every datagram at the source and decrypt it at the destination, providing confidentiality for every transport-layer segment riding inside it|

This is only a partial list — countless further variations are possible. But here's the genuinely surprising fact about the real Internet:

### Worked Example: What "Best-Effort" Actually Means in Practice

**The Internet's network layer provides exactly one service: best-effort service.** Under best-effort service, packets are neither guaranteed to be received in the order they were sent, nor is their eventual delivery guaranteed _at all_. There's no guarantee on end-to-end delay, and no minimal-bandwidth guarantee either.

Concretely, suppose Host A sends three packets, P1, P2, and P3, ten milliseconds apart, to Host B:

|Service Model|What Could Actually Happen to P1, P2, P3|
|---|---|
|**Guaranteed delivery + 100 msec bound**|All three packets are guaranteed to arrive at B, and each one is guaranteed to arrive within 100 msec of being sent|
|**In-order delivery**|The three packets are guaranteed to arrive at B in the exact order P1, P2, P3 — never P2 before P1|
|**Best-effort (the Internet's actual model)**|P1 might arrive in 20 msec, P3 might arrive _before_ P2 because it happened to take a faster path, P2 might take 600 msec to arrive, or P2 might simply **never arrive at all** — and none of this is treated as an error condition by the network layer itself|

It might initially seem like "best-effort service" is just a euphemism for "no service at all" — after all, a network that delivered _zero_ packets to their destination would technically still satisfy the definition of best-effort delivery! (Nothing in that definition was actually violated by delivering nothing.)

### Other Architectures Aimed Higher

Other network architectures have defined, and actually implemented, service models that go well beyond the Internet's bare best-effort approach. The **ATM network architecture** [Black 1995], for instance, provides guaranteed in-order delivery, a bounded delay, _and_ a guaranteed minimal bandwidth — essentially the richer end of the menu above, built directly into the network itself. There have also been serious proposed extensions to the Internet's own architecture along these lines — for example, the **Intserv architecture** [RFC 1633] aims to provide end-to-end delay guarantees and congestion-free communication.

> **The interesting twist:** despite these genuinely well-developed, richer alternatives existing, the Internet's plain best-effort service model — combined with simply provisioning _adequate_ bandwidth, and pairing it with bandwidth-adaptive application-level protocols like the **DASH** multimedia streaming protocol (encountered back in Chapter 2) — has arguably proven to be more than "good enough" in practice. This bare-bones combination has still managed to enable an enormous range of demanding applications: streaming video services like Netflix, and real-time conferencing applications like Zoom and FaceTime, all running successfully on top of a network layer that, strictly speaking, promises almost nothing.

---

## An Overview of the Rest of This Chapter

Having now covered the network layer's high-level overview, the remaining sections of Chapter 4 work through the data plane piece by piece:

|Section|What It Covers|
|---|---|
|**4.2**|The internal hardware operations of a router — input and output packet processing, the router's internal switching mechanism, and packet queueing/scheduling|
|**4.3**|Traditional IP forwarding, where packets are forwarded to output ports based purely on their destination IP address — including IP addressing and the IPv4/IPv6 protocols in detail|
|**4.4**|More generalized forwarding, where packets may be forwarded based on a much larger set of header values (not only the destination IP address), and may be blocked, duplicated, or have header fields rewritten entirely under software control — a key building block of modern data planes, including SDN's|
|**4.5**|"Middleboxes" — devices that perform functions _beyond_ plain forwarding|

---

## Terminology Cleanup: Router, Switch, and Packet Switch

The terms **forwarding** and **switching** are often used interchangeably by computer-networking researchers and practitioners alike, and this book follows that same convention. Two _other_ commonly-interchanged terms, though, get used more carefully here:

|Term|This Book's Precise Meaning|
|---|---|
|**Packet switch**|A _general_ packet-switching device that transfers a packet from an input link interface to an output link interface, based on values found in the packet's header fields — a broad umbrella term covering both of the categories below|
|**Router**|A packet switch that bases its forwarding decision on header-field values found in the **network-layer datagram** (studied in depth in _this_ chapter)|
|**Switch**|A packet switch that bases its forwarding decision on header-field values found in the **link-layer frame** (studied in depth later, in Chapter 6)|

Recall from Section 1.5.2 that network-layer (Layer 3) packets are called **datagrams**, while link-layer (Layer 2) packets are called **frames** — and both datagram headers and frame headers carry their own distinct sets of fields. Because of this distinction, **routers are considered network-layer devices**, while **switches are considered link-layer devices**. Since this chapter's focus is squarely on the network layer, it will mostly use the term **"router"** in place of the more general "packet switch" going forward.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Best-effort service makes no guarantees, including about _who_ sent a packet**|Because the basic network layer provides no built-in authentication of source addresses, nothing structurally prevents a sender from putting a false source address on a packet — a foothold for spoofing-based attacks|Source-address verification has to be handled by mechanisms layered on top of (or alongside) the basic network layer, not assumed as a free guarantee from the service model itself|
|**Generalized forwarding (Section 4.4) lets packets be blocked, duplicated, or rewritten under software control**|The same mechanism that lets a network operator build a firewall or load balancer is, in principle, equally capable of being misused to silently duplicate, redirect, or drop someone else's traffic if that control plane is ever compromised|This dual-use nature is exactly why controlling _who_ can program a router's (or SDN controller's) forwarding behavior matters as much as the forwarding mechanism itself|
|**SDN concentrates control-plane logic in one remote controller**|A centralized controller is an attractive single point of attack — compromising it could let an attacker reprogram forwarding behavior across _every_ router in the network at once, rather than needing to compromise routers individually|The reliability and redundancy built into a well-designed SDN controller deployment needs to extend to its security posture too; centralizing control for manageability also centralizes risk if that single point isn't hardened accordingly|

---

## Questions I Still Have

- [ ] The forwarding table example used a single 4-bit header value to look up an output interface — in a real router with a full IP address as the lookup key, how does that lookup actually stay fast enough to happen in nanoseconds, given how much larger an IP address space is than a 4-bit field?
- [ ] In the traditional control-plane approach, routing protocol messages get exchanged directly between routers' routing components — does this mean routing protocol traffic itself travels over the same data plane it's helping to configure, or does it use some separate out-of-band channel?
- [ ] For SDN's remote controller: if the link between a router and its controller is itself disrupted, does the router fall back to its last-known forwarding table, or does it lose the ability to forward anything until contact is reestablished?
- [ ] The chapter says Intserv aims to provide end-to-end delay guarantees and congestion-free communication as an extension to the Internet architecture — given how dominant best-effort plus DASH-style adaptive streaming has become instead, did Intserv see any real-world deployment, or did it stay mostly theoretical?
- [ ] Now that "packet switch" is the umbrella term and "router" vs. "switch" are distinguished by which header (network vs. link layer) they read — are there real devices that do both simultaneously, or is that distinction always cleanly separable in practice?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Data plane**|The per-router functions of the network layer — how an arriving datagram on one input link gets forwarded to one output link|
|**Control plane**|The network-wide logic that determines the end-to-end path a datagram takes across multiple routers from source to destination|
|**Forwarding**|The router-local, nanosecond-timescale action of moving a packet from an input link interface to the correct output link interface|
|**Routing**|The network-wide, longer-timescale process of determining the end-to-end paths packets take from sender to receiver|
|**Forwarding table**|A table inside every router mapping header field values to output link interfaces, used to make forwarding decisions|
|**Routing algorithm**|An algorithm that computes the paths packets should take across the network; output feeds into routers' forwarding tables|
|**Routing protocol**|The shared message format/protocol routers use to exchange routing information with each other in the traditional control-plane approach|
|**Software-Defined Networking (SDN)**|An architecture that separates the control plane from the router entirely, implementing it instead as a remote, centralized controller service|
|**Remote controller**|In SDN, the centralized, physically separate component that computes and distributes forwarding tables to every router in the network|
|**Network service model**|The defined characteristics of end-to-end packet delivery (guarantees on delivery, order, delay, bandwidth) that the network layer provides to the layer above it|
|**Best-effort service**|The Internet's actual network-layer service model: no guarantee of delivery, ordering, delay bound, or minimum bandwidth|
|**ATM network architecture**|A network architecture that, unlike the Internet, provides guaranteed in-order delivery, bounded delay, and guaranteed minimal bandwidth|
|**Intserv**|A proposed extension to the Internet architecture aiming to provide end-to-end delay guarantees and congestion-free communication|
|**Packet switch**|The general term for any device that forwards a packet from an input link interface to an output link interface based on header field values|
|**Router**|A packet switch that makes its forwarding decision based on network-layer (datagram) header fields|
|**Switch**|A packet switch that makes its forwarding decision based on link-layer (frame) header fields|

---

## Related Concepts

---

→ Next: [[4.2 Inside a Router]]