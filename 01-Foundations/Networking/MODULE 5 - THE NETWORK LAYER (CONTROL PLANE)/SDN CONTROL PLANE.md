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

> **One-Line Summary:** SDN takes the match-plus-action data-plane abstraction from Section 4.4 and builds a complete **network-wide control plane** on top of it, defined by four key characteristics — flow-based forwarding on any combination of header fields, a hard separation of the (simple, fast) data plane from the (software-based) control plane, control functions that run **external** to the switches themselves on remote servers, and a genuinely **programmable** control plane driven by network-control applications — and this architecture represents a deliberate "unbundling" of network functionality (switches, controller, and applications as independently-sourced pieces) directly analogous to the historical shift from monolithic mainframes to modular personal computers; concretely, the SDN controller itself is built from three layers (a southbound communication layer, a network-wide state-management layer, and a northbound application-interface layer), with **OpenFlow** serving as the most widely studied protocol implementing that bottom, southbound communication layer.

---

## Core Idea: From Data-Plane Mechanism to Control-Plane Architecture

Section 4.4 introduced the **mechanism** of generalized SDN forwarding — the match-plus-action flow table sitting inside each packet switch. Section 5.1 introduced the **high-level distinction** between per-router control and logically centralized control. This section fuses both threads: it dives into the actual **network-wide logic** that controls packet forwarding among a network's SDN-enabled devices, as well as the configuration and management of those devices and their services.

Terminology carries over directly from Section 4.4: network-forwarding devices are still referred to as **"packet switches"** (or just "switches," with "packet" being understood), since forwarding decisions can be made on the basis of network-layer source/destination addresses, link-layer source/destination addresses, or many other values in transport-, network-, and link-layer packet-header fields. This same architectural foundation — a logically centralized controller driving simple data-plane devices — is also what underlies the control plane discussion for 5G cellular networks, which adopts an SDN-like approach.

---

## Four Key Characteristics of an SDN Architecture [Kreutz 2015]

### 1. Flow-Based Forwarding

Packet forwarding by SDN-controlled switches can be based on **any number of header field values** in the transport-layer, network-layer, or link-layer header. Recall from Section 4.4 that the OpenFlow 1.0 abstraction allows forwarding decisions based on **eleven** different header field values.

**This contrasts sharply** with the traditional approach to router-based forwarding studied in Sections 5.2–5.4 (LS, DV, OSPF, BGP), where forwarding of IP datagrams was based **solely** on a datagram's destination IP address. Recall from Figure 5.2 that packet-forwarding rules are specified in a switch's **flow table**; it is the job of the SDN control plane to **compute, manage, and install** flow table entries in all of the network's switches.

### 2. Separation of Data Plane and Control Plane

This separation is shown clearly in Figures 5.2 and 5.14:

|Plane|Contents|
|---|---|
|**Data plane**|Relatively simple (but fast) devices that execute the "match plus action" rules in their flow tables|
|**Control plane**|Servers and software that determine and manage the switches' flow tables|

This is a direct architectural echo of Section 5.1's Figure 5.2 (logically centralized control) — SDN is that model, made concrete and fully specified.

### 3. Network Control Functions: External to Data-Plane Switches

Given that the "S" in SDN stands for "**software**," it's perhaps unsurprising that the SDN control plane is implemented in software. **Unlike traditional routers, however, this software executes on servers that are both distinct and remote from the network's switches.**

![[Figure 5.14.png]]

```
FIGURE 5.14 -- SDN ARCHITECTURE COMPONENTS

        Network-control Applications
+-------+  +-------+  +-------+
| Route |  | Access|  | LoadB |
+-------+  +-------+  +-------+
    |          |          |
    -----------+-----------
               |   Northbound API
Control
   +------------------------+
   |   SDN Controller       |
   |  (network op. sys)     |
   +------------------------+
=============================
Data
               |   Southbound API
   +------------------------+
   |  SDN-Controlled        |
   |  Switches (mesh)       |
   +------------------------+
```

