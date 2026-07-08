---
title: ROUTING AMONG ISPS - BGP
date: 2026-07-07
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 5.4 Routing Among the ISPs: BGP

> **One-Line Summary:** Section 5.3's OSPF only ever answers "how do I get to a destination _inside my own AS_?" — for everything outside it, the Internet relies on a single, universal glue protocol, **BGP (Border Gateway Protocol)**, which lets every AS on Earth learn which prefixes exist and through which sequence of other ASes they can be reached, then lets each router locally pick the "best" of potentially many such routes using a four-rule elimination process that blends **policy** (local preference, set by an AS's own administrator), **path length** (AS-PATH), and **selfish cost-minimization** (hot-potato routing) — and because BGP is fundamentally policy-driven rather than performance-driven, it also becomes the mechanism that lets ISPs enforce business relationships (who's a customer, who's a peer), lets CDNs and DNS root servers appear to be "everywhere at once" via IP-anycast, and is literally the last piece that turns a newly-registered domain name into something the rest of the Internet can actually route packets to.

---

## Core Idea: OSPF Handles "Inside," BGP Handles "Outside"

As we've learned, for destinations that are **within the same AS**, the entries in a router's forwarding table are determined by the AS's intra-AS routing protocol (Section 5.3's OSPF, for example). But what about destinations that are **outside** the AS? This is precisely where **BGP** comes to the rescue.

BGP — the **Border Gateway Protocol** [RFC 4271] — provides each AS a means to obtain prefix reachability information from neighboring ASes, and to propagate that reachability information to _all_ routers internal to the AS. BGP is decentralized and asynchronous, very much in the spirit of the distance-vector algorithm studied in Section 5.2.2 — but as we'll see, it is a considerably more policy-laden, complicated beast than DV.

> **Why This Section Matters More Than Its Length Suggests:** BGP glues together tens of thousands of independently-administered ISPs into a single global Internet. Arguably, only IP itself rivals BGP in terms of sheer foundational importance to how the Internet actually works — every single packet that crosses an AS boundary anywhere on Earth got there because some router, somewhere, made a BGP decision.

### BGP Doesn't Route to Addresses — It Routes to Prefixes

In BGP, packets are **not** routed to a specific destination address. Instead, they are routed to **CIDRized prefixes**, with each prefix representing a subnet or a collection of subnets. In the world of BGP, a destination may take the form `138.16.68/22`, which for this example includes 1,024 IP addresses. Thus, a router's forwarding table will have entries of the form **`(x, I)`**, where `x` is a prefix (such as `138.16.68/22`) and `I` is an interface number for one of the router's interfaces.

### The Two Jobs BGP Does, as an Inter-AS Routing Protocol

|Job|What It Means|
|---|---|
|**1. Obtain prefix reachability information from neighboring ASes**|BGP allows each subnet to advertise its existence to the rest of the Internet. A subnet screams, _"I exist and I am here,"_ and BGP makes sure all the routers in the Internet know about this subnet. If it weren't for BGP, each subnet would be an isolated island — alone, unknown, and unreachable by the rest of the Internet|
|**2. Determine the "best" routes to the prefixes**|A router may learn about two or more different routes to a specific prefix. To determine the best route, the router will locally run a **BGP route-selection procedure** (using the prefix reachability information it obtained via neighboring routers). The best route will be determined based on **policy** as well as the reachability information|

---

## 5.4.1 Advertising BGP Route Information

### Gateway Routers vs. Internal Routers

Consider a network with three autonomous systems: AS1, AS2, and AS3, where AS3 includes a subnet with prefix `x`. For each AS, each router is either a **gateway router** or an **internal router**. A **gateway router** is a router on the edge of an AS that directly connects to one or more routers in _other_ ASes. An **internal router** connects only to hosts and routers within its _own_ AS.

#### The High-Level Story

