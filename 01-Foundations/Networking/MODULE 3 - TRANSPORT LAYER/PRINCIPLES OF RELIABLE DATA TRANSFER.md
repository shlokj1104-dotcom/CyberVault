---
title: PRINCIPLES OF RELIABLE DATA TRANSFER
date: 2026-06-21
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 3.4 Principles of Reliable Data Transfer

> **One-Line Summary:** Reliable data transfer is the general problem of making an imperfect, error-prone, packet-dropping channel _look_ perfect to the layers above it — and the entire solution boils down to combining just five tools (checksums, acknowledgments, sequence numbers, timers, and windows) in increasingly clever ways, a progression that begins with the trivial `rdt1.0` and ends at `rdt3.0`, then scales up via pipelining into Go-Back-N and Selective Repeat — the exact toolkit TCP itself uses.

---

## Core Idea: Making an Unreliable Channel Look Reliable

This is one of the most important ideas in all of networking — important enough that Kurose & Ross call it a candidate for the "top-ten" list of foundational networking problems. It isn't even specific to the transport layer: the _exact same problem_ shows up at the link layer (getting bits across one physical wire correctly) and at the application layer (e.g., a file-sync app making sure every byte arrives). Once you understand it here, you understand a recurring pattern that appears everywhere in networking.

### The Service Abstraction (Figure 3.8)

![[Pasted image 20260622205901.png]] _(Figure 3.8 — Left: the "provided service" view, where sending and receiving processes simply hand data into / pull data out of a "reliable channel," as if it were a pipe with no leaks. Right: the actual "service implementation" — the reliable channel is faked by a reliable data transfer protocol sitting on top of a genuinely unreliable channel underneath.)_

The upper layers are promised a **reliable channel**: a pipe where every bit arrives uncorrupted, nothing is lost, and everything shows up in the order it was sent. This is _exactly_ the service model TCP advertises to applications.

**Analogy:** Think of a reliable channel like a courier service with a money-back delivery guarantee. As a customer (the upper-layer application), you don't care that the actual delivery van might get a flat tire, take a wrong turn, or that the package might get rained on along the way — you just want your package to show up, intact, in the order you mailed multiple packages. The "reliable data transfer protocol" is the courier's entire internal operation (insurance, tracking numbers, redelivery attempts) that makes that guarantee true despite a messy, error-prone world underneath.

The job of building this illusion is hard precisely _because_ the layer underneath might be unreliable. TCP is the most famous example: it's a reliable data transfer protocol built entirely on top of IP, which itself makes **zero guarantees** (no ordering, no delivery, no corruption checking).

### The Interface — Naming Conventions

To talk about protocols generically (since this same logic applies at any layer), the textbook uses the word **packet** instead of "segment," and defines four interface events:

|Call|Direction|Meaning|
|---|---|---|
|`rdt_send(data)`|Upper layer → sender|"Here's data, please deliver it reliably."|
|`deliver_data(data)`|Receiver → upper layer|"Here's data that arrived correctly."|
|`udt_send(packet)`|Either side → unreliable channel|"Send this packet, no guarantees." (`udt` = _unreliable data transfer_)|
|`rdt_rcv(packet)`|Unreliable channel → either side|"A packet just arrived from below."|

```
       Application layer
            │  rdt_send()          ▲ deliver_data()
            ▼                      │
       ┌─────────────────────────────────┐
       │   Reliable Data Transfer (rdt)   │   ← the protocol we are designing
       └─────────────────────────────────┘
            │  udt_send()          ▲ rdt_rcv()
            ▼                      │
       ┌─────────────────────────────────┐
       │     Unreliable channel           │   ← can corrupt and/or lose packets
       └─────────────────────────────────┘
```

### Two Simplifying Assumptions Used Throughout This Section

1. **Unidirectional data flow** is studied (sender → receiver only) — purely to keep the FSMs from exploding in complexity. In reality, both sides exchange packets in both directions, since the receiver must send control packets (ACKs, NAKs) back. Full bidirectional ("full-duplex") reliable transfer is conceptually identical but tedious to draw.
2. **No reordering** — the channel may corrupt or lose packets, but anything that _does_ arrive, arrives in the order it was sent. (This assumption is revisited and partially relaxed at the very end of the section.)

---

## The Big Picture Before the Details

Here's the roadmap — keep this table in mind as an anchor while reading the step-by-step build-up below. Every version of the protocol exists _only_ because the previous version had exactly one specific flaw, and each version fixes that one flaw by adding exactly one new tool:

|Protocol|Channel assumed|The one flaw being fixed|New tool added|
|---|---|---|---|
|**rdt1.0**|Perfect — no errors, no loss|(baseline; nothing to fix)|—|
|**rdt2.0**|Can corrupt bits|Receiver has no way to know data arrived intact|Checksum + ACK/NAK feedback + retransmission|
|**rdt2.1**|Can corrupt bits (incl. ACK/NAK packets)|A corrupted ACK/NAK leaves the sender unable to tell if its last packet got through|Sequence numbers (1 bit is enough)|
|**rdt2.2**|Can corrupt bits|NAKs are redundant — same info can be inferred from ACKs|NAK-free design: a duplicate ACK _means_ "NAK"|
|**rdt3.0**|Can corrupt **and lose** packets|Silence (no ACK or NAK at all) needs a way to be detected|Countdown timer → timeout-triggered retransmission|
|**Go-Back-N**|Same as rdt3.0, but pipelined|Stop-and-wait wastes almost all the link's capacity|Sliding window + cumulative ACK|
|**Selective Repeat**|Same as GBN|GBN over-retransmits after a single error|Individual ACKs + individual timers + receiver buffering|

This is the entire intellectual arc of the section. Now let's walk through _why_ each fix was necessary, not just _what_ it does.

---

## 3.4.1 Building a Reliable Data Transfer Protocol, One Flaw at a Time

### `rdt1.0` — Reliable Transfer Over a Perfectly Reliable Channel

This is the warm-up case: assume the channel underneath **never** corrupts or drops anything. If that's true, the sender can be almost embarrassingly lazy.

![[Pasted image 20260622210221.png]]

Both sender and receiver are **single-state FSMs** (finite-state machines) — there's only one state because there's nothing to track. On the sending side, the moment `rdt_send(data)` fires, the protocol wraps the data into a packet via `make_pkt()` and ships it via `udt_send()`. On the receiving side, `rdt_rcv(packet)` fires, the protocol unwraps the data via `extract()`, and hands it up via `deliver_data()`.

Notice what's _absent_: no feedback from receiver to sender at all. Since the channel is perfect, the sender never needs to know "did that arrive ok?" — the answer is always yes. There's also no mechanism for the receiver to ask the sender to slow down; it's assumed the receiver can always keep up.

**Analogy:** This is like handing a letter directly to someone standing right in front of you. You don't need a tracking number or a return receipt — the "channel" (the open air between your hand and theirs) cannot possibly lose or garble it.

---

### `rdt2.0` — Reliable Transfer Over a Channel That Can Corrupt Bits

Now make the channel slightly more realistic: bits can flip due to noise on a physical link, electrical interference, or corruption while a packet sits buffered in a router's memory. Packets still arrive in order, and none are _lost_ — just possibly _damaged_.

**Human analogy first:** Imagine dictating a long message over a noisy phone line, one sentence at a time. After each sentence, the listener says **"OK"** if they heard it clearly, or **"Please repeat that"** if it was garbled. This gives the dictation protocol two kinds of feedback:

- **Positive acknowledgment (ACK)** — "got it, send the next one."
- **Negative acknowledgment (NAK)** — "didn't get it, say that again."

Any reliable transfer protocol built around this retransmit-on-negative-feedback idea is called an **ARQ (Automatic Repeat reQuest) protocol**. ARQ protocols need exactly three new capabilities beyond `rdt1.0`:

1. **Error detection** — some way for the receiver to tell a packet got corrupted. (This is _literally_ what UDP's checksum does — see [[3.3 Connectionless Transport UDP]]. Here, those checksum bits get bundled into the packet's checksum field.)
2. **Receiver feedback** — since sender and receiver may be thousands of miles apart, the _only_ way the sender learns anything about what the receiver saw is if the receiver explicitly tells it. ACK/NAK packets are that feedback channel. (In principle each only needs to be a single bit: 0 = NAK, 1 = ACK.)
3. **Retransmission** — if a packet is reported as damaged, the sender resends it.

```
SENDER (2 states)                                   RECEIVER (1 state)
┌─────────────────────┐                          ┌────────────────────────┐
│  Wait for call        │                          │  Wait for call          │
│  from above            │                          │  from below              │
└──────────┬────────────┘                          └──────────┬─────────────┘
           │ rdt_send(data):                                   │ rdt_rcv(rcvpkt) && corrupt(rcvpkt):
           │  sndpkt = make_pkt(data, checksum)                 │  sndpkt = make_pkt(NAK)
           │  udt_send(sndpkt)                                  │  udt_send(sndpkt)
           ▼                                                    │
┌─────────────────────┐                                        │ rdt_rcv(rcvpkt) && notcorrupt(rcvpkt):
│  Wait for ACK          │── rdt_rcv && isNAK ──► resend sndpkt │  extract(rcvpkt, data); deliver_data(data)
│  or NAK                 │   (self-loop)                       │  sndpkt = make_pkt(ACK)
└──────────┬────────────┘                                        │  udt_send(sndpkt)
           │ rdt_rcv(rcvpkt) && isACK(rcvpkt)                    │  (self-loop back to same state)
           ▼
   back to "Wait for call from above"
```

This is called a **stop-and-wait** protocol — and _that name matters a lot later_. While the sender is in the "Wait for ACK or NAK" state, it **cannot accept new data from the application at all** — `rdt_send()` simply doesn't fire again until the sender has confirmation about the current packet. The sender refuses to get ahead of itself.

