---
title: OSPF
date: 2026-07-06
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 5.3 Intra-AS Routing in the Internet: OSPF

> **One-Line Summary:** Section 5.2's routing algorithms silently assumed a "flat," homogeneous network where every router speaks the same protocol and knows about every other router — a model that shatters at Internet scale (hundreds of millions of routers) and ignores that ISPs are independent businesses that want to run whatever routing logic they like internally while hiding their internals from the outside world; the Internet solves both problems by carving itself into **Autonomous Systems (ASes)**, each running its own **intra-AS routing protocol** internally — and **OSPF (Open Shortest Path First)**, the dominant intra-AS protocol, is simply Section 5.2's Link-State/Dijkstra algorithm made production-ready: authenticated flooding to _every_ router in the AS, periodic "trust-but-verify" re-advertisement even when nothing changed, and an optional two-level **area hierarchy** (a mandatory backbone Area 0 plus peripheral areas) that lets OSPF itself scale down the same way ASes let the whole Internet scale.

---

## Core Idea: Why "One Big Flat Network" Doesn't Work

Section 5.2 studied routing algorithms as if the network were simply **a collection of interconnected routers**, indistinguishable from one another, all executing the identical routing algorithm to compute paths through the entire network. In practice, this simplistic, homogeneous model breaks down for **two distinct reasons** — and it's worth being precise about which reason causes which problem, because the Internet's architecture solves them with two _different_ mechanisms.

### Reason 1: Scale

As the number of routers becomes large, the overhead involved in communicating, computing, and storing routing information becomes **prohibitive**. Today's Internet consists of **hundreds of millions of routers**. Storing routing information for every possible destination at each of these routers would require enormous amounts of memory, and the overhead of broadcasting connectivity and link-cost updates among _all_ of them would be huge. Worse: **a distance-vector algorithm that iterated among such a large number of routers would surely never converge** — DV's slow, gossip-based convergence (Section 5.2.2) simply does not scale to Internet size.

### Reason 2: Administrative Autonomy

The Internet is a network of **ISPs**, with each ISP consisting of its own network of routers. An ISP generally wants to operate its own network _as it pleases_ — running whatever routing algorithm it chooses internally, and potentially hiding aspects of its network's internal organization from the outside world. Ideally, an organization should be able to operate and administer its own network as it wishes, while _still_ being able to connect that network to other outside networks.

> **Analogy — Countries and Passports:** Think of the "flat network" model as imagining the entire world as one giant country with a single government making every local decision. Scale is the problem that no single government could realistically administer billions of individual streets. Administrative autonomy is the _separate_ problem that even if it _could_, each country still wants its own laws, its own internal police force, and the right to keep its internal affairs private — while still agreeing on common rules (passports, customs) for traffic crossing borders. The Internet's answer to both is the same as the real world's: partition into self-governing regions (**Autonomous Systems**), each free to run its own internal rules (**intra-AS routing**), connected by a shared, common protocol at the borders (**BGP**, inter-AS routing, Section 5.4).

### Solving Both Problems: Organizing Routers into Autonomous Systems (ASes)

Both of these problems are solved by organizing routers into **autonomous systems (ASes)**, with each AS consisting of a group of routers that are under the **same administrative control**. Often, the routers in an ISP, and the links that interconnect them, constitute a single AS. Some ISPs, however, partition their network into _multiple_ ASes. In particular, some tier-1 ISPs use one gigantic AS for their entire network, whereas others break up their ISP into tens of interconnected ASes.

An autonomous system is identified by its globally unique **autonomous system number (ASN)** [RFC 1930]. AS numbers, like IP addresses, are assigned by ICANN regional registries [ICANN 2025].

**Routers within the same AS all run the same routing algorithm and have information about each other.** The routing algorithm running _within_ an autonomous system is called an **intra-autonomous system routing protocol**.

