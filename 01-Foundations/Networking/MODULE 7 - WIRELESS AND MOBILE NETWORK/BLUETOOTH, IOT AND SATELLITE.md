---
title: BLUETOOTH, IOT AND SATELLITE
date: 2026-07-24
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 7.6 Bluetooth, Satellite, and IoT Wireless Networks

> **One-Line Summary:** Beyond WiFi and 5G, three additional, more specialized wireless network families each solve a narrower problem with interestingly different architectures — **Bluetooth** (a low-power, short-range, ad hoc **piconet** for cable replacement, using frequency-hopping and a centralized-controller/peripheral structure with no network or transport layer), **satellite networks** (increasingly dominated by fast-moving, low-altitude **LEO constellations** rather than fixed **GEO** satellites, raising the novel challenge of _infrastructure_-side rather than device-side mobility), and **IoT networks** (a proliferating family of technologies — 802.11ah, BLE, Zigbee, LoRaWAN, NB-IoT, LTE-M — that each trade off area coverage, energy consumption, and data rate differently, with no single winner-take-all solution).

---

## Core Idea

Sections 7.3 through 7.5 focused primarily on WiFi and 5G — the two dominant wireless network types most readers interact with daily. But, as the chapter is careful to point out, these are far from the _only_ widely deployed wireless network types. This section briefly surveys three more: **Bluetooth**, **satellite**, and **IoT wireless networks**. Because each of these provides more specialized services than the general-purpose connectivity WiFi and 5G aim for, each has evolved an interestingly different architecture and protocol design suited to its own particular niche — short-range cable replacement for Bluetooth, global coverage with minimal ground infrastructure for satellite, and an enormous diversity of low-power/wide-area/high-rate tradeoffs for IoT. A boxed discussion of **GPS and WiFi-based location discovery** — the technology underlying how a smartphone actually determines its own position — is also covered here, sitting naturally between the Bluetooth and satellite discussions since it draws on both GPS satellites and WiFi beacon signals.

---

## 7.6.1 Bluetooth Networks

### What Bluetooth Is For

Bluetooth networks have quickly become part of everyday life — connecting a computer to a wireless keyboard or mouse, connecting wireless earbuds, a speaker, a watch, or a health-monitoring band to a smartphone, or connecting a smartphone to a car's audio system. In every one of these cases, Bluetooth operates over **short ranges** (tens of meters or less), at **low power**, and at **low cost**. For this reason, Bluetooth networks are sometimes called **wireless personal area networks (WPANs)** or, because of their limited range, **piconets**.

Despite being small and relatively simple by design, Bluetooth networks are packed with many of the link-level networking techniques studied earlier in the chapter: **time division and frequency division multiplexing** (Section 6.3.1), **randomized backoff** (Section 6.3.2), **polling** (Section 6.3.3), **error detection and correction** (Section 6.2), and **reliable data transfer and flow control** (Chapter 3) — and that's just considering Bluetooth's link layer!

> **Analogy — A USB Cable You Can't See:** Bluetooth's whole reason for existing is captured well by thinking of it as an _invisible, short USB cable_. A USB cable is cheap, short-range by nature, and just barely smart enough to move bytes reliably between two nearby devices — nobody expects a USB cable to route traffic across a city. Bluetooth aims for exactly that same modest, unambitious goal, just without the physical wire.

### The Bluetooth Channel: TDM + FDM Combined, and Frequency Hopping — Figure 7.46

Bluetooth networks operate in the **unlicensed 2.4 GHz Industrial, Scientific, and Medical (ISM)** radio band, shared with other household devices like microwaves, garage-door openers, and cordless phones. As a direct consequence, Bluetooth is designed explicitly with **noise and interference in mind**.

The Bluetooth wireless channel is operated in a combined **TDM and FDM** manner: there are **79 frequency channels**, and **625-microsecond time slots** on each channel. During each time slot, a sender transmits on just one of the 79 channels — and then the sender changes the channel (frequency) used, in a **pre-determined manner**, from one slot to the next.

![[Pasted image 20260724165544.png]]

This form of channel hopping is known as **frequency-hopping spread spectrum (FHSS)**. It's used so that interference from another device operating at a _given_ frequency in the ISM band will only ever interfere with Bluetooth communications during a small number of slots — not the entire conversation. Additionally, slots that experience significant interference (manifested, or revealed, as a low signal-to-noise ratio) can be **adaptively removed** from the hopping pattern altogether. Multiple Bluetooth networks can co-exist in the same physical space so long as each uses a **different** frequency-hopping pattern (note network 1 and network 2's distinct patterns in Figure 7.46). Bluetooth data rates can reach up to 2 to 3 Mbps.

