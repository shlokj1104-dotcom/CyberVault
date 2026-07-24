---
title: INTRODUCTION TO WIRELESS LINK
date: 2026-07-24
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# Chapter 7: Wireless and Mobile Networks

## 7.1 Introduction

> **One-Line Summary:** Every **wireless network** — whether a cellular network, a WiFi LAN, or an ad hoc network — can be decomposed into a common **taxonomy (classification scheme)** of elements: a **wireless access network** (the outermost edge, where devices communicate over a radio channel with a **base station**), a **wireless core network** (the interior **infrastructure** of servers, storage, and routers that provides control-plane services like identity and mobility management, and data-plane connectivity to the wider Internet), and the **wireless devices** themselves — which may operate in either **infrastructure mode** (relying on that base station and core) or **ad hoc mode** (self-organizing, with no base station at all).

---

## Core Idea: A Breathtaking Fifteen Years, and a New Kind of Chapter

The past 15 years have witnessed a **breathtaking (astonishingly rapid)** increase in the deployment and use of wireless and mobile networks. In **2008**, the number of fixed (wired) broadband Internet subscribers was **approximately the same** as the number of broadband subscribers attached via wireless technologies. While both subscriber bases have grown since then, there are now (in 2025) **five times as many** Internet-connected wireless subscribers as wired subscribers. An estimated **60 percent** of website traffic was directed to **mobile** versus fixed devices in 2025 [Statista Mobile 2025].

The advantages of wireless access are **evident to all** — anywhere, anytime, **untethered (unattached, free of a physical cord)** access to the global Internet via a **highly portable, lightweight** device. Increasingly, devices such as gaming consoles, home appliances, security systems, watches, eyeglasses, cars, and wearable clothing are being wirelessly connected to the Internet. Indeed, some have **posited (put forward as a hypothesis)** that a **profoundly (deeply, fundamentally) important**, **fourth industrial revolution** [Schwab 2016] — **enabled** by wireless networks and **cyber-physical systems** that **integrate** the physical and digital worlds — is now **underway**.

### Why an Entire Chapter, and Why "Bottom-Up" This Time

The challenges posed by networking wireless and mobile devices are so **markedly different (noticeably distinct)** from those of traditional wired computer networks that an individual chapter **devoted to** the study of wireless and mobile networks (this chapter) is **warranted (justified, called for)**. In this chapter, both the **principles** and the **practice** of wireless and mobile networks will be covered. Section 7.1 sets the **"big picture" context**.

As a reader of this textbook, you're by now well aware that we're **huge fans** of a **top-down approach**. But **so many** aspects of wireless networking result from the **complex, time-varying, capacity-limited, and shared** nature of the **physical wireless channel** that an early **mastery (thorough command, deep competence)** of radio channels is needed to **practically** appreciate any other aspect of wireless networking. Thus, our in-depth study of wireless and mobile networks will begin, this one time, **bottom-up**:

|Section|What It Covers|
|---|---|
|**7.2**|Properties of **wireless radio channels**|
|**7.3**|The **first-hop wireless access network** — techniques for sharing the radio channel, network discovery and device attachment, frame transmission scheduling, energy conservation, and more|
|**7.4**|The wireless network **"core"** — control-plane services such as identity and mobility management, and data-plane connectivity to the larger Internet|
|**7.5**|**Device mobility** — a **unique (distinctive, one-of-a-kind)** feature of wireless networks, warranting an entire section of its own|
|**7.6**|**Bluetooth, satellite, and IoT** wireless networks (briefly)|
|**7.7**|Concludes the chapter (security in wireless networks is **deferred (postponed)** to the next chapter)|

### The Guiding Philosophy: Separating the "Why" from the "What"

Throughout Sections 7.2–7.4, this text will begin by **identifying and studying fundamental principles** that **transcend (rise above, extend beyond)** any particular networking technology, and will then **illustrate** these principles in practice in the context of **WiFi and cellular networks**. The **goal** here is to **separate out** the **"why"** and the **"how"** from the **"what."**

