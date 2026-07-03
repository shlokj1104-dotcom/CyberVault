---
title: IPV4, NAT & IPV6
date: 2026-07-03
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 4.3 The Internet Protocol (IP): IPV4, Addressing, IPV6, and More

> **One-Line Summary:** The IP protocol is the Internet's universal envelope format and addressing scheme — IPv4's 20-byte datagram header carries a packet across the network while a 32-bit address, structured hierarchically via CIDR into network-prefix and host-suffix bits, tells every router along the way exactly which direction to forward it; because IPv4's ~4.3 billion addresses are running out, three complementary patches — subnetting, DHCP for automatic host configuration, and NAT for private-address sharing — have kept it alive far longer than anyone expected, while IPv6 (with a 128-bit address space and a streamlined 40-byte fixed header) waits in the wings, spreading slowly via dual-stack deployment and IPv4-tunneling because network-layer protocols, unlike application-layer ones, are almost impossible to swap out quickly.

---

## Core Idea: Two Separate Problems, One Protocol

Section 4.2 opened the router's black box. This section asks a different question: what exactly is inside the packets flowing through that box, and how does every device on Earth get a unique, structured address so that routing actually works? IP solves two distinct problems bundled into one protocol suite:

1. **Datagram format** — the syntax and semantics of the bits that make up a network-layer packet (IPv4 in 4.3.1, IPv6 in 4.3.4).
2. **Addressing** — how a 32-bit (or 128-bit) number is assigned to every interface on Earth such that routers can make fast, scalable forwarding decisions using nothing more than prefix matching (4.3.2), how private networks can share a single public address (NAT, 4.3.3), and how a host obtains an address in the first place without manual configuration (DHCP).

> **Analogy — Postal System:** Think of the IP datagram as a physical envelope with a fixed-format address block (version, length, TTL, addresses) and a letter inside (the payload). IP addressing is the postal code system — country, region, street — that lets sorting facilities (routers) route the envelope closer to its destination at every hop without knowing the recipient's exact house from the very first sorting center. CIDR prefixes are exactly like a postal code's hierarchical structure: a 5-digit ZIP code narrows a letter to a city; the last two digits narrow it to a specific block.

---

## 4.3.1 IPv4 Datagram Format

### Why the Datagram Format Matters

Every networking student eventually has to internalize the datagram format because **every router along a packet's path inspects it** to decide what to do next. Getting this right is foundational — it's the shared contract that lets equipment from thousands of different vendors interoperate.

![[Pasted image 20260703153934.png]]

### The IPv4 Header, Field by Field

The header is a stack of 32-bit rows. Here's what each field does and _why_ it exists:

|Field|Size|Purpose|Why It Exists|
|---|---|---|---|
|**Version**|4 bits|Identifies the IP version (4 for IPv4)|So a router knows how to interpret the rest of the bits — IPv4 and IPv6 datagrams have _completely different_ layouts, and this field is checked first|
|**Header length**|4 bits|Specifies where the header ends and the payload (data) begins|Necessary because the header can be a variable length (due to the optional **Options** field)|
|**Type of service (TOS)**|8 bits|Lets different types of datagrams (e.g. real-time VoIP vs. bulk FTP) be distinguished from each other|A policy hook — a network administrator configures how a router should prioritize traffic. Two of these bits are also reused for Explicit Congestion Notification (ECN), covered in Ch. 3|
|**Datagram length**|16 bits|Total length of the datagram (header + data), in bytes|16 bits theoretically allows up to 65,535 bytes, but datagrams are rarely larger than 1,500 bytes in practice — sized to fit inside a maximally-sized Ethernet frame payload|
|**Identifier, flags, fragmentation offset**|16 + 3 + 13 bits|Used for IP **fragmentation** — splitting a large datagram into several smaller ones that travel independently and get reassembled at the destination before being passed up to the transport layer|Different physical links have different maximum frame sizes (MTUs); a datagram too big for a given link must be split. **IPv6 does not support fragmentation at all** — a deliberate simplification (see 4.3.4)|
|**Time-to-live (TTL)**|8 bits|Decremented by 1 at every router that processes the datagram; the router discards the datagram if TTL reaches 0|Prevents a datagram from circulating forever due to a routing loop|
|**Protocol**|8 bits|Only meaningful at the final destination — tells the receiving host which transport-layer protocol should get the payload (6 = TCP, 17 = UDP)|The **network-to-transport-layer glue**, exactly analogous to how a transport-layer port number is the glue binding the transport and application layers together|
|**Header checksum**|16 bits|Helps a router detect bit errors in the _received_ header|Computed via the same 1s-complement "Internet checksum" method used in Ch. 3. Must be **recomputed at every router**, since TTL (and possibly Options) changes hop-to-hop. Routers typically discard any datagram whose checksum doesn't match|
|**Source IP address**|32 bits|The sender's IP address, inserted by the source host|The destination address is normally learned via a DNS lookup (Ch. 2)|
|**Destination IP address**|32 bits|The ultimate destination's IP address|This is the field routers actually match against when forwarding (Section 4.2.1's longest-prefix-match lookup)|
|**Options**|Variable|Allows the header to be extended|Rarely used in practice — deliberately, to save overhead. Because header length becomes variable once options are present, a router cannot know a priori where the data field begins, which slows processing. **This is exactly why options were dropped from the IPv6 header entirely**|
|**Data (payload)**|Variable|The actual content being delivered — typically a TCP or UDP segment, but can also carry ICMP messages (Section 5.6) or other data|The _raison d'être_ of the whole datagram — everything else is overhead in service of getting this payload to its destination correctly|

### Worked Example: Why Fast Checksum/TTL Recomputation Matters