As shown in Figure 5.14, the control plane itself consists of **two components**: an **SDN controller** (or _network operating system_) [Gude 2008], and a set of **network-control applications**. The controller:

- **Maintains** accurate network state information (the state of remote links, switches, and hosts)
- **Provides** this state information to the network-control applications running in the control plane
- **Provides** the means through which those network-control applications can monitor, program, and control the underlying network devices

**On centralization vs. distribution:** Although the controller in Figure 5.14 is shown as a single central server, in practice the controller is only **logically centralized** — it is typically implemented on **several servers** that provide coordinated, scalable performance and high availability. This is the exact same "logically centralized" concept already introduced in Section 5.1: one logical service, potentially many physical servers underneath.

### 4. A Programmable Network Control Plane

The network control plane is **programmable** through the network-control applications running in the control plane. These applications represent the **"brains"** of the SDN control plane, using the APIs provided by the SDN controller to specify and control the data plane in the network devices.

|Application Type|What It Does|
|---|---|
|**Routing**|Might determine the end-to-end paths between sources and destinations — for example, by executing Dijkstra's algorithm using the node-state and link-state information maintained by the SDN controller (a direct echo of Section 5.2.1's LS algorithm, now running as a controller-side application rather than distributed router firmware)|
|**Access control**|Might determine which packets are to be **blocked** at a switch — this is exactly the third (firewalling) example already worked through in Section 4.4.3|
|**Load balancing**|Might have switches forward packets in a manner that performs server load balancing — this is exactly the second example worked through in Section 4.4.3|

> **Notice what just happened here:** the three OpenFlow examples worked through concretely in Section 4.4.3 (simple forwarding, load balancing, firewalling) are now revealed to be exactly the kinds of behavior a _network-control application_ would compute and hand down to the controller for installation. Section 4.4 showed _what the flow tables looked like_; this section explains _what actually computes them_.

---

## The Unbundling of Network Functionality

From this discussion, SDN represents a significant **"unbundling"** of network functionality — data-plane switches, SDN controllers, and network-control applications are **separate entities** that may each be provided by **different vendors and organizations**.

**This contrasts sharply with the pre-SDN model**, in which a switch/router (together with its embedded control-plane software and protocol implementations) was **monolithic**, vertically integrated, and sold by a single vendor.

> **Analogy — Mainframes vs. Personal Computers:** This unbundling of network functionality has been likened to the earlier evolution from **mainframe computers** (where hardware, system software, and applications were all provided as one bundled, vertically-integrated package by a single company) to **personal computers** (with their separate hardware, separate operating systems, and separately-sourced applications, often from entirely different companies). Just as unbundling the PC into hardware/OS/applications led to an explosion of innovation — because any company could now build a better piece of _any one_ layer without needing to rebuild the whole stack — the unbundling of computing hardware, system software, and applications in the network world has led to a **rich, open ecosystem** driven by innovation in all three areas independently. The hope for SDN is that it will continue to drive and enable exactly this kind of rich, cross-vendor innovation.

---

## 5.5.1 The SDN Control Plane: SDN Controller and SDN Network-Control Applications

Given the architecture of Figure 5.14, several natural questions arise: How and where are the flow tables actually **computed**? How are these tables **updated** in response to events at SDN-controlled devices (e.g., an attached link going up/down)? And how are the flow table entries at **multiple** switches coordinated in such a way as to result in orchestrated, network-wide functionality (e.g., end-to-end paths, or coordinated distributed firewalls)? Providing these — and many other — capabilities is the role of the SDN control plane.

The SDN control plane divides broadly into its two components: the **SDN controller** and the **SDN network-control applications**. Let's explore the controller's structure, organized bottom-up into three layers.

![[Figure 5.15.png]]

