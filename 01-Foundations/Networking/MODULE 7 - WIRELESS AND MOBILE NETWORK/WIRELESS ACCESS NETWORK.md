---
title: WIRELESS ACCESS NETWORK
date: 2026-07-24
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 7.3 The Wireless Access Network

> **One-Line Summary:** Building on the physical-layer foundation of Section 7.2, the wireless **access network** — 802.11 **WiFi** and the 4G/5G **Radio Access Network (RAN)** — solves link-layer problems unique to a shared, unreliable radio medium via **OFDMA** (channel partitioning in frequency and time) and **CSMA/CA** (random access with collision _avoidance_, not detection), then layers on discovery, scheduling, and **energy-aware sleep/wake cycles** to make the whole thing practical on battery-powered devices.

---

## Core Idea

Section 7.2 covered the wireless **physical layer** — how bits become radio waveforms. This section moves up one layer, to the **link-layer** principles and practices of wireless **access networks**: how the radio channel is actually _shared_ among many competing devices and a base station (7.3.1), and how those sharing techniques are specialized into two major real-world access networks — 802.11 WiFi (7.3.2) and the 4G/5G RAN (7.3.3). Later subsections (7.3.4–7.3.6) build further on this common foundation: how a device _discovers_ a network and joins it, how bandwidth is _scheduled_ across upstream and downstream channels, and how devices conserve _energy_ through sleep/wake cycling.

This note now covers the section in full: 7.3.1 through 7.3.6.

---

## 7.3.1 Sharing the Wireless Channel

Section 6.3 introduced three general families of multiple-access technique: **channel partitioning** (TDM, FDM), **random access** (Aloha, CSMA), and **taking-turns** protocols (polling). Wireless networks use _all_ of these, but two combinations dominate modern practice: **OFDMA**, used in 4G, 5G, and WiFi-6/7, and **CSMA/CA**, used in every generation of WiFi.

### FDM, OFDM: Channels and Subchannels

A single wideband channel concentrates a device's entire power spectrum in one contiguous block (Figure 7.18a). Because wideband hardware is difficult to build cheaply, and different devices rarely need the full channel width, a band is more often divided into multiple **narrowband channels** — classic **frequency-division multiplexing (FDM)**. FDM channels are separated by **guard bands**: idle slices of spectrum that prevent one channel's transmission from bleeding into its neighbor (Figure 7.18b).

```
Fig 7.18 -- Wideband vs. FDM vs. OFDM Spectrum
────────────────────────────────────────
 a. Wideband:   [###################]
                -0.5      freq      0.5

 b. FDM:  |ch|gap|ch|gap|ch|gap|ch|
              (guard bands waste spectrum)

 c. OFDM: overlapping, orthogonal lobes
          ~/\~/\~/\~/\~/\~/\~/\~/\~
          (zero-crossings align at peaks
           of neighbors -> no interference,
           no guard bands needed)
```

![[Pasted image 20260724150000.png]]

**Orthogonal frequency division multiplexing (OFDM)** modulates the FDM subchannel carriers so that, although adjacent channels' power spectra now _overlap_, each subchannel's signal is mathematically zero exactly at its neighbors' peak frequencies — the signals cancel each other out at the points that would otherwise cause interference. Because guard bands become unnecessary, OFDM packs strictly more usable spectrum into the same bandwidth than FDM — it has higher **spectral efficiency**, a genuinely valuable property (recall from 7.2 that spectrum is a scarce, auctioned commodity in licensed bands).

> **Analogy — Choir Risers vs. Overlapping Voices:** FDM is like seating a choir with an empty riser between every singer so voices never blend — safe, but wasteful of stage space. OFDM is like seating singers shoulder-to-shoulder but training each to hit a note precisely at the instant their neighbor's note passes through silence — their voices interleave without ever actually clashing, so the same stage fits more singers.

### Orthogonal Frequency Division Multiple Access (OFDMA)

**OFDMA** combines OFDM's frequency-division with **time-division** multiplexing. Each OFDM subchannel is further sliced into short time **minislots**, producing a two-dimensional grid of frequency × time cells. The smallest addressable cell — one minislot at one subchannel frequency, carrying exactly one modulation symbol — is called a **Resource Element (RE)**. In 4G/5G, an RE spans 15 kHz and lasts 66.6 μsec; in WiFi (which supports OFDMA from WiFi-6 onward), an RE spans 78 kHz and lasts 12.8 μsec.

```
Fig 7.19 -- OFDMA: Frequency x Time Grid
────────────────────────────────────────
 Freq ▲ ● ● ● ● ● ● ●   subchannel 12
      │ ● ● ● ● ● ● ●     ...
      │ ● ● ● ● ● ● ●   subchannel 2
      │ ● ● ● ● ● ● ●   subchannel 1
      └───────────────▶ Time (minislots)
   each ● = one Resource Element (RE)
```

![[Pasted image 20260724150200.png]]

REs are never scheduled individually. Instead, adjacent REs are grouped — same modulation, same power — into a **resource block (RB)** in 4G/5G, or a **Resource Unit (RU)** in WiFi. A base station or wireless device transmits its data as symbols across one or more RBs/RUs, as scheduled in Section 7.3.5.

### Random Access: Aloha and CSMA/CA

**Aloha** is the simplest random-access protocol: a node with data simply transmits, using the full channel bandwidth, whenever it has something to send. If two transmissions overlap in time, they **collide**, and a higher-level retransmission mechanism recovers the lost data. Aloha, over 50 years old, still underlies certain control-plane functions in 4G/5G and IoT networks today.

**Carrier Sense Multiple Access with Collision Avoidance (CSMA/CA)** has been WiFi's channel-access method since its inception, and remains available (alongside OFDMA) in every WiFi generation through WiFi-6/7. Like the wired CSMA protocols of Section 6.3.2, it listens before transmitting — but a wireless channel forces two adaptations absent from wired Ethernet's CSMA/CD:

- **The hidden terminal problem** (introduced physically in 7.2.1): a transmitter may not hear a "hidden" second transmitter that is also sending to the same intended destination, so sensing the channel as idle doesn't guarantee it actually is.
- **No collision detection.** Detecting a collision requires transmitting and receiving _simultaneously_ on the same radio — but a device's own transmitted signal is vastly stronger, at its own antenna, than any incoming signal it might be trying to detect underneath it. Consequently, once a WiFi station begins sending a frame, it transmits the frame in its **entirety**; there's no turning back mid-transmission the way Ethernet aborts a colliding frame.

Because collisions at the _destination_ are consequently more prevalent in wireless networks than wired ones, CSMA/CA adds an explicit **link-layer acknowledgment**: after correctly receiving a frame (verified via the CRC), the destination waits a brief **Short Inter-frame Spacing (SIFS)** and sends back an ACK.

```
Fig 7.20 -- CSMA/CA Link-Layer Acknowledgment
────────────────────────────────────────
 Source              Destination
   │  DIFS               │
   ├───────DATA─────────>│
   │                 SIFS│
   │<───────ACK──────────┤
```

![[Pasted image 20260724150400.png]]

If no ACK arrives, the sender assumes a collision (or other error) occurred and retransmits, up to some fixed number of retries, using CSMA/CA anew.

The full **CSMA/CA protocol** runs as follows:

1. If the station senses the channel idle, it transmits its frame after a short **Distributed Inter-Frame Space (DIFS)**.
2. Otherwise, it chooses a random **binary exponential backoff** value (recall Section 6.3.2) and counts it down while the channel is sensed idle, after DIFS. The counter _freezes_ while the channel is busy.
3. When the counter reaches zero (which can only happen while the channel is idle), the station transmits the entire frame and waits for an acknowledgment.
4. If an ACK is received and another frame is queued, the station re-enters the protocol at step 2. If no ACK is received, the station re-enters the backoff phase (step 2), now drawing from a **larger** interval.

**Why does CSMA/CA differ from Ethernet's CSMA/CD here?** Consider two stations, each with a frame ready, both sensing that a third station is already transmitting. Under CSMA/CD, both immediately transmit the instant the third station finishes — a resulting collision is _cheap_, because both stations detect it and abort. Under CSMA/CA, by contrast, a frame suffers no such graceful abort once sent — so instead, _both_ stations enter random backoff immediately upon sensing the channel busy, and continue counting down only once it's sensed idle again. If the two stations draw sufficiently different backoff values, the "winning" station transmits first; the "losing" station, hearing the winner's transmission, freezes its counter and defers — a costly collision is thereby _avoided_. Collisions can still occur, though, if the two stations are hidden from each other, or if their randomly chosen backoff values are close enough that the second station starts before it can hear the first.

|Property|CSMA/CD (Ethernet)|CSMA/CA (WiFi)|
|---|---|---|
|**Collision detection**|Yes — sender listens while sending|No — sender can't reliably hear over its own transmission|
|**Response to sensed-busy channel**|Transmits immediately once idle is sensed|Enters random backoff _before_ attempting, even when channel is currently idle|
|**Cost of a collision**|Low — both sides detect and abort mid-frame|High — colliding frames are sent to completion|
|**Acknowledgment**|Not required at this layer|Explicit link-layer ACK required|

> **Analogy — A Radio Walkie-Talkie vs. a Telephone Call:** A telephone lets you talk and listen at the same time, so you'd immediately notice if someone spoke over you — that's CSMA/CD. A walkie-talkie, by contrast, is half-duplex: pressing "talk" deafens you to anything else on the channel, so you have no way of knowing mid-sentence whether someone else stepped on your transmission — you only find out once you release the button and get no reply. That's precisely CSMA/CA's predicament, and exactly why it insists on a deliberate pause-and-listen backoff _before_ ever pressing "talk," rather than detecting trouble after the fact.

