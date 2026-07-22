---
title: DATA CENTER NETWORKING
date: 2026-07-22
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 6.6 Data Center Networking

> **One-Line Summary:** A **data center network** is the complex internal network that interconnects the tens to hundreds of thousands of **hosts (blades)** inside a massive **data center**, organized into **racks** under **Top-of-Rack (TOR) switches**, structured according to an evolving family of topologies — from simple **hierarchical** trees, to highly interconnected **three-tier Clos networks**, to today's **leaf-spine** designs — all engineered around the twin **imperatives (pressing, unavoidable requirements)** of **bandwidth at scale** and **east-west (server-to-server) traffic**, and increasingly shaped by trends toward **centralized SDN control**, **virtualization**, **physical constraints**, **hardware modularity**, and **energy efficiency**.

---

## Core Idea: The Network Behind the Cloud

Internet companies such as **Google, Microsoft, Amazon, and Alibaba** have built **massive data centers**, each housing **tens to hundreds of thousands** of hosts. As briefly touched on in Section 1.2, data centers are not only connected to the Internet, but also internally include **complex computer networks**, called **data center networks**, which interconnect their internal hosts. This section provides a brief introduction to data center networking for cloud applications.

### Three Purposes a Data Center Serves

Broadly speaking, data centers serve **three purposes**:

|Purpose|Description|
|---|---|
|**1. Content delivery**|They provide content such as **Web pages, search results, AI chatbots, e-mail, or streaming video** to users|
|**2. Massively-parallel computing**|They serve as **massively-parallel computing infrastructures** for specific data-processing tasks, such as **distributed index computations** for search engines|
|**3. Cloud computing**|They provide **cloud computing** to other companies|

Indeed, today a major trend in computing is for companies to use a **cloud provider** — such as **Amazon Web Services, Microsoft Azure, and Alibaba Cloud** — to handle **essentially all** of their IT needs, rather than **provisioning (setting up and maintaining)** their own infrastructure.

---

## 6.6.1 Data Center Architectures

Data center designs are carefully kept **company secrets**, as they often provide **critical competitive advantages** to leading cloud-computing companies. In 2024, the cost to build a large-scale data center can range from **many hundreds of millions of dollars to more than a billion dollars**.

### Where the Money Goes

|Cost Category|Approximate Share|Details|
|---|---|---|
|**Hosts themselves**|~45%|Need to be **replaced every 3–4 years**|
|**Infrastructure**|~25%|Transformers, **uninterruptable power supplies (UPS)** systems, generators for long-term outages, and cooling systems|
|**Electric utility costs**|~15%|For the raw **power draw**|
|**Networking**|~15%|Network gear (switches, routers, and load balancers), external links, and transit traffic costs|

_(In these percentages, equipment costs are **amortized (spread out over time)** so that a common cost metric applies to both one-time purchases and ongoing expenses such as power.)_ While networking is **not** the largest cost category, networking **innovation** is nonetheless the **key** to reducing overall cost and maximizing performance [Greenberg 2009a].

### The Worker Bees: Hosts, Racks, and TOR Switches

**The worker bees in a data center are the hosts.** The hosts in data centers — called **blades**, and resembling pizza boxes in shape — are generally **commodity hosts** that include CPU, memory, and disk storage. The hosts are **stacked in racks**, with each rack typically having **20 to 40 blades**. At the top of each rack, there is a switch, aptly named the **Top of Rack (TOR) switch**, that **interconnects** the hosts in the rack with each other and with other switches in the data center.

```
Fig -- Inside a Rack: Blades + TOR Switch
────────────────────────────────────────
        ┌────────────────┐
        │  TOR switch     │◀─ uplinks to
        │ (Top of Rack)   │   other switches
        └───┬───┬───┬────┘
            │   │   │
        ┌───▼┐┌─▼──┐┌▼───┐
        │Blade││Blade││...│  20-40 blades
        │(CPU,││(CPU,││   │  per rack, each
        │mem, ││mem, ││   │  its own data-
        │disk)││disk)││   │  center-internal
        └─────┘└─────┘└───┘  IP address
```

