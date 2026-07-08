---
title: NETWORK MANAGEMENT
date: 2026-07-08
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 5.7 Network Management and SNMP, NETCONF/YANG

> **One-Line Summary:** **Network management** is the broad discipline of deploying, monitoring, configuring, and controlling all the hardware, software, and human elements of a network to meet real-time performance and QoS goals at reasonable cost — realized through a three-part **framework** (a centralized **managing server**, distributed **managed devices** each running a **network management agent**, and a **network management protocol** connecting them) that has historically been implemented via **SNMP/MIB** (a simple, device-agnostic query/set protocol paired with structured device-state objects) and, more recently, via the more configuration-focused, transactional, XML/RPC-based **NETCONF/YANG** approach.

---

## Core Idea: Taming Complexity at Scale

By this point in the network layer's study, it's clear that a network consists of many complex, interacting pieces of hardware and software — links, switches, routers, hosts, and the many protocols that control and coordinate these devices. When hundreds or thousands of such components are brought together by an organization to form a network, keeping that network "up and running" is a genuine challenge. Section 5.5 showed that a logically centralized SDN controller can help with this in an SDN context — but the broader challenge of **network management** predates SDN by decades, with its own rich set of tools and approaches that have co-evolved alongside SDN rather than being replaced by it.

### The Formal Definition

An often-asked question is "What is network management?" A well-conceived, single-sentence (albeit rather long, run-on) definition from [Saydam 1996] is:

> _Network management includes the deployment, integration, and coordination of the hardware, software, and human elements to monitor, test, poll, configure, analyze, evaluate, and control the network and element resources to meet the real-time, operational performance, and Quality of Service requirements at a reasonable cost._

> **Analogy — Running a Hospital, Not Just a Single Ward:** A single router is like one hospital ward — manageable by direct observation. But an entire network is like an entire hospital system: hundreds of wards (devices), each with its own equipment, staff, and status, all needing centralized oversight (the managing server / NOC) so that administrators can monitor vitals (device statistics), issue orders (configuration commands), and get paged immediately when something goes wrong (traps/notifications) — all without an administrator having to physically visit every ward every time something changes.

This section deliberately covers only the **rudiments** of network management — the architecture, protocols, and data used by a network administrator in performing their task. It does **not** cover the administrator's decision-making processes (fault identification, anomaly detection, network design/engineering to meet SLAs) — those are treated as separate, more advanced topics in the broader literature.

---

## 5.7.1 The Network Management Framework

Figure 5.20 shows the key components of network management — a framework consisting of four core pieces:

### 1. Managing Server

The **managing server** is an application, typically with **network managers** (humans) in the loop, running in a centralized **network management station** in the **network operations center (NOC)**. The managing server is the **locus of activity** for network management: it controls the collection, processing, analysis, and dispatching of network management information and commands. It is here that actions are initiated to **configure, monitor, and control** the network's managed devices. In practice, a network may have several such managing servers.

### 2. Managed Device

A **managed device** is a piece of network equipment (including its software) that resides on a managed network. A managed device might be a host, router, switch, middlebox, modem, thermometer, or other network-connected device. The device itself will have many manageable **components** (e.g., a network interface is one component of a host or router), and **configuration parameters** for these hardware and software components (e.g., an intra-AS routing protocol, such as OSPF).

### 3. Data

Each managed device has data, also known as **"state,"** associated with it. There are several types:

|Data Type|Description|
|---|---|
|**Configuration data**|Device information explicitly configured by the network manager — e.g., a manager-assigned/configured IP address or interface speed for a device interface|
|**Operational data**|Information that the device acquires as it operates — e.g., the list of immediate neighbors in the OSPF protocol|
|**Device statistics**|Status indicators and counts updated as the device operates — e.g., the number of dropped packets on an interface, or the device's cooling fan speed|