There are many books and other documents that describe **"what"** is done in a particular wireless network architecture. Indeed, there are (rather dry) **standards**, many of them **thousands of pages long**, describing the "what" in great detail. Sections 7.2–7.4 instead aim to provide a **foundation** to understand the "**why**" — why an architecture is structured a certain way, or why a protocol takes a particular approach to **accomplish a task**. This **deeper understanding** will also last **far beyond** the lifetime of any specific wireless technology, helping to **future-proof (safeguard against future obsolescence)** your knowledge of wireless and mobile networks!

---

## Elements of a Wireless Network

Figure 7.1 shows the **setting** for this discussion of wireless and mobile networks. The discussion will begin by keeping things **general enough** to cover a wide range of networks, including **5G cellular networks, WiFi local area networks (WLANs)**, and **ad hoc networks** including **Bluetooth and IoT networks**.

Several **key common elements** can be **identified** in a wireless network:

```
Fig -- Elements of a Wireless Network
────────────────────────────────────────
   User Devices          Servers,
  (phone,laptop,            Storage
   car, drone)                 │
       )))                     │
        │ radio                ▼
        ▼ channel      ┌───────────┐
   ┌──────────┐        │  Gateway  │
   │  Base     │────────│  (to the  │
   │  Station  │        │ Internet) │
   └──────────┘        └───────────┘
       │                     │
  <-- Wireless -->    <-- Wireless -->
     access                 CORE
    network                network
```

![[Pasted image 20260724133501.png]]

### 1. Wireless Access Networks

At the highest level, a wireless network has one or more **wireless access networks**. The **wireless access network** lies at the **outermost "edge"** of the entire network, connecting devices into a larger network via a **wireless radio channel**. An access network can be thought of as a **type of local area network**, as studied in Chapter 6: it has **limited geographic scope**, is under the control of a **single service provider**, and **primarily** provides a **link-layer service**.

|Network Type|What the Access Network Is Called|
|---|---|
|**Cellular networks**|The **Radio Access Network (RAN)**|
|**WiFi networks**|A **Wireless Local Area Network (WLAN)**|

### 2. Wireless Core Network

The **wireless core network** consists of **servers, storage, routers**, and (typically wired) **links** that lie between the devices in a provider's access network and the **external network**. The core network may be **geographically distributed**, e.g., with some parts **collocated (physically located together)** with infrastructure in the access network, and other parts located in **distant data centers**.

The core network **implements**:

- **Higher-level control-plane and management-plane services** (e.g., for **user and device identity**, **mobility**, and **access-security management**)
- **Data-plane services** that **forward datagrams** from devices in the access network to the larger Internet, and **vice versa (the other way around)**

In **cellular networks**, there is a **well-defined** core network operated by the **cellular network provider** — this will be studied in Section 7.4. **Unlike 4G/5G**, there is **no corresponding wireless core network for WiFi**. Instead, an **enterprise** uses the **same infrastructure** for its WLANs that it uses for its **wired LANs**. In this sense, an enterprise views WiFi as **just another link-layer technology** within its enterprise network.

### 3. Wireless Devices

A **wireless device** might be a **smartphone, tablet, or laptop**, or an **IoT device** such as a **sensor or appliance**. As in the case of wired networks, it is these devices that **host and run applications**. As shown in Figure 7.1, wireless devices **attach** to a wireless access network. Note that wireless devices **may or may not** be **mobile** — a topic to be **returned to** in Section 7.5.

### 4. Base Station

The **base station** is **arguably (with good reason, debatably but defensibly) the most important and most unique** part of a wireless network, as it has **no counterpart (equivalent)** in a wired network. A base station is **responsible for** sending and receiving packets to and from the wireless user devices that are **associated with** that base station. That is, **all** wireless devices that send packets into, or receive packets from, the Internet do so **via** a base station, which will **typically** be **responsible for** multiple associated wireless devices.

**What does "associated" actually mean?** When a wireless device is said to be "associated" with a base station, this means **(1)** the device is within the **wireless communication distance** of the base station, and **(2)** the device and base station have **executed a protocol** that has resulted in the base station **serving as a relay** of packets between the device and the larger Internet.

```
Fig -- Downlink vs Uplink Channel
────────────────────────────────────────
                Base Station
                 (gNB / eNB
                  / AP)
              ╱          ╲
     downstream/           upstream/
     downlink               uplink
     channel                channel
        ╱                       ╲
 Associated                 Associated
  Device 1                   Device 2

 "Associated": device is within radio
 range AND has completed a protocol
 exchange so the base station relays
 its packets to/from the Internet.
```