```
THE INTERNET: PARTITIONED INTO AUTONOMOUS SYSTEMS (ASes)
────────────────────────────────────────────────────────

┌──────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│     AS 1 (ISP A)     │     │     AS 2 (ISP B)     │     │     AS 3 (ISP C)     │
│   routers running    │     │   routers running    │     │   routers running    │
│    SAME intra-AS     │     │    SAME intra-AS     │     │    SAME intra-AS     │
│   routing protocol   │     │   routing protocol   │     │   routing protocol   │
└──────────────────────┘     └──────────────────────┘     └──────────────────────┘

            ●────────────────────────────●────────────────────────────●
               BGP (inter-AS routing)

Each AS independently runs its OWN intra-AS (interior gateway)
routing protocol internally (e.g. OSPF) -- invisible to other ASes.
ASes connect to EACH OTHER via BGP, the Internet's inter-AS protocol.
```

Notice exactly how this maps back onto the two original problems:

|Problem|How the AS Model Solves It|
|---|---|
|**Scale**|A single intra-AS routing algorithm only ever needs to compute paths among the routers _inside one AS_ — a vastly smaller problem than all Internet routers at once. The Internet-wide problem is then handled separately, one layer up, by an inter-AS protocol (BGP) that only needs to reason about a much smaller number of AS-level "nodes," not individual routers|
|**Administrative autonomy**|Because each AS runs its _own_ intra-AS protocol independently, an ISP is completely free to choose whatever routing algorithm it likes internally — OSPF, IS-IS, or otherwise — without needing agreement from, or even visibility into, any other AS|

---

## Open Shortest Path First (OSPF)

OSPF routing, and its closely related cousin **IS-IS**, are widely used for intra-AS routing in the Internet. The **"Open"** in OSPF indicates that the routing protocol specification is publicly available — in contrast to, for example, Cisco's **EIGRP** protocol, which only recently became open [Savage 2015], after roughly 20 years as a Cisco-proprietary protocol. The most recent version, **OSPF version 2**, is defined in **[RFC 2328]**, a public document.

### What Kind of Algorithm Is OSPF?

**OSPF is a link-state protocol** that uses **flooding of link-state information** and a **Dijkstra least-cost path algorithm** — i.e., it is a direct, production implementation of exactly the LS algorithm studied in Section 5.2.1. With OSPF, each router constructs a **complete topological map** (that is, a graph) of the _entire_ autonomous system. Each router then locally runs Dijkstra's shortest-path algorithm to determine a shortest-path tree to all **subnets**, with itself as the root node.

### Who Sets the Link Costs?

Individual link costs are configured by **the network administrator** — OSPF itself does not mandate a policy for _how_ link weights are set (that's the administrator's job); it only provides the _mechanism_ (the protocol) for determining least-cost path routing given whatever set of link weights the administrator has chosen.

|Administrator's Choice|Resulting Behavior|
|---|---|
|Set all link costs to `1`|Achieves pure **minimum-hop routing** — the least-cost path is simply the path with the fewest links|
|Set weights **inversely proportional** to link capacity|**Discourages** traffic from using low-bandwidth links, since a low-capacity link ends up with a _higher_ cost and looks less attractive to Dijkstra's algorithm|

### Principles in Practice: Setting OSPF Link Weights — The Reversed Cause-and-Effect

The discussion of link-state routing so far has implicitly assumed a simple, forward cause-and-effect chain: link weights are set _first_, a routing algorithm like OSPF is run _second_, and traffic then flows according to the routing tables the algorithm computes _third_. In this viewpoint, link weights reflect the _cost_ of using a link — for example, if weights are inversely proportional to capacity, high-capacity links get smaller weights and thus look more attractive from a routing standpoint, and Dijkstra's algorithm serves to minimize overall cost.

**In practice, this cause-and-effect relationship is often _reversed_.** Network operators frequently configure link weights _in order to_ obtain routing paths that achieve specific traffic-engineering goals [Fortz 2000, Fortz 2002].

#### Worked Example: The Traffic Engineer's Actual Problem

Suppose a network operator has an _estimate_ of traffic flow entering the network at each ingress point and destined for each egress point. The operator may then want to put in place a specific routing of ingress-to-egress flows that **minimizes the maximum utilization** over all of the network's links.