Specifically, each host in the rack has a **network interface** that connects to its TOR switch, and each TOR switch has additional ports that can be connected to other switches. Today, hosts typically have **40 Gbps or 100 Gbps** Ethernet connections to their TOR switches [FB 2019; Greenberg 2015; Roy 2015; Singh 2015]. Each host is also assigned its own **data-center-internal IP address**.

### Two Types of Traffic, and Border Routers

The data center network supports **two types of traffic**:

1. Traffic flowing **between external clients and internal hosts**.
2. Traffic flowing **between internal hosts**.

To handle flows between external clients and internal hosts, the data center network includes one or more **border routers**, connecting the data center network to the **public Internet**. The data center network therefore interconnects the racks with each other **and** connects the racks to the border routers. **Data center network design** — the art of designing the interconnection network and protocols that connect the racks with each other and with the border routers — has become an important branch of computer networking research in recent years.

### Load Balancing

A cloud data center — such as one operated by Google, Microsoft, Amazon, or Alibaba — provides **many applications concurrently** (e.g., search, e-mail, and video applications). To support requests from external clients, each application is associated with a **publicly visible IP address** to which clients send their requests and from which they receive responses.

Inside the data center, external requests are **first directed to a load balancer** whose job is to **distribute** requests to the hosts, balancing the load across the hosts as a function of their current load [Patel 2013; Eisenbud 2016]. A large data center will often have **several load balancers**, each **devoted to** a set of specific cloud applications. Such a load balancer is sometimes referred to as a "**layer-4 switch**," since it makes decisions based on the **destination port number** (layer 4) as well as the destination IP address in the packet.

Upon receiving a request for a particular application, the load balancer **forwards** it to one of the hosts that handles that application. (A host may then **invoke the services of other hosts** to help process the request.) The load balancer not only balances the workload across hosts, but also provides a **NAT-like function** — translating the public external IP address to the internal IP address of the appropriate host, and translating packets back in the reverse direction to the clients.

**Security benefit:** this **prevents clients from contacting hosts directly**, which has the security benefit of **hiding the internal network structure** and preventing clients from directly interacting with the hosts.

### Hierarchical Architecture

For a **small** data center housing only a few thousand hosts, a simple network consisting of a border router, a load balancer, and a few tens of racks all interconnected by a single Ethernet switch could possibly suffice. But to scale to tens to hundreds of thousands of hosts, a data center often employs a **hierarchy** of routers and switches.

```
Fig -- Hierarchical Data Center Topology
────────────────────────────────────────
              Internet
                 │
           Border router
             /        \
      Access router  Access router
        /      \         \
     Tier-1   Tier-1    Tier-1
     switch   switch    switch
      / \       |          \
   Tier-2 Tier-2 Tier-2   Tier-2
     |      |      |         |
    TOR    TOR    TOR       TOR
     |      |      |         |
   Rack1  Rack5  Rack3     Rack7
 (load balancer sits near
  access-router level)
```

![[Pasted image 20260722212059.png]]

At the **top** of the hierarchy, the **border router** connects to **access routers** (only two are shown in the figure, but there can be many more). Below each access router, there are **three tiers of switches**: each access router connects to a **top-tier** switch, and each top-tier switch connects to multiple **second-tier** switches and a load balancer. Each second-tier switch, in turn, connects to multiple racks via the racks' **TOR switches** (third-tier switches). All links typically use **Ethernet** for their link-layer and physical-layer protocols, with a mix of **copper and fiber cabling**. With such a hierarchical design, it's possible to scale a data center to **hundreds of thousands of hosts**.

Because it's **critical** for a cloud application provider to continually provide applications with **high availability**, data centers also include **redundant** network equipment and redundant links in their designs [Cisco 2012; Greenberg 2009b]. For example, each TOR switch can connect to **two** tier-2 switches, and each access router, tier-1 switch, and tier-2 switch can be **duplicated and integrated** into the design. Also observe that the hosts below each access router form a **single subnet** — in order to **localize** ARP broadcast traffic, each of these subnets is further **partitioned** into smaller **VLAN subnets**, each comprising a few hundred hosts [Greenberg 2009a].

### The Limitation: Restricted Host-to-Host Capacity

