---
title: WIRELESS CORE NETWORK
date: 2026-07-24
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 7.4 The Wireless Core Network

> **One-Line Summary:** The **5G Core** is the centralized, all-IP collection of **Network Functions** — chief among them the data-plane **User Plane Function (UPF)** and the control-plane **Access and Mobility Management Function (AMF)** and **Session Management Function (SMF)** — that gives a cellular network the identity, security, mobility, and billing services a plain wired IP network was never built to provide, delivered via **tunneling** to the RAN and established for each device through a two-phase **registration** and **PDU session establishment** process.

---

## Core Idea

Section 7.3 covered the wireless **edge** — the RAN, whether WiFi or cellular. This section moves inward, to the wireless **core**: the part of a cellular network that sits behind the base station and connects it to the broader Internet. The Core is worth an entire section of its own in cellular networking in a way it simply isn't for WiFi, and the reason is historical rather than technical. WiFi is, from the network's point of view, just another **link-layer** technology bolted onto an ordinary wired IP network — an enterprise WiFi deployment uses the exact same control- and management-plane machinery as wired Ethernet. Cellular networks, by contrast, trace their lineage back to 1G and 2G telephone-only networks, with 3G still fundamentally a telephone network that had packet data grafted on as an afterthought. Those early networks needed native support for identity (via SIM cards), mobility within and roaming across provider networks, and billing/settlement — services that had no equivalent anywhere in the deployed Internet at the time. 4G and 5G networks have since become fully "all-IP," adopting many of the services the Internet pioneered, but that legacy of identity, mobility, and billing as **first-class, built-in** network services — rather than something layered on top by an application — survives as the defining difference between cellular networks and traditional IP networks today, and it's precisely what the 5G Core exists to provide.

This note covers Section 7.4 in full: the Core's overview and motivation, 7.4.1 (the 5G Core and its Network Functions), 7.4.2 (the User Plane Function and tunneling), and 7.4.3 (user identity, registration, and session establishment). Mobility itself — a topic so central it gets its own treatment — is deferred to Section 7.5.

---

## Why a Core Network at All? WiFi vs. Cellular

Before diving into the 5G Core's internals, it's worth being explicit about _why_ this section exists only for cellular networks and not for WiFi — a question the textbook itself raises directly.

|Consideration|WiFi (802.11)|4G / 5G Cellular|
|---|---|---|
|**Historical origin**|Descended directly from wired Ethernet LANs|Descended from 1G/2G telephone-only networks|
|**Identity mechanism**|None built in — a WiFi device is just "another device with an IP address" once associated (Section 7.3.4)|Native, SIM-based global identity (IMSI) issued by a home network|
|**Mobility support**|None native — relies on ordinary IP-layer mechanisms if any|Native mobility within and across provider networks (Section 7.5)|
|**Roaming / billing**|Not a network-layer concept at all|Native roaming and inter-operator settlement, via functions like AUSF and SEPP|
|**Control/management plane**|Identical to any wired Ethernet LAN — no separate "WiFi Core"|A dedicated, centralized 5G Core with its own Network Functions|

> **Analogy — A Landline Extension Cord vs. a National Phone Company:** Plugging a WiFi AP into a router is like running a slightly longer extension cord from a wall jack — the device on the far end of that cord is still just an ordinary member of the same wired household network, with no need for the household to track _who_ it belongs to or _where_ it's roaming. A cellular network, by contrast, inherited the DNA of a national phone company: it was built from day one around the idea that a specific subscriber, identified by a card in their pocket, might show up on _any_ tower, in _any_ city, operated by _any_ partner carrier, and still expect a call to connect and a bill to arrive correctly at month's end. The 5G Core is the direct descendant of that phone company's back office, rebuilt on modern, IP-based, service-oriented (organized as independently callable services) machinery.

---

## 7.4.1 The 5G Core and Network Functions

### Service-Based Architecture