But with a routing algorithm such as OSPF, the operator's main "knobs" for tuning the routing of flows through the network are _only_ the link weights. Thus, in order to achieve the goal of minimizing the maximum link utilization, the operator must find the **set of link weights that achieves this goal** — a direct reversal of the naive cause-and-effect relationship: the _desired routing of flows_ is known first, and the OSPF link weights must then be _reverse-engineered_ such that OSPF's own algorithm results in that desired routing of flows.

> **Analogy — Reverse-Engineering a Thermostat Schedule:** The naive view is like setting a thermostat's temperature dial and then observing what temperature the room reaches. The traffic-engineering reality is more like already knowing you want the room at exactly 68°F at 7am tomorrow, and having to work _backward_ to figure out what dial setting, combined with the room's known physics, will produce that exact outcome at that exact time. The "physics" here is Dijkstra's algorithm; the "dial" is the set of link weights; and the target isn't a temperature but a traffic-utilization pattern across the whole network.

---

## How OSPF Actually Operates: Broadcast, Not Just Neighbor-Gossip

This is the single most important operational contrast with the distance-vector world of Section 5.2.2:

**With OSPF, a router broadcasts routing information to _all_ other routers in the autonomous system — not just to its neighboring routers.** (Recall DV's philosophy: talk _only_ to directly attached neighbors, but tell them your full distance vector. OSPF's philosophy is the LS mirror image: talk to _everyone_, but only tell them about your own directly attached links.)

### When Does a Router Broadcast?

|Trigger|Behavior|
|---|---|
|**A link's state changes**|A router broadcasts link-state information whenever there is a change in a link's state — for example, a change in cost, or a change in up/down status|
|**Periodic re-advertisement**|A router _also_ broadcasts a link's state **periodically** — at least once every **30 minutes** — **even if the link's state has not changed at all**|

#### Why Re-advertise Something That Hasn't Changed?

RFC 2328 explains the rationale directly: _"this periodic updating of link state advertisements adds robustness to the link state algorithm."_ Think of it as a heartbeat rather than an update — periodic re-flooding guards against a lost or corrupted advertisement silently leaving some router's topology map permanently stale; even if nothing changed, hearing "still here, still cost 3" every 30 minutes lets the network self-heal from any missed or dropped message.

### OSPF Advertisements Ride Directly on IP

**OSPF advertisements are contained in OSPF messages that are carried directly by IP**, with an upper-layer protocol number of **89** for OSPF. This means OSPF sits directly on top of the network layer (unlike, say, RIP, which rides on top of UDP) — and as a consequence, **the OSPF protocol itself must implement functionality such as reliable message transfer and link-state broadcast**, since IP itself provides neither of those guarantees.

The OSPF protocol also checks that links are operational via a **HELLO** message that is sent to an attached neighbor, and allows an OSPF router to obtain a neighboring router's database of network-wide link state.

```
                    OSPF's POSITION IN THE PROTOCOL STACK
                    ──────────────────────────────────────

        ┌─────────────────────────────────────────────┐
        │              OSPF (protocol 89)               │
        │  - link-state advertisements (LSAs)           │
        │  - HELLO messages (neighbor liveness check)   │
        │  - reliable message transfer (self-implemented)│
        │  - link-state broadcast (self-implemented)    │
        └───────────────────────┬───────────────────────┘
                                 │  carried DIRECTLY by IP
                                 ▼
        ┌─────────────────────────────────────────────┐
        │                      IP                       │
        │        (no reliability guarantee here --      │
        │         that's why OSPF builds its own)       │
        └─────────────────────────────────────────────┘

  Contrast: RIP rides on top of UDP; OSPF rides directly on IP,
  and therefore has to reinvent (inside OSPF itself) the reliability
  functions a transport-layer protocol would normally provide.
```

---

## The Advances Embodied in OSPF

Beyond the core link-state-plus-Dijkstra mechanism, OSPF incorporates several features that go beyond the bare-bones algorithm of Section 5.2.1:

### Security

