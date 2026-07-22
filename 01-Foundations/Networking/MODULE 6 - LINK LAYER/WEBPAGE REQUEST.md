---
title: WEBPAGE REQUEST
date: 2026-07-22
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 6.7 Retrospective: A Day in the Life of a Web Page Request

> **One-Line Summary:** This retrospective section stitches together **DHCP, ARP, DNS, intra-domain routing, TCP's three-way handshake, and HTTP** — protocols studied in isolation across five chapters — into a single **integrated, holistic (whole-system)** trace of everything that happens **under the hood** when a student, Bob, boots his laptop and simply requests **www.google.com**'s home page.

---

## Core Idea: Zooming Out to See the Whole Machine

Having now covered the **link layer**, the book's journey down the protocol stack is **complete** — application, transport, network, and link layers have each been studied in their own chapter. Before moving into the second half of the book's more specialized topics, this section pauses to take an **integrated, "big picture" view**: instead of asking "what does protocol X do?" one at a time, it asks what **many protocols together** must do to satisfy even the **simplest** possible request — fetching one Web page.

The scenario (Figure 6.34): a student, **Bob**, connects his laptop to his school's **Ethernet switch**. The school's router connects to an ISP, **comcast.net**, which in this example also happens to **host the DNS service** for the school (so the DNS server lives in Comcast's network rather than the school's own). Bob wants to load the home page of **www.google.com**, served from Google's network. Nothing about this is exotic — it's the most mundane possible act — and yet, as the walkthrough shows, it takes **dozens of messages across four protocol layers and three separate networks** before a single pixel of the page reaches Bob's screen.

```
Fig -- Network Setting (Figure 6.34)
────────────────────────────────────────
 Bob's laptop                comcast.net
 00:16:D3:23:68:8A                DNS server
 68.85.2.101                 68.87.71.226
      │                             ▲
      ▼                             │
  [Ethernet switch]                 │
      │                             │
      ▼                             │
  School router 68.85.2.1           │
  MAC 00:22:6B:45:1F:1B             │
  (School net 68.80.2.0/24)         │
      │                             │
      ▼                             │
  Comcast's network 68.80.0.0/13 ───┘
      │
      ▼
  Google's network 64.233.160.0/19
      │
      ▼
  www.google.com Web server
  64.233.169.105
```

![[Pasted image 20260722000001.png]]

> **Analogy — Watching a Play from Backstage Instead of the Audience:** Reading about DHCP, DNS, TCP, and HTTP separately, chapter by chapter, is like learning each actor's individual lines and blocking (movements) in a play, one actor at a time, in an empty rehearsal room. This section is like finally watching **opening night from backstage**: you see how the lighting cue, the sound cue, and three actors' entrances all have to line up **within seconds of each other** for a single scene to work — the "big picture" only becomes visible once you watch the whole cast perform **together**, in order, under real time pressure.

---

## 6.7.1 Getting Started: DHCP, UDP, IP, and Ethernet

When Bob's laptop first boots and connects to the Ethernet cable, it **can't do anything** — not even send a single useful packet — because it doesn't yet have an IP address. The very first network-related action the laptop takes is therefore to run **DHCP** to obtain an IP address (and other configuration) from the local DHCP server, which in this scenario runs inside the school's router.

The **why** here matters: every protocol above the link layer (IP, TCP, UDP, DNS, HTTP) needs a source IP address to build its messages, so **nothing else in this entire scenario can begin** until DHCP completes. DHCP is thus the true "step zero" of getting online.

**The seven-step DHCP exchange:**