The 5G Core adopts a **Service-Based Architecture (SBA)**: architectural elements of the Core are defined not as monolithic boxes but as **Network Functions (NFs)** and the services each one exposes. This is a deliberate and significant break from 4G and earlier cellular cores, which defined network "entities" that interacted via fixed protocols and were typically implemented as standalone, single-vendor, single-function hardware boxes. Under 5G's service-oriented model, the specifications describe _what_ each NF must do and what services it must expose — not _how_ it must be implemented — and a natural implementation maps each NF onto a **microservice** running in a logically centralized data center. This mirrors the same control/data-plane separation and software-defined mindset introduced with SDN in Chapter 5, and reflects a broader industry evolution from purpose-built "boxes" to flexible, virtualized "services."

### The Big Picture: Figure 7.40

![[Pasted image 20260724163927.png]]

Figure 7.40 (adapted from the 3GPP 23.501 specification) lays out the Core's major components. Of the roughly dozen standardized Network Functions, three deserve special attention as arguably the "top three" — the ones without which nothing else in the Core would function.

### The "Top Three" Network Functions

- **User Plane Function (UPF).** The UPF is responsible for **data-plane** forwarding of traffic between the RAN and the larger Internet. It is the _only_ 5G Network Function that lives in the data plane, which gives it an outsized, central role: every single datagram sent by a device to the Internet, and every datagram sent back, passes through that device's UPF instance. Section 7.4.2 covers the UPF's tunneling machinery in detail.
- **Access and Mobility Management Function (AMF).** The AMF plays the key role in authorizing and establishing a device's access to 5G services, and in managing device mobility (Section 7.5). It is arguably _the_ central Network Function in the 5G control plane: it is the only control-plane NF that exchanges control messages directly with a device (over the **N1** interface) and is one of only two NFs that directly exchange control messages with a base station (over the **N2** interface). To do its job, the AMF works closely with other NFs — for instance, interacting with the **Authentication Server Function (AUSF)** to authenticate a device that wants to join the network. A numbered interface such as N1 is formally called a **reference point** in 3GPP terminology, and defines the interaction between a Core NF and another Core NF (or a device or base station) — the specifications standardize more than 60 such reference points.
- **Session Management Function (SMF).** The SMF manages each device's **session**, including IP address allocation and assignment (services conceptually similar to NAT and DHCP from Chapter 4), the installation of per-device state for policy-based access control and charging, and the establishment of device data-plane forwarding via the UPF.

|Function|Plane|Core Responsibility|Talks Directly To|
|---|---|---|---|
|**UPF**|User (data)|Forwards device traffic to/from the Internet|Base station (N3), Internet|
|**AMF**|Control|Access authorization, mobility management|Device (N1), base station (N2), AUSF, SMF|
|**SMF**|Control|Session/IP management, UPF tunnel setup|AMF, UDM, UPF|

> **Analogy — A Hospital's Admitting Desk, Case Manager, and Orderly:** Think of a patient (the device) arriving at a large hospital (the 5G network). The **AMF** is the admitting desk — the _only_ staff member the patient interacts with directly, who checks the patient's ID, decides whether they're allowed in, and tracks which ward they're currently in as they move around the building. The **SMF** is the case manager working behind the scenes, who isn't seen by the patient at all but who arranges the patient's room assignment, coordinates their ongoing care plan, and bills accordingly. The **UPF** is the orderly who actually wheels the patient's test samples and paperwork physically between departments — the only one of the three who ever touches the actual "payload."

### The Supporting Cast: Data, Identity, and Exposure

Beyond the top three, several other NFs handle data storage, identity, and controlled external access:

- **Unified Data Management (UDM)** manages device profile, service, and authentication information. The UDM may store this data internally, or externally in the **Unified Data Repository (UDR)** — a design choice that, when made, allows stateless client/server transactions (much like the stateless request/response nature of HTTP itself) between the UDM and UDR, which in turn naturally supports scalable, resilient NF implementations in a cloud environment. Other NFs and third-party applications can similarly store and retrieve data from the UDR and from the **Unstructured Data Storage Function (UDSF)**.
- **Local breakout**, introduced in Section 7.3.3, is the capability for application code to run on servers in the 5G Core — or even at the base station itself — and communicate directly with devices to implement an application's service. The **Network Exposure Function (NEF)** and **Application Function (AF)** are what expose select network data and NF services to these local-breakout applications, and vice versa.
- **Roaming** support: when a device wants to join a network that isn't its own **home network** (i.e., not the network associated with its paid subscription plan), the AUSF in the network being joined (the **visited network**) interacts with its peer AUSF in the device's home network, to perform authentication and to determine what services the device is entitled to. Two networks' authentication functions communicate securely with each other via their **local Security Edge Protection Proxy (SEPP)** Network Function — a dedicated, trusted gateway for exactly this kind of inter-operator control-plane traffic.
- **Network Repository Function (NRF).** With potentially dozens of NF instances active in a large deployment, NFs need a way to register their own services and discover the services of others. The NRF provides this automatic registration and discovery, obviating (removing the need for) complex manual configuration every time a new NF instance comes online.
- **Policy Control Function (PCF).** The PCF provides control, management, and a centralized repository of control policies used for mobility management, per-device quality of service, and roaming.
- **Network slicing** and the **Network Slice Selection Function (NSSF)**. Network slicing is one of 5G's genuinely new concepts: the virtualization of RAN and Core resources so that multiple _logical_ networks can share the same underlying _physical_ resources — conceptually much like the way multiple virtual private networks (VPNs, Section 6.5) can operate over the same set of physical resources in traditional IP networks. The NSSF is the NF that plays the central role in selecting the appropriate slice for a device.

> **Analogy — Carving One Office Building into Separate Suites:** Network slicing is like a landlord subdividing one physical office building into several fully separate leased suites — one tenant's guests never wander into another tenant's floor, and each suite can be furnished and configured differently (different QoS, different security policy), even though every suite ultimately shares the same building foundation, elevators, and wiring (the same physical RAN and Core hardware).

### How Network Functions Talk to Each Other

NF services may be accessed using two different styles of client/server interaction, and the standards deliberately don't mandate or recommend one approach over the other:

- The traditional **request/response** model (Chapter 2), which in a 5G data-center setting is typically implemented via a protocol like HTTP, or via remote procedure calls.
- The **subscribe/notify** model — also known as the **publish/subscribe model** in the distributed-systems literature — in which a client registers its interest in a service, and the server later notifies (pushes results to) the client once results become available. This is the identical model that underpins Google's Orion SDN platform, encountered back in Chapter 5.

|Model|How It Works|Good Fit For|
|---|---|---|
|**Request/response**|Client asks, server answers immediately (HTTP, RPC)|One-off lookups, immediate data needs|
|**Subscribe/notify**|Client registers interest once, server pushes updates later|Ongoing state changes an NF needs to react to (e.g., mobility events)|

---

## 7.4.2 User Plane Function

### Recap: What the UPF Does, and Why It Needs More Than Plain IP

As established above, the UPF handles the data-plane forwarding of traffic between the RAN and the larger Internet. In many ways, this is functionally identical to what we've already studied in Chapters 4 and 5 — ordinary IP-layer forwarding through routers and links. However, the 5G control plane configures this data plane to support two capabilities not always found in traditional IP networks: **quality of service (QoS) support** (an advanced topic beyond this section's scope) and **tunneling**, which is the focus here.

Recall from Chapters 4 and 6 that tunneling provides the abstraction of a virtual link between two IP devices — a way of wrapping one datagram inside another so that the intermediate infrastructure along the path can remain entirely oblivious (unaware) of the payload it's carrying. In a 5G setting, a tunnel is established for **each individual device**, between its base station and its own instance of the User Plane Function.

### Figure 7.41: Tunneling in the 5G Protocol Stack

![[Pasted image 20260724164149.png]]

Having studied the RAN protocol stacks in detail in Section 7.3, we can now focus specifically on the data-plane protocol stacks at the base station and at the UPF in the 5G Core. As discussed in Section 7.4.3, the SMF establishes a tunnel between the base station and the device's UPF instance when the device joins the network. When a UPF receives a datagram from the external Internet destined for one of "its" devices, it encapsulates that datagram inside a UDP segment using the **GPRS Tunneling Protocol (GTP)** — specifically its user-plane variant, **GTP-U**. That UDP segment then becomes the payload of a _new_ IP datagram, which is forwarded from the UPF, through the **backhaul network** (the sea of routers and links between the base station and the Core), to the base station. On the receiving end, the base station decapsulates the tunneled UDP datagram, extracts the original encapsulated IP datagram, and forwards it over the RAN to the device — exactly as we'd carefully examine addresses in any "datagram-within-a-datagram" scenario, echoing the mixed IPv4/IPv6 tunneling discussion of Section 4.3.