Exchanges between OSPF routers (for example, link-state updates) can be **authenticated**. With authentication, only trusted routers can participate in the OSPF protocol within an AS, thus preventing malicious intruders (or networking students taking their newfound knowledge out for a joyride) from injecting incorrect information into router tables. By default, OSPF packets between routers are **not** authenticated and could be forged.

|Authentication Type|How It Works|Security Level|
|---|---|---|
|**Simple authentication**|The same password is configured on each router. When a router sends an OSPF packet, it includes the password in **plaintext**|Not very secure — the password is visible to anyone who can observe the packet|
|**MD5 authentication**|Based on shared secret keys configured in all the routers. For each OSPF packet it sends, the router computes the **MD5 hash** of the OSPF packet content appended with the secret key, then includes the resulting hash value in the packet. The receiving router computes an MD5 hash of the packet using the preconfigured secret key, and compares it with the hash value the packet carries, thus verifying the packet's authenticity. **Sequence numbers** are also used with MD5 authentication to protect against replay attacks|Considerably stronger — the plaintext password itself is never transmitted|

> (See Chapter 8 for a fuller discussion of MD5 and authentication in general, and the discussion of message authentication codes in Chapter 8 for more on the hashing technique used here.)

### Multiple Same-Cost Paths

When multiple paths to a destination have the **same** cost, OSPF allows multiple paths to be used (that is, a single path need not be chosen for carrying _all_ traffic when multiple equal-cost paths exist). This is a direct, practical answer to a question left implicit in Section 5.2's presentation of Dijkstra's algorithm — real deployments don't have to arbitrarily break ties the way the pseudocode's `min` operation might suggest; they can spread load across all equally-good options.

### Integrated Support for Unicast and Multicast Routing

**Multicast OSPF (MOSPF)** [RFC 1584] provides simple extensions to OSPF to support multicast routing. MOSPF uses the _existing_ OSPF link-state database and adds a _new type_ of link-state advertisement to the existing OSPF link-state broadcast mechanism — it doesn't require a parallel, separate infrastructure, just an extension bolted onto the machinery already built for unicast.

### Support for Hierarchy Within a Single AS

This is the single most architecturally significant "advance," because it directly echoes the same scale-driven partitioning logic that motivated ASes in the first place — but applied _one level down_, inside a single AS.

**An OSPF autonomous system can be configured hierarchically into areas.** Each area runs its own OSPF link-state routing algorithm, with each router in an area broadcasting its link state to _all other routers in that area_ — link-state flooding is scoped to the area, not the whole AS. This is exactly how OSPF avoids re-encountering the same scale problem that motivated ASes to begin with: instead of every router in a (potentially huge) AS flooding link-state to every _other_ router in that entire AS, flooding is contained within much smaller areas.

#### The Two Router Roles

|Role|Responsibility|
|---|---|
|**Non-border routers**|Ordinary routers living entirely inside a single area, running OSPF link-state routing purely within that area's boundaries|
|**Area border routers (ABRs)**|Within each area, **one or more** routers are responsible specifically for routing packets **outside the area**. One or more ABRs are configured per area|

#### The Backbone Area

Exactly **one** OSPF area in the AS is configured to be the **backbone area**. The primary role of the backbone is to route traffic **between** the other areas in the AS. The backbone **always** contains all of the area border routers in the AS, and may also contain non-border routers as well.

#### Inter-Area Routing: A Three-Stage Journey

Inter-area routing within the AS requires that a packet be:

1. First routed to an area border router (**intra-area routing**),
2. Then routed through the backbone to the area border router that is in the destination area,
3. Then routed to the final destination (again, **intra-area routing**, but now within the destination area).

