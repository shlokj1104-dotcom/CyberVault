---
title: MOBILITY
date: 2026-07-24
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 7.5 Mobility

> **One-Line Summary:** **Mobility** — a device's ability to keep communicating uninterrupted while physically moving — ranges across three degrees of difficulty (within a single access network, among access networks in one provider's network, and among multiple providers), is handled by **WiFi** only at the **link layer** within a single subnet-bound **Extended Service Set (ESS)**, but is handled by **5G** as a full **network-wide** undertaking spanning both the **RAN** and the **5G Core** via a carefully orchestrated three-phase **handover**, while the traditional wireline Internet has never deployed its own **Mobile IP** standard despite it existing for 25+ years.

---

## Core Idea

Sections 7.2 through 7.4 built up the pieces of a wireless network one layer at a time: the physical layer and link characteristics (7.2), the access network itself — WiFi or cellular RAN (7.3) — and the wireless core network sitting behind the base station (7.4). This section turns to what the textbook calls one of the most unique and interesting aspects of wireless networking: the need to support device **mobility**. It's a topic important enough to earn an entire section of its own, because a device that can move while communicating — without dropping its ongoing TCP connections or needing to rejoin the network from scratch — requires machinery that simply doesn't exist in a stationary wired network.

The section unfolds in four parts, matching the chapter's own structure: 7.5.1 introduces mobility **principles** — what "mobility" actually means, the three degrees of it, and the fundamental design questions any solution must answer. 7.5.2 and 7.5.3 then show how those principles play out in two very different real systems: WiFi's minimal, link-layer-only approach, and 5G's extensive, network-wide approach involving both RAN and Core. Finally, 7.5.4 zooms out to ask why the _traditional_, non-cellular Internet — despite having a technical solution (Mobile IP) ready for over 25 years — never actually deployed native mobility support, and what dual-technology compromise we've settled into instead.

---

## 7.5.1 Mobility Principles

### What Is "Mobility"?

The chapter is deliberately careful in defining mobility, since the word could mean several different things. The definition used here: a device is "on" and communicating _while it moves_, and the question of interest is how _far_ such a device can move **while communicating without interruption** — meaning its ongoing TCP connections keep flowing, and it never has to explicitly rejoin any new network from scratch. Drawing on a distinction familiar from Chapter 4's treatment of inter-domain versus intra-domain routing, three distinct scenarios emerge, depending on whether a device stays within one access network, moves among several access networks under one provider, or moves among multiple different providers' networks entirely.

### The Three Degrees of Mobility — Figure 7.43

```
Fig 7.43 -- Degrees of Mobility
------------------------------------------------
 Limited <----------------------------> High
   (a)          (b)              (c)
 single AN   multi-AN, ONE     multiple
 (assoc/     provider (HO,     providers
 disassoc)   same subnet)      (coord HO)
```

![[Pasted image 20260724170000.png]]

- **(a) Mobility only within a single access network.** At the low-mobility end of the spectrum, a device maintains connectivity only while attached to one given access network. To move between access networks, the device must fully **disconnect** from its current base station/AP and then **connect** to a new one elsewhere — no continuity is preserved across that gap. The textbook's own example: a student disconnects and powers down their laptop leaving a classroom WiFi network, walks to the dining commons, and connects fresh to that network's WiFi — then repeats the same disconnect/reconnect dance walking to the library. The device simply serially **associates** with (joins) and later **disassociates** from (leaves) each wireless network it encounters in turn. Critically, this limited case requires _no new mechanisms at all_ — it's fully handled by machinery already covered in Sections 7.3 and 7.4.
- **(b) Mobility among access networks within a single provider network.** This is where real interest in device mobility begins: a device moves among _multiple_ access networks belonging to one provider, while continuing to send and receive datagrams _and_ while maintaining higher-layer connections (e.g., an ongoing TCP session) throughout. This requires the network to provide **handover** — a transfer of responsibility for forwarding a device's datagrams from one base station to another, as the device moves among WLANs or RANs. Because the handover happens entirely within one provider's network, that single provider can orchestrate it end-to-end, on its own, without needing to coordinate with anyone else.
- **(c) Mobility among multiple network providers.** At the high-mobility end, a device roams between access networks belonging to _different_ providers entirely. Here, the providers themselves must coordinate handover to ensure communication continues uninterrupted — a significantly more complex undertaking than case (b), precisely because it now requires inter-provider agreement and signaling rather than one operator acting unilaterally. The textbook notes this scenario occurs less often as cellular providers' geographical footprints keep growing, and defers a full treatment of it to earlier editions of the book.

