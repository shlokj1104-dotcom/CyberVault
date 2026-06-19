---
title: THE NETWORK EDGE
date: 2026-05-13
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 1.2 The Network Edge

> **One-Line Summary:** The network edge consists of end systems (clients and servers), access networks that connect them to ISPs, and physical media that carry signals — from DSL, cable, FTTH, and fixed wireless to fiber optics, WiFi, and 4G/5G wireless cellular technologies.

---

## End Systems and Hosts

Computers and other devices connected to the Internet are called **end systems** because they sit at the edge of the Internet (Figure 1.3). End systems include desktop computers (desktop PCs, Macs, Linux boxes), servers (Web and e-mail servers), and mobile devices (laptops, smartphones, tablets). An increasing number of non-traditional "things" are also being attached to the Internet as end systems (IoT devices).

![[Pasted image 20260619124259.png]] _(Figure 1.3 — Internet overview: Mobile, Home, and Enterprise networks connecting through Local/Regional and National/Global ISPs to Datacenter and Content Provider networks)_

End systems are also referred to as **hosts** because they host (run) application programs — a Web browser, a Web server program, an e-mail client or server program, and so on. Throughout this book: **host = end system**.

### Clients vs. Servers

|Type|Characteristics|Examples|
|---|---|---|
|**Clients**|Desktop and mobile PCs, smartphones, tablets — lower-powered, user-facing devices|End-user devices|
|**Servers**|More powerful machines that store and distribute data|Web servers, email servers, video streaming servers|

> Note: The terms **hosts** and **end systems** are used interchangeably with clients/servers throughout the book.

Today, most servers from which we receive search results, e-mail, Web pages, videos, and mobile app content reside in large **data centers**. As of late 2024, **Google has 33 data centers on four continents**, collectively containing **several million servers**.

> [!info] Case History — Data Centers and Cloud Computing Internet companies such as Google, Microsoft, Amazon, and Alibaba have built massive data centers, each housing tens to hundreds of thousands of hosts. These data centers are connected to the Internet, but also internally include complex computer networks that interconnect the datacenter's hosts.
> 
> Broadly, data centers serve three purposes (using Amazon as an example):
> 
> 1. **Serve content directly** — e.g., e-commerce pages, product and purchase info
> 2. **Massively parallel computing infrastructure** — for company-specific data processing tasks
> 3. **Cloud computing** — providing infrastructure to other companies. Today a major trend is for companies to outsource essentially all of their IT needs to a cloud provider. For example, Airbnb and many other Internet-based companies don't own/manage their own data centers but instead run their entire Web-based services in the Amazon cloud, called **Amazon Web Services (AWS)**.
> 
> The "worker bees" in a data center are the hosts. They serve content (Web pages, videos), store e-mails and documents, and perform massively distributed computations. The hosts in a data center, called **blades** (resembling pizza boxes), are generally commodity hosts with CPU, memory, and disk storage. The hosts are stacked in **racks**, typically **20–40 blades per rack**. The racks are then interconnected using sophisticated, evolving data center network designs — discussed in greater detail in Chapter 6.

---

# 1.2.1 Access Networks

Having considered the applications and end systems at the "edge of the network," let's next consider the **access network** — the network that physically connects an end system to the first router (also called the **edge router**) on a path from the end system to any other distant end system. Figure 1.4 shows several types of access networks with the settings (home, enterprise, and wide-area mobile wireless) in which they are used:

- **Mobile Network** — cellular towers connecting mobile devices
- **Home Network** — residential broadband (DSL, cable, fiber, fixed wireless)
- **Enterprise Network** — corporate LANs with switches and routers

![[Pasted image 20260619124506.png]] _(Figure 1.4 — Access Networks: Mobile, Home, and Enterprise connecting to ISPs)_

> As of 2023, **more than 80% of households in Europe and the USA** have Internet access — although a "digital divide" in access exists around the world and among different demographics.

---

## What Is an Access Network?

An **access network** is the network that physically connects an end system to the first router (also called the **edge router**) on a path from the end system to any other distant end system.

> Note: The terms **hosts** and **end systems** are used interchangeably with clients/servers. Throughout this book, we use them all to mean the same thing.

---

## Home Access: DSL, Cable, FTTH, Fixed Wireless, and LEO Satellites