The network manager can **query** remote device data, and in some cases **control** the remote device by **writing** device data values. The managing server also maintains its own copy of configuration, operational, and statistics data from its managed devices, as well as network-wide data (e.g., the network's topology).

### 4. Network Management Agent and Protocol

The **network management agent** is a software process running in the managed device that communicates with the managing server, taking local actions at the managed device under the command and control of the managing server. (The agent is similar in spirit to the routing agent seen elsewhere in the network layer.)

The **network management protocol** is the final component — it runs between the managing server and the managed devices, allowing the managing server to **query the status of managed devices and take actions** at these devices via its agents. Agents can use the protocol to **inform** the managing server of exceptional events (e.g., component failures or violation of performance thresholds).

> **Important, subtle distinction:** The network management protocol does **not itself manage** the network. Instead, it provides the _capabilities_ that network managers can use to manage ("monitor, test, poll, configure, analyze, evaluate, and control") the network. The intelligence and decision-making remain with the human managers and their tools — the protocol is just the communication substrate.

```
Fig 5.20 -- Network Management Framework
────────────────────────────────────────

   Network Managers (humans, in NOC)
                │
                ▼
   ┌─────────────────────────┐
   │   Managing Server /       │
   │      Controller            │
   │  (config + operational     │
   │   data store)               │
   └───────────┬─────────────┘
               │ Controller-to-device
               │ protocol (SNMP /
               │ NETCONF)
     ┌─────────┼─────────┬─────────┐
     ▼         ▼         ▼         ▼
  ┌──────┐ ┌──────┐  ┌──────┐  ┌──────┐
  │Agent │ │Agent │  │Agent │  │Agent │
  │Device│ │Device│  │Device│  │Device│
  │ data │ │ data │  │ data │  │ data │
  └───┬──┘ └───┬──┘  └───┬──┘  └───┬──┘
      ▼        ▼         ▼         ▼
   Managed  Managed   Managed   Managed
   Device   Device    Device    Device
   (host)   (switch)  (router)  (modem)
```

### Three Ways to Manage a Network in Practice

|Approach|Description|
|---|---|
|**CLI (Command Line Interface)**|A network operator issues direct CLI commands to the device — typed on a managed device's console if physically present, or via scripting over Telnet or secure shell (SSH) between the managing server/controller and the managed device. CLI commands are **vendor- and device-specific** and can be rather arcane. Seasoned network wizards can use CLI to flawlessly configure devices, but CLI is **prone to errors** and difficult to automate or efficiently scale for large networks. (Consumer-oriented devices, like a home wireless router, may export a management menu accessible via HTTP — this works well for single, simple devices but doesn't scale to larger networks either.)|
|**SNMP/MIB**|The network operator queries/sets data in a device's **Management Information Base (MIB)** using SNMP. Some MIBs are device- and vendor-specific; others (e.g., datagrams discarded at a router due to header errors, or UDP segments received at a host) are **device-agnostic**, providing abstraction and generality. A network operator would most typically use this approach to **query and monitor** operational state and device statistics, then use CLI to actively control/configure the device.|
|**NETCONF/YANG**|A more abstract, network-wide, and holistic approach, with a much stronger emphasis on **configuration management**, including specifying correctness constraints and providing **atomic management operations** over multiple controlled devices at once.|

It is important to note that **both SNMP and CLI approaches manage devices individually** — this individual-device focus is precisely one of the shortcomings that motivated the newer NETCONF/YANG approach.

A network-management workshop convened by the Internet Architecture Board in 2002 [RFC 3535] noted not only the value of the SNMP/MIB approach for device _monitoring_, but also noted its shortcomings, particularly for device _configuration_ and network management **at scale**. This gave rise to the most recent approach for network management, using **NETCONF and YANG**.

---

## 5.7.2 The Simple Network Management Protocol (SNMP) and the Management Information Base (MIB)

### What SNMP Is

The **Simple Network Management Protocol version 3 (SNMPv3)** [RFC 3410] is an **application-layer protocol** used to convey network-management control and information messages between a managing server and an agent executing on behalf of that managing server. SNMP has two main usage patterns:

1. **Request-response mode:** An SNMP managing server sends a **request** to an SNMP agent, who receives the request, performs some action, and sends a **reply** back. Typically, a request is used to **query (retrieve)** or **modify (set)** MIB object values associated with a managed device.
2. **Trap mode:** An agent sends an **unsolicited** message, known as a **trap**, to a managing server. Trap messages **notify** a managing server of an exceptional situation (e.g., a link interface going up or down) that has resulted in changes to MIB object values.