|Direction|Also Called|Description|
|---|---|---|
|**Base station → associated devices**|**Downstream** or **downlink** channel|Transmission of frames **from** the base station **to** the devices|
|**Associated devices → base station**|**Upstream** or **uplink** channel|Transmission of frames **from** the devices **to** the base station|

### A Bewildering Zoo of Acronyms for "Base Station"

- In **5G networks**, the base station is known as a **gNodeB**, **abbreviated gNB**, which stands for "**Next Generation Node B**" in the 5G standards.
- In **4G networks**, the base station is referred to as an **eNodeB**, **abbreviated eNB**, which stands for "**evolved Node B**."

These are **possibly** the **least intuitive, least self-explanatory, and most impenetrable (impossible to understand or make sense of)** acronyms in **all** of networking! For that reason, and **always preferring** to use **descriptive language** rather than **jargon (specialized, insider terminology)**, this text will refer to base stations in 4G and 5G networks simply as "**base stations**." However, if you're going to **dig into** the technical standards, keep in mind that "**eNB**" and "**gNB**" are **3GPP terminology** for "base station" in a 4G or 5G network, respectively.

- In **WiFi networks**, a base station is referred to as an **Access Point (AP)**. Since that terminology is more **descriptive** and in **common use**, this text will also **occasionally** refer to WiFi base stations as **APs**.

### Infrastructure Mode vs. Ad Hoc Mode

Devices **associated with a base station** are often referred to as operating in **infrastructure mode**, since **traditional network services** (e.g., **address assignment, identity and access privileges, and routing**) are **provided** by the network infrastructure to which a device is connected via the base station. In **ad hoc networks**, wireless devices have **no such infrastructure** with which to connect. In the **absence** of such infrastructure, the devices **themselves** must **self-organize** and **provide** for services such as **routing, address assignment, DNS-like name translation**, and more.

```
Fig -- Infrastructure Mode vs Ad Hoc Mode
────────────────────────────────────────
 INFRASTRUCTURE MODE (e.g. WiFi, 4G/5G,
 Bluetooth):
   Device <--> Base Station/AP <--> Net
   (addressing, identity, routing all
    PROVIDED by the network infra-
    structure)

 AD HOC MODE (no base station):
   Device <--> Device <--> Device
   (devices self-organize: provide
    their OWN routing, addressing,
    DNS-like name translation, etc.)
```

|Mode|Description|Examples|
|---|---|---|
|**Infrastructure mode**|Devices connect via a base station, which provides addressing, identity, and routing services|4G/5G cellular, WiFi (typically), Bluetooth|
|**Ad hoc mode**|No base station; devices must self-organize to provide their own routing, addressing, and naming services|Purely ad hoc networks|

WiFi networks **can operate** in **either** infrastructure or ad hoc mode, but this chapter will **assume throughout** that WiFi networks have a base station — i.e., operate in **infrastructure mode**. **Bluetooth networks**, which will be studied in Section 7.6, are an example of **purely ad hoc networks**. [Mohapatra 2004] provides a **detailed** study of ad hoc networks.

### The Challenge of Mobility

When a **mobile device** moves **beyond the range** of one base station and **into the range** of another, it will **change its point of attachment** to the larger network — i.e., **change the base station** with which it is associated — a process referred to as **handoff** or **handover**.

Such mobility raises **many challenging questions**, including:

- If a wireless device can move, how does one find the mobile device's **current location** in the network so that data can be **forwarded** to that device?
- How is **addressing performed**, given that a device can be in **one of many possible locations**?
- If the device moves **during** a TCP connection or a voice/video call, how are datagrams forwarded so that the connection **continues uninterrupted**?

