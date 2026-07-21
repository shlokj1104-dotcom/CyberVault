---
title: SWITCHED LANS
date: 2026-07-21
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 6.4 Switched Local Area Networks

> **One-Line Summary:** A **switched LAN** connects hosts, servers, and routers via **link-layer switches** that forward **frames** using flat, universal **MAC addresses** (resolved from IP addresses via **ARP**) rather than hierarchical IP addresses; the dominant switched-LAN technology is **Ethernet**, whose switches are **self-learning**, **plug-and-play**, and collision-free — and can be further subdivided into isolated **VLANs (Virtual LANs)** to curb the proliferation (uncontrolled spread) of broadcast traffic across an institution.

---

## Core Idea: From Broadcast Coordination to Switched Forwarding

Having examined broadcast networks and multiple access protocols in Section 6.3, this section pivots (shifts focus) to **switched local networks**. Figure 6.15 depicts an institutional network connecting three departments, two servers, and a router — all interconnected by **four switches**. Because these switches operate at the link layer, they switch **link-layer frames** (rather than network-layer datagrams), are oblivious to (unaware of) network-layer addresses, and don't rely on routing algorithms like OSPF to determine paths through the network of layer-2 switches. Instead of IP addresses, switches use **link-layer addresses** to forward frames through the network.

This section builds up switched LANs in four stages:

1. **Link-layer addressing and ARP** (6.4.1)
2. **The celebrated (widely admired) Ethernet protocol** (6.4.2)
3. **How link-layer switches operate** (6.4.3)
4. **How switches build large-scale LANs, including VLANs** (6.4.4)

---

## 6.4.1 Link-Layer Addressing and ARP

Hosts and routers have link-layer addresses — a fact that may seem redundant (unnecessarily repetitive) given that hosts and routers already possess network-layer addresses. This subsection elucidates (clarifies) why two separate layers of addressing turn out to be genuinely indispensable (impossible to do without), and introduces the **Address Resolution Protocol (ARP)**, the mechanism that translates between them.

### MAC Addresses

A crucial, and initially counterintuitive, nuance (subtle distinction): it is **not** hosts and routers themselves that possess link-layer addresses, but rather their **adapters** (network interfaces). A host or router with multiple network interfaces will accordingly (correspondingly) have multiple link-layer addresses associated with it — just as it would have multiple IP addresses.

```
Fig -- Adapters Have MAC Addresses
────────────────────────────────────────
 Host A         Host B         Host C
 1A-23-F9...    5C-66-AB...    49-BD-D2...
    │               │              │
    └───────┐   ┌────┘    ┌────────┘
             ▼   ▼        ▼
           [ Switch ]───[Router]
                          88-B2-2F...

 Each ADAPTER (interface) has its own
 fixed, flat MAC address -- not the
 host itself.
```

![[Pasted image 20260718000014.png]]

Notably, link-layer **switches** do **not** have link-layer addresses associated with the interfaces that connect to hosts and routers — this is because the job of a switch is to carry datagrams between hosts and routers **transparently** (invisibly, without the sender needing to know), without the host or router having to explicitly address the frame to the intervening (in-between) switch.

A link-layer address is variously called a **LAN address**, a **physical address**, or a **MAC address**. Because "MAC address" is the most popular term, this document will henceforth (from this point onward) refer to link-layer addresses as MAC addresses. For most LANs (including Ethernet and 802.11 wireless LANs), the MAC address is **6 bytes long**, giving **2⁴⁸ possible MAC addresses**. These 6-byte addresses are typically expressed in **hexadecimal notation**, with each byte expressed as a pair of hexadecimal digits. Although MAC addresses were originally designed to be permanent, it is now possible to change an adapter's MAC address via software.

### The Flat Structure of MAC Addresses

One salient (noteworthy) property of MAC addresses is that **no two adapters have the same address**. This might seem surprising given that adapters are manufactured in many countries by many companies — how does a company manufacturing adapters in Taiwan ensure it isn't duplicating (repeating) addresses used by a company in Belgium? The answer: the **IEEE manages the MAC address space**. When a company wants to manufacture adapters, it purchases a chunk of the address space consisting of 2²⁴ addresses for a nominal (small, token) fee. IEEE allocates this chunk by fixing the first 24 bits of a MAC address and letting the company create unique combinations of the last 24 bits for each adapter.

An adapter's MAC address has a **flat structure** (as opposed to a **hierarchical** one) and doesn't change no matter where the adapter physically relocates. A laptop with an Ethernet interface always retains the same MAC address, no matter where the computer travels.

|Address Type|Structure|Changes When Device Moves?|Analogy|
|---|---|---|---|
|**MAC address**|Flat (no embedded network hierarchy)|**No** — permanent, tied to the adapter itself|A person's **social security number**|
|**IP address**|Hierarchical (network part + host part)|**Yes** — must be updated to reflect the new network attached to|A person's **postal address**|

> **Analogy — A Permanent ID Card vs. a Mailing Address:** Just as it's genuinely useful (advantageous) for a person to possess **both** a postal address and a social security number — one that stays constant regardless of relocation, one that reflects wherever they currently reside — it's similarly beneficial for a host or router interface to have **both** a network-layer address and a MAC address, each serving a distinct, complementary (mutually reinforcing) purpose.

### How a Frame Actually Gets Addressed

When an adapter wants to send a frame to some destination adapter, the sending adapter inserts the **destination adapter's MAC address** into the frame and then sends the frame into the LAN. As will soon become apparent, a switch occasionally broadcasts an incoming frame onto **all** of its interfaces (802.11 similarly broadcasts frames, a topic covered in Chapter 7). Thus, an adapter may receive a frame that isn't addressed to it. Consequently, when an adapter receives a frame, it checks whether the destination MAC address in the frame matches its own MAC address.

- **If there's a match**, the adapter extracts the enclosed datagram and passes it up the protocol stack.
- **If there isn't a match**, the adapter **discards** the frame, without passing the network-layer datagram up. Thus, the destination host is only interrupted when a frame genuinely intended for it is received.

However, a sending adapter sometimes **does** want all other adapters on the LAN to receive and process the frame it's about to send. In this case, it inserts a special MAC **broadcast address** into the destination address field. For LANs that use 6-byte addresses (such as Ethernet and 802.11), the broadcast address is a string of **48 consecutive 1s** (that is, **FF-FF-FF-FF-FF-FF** in hexadecimal notation).

### Address Resolution Protocol (ARP)

