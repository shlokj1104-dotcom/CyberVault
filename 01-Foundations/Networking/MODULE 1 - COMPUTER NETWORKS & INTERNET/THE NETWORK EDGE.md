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

> **One-Line Summary:** The network edge consists of end systems (clients and 
> servers), access networks that connect them to ISPs, and physical media that 
> carry signals — from DSL and cable to fiber optics, WiFi, and wireless 
> cellular technologies.

---

# 1.2.1 Access Networks

Having considered the high-level Internet architecture, we now zoom in on the 
components between end systems and the network core.

## What Is an Access Network?

An **access network** is the network that physically connects an end system to 
the first router (also called the **edge router**) on a path from the end system 
to any other distant end system.

Figure 1.4 shows several types of access networks:
- **Mobile Network** — cellular towers connecting mobile devices
- **Home Network** — residential broadband (DSL, cable, fiber)
- **Enterprise Network** — corporate LANs with switches and routers

![[Pasted image 20260513194806.png]]
*(Figure 1.4 — Access Networks: Mobile, Home, and Enterprise connecting to ISPs)*

---

## Clients vs. Servers

Hosts are sometimes further divided into two categories:

| Type | Characteristics | Examples |
|---|---|---|
| **Clients** | Desktop and mobile PCs, smartphones, tablets | Tend to be lower-powered, user-facing devices |
| **Servers** | More powerful machines that store and distribute data | Web servers, email servers, video streaming servers |

> Note: The terms **hosts** and **end systems** are used interchangeably with 
> clients/servers. Throughout this book, we use them all to mean the same thing.

Today, most servers live in large **data centers**. For example, Google operates 
30–50 data centers with many having 100,000+ servers each.

---

## Home Access: DSL, Cable, FTTH, Dial-Up, Satellite

### DSL — Digital Subscriber Line

**What it is:** One of the two most common home broadband access technologies. 
Uses the existing **local telephone line** (from a telephone company like Telco) 
to provide Internet access.

**How it works:**

1. A customer obtains DSL access from the **same local telephone company** that 
   provides phone service.
2. The customer's home has a **DSL modem** — an external device connected to 
   the PC via Ethernet.
3. The modem uses the existing twisted-pair telephone line to communicate with a 
   **DSLAM** (DSL Access Multiplexer) in the Telco's central office (CO).
4. The DSLAM separates the data and phone signals — translates digital data back 
   to high-frequency tones for transmission over telephone wires.

**Key detail — The Splitter:**

A **splitter** is installed at the customer's home that separates:
- Data signals → sent to DSL modem
- Phone signals → sent to home telephone

![[Pasted image 20260513195025.png]]
*(Figure 1.5 — DSL Internet Access: Modem, Splitter, DSLAM, Central Office)*

**DSL Uses Frequency-Division Multiplexing (FDM):**

The residential telephone line carries three channels simultaneously, encoded at 
different frequencies:

| Channel | Frequency Band | Direction | Purpose |
|---|---|---|---|
| High-speed downstream | 50 kHz – 1 MHz | Toward home | Internet data down |
| Medium-speed upstream | 4 kHz – 50 kHz | Away from home | Internet data up |
| Ordinary two-way telephone | 0 – 4 kHz | Bidirectional | Phone calls |

This approach makes the single DSL link appear as if there are three separate 
channels.

**DSL Speeds & Asymmetry:**

- **Downstream** (to home): ~12 Mbps
- **Upstream** (from home): ~1.8 Mbps
- Actual speeds depend on distance from CO and line quality.
- The access is **asymmetric** — download rates exceed upload rates.
- If a residence is more than 5–10 miles from the CO, must use alternative access.

---

### Cable Internet Access

**What it is:** The second most common home broadband technology. Uses the cable 
television company's **existing cable television infrastructure**.

**How it works:**

1. A residence obtains cable Internet from the same cable TV company.
2. Uses a **cable modem** — external device connecting home PC to the cable system.
3. The cable network uses a **hybrid fiber-coaxial (HFC)** access network:
   - **Fiber** connects the cable head end to neighborhood junctions
   - **Coaxial cable** from junctions to individual homes

**Cable Topology:**

![[Pasted image 20260513195217.png]]
*(Figure 1.6 — Hybrid Fiber-Coaxial Access Network: Fiber backbone with coaxial 
last-mile to homes)*

Hundreds to thousands of homes connect to a single coaxial cable segment. The 
cable modem terminates are at the **cable head end**, where a device called 
**CMTS** (Cable Modem Termination System) serves a similar function to DSLAM.