The answers to these and many other mobility-related questions will be **explored** in Section 7.5.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**The shared, broadcast nature of the wireless (radio) channel**|Unlike a wired link, a radio channel can be **passively intercepted (eavesdropped upon)** by anyone within physical range, with no need to physically tap a cable — dramatically **lowering the barrier** to interception attempts|Because the channel itself is inherently exposed, security in wireless networks must be provided by **cryptographic** means (encryption, authentication) rather than relying on physical inaccessibility, a theme this text **defers** in detail to the next chapter|
|**The "association" process is a trust-establishing handshake**|A malicious actor could attempt to **impersonate** a legitimate base station (a so-called **"rogue" or "evil-twin" base station**), tricking devices into associating with it and thereby **routing their traffic through the attacker**|Robust **mutual authentication** during the association process — verifying both that the device is legitimate **and** that the base station is legitimate — is essential to prevent such impersonation|
|**Base stations are a concentrated point of control**|Because all wireless devices' traffic passes through their associated base station, compromising a single base station grants an attacker **visibility and control** over every device currently associated with it|Base stations, much like the managing servers or controllers discussed elsewhere in this text, warrant **hardened security** given the **concentrated (centralized) blast radius** a compromise would create|
|**Ad hoc networks lack centralized infrastructure to enforce security policy**|With no base station providing identity and access management, ad hoc networks can be **more vulnerable** to devices joining without proper vetting (screening/verification), or to malicious nodes disrupting the self-organized routing process|Ad hoc network protocols must build trust and security mechanisms **into the routing and addressing protocols themselves**, since there's no centralized infrastructure to fall back on|
|**Handoff/handover introduces a vulnerable transition window**|As a device switches its point of attachment from one base station to another, there may be a brief window during which authentication state is being transferred or re-established — a window an attacker could potentially exploit to **hijack** the session or **impersonate** the device|Secure handoff protocols aim to make this transition both **fast** (to avoid service interruption) and **cryptographically sound** (to avoid a security gap), a genuinely difficult balance to strike|

---

## Questions I Still Have

- [ ] The text says WiFi has no dedicated "core network" the way 4G/5G does, and instead just uses the enterprise's existing wired LAN infrastructure — does this mean a WiFi network fundamentally lacks the sophisticated mobility-management capabilities of a cellular core network, or are equivalent capabilities achieved some other way within enterprise WiFi deployments?
- [ ] For a device operating in "ad hoc mode," how exactly does address assignment work without any centralized infrastructure like DHCP — do the devices negotiate addresses amongst themselves, and how do they avoid address collisions?
- [ ] The text distinguishes control-plane services (identity, mobility management) from data-plane services (forwarding datagrams) within the wireless core network — is this the same control-plane/data-plane distinction introduced for SDN in Chapter 5, or a related-but-distinct usage of the same terminology in a wireless context?
- [ ] Given that the base station has "no counterpart in a wired network," what wired-network component comes conceptually closest to fulfilling a similar function (e.g., is it most analogous to a switch, an access point in the wired sense, or something else entirely)?
- [ ] The text notes that 60 percent of website traffic in 2025 was directed to mobile versus fixed devices — is this measured by the number of requests, the volume of data transferred, or some other metric, and does the answer change the interpretation much?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Wireless access network**|The outermost edge of a wireless network, where devices connect via a wireless radio channel; functions similarly to a local area network with limited geographic scope|
|**Radio Access Network (RAN)**|The term used for the wireless access network in cellular networks|
|**Wireless Local Area Network (WLAN)**|The term used for the wireless access network in WiFi networks|
|**Wireless core network**|The interior infrastructure (servers, storage, routers, links) providing control-plane (identity, mobility, access-security) and data-plane (forwarding) services between the access network and the external network|
|**Wireless device**|An end device (smartphone, tablet, laptop, IoT sensor/appliance) that hosts and runs applications, and attaches to a wireless access network|
|**Base station**|The most unique element of a wireless network, responsible for sending/receiving packets to/from associated wireless devices and relaying them to/from the Internet; called a gNB in 5G, an eNB in 4G, and an Access Point (AP) in WiFi|
|**Associated**|A device's state when it is (1) within wireless communication range of a base station and (2) has completed a protocol exchange enabling the base station to relay its packets|
|**Downstream/downlink channel**|The direction of transmission from the base station to its associated devices|
|**Upstream/uplink channel**|The direction of transmission from associated devices to the base station|
|**Infrastructure mode**|A mode of operation where devices connect via a base station, which provides addressing, identity, and routing services|
|**Ad hoc mode/network**|A mode of operation with no base station, where devices must self-organize to provide their own routing, addressing, and naming services|
|**Handoff/handover**|The process by which a mobile device changes its point of attachment from one base station to another as it moves|

---

## Related Concepts

---

→ Next: [[PHYSICAL LAYER]]