```
Fig -- SNMP Request-Response vs. Trap
────────────────────────────────────────

  MANAGING SERVER              AGENT
  (network manager)      (managed device)

  Request-response mode:
     │  GetRequest /            │
     │  SetRequest / etc.       │
     ├─────────────────────────▶│
     │                          │ performs
     │                          │ action
     │       Response           │
     │◀─────────────────────────┤

  Trap (unsolicited) mode:
     │                          │  link goes
     │                          │  up/down
     │     SNMPv2-Trap          │
     │◀─────────────────────────┤
     │  (no response required)  │
```

> **Analogy — A Smart Home System with a Check-In App and a Fire Alarm:** Request-response mode is like opening an app on your phone to _ask_ your smart thermostat what the current temperature is, or to _set_ a new target temperature — you initiate the exchange. Trap mode is like the smoke detector that doesn't wait to be asked — it proactively pushes a notification to your phone the instant something exceptional happens (smoke detected), with no expectation of a reply.

### The Management Information Base (MIB)

**MIB objects** are specified in a data description language known as **SMI (Structure of Management Information)** [RFC 2578; RFC 2579; RFC 2580] — a rather oddly named component of the network management framework whose name gives no hint of its functionality. SMI is a **formal definition language** used to ensure that the syntax and semantics of network management data are well-defined and unambiguous. Related MIB objects are gathered into **MIB modules**. As of late 2024, there are more than **400 MIB-related RFCs**, and a much larger number of vendor-specific (private) MIB modules.

While MIB-related RFCs make for rather tedious and dry reading, they are nonetheless instructive (much like eating vegetables, "it is good for you"). Consider a concrete example: the **`ipSystemStatsInDelivers`** object-type definition from [RFC 4293] — a 32-bit read-only counter that keeps track of the number of IP datagrams that were received at the managed device and successfully delivered to an upper-layer protocol.

```
Fig -- MIB Object Example (SMI-defined)
────────────────────────────────────────

  MIB Module (e.g., RFC 4293, for IP/ICMP)
    │
    ├── ipSystemStatsInDelivers
    │     SYNTAX: Counter32
    │     ACCESS: read-only
    │     STATUS: current
    │     "total IP datagrams delivered
    │      to upper-layer protocols"
    │
    ├── (other IP/ICMP counters...)
    │
  Each object = one measurable/
  configurable piece of device state
```

In this example, `Counter32` is one of the basic data types defined by the SMI. Two example MIB modules cited in this section:

- **[RFC 4293]** specifies the MIB module for managing implementations of the **Internet Protocol (IP)** and its associated **Internet Control Message Protocol (ICMP)**
- **[RFC 4022]** specifies the MIB module for **TCP**
- **[RFC 4113]** specifies the MIB module for **UDP**

### SNMPv3 PDU Types

SNMPv3 defines **seven types of messages**, known generically as **protocol data units (PDUs):**

|SNMPv3 PDU Type|Sender → Receiver|Description|
|---|---|---|
|**GetRequest**|manager → agent|Get the value of one or more MIB object instances|
|**GetNextRequest**|manager → agent|Get the value of the next MIB object instance in a list or table|
|**GetBulkRequest**|manager → agent|Get values in a large block of data — e.g., values in a large table|
|**InformRequest**|manager → manager|Inform a remote managing entity of MIB values remote to its access|
|**SetRequest**|manager → agent|Set the value of one or more MIB object instances|
|**Response**|agent → manager, or manager → manager|Generated in response to a `GetRequest`, `GetNextRequest`, `GetBulkRequest`, `SetRequest`, or `InformRequest`|
|**SNMPv2-Trap**|agent → manager|Inform the manager of an exceptional event|

These types differ in the **granularity** of their data requests:

- **`GetRequest`** can request an arbitrary set of MIB values
- Multiple **`GetNextRequest`**s can be used to sequence through a list or table of MIB objects
- **`GetBulkRequest`** allows a large block of data to be returned in one shot, avoiding the overhead that would be incurred if multiple `GetRequest` or `GetNextRequest` messages were sent instead

In all three retrieval cases, the agent responds with a **`Response`** PDU containing the value of the object identifiers and their associated values.