Because there exist **both** network-layer addresses (e.g., IP addresses) **and** link-layer addresses (MAC addresses), there arises a genuine need to **translate** between them. For the Internet, this is the job of the **Address Resolution Protocol (ARP)** [RFC 826].

```
Fig -- ARP: Resolving IP to MAC on a LAN
────────────────────────────────────────
 Host A                 Host B
 IP 222.222.222.220   IP 222.222.222.222
 MAC 1A-23-F9-CD-06-9B MAC 49-BD-D2-C7-56-2A

 A wants to send to B, knows only B's
 IP.  A broadcasts an ARP query:
   "Who has 222.222.222.222?"
 Every adapter on the LAN receives it;
 only B replies with its MAC address.
 A caches the mapping in its ARP table.
```

![[Pasted image 20260718000015.png]]

Suppose host **222.222.222.220** wants to send an IP datagram to host **222.222.222.222**, where both source and destination reside in the same subnet. To send a datagram, the source must furnish (provide) its adapter not only with the IP datagram but **also with the MAC address for destination 222.222.222.222**. The sending adapter will then construct a link-layer frame containing the destination's MAC address and send the frame into the LAN.

**The pivotal question:** how does the sending host determine the MAC address for a destination host given only its IP address? By using **ARP**. An ARP module in the sending host takes any IP address on the same LAN as input, and returns the corresponding MAC address.

So ARP resolves an IP address to a MAC address. In many ways it is analogous to **DNS**, which resolves host names to IP addresses. However, one important difference: **DNS resolves host names for hosts anywhere on the Internet**, whereas **ARP resolves IP addresses only for hosts and router interfaces on the same subnet**. If a node in California were to try to use ARP to resolve the IP address for a node in Mississippi, ARP would return an error.

### The ARP Table

Every host and router maintains an **ARP table** in its memory, containing mappings of IP addresses to MAC addresses, along with a **time-to-live (TTL)** value indicating when each mapping should be purged (removed) from the table.

```
Fig -- A Sample ARP Table (at host A)
────────────────────────────────────────
  IP Address         MAC Address     TTL
  222.222.222.221 88-B2-2F-54-1A-0F 13:45
  222.222.222.223 5C-66-AB-90-75-B1 13:52

 Not every host/router on the subnet
 need appear; entries expire (~20 min)
 and are then re-queried via ARP.
```

![[Pasted image 20260718000016.png]]

Note that a table need **not** contain an entry for every host and router on the subnet — some may never have been entered into the table, and others may have expired. A **typical expiration time** for an entry is **20 minutes** from when it was placed in the table.

**What happens when there's no entry?** Suppose the sender wants to send a datagram to a destination for which the ARP table doesn't currently have an entry. The sender constructs a special packet called an **ARP packet**, with fields including sending and receiving IP and MAC addresses. Both ARP **query** and **response** packets share the same format. The purpose of the ARP query packet is to query all other hosts and routers on the subnet to determine the MAC address corresponding to the IP address being resolved.

> **Analogy — Shouting a Question Across a Crowded Room:** Recalling the social security number/postal address analogy, an ARP query is equivalent to a person **shouting out** in a crowded room of cubicles at some company: "What is the social security number of the person whose postal address is Cubicle 13, Room 112, AnyCorp, Palo Alto, California?" Everyone in earshot (within hearing range) hears the question, but only the one person whose postal address genuinely matches bothers to **reply** with their answer.

The sending host passes the ARP query packet to the adapter, along with an instruction to send the packet to the MAC **broadcast** address (FF-FF-FF-FF-FF-FF). The frame containing the ARP query is received by **all** other adapters on the subnet, and each adapter passes the ARP packet up to its ARP module. Each module checks whether its own IP address matches the destination IP address in the packet — the one with a match sends back a **response ARP packet** containing the desired mapping. The querying host can then update its ARP table and send its IP datagram, encapsulated in a link-layer frame whose destination MAC is that of the host or router that responded.

**Two interesting properties of ARP:**

1. **The query is broadcast, but the response is unicast.** The ARP query message is sent within a **broadcast** frame, whereas the response message is sent within a **standard (unicast)** frame — worth pondering (thinking carefully about) why this asymmetry (imbalance) makes sense.
2. **ARP is plug-and-play.** An ARP table gets built automatically — it doesn't require configuration by a system administrator. And if a host becomes disconnected from the subnet, its entry is eventually purged from other hosts' ARP tables.

**Is ARP a link-layer protocol, or a network-layer protocol?** Students often ponder this question. An ARP packet is encapsulated within a link-layer frame, and thus architecturally lies **above** the link layer. However, an ARP packet has fields containing link-layer addresses and is thus arguably a link-layer protocol — but it also contains network-layer addresses and is thus arguably a network-layer protocol too. In the end, ARP is probably best considered a protocol that **straddles (sits across) the boundary** between the link and network layers — not fitting neatly into the tidy, simplified layered protocol stack introduced in Chapter 1. Such are the complexities of real-world protocols!

### Sending a Datagram off the Subnet

Now consider the more elaborate (intricate) scenario of a host on one subnet wanting to send a datagram to a host **off the subnet** (that is, across a router). Figure 6.19 depicts a network of two subnets interconnected by a router.

```
Fig -- Sending a Datagram Off the Subnet
────────────────────────────────────────
 Subnet 1 (111.111.111.xxx)
   Host ---> Router iface 111.111.111.110
             (MAC found via ARP, Subnet1)
             Router
   Subnet 2 (222.222.222.xxx)
             Router iface 222.222.222.220
   Host <--- (MAC found via ARP, Subnet2)

 Frame's dest MAC = next hop (router),
 even though dest IP = final host.
 MAC address changes at every hop;
 IP address stays the same throughout.
```

![[Pasted image 20260718000017.png]]

Several noteworthy details:

- Each host has exactly one IP address and one adapter. A **router**, however, has an IP address for **each** of its interfaces, and correspondingly (accordingly) an ARP module and an adapter **per interface**.
- Subnet 1 has network address **111.111.111/24**, and Subnet 2 has network address **222.222.222/24**.

Suppose host **111.111.111.111** wants to send an IP datagram to host **222.222.222.222**. One might be tempted to guess the appropriate destination MAC address should be that of host 222.222.222.222 itself — but this guess would be **wrong**! Were the sending adapter to use that MAC address, none of the adapters on Subnet 1 would bother passing the datagram up to the network layer, since the frame's destination address wouldn't match any adapter on Subnet 1 — the datagram would simply die and "go to datagram heaven."