Consider a datagram traversing 5 routers on its path (hop count = 5). At **every single router:**

```
1. TTL field is read
2. TTL is decremented by 1 (TTL_new = TTL_old − 1)
3. If TTL_new == 0 → datagram is DISCARDED, no forwarding occurs
4. Header checksum is RECOMPUTED (since TTL changed, the old checksum is now invalid)
5. New checksum replaces old checksum in the header
6. Datagram forwarded to next hop
```

If a datagram starts with TTL = 4 and must cross 5 routers to reach its destination:

```
Router 1: TTL 4 → 3   (checksum recomputed, forwarded)
Router 2: TTL 3 → 2   (checksum recomputed, forwarded)
Router 3: TTL 2 → 1   (checksum recomputed, forwarded)
Router 4: TTL 1 → 0   → DISCARDED. Destination never reached.
```

This is precisely the mechanism `traceroute` exploits: by deliberately sending datagrams with increasing TTL values (1, 2, 3, ...) and observing which router sends back the "TTL expired" ICMP message, a tool can map out every hop on a path.

### Worked Example: Header Overhead in a Real TCP Connection

A typical IPv4 datagram carrying a TCP segment (no options on either header):

```
IP header:   20 bytes  (no options)
TCP header:  20 bytes  (no options)
─────────────────────────────────
Total overhead per datagram: 40 bytes
```

So if an application-layer message is 1000 bytes, the datagram actually sent over the wire is:

```
1000 bytes (application data) + 40 bytes (headers) = 1040 bytes total
Overhead percentage = 40/1040 ≈ 3.85%
```

For small messages (e.g. a 20-byte DNS query), this overhead becomes proportionally much larger — a key reason protocol designers care deeply about keeping headers compact.

---

## 4.3.2 IPv4 Addressing

### Interfaces, Not Hosts, Own Addresses

Before addressing itself, one subtlety matters: **an IP address is technically associated with an interface, not with the host or router that contains that interface.**

- A **host** typically has a single interface (a single link into the network).
- A **router**, by definition, has two or more interfaces — one for each link it connects to, since its entire job is to receive a datagram on one link and forward it out a different one.

> **Analogy:** Think of a router as a building with multiple doors (interfaces), each opening onto a different street (subnet). The building itself doesn't have one address — each _door_ has its own street address, because mail delivered to that door needs to know which street it's on.

### Dotted-Decimal Notation

Each IPv4 address is 32 bits (4 bytes), giving 2³² ≈ 4.3 billion possible addresses. Addresses are written in **dotted-decimal notation** — each byte written as its decimal equivalent, separated by periods.

#### Worked Example: Binary ↔ Dotted-Decimal Conversion

Take the address `193.32.216.9`:

```
193  →  11000001
32   →  00100000
216  →  11011000
9    →  00001001

Full 32-bit binary form: 11000001 00100000 11011000 00001001
```

Each dotted-decimal number 0–255 maps directly to one 8-bit byte — this is why every octet in an IPv4 address is constrained to the range 0–255 (2⁸ possible values).

### Subnets: The Core Addressing Building Block

**A subnet** (also called an "IP network" or simply a "network") is the set of device interfaces that can physically reach each other **without passing through any router**. IP addressing assigns an address block to the subnet as a whole.

![[Pasted image 20260703154144.png]]

![[Pasted image 20260703154517.png]]

#### The General Recipe for Identifying Subnets

> **To determine the subnets in any interconnected system: detach each interface from its host or router, creating islands of isolated networks, with interfaces terminating the endpoints of the isolated networks. Each of these isolated networks is a subnet.**

This recipe is important because it generalizes beyond simple Ethernet-LAN topologies to arbitrary router-router interconnections (point-to-point links included).

#### Worked Example: Applying the Subnet Recipe

Consider three routers (R1, R2, R3) interconnected by point-to-point links, each also serving a broadcast LAN of two hosts. Each router has **three interfaces** (one per point-to-point link, one for its LAN).

```
Applying the "detach every interface" recipe:

  ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
  │ 223.1.1.0/24│         │ 223.1.2.0/24│         │ 223.1.3.0/24│
  │  (R1's LAN) │         │  (R2's LAN) │         │  (R3's LAN) │
  └─────────────┘         └─────────────┘         └─────────────┘

  Point-to-point subnets (each an isolated 2-interface island):
  223.1.9.0/24   → R1 ↔ R2 link
  223.1.8.0/24   → R2 ↔ R3 link
  223.1.7.0/24   → R3 ↔ R1 link

TOTAL SUBNETS: 6  (three LAN subnets + three point-to-point subnets)
```

Even though a point-to-point link has only two interfaces total (one on each end), it still forms its own subnet with its own /24 address block — because those two interfaces can reach each other directly without a router in between, satisfying the subnet definition exactly.

### CIDR: Classless Interdomain Routing

Before understanding modern addressing, it helps to see the problem CIDR solved.

#### The Old System: Classful Addressing (and Why It Failed)

Pre-CIDR, network portions of an address were constrained to exactly **8, 16, or 24 bits** — Class A, B, and C networks respectively. This rigidity was fatal for scalability:

|Class|Subnet size (host bits)|Max hosts|Problem|
|---|---|---|---|
|**C** (/24)|8 bits|2⁸ − 2 = 254|Too small for many organizations|
|**B** (/16)|16 bits|2¹⁶ − 2 = 65,534|Way too large for an org with, say, 2,000 hosts — massive waste|
|**A** (/8)|24 bits|2²⁴ − 2 ≈ 16.7 million|Only usable by a tiny handful of huge organizations|

#### Worked Example: Classful Addressing Waste

An organization needs addresses for 2,000 hosts.