1. Bob's laptop's OS creates a **DHCP request message**, placed inside a **UDP segment** (destination port 67 for the DHCP server, source port 68 for the DHCP client).
2. That UDP segment is placed inside an **IP datagram** with a **broadcast** destination IP of 255.255.255.255 and a source IP of 0.0.0.0, since the laptop has no IP address yet to put as its source.
3. The IP datagram is placed inside an **Ethernet frame** with destination MAC **FF:FF:FF:FF:FF:FF** (broadcast) so every device on the switch — hopefully including a DHCP server — receives it; the source MAC is the laptop's own, 00:16:D3:23:68:8A. This broadcast frame is sent to the switch, which floods it out every port except the one it arrived on.
4. The router receives the broadcast frame on its interface (MAC 00:22:6B:45:1F:1B). Because the datagram's destination IP is a broadcast address, the router's own upper-layer protocols process it: the payload is **demultiplexed** up to UDP, and the DHCP request is extracted.
5. The DHCP server running inside the router allocates an address from its **CIDR** block (68.85.2.0/24) — say, **68.85.2.101** — and builds a **DHCP ACK message** containing that IP, the default gateway's IP (68.85.2.1), the DNS server's IP (68.87.71.226), and the subnet mask (68.85.2.0/24). This is wrapped in UDP, then IP, then an Ethernet frame addressed (unicast, not broadcast) to the laptop's MAC.
6. The switch forwards this Ethernet frame **only out the port leading to Bob's laptop**, because the switch is **self-learning** — having already seen a frame from the laptop's MAC address (the original broadcast request), it knows exactly which port to use instead of flooding again.
7. Bob's laptop receives the DHCP ACK, extracts its assigned IP address and the default gateway/DNS server addresses, and installs a default-gateway entry in its **IP forwarding table** (any destination outside 68.85.2.0/24 goes to the gateway). The laptop is now network-configured. _(Notably, the text points out that of the four DHCP steps normally described in Chapter 4 — discover, offer, request, ACK — only the last two are strictly necessary here, since this trace assumes the router already knows to simply request-and-ack rather than run a full discovery round.)_

```
Fig -- DHCP: Getting an IP Address
────────────────────────────────────────
 Bob's laptop            Switch    Router
 (no IP yet)                    (DHCP srv)
     │                             │
     │--DHCP REQUEST (broadcast)-->│
     │  src 0.0.0.0 : 68           │
     │  dst 255.255.255.255 : 67   │
     │                             │
     │<--DHCP ACK (unicast)--------│
     │   IP:      68.85.2.101      │
     │   Gateway: 68.85.2.1        │
     │   DNS:     68.87.71.226     │
     ▼                             ▼
 Laptop now configured      Router/switch now
 (IP, gateway, DNS)         know laptop's MAC
```

![[Pasted image 20260722000002.png]]

> **Analogy — Checking Into a Hotel:** DHCP is like arriving at a hotel with no reservation confirmation in hand. You **announce yourself at the front desk** (broadcast request) rather than mailing a letter to a specific person, because you don't yet know who's working the desk. The front desk hands you back a **room key, a map showing the elevator to the lobby (default gateway), and the phone number for the concierge (DNS server)** — all four things you need before you can do anything else in the building. Only after checking in can you sensibly ask the concierge to look anything up for you.

---

## 6.7.2 Still Getting Started: DNS and ARP

With an IP address in hand, Bob's Web browser begins the process of loading www.google.com by creating a **TCP socket** that will eventually carry an **HTTP request**. But to open a TCP connection, the browser first needs **www.google.com's IP address** — which means invoking **DNS**.

**Step 1 — building the DNS query:** The laptop's OS creates a **DNS query message** with the question "www.google.com?", placed in a UDP segment (destination port 53), then an IP datagram addressed to the DNS server's IP (68.87.71.226, learned from the DHCP ACK) with source IP 68.85.2.101.

**The snag — a missing MAC address:** The laptop knows the school gateway router's **IP** address (68.85.2.1, from DHCP) but not its **MAC** address, and an Ethernet frame absolutely requires a destination MAC. This is precisely the gap **ARP** exists to fill.

**The ARP resolution (steps 2–5):** 2. The laptop creates an **ARP query message** — target IP 68.85.2.1 — inside a broadcast Ethernet frame (dst MAC FF:FF:FF:FF:FF:FF) and sends it to the switch, which floods it to all connected devices, including the gateway router. 3. The gateway router recognizes the target IP (68.85.2.1) as its own interface and prepares an **ARP reply**, stating its MAC address is 00:22:6B:45:1F:1B. 4. The router places this reply in an Ethernet frame addressed (unicast) to the laptop's MAC and sends it via the switch. 5. The laptop receives the ARP reply and now has the gateway's MAC address cached.

**Finishing the DNS query (step 6):** the laptop can _finally_ address the Ethernet frame carrying its DNS query. Note the layering subtlety here: the **IP datagram's destination** is the DNS server (68.87.71.226) far away in Comcast's network, but the **Ethernet frame's destination** is only the gateway router's MAC (00:22:6B:45:1F:1B) — the very next hop. The frame only needs to get as far as the next device on the local wire; IP routing takes over from there.