```
OSPF HIERARCHY WITHIN A SINGLE AUTONOMOUS SYSTEM
────────────────────────────────────────────────

         ┌────────────────────────────────────────────────────┐
         │               BACKBONE AREA (Area 0)               │
         │                                                    │
         │            ABR-1 ─────────────── ABR-2             │
         │                                                    │
         │         Contains ALL area border routers;          │
         │            routes traffic BETWEEN areas            │
         └──────┬───────────────────────────────────────┬─────┘
                ▼                                       ▼

┌──────────────────────────────┐        ┌──────────────────────────────┐
│            AREA 1            │        │            AREA 2            │
│                              │        │                              │
│        R1 ── R2 ── R3        │        │        R4 ── R5 ── R6        │
│                              │        │                              │
│   (intra-area LS flooding    │        │   (intra-area LS flooding    │
│    stays within this area)   │        │    stays within this area)   │
└──────────────────────────────┘        └──────────────────────────────┘

Routing a packet from R1 (Area 1) to R6 (Area 2):
  R1 -> R2 -> R3 -> ABR-1 -> [backbone] -> ABR-2 -> R4 -> R5 -> R6
       (intra-area)      (inter-area, via backbone)   (intra-area)
```

> **Analogy — A Country's Regional Highway System:** Think of each OSPF area as a state, with the backbone area as the interstate highway network connecting states together. Local traffic within a state (non-border routers) uses state roads (intra-area routing) and never needs to know the detailed street layout of any _other_ state. To travel between states, you first drive to an on-ramp (an area border router), take the interstate (the backbone) to the off-ramp nearest your destination state (the destination area's ABR), and then use _that_ state's local roads to reach your final address. No driver needs a map of the entire country's every street — only their own state's roads, plus the interstate system connecting the on/off-ramps.

> OSPF is a relatively complex protocol, and this coverage has necessarily been brief; [Huitema 1998; Moy 1998; RFC 2328] provide additional details.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**By default, OSPF packets are unauthenticated and could be forged**|Without authentication enabled, any device that can reach the OSPF broadcast domain could inject forged link-state advertisements, potentially poisoning routing tables across the entire area or AS|Enabling authentication (at minimum simple, ideally MD5) is not optional hardening — it is the only thing standing between "any router in the broadcast domain" and "any device on the wire" being able to participate in routing|
|**Simple authentication sends the password in plaintext**|An attacker with any visibility into the wire (a compromised switch port, a tap, a misconfigured span port) can trivially observe the plaintext password and then impersonate a legitimate router indefinitely|MD5 authentication should be preferred in any real deployment — the secret key itself is never transmitted, only a hash of packet content plus key|
|**MD5 authentication uses sequence numbers specifically to block replay attacks**|Without sequence-number checking, an attacker could simply capture and later re-transmit ("replay") a previously valid, correctly-authenticated OSPF packet to reintroduce stale routing information|Sequence numbers must be checked and enforced on every authenticated packet, not just the hash itself — authentication without replay protection only proves a message was _once_ legitimate, not that it still is|
|**Periodic (30-minute) re-advertisement is a robustness feature, but also a predictable signal**|An attacker monitoring OSPF traffic can use the predictable ~30-minute heartbeat cadence to infer topology stability, or specifically time an injected forged advertisement to arrive just after a legitimate periodic refresh (when it might sit "trusted" the longest before the next correction)|This is a genuine tension: the same periodicity that adds robustness against lost messages also creates a predictable window; authentication (above) is what actually closes that window, not obscurity of timing|
|**Area border routers (ABRs) and the backbone area are architecturally privileged chokepoints**|Because _all_ inter-area traffic must transit through an ABR and the backbone, compromising a single ABR (or backbone-area router) gives an attacker leverage over traffic between multiple areas at once — a much higher-value target than an ordinary non-border router|ABRs deserve disproportionate hardening and monitoring relative to ordinary intra-area routers, precisely because the hierarchy that makes OSPF scale also concentrates inter-area trust onto a small number of devices|

---

## Questions I Still Have