```
Class C (/24) option: max 254 hosts → NOT ENOUGH, must be denied
Class B (/16) option: max 65,534 hosts → GRANTED (only option that fits)

Addresses actually used:       2,000
Addresses allocated:           65,534
Addresses WASTED:              63,534   (>96% of the block unused)
```

Multiply this waste across thousands of organizations getting Class B allocations for small-to-medium host counts, and the entire 32-bit address space depletes far faster than necessary. This is precisely the failure mode CIDR was designed to eliminate.

#### CIDR: Variable-Length Prefixes

CIDR generalizes subnet addressing: the 32-bit address is divided into two parts, written `a.b.c.d/x`, where:

- **`x`** = the number of bits in the first part of the address — the **prefix** (or **network prefix**)
- The remaining **32 − x** bits distinguish specific devices _within_ the organization

**Why this matters for router table size:** Routers _outside_ the organization only need to consider the leading `x` bits of the destination address. A single forwarding-table entry (`a.b.c.d/x`) suffices to forward packets toward _any_ destination within that organization — dramatically shrinking external routing tables compared to needing one entry per individual host.

#### Worked Example: CIDR Address Structure

Given the CIDR address `a.b.c.d/21`:

```
Total bits: 32
Network prefix (x): 21 bits   → identifies the organization to the outside world
Remaining bits: 32 − 21 = 11 bits  → identifies specific hosts WITHIN the organization

2^11 = 2,048 possible host addresses within this organization's block
```

Those remaining 11 bits may _further_ be subdivided internally — e.g., an organization might use its rightmost bits for its own internal subnetting structure, invisible to the outside world, which only ever looks at the leading 21 bits.

### Address Aggregation (Route Summarization)

CIDR's real power shows up when ISPs allocate address blocks hierarchically to client organizations.

![[Figure 4.21.png]]

#### Worked Example: Address Aggregation

An ISP ("Fly-By-Night-ISP") owns the block `200.23.16.0/20` and has allocated eight equal-sized /23 sub-blocks to eight client organizations:

```
ISP's full block:     200.23.16.0/20    11001000 00010111 0001 0000 00000000
Organization 0:       200.23.16.0/23    11001000 00010111 0001 0000 00000000
Organization 1:       200.23.18.0/23    11001000 00010111 0001 0010 00000000
Organization 2:       200.23.20.0/23    11001000 00010111 0001 0100 00000000
...
Organization 7:       200.23.30.0/23    11001000 00010111 0001 1110 00000000
```

Rather than the rest of the Internet needing eight separate routing-table entries (one per organization), the ISP advertises **a single prefix**: "send me anything beginning `200.23.16.0/20`." Every other router in the world sees one compact entry instead of eight. This technique — advertising one aggregated prefix that covers many downstream sub-networks — is called **address aggregation** or **route aggregation/summarization**.

#### Worked Example: When Aggregation Breaks — Longest Prefix Match to the Rescue

What happens if Fly-By-Night-ISP acquires a subsidiary, ISPs-R-Us, and Organization 1 (previously under Fly-By-Night-ISP) is reassigned to connect through ISPs-R-Us — but Organization 1's addresses (`200.23.18.0/23`) remain _inside_ Fly-By-Night-ISP's advertised `200.23.16.0/20` block?

```
Fly-By-Night-ISP still advertises:  "Send me anything matching 200.23.16.0/20"
ISPs-R-Us advertises:               "Send me anything matching 199.31.0.0/16"
                                     "Send me anything matching 200.23.18.0/23" (NEW, more specific)

A packet destined for 200.23.18.5 (inside Organization 1) arrives at a distant router.

Candidate matches:
  200.23.16.0/20  (20-bit prefix match, via Fly-By-Night-ISP)  ✓ matches
  200.23.18.0/23  (23-bit prefix match, via ISPs-R-Us)          ✓ matches — LONGER

LONGEST PREFIX MATCH WINS → packet routed via ISPs-R-Us, correctly reaching Org 1
```

This is the exact same longest-prefix-matching rule from Section 4.2.1's TCAM lookup discussion — reappearing here at the _inter-organization, global-routing_ scale rather than the single-router scale. Without renumbering Organization 1's entire address space (a costly, disruptive operation), a single more-specific route entry fixes the exception cleanly. This is precisely how real-world exceptions to hierarchical address allocation get handled without breaking the aggregation benefit for everyone else.

### The IP Broadcast Address

`255.255.255.255` is reserved as the IP broadcast address. A datagram sent to this address is delivered to **all hosts on the same subnet**. Routers _may_ optionally forward such broadcast messages into neighboring subnets as well, although in practice they usually don't (to avoid broadcast storms propagating across the entire Internet).

### Obtaining a Block of Addresses

The address-allocation hierarchy works top-down:

```
ICANN (Internet Corporation for Assigned Names and Numbers)
   │  allocates address blocks to...
   ▼
Regional Internet Registries (ARIN, RIPE, APNIC, LACNIC — the "Address Supporting
Organization" of ICANN)
   │  allocate address blocks to...
   ▼
ISPs
   │  sub-allocate address blocks to...
   ▼
Client organizations (contact their ISP for a block from the ISP's larger allocation)
```

ICANN's role extends beyond just IP address allocation — it also manages the DNS root servers and resolves domain name disputes, making it one of the Internet's most consequential (and historically contentious) governance bodies.

### Obtaining a Host Address: DHCP

Once an organization has its address block, individual host/router interfaces still need specific addresses assigned. Router interfaces are typically configured manually (often remotely via a network management tool). Host addresses, however, are typically assigned automatically via **DHCP (Dynamic Host Configuration Protocol)**.