Although the conventional hierarchical architecture just described solves the problem of scale, it **suffers from limited host-to-host capacity** [Greenberg 2009b]. To understand this limitation, consider again the hierarchical topology, and suppose each host connects to its TOR switch with a **10 Gbps** link, whereas the links **between switches** are **100 Gbps** Ethernet links. Two hosts in the **same rack** can always communicate at a full **10 Gbps**, limited only by the rate of the hosts' own network interface controllers.

**However**, if there are **many simultaneous flows** in the data center network, the maximum rate **between hosts in different racks** can be much less.

```
Fig -- Bandwidth Bottleneck Higher Up
────────────────────────────────────────
 Each host <-> TOR link: 10 Gbps
 Switch <-> switch links: 100 Gbps

 40 flows from Rack1 hosts all cross
 the SAME 100 Gbps link toward Rack5:
   100 Gbps / 40 flows = 2.5 Gbps each

 ...far below the 10 Gbps each host's
 own network interface could deliver!
 Bottleneck = shared links higher up
 the hierarchy, not the host NICs.
```

**Worked example:** consider a traffic pattern consisting of **40 simultaneous flows** between 40 pairs of hosts in different racks — say, each of 10 hosts in rack 1 sends a flow to a corresponding host in rack 5, with similar simultaneous flows between racks 2 & 6, racks 3 & 7, and racks 4 & 8. If each flow **evenly shares** a link's capacity with other flows traversing that link, then the 40 flows crossing the shared 100 Gbps link will **each** receive only **100 Gbps / 40 = 2.5 Gbps** — significantly less than the 10 Gbps native network interface rate! The problem becomes even more **acute (severe, pronounced)** for flows between hosts that need to travel even **higher up** the hierarchy.

### Three Possible Solutions to Limited Capacity