### Dealing with Hidden Terminals: RTS and CTS

CSMA/CA offers an optional **reservation scheme** to further mitigate hidden terminals: the **Request to Send (RTS)** and **Clear to Send (CTS)** control frames. When a sender wants to transmit a DATA frame to an access point (AP), it first sends a short RTS to the AP, stating the total time needed for the DATA and its ACK. The AP replies by **broadcasting** a CTS — heard by every station within the AP's own range, including stations hidden from the original sender — which grants the sender permission to transmit _and_ instructs all other stations to defer for the reserved duration.

```
Fig 7.21 -- Collision Avoidance Using RTS/CTS
────────────────────────────────────────
 Source    Destination    All other nodes
  │ DIFS        │                │
  ├────RTS─────>│                │
  │         SIFS│                │
  │<────CTS─────┼──────CTS──────>│ (defer)
  │SIFS          │                │
  ├────DATA─────>│                │
  │         SIFS│                │
  │<────ACK─────┼──────ACK──────>│
```

![[Pasted image 20260724150600.png]]

RTS/CTS improves performance two ways: the hidden-station problem is mitigated, since the full DATA frame is only sent after the channel is reserved; and because RTS/CTS frames are short, a collision **involving** an RTS or CTS wastes only that short frame's duration rather than an entire long DATA frame. (An RTS/CTS exchange itself isn't foolproof — the CTS or RTS can still be lost to fading or noise.) Because the exchange adds delay and consumes channel resources, it's typically only used — if at all — to reserve the channel ahead of a **long** DATA frame.

> **Analogy — Reserving a Conference Room:** Rather than simply walking up to a shared meeting room and hoping it's free (plain CSMA/CA), RTS/CTS is like phoning the receptionist (the AP) to request the room for the next twenty minutes. The receptionist's confirmation is broadcast over the building intercom — heard even by people down a hallway who couldn't have overheard your original phone call — so _everyone_, not just people near you, knows to stay out until your meeting ends.

---

## 7.3.2 WiFi: The 802.11 Wireless LAN

### Overview of 802.11 Wireless LANs

The IEEE 802.11 standards committee has, since 2000, branded successive standards as "WiFi," each corresponding to a numbered generation loosely paralleling cellular "G" numbering.

**Table 7.2 — Evolution of Major 802.11 Standards**

|802.11 Standard|WiFi Gen|Year|Max Data Rate (theoretical)|Frequency (GHz)|Bandwidth|PHY|
|---|---|---|---|---|---|---|
|802.11b|—|1999|11 Mbps|2.4|22|Direct-sequence spread spectrum|
|802.11g|—|2003|54 Mbps|2.4|20|OFDM|
|802.11n|WiFi 4|2009|600 Mbps|2.4, 5|20, 40|OFDM|
|802.11ac|WiFi 5|2013|6.9 Gbps|5|20, 40, 80, 160|OFDM, MIMO|
|802.11ax|WiFi 6|2020|9.5 Gbps|2.4, 5, 6|20, 40, 80, 160|OFDM, OFDMA, MIMO|
|802.11be|WiFi 7|2024|30+ Gbps|2.4, 5, 6|20, 40, 80, 160, 320|OFDM, OFDMA, MIMO|

Three trends stand out beyond the relentless increase in theoretical throughput:

- **Frequency:** WiFi has expanded from the crowded 2.4 GHz band, to 5 GHz, and now 6 GHz — all unlicensed. (802.11p, used for vehicle-to-vehicle communication, is a notable exception operating in licensed spectrum.)
- **Bandwidth:** the minimum channel width is 20 MHz; wider channels are achieved by **bonding** adjacent 20 MHz channels together (Figure 7.23).
- **The physical layer:** WiFi's PHY has evolved from spread-spectrum, through OFDM, to OFDMA — the same modulation family used in 4G/5G for dense-deployment scenarios. Because an AP must simultaneously serve legacy 802.11g devices alongside modern WiFi-6 devices, a single AP must support OFDM, OFDMA, _and_ CSMA/CA all at the same time (resolved in "Putting it all together," below).

The fundamental building block of an 802.11 network is the **basic service set (BSS)**: two or more wireless nodes, typically including a central base station called an **access point (AP)**.

```
Fig 7.22 -- Elements of an 802.11 WLAN
────────────────────────────────────────
   BSS 1                   BSS 2
 (::)Station             (::)   (::)
        \    Switch/         (::)
   (::)AP ─── Router ──── AP(::)
              │
           Internet
```

![[Pasted image 20260724150800.png]]

Each AP connects, in turn, to an interconnection device (switch or router), which leads to the Internet. Although in casual 802.11 parlance "AP" is often used loosely to mean the whole combined box (AP + switch + router, common in home deployments), the 802.11 standard itself defines the AP as a strictly **layer-2-only** device: wireless stations transmit and receive only to/from the AP, and cannot communicate device-to-device directly (in this infrastructure-mode configuration).

### WiFi Channel Structures

Figure 7.23 shows how the 2.4 GHz and 5 GHz bands are divided into numbered, 20-MHz-wide channels. In the 2.4 GHz band, adjacent channels **overlap**; two channels are non-overlapping only if separated by four or more channel numbers, making **{1, 6, 11}** the only set of three simultaneously non-overlapping channels — a real practical constraint when configuring neighboring home WiFi networks. The 5 GHz band offers a much larger, cleaner set of non-overlapping options, and its higher channel-bandwidth tiers (40/80/160 MHz) are formed by **bonding** together multiple adjacent 20 MHz channels.

```
Fig 7.23a -- 2.4 GHz WiFi Channels (USA)
────────────────────────────────────────
 2.40 ─ch1─ch2─ch3...ch6...ch9─ch11─ 2.48
      |<--20MHz-->|          GHz
 non-overlapping set = {1, 6, 11}
```

![[Pasted image 20260724151000.png]]

```
Fig 7.23b -- 5 GHz WiFi Channel Bonding
────────────────────────────────────────
  20MHz:  |36|40|44|48|...|161|165|
  40MHz:  |  38  |  46  |...| 159  |
  80MHz:  |     42     |...| 155  |
 160MHz:  |          50          |
  (wider bonded channels absorb several
   narrower channel numbers underneath)
```

![[Pasted image 20260724151200.png]]

An individual WiFi network operates on only **one** of these channels (potentially dynamically bonding secondary channels to its primary channel for extra bandwidth) — never several simultaneously.

> **Analogy — Radio Station Presets vs. a Combined Broadcast:** Picking a fixed WiFi channel is like tuning to a single radio station preset — simple, but you're capped at that station's bandwidth. Channel bonding is like a station temporarily borrowing the frequency slots of its immediate neighbors to broadcast in richer, wider-band stereo — more capacity, but only if those neighboring slots happen to be free to borrow.

### Putting It All Together: CSMA/CA, OFDM, and OFDMA in WiFi 6

An AP can operate using just one fixed-bandwidth channel (plain OFDM), with all devices in the BSS sharing that single channel. WiFi 6 (and onward) additionally supports **OFDMA**, dividing a channel into subchannels usable by multiple devices _simultaneously_ — genuinely useful in dense deployments.

The mechanism that lets OFDM-only, OFDMA-capable, and plain CSMA/CA-only devices all **coexist** on the same AP is a clever extension of RTS/CTS: the **multi-user RTS (MU-RTS)**. In the downlink, the AP sends an MU-RTS specifying a list of RU allocations (which RUs go to which devices) and the reserved duration. An OFDMA-capable device responds with a CTS transmitted only in its assigned frequencies; an OFDM-only (legacy) device simply responds to the RTS with an ordinary CTS, as in Figure 7.20. Once the MU-RTS/CTS exchange succeeds, the AP has reserved the channel and can schedule RUs to the multiple devices using OFDMA. In the uplink direction, device transmissions to the AP are likewise coordinated via MU-RTS/CTS.

```
Fig 7.24 -- Intervals of CSMA/CA, OFDM, OFDMA
────────────────────────────────────────
 t0   t1        t2            t3      t4
 │dev_a│ dev_b │ MU-RTS/CTS │dev_5  │OFDMA│
 [OFDM] [OFDM] [  reserve  ][ OFDM ][ RUs:│
                [  via CSMA/CA ]    [1,2,3]
  (legacy devices use plain OFDM one at a
   time; AP uses MU-RTS to reserve, then
   several devices share via OFDMA RUs)
```

![[Pasted image 20260724151400.png]]

In this way, WiFi 6's three sharing mechanisms — CSMA/CA (to gain initial channel access), OFDM (single-device full-channel use), and OFDMA (multi-device simultaneous sub-channel use) — interoperate on one AP serving a heterogeneous mix of old and new devices.

### 802.11 MAC: The WiFi Wireless Link Layer

The IEEE 802.11 **medium access control (MAC)** layer sits directly above the physical layer and below the network layer on every wireless device, and has both **data-plane** and **control-plane** components. The data plane transports link-layer frames end-to-end between application-facing protocol layers; the control plane implements functions such as RTS/CTS, ACKs/NAKs, beaconing (Section 7.3.4), and power control (Section 7.3.6). Because control-plane modules on the device and the AP exchange control messages as link-layer frames, a logical control connection is often drawn (with dashed lines) between the two sides. Both data and control frames are scheduled for transmission by the **MAC scheduler** (Section 7.3.5).