**Why DHCP matters — the "plug-and-play" problem:** Manually configuring an IP address into every laptop, phone, and IoT device on a network doesn't scale — and it's actively hostile to _mobility_. Consider a student carrying a laptop from a dorm room, to a library, to a classroom in one day: each location is potentially a different subnet, requiring a different valid IP address at each stop. DHCP automates this completely, which is why it's described as a **plug-and-play** (or **zeroconf**, zero-configuration) protocol.

DHCP additionally lets a host learn:

- Its **subnet mask**
- The address of its **first-hop router** (often called the **default gateway**)
- The address of its **local DNS server**

DHCP is **client-server**: a newly arriving host is the client, requesting configuration info from a DHCP server. If no DHCP server exists on the local subnet, a **DHCP relay agent** (typically the local router) forwards the request to a DHCP server elsewhere.

#### The Four-Step DHCP Process (DORA)

![[Figure 4.23.png]]

![[Figure 4.24.png]]

```
                         DHCP FOUR-STEP HANDSHAKE
                         ────────────────────────

  Newly arriving DHCP client                          DHCP Server
  (has NO IP address yet — uses 0.0.0.0)               (223.1.2.5)
          │                                                   │
          │  1. DHCP DISCOVER                                 │
          │  src: 0.0.0.0:68                                  │
          │  dest: 255.255.255.255:67 (BROADCAST — client      │
          │        doesn't know the server's address yet!)    │
          │──────────────────────────────────────────────────►│
          │                                                   │
          │  2. DHCP OFFER (also broadcast — client has        │
          │     no address to unicast to yet)                 │
          │  proposed yiaddr: 223.1.2.4                        │
          │  server ID: 223.1.2.5, lifetime: 3600s              │
          │◄──────────────────────────────────────────────────│
          │        (may receive MULTIPLE offers if several      │
          │         DHCP servers are present on the subnet)     │
          │                                                   │
          │  3. DHCP REQUEST                                   │
          │  echoes back the CHOSEN offer's parameters          │
          │──────────────────────────────────────────────────►│
          │                                                   │
          │  4. DHCP ACK                                       │
          │  confirms the requested parameters, finalizes       │
          │  the address LEASE (lifetime: 3600s)                │
          │◄──────────────────────────────────────────────────│
          │                                                   │
   Client now uses 223.1.2.4 for the lease duration
```

|Step|Message|Purpose|
|---|---|---|
|**1. DHCP server discovery**|`DHCPDISCOVER`, broadcast within a UDP packet to port 67|Client doesn't know any DHCP server's address, so it broadcasts to `255.255.255.255` with source `0.0.0.0`|
|**2. DHCP server offer(s)**|`DHCPOFFER`, broadcast back to `255.255.255.255`|Server proposes an IP address (`yiaddr`), network mask, and an **address lease time** (how long the address is valid — commonly hours or days)|
|**3. DHCP request**|`DHCPREQUEST`|Client picks from among possibly multiple offers and echoes back the chosen server's configuration parameters|
|**4. DHCP ACK**|`DHCPACK`|Server confirms the requested parameters; the handshake is complete and the client may now use the allocated address|

**DHCP's significant mobility shortcoming:** Because a new IP address is obtained from DHCP _every time a node connects to a new subnet_, an existing TCP connection **cannot be maintained** as a mobile node moves between subnets — the address underneath the connection literally changes. (Chapter 7 covers how mobile cellular networks solve this as devices roam between base stations.)

---

## 4.3.3 Network Address Translation (NAT)

### The Problem NAT Solves

Every small office/home office (SOHO) network wants to connect multiple devices — phones, tablets, gaming consoles, smart TVs, printers — to the Internet. The "obvious" solution (asking the ISP to allocate a whole range of public addresses for every device) doesn't scale: there are hundreds of thousands of home networks, and IPv4 simply doesn't have enough spare addresses for every device on Earth to get a unique, globally-routable one.

### Private Address Space

**NAT (network address translation)** is the practical fix. It relies on address blocks reserved by RFC 1918 as **private network** (or "realm with private addresses") space — meaning these addresses only have meaning _within_ a single private network and are never used as source/destination on the global Internet:

|Private block|Notation|
|---|---|
|`10.0.0.0/8`|Class A-style private block|
|`172.16.0.0/12`|Class B-style private block|
|`192.168.0.0/16`|Class C-style private block|

Because these blocks are reused by hundreds of thousands of independent home and enterprise networks simultaneously, packets addressed to `10.0.0.1` mean something completely different depending on _which_ private network they're on — which is precisely why they can never be routed on the public Internet as-is.

![[Figure 4.25.png]]

### How NAT Actually Works

The NAT-enabled router "hides" the internal structure of the home network from the outside world entirely. From the outside world's point of view, the NAT router behaves like a **single device with a single public IP address** — even though it may be forwarding traffic for dozens of internal hosts simultaneously.

The mechanism relies on a **NAT translation table** that maps `(WAN-side IP, WAN-side port)` pairs to `(LAN-side IP, LAN-side port)` pairs.

#### Worked Example: NAT Translation Step by Step

Setup: home host `10.0.0.1` wants to fetch a web page (port 80) from web server `128.119.40.186`. The NAT router's WAN-side (public) address is `138.76.29.7`.