> **Analogy — Changing Seats, Changing Rooms, Changing Buildings:** Case (a) is like getting up from your desk, packing your bag entirely, and sitting down fresh at a different desk in a different room — nothing about your "session" carries over automatically. Case (b) is like a walkie-talkie conversation that seamlessly continues as you walk from one room of a single office building to another, because the building's own internal PA system quietly hands your signal off from one antenna to the next without you noticing. Case (c) is like that same walkie-talkie conversation continuing as you walk out of your own company's building entirely and into a neighboring company's building across the street — which only works at all because the two companies had _already_ struck a deal in advance to relay each other's signals.

### Two Fundamental Design Questions

Any approach to mobility within a single provider's network (case b) must resolve two fundamental questions:

1. **Is mobility supported only at the link layer, or is the network layer involved as well?** If mobility is supported _only_ at the link layer, a device can move among wireless access networks, but — because incoming datagrams from the larger Internet are always forwarded to the device's current **subnet** (in the addressing sense of Section 4.3) — its mobility range is restricted to staying within that one subnet while communicating. This is the approach WiFi takes (Section 7.5.2). If, on the other hand, mobility is supported across the provider's _entire_ network, network-layer (and/or higher-layer) support is needed: specifically, the core network must track which particular access network a device is currently attached to, so that incoming Internet datagrams can be forwarded to the correct subnet for delivery. This is the approach 5G (and 3G/4G) cellular networks take (Section 7.5.3).
2. **Why is handover initiated, and who initiates it?** As a device moves, the quality of the radio signal between it and its base station (and hence achievable throughput and latency) changes — a device might change access networks purely to improve its own performance. But the _network_ itself might also choose to move a device from one access network to another in order to **load-balance** devices among its access networks. Neither the WiFi nor the 5G standards specify a particular algorithm for _when_ to hand over or _which_ target base station to pick — that decision is deliberately left to individual network operators and remains an active area of research. To make that decision at all, a device typically measures the signal quality of every base station within range and reports this back: in 5G, base stations periodically broadcast reference signal symbols, and a device periodically reports back a 4-bit **Channel Quality Indication (CQI)**; in WiFi, devices measure and report a **Received Signal Strength Indicator (RSSI)** for each nearby AP. More sophisticated approaches can even use machine learning to _predict_ future mobility and signal strength, proactively triggering handover rather than reacting only to current measurements.

|Design Question|Link-Layer-Only Answer|Network-Layer-Involved Answer|
|---|---|---|
|**Mobility range**|Confined to staying within current subnet|Can span the provider's entire network|
|**Who tracks device location**|Nobody beyond the current AP/subnet|Core network explicitly tracks current access network|
|**Real-world example**|Enterprise WiFi (Section 7.5.2)|3G/4G/5G cellular (Section 7.5.3)|

### How Is Handoff Accomplished?

Once a handover is deemed necessary, two important tasks must be accomplished:

- **Transferring device state.** Information such as a device's authenticated status and its allowed services within the WLAN or RAN control plane, held at the _old_ base station, must be transferred over to the _new_, neighboring base station. Without this transfer, a device would effectively have to disassociate from the old base station and associate with the new one completely from scratch, as if it were joining fresh.
- **Redirecting the data-plane flow.** To keep datagrams flowing to and from the device uninterrupted, forwarding tables in routers and switches along the path must be modified to redirect traffic to and from the device's new point of attachment to the network.