```
FIGURE 5.15 -- SDN CONTROLLER COMPONENTS

  Routing   Access-Ctrl   LoadBal
    (network-control applications)
          |  Northbound API
  +--------------------------------+
  | Interface / abstractions for   |
  |  net-control apps: Network     |
  |  graph, RESTful API, Intent    |
  +--------------------------------+
  | Network-wide distributed,      |
  |  robust state management:      |
  |  Statistics, Flow tables,      |
  |  Link-state, Host, Switch info |
  +--------------------------------+
  | Communication to/from          |
  |  controlled devices:           |
  |  OpenFlow, SNMP, ...           |
  +--------------------------------+
          |  Southbound API
    (mesh of SDN-controlled devices)
```

### Layer 1 (Bottom): The Communication Layer

**Communicating between the SDN controller and controlled network devices.** If an SDN controller is going to control the operation of a remote SDN-enabled switch, host, or other device, a **protocol** is needed to transfer information between the controller and that device. In addition, a device must be able to communicate locally-observed events **to** the controller — for example:

- A message indicating that an attached link has gone up or down
- A message indicating that a device has just joined the network
- A **heartbeat** message indicating that a device is up and operational

These events provide the SDN controller with an up-to-date view of the network's state. This protocol constitutes the **lowest layer** of the controller architecture, shown in Figure 5.15. **The communication between the controller and the controlled devices crosses what has come to be known as the controller's "southbound" interface.** Section 5.5.2 studies **OpenFlow** — a specific protocol that provides this communication functionality, and is implemented in most, if not all, SDN controllers.

### Layer 2 (Middle): The Network-Wide State-Management Layer

The ultimate control decisions made by the SDN control plane — for example, configuring flow tables at all switches to achieve the desired end-to-end forwarding, to implement load balancing, or to implement a particular firewalling capability — will require the controller to have **up-to-date information** about the state of the network's hosts, links, switches, and other SDN-controlled devices.

A switch's flow table also contains **counters** whose values might profitably be used by network-control applications; these values should thus be made available to the applications too. Since the ultimate aim of the control plane is to determine flow tables for the various controlled devices, a controller might also maintain **a copy of these tables** itself. All of these pieces of information — link state, host info, switch info, flow tables, statistics/counters — together constitute examples of the network-wide **"state"** maintained by the SDN controller.

### Layer 3 (Top): The Interface to the Network-Control Application Layer

The controller interacts with network-control applications through its **"northbound"** interface. This API allows network-control applications to **read/write** network state and flow tables within the state-management layer. Applications can also **register** to be notified when state-change events occur, so that they can take actions in response to network event notifications sent from SDN-controlled devices.

Different types of APIs may be provided by different controllers; two popular SDN controllers communicate with their applications using a **REST** [Fielding 2000] request-response interface.

### Logically Centralized, Physically Distributed — Again

We have noted several times already that an SDN controller can be considered to be **"logically centralized"** — that is, the controller may be viewed externally (from the point of view of SDN-controlled devices and network-control applications) as a **single, monolithic service**. However, these services and the databases used to hold state information are implemented in practice by a **distributed set of servers**, for fault tolerance, high availability, and performance-scalability reasons.

**With controller functions being implemented by a set of servers**, the semantics of the controller's internal operations — maintaining logical time ordering of events, consistency, consensus, and more — must be carefully considered. These concerns are common across many different distributed systems (see [Lamport 1989] and [Lampson 1996] for elegant solutions to these challenges).