|Solution|Description|Trade-off|
|---|---|---|
|**Higher-rate switches/routers**|Deploy higher-rate switches and routers throughout the hierarchy|Significantly **increases the cost** of the data center, since high-port-speed switches and routers are very expensive|
|**Co-locate related services/data**|**Co-locate (place close together)** related services and data as close to one another as possible — e.g., within the same rack, or a nearby rack — to minimize inter-rack communication via tier-2 or tier-1 switches [Roy 2015; Singh 2015]|Can only go **so far**, as a key requirement in data centers is **flexibility** in the placement of computation and services [Greenberg 2009b; Farrington 2010]. Search engines and cloud services (like AWS/Azure placing a customer's VMs) may need to spread across multiple racks irrespective of location — otherwise network bottlenecks can result in poor performance.|
|**Increased connectivity between tiers**|Provide **increased connectivity** between TOR switches (often called **access switches**), tier-2 switches (often called **aggregation switches**), and tier-1 switches (often called **core switches**)|Requires a more elaborate, highly interconnected topology (see below)|

---

### Increased Connectivity: The Three-Tier, Highly Interconnected Topology

For example, each TOR switch could be connected to **two** tier-2 switches, which then provide **multiple link- and switch-disjoint (non-overlapping) paths** between racks. In such a design, there could be **four distinct paths** between a given first tier-2 switch and a given second tier-2 switch — together providing an **aggregate capacity of 400 Gbps** between just those two tier-2 switches.

```
Fig -- Three-Tier, Highly Interconnected
────────────────────────────────────────
        Core switches (top)
        /   |    |    \
   Aggregation switches (mesh-
   connected to MULTIPLE core
   and MULTIPLE access switches)
        \   |    |    /
        Access switches
        /   |    |    \
   Server racks 1..16

 Multiple link-disjoint paths exist
 between any two racks -> multipath
 routing (e.g. ECMP) becomes possible.
```

![[Pasted image 20260722212257.png]]

Increasing the **degree of connectivity** between tiers has **two significant benefits**: there's both **increased capacity** and **increased reliability** (because of **path diversity (multiple independent routes to choose among)**) between switches. In **Facebook's data center** [FB 2019], each TOR is connected to **16 different tier-2 switches**, and each tier-2 switch is connected to **16 different tier-1 switches**.

### Multi-Path Routing: A Direct Consequence

A **direct consequence** of the increased connectivity between tiers in data center networks is that **multi-path routing** can become a **first-class citizen (a core, natively supported feature rather than an afterthought)** in these networks — flows are, **by default**, **multipath**. A very simple scheme to achieve multi-path routing is **Equal Cost Multi-Path (ECMP)** [RFC 2992], which performs a **randomized next-hop selection** among the switches between source and destination. **Advanced schemes** using **finer-grained load balancing** have also been proposed [Alizadeh 2014; Noormohammadpour 2018]. While these schemes perform multi-path routing at the **flow** level, there are also designs that route **individual packets** within a flow among multiple paths [He 2015; Raiciu 2010].

### From Three-Tier to Leaf-Spine

The three-layer hierarchy has **evolved** into a **two-layer** data-center interconnection **fabric (the overall interconnected structure)** known as a **leaf-spine** topology [Alizadeh 2013].

```
Fig -- Leaf-Spine Topology (2 layers)
────────────────────────────────────────
        Spine switches (core)
      /  |  \      /  |  \
     /   |   \    /   |   \
  Leaf  Leaf  Leaf  Leaf  Leaf   <- each
 switch switch switch switch switch  leaf
   |      |      |      |      |   connects
 Racks  Racks  Racks  Racks  Racks  to EVERY
                                    spine

 Every leaf <-> every spine: any two
 hosts are EXACTLY 2 switch hops
 apart -- no variable-depth hierarchy.
```

![[Pasted image 20260718000029.png]]

In the **leaf-spine** topology, the **access switch** (known as a **leaf switch**) is connected to **each and every** **core switch** (known as a **spine switch**). The interconnection capacity of the leaf-spine topology can scale by **either** upgrading link speeds between the leaf and spine nodes, **or** by adding more spine nodes **horizontally**. In a leaf-spine topology, **all communication** within the data center — beyond the access switch — requires **exactly two switch hops**, as opposed to the **variable and potentially larger** number of hops in the earlier hierarchical/three-tier topologies.

**Why has this evolution happened?** The evolution toward leaf-spine topologies reflects the **increase of communication among servers within the data center** — so-called **"east-west" traffic** — as opposed to communication between a server and something **outside** the data center (so-called **"north-south" traffic**). There is also increased interest in **reconfigurable direct optical interconnects**, under SDN control, among leaf (or aggregate) switches, thereby **bypassing** the need for spine switches altogether [Poutievski 2022].

> **Analogy — Airport Hub-and-Spoke Compared to a Fully-Meshed Regional Shuttle Network:** A traditional hierarchical data center topology is somewhat like an airline that only offers flights through a **chain of progressively larger hub airports** — a small regional airport connects to a mid-sized hub, which connects to a mega-hub, which finally connects to the destination's mid-sized hub, and so on. Every trip potentially involves **several connecting flights**, and if a particular connecting flight is overbooked (a link is saturated), your journey suffers. A **leaf-spine** topology, by contrast, is like a regional shuttle network where **every small airport (leaf) has a direct flight to every one of a handful of large central airports (spine)** — no matter which two small airports you're traveling between, your journey **always** involves passing through exactly **one** central airport: a fixed, predictable, and short two-hop journey, with **many alternative central airports** to route through if one happens to be congested.

### Clos Networks: The Underlying Theory

Multi-switch **layered (tiered, multistage)** interconnection networks such as those illustrated by the three-tier and leaf-spine topologies are known as **Clos networks**, named after **Charles Clos**, who studied such networks [Clos 1953] in the context of **telephony switching**. Since then, a rich **theory** of Clos networks has been developed, finding additional use in **data center networking** and in **multiprocessor interconnection networks**. For some of the largest data center operators, switches in the interconnection network are being built **in-house** from **commodity, off-the-shelf, merchant silicon** [Greenberg 2009b; Roy 2015; Singh 2015; Poutievski 2022], rather than being purchased from switch vendors.

---

## 6.6.2 Trends in Data Center Networking

Data center networking is evolving rapidly, with the trends being driven by **cost reduction, virtualization, physical constraints, manageability, expandability, and customization**.

```
Fig -- Data Center Networking Trends
────────────────────────────────────────
 1. Centralized SDN control & mgmt
    (logically centralized, like
     Google's Orion + B4 network)

 2. Virtualization
    (VMs decoupled from physical
     hosts; DNS-style ARP lookup
     replaces broadcast, so VMs can
     migrate across racks/switches)

 3. Physical constraints
    (high bandwidth + very low
     delay -> small buffers, fast-
     reacting congestion control)

 4. Hardware modularity (MDCs)
    (shipping-container "mini data
     centers"; graceful degradation)

 5. Energy- and carbon-efficiency
    (power is a major cost driver;
     availability zones for
     resilience)
```

### Centralized SDN Control and Management

Because a data center is managed by a **single organization**, it's perhaps natural that a number of the largest data center operators — including Google, Microsoft, and Facebook — are **embracing (adopting enthusiastically)** the notion of **SDN-like logically centralized control**. For example, Google's **Orion SDN platform** is used to control and manage its data-center networks, as well as its **wide-area B4 network** that connects data centers together. These architectures also reflect a **clear separation** of a **data plane** (comprised of relatively simple, commodity switches) and a **software-based control plane**, as first covered in Section 5.5. Due to the **immense scale** of their data centers, automated configuration and operational state management (as encountered in Section 5.7) are also **crucial**.

### Virtualization

**Virtualization** has been a **driving force** for much of the growth of cloud computing and data center networks more generally. **Virtual Machines (VMs)** **decouple (separate, disentangle)** software running applications from the underlying physical hardware. This decoupling also allows **seamless migration** of VMs between physical servers — which might be located on **different racks**.

**The problem:** standard Ethernet and IP protocols have **limitations** in enabling the movement of VMs while maintaining active network connections across servers. Since all data center networks are managed by a **single administrative authority**, an **elegant solution** to this problem is to treat the entire data center network as a **single, flat, layer-2 network**. Recall that in a typical Ethernet network, the ARP protocol maintains the binding between the IP address and hardware (MAC) address on an interface.

**How the illusion of "one switch" is achieved:** to emulate the effect of having all hosts connect to a "**single switch**," the ARP mechanism is **modified** to use a **DNS-style query system** instead of a broadcast, and a **directory** maintains a mapping of the IP address assigned to a VM and **which physical switch** the VM is currently connected to in the data-center network. Scalable schemes implementing this basic design have been proposed [Mysore 2009; Greenberg 2009b] and have been successfully deployed in modern data centers.

> **Analogy — A Company's Internal Directory Instead of Shouting Down the Hallway:** Recall that ordinary ARP works by essentially **shouting a question down every hallway** (broadcasting) until the right recipient answers. This works fine in a small building, but becomes wildly inefficient — and worse, becomes a broadcast storm risk — across a data center with hundreds of thousands of hosts spread over many physical switches, especially when employees (VMs) might **relocate offices** (migrate between physical servers) at any moment. The data-center solution is like maintaining a **centralized company directory** (a DNS-style lookup service): instead of shouting down every hallway, you simply **look up** the current desk location of the person you want in the directory — and whenever someone changes offices, the directory is simply **updated**, with no need to re-shout anything down every hallway in the building.

### Physical Constraints

Unlike the wide-area Internet, data center networks operate in environments that not only have **very high capacity** (40 Gbps and 100 Gbps links are now commonplace) but also **extremely low delays** (microseconds). **Consequently**, buffer sizes are **small**, and congestion-control protocols such as **TCP** and its variants **do not scale well** in data centers. In data centers, congestion-control protocols have to **react fast** and operate in **extremely low loss regimes**, as loss recovery and timeouts can lead to **extreme inefficiency**.

Several approaches to tackle this issue have been **proposed and deployed**, ranging from:

- **Data-center-specific TCP variants** [Alizadeh 2010]
- Implementing **Remote Direct Memory Access (RDMA)** technologies on standard Ethernet [Zhu 2015; Moshref 2016; Guo 2016]

**Scheduling theory** has also been applied to develop mechanisms that **decouple flow scheduling from rate control**, enabling very simple congestion-control protocols while maintaining **high utilization** of the links [Alizadeh 2013; Hong 2012].

### Hardware Modularity and Customization

Another **major trend** is to employ **shipping container–based modular data centers (MDCs)** [YouTube 2009; Waldrop 2007]. In an MDC, a factory builds — within a standard **12-meter shipping container** — a "**mini data center**," and **ships the container** to the data center location. Each container houses up to a **few thousand hosts**, stacked in tens of racks, which are packed **closely together**. At the data center location, **multiple containers** are interconnected with each other and also with the Internet.

**A design implication for reliability:** once a **prefabricated (built ahead of time in a factory)** container is deployed at a data center, it's often **difficult to service**. Thus, each container is designed for **graceful performance degradation**: as components (servers and switches) **fail over time**, the container continues to operate but with **degraded performance**. When many components have failed and performance has dropped **below a threshold**, the **entire container** is removed and replaced with a fresh one.

**A networking challenge this creates:** building a data center out of containers introduces **new** networking challenges. With an MDC, there are **two types of networks**: the **container-internal** network within each of the containers, and the **core network** connecting each container [Guo 2009; Farrington 2010]. Within each container, at the scale of up to a few thousand hosts, it's possible to build a **fully connected network** using inexpensive **commodity Gigabit Ethernet switches**. However, the design of the **core network** — interconnecting hundreds to thousands of containers while providing high host-to-host bandwidth **across containers** for typical workloads — remains a **challenging problem**. A **hybrid electrical/optical switch architecture** for interconnecting containers is described in [Farrington 2010].

### Energy Efficiency and Carbon Efficiency

The cost of **power** is a significant factor in operating a data center. With an increase in the **size and number** of data centers, power now represents a **non-negligible fraction** of overall energy demand. [Goldman 2024] estimates that data centers **consume approximately 400 Terawatt hours** of energy in 2024 — between **1% and 2%** of global electricity use. That demand is expected to **rise to 3–4%** of global energy use by the end of the decade, driven in part by **increasing demands** of data- and compute-intensive **AI-based applications**, including **Large Language Models**. Many data center operators are thus focused on **energy efficiency**, e.g., [Microsoft 2025].

**Beyond energy efficiency alone**, an increased emphasis on **sustainability** of human-built infrastructure has resulted in an emphasis on **carbon efficiency** — optimizing operations to **reduce their overall carbon footprint** [Shenoy 2022].

**A final important trend:** large cloud providers are increasingly building or **customizing** just about everything found in their data centers, including **network adapters, switches, routers, TORs, software, and networking protocols** [Greenberg 2015; Singh 2015]. Another trend, **pioneered by Amazon**, is to improve reliability with "**availability zones**" — which essentially **replicate** distinct data centers in different **nearby buildings**. By having the buildings nearby (a few kilometers apart), transactional data can be **synchronized** across data centers in the **same** availability zone, while still providing **fault tolerance** [Amazon 2014]. Many more innovations in data center design are likely to continue to come.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Load balancers hide internal structure via NAT**|An attacker outside the data center cannot directly enumerate or target internal hosts, since only the load balancer's public-facing IP address is externally visible — internal addressing and topology remain concealed|This NAT-like translation is itself a valuable security boundary, and defenders should ensure it isn't accidentally bypassed (e.g., via misconfigured direct-server-return setups) which would re-expose internal hosts|
|**A single flat layer-2 network (for VM mobility) enlarges the blast radius**|Treating the entire data center as one flat, layer-2 network to support seamless VM migration means that a compromised host or VM could potentially reach — or spoof traffic toward — many more hosts than in a segmented design, since layer-2 broadcast/ARP-style mechanisms span the whole facility|Data centers mitigate this with the DNS-style, directory-based ARP replacement described above (rather than true broadcast), plus additional micro-segmentation (VLANs, security groups, or SDN-enforced policies) layered on top of the flat fabric to constrain lateral movement|
|**Highly interconnected topologies (leaf-spine, three-tier) increase attack-path diversity**|The same rich interconnectivity that provides bandwidth and redundancy also means there are many more potential paths an attacker's traffic (or a compromised host's traffic) could take to reach a target, complicating simple perimeter- or path-based defenses|Centralized SDN control provides a single point from which security policy can be consistently enforced network-wide, regardless of which specific physical path a flow happens to take|
|**Modular/container-based data centers (MDCs) are physically harder to service and inspect**|Because MDCs are designed to be sealed, prefabricated units that degrade gracefully rather than being individually serviced, physical tampering or hardware-level compromise inside a container could go undetected for longer, given the "leave it running until it's replaced" design philosophy|Supply-chain security and pre-deployment hardware verification become especially important for MDCs, since post-deployment physical inspection is intentionally minimized by design|
|**Availability zones concentrate trust across "nearby" buildings**|Because availability zones synchronize transactional data across nearby buildings for fault tolerance, an attacker who compromises the synchronization channel or one zone's infrastructure could potentially affect data consistency or availability across the paired zone as well|Strong authentication and integrity protection on inter-zone synchronization links is essential, since the very mechanism that provides fault tolerance also creates a trust relationship between physically separate facilities|