> **Analogy — A Relay Baton Pass, Not a Restart:** A well-executed handover is like a relay race baton pass — the incoming runner (old base station) doesn't just drop the baton on the ground for the outgoing runner (new base station) to go find; the two runners run alongside each other briefly, physically transfer the actual baton (device state), and the race clock (the ongoing TCP connection) never stops ticking. Compare this to case (a) mobility above, which is more like each runner sprinting their leg entirely separately, with a full stop-and-restart of the race in between.

---

## 7.5.2 Mobility in a WiFi Network

The IEEE 802.11 WiFi standard adopts a decidedly **link-layer-only** approach to mobility — unsurprising, given that 802.11 is itself fundamentally a link-layer standard. Consequently, WiFi mobility is supported only among access points that lie _within the same subnet_ (in the addressing sense of Section 4.3).

### From BSS to ESS — Figure 7.44

Recall from Section 7.3.2 that the traditional WiFi network, the **Basic Service Set (BSS)**, contains a single AP and one or more attached wireless devices. An **Extended Service Set (ESS)** consists of _multiple_ BSSs — for example BSS₂ and BSS₃ below — and their respective access points, AP₂ and AP₃.

```
Fig 7.44 -- WiFi: handover within the ESS
--------------------------------------------
              Internet
                 |
              Router R1
                 |
              Switch S1
             /          \
          BSS1        Switch S2
          (AP1)        /      \
                    BSS2      BSS3
                    (AP2)     (AP3)
                     o ---> o   (device moves)
        [       Extended Service Set (ESS)      ]
```

![[Pasted image 20260724170200.png]]

Assuming every AP in the ESS uses the same **SSID**, the entire ESS appears logically to be just a single WLAN — both from the device's point of view and from the network layer's point of view — because every device and every AP within that ESS shares the same **layer-3 subnet**. This is the crucial, almost counter-intuitive fact underlying Figure 7.44: a device moving between BSS₂ and BSS₃ is not even "mobile" from a network-layer perspective at all, since it _never leaves its own subnet_ the whole time. Because IP routing decisions only ever look as deep as the subnet, and the device's subnet membership never changes, no network-layer mobility mechanism is needed here whatsoever — ordinary link-layer association/disassociation and switch forwarding-table updates are sufficient.

### Fast BSS Transition and Vendor Extensions

The IEEE 802.11 standard introduced **Fast BSS Transition (FT)** specifically to accelerate the 802.1X authentication process when a device moves between APs within an ESS, and to let APs proactively suggest alternative APs a device might wish to attach to within the ESS. Beyond these standardized capabilities, large WiFi equipment vendors additionally provide their own **proprietary** solutions for mobility — for example, mechanisms to manage device handover between two APs within an ESS that go beyond what the 802.11 standard itself mandates.

> **Analogy — One Building, Many Wi-Fi Repeaters, Same Front Door:** WiFi mobility within an ESS is like moving between rooms of a single large house that happens to have several Wi-Fi range-extenders scattered around it, all broadcasting the exact same network name. Walking from the living room to the upstairs bedroom, your laptop silently switches from talking to the downstairs extender to talking to the upstairs one — but as far as the house's own internet gateway (its one "front door" to the ISP) is concerned, you never moved anywhere at all: you're still just "a device on the house network," full stop.

|Aspect|WiFi (802.11) Mobility|
|---|---|
|**Layer of support**|Link layer only|
|**Scope**|Within a single ESS (same subnet)|
|**Standard acceleration mechanism**|Fast BSS Transition (FT)|
|**Beyond-standard support**|Proprietary vendor solutions (e.g., Cisco 802.11r-based handover)|
|**Limitation**|No mobility once a device would need to cross into a _different_ subnet|

---

## 7.5.3 Mobility in a 5G Network

While 802.11 mobility occurs exclusively at the link layer, mobility in a cellular 5G network is a genuinely **network-wide** undertaking, involving both the RANs _and_ the 5G Core network (Section 7.4). This extensive support for mobility directly reflects 5G's roots in the mobile cellular telephony world discussed back in Section 7.4's Core Idea: the very earliest generations of cellular networks provided call handover with uninterrupted voice service as a user physically ranged from cell tower to cell tower across a single provider's network, and even from one provider's network to another. Because mobility was "baked in" as a first-class service from the earliest cellular network architectures, the later addition of mobile _data_ services followed quite naturally in subsequent generations.