> **Analogy — A Conversation That Keeps Switching Rooms:** Imagine two people trying to have a private conversation in a noisy, crowded house with 79 different rooms. Rather than staying in one room where a noisy party guest (interference) might camp out and drown them out the entire time, the two conspirators agree in advance on a _pre-arranged sequence_ of room changes every fraction of a second — so even if the noisy guest occasionally wanders into whichever room they're currently in, it only ruins one brief snippet of the conversation, never the whole thing. A second pair of conspirators in the same house can hold their _own_ private conversation at the same time, entirely undisturbed, so long as they've agreed on a _different_ sequence of room-hops.

### Bluetooth as an Ad Hoc Network: The Piconet — Figure 7.47

Bluetooth networks are **ad hoc networks**, meaning no network infrastructure (i.e., no base station) is needed at all. In most usage cases, Bluetooth communication is simply between two devices — for example, between wireless earphones and a smartphone. But the Bluetooth standard also allows devices to organize themselves into larger networks, and so a **multiple access protocol** is required.

Bluetooth devices organize themselves into a **piconet** of up to **eight active devices**. One of these devices is designated as the **centralized controller**, with the remaining devices acting as **clients** in a "peripheral" role.

![[Pasted image 20260724165245.png]]

The centralized controller node truly **rules** the piconet:

- Its clock determines timing in the piconet (e.g., it determines **TDM slot boundaries**).
- It controls the piconet's **frequency-hopping sequence**.
- It controls **entry** of client devices into the piconet.
- It controls the **power** (100 mW, 2.5 mW, or 1 mW) at which peripheral devices transmit.
- Using **polling**, it grants transmission privileges to a peripheral device, once that device has been admitted to the network.

Critically, in Bluetooth, communication occurs **only between the central and peripheral nodes** — there is **no direct peripheral-to-peripheral communication** at all.

|Role|Responsibilities|Communicates Directly With|
|---|---|---|
|**Centralized controller**|Sets TDM slot boundaries, sets frequency-hopping sequence, admits clients, sets peripheral transmit power, polls for transmission rights|Every peripheral in the piconet|
|**Peripheral (client)**|Transmits only when polled by the controller, at the power level the controller assigned|Only the centralized controller — never another peripheral|

> **Analogy — A Strict Classroom Teacher, Not a Study-Group Free-for-All:** A Bluetooth piconet works less like a free-flowing study group where anyone can chime in whenever they like, and much more like a strict classroom where the teacher (centralized controller) alone decides who speaks and when — students (peripherals) must wait to be called on (polled), may only speak at a volume the teacher permits (transmit power), and are never allowed to whisper directly to each other; every exchange, even student-to-student information, must pass back through the teacher first.

### Bootstrapping the Piconet: Neighbor Discovery and Paging

Because Bluetooth ad hoc networks must be **self-organizing**, it's worth looking into exactly how they bootstrap their own network structure from nothing.

**Phase 1 — Neighbor discovery.** When a centralized-controller-to-be wants to form a Bluetooth network, it must first determine which other Bluetooth devices are within range; this is the **neighbor-discovery problem**. The centralized controller does this by broadcasting a series of **32 inquiry messages**, each on a different frequency channel, and repeats this entire transmission sequence up to **128 times**. A client device listens on its chosen frequency, hoping to hear one of the centralized controller's inquiry messages on that frequency. When it hears an inquiry message, it backs off a **random amount of time between 0 and 0.3 seconds** — reminiscent of WiFi's and Ethernet's binary backoff, used here to avoid collisions with other responding nodes — and then responds to the centralized controller with a message containing its **device ID**.

**Phase 2 — Paging.** As the Bluetooth centralized controller discovers potential clients within range, it can then invite those clients it wishes to join its piconet. This second phase is known as **Bluetooth paging** (not to be confused with paging in 5G networks, Section 7.5.3) and is reminiscent of 802.11 clients associating with a base station. The controller begins the paging process by again sending **32 identical paging invitation messages**, each now addressed to a _specific_ client — but again using different frequencies, since that client doesn't yet know the frequency-hopping pattern the controller intends to use. Once the client replies with an ACK to the paging invitation message, the centralized controller sends **frequency-hopping information, clock synchronization information, and an active member address** to the client. It then finally **polls** the client, now using the frequency-hopping pattern, to confirm that the client is truly connected into the network.

|Phase|Purpose|Key Mechanism|
|---|---|---|
|**Neighbor discovery**|Find out which devices are nearby at all|32 inquiry messages per sweep, up to 128 sweeps; client responds with device ID after random backoff|
|**Paging**|Invite a _specific_ discovered client to join|32 paging invitations to that client; controller then shares hopping pattern, clock sync, member address|