```
Fig -- ARP: Finding the Gateway's MAC
────────────────────────────────────────
 Bob's laptop                     Router
 knows gateway IP                 68.85.2.1
 68.85.2.1, not its MAC
     │                               │
     │--ARP query (broadcast)------->│
     │  "Who has 68.85.2.1?"         │
     │                               │
     │<--ARP reply (unicast)---------│
     │  "68.85.2.1 is at             │
     │   00:22:6B:45:1F:1B"          │
     ▼                               ▼
 Laptop can now address        (no change --
 Ethernet frames to the        router just
 gateway's real MAC            answers)
```

![[Pasted image 20260722000003.png]]

> **Analogy — Knowing Someone's Name but Not Their Extension:** DNS is like knowing a company's **name** ("www.google.com") but needing the **phone directory service** to give you a number to actually dial (an IP address). ARP is a _different_, more local problem: you already have your own **receptionist's four-digit internal extension** (the gateway's IP) written on a sticky note, but internal phone systems route by **physical line number**, not extension — so before you can even dial the receptionist to relay your outside call, you have to ask "which physical line is extension 2.1 on?" That's ARP: translating a known local address into the physical (MAC) identifier the wire itself understands.

---

## 6.7.3 Still Getting Started: Intra-Domain Routing to the DNS Server

The gateway router now has the DNS query datagram and must get it to 68.87.71.226 inside Comcast's network. This is where **intra-domain routing** — the subject of Section 5.3 — takes over.

**The path to the DNS server:**