**Key Difference from DSL:**

Cable is a **shared broadcast medium**. Every packet sent by the head end travels 
to every home, and every home's upstream packet travels to the head end.

- If multiple users download simultaneously, the downstream rate **drops**.
- If upstream traffic is heavy, upstream channel **saturates**.
- Requires **multiple access protocols** to coordinate transmissions (discussed 
  in Chapter 5).

**Cable Speeds:**

- **Downstream**: up to 42.8 Mbps (DOCSIS 2.0 standard)
- **Upstream**: up to 30.7 Mbps
- Actual rates depend on number of active users and contracted service tier.

---

### FTTH — Fiber to the Home

**What it is:** The newest and fastest home Internet technology. An optical 
fiber path runs directly from the central office (CO) to the home.

**Why it's better:**

- Supports very high rates — gigabits per second range.
- Unlike DSL (limited by copper) and cable (shared medium), fiber provides 
  dedicated, point-to-point connection.

**FTTH Architecture (PON):**

The simplest optical distribution architecture is **direct fiber** — one fiber 
per home from CO. More common: **Passive Optical Network (PON)** —

![[Pasted image 20260513195334.png]]
*(Figure 1.7 — FTTH Internet Access using PON architecture)*

- Each home has an **ONT** (Optical Network Terminator) — connected by dedicated 
  fiber to a neighborhood **optical splitter**.
- The splitter combines ~100 homes onto a single, shared optical fiber.
- This shared fiber connects to an **OLT** (Optical Line Terminator) in the CO.
- The OLT provides conversion between optical signals and electrical signals for 
  Internet connection via router.

**FTTH Performance:**

- Can provide rates in gigabits per second range.
- Average FTTH downstream speed: ~20 Mbps in 2011 (vs 13 Mbps for cable, 5 Mbps 
  for DSL).
- Verizon's FIOS service is a well-known FTTH provider.

**Deployment Status:**

- Fiber-to-home still represents <10% of broadband access in the US.
- Higher deployment costs limit uptake — but expanding rapidly.

---

### Other Home Access Technologies

**Dial-Up Access:**
- Traditional modem over standard telephone line.
- Extremely slow: ~56 kbps.
- Rarely used today where broadband is available.

**Satellite Access:**
- Users in rural areas without DSL/cable/fiber can use satellite links.
- One satellite type: **geostationary satellite** — remains above same spot on 
  Earth (36,000 km altitude).
- Propagation delay: ~280 milliseconds (satellite↔ground).
- Speed: hundreds of Mbps, but high latency makes it poor for interactive apps 
  (like gaming, video calls).
- **LEO satellites** (Low Earth Orbit) orbit much closer and may improve 
  latency — still in development.

---

## Access in Enterprise (and Home): Ethernet and WiFi

### Ethernet

**What it is:** The dominant access technology in corporate, university, and 
increasingly home networks.

**How it works:**

- End systems connect to an **Ethernet switch** via **twisted-pair copper wire**.
- The switch, in turn, connects to the **institutional router** (edge router), 
  which connects to the ISP.
- Typically provides **100 Mbps** access to the switch (though newer Gigabit 
  Ethernet provides 1 Gbps).

![[Pasted image 20260513195541.png]]
*(Figure 1.8 — Ethernet Internet Access: Multiple devices to switch, switch to 
router, router to ISP)*

**Why Ethernet over twisted-pair?**

- Cheap, easy to install (just plug in a cable)
- No RF interference issues
- Widely supported by all manufacturers

---

### WiFi — Wireless LAN

**What it is:** Wireless access technology based on **IEEE 802.11** standard.

**Coverage:**
- Users can be within ~10–20 meters of a WiFi base station (AP — Access Point).
- Provides shared transmission rate up to 54 Mbps (older 802.11g).
- Works everywhere today — universities, offices, cafés, airports, homes.

**Important note:**

A wireless LAN is often called a **WLAN**. A user in a WLAN must be within a 
few tens of meters of the access point and **still ultimately connect to a 
wired network** (typically Ethernet) that leads to the Internet. A wireless 
link is just the last connection; the institutional network behind it is wired.

**Typical Home Network:**

![[Pasted image 20260513195644.png]]
*(Figure 1.9 — Typical Home Network: Devices to WiFi router, router to cable/DSL 
modem, modem to ISP)*