- [ ] The chapter notes OSPF advertisements ride directly on IP (protocol 89) rather than on top of a transport protocol like UDP or TCP — given that OSPF then has to reimplement reliable message transfer itself, what specific mechanism does OSPF use for that reliability (acknowledgments, retransmission timers), and how does it compare to what TCP already provides?
- [ ] With areas and a mandatory backbone, what actually happens if the backbone area itself becomes partitioned (e.g., a link failure splits Area 0 into two disconnected pieces) — does inter-area routing between the affected areas simply fail until the partition is repaired, or is there a virtual-link mechanism to route around it?
- [ ] For "multiple same-cost paths" — when OSPF spreads traffic across several equal-cost paths, is the actual packet-by-packet (or flow-by-flow) selection among those paths handled by OSPF itself, or is that a separate mechanism (like ECMP hashing) sitting below OSPF's routing-table computation?
- [ ] MOSPF adds multicast support by extending the existing link-state database with a new advertisement type — does this mean every router in the OSPF area (even ones with no interest in the multicast group) still has to process and store the multicast-related link-state advertisements, or is that flooding somehow scoped only to interested routers?
- [ ] Given that the "Principles in Practice" sidebar describes reverse-engineering link weights to achieve a desired traffic-engineering outcome — is this weight-computation process (Fortz 2000, Fortz 2002) something operators actually run as an automated optimization tool today, or is it still largely manual trial-and-error in production networks?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Autonomous System (AS)**|A group of routers under the same administrative control, all running the same intra-AS routing protocol; identified by a globally unique AS number (ASN)|
|**Autonomous System Number (ASN)**|A globally unique identifier for an AS, assigned by ICANN regional registries, analogous to how IP address blocks are assigned|
|**Intra-autonomous system routing protocol**|The routing algorithm run by all routers within a single AS (e.g., OSPF, IS-IS, RIP, EIGRP) — as opposed to the inter-AS protocol (BGP) that connects ASes to each other|
|**OSPF (Open Shortest Path First)**|A widely-used, publicly-specified (hence "Open") intra-AS link-state routing protocol; the current version is OSPF version 2, defined in RFC 2328|
|**IS-IS**|A routing protocol closely related to OSPF, also widely used for intra-AS routing in the Internet|
|**EIGRP**|Cisco's intra-AS routing protocol; historically proprietary for roughly 20 years before becoming open in 2015|
|**Link-state broadcast (OSPF)**|OSPF's mechanism for flooding a router's link-state information to all other routers within the same area (or AS, if no areas are configured)|
|**HELLO message**|An OSPF message sent to an attached neighbor to check that a link is operational and to obtain the neighbor's link-state database|
|**Protocol 89**|The IP upper-layer protocol number identifying OSPF messages; OSPF is carried directly by IP rather than riding on UDP or TCP|
|**Simple authentication (OSPF)**|An OSPF authentication method using a shared, plaintext password configured on each router — not very secure|
|**MD5 authentication (OSPF)**|An OSPF authentication method using a shared secret key and an MD5 hash of each packet's content plus the key; combined with sequence numbers to prevent replay attacks|
|**Area (OSPF)**|A hierarchical subdivision of an OSPF AS; each area runs its own OSPF link-state algorithm, with link-state flooding scoped to within that area|
|**Non-border router**|An OSPF router that lives entirely within a single area and does not route packets outside that area|
|**Area border router (ABR)**|An OSPF router responsible for routing packets outside its area; every OSPF area has one or more ABRs|
|**Backbone area**|The single OSPF area (Area 0) configured to route traffic between all other areas in the AS; always contains every area border router, and may contain non-border routers too|
|**Inter-area routing**|The three-stage process of routing a packet between two different OSPF areas: intra-area to the source area's ABR, across the backbone to the destination area's ABR, then intra-area to the final destination|
|**MOSPF (Multicast OSPF)**|An extension to OSPF (RFC 1584) providing multicast routing support by adding a new link-state advertisement type to the existing OSPF flooding mechanism|

---

## Related Concepts

- [[5.2 Routing Algorithms]] — OSPF is a direct, production-grade implementation of this section's Link-State (LS) algorithm and Dijkstra's shortest-path computation, applied within the scope of a single AS
- [[5.4 Inter-AS Routing: BGP]] — the protocol that connects separately-administered ASes to each other, picking up exactly where intra-AS routing (this note) leaves off

---

→ Next: [[5.4 Inter-AS Routing: BGP]]