### 5G Handover — the Setup

Consider an active 5G device currently associated with a base station in one RAN — the **source base station** for the handover — that will be handed over to a second base station in another RAN — the **target base station**. The device is "active" in the specific sense that it has an ongoing TCP connection with a remote host on the larger Internet, carried over an active data-plane tunnel to/from its User Plane Function (UPF) in the Core (recall GTP-U tunneling from Section 7.4.2). After the handover completes: the device will be associated with the target base station; the source base station will have released the device state it had been maintaining; the data-plane tunnel to and from the device will now end at the target RAN (assuming, for simplicity, that the same UPF instance continues to serve the device); and every piece of device state held in the 5G Core will reflect that the device is now located at the newly joined base station. That's a great many things that all need to change in a coordinated fashion — which is exactly why 5G handover unfolds in three distinct, carefully sequenced phases.

### Figure 7.45: The Three Phases of 5G Handover

```
Fig 7.45 -- 5G Handover: RAN and Core Actions
------------------------------------------------
Device  SrcBS   TgtBS    AMF   SMF   UPF
 |<--data-->|                        |
 |--meas-->|                         |    Phase 1:
 |         |--HO request-->|        |    Handover
 |         |<--accept?-----|        |    decision
 |<-RRC reconfig-|                  |
 |--detach, attach------->|         |    Phase 2:
 |         |--xfer status, buffer-->|    RAN
 |--RRC reconfig complete->|         |    transfer
 |         |--HO success------------>|
 |         |    (data end marker)<---|
 |<===== device data via target BS =====>|
                |                    |    Phase 3:
                |--path switch-->|--mod req/resp-->  Core
                |<--path switch ack--|<--response---  update
```

![[Pasted image 20260724170400.png]]