A home network typically consists of:
- A **WiFi base station/router** (provides WiFi coverage + wired router function)
- **Cable modem** or **DSL modem** (bridges home network to ISP)
- Devices can roam throughout home — WiFi signal in kitchen, bedroom, backyard

---

## Wide-Area Wireless Access: 3G and LTE

**What it is:** Mobile users with iPhones, Android phones, and tablets use the 
same wireless infrastructure deployed for cellular telephony.

**Key difference from WiFi:**

- Mobile devices **do not need to be near an access point** — they communicate 
  with a **base station** (cell tower) operated by the cellular network provider.
- Base station covers a much wider area than WiFi (several kilometers).
- Much more sophisticated frequency allocation and handoff protocols.

**Current Standards:**

| Standard | Speed | Status |
|---|---|---|
| **3G** (3rd generation) | Over 1 Mbps | Deployed |
| **LTE** (Long-Term Evolution) | 10+ Mbps | Deployed, expanding |
| **4G** (4th generation) | Speeds in gigabits/second range | Candidate standard |

Telecommunications companies have invested enormously in 3G wireless. LTE is an 
evolution of 3G and represents the next step (before full 4G). We'll discuss 
wireless technologies in detail in Chapter 6.

---

# 1.2.2 Physical Media

In Section 1.2.1, we gave an overview of access network technologies and the 
physical media used. Now we dive into the characteristics of transmission media.

## Guided vs. Unguided Media

Physical media are broadly divided into two categories:

| Type | Description | Examples |
|---|---|---|
| **Guided Media** | Waves guided along a solid medium | Twisted-pair copper, coaxial cable, fiber-optic |
| **Unguided Media** | Waves propagate through atmosphere/outer space | Radio spectrum, satellite |

---

## Bit Propagation Through Physical Media

**Key concept:** When data travels from source to destination, it passes through 
a series of transmission-receiver pairs. A bit is transmitted, the first router 
receives it, transmits it, the second router receives it, and so on.

For each transmission-receiver pair:
- The source **transmits the bit** onto the physical medium.
- Shortly thereafter, the first router **receives the bit**.
- The first router then **transmits the bit** to the second router.
- And so on...

The physical medium takes many shapes and doesn't have to be the same type for 
each transmission-receiver pair.

---

## Twisted-Pair Copper Wire

**Cost consideration:**

The actual cost of physical link installation (labor, conduit, etc.) often 
exceeds the cost of the material. For this reason, it's common to install 
multiple types of media in the same physical path.

### Unshielded Twisted Pair (UTP)

**What it is:**

The **least expensive and most commonly used guided transmission medium**. Used 
for over a hundred years in telephone networks.

**Construction:**

- Two insulated copper wires — twisted together in a regular spiral pattern.
- The twist reduces electrical interference from adjacent wire pairs.
- Typically, multiple pairs are bundled together into a cable with protective 
  shielding.
- One pair constitutes a single communication link.

**Data rates for LANs:**

- **10 Mbps** — older installations
- **100 Mbps** — modern twisted pair (Category 5 cable)
- **1 Gbps** — newer high-speed twisted pair (Category 6+)

**Why twisted pair for LANs but not higher speeds?**

When fiber-optic technology emerged in the 1980s, people expected twisted pair 
would be completely replaced due to its "low" rates. However:

- Twisted-pair technology itself evolved (Category 5, 5e, 6).
- Newer standards achieved 10 Gbps for distances up to 100 meters.
- Installation cost of copper is much lower than fiber.
- Twisted pair remains the dominant solution for **high-speed LAN networking**.

**Also commonly used for:**
- **Residential broadband access** (DSL)
- **Dial-up modem access** over traditional phone lines

---

### Coaxial Cable

**Construction:**

- Two copper conductors, but **concentric rather than parallel**.
- Inner conductor is surrounded by insulation, then an outer conductor (shield), 
  then protective outer sheath.
- Can achieve higher data transmission rates than twisted pair.

**Characteristics:**

- **Shared broadcast medium** — multiple end systems can be connected directly to 
  the cable, with each receiving whatever is sent by others.
- Used extensively in **cable television systems**.
- Recently coupled with cable modems to provide residential Internet access.

**Cable TV to Internet Access:**

- Shift the digital signal to a specific **frequency band**.
- Transmit the modulated signal from transmitter to one or more receivers.
- The transmitter-side signal is **shared** — every home on the cable segment 
  receives whatever is sent on that frequency.

**Data Rates:**

- **HFC networks** (Hybrid Fiber-Coaxial) use coaxial cable for the last segment 
  to homes.