1. The gateway router looks up 68.87.71.226 in its forwarding table and determines the datagram should go to the leftmost router in Comcast's network, sending the frame over the link connecting school and Comcast.
2. That Comcast router extracts the datagram, examines the destination IP, and forwards it onward toward the DNS server using its own forwarding table — a table that was populated by an **intra-domain routing protocol** such as **RIP, OSPF, or IS-IS**. (Note that if the DNS server had lived outside Comcast entirely, **BGP**, the Internet's inter-domain protocol, would instead govern how the datagram crosses between Comcast and whatever other network hosted it.)
3. The datagram eventually arrives at the DNS server, which extracts the query, looks up "www.google.com" in its database, and finds a **cached DNS resource record** mapping it to **64.233.169.105** — cached data that originally came from google.com's own **authoritative DNS server** (Section 2.5.2), but which the Comcast resolver can now answer directly from its own cache without re-querying anyone.
4. The DNS server packages this answer as a **DNS reply message**, placed in a UDP segment and IP datagram addressed back to Bob's laptop (68.85.2.101), and forwards it back through Comcast's network, the school's router, and the Ethernet switch. Bob's laptop _finally_ has Google's IP address.

```
Fig -- DNS Query Reaches the DNS Server
────────────────────────────────────────
 Bob's laptop        Comcast intra-domain      DNS
 (via gateway)        routing (RIP/OSPF/    server
                       IS-IS) forwards it   68.87.71.226
      │                                          │
      │---DNS query: "www.google.com?"-------->│
      │   dst IP 68.87.71.226                    │
      │                                          │
      │<--DNS reply: cached A record-------------│
      │   www.google.com = 64.233.169.105        │
      ▼                                          ▼
 Laptop now has Google's IP, ready
 to open a TCP socket to it
```

![[Pasted image 20260722000004.png]]

> **Analogy — Domestic Mail vs. an International Courier Handoff:** Getting a datagram from Bob's laptop to the DNS server, entirely inside Comcast's network, is like **domestic mail routing within one country's postal service** — a single organization's internal sorting facilities (routers running RIP/OSPF/IS-IS) know exactly how to route a letter (the DNS query) from a local post office to any address within that same country, using nothing but their own internal address book. If the destination had instead been in a **different** country (a different Autonomous System), the letter would need to be handed off at a border crossing according to a treaty between postal services — that handoff protocol is BGP, and it only comes into play the moment a packet needs to leave the network that originated it.

---

## 6.7.4 Web Client-Server Interaction: TCP and HTTP

With Google's IP address (64.233.169.105) finally in hand, Bob's browser can create the **TCP socket** it needs to send an **HTTP GET** request.

**Establishing the TCP connection:**

1. Bob's laptop's TCP creates a **TCP SYN segment** with destination port 80 (HTTP), placed inside an IP datagram (destination IP 64.233.169.105), placed inside a frame addressed to the gateway router's MAC — this is the **first step of the three-way handshake**.
2. The routers in the school network, Comcast's network, and Google's network forward the SYN datagram using their respective forwarding tables — the same intra-domain (RIP/OSPF/IS-IS) and inter-domain (BGP) machinery from Section 6.7.3 governs this forwarding, just now carrying TCP traffic instead of a DNS query.
3. The SYN arrives at www.google.com. It is demultiplexed to the server's **welcome socket** on port 80, a new **connection socket** is created specifically for this TCP connection with Bob's laptop, and the server generates a **TCP SYNACK** segment in reply, addressed back to Bob's laptop.
4. The SYNACK is forwarded back through Google's, Comcast's, and the school's networks, arrives at Bob's laptop, and is demultiplexed to the socket created in step 1 — completing the handshake and bringing the socket into the **connected** state.
5. With the socket connected, Bob's browser writes the **HTTP GET message** requesting the page (naming the URL to fetch) into the socket; this becomes the payload of a TCP segment, placed in a datagram, and sent to www.google.com.
6. The HTTP server at www.google.com reads the GET message from its TCP socket, builds an **HTTP response message** containing the requested page content in its body, and sends it back into the socket.
7. The datagram carrying the HTTP response is forwarded back through all three networks and arrives at Bob's laptop. His browser reads the response from the socket, extracts the page's HTML from the response body, and — _finally_ — **displays the Web page.**

```
Fig -- TCP 3-Way Handshake + HTTP
────────────────────────────────────────
 Bob's laptop                  www.google.com
     │                                │
     │---TCP SYN, dst port 80------->│
     │                                │
     │<---TCP SYNACK-----------------│
     │   (connection socket created) │
     │                                │
     │---ACK / HTTP GET------------->│
     │   socket now "connected"       │
     │                                │
     │<---HTTP response (Web page)---│
     ▼                                ▼
 Browser renders            Server read GET,
 the Web page               sent page content
```

![[Pasted image 20260722000005.png]]

> **Analogy — Answering the Phone Before Talking Business:** The TCP three-way handshake is like a **phone call's opening ritual** before any real conversation happens: "Hello?" (SYN) → "Hello, who's calling?" (SYNACK) → "It's me, ready to talk" (final ACK, here carrying the HTTP GET as its first real content). Only once both parties have confirmed they can hear each other does the actual **business of the call** — the HTTP request and response, like asking a librarian for a specific book and having them hand it over — take place.

---

## Coda: What This Walkthrough Deliberately Left Out

The book is explicit that this integrated trace, while covering **a lot** of ground, is still a **simplified, "nuts and bolts"** view focused on **how** the protocols interoperate rather than **why** they're designed the way they are. Several real-world elements are noted as intentionally **omitted** from the scenario for clarity: **NAT** running in the school's gateway router, **wireless access** to the school's network, **security protocols** for encrypting segments/datagrams or for accessing the school's network, and **network management protocols**. The retrospective also leaves aside broader considerations like **DNS hierarchy** and **web caching**, which the book takes up in its second half. Readers wanting the more **reflective, "why"-oriented** counterpart to this "how"-oriented walkthrough are pointed back to the **"Architectural Principles of the Internet"** discussion in Section 4.5.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Broadcast DHCP and ARP messages**|Since both the DHCP request and the ARP query are broadcast to every device on the local Ethernet segment, an attacker sharing that segment could race to send a **forged DHCP ACK or ARP reply** before the legitimate server/router answers, redirecting Bob's laptop to a rogue gateway, DNS server, or MAC binding|Switch-level protections such as **DHCP snooping** and **dynamic ARP inspection** validate that DHCP/ARP replies originate only from trusted, designated ports before forwarding them onward|
|**Cached DNS records at the resolver**|Because Comcast's DNS server answers www.google.com's address straight from its own **cache** rather than re-querying the authoritative server every time, a successful cache-poisoning attack against that one resolver could silently redirect every client relying on the poisoned record until it expires|**DNSSEC** and strict validation of DNS response authenticity reduce the chance of a forged record ever being accepted into the cache in the first place|
|**Spoofed TCP SYN segments (unauthenticated handshake)**|Anyone able to spoof a source IP address can send SYN segments toward www.google.com claiming to be Bob's laptop, potentially exhausting server-side connection state (a **SYN flood**) without ever completing a real handshake|**SYN cookies** let a server avoid committing real per-connection state until the handshake's final ACK proves the client can actually complete the round trip|
|**Plaintext HTTP request and response**|Because this scenario uses **HTTP**, not HTTPS, anyone observing traffic anywhere along the path — the school network, Comcast, or Google's network — can read the exact URL requested and the full page content in the clear|Encrypting the exchange with **HTTPS/TLS** (deliberately outside this simplified walkthrough, per the text's own note about omitted security protocols) protects the confidentiality and integrity of the request and response|
|**No NAT or internal-topology hiding in this trace**|Unlike the data-center load-balancer scenario (Section 6.6), this walkthrough explicitly has **no NAT** at the school gateway, so Bob's real internal IP and MAC addresses are visible, unobscured, at every hop of the path|Production access networks typically combine this basic DHCP/ARP/DNS/TCP/HTTP stack with NAT, firewalls, and encryption layers that this idealized retrospective intentionally strips away for teaching clarity|