**Phase 1 — Making the handover decision.** Handover begins in the RAN, with the device measuring the quality of the radio channels of every base station it can hear, then reporting these measurements back to the source base station (recall CQI reporting from Section 7.5.1). If the source base station judges a handover to be desirable, it sends a handover request message to the target base station, which then decides whether or not to accept the handover and returns that decision (a positive acknowledgment, in the textbook's own worked example) to the source base station.

**Phase 2 — Transferring the device between base stations.** The source base station, target base station, and device now all work together to transfer the device's RAN connection over. This begins with the source base station sending two messages: a **reconfiguration message** to the device itself, and a **transfer status message** to the target base station that hands over the device's held state. The device and the target base station then set up a brand-new RAN connection between themselves. This phase completes when the device sends a **reconfiguration complete** message to the target base station, and the target and source base stations perform a final handshake. Crucially, the endpoint of the data-plane tunnel between the base station and the User Plane Function does _not_ change during this phase — but rather than continuing to deliver datagrams to the device over the _source_ RAN, the source base station instead begins **forwarding** datagrams addressed to the device onward to the target base station, which **buffers** them for later delivery once the device fully reattaches. This buffering is precisely what ensures no incoming datagram destined for the device is ever lost during the handover process.

**Phase 3 — Updating device state and the UPF tunnel in the Core.** Handover activity now shifts fully into the Core. The target base station — now completely in charge of the device's RAN connection — signals to the **Session Management Function (SMF)**, via the AMF, that it is the new base station serving the device being handed over. The SMF then signals the **User Plane Function (UPF)** to change the device's tunnel endpoint over to the target base station. The UPF sends a **data-end marker** to the source base station, letting it know the device's tunnel endpoint has officially switched; the source base station then informs the target base station of this. At this point, the target base station can finally deliver to the device any datagrams it had been buffering from the source base station during handover, and — going forward — datagrams flow directly from the device's UPF straight to the target base station. Following a bit of final cleanup handshaking, the handover process is complete.

|Phase|Where It Happens|Key Actors|What Gets Established|
|---|---|---|---|
|**1. Handover decision**|RAN|Device, source BS, target BS|Agreement to hand over, target BS accepts|
|**2. RAN transfer**|RAN|Device, source BS, target BS|New RAN connection; source BS forwards & target BS buffers datagrams|
|**3. Core update**|5G Core|Target BS, AMF, SMF, UPF|Tunnel re-anchored to target BS; buffered data released|

> **Analogy — Passing a Live Phone Call Between Two Colleagues' Desks:** Imagine you're on a live phone call, and you need to physically move from one colleague's desk (source base station) to another's desk across the office (target base station) without ever hanging up. Phase 1 is your colleague at the new desk agreeing over instant message, in advance, to take over the call ("yes, transfer it here"). Phase 2 is the actual moment of walking over — your original colleague keeps quietly jotting down anything said on the call in the meantime (buffering) rather than letting it get lost, and hands that note over to your new desk the instant you arrive. Phase 3 is the phone company's own back-office switchboard (the Core) quietly updating its own internal records of which desk's phone line is now live for that call, and confirming to your original colleague's desk that it's safe to stop jotting notes because the switchboard has now officially rerouted everything to the new desk directly.

### Beyond the Basic Case: Idle Devices, Tracking Areas, and Inter-Provider Handover

The description above covers only the simplest handover scenario — a device that is fully "active" throughout. Several important real-world complications go beyond it:

- **Idle and Inactive devices.** Recall from Section 7.3 that after periods of inactivity — even longer than the DRX sleep cycles discussed there — a device first enters the **Idle** state and then the **Inactive** state to conserve power. In these states, additional steps must be taken in _both_ the RAN and the Core before the device can send or receive user data again. In particular, if a device moves base stations while Idle or Inactive, the 5G Core may not even know which RAN the device is currently located in at all.
- **Tracking area update** procedures let the Core keep track of the general _area_ in which an Idle/Inactive device is currently located, even without knowing its precise base station.
- **Paging** procedures are then used by the Core to pinpoint the device's specific RAN once it actually needs to be reached — for instance, when an incoming call or datagram arrives for it.
- **Inter-provider handover.** A more complex handover altogether occurs when a device is handed from the RAN of one provider's mobile cellular network over to the RAN of an entirely different provider's network. In this case, the cellular provider to which the device is attached actually changes, and additional inter-provider signaling (beyond the scope of this section) is required.

|Complication|What Changes vs. the Basic Case|
|---|---|
|**Idle/Inactive device**|Device must first re-establish active RAN presence before user data can flow|
|**Tracking area update**|Core tracks a general area, not a precise base station, for sleeping devices|
|**Paging**|Core pinpoints exact RAN only once the device actually needs to be reached|
|**Inter-provider handover**|The serving cellular provider itself changes; requires inter-provider signaling|

---

## 7.5.4 Internet Mobility

### The Puzzle: A Solution That Was Never Deployed

Today's traditional, non-cellular Internet has no widely deployed infrastructure providing the kind of "on the go" mobility services that 5G cellular networks offer. Remarkably, this gap has nothing to do with a lack of _technical_ solutions: a **Mobile IP** architecture and accompanying protocols were standardized in Internet RFCs (RFC 5944) more than 25 years ago, and research into new, more secure, and more generalized Internet mobility solutions has continued since.

### Why Wasn't It Deployed? Business Cases, Not Technology

Given that Mobile IP's technical solutions already existed, the textbook points to the _business_ side of the story as the real explanation: there was arguably a lack of motivating business cases and use cases for it, combined with the timely, first-mover development and deployment of _alternative_ mobility solutions inside cellular networks themselves, which blunted Mobile IP's own deployment prospects. Specifically, 25 years ago, 3G cellular networks had _already_ provided a working solution for mobile voice services — cellular's own "killer app" for mobile users — as well as the beginnings of global mobile data services. Crucially, behind those cellular services sat an entire paid-subscription business model, complete with a home network provider and inter-provider business agreements and payment arrangements that allowed users to roam from their home network onto visited networks (recall Section 7.4.3's discussion of home vs. visited networks). The traditional Internet, by contrast, has simply never had this kind of business infrastructure in place to support roaming.