### Why Tunnel At All? The Mobility Answer

The protocol mechanics of GTP tunneling are fairly straightforward on their own; the more fundamental question is _why_ tunneling is used in the first place. Chapter 4 taught us that tunneling lets IPv4 and IPv6 networks interoperate — but a pure 5G network doesn't have that problem. The real advantage here lies squarely in **device mobility** (Section 7.5): a device's serving base station changes as it physically moves through the cellular provider's network.

Consider what would happen _without_ tunneling. Every single router in the backhaul network would need to maintain up-to-date, granular (fine-grained, per-item) information about which specific base station each device is currently attached to, in order to correctly forward a datagram to that device — and it would need this for **every device** in the provider's network simultaneously. That's a genuinely onerous (burdensome) bookkeeping requirement to push onto ordinary backhaul routers whose only real job should be moving packets efficiently.

With tunneling, by contrast, only the tunnel _endpoint_ — the UPF — needs to know the identity of the base station to which a given device is currently attached; the backhaul routers only need to know how to route datagrams to and from base stations in general, a much simpler and far more stable piece of information. The UPF thus serves as an **anchor point**: a fixed, stable reference point for forwarding datagrams between a device and the larger Internet, regardless of which particular base station that device happens to be attached to at any given moment.

> **Analogy — A Single Forwarding Office vs. Notifying Every Post Office:** Imagine you move apartments frequently within a large city. Without a forwarding system, _every single postal sorting facility_ in the city would need to keep an up-to-date record of your current address, just in case a letter passed through it. That's obviously impractical. Instead, the postal service uses exactly one forwarding office: you tell it your current address, and it alone is responsible for redirecting mail to wherever you actually live _right now_ — every other sorting facility along the route just needs to know how to get mail to that forwarding office, nothing more. The UPF is that forwarding office, and the tunnel between it and your current base station is the "we know where you actually live today" piece of information that only the UPF needs to keep current.

|Without Tunneling|With Tunneling|
|---|---|
|Every backhaul router must track every device's current base station|Only the UPF (tunnel endpoint) must track a device's current base station|
|Bookkeeping scales with (routers) x (devices)|Bookkeeping is isolated to one anchor point per device|
|Mobility requires updating the entire backhaul network|Mobility only requires re-anchoring the tunnel at the UPF|

---

## 7.4.3 User Identity, Registration, and Session Establishment

### Identity: SIM Cards

Section 7.3.4 covered how a device discovers a 5G network's existence and makes its initial attachment to a base station — but as noted there, the device still hasn't actually _joined_ the network at that point, since it hasn't yet exchanged any control messages with the 5G Core. Joining requires the device to first identify itself.

As most cell phone owners already know, a 4G/5G device needs a physical **SIM (Subscriber Identity Module)** card, or more recently a software-only **eSIM**, to attach to a network. Your SIM uniquely identifies you and your **home network** to cellular providers globally — in the cellular world, your SIM is, quite literally, your global network identity. The information stored on a SIM typically includes:

- **IMSI (International Mobile Subscriber Identity).** An IMSI has three parts: a 3-digit (base-10) **mobile country code (MCC)**, a 2- or 3-digit **mobile network code (MNC)**, and a **mobile subscription identification number (MSIN)**. The MCC and MNC together define the SIM's home network — i.e., which cellular provider issued the SIM and with whom you hold a subscription plan. The MSIN then uniquely identifies your specific SIM _within_ that home network. Your SIM also has an associated cellular phone number and an associated set of cellular services you're entitled to access when attached to your home network. When you're in an area not served by your home network, you may attach to another provider's network instead — you're then said to be **roaming** on a **visited network**, and the set of services available to you may differ, depending on your subscription plan and on the charging/settlement arrangement between your home network and the visited network. Some visited networks may, with great frustration, simply be off-limits to you entirely.
- **Cryptographic keys and PINs**, used for authenticating the device when it joins the network (a topic developed further in Chapter 8).
- **Local Area Identity information**, listing current and recently visited networks, used to help identify which network a device should try first when seeking cellular service.
- **Contact lists and text messages**, which may also be stored directly on the card.