Today, the three most prevalent types of broadband residential access are **digital subscriber line (DSL)**, **cable**, and **fiber to the home (FTTH)** — with **fixed wireless** and **LEO satellite** access growing quickly as well.

### DSL — Digital Subscriber Line

**What it is:** One of the most common home broadband access technologies. Uses the existing **local telephone line** (from a telephone company like Telco) to provide Internet access.

**How it works:**

1. A customer obtains DSL access from the **same local telephone company** that provides phone service — so the telco is also the customer's ISP.
2. The customer's home has a **DSL modem** — an external device connected to the PC via Ethernet.
3. The modem uses the existing twisted-pair telephone line to exchange data with a **DSLAM** (DSL Access Multiplexer) in the Telco's central office (CO).
4. The home's DSL modem takes digital data and translates it to high-frequency tones for transmission over the phone wires to the CO; the DSLAM translates the analog signals from many such houses back into digital format.

**Key detail — The Splitter:**

A **splitter** is installed at the customer's home that separates:

- Data signals → sent to DSL modem
- Phone signals → sent to home telephone

![[Pasted image 20260619124719.png]] _(Figure 1.5 — DSL Internet Access: Modem, Splitter, DSLAM, Central Office)_

**DSL Uses Frequency-Division Multiplexing (FDM):**

The residential telephone line carries three channels simultaneously, encoded at different frequencies:

|Channel|Frequency Band|Direction|Purpose|
|---|---|---|---|
|High-speed downstream|50 kHz – 1 MHz|Toward home|Internet data down|
|Medium-speed upstream|4 kHz – 50 kHz|Away from home|Internet data up|
|Ordinary two-way telephone|0 – 4 kHz|Bidirectional|Phone calls|