- The **`SetRequest`** PDU is used by a managing server to set the value of one or more MIB objects in a managed device. An agent replies with a `Response` PDU containing the "noError" status to confirm that the value has indeed been set.
- The **`InformRequest`** PDU is used by a managing server to notify **another managing server** of MIB information that is remote to the receiving server.
- The **`Response`** PDU is typically sent from a managed device to the managing server in response to a request message, returning the requested information.
- The final PDU type is the **trap message**. Trap messages are generated **asynchronously** — that is, they are not generated in response to a received request, but rather in response to an event for which the managing server requires notification. [RFC 3418] defines well-known trap types that include a **cold or warm start** by a device, a **link going up or down**, the **loss of a neighbor**, or an **authentication failure event**. A received trap request has no required response from a managing server.

### SNMP's Relationship with UDP

Given the request-response nature of SNMP, it is worth noting that although SNMP PDUs can be carried via many different transport protocols, the **SNMP PDU is typically carried in the payload of a UDP datagram**. Indeed, **RFC 3417 states that UDP is "the preferred transport mapping."** However, since UDP is an **unreliable** transport protocol, there is **no guarantee** that a request, or its response, will be received at the intended destination. Interestingly, the SNMP standard does **not mandate any particular procedure** for retransmission, or even if retransmission is to be done at all — it only requires that "the managing server needs to act responsibly in respect to the frequency and duration of retransmissions." This, of course, leads one to wonder how a "responsible" protocol should act!

### SNMP's Evolution Across Three Versions

SNMP has evolved through three versions. The designers of SNMPv3 have said that "SNMPv3 can be thought of as SNMPv2 with additional security and administration capabilities" [RFC 3410]. Certainly, there are changes in SNMPv3 over SNMPv2, but nowhere are those changes more evident than in the area of **administration and security**. The central role of security in SNMPv3 was particularly important, since the **lack of adequate security** resulted in SNMP being used primarily for **monitoring rather than control** (for example, `SetRequest` is rarely used in SNMPv1). Once again, security — a topic covered in detail elsewhere — is of critical concern, and once again a concern whose importance had been realized perhaps a bit late and only then "added on."

---

## 5.7.3 The Network Configuration Protocol (NETCONF) and YANG

### Why NETCONF Emerged

The SNMP/MIB approach's device-by-device, individually-managed nature, along with its comparatively weak configuration-management capabilities, motivated a newer approach. NETCONF and YANG take **a more abstract, network-wide, and holistic view** toward network management, with a much stronger emphasis on **configuration management**, including specifying **correctness constraints** and providing **atomic management operations over multiple controlled devices**.

- **YANG** [RFC 6020] is a **data modeling language** used to model configuration and operational data.
- **NETCONF** [RFC 6241] is the **protocol** used to communicate YANG-compatible actions and data to/from/among remote devices.

### How NETCONF Works

The **NETCONF protocol** operates between the managing server and the managed network devices, providing messaging to:

1. **Retrieve, set, and modify** configuration data and statistics at managed devices
2. **Query** operational data and statistics at managed devices
3. **Subscribe** to notifications generated by managed devices

The managing server **actively controls** a managed device by sending it configurations, which are specified in a **structured XML document**, and activating a configuration at the managed device. NETCONF uses a **remote procedure call (RPC)** paradigm, where protocol messages are encoded in XML and exchanged between the managing server and a managed device over a **secure, connection-oriented session** such as the **TLS (Transport Layer Security)** protocol over TCP.

> **Terminology note:** In NETCONF parlance, the managing server is actually referred to as the **"client,"** and the managed device as the **"server"** — since the managing server establishes the connection to the managed device. (This is somewhat confusing and inverted relative to the longer-standing network-management server/agent terminology used elsewhere in this framework — for consistency, this document continues to use "managing server" and "managed device.")

### A Worked Example: The NETCONF Session

Figure 5.21 shows an example NETCONF session:

1. First, the managing server establishes a **secure connection** to the managed device.
2. Once a secure connection has been established, the managing server and managed device exchange **`<hello>`** messages, declaring their **"capabilities"** — NETCONF functionality that supplements the base NETCONF specification in [RFC 6241].
3. Interactions between the managing server and managed device take the form of a **remote procedure call**, using the **`<rpc>`** and **`<rpc-response>`** messages. These messages are used to retrieve, set, query, and modify device configurations, operational data and statistics, and to **subscribe to device notifications**.
4. Device notifications themselves are proactively sent from a managed device to the managing server using NETCONF **`<notification>`** messages.
5. A session is closed with the **`<close-session>`** message.

```
Fig 5.21 -- NETCONF Session Sequence
────────────────────────────────────────

  Managing Server /            Agent /
     Controller            Managed Device
       │                          │
       │   secure TLS-over-TCP    │
       │◀────────connection──────▶│
       │                          │
       │   <hello> (capabilities) │
       ├─────────────────────────▶│
       │        <hello>           │
       │◀─────────────────────────┤
       │                          │
       │        <rpc>             │
       ├─────────────────────────▶│
       │     <rpc-reply>          │
       │◀─────────────────────────┤
       │           ⋮               │
       │        <rpc>             │
       ├─────────────────────────▶│
       │     <rpc-reply>          │
       │◀─────────────────────────┤
       │                          │
       │    <notification>        │
       │◀─────────────────────────┤
       │                          │
       │        <rpc>             │
       ├─────────────────────────▶│
       │     <rpc-reply>          │
       │◀─────────────────────────┤
       │                          │
       │   <close-session>        │
       ├─────────────────────────▶│
```

### Selected NETCONF Operations

|NETCONF Operation|Description|
|---|---|
|**`<get-config>`**|Retrieve all or part of a given configuration. A device may have multiple configurations; there is always a **`running`** configuration that describes the device's current (running) configuration|
|**`<get>`**|Retrieve all or part of **both** configuration state and operational state data|
|**`<edit-config>`**|Change all or part of a specified configuration at the managed device. If the `running` configuration is specified, then the device's current (running) configuration will be changed. If the managed device was able to satisfy the request, an `<rpc-reply>` is sent containing an `<ok>` element; otherwise an `<rpc-error>` response is returned. On error, the device's configuration state **can be rolled back** to its previous state|
|**`<lock>`, `<unlock>`**|Allows the managing server to lock (unlock) the entire configuration datastore system of a managed device. Locks are intended to be short-lived and allow a client to make a change without fear of interaction with other NETCONF, SNMP, or CLI commands from other sources|
|**`<create-subscription>`, `<notification>`**|Initiates an event notification subscription that will send asynchronous event `<notification>`s for specified events of interest from the managed device to the managing server, until the subscription is terminated|

As with SNMP, NETCONF provides operations for retrieving operational state data (`<get>`) and for event notification. However, the **`<get-config>`, `<edit-config>`, `<lock>`, and `<unlock>`** operations demonstrate NETCONF's particular emphasis on device **configuration**. Using these basic operations, it is possible to create a **set** of more sophisticated network management transactions that either **complete atomically** (i.e., as a group) on a set of devices, or are **fully reversed** and leave the devices in their pre-transaction state. Such multi-device transactions — "enabl[ing] operators to concentrate on the _configuration_ of the network as a whole rather than individual devices" — was an important operator requirement put forth in [RFC 3535].

### YANG: The Data Modeling Language Underneath NETCONF

**YANG** is the data modeling language used to precisely specify the **structure, syntax, and semantics** of network management data used by NETCONF, in much the same way that the **SMI is used to specify MIBs in SNMP**. All YANG definitions are contained in **modules**, and an XML document describing a device and its capabilities can be generated from a YANG module.

YANG features a small set of built-in data types (as does SMI), and also allows data modelers to express **constraints** that must be satisfied by a valid NETCONF configuration — a powerful aid in helping ensure that NETCONF configurations satisfy specified correctness and consistency constraints. YANG is also used to specify NETCONF notifications.

> **Analogy — SNMP/MIB/SMI vs. NETCONF/YANG as "Reading a Dashboard" vs. "Editing a Blueprint":** SNMP with its MIB objects is like reading gauges on a car's dashboard — great for glancing at individual readouts (speed, fuel, temperature) and occasionally flipping a single switch, one at a time, with little protection against inconsistent states. NETCONF with YANG is more like handing an architect a validated, constraint-checked blueprint for renovating an entire building at once — you specify the complete desired configuration, the system checks it against structural rules (YANG constraints) before applying it, and if something goes wrong partway through, the whole change can be rolled back atomically rather than leaving the building half-renovated.

