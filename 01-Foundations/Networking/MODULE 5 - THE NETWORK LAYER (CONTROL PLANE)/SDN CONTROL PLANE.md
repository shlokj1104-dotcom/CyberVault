---
title: SDN CONTROL PLANE
date: 2026-07-08
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 5.5 The SDN Control Plane

> **One-Line Summary:** Where Sections 5.2–5.4 showed the control plane implemented the traditional way — per-router, with each router running its own piece of a distributed algorithm (Dijkstra for OSPF, distance-vector-like policy exchange for BGP) — **SDN (Software-Defined Networking)** rips that logic out of the individual switches entirely and centralizes it into a **logically centralized SDN controller** (built in three layers: southbound communication, network-wide state management, and a northbound API) that talks to dumb, fast "packet switches" via a standard protocol (**OpenFlow**), while separately-written **network-control applications** (routing, access control, load balancing) sit on top and simply read state and write flow-table rules through that controller — an architectural "unbundling" of hardware, switch OS, and control logic that traces its roots back through Ethane and OpenFlow to earlier circuit-switched and ATM control-plane separation efforts, and is now embodied in production-grade distributed systems like ONOS and Google's Orion.

---

## Core Idea: Pulling the "Brains" Out of Every Router

In Sections 5.2 through 5.4, we studied the network control plane implemented as part of the routers themselves — as a piece of a **distributed** routing algorithm, with a piece of that distributed computation running in every router, and with the routers communicating with each other as needed. OSPF's link-state flooding and Dijkstra computation happen locally inside each router; BGP's route-selection algorithm runs locally inside each router.

**SDN takes a fundamentally different approach.** The network-wide logic that controls packet forwarding, along with the configuration and management of the devices, is instead implemented in a **logically centralized controller**, typically realized by several coordinated servers, physically separate from the forwarding devices. Forwarding devices themselves become simple, fast, standardized "packet switches" that do nothing more than compute a match-plus-action rule lookup on each arriving packet.

> **Analogy — An Orchestra vs. a Flock of Birds:** Traditional per-router routing (OSPF/BGP) is like a flock of birds: no bird has the "big picture," yet coordinated flight emerges purely from each bird reacting to its immediate neighbors according to shared local rules. SDN is more like an orchestra: a conductor (the SDN controller) holds the full score (network-wide state) and tells every musician (switch) exactly what to play and when, while the musicians themselves don't need to understand the whole piece — they just execute the notes they're handed.

### Terminology Carried Over from Section 4.4

This section builds directly on the earlier discussion of generalized SDN forwarding in Section 4.4 and control-plane basics in Section 5.1. As in Section 4.4, forwarding devices are referred to as **"packet switches"** (or just **switches**), since forwarding decisions can be made not only on network-layer destination address but on **any number of header-field values** — transport-layer, network-layer, and link-layer fields alike. This is a deliberate broadening of vocabulary: a traditional router forwards solely on destination IP; an SDN packet switch's forwarding rule can match on far more.

> This SDN control-plane discussion also lays the groundwork for the 5G cellular control plane covered later, which adopts an SDN-like approach.

---

## Four Key Characteristics of an SDN Architecture [Kreutz 2015]