> **Analogy — Shouting Into a Crowd, Then Sending a Personal Invitation:** Neighbor discovery is like standing in a crowded room and repeatedly shouting "is anybody there?" in every possible direction (frequency), hoping someone within earshot shouts back their name. Paging is the very different second step that follows: once you know Priya is in the room, you stop shouting generally and instead send her, specifically, a personalized invitation card with the actual party details (frequency-hopping schedule, clock sync) written on it — details you'd never have broadcast to the whole room during the first "is anybody there?" phase.

### The Bluetooth Protocol Stack — Figure 7.48

Because Bluetooth was designed for a specific, limited set of services (unlike the general-purpose Internet or 5G networks), it's instructive to take a quick look at its protocol stack.

![[Pasted image 20260724165745.png]]

Perhaps most strikingly, the Bluetooth protocol stack **lacks both a network layer and a traditional transport layer**! This lack of a network layer is perhaps unsurprising, given that Bluetooth networks are generally **single-hop**, with only controller-to-peripheral or peripheral-to-controller transmissions — so multi-hop forwarding and routing simply aren't a concern. (Multi-hop extensions to the basic Bluetooth mechanisms do exist, in the **Bluetooth Scatternet** and, more recently, the **Low Energy Mesh** specifications, but these have seen extremely limited deployment.)

Many of the Internet's end-to-end transport-layer services — including reliable data transfer, flow control, segmentation, connection-oriented and connectionless services, and an API for upper-layer applications — are instead provided by Bluetooth's own **Logical Link Control and Adaptation Protocol (L2CAP)** layer. L2CAP additionally provides a service that **reserves periodic slots** for a connection's transmissions — useful for multimedia applications that have known bandwidth requirements and fixed timing needs.

|Layer Present in Bluetooth?|Internet Equivalent|Notes|
|---|---|---|
|**Physical**|Physical layer|FHSS across 79 channels, 625 us slots|
|**Link**|Link layer|Handles piconet TDM, polling, controller role|
|**L2CAP**|Rolls up transport-layer functions|Reliable transfer, flow control, segmentation, connection setup, reserved periodic slots|
|**Traditional transport layer**|— (absent)|Not needed: no multi-hop forwarding to coordinate across|
|**Network layer**|— (absent)|Not needed: Bluetooth is fundamentally single-hop|

> **Analogy — A Building With No Elevator Because It Only Has One Floor:** Asking why Bluetooth's protocol stack has no network layer is a bit like asking why a single-story cottage has no elevator shaft. An elevator (network-layer routing) only becomes necessary once you need to move things _between_ multiple floors (multiple hops). A single-story cottage — like a single-hop Bluetooth piconet — simply has nowhere else to route _to_, so the whole "vertical transportation" problem never arises in the first place.

---

## Location Discovery: GPS and WiFi Positioning

### The Two-System Combination Behind "Where Am I?"

Many of today's most useful smartphone apps — Foursquare, Yelp, Uber, Pokémon Go, and Waze among them — are **location-based mobile apps**, relying on an API that lets them extract the phone's current geographical position directly. That position is actually obtained by combining **two separate systems**: the **Global Positioning System (GPS)** and the **WiFi Positioning System (WPS)**.

### GPS: Trilateration From Orbit

GPS, with a constellation of **30+ satellites**, broadcasts satellite location and timing information, which each GPS receiver then uses to estimate its own **geolocation**. The system is created, maintained, and made freely accessible to anyone with a GPS receiver by the United States government. The satellites carry extremely stable **atomic clocks**, synchronized both with one another and with ground clocks. Each GPS satellite continuously broadcasts a radio signal containing its current time and position; if a receiver obtains this information from at least **four** satellites, it can solve **trilateration equations** to estimate its own position.

GPS, however, cannot always provide accurate geolocations — specifically when a receiver lacks line-of-sight with at least four GPS satellites, or when interference from other high-frequency communication systems is present. This is especially true in **urban environments**, where tall buildings frequently block GPS signals entirely.

### WiFi Positioning: Crowdsourced Access-Point Databases

This is precisely where WiFi positioning systems come to the rescue. WiFi positioning systems make use of **databases of WiFi access points**, independently maintained by various Internet companies, including **Google, Apple, and Microsoft**. Each database contains information about millions of WiFi access points, including each access point's **SSID** and an estimate of its **geographic location**.

To see how a WiFi positioning system actually works, consider an Android smartphone using the Google location service. From each nearby access point, the smartphone receives and measures the signal strength of **beacon signals** (recall Section 7.3.4). The smartphone can then continually send messages to the Google location service (in the cloud) that include the SSIDs of nearby access points along with their corresponding signal strengths. It will also send its GPS position (when available), obtained via the satellite broadcast signals described above. Using the signal-strength information, Google will estimate the _distance_ between the smartphone and each of the WiFi access points; leveraging these estimated distances, it can then solve trilateration equations to estimate the smartphone's geolocation. Finally, this WiFi-based estimate is combined with the GPS satellite-based estimate to form an **aggregate estimate**, which is then sent back to the smartphone and used by the location-based mobile app.