|Dimension|SNMP / MIB / SMI|NETCONF / YANG|
|---|---|---|
|**Primary strength**|Monitoring (querying operational state, statistics)|Configuration management|
|**Data description language**|SMI (Structure of Management Information)|YANG|
|**Transport**|Typically UDP (unreliable; "preferred" per RFC 3417, not mandated)|Secure, connection-oriented (TLS over TCP)|
|**Message encoding**|Compact binary PDUs|Structured XML, RPC paradigm|
|**Device scope**|Manages devices individually|Supports network-wide, multi-device atomic transactions|
|**Correctness guarantees**|Minimal — no built-in rollback|Constraint validation, rollback on error via `<edit-config>`|
|**Historical era**|In use since the 1980s|Emerged after 2002 IAB workshop identified SNMP's configuration shortcomings|

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**SNMP historically had weak security (especially SNMPv1/v2)**|Because early SNMP versions offered little authentication, an attacker who could reach a device's SNMP agent could potentially read sensitive operational data, or in the worst case issue `SetRequest` commands to reconfigure the device|SNMPv3's most significant changes over SNMPv2 were precisely in security and administration; best practice is to disable SNMPv1/v2 entirely, use SNMPv3 with authentication/encryption, and restrict `SetRequest` capability tightly, since write access to MIB objects is direct write access to live device configuration|
|**SNMP is typically carried over UDP with no mandated retransmission or delivery guarantee**|An attacker on-path could potentially spoof UDP-based SNMP responses or drop/delay legitimate traps (e.g., suppressing a "link down" trap to hide an attack in progress), since SNMP itself provides no transport-layer integrity guarantee|Running SNMP traffic over authenticated, encrypted channels, and monitoring for missing/anomalous trap patterns, mitigates spoofing and drop-based evasion|
|**The managing server is a single, high-value point of control over the entire managed network**|A compromised managing server (or a stolen set of SNMP community strings / NETCONF credentials) gives an attacker centralized read/write access to every managed device's configuration and data — a single breach with network-wide blast radius, structurally similar to the SDN-controller risk discussed in Section 5.5|The managing server/NOC should be hardened as a critical asset: strong authentication, network segmentation limiting which hosts can reach management interfaces, and audit logging of all configuration changes|
|**NETCONF's atomic, network-wide transactions are powerful but also higher-stakes**|If an attacker gains NETCONF access, a single malicious `<edit-config>` transaction could reconfigure many devices simultaneously and consistently — turning what would have been many separate individual mistakes/attacks (under CLI/SNMP) into one coordinated, network-wide change|NETCONF's `<lock>`/`<unlock>` mechanism and rollback-on-error behavior should be paired with strict access control and change-approval workflows, since the same atomicity that protects against partial misconfiguration also means a malicious transaction succeeds or fails as a coordinated whole|
|**MIB objects can reveal detailed operational and topology information**|Read access to MIB data (e.g., interface counters, neighbor lists, routing information) can give an attacker reconnaissance-grade visibility into network structure and load, without needing to compromise any device directly|MIB read access (via SNMP community strings or SNMPv3 credentials) should be treated as sensitive and scoped only to authorized management stations|

---

## Questions I Still Have

- [ ] The text says SNMP's standard doesn't mandate any retransmission procedure, only that the managing server act "responsibly" — in practice, what retransmission/timeout strategies do real-world SNMP implementations actually use, and how do they avoid overwhelming an already-struggling device?
- [ ] Given that NETCONF uses XML/RPC over TLS while SNMP traditionally runs over unreliable UDP, is there a newer transport-encoding variant of NETCONF-style management (e.g., using protocols like RESTCONF or gNMI/gRPC) that has since become more common in practice, and how does it compare?
- [ ] How exactly does a YANG "constraint" get enforced at `<edit-config>` time — is validation done entirely client-side before sending the RPC, entirely device-side upon receipt, or both?
- [ ] The chapter notes MIB modules are "device-agnostic" in some cases (e.g., IP-datagram-discard counters) but "device- and vendor-specific" in others — how does a network manager know, without deep vendor-specific documentation, which MIB modules will actually be supported by a given piece of equipment?
- [ ] How does the NETCONF/YANG approach interact with, or get subsumed by, the SDN controller's northbound/southbound architecture from Section 5.5 — are they typically layered together in practice, or considered separate/competing management planes?