> **Analogy — A Better Bridge Nobody Needed to Cross:** Mobile IP is a bit like a perfectly good, engineered pedestrian bridge that got built across a river 25 years ago — except by the time it was finished, a ferry service (3G/4G/5G cellular) had _already_ been running reliably and profitably for years, with paying customers, a ticketing system, and agreements with ferry operators on the opposite bank. The bridge wasn't a bad piece of engineering; there just was never enough unmet demand, or money changing hands, to justify anyone actually walking across it instead of taking the ferry they already trusted.

### Today's Reality: A Dual-Technology Compromise

Mobile Internet connectivity today is provided via a **dual-technology solution**: mobile services over cellular networks when we are truly mobile and "on the go" (the center-and-rightmost side of the mobility spectrum in Figure 7.44 — i.e., cases b and c), and Internet services over 802.11 networks or wireline networks when we are stationary, or moving only locally (the center-and-leftmost side of that same spectrum — i.e., case a). The one indisputable fact, however, is that fifteen years ago there were approximately equal numbers of subscribers attached to the Internet via fixed (wired) broadband technologies and via wireless technologies — and while both subscriber bases have grown since then, there are now roughly **five times as many** Internet-connected wireless subscribers as wired subscribers. As the textbook frames it, customers have effectively "voted with their feet," and that dual-technology split — rather than any unified Mobile IP deployment — is what appears likely to persist for the foreseeable future.