```
Fig 7.25 -- WiFi: The Wireless Link Layer
────────────────────────────────────────
 Device                          AP  Router
 Application     Data  Control        
 Transport       plane plane          
 Network                              
 802.11 MAC <== control ==> 802.11│802.3
 802.11 PHY                  MAC │ MAC
                              PHY│ PHY
        (data plane frames flow through)
```

![[Pasted image 20260724151600.png]]

### 802.11 Frame Format

The 802.11 frame (Figure 7.26) shares broad similarities with an Ethernet frame, but adds several wireless-specific fields.

```
Fig 7.26 -- 802.11 Frame Format (bytes)
────────────────────────────────────────
|FrameCtl|Dur/ID|Addr1|Addr2|Addr3|
|   2    |  2   |  6  |  6  |  6  |
|SeqCtl|Addr4|Payload  |CRC|
|  2   |  6  | 0-2312  | 4 |

Frame Control subfields (bits):
|Ver|Type|Subtyp|ToAP|FromAP|MoreFrag|
| 2 | 2  |  4   | 1  |  1   |   1    |
|Retry|PwrMgt|MoreData|Protect|+HTC/Ord|
|  1  |  1   |   1    |   1   |   1    |
```

![[Pasted image 20260724151800.png]]

**Payload and CRC fields.** At the heart of the frame is the payload — typically an IP datagram, a MAC control frame, or an ARP packet — permitted up to 2,312 bytes, though in practice usually well under 1,500. As with Ethernet, a 32-bit **CRC** lets the receiver detect bit errors; given that bit errors are considerably more common on wireless links than wired ones (per 7.2.1), the CRC is even more indispensable here.