**The correct approach:** for a datagram to travel from 111.111.111.111 to a host on Subnet 2, the frame must be sent **first** to the router interface **111.111.111.110** — the IP address of the **first-hop router** on the path to the final destination. The appropriate destination MAC address is thus the address of the adapter for that router interface, obtained via **ARP**.

Once the sending host has this MAC address, it creates a frame (containing the datagram addressed to 222.222.222.222) and sends the frame into Subnet 1. The router adapter sees the frame is addressed to it, and passes the frame up to the router's network layer — the IP datagram has successfully moved from source host to router! But the journey isn't finished — the router still has to forward the datagram onward, consulting its **forwarding table** to determine the appropriate outgoing interface. This interface then passes the datagram to its own adapter, which encapsulates the datagram in a **new** frame and sends it into Subnet 2 — with the destination MAC address now genuinely being that of the ultimate destination (obtained, once again, via ARP).

**Key takeaway:** the frame's destination MAC address changes at **every** hop along the path, even though the datagram's destination IP address remains **unchanged** throughout the entire journey.

---

## 6.4.2 Ethernet

Ethernet has, for all intents and purposes (practically speaking), **taken over the wired LAN market**. In the 1980s and early 1990s, Ethernet faced considerable (substantial) competition from other LAN technologies, including token ring, FDDI, and ATM. Some of these succeeded in capturing a portion of the market for a few years, but since its invention in the mid-1970s, Ethernet has continued to evolve and grow, holding onto its dominant position. One might say Ethernet has been to local area networking what the Internet has been to global networking.

### Reasons for Ethernet's Success

1. **Ethernet was the first widely deployed high-speed LAN.** Because it was deployed early, network administrators became intimately familiar with it — its wonders and its quirks (idiosyncrasies, peculiar characteristics) — and were reluctant to switch to other technologies as they emerged.
2. **Token ring, FDDI, and ATM were more complex and expensive** than Ethernet, which further discouraged administrators from switching.
3. **The most compelling reason to switch to another technology** was usually the promise of a higher data rate — however, Ethernet always fought back, producing versions that operated at equal or higher data rates. **Switched Ethernet** was also introduced in the early 1990s, further boosting its effective data rates.
4. **Because Ethernet has been so ubiquitous (widespread, found everywhere)**, its hardware — adapters and switches in particular — has become a **commodity**, and is remarkably cheap.

### A Brief Evolutionary History