|Consideration|Mobile IP (Traditional Internet)|Cellular Mobility (3G/4G/5G)|
|---|---|---|
|**Technical readiness**|Standardized 25+ years ago (RFC 5944)|Deployed and refined over successive generations|
|**Business model behind it**|None — no home/visited network billing infrastructure|Subscription-based, with inter-provider roaming agreements|
|**Real-world deployment**|Never widely deployed|Universally deployed, underpinning "killer app" mobile voice/data|
|**Current subscriber trend**|Roughly flat since equal footing 15 years ago|~5x more wireless subscribers than wired today|

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Device state transfer during handover**|If a device's authenticated status and allowed-services state is intercepted or spoofed as it's transferred from source to target base station, an attacker could potentially impersonate a legitimate device's session or inject forged state|Transfer status and reconfiguration messages between base stations should be integrity-protected and authenticated, so a target base station can trust the state it receives really came from the legitimate source|
|**Buffering at the target base station**|Datagrams held in a target base station's buffer during handover represent a window of stored, in-flight user data that could be a target for local interception if the base station itself is compromised|Buffered data should remain encrypted end-to-end where possible, and buffer contents should be purged promptly once delivered or upon handover failure|
|**Handover decision left unstandardized**|Because the standards deliberately don't specify _which_ algorithm decides when/where to hand over, a malicious or rogue base station could try to lure devices into handing over to it by reporting misleadingly attractive signal characteristics|Devices and networks should cross-validate reported signal quality against multiple sources and be cautious of anomalous, too-good-to-be-true handover targets|
|**Data-end marker as a trust signal**|A forged or delayed data-end marker sent to the source base station could be used to disrupt the clean handoff of the tunnel endpoint, potentially causing dropped or misdirected datagrams|The data-end marker exchange between UPF and source base station should be authenticated as part of the broader trusted Core signaling fabric (see Section 7.4's SEPP/AUSF discussion)|
|**Inter-provider handover trust**|A handover to a rogue or compromised visited-network base station introduces the same inter-operator trust risks flagged in Section 7.4 (roaming, SEPP) — now specifically at the moment of an active, live handover|Inter-provider handover signaling should rely on the same rigorously validated inter-operator trust channels (SEPP) already used for registration and authentication|
|**Idle/Inactive device tracking**|Tracking-area and paging procedures inherently require the Core to know a device's general whereabouts even while it's "asleep" — a broad location-tracking capability that, if leaked or abused, has real privacy implications for the subscriber|Access to tracking-area and paging data should be tightly restricted within the Core, consistent with the general IMSI/location privacy concerns already raised in Section 7.4|

---

## Questions I Still Have

- [ ] When the standards leave the handover decision algorithm entirely up to network operators, are there any published best-practice algorithms in wide use today, or does every major carrier genuinely run something proprietary and different?
- [ ] During Phase 2 of 5G handover, exactly how long can the target base station keep buffering forwarded datagrams before something (a timeout, a buffer limit) forces those datagrams to be dropped anyway?
- [ ] For case (c) — mobility among multiple network providers — what does the actual inter-provider signaling and coordination look like in practice, beyond the textbook's note that it's "significantly more complex"?
- [ ] How exactly does paging work end-to-end: does the Core broadcast a page across every base station in a tracking area simultaneously, or is there a more targeted mechanism to narrow the search first?
- [ ] Given that Mobile IP was standardized 25+ years ago and never deployed, is there any realistic future scenario (e.g., new satellite-Internet business models) where Mobile IP-style network-layer mobility could still see real-world adoption?
- [ ] How does the 5G handover process interact with the QoS guarantees mentioned back in Section 7.4.2 — does an active handover risk any temporary QoS degradation for the device's ongoing session?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Mobility**|A device's ability to remain "on" and communicating while physically moving, without needing to interrupt or explicitly rejoin any ongoing communication|
|**Handover**|A transfer of responsibility for forwarding a device's datagrams from one base station to another, as the device moves among access networks|
|**Association / Disassociation**|The act of a device joining (associating with) or leaving (disassociating from) a wireless access network|
|**Mobility only within a single access network**|The lowest-mobility case: a device must fully disconnect and reconnect when moving between access networks; requires no new mechanisms|
|**Mobility among access networks within a single provider network**|The intermediate case: a device moves among access networks of one provider while maintaining ongoing communication; requires handover|
|**Mobility among multiple network providers**|The highest-mobility case: a device roams between different providers' networks, requiring inter-provider handover coordination|
|**Link-layer-only mobility**|Mobility support confined to the link layer, restricting a device's range to its current IP subnet (WiFi's approach)|
|**Network-layer mobility support**|Mobility support extended into the network layer, letting a device roam across an entire provider's network (5G's approach)|
|**Load-balancing (handover)**|A network-initiated handover performed to redistribute devices more evenly among access networks, rather than to improve one device's own signal|
|**Channel Quality Indication (CQI)**|A 4-bit value a 5G device periodically reports to its base station, based on measured reference signal quality|
|**Received Signal Strength Indicator (RSSI)**|A measured signal-strength value a WiFi device reports for each nearby access point|
|**Basic Service Set (BSS)**|A single WiFi access point together with its one or more associated wireless devices|
|**Extended Service Set (ESS)**|Multiple BSSs (and their APs) sharing the same SSID and the same layer-3 subnet, appearing as a single logical WLAN|
|**Fast BSS Transition (FT)**|An 802.11 standard feature that accelerates 802.1X authentication, and enables AP-suggested handover targets, within an ESS|
|**Source base station**|The base station a device is associated with at the start of a handover|
|**Target base station**|The base station a device is being handed over to|
|**Reconfiguration message**|A message sent by the source base station to the device, beginning the RAN-level transfer of its connection|
|**Transfer status message**|A message sent by the source base station to the target base station, transferring the device's held state|
|**Reconfiguration complete message**|A message sent by the device to the target base station, completing the new RAN connection setup|
|**Data-end marker**|A message the UPF sends to the source base station, signaling that the device's tunnel endpoint has switched to the target base station|
|**Idle state / Inactive state**|Power-conserving device states (beyond ordinary DRX sleep) requiring extra steps in RAN and Core to resume active communication|
|**Tracking area update**|A Core procedure that keeps track of the general area (not precise base station) of an Idle/Inactive device|
|**Paging**|A Core procedure used to pinpoint an Idle/Inactive device's specific RAN when it needs to be reached|
|**Inter-provider handover**|A handover in which a device's serving cellular provider itself changes, requiring additional inter-provider signaling|
|**Mobile IP**|A standardized (RFC 5944), network-layer mobility architecture for the traditional Internet, never widely deployed|
|**Dual-technology solution**|Today's real-world mobility compromise: cellular networks while mobile, WiFi/wireline networks while stationary|

---

## Related Concepts

---

→ Next: [[BLUETOOTH, IOT AND SATELLITE]]