---

## Key Terms — Quick Reference

| Term                                             | Definition                                                                                                                                                                                                                                          |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Network management**                           | The deployment, integration, and coordination of hardware, software, and human elements to monitor, test, poll, configure, analyze, evaluate, and control network resources to meet real-time, operational, and QoS requirements at reasonable cost |
| **Managing server**                              | A centralized application (with human network managers in the loop) that is the locus of activity for network management — collects, processes, analyzes, and dispatches management information/commands                                            |
| **Managed device**                               | A piece of network equipment (host, router, switch, middlebox, etc.) that resides on a managed network and has configuration, operational, and statistics data associated with it                                                                   |
| **Configuration data**                           | Device information explicitly set by the network manager (e.g., a configured IP address)                                                                                                                                                            |
| **Operational data**                             | Information a device acquires as it operates (e.g., OSPF neighbor list)                                                                                                                                                                             |
| **Device statistics**                            | Status indicators/counts updated during device operation (e.g., dropped packet counts)                                                                                                                                                              |
| **Network management agent**                     | A software process running on a managed device that communicates with the managing server and takes local actions under its command                                                                                                                 |
| **Network management protocol**                  | The protocol (SNMP or NETCONF) that runs between the managing server and managed devices' agents to query status and take actions                                                                                                                   |
| **CLI (Command Line Interface)**                 | Direct, vendor/device-specific commands issued to a device console or via Telnet/SSH; powerful but error-prone and hard to scale/automate                                                                                                           |
| **SNMP (Simple Network Management Protocol)**    | An application-layer protocol (currently SNMPv3) for conveying management control and information messages between a managing server and agents, in request-response and trap modes                                                                 |
| **MIB (Management Information Base)**            | The structured collection of data objects (organized into modules) representing a managed device's configuration/operational state, queried and set via SNMP                                                                                        |
| **SMI (Structure of Management Information)**    | The formal data description language used to define MIB objects unambiguously                                                                                                                                                                       |
| **GetRequest / GetNextRequest / GetBulkRequest** | SNMPv3 PDUs sent manager→agent to retrieve MIB object values, individually, sequentially, or in bulk, respectively                                                                                                                                  |
| **SetRequest**                                   | SNMPv3 PDU sent manager→agent to set the value of one or more MIB objects                                                                                                                                                                           |
| **InformRequest**                                | SNMPv3 PDU sent manager→manager to notify another managing server of remote MIB information                                                                                                                                                         |
| **Response**                                     | SNMPv3 PDU returned in reply to Get/Set/Inform requests                                                                                                                                                                                             |
| **SNMPv2-Trap**                                  | An asynchronous, unsolicited SNMPv3 PDU sent agent→manager to report an exceptional event (e.g., link up/down, cold/warm start, authentication failure)                                                                                             |
| **NETCONF (Network Configuration Protocol)**     | An XML/RPC-based protocol operating between managing server and managed devices, emphasizing configuration retrieval, modification, and atomic multi-device transactions, typically run over TLS/TCP                                                |
| **YANG**                                         | The data modeling language used to specify the structure, syntax, semantics, and constraints of NETCONF-managed configuration and operational data                                                                                                  |
| **`<get-config>` / `<get>` / `<edit-config>`**   | Core NETCONF operations to retrieve configuration, retrieve config+operational data, and change configuration (with rollback on error), respectively                                                                                                |
| **`<lock>` / `<unlock>`**                        | NETCONF operations allowing a managing server to reserve exclusive short-term access to a device's configuration datastore                                                                                                                          |
| **`<create-subscription>` / `<notification>`**   | NETCONF mechanism for subscribing to, and receiving, asynchronous event notifications from a managed device                                                                                                                                         |

---

## Related Concepts

---

→ Next: [[ICMP]]