```
STEP 1 — Outbound request:
  Host 10.0.0.1 assigns itself an arbitrary source port: 3345
  Datagram sent into the LAN:
     src = 10.0.0.1:3345
     dst = 128.119.40.186:80

STEP 2 — NAT router intercepts and rewrites:
  Router generates a NEW source port: 5001 (any port not currently
  in use in its translation table)
  Router creates translation table entry:
     WAN side: 138.76.29.7, 5001   ↔   LAN side: 10.0.0.1, 3345
  Router rewrites the datagram:
     src = 138.76.29.7:5001   (router's own public address + new port)
     dst = 128.119.40.186:80
  → forwarded onto the larger Internet

STEP 3 — Web server responds (unaware NAT even exists):
     src = 128.119.40.186:80
     dst = 138.76.29.7:5001

STEP 4 — NAT router receives reply, INDEXES its translation table using
  (destination IP, destination port) = (138.76.29.7, 5001):
     table lookup → (10.0.0.1, 3345)
  Router rewrites the datagram's destination:
     src = 128.119.40.186:80
     dst = 10.0.0.1:3345
  → forwarded into the home LAN, delivered to the correct original host
```

> **Analogy — Corporate Mail Room:** Imagine an office building with one street address but hundreds of employees. Outgoing mail all shows the building's single street address as the return address, but the mail room stamps a unique internal tracking code on each envelope corresponding to which employee sent it. When a reply comes back addressed to that tracking code, the mail room looks it up and routes the reply to the correct employee's desk — even though the outside world never learns anyone's individual internal office number. That's exactly what the (port number → internal host) translation table does.

**Why this scales so well:** Because a port number field is 16 bits, a single WAN-side public IP address can theoretically support **over 60,000 simultaneous connections** through one NAT router — turning one scarce public IPv4 address into shared infrastructure for an entire household or small office.

### Where NAT Gets Deployed: Beyond the Home

NAT isn't just a home-router feature — it's genuinely widespread across the entire access-network hierarchy:

- **Almost every home or enterprise device** you connect to a network gets a private-prefix address (`10.0.0.0/8`, `172.16.0.0/12`, or `192.168.0.0/16`).
- **ISPs use NAT too.** A cable ISP with tens of thousands of home gateway routers in its network (each already running NAT internally) may itself use NAT to assign _private_ addresses to those home routers, rather than burning a scarce public address on every single home gateway.
- This creates **double-NAT** scenarios: a datagram sourced by a device inside the home passes through **two** NAT'd networks — the home network's NAT, then the ISP network's NAT — before reaching the public Internet.
- This large-scale ISP-side deployment is called **Carrier-grade NAT (CGN)**, implementing NAT translation at high speeds for enormous numbers of devices. A dedicated private block, `100.64.0.0/10`, was created specifically for ISP-only use in this scenario — to avoid overlap conflicts between a first-hop home/enterprise NAT and an ISP NAT that might otherwise choose the same private address space.

### NAT's Detractors: Two Classes of Objection

|Objection Type|The Problem|
|---|---|
|**Practical (breaks P2P)**|Port numbers were architecturally meant for addressing _processes_, not _hosts_. Server processes and P2P peers wait for incoming connections at well-known ports — but how can a peer connect _into_ a host sitting behind a NAT with a dynamically DHCP-assigned private address? Technical fixes exist (**NAT traversal** tools), but they're workarounds, not clean solutions|
|**Architectural ("philosophical")**|Routers are architecturally meant to be **layer-3 devices** — they should process packets only up through the network layer. NAT violates this: it actively rewrites port numbers, which belong to the _transport_ layer, meaning hosts are no longer truly talking directly to each other without interference from intermediate nodes. This becomes part of the broader **middlebox** debate (Section 4.5)|

---

## Focus on Security: Firewalls and Intrusion Detection Systems

Any network administrator faces a constant threat: attackers who know a network's IP address range can send datagrams anywhere within it — ping sweeps, port scans, malformed packets designed to crash vulnerable hosts, open-port scanning, and outright malware delivery.

### Firewalls

A **firewall**, installed between a network and the Internet, inspects datagram and segment header fields and **denies suspicious datagrams entry** into the internal network. Most access routers today have built-in firewall capability. Concrete capabilities:

- Block all ICMP echo request packets (Section 5.6) — preventing an attacker from running a traditional port scan across an IP address range
- Block packets based on source and destination IP addresses and port numbers
- **Track TCP connections**, granting entry only to datagrams belonging to already-approved, legitimate connections

### Intrusion Detection Systems (IDS)

An IDS, typically situated at the network boundary, performs **"deep packet inspection"** — examining not just header fields but also the actual application-layer payload data inside the datagram. It maintains a **database of known attack signatures**, automatically updated as new attacks are discovered. When incoming packets match a signature, an alert is generated.

An **intrusion prevention system (IPS)** is nearly identical to an IDS, but goes one step further: it actually **blocks** the offending packets rather than merely alerting on them.

**The honest limitation:** Neither mechanism can fully shield a network from all attacks — attackers continually develop new attacks for which no signature yet exists. Firewalls and traditional signature-based IDSs are useful specifically for protecting against **known** attacks, not zero-days. (Firewalls and IDSs are revisited in more depth in Section 4.5 and again in Chapter 8.)

---

## 4.3.4 IPv6

### Why IPv6 Exists

The core motivation, dating to the early 1990s IETF effort, was stark: the **32-bit IPv4 address space was being consumed at a breathtaking rate** as new subnets and IP nodes attached to the Internet. Two IETF working-group estimates in the mid-1990s projected complete IPv4 exhaustion by 2008 or 2018. In reality, **IANA allocated the last remaining pool of unassigned IPv4 addresses to regional registries in February 2011** — and while those regional registries still had some inventory left within their own pools for a while afterward, once _that_ is exhausted, there is **no further central pool to draw from**. The designers of IPv6 used this crisis as an opportunity to also fix various other accumulated IPv4 design pain points based on operational experience.

### IPv6 Datagram Format

![[Figure 4.26.png]]

The most important structural changes IPv6 introduces:

|Change|Detail|
|---|---|
|**Expanded addressing**|Address size grows from 32 bits to **128 bits** — ensuring the world won't run out of IP-addressable "grains of sand" for a very long time. IPv6 also introduces a new address type, the **anycast address**, which delivers a datagram to _any one_ member of a group of hosts (e.g., routing an HTTP GET request to the nearest of several mirror servers hosting the same content)|
|**Streamlined 40-byte fixed header**|Several IPv4 fields were dropped or made optional, producing a fixed-length 40-byte header for **faster router processing**. Options are no longer embedded directly — instead, a new flexible encoding treats options as additional "next headers" chained after the base header|
|**Flow labeling**|A new capability for labeling packets belonging to a particular "flow" for which the sender requests special handling (e.g. non-default QoS or real-time service). RFC 2460 deliberately leaves the definition of a "flow" elusive — the designers foresaw the eventual need to differentiate flows even before the exact meaning had been fully worked out|

### IPv6 Field-by-Field Comparison to IPv4

|IPv6 Field|Size|Purpose|Compare to IPv4|
|---|---|---|---|
|**Version**|4 bits|Identifies IP version (6)|Same role as IPv4's version field. Note: setting this field to 4 does **not** create a valid IPv4 datagram — the rest of the layout is completely different|
|**Traffic class**|8 bits|Prioritize datagrams within a flow, or across applications (e.g. VoIP over SMTP email)|Analogous to IPv4's TOS field|
|**Flow label**|20 bits|Identifies datagrams belonging to the same flow|**New in IPv6** — no IPv4 equivalent|
|**Payload length**|16 bits|Number of bytes in the datagram _following_ the fixed 40-byte header|Roughly analogous to IPv4's datagram-length field, but excludes the header itself|
|**Next header**|8 bits|Identifies which protocol the payload should be delivered to (TCP, UDP, or another chained IPv6 header)|Same role as IPv4's protocol field|
|**Hop limit**|8 bits|Decremented by 1 at every router; datagram discarded if it reaches 0|Directly analogous to IPv4's TTL field|
|**Source / destination addresses**|128 bits each|The (much larger) IPv6 addresses|Direct analogue, just 4× the bit-width|
|**Data**|Variable|The payload|Same role as IPv4's data field|

### What IPv4 Fields Were Removed, and Why

|Removed field|Why it was cut|
|---|---|
|**Fragmentation/reassembly**|IPv6 **does not allow fragmentation and reassembly at intermediate routers at all** — these operations are performed only by the source and destination end systems. If an IPv6 datagram arrives at a router too large to be forwarded on the outgoing link, the router simply **drops it** and sends a "Packet Too Big" ICMP error message back to the sender, which then resends using a smaller datagram size. Removing this functionality from routers considerably speeds up IP forwarding, since routers no longer need to do this time-consuming, per-packet work|
|**Header checksum**|Considered redundant, since transport-layer (TCP/UDP) and link-layer (Ethernet) protocols already perform their own checksumming. Removing it was a deliberate concern for processing speed — recall that in IPv4, the header checksum must be recomputed at _every_ router because the TTL field changes at every hop; removing the field removes that per-hop recomputation cost entirely|
|**Options**|No longer part of the standard fixed header — instead, an options field is just one of the possible "next headers" pointed to from within the base IPv6 header, similar to how TCP or UDP headers themselves are pointed to as a "next header." This keeps the base header a clean, fixed 40 bytes|

### Worked Example: Why Removing the Header Checksum Actually Matters at Scale

Consider a datagram traversing 10 routers (10-hop path):

```
IPv4 behavior — at EACH of the 10 routers:
  1. Recompute the entire header checksum (1s-complement sum over the header)
  2. Compare to received checksum
  3. Store the new, recomputed checksum back into the header

  → 10 checksum recomputations total for this single datagram's journey

IPv6 behavior — at EACH of the 10 routers:
  1. (No checksum field exists — this step is simply skipped)

  → 0 checksum recomputations. Forwarding logic is simpler and faster in hardware.
```

Multiply this by the nanosecond-scale time budgets established in Section 4.2 (recall: a 100 Gbps link gives a router only ~5.12 ns per 64-byte datagram) and the value of removing _any_ per-hop recomputation step becomes concrete — every operation cut from the per-packet hot path directly helps hardware keep pace with rising line rates.

### Transitioning from IPv4 to IPv6: The Hard Problem

New IPv6-capable systems can be made backward-compatible (able to send, receive, and route IPv4 datagrams). But the reverse isn't automatically true: **already-deployed IPv4-only systems cannot handle IPv6 datagrams at all.**

**Why a "flag day" is impossible:** One option would be a single flag day where every Internet-connected machine switches from IPv4 to IPv6 simultaneously. The last comparable transition — from NCP to TCP for reliable transport, nearly 40 years ago — happened when the Internet was tiny and administered by a handful of "wizards." A flag day today, across billions of devices under no central authority, is simply unthinkable.

### The Adopted Solution: Tunneling

![[Figure 4.27.png]]