**Modern controllers, including ONOS** [ONOS 2025] **and ORION** [Ferguson 2021] (Google's production SDN controller, already previewed in Section 5.1), **have placed considerable emphasis on architecting a logically centralized but physically distributed controller platform** that provides scalable services and high availability to both the controlled devices and the network-control applications alike.

The architecture depicted in Figure 5.15 closely resembles the architecture of the originally proposed **NOX** controller in 2008 [Gude 2008], as well as today's production controllers.

---

## 5.5.2 OpenFlow Protocol

Having examined the SDN controller's internal structure in the abstract, let's now examine the OpenFlow protocol — the earliest, and one of several protocols that can be used for communication between an SDN controller and a controlled device, at the controller's **communication layer**.

The **OpenFlow protocol** [OpenFlow 2025] operates **between** an SDN controller and an SDN-controlled switch or other device implementing the OpenFlow API studied earlier in Section 4.4. The OpenFlow protocol operates over **TCP**, with a default port number of **6653**.

### Messages Flowing From the Controller to the Controlled Switch

|Message|Purpose|
|---|---|
|**Configuration**|Allows the controller to query and set a switch's configuration parameters|
|**Modify-State**|Used by a controller to **add/delete/modify** entries in the switch's flow table, and to set switch port properties|
|**Read-State**|Used by a controller to **collect statistics and counter values** from the switch's flow table and ports|
|**Send-Packet**|Used by the controller to send a **specific packet out of a specified port** at the controlled switch. The message itself contains the packet to be sent, in its payload|

### Messages Flowing From the SDN-Controlled Switch to the Controller

|Message|Purpose|
|---|---|
|**Flow-Removed**|Informs the controller that a flow table entry has been removed — for example, by a **timeout**, or as the result of a received Modify-State message|
|**Port-status**|Used by a switch to inform the controller of a **change in port status**|
|**Packet-in**|Recall from Section 4.4 that a packet arriving at a switch port and **not matching** any flow table entry is sent to the controller for additional processing. Matched packets may also be sent to the controller, as an action to be taken on a match. The **Packet-in** message is used to send such packets to the controller|

> **Why this message set matters architecturally:** Notice that this exact set of messages is what makes the "southbound communication layer" from Figure 5.15 concrete. Every one of the four key SDN characteristics discussed earlier depends on this channel actually existing: flow-based forwarding needs Modify-State to install rules; the separated control plane needs Read-State and Flow-Removed to stay synchronized with data-plane reality; external control functions need this entire protocol to reach across the network to remote switches at all; and a programmable control plane needs Packet-in to let network-control applications react to genuinely novel traffic they haven't seen a rule for yet.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**The southbound OpenFlow channel is a single communication path carrying all controller-to-switch commands**|Intercepting or spoofing OpenFlow messages (Modify-State in particular) could let an attacker rewrite flow tables across the network — an escalation of the same SDN-controller compromise risk already flagged in Sections 4.4 and 5.1, now concretized to a specific TCP port (6653) and message set|OpenFlow deployments need TLS/TCP-layer protection on the controller-switch channel; port 6653 traffic should be treated as maximally sensitive control-plane traffic, not general data traffic|
|**Packet-in messages forward unmatched packets to the controller for inspection**|An attacker could potentially craft a flood of packets deliberately designed to miss existing flow table entries, forcing a storm of Packet-in messages to the controller — a control-plane-targeted denial-of-service vector distinct from a typical data-plane flood|Rate-limiting Packet-in generation, and ensuring flow tables have sensible default/wildcard entries to avoid unnecessary controller escalation, mitigates this specific amplification path|
|**"Logically centralized, physically distributed" controllers must solve distributed-systems consensus problems**|Exploiting inconsistency windows between the multiple physical servers implementing one logical controller (before consensus is reached) could let an attacker exploit a stale or conflicting view of network state at different servers|Controller platforms (ONOS, ORION) explicitly engineer for logical time ordering, consistency, and consensus — this is treated as a first-class distributed-systems security and correctness concern, not an afterthought|
|**The "unbundling" of switches/controller/apps across vendors widens the supply-chain trust boundary**|A compromised or malicious network-control application — potentially from an entirely different vendor than the controller or the switches — could abuse its northbound API access (read/write network state, flow tables) to install malicious forwarding rules network-wide|Northbound API access should be scoped and authenticated per-application, following least-privilege principles, precisely because SDN's vendor-unbundling model means applications can no longer be implicitly trusted just because they share a vendor with the switches|

---

## Questions I Still Have

- [ ] The northbound REST API lets applications register for state-change notifications — is there a standard mechanism for resolving conflicting instructions if two different network-control applications (say, a routing app and a load-balancing app) try to write conflicting flow-table entries for the same traffic at the same time?
- [ ] Given that OpenFlow operates over plain TCP by default (port 6653) — is TLS/encryption for this channel optional, a later addition, or built into the base OpenFlow spec from the start? The note doesn't specify.
- [ ] For "logically centralized, physically distributed" controllers like ONOS and ORION — do they use a specific named consensus protocol (Raft, Paxos) explicitly, or a custom-built equivalent, and how does that choice affect controller failover time during a server outage?
- [ ] The Packet-in mechanism sends unmatched packets to the controller — at very high line rates (recalling Section 4.2's nanosecond-scale budgets), does this create an inherent scalability ceiling on how much "genuinely novel" traffic an SDN network can handle before the controller becomes the bottleneck?
- [ ] With network-control applications potentially coming from different vendors than the controller itself — is there a standard certification/vetting process for northbound API access (analogous to how OS-level app stores vet apps), or is this currently left to each individual network operator's internal trust policies?

---

## Key Terms — Quick Reference

| Term                                          | Definition                                                                                                                                                                                                               |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **SDN control plane**                         | The network-wide logic that controls packet forwarding among a network's SDN-enabled devices, as well as their configuration and management                                                                              |
| **Flow-based forwarding**                     | Forwarding decisions made on the basis of any number of header field values across transport, network, and link layers, rather than solely on destination IP address                                                     |
| **SDN controller (network operating system)** | The software component maintaining accurate network state and providing APIs (northbound and southbound) through which network-control applications can monitor, program, and control the network                        |
| **Network-control application**               | Software running atop the SDN controller (via its northbound API) that implements specific network functions such as routing, access control, or load balancing — described as the control plane's "brains"              |
| **Northbound API / interface**                | The interface between the SDN controller and network-control applications, allowing apps to read/write network state and flow tables and register for event notifications                                                |
| **Southbound API / interface**                | The interface between the SDN controller and the controlled data-plane switches/devices, over which protocols like OpenFlow operate                                                                                      |
| **Unbundling (of network functionality)**     | The SDN-driven separation of data-plane switches, controllers, and network-control applications into independently sourced components, contrasted with the pre-SDN monolithic, vertically-integrated switch/router model |
| **OpenFlow protocol**                         | The earliest and most widely implemented southbound communication protocol between an SDN controller and controlled switches, operating over TCP on default port 6653                                                    |
| **Configuration (OpenFlow message)**          | Controller-to-switch message allowing the controller to query/set a switch's configuration parameters                                                                                                                    |
| **Modify-State (OpenFlow message)**           | Controller-to-switch message used to add/delete/modify flow table entries and switch port properties                                                                                                                     |
| **Read-State (OpenFlow message)**             | Controller-to-switch message used to collect statistics and counters from the switch's flow table and ports                                                                                                              |
| **Send-Packet (OpenFlow message)**            | Controller-to-switch message instructing the switch to send a specific packet out a specified port                                                                                                                       |
| **Flow-Removed (OpenFlow message)**           | Switch-to-controller message informing the controller that a flow table entry has been removed (via timeout or a Modify-State message)                                                                                   |
| **Port-status (OpenFlow message)**            | Switch-to-controller message informing the controller of a change in port status                                                                                                                                         |
| **Packet-in (OpenFlow message)**              | Switch-to-controller message forwarding a packet that either matched no flow table entry, or matched an entry whose action specifies forwarding to the controller                                                        |
| **Logically centralized (controller)**        | The controller is accessed and viewed as a single monolithic service, while its actual implementation is distributed across multiple physical servers for fault tolerance and scalability                                |

---

## Related Concepts

---

→ Next: [[5.5.3 The OpenDaylight (ODL) Controller]]