> **Analogy — A Passport and Its Issuing Office:** An IMSI functions much like a passport number combined with the country and office that issued it. The MCC+MNC pair is like "issued by the passport office of Country X, branch Y" — it tells any border agent (any network) exactly which authority to contact to verify you. The MSIN is your individual passport number within that office's records — unique to you, but meaningless without first knowing which office issued it. And just as a passport lets you travel to (roam into) other countries under terms your home country has negotiated with them, your SIM lets your device roam onto other operators' networks under whatever settlement terms your home network has struck with them.

### Two Steps to Fully Joining: Registration and PDU Session Establishment

When a device wants to join a 5G network, there are two distinct sets of actions undertaken:

1. **Registration** — the device identifies and authenticates itself to the network, and vice versa.
2. **PDU Session Establishment** — the network allocates an IP address for the device and creates a data-path between the device and a User Plane Function. ("PDU" is 3GPP jargon for "Protocol Data Unit" — essentially, a packet.)

After both steps complete, the device is fully attached to the network and ready to go — able to communicate while remaining mobile. Both registration and session establishment are performed by Network Functions in the 5G Core, with the **AMF playing the central, orchestrating role** throughout — it is, recall, the only Network Function that communicates directly with both the base station and the device, and it coordinates authentication (with the AUSF) and session establishment (with the SMF) on the device's behalf.

### Figure 7.42: The Full Sequence

![[Pasted image 20260724164354.png]]

**Registration, walked through.** Recall from Section 7.3.4 that the device and base station first exchange RRC connection setup request/response messages, establishing a RAN signaling channel between themselves — but the device still isn't yet registered with the Core. After this initial exchange, the base station selects an instance of the AMF for that device and sends it a registration request. The AMF contacts the device and requests device identity information, which is then passed to the Authentication Function to perform mutual authentication with the device (the full details of this authentication are postponed to Chapter 8). If the network being joined is _not_ the device's home network, the local Authentication Function in the visited network serves purely as a proxy to the Authentication Function in the device's home network, which actually performs the authentication. Following successful authentication, the AMF registers the device's identity and subscribed service information in the local UDM, and returns a registration acceptance message to the device.

**Session establishment, walked through.** The device initiates its own data-plane setup by sending a PDU session establishment request to the AMF, which in turn selects an SMF that will be responsible for establishing and managing the device's data plane while it remains attached to the network. After verifying, via the User Data Management Function, that the device's requested session parameters are permissible, the SMF selects an instance of the User Plane Function and establishes the tunnel between the base station and that UPF instance (exactly the GTP-U tunnel described in Section 7.4.2). The SMF then returns the tunnel, IPv4 address information, and additional session information to the device via the AMF and the base station. The final step is for the base station and the device to interact to configure the device's RAN data plane — once that's done, the device is ready to send its first datagram into the Internet.

|Phase|Trigger|Key Actor|Outcome|
|---|---|---|---|
|**Registration**|Device sends registration request via base station|AMF (orchestrates), AUSF (authenticates)|Device identity verified; registered in UDM|
|**Session establishment**|Device sends PDU session establishment request|AMF selects SMF; SMF selects UPF|IP address assigned; GTP tunnel to UPF established|

> **Analogy — Checking Into a Hotel, Then Being Handed a Room Key:** Registration is like walking up to a hotel's front desk (the AMF) and presenting ID (identity request/response) so the desk clerk can confirm, via a phone call to a central reservations system (the AUSF, possibly proxying to your home hotel chain if you're at a partner property), that you really are who you say you are and that you have a valid reservation. Only _after_ that is confirmed does the desk actually assign you a specific room and hand you a key — that's PDU session establishment, where the SMF (a case-manager-like role, again) works out exactly which "room" (UPF, tunnel, IP address) you'll be using for the rest of your stay.