The original Ethernet LAN was invented in the mid-1970s by **Bob Metcalfe** and **David Boggs**, using a **coaxial bus** to interconnect nodes. Bus topologies persisted throughout the 1980s and into the mid-1990s. Ethernet with a bus topology is a **broadcast LAN** — all transmitted frames travel to, and are processed by, **all** adapters connected to the bus (recall Ethernet's original **CSMA/CD** multiple access protocol with binary exponential backoff, covered in Section 6.3.2).

By the late 1990s, most companies and universities had replaced their bus-topology LANs with **hub-based star topologies**. In such an installation, hosts (and routers) connect directly to a **hub** via twisted-pair copper wire. A hub is a **physical-layer** device that acts on individual bits rather than frames — when a bit arrives on one interface, the hub simply re-creates it, boosts its energy, and retransmits it onto all the other interfaces. Thus, Ethernet with a hub-based star topology is **still a broadcast LAN**.

In the early 2000s, Ethernet underwent yet another major evolutionary transformation: installations continued to use a star topology, but the **hub** at the center was replaced with a **switch**. Unlike a hub, a switch is not only "collision-less" but is also a **bona fide (genuine, authentic)** store-and-forward packet switch — though, unlike routers (which operate up through layer 3), a switch operates **only up through layer 2**.

### Ethernet Frame Structure

To ground this discussion, consider sending an IP datagram from one host to another, with both hosts on the same Ethernet LAN. Let the sending adapter, **adapter A**, have MAC address **AA-AA-AA-AA-AA-AA**, and the receiving adapter, **adapter B**, have MAC address **BB-BB-BB-BB-BB-BB**. The sending adapter encapsulates the IP datagram within an Ethernet frame and passes the frame to the physical layer; the receiving adapter extracts the datagram and passes it up to the network layer.

```
Fig -- Ethernet Frame Structure
────────────────────────────────────────
┌────────┬──────┬──────┬────┬─────┬───┐
│Preamble│ Dest │Source│Type│Data │CRC│
│ 8 bytes│ 6 B  │ 6 B  │2 B │46-  │4 B│
│        │      │      │    │1500B│   │
└────────┴──────┴──────┴────┴─────┴───┘
 Preamble: wakes/syncs receiver clock
 Type: which network-layer protocol
       (demultiplexing, e.g. IP, ARP)
 CRC: bit-error detection (Sec 6.2.3)
```

![[Pasted image 20260718000018.png]]

|Field|Size|Purpose|
|---|---|---|
|**Preamble**|8 bytes|Each of the first 7 bytes has the value **10101010**; the last byte is **10101011**. The first 7 bytes serve to "wake up" the receiving adapter and **synchronize its clock** with that of the sender. The last two 1s alert the receiving adapter that the "important stuff" is about to commence (begin).|
|**Destination address**|6 bytes|The MAC address of the destination adapter. If the address matches (or is the MAC broadcast address), the receiver passes the frame's data field content up to the network layer; otherwise, the frame is discarded.|
|**Source address**|6 bytes|The MAC address of the adapter that transmitted the frame onto the LAN.|
|**Type**|2 bytes|Permits Ethernet to **multiplex** several distinct network-layer protocols — since a given host may support multiple protocols for different applications (IP, ARP, Novell IPX, AppleTalk, etc.), each with its own standardized type number. This field is analogous to the protocol field in the network-layer datagram, or the port-number fields in the transport-layer segment — all serve to glue a protocol at one layer to a protocol at the layer above.|
|**Data**|46 to 1,500 bytes|Carries the IP datagram. The **maximum transmission unit (MTU)** of Ethernet is 1,500 bytes — if the IP datagram exceeds this, the host must **fragment** it. The **minimum** data-field size is 46 bytes — if the datagram is smaller, the data field is "**stuffed**" to fill it out; the network layer strips this stuffing off using the length field in the IP datagram header.|
|**CRC**|4 bytes|Allows the receiving adapter to **detect bit errors** in the frame (as discussed in Section 6.2.3).|

### Why Synchronization Is Needed (and How the Preamble Solves It)

**Why should the clocks be out of synchronization in the first place?** Keep in mind that adapter A aims to transmit the frame at 10 Mbps, 100 Mbps, or 1 Gbps depending on the type of Ethernet LAN. However, because nothing is ever absolutely perfect, adapter A will not transmit at _exactly_ the target rate — there will always be some **drift** (small, gradual deviation) that isn't known **a priori** (in advance) by the other adapters on the LAN. A receiving adapter can lock onto adapter A's clock simply by locking onto the bits in the first 7 bytes of the preamble.

### Connectionless and Unreliable Service

All Ethernet technologies provide **connectionless service** to the network layer. That is, when adapter A wants to send a datagram to adapter B, adapter A encapsulates the datagram in an Ethernet frame and sends the frame into the LAN, **without first handshaking** with adapter B. This layer-2 connectionless service is analogous (comparable) to IP's layer-3 datagram service and UDP's layer-4 connectionless service.

Ethernet also provides an **unreliable** service to the network layer. Specifically, when adapter B receives a frame from adapter A, it runs the frame through a CRC check, but neither sends an **acknowledgment** when a frame **passes** the check, nor sends a **negative acknowledgment** when a frame **fails** the check. When a frame fails the CRC check, adapter B simply **discards** it. Thus, adapter A has no idea whether its transmitted frame ever reached B and passed the CRC check. This lack of reliable transport (at the link layer) helps keep Ethernet **simple and cheap** — but it also means the stream of datagrams passed up to the network layer can have **gaps** (missing pieces).

**Do these gaps matter to the application?** This hinges on (depends on) whether the application uses **UDP** or **TCP**. With UDP, the application will indeed see gaps in the data. With TCP, however, TCP at the receiving host will simply **not acknowledge** the data contained in the discarded frames, causing the sending TCP to eventually **retransmit**. Note that when TCP retransmits data, that data eventually returns to the **same** Ethernet adapter at which it was originally discarded — so in a sense, Ethernet does effectively **retransmit** data, although Ethernet itself remains **unaware** of whether it's transmitting brand-new data or data that has already been transmitted at least once before.

> **Case History — Bob Metcalfe and Ethernet:** As a PhD student at Harvard University in the early 1970s, Bob Metcalfe worked on the ARPAnet at MIT. During his studies, he was also exposed to Abramson's work on ALOHA and random access protocols. After completing his PhD, and just before beginning a job at **Xerox Palo Alto Research Center (Xerox PARC)**, he visited Abramson and his University of Hawaii colleagues for three months to get a firsthand look at ALOHAnet. At Xerox PARC, Metcalfe became exposed to **Alto** computers, which in many ways were forerunners (precursors, early ancestors) of the personal computers of the 1980s. Metcalfe saw the need to network these computers in an inexpensive manner. So, armed with his knowledge of ARPAnet, ALOHAnet, and random access protocols, Metcalfe — along with colleague David Boggs — **invented Ethernet**. Metcalfe and Boggs's original Ethernet ran at **2.94 Mbps** and linked up to **256 hosts** separated by up to one mile. Metcalfe then forged (formed, established) an alliance between Xerox, Digital, and Intel to establish Ethernet as a **10 Mbps** standard, ratified (formally approved) by the IEEE. In 1979, Metcalfe formed his own company, **3Com**, which developed and commercialized Ethernet cards for the immensely popular IBM PCs of the early 1980s. In **2022**, Metcalfe received the **ACM A.M. Turing Award** (often referred to as the "Nobel Prize of Computer Science") for the invention, standardization, and commercialization of Ethernet [ACM 2022].

### Ethernet Technologies: A Bewildering (Confusing) Array of Acronyms, but Real Order Underneath

Ethernet comes in many flavors, with somewhat bewildering acronyms such as **10BASE-T**, **10BASE-2**, **100BASE-T**, **1000BASE-LX**, **10GBASE-T**, and **40GBASE-T**. These, and many other Ethernet technologies, have been standardized over the years by the **IEEE 802.3 CSMA/CD working group**. While these acronyms may appear bewildering, there is actually considerable order here:

|Portion of Acronym|Meaning|
|---|---|
|**First part** (e.g., 10, 100, 1000, 10G)|The **speed** of the standard: 10, 100, 1000, or 10G Mbps for 10 Megabit, 100 Megabit, Gigabit, and 10 Gigabit, respectively|
|**"BASE"**|Refers to **baseband Ethernet**, meaning the physical media only carries Ethernet traffic; almost all 802.3 standards are baseband standards|
|**Final part** (e.g., -T, -LX)|Refers to the **physical media** itself. Ethernet is both a link-layer _and_ a physical-layer specification, covering a variety of physical media including coaxial cable, copper wire, and fiber. Generally, a "**T**" refers to twisted-pair copper wires.|

Historically, Ethernet was initially conceived as a segment of **coaxial cable**. The early 10BASE-2 and 10BASE-5 standards specify 10 Mbps Ethernet over two types of coaxial cable, each limited in length to 500 meters. Longer runs could be obtained using a **repeater** — a physical-layer device that receives a signal on the input side and **regenerates** the signal on the output side. A coaxial cable corresponds nicely to our view of Ethernet as a **broadcast medium**: all frames transmitted by one interface are received at other interfaces, and Ethernet's CSMA/CD protocol nicely solves the multiple access problem. Nodes simply attach to the cable, and _voilà_ (there it is / behold), we have a local area network!

Ethernet has passed through a series of evolutionary steps, and today's Ethernet is very different from the original bus-topology designs using coaxial cable. In most installations today, nodes connect via **point-to-point** segments made of copper or fiber-optic cable. By the mid-2000s, various standards existed for **Gigabit Ethernet**, transmitting bits at a gigabit per second. The original Ethernet MAC protocol and frame format from earlier 10 Mbps and 100 Mbps standards were **preserved**, but higher-speed physical layers were defined for twisted-pair copper wire, optical fiber, and copper cable. Allowable Ethernet distances range from tens of meters (twisted pair) to tens of kilometers (some fiber links). While Gigabit Ethernet remains in use today, higher-speed standards have since been developed for **10, 40, 100, 200, and 400 Gbps**.

### Is Ethernet Still "Really" Ethernet?

An intriguing (thought-provoking) question worth posing: in the days of bus topologies and hub-based star topologies, Ethernet was clearly a **broadcast link** in which frame collisions occurred when nodes transmitted at the same time. To handle these collisions, the standard included the **CSMA/CD protocol**, particularly effective for a wired broadcast LAN spanning a small geographical area. But if the prevalent use of Ethernet today is a **switch-based star topology** using store-and-forward packet switching, is there really any need for an Ethernet MAC protocol at all?

As will soon become apparent, a switch coordinates its transmissions and never forwards more than one frame onto the same interface at any given time. Furthermore, modern switches are **full-duplex**, so that a switch and a node can each send frames to each other **at the same time without interference**. In other words, in a switch-based Ethernet LAN, there are **no collisions**, and therefore, **no need for a MAC protocol**!

So, is all of this _really_ still Ethernet? The answer, of course, is "yes, by definition." It's interesting to note, however, that through all of these changes, there has indeed been one enduring (long-lasting) constant — **Ethernet's frame format has remained unchanged for nearly 50 years** [Metcalfe 2025]. Perhaps this, then, is the one true and timeless **centerpiece (core, defining element)** of the Ethernet standard.

---

## 6.4.3 Link-Layer Switches

Up until this point, this document has been purposefully **vague** (deliberately imprecise) about what a switch actually does and how it works. **The role of the switch is to receive incoming link-layer frames and forward them onto outgoing links.** Crucially, the switch itself is **transparent** to the hosts and routers in the subnet — a host addresses a frame to _another_ host/router (rather than addressing the frame to the switch), and happily sends the frame into the LAN, unaware that a switch will be receiving the frame and forwarding it onward. The rate at which frames arrive at any one of a switch's output interfaces may temporarily **exceed** that interface's link capacity — to accommodate (deal with) this problem, switch output interfaces have **buffers**, much like router output interfaces buffer datagrams.

### Forwarding and Filtering

|Function|Definition|
|---|---|
|**Filtering**|The switch function that determines whether a frame **should be forwarded** to some interface, or **just dropped**|
|**Forwarding**|The switch function that determines the interfaces to which a frame should be directed, and then moves the frame to those interfaces|

Switch filtering and forwarding are both accomplished with a **switch table**. The switch table contains entries for some — but **not necessarily all** — of the hosts and routers on a LAN. An entry in the switch table contains **(1)** a MAC address, **(2)** the switch interface that leads toward that MAC address, and **(3)** the time at which the entry was placed in the table.

```
Fig -- Switch Table (Self-Learning)
────────────────────────────────────────
  Address (MAC)     Interface   Time
  62-FE-F7-11-89-A3     1       9:32
  7C-8A-B2-B4-91-10     3       9:36
  01-12-23-34-45-56     2       9:39

 Built automatically: switch records
 SOURCE MAC + arrival interface of
 every frame it sees. Stale entries
 purged after an "aging" timeout.
```

![[Pasted image 20260718000019.png]]

Interestingly, many modern layer-2 switches can be configured to forward on the basis of **either** layer-2 destination MAC addresses (functioning as a layer-2 switch) **or** layer-3 IP destination addresses (functioning as a layer-3 router). Nonetheless, the important distinction retained here: switches forward packets based on **MAC addresses**, rather than on IP addresses — and a traditional (non-SDN) switch table is constructed in a **very different manner** from a router's forwarding table.

### The Three Cases of Switch Filtering and Forwarding

Suppose a frame with destination address **DD-DD-DD-DD-DD-DD** arrives on interface **x**. The switch **indexes** (looks up) its table with this MAC address. There are three possible cases:

|Case|Situation|Switch's Action|
|---|---|---|
|1|**No entry** exists in the table for DD-DD-DD-DD-DD-DD|**Broadcast** — forward copies of the frame to the output buffers preceding **all** interfaces except x|
|2|An entry exists, associating DD-DD-DD-DD-DD-DD with interface **x** (the same interface the frame arrived on)|**Filter** — discard the frame, since the frame is already on the LAN segment containing its destination|
|3|An entry exists, associating DD-DD-DD-DD-DD-DD with interface **y ≠ x**|**Forward** — put the frame in the output buffer preceding interface y|

```
Fig -- Switch Filtering & Forwarding Logic
────────────────────────────────────────
 Frame arrives on interface x, dest=DD
    │
    ▼
 Is DD in the switch table?
  NO  -> FLOOD: forward to ALL
         interfaces except x
  YES, mapped to interface x
      -> FILTER: discard (already
         on that same segment)
  YES, mapped to interface y != x
      -> FORWARD only to interface y
```

As long as the switch table is **complete and accurate**, the switch forwards frames toward their destinations **without any broadcasting**. In this sense, a switch is "smarter" than a hub.

### Self-Learning

But how does this switch table get configured in the first place? Fortunately, a switch possesses a wonderful property (particularly gratifying (pleasing) to the already-overworked network administrator) — its table is built **automatically, dynamically, and autonomously** (independently, without external direction) — without any intervention from a network administrator or a configuration protocol. In other words, switches are **self-learning**. This capability is accomplished as follows:

1. The switch table is **initially empty**.
2. For **each** incoming frame received on an interface, the switch stores in its table: **(1)** the MAC address in the frame's **source** address field, **(2)** the interface from which the frame arrived, and **(3)** the current time. In this manner, the switch records the LAN segment on which the sender resides — if every host on the LAN eventually sends a frame, then every host will eventually get recorded in the table.
3. The switch **deletes an address** from the table if no frames are received with that address as the source address after some period of time (the **aging time**). In this manner, if a PC is replaced by another PC (with a different adapter), the original PC's MAC address will eventually be **purged** from the switch table.

Switches are **plug-and-play devices** because they require **no intervention** from a network administrator or user. A network administrator wanting to install a switch need do nothing more than connect the LAN segments to the switch's interfaces. The administrator need not configure the switch tables at the time of installation, nor when a host is removed from one of the LAN segments. Switches are also **full-duplex**, meaning any switch interface can send and receive **simultaneously**.

---

## Properties of Link-Layer Switching

Having described the basic operation of a link-layer switch, consider now its salient (notable) features and properties — several **advantages** of using switches rather than broadcast links such as buses or hub-based star topologies:

|Advantage|Explanation|
|---|---|
|**Elimination of collisions**|In a LAN built from switches (and without hubs), there is **no wasted bandwidth due to collisions**. Switches buffer frames and never transmit more than one frame on a segment at any one time. As with a router, the maximum aggregate (combined) throughput of a switch is the **sum** of all its interface rates — a significant performance improvement over LANs with broadcast links.|
|**Heterogeneous links**|Because a switch **isolates** one link from another, the different links in the LAN can operate at **different speeds** and can run over **different physical media**. For example, one switch might simultaneously have 1 Gbps copper links, 100 Mbps fiber links, and a 100 Mbps copper link — thus, a switch is **ideal for mixing legacy equipment with new equipment**.|
|**Management**|In addition to providing enhanced security, a switch eases **network management**. For example, if an adapter malfunctions and continually sends Ethernet frames (called a **jabbering** adapter), a switch can **detect** the problem and internally disconnect the malfunctioning adapter. Similarly, a cable cut disconnects only the one host that was using that cable — in the days of coaxial cable, network managers spent hours "**walking the line**" (or, more accurately, "crawling the floor") to locate a cable break that brought down the **entire** network. Switches also gather **statistics** on bandwidth usage, collision rates, and traffic types, which can be used to **debug and correct problems**, and to **plan how the LAN should evolve** in the future.|

> **Focus on Security — Sniffing a Switched LAN: Switch Poisoning:** When a host is connected to a switch, it typically only receives frames genuinely intended for it — if host A sends a frame to host B, and an entry for host B exists in the switch table, the switch will forward the frame **only** to host B. If a would-be eavesdropper (someone secretly listening in) is running a **sniffer** (packet-capture tool) on host C, that host generally cannot intercept (capture) this A-to-B frame — making it noticeably harder for an attacker to sniff frames on a switched LAN, compared to a broadcast link environment (such as hub-based Ethernet or 802.11 LANs). **However**, because a switch _does_ broadcast frames whose destination isn't yet in its table, a sniffer can still catch **some** such frames, plus all genuinely broadcast Ethernet frames (destination FF-FF-FF-FF-FF-FF). A well-known attack against a switch, called **switch poisoning**, is to flood the switch with tons of packets bearing many different **bogus (fake, fraudulent)** source MAC addresses, thereby filling the switch table with counterfeit (fake) entries and leaving no room for the legitimate hosts' MAC addresses — this forces the switch to **broadcast most frames**, which can then be picked up by the sniffer [Skoudis 2006]. Because this attack is rather involved, even for a sophisticated attacker, switches remain significantly less vulnerable to sniffing than hubs or wireless LANs.

---

## Switches Versus Routers

Routers are **store-and-forward packet switches** that forward packets using **network-layer addresses**. Although a switch is also technically a store-and-forward packet switch, it is **fundamentally different** from a router in that it forwards packets using **MAC addresses**. Whereas a router is a **layer-3** packet switch, a switch is a **layer-2** packet switch. Recall, however, that modern switches using the **"match plus action"** operation can forward a layer-2 frame based on its destination MAC address **as well as** a layer-3 datagram based on its destination IP address — indeed, switches using the **OpenFlow** standard can perform generalized packet forwarding based on any of eleven different frame, datagram, and transport-layer header fields.

```
Fig -- Packet Processing: Switch vs Router
────────────────────────────────────────
 Host --- Switch --- Router --- Host
  App                             App
  Transport                  Transport
  Network             Network  Network
  Link      Link  Link  Link     Link
  Phys      Phys  Phys  Phys     Phys

 Switch: processes up through LINK (L2)
 Router: processes up through NETWORK
         (L3) -- one extra layer of work
```

![[Pasted image 20260718000020.png]]

Even though switches and routers are fundamentally different, network administrators must often choose between them when installing an interconnection device. For example, the network administrator building the network in Figure 6.15 could just as easily have used a router instead of a switch to connect the department LANs, servers, and internet gateway router — indeed, a router would permit interdepartmental communication **without creating collisions**. Given that both switches and routers are viable candidates for interconnection, what are the pros and cons of each?

### The Case for Switches

- Switches are **plug-and-play** — a property cherished (highly valued) by the overworked network administrators of the world.
- Switches can have relatively **high filtering and forwarding rates** — as shown in Figure 6.24, switches only need to process frames **up through layer 2**, whereas routers need to process datagrams **up through layer 3**.
- **On the other hand:** to prevent the perpetual cycling (endless looping) of broadcast frames, the active topology of a switched network is restricted to a **spanning tree**. A large switched network would also require **large ARP tables** and would generate substantial ARP traffic and processing. Furthermore, switches are **susceptible to (vulnerable to) broadcast storms** — if one host goes haywire (malfunctions erratically) and transmits an endless stream of Ethernet broadcast frames, the switches will forward **all** of these frames, potentially causing the **entire network to collapse**.

### The Case for Routers

- Because network addressing is often **hierarchical** (and not flat, as with MAC addressing), packets do not normally cycle through routers even when the network has redundant paths (though packets **can** cycle if router tables are misconfigured — IP uses a special datagram header field to limit this cycling). Thus, packets are **not restricted** to a spanning tree, and can use the **best path** between source and destination.
- Because routers do not have the spanning-tree restriction, they have allowed the Internet to be built with a **rich topology** that includes, for example, multiple active links between Europe and North America.
- Routers provide **firewall protection** against layer-2 broadcast storms.
- **Perhaps the most significant drawback of routers**, though, is that they are **not plug-and-play** — hosts that connect to them need their IP addresses configured. Routers also often require **more per-packet processing time** than switches, since they must process up through the layer-3 fields.
- (A perennial (never-ending) point of trivial (minor) contention: whether to pronounce the word "**router**" either as "**rooter**" or as "**rowter**," with people wasting a fair amount of time arguing over the proper pronunciation [Perlman 1999].)

|Feature|Hubs|Routers|Switches|
|---|---|---|---|
|**Traffic isolation**|No|Yes|Yes|
|**Plug and play**|Yes|No|Yes|
|**Optimal routing**|No|Yes|No|

### Choosing Between Them in Practice

Given that both switches and routers have their pros and cons, when should an institutional network use switches, and when should it use routers?

- **Typically, small networks** consisting of a few hundred hosts have a few LAN segments — switches suffice for these small networks, as they localize traffic and increase aggregate throughput **without requiring any configuration of IP addresses**.
- **But larger networks** consisting of thousands of hosts typically include routers **within** the network (in addition to switches). Routers provide more robust isolation of traffic, control broadcast storms, and use more "intelligent" routes among the hosts in the network.

---

## 6.4.4 Virtual Local Area Networks (VLANs)

Returning to Figure 6.15, note that modern institutional LANs are often configured **hierarchically** — with each workgroup (department) having its own switched LAN connected to the switched LANs of other groups via a switch hierarchy (a chain of interconnected switches). While such a configuration works well in an ideal world, the real world is often **far from ideal**. Three drawbacks can be identified with this configuration:

|Drawback|Description|
|---|---|
|**Lack of traffic isolation**|Although the hierarchy localizes group traffic to within a single switch, **broadcast traffic** (e.g., frames carrying ARP and DHCP messages, or frames whose destination hasn't yet been learned by a self-learning switch) must still **traverse (travel across) the entire institutional network**. Limiting the scope of such broadcast traffic would improve LAN performance — and, perhaps more importantly, it may also be desirable to limit LAN broadcast traffic for **security/privacy reasons**. For example, if one group contains the company's executive management team and another group contains disgruntled (dissatisfied, resentful) employees running Wireshark packet sniffers, the network manager may well prefer that the executives' traffic **never even reaches** employee hosts. (This isolation could be provided by replacing the center switch with a router — but it can also be achieved via a switched, layer-2 solution, as will be shown.)|
|**Inefficient use of switches**|If, instead of three groups, the institution had **10 groups**, then 10 first-level switches would be required. If each group were small (say, less than 10 people), a single 96-port switch would likely be large enough to accommodate everyone — but this single switch would **not** provide traffic isolation.|
|**Managing users**|If an employee moves between groups, the physical cabling must be **changed** to connect the employee to a different switch. Employees belonging to two groups simultaneously make the problem even harder.|

### The VLAN Solution

Fortunately, each of these difficulties can be handled by a switch that supports **virtual local area networks (VLANs)**. As the name suggests (implies), a switch that supports VLANs allows multiple **virtual** local area networks to be defined over a single **physical** local area network infrastructure. Hosts within a VLAN communicate with each other as if they (and **no other** hosts) were connected to the switch.

In a **port-based VLAN**, the switch's ports (interfaces) are divided into groups by the network manager. Each group constitutes a VLAN, with the ports in each VLAN forming a **broadcast domain** (i.e., broadcast traffic from one port can only reach other ports **in the same group**).

```
Fig -- Port-Based VLAN (single switch)
────────────────────────────────────────
 Ports 2-8  = EE VLAN (Electrical Eng.)
 Ports 9-15 = CS VLAN (Computer Sci.)
 Port 1,16  = unassigned

 Switch hardware only forwards frames
 BETWEEN ports of the SAME VLAN --
 broadcast domains stay separate even
 though it's one physical switch.
```

![[Pasted image 20260718000021.png]]

One can readily (easily) picture how a VLAN switch is configured and operates — the network manager declares a port to belong to a given VLAN (with undeclared ports belonging to a default VLAN) using switch management software; a table of port-to-VLAN mappings is maintained within the switch; and switch **hardware** only delivers frames between ports belonging to the **same** VLAN.

### The New Problem VLANs Introduce (and How Trunking Solves It)

By completely isolating two VLANs from one another, a **new difficulty** is introduced: how can traffic from one department (say, EE) be sent to another department (say, CS)? One way to handle this would be to connect a VLAN switch port to an **external router** and configure that port to belong to **both** VLANs. In this configuration, even though the two departments share the **same physical switch**, the **logical configuration** would look as if the two departments had separate switches connected via a router. Fortunately, switch vendors make such configurations easy by building a **single device** that contains **both** a VLAN switch **and** a router, so a separate external router isn't strictly necessary.

Now suppose that, rather than having a separate Computer Engineering department, some EE and CS faculty are housed in a **separate building**, where they need network access and would like to be part of their respective department's VLAN. A **second** 8-port switch would need to be interconnected with the first. But how should these two switches be interconnected? One straightforward (but poorly **scalable (able to grow efficiently)**) solution would be to define a port belonging to the CS VLAN on each switch, and to connect these ports to each other — the drawback being that **N** VLANs would require **N** ports on each switch simply to interconnect the two switches.

### VLAN Trunking

A more **scalable** approach to interconnecting VLAN switches is known as **VLAN trunking**. A special port on each switch (say, port 16 on the left switch and port 1 on the right switch) is configured as a **trunk port** to interconnect the two VLAN switches. **The trunk port belongs to all VLANs**, and frames sent to **any** VLAN are forwarded over the trunk link to the other switch.

```
Fig -- VLAN Trunking (two switches)
────────────────────────────────────────
 Switch 1 (EE 2-8, CS 9-15) --Trunk--
 Switch 2 (EE 2,3,6, CS 4,5,7)

 Trunk PORT belongs to ALL VLANs.
 Frames crossing the trunk carry an
 802.1Q TAG naming their VLAN, so the
 receiving switch knows which VLAN's
 ports should get the frame.
```

![[Pasted image 20260718000022.png]]

**A new question this raises:** how does a switch know that a frame arriving on a trunk port belongs to a particular VLAN? The IEEE has defined an **extended Ethernet frame format**, **802.1Q**, specifically for frames crossing a VLAN trunk.

### The 802.1Q Frame Format

An 802.1Q frame consists of the standard Ethernet frame with a **four-byte VLAN tag** added into the header, carrying the identity of the VLAN to which the frame belongs. The VLAN tag is added into a frame by the switch at the **sending side** of a VLAN trunk, and parsed (analyzed and interpreted) and removed by the switch at the **receiving side** of the trunk.

```
Fig -- 802.1Q VLAN Tag
────────────────────────────────────────
 Original:
 |Pre|Dest|Src|Type|   Data   |CRC |

 802.1Q-tagged (4-byte tag inserted):
 |Pre|Dest|Src|TAG|Type| Data |CRC*|
               └┬┘
        2B TPID(0x8100) + 2B TCI
        (12-bit VLAN ID + 3-bit
         priority)
 * CRC recomputed since header changed
```

![[Pasted image 20260718000023.png]]

The VLAN tag itself consists of:

|Sub-field|Size|Contents|
|---|---|---|
|**Tag Protocol Identifier (TPID)**|2 bytes|A fixed hexadecimal value of **81-00**, signaling that this is an 802.1Q-tagged frame|
|**Tag Control Information**|2 bytes|Contains a **12-bit VLAN identifier field**, and a **3-bit priority field** that is similar in intent (purpose) to the IP datagram's **TOS (Type of Service)** field|

### Beyond Port-Based VLANs

This discussion has only briefly touched on VLANs, focusing on **port-based** VLANs. It's worth mentioning, though, that VLANs can be defined in several other ways:

- **MAC-based VLANs:** the network manager specifies the set of MAC addresses that belong to each VLAN; whenever a device attaches to a port, the port is connected into the appropriate VLAN based on the MAC address of the device.
- **Network-layer-protocol-based VLANs:** VLANs can also be defined based on network-layer protocols (e.g., IPv4, IPv6, or AppleTalk) and other criteria (see the 802.1Q standard [IEEE 802.1q 2005] for further details).
- **VLANs that span routers:** it's also possible for VLANs to be extended across IP routers, allowing islands (isolated clusters) of LANs to be connected together to form a single VLAN that could span the **globe** [Yu 2011]. Indeed, **VXLAN (Virtual Extensible LAN)** technology (covered in Section 6.5.2) allows virtual LANs to be created that cover far greater geographical distances than traditional VLANs.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**MAC addresses can be forged (spoofed)**|Because MAC addresses are set in software and unauthenticated (not cryptographically verified), an attacker can trivially **spoof** (impersonate) another device's MAC address — e.g., to bypass MAC-based access control, or to intercept traffic intended for the legitimate device|MAC-based access control alone is a weak defense; it should be paired with stronger authentication (e.g., 802.1X port-based authentication) rather than relied upon in isolation|
|**ARP has no built-in authentication**|An attacker on the same subnet can send **forged ARP replies** (a technique known as **ARP spoofing** or **ARP cache poisoning**), tricking victim hosts into associating the attacker's MAC address with another host's (or the router's) IP address — enabling **man-in-the-middle** interception of traffic|Static ARP entries, ARP-spoofing detection tools, and switch features like **Dynamic ARP Inspection (DAI)** can mitigate this well-known and long-standing vulnerability|
|**Switch poisoning degrades a switch's isolation guarantees**|As discussed above, flooding a switch table with bogus source addresses can force it into broadcasting most traffic, undermining the very isolation that makes switched LANs harder to sniff than broadcast links|Port security features (limiting the number of MAC addresses learnable per port) and monitoring for abnormal switch-table churn (rapid, excessive turnover) help detect and prevent this attack|
|**VLANs provide isolation, but aren't a substitute for genuine security boundaries**|VLAN hopping attacks (e.g., exploiting misconfigured trunk ports, or double-tagging frames) can sometimes allow an attacker on one VLAN to inject traffic into another, undermining the isolation VLANs are meant to provide|VLANs should be treated as a traffic-management and broadcast-domain tool, not as a hardened security perimeter; genuinely sensitive isolation (e.g., between an executive network and a general employee network) often still warrants router-level (or firewall-level) separation in addition to VLANs|
|**Broadcast storms are a potential denial-of-service vector**|A malicious or malfunctioning ("jabbering") host that floods a switched LAN with broadcast frames can, in principle, be deliberately triggered by an attacker to degrade or collapse an entire broadcast domain|Switches with storm-control features, and network segmentation via VLANs/routers to limit the blast radius (scope of damage) of any single broadcast domain, both help contain this risk|

---

## Questions I Still Have

- [ ] The text notes that an ARP query is sent as a broadcast frame while the ARP response is sent as a standard (unicast) frame — beyond the practical reason (the querier's MAC address is already known to the responder), is there a deeper design principle here about minimizing unnecessary broadcast traffic on the LAN?
- [ ] Given that switch tables are built dynamically via self-learning and have an aging timeout, what happens to in-flight frames addressed to a host whose entry has just expired but hasn't yet sent a new frame to refresh it — does the switch simply fall back to flooding until it relearns the entry?
- [ ] The text mentions that modern switches using "match plus action" (OpenFlow) can forward on up to eleven different header fields — how does a switch decide, in practice, whether to treat an ambiguous case as a layer-2 forwarding decision versus a layer-3 one, or is that entirely dictated by the specific SDN application/controller logic?
- [ ] For VLAN trunking with 802.1Q, since the VLAN tag adds 4 bytes to the frame, does this ever cause any of the same MTU/fragmentation concerns discussed for the standard Ethernet data field, especially for maximum-size frames?
- [ ] The text says routers prevent packet cycling because IP addressing is hierarchical, while switches restrict their active topology to a spanning tree to prevent broadcast-frame cycling — in a hybrid network using both switches and routers together (as in most large institutional networks), how do these two very different cycle-prevention mechanisms interact or coexist without interfering with each other?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**MAC address**|A 6-byte, flat (non-hierarchical), globally unique link-layer address assigned to a network adapter (not the host itself); also called a LAN address or physical address|
|**Broadcast address**|The special MAC address FF-FF-FF-FF-FF-FF, used to address a frame to all adapters on a LAN|
|**ARP (Address Resolution Protocol)**|A protocol that resolves IP addresses to MAC addresses for hosts and routers on the same subnet, via broadcast query and unicast response|
|**ARP table**|A cache, maintained by each host and router, mapping IP addresses to MAC addresses with an associated time-to-live before expiration|
|**Ethernet**|The dominant wired LAN technology; originally a broadcast bus/hub-based technology using CSMA/CD, now typically deployed as a switch-based, collision-free star topology|
|**Ethernet frame**|The link-layer frame format used by Ethernet, consisting of preamble, destination address, source address, type, data (46–1,500 bytes), and CRC fields — unchanged for nearly 50 years|
|**MTU (Maximum Transmission Unit)**|The largest data-field size an Ethernet frame can carry (1,500 bytes); larger IP datagrams must be fragmented|
|**Hub**|A physical-layer (bit-level) broadcast device that regenerates and retransmits every incoming bit to all other interfaces|
|**Link-layer switch**|A store-and-forward, layer-2 device that forwards frames based on MAC addresses using a self-learning switch table, without the collisions of a broadcast link|
|**Switch table**|A table mapping MAC addresses to switch interfaces (plus a timestamp), used by a switch for filtering and forwarding decisions|
|**Self-learning**|A switch's ability to build its switch table automatically, by recording the source MAC address and arrival interface of every frame it receives|
|**Filtering**|The switch function of deciding whether a frame should be forwarded to an interface, or discarded|
|**Forwarding**|The switch function of directing a frame to the correct output interface(s)|
|**Switch poisoning**|An attack that floods a switch with bogus source MAC addresses to fill its table and force it into broadcasting most traffic, aiding eavesdropping|
|**VLAN (Virtual Local Area Network)**|A configuration allowing a single physical switch/LAN infrastructure to be logically divided into multiple isolated broadcast domains|
|**Port-based VLAN**|A VLAN configuration in which a switch's ports are grouped by the network manager into separate broadcast domains|
|**VLAN trunking**|A scalable method of interconnecting VLAN switches using a single trunk port/link that carries traffic for all VLANs, tagged by VLAN identity|
|**802.1Q**|The IEEE standard defining the extended Ethernet frame format (adding a 4-byte VLAN tag) used for frames crossing a VLAN trunk|

---

## Related Concepts

---

→ Next: [[LINK VIRTUALIZATION]]