#### The Fatal Flaw of `rdt2.0`

Here's the part that's easy to gloss over but is actually the crux of the whole next several pages: **what happens if the ACK or NAK packet itself gets corrupted on the way back?**

Stop and think about why this is genuinely hard, not just a minor edge case: if the sender receives a garbled control packet, it has **no way of knowing** whether it was originally an ACK or a NAK. The sender is stuck. It doesn't know whether the receiver got the data correctly or not.

Three possible fixes were considered, and it's worth understanding _why two of them fail_ before landing on the one that actually works (sequence numbers):

1. **"What did you say?"** — Ask the receiver to repeat the ACK/NAK. This just introduces a _new_ kind of packet that can _also_ get corrupted, and now the receiver doesn't know if the (possibly corrupted) "what did you say?" message is a repeat-request or new data. You've just pushed the same unsolved problem one level deeper — not actually solved it.
2. **Add error-correcting bits (not just error-detecting)** to the ACK/NAK, strong enough to recover from corruption directly. This actually _can_ work for a channel that corrupts but never loses packets — but it doesn't help once packets can also be **lost** entirely (coming in `rdt3.0`), since there's nothing to "correct" if nothing arrives at all.
3. **Just resend the data packet whenever a garbled ACK/NAK shows up.** This is the one that's adopted — but it introduces a brand-new problem: **duplicate packets**. If the receiver's own ACK got corrupted in transit, the receiver has no way of knowing whether an _incoming_ packet is fresh data or a retransmission of something it already received and acknowledged.

The actual solution: **give every data packet a sequence number.** The receiver just has to check this number to tell "is this the same packet again, or a new one?" For a simple stop-and-wait protocol, a **single bit** of sequence number (0 or 1, alternating) is all that's needed — because at any given moment there's only ever one outstanding unacknowledged packet, so "same as last time" vs. "the other value" is unambiguous.

---

### `rdt2.1` — Fixing Corrupted ACKs/NAKs With Sequence Numbers

`rdt2.1` is `rdt2.0` plus 1-bit sequence numbers on data packets. Because the sender now alternates between expecting to send sequence number 0 and sequence number 1, **both the sender and receiver FSMs double in size** — to four states and two states respectively — purely to track "which sequence number am I currently dealing with?"

```
                    SENDER (rdt2.1) — 4 states, alternating 0 ⇄ 1
   ┌───────────────────┐  rdt_send(data): make pkt0, send  ┌────────────────────┐
   │ Wait for call 0      │ ─────────────────────────────────▶│ Wait for ACK/NAK 0   │
   │  from above           │◄─────────────────────────────────│                        │
   └───────────┬────────┘  good ACK0 received                 └──────────┬──────────┘
               │                                                          │ corrupt or NAK
               │                                                          │ → resend pkt0 (self-loop)
   ┌───────────▼────────┐                                     ┌──────────▼──────────┐
   │ Wait for call 1      │ ─────────────────────────────────▶│ Wait for ACK/NAK 1   │
   │  from above           │◄─────────────────────────────────│                        │
   └────────────────────┘  good ACK1 received                 └────────────────────┘
                                                                    │ corrupt or NAK
                                                                    └──► resend pkt1 (self-loop)

                    RECEIVER (rdt2.1) — 2 states
   ┌────────────────────┐  got good pkt0 → deliver, send ACK   ┌────────────────────┐
   │ Wait for 0            │ ─────────────────────────────────▶│ Wait for 1            │
   │  from below            │◄─────────────────────────────────│  from below            │
   └────────────────────┘  got good pkt1 → deliver, send ACK   └────────────────────┘
   (got pkt1 again / corrupt → resend last ACK, self-loop)    (got pkt0 again / corrupt → resend last ACK, self-loop)
```

Note that `rdt2.1` still uses **both ACKs and NAKs** — and since the channel can't lose packets yet (just corrupt them), ACK/NAK packets themselves don't even need their own sequence number: the sender always knows any ACK/NAK it receives is responding to whichever packet it _most recently_ sent.

---

### `rdt2.2` — A NAK-Free Protocol

This step is a small but elegant simplification: **NAKs are redundant**, because the same information can be communicated using only ACKs.

The trick: instead of the receiver sending an explicit NAK when a packet is corrupted, it simply **resends an ACK for the last correctly received packet**. If the sender then receives **two ACKs for the same sequence number in a row** (a **duplicate ACK**), that _itself_ is the signal: "the packet that was supposed to follow the one I just double-acknowledged did _not_ arrive correctly." A duplicate ACK functions exactly like a NAK, without needing a separate NAK packet type at all.

The only structural change from `rdt2.1`: ACK packets must now explicitly **carry the sequence number being acknowledged** (e.g. `ACK,0` or `ACK,1`), since the sender needs to check _which_ packet a duplicate ACK is duplicating.

**Analogy:** Imagine a teacher taking roll call by saying student names aloud, and each student responds "present" when their name is called. If the teacher calls "Sam" and hears "present" twice in a row (because the next name was inaudible), the teacher knows something after Sam went wrong — without anyone needing to shout "I missed that!"

---

### `rdt3.0` — Reliable Transfer Over a Lossy Channel

So far, the channel has only ever _corrupted_ packets, never _lost_ them outright. Real channels (including the Internet) do both. This is the genuinely new problem `rdt3.0` solves: **what if a data packet, or its ACK, simply vanishes?**

Here's the subtlety that makes loss harder than corruption: with corruption, _something_ always arrives at the sender (a garbled ACK/NAK) that at least signals "a problem occurred." With **loss**, nothing arrives _at all_. Pure silence. The sender has no event to react to — and from the sender's point of view, a lost data packet, a lost ACK, and a perfectly fine packet that's simply taking a long time to arrive are all **completely indistinguishable**.

#### The Fix: A Countdown Timer

The solution is to stop waiting for an _event_ and instead set a **deadline**. The sender:

1. **Starts a countdown timer** the moment it sends a packet (whether a fresh packet or a retransmission).
2. If the timer **expires before an ACK arrives**, the sender assumes loss occurred and **retransmits**.
3. **Stops the timer** once a valid ACK finally does arrive.

How long should the timeout be? Ideally, at least one full round-trip time (RTT) plus packet-processing time — but in practice this worst-case delay is hard to know with certainty. So in real systems, the sender picks a timeout value chosen to make loss _likely_ (not guaranteed) to have occurred, accepting that this introduces a new wrinkle: **a packet might just be unusually slow, not actually lost**, in which case the timer fires "prematurely" and a perfectly fine packet gets needlessly retransmitted. This is fine — `rdt2.2`'s sequence numbers already handle duplicate detection, so a spurious retransmission causes no correctness problem, just a little wasted bandwidth.

> **Analogy:** This is like sending someone a text message and deciding "if I don't get a reply within 10 minutes, I'll assume it didn't go through and resend it" — even though it's entirely possible they're just slow to reply, not that the message actually failed.

```
                    SENDER (rdt3.0) — same 4-state ring as rdt2.1, plus a timer
   ┌───────────────────┐  rdt_send(data): make pkt0,        ┌────────────────────┐
   │ Wait for call 0      │  send, START TIMER                │ Wait for ACK 0       │
   │  from above           │ ─────────────────────────────────▶│                        │
   └───────────┬────────┘                                     └──────────┬──────────┘
               ▲ good ACK0 received → STOP TIMER                          │ corrupt/NAK'd ACK → resend (self-loop)
               │                                                          │ TIMEOUT → resend pkt0, RESTART TIMER
               │                                                          │
   (mirror image for sequence number 1: Wait for call 1 → Wait for ACK 1, same timer logic)
```

Because the packet's sequence number flips back and forth between 0 and 1 with every successful round, `rdt3.0` is also nicknamed the **alternating-bit protocol**.

#### `rdt3.0` Under Four Different Scenarios

|Scenario|What happens|
|---|---|
|**(a) No loss**|pkt0 sent → ACK0 received → pkt1 sent → ACK1 received → ... clean alternation, no timer ever fires.|
|**(b) Lost data packet**|pkt1 is sent and lost in the channel. No ACK1 ever comes back. Sender's timer for pkt1 **expires** → resends pkt1 → this time it arrives → ACK1 comes back → sender proceeds normally.|
|**(c) Lost ACK**|pkt1 arrives fine and the receiver sends ACK1 — but ACK1 itself is lost. The sender's timer expires (it has no way to know the _data_ arrived, only that no ACK came back) → resends pkt1 → receiver gets pkt1 **again**, recognizes it (via the sequence number) as a duplicate, discards it, but still re-sends ACK1 → sender finally gets ACK1 and moves on.|
|**(d) Premature timeout**|pkt1 is sent, and _before_ ACK1 manages to arrive back, the sender's timer fires too early → sender resends pkt1 unnecessarily → receiver now sees pkt1 twice, detects the duplicate via sequence number, discards the second copy but re-ACKs it → sender eventually receives both ACK1s, the second of which it simply ignores since it's already moved on.|

The key correctness guarantee across all four cases: **sequence numbers absorb every kind of duplicate that retransmission can create**, and **the timer absorbs every kind of silence that loss can create.** Together, that's enough for a fully correct (if slow) reliable data transfer protocol.

---

## Why Stop-and-Wait Protocols Are Painfully Slow

`rdt3.0` is _correct_ — but almost nobody would be happy with how it _performs_, especially on today's fast, high-bandwidth links. The root cause is baked into its name: **stop-and-wait** means the sender transmits exactly one packet, then does nothing — literally nothing — until that packet is acknowledged.

### Worked Example: Coast-to-Coast Stop-and-Wait

Consider two hosts on opposite U.S. coasts, connected by a 1 Gbps (10⁹ bits/sec) link, with a round-trip propagation delay (RTT) of 30 milliseconds, sending 1,000-byte (8,000-bit) packets.

**Step 1 — time to push one packet onto the wire:**

$$d_{trans} = \frac{L}{R} = \frac{8000 \text{ bits}}{10^9 \text{ bits/sec}} = 8 \text{ microseconds}$$

That's how long it takes just to _transmit_ the packet's bits onto the link — almost instantaneous.

**Step 2 — total round trip before the next packet can go out:**

- Sender starts transmitting at _t_ = 0; finishes transmitting at _t_ = 8 microseconds.
- The packet's last bit physically arrives at the receiver after a 15 ms one-way propagation delay, at _t_ = RTT/2 + L/R ≈ 15.008 ms.
- Assuming the ACK is tiny and sent instantly, it arrives back at the sender at _t_ = RTT + L/R = 30.008 ms.
- **Only then** can the sender push out packet number two.

**Step 3 — compute utilization** (the fraction of time the sender is actually sending bits, rather than sitting idle waiting):

$$U_{sender} = \frac{L/R}{RTT + L/R} = \frac{0.008}{30.008} \approx 0.00027$$

In other words, the sender is **busy roughly 0.027% of the time.** Despite renting a full 1 Gbps link, the _effective_ throughput works out to roughly **267 kbps** — about 3,700 times slower than what the link is actually capable of. The bottleneck has nothing to do with the hardware; it's a direct consequence of the _protocol's design_, which insists on total silence after every single packet.

**Intuition:** Picture a single-lane toll bridge where, by rule, only one car may be on the bridge at any time — the next car must wait until the previous one has completely crossed _and driven all the way back to confirm it arrived safely_ before it's allowed to start. Even if the bridge itself could comfortably hold a hundred cars at once, the rule throttles total traffic down to a trickle.

---

## 3.4.2 Pipelining

The fix is conceptually simple: **stop making the sender wait.** Allow it to send several packets back-to-back, without pausing for an acknowledgment after each one. Because many packets are now "in flight" between sender and receiver simultaneously — filling up the transmission path the way water fills a pipe — this technique is called **pipelining**.

```
   STOP-AND-WAIT                         PIPELINED
   (one packet "in flight" at a time)    (multiple packets in flight at once)

   Sender         Receiver               Sender         Receiver
     │                │                    │░░░░░░░░░░░░░░░│
     │──── pkt ───────▶│                    │░░░ pkt,pkt ░░░▶│   (several packets
     │                │                    │░░░░░░░░░░░░░░░│    traveling together)
     │◄──── ACK ───────│                    │◄── ACK,ACK ───│
     │  (then repeat)  │                    │  (then repeat with more in flight) │
```

Going back to the same coast-to-coast example: if the sender is allowed to transmit **three packets before waiting** for any acknowledgments, sender utilization roughly **triples** — for essentially zero added hardware cost, just a change in protocol logic.

Pipelining isn't free, though — it forces three structural consequences onto any reliable transfer protocol:

1. **The range of sequence numbers must grow.** With multiple packets in flight simultaneously, each one needs a _distinct_ sequence number so the receiver can tell them apart — 1 bit is no longer enough.
2. **Buffering is required on both ends.** At minimum, the sender must hold onto copies of packets it's sent but not yet had acknowledged (in case it needs to retransmit them). The receiver may also need to buffer correctly-received packets that arrive out of order.
3. **A design choice opens up for how to recover from errors** when many packets are in flight: should a single lost/corrupted packet force retransmission of _everything_ sent after it, or only that one packet specifically? This fork in the road produces the two approaches covered next: **Go-Back-N** and **Selective Repeat**.

---

## 3.4.3 Go-Back-N (GBN)

### The Sliding Window

GBN's central idea: cap the number of unacknowledged, "in-flight" packets at some maximum value **N**, called the **window size**.

![[Pasted image - Fig 3.19 GBN sender sequence-number view]] _(Figure 3.19 — A horizontal strip of sequence numbers, divided into four zones: already-ACK'd, sent-but-not-yet-ACK'd, usable-not-yet-sent, and not-usable, with a bracket of width N labeled "Window size" spanning the middle two zones.)_

```
   [ 0 ... base-1 ]   [ base ... nextseqnum-1 ]   [ nextseqnum ... base+N-1 ]   [ base+N ... ]
    already ACK'd       sent, not yet ACK'd          usable, not yet sent        not usable yet
                        |←──────────────── window of size N ─────────────────→|
```

- **`base`** = sequence number of the _oldest unacknowledged_ packet.
- **`nextseqnum`** = the _next_ sequence number available to use.
- As ACKs come in, **the window slides forward** over the sequence-number space — hence GBN (and SR) are both called **sliding-window protocols**.

Sequence numbers in practice live in a _fixed-width_ header field (k bits → range [0, 2^k − 1]), so all arithmetic on them wraps around modulo 2^k, like a circular dial. (`rdt3.0`'s single bit was the smallest possible such field — a "dial" with only two positions, 0 and 1.)

### GBN Sender — Three Events to Handle

1. **Invocation from above (`rdt_send`)** — if the window isn't full (fewer than N unacknowledged packets outstanding), package and send the new data immediately, advancing `nextseqnum`. If the window _is_ full, the sender simply can't accept the data right now — in a real implementation this means buffering it or blocking the upper layer until room opens up.
2. **Receipt of an ACK** — GBN uses **cumulative acknowledgments**: an ACK for sequence number _n_ means "everything up to and including _n_ has been correctly received," not just packet _n_ alone. This single design choice is what makes GBN simple — and is also exactly where its weakness comes from (see below).
3. **A timeout** — the protocol's namesake behavior. If the timer for the _oldest_ unacknowledged packet expires, the sender **resends every packet currently in the window** (every packet sent but not yet acknowledged), not just the one that timed out — hence "Go-Back-**N**." Only a **single timer** is needed (for the oldest outstanding packet); it restarts whenever an ACK arrives but unacknowledged packets remain, and stops entirely once the window is empty.

### GBN Receiver — Strictly In-Order, Nothing Else Accepted

The receiver's logic is almost aggressively simple: if an arriving packet has exactly the sequence number it's currently expecting, and is uncorrupted, it gets ACK'd and delivered up to the application; **anything else gets discarded**, and the receiver re-sends an ACK for the most recently received _in-order_ packet.

This includes a subtle and important case: if the receiver is expecting packet _n_ and packet _n+2_ shows up instead (because packet _n+1_ was lost), GBN's receiver **throws packet n+2 away** — even though its data will eventually need to be delivered anyway. Why not just buffer it for later? Doing so is _possible_ (and real implementations sometimes do exactly that as an optimization) — but it adds real complexity and memory cost to the receiver, and "basic" GBN as specified here keeps the receiver dumb on purpose, pushing all the complexity onto the sender's retransmission logic instead.

### GBN in Action — Why a Single Lost Packet Hurts

Picture a window size of 4, sending packets 0, 1, 2, 3 back to back. Packet 2 is lost in transit.

```
SENDER                          RECEIVER
send pkt0 ───────────────────▶  rcv pkt0 → ACK0
send pkt1 ───────────────────▶  rcv pkt1 → ACK1
send pkt2 ───X (lost)
send pkt3 (wait, window full)──▶ rcv pkt3, OUT OF ORDER → discard, resend ACK1
rcv ACK0 → send pkt4 ─────────▶ rcv pkt4, OUT OF ORDER → discard, resend ACK1
rcv ACK1 → send pkt5 ─────────▶ rcv pkt5, OUT OF ORDER → discard, resend ACK1
[pkt2's timer expires]
resend pkt2, pkt3, pkt4, pkt5 ▶ rcv pkt2 → deliver pkt2,3,4,5 → send ACK2, ACK3
```

Notice the cost: losing **just one packet (pkt2)** forces the sender to retransmit **four packets** (2, 3, 4, and 5) — even though 3, 4, and 5 had already arrived safely the first time and were simply discarded by the receiver for being "out of order." The bigger the window (and the longer the bandwidth-delay product), the worse this gets: a single error can force re-sending dozens or hundreds of perfectly good packets.

**Analogy:** Imagine dictating 1,000 words at a time as a single "window," and if even one word in the middle gets garbled, the _entire_ 1,000-word block must be re-dictated from that word onward — even the words the listener already heard correctly. That's an enormous, unnecessary cost for one small mistake.

---

## 3.4.4 Selective Repeat (SR)

### The Fix: Be Surgical, Not Blunt

Selective Repeat keeps GBN's core idea (a sliding window of size N, multiple packets in flight) but removes its wasteful, "resend everything" reflex. As the name suggests, SR **retransmits only the specific packets that were actually lost or corrupted** — nothing else.

To make this possible, SR requires two structural changes from GBN:

1. **Individual acknowledgments**, not cumulative ones — every correctly received packet is ACK'd on its own, regardless of order, so the sender knows precisely _which_ packets made it and which didn't.
2. **Individual, per-packet timers** — since any one specific packet might need to be retransmitted on its own, each outstanding packet needs its own logical timeout clock (in practice, often simulated using a single hardware timer pretending to be many).

```
   SR sender's view of sequence numbers (Figure 3.23)
   [ already ACK'd ] [ sent, not yet ACK'd ] [ usable, not yet sent ] [ not usable ]
                      |←──────── window size N ─────────→|
```

Unlike GBN, **the sender will already have individually-confirmed ACKs for _some_ packets inside its window** even while others in that same window remain unacknowledged — there's no requirement that everything be confirmed strictly in order.

### SR Sender — Three Events

1. **Data from above** — if the next sequence number falls inside the sender's window, packetize and send it; otherwise buffer it (or hand it back to the upper layer) for later, exactly as in GBN.
2. **Timeout** — only the _one specific_ packet whose timer expired gets resent. Nothing else.
3. **ACK received** — mark that specific packet as received. If its sequence number equals `send_base` (the window's lower edge), slide the window forward to the next still-unacknowledged packet, and immediately transmit any newly-eligible packets that fall into the window as a result.

### SR Receiver — Buffer, Don't Discard

The receiver acknowledges **every** correctly received packet individually, whether or not it arrived in order:

- If a packet falls within the receiver's current window, it is **buffered** (if it's the first copy) and individually ACK'd. If its sequence number equals the window's base, that packet — _plus_ any previously-buffered packets that are now consecutively numbered starting from the base — get delivered **together, as a batch**, to the upper layer, and the window slides forward by however many packets were just delivered.
- If a packet's sequence number falls in the range _just behind_ the current window (meaning the receiver has already seen and ACK'd it before), the receiver **still sends an ACK again** — even though this looks redundant.
- Anything else is simply ignored.

### SR in Action — Why It's More Efficient Than GBN

Reusing the same scenario as before — window size 4, packet 2 lost:

```
SENDER                          RECEIVER
send pkt0 ───────────────────▶  rcv pkt0 → deliver immediately → ACK0
send pkt1 ───────────────────▶  rcv pkt1 → deliver immediately → ACK1
send pkt2 ───X (lost)
send pkt3 (window full)───────▶ rcv pkt3 → BUFFER (out of order) → ACK3
rcv ACK0 → send pkt4 ─────────▶ rcv pkt4 → BUFFER → ACK4
rcv ACK1 → send pkt5 ─────────▶ rcv pkt5 → BUFFER → ACK5
[pkt2's individual timer expires]
resend ONLY pkt2 ─────────────▶ rcv pkt2 → deliver pkt2, then immediately
                                  also deliver buffered pkt3, pkt4, pkt5 → ACK2
```

Only **one** packet (pkt2) ever needs to be resent — pkt3, pkt4, and pkt5 were correctly buffered the first time around and are simply released to the application once the missing piece (pkt2) finally shows up.

**Analogy:** This is like ordering five grocery items for delivery and only one item is out of stock. SR's approach: the store ships the four available items right away, holds them in your fridge (figuratively), and re-ships only the missing item once it's back in stock — then hands you everything together. GBN's approach, by contrast, would have canceled and re-shipped _all five_ items just because one was unavailable.

### Why the "Redundant" Re-ACK Step Actually Matters

It might seem wasteful for the SR receiver to re-acknowledge a packet it has _already_ acknowledged before — but this step is not optional. If there's no ACK at all currently traveling back to the sender for the packet at `send_base`, the sender will eventually time out and **retransmit it again**, even though (from our God's-eye view) the receiver clearly already has it. If the receiver didn't re-ACK in that case, the sender's window would **never be able to slide forward** — it would be stuck waiting forever for confirmation of something the receiver already has, simply because the _first_ ACK never made it back.

This points to a deeper, important truth about SR (and many real-world protocols): **the sender and the receiver do not necessarily have an identical view of what's been correctly received.** Because of this, their respective windows over the sequence-number space won't always perfectly line up with each other — and the protocol has to be designed defensively around that mismatch, rather than assuming both sides are always perfectly in sync.

### A Closing Caveat: What If the Channel Can Reorder Packets?

Everything above assumed the channel **never reorders** packets — reasonable when sender and receiver are connected by a single physical wire, but _not_ reasonable once the "channel" is an actual network, where packets can take different paths and arrive out of the order they were sent.

Reordering introduces a sneaky failure mode: an old, stale copy of a packet bearing some sequence number _x_ could resurface and arrive at some arbitrary point in the future — even after _neither_ the sender's nor the receiver's current window still contains _x_. Since sequence numbers eventually get reused (the numbering space is finite), there's a real risk of a late-arriving "ghost" packet being mistaken for a brand-new one with the same number.

The practical fix: never reuse a sequence number until you're confident **no old packet carrying that same number could still possibly be lurking in the network**. This is generally handled by assuming a **maximum packet lifetime** — the TCP extensions for high-speed networks (RFC 7323) assume roughly **three minutes** as that upper bound.

---

## Putting It All Together

GBN and SR together account for almost every reliable-data-transfer technique that real-world TCP uses: sequence numbers, cumulative _and_ selective-style acknowledgment behavior, checksums, sliding windows, and timeout/retransmission logic. The next section (3.5) builds directly on top of everything covered here — TCP is, in essence, a hybrid that borrows GBN's cumulative-ACK simplicity while incorporating several SR-like optimizations (such as selective ACK extensions) to avoid GBN's worst-case over-retransmission problem.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Sequence numbers identify packets, not authenticate them**|If sequence numbers are predictable, an attacker can craft forged packets that the receiver's FSM will accept as legitimate (the basis of classic TCP sequence-number prediction / session-hijacking attacks)|Use unpredictable, randomized initial sequence numbers; never rely on sequence numbers alone as a security boundary|
|**ACKs are trusted blindly by the sender FSM**|Spoofed ACK packets (with the right sequence number) can trick a sender into believing data was delivered, or can be used to manipulate window/flow-control behavior|Pair sequence-number-based reliability with cryptographic integrity (e.g., TLS) when tampering matters, not just accidental corruption|
|**Timers and retransmission create exploitable timing behavior**|Deliberately inducing packet loss or delay (forcing retransmissions) can be used to degrade performance or as a side channel to infer network conditions|Rate-limit retransmission storms; monitor for abnormal retransmission patterns as a sign of active interference|
|**GBN discards out-of-order packets — a predictable, exploitable rule**|An attacker who can selectively drop just the _right_ packet (e.g., always dropping packet _n_ in a stream) can force large, repeated cascades of retransmission with very little effort — a low-cost denial-of-service amplification against GBN-style senders|Prefer SR-style selective retransmission and/or rate-limit "go-back" cascades; detect repeated suspiciously-targeted single-packet loss|
|**Stop-and-wait's strict single-flight-packet rule is easy to starve**|An attacker who can delay or drop just the single in-flight packet/ACK pair can stall an entire stop-and-wait exchange indefinitely with minimal effort|Avoid stop-and-wait for anything performance- or availability-sensitive; pipelined protocols are inherently more resilient to single-point interference|

---

## Questions I Still Have

- [ ] In `rdt2.2`'s NAK-free design, is there a real risk of the sender misreading two _unrelated_ ACKs as a "duplicate ACK" if timing lines up unluckily, or does the sequence number make that impossible by construction?
- [ ] Real TCP doesn't strictly do pure GBN _or_ pure SR — how exactly does TCP's selective acknowledgment (SACK) option blend these two models, and where does TCP's actual behavior diverge from the "textbook" GBN sketch in 3.4.3?
- [ ] For SR's per-packet timers, the text mentions a single hardware timer can simulate multiple logical ones — what's the actual data structure/algorithm used in real OS network stacks to do this efficiently for thousands of simultaneous connections?
- [ ] Given the three-minute "maximum packet lifetime" assumption (RFC 7323) used to justify safely reusing sequence numbers — what actually happens in extremely high-speed, high-RTT scenarios where this assumption gets stretched thin (this seems closely tied to why TCP eventually needed 32-bit, not smaller, sequence numbers)?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Reliable data transfer protocol**|Builds a "no loss, no corruption, in-order" service abstraction on top of a genuinely unreliable lower-layer channel|
|**ARQ (Automatic Repeat reQuest)**|Family of protocols that achieve reliability via error detection, feedback (ACK/NAK), and retransmission|
|**Stop-and-wait**|A protocol that sends exactly one unacknowledged packet at a time before pausing|
|**Sequence number**|A field that lets the receiver tell new data apart from a retransmitted duplicate|
|**NAK-free design (`rdt2.2`)**|Uses only ACKs; a duplicate ACK for the same sequence number functions as an implicit NAK|
|**Alternating-bit protocol**|Nickname for `rdt3.0`, since its 1-bit sequence number flips 0↔1 with every successful round|
|**Countdown timer**|Mechanism that triggers retransmission after a chosen interval if no ACK has arrived, used to detect packet/ACK loss|
|**Utilization (U_sender)**|Fraction of total round-trip time the sender actually spends transmitting bits; the metric stop-and-wait performs badly on|
|**Pipelining**|Allowing multiple packets to be in flight, unacknowledged, at the same time|
|**Sliding window**|The range of permissible in-flight sequence numbers, which slides forward over time as ACKs arrive|
|**Go-Back-N (GBN)**|Pipelined protocol using cumulative ACKs; on timeout/error, resends _every_ unacknowledged packet in the window|
|**Cumulative acknowledgment**|An ACK for packet _n_ that implicitly confirms all packets up to and including _n_|
|**Selective Repeat (SR)**|Pipelined protocol using individual ACKs and individual timers; retransmits only the specific packet(s) that were lost/corrupted|
|**Receiver buffering (SR)**|Holding correctly-received but out-of-order packets until the missing earlier packet(s) arrive, then delivering them as a batch|
|**Maximum packet lifetime**|Assumed upper bound (~3 minutes per RFC 7323) on how long a packet can persist in the network, used to safely justify reusing sequence numbers|

---

## Related Concepts

- [[3.3 Connectionless Transport UDP]] — the checksum mechanism reused directly in `rdt2.0`'s error detection
- [[3.2 Multiplexing and Demultiplexing]] — the layer below this one's interface assumptions
- [[End-to-end Principle]] — the same Saltzer 1984 idea that justified UDP's checksum also explains why reliability mechanisms recur at multiple layers

---

→ Next: [[3.5 Connection-Oriented Transport TCP]]