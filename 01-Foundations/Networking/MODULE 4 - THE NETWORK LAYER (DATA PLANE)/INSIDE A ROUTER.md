---
title: INSIDE A ROUTER
date: 2026-07-02
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 4.2 What's Inside a Router?

> **One-Line Summary:** A router's forwarding function — the actual transfer of packets from incoming links to the right outgoing links — decomposes into four cooperating components: input ports that terminate links and run the lookup deciding where each packet goes, a switching fabric that physically moves packets from input to output, output ports that buffer and transmit, and a routing processor that maintains the forwarding table and handles all control-plane logic; the data-plane trio (input, fabric, output) runs in nanosecond-speed hardware, while the routing processor runs in millisecond-speed software — and the interplay among them determines where queues form, where packets get dropped, and how fairly bandwidth is shared among competing flows.

---

## Core Idea: Unpacking the Black Box

Section 4.1 treated the router as a "box" with a forwarding table inside. This section opens the box. The key engineering insight is a **hard separation between timescales**: the data plane (input ports → switching fabric → output ports) must operate at line speed — nanoseconds — so it runs in dedicated hardware. The control plane (the routing processor that computes the forwarding table) operates over much longer windows — milliseconds to seconds — and runs in software on what is essentially a general-purpose CPU embedded inside the router.

### Worked Example: Why Hardware Is Non-Negotiable

With a **100 Gbps input link** and a standard **64-byte IP datagram**, how much time does an input port have to process one datagram before the next one arrives?

```
64 bytes × 8 bits/byte = 512 bits per datagram
100 Gbps = 100 × 10⁹ bits per second

Time per datagram = 512 / (100 × 10⁹) = 5.12 nanoseconds
```

Five nanoseconds. At that rate, if the router also supports _N_ ports on a single line card, the datagram-processing pipeline must operate _N_ times faster still. No software running on a general-purpose CPU can execute a table-lookup-and-forward cycle in under 5 ns; this is why forwarding hardware is implemented in custom ASICs or merchant-silicon chips (e.g., products from Intel and Broadcom), not software.

---

## The Four Router Components

![[Pasted image 20260702234507.png]]

|Component|Plane|Speed|Key Job|
|---|---|---|---|
|**Input ports**|Data|Nanoseconds (hardware)|Terminate the incoming physical link; run link-layer functions; perform the forwarding-table lookup that determines the output port|
|**Switching fabric**|Data|Nanoseconds (hardware)|Physically move packets from an input port to the correct output port|
|**Output ports**|Data|Nanoseconds (hardware)|Buffer packets received from the switching fabric; perform link-layer and physical-layer functions to transmit them onto the outgoing link|
|**Routing processor**|Control|Milliseconds–seconds (software)|In traditional routers: execute routing protocols, maintain routing/forwarding tables. In SDN routers: communicate with the remote SDN controller to receive forwarding table entries, install them in the input ports|

> **A note on the word "port" here:** A router's input and output "ports" are the **physical hardware link interfaces** — completely distinct from the software port numbers (16-bit integers) used in Chapters 2 and 3 to identify sockets and applications. A large ISP edge router like the Juniper MX2020 can support hundreds of 10 Gbps ports with an overall system capacity of 800 Tbps — these are physical connectors on hardware cards, not TCP/UDP port numbers.

---

### The Roundabout Analogy — Identifying Bottlenecks

The roundabout analogy helps locate exactly where problems can occur. Map the router components onto it directly:

```
Entry road  ──►  Entry station (attendant)  ──►  Roundabout  ──►  Exit road
(incoming link)   (input port + lookup)       (switch fabric)   (output port)
```

|Router Component|Roundabout Equivalent|
|---|---|
|Input port (with lookup)|Entry station where an attendant consults a map and tells the driver which exit to take|
|Switch fabric|The roundabout itself — cars traverse it to reach the right exit|
|Output port|The roundabout exit road|

**Destination-based forwarding:** The attendant looks up the car's _final_ destination (not the local roundabout) and directs it to the exit leading toward that destination.

**Generalized forwarding:** The attendant makes the exit decision based on _many_ factors — destination, origin state, car model, license plate. A car deemed "not roadworthy" might be blocked from entering the roundabout at all.

**Where bottlenecks emerge:** What if cars arrive blazingly fast (100 Gbps!) but the attendant is slow? Backups form at the entry station. What if cars traverse the roundabout slowly, even with a fast attendant? Backups form at the roundabout itself. What if all cars want the same exit at once? Backups form at the exit road. Each of these maps to a different queuing location inside the real router, studied in detail in Section 4.2.4.

---

## 4.2.1 Input Port Processing and Destination-Based Forwarding

![[Pasted image 20260702234735.png]]

An input port performs three functions in a left-to-right pipeline:

|Stage|Box in Figure 4.5|What Happens|
|---|---|---|
|**Line termination**|Leftmost box|Physical-layer bit reception — terminates the incoming physical link|
|**Data link processing**|Middle box|Link-layer functions, including decapsulation of the arriving frame|
|**Lookup, forwarding, queuing**|Rightmost box|**The most critical step:** consults the forwarding table to determine which output port this packet should be sent to via the switching fabric|

```
                    INPUT PORT PROCESSING FLOW
                    ───────────────────────────

  Arriving bits on physical link
            │
            ▼
  ┌─────────────────────┐
  │   Line Termination   │  ← Physical layer: receive bits, recover signal
  └──────────┬──────────┘
             │
             ▼
  ┌─────────────────────────────┐
  │   Data Link Processing      │  ← Decapsulate frame, check link-layer header
  │   (protocol, decapsulation)  │
  └──────────┬──────────────────┘
             │
             ▼
  ┌──────────────────────────────────────────┐
  │   Forwarding Table Lookup (TCAM)          │
  │   → Longest prefix match on dest. IP      │
  │   → Determine output port                 │
  │   Also: check version, TTL, checksum;     │
  │   update counters                         │
  └──────────┬───────────────────────────────┘
             │
             ▼
     Is switching fabric free?
        /           \
      YES             NO
       │               │
       ▼               ▼
  Send packet     Queue packet at
  to switching    input port buffer
  fabric    ──►  (risk: HOL blocking
                  if head-of-line packet
                  is already waiting for
                  a congested output port)
```

The forwarding table is computed by the routing processor and **copied to each input line card via an internal bus** (e.g., a PCI bus). This is crucial: with a **shadow copy of the forwarding table at every line card**, forwarding decisions can be made _locally at each input port_, without routing every packet up to a centralized routing processor. This local lookup eliminates what would otherwise be a catastrophic per-packet processing bottleneck.

After lookup, the input port also handles: checking the packet's version number, checksum, and time-to-live field; updating relevant fields; and updating counters for network management.

### Why a Brute-Force Forwarding Table Is Impossible

The "simplest" approach to destination-based forwarding would be: give every possible 32-bit IPv4 destination address its own forwarding-table entry directly mapping to an output port. Since 32-bit addresses span a space of **2³² ≈ 4.3 billion** possible addresses:

```
4,294,967,296 entries × ~10 bytes each ≈ 43 GB just for the lookup table
```

This is "totally out of the question" — both in memory size and in lookup time.

### Prefix Matching: The Practical Solution

Instead, routers group destination addresses into **address ranges** sharing a common prefix, and match an incoming packet's destination address against these prefixes. A four-entry forwarding table can cover the entire address space:

#### Worked Example — Prefix Matching Table

Suppose a router has four output link interfaces (0 through 3) and the following prefix-based forwarding table:

|Prefix|Link Interface|
|---|---|
|`11001000 00010111 00010` (21 bits)|0|
|`11001000 00010111 00011000` (24 bits)|1|
|`11001000 00010111 00011` (21 bits)|2|
|Otherwise|3|

**(Alternatively expressed with the full range for clarity):**

|Destination Address Range|Link Interface|
|---|---|
|`11001000 00010111 00010000 00000000` through `11001000 00010111 00010111 11111111`|0|
|`11001000 00010111 00011000 00000000` through `11001000 00010111 00011000 11111111`|1|
|`11001000 00010111 00011001 00000000` through `11001000 00010111 00011111 11111111`|2|
|Otherwise|3|

**Packet A:** Destination = `11001000 00010111 00010110 10100001`

```
Prefix 0 (21 bits): 11001000 00010111 00010   ✓ matches first 21 bits
→ Forward to interface 0
```

**Packet B:** Destination = `11001000 00010111 00011000 10101010`

```
Prefix 1 (24 bits): 11001000 00010111 00011000   ✓ matches first 24 bits
Prefix 2 (21 bits): 11001000 00010111 00011      ✓ also matches first 21 bits

TWO MATCHES — which wins?
```

### The Longest Prefix Matching Rule

When a destination address matches more than one entry, the router uses the **longest prefix match** — forwarding the packet to the link interface associated with the _longest_ matching prefix in the table. In Packet B's case, the 24-bit prefix (interface 1) beats the 21-bit prefix (interface 2), so the packet goes to **interface 1**.

> **Analogy — Most Specific Postal Address Wins:** Suppose you're mailing a letter. One rule says "anything going to New York City → truck depot A" and a more specific rule says "anything going to 742 Evergreen Terrace, NYC → hand-deliver." Both rules match your letter — but the _more specific_ (longer, more restrictive) match wins, and the letter goes directly to the door. Longest prefix matching works the same way: a more specific prefix match always overrides a shorter, more general one.

### How Fast Is the Lookup? TCAMs

At Gigabit transmission rates, lookup must happen in **nanoseconds** — simply scanning the table entry by entry would be far too slow. In practice, **Ternary Content Addressable Memories (TCAMs)** are used [Yu 2004]:

|Property|Detail|
|---|---|
|**How it works**|A 32-bit IP address is presented to the TCAM; the TCAM returns the matching forwarding table entry in **one clock cycle** — regardless of how many entries the table has|
|**"Ternary"**|Each bit in the TCAM can store 0, 1, or "don't care" (X) — making it well-suited for prefix matching, since a prefix like `11001000 00010` is stored as those bits followed by all don't-cares|
|**Capacity**|The Cisco Catalyst 6500/7500 series can hold a million TCAM forwarding table entries; Intel's Aurora switch can hold upwards of 1.2 million entries|
|**Trade-off**|TCAMs are expensive and have high energy consumption; hybrid approaches combining static RAM (SRAM) and purely algorithmic lookup have also been proposed and used in practice|

### Match Plus Action: A More General View

The input port's process of (a) looking up a destination IP address and then (b) sending the packet into the switching fabric toward the specified output port is, more abstractly, a **"match plus action"** operation — match a field value, take an action. This same abstraction appears across many networked devices:

|Device|Match On|Action|
|---|---|---|
|Router (traditional)|Destination IP address|Forward to output port|
|Link-layer switch|Destination MAC address|Forward to switch output port|
|Firewall|Source/destination IP + transport-layer port|Drop or permit|
|NAT|Transport-layer port number|Rewrite port number|

The match-plus-action abstraction becomes the foundation of the generalized forwarding studied in Section 4.4.

---

## 4.2.2 Switching

The switching fabric is **at the very heart of a router** — it is literally where packets get forwarded from an input port to an output port. Three main switching approaches have been used:

![[Pasted image 20260702235024.png]]

### Switching Via Memory

The simplest, earliest routers were traditional computers: switching between input and output ports was done under direct control of the **CPU (routing processor)**.

**How it works:**

```
Packet arrives at input port
    → Input port signals CPU via interrupt
    → Routing processor copies packet from input port into processor memory
    → Routing processor extracts destination address, looks up output port
    → Routing processor copies packet to output port's buffer
```

**The bottleneck:** If the memory bandwidth allows a maximum of _B_ packets per second to be read from, or written to, memory, the total forwarding throughput must be less than **B/2** (because each packet requires _two_ memory operations — read and write). Additionally, **two packets cannot be forwarded at the same time** even if they have different destination ports, since only one memory read/write can occur on the shared system bus at a time.

> Some modern routers still switch via memory — but with a key difference: the lookup and storing of the packet into the appropriate memory location happens **on the input line cards themselves**, rather than on the central routing processor. This makes them look like shared-memory multiprocessors. Cisco's Catalyst 8500 series switches packets via a shared memory in this way.

### Switching Via Bus

An input port transfers a packet **directly to the output port over a shared bus**, without routing processor intervention. Practically, the input port prepends an internal switch-header label to the packet specifying the target output port; all output ports receive the packet on the shared bus, but only the port matching the label keeps it (and removes the label before transmission).

**The bottleneck:** Only **one packet can cross the bus at a time**. If multiple packets arrive simultaneously at different input ports, all but one must wait. The switching speed of the entire router is limited to the bus speed.

> In the roundabout analogy, switching via bus is equivalent to a roundabout that can only contain **one car at a time** — even if cars are heading to completely different exits, only one can be in the roundabout simultaneously. Despite this limitation, bus switching is often sufficient for enterprise and small local-area network routers. The Cisco Catalyst 9600 switches over a backplane bus; the Cisco Catalyst 9200 uses a ring (a direction bus with connected ends).

### Switching Via Interconnection Network (Crossbar Switch)

A **crossbar switch** consists of **2N buses** connecting _N_ input ports to _N_ output ports. Each vertical bus intersects each horizontal bus at a **crosspoint** that can be opened or closed by the switch fabric controller.

**How it works (example: packet from port A to port Y):**

```
Crosspoint at intersection of bus A and bus Y is CLOSED
Port A sends packet onto bus A
Only bus Y picks up the packet (all other horizontal buses ignore it)
```

**Why this is powerful:** A packet from port B can simultaneously be forwarded to port X — the A-to-Y and B-to-X transfers use _different_ vertical and horizontal buses, so they happen **in parallel**. A crossbar switch is **non-blocking**: a packet being forwarded to output port Y will not be blocked from reaching that output port as long as no other packet is currently being forwarded to Y.

**The remaining constraint:** If two packets from two _different_ input ports are both destined for the _same_ output port, one of them must still wait — only one packet can be sent over any given bus at a time.

**Scaling beyond a single crossbar:** More sophisticated routers use **multi-stage switching fabrics** — multiple stages of smaller switching elements that together allow packets from multiple input ports to proceed toward the same output port simultaneously through the multi-stage fabric. A router's switching capacity can also be scaled by **running multiple switching fabrics in parallel**: input ports break a packet into _K_ smaller chunks and spray the chunks through _K_ of _N_ switching fabrics, with the selected output port reassembling the original packet. Cisco's 8000 series uses a two-stage leaf-spine interconnection; the Cisco CRS uses a three-stage non-blocking strategy.

### Summary: Switching Approaches Compared

|Method|Parallel Forwarding|Speed Bottleneck|Typical Use|
|---|---|---|---|
|**Via memory**|No (one bus R/W at a time)|Memory bandwidth B/2|Early/simple routers|
|**Via bus**|No (one packet on bus at a time)|Bus speed|Enterprise/small networks|
|**Via interconnection (crossbar)**|Yes (different input/output pairs simultaneously)|Output port contention|High-performance routers|

---

## 4.2.3 Output Port Processing

![[Pasted image 20260702235227.png]]

Output port processing is the mirror image of input port processing, running in reverse order:

```
From switching fabric
    → Queuing (buffer management)
    → Data link processing (protocol, encapsulation)
    → Line termination
        → Out onto the physical outgoing link
```

The output port stores packets received from the switching fabric, **selects** (via packet scheduling, Section 4.2.5) and **de-queues** packets for transmission, and performs the necessary link-layer and physical-layer functions. When a link is bidirectional (carries traffic in both directions), an output port will typically be paired with the input port for that link on the same line card.

---

## 4.2.4 Where Does Queuing Occur?

Queues can form at **both input ports and output ports**. Which one dominates depends on the traffic load, the switch fabric speed relative to line speed, and the specific traffic pattern. When queues grow large enough to exhaust a router's available memory, **packet loss occurs** — this is the real mechanism behind packets being "dropped at a router" or "lost within the network." Everything discussed abstractly in Chapter 3 as "packet loss" happens here, at these queues.

**Setup:** Consider a router with _N_ input ports and _N_ output ports, all input and output links at transmission rate _R_line_, and the switch fabric transfer rate _R_switch_. Assume fixed-length packets arriving synchronously (one packet per input link per "batch").

### Input Queueing and Head-of-Line (HOL) Blocking

If _R_switch_ is **not** significantly faster than _N × R_line_, the switch fabric can't keep up with arriving packets, and input queues form. But even if the fabric _is_ fast enough overall, a subtle problem can still cause input-queue growth: **Head-of-Line (HOL) blocking**.

![[Pasted image 20260702235532.png]]

#### Worked Example: HOL Blocking Step by Step

Setup: 4 input ports, 4 output ports, crossbar switching fabric. Three packets arrive simultaneously at three different input ports:

```
At time t:

Input queue 1 (top):    [Packet A → Upper output port] [Packet C → Middle output port]
Input queue 2 (middle): [Packet B → Upper output port]
Input queue 3 (bottom): [Packet D → Lower output port]

Output port availability: upper, middle, lower all FREE
```

Step 1: The switch fabric tries to transfer the front packets:

```
Queue 1 front: Packet A → Upper output  ✓ chosen to proceed
Queue 2 front: Packet B → Upper output  ✗ BLOCKED — upper port already taken by A
Queue 3 front: Packet D → Lower output  ✓ proceeds simultaneously with A
```

Step 2: Packet B must wait at its input queue. Behind Packet B is Packet X (destined for the middle output port — which is completely free):

```
Queue 2: [Packet B → Upper (BLOCKED)] [Packet X → Middle (FREE, but stuck)]
```

**Packet X cannot move** even though its destination (middle output port) is entirely free and no other packet is heading there. It's trapped behind Packet B at the head of the line. This is **head-of-line (HOL) blocking** — a queued packet in an input queue must wait for transfer through the switch fabric even though its _own_ output port is uncontested, simply because a different packet ahead of it in the same input queue is blocked.

**Why this matters quantitatively:** Due to HOL blocking alone, the input queue will grow to unbounded length (informally, significant packet loss occurs) when the packet arrival rate on the input links reaches only **58 percent of their capacity** — even with a perfectly non-blocking crossbar fabric. At a router operating at 58% utilization, the input queues are already unstable. Various solutions to HOL blocking have been proposed and studied [McKeown 1997].

### Output Queueing

Suppose _R_switch_ = _N × R_line_ (the fabric is N times faster than any individual line). Now consider all N input ports simultaneously receiving a packet destined for the **same** output port. In the time it takes to transmit one packet on the outgoing link, N new packets have arrived at that output port from the fabric. Since the output port can transmit only one packet per transmission time, **N packets must queue for transmission**.

#### Worked Example: Output Queue Growth Over Time

Setup: _N_ = 3 inputs, _R_switch_ = 3 × _R_line_, all 3 input packets at time t destined for the same (upper) output port.

```
At time t:
  Three packets simultaneously routed to upper output port via fabric.
  Upper output can transmit ONE packet per unit time.

Time t:       transmit packet 1, queue = {packet 2, packet 3}
Time t+1:     transmit packet 2, AND 3 new packets arrive again at the upper port
              queue = {packet 3, new_A, new_B, new_C}

If this pattern persists, the queue grows WITHOUT BOUND until memory is exhausted
→ PACKET LOSS at the output port
```

This illustrates that **output queueing can occur even when the switching fabric is N times faster than the line speed** — all that's needed is for traffic patterns to route many flows toward the same output port simultaneously.

### What Happens When the Output Buffer Is Full?

When there isn't enough memory to buffer an incoming packet, a decision must be made. The policy options:

|Policy|What Happens|
|---|---|
|**Drop-tail**|The newly arriving packet is dropped (discarded)|
|**Drop existing**|One or more already-queued packets are removed to make room for the new arrival|
|**Mark (before full)**|A packet's header is _marked_ (using ECN bits, Section 3.7.3) before the buffer is actually full, proactively signaling congestion to the sender|

The third option — dropping or marking _before_ the buffer fills — is the philosophy behind **Active Queue Management (AQM)** algorithms. Several AQM algorithms have been proposed and studied:

|Algorithm|Basis|
|---|---|
|**RED (Random Early Detection)**|[Christiansen 2001] — drops/marks packets probabilistically as the queue grows, before it's full|
|**PIE (Proportional Integral controller Enhanced)**|[RFC 8033]|
|**CoDel**|[Nichols 2012]|

DOCSIS 3.1 (the cable network standard covered in Chapter 6) recently added a specific AQM mechanism [RFC 8033, RFC 8034] specifically to combat **bufferbloat** while preserving bulk throughput.

### How Much Buffer Is "Enough"?

The old rule of thumb [RFC 3439]:

```
B = RTT × C

where: B = buffer size needed
       RTT = average round-trip time (e.g., 250 ms)
       C = link capacity (e.g., 10 Gbps)

→ B = 250 ms × 10 Gbps = 2.5 Gbits of buffering
```

This was derived from analysis of a small number of TCP flows. More recent research [Appenzeller 2004] suggests that when a **large number of independent TCP flows (N)** all pass through the same link, much less buffering is needed:

```
B = RTT × C / √N
```

As N grows (as is typical in core backbone routers), the required buffer size decreases significantly. The intuition: independent TCP flows' saw-tooth patterns are statistically uncorrelated — they don't all ramp up and crash at exactly the same moment — so their peaks and troughs partially cancel, reducing the total buffer needed to absorb bursts.

### Bufferbloat: When Too Much Buffer Becomes Its Own Problem

It's tempting to think that _more_ buffering is always better — larger buffers absorb larger traffic bursts, reducing packet loss. But this ignores a critical second-order effect: **larger buffers also mean longer queueing delays** when those buffers fill up.

> _"Buffering is a bit like salt — just the right amount makes food better, but too much makes it inedible."_

**Bufferbloat** is the phenomenon where long, persistent queueing delays arise specifically because of excessive buffering at a bottleneck link [Kleinrock 2018].

#### Worked Example: The Gamer's Persistent Queue

Setup: A home network, home router → ISP link with a large output buffer.

```
Parameters:
  Packet transmission time at home router outgoing link: 20 ms
  RTT to game server (negligible queueing elsewhere): 200 ms
```

At time t = 0: **25 packets** arrive in a burst at the home router's output queue.

```
t = 0:    Burst of 25 packets queued. Router begins transmitting: 1 packet per 20 ms.
t = 20:   Packet 1 sent. Queue = 24.
t = 40:   Packet 2 sent. Queue = 23.
...
t = 200:  Packet 10 sent. Queue = 15.
          ALSO: First ACK arrives from game server (RTT = 200 ms).
          → TCP sender sends a new segment. It is immediately queued.
          Queue = 15 + 1 = 16. Wait, let's redo this precisely:
```

At `t = 200`, the ACK for packet 1 arrives (which was sent at t=0). TCP sends another packet → immediately queued. Now:

```
t = 200:  Queue size ≈ 5 packets (25 original - those already sent + newly queued ones)
           The ACK clocking effect means ONE new packet arrives every 20 ms (the
           transmission time), and ONE packet is sent every 20 ms.
           → Queue size stabilizes at approximately 5 packets, PERSISTENTLY.

Each new packet waits approximately 5 × 20 ms = 100 ms in the queue
before even reaching the front. Plus the 200 ms propagation RTT.
→ Effective RTT seen by the gamer: 200 ms + 100 ms = 300 ms, persistently.
```

The parent (who might even know Wireshark) is baffled: there's no other traffic on the home network, yet the gamer experiences constant 300ms+ lag. The queue is always full, the delay is constant and high — not because of network congestion elsewhere, but purely because the router's large buffer is full and the ACK-clocking effect keeps it persistently full. **This is bufferbloat.** The solution is not less bandwidth but _smaller, well-managed buffers_ combined with AQM — keeping delay down even at the cost of slightly higher loss rates.

---

## 4.2.5 Packet Scheduling

Having established that output queues form, the question becomes: **in what order should a router transmit queued packets?** This is the packet scheduling problem.

### First-In-First-Out (FIFO / FCFS)

![[Pasted image 20260702235850.png]]

The FIFO (also called First-Come-First-Served, FCFS) discipline transmits packets in **exactly the order they arrived** at the output queue. Packets arriving when the link is busy wait in the queue; packets arriving when the queue is full may be dropped (per the AQM policy in use). When a packet is completely transmitted, it is removed from the queue.

![[Pasted image 20260703000007.png]]

#### Worked Example: FIFO Queue Operation

Assume each packet takes **3 time units** to transmit.

```
t= 0: Packet 1 arrives, link idle → begins transmitting immediately
t= 2: Packet 2 arrives, link busy (packet 1 still transmitting) → joins queue
t= 3: Packet 1 departs. Packet 2 begins transmitting (was waiting).
t= 4: Packet 3 arrives, link busy → joins queue
t= 6: Packet 2 departs. Packet 3 begins transmitting.
t= 9: Packet 3 departs. Queue empty. Link goes IDLE.
t=12: Packet 4 arrives, link idle → begins transmitting immediately
t=12: Packet 5 arrives simultaneously → joins queue
t=15: Packet 4 departs. Packet 5 begins transmitting.
t=18: Packet 5 departs.

Departure order: 1, 2, 3, 4, 5 — exactly the arrival order. ✓
Note: the link was IDLE from t=9 to t=12, since no packets were waiting.
```

### Priority Queuing

![[Pasted image 20260703000426.png]]

Under priority queuing, packets are **classified into priority classes** on arrival at the output queue. Each priority class has its own queue. On choosing a packet to transmit, the scheduler selects a packet from the **highest-priority class that currently has a non-empty queue**. Within the same priority class, packets are typically served in FIFO order.

**Non-preemptive priority:** The transmission of a packet is never interrupted once it has begun, even if a higher-priority packet arrives mid-transmission.

**Example configuration:** A network operator might configure:

- **High priority:** Network management packets (SNMP datagrams, port 161), real-time VoIP
- **Low priority:** Bulk file transfer, non-real-time email

```
          PRIORITY QUEUING — "WHICH PACKET NEXT?" DECISION
          ──────────────────────────────────────────────────

  Link becomes free (or router starts up)
                │
                ▼
      ┌─────────────────────────┐
      │  Class 1 queue          │
      │  non-empty?             │
      └────┬──────────┬─────────┘
          YES         NO
           │           │
           ▼           ▼
     Transmit      ┌─────────────────────────┐
     Class 1  ◄─   │  Class 2 queue          │
     packet        │  non-empty?             │
                   └────┬──────────┬─────────┘
                       YES         NO
                        │           │
                        ▼           ▼
                  Transmit      ┌─────────────────────────┐
                  Class 2  ◄─   │  Class 3 queue          │
                  packet        │  non-empty?             │
                                └────┬──────────┬─────────┘
                                    YES         NO
                                     │           │
                                     ▼           ▼
                               Transmit       Link goes IDLE
                               Class 3        (work-conserving:
                               packet         immediately re-check
                                              when next packet arrives)

  KEY RULE: A higher-priority class can jump the queue,
  but NEVER interrupts a packet already being transmitted
  (non-preemptive). The decision above only runs when
  the link finishes a transmission and becomes free.
```

![[Pasted image 20260703000636.png]]

#### Worked Example: Priority Queue in Operation

Packets 1, 3, 4 → **high priority**. Packets 2, 5 → **low priority**. Each takes 3 time units.

```
t= 0: Packet 1 (HIGH) arrives, link idle → begins transmitting.
t= 2: Packet 2 (LOW) arrives → queued in low-priority queue.
t= 3: Packet 3 (HIGH) arrives → queued in high-priority queue.
       Packet 1 departs.
       Scheduler checks: high-priority queue has packet 3. → Packet 3 starts.

t= 6: Packet 3 departs.
       Scheduler checks: high-priority queue empty; low-priority queue has packet 2.
       → Packet 2 starts.
t= 8: Packet 4 (HIGH) arrives during packet 2's transmission → high-priority queue.
       Non-preemptive: packet 2's transmission is NOT interrupted.
t= 9: Packet 2 departs.
       Scheduler checks: high-priority queue has packet 4. → Packet 4 starts.
t=12: Packet 4 departs.
t=14: Packet 5 (LOW) arrives, link idle → begins transmitting immediately.
t=17: Packet 5 departs.

Departure order: 1, 3, 2, 4, 5
Note: Packet 3 (HIGH, arrived late) departs BEFORE packet 2 (LOW, arrived earlier).
      Packet 4 (HIGH) jumps the queue ahead of any future low-priority packets.
```

### Round Robin and Weighted Fair Queuing (WFQ)

Under **round robin** queuing, packets are classified into classes (as in priority queuing), but rather than strict priority, the scheduler **alternates service among classes** in a circular sequence. A class 1 packet is sent, then a class 2 packet, then a class 1 packet, and so on.

**Work-conserving:** A round-robin scheduler will never leave the link idle when any packets of _any_ class are queued. If the scheduler looks for a packet from a given class and finds none, it **immediately advances** to the next class in the round-robin sequence — it doesn't wait.

![[Pasted image 20260703000839.png]]

#### Worked Example: Two-Class Round Robin in Operation

Packets 1, 2, 4 → **class 1**. Packets 3, 5 → **class 2**. Each takes 3 time units.

```
t= 0: Packet 1 (class 1) arrives, link idle → begins transmitting.
t= 2: Packet 2 (class 1) arrives → class 1 queue.
t= 4: Packet 3 (class 2) arrives → class 2 queue.
       Packet 1 departs (t=3). Scheduler: look for class 2 packet → finds packet 3.

Wait, let me redo with transmission time = 3:

t= 0: Packet 1 (C1) arrives → begins transmitting.
t= 2: Packet 2 (C1) arrives.
t= 3: Packet 1 departs. 
       RR: next is class 2. Class 2 queue empty → skip (work-conserving).
       Class 1 has packet 2 → Packet 2 starts.
t= 4: Packet 3 (C2) arrives → class 2 queue.
t= 6: Packet 2 departs.
       RR: next is class 2. Class 2 has packet 3 → Packet 3 starts.
t= 9: Packet 3 departs.
       RR: next is class 1. Packet 4 → begins transmitting.
t=10: Packet 5 (C2) arrives → class 2 queue.
t=12: Packet 4 departs.
       RR: next is class 2. Packet 5 → begins transmitting.
t=15: Packet 5 departs.

Departure order: 1, 2, 3, 4, 5
```

### Weighted Fair Queuing (WFQ)

WFQ is a **generalized round robin** where each class _i_ is assigned a weight _wᵢ_. In any interval of time where class _i_ has packets to send, that class is **guaranteed** to receive a fraction of service equal to:

```
wᵢ / (Σwⱼ)    (where the sum is over all classes with packets to send)
```

For a link of transmission rate _R_, class _i_ always achieves a throughput of **at least:**

```
R × wᵢ / (Σwⱼ)
```

#### Worked Example: WFQ Bandwidth Guarantee

Link rate R = 12 Mbps. Three classes with weights w₁ = 4, w₂ = 2, w₃ = 1. All three queues have packets waiting.

```
Σwⱼ = 4 + 2 + 1 = 7

Guaranteed minimum bandwidth:
  Class 1: 12 × 4/7 ≈ 6.86 Mbps
  Class 2: 12 × 2/7 ≈ 3.43 Mbps
  Class 3: 12 × 1/7 ≈ 1.71 Mbps

Total:                  12.00 Mbps ✓
```

Even in the worst case — all three classes always have packets queued simultaneously — class 3 is still guaranteed its 1.71 Mbps share, regardless of how much traffic classes 1 and 2 are generating. This is the property that makes WFQ valuable for providing **differentiated quality-of-service guarantees** to different traffic classes.

WFQ is work-conserving: if one class's queue is empty, its share is redistributed among the remaining classes rather than leaving the link idle.

![[Pasted image 20260703000951.png]]

---

## Principles in Practice: Net Neutrality

Packet scheduling mechanisms give ISPs powerful tools. A router can classify packets based on _any_ field in the IP datagram header — including source IP address, destination IP address, source/destination transport-layer port numbers, or protocol type — and apply priority, throttling, or blocking based on that classification.

**What this means concretely:** An ISP could:

- Assign higher priority to its own voice-over-IP service (port-based classification) over a competitor's (e.g., Vonage)
- Give preferential treatment to datagrams from companies that have paid for priority delivery
- Block or throttle traffic associated with specific services or content providers
- Use TCP RST packets to interfere with peer-to-peer traffic flows

Several ISPs were observed doing exactly these things before regulatory intervention:

- In 2005, a North Carolina ISP agreed to stop blocking customers from using Vonage, a competing VoIP service
- In 2007, Comcast was found to be interfering with BitTorrent P2P traffic by generating and sending TCP RST packets to BitTorrent senders and receivers

**Net neutrality** emerged as the policy response. The March 2015 FCC _Order on Protecting and Promoting an Open Internet_ established three "clear, bright line" rules:

|Rule|What It Prohibits|
|---|---|
|**No Blocking**|ISPs providing broadband Internet access service shall not block lawful content, applications, services, or non-harmful devices|
|**No Throttling**|ISPs shall not impair or degrade lawful Internet traffic on the basis of content, application, service, or use of a non-harmful device|
|**No Paid Prioritization**|ISPs shall not engage in paid prioritization — using traffic shaping, prioritization, resource reservation, or preferential traffic management in exchange for payment|

The policy landscape has subsequently oscillated: the 2015 Order was superseded by the 2017 FCC _Restoring Internet Freedom Order_ which rolled back these prohibitions (focusing instead on ISP transparency requirements); in 2024, the FCC reinstated much of the 2015 Order and re-classified broadband Internet access service (BIAS) as a "telecommunications service" rather than an "information service" — a reclassification with significant regulatory implications. Net neutrality law in the US (and elsewhere) remains actively contested, and "we're nowhere near having seen the final chapter written on net neutrality."

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Forwarding table determines where every packet goes**|Compromising the routing processor or the SDN controller that populates the forwarding table lets an attacker silently redirect traffic to any destination — a "man-in-the-middle" at the infrastructure level, invisible to TCP/TLS|The routing processor and SDN controller are extremely high-value attack targets; their communications with neighbors/controllers must be authenticated (covered in Chapter 8)|
|**TCAMs return a match in one clock cycle — but what they match against is exactly what a spoofed source address exploits**|An attacker spoofing a source address changes nothing about how the router forwards _toward_ the destination — the forwarding decision is based on destination, not source. But the TCAM-based "match plus action" in generalized forwarding (Section 4.4) can also match on source IP — meaning a mis-configured or compromised filter rule could be bypassed if source addresses can be spoofed|Source-address verification (BCP38 / RFC 2827) at ingress routers can filter packets with spoofed source addresses before they enter the network|
|**Output queue AQM drops packets proactively before the buffer fills — but an attacker generating high-volume traffic toward one destination can deliberately fill output queues, causing collateral packet loss for everyone else sharing that output port**|Flooding a router's output queue (denial-of-service) forces AQM to drop packets indiscriminately — legitimate traffic shares the pain|Per-source rate limiting and traffic classification (priority queuing giving network management traffic highest priority) are partial defenses; fundamentally, DoS mitigation requires upstream filtering|
|**Packet scheduling lets ISPs discriminate by content or source**|A well-resourced party could pay an ISP for paid prioritization to effectively slow-path competitors' traffic|Net-neutrality regulation is specifically the policy mechanism intended to prevent this — but as noted above, the regulatory situation remains fluid|

---

## Questions I Still Have

- [ ] TCAMs operate in one clock cycle for any table size — but if a router has both an IPv4 TCAM lookup _and_ generalized-forwarding ACL matching to perform on the same packet, are these done sequentially or in parallel pipelines?
- [ ] The shadow-copy of the forwarding table is sent to each line card over an internal PCI bus — when the routing processor updates the table (because a route changed), is there a brief window where some line cards have the old table and others have the new one? Could that cause forwarding inconsistency mid-update?
- [ ] WFQ guarantees minimum bandwidth fractions but "has not considered the packetization issue" — what exactly goes wrong with WFQ when packets have variable sizes, and what real implementations do to compensate?
- [ ] The bufferbloat example shows that ACK clocking keeps the queue persistently full even with no competing traffic — does using a smaller buffer actually fix this, or does it just trade one persistent queue for a different persistent packet-loss rate?
- [ ] For HOL blocking: the 58% saturation threshold — does this apply only to FCFS input scheduling, or does it also affect more sophisticated input scheduling schemes that allow packets _behind_ a blocked head-of-line packet to be served?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Input port**|The router component that terminates an incoming physical link, performs link-layer processing, and looks up the forwarding table to determine the output port|
|**Switching fabric**|The internal router component that physically moves packets from an input port to the correct output port|
|**Output port**|The router component that buffers packets from the switching fabric, selects packets for transmission (scheduling), and performs link/physical-layer functions to send them|
|**Routing processor**|The router component implementing control-plane functions: running routing protocols, maintaining routing tables, computing the forwarding table (traditional) or communicating with the SDN controller (SDN)|
|**Shadow copy**|A copy of the forwarding table stored at each input line card, enabling fully local per-packet forwarding decisions without involving the routing processor|
|**Longest prefix matching**|The rule that when a destination address matches multiple forwarding-table prefixes, the router uses the entry with the longest (most specific) matching prefix|
|**TCAM (Ternary Content Addressable Memory)**|Associative memory hardware that returns the matching forwarding table entry for a 32-bit IP address in one clock cycle; supports "don't care" bits for prefix matching|
|**Match plus action**|The general abstraction of looking up a packet header field value and taking an associated action; underlies forwarding, firewalls, NATs, switches, and generalized forwarding|
|**Switching via memory**|Early switching approach using the CPU to copy packets between input and output ports via system memory; throughput limited to B/2|
|**Switching via bus**|A shared bus connecting all input and output ports; only one packet can be on the bus at a time, limiting overall throughput to bus speed|
|**Crossbar switch**|An interconnection network of 2N buses connecting N inputs to N outputs; non-blocking — packets with different output ports can be forwarded in parallel|
|**HOL (Head-of-Line) Blocking**|The phenomenon where a queued packet in an input queue is blocked from reaching its (free) output port because the packet at the head of the same input queue is waiting for a _different_ (congested) output port|
|**Drop-tail**|A buffer management policy that drops the newly arriving packet when the buffer is full|
|**AQM (Active Queue Management)**|Buffer management policies that drop or mark packets _before_ the buffer is completely full, to provide earlier congestion signals|
|**RED (Random Early Detection)**|An AQM algorithm that drops/marks packets probabilistically as queue length grows, before the buffer fills|
|**Bufferbloat**|Persistent high latency caused by excessive buffering at a bottleneck link, where ACK clocking from TCP fills the buffer persistently even under moderate load|
|**FIFO / FCFS**|Packet scheduling discipline that transmits packets in the exact order they arrived at the output queue|
|**Priority queuing**|Packet scheduling that classifies packets into priority classes; always serves the highest-priority non-empty class; non-preemptive|
|**Round robin**|Work-conserving packet scheduling that alternates service among classes in circular order, skipping empty classes|
|**Weighted Fair Queuing (WFQ)**|Generalized round-robin scheduling that assigns weights to classes, guaranteeing each class _i_ a minimum bandwidth fraction of wᵢ/(Σwⱼ) of the link rate|
|**Work-conserving**|A scheduling discipline property: the link is never left idle if any queued packet (of any class) is waiting for transmission|
|**Net neutrality**|The principle that ISPs should treat all Internet traffic equally, without blocking, throttling, or paid prioritization of specific content or sources|

---

## Related Concepts

---

→ Next: [[IPV4, NAT & IPV6]]