This approach makes the single DSL link appear as if there were three separate links, so a phone call and an Internet connection can share the DSL link at the same time. (We'll describe FDM in Section 1.3.1.)

**DSL Speeds & Asymmetry:**

- Typical real-world deployment: **~12 Mbps downstream / ~1.8 Mbps upstream** (actual rates depend heavily on distance from the CO and line quality)
- The **most recent DSL standard (ITU DSL 2019)** provides for up to **1 Gbps downstream** over short distances and up to **500 Mbps upstream** transmission rates
- Because downstream and upstream rates differ, access is said to be **asymmetric**
- Achieved rates may be lower than the maximums, since providers may purposefully limit a residential rate when offering tiered service at different prices
- The maximum rate is also limited by the distance between the home and the CO and the gauge/quality of the twisted-pair line — generally, if the residence is not within **5–10 miles of the CO**, it must resort to an alternative form of Internet access

---

### Cable Internet Access

**What it is:** The second most common home broadband technology. Uses the cable television company's **existing cable television infrastructure**.

**How it works:**

1. A residence obtains cable Internet from the same cable TV company.
2. Uses a **cable modem** — external device connecting home PC to the cable system (typically via Ethernet, like a DSL modem).
3. The cable network uses a **hybrid fiber-coaxial (HFC)** access network:
    - **Fiber** connects the cable head end to neighborhood junctions
    - **Coaxial cable** from junctions to individual homes

**Cable Topology:**

![[Pasted image 20260513195217.png]] _(Figure 1.6 — Hybrid Fiber-Coaxial Access Network: Fiber backbone with coaxial last-mile to homes)_

**Each neighborhood junction typically supports 500 to 5,000 homes.** The cable modem terminates at the **cable head end**, where a device called **CMTS** (Cable Modem Termination System) serves a similar function to the DSLAM.

**Key Difference from DSL:**

Cable is a **shared broadcast medium**. Every packet sent by the head end travels downstream to every home, and every home's upstream packet travels to the head end.

- If multiple users download simultaneously, the downstream rate each user receives **drops** below the aggregate rate.
- If only a few users are active and mostly web-surfing, they may each receive the full downstream rate, since they rarely request a page at exactly the same time.
- Requires **multiple access protocols** to coordinate transmissions and avoid collisions (discussed in Chapter 6).

**Cable Speeds (DOCSIS standards):**

|Standard|Downstream|Upstream|
|---|---|---|
|**DOCSIS 2.0**|up to 40 Mbps (commonly cited as 42.8 Mbps)|up to 30 Mbps (commonly cited as 30.7 Mbps)|
|**DOCSIS 3.0**|up to **1.2 Gbps**|up to **100 Mbps**|

As with DSL, the maximum achievable rate may not be realized due to lower contracted data rates or media impairments. Access is typically asymmetric, with downstream allocated a higher rate than upstream.

---

### FTTH — Fiber to the Home

**What it is:** An up-and-coming home Internet technology providing even higher speeds than DSL/cable. An optical fiber path runs from the CO toward (and often directly into) the home, potentially providing Internet access rates in the **gigabits per second** range.

**Why it's better:**

- Unlike DSL (limited by copper) and cable (shared medium), fiber provides a much higher-capacity, less contended connection.

**Two competing optical-distribution architectures** split fiber from the CO out to homes:

|Architecture|Description|
|---|---|
|**AON** (Active Optical Network)|Essentially **switched Ethernet** (discussed in Chapter 6)|
|**PON** (Passive Optical Network)|Shared fiber split passively among many homes (detailed below) — used in Verizon's FiOS service|

**PON Architecture in Detail:**

![[Pasted image 20260513195334.png]] _(Figure 1.7 — FTTH Internet Access using PON architecture)_

- Each home has an **ONT** (Optical Network Terminator) — connected by dedicated fiber to a neighborhood **optical splitter**.
- The splitter combines a number of homes (typically **fewer than 100**) onto a single, shared optical fiber.
- This shared fiber connects to an **OLT** (Optical Line Terminator) in the CO.
- The OLT provides conversion between optical and electrical signals for Internet connection via a telco router.
- At home, users connect a home router (typically wireless) to the ONT and access the Internet via this home router.
- All packets sent from the OLT to the splitter are replicated at the splitter (similar to a cable head end).

**FTTH Deployment Status:**

- Fiber-to-the-home still represents a minority of broadband access overall, though it's expanding rapidly as deployment costs come down.
- The simplest optical distribution architecture is **direct fiber** — one fiber leaving the CO per home — but it's far less common than the shared/split architectures (AON/PON) above.

---

### Fixed Wireless Internet (FWI)

**What it is:** A newer residential access technology that delivers high-speed Internet **without** installing costly, failure-prone cabling from the telco's CO to the home.

**How it works:**

- Uses **5G fixed wireless** with **beam-forming technology**.
- Data is sent wirelessly from the provider's base station directly to a modem in the home.
- A WiFi wireless router connects to that modem — possibly bundled into a single device — similar to how a WiFi router connects to a cable or DSL modem.

**Why it matters:** It bypasses the "last mile" cabling problem entirely, making it attractive in areas where trenching fiber or copper is expensive, slow, or impractical.

---

### LEO Satellites (Home Access)

In addition to DSL, cable, FTTH, and FWI, **low-Earth-orbit (LEO) satellites** are increasingly used for broadband Internet access, particularly in rural and remote areas. Companies such as **SpaceX's Starlink** deploy large constellations of satellites, providing high-speed access with signal propagation delays **much lower** than geostationary satellites.

_(Full technical comparison of geostationary vs. LEO satellites — altitude, propagation delay, GPS — is covered under Physical Media → Satellite Radio Channels below.)_

---

### Other Home Access Technologies

**Dial-Up Access:**

- Traditional modem over standard telephone line.
- Extremely slow: ~56 kbps.
- Rarely used today where broadband is available.

**Satellite Access (general):**

- Users in rural areas without DSL/cable/fiber/fixed-wireless can use satellite links.
- Two main types: **geostationary** and **LEO** (details below).

---

## Access in Enterprise (and Home): Ethernet and WiFi

### Ethernet

**What it is:** The dominant access technology in corporate, university, and increasingly home networks.

**How it works:**

- End systems connect to an **Ethernet switch** via **twisted-pair copper wire**.
- The switch, in turn, connects to the **institutional router** (edge router), which connects to the ISP.
- With Ethernet access, users typically have **100 Mbps to tens of Gbps** access to the Ethernet switch, whereas **servers** may have **1 Gbps to 10 Gbps** access.

![[Pasted image 20260513195541.png]] _(Figure 1.8 — Ethernet Internet Access: Multiple devices to switch, switch to router, router to ISP)_

**Why Ethernet over twisted-pair?**

- Cheap, easy to install (just plug in a cable)
- No RF interference issues
- Widely supported by all manufacturers

---

### WiFi — Wireless LAN

**What it is:** Wireless access technology based on the **IEEE 802.11** standard, more colloquially known as **WiFi**.

**Coverage:**

- A wireless LAN user must typically be within a **few tens of meters** of the access point (AP).
- **802.11 today provides a shared transmission rate of up to more than 100 Mbps.**
- WiFi is now just about everywhere — universities, business offices, cafés, airports, homes, and even airplanes.

**Important note:**

A wireless LAN is often called a **WLAN**. A user transmits/receives packets to/from an access point that is connected into the enterprise's network (most likely using wired Ethernet), which in turn connects to the wired Internet. A wireless link is just the last connection; the institutional network behind it is wired.

**Typical Home Network:**

![[Pasted image 20260513195644.png]] _(Figure 1.9 — Typical Home Network: Devices to WiFi router, router to cable/DSL modem, modem to ISP)_

A home network typically consists of:

- A roaming laptop, multiple Internet-connected appliances, and a wired PC
- A **base station** (the wireless access point), which communicates with wireless devices in the home
- A **home router** that connects the wireless access point, and any other wired home devices, to the Internet

This network allows household members to have broadband Internet access with one member roaming from the kitchen to the backyard to the bedrooms.

---

## Wide-Area Wireless Access: 4G and 5G

**What it is:** Mobile devices such as iPhones and Android devices are used to message, share photos and video, make mobile payments, watch movies, stream music, video conference, and more — all while on the run. These devices employ the **same wireless infrastructure used for cellular telephony** to send/receive packets through a base station operated by the cellular network provider.

**Key difference from WiFi:**

- Mobile devices do **not** need to be near an access point — they communicate with a base station (cell tower).
- Unlike WiFi, a user need only be within a few **tens of kilometers** (as opposed to a few tens of meters) of the base station.
- Much more sophisticated frequency allocation and handoff protocols.

**Current Standards:**

|Standard|Speed|Status|
|---|---|---|
|**4G** (fourth generation)|Real-world download speeds of up to **~60 Mbps**|Deployed — telecoms have made enormous investments|
|**5G** (fifth generation)|Higher-speed wide-area wireless access|Already being deployed|

> _Earlier 3G/LTE reference (for historical context): 3G offered just over 1 Mbps; LTE offered 10+ Mbps. Both have since been superseded by 4G as the mainstream deployed standard, with 5G now rolling out._

We'll cover the basic principles of wireless networks and mobility, as well as WiFi, 4G, and 5G technologies (and more) in **Chapter 7**.

---

# 1.2.2 Physical Media

In Section 1.2.1, we gave an overview of access network technologies and the physical media used. Now we dive into the characteristics of transmission media. Examples of physical media include **twisted-pair copper wire, coaxial cable, multimode fiber-optic cable, terrestrial radio spectrum, and satellite radio spectrum**.

## Guided vs. Unguided Media

Physical media are broadly divided into two categories:

|Type|Description|Examples|
|---|---|---|
|**Guided Media**|Waves guided along a solid medium|Twisted-pair copper, coaxial cable, fiber-optic|
|**Unguided Media**|Waves propagate through atmosphere/outer space|Radio spectrum, satellite|

---

## Bit Propagation Through Physical Media

**Key concept:** When data travels from source to destination, it passes through a series of transmission-receiver pairs. A bit is transmitted, the first router receives it, transmits it, the second router receives it, and so on.

For each transmission-receiver pair:

- The source **transmits the bit** onto the physical medium.
- Shortly thereafter, the first router **receives the bit**.
- The first router then **transmits the bit** to the second router.
- And so on...

The physical medium takes many shapes and doesn't have to be the same type for each transmission-receiver pair.

---

## Twisted-Pair Copper Wire

**Cost consideration:**

The actual cost of physical link installation (labor, conduit, etc.) often exceeds the cost of the material. For this reason, it's common to install multiple types of media in the same physical path.

### Unshielded Twisted Pair (UTP)

**What it is:**

The **least expensive and most commonly used guided transmission medium**. Used for over a hundred years in telephone networks. In fact, **more than 99% of the wired connections from a telephone handset to the local telephone switch use twisted-pair copper wire.**

**Construction:**

- Two insulated copper wires — twisted together in a regular spiral pattern.
- The twist reduces electrical interference from adjacent wire pairs.
- Typically, multiple pairs are bundled together into a cable with protective shielding.
- One pair constitutes a single communication link.

**Data rates for LANs:**

Data rates for LANs using twisted pair today range from **10 Mbps to 10 Gbps**:

- **10 Mbps** — older installations
- **100 Mbps** — modern twisted pair (Category 5 cable)
- **1 Gbps** — high-speed twisted pair (Category 6+)
- **10 Gbps** — **Category 6a** cable, achievable for distances **up to a hundred meters**

**Why twisted pair for LANs but not higher speeds?**

When fiber-optic technology emerged in the 1980s, many people disparaged twisted pair because of its relatively low bit rates, and some felt fiber-optic technology would completely replace it. But twisted-pair technology itself kept evolving (Category 5, 5e, 6, 6a):

- Newer standards achieve up to 10 Gbps for distances up to 100 meters.
- Installation cost of copper is much lower than fiber.
- Twisted pair has emerged as the **dominant solution for high-speed LAN networking**.

**Also commonly used for:**

- **Residential broadband access** (DSL)
- **Dial-up modem access** over traditional phone lines (up to 56 kbps)

---

### Coaxial Cable

**Construction:**

- Two copper conductors, but **concentric rather than parallel**.
- Inner conductor is surrounded by insulation, then an outer conductor (shield), then a protective outer sheath.
- Can achieve higher data transmission rates than twisted pair through special insulation and shielding.

**Characteristics:**

- **Shared broadcast medium** — multiple end systems can be connected directly to the cable, with each receiving whatever is sent by others.
- Used extensively in **cable television systems**.
- In cable TV/Internet access, the transmitter shifts the digital signal to a specific frequency band, and the resulting analog signal is sent from the transmitter to one or more receivers.

**Data Rates:**

- Coupled with cable modems, coaxial cable provides residential users with Internet access at rates of **hundreds of Mbps**.
- **HFC networks** (Hybrid Fiber-Coaxial) use coaxial cable for the last segment to homes.

---

## Fiber Optics

**What it is:**

A thin, flexible medium that **conducts pulses of light**, with each pulse representing a bit.

**Characteristics:**

- A single optical fiber can support tremendously high bit rates — up to **tens or hundreds of gigabits per second**.
- **Immune to electromagnetic interference** — light pulses are not affected by EM noise.
- Very **low signal attenuation** — can travel distances up to **100 km** without needing amplification, making fiber the preferred medium for long-haul links (especially overseas).
- **Very hard to tap** — extremely difficult to tap into a fiber line without detection (security advantage).

**Optical Carrier (OC) Standards:**

Fiber link speeds are often specified using the OC-_n_ standard, where the link speed equals **n × 51.8 Mbps**:

|Designation|Speed|
|---|---|
|OC-1|51.8 Mbps|
|OC-3|155.4 Mbps|
|OC-12|622.1 Mbps|
|OC-24|1.24 Gbps|
|OC-48|2.49 Gbps|
|OC-96|4.98 Gbps|
|OC-192|9.95 Gbps|
|OC-768|39.8 Gbps|

> Overall range: **51.8 Mbps to 39.8 Gbps**.

**Cost Limitation:**

The **very high cost of optical devices** — transmitters, receivers, switches — has hindered fiber deployment in short-haul applications (e.g., within a LAN or into a home).

**Where fiber IS deployed:**

- **Long-distance telephone networks** throughout the US and elsewhere.
- **Internet backbone** — the core of the Internet uses fiber extensively.
- **Fiber-to-the-home (FTTH)** — increasingly deployed for residential access (via AON or PON).

> Fiber optics are the preferred long-distance transmission media and are now prevalent in the Internet backbone. However, the high cost of optical devices has limited their deployment for short-haul transport.

---

## Terrestrial Radio Channels

**What they are:**

Radio channels carry signals through the **electromagnetic spectrum**. They are an attractive medium because:

- No physical wire installation needed
- Can penetrate walls
- Carry signals for long distances
- Potentially carry a signal for long time periods

**Environmental Effects on Radio:**

Radio channel characteristics depend on:

- **Path loss** — signal strength decreases as it travels distance
- **Shadow fading** — signal blocked by large obstacles (buildings, terrain)
- **Multipath fading** — signal reflects off obstacles, arriving at receiver via multiple paths with different delays
- **Interference** — signal collides with transmissions from other sources

### Radio Channels by Distance/Area

Terrestrial radio channels are broadly classified into three groups:

|Group|Coverage|Examples|Use Case|
|---|---|---|---|
|**Very short distance**|Meter scale|Remote controls, medical implants, wireless headsets|Personal area|
|**Local area**|Tens–hundreds of meters|WiFi (802.11), Bluetooth|LAN, home networks|
|**Wide area**|Kilometers|Cellular (4G, 5G)|Mobile communication|

**Details on each:**

**Personal Devices (very short range):**

- Wireless headsets, keyboards, mice operate over very short distances.
- Medical devices operate within a few meters.
- Example: Bluetooth, Zigbee protocols (not covered in this course).

**Wireless LANs (local area):**

- **IEEE 802.11** technologies (WiFi) operate in local areas.
- Span from ~10 to ~100 meters.
- Provide a shared transmission rate of up to more than 100 Mbps.

**Cellular/Wide-Area Networks (long range):**

- **4G** and **5G** wide-area wireless access technologies.
- Base stations managed by cellular network providers.
- 4G provides real-world download speeds up to ~60 Mbps; 5G offers higher-speed access and is being actively deployed.
- Discussed in detail in Chapter 7.

---

## Satellite Radio Channels

**What they are:**

A communication satellite links two or more **Earth-based microwave transmitter/receiver** ground stations. The satellite:

1. Receives transmissions on one frequency band
2. Regenerates the signal using a repeater
3. Transmits the signal on another frequency

**Two types of satellites:**

### Geostationary Satellites

- **Altitude:** 36,000 km above Earth's surface
- **Coverage:** Remain above the same spot on Earth (stationary relative to Earth)
- **Propagation delay:** ~280 milliseconds (satellite↔ground) — a substantial delay, quite large for interactive applications like VoIP, gaming, video calls
- **Bandwidth:** Satellite links can operate at speeds of hundreds of Mbps
- **Usage:** Often used in areas without access to DSL or cable-based Internet access
- **Also:** Geostationary satellites are the central component of the **Global Positioning System (GPS)**, on which many location-based Internet applications rely

---

### Low-Earth Orbiting (LEO) Satellites

- **Altitude:** Much closer to Earth than geostationary satellites
- **Orbit:** Continuously rotate around Earth (just as the Moon does) and do not remain permanently above one spot
- **Communication:** May communicate with each other, as well as with ground stations
- **Propagation delay:** Much lower round-trip delay than geostationary — about **10 milliseconds**
- **Current status:** Several LEO constellation systems in development/ deployment (e.g., **SpaceX's Starlink**, **Project Kuiper**)
- **Future potential:** Increasingly used for Internet access, particularly in rural and remote areas not easily served by land-based infrastructure

> Geostationary and LEO satellites remain above Earth at different altitudes and have different characteristics. LEO satellite technology is an active area of research and development for providing global Internet coverage.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Shared media (cable)**|Easy to sniff packets in HFC networks if not encrypted|Require HTTPS/TLS; segment networks|
|**Wireless (WiFi/4G/5G)**|RF signals can be intercepted; weaker protocols vulnerable to eavesdropping|Use strong encryption (WPA3 for WiFi); VPN for cellular|
|**Fixed Wireless (5G beamforming)**|Directional beams are harder to intercept off-axis, but still RF — base-station spoofing remains a risk|Mutual authentication between modem and base station; encryption|
|**Fiber optics**|Very hard to tap without detection|Still encrypt end-to-end; physical security important|
|**DSL line**|Vulnerable if not encrypted between modem and ISP|Encrypt traffic; use VPN|
|**Satellite / GPS**|Signal jamming or spoofing of GPS affects location-based apps|Use multi-source location verification; signal authentication|
|**Cloud / Data centers**|Shared infrastructure (multi-tenant) widens attack surface|Shared-responsibility security model; access control, isolation|
|**Access network design**|Understand topology to find weak points|Know your topology; segment appropriately|

---

## Questions I Still Have

- [ ] How exactly does FDM (frequency-division multiplexing) work in DSL? How are frequencies assigned?
- [ ] What is the latency difference between cable and fiber in practice?
- [ ] How does WiFi collision detection work when multiple devices transmit simultaneously?
- [ ] Why is the upstream channel slower than downstream in cable/DSL?
- [ ] How do base stations handle handoff when a mobile device moves between towers?
- [ ] How does 5G beam-forming actually target a specific home for fixed wireless?
- [ ] How is GPS accuracy affected by atmospheric conditions vs. satellite geometry?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**End system / Host**|Device at the edge of the Internet that runs applications (host = end system)|
|**Access network**|Network connecting end system to first router (edge router)|
|**Client**|Lower-powered end system (PC, phone, tablet)|
|**Server**|Powerful end system storing and serving content|
|**Data center**|Facility housing large numbers of servers ("blades") in racks|
|**Blade**|Commodity host (CPU, memory, disk) stacked in data center racks|
|**Cloud computing**|Renting data-center infrastructure/services from a provider (e.g., AWS)|
|**DSL**|Digital Subscriber Line — uses existing phone lines for broadband|
|**DSLAM**|DSL Access Multiplexer — device in CO that separates DSL signals|
|**Cable modem**|Device that enables cable TV company infrastructure for Internet|
|**HFC**|Hybrid Fiber-Coaxial — fiber backbone + coaxial last-mile|
|**CMTS**|Cable Modem Termination System — device at cable head end|
|**DOCSIS**|Cable data-over-cable standard; 2.0 and 3.0 define current speed tiers|
|**FTTH**|Fiber To The Home — dedicated/shared optical fiber to home|
|**AON**|Active Optical Network — essentially switched Ethernet for FTTH|
|**PON**|Passive Optical Network — shared fiber from CO to <100 homes|
|**ONT**|Optical Network Terminator — home equipment for FTTH|
|**OLT**|Optical Line Terminator — CO equipment for FTTH|
|**FWI**|Fixed Wireless Internet — 5G beam-forming access from base station to home modem|
|**Ethernet**|Dominant wired LAN technology using twisted-pair copper|
|**WiFi**|IEEE 802.11 wireless LAN technology|
|**4G/5G**|Cellular wide-area wireless access technologies|
|**Physical media**|Transmission medium (guided or unguided)|
|**Guided media**|Waves guided along solid medium (copper, fiber)|
|**Unguided media**|Waves propagate through air/space (radio, satellite)|
|**Twisted pair**|Two insulated copper wires twisted together; cheapest guided medium|
|**UTP**|Unshielded twisted pair — most common LAN media|
|**Coaxial cable**|Two concentric copper conductors; used in cable TV and HFC|
|**Fiber optic**|Light pulses in thin glass fiber; very high speed, secure|
|**OC-n**|Optical Carrier standard; speed = n × 51.8 Mbps|
|**Radio channel**|Electromagnetic spectrum transmission; no wire needed|
|**Terrestrial radio**|Radio transmission through atmosphere (WiFi, cellular)|
|**Satellite link**|Transmission via orbiting satellites (geostationary or LEO)|
|**GPS**|Global Positioning System; relies on geostationary satellites|
|**Propagation delay**|Time it takes a signal to travel from sender to receiver|

---

## Summary Table — Access Technologies Comparison

|Technology|Medium|Speed (downstream)|Asymmetric?|Coverage|Cost|Use Case|
|---|---|---|---|---|---|---|
|**Dial-up**|Phone line|56 kbps|No|Point-to-point|Low|Legacy (rarely used)|
|**DSL**|Phone line|~12 Mbps typical; up to 1 Gbps (newest standard)|Yes|5–10 miles from CO|Medium|Residential|
|**Cable**|Coax/HFC|42.8 Mbps (DOCSIS 2.0); up to 1.2 Gbps (DOCSIS 3.0)|Yes|Shared segment (500–5,000 homes)|Medium|Residential|
|**FTTH**|Fiber (AON/PON)|Gbps range|No|Growing|High|Residential (new)|
|**Fixed Wireless (FWI)**|5G beam-formed radio|High-speed|—|No CO-to-home cabling needed|Medium|Residential (new)|
|**Ethernet**|Twisted pair|100 Mbps – tens of Gbps (clients); 1–10 Gbps (servers)|No|~100 meters|Low|Enterprise/LAN|
|**WiFi**|Radio (802.11)|>100 Mbps|No|10–20 meters|Low|Home/office wireless|
|**4G/5G**|Radio (cellular)|4G: up to ~60 Mbps real-world; 5G: higher|No|Tens of kilometers|Medium|Mobile devices|
|**Satellite (LEO)**|Radio (LEO)|High-speed|Varies|Global, rural/remote|High|Rural areas (e.g., Starlink)|
|**Satellite (GEO)**|Radio (geostationary)|Hundreds Mbps|Varies|Global|High|Rural areas, GPS|

---

## Related Concepts

- 

---

→ Next: [[THE NETWORK CORE]]