- Transmission rates: **tens of Mbps** for residential Internet.

---

## Fiber Optics

**What it is:**

A thin, flexible medium that **conducts pulses of light**, with each pulse 
representing a bit.

**Characteristics:**

- Single optical fiber can support **tremendously high bit rates** — up to tens 
  or hundreds of gigabits per second.
- **Immune to electromagnetic interference** — light pulses are not affected by 
  EM noise.
- Have **very low signal attenuation** — can travel distances exceeding 100 km 
  without signal degradation.
- **Very hard to tap** — extremely difficult to tap into a fiber line without 
  detection (security advantage).

**Cost Limitation:**

The **very high cost of optical devices** — transmitters, receivers, switches — 
has hindered fiber deployment in short-haul applications (e.g., within a LAN or 
into a home).

**Where fiber IS deployed:**

- **Long-distance telephone networks** throughout the United States and elsewhere.
- **Internet backbone** — the core of the Internet uses fiber extensively.
- **Fiber-to-the-home (FTTH)** — increasingly deployed for residential access.

> Fiber optics are the preferred long-distance transmission media and are now 
> prevalent in the Internet backbone. However, the high cost of optical 
> devices—transmitters, receivers, switches—has limited their deployment for 
> short-haul transport.

---

## Terrestrial Radio Channels

**What they are:**

Radio channels carry signals through the **electromagnetic spectrum**. They are 
an attractive medium because:
- No physical wire installation needed
- Can penetrate walls
- Carry signals for long distances
- Potentially carry a signal for long time periods

**Environmental Effects on Radio:**

Radio channel characteristics depend on:
- **Path loss** — signal strength decreases as it travels distance
- **Shadow fading** — signal blocked by large obstacles (buildings, terrain)
- **Multipath fading** — signal reflects off obstacles, arriving at receiver 
  via multiple paths with different delays
- **Interference** — signal collides with transmissions from other sources

### Radio Channels by Distance/Area

Terrestrial radio channels are broadly classified into three groups:

| Group | Coverage | Examples | Use Case |
|---|---|---|---|
| **Very short distance** | Meter scale | Remote controls, medical implants, wireless headsets | Personal area |
| **Local area** | Tens–hundreds of meters | WiFi (802.11), Bluetooth | LAN, home networks |
| **Wide area** | Kilometers | Cellular (3G, LTE, 4G) | Mobile communication |

**Details on each:**

**Personal Devices (very short range):**
- Wireless headsets, keyboards, mice operate over very short distances.
- Medical devices operate within a few meters.
- Example: Bluetooth, Zigbee protocols (not covered in this course).

**Wireless LANs (local area):**
- **IEEE 802.11** technologies (WiFi) operate in local areas.
- Span from ~10 to ~100 meters.
- Provide data rates from a few Mbps to over 50 Mbps.
- Described in detail in Section 1.2.1.

**Cellular/Wide-Area Networks (long range):**
- **3G**, **LTE**, **4G** wide-area wireless access technologies.
- Base stations managed by cellular network providers.
- Provide packet-switched wide-area access at speeds exceeding 1 Mbps.
- Will be discussed in detail in Chapter 6.

---

## Satellite Radio Channels

**What they are:**

A communication satellite links two or more **Earth-based microwave transmitter/
receiver** ground stations. The satellite:
1. Receives transmissions on one frequency band
2. Regenerates the signal using a repeater
3. Transmits the signal on another frequency

**Two types of satellites:**

### Geostationary Satellites

- **Altitude:** 36,000 km above Earth's surface
- **Coverage:** Remain above the same spot on Earth (stationary relative to Earth)
- **Propagation delay:** ~280 milliseconds (satellite↔ground)
  - This is quite large for interactive applications like VoIP, gaming, video 
    calls
- **Bandwidth:** Satellite links can operate at hundreds of Mbps
- **Usage:** Often in areas without access to DSL or cable broadband

---

### Low-Earth Orbiting (LEO) Satellites

- **Altitude:** Much closer to Earth than geostationary satellites
- **Orbit:** Continuously move around Earth
- **Rotation:** Rotate around Earth (unlike geostationary which appear stationary)
- **Advantage:** Lower propagation delay than geostationary
- **Challenge:** Communicate with each other and ground stations
- **Current status:** Several LEO constellation systems in development (e.g., 
  Starlink, Project Kuiper)
- **Future potential:** May provide continuous satellite-based Internet access

> Geostationary and LEO satellites remain above Earth at different altitudes and 
> have different characteristics. LEO satellite technology is an active area of 
> research and development for providing global Internet coverage.