The description above illustrates only the most important interactions among the device, base station, and the various Core Network Functions. The **Policy Control Function** and a **Charging Function** are also involved in session establishment, and additional steps are needed to configure IPv6, if desired — the full process, described across some thirty-four rather dense pages of the 3GPP TS 23.502 specification, is considerably more elaborate than this summary suggests.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Global, persistent IMSI identity**|An IMSI is a stable, long-lived global identifier; if intercepted (e.g., by a rogue base station acting as an "IMSI catcher," as flagged in Section 7.3), it can be used to track or impersonate a subscriber across networks and over time|Minimizing how often the raw IMSI is transmitted in the clear, and pairing identity exchange with strong mutual authentication (Chapter 8) before any sensitive data is trusted, limits exposure|
|**AMF as sole direct contact point**|Because the AMF is the _only_ Core Network Function communicating directly with both device and base station, compromising or overloading it is a uniquely high-leverage single point of failure for the entire Core's access control|Redundant AMF instances, strict input validation on N1/N2 messages, and rate-limiting registration requests reduce the blast radius of any single AMF compromise|
|**Network slice isolation**|A misconfigured or exploited boundary between logical network slices sharing the same physical RAN/Core resources could let one tenant's traffic or control data leak into or interfere with another slice|Rigorous enforcement of slice isolation at the NSSF and throughout the shared infrastructure, plus regular auditing of per-slice resource boundaries, is essential in any multi-tenant slicing deployment|
|**Inter-operator roaming trust (AUSF/SEPP)**|A malicious or compromised visited network's AUSF, communicating with a subscriber's home AUSF via SEPP, sits in a trusted position to manipulate authentication exchanges or harvest subscriber data during roaming|SEPP's role as a dedicated security edge proxy should enforce strict validation of inter-operator messages, and home networks should be cautious about which visited networks they extend trust to|
|**UPF as a trusted tunnel anchor**|Because backhaul routers simply forward tunneled traffic to whatever UPF instance is named as the tunnel endpoint, a spoofed or compromised UPF is well-positioned to intercept or redirect a device's entire data-plane traffic — a data-plane man-in-the-middle|Mutual authentication and integrity protection on the GTP-U tunnel setup, and monitoring for unexpected UPF reassignment, help ensure the "anchor" a device is relying on is genuinely legitimate|
|**Automatic NF registration via NRF**|Because the NRF exists specifically to automate NF registration and discovery, a rogue or spoofed Network Function that successfully registers with the NRF could be discovered and queried by legitimate NFs as if it were trustworthy|Strong authentication and authorization requirements for any entity registering with the NRF, plus continuous validation of registered NF identities, prevents this class of impersonation|

---

## Questions I Still Have

- [ ] What exactly determines whether the UDM stores a device's data internally versus delegating to the external UDR — is that a deployment-time choice, or does it vary systematically by the type of data involved?
- [ ] How does a local SEPP actually authenticate its peer SEPP in another operator's network before extending trust to inter-provider control-plane messages during roaming?
- [ ] The text notes NF communication may use either request/response or subscribe/notify — is that choice standardized per specific reference point (like N1, N2, N11), or left up to individual vendor implementations?
- [ ] When a device performs a handover to a new base station mid-session (Section 7.5), does the SMF establish an entirely new GTP tunnel to the same UPF, or does the existing tunnel simply get re-anchored to the new base station?
- [ ] The PDU session establishment sequence mentions the Policy Control Function and a Charging Function are "also involved" — what specific policy decisions or billing events actually fire during this exchange that Figure 7.42 doesn't show?
- [ ] For eSIM devices specifically, does the IMSI/MCC/MNC/MSIN structure work identically to a physical SIM, or does remote eSIM provisioning introduce different or additional registration steps?

---

## Key Terms — Quick Reference