**Address fields.** The most striking structural difference from Ethernet is that an 802.11 frame carries **four** MAC address fields rather than two — needed for internetworking purposes, specifically to move a network-layer datagram from a wireless station, through an AP, to a router interface (in infrastructure-mode networks; the fourth address is reserved for ad hoc, AP-less networks and isn't considered further here):

- **Address 2** is the MAC address of whichever device — station or AP — is _transmitting_ the frame.
- **Address 1** is the MAC address of whichever device is meant to _receive_ the frame (the destination AP, if a station is sending; the destination station, if the AP is sending).
- **Address 3** contains the MAC address of the **router interface** that connects the BSS's subnet onward to other subnets — a field the AP relies on because the AP itself is a purely layer-2 device with no notion of IP addressing.

```
Fig 7.27 -- Use of Address Fields: H1 <-> R1
────────────────────────────────────────
        Router
        R1 │
   ┌────────┴────────┐    Internet
  AP                AP
 (::)H1 (::)      (::)  (::)
   BSS 1              BSS 2
```

![[Pasted image 20260724152000.png]]

Walking through the example above: the router at R1, unaware any AP even exists, uses **ARP** to learn H1's MAC address exactly as on any Ethernet LAN, then sends an _Ethernet_ frame with source R1 and destination H1. When this Ethernet frame reaches the AP, the AP converts it to an 802.11 frame — filling **Address 1** with H1's MAC (the receiving station) and **Address 2** with the AP's own MAC (the transmitter), while **Address 3** carries R1's MAC, letting H1 later determine which router interface originally sent the datagram into the subnet.

Conversely, when H1 replies, it creates an 802.11 frame with **Address 1** = AP's MAC, **Address 2** = H1's own MAC, and **Address 3** = R1's MAC. When the AP receives this frame, it converts it back into an Ethernet frame with source address = H1's MAC and destination address = R1's MAC — reading R1's MAC straight out of Address 3. In short, Address 3 is what lets the AP correctly bridge a BSS onto the wired LAN.

**Sequence number, duration, and frame control fields.** Just as sequence numbers let a transport-layer receiver distinguish a genuinely new segment from a retransmission (rdt2.1, Chapter 3), the 802.11 **sequence number** field serves the identical purpose here at the link layer, given that ACKs (and hence data frames) can be lost. The **duration** field lets a transmitting station reserve the channel for a period covering both the DATA transmission and its ACK — used in both DATA frames and RTS/CTS control frames. Within the **frame control** field, the type/subtype subfields distinguish control frames (RTS, CTS, ACK) from data and management frames, and the **power management** bit signals to the AP that a device is about to go to sleep (Section 7.3.6).

|Address Field|Meaning|
|---|---|
|**Address 1**|MAC of the intended _receiving_ wireless device (station or AP)|
|**Address 2**|MAC of the _transmitting_ wireless device (station or AP)|
|**Address 3**|MAC of the router interface connecting the BSS's subnet onward|
|**Address 4**|Used only in AP-less ad hoc mode (device-to-device forwarding)|

> **Analogy — A Mail Forwarding Slip Stapled to Every Envelope:** Because an AP is a purely link-layer forwarder with no concept of IP, it needs each 802.11 envelope (frame) to carry more than the usual "from/to" — it needs a third label, permanently stapled on, naming the _original wired-side mail carrier_ (the router interface) that either sent this piece of mail into the wireless neighborhood or should eventually collect it. Address 3 is that stapled forwarding slip, letting the AP correctly relabel each envelope as it crosses between the wireless and wired worlds.

---

## 7.3.3 The 5G Radio Access Network

### Architectural Components of 5G Networks: The Big Picture

A cellular network — 3G, 4G, or 5G — has two principal components: the **Radio Access Network (RAN)**, at the network's edge, and the **Core**. The RAN comprises (i) end-user wireless devices, called **user equipment (UE)** in cellular parlance; (ii) the local 5G wireless channel, called **New Radio (NR)**, to distinguish it from the "old" 4G LTE radio channel; and (iii) a base station, called a **Next Generation Node (gNB)**. This document favors the more intuitive terms "base station," "user device," and "wireless channel" over "gNB," "UE," and "NR" respectively. The RAN is the cellular counterpart of a WiFi LAN's first-hop wireless link, though a 5G base station is considerably more multi-functional and sophisticated than a WiFi AP.

The **5G Core** consists of the links, routers, and servers connecting a RAN to the broader Internet (and other provider networks), using standard wired Internet protocols and technology. Just as with the layer separation from Chapters 4–5, the 5G Core separates a **user plane** (data) from a **control plane** — so foundational to 5G architecture that it has its own acronym, **CUPS** (Control Plane and User Plane Separation). The Core's set of control and management functions are collectively the **Core Network Functions**, providing RAN access/authorization and mobility support (Figure 7.28): AMF (Access, Mobility Management), AUSF (Authentication Server), SMF (Session Management), NSSF (Network Slice Selection), NRF (Network Repository), AF (Application Function), UPF (User Plane), PCF (Policy Control), UDM (Unified Data Management), NSSAAF (NSS Authorization), and NEF (Network Exposure). Where these Core services are physically implemented — co-located at the base station for a small private 5G network, in an edge-cloud near the RAN, or in a large distant data center — varies by deployment; the only firm requirement is IP reachability. Application code may even run at the base station or in the RAN itself, so a user's client-server traffic never leaves the cellular network — a pattern called **local breakout**.

```
Fig 7.28 -- Major Elements of a 5G Network
────────────────────────────────────────
|AMF|AUSF|SMF |NSSF|NRF|AF|   Core Network
|UPF|PCF |UDM |NSSAAF|NEF|   Functions
          │
 (::)     ▼
 (::)──[Base Station]──[router]──Internet
 (::)
 |<---- RAN ---->|<---- 5G Core ---->|
```

![[Pasted image 20260724152200.png]]

### 4G, 5G RAN: The Physical Radio Channel

4G and 5G RANs implement OFDMA channel sharing, using exactly the same RE / RB building blocks introduced generally in Section 7.3.1. In 4G, the smallest subchannel bandwidth is 15 kHz, with a 66.6 μsec transmission time per minislot. 5G defines several larger subchannel bandwidths (30, 60, 120, 240, 480, and 960 kHz) with proportionally _shorter_ minislot durations — trading subchannel width for finer time granularity.

```
Fig 7.29 -- 4G Resource Blocks (RBs)
────────────────────────────────────────
 Freq ▲ [][][][][]...[][]   50 RBs, each
      │ [][][][][]...[][]   180 kHz wide
      │ [][][][][]...[][]
      └────────────────────▶ Time
          |<---- 1 ms ---->|
  1 RB = 84 REs (12 subcarriers x 7
          consecutive 66.6us minislots)
```

![[Pasted image 20260724152400.png]]

A 10 MHz-wide 4G channel, for instance, consists of 50 time-parallel RBs, each occupying one of 50 different 180 kHz-wide frequency subchannels. 5G defines several additional ways to bundle REs into an RB, and incorporates all the earlier 4G channel-bandwidth definitions (1.4, 3, 5, 10, 15, 20 MHz) while adding wider channels up to 100 MHz.

There is a still higher level of bandwidth aggregation: a **band** is an aggregation of one or more **channels**, with each channel itself consisting of a number of different subcarriers/subchannels (Figure 7.30). National governments allocate licensed spectrum within these bands to carriers, typically via auction (recall 7.2.1); once allocated a band, an operator decides how to divide the band into channels, and the channels into subcarriers.

```
Fig 7.30 -- Bands, Channels, and Subcarriers
────────────────────────────────────────
 One Band
  ├─ Channel 1 ── [subcarrier][subcarrier]
  ├─  ...
  └─ Channel N ── [subcarrier][subcarrier]
```

![[Pasted image 20260724152600.png]]

### 4G, 5G RAN Radio: Physical and Logical Control and Data Channels

Each RB also has a **direction**: **downlink** (base station → user device) or **uplink** (user device → base station). There is typically far more downstream than upstream traffic — roughly a 10:1 ratio — and a network operator sets this ratio by determining what fraction of RBs go to each direction. Two schemes are used:

- **Time Division Duplexing (TDD):** downlink and uplink share the _same_ frequency, with time slots assigned to each direction.
- **Frequency Division Duplexing (FDD):** distinct channel frequencies are reserved exclusively for uplink versus downlink.

RBs in the downlink direction are further partitioned into three **downlink physical channels**, and RBs in the uplink into three **uplink physical channels**:

|Direction|Channel|Purpose|
|---|---|---|
|Downlink|**PDSCH** (Physical Downlink Shared Channel)|Carries all downstream user-plane data plus base-station-to-device control-plane messages|
|Downlink|**PDCCH** (Physical Downlink Control Channel)|Informs devices which RBs will carry their downlink data, and which RBs they may use for uplink transmission|
|Downlink|**PBCH** (Physical Broadcast Channel)|Carries the information a device needs to _discover_ and join the cellular network (Section 7.3.4)|
|Uplink|**PUSCH** (Physical Uplink Shared Channel)|The upstream equivalent of PDSCH — upstream user-plane data plus control-plane messages to the base station|
|Uplink|**PRACH** (Physical Random-Access Channel)|Used by a device to request an initial connection to the base station, accessed via a random-access scheme|
|Uplink|**PUCCH** (Physical Uplink Control Channel)|Requests future PUSCH RB allocation, and carries measurement data and link-layer ACKs/NAKs|

> **Analogy — An Airport's Departure and Arrival Concourses:** Splitting RBs into six purpose-built physical channels is like an airport separating departures from arrivals, and further separating a general boarding concourse (PDSCH/PUSCH — bulk passenger traffic) from an information-announcement channel (PDCCH — tells you your gate), a public-address welcome broadcast anyone can tune into on arrival (PBCH — helps newcomers get oriented), and a dedicated request desk where a new arrival first checks in before being assigned a gate at all (PRACH) or requests future boarding slots (PUCCH). Each channel is physically the same kind of "space" (an RB) but is functionally dedicated to a distinct kind of traffic.

There are also multiple **logical channels** defined in the uplink and downlink directions, which are then mapped onto the physical uplink and downlink channels described above; all information flowing on a given logical channel is semantically related. Given these two layers of definitions — physical channels, logical channels, and the earlier frequency-domain sense of "channel" from Section 7.2 and radio-channel bands from Figure 7.30 — it's fair to say **"channel"** is one of the most overused words in all of wireless networking, with a genuinely different meaning depending on context.

### 5G RAN: The Wireless Link Layer

Having studied WiFi's link-layer protocol stack, we turn to the considerably richer link-layer stack of the 5G RAN (Figure 7.31). As with WiFi, link layers in both the device and the base station have a **user plane** and a **control plane**. The user plane's job is to move link-layer frames between the device's applications and the base station (and _vice versa_). The control plane's job is to configure and control the RAN's radio resources, provide control functions for user-plane frame transfer (e.g., ARQ and quality of service), and support additional services such as device mobility (Section 7.4.2), network attachment (Section 7.3.4), channel-quality measurement and reporting (Section 7.3.5), and power control (Section 7.3.6). Because control-plane actions typically involve message exchange between the device and the base station, a logical connection (dashed line) is shown between control modules in the sublayers on both sides of the wireless link; control messages are themselves carried in link-layer frames over the appropriate 5G physical control channels.

```
Fig 7.31 -- 5G RAN Protocol Stacks
────────────────────────────────────────
  Device                 Base Station
 Net|SDAP|RRC        RRC|SDAP|Net
    |PDCP| :            : |PDCP|
    |RLC | :            : |RLC |
    |MAC | :            : |MAC |
    |PHY | :            : |PHY |
   (::) <====RAN link====> (tower)
  (dashed ctrl lines link RRC to each
   sublayer's control-plane component)
```

![[Pasted image 20260724153200.png]]

The 5G RAN link layer is divided into a number of sublayers:

- **RRC (Radio Resource Control):** the control-plane function responsible for (i) establishing, maintaining, and releasing link-layer logical channels; (ii) managing the per-device link-layer connection state (Sections 7.3.4 and 7.3.6); (iii) managing aspects of device mobility (Section 7.4); and (iv) performing channel-quality measurement and reporting (Section 7.3.5). RRC accomplishes this work by configuring local link- and physical-layer state, using the RRC protocol to send control messages between the base station and a device.
- **SDAP (Service Data Adaptation Protocol):** the highest sublayer in the link-layer _user_ plane. When quality-of-service guarantees are to be provided, SDAP assigns QoS Flow IDs and maps the flows onto underlying radio resources.
- **PDCP (Packet Data Convergence Protocol):** responsible for IP header compression and, optionally, encryption and cryptographic message integrity checks.
- **RLC (Radio Link Control):** responsible for segmentation/reassembly and reliable frame transfer, using a simple ARQ (Automatic Repeat reQuest) protocol with sequence numbers, cyclic redundancy checks, and ACK/NACK messages — the same core rdt machinery studied in Chapter 3.
- **MAC (Medium Access Control):** responsible for scheduling the transmission of link-layer user- and control-plane frames between the base station and user devices. Data in these frames is segmented into smaller chunks that are then mapped onto OFDMA Resource Blocks for transmission (Section 7.3.5).

|Sublayer|Plane|Primary Job|
|---|---|---|
|**RRC**|Control|Establish/maintain/release logical channels; connection state; mobility; channel-quality reporting|
|**SDAP**|User (top)|QoS Flow ID assignment, mapping flows to radio resources|
|**PDCP**|Both|IP header compression; optional encryption and integrity checks|
|**RLC**|Both|Segmentation/reassembly; reliable transfer via ARQ (sequence numbers, CRC, ACK/NACK)|
|**MAC**|Both|Scheduling frames onto OFDMA Resource Blocks|
|**Physical**|Both|Radio transmission (Sections 7.2, 7.3.1, 7.3.3)|

While the WiFi link-layer frame (Figure 7.26) is relatively simple, a 5G RAN link-layer frame is considerably more complex, owing to these multiple sublayers. A 5G frame carries an IP datagram or an RRC control-plane message, with SDAP, PDCP, RLC, and MAC header fields all layered on top — each sublayer contributing its own user/control message bit, sequence number or identifier, payload field, and other bookkeeping needed to accomplish that sublayer's function.

```
Fig 7.32 -- 5G Base Station RAN Pipeline
────────────────────────────────────────
 Control   RRC
 plane      │ control messages (to all
            │ sublayers below, dashed)
            ▼
 User    SDAP─►PDCP─►RLC─►MAC─►PHY──)))
 plane  (datagrams to/from device)
                          Device <──►
                       Base station
```

![[Pasted image 20260724153400.png]]

> **Analogy — A Cargo Ship's Loading Chain vs. a Single Dockworker:** WiFi's simple frame format is like one dockworker who personally checks, labels, and loads every crate onto a ship. The 5G RAN stack is like an entire loading chain — a QoS clerk sorting crates by priority (SDAP), a compression crew shrink-wrapping and sealing them (PDCP), a reliability inspector who re-sends anything damaged in transit (RLC), and a scheduling foreman deciding which crane loads which crate onto which cargo slot and when (MAC) — with a dockmaster (RRC) issuing standing instructions to every stage of the chain at once.

### Disaggregating the 5G Base Station: RU, DU, and CU

In earlier 2G, 3G, and 4G networks, a RAN's cellular base station was implemented as a **monolithic** unit — essentially a specialized, single-vendor "box" implementing that generation's standardized functions end to end. Beginning with 5G, there has been a deliberate move to **disaggregate** RAN functions into multiple functional units that interoperate through standardized interfaces. This disaggregated architecture allows multiple vendors to offer competing implementations of individual pieces of the base station, lowering the barrier to entry and increasing competition and innovation — a motivation genuinely similar to the one behind SDN's development (Chapter 5).

```
Fig 7.33 -- Base Station Split: CU | DU | RU
────────────────────────────────────────
 Central Unit | Distributed | Radio
    (CU)      |  Unit (DU)  | Unit (RU)
 Ctrl: RRC    |             |
 User: SDAP,  | RLC, MAC,   | D/A conv,
       PDCP   | (part) PHY  | RF front end
 |<-- near-real-time -->|<-- real-time -->|
```

![[Pasted image 20260724153600.png]]

The base station's functions are split into three units: the **Central Unit (CU)** hosts RRC (control plane) and SDAP/PDCP (user plane); the **Distributed Unit (DU)** hosts RLC, MAC, and part of the physical layer; and the **Radio Unit (RU)** hosts the remaining physical-layer functions closest to the antenna (D/A conversion, RF front end). _Where_ each functional unit is actually situated — co-located at the base station, in an edge cloud, or in a central cloud — is an implementation decision, typically driven by communication delays and real-time requirements: functions closer to the RU generally demand tighter, more real-time coordination than functions closer to the CU. The RU/DU/CU split is part of the 3GPP 5G specification itself; the O-RAN Alliance takes this disaggregation one step further by specifying additional interfaces among these units.

|Unit|Hosts|Timing Requirement|
|---|---|---|
|**CU (Central Unit)**|RRC, SDAP, PDCP|Near-real-time — most tolerant of distance/delay|
|**DU (Distributed Unit)**|RLC, MAC, part of PHY|Tighter real-time coordination|
|**RU (Radio Unit)**|Remaining PHY (D/A conversion, RF front end)|Strictest real-time — physically nearest the antenna|

> **Analogy — A Restaurant's Back Office, Kitchen, and Grill Station:** A monolithic base station is like a single chef doing everything — planning the menu, cooking, and plating — in one cramped stall. Disaggregation is like splitting that stall into a back office that plans the menu and handles reservations at its own relaxed pace (CU), a kitchen that must prepare dishes on a tighter clock (DU), and a grill station right at the pass that has to sear each item in the split-second window before it's plated (RU) — three specialized roles, each with its own timing pressure, that different vendors could staff independently as long as they hand work off through agreed protocols.

### Implementing the 5G RAN Control Plane: SD-RAN

The RU/DU/CU split is only _one_ of two disaggregations found in 5G. A second arises within the control plane itself. Because the RRC control plane is architecturally separate from the user plane — in much the same way the Internet's control and data planes are separate — it becomes possible to implement the standardized RRC functions in a genuinely modern, SDN-like manner.

```
Fig 7.34 -- SD-RAN Control Plane
────────────────────────────────────────
 Control-plane proxy <──► RIC (xApps:
   (interface to           load balancing,
    5G Core's                handover, QoS)
    control plane)              │
        │  control msgs      R-NIB
        ▼
 SDAP─►PDCP─►RLC─►MAC─►PHY──))) tower
        (data plane, unchanged)
```

![[Pasted image 20260724153800.png]]

An **SD-RAN** controller re-factors the monolithic RRC into: a **control-plane proxy**, implementing a 3GPP-compliant, well-defined service-based interface to each of the 5G Core's Core Network Functions; and a **Real-Time Intelligent Controller (RIC)**, serving a role directly analogous to an SDN controller. The RIC itself has three major components — a lower-layer communication component that issues commands to the data-plane RAN components; a RAN-wide state-management layer holding up-to-date channel state, connection state, device identity, and QoS requirements in a local **RAN Node Information Base (R-NIB)**; and an interface to a set of application-layer control processes.

Those RIC control processes, called **xApps** in OpenRAN parlance, make the actual RAN control decisions — load balancing, device handover, QoS provisioning, and more — by reading and writing information in the R-NIB, and are implemented as separate software processes executing _outside_ the RIC itself. **OpenRAN** is one of the most prominent real-world examples of an SD-RAN, "open" in the sense that well-defined, standardized interfaces between its components let multiple vendors' implementations interoperate.

Crucially, as long as a re-factored SD-RAN implementation (Figure 7.34) implements all the same functional interfaces to the 5G Core and the user plane that the monolithic RRC implementation does (Figure 7.33), the two are **indistinguishable** as far as the rest of the 5G network is concerned — precisely the same substitutability principle that made SDN's separation of control and data planes so powerful in Chapter 5.

|Approach|RRC Implementation|Extensibility|
|---|---|---|
|**Monolithic (2G–4G style)**|Single-vendor, tightly integrated "box"|Low — vendor-locked, slow to add new control logic|
|**SD-RAN / OpenRAN (5G)**|Control-plane proxy + RIC, with pluggable xApps|High — new control logic added as independent xApps, multi-vendor interfaces|

> **Analogy — A Fixed Thermostat vs. a Smart-Home Hub:** A monolithic RRC is like an old mechanical thermostat: one manufacturer, one fixed set of behaviors, take it or leave it. An SD-RAN is like a smart-home hub that exposes a standard interface (the control-plane proxy) to any app (xApp) a homeowner wants to install — a scheduling app, an energy-saving app, a security app — all reading and writing shared state (the R-NIB) without needing to know anything about each other or about the underlying wiring.

---

## 7.3.4 Discovery: Attaching to a Wireless Network

Having discussed both cellular RANs and WiFi LANs in detail, we now turn to a capability needed in **any** edge network, whether a wired Ethernet LAN, a 5G RAN, or a WiFi WLAN — the ability for a device to detect the existence of network infrastructure and "attach" to that network. This process is known as **network association**.

### Beaconing

A base station, whether in a 5G network or a WiFi network, periodically transmits **beacon messages** that announce the existence of the access network and information about its characteristics — such as its name, its transmission rates, capabilities, and physical-layer parameters about its radio channels. A device wishing to attach to a network passively scans the various possible wireless channels, listening for such beacon messages.

Wireless network standards don't specify the algorithm a device uses to select _which_ of the advertised networks to associate with — that choice is left up to the device (or user) and to the designers of its firmware and software. Typically, a device may choose to associate with the network it was most recently associated with, choose from an ordered list, or join whichever advertised network has the strongest signal. In public 5G networks, a device will almost always preferentially associate with the 5G network operated by the provider that issued the device's SIM card and with which the device has a paid service plan.

### Attaching to a WiFi Network

After scanning for available networks and selecting an AP to associate with, a WiFi device sends an 802.11 **association request** frame to that AP; the AP responds with an **association response** frame.

```
Fig 7.35 -- Associating with a WiFi Network
────────────────────────────────────────
  BSS 1                       BSS 2
 (())AP1   1:beacon    AP2(())  1:beacon
        \                    /
         \ 2:assoc request  /
          \      H1        /
           \  3:assoc     /
            \  response  /
     (H1 picks an AP, sends the request,
      receives the response)
```

![[Pasted image 20260724154000.png]]

The device will also authenticate itself to the network using **WPA (WiFi Protected Access)** security after receiving a beacon, either before or after association. Once associated with a WiFi AP, the device will want to join the subnet (in the IP-addressing sense of Section 4.3.3) to which the AP belongs, so it typically sends a DHCP discovery message into the subnet via the AP to obtain an IP address. Once that address is obtained, the rest of the world simply views the device as another ordinary device with an IP address in that subnet.

### Attaching to a 5G Network

The process by which a device attaches to a 5G network to obtain its own collision-free control and data channels is considerably more involved than attaching to a WiFi network, since a 5G device can't rely on a single beacon message containing everything it needs. Instead, it goes through a **multistep** process (Figure 7.36):

1. Each base station broadcasts multiple pieces of information a device needs to decide whether to join. A network's **primary synchronization signal (PSS)** is first located and used by the device to synchronize its clock with the base station's; the **secondary synchronization signal (SSS)**, providing an initial network identifier, is located next. Both PSS and SSS are broadcast every 5 msec, on the center 127 subcarriers of every 5G channel, by every 5G base station. The device then obtains the **Master Information Block (MIB)** for that channel, which includes information about subcarrier spacing and whether the cell is even open for association. PSS, SSS, and MIB together form the **System Signal Block (SSB)**. With this information, the device can locate and receive up to 20 **System Information Blocks (SIBs)** that characterize the 5G network and its operation. The most important of these is **SIB1**, transmitted every 80 msec, which contains the mobile country code (MCC) and mobile network code (MNC) for the carrier operating this channel, the base station identifier, and the minimum acceptable reception level (channel quality).
2. Suppose the device decides it wants to associate with this network. The base station, however, isn't yet even aware of the device's existence, and the device hasn't yet been allocated any uplink RBs it could use to communicate. The device therefore uses the cell's upstream random-access channel (PRACH) to send a **Radio Resource Control (RRC) connection setup request** to the base station — similar in spirit to the 802.11 association request frame.
3. The base station responds to the device's connection setup request with an **RRC connection setup response** message — similar in spirit to the 802.11 association response frame.

```
Fig 7.36 -- Associating with a 5G Network
────────────────────────────────────────
 Network 1                  Network 2
 BaseSta.1  1:SSB,SIBs   BaseSta.2(())
   (())  ───broadcast───►    │
                   H1 ◄──────┘
             2:RRC conn setup req───►
             3:RRC conn setup resp◄──
```

![[Pasted image 20260724154200.png]]

At this point, the user device and base station can exchange control messages with each other in the RAN — but the device _still_ hasn't actually joined the 5G network, since it hasn't yet identified or authenticated itself, nor does it have an IP address. Completing that process requires the 5G Core (Section 7.4) and authentication mechanisms (Section 8.8), covered later.

|Aspect|WiFi Attachment|5G Attachment|
|---|---|---|
|**Discovery mechanism**|Single beacon frame, scanned passively|Multistep: PSS → SSS → MIB (SSB), then up to 20 SIBs|
|**Join request**|802.11 association request|RRC connection setup request, sent via PRACH|
|**Join confirmation**|802.11 association response|RRC connection setup response|
|**Security**|WPA authentication, before/after association|Deferred to later Core-network authentication (Section 8.8)|
|**IP address**|Obtained via DHCP after association|Obtained later, via the 5G Core (Section 7.4)|

> **Analogy — Walking Into a Café vs. Checking Into a Secure Building:** Joining a WiFi network is like walking into a café, glancing at a single posted sign for the WiFi name, and asking the barista for the password — one quick exchange and you're online. Joining a 5G network is like checking into a high-security building: first you read a lobby directory just to learn the building's name and whether it's even accepting visitors today (PSS/SSS/MIB), then you review a whole binder of building policies (the SIBs), and only _then_ do you formally request a badge at the front desk (RRC connection setup) — and even with that badge in hand, you still haven't been individually verified or issued an office key (Core-network authentication and an IP address) yet.

---

## 7.3.5 Scheduling Transmissions over the RAN/WLAN

One of the most important functions of the MAC layer is to schedule the **order** in which data is transmitted in the downlink direction from a base station to user devices, and in the uplink reverse direction. This is the role of the **MAC scheduler**, particularly important in dense deployment scenarios — 4G/5G networks and WiFi-6/7 OFDMA deployments — where there may be tens or hundreds of users with data to transmit at a given time. The discussion below focuses on the common case of a central MAC scheduler located at the base station. (Distributed scheduling in pure random-access environments is also a topic of interest but isn't covered here.)

Recall the link-layer scheduling studied in a _wired_ setting (Section 4.2.5): overall link-layer performance there — throughput aggregated across all traffic on the link — was essentially **independent** of the specific order in which frames were transmitted, since a fixed number of frames were sent per second regardless of which frames were chosen. In a **wireless** setting, this is decidedly _not_ the case. As Section 7.2 established, a base station can transmit at a higher bit rate (via a higher-order modulation scheme) to a device that is physically close with a clear line-of-sight than to a device that is far away and suffering multipath effects. WLAN/RAN throughput therefore depends on **which** devices are scheduled as transmitters and receivers. MIMO's spatial multiplexing and spatial diversity (Section 7.2.1) offer additional degrees of freedom — sending single or multiple streams to a given receiver — and beamforming offers yet another: sending to different sets of receivers simultaneously. Channel characteristics are, on top of all this, constantly changing over time. Consequently, **overall aggregate wireless link-layer performance depends heavily on MAC scheduling choices** in a way it simply doesn't in a wired network.

Since neither the 3GPP nor the WiFi standards mandate — or even describe — specific MAC scheduling algorithms, the choice of algorithm is entirely in the hands of the network operator. Scheduling is thus part of the "secret sauce" that can distinguish one network provider from another; many wireless hardware and software vendors tout a sophisticated scheduling approach as a genuine competitive differentiator.

Four key considerations shape any MAC scheduling algorithm:

- **Channel quality awareness.** Does the scheduler use estimates or measurements of radio-channel quality between the base station and each device? A device with a high-quality channel can send/receive at higher rates, so a network can achieve higher _aggregate_ throughput by preferentially serving devices with high-quality channels.
- **Fairness among devices.** Does the scheduler provide a degree of fairness (e.g., roughly equal throughput) among devices? If so, what exactly counts as "fair"? A policy that always favors the highest-quality channel can achieve high aggregate throughput but poor fairness; providing fairness generally exacts a "price" in the form of some reduction in maximum achievable aggregate throughput.
- **Prioritization among classes of traffic.** Does the scheduler distinguish control-plane from user-plane traffic (e.g., prioritizing the former, or reserving a fixed minimum share of resources for one or both)? Does it distinguish real-time traffic (control applications, interactive voice/video) from non-real-time traffic (web browsing)? The earlier discussion of priority scheduling (Section 4.2.5) applies directly here.
- **Quality of Service (QoS) awareness.** Does the scheduler make decisions so that quality-of-service performance guarantees are actually provided to devices?

Thousands of research papers, and nearly as many MAC scheduling algorithms, have been proposed over the years. Four representative algorithms are described below (Figure 7.37), all scheduling downstream transmissions from a base station to six associated user devices — though the same considerations apply symmetrically to the base station's scheduling of upstream channel resources. With an OFDMA downstream channel, the scheduler's decision at each scheduling instant determines which device's queued frames get transmitted in which Resource Blocks before the next scheduling instant. In 4G/5G systems, scheduling decisions are made roughly every 1 ms; in WiFi 6, the interval between scheduling decisions can be even shorter.

```
Fig 7.37 -- Four MAC Scheduling Algorithms
────────────────────────────────────────
 a. RR:  UE1..UE6 queues -> Scheduler
         (equal RBs to each, no estimate)
 b. MT:  queues + channel-quality est.
         -> Scheduler (best channel wins)
 c. BET: queues + per-UE avg throughput
         -> Scheduler (lowest throughput
                        device wins)
 d. PF:  queues + channel est. + avg
         throughput -> Scheduler (best
         quality-to-throughput ratio wins)
```

![[Pasted image 20260724154400.png]]

### Round Robin (RR) Scheduling

Recall RR scheduling from the wired setting (Section 4.2.5). In a wireless setting, RR allocates an **equal number of RBs** to each device for the next transmission round. RR is channel-_unaware_ and QoS-_unaware_. It is fair "by turns," since every device is allocated the same number of RBs — but not necessarily throughput-fair, since different devices may experience very different channel conditions and thus realize very different actual throughputs from the same number of RBs.

### Maximum Throughput (MT) Scheduling

MT scheduling assigns RBs to whichever user devices have the **highest measured channel quality** for that specific RB and have data ready to send. Channel quality can be determined in several ways; in 5G networks, a base station periodically broadcasts reference signal symbols at known positions within resource blocks, and each device periodically reports back a 4-bit **Channel Quality Indication (CQI)** based on its measured reception of those reference signals.

At each scheduling instant _t_, the base station computes _d__ᵢ_ₖ(_t_): the expected throughput for device _i_ using RB _k_ at time _t_, derived from device-reported CQIs (which in turn determine the modulation technique to use, which in turn determines how many bits per symbol that RB can carry). The MT scheduler then finds the device _i_ with the highest reported channel quality across all RBs _k_:

```
max_{i,k} { d^i_k(t) }
```

and assigns RB _k_ to device _i_. This process repeats until all RBs are assigned. MT scheduling is channel-aware but genuinely **unfair** — in the most extreme case, a single device with the best channel quality among all devices could be assigned _every_ available RB, provided it has enough queued data to use them.

### Blind Equal Throughput (BET) Scheduling

BET explicitly aims for fairness by providing **equal average throughput** to all user devices. The scheduler maintains a running estimate of each device's average throughput: letting _r_ⁱ(_t_) be device _i_'s measured throughput during time period _t_, its exponentially weighted moving average (EWMA) at time _t_, _R_ⁱ(_t_), is

```
R^i(t) = (1 - β)·r^i(t) + β·R^i(t-1)
```

where β (0 < β < 1) sets the relative weight of past history versus the most recent measurement — the identical EWMA formula encountered when estimating TCP round-trip times (Section 3.5.3). BET simply assigns each available RB to whichever device with queued data currently has the **smallest** estimated average throughput, repeating until all RBs are assigned. BET is explicitly throughput-fair, QoS-_unaware_, and arguably somewhat channel-aware, in that it uses each device's _realized_ throughput — which itself depends on channel quality.

### Proportional Fairness (PF) Scheduling

PF scheduling seeks to strike a balance between maximizing aggregate performance and providing fairness. Using the same _d_ⁱₖ(_t_) and _R_ⁱ(_t_) defined above, at each scheduling time _t_ the PF scheduler finds the device _i_ and RB _k_ that achieves

```
max_{i,k} { d^i_k(t) / R^i(t-1) }
```

and assigns RB _k_ to device _i_. The intuition: since _d_ⁱₖ(_t_) sits in the numerator, higher instantaneous throughput raises the value being maximized, favoring good channels — but since _R_ⁱ(_t_−1) sits in the _denominator_, a device with **low** average throughput also raises the value being maximized, favoring devices that have been historically underserved. This process repeats until all RBs are assigned. PF scheduling is channel-aware and aims to provide a genuine measure of fairness, without being _purely_ fair like BET or _purely_ greedy like MT.

|Algorithm|Channel-Aware?|Fair?|QoS-Aware?|Core Rule|
|---|---|---|---|---|
|**Round Robin (RR)**|No|By turns (equal RBs), not by throughput|No|Cycle through devices equally|
|**Maximum Throughput (MT)**|Yes|No — can be extremely unfair|No|Give each RB to the best-channel device|
|**Blind Equal Throughput (BET)**|Somewhat (via realized throughput)|Yes — equal average throughput|No|Give each RB to the lowest-throughput device|
|**Proportional Fairness (PF)**|Yes|Balanced|No|Maximize channel-quality ÷ average-throughput ratio|

Additional scheduling disciplines exist beyond these four representative algorithms — hard or soft reservation of resources for QoS guarantees, priority scheduling, scheduling tuned for real-time traffic, and virtualized scheduling — but are beyond this section's scope.

> **Analogy — Four Ways to Seat a Restaurant Line:** Round Robin is a host seating parties strictly in the order they arrived, ignoring how quickly each table's kitchen station can actually serve them. Maximum Throughput seats whichever party's order the kitchen can prepare _fastest_ right now, even if that means some parties wait indefinitely. Blind Equal Throughput tracks each party's running average wait and always seats whoever has been served least so far, regardless of kitchen speed. Proportional Fairness splits the difference — it leans toward fast-to-serve parties, but with a built-in bias correcting for whoever's been waiting the longest, so nobody with a slow-cooking order gets permanently starved out.

---

## 7.3.6 Energy Considerations: Wake/Sleep

Energy is a precious resource on battery-powered wireless devices, so most wireless networks provide explicit **power-management** capabilities letting a device minimize how long its sense, transmit, and receive circuitry stays "on." An LTE radio can consume between 1,000 and 3,500 mW while transmitting or receiving, but less than 15 mW when powered off — and measurements show that 20% to over 50% of a 4G/5G device's total energy use is spent purely on communication. Managing radio **sleep/wake cycles** is therefore central to overall device energy management, and falls into two broad approaches:

- **Coordinated sleep/wake:** a protocol between device and base station coordinates the device's sleep/wake cycle, so both sides know when the device is asleep and when it will next wake. The base station can then queue arriving datagrams destined for the sleeping device and deliver them once it wakes. Both WiFi and 5G adopt this approach.
- **Uncoordinated, wake-and-send:** a device wakes _only_ when it has data of its own to send — typical of simple IoT devices reporting occasional sensor readings. Bluetooth, LoRaWAN, and 4G LTE Narrowband IoT (NB-IoT) adopt this approach.

> **Analogy — A Scheduled Shift vs. An On-Call Worker:** Coordinated sleep/wake is like a factory worker on a published shift schedule — the foreman (base station) knows exactly when the worker will next be at their station and can stack up tasks (buffered datagrams) to hand over the moment they clock in. Uncoordinated wake-and-send is like an on-call plumber who only picks up the phone (wakes) when _they_ decide to check in — reliable for the plumber's own errands, but nobody can proactively queue work for them to receive.

### Coordinated Sleep/Wake in IEEE 802.11 Networks

802.11 networks have supported power management since their inception. A WiFi device indicates to the AP that it is about to sleep by setting the **power-management bit** in its 802.11 frame header (recall Figure 7.26) to 1. Knowing this, the AP buffers any arriving frames destined for that device until it wakes. Upon waking, the device scans for a **beacon frame** from the AP containing a **Traffic Indication Map (TIM)** — a list of devices with frames currently buffered and awaiting delivery at the AP. If the device finds itself listed, it sends a polling message asking the AP to transmit the buffered frames; otherwise, it simply returns to sleep.

More recent power-management capabilities have been layered on top of this basic mechanism. WiFi 6 introduced a **Target Wake Time (TWT)**, letting a device _negotiate_ the length of its own sleep period with the AP — the maximum negotiable TWT value is an almost comically long **4 years, 5 months, 16 days, 13 hours, 37 minutes, and 39.99 seconds**. IEEE 802.11ah, designed specifically for IoT applications, inherits and further extends 802.11's power-saving capabilities (e.g., reduced header sizes) to conserve additional energy.

> **Analogy — Booking a Custom Wake-Up Call:** Polling a fixed beacon interval is like a hotel guest checking the front desk at rigid, hotel-set intervals to ask "any messages for me?" A Target Wake Time is like that same guest instead _booking a custom wake-up call_ for precisely when they expect to need it — sometimes minutes away, sometimes (per TWT's maximum) years away — letting the front desk hold every message quietly in the meantime without the guest ever needing to check in early.

### Coordinated Sleep/Wake in 5G Networks

Just as with WiFi, a 5G user device's radio has no reason to stay continuously "on" when there's nothing to send or receive, so 5G devices also cycle through sleep/wake states. There are two forms of 5G device sleep: a **light sleep**, roughly akin to a short nap, and a **deeper sleep**.

The light sleep cycle, known as **Discontinuous Reception (DRX)** in the Connected State, has two phases — a **short DRX cycle** and a **long DRX cycle** — differing only in how long the device sleeps between wake-checks, trading latency (delay until a buffered datagram is delivered) against the energy cost of periodically powering the radio back up.

```
Fig 7.38 -- 5G Light Sleep: Short & Long DRX
────────────────────────────────────────
active  short-DRX cycles   long-DRX cycles  active
 t0    t1            t2              t3,t4  t5
 │ Rx   │inactivity   │  periodic wake, │  pkt
 │      │timeout ->   │  nothing to    │  arrives
 │      │short DRX    │  send/recv ->  │  buffered
 │      │             │  long DRX      │  at t3,
 │starts│                              │  delivered
 │timer │                              │  at t4/t5
```

![[Pasted image 20260724152800.png]]

Walking through the example: at _t₀_, the device receives a transmission and starts an **activity timer**. At _t₁_, having neither received nor sent anything since _t₀_, the inactivity timer expires and the device enters a short DRX sleep cycle, waking periodically to check for pending traffic. After several such short cycles with no activity, the device transitions to a _longer_ DRX cycle at _t₂_. At _t₃_, a packet destined for the device arrives at the base station and is **buffered**, since the device is asleep. Only at the end of the current long sleep cycle, _t₄_, does the device wake, discover the base station is advertising a pending transmission, enter the active state, receive the packet at _t₅_, and restart its activity timer. The added latency this sleep cycle imposes on the arriving packet is exactly _t₅ − t₃_.

Beyond DRX, a device can enter still deeper **Idle** or **Inactive** states after longer stretches of inactivity — states requiring additional signaling steps to re-establish the device's active presence in the RAN before it can send or receive user data again.

|DRX Phase|Sleep Duration|Trade-off|
|---|---|---|
|**Short DRX cycle**|Shorter sleep intervals, wakes more often|Lower latency, higher energy cost|
|**Long DRX cycle**|Longer sleep intervals, wakes less often|Higher latency, lower energy cost|
|**Idle / Inactive state**|Deepest sleep|Lowest energy cost, but requires re-establishing RAN presence before resuming data transfer|

### Uncoordinated, Wake-and-Send in LoRaWAN

**LoRa** is a low-power, low-bitrate wireless networking technology built for simple IoT devices, whose typical job is reporting sensor measurements to a nearby gateway, usually within a few kilometers. The simplest LoRa device type, **Class A**, uses its radio in a purely uncoordinated, wake-and-send fashion: it wakes up entirely on its own schedule, transmits its data without any prior coordination with — or even necessarily any awareness of — a gateway, optionally waits a brief window for any downlink data the gateway might have queued for it, and then goes back to sleep. Device-energy conservation is the first-class design priority here, with the device's default sleep state disturbed only by its own self-initiated wake-ups.

```
Fig 7.39 -- LoRa Wake-and-Send
────────────────────────────────────────
 Gateway                    │LoRa frame│
                             (queued msg,
                              sent after
                              next wake)
 IoT device: t0  t1    t2  t3    t4  t5
   wakes,   sends  sleeps  wakes, sends
   checks   frame          checks msgs,
   for msgs                receives at t4
```

![[Pasted image 20260724153000.png]]

At _t₀_, the device wakes and sends a LoRa frame (e.g., a sensor reading) to the gateway, then briefly listens for any messages the gateway might have — hearing none, it returns to sleep at _t₁_. At _t₂_, the gateway acquires a message destined for the device but must wait until the device's _next_ self-initiated contact to deliver it. That contact happens at _t₃_, when the device again wakes and sends data; the gateway's queued message is finally delivered following _t₃_, received by the device at _t₄_, which then returns to sleep at _t₅_.

Like LoRa, **Bluetooth Low Energy (BLE)** and **4G LTE Narrowband IoT (NB-IoT)** also support device-initiated wake-and-send transmission — though in those cases, the device must first make contact with a controller node or base station upon waking in order to be allocated channel transmission slots for its data.

|Approach|Who Initiates Wake|Latency for Base-Station-to-Device Data|Networks Using It|
|---|---|---|---|
|**Coordinated sleep/wake**|Both sides negotiate a known schedule|Bounded by the negotiated sleep-cycle length|WiFi, 4G/5G|
|**Uncoordinated wake-and-send**|Device alone, on its own schedule|Bounded only by how long until the device _next_ chooses to wake|LoRaWAN (Class A), BLE, NB-IoT|

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**RTS/CTS channel reservation**|An attacker can forge or replay RTS/CTS (or MU-RTS) frames to reserve the channel for durations it never intends to use, forcing every legitimate station to defer — a low-cost, targeted **denial-of-service (DoS)**|Monitoring for anomalously frequent or implausibly long channel reservations, and treating RTS/CTS duration fields as untrusted input, helps flag this class of jamming-by-reservation|
|**Four-address-field trust in 802.11 frames**|An 802.11 frame's address fields carry no built-in authentication; a **rogue AP** can insert itself as Address 1/2 and silently relay, inspect, or manipulate traffic — the mechanism behind classic "evil twin" attacks|WPA2/WPA3 (link-layer encryption/authentication, layered on top of the frame format discussed here) and mutual AP authentication are essential, since the 802.11 frame's addressing alone offers no integrity guarantee|
|**CRC-only integrity checking**|The 802.11 frame's 32-bit CRC catches accidental bit errors but is trivially forgeable by an active attacker — it is not a cryptographic integrity mechanism, so a deliberately tampered frame can still "pass" the CRC check|Genuine integrity requires a cryptographic MAC (e.g., as provided by WPA2/3's encryption suite) layered above this physical/link-layer CRC, never relying on the CRC itself for security|
|**PRACH / random-access flooding**|Because a device's very first contact with a 5G base station happens over the contention-based PRACH, an attacker can flood this channel with bogus access requests, exhausting base-station resources and denying legitimate devices the ability to even join the network|Rate-limiting and anomaly detection on PRACH access attempts, plus authentication as early as architecturally feasible in the connection-establishment sequence, help contain this exposure|
|**Sleep/wake schedule exploitation**|An attacker who learns a device's DRX cycle, TWT interval, or TIM-polling pattern can time transmissions to coincide with the device's sleep windows to evade detection, or deliberately trigger extra wake cycles to drain the device's battery — a so-called **sleep-deprivation attack**, particularly potent against resource-constrained IoT devices|Randomizing wake-schedule timing where the standard permits it, and rate-limiting how often an external party can trigger a device wake-up, mitigates both evasion and battery-drain exploitation|
|**Unauthenticated beaconing and RRC connection setup**|Because a device decides which network to join purely from broadcast beacon/SSB content, an attacker operating a rogue base station or fake AP that mimics a legitimate carrier's broadcast (a classic **"IMSI catcher"** in cellular terms, or "evil twin" in WiFi terms) can lure devices into associating with it before any real authentication occurs — precisely the gap the note above flags between "RAN-level attachment" and genuine network authentication|Real authentication (Section 8.8) must happen before any sensitive traffic is trusted to a newly attached base station/AP; devices and operators should also monitor for anomalous or duplicate broadcast identifiers (MCC/MNC, SSID) in the area|
|**Disaggregated RAN (O-RAN/SD-RAN) interfaces**|Splitting a base station into RU/DU/CU and a control-plane proxy/RIC multiplies the number of standardized, multi-vendor network interfaces — each one is a potential new attack surface (e.g., a malicious or compromised xApp abusing its R-NIB access to manipulate handover or QoS decisions) that simply didn't exist in a monolithic 2G–4G base station|Each disaggregated interface needs its own authentication, authorization, and integrity protections; xApps should run with least-privilege access to the R-NIB, and their control decisions should be auditable|
|**CQI reporting and scheduler manipulation**|Since schedulers like MT and PF rely on devices honestly self-reporting Channel Quality Indication (CQI) values, a malicious device could report false CQI to unfairly monopolize Resource Blocks at other devices' expense — a resource-starvation attack invisible at the physical layer|Base stations can cross-check reported CQI against independently measured signal characteristics and rate-limit or flag devices whose reported quality is implausibly and persistently favorable|

---

## Questions I Still Have

- [ ] How exactly does an AP decide the fair or efficient allocation of RUs among OFDMA-capable devices during an MU-RTS-reserved interval — is it round-robin, channel-quality-aware, or something more elaborate (this seems closely tied to the scheduling material in the not-yet-covered Section 7.3.5)?
- [ ] What real-world TWT values do WiFi-6 IoT deployments actually negotiate in practice, and how does an AP's buffer capacity limit how long a TWT interval can safely be, before buffered data risks overflow or staleness?
- [ ] The text notes downlink RB duration/minislot timing is fixed per generation (15 kHz/66.6 μs for 4G) — what governs 5G's choice among its several larger subchannel bandwidths (30–960 kHz) for a given deployment, and is that choice made once per network or dynamically per device?
- [ ] Since RTS/CTS is described as optional and used only "if at all" for long frames, what's the actual threshold (frame size or configuration) that triggers its use in real 802.11 deployments, and who sets that threshold?
- [ ] How does a 5G device decide the exact number of short DRX cycles to complete before transitioning into a long DRX cycle — is this threshold standardized, or is it base-station/operator-configurable?
- [ ] For LoRa's uncoordinated wake-and-send model, is there any upper bound the standard places on how long a gateway may hold a queued downlink message before it's discarded as stale, given the device controls entirely when it next "checks in"?

---

## Key Terms — Quick Reference

| Term                                                      | Definition                                                                                                                                       |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Channel partitioning / random access / taking-turns**   | The three general multiple-access technique families (from Section 6.3) that wireless networks specialize for radio channels                     |
| **FDM (frequency-division multiplexing)**                 | Dividing a band into narrowband channels separated by idle guard bands                                                                           |
| **Guard band**                                            | An idle slice of spectrum between FDM channels, preventing cross-channel interference                                                            |
| **OFDM (orthogonal frequency division multiplexing)**     | A technique modulating overlapping subchannel carriers so they don't interfere, eliminating the need for guard bands                             |
| **Spectral efficiency**                                   | How much usable data capacity a technique packs into a given amount of spectrum                                                                  |
| **OFDMA (orthogonal frequency division multiple access)** | OFDM combined with time-division multiplexing, allowing multiple devices to share a channel across both frequency and time                       |
| **Resource Element (RE)**                                 | The smallest OFDMA resource unit: one minislot at one subchannel frequency, carrying one symbol                                                  |
| **Resource Block (RB)**                                   | A group of adjacent REs, the smallest schedulable unit in 4G/5G networks                                                                         |
| **Resource Unit (RU)**                                    | A group of adjacent REs, the smallest schedulable unit in WiFi OFDMA networks                                                                    |
| **Aloha**                                                 | A simple random-access protocol: transmit whenever data is ready; retransmit on collision                                                        |
| **CSMA/CA**                                               | Carrier Sense Multiple Access with Collision Avoidance; WiFi's channel-access protocol, avoiding rather than detecting collisions                |
| **Hidden terminal problem**                               | Two stations, both in range of a common third station, cannot hear each other and may transmit simultaneously, colliding at that third station   |
| **No collision detection**                                | A wireless transmitter cannot reliably detect its own signal colliding with another's, since its own transmission overwhelms any incoming signal |
| **DIFS (Distributed Inter-Frame Space)**                  | The wait period a station observes before transmitting on a sensed-idle channel                                                                  |
| **SIFS (Short Inter-Frame Space)**                        | The brief wait period before a destination sends a link-layer ACK                                                                                |
| **Binary exponential backoff**                            | A randomized, doubling-interval retransmission delay used to reduce repeated collisions                                                          |
| **RTS (Request to Send)**                                 | A short control frame reserving the channel ahead of a DATA transmission                                                                         |
| **CTS (Clear to Send)**                                   | The AP's broadcast reply to an RTS, granting access and instructing others to defer                                                              |
| **MU-RTS (multi-user RTS)**                               | WiFi 6's extension of RTS, used by an AP to reserve the channel for simultaneous OFDMA transmissions to multiple devices                         |
| **BSS (Basic Service Set)**                               | The fundamental 802.11 building block: two or more wireless nodes, typically including a central AP                                              |
| **AP (Access Point)**                                     | The layer-2-only central base station of a BSS                                                                                                   |
| **Channel bonding**                                       | Combining multiple adjacent 20 MHz WiFi channels to form a wider channel                                                                         |
| **802.11 MAC data plane**                                 | The component transporting link-layer frames to/from end-user applications                                                                       |
| **802.11 MAC control plane**                              | The component implementing RTS/CTS, ACKs/NAKs, beaconing, and power control                                                                      |
| **MAC scheduler**                                         | The component scheduling data and control frames for transmission across the wireless medium                                                     |
| **Frame control field**                                   | The 802.11 frame subfield identifying frame type/subtype and carrying bits like power-management and retry                                       |
| **Address 1/2/3/4**                                       | The four 802.11 MAC address fields: receiving device, transmitting device, router interface, and (ad hoc mode only) forwarding device            |
| **Sequence number**                                       | An 802.11 frame field letting a receiver distinguish new frames from retransmissions                                                             |
| **Duration field**                                        | An 802.11 frame field reserving channel time for a transmission and its ACK                                                                      |
| **RAN (Radio Access Network)**                            | The edge of a cellular network: user devices, the wireless channel, and the base station                                                         |
| **UE (User Equipment)**                                   | Cellular term for an end-user wireless device                                                                                                    |
| **NR (New Radio)**                                        | The 5G wireless channel technology, as distinguished from 4G LTE's radio channel                                                                 |
| **gNB (Next Generation Node)**                            | The 5G base station                                                                                                                              |
| **5G Core**                                               | The links, routers, and servers connecting a RAN to the broader Internet, using standard wired protocols                                         |
| **CUPS (Control Plane and User Plane Separation)**        | The 5G architectural principle separating control-plane and user-plane functions                                                                 |
| **Core Network Functions**                                | The 5G Core's control/management functions (AMF, AUSF, SMF, NSSF, NRF, AF, UPF, PCF, UDM, NSSAAF, NEF)                                           |
| **Local breakout**                                        | Running application logic at the base station or in the RAN so client-server traffic never leaves the cellular network                           |
| **TDD (Time Division Duplexing)**                         | Sharing one frequency between uplink and downlink via time-slot assignment                                                                       |
| **FDD (Frequency Division Duplexing)**                    | Reserving distinct frequencies exclusively for uplink versus downlink                                                                            |
| **PDSCH**                                                 | Physical Downlink Shared Channel — carries downstream user-plane data and control messages                                                       |
| **PDCCH**                                                 | Physical Downlink Control Channel — informs devices of downlink/uplink RB assignments                                                            |
| **PBCH**                                                  | Physical Broadcast Channel — carries network-discovery information for joining devices                                                           |
| **PUSCH**                                                 | Physical Uplink Shared Channel — carries upstream user-plane data and control messages                                                           |
| **PRACH**                                                 | Physical Random-Access Channel — used by a device to request its initial connection                                                              |
| **PUCCH**                                                 | Physical Uplink Control Channel — requests future RB allocation, carries measurement/ACK data                                                    |
| **Coordinated sleep/wake**                                | A device and base station jointly negotiate and track a known sleep/wake schedule                                                                |
| **Uncoordinated wake-and-send**                           | A device wakes only on its own initiative, whenever it has data to send                                                                          |
| **TIM (Traffic Indication Map)**                          | A field in an 802.11 beacon frame listing devices with buffered, awaiting-delivery frames                                                        |
| **TWT (Target Wake Time)**                                | A WiFi 6 mechanism letting a device negotiate a custom sleep-period length with the AP                                                           |
| **DRX (Discontinuous Reception)**                         | The 5G Connected-State light-sleep mechanism, with short and long cycle phases                                                                   |
| **LoRa Class A**                                          | The simplest LoRa device type, using fully uncoordinated wake-and-send behavior                                                                  |

---

## Related Concepts

---

→ Next: [[7.3.4 WiFi and Cellular Network Discovery]]