---

## Questions I Still Have

- [ ] The text notes only the last two of DHCP's usual four steps (discover/offer/request/ACK) are shown here — what assumption lets the scenario skip straight to request-and-ACK, and what would have to happen differently if that assumption didn't hold?
- [ ] How long is a typical DNS **TTL** for a domain like google.com, and how would this entire trace change if the cached record at Comcast's resolver had already expired, forcing a full referral chain back to an authoritative server?
- [ ] The three-way handshake description has Bob's laptop send an "ACK / HTTP GET" as one combined step — is the handshake's final ACK actually sent as its own bare TCP segment, or is it standard practice to piggyback the first data (the GET) directly onto that same ACK segment?
- [ ] Both the DHCP request and the ARP query use broadcast Ethernet frames, yet the DHCP ACK and ARP reply are forwarded as unicast by a **self-learning** switch — what specifically does the switch learn from a broadcast frame that lets it avoid flooding the _reply_?
- [ ] Given that a datagram between Comcast and Google's network would, in the general case, need to cross an inter-domain BGP boundary, why does this particular trace (Section 6.7.3) never need to invoke BGP explicitly?
- [ ] The text explicitly omits NAT, wireless access, security protocols, and network management protocols from this scenario — roughly how many additional messages would a **realistic** version of this same page load involve once TLS negotiation and Wi-Fi association are added back in?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**DHCP (Dynamic Host Configuration Protocol)**|Protocol that lets a host obtain an IP address and other configuration (gateway, DNS server, subnet mask) automatically from a DHCP server upon connecting to a network|
|**DHCP request / DHCP ACK message**|The client's broadcast message requesting configuration, and the server's message (here sent unicast) granting an IP address and network settings in reply|
|**UDP segment**|The transport-layer container used by DHCP and DNS messages, providing simple, connectionless delivery without the overhead of a handshake|
|**Broadcast IP/MAC address**|A special destination address (255.255.255.255 for IP, FF:FF:FF:FF:FF:FF for Ethernet) meaning "deliver to every device on this network/segment"|
|**Self-learning switch**|An Ethernet switch that builds a table of which MAC addresses are reachable via which ports by observing the source addresses of frames it has already seen, allowing it to forward later frames as unicast instead of flooding|
|**ARP (Address Resolution Protocol)**|Protocol used to discover the MAC address corresponding to a known IP address on the same local network, via a broadcast query and a unicast reply|
|**DNS query / DNS reply message**|The message asking for a hostname-to-IP-address mapping, and the message returning that mapping (or an error) from a DNS server|
|**DNS resource record**|A database entry at a DNS server mapping a hostname to an IP address (among other record types), which may be freshly authoritative or simply cached|
|**Authoritative DNS server**|The DNS server that is the ultimate source of truth for a given domain's records, from which other resolvers' cached copies originally derive|
|**Intra-domain routing protocol (RIP, OSPF, IS-IS)**|A routing protocol used to build forwarding tables _within_ a single network/AS, governing how packets are routed to destinations inside that same administrative domain|
|**BGP (Border Gateway Protocol)**|The Internet's inter-domain routing protocol, governing how packets are routed _between_ separate networks/Autonomous Systems|
|**TCP socket**|An endpoint of a TCP connection, created by an application (like a Web browser) to send and receive a reliable, ordered byte stream|
|**Three-way handshake**|The SYN / SYNACK / ACK exchange TCP uses to establish a connection before any application data is reliably exchanged|
|**TCP SYN / SYNACK segment**|The first and second segments of the three-way handshake, used to synchronize sequence numbers and establish the connection|
|**HTTP GET message / HTTP response message**|The client's request for a specific Web resource, and the server's reply containing the requested content (or an error) in its body|

---

## Related Concepts

---

→ Next: Chapter 7 — Wireless and Mobile Networks