| Term                                                | Definition                                                                                                                                                                               |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **5G Core**                                         | The centralized, all-IP collection of Network Functions providing control-plane and management-plane services (identity, mobility, security) plus data-plane forwarding for a 5G network |
| **Service-Based Architecture (SBA)**                | 5G's architectural approach defining Core elements as independently callable Network Functions and their exposed services, rather than fixed-protocol "boxes"                            |
| **Network Function (NF)**                           | A standardized, individually specified Core component (e.g., AMF, SMF, UPF) defined by the service it provides rather than by a specific implementation                                  |
| **User Plane Function (UPF)**                       | The sole data-plane NF; forwards device traffic between the RAN and the Internet, and serves as the tunneling anchor point for each device                                               |
| **Access and Mobility Management Function (AMF)**   | The central control-plane NF; the only NF communicating directly with both device (N1) and base station (N2); manages access authorization and mobility                                  |
| **Session Management Function (SMF)**               | Manages a device's session: IP address allocation, per-device policy/charging state, and UPF tunnel establishment                                                                        |
| **Unified Data Management (UDM)**                   | Manages device profile, service, and authentication information, optionally delegating storage to the UDR                                                                                |
| **Unified Data Repository (UDR)**                   | External data-storage NF the UDM (and others) can use, enabling stateless, cloud-friendly data access                                                                                    |
| **Unstructured Data Storage Function (UDSF)**       | Stores unstructured data on behalf of other NFs and third-party applications                                                                                                             |
| **Network Exposure Function (NEF)**                 | Exposes select network data and NF services to local-breakout applications                                                                                                               |
| **Application Function (AF)**                       | A local-breakout application-layer function that interacts with exposed NF services                                                                                                      |
| **Local breakout**                                  | Running application code at the base station or in the 5G Core so client-server traffic never leaves the cellular network                                                                |
| **Authentication Server Function (AUSF)**           | Performs device authentication, interacting with the AMF and, during roaming, with a peer AUSF in the device's home network                                                              |
| **Security Edge Protection Proxy (SEPP)**           | The secure gateway through which two operators' authentication functions communicate during roaming                                                                                      |
| **Network Repository Function (NRF)**               | Provides automatic registration and discovery services for Network Functions                                                                                                             |
| **Policy Control Function (PCF)**                   | Provides centralized control-policy repository and management for mobility, QoS, and roaming                                                                                             |
| **Network slicing**                                 | Virtualizing RAN and Core resources into multiple logical networks sharing the same physical infrastructure                                                                              |
| **Network Slice Selection Function (NSSF)**         | The NF responsible for selecting the appropriate network slice for a device                                                                                                              |
| **Reference point**                                 | 3GPP's term for a standardized, numbered interface (e.g., N1, N2, N3) defining interaction between two Core components                                                                   |
| **Request/response model**                          | A client/server interaction style where a client asks and immediately receives an answer (e.g., via HTTP or RPC)                                                                         |
| **Subscribe/notify (publish/subscribe) model**      | A client/server interaction style where a client registers interest once and the server pushes results later                                                                             |
| **Tunneling**                                       | Wrapping one datagram inside another to create the abstraction of a virtual link between two IP devices                                                                                  |
| **GPRS Tunneling Protocol (GTP / GTP-U)**           | The protocol used to encapsulate user-plane datagrams within UDP for tunneling between a base station and the UPF                                                                        |
| **Backhaul network**                                | The routers and links connecting a base station to the 5G Core                                                                                                                           |
| **Anchor point**                                    | A fixed reference point (the UPF) for forwarding a device's traffic, independent of which base station it's currently attached to                                                        |
| **SIM (Subscriber Identity Module)**                | A physical card uniquely identifying a device/subscriber and their home network to cellular providers globally                                                                           |
| **eSIM**                                            | A software-only equivalent of a physical SIM card                                                                                                                                        |
| **IMSI (International Mobile Subscriber Identity)** | A subscriber's global identity: MCC + MNC (home network) + MSIN (unique ID within that network)                                                                                          |
| **MCC / MNC**                                       | Mobile Country Code / Mobile Network Code — together identify a SIM's home network                                                                                                       |
| **MSIN**                                            | Mobile Subscription Identification Number — uniquely identifies a SIM within its home network                                                                                            |
| **Home network**                                    | The cellular network with which a device holds its paid subscription plan                                                                                                                |
| **Visited network**                                 | A network a device attaches to while roaming outside its home network's coverage                                                                                                         |
| **Roaming**                                         | Attaching to a visited network when outside home-network coverage, under home/visited settlement terms                                                                                   |
| **Registration**                                    | The process by which a device identifies and authenticates itself to a 5G network (and vice versa)                                                                                       |
| **PDU Session Establishment**                       | The process allocating a device's IP address and creating its data-path to a UPF instance                                                                                                |
| **PDU (Protocol Data Unit)**                        | 3GPP terminology for what is, in effect, a packet                                                                                                                                        |
| **N1 / N2 / N3**                                    | Reference points connecting, respectively, device↔AMF, base station↔AMF, and base station↔UPF                                                                                            |

---

## Related Concepts

---

→ Next: [[MOBILITY]]