### The Other Half of the Loop: How the AP Database Itself Gets Built

You may still be wondering how Google (and Apple, Microsoft, and so on) obtain and maintain the database of access points in the first place — and in particular, each access point's geographic location. Recall that for a given access point, **every** nearby Android smartphone will send to the Google location service the strength of the signal received from that access point, along with the smartphone's own _estimated_ location. Given that thousands of smartphones may be passing by any single access point during any given day, Google's location service ends up with **lots of data** at its disposal for estimating the access point's own position — again, by solving trilateration equations, just from the opposite direction. Thus, in a neat closed loop: **access points help smartphones determine their locations, and in turn the smartphones help the access points determine their locations!**

> **Analogy — Two People Guessing Each Other's Position by Comparing Notes:** Picture two people lost in a large fog-covered field, each holding only a rough guess of where they stand. Every time they pass near each other, they shout out "I think I'm _roughly here_, and you sound like you're about this far away." Neither guess alone is very reliable — but after enough of these brief exchanges, accumulated across thousands of such encounters, both people's estimates of their own positions (and of each other's _fixed_ landmarks) get sharper and sharper, purely from comparing distance notes with one another. That's essentially the GPS/WiFi mutual bootstrapping loop, just replacing "people in fog" with "smartphones and access points."

|System|Source of Position Data|Best Suited For|Weakness|
|---|---|---|---|
|**GPS**|30+ orbiting satellites, atomic clocks, trilateration|Open outdoor areas with clear sky view|Blocked by tall buildings, urban canyons, indoor spaces|
|**WiFi Positioning (WPS)**|Crowdsourced AP signal-strength + SSID database (Google/Apple/Microsoft)|Dense urban and indoor environments|Requires a sufficiently populated, up-to-date AP database|
|**Combined estimate**|Aggregate of GPS + WPS trilateration results|General smartphone location-based apps|Accuracy still depends on both underlying systems' data quality|

---

## 7.6.2 Satellite Networks

### GEO vs. LEO: A Rapidly Shifting Landscape

The world of satellite communications has evolved dramatically over the last 10 years. Until roughly 10 years ago, deployed satellite applications primarily used satellites in **geostationary orbit (GEO)**. A geostationary satellite is located roughly **35,000 km above the Earth's equator** and rotates with the Earth so that it remains in the _same_ position in the sky relative to observers on Earth. GEO satellites have been in use for more than 60 years, for applications including broadcast TV, sensing (e.g., of the weather, atmospheric, or surface conditions), and Internet connectivity in areas not served by the terrestrial Internet.

The past 10 years, however, have seen a tremendous increase in interest in a different class of satellite networks known as **low earth orbit (LEO)** satellites. While the applications remain broadly similar (particularly sensing and Internet connectivity), **LEO satellites** are generally smaller, less expensive, and positioned much closer to the Earth than their GEO counterparts. Located between **500 and 1,200 km** above the surface, LEO satellites thus have much shorter round-trip times (**~30 msec**) than GEO satellites (**~800 msec**).

As of November 2022, only **6,800 active satellites** had been launched in all of history. But in just five years (2019–2024), **Starlink alone** has launched an equivalent number (**6,281**) of LEO satellites. Today, well over **90%** of all satellites launched are LEO satellites — clear evidence of just how much recent interest there is in LEO satellite networks specifically.

|Consideration|GEO Satellites|LEO Satellites|
|---|---|---|
|**Altitude**|~35,000 km|500–1,200 km|
|**Position relative to Earth**|Fixed (rotates with Earth)|Constantly moving relative to the ground|
|**Round-trip time**|~800 msec|~30 msec|
|**History**|60+ years of deployment|Dramatic growth only in the last 10 years|
|**Cost/size**|Larger, more expensive|Smaller, less expensive|
|**Recent launch share**|Small minority today|Over 90% of satellites launched today|

> **Analogy — A Fixed Lighthouse vs. a Swarm of Low-Flying Drones:** A GEO satellite is like a single, enormous lighthouse built so far out to sea, and so precisely matched to the Earth's own rotation, that from the shore it always appears to sit in exactly the same spot in the sky — reliable, but its light has to travel an awfully long distance to reach you. A LEO constellation, by contrast, is like a swarm of low-flying drones constantly zipping overhead in relay — no single drone stays overhead for long, but because each one is so much closer, its "light" (signal) reaches you far faster, at the cost of needing a _swarm_ of them, and a constant hand-off between neighboring drones, to maintain continuous coverage at all.

### LEO Satellite Network Architecture — Figure 7.49

![[Pasted image 20260724170039.png]]

- **LEO satellites.** Orbiting LEO satellites are often referred to collectively as a **constellation**. Starlink reports more than **6,750 satellites** in its constellation; in early 2025 there were approximately **650 LEO satellites** in the OneWeb constellation. Because of their relatively low altitude, LEO satellites are **not geostationary**, but instead are in constant motion with respect to the Earth below, traveling at roughly **27,000 km/hr** relative to the ground — as we'll see, this takes the entire topic of "network mobility" in a whole new direction.
- **Satellite links.** The coverage area of a satellite's wireless transmitter/receiver on the ground below is known as its **footprint**. In the downlink direction, a satellite's transmissions are **broadcast** in nature, received by all ground stations within the footprint. In the uplink direction, transmissions from the various ground stations within a satellite's footprint are **unicast** to the satellite — a classical **multiple access channel**! The footprint of a typical LEO satellite is **15 miles in diameter**. A ground station will remain in the footprint of a satellite link's beam for only about **10 minutes or so**, until the satellite's own motion takes it out of view of that ground station.
- **Inter-satellite links (ISLs).** Satellites may also be directly connected to each other using **optical inter-satellite links**, although this type of infrastructure link is not yet widely used in practice.
- **Ground stations.** On or near the Earth's surface, ground stations transmit and receive datagrams over the satellite link. These ground stations may be connected _only_ to the satellite, or may be connected both to the satellite _and_ to the terrestrial Internet, in which case the ground station serves as a **gateway** between the satellite network and the terrestrial Internet. LEO ground stations often have small dish or flat-panel antennas. A low-bandwidth **SOS emergency texting service via satellite** is currently available for some Apple cellphones, and the provision of additional 5G-like data services via LEO satellites is currently under study.

> **Analogy — A Relay Race Where the Runners, Not the Baton, Keep the Race Alive:** The distinction between a satellite's downlink and uplink channel mirrors a familiar pattern from the rest of this chapter: downlink broadcast, uplink unicast-into-a-shared-channel, exactly like a WiFi AP's own downlink/uplink asymmetry (Section 7.3). What's genuinely new here is the _footprint_ itself sweeping across the ground at 27,000 km/hr — like a spotlight on a rotating stage that only illuminates any one seat in the audience for about ten minutes before moving on to the next section entirely.

### Mobility: The Infrastructure Moves, Not the Device

With LEO satellites whizzing through the air at 27,000 km/hr, and with ground stations that may themselves be mobile (on trucks, boats, or airplanes), LEO constellations are wireless networks with **extreme mobility**! In sharp contrast to WiFi and 5G networks, however, it is the **LEO network infrastructure (the satellites)**, not the _user device_, that is the primary source of "mobility" here.

Since a ground station remains attached to any one satellite for only about 10 minutes, **ground-station handover between orbiting satellites** is a common and important occurrence, as one satellite moves out of range of the ground station while another satellite moves into range. Recent measurements suggest that satellite handover is **much less well-developed** than 5G handover (Section 7.5.3), with suspected **packet loss** occurring during these handovers. Also, unlike 5G networks, there is **no LEO satellite network standardization body** that is similar in spirit to the **3GPP** in the cellular network world. As a result, satellite network implementations remain largely **proprietary**.

|Comparison Point|5G Mobility (Section 7.5.3)|LEO Satellite Mobility|
|---|---|---|
|**What actually moves**|The user device|The network infrastructure (satellites) itself|
|**Handover trigger**|Device signal quality / load balancing|Satellite's own orbital motion (~10-min windows)|
|**Standardization**|3GPP-defined, network-wide handover process|No equivalent standards body; proprietary|
|**Handover maturity**|Well-developed, orchestrated 3-phase process|Less well-developed; packet loss suspected|

> **Analogy — Waiting at a Bus Stop That Never Moves, While the Buses Keep Changing:** In 5G mobility, it's _you_ (the device) walking from one bus stop's coverage to another's. In LEO satellite mobility, it's the reverse: you (the ground station) stand at a single, fixed bus stop, and it's the _buses themselves_ (satellites) that keep arriving, lingering only about ten minutes, and then speeding off to be replaced by the next bus in the queue — the handover burden falls on tracking which bus is currently serving your stop, not on your own movement at all.

- **(a) "Bent pipe" architecture.** Each satellite is treated as a single **link**, with ground stations at each end of each link. An end-to-end path thus consists of a series of **single-hop satellite links**, with a packet returning to ground after each and every hop.
- **(b) A network in the sky.** Here, **inter-satellite links (ISLs)** directly connect satellites to one another, and a packet may cross **multiple hops among satellites** before ever returning to ground. Because satellites are non-stationary both with respect to the ground _and_ with respect to each other, the ISL links in a "network in the sky" are **dynamically changing** — making routing in LEO satellite (LEOS) networks a genuinely challenging problem.

|Architecture|Packet's Path|Complexity|Current Deployment Status|
|---|---|---|---|
|**Bent pipe**|Ground → satellite → ground, one hop at a time|Simple: satellite is just a relayed link|More common today|
|**Network in the sky**|May cross several satellites via ISLs before reaching ground|Complex: routing must handle constantly shifting topology|Emerging; ISLs "not yet widely used"|

> **Analogy — Bouncing a Ball Straight Down vs. Passing It Overhead Across a Crowd:** A "bent pipe" satellite link is like bouncing a ball straight down off one specific rebounder and catching it yourself immediately after — simple, but you only ever reach exactly as far as that one rebounder can throw. A "network in the sky" is like a crowd of people passing that ball hand-to-hand _overhead_, potentially quite far across a stadium, before finally letting it drop back down to someone waiting below — much more powerful reach, but now everyone in the passing chain has to constantly recalculate who's actually standing where, since the whole crowd (the satellite constellation) is itself in motion.

---

## 7.6.3 Internet of Things (IoT) Networks

### What IoT Is, and Why It's a Big Deal

The **Internet of Things (IoT)** is about connecting devices which are often small and **energy-constrained**, with computation used to sense, monitor, report, and control phenomena in the physical world. Myriad IoT applications abound across many areas, including but not limited to:

- **(i) Manufacturing** — where physical characteristics such as pressure, vibration, temperature, humidity, and switch settings, and gauge levels, are monitored, often with actions then taken.
- **(ii) Logistics and supply chains** — where asset and transport conditions are monitored and tracked.
- **(iii) Agriculture and the environment** — where rainfall, temperature, wind, humidity, animal tracking/health, and soil conditions are monitored.
- **(iv) Smart cities** — where sound, air, water, weather, lighting, traffic, and crowds are measured, with transportation systems and services activated and deactivated in response.
- **(v) Personal health monitoring and care.**

Some have characterized IoT's impact as being so pervasive and profound as to constitute a **fourth industrial revolution**, or **"Industry 4.0."** With more than **three times as many** connected IoT devices deployed as PCs, laptops, tablets, and cellphones **combined** by the end of this decade, IoT is on pace to soon become a **$300 billion market**.

With so many different applications, and so many different requirements, and such a large potential market, it's unsurprising that there are many different competing technologies angling to provide networked services for IoT.

### The Three-Way Tradeoff — Figure 7.51

Figure 7.51 provides a **three-faceted characterization** of IoT networking technologies, illustrating the extent to which a particular technology emphasizes **area coverage**, **energy consumption**, and/or **high data rates**. Each of the technologies shown emphasizes _two_ of these three characteristics — **none emphasize all three**. This suggests that, rather than a winner-take-all outcome in IoT networking, we're likely to see **different solutions adopted in practice**, depending on the specific context.

![[Pasted image 20260724170133.png]]

### Emphasizing Low Energy and Low Data Rate: Short-Range Technologies

**IEEE 802.11ah**, **Bluetooth Low Energy (BLE)**, and **Zigbee** are three wireless IoT technologies designed for short-range coverage:

- **IEEE 802.11ah** is a "flavor" of WiFi, inheriting much of WiFi's own architecture, including the **BSS structure**, the notion of channels, and a stripped-down 802.11 frame format. 802.11ah provides data rates of hundreds of kbps up to roughly 5–10 Mbps. Operating in the **900 MHz unlicensed band** and with relays, these networks can span up to **a kilometer** in practice.
- **Bluetooth Low Energy (BLE)** is a significant expansion of the original Bluetooth standard (Section 7.6.1) to support IoT devices. Like **LoRa Class A** and **4G LTE Narrowband IoT**, BLE supports **device-initiated wake-and-send transmission** by the IoT device.
- **Zigbee**, introduced in the late 1990s, is one of the earliest IoT networking technologies. It is built on the **physical and MAC layers of the IEEE 802.15.4** standard for low-rate wireless personal area networks.

### Emphasizing Low Energy and Wider Area Coverage: LoRaWAN and NB-IoT

- **LoRaWAN (Long Range Wide Area Network)** is standardized by the **International Telecommunication Union (ITU)**. LoRaWAN is a **network-wide** (as opposed to a link-layer-only) standard, with servers, gateways, and IoT devices operating at data rates ranging from **0.3 kbit/s to 50 kbit/s** per channel in the **902–928 MHz unlicensed band**. LoRaWAN networks span wide areas using a **star-of-star** network topology, with a LoRaWAN **gateway** serving as a coordinating central node in each edge-star network.
- **4G Narrowband Internet of Things (NB-IoT)** is a **3GPP 4G standard** that naturally contains many of the architectural features of the 4G RAN, including base stations, user devices, downlink OFDM, and uplink single-carrier FDMA — all of which were studied earlier in Section 7.3. NB-IoT also supports device-initiated wake-and-send transmission, just like BLE. With **200 kHz channel bandwidths**, LTE NB-IoT data rates are limited to approximately **250 Kbps**.

### Emphasizing Wider Area Coverage and Higher Data Rates: LTE-M

**Long Term Evolution for Machines (LTE-M)** is a second, more fully functional 3GPP LTE standard aimed at IoT applications. It supports **device mobility among base stations** and has channel bandwidths that are **5 times larger** than LTE NB-IoT. The ITU, which works with 3GPP to set the vision for mobile cellular network standards, notes that for 5G, [further detail on 5G-era IoT vision continues beyond the excerpted material].

|Technology|Primary Emphasis (2 of 3)|Typical Data Rate|Typical Range|Standardization Body|
|---|---|---|---|---|
|**IEEE 802.11ah**|Short range, higher data rate|100s kbps – ~5–10 Mbps|Up to ~1 km (with relays)|IEEE|
|**BLE**|Short range, low power|Low (optimized for energy)|Short-range (WPAN scale)|Bluetooth SIG|
|**Zigbee**|Short range, low power|Low-rate|Short-range (WPAN scale)|IEEE 802.15.4-based|
|**LoRaWAN**|Wide area, low power|0.3–50 kbit/s per channel|Wide area (star-of-star)|ITU|
|**NB-IoT**|Wide area, low power|~250 Kbps (200 kHz channels)|Cellular-scale (4G RAN)|3GPP|
|**LTE-M**|Wide area, higher data rate|Higher than NB-IoT (5x bandwidth)|Cellular-scale, supports mobility|3GPP / ITU|

> **Analogy — Choosing a Vehicle for the Right Errand, Not One Car for Everything:** The IoT landscape's "no single winner" pattern is like a household that owns a bicycle, a delivery van, and a highway-capable car, rather than trying to force one vehicle to handle grocery runs, moving furniture, and cross-country road trips all equally well. A bicycle (BLE/Zigbee) is perfect for tiny, short, frequent errands where saving energy matters most; a delivery van (LoRaWAN/NB-IoT) is built for going a long way efficiently, even if it's not fast; and a highway car (LTE-M) is worth the extra fuel cost when both distance _and_ speed genuinely matter. No single one of these vehicles "wins" outright — the right choice always depends on the errand.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Bluetooth's unencrypted discovery phase**|Inquiry and paging messages are broadcast openly across many frequencies, potentially letting an attacker passively enumerate nearby Bluetooth devices and their device IDs before any security association is even established|Devices should minimize how long they remain discoverable, and treat device IDs harvested during discovery as non-sensitive-but-trackable identifiers|
|**Centralized controller as a single point of trust**|Because the centralized controller alone governs entry, power levels, and polling for an entire piconet, compromising it hands an attacker control over every peripheral's transmission privileges|Peripheral devices should independently validate any commands, and piconet formation should require mutual authentication, not just controller-side admission control|
|**Crowdsourced WiFi-positioning databases**|Because Google/Apple/Microsoft's AP-location databases are built from crowdsourced smartphone reports, a malicious actor could potentially submit spoofed signal-strength/location reports to corrupt an access point's recorded position, misleading other users' location estimates|Location-service providers should apply outlier detection and require corroboration across many independent devices before updating an AP's stored location|
|**SOS/emergency satellite texting**|A spoofed or intercepted satellite uplink transmission near an emergency ground station could, in principle, be used to inject false distress signals or disrupt legitimate ones|Authentication and integrity protection on satellite uplink emergency channels is essential given their life-safety role|
|**Lack of a LEO satellite standardization body**|Because satellite network implementations remain largely proprietary with no 3GPP-like standards body, security practices for inter-satellite and ground-station handover may vary widely and inconsistently across providers|Operators should independently adopt strong authentication for ISLs and ground-station handover until industry-wide security standards mature|
|**IoT devices' resource constraints**|Small, energy-constrained IoT devices often cannot afford the computational overhead of strong cryptography, making them attractive, comparatively soft targets for compromise or use as botnet nodes|Lightweight cryptographic protocols and firmware update mechanisms designed specifically for constrained devices help close this gap without exhausting limited power budgets|

---

## Questions I Still Have

- [ ] When a Bluetooth piconet's centralized controller device itself needs to leave (e.g., its battery dies or it's powered off), does piconet leadership actually transfer to another device, or does the entire piconet simply dissolve?
- [ ] How exactly does the "adaptive removal" of interference-heavy slots from a Bluetooth frequency-hopping pattern get coordinated and re-synchronized between the controller and all its peripherals in real time?
- [ ] For WiFi positioning, how does Google's location service actually detect and filter out obviously bad or spoofed AP location reports submitted by malicious or malfunctioning smartphones?
- [ ] Given that LEO satellite handover is described as "much less well-developed" than 5G handover with suspected packet loss, are there concrete engineering efforts underway (analogous to 5G's 3-phase process) to close that gap?
- [ ] With no LEO satellite standards body equivalent to 3GPP, how do different satellite operators (Starlink, OneWeb, etc.) handle interoperability at all — or is there currently no expectation that a ground station could ever hand over between two _different_ companies' constellations?
- [ ] Among the three IoT technology "corners" in Figure 7.51 (short-range, low-power-wide-area, traditional cellular), is there a genuinely emerging fourth approach attempting to satisfy all three properties simultaneously, or does the physics/economics of radio truly make that a fundamental trilemma?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Bluetooth / WPAN / piconet**|A low-power, short-range, ad hoc wireless personal area network technology, sometimes called a piconet due to its limited range|
|**ISM band**|The unlicensed 2.4 GHz Industrial, Scientific, and Medical radio band Bluetooth shares with microwaves, garage-door openers, and cordless phones|
|**Frequency-hopping spread spectrum (FHSS)**|Bluetooth's channel-hopping scheme across 79 channels in 625 microsecond slots, limiting interference to brief windows and allowing multiple co-located networks|
|**Ad hoc network**|A network requiring no fixed infrastructure (no base station), such as a Bluetooth piconet|
|**Piconet**|A Bluetooth network of up to 8 active devices, with one centralized controller and the rest as peripherals|
|**Centralized controller**|The Bluetooth piconet device that sets timing, hopping sequence, admits clients, sets transmit power, and polls peripherals|
|**Peripheral**|A Bluetooth client device that transmits only when polled, and never communicates directly with other peripherals|
|**Neighbor-discovery problem**|The challenge of a Bluetooth controller determining which devices are within range, solved via repeated inquiry message broadcasts|
|**Bluetooth paging**|The second bootstrap phase, in which a controller invites a specific, already-discovered client to formally join its piconet|
|**L2CAP (Logical Link Control and Adaptation Protocol)**|Bluetooth's layer providing transport-like services (reliable transfer, flow control, segmentation, reserved periodic slots) in the absence of a true transport layer|
|**Bluetooth Scatternet / Low Energy Mesh**|Limited-deployment multi-hop extensions to Bluetooth's basic single-hop mechanisms|
|**GPS (Global Positioning System)**|A US-government-maintained constellation of 30+ satellites broadcasting time/position for trilateration-based geolocation|
|**Trilateration**|The mathematical technique of estimating position from distance estimates to multiple known reference points|
|**WiFi Positioning System (WPS)**|A location system using crowdsourced databases of WiFi access point SSIDs and estimated locations, maintained by Google, Apple, and Microsoft|
|**Geostationary orbit (GEO)**|A ~35,000 km orbit in which a satellite rotates with the Earth, appearing fixed in the sky|
|**Low earth orbit (LEO)**|A 500–1,200 km orbit; satellites here are non-stationary relative to Earth, offering much shorter RTTs than GEO|
|**Constellation**|The collective set of orbiting satellites belonging to one operator (e.g., Starlink, OneWeb)|
|**Footprint**|The coverage area of a satellite's wireless transmitter/receiver on the ground below|
|**Inter-satellite link (ISL)**|An optical link directly connecting two satellites, not yet widely deployed|
|**Ground station**|Earth-surface equipment that transmits/receives over the satellite link, sometimes gatewaying to the terrestrial Internet|
|**"Bent pipe" architecture**|A satellite network design where each satellite is treated as a single-hop link, with ground stations at each end|
|**Network in the sky**|A satellite network design where ISLs let a packet hop between multiple satellites before returning to ground|
|**Internet of Things (IoT)**|The domain of connecting small, energy-constrained devices that sense, monitor, report, and control physical-world phenomena|
|**Industry 4.0**|A characterization of IoT's impact as constituting a fourth industrial revolution|
|**IEEE 802.11ah**|A WiFi "flavor" IoT technology using the 900 MHz band, BSS structure, and a stripped-down 802.11 frame format|
|**Bluetooth Low Energy (BLE)**|An IoT-focused expansion of Bluetooth supporting device-initiated wake-and-send transmission|
|**Zigbee**|An early IoT technology built on IEEE 802.15.4's physical/MAC layers for low-rate WPANs|
|**LoRaWAN**|An ITU-standardized, network-wide, low-power wide-area IoT technology using a star-of-star topology|
|**NB-IoT (Narrowband IoT)**|A 3GPP 4G-based IoT standard with 200 kHz channels and ~250 Kbps data rates|
|**LTE-M**|A more fully functional 3GPP LTE IoT standard supporting device mobility and 5x the bandwidth of NB-IoT|

---

## Related Concepts

---

→ Next: [[8 Security in Computer Networks]]