---

## Questions I Still Have

- [ ] The text says data-center congestion control has to "react fast" in "extremely low loss regimes" because standard TCP doesn't scale well — what specifically breaks about TCP's own congestion-control algorithm (e.g., its RTT estimation or backoff behavior) at microsecond-scale delays that doesn't break at wide-area millisecond-scale delays?
- [ ] For the DNS-style ARP replacement used to support VM mobility, how does the directory itself get updated in real time when a VM migrates — is there some brief window where stale entries could cause a packet to be misdelivered to the VM's old physical switch?
- [ ] Given that leaf-spine topologies guarantee exactly two switch hops for any host-to-host communication, why hasn't this design fully displaced the older three-tier hierarchical topology in practice — are there specific scale thresholds or cost trade-offs where three-tier is still preferable?
- [ ] The text mentions Equal Cost Multi-Path (ECMP) performs "randomized next-hop selection," while more advanced schemes do finer-grained, flow-level or even packet-level load balancing — what specific problems (e.g., packet reordering) arise from routing individual packets of the same flow across different paths, and how do those advanced schemes address them?
- [ ] For modular data centers (MDCs) designed for "graceful performance degradation," what is the actual threshold or decision process used to determine when a container has degraded enough to warrant full replacement, and who/what makes that call?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Data center network**|The internal computer network that interconnects the hosts within a data center, and connects the data center to the public Internet|
|**Blade**|A commodity host (with CPU, memory, and disk) used in data centers, typically resembling a pizza box in shape|
|**Rack**|A physical structure holding 20–40 blades, interconnected via a Top-of-Rack switch|
|**TOR (Top of Rack) switch**|The switch at the top of each server rack that interconnects the rack's hosts with each other and with other switches in the data center|
|**Border router**|A router connecting the data center network to the public Internet, handling traffic between external clients and internal hosts|
|**Load balancer**|A device that distributes incoming client requests across hosts based on current load, and performs NAT-like translation between public and internal IP addresses; sometimes called a "layer-4 switch"|
|**Hierarchical architecture**|A data center topology in which a border router connects to access routers, which connect through tiers of switches down to TOR switches and racks|
|**East-west traffic**|Traffic flowing between servers within the data center (as opposed to north-south traffic, between servers and the outside world)|
|**North-south traffic**|Traffic flowing between a data center server and something outside the data center|
|**Three-tier, highly interconnected topology**|A data center design with multiple links/paths between access, aggregation, and core switches, enabling multi-path routing and higher aggregate capacity|
|**ECMP (Equal Cost Multi-Path)**|A simple multi-path routing scheme performing randomized next-hop selection among available switches between source and destination|
|**Leaf-spine topology**|A two-layer data center interconnection fabric where every leaf (access) switch connects to every spine (core) switch, guaranteeing exactly two switch hops for any host-to-host communication|
|**Clos network**|A class of multi-switch, layered interconnection networks (including three-tier and leaf-spine designs), named after Charles Clos, originally studied in the context of telephony switching|
|**Virtual Machine (VM)**|Software that decouples an application from the underlying physical hardware, enabling migration between physical servers|
|**RDMA (Remote Direct Memory Access)**|A technology enabling low-latency, low-overhead data transfer between hosts, used in data centers to help address TCP's poor scaling under microsecond-scale delays|
|**MDC (Modular Data Center)**|A shipping-container-based "mini data center," prefabricated and shipped as a self-contained unit designed for graceful performance degradation over time|
|**Availability zone**|A design (pioneered by Amazon) replicating distinct data centers in nearby buildings to synchronize data and provide fault tolerance|

---

## Related Concepts

---

→ Next: 6.7 Retrospective — A Day in the Life of a Web Page Request