|#|Characteristic|What It Means|
|---|---|---|
|**1**|**Flow-based forwarding**|Packet forwarding by SDN-controlled switches can be based on **any combination** of transport-layer, network-layer, and link-layer header-field values — not just destination IP. Recall from Section 4.4 that the OpenFlow 1.0 abstraction allows forwarding based on **eleven different header field values**. This is a sharp contrast with traditional router-based forwarding (Sections 5.2–5.4), which forwards an IP datagram solely on the datagram's destination IP address|
|**2**|**Separation of data plane and control plane**|Clearly shown in Figures 5.2 and 5.14. The **data plane** consists of relatively simple (but fast) switches that execute "match plus action" rules in their flow tables. The **control plane** consists of servers and software that determine and manage the switches' flow tables. It is the job of the SDN control plane to compute, manage, and install flow-table entries in **all** of the network's switches|
|**3**|**Network control functions: external to data-plane switches**|Given that the "S" in SDN is for "software," it's perhaps unsurprising that the SDN control plane is implemented in software. Unlike traditional routers, however, this software executes on servers that are both **distinct and remote** from the network's switches. As shown in Figure 5.14, the control plane itself is composed of two components — an **SDN controller** (or **network operating system** [Gude 2008]) and a set of **network-control applications**. The controller maintains accurate network state information (e.g., the state of remote links, switches, and hosts); provides this information to the network-control applications running in the control plane; and provides the means through which these applications can monitor, program, and control the underlying network devices. Although the controller is shown as a single central server in Figure 5.14, in practice the controller is only **logically centralized** — it is typically implemented on several servers for coordinated, scalable performance and high availability|
|**4**|**A programmable network control plane**|The network control plane is **programmable** through the network-control applications running in the control plane. These applications represent the "brains" of the SDN control plane, using the APIs provided by the SDN controller to specify and control the data plane in the network devices. For example, a **routing** network-control application might determine end-to-end paths between sources and destinations (e.g., by executing Dijkstra's algorithm using the node-state and link-state information maintained by the SDN controller). Another network application might perform **access control** — determining which packets are to be blocked at a switch. Yet another might have switches forward packets in a manner that performs server **load balancing**|

![[Figure 5.14 - Components of the SDN architecture.png]]

> **The Big Picture — SDN as "Unbundling":** From this discussion, SDN represents a significant **unbundling** of network functionality — data-plane switches, SDN controllers, and network-control applications are separate entities that may each be provided by **different vendors and organizations**. This contrasts with the pre-SDN model, in which a switch/router (together with its embedded control-plane software and protocol implementations) was monolithic, vertically integrated, and sold by a single vendor. This unbundling of network functionality in SDN has been likened to the earlier evolution of mainframe computers (where hardware, system software, and applications were all provided by a single vendor) into personal computers (with their separate hardware, operating systems, and applications). The unbundling of computing hardware, system software, and applications led to a rich, open ecosystem driven by innovation in all three of these areas; one hope for SDN is that it will continue to drive and enable such rich innovation.

![[Pasted image 20260708220153.png]]

```
Fig 5.2/5.14 -- The Core SDN Split
────────────────────────────────────

  ┌─────────────────────────────────┐
  │   NETWORK-CONTROL APPLICATIONS   │   <- "brains": routing,
  │   Routing | Access Ctrl | LB     │      access control, LB
  └────────────────┬──────────────── ┘
                    │ Northbound API
  ┌─────────────────▼─────────────────┐
  │         SDN CONTROLLER            │  <- "network operating
  │   (network operating system)      │     system"; logically
  └────────────────┬──────────────────┘     centralized
                    │ Southbound API
  ┌─────────────────▼─────────────────┐
  │      SDN-CONTROLLED SWITCHES      │  <- fast, simple,
  │   (flow-table match+action only)  │     "dumb" data plane
  └────────────────────────────────────┘

  CONTROL PLANE = top two boxes (external, remote, in software)
  DATA PLANE    = bottom box (fast, embedded, in hardware)
```

Given this architecture, natural questions arise: **How and where are the flow tables actually computed? How are these tables updated in response to events at SDN-controlled devices** (e.g., an attached link going up/down)? **And how are the flow-table entries at multiple switches coordinated** in such a way as to result in orchestrated, network-wide functionality (e.g., end-to-end paths for forwarding packets from sources to destinations, or coordinated distributed firewalls)? It is the role of the SDN control plane to provide these, and many other, capabilities.

---

## 5.5.1 The SDN Control Plane: SDN Controller and SDN Network-Control Applications

As noted above, the SDN control plane divides broadly into two components — the **SDN controller** and the **SDN network-control applications**. Let's explore the controller first, considering its structure in an uncharacteristically bottom-up fashion, organized into **three layers**.

### Layer 1 (Bottom): The Communication Layer — Southbound Interface

If an SDN controller is going to control the operation of a remote SDN-enabled switch, host, or other device, a **protocol is needed to transfer information** between the controller and that device. In addition, a device must be able to communicate **locally-observed events** back to the controller — for example:

- A message indicating that an attached link has gone up or down
- A message indicating that a device has just joined the network
- A heartbeat indicating that a device is up and operational

These events provide the SDN controller with an **up-to-date view of the network's state**. This protocol constitutes the **lowest layer** of the controller architecture, as shown in Figure 5.15. The communication between the controller and the controlled devices crosses what has come to be known as the controller's **"southbound" interface**. In Section 5.5.2, we study **OpenFlow** — a specific protocol that provides this communication functionality.

### Layer 2 (Middle): The Network-Wide State-Management Layer

The ultimate control-plane decisions made by the SDN control plane — for example, configuring flow tables in all switches to achieve the desired end-to-end forwarding, to implement load balancing, or to implement a particular firewalling capability — require the controller to have **up-to-date information** about the state of the network's hosts, links, switches, and other SDN-controlled devices.

A switch's flow table also contains **counters** whose values might profitably be used by network-control applications; these values should thus be available to the applications too. Since the ultimate aim of the control plane is to determine flow tables for the various controlled devices, a controller might also maintain a **copy of these tables**. All of these pieces of information collectively constitute the **network-wide "state"** maintained by the SDN controller.

![[Pasted image 20260708220455.png]]

### Layer 3 (Top): The Interface to the Network-Control Application Layer — Northbound Interface

The controller interacts with network-control applications through its **"northbound" interface**. This API allows network-control applications to **read/write network state and flow tables** within the state-management layer. Applications can register to be notified when state-change events occur, so that they can take actions in response to network event notifications sent from SDN-controlled devices. Different types of APIs may be provided; two popular SDN controllers communicate with their applications using a **REST** [Fielding 2000] request-response interface.

```
Fig 5.15 -- Components of an SDN Controller
────────────────────────────────────────────

                                       Northbound API
   ┌─────────────────────────────────────────┐
   │  Interface / abstractions for network    │
   │  control apps                            │
   │  ┌─────────┐ ┌──────────┐ ┌───────┐      │
   │  │ Network │ │ RESTful  │ │Intent │      │  <- TOP LAYER
   │  │ graph   │ │ API      │ │       │      │
   │  └─────────┘ └──────────┘ └───────┘      │
   └───────────────────┬───────────────────────┘
   ┌───────────────────▼───────────────────────┐
   │  Network-wide distributed, robust state    │
   │  management                                │
   │  ┌──────────┐┌───────────┐┌──────────┐    │
   │  │Statistics││Flow tables││Link-state│    │  <- MIDDLE LAYER
   │  │          ││           ││info      │    │
   │  └──────────┘└───────────┘└──────────┘    │
   │  ┌──────────┐┌───────────┐                │
   │  │Host info ││Switch info│                │
   │  └──────────┘└───────────┘                │
   └───────────────────┬───────────────────────┘
   ┌───────────────────▼───────────────────────┐
   │  Communication to/from controlled devices  │
   │  ┌───────────┐        ┌────────┐           │  <- BOTTOM LAYER
   │  │ OpenFlow  │  ...   │  SNMP  │           │
   │  └───────────┘        └────────┘           │
   └───────────────────┬───────────────────────┘
                        │ Southbound API
              ┌─────────▼─────────┐
              │  SDN-Controlled   │
              │     Switches      │
              │   (mesh network)  │
              └────────────────────┘
```

### "Logically Centralized" ≠ One Physical Box

We have noted several times that an SDN controller can be considered to be **"logically centralized"** — that is, the controller may be viewed externally (from the point of view of SDN-controlled devices and external network-control applications) as a **single, monolithic entity**. However, these services and the databases used to hold state information are implemented in practice by a **distributed set of servers**, for fault tolerance, high availability, and performance reasons.

> **Analogy — A Company's "Head Office":** Employees across every branch office treat "head office" as a single decision-making authority — they don't need to know or care that head office is actually a building full of many different departments and staff, spread across floors, coordinating internally. Similarly, a switch talking to the SDN controller doesn't need to know (or care) that "the controller" is actually several physically distributed servers coordinating behind the scenes — from the switch's point of view, it's a single logical authority.

With controller functions implemented by a set of servers, questions of logical time ordering of events, consistency, and consensus arise naturally — concerns common across many distributed systems (see [Lamport 1989; Lampson 1996] for elegant solutions). Modern controllers such as **ONOS** [ONOS 2025] and **ORION** [Ferguson 2021] (covered in the sidebar below) have placed considerable emphasis on architecting a **logically centralized but physically distributed** controller platform that provides scalable services and high availability to the controlled devices and network-control applications alike.

> The architecture depicted in Figure 5.15 closely resembles the architecture of the originally proposed **NOX controller** in 2008 [Gude 2008], as well as today's controllers.

---

## 5.5.2 OpenFlow Protocol

The **OpenFlow protocol** [OpenFlow 2025] operates between an SDN controller and an SDN-controlled switch (or other device implementing the OpenFlow API that we studied earlier in Section 4.4). The OpenFlow protocol operates **over TCP**, with a default port number of **6653**.

> This is directly analogous to how BGP (Section 5.4) also operates over TCP, with its own default port (179) — both inter-AS routing and SDN southbound communication rely on TCP's reliability rather than reinventing reliable delivery.

### Messages: Controller → Switch

|Message|Purpose|
|---|---|
|**Configuration**|Allows the controller to **query and set** a switch's configuration parameters|
|**Modify-State**|Used by a controller to **add/delete or modify** entries in the switch's flow table, and to set switch port properties|
|**Read-State**|Used by a controller to **collect statistics and counter values** from a switch's flow table and ports|
|**Send-Packet**|Used by the controller to send a **specific packet out of a specified port** at the controlled switch. The message itself contains the packet to be sent, in its payload|

### Messages: Switch → Controller

|Message|Purpose|
|---|---|
|**Flow-Removed**|Informs the controller that a flow-table entry has been **removed** — e.g., by a timeout, or as the result of a received Modify-State message|
|**Port-status**|Used by a switch to inform the controller of a **change in port status**|
|**Packet-in**|Recall from Section 4.4 that a packet arriving at a switch port and **not matching any flow-table entry** is sent to the controller for additional processing. Matched packets may also be sent to the controller, as an action to be taken on a match. The **Packet-in** message is used to send such packets to the controller|

> **Analogy — A Store Manager and a Cashier with a Rulebook:** The switch is a cashier following a fixed rulebook (the flow table) of "if this item, ring it up this way." Most transactions the cashier handles alone, at full speed, without ever calling the manager. But if a transaction doesn't match any rule in the book (Packet-in), the cashier calls the manager (controller) for a decision — and the manager can also proactively rewrite pages of the rulebook (Modify-State), ask for a summary of what's been rung up (Read-State), or occasionally hand the cashier an item to process directly (Send-Packet).

---

## 5.5.3 Data and Control Plane Interaction: A Worked Example

In order to solidify our understanding of the interaction between SDN-controlled switches and the SDN controller, consider the example shown in Figure 5.16, in which **Dijkstra's algorithm** (which we studied in Section 5.2) is used to determine shortest-path routes.

This SDN scenario has **two important differences** from the earlier per-router-control scenario of Sections 5.2.1 and 5.3, where Dijkstra's algorithm was implemented in each and every router and link-state updates were flooded among all network routers:

|Traditional (OSPF-style)|SDN|
|---|---|
|Dijkstra's algorithm runs **inside every router**|Dijkstra's algorithm is executed as a **separate application**, outside the packet switches|
|Routers **flood** link-state updates to **all** other routers directly|Packet switches send link updates to the **SDN controller**, and **not to each other**|

### Setup

Assume the link between switch **s1** and **s2** goes down; consequently, incoming and outgoing flow-forwarding rules at s1, s3, and s4 are affected, but s2's operation is unchanged. Assume OpenFlow is used as the communication-layer protocol, and that the control plane performs no other function than link-state routing.

```
Fig 5.16 -- SDN Controller Scenario: Link-State Change
──────────────────────────────────────────────────────

                    Dijkstra's link-state
                          Routing
                             ▲
              ┌──────────────┼──────────────┐
              │   (4)  Network graph   ...  Intent  (5)
              │              ▲                       │
              │   (3)  Statistics    Flow tables      │
              │        Link-state    Host    Switch  │
              │        info                  info     │
              │              ▲                       │
              │   (2)  OpenFlow    ...    SNMP        │
              └──────────────┼───────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │   s1 ══X══ s2     │  <- link fails (1)
                    │    ╲       ╱      │
                    │     s3 ─ s4       │
                    └────────────────────┘
```

### Step-by-Step Sequence

1. Switch **s1**, experiencing a link failure to s2, notifies the SDN controller of the link-state change using the OpenFlow **port-status** message.
2. The SDN controller receives the OpenFlow message indicating the link-state change, and notifies the **link-state manager**, which updates its link-state database.
3. The network-control application that implements **Dijkstra's routing** has previously registered to be notified when link-state changes occur. That application receives the notification of the link-state change.
4. The link-state routing application interacts with the link-state manager to get updated link state; it might also consult other components in the state-management layer. The link-state routing application then computes the new least-cost paths.
5. The link-state routing application then interacts with the **flow-table manager**, which determines the new flow-table entries to be updated at affected switches — **s1** (which will now route packets destined to s2 via s4), **s2** (which will now begin receiving packets from s1 indirectly via s4), and **s4** (which must now forward packets from s1 destined to s2).
6. The flow-table manager then uses the OpenFlow protocol to update flow-table entries at the affected switches.

This example is simple, but it illustrates how the SDN control plane provides **control-plane services** (in this case, network-layer routing) that had previously been implemented with **per-router control** exercised in each and every network router. One can now easily appreciate how an SDN-enabled ISP could easily switch from per-router path routing to a more hands-off approach to routing. Indeed, since the controller can tailor the flow tables as it pleases, it can implement **any** form of forwarding that it pleases — simply by changing its application-control software. This ease of change would be far different from the case of a traditional per-router control plane, where software in all routers (which might be provided to the ISP by multiple independent vendors) must be changed.

---

## Principles in Practice: SDN Controller Case Studies — ONOS and Orion

In the earliest days of SDN, there was a single SDN protocol (**OpenFlow** [McKeown 2008; OpenFlow 2025]) and a single SDN controller (**NOX** [Gude 2008]). Since then, the number of SDN controllers has grown significantly [Kreutz 2015]. Some SDN controllers are company-specific and proprietary, particularly when used to control internal proprietary networks (e.g., among a company's data centers). Below, two controllers are described that demonstrate how the principle of a logically centralized control plane is implemented in practice.

### The ONOS Controller

**Open Network Operating System (ONOS®)** [ONOS 2025] is an open-source SDN controller created, distributed, and supported by the Open Networking Foundation [OpenNetworking 2025].

```
Fig 5.17 -- ONOS Controller Architecture
──────────────────────────────────────────

  Network control apps  (routing, etc.)
              ▲
              │ Northbound abstractions/protocols
   ┌──────────┴───────────────┐
   │  REST API   |   Intent   │
   └──────────┬────────────────┘
   ┌──────────▼────────────────┐
   │  ONOS Distributed Core     │
   │  Hosts | Paths | Flow rules│
   │  Devices|Links |Statistics │
   │              Topology      │
   └──────────┬────────────────┘
              │ Southbound abstractions/protocols
   ┌──────────▼────────────────┐
   │ Device | Link | Host | Flow│
   │        | Packet             │
   │  OpenFlow | Netconf | OVSDB │
   └──────────┬────────────────┘
              │
        (mesh of network devices)
```

![[Figure 5.17 - ONOS controller architecture.png]]

Figure 5.17 presents a simplified view of ONOS — similar to the canonical controller in Figure 5.15. Three layers can be identified in the ONOS architecture:

|Layer|Description|
|---|---|
|**Northbound abstractions and protocols**|A unique feature of ONOS is its **intent framework**, which allows an application to request a high-level service (e.g., to set up a connection between host A and host B, or conversely to tell host A and host B how to communicate) **without needing to know the details of how this service is performed**. State information is provided to network-control applications across the northbound API either synchronously (via query) or asynchronously (via listener callbacks, e.g. when network state changes)|
|**Distributed core**|The state of the network's links, hosts, and devices is maintained in ONOS's **distributed core**. ONOS is deployed on a service set of interconnected servers, each running an identical copy of the ONOS software, with an increased number of servers often increasing network service capacity. The ONOS core provides mechanisms for service replication and coordination among services, providing the applications above and the network devices below with the abstraction of logically centralized core services|
|**Southbound abstractions and protocols**|These abstractions mask the heterogeneity of the underlying hosts, links, switches, and protocols, allowing the distributed core to be both device- and protocol-agnostic. Because of this abstraction, the southbound interface between the distributed core is logically higher than in our canonical controller in Figure 5.14|

### Orion — Google's Second-Generation SDN Platform

**Orion** [Ferguson 2021] is the second-generation SDN platform used to control **Google's Jupiter datacenter networks** [Singh 2015] as well as the internal **B4 wide-area network** [Jain 2013] that globally connects Google's data centers. Figure 5.18 presents a high-level view of Orion's overall structure and context that is remarkably similar to ONOS, shown in Figure 5.17. Indeed, Orion creates now [Ferguson 2021] how the "architecture of Orion ... maps to the textbook view" [ONF 2013]. We see the SDN core at the center of Figure 7.18, with a northbound interface to the Orion apps, and a southbound interface to the SDN-controlled routers and switches. We can see many important differences from this common high-level view.

```
Fig 5.18 -- Google's Orion SDN
─────────────────────────────────────

  Orion apps
  ┌─────────┐ ┌─────────┐ ┌────────┐ ┌────────┐
  │  Raven   │ │  Path   │ │ ...    │ │ other  │
  │(BGP/ISIS)│ │Exporter │ │        │ │ Orion  │
  │          │ │         │ │        │ │  apps  │
  └─────┬────┘ └────┬────┘ └────────┘ └────────┘
        │   Routing │  TE App        IBR-C
        │   Engine  │
  ┌─────▼────────────▼──────────────────────────┐
  │              Orion core                      │
  │  ┌────────────────────┐  ┌─────────────────┐│
  │  │ Network Information │  │  Configuration  ││
  │  │ Base (NIB)           │  │    Manager      ││
  │  │ pub/sub data         │  └─────────────────┘│
  │  └────────────────────┘                       │
  │  Flow Manager: flow state reconciliation       │
  │  Topology Manager: nodes, links, ports,        │
  │                     interfaces                  │
  └─────────────────────┬──────────────────────────┘
  ┌─────────────────────▼──────────────────────────┐
  │              OpenFlow Front End                  │
  └─────────────────────┬──────────────────────────┘
              Routers, switches in Orion domain
                    (mesh of devices)
```

Each Google network is divided into one or more **domains**, with an Orion controller instance being responsible for managing and controlling the devices within its domain. The use of **multiple domains** permanently limits the scope and impact of any failures, as well as the amount of work that must be performed by an Orion controller.

![[Figure 5.18 - Google's Orion SDN.png]]

At the heart of the core is the **Network Information Base (NIB)**. The NIB is a **logically centralized data store**, coupled with a data-driven **publish/subscribe (pub/sub)** system that allows Orion apps and core components to read/write NIB data, as well as receive notifications when NIB data changes. A logically centralized data store also has the advantage of globally ordering network events, which significantly helps with debugging and troubleshooting problems. [Ferguson 2021] notes: "Based on years of experience, the NIB has been one of our most successful design elements." An Orion NIB can process roughly **a million read/write events per second**.

The following components, implemented as **microservices**, are found in the Orion core:

|Component|Function|
|---|---|
|**The Topology Manager**|Maintains the state of the links, routers, and switches in the Orion controller's domain. Also queries and receives runtime port statistics|
|**The Config Manager**|Configures components in the Orion controller's domain by setting configuration parameter properties in the NIB table. The Config Manager can also check and validate configurations for correctness|
|**The Flow Manager**|Ensures that the intended network state and behavior recorded by apps in the NIB matches the collective state reported by the controlled devices, and performs **flow state reconciliation**, as needed|

### Intent-Based Networking

Orion enables an approach to network management and control known as **"intent-based" networking**. [Ferguson 2021] notes that "intent-based specify management or design changes by describing the new intended end-state of the network (the 'what') rather than prescribing the modification to bring the network to that end-state (the 'how')." For example, the intended network topology is specified in terms of an abstract model — this specification is **authoritative**, in that it requires that lower-level network apps (e.g., link activation, routing) monitor, set, and change their configurations so that the network's intended state, at longer time scales, and network state, at shorter time scales, remain fully in sync. When high-level intent and monitored device state occur bottom up and change more rapidly as network conditions change, future configuration settings, and monitored device state are all stored in the NIB, and the NIB's pub/sub mechanism is used to notify apps when they need to take actions in response to a changed NIB value.

> **Analogy — a GPS Nav App vs. Turn-by-Turn Micromanagement:** Traditional device-by-device configuration is like manually telling a driver "turn left here, now turn right, now merge" — every instruction must be issued and re-issued as conditions change. Intent-based networking is instead like telling a GPS app your **destination** ("get me to the airport by 6pm") and letting it continuously work out and adjust the turn-by-turn instructions itself — you specify the desired _end-state_, not the sequence of low-level actions to get there.

---

## 5.5.4 SDN: Past and Future

Although the intense interest in SDN is a relatively recent phenomenon, a number of SDN's technical roots — the separation of the data and control planes in particular — go back considerably further, back to the early 1970s' fully decentralized ARPAnet, which purposefully embraced a decentralized protocol-based approach to network control.

### The Historical Arc

|Era / System|Contribution|
|---|---|
|**ARPAnet (early 1970s)**|The ancestor of today's networks — purposefully embraced a **decentralized** protocol-based approach to network control|
|**TYMNET I** [Tymes 1981]|A virtual-circuit-based network that had a centralized controller physically separate from the data-plane portions of TYMNET nodes, connected to the controlled portions of TYMNET via an arbitrary mesh topology|
|**AT&T's Spider network** [Fraser 1993]|Had a clear data/control-plane separation, with a dedicated Vaxfull router to sending control messages via out-of-band signaling to/within the node and the controller|
|**1990s–2000s ATM / data-plane-control-plane separation**|[van der Merwe 1998] describes a control framework for ATM switches, each controlling a number of ATM switches. In 2004, **Forces** [Feamster 2004; Laskshman 2004; RFC 3746] all argued in favor of the separation of the network's data and control planes, in addition to other attributes often associated with network virtualization, softwarization, and programmability [Feamster 2014; Anenousis 2021] both delve into the history of the concepts underlying SDN|
|**Ethane project** [Casado 2007]|Pioneered the notion of a network of simple flow-based Ethernet switches with match-plus-action rule tables, and a centralized controller that managed flow admission and routing, and the forwarding of unmatched packets from the switch to the controller. A network of more than 300 Ethane switches was operational in 2007|
|**Ethane → OpenFlow**|Ethane quickly evolved into the OpenFlow project, and the rest (as the saying goes) is history!|

> **Analogy — SDN's Roots Are Older Than "SDN" Itself:** SDN is often talked about as a modern revolution, but the core idea — keeping "dumb," fast forwarding hardware separate from a smarter, centralized decision-making brain — is more like rediscovering an old architectural pattern (seen already in TYMNET, AT&T's Spider, and ATM control frameworks) and finally giving it a standard, open, widely-adopted protocol (OpenFlow) and a catchy name.

### Where SDN Is Headed

Numerous research efforts are aimed at developing future SDN architectures and capabilities. As we have seen, the SDN revolution is leading to the disruptive replacement of dedicated monolithic switches and routers (with both data and control planes) by **simple commodity switching hardware and a sophisticated software control plane**.

|Future Direction|Description|
|---|---|
|**Network Functions Virtualization (NFV)**|A generalization of SDN known as **network functions virtualization** (which we discussed earlier in Section 4.5) similarly aims at disruptive replacement of sophisticated middleboxes (such as middleboxes with dedicated hardware and proprietary software for media caching/service) with **simple commodity servers, switching, and storage**|
|**Failure resilience**|Another important challenge is to structure SDN controllers to be **robust with respect to failures** (e.g., router or computer crashes, loss of connectivity) and to be able to recover rapidly when failures do occur [Ferguson 2021; Krentsel 2024]|

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**The SDN controller is a single logical point of control over the entire data plane**|A compromised or spoofed controller (or a successful attack against the controller's southbound channel) can push malicious flow-table rules to every switch in the network simultaneously — turning a single breach into network-wide traffic interception, blackholing, or redirection|Controllers are deployed as a **distributed** set of coordinated servers (not a single physical box) precisely to reduce single-point-of-failure risk, and OpenFlow sessions should be run over authenticated, encrypted channels (e.g., TLS) to prevent spoofing of controller-switch communication|
|**OpenFlow's Packet-in mechanism sends unmatched packets to the controller**|An attacker can craft a flood of packets deliberately designed to miss every flow-table entry, forcing a storm of Packet-in messages to the controller — a control-plane saturation / denial-of-service vector unique to the SDN architecture|Rate-limiting Packet-in traffic, installing default "catch-all" flow rules, and monitoring for abnormal Packet-in volume are practical mitigations|
|**Flow-table rules can match on any of eleven+ header fields, giving fine-grained control**|An attacker who gains northbound API access (i.e., compromises a network-control application) can install access-control or routing rules that silently exfiltrate or reroute specific classes of traffic, since flow rules can be extremely surgical (matching very specific field combinations)|Northbound API access should be tightly authenticated and authorized per-application, since any app with write access to flow tables effectively has significant control over live traffic|
|**Intent-based systems (Orion) specify the desired end-state, with lower-level apps reconciling actual state to match it**|If an attacker can inject a false "intent" or corrupt the NIB's stored state, the reconciliation logic will actively and continuously work to enforce the attacker's malicious end-state across the network, rather than a single one-off bad config|Validating and authenticating intent submissions, and auditing NIB writes, are essential — the same reconciliation mechanism that makes intent-based networking powerful also makes a poisoned intent especially persistent|
|**Multi-domain architectures (like Orion) limit blast radius**|An attacker who compromises one domain's controller is still constrained — they cannot directly manipulate the flow tables of switches in a different domain|Dividing large networks into multiple independently-controlled domains is itself a defensive design pattern, deliberately limiting the scope and impact of any single controller compromise or failure|

---

## Questions I Still Have

- [ ] The text says an SDN controller is "logically centralized" but implemented as a distributed set of servers — what specific consensus protocol (e.g., Paxos, Raft) do real controllers like ONOS typically use to keep that distributed state consistent, and how does that choice trade off against Orion's NIB pub/sub model?
- [ ] For the OpenFlow Packet-in mechanism — in the worked example (Fig 5.16), does every single packet destined across the failed s1–s2 link generate a Packet-in message during the (presumably brief) window before the flow tables are updated, or is there some buffering/default behavior that avoids flooding the controller during convergence?
- [ ] ONOS's "intent framework" lets an app request a high-level service without knowing how it's performed — how does ONOS's intent compiler actually translate an abstract intent (e.g., "connect host A and host B") down into concrete flow-table entries at specific switches, and does this involve something like Dijkstra internally?
- [ ] Given that NFV is described as "a generalization of SDN" — is NFV strictly built on top of SDN infrastructure in practice, or can NFV be deployed independently of an SDN control plane?
- [ ] The chapter mentions Orion's NIB can process roughly a million read/write events per second — is this cited as an unusually high number relative to other production controllers (e.g., ONOS), or is this a fairly typical throughput requirement at Google's datacenter scale?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**SDN (Software-Defined Networking)**|An architecture that separates the data plane (fast, simple packet switches) from the control plane (a logically centralized SDN controller plus network-control applications), implementing network-wide control logic in software rather than per-router|
|**Packet switch**|SDN terminology for a forwarding device; forwarding decisions can be based on transport-, network-, and link-layer header fields, not just destination IP|
|**Flow-based forwarding**|Forwarding based on any combination of header-field values (up to eleven in OpenFlow 1.0), rather than solely destination IP address|
|**Data plane**|The layer consisting of relatively simple, fast switches executing "match plus action" flow-table rules|
|**Control plane**|The layer consisting of the SDN controller and network-control applications, external and remote from the switches, that computes and manages flow tables|
|**SDN controller**|Also called the "network operating system"; maintains network state, and provides that state to network-control applications via a northbound API, and communicates with switches via a southbound protocol (e.g., OpenFlow)|
|**Network-control application**|Software (e.g., routing, access control, load balancing) that runs atop the SDN controller's northbound API and represents the "brains" specifying desired data-plane behavior|
|**Logically centralized**|Appearing externally as a single, monolithic controller, while actually being implemented internally by a distributed set of coordinated servers|
|**Southbound interface**|The communication layer/protocol between the SDN controller and the controlled switches (e.g., OpenFlow, SNMP)|
|**Northbound interface / API**|The interface between the SDN controller and network-control applications, often REST-based, allowing apps to read/write network state and flow tables|
|**OpenFlow**|The earliest and most widely used SDN southbound protocol; operates over TCP, port 6653, with defined controller→switch and switch→controller message types|
|**Packet-in**|An OpenFlow message sent from switch to controller when an arriving packet does not match any flow-table entry (or as an explicit match action)|
|**Port-status**|An OpenFlow message used by a switch to inform the controller of a change in port status (e.g., a link going down)|
|**Modify-State**|An OpenFlow message used by the controller to add, delete, or modify flow-table entries at a switch|
|**ONOS (Open Network Operating System)**|An open-source SDN controller with a distinctive northbound "intent framework," a distributed core, and device/protocol-agnostic southbound abstractions|
|**Intent framework**|ONOS's northbound abstraction allowing applications to request high-level services (e.g., "connect host A and B") without specifying how the service is implemented|
|**Orion**|Google's second-generation SDN platform, controlling the Jupiter datacenter networks and the B4 wide-area network; centered on the Network Information Base (NIB)|
|**Network Information Base (NIB)**|Orion's logically centralized, pub/sub-based data store coupling network state and configuration, enabling globally ordered event tracking|
|**Intent-based networking**|A management approach (used by Orion) where the desired network end-state ("what") is specified declaratively, and lower-level components continuously reconcile actual state to match it, rather than prescribing step-by-step configuration changes ("how")|
|**Ethane**|A 2007 research project pioneering flow-based Ethernet switches with a centralized controller for flow admission and routing — the direct precursor to OpenFlow|
|**NFV (Network Functions Virtualization)**|A generalization of SDN aimed at replacing dedicated, proprietary middlebox hardware (caching, media services, etc.) with simple commodity servers, switching, and storage|

---

## Related Concepts

---

→ Next: [[ICMP]]