---

# Why It Matters for Security

| Concept | Attacker's Perspective | Defender's Perspective |
|---|---|---|
| **Shared media (cable)** | Easy to sniff packets in HFC networks if not encrypted | Require HTTPS/TLS; segment networks |
| **Wireless (WiFi/3G)** | RF signals can be intercepted; weaker protocols vulnerable to eavesdropping | Use strong encryption (WPA3 for WiFi); VPN for cellular |
| **Fiber optics** | Very hard to tap without detection | Still encrypt end-to-end; physical security important |
| **DSL line** | Vulnerable if not encrypted between modem and ISP | Encrypt traffic; use VPN |
| **Access network design** | Understand topology to find weak points | Know your topology; segment appropriately |

---

## Questions I Still Have

- [ ] How exactly does FDM (frequency-division multiplexing) work in DSL? How 
      are frequencies assigned?
- [ ] What is the latency difference between cable and fiber in practice?
- [ ] How does WiFi collision detection work when multiple devices transmit 
      simultaneously?
- [ ] Why is the upstream channel slower than downstream in cable/DSL?
- [ ] How do base stations handle handoff when a mobile device moves between 
      towers?

---

## Key Terms — Quick Reference

| Term | Definition |
|---|---|
| **Access network** | Network connecting end system to first router (edge router) |
| **Client** | Lower-powered end system (PC, phone, tablet) |
| **Server** | Powerful end system storing and serving content |
| **DSL** | Digital Subscriber Line — uses existing phone lines for broadband |
| **DSLAM** | DSL Access Multiplexer — device in CO that separates DSL signals |
| **Cable modem** | Device that enables cable TV company infrastructure for Internet |
| **HFC** | Hybrid Fiber-Coaxial — fiber backbone + coaxial last-mile |
| **CMTS** | Cable Modem Termination System — device at cable head end |
| **FTTH** | Fiber To The Home — dedicated optical fiber to home |
| **ONT** | Optical Network Terminator — home equipment for FTTH |
| **OLT** | Optical Line Terminator — CO equipment for FTTH |
| **PON** | Passive Optical Network — shared fiber from CO to ~100 homes |
| **Ethernet** | Dominant wired LAN technology using twisted-pair copper |
| **WiFi** | IEEE 802.11 wireless LAN technology |
| **3G/LTE** | Cellular wide-area wireless access technologies |
| **Physical media** | Transmission medium (guided or unguided) |
| **Guided media** | Waves guided along solid medium (copper, fiber) |
| **Unguided media** | Waves propagate through air/space (radio, satellite) |
| **Twisted pair** | Two insulated copper wires twisted together; cheapest guided medium |
| **UTP** | Unshielded twisted pair — most common LAN media |
| **Coaxial cable** | Two concentric copper conductors; used in cable TV and HFC |
| **Fiber optic** | Light pulses in thin glass fiber; very high speed, secure |
| **Radio channel** | Electromagnetic spectrum transmission; no wire needed |
| **Terrestrial radio** | Radio transmission through atmosphere (WiFi, cellular) |
| **Satellite link** | Transmission via orbiting satellites (geostationary or LEO) |
| **Propagation delay** | Time it takes a signal to travel from sender to receiver |

---

## Summary Table — Access Technologies Comparison

| Technology | Medium | Speed (downstream) | Asymmetric? | Coverage | Cost | Use Case |
|---|---|---|---|---|---|---|
| **Dial-up** | Phone line | 56 kbps | No | Point-to-point | Low | Legacy (rarely used) |
| **DSL** | Phone line | 12 Mbps | Yes | 5–10 miles from CO | Medium | Residential |
| **Cable** | Coax/HFC | 42 Mbps | Yes | Shared segment | Medium | Residential |
| **FTTH** | Fiber | Gbps | No | Growing | High | Residential (new) |
| **Ethernet** | Twisted pair | 100 Mbps – 1 Gbps | No | ~100 meters | Low | Enterprise/LAN |
| **WiFi** | Radio (802.11) | ~54 Mbps | No | 10–20 meters | Low | Home/office wireless |
| **3G/LTE** | Radio (cellular) | 1–10+ Mbps | No | Kilometers | Medium | Mobile devices |
| **Satellite** | Radio (LEO/GEO) | Hundreds Mbps | Varies | Global | High | Rural areas |

---

## Related Concepts

- [[XX]]

---

→ Next: [[THE NETWORK CORE]]