Let's consider the task of advertising reachability information for prefix `x` to all of the routers in this network. At a high level, this is straightforward: first, AS3 sends a BGP message to AS2, saying that `x` exists and is in AS3; let's denote this message as **"AS3 x."** Then AS2 sends a BGP message to AS1, saying that `x` exists and that you can get to `x` by first passing through AS2 and then going to AS3; let's denote that message **"AS2 AS3 x."** In this manner, each of the autonomous systems will not only learn about the _existence_ of `x`, but also learn about a _path_ of autonomous systems that leads to `x`.

> Although the discussion above should get the general idea across, it is _not_ precise, in the sense that autonomous systems do not actually send messages to each other — instead, **routers** do.

### The Real Mechanism: eBGP and iBGP

In BGP, **pairs of routers exchange routing information over semi-permanent TCP connections using port 179**. Each such TCP connection, along with all the BGP messages sent over it, is called a **BGP connection**. Furthermore:

|Connection Type|Spans|
|---|---|
|**External BGP (eBGP)**|A BGP connection that spans **two different ASes** — there is typically one eBGP connection for each link that directly connects gateway routers in different ASes|
|**Internal BGP (iBGP)**|A BGP session between routers within the **same AS** — typically, one BGP connection is configured for **each pair** of routers internal to an AS, creating a full **mesh** of TCP connections within each AS|

> Note that iBGP connections do not always correspond to physical links.

![[Pasted image 20260708214222.png]]

### Worked Example: Propagating Reachability for Prefix x

In order to propagate reachability information, **both iBGP and eBGP sessions are used.** Consider advertising the reachability information for prefix `x` to _all_ routers in AS1 and AS2:

```
BGP ROUTE PROPAGATION FOR PREFIX x (in AS3)
────────────────────────────────────────────

Step 1 (eBGP):  3a --"AS3 x"--> 2c
Step 2 (iBGP):  2c --"AS3 x"--> 2a (+other AS2 rtrs)
Step 3 (eBGP):  2a --"AS2 AS3 x"--> 1c
Step 4 (iBGP):  1c --"AS2 AS3 x"--> 1a,1b,1d

Result: every router in AS1 and AS2 now knows x
exists, reachable via AS-PATH "AS2 AS3".
```

Gateway router 3a first sends the eBGP message **"AS3 x"** to gateway router 2c. Gateway router 2c then sends the iBGP message **"AS3 x"** to _all_ of the other routers in AS2, including gateway router 2a. Gateway router 2a then sends the eBGP message **"AS2 AS3 x"** to gateway router 1c. Finally, gateway router 1c uses iBGP to send **"AS2 AS3 x"** to all the routers in AS1.

> **Analogy — A Relay Race Where Every Runner Also Shouts to Their Own Teammates:** Think of eBGP as the baton handoff _between_ teams (crossing an AS boundary) and iBGP as each team's runners shouting the news back to every other member of their own team the instant they receive the baton. The message itself grows a little at each cross-team handoff (the AS-PATH gets one AS longer), but within a team, every teammate hears the exact same message the frontrunner just received — nobody on the team is left in the dark, even though they never personally touched the baton.

### Real Networks Have Multiple Paths

In a real network, from a given router, there may be many different paths to a given destination, with each path passing through a different sequence of autonomous systems. For example, suppose there is an _additional_ physical link directly connecting router 1d in AS1 to router 3d in AS3. In this scenario, each router in AS1 becomes aware of **two** distinct BGP routes to prefix `x` — one via the AS-PATH "AS2 AS3," and one via the (shorter) AS-PATH "AS3" that bypasses AS2 entirely. Determining which of these routes is actually "best" is the subject of the rest of this section.

---

## 5.4.2 Determining the Best Routes

### The Anatomy of a BGP Route

Here, each **BGP route** is written as a list with three components: **NEXT-HOP; AS-PATH; destination prefix.** (In practice, a BGP route includes additional attributes, which are ignored here for simplicity.)

|Attribute|What It Captures|
|---|---|
|**AS-PATH**|The list of ASes through which the prefix advertisement has passed. As the ad passes through an AS, that AS adds its own ASN to the list. BGP routers also use AS-PATH to **detect and prevent looping advertisements**: if a router sees its own AS already listed in an incoming AS-PATH, it rejects the advertisement (since accepting it would create a loop)|
|**NEXT-HOP**|The critical link between inter-AS and intra-AS routing: this is the **IP address of the router interface that begins the AS-PATH.** Importantly, the NEXT-HOP attribute is an IP address of a router that does **not** belong to the receiving AS; however, the subnet containing this IP address directly attaches to the receiving AS|

#### Worked Example: The Two Routes to x, in Full

Continuing the scenario with the extra 1d–3d link, each router in AS1 becomes aware of exactly two BGP routes to prefix `x`:

```
IP address of leftmost interface of router 2a ; AS2 AS3 ; x
IP address of leftmost interface of router 3d ; AS3     ; x
```

---

## Hot Potato Routing

We are now finally in a position to talk about BGP routing algorithms in a precise manner. We'll begin with one of the simplest, **hot potato routing**.

Consider router 1b in this network. As just described, this router will learn about two possible BGP routes to prefix `x`. **In hot potato routing, the route chosen (from among all possible routes) is that route with the least cost to the NEXT-HOP router beginning that route.**

#### Worked Example: Router 1b's Decision

Router 1b would consult its intra-AS routing information to find the least-cost intra-AS path to NEXT-HOP router 2a and the least-cost intra-AS path to NEXT-HOP router 3d, then select the route with the smallest of these two least-cost paths. For example, suppose cost is defined as the number of links traversed:

```
Least cost from router 1b to router 2a = 2
Least cost from router 1b to router 3d = 3

Since 2 < 3, router 2a is SELECTED.
```

Router 1b would then consult its forwarding table (configured by its intra-AS algorithm) to find the interface `I` that is on the least-cost path to router 2a. It then adds `(x, I)` to its forwarding table.

### The Four-Step Process (Figure 5.11)

![[Pasted image 20260708215107.png]]

It is important to note that when adding an outside-AS prefix to a router's forwarding table, **both** the inter-AS routing protocol (BGP) _and_ the intra-AS routing protocol (e.g., OSPF) are used.

### Why "Hot Potato"?

The idea behind hot-potato routing is for router 1b to get packets **out of its AS as quickly as possible** (more specifically, at the least cost possible) — without worrying about the cost of the remaining portions of the path _outside_ its AS.

> **Analogy — Passing a Literally Hot Potato:** In the name "hot potato routing," a packet is analogous to a hot potato that is burning your hands. Because it's burning hot, you want to pass it off to another person (another AS) as quickly as possible — you don't particularly care _which_ person catches it next, or how far _they_ then have to carry it; you just want it off your own hands, immediately. Hot potato routing is thus a **selfish algorithm** — it tries to reduce the cost incurred within its own AS while completely ignoring the other components of the end-to-end cost that lie outside its AS.

Note that with hot-potato routing, **two routers in the same AS may choose different AS paths.** In this example, router 1b would send packets through AS2 to reach `x`. However, router 1d — sitting right next to the new 1d–3d link — would instead bypass AS2 entirely and send packets directly to AS3 to reach `x`.

---

## The Full Route-Selection Algorithm

In practice, BGP uses an algorithm that is more complicated than hot potato routing, but nevertheless _incorporates_ hot potato routing as one of its steps. For any given destination prefix, the input into BGP's route-selection algorithm is the set of all routes that have been learned and accepted for that prefix. If there is only one such route, BGP obviously selects it. If there are two or more routes to the same prefix, BGP sequentially invokes the following elimination rules until exactly one route remains:

|Step|Elimination Rule|Notes|
|---|---|---|
|**1**|Routes are assigned a **local preference** value as one of their attributes (in addition to AS-PATH and NEXT-HOP). The routes with the **highest** local preference value are selected|The local preference of a route could have been set by the router itself, or learned from another router in the same AS. This attribute is a **policy decision** left entirely up to the AS's own network administrator|
|**2**|From the remaining routes (all with the same highest local preference), the route with the **shortest AS-PATH** is selected|If this were the _only_ rule for route selection, BGP would effectively be a DV algorithm, where the distance metric is the number of AS hops rather than the number of router hops|
|**3**|From the remaining routes (all with the same highest local preference _and_ the same AS-PATH length), **hot potato routing** is used — the route with the closest NEXT-HOP router is selected|This is exactly the algorithm worked through above, now demoted to the _third_ tiebreaker rather than the primary decision rule|
|**4**|If more than one route still remains, the router uses **BGP identifiers** to select the route|See [Stewart 1999] for further detail|

### Worked Example: Router 1b, Revisited With the Full Algorithm

Let's reconsider router 1b. Recall there are exactly two BGP routes to prefix `x` — one that passes through AS2 (AS-PATH "AS2 AS3"), and one that bypasses AS2 (AS-PATH "AS3"). Recall also that if hot-potato routing alone were used, BGP would route packets through AS2 (since AS2's NEXT-HOP, 2a, was found to be closer than AS3's NEXT-HOP, 3d).

But in the _full_ route-selection algorithm, **Rule 2 is applied before Rule 3** — and Rule 2 causes BGP to select the route that **bypasses** AS2, since that route has the shorter AS-PATH ("AS3," length 1, versus "AS2 AS3," length 2). So we see that with the full route-selection algorithm, BGP is **no longer a purely selfish algorithm** — it first looks for routes with short AS paths (thereby likely reducing end-to-end delay for the _whole_ journey), and only falls back to pure self-interest (hot potato) as a last-resort tiebreaker.

> As noted above, BGP is the **de facto standard** for inter-AS routing for the Internet. To see the contents of various (large!) BGP routing tables extracted from routers around the world, see, e.g., routeviews.org — real BGP tables often contain over half a million routes.

---

## 5.4.3 IP-Anycast

BGP's route-selection algorithm turns out to have a second, entirely different use: it can implement an **IP-anycast** service [RFC 1546, RFC 7094], commonly used by the DNS system.

### The Motivation

Many applications want two things simultaneously:

1. To **replicate the same content** on servers in different, geographically dispersed locations
2. To have each user access that content from the **closest** such server

A CDN, for example, may replicate videos and other objects on servers in many different countries; the DNS system, similarly, replicates DNS records on DNS servers scattered all over the world. In both cases, we'd like to point a user to the "nearest" server holding the replicated content — and BGP's route-selection algorithm, entirely incidentally, provides an easy, natural mechanism for doing exactly this.

### How It Works

During an initial "IP-anycast configuration" stage, a CDN assigns the **exact same IP address** to each of its servers, and uses standard BGP to advertise this one IP address from **each** server's location. A BGP router that receives multiple route advertisements for this IP address treats them as though they were simply providing different _paths_ to the very same physical location — when, in fact, they are genuinely different paths to genuinely _different_ physical locations. Each router locally uses the BGP route-selection algorithm described above to pick the "best" (in practice, the closest, as determined by AS-hop count) route to that IP address.

![[Pasted image 20260708215124.png]]

```
Fig 5.12 -- IP-Anycast: Reaching the Closest
           of Multiple CDN Servers
────────────────────────────────────────────

CDN Server A                  CDN Server B
(attached to AS3)              (attached to AS1)
advertises 212.21.21.21        advertises 212.21.21.21
        │                              │
        ▼                              ▼
    ┌───────┐                      ┌───────┐
    │  AS3  │                      │  AS1  │
    └───┬───┘                      └───┬───┘
        │                              │
        ▼                              │
    ┌───────┐                          │
    │  AS4  │                          │
    └───┬───┘                          │
        │                              │
        └──────────────┬───────────────┘
                        ▼
                   ┌────────┐
                   │  AS2   │  receives ads for
                   │(router)│  212.21.21.21 from
                   └────────┘  BOTH AS1 and AS4

AS1's ad is only 1 AS-hop away; AS4's ad (leading
to Server A via AS3) is 2+ hops away. AS2 therefore
forwards toward Server B, since it is CLOSER.
```

For example, if one BGP route (corresponding to one location) is only one AS hop away from the router, and all other BGP routes (corresponding to other locations) are two or more AS hops away, then the BGP router would choose to route packets to the location that is one hop away. After this initial BGP address-advertisement phase, the CDN can do its main job: distributing content. When a client requests a video, the CDN returns the **common** IP address used by all of the geographically dispersed servers, no matter where the client is actually located. When the client then sends a request to that IP address, Internet routers forward the request packet to the **"closest"** server, exactly as defined by the BGP route-selection algorithm.

### Why CDNs Generally _Avoid_ IP-Anycast in Practice

Although the CDN example nicely illustrates how IP-anycast **can** be used, in practice, CDNs generally choose **not** to use IP-anycast — because BGP routing changes can cause different packets of the _same_ TCP connection to arrive at _different_ instances of the Web server, silently breaking the connection.

But IP-anycast **is** extensively used by the DNS system, to direct DNS queries to the closest **root DNS server**. Recall from Section 2.4 that there are currently 13 IP addresses for root DNS servers. But corresponding to each of these 13 addresses, there are actually _multiple_ DNS root servers — with some of these addresses having over 100 DNS root servers scattered all over the corners of the world. When a DNS query is sent to one of these 13 IP addresses, IP-anycast is used to route the query to the _nearest_ of the (100+) physical DNS root servers that is responsible for that address. [Li 2018; Zhou 2023] present recent measurements illustrating Internet anycast use, performance, and challenges.

---

## 5.4.4 Routing Policy

When a router selects a route to a destination, the **AS routing policy can trump all other considerations** — shortest AS path, hot-potato routing, anything else. Indeed, in the route-selection algorithm above, routes are first selected according to the **local-preference attribute**, whose value is fixed entirely by the policy of the local AS — before either AS-PATH length or hot-potato cost is ever even consulted.

### A Worked Example: Six Interconnected ASes

Consider six interconnected autonomous systems: **A, B, C, W, X,** and **Y** (these are ASes, not individual routers). Assume that W, X, and Y are **access ISPs**, and that A, B, and C are **backbone provider networks**. A, B, and C directly send traffic to each other, and provide full BGP information to their customer networks.

![[Pasted image 20260708215741.png]]

```
Fig 5.13 -- Connectivity Among A,B,C,W,X,Y
─────────────────────────────────────────

A - B, B - C, A - C
  (backbone providers peer directly w/ each other)

W - A
  (W is a single-homed customer of A only)

Y - C
  (Y is a single-homed customer of C only)

X - B, X - C
  (X is MULTI-HOMED: a customer of BOTH B and C)
```

### The Access-ISP Rule

**All traffic entering an ISP access network must be destined for that network, and all traffic leaving an ISP access network must have originated in that network.** W and Y are clearly access networks of this sort — they are each stubs, single-homed to one backbone provider.

X, however, is a **multi-homed access ISP** — connected to the rest of the network via **two** different providers (B and C). This is increasingly common in practice, but it raises a real question: X itself must still be the ultimate source or destination of _all_ traffic leaving or entering X — **how, then, is X prevented from forwarding transit traffic between B and C?**

#### The Answer: Selective Route Advertisement

X functions as a proper access-ISP network _precisely_ by controlling **which** routes it advertises to its neighbors. X advertises to B and C only that it has paths to _itself_ — no paths to any other destination. Even though X may internally know a path (say, "X-C-Y") that would reach network Y, **X simply never advertises this path to B.** X is fully aware of the path's existence, but chooses not to tell B about it — since X does not want to carry transit traffic destined for Y (or C) on B's behalf. This selective, deliberate withholding of route advertisements is precisely how customer/provider routing relationships get implemented in practice — the mechanism is entirely one of _what gets advertised_, not some separate blocking rule bolted on top of BGP.

#### The Provider-to-Provider Case

Now consider a genuine backbone provider, AS B. Suppose B learns (from A) that A has a path "AW" reaching W. B installs the route "AW" into its own routing information base, and — since it wants its customer X to be able to route to W via B — advertises the path "BAW" to X.

But should B also advertise "BAW" to **C**? If it did, C could then route to W via B (through "CBAW"). Since A, B, and C are all backbone providers, B might reasonably feel it shouldn't have to shoulder the cost of carrying _transit_ traffic between A and C — that's arguably A's and C's own job, to make sure C can route to and from A's customers via their own direct connection, not via a free ride through B.

> There are **no official standards** governing how backbone ISPs route among each other. The general rule of thumb followed by most commercial ISPs is this: **any traffic flowing across an ISP's backbone network must have either its source or its destination (or both) in a network that is a customer of that ISP** — otherwise, that traffic is getting a "free ride" on the ISP's network. Individual peering agreements are negotiated bilaterally between pairs of ISPs, and the specific terms are often kept confidential.

### Principles in Practice: Why Are There Different Inter-AS and Intra-AS Routing Protocols?

The differences between OSPF-style intra-AS routing and BGP-style inter-AS routing get at the heart of a genuinely different set of _goals_:

|Reason|Explanation|
|---|---|
|**Policy**|Among ASes, policy issues dominate. It may be important that traffic originating in a given AS not be able to pass through another specific AS. Similarly, a given AS may want to control what transit traffic it carries between other ASes. BGP carries path attributes and provides for controlled distribution of routing information so that such policy-based routing decisions can be made. Within an AS, everything is nominally under the same administrative control, and thus policy issues play a much less important role in choosing routes within the AS|
|**Scale**|The ability of a routing algorithm and its data structures to scale to route to/among large numbers of networks is a critical issue in inter-AS routing. Within an AS, scalability is less of a concern — for one thing, if a single ISP becomes too large, it is always possible to divide it into two ASes and perform inter-AS routing between the two new ASes (recall that OSPF allows exactly such a hierarchy to be built by splitting an AS into areas — Section 5.3)|
|**Performance**|Because inter-AS routing is so policy-oriented, the quality (e.g., performance) of the routes used is often a secondary concern — a longer or more costly route that satisfies certain policy criteria may well be taken over a shorter route that does not meet that criteria. Indeed, among ASes, there is not even a real notion of _cost_ (other than AS hop count) associated with routes. Within a single AS, however, such policy concerns are of far less importance, allowing routing to focus more on the level of performance actually realized on a route|

---

## 5.4.5 Putting the Pieces Together: Obtaining Internet Presence

Although this subsection is not about BGP _per se_, it brings together many of the protocols and concepts studied so far — IP addressing, DNS, and BGP.

### Worked Example: Launching a Small Company Online

Suppose you have just created a small company with a public Web server, a mail server, and a DNS server. Naturally, you'd like the entire world to be able to visit your website, and you'd like your employees to be able to send and receive e-mail to potential customers throughout the world.

#### Step 1: Obtain Internet Connectivity

To meet these goals, you first need to obtain Internet connectivity, which is done by contracting with, and connecting to, a **local ISP**. Your company will have a gateway router, connected to a router in your local ISP (this connection might be a DSL connection through the existing telephone line, a leased line, or some other access technology).

#### Step 2: Get an Address Range

Your local ISP will also provide you with a **range of IP addresses** — for example, a `/24` range consisting of 256 addresses. With your IP address range in hand, you would then assign one IP address to your Web server, one to your mail server, one to your DNS server, one to your gateway router, and one to each of your other networked devices. (If your ISP instead uses NAT, address assignment gets somewhat more complicated — see Section 4.3.3.)

#### Step 3: Register a Domain Name

You also need to contract with an **Internet registrar** to obtain a domain name for your company (Chapter 2 covers this in detail) — say, `xanadu.com`. You must provide the registrar with the IP address of your DNS server; the registrar then puts an entry for your DNS server (domain name plus IP address) into the appropriate top-level-domain (`.com`) servers. After this, anyone in the world can obtain the IP address of your DNS server via the DNS system.

#### Step 4: Populate Your Own DNS Server

You need entries in your own DNS server mapping host names (like `www.xanadu.com`) to IP addresses — and similarly for your mail server. Now, when Alice wants to browse your website, the DNS system contacts your DNS server, obtains the IP address for `www.xanadu.com`, and gives it to Alice; Alice then establishes a TCP connection directly with your Web server.

#### Step 5: The Crucial Piece BGP Provides

There remains one crucial step, without which none of the above actually reaches anyone: allowing outsiders to actually _route_ packets to your servers. When Alice sends an IP datagram (say, a TCP SYN segment) to your Web server's IP address, that datagram must be routed through the Internet, across potentially many ASes, to reach your gateway router. Every router along that path needs a forwarding-table entry pointing toward your company's `/24` prefix.

**How does a router anywhere in the world become aware of your prefix?** Via **BGP**. When your company contracts with its local ISP and is assigned its address prefix, the local ISP uses BGP to advertise this new prefix to the other ISPs it connects to. Those ISPs, in turn, use BGP to propagate the advertisement further outward. Eventually, all Internet routers know about your prefix (or an aggregate CIDR block containing it), and are able to forward datagrams destined to your servers — turning what started as a private contract with a single local ISP into global reachability, entirely through BGP's ordinary, everyday operation.

This completes our brief introduction to BGP. Understanding BGP is important precisely because it plays this central, load-bearing role in the Internet. Consult the BGP references in this section (in particular [Stewart 1999; Huston 2019a]) and the outstanding online BGP resources available at [NSRC 2025] for further depth.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**BGP has no built-in cryptographic verification that an AS actually owns the prefix it's advertising**|An attacker (or a misconfigured router) can advertise a prefix it doesn't legitimately own — a **BGP prefix hijack** — causing traffic destined for that prefix to be misrouted toward the attacker's AS instead of the legitimate owner, anywhere in the world that advertisement propagates to|**RPKI (Resource Public Key Infrastructure)** and route-origin validation let an AS cryptographically prove it is authorized to originate a given prefix; ISPs that filter on RPKI validity substantially reduce hijack blast radius|
|**AS-PATH is trusted at face value by receiving routers**|An attacker can forge a _shorter_ AS-PATH to make a malicious route look more attractive under Rule 2 of the route-selection algorithm, drawing traffic toward itself even without a full prefix hijack|Path validation mechanisms (e.g., BGPsec) aim to cryptographically verify that an AS-PATH reflects the actual sequence of ASes an advertisement passed through, not just what the message claims|
|**Selective route advertisement is entirely policy, with no technical enforcement of "honesty"**|Nothing stops an AS from _breaking_ its own advertised policy — e.g., a supposed access ISP could advertise (or simply carry) transit traffic between two providers it isn't authorized to bridge, effectively getting a "free ride" while appearing compliant on paper|Peering agreements are largely a matter of trust and business relationships rather than cryptographic enforcement; traffic analysis and monitoring for unexpected transit patterns are the practical (imperfect) check|
|**iBGP's full-mesh trust model means any single compromised router inside an AS can inject false routes to every other router in that AS**|A single compromised internal router can use its legitimate iBGP session to spread a poisoned route to the entire AS's other routers — no eBGP boundary crossing required at all|Internal route filtering, monitoring, and limiting which routers are trusted to originate iBGP advertisements reduce (but do not eliminate) this internal blast radius|
|**IP-anycast for CDN/DNS relies on the same trust-the-advertisement model as ordinary BGP**|An attacker who can inject a forged BGP advertisement for an anycasted IP (e.g., a DNS root server address) can potentially divert queries meant for that service to an attacker-controlled endpoint, exactly as with an ordinary prefix hijack|The same RPKI / route-origin-validation defenses that protect ordinary prefixes apply to anycasted ones — anycast doesn't introduce a fundamentally new vulnerability, but it does mean a single successful hijack can affect traffic meant for many geographically dispersed "closest" instances at once|

---

## Questions I Still Have

- [ ] The route-selection algorithm's Rule 1 (local preference) is described as "a policy decision left entirely up to the AS's own network administrator" — in practice, what kinds of business relationships (customer, peer, provider) typically map to which local-preference values, and is there an industry-standard convention, or does every AS set this completely independently?
- [ ] For hot-potato routing's Rule 3 tiebreak — if two candidate NEXT-HOP routers happen to have the exact same least-cost intra-AS path length, does BGP fall through directly to Rule 4 (BGP identifiers), or is there an intermediate tiebreak specific to this scenario?
- [ ] The chapter notes CDNs generally avoid IP-anycast because BGP routing changes can cause packets of the same TCP connection to land on different server instances — given how common CDN usage is today, what _do_ CDNs use instead to achieve "route to the closest server" if not anycast (DNS-based geo-routing, something else)?
- [ ] For the customer/provider selective-advertisement mechanism — is there any technical audit trail (route monitoring services, looking glasses) that lets a provider verify a customer AS is actually honoring its "no transit" agreement, or is this purely a trust-and-contract relationship with no real-time technical enforcement?
- [ ] Given that BGP tables can contain over half a million routes — how does a router's local route-selection computation (with its four sequential elimination rules) stay fast enough to keep up with real-time route churn at that scale, especially compared to OSPF's more centralized, less frequently-changing intra-AS computation?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**BGP (Border Gateway Protocol)**|The Internet's inter-AS routing protocol [RFC 4271]; lets ASes learn prefix reachability from neighbors and select the "best" route via a policy-driven procedure|
|**Prefix**|A CIDRized block of addresses (e.g., `138.16.68/22`) that BGP routes to — BGP does not route to individual destination addresses|
|**Gateway router**|A router on the edge of an AS that directly connects to one or more routers in other ASes|
|**Internal router**|A router that connects only to hosts and routers within its own AS|
|**BGP connection**|A semi-permanent TCP connection (port 179) between a pair of routers, over which BGP messages are exchanged|
|**eBGP (External BGP)**|A BGP connection that spans two different ASes, typically one per direct link between gateway routers in different ASes|
|**iBGP (Internal BGP)**|A BGP session between routers within the same AS; typically forms a full mesh of TCP connections across all routers internal to that AS|
|**AS-PATH**|A BGP route attribute listing the sequence of ASes an advertisement has passed through; used both for path-length comparison and loop prevention|
|**NEXT-HOP**|A BGP route attribute giving the IP address of the router interface that begins the AS-PATH; the critical link between inter-AS (BGP) and intra-AS (e.g., OSPF) routing|
|**BGP route**|A list of three components: NEXT-HOP, AS-PATH, and destination prefix|
|**Hot potato routing**|A BGP route-selection strategy that chooses the route with the least intra-AS cost to the NEXT-HOP router, ignoring costs outside the AS — a "selfish" algorithm|
|**Local preference**|A BGP route attribute, set entirely by local AS policy, that is the first and most powerful tiebreaker in BGP's route-selection algorithm|
|**Route-selection algorithm**|BGP's full four-rule elimination procedure: local preference, then AS-PATH length, then hot-potato routing, then BGP identifiers|
|**IP-anycast**|A technique [RFC 1546, RFC 7094] where the same IP address is advertised from multiple geographically dispersed locations via BGP, letting each router's route-selection algorithm effectively route to the "closest" instance|
|**Root DNS servers**|The 13 IP addresses backing the DNS root, each of which is actually served by 100+ physically distributed servers reached via IP-anycast|
|**Routing policy**|The AS-level rules (implemented via selective BGP route advertisement) that determine which traffic an AS is willing to carry, e.g. distinguishing customer, provider, and peer relationships|
|**Access ISP**|An AS whose traffic must always either originate or terminate within its own network — it does not carry transit traffic between other ASes|
|**Multi-homed access ISP**|An access ISP connected to the rest of the Internet via two or more different provider ASes|
|**Backbone provider network**|An AS (like A, B, or C in the worked example) that directly peers with other backbone providers and carries traffic on behalf of its access-ISP customers|
|**Transit traffic**|Traffic that neither originates nor terminates within the AS carrying it — the thing access ISPs are specifically prevented from carrying, and that backbone providers negotiate bilaterally about|

---

## Related Concepts

---

→ Next: [[SDN CONTROL PLANE]]