**Tunneling** is the practical mechanism that has seen the most widespread real-world adoption for IPv4-to-IPv6 transition. The core idea generalizes far beyond this specific use case (it's reused extensively in all-IP cellular networks, covered in Chapter 7).

#### Worked Example: IPv6 Tunneling Through an IPv4 "Cloud"

Setup: two IPv6 nodes, **B** and **E**, want to communicate using native IPv6 datagrams, but the _intervening_ routers between them (**C** and **D**) only understand IPv4.

```
LOGICAL VIEW (what B and E believe is happening):
   A(IPv6) ── B(IPv6) ══TUNNEL══ E(IPv6) ── F(IPv6)
                     (looks like ONE direct hop)

PHYSICAL VIEW (what's actually happening):
   A(IPv6) → B(IPv6) → C(IPv4) → D(IPv4) → E(IPv6) → F(IPv6)

STEP-BY-STEP DATA FLOW:

1. A → B: native IPv6 datagram
      Flow: X, Source: A, Dest: F   [data]

2. AT NODE B (tunnel entry point):
      B takes the ENTIRE IPv6 datagram (header + payload) and stuffs it whole
      into the DATA (payload) field of a brand-new IPv4 datagram.
      That IPv4 datagram is addressed to E (the receiving end of the tunnel):

      IPv4 header: Source: B, Dest: E, Protocol field = 41
                   (41 specifically signals: "my payload is a complete IPv6 datagram")
      IPv4 payload: [ Flow: X, Source: A, Dest: F | original IPv6 data ]

3. B → C → D (intervening IPv4 routers C and D):
      C and D route this IPv4 datagram completely normally, exactly as they
      would ANY other IPv4 datagram — utterly unaware that its payload
      secretly contains a full IPv6 datagram. The IPv6 content is
      "invisible cargo" to them.

4. AT NODE E (tunnel exit point):
      E receives the IPv4 datagram (it IS the addressed destination).
      E inspects the protocol field, sees 41, and concludes:
      "this IPv4 payload is actually a complete IPv6 datagram."
      E EXTRACTS the original IPv6 datagram from the IPv4 payload.
      E then routes this extracted IPv6 datagram exactly as it would have
      if it had received it directly from a directly-connected IPv6 neighbor.

5. E → F: native IPv6 datagram continues its journey unmodified.
```

> **Analogy — Shipping a Car Overseas Inside a Cargo Container:** You can't drive a car across an ocean. So you put the _entire car_ (IPv6 datagram) inside a standard shipping container (IPv4 datagram) that ships handle just fine. Every port and crane along the way (the IPv4 routers) just sees "a container" and moves it exactly like any other container — they have no idea there's a car inside. Only at the destination port (the tunnel's IPv6-capable exit node) does someone open the container and drive the car back out onto the road.

### IPv6 Adoption: Slow but Building

Momentum has been gradually building since IPv6's initially slow uptake in the early 2000s. Some concrete adoption figures:

- A 2025 US-government-domain survey found nearly 1,400 domains, with approximately 40% of mail, web, and DNS services configured for IPv6.
- Google reports that nearly 50% of clients accessing Google services now do so via IPv6.
- Akamai noted end-user IPv6 adoption levels of roughly 44–62% across the top 10 global economies (by percentage of requests to Akamai's dual-stacked IPv4+IPv6 CDN content) in 2022.
- For perspective on the growth curve: Google reported an IPv6 adoption rate of just **1 percent** in 2012 — meaning adoption has climbed roughly 50 percentage points in about a decade.
- The proliferation of IP-enabled phones and other portable devices provides continual additional pressure toward wider IPv6 deployment.

### The Deeper Lesson: Network-Layer vs. Application-Layer Change

The IPv6 rollout experience teaches a broader architectural lesson: **it is enormously difficult to change a network-layer protocol.** Numerous new network-layer protocols (IPv6 itself, multicast protocols, resource reservation protocols) have been trumpeted since the early 1990s as "the next major revolution for the Internet," yet most have seen only limited penetration to date.

> **Analogy — Foundation vs. Paint:** Introducing a new _network-layer_ protocol is like replacing the **foundation** of a house — it's extremely difficult to do without tearing the whole house down, or at least temporarily relocating its residents (i.e., disrupting live traffic for billions of users). Introducing a new _application-layer_ protocol, by contrast, is like adding a fresh coat of **paint** — comparatively easy, and if you pick something attractive (a compelling new app or service), others in the neighborhood will happily copy you. This is exactly why the Web, messaging, streaming media, video conferencing, distributed games, and social media have all evolved and proliferated at breakneck pace, while the network layer underneath has changed only very slowly.

**Bottom line going forward:** expect continued (if slow) network-layer evolution, but on a timescale that will always lag far behind the pace of change happening one layer up, at the application layer.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**TTL/Hop-limit expiration reveals path topology**|`traceroute`-style TTL manipulation lets an attacker map internal network topology and identify live routers for reconnaissance before an attack|Rate-limiting or filtering ICMP "TTL expired" responses at network boundaries reduces topology reconnaissance surface|
|**Header checksum only catches accidental bit errors, not tampering**|The IPv4 header checksum is _not_ a security mechanism — it can't detect deliberate manipulation, only random transmission errors. An attacker modifying header fields in transit and recomputing a valid checksum passes this check trivially|Integrity and authenticity require cryptographic mechanisms (IPsec, TLS) layered on top — the base IP header checksum provides zero security guarantee, only bit-error detection|
|**NAT provides accidental, incidental security — not real security**|Because NAT hides internal private addresses from the outside world, unsolicited _inbound_ connections generally can't reach an internal host directly — but this is a side effect of NAT's translation-table design (only established/expected mappings route inbound), not a designed security boundary. Techniques like NAT traversal can punch holes through it|Firewalls should be treated as the actual, deliberate security control; NAT's address-hiding effect should never be relied upon as a substitute for one|
|**Firewalls/IDS only catch known attack signatures**|Zero-day exploits and novel attack patterns simply have no signature yet, and sail through undetected|Signature databases must be continuously and automatically updated; defense-in-depth (firewall + IDS/IPS + endpoint protection) compensates for the fact that no single layer catches everything|
|**DHCP has no built-in authentication**|A rogue DHCP server on a subnet can hand out malicious configuration (e.g., a spoofed default gateway or DNS server address), silently redirecting a victim's traffic through an attacker-controlled machine — a classic "evil DHCP" / man-in-the-middle setup|DHCP snooping (validating DHCP messages against a trusted list of legitimate server ports) on managed switches is a standard mitigation|
|**IPv6 tunneling can smuggle traffic past IPv4-only security devices**|An attacker (or misconfigured device) can tunnel unauthorized IPv6 traffic inside protocol-41 IPv4 packets, potentially bypassing firewalls/IDS that were configured only to inspect native IPv6 traffic and don't unpack tunneled payloads|Security appliances need explicit protocol-41 (IPv6-in-IPv4) tunnel inspection, not just native dual-stack awareness|

---

## Questions I Still Have

- [ ] The 16-bit datagram-length field theoretically allows up to 65,535 bytes, but real datagrams rarely exceed ~1,500 bytes because of Ethernet's MTU — what's the actual mechanism (Path MTU Discovery?) that lets a sender learn the _smallest_ MTU along an entire path before ever sending, especially now that IPv6 pushes all fragmentation responsibility onto end systems?
- [ ] For CGN (Carrier-grade NAT) with double-NAT, does the ISP-side translation table risk running out of available (port, IP) combinations faster than a normal home NAT, given it's now servicing potentially tens of thousands of home networks through the SAME small pool of public addresses?
- [ ] With IPv6's anycast addressing, how does the network actually decide which member of the anycast group is "nearest" — is this purely BGP-path-length based, or does latency/RTT factor in at all?
- [ ] The note says DHCP offers can arrive from multiple servers on a subnet — is there a formal tie-breaking or preference rule the client applies beyond "first offer received," or is it purely implementation-defined?
- [ ] Given how hard network-layer protocol change is (the "foundation vs. paint" analogy), what specific technical or economic mechanism, if any, is expected to finally push the Internet past majority IPv6 adoption — is it pure IPv4-exhaustion pressure, or something else entirely?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Datagram**|The Internet's network-layer packet; the unit IP actually transmits|
|**Fragmentation**|Splitting an oversized IPv4 datagram into several smaller ones that travel independently and are reassembled at the destination; **not supported at all in IPv6**|
|**Interface**|The boundary between a host/router and one of its physical links; IP addresses are technically bound to interfaces, not to hosts or routers themselves|
|**Subnet**|A set of device interfaces that can reach each other without passing through any router; found by conceptually detaching every interface and identifying the resulting isolated network islands|
|**CIDR (Classless Interdomain Routing)**|The Internet's current address-assignment strategy; divides a 32-bit address into an `x`-bit network prefix and a `(32−x)`-bit host-identifying suffix, written `a.b.c.d/x`|
|**Classful addressing**|The pre-CIDR scheme constraining network portions to exactly 8, 16, or 24 bits (Class A/B/C); replaced due to severe address-space waste|
|**Network prefix**|The `x` most significant bits of a CIDR address, shared by every device within an organization|
|**Address aggregation (route summarization)**|Advertising one prefix to represent many downstream, more-specific sub-networks, dramatically shrinking external routing-table sizes|
|**DHCP (Dynamic Host Configuration Protocol)**|A plug-and-play protocol that automatically allocates an IP address (and subnet mask, default gateway, DNS server) to a newly connecting host|
|**Default gateway**|The address of a host's first-hop router, typically learned via DHCP|
|**Address lease time**|The duration for which a DHCP-allocated IP address remains valid before needing renewal|
|**DHCP relay agent**|A device (typically a router) that forwards DHCP messages to a DHCP server when none exists on the local subnet|
|**NAT (Network Address Translation)**|A technique letting an entire private network share a single public IP address, via port-number-based translation table entries|
|**Private address space**|Address blocks (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) reserved for use only within private networks, never globally routable|
|**NAT translation table**|The table mapping WAN-side (public IP, port) pairs to LAN-side (private IP, port) pairs, enabling correct reverse-direction routing of replies|
|**Carrier-grade NAT (CGN)**|Large-scale NAT deployment by an ISP itself, translating addresses for many home routers, often creating a "double-NAT" scenario|
|**NAT traversal**|Technical workarounds enabling inbound P2P connections to reach hosts sitting behind a NAT|
|**Firewall**|A device inspecting datagram/segment headers and denying suspicious traffic entry into an internal network|
|**IDS (Intrusion Detection System)**|A device performing deep packet inspection against a database of known attack signatures, generating alerts on matches|
|**IPS (Intrusion Prevention System)**|Like an IDS, but actively blocks offending packets rather than only alerting|
|**IPv6**|The successor network-layer protocol with 128-bit addresses and a streamlined, fixed 40-byte header, designed primarily to address IPv4 exhaustion|
|**Anycast address**|An IPv6 address type that delivers a datagram to any one member of a designated group of hosts (e.g. the nearest mirror server)|
|**Flow label**|A 20-bit IPv6 field identifying datagrams belonging to a particular flow requiring special handling|
|**Tunneling**|Encapsulating an entire datagram of one protocol (e.g. IPv6) inside the payload of a datagram of another protocol (e.g. IPv4) so it can transit routers that don't understand the inner protocol|
|**Dual-stack**|A device or network configured to handle both IPv4 and IPv6 natively|

---

## Related Concepts

- [[4.2 What's Inside a Router?]] — Section 4.2.1's longest-prefix-match lookup is the exact same mechanism reused at the CIDR/global-routing scale in this note's address-aggregation examples
- [[3.7 Congestion Control|ECN]] — Two IPv4 TOS bits are reused for Explicit Congestion Notification, covered in Chapter 3

---

→ Next: [[4.4 Generalized Forwarding and SDN]]