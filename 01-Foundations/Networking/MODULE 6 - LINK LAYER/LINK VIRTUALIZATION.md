---
title: LINK VIRTUALIZATION
date: 2026-07-21
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 6.5 Link Virtualization: A Network as a Link Layer

> **One-Line Summary:** A **virtual link** between two devices can behave, from the outside, exactly like a single **physical link**, while actually being **implemented by an entire underlying network** of interconnected devices — a **paradigm (governing model, way of framing a concept)** demonstrated by **MPLS (Multiprotocol Label Switching)**, which turns a routed IP network into a **label-switched, virtual-circuit network** capable of engineering traffic along specific paths, and by **VXLAN (Virtual Extensible LAN)**, which **tunnels (encapsulates and carries through)** entire Ethernet LANs across an IP network to create a single logical LAN spanning the globe.

---

## Core Idea: The Evolving Meaning of "Link"

Because this chapter concerns link-layer protocols, and given that it's now nearing its end, it's worth reflecting on how the understanding of the term **link** has **evolved (gradually changed and developed)** throughout the chapter. The chapter began by viewing a link as a **physical wire** connecting two communicating hosts. It then became apparent that multiple hosts could be connected by a **shared** wire, and that the "wire" connecting hosts could actually be **radio spectra** or other media — this led to viewing the link a bit more **abstractly (in a more generalized, less literal way)**, as a **channel**, rather than as a literal wire.

In the study of Ethernet LANs (Section 6.4), it became clear that the interconnecting medium could actually be a rather **complex switched infrastructure**. Throughout this evolution, however, the **hosts themselves** maintained the view that the interconnecting medium was simply a link-layer connection between two or more hosts. An Ethernet host, for example, can be **blissfully unaware (happily oblivious)** of whether it's connected to other LAN hosts by a single short cable segment, by a **geographically dispersed (spread out over distance)** switched LAN, or by a VLAN.

### Two More Examples of "The Network as a Link"

Consider a **dialup modem** connection between two hosts — the link connecting them is actually the **telephone network**: a logically separate, global telecommunications network with its own switches, links, and protocol stacks for data transfer and signaling. From the Internet link-layer point of view, however, the dialup connection through the telephone network is viewed as a **simple "wire."** In this sense, the Internet **virtualizes** the telephone network, viewing it as a link-layer technology that provides link-layer connectivity between two Internet hosts.

(This should sound familiar — it's analogous to the earlier discussion of **overlay networks**, back in Chapter 2, which similarly view the Internet as a means for providing connectivity between overlay nodes, seeking to **overlay** the Internet in the same way that the Internet overlays the telephone network.)

This section covers **two technologies** in which a **virtual link** between two devices logically behaves like a **single physical link**, but is in fact implemented by an underlying complex network of interconnected devices:

|Technology|What It Virtualizes|Covered In|
|---|---|---|
|**MPLS (Multiprotocol Label Switching)**|A packet-switched, virtual-circuit network in its own right — routers virtualized as circuit-switches|Section 6.5.1|
|**VXLAN (Virtual Extensible LAN)**|An entire Ethernet LAN, tunneled across an underlying IP network|Section 6.5.2|

Unlike the **circuit-switched telephone network**, MPLS is a **packet-switched, virtual-circuit network** in its own right — it has its own packet formats and forwarding behaviors. Thus, from a **pedagogical (relating to teaching/learning) viewpoint**, a discussion of MPLS fits well into a study of either the network layer or the link layer. From an **Internet viewpoint**, however, MPLS can be considered — much like the telephone network and switched Ethernets — as a link-layer technology that serves to **interconnect IP devices**.

---

## 6.5.1 Multiprotocol Label Switching (MPLS)

### Why MPLS Was Invented

MPLS **evolved (developed gradually over time)** from a number of industry efforts in the **mid-to-late 1990s** to improve the forwarding speed of IP routers, by **adopting a key concept from the world of virtual-circuit networks**: a **fixed-length label**.

**Crucially**, the goal was **not to abandon** the destination-based, IP-datagram-forwarding infrastructure in favor of one based purely on fixed-length labels and virtual circuits, but rather to **augment (add to, enhance)** it — by selectively **labeling** datagrams and allowing routers to forward datagrams based on **fixed-length labels** (rather than destination IP addresses) **when possible**. Importantly, these techniques work **hand-in-hand** with IP, using existing IP addressing and routing. The IETF **unified** these efforts in the MPLS protocol [RFC 3031, RFC 3032], effectively **blending** virtual-circuit (VC) techniques into a routed datagram network.

### The MPLS Frame Format

Consider the format of a link-layer frame handled by an **MPLS-capable router**. Such a link-layer frame transmitted between MPLS-capable devices has a **small MPLS header** added between the layer-2 (e.g., Ethernet) header and the layer-3 (i.e., IP) header. RFC 3032 defines the format of the MPLS header for such links; headers are also defined for **ATM** and **frame-relayed** networks in other RFCs.

```
Fig -- MPLS Header: Between L2 and L3
────────────────────────────────────────
┌─────────┬──────┬────┬───────────────┐
│PPP/Eth   │ MPLS │ IP │  Remainder of │
│ header   │header│ hdr│ link-layer frm│
└─────────┴──────┴────┴───────────────┘
            │
            ▼
       ┌─────┬───┬─┬─────┐
       │Label│Exp│S│ TTL │
       └─────┴───┴─┴─────┘
        20b   3b  1b  8b
```

|Field in MPLS Header|Purpose|
|---|---|
|**Label**|The fixed-length label itself, used for forwarding|
|**Exp (3 bits)**|**Reserved for experimental use**|
|**S (1 bit)**|Used to indicate the **end of a series of "stacked" MPLS headers** (an advanced topic not covered in depth here)|
|**TTL**|A **time-to-live** field, functioning much like the TTL in an IP datagram|

It's **immediately evident (obvious upon inspection)** from the frame format that an MPLS-enhanced frame can only be sent **between routers that are both MPLS capable** — a non-MPLS-capable router would be thoroughly **confused (unable to correctly process the frame)** when it found an MPLS header where it had expected to find the IP header!

### The Label-Switched Router

An MPLS-capable router is often referred to as a **label-switched router**, since it forwards an MPLS frame by looking up the MPLS label in its **forwarding table**, and then **immediately** passing the datagram to the appropriate output interface. Thus, the MPLS-capable router **need not** extract the destination IP address and perform a lookup of the **longest prefix match** in the (ordinary IP) forwarding table — the label lookup alone suffices.

But how does a router know if its neighbor is indeed MPLS capable, and how does a router know **which** label to associate with a given IP destination? To answer these questions, it's necessary to examine the **interaction** among a group of MPLS-capable routers.

### Worked Example: A Group of MPLS-Capable Routers

In the network under consideration, **routers R1 through R4** are MPLS capable, while **R5 and R6** are standard (ordinary) IP routers.

- **R1** has advertised to **R2 and R3** that it (R1) can route to destination **A**, and that a received frame with MPLS label **6** will be forwarded to destination A.
- **Router R3** has advertised to router **R4** that it can route to destinations **A and D**, and that incoming frames with MPLS labels **10 and 12**, respectively, will be switched toward those destinations.
- **Router R2** has also advertised to router **R4** that it (R2) can reach destination A, and that a received frame with MPLS label **8** will be switched toward A.

The **broad picture** painted by this example is that IP devices R5, R6, A, and D are connected together via an **MPLS infrastructure** (MPLS-capable routers R1, R2, R3, and R4), in much the same way that a **switched LAN** or an **ATM network** can connect together IP devices. And, like a switched LAN or ATM network, the MPLS-capable routers R1 through R4 do so **without ever touching the IP header of a packet**.

```
Fig -- MPLS Label-Switched Paths to A
────────────────────────────────────────
 R5,R6 (ordinary IP) -> R4 -> R3 -> D
                         │
                        R2 -> R1 -> A

 R4's forwarding table:
  in    out   dest   out
 label label        iface
  10     12    A      0
   8     10    A      1

 R4 has TWO paths to A: via iface 0
 (label 10) and via iface 1 (label 8)
 -- impossible with plain longest-
 prefix-match IP routing (only one
 path would normally be chosen).
```

![[Pasted image 20260722211459.png]]

> **Analogy — Coat-Check Tickets Instead of Reading Descriptions:** Imagine a large coat-check counter (a checkroom where guests hand over their coats and receive numbered tickets in return) at a huge event, with **multiple attendants** passing coats along a chain to reach the final storage rack. Ordinary IP forwarding is like each attendant having to **re-examine** the coat itself every time — checking its color, size, and owner's description — to figure out which direction to pass it. MPLS is like handing out a small numbered **ticket (the label)** the moment the coat arrives: from then on, every attendant down the chain just glances at the ticket **number** and instantly knows which direction to pass it, without ever having to re-inspect the coat itself. This is dramatically faster and simpler, especially when the same coat might need to travel down one of **several possible chains** depending on which ticket it was given — something that would be far harder to arrange if every attendant had to make an independent judgment call based solely on the coat's description.

### What About the Signaling Protocol That Distributes These Labels?

The specific protocol used to **distribute labels** among MPLS-capable routers is not detailed here, as it's well beyond scope — however, it's worth noting that the IETF working group on MPLS has specified that an **extension of the RSVP protocol**, known as **RSVP-TE** [RFC 3209], is the focus of current efforts for MPLS signaling. It's also worth noting that **existing link-state routing algorithms** (e.g., **OSPF**) have been **extended** to flood this label-related information to MPLS-capable routers — interestingly, the **actual path-computation algorithms** are **not standardized**, and are currently **vendor-specific (unique to and determined by each particular equipment manufacturer)**.

### The Real Advantage of MPLS: Not Speed, But Traffic Engineering

Thus far, the emphasis of this discussion has been on the fact that MPLS performs switching based on **labels**, without needing to consider the IP address of a packet. The **true advantages** of MPLS, and the reason for **current interest** in MPLS, however, lie **not** in the potential increases in switching speed, but rather in the **new traffic management capabilities that MPLS enables**.

As noted above, R4 has **two** MPLS paths to A. If forwarding were performed up at the IP layer on the basis of IP address — using the routing protocols studied in Chapter 5 — this would specify only a **single, least-cost path** to A. **MPLS**, by contrast, provides the ability to forward packets along routes that would **not** be possible using standard IP routing protocols. This is one simple form of **traffic engineering** using MPLS [RFC 3346; RFC 3272; RFC 2702; Xiao 2000], in which a network operator can **override** normal IP routing and force some traffic headed toward a given destination along **one path**, and other traffic destined for the **same** destination along **another path** (whether for policy, performance, or some other reason).

### Other Uses of MPLS

|Use Case|Description|
|---|---|
|**Performing many other purposes**|MPLS forwarding paths can be used for **fast restoration** — e.g., rerouting traffic over a **precomputed failover path** in response to a link failure [Kar 2000; Huang 2002; RFC 3469]|
|**Implementing virtual private networks (VPNs)**|In implementing a **VPN** for a customer, an ISP uses its MPLS-enabled network to connect together the customer's various networks. MPLS can be used to **isolate** the resources and addressing used by the customer's VPN from that of other customers' traffic crossing the ISP's network; see [DeClercq 2002] for further details. VPNs are covered in more detail in Section 8.7.1.|

**One final historical note:** MPLS **rose to prominence** _before_ the development of **software-defined networking** (studied in Chapter 5), and many of MPLS's traffic-engineering capabilities can **also** be achieved via SDN and the generalized forwarding paradigm covered in Chapter 4. Only the future will tell whether MPLS and SDN will continue to **coexist**, or whether newer technologies (such as SDN) will eventually **replace** MPLS.

---

## 6.5.2 VXLANs: Ethernet Over IP

### The Two Limitations of Standard VLANs

The **802.1Q VLANs** studied in Section 6.4.4 have **two important limitations**:

1. They are **constrained (restricted)** to have **no more than 4,096 VLANs**, owing to the **12-bit VLAN identifier field** in the 802.1Q VLAN tag.
2. They **must be contained within a single, all-switched-Ethernet, end-to-end LAN infrastructure**.

**Is it possible** to build even larger VLANs, or to connect layer-2 devices that are located **around the globe** into a virtual network? The answer to both these questions is **"Yes!"** — using an approach known as **Virtual eXtensible Local-Area Networks, or VXLANs** [RFC 7348].

### The Basic Scenario

Consider the simple case of two **physically separate** Ethernet LANs — one in **Sunnyvale, California** and one in **Bangalore, India** — that are combined into a single **logical VXLAN**, such that an Ethernet frame sent from a host in the Sunnyvale Ethernet segment addressed to the MAC address of a host in the Bangalore Ethernet segment will be **received** at that host in Bangalore!

This is **accomplished** by creating a link-layer **tunnel** with one endpoint on a device on the Sunnyvale LAN and the other endpoint on a device on the Bangalore LAN. An Ethernet frame **entering** the tunnel on the Sunnyvale LAN will **exit** the tunnel on the Bangalore LAN. A VXLAN tunnel thus **extends** an Ethernet, or more generally a layer-2 subnet, **across layer-3 network boundaries**.

> **Analogy — A Pneumatic Tube System Connecting Two Buildings' Internal Mail Rooms:** Imagine two office buildings — one in Sunnyvale, one in Bangalore — each with its own internal **pneumatic tube system (a network of tubes that shoots canisters between rooms using air pressure)** for delivering mail _within_ that building. Now imagine engineers install a **single, dedicated long-distance tube** connecting a specific mail slot in the Sunnyvale building directly to a specific mail slot in the Bangalore building — one that happens to pass through the **ordinary postal system** to get there (the underlying IP network), but arrives so seamlessly (smoothly, without any visible interruption) that from the perspective of anyone inside either building, it's simply **one more tube in their own local pneumatic system**. A canister (an Ethernet frame) dropped into the Sunnyvale system, addressed to a mail slot that happens to physically sit in Bangalore, would simply **pop out the far end of the long-distance tube** in Bangalore — indistinguishable, to the buildings' occupants, from an ordinary local delivery.

Recall from the discussion of **tunneling** in Chapter 4, where a tunnel was used to connect one IPv6 network to another IPv6 network through an **intermediary** IPv4 network. With VXLANs, there's another example of how an **entire network** (in this case, the IP network _between_ the tunnel endpoints) is **virtualized** to be a single **logical link**. (This same idea — tunneling — also appears widely in cellular networks, as covered further in Chapter 7.)

### VXLAN Tunnel Endpoints (VTEPs)

The **VXLAN Tunnel Endpoints (VTEPs)** play a **crucial (essential, pivotal)** role in VXLANs. A VTEP is an **ordinary physical switch or router**, or a **virtualized switch or router in a data center**, which is configured to implement VXLANs.

```
Fig -- VXLAN: Ethernet Frame Tunneled
       Over an IP Network
────────────────────────────────────────
 Sunnyvale LAN          Bangalore LAN
 Host A                  Host B
   │                        │
 VTEP x ==VXLAN Tunnel==> VTEP y
  (Ethernet frame A->B wrapped in:)

 |x,y VXLAN hdr|Ethernet frame A->B|
 └─────────────────────────────────┘
           UDP segment
 └─────────────────────────────────┘
           IP datagram (x to y)

 To the Internet in between: just an
 ordinary IP datagram between x and y.
 To hosts A and B: looks like ONE LAN.
```

![[Pasted image 20260718000026.png]]

### Step-by-Step: How a Frame Actually Crosses a VXLAN Tunnel

1. **Host A**, in Sunnyvale, sends an Ethernet frame with the **MAC destination address of Host B**, which is in Bangalore.
2. Being **physically attached** to the Sunnyvale LAN, **VTEP x** receives the frame and notes that it is **destined for Host B**, which it knows lies at the **other end of the tunnel**.
3. **VTEP x** then places the Ethernet frame **in its entirety**, together with some additional **VXLAN header information**, into the **payload** of a standard **UDP datagram**. This header information includes a **24-bit VXLAN Network Identifier (VNI)**, which allows **16 million VXLANs** to be identified per extended LAN — a dramatically larger number than the mere 4,096 possible with standard 802.1Q VLANs, and **particularly valuable in data center use cases**.
4. The **UDP datagram**, containing the original Ethernet frame plus header information, is then placed inside an **IP datagram** that is addressed to **VTEP y**, which implements the **other end** of the VXLAN tunnel.
5. The **IP network** between x and y then delivers this IP datagram to y, **exactly as it would normally deliver any datagram** — being entirely **oblivious (unaware)** to the contents carried within.
6. **VTEP y** receives the datagram, notices it contains a **UDP segment carrying a VXLAN Ethernet frame**, extracts it, and **transmits the Ethernet frame** onto the Bangalore Ethernet.
7. The **original Ethernet frame** — with source MAC address A and destination MAC address B — is then **received at Host B**, exactly as if A and B had been on the same local Ethernet segment all along.

> **A note of genuine intellectual satisfaction:** if you can **wrap your mind around** the rather **unusual layering** of encapsulating an Ethernet datagram inside a UDP segment, inside an IP datagram, as shown above, then you have truly **mastered** not only the concept of **tunneling**, but also of **link virtualization**, and the notion of an **entire network serving as a single virtual link**!

### What This Brief Treatment Leaves Out

This brief discussion of VXLANs has **skipped over** some important issues that are worth being aware of, even without full detail:

- **How** did VTEP x actually **know** that Host B lay at the other end of the tunnel in the first place?
- **What** VXLAN header information is specifically included inside the UDP datagram?
- **What** are the various **use cases** that actually require such a large number of VXLANs?

Good sources of further information on the details of VXLANs and their use include [Arista VXLAN 2017; Cisco VXLAN 2014; Juniper VXLAN 2024; RFC 7348].

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**MPLS forwarding bypasses IP-header inspection**|Since label-switched routers forward frames purely on the basis of the MPLS label without examining the IP header, a security appliance positioned to inspect IP-layer traffic (e.g., a firewall relying on IP addresses/ports) could potentially be **bypassed** if it isn't also MPLS-aware, since the malicious payload's true IP header is never consulted mid-path|Security devices deployed within or at the edges of an MPLS network need explicit MPLS awareness (label inspection, or placement only at label-imposition/label-removal points) to maintain effective inspection coverage|
|**MPLS VPNs rely on correct label-based isolation**|If an attacker can somehow **inject or manipulate labels** (e.g., through a misconfigured or compromised MPLS-capable router), they could potentially cause traffic from one customer's VPN to be forwarded into another customer's VPN, violating the isolation MPLS VPNs are meant to guarantee|ISPs offering MPLS-based VPNs must carefully control which routers are permitted to participate in label distribution and enforce strict configuration discipline, since the isolation guarantee is only as strong as the correctness of the label-switching infrastructure|
|**VXLAN tunnels traverse the public Internet in plaintext by default**|Because a VXLAN simply wraps an Ethernet frame inside an ordinary UDP/IP datagram, an eavesdropper positioned anywhere along the path between VTEP x and VTEP y can potentially **capture and read** the encapsulated Ethernet frame, since VXLAN itself provides no encryption|VXLAN traffic crossing untrusted networks (e.g., the public Internet, as opposed to a private data-center backbone) should be layered with additional encryption (e.g., IPsec between VTEPs) to protect confidentiality and integrity|
|**A compromised VTEP has outsized reach**|Because a VTEP is the single point that decides which frames enter and exit a VXLAN tunnel, a compromised or misconfigured VTEP could inject forged frames directly into a remote LAN segment thousands of miles away, effectively granting an attacker a foothold **inside** a network they have no physical proximity to|VTEPs — being high-value, high-trust infrastructure — warrant the same hardening, access control, and monitoring rigor as core routers or firewalls, given the scope of what a compromise would expose|
|**VXLAN's massive namespace (16 million VNIs) increases the attack surface for misconfiguration**|A larger, more complex identifier space (24-bit VNI vs. 12-bit VLAN ID) increases the chance of accidental **VNI collision or overlap** across tenants in a large multi-tenant data center, potentially leaking traffic between organizations that believe themselves fully isolated|Rigorous VNI allocation and auditing practices, especially in shared/multi-tenant cloud and data-center environments, are essential to prevent unintentional cross-tenant traffic leakage|

---

## Questions I Still Have

- [ ] The text notes that actual MPLS path-computation algorithms are vendor-specific and not standardized, even though label distribution protocols (like RSVP-TE) and link-state flooding (via extended OSPF) are standardized — what specific competitive or technical advantage do vendors gain by keeping the path-computation logic itself proprietary?
- [ ] For the "S bit" in the MPLS header, used to indicate the end of a series of "stacked" MPLS headers — what real-world scenario actually requires multiple MPLS labels to be stacked on a single frame, and why would a network ever need more than one label layer at once?
- [ ] Given that many of MPLS's traffic-engineering capabilities can also be achieved via SDN, what specific practical or economic factors (rather than purely technical ones) are currently determining whether network operators choose to keep MPLS deployments running versus migrating fully to SDN-based generalized forwarding?
- [ ] The text mentions VXLAN's 24-bit VNI supports 16 million VXLANs, "particularly valuable in data center use cases" — what specific data-center scenario actually needs anywhere close to that many separate VXLANs, given that even large cloud providers presumably have far fewer than millions of distinct tenants at any one location?
- [ ] Since a VXLAN tunnel between VTEP x and VTEP y is delivered as an ordinary IP datagram by the intermediary IP network, how does that IP network handle the fact that the tunneled payload (with its own encapsulated Ethernet frame) could itself already exceed the outer network's own MTU — is fragmentation handled at the VTEP, or does this require careful MTU planning across the whole path?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Link virtualization**|The general concept of an entire underlying network (rather than a single physical wire) behaving, from the outside, like a single logical link|
|**MPLS (Multiprotocol Label Switching)**|A packet-switched, virtual-circuit-style network technology that augments IP forwarding with fixed-length label-based switching, enabling traffic engineering beyond what standard IP routing allows|
|**MPLS header**|A small header inserted between the layer-2 header and the IP header of a frame, containing a Label, Exp (experimental) bits, an S (stack) bit, and a TTL field|
|**Label-switched router**|An MPLS-capable router that forwards frames by looking up the MPLS label in its forwarding table, without needing to examine the destination IP address|
|**Traffic engineering**|The practice of overriding standard, single-least-cost-path IP routing to deliberately route different traffic toward the same destination along different paths, for reasons of policy, performance, or load balancing|
|**RSVP-TE**|An extension of the RSVP protocol used as the basis for MPLS label-distribution signaling|
|**VPN (Virtual Private Network)**|A network service (which can be implemented via MPLS) in which an ISP connects a customer's various sites together over the ISP's own network, while isolating that customer's traffic and addressing from other customers|
|**VLAN limitations**|Standard 802.1Q VLANs are capped at 4,096 possible VLANs (12-bit identifier) and must remain within a single switched-Ethernet infrastructure|
|**VXLAN (Virtual eXtensible Local-Area Network)**|A technology that tunnels Ethernet frames across an IP network, allowing geographically separated Ethernet LAN segments to behave as a single logical LAN, supporting up to 16 million distinct VXLANs via a 24-bit identifier|
|**Tunnel (link-layer tunnel)**|A logical connection between two endpoint devices across an intermediary network, such that data entering one end emerges unchanged at the other, regardless of the underlying network's own structure|
|**VTEP (VXLAN Tunnel Endpoint)**|A physical or virtualized switch/router configured to implement one end of a VXLAN tunnel — encapsulating outbound Ethernet frames into UDP/IP, and decapsulating inbound ones|
|**VNI (VXLAN Network Identifier)**|The 24-bit field within the VXLAN header identifying which of up to 16 million VXLANs a given tunneled frame belongs to|

---

## Related Concepts

---

→ Next: 6.6 Data Center Networking