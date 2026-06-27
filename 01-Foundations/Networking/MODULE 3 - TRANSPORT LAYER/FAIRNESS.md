---
title: FAIRNESS
date: 2026-06-27
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 3.7 TCP Congestion Control

> **One-Line Summary:** TCP congestion control is the part of TCP that has nothing to do with correctness and everything to do with courtesy — using only the implicit signals available at the sender (ACKs arriving, ACKs failing to arrive, and now sometimes an explicit network-set bit), each TCP sender continuously feels out how much of the shared network it can use without tipping it into collapse, growing boldly when things look clear and backing off sharply the moment they don't.

---

## Core Idea: A Second, Separate Job for TCP

Section 3.5 covered how TCP turns IP's unreliable service into a _reliable_ one. Congestion control is TCP's other major responsibility, and it's a genuinely different problem: reliable data transfer is about not losing or corrupting _your own_ data; congestion control is about not contributing to the network's collective overload.

The IP layer gives transport protocols **zero explicit feedback** about congestion — no router tells a sender "you're sending too fast." Historically, TCP has had to _infer_ congestion purely from how its segments and ACKs behave, an approach this section calls **"Classic" TCP** (standardized in RFC 2581, then RFC 5681). This section walks through four real congestion-control "flavors," in increasing order of sophistication: **Classic TCP**, then newer end-to-end approaches that interpret signals differently (**TCP Vegas**, **CUBIC**, **BBR**), then an approach that gets actual help from the network layer (**Explicit Congestion Notification**), before closing with the question of **fairness** — whether competing flows actually end up sharing a link reasonably.

Three questions frame everything that follows: How does a sender _limit_ its rate? How does a sender _perceive_ that there's congestion? And what _algorithm_ should govern how the sender adjusts its rate in response?

---

## 3.7.1 Classic End-to-End TCP Congestion Control

### The Congestion Window: A Second Constraint on Top of Flow Control

Recall from Section 3.5 that each side of a TCP connection maintains a receive buffer, a send buffer, and a handful of variables (`LastByteRead`, `rwnd`, and so on). Congestion control introduces one more sender-side variable: the **congestion window**, denoted **`cwnd`**.

Where flow control's `rwnd` constrains the sender based on what the _receiver's_ buffer can hold, `cwnd` constrains the sender based on what the _network_ can currently absorb. The two constraints combine — the amount of unacknowledged data a sender may have outstanding can never exceed the smaller of the two:

```
LastByteSent − LastByteAcked ≤ min{cwnd, rwnd}
```

To isolate congestion control from flow control for the rest of this discussion, assume the receive buffer is generously large, so `rwnd` is never the binding constraint — `cwnd` alone governs how much unacknowledged data the sender may have in flight. Also assume the sender always has data ready to send, so every byte the congestion window allows actually gets sent.

> **Analogy — Two Independent Speed Limits:** Imagine a delivery driver constrained by two separate rules: how much the customer's loading dock can physically hold (flow control, `rwnd`) and how much traffic the road can handle without gridlock (congestion control, `cwnd`). The driver always obeys whichever rule is currently stricter. Assuming the dock has effectively unlimited space lets us study the _traffic_ rule in isolation — which is exactly the simplifying assumption made here.

### From Window Size to Sending Rate

Because the constraint above limits the amount of unacknowledged data outstanding, it indirectly limits the sender's actual transmission _rate_. Consider a connection where losses and transmission delays are negligible: at the start of any RTT, the sender can push roughly `cwnd` bytes into the connection; by the end of that RTT, acknowledgments for that data arrive back. So the sender's effective send rate works out to roughly:

```
send rate ≈ cwnd / RTT   (bytes per second)
```

By adjusting the value of `cwnd`, the sender directly adjusts how fast it sends — which is precisely _the_ mechanism the rest of this section is about tuning.

### Defining a "Loss Event"

For classic TCP's purposes, a **loss event** at the sender is the occurrence of _either_ a timeout _or_ the receipt of three duplicate ACKs (recall Section 3.5.4's fast retransmit, triggered by exactly this condition). When excessive congestion causes a router buffer along the path to overflow, a datagram gets dropped — and that dropped segment surfaces at the sender as one of these two loss events. Other, more nuanced congestion _signals_ exist too — changes in RTT, for instance — and some of the newer flavors covered later in this section make direct use of them, but classic TCP relies on loss events alone.

### Self-Clocking: ACKs Drive Everything

When the network is congestion-free — that is, when a loss event _doesn't_ occur — acknowledgments for previously unacknowledged segments keep arriving at the sender. Classic TCP treats the arrival of these ACKs as the signal that things are going well, and uses them to _trigger_ increases in the congestion window. Because the very act of receiving an ACK is what causes `cwnd` to grow, TCP is said to be **self-clocking**: there's no external clock ticking off "increase now" — the network's own acknowledgment traffic provides the timing. A consequence worth noticing: if ACKs arrive slowly (high-delay or low-bandwidth path), `cwnd` grows slowly too; if ACKs arrive quickly, `cwnd` grows quickly.

### Two Guiding Principles

Classic TCP's congestion-control algorithm is built from two simple, almost commonsense rules:

|Principle|Statement|
|---|---|
|**Loss implies congestion**|A lost segment implies congestion, and hence the sender's rate should be _decreased_ when a segment is lost. (Recall: a timeout, or the implicit signal of three duplicate ACKs, both mean a segment is presumed lost.)|
|**An ACK implies the network is delivering successfully**|An acknowledged segment indicates the network is successfully delivering the sender's data to the receiver — so the sender's rate _can be increased_ when an ACK arrives for previously-unacknowledged data.|

Put together, these two rules describe a strategy of **bandwidth probing**: increase the sending rate in response to arriving ACKs until a loss event occurs, at which point cut back — then start probing again to see if conditions have improved.

> **Analogy — The Child Who Keeps Asking:** A child who wants more treats keeps asking for "just one more" until finally told "No!" — at which point they back off a bit, wait, and then start asking again a little later. TCP's sender behaves identically: it keeps nudging its rate upward as long as ACKs keep confirming success, and only pulls back the moment it's told "no" (a loss event) — then resumes probing shortly after. Crucially, there's no central referee handing out "yes" and "no" answers to everyone at once — each sender is privately, asynchronously probing based only on what it personally observes.

### The Three Components of Classic TCP Congestion Control

Classic TCP's congestion-control algorithm — first described by Jacobson (1988) and standardized in RFC 5681 — has three major pieces:

|Component|Required?|What It Does|
|---|---|---|
|**Slow start**|Mandatory|Grows `cwnd` _rapidly_ at the start of a connection (or after a severe loss event)|
|**Congestion avoidance**|Mandatory|Grows `cwnd` more _cautiously_ once the sender believes it's getting close to the available bandwidth|
|**Fast recovery**|Recommended, not required|Handles recovery from a triple-duplicate-ACK loss event without falling all the way back to slow start|

---

### Slow Start

When a TCP connection first begins, `cwnd` is typically initialized to a small value — **1 MSS** (RFC 3390) — giving an initial sending rate of roughly MSS/RTT.

**Worked example:** With MSS = 500 bytes and RTT = 200 ms, the initial sending rate works out to only about 20 kbps — almost certainly far below what the available bandwidth could actually support. So in the **slow-start** state, `cwnd` begins at 1 MSS and **doubles every time a transmitted segment is first acknowledged.** Concretely: the sender transmits one segment and waits; when its ACK arrives, `cwnd` increases by one MSS, and the sender now sends _two_ maximum-sized segments; when both are acknowledged, `cwnd` increases again, this time to 4 MSS — and so on. Each round trip doubles the send rate.

> Despite the name, slow start is anything but slow in its _growth rate_ — it grows the send rate **exponentially fast**, doubling every RTT. The name refers only to the conservatively small _starting point_ (1 MSS), not the pace of growth from there.

![[Pasted image 20260627122549.png]]

### When Does Slow Start End?

Two distinct events end the exponential-growth phase:

|Trigger|What Happens|
|---|---|
|**A loss event occurs (timeout)**|`cwnd` is reset to 1 MSS, and the slow-start process begins anew. A state variable `ssthresh` ("slow-start threshold") is also set to `cwnd/2` — half the congestion-window value at the moment congestion was detected|
|**`cwnd` reaches `ssthresh`**|It would be reckless to keep doubling indefinitely once the sender is approaching a previously-known danger zone. When `cwnd` equals `ssthresh`, slow start ends and TCP transitions into **congestion-avoidance** mode|

Because `ssthresh` is _directly tied_ to the `cwnd` value present when congestion was last detected, it functions as TCP's institutional memory of "here's roughly where things got dangerous last time" — useful precisely because it tells the sender when to start being careful again on the way back up.

---

### Congestion Avoidance

On entering congestion avoidance, `cwnd` is approximately half of what it was when congestion last occurred — meaning trouble could plausibly be just around the corner again. So rather than continuing to double `cwnd` every RTT, TCP switches to a far more conservative strategy: increasing `cwnd` by just **one MSS per RTT** (RFC 5681).

In practice, this linear increase is commonly implemented per-ACK rather than per-RTT: the sender increases `cwnd` by `MSS²/cwnd` bytes for every new ACK that arrives. (**Worked example:** if MSS = 1,460 bytes and `cwnd` = 14,600 bytes, exactly 10 segments fit in one RTT. Each arriving ACK increases `cwnd` by 1/10 MSS, so after all 10 ACKs for that round trip have arrived, `cwnd` has grown by exactly one full MSS — reproducing the "one MSS per RTT" behavior from many small per-ACK increments.)

### What Happens on a Loss Event During Congestion Avoidance?

The response now _differs_ depending on which kind of loss event occurred:

|Loss Signal|Response|
|---|---|
|**Timeout**|Behaves the same as slow start's response: `cwnd` is reset to 1 MSS, and `ssthresh` is updated to half the `cwnd` value at the time of the loss|
|**Triple duplicate ACK**|A _less drastic_ response, because triple duplicate ACKs mean the network is _still_ delivering some segments (the receiver's duplicate ACKs prove data is getting through) — `cwnd` is **halved** (with an added +3 MSS "for good measure," accounting for the three duplicate ACKs already received), and `ssthresh` is also set to half the `cwnd` value at the loss|

> **Why treat the two loss signals differently?** A timeout suggests a _severe_ disruption — nothing at all has been heard back for an extended period, so TCP responds as cautiously as it did when starting completely from scratch. A triple duplicate ACK is comparatively _good news in disguise_ — it means the receiver is still actively receiving and acknowledging segments past the lost one — so TCP only needs to back off, not restart from zero.

---

### Fast Recovery

If implemented, **fast recovery** manipulates `cwnd` differently than congestion avoidance does, specifically following a triple-duplicate-ACK event: `cwnd` is increased by 1 MSS for _every additional duplicate ACK_ received for the missing segment. As soon as an ACK arrives that acknowledges new data — or a timeout occurs — TCP returns to congestion avoidance or slow start, respectively.

### TCP Tahoe vs. TCP Reno

|Version|Response to _Any_ Loss Event (Timeout OR Triple-Dup-ACK)|
|---|---|
|**TCP Tahoe** (one of the earliest versions)|_Always_ cuts `cwnd` all the way to 1 MSS and re-enters slow start, regardless of which kind of loss event occurred|
|**TCP Reno** (the next major version)|Responds _less drastically_ to a triple-duplicate-ACK loss: instead of collapsing to 1 MSS, it **halves** `cwnd` (and adds three) — the change described above|

**Worked comparison (Figure 3.50's scenario):** Suppose `ssthresh` starts at 8 MSS for both versions. For the first eight transmission rounds, Tahoe and Reno behave identically — climbing exponentially during slow start until they hit the threshold at round 4, then climbing _linearly_ (congestion avoidance) until a triple-duplicate-ACK event occurs just after round 8, when `cwnd` is 12 MSS.

![[Pasted image 20260627122939.png]]

| |Reno (after the round-8 triple-dup-ACK event)|Tahoe (same event)|
|---|---|---|
|**New `ssthresh`**|0.5 × 12 MSS = **6 MSS**|6 MSS|
|**New `cwnd`**|**9 MSS** (half of 12, plus 3) — then grows linearly|**1 MSS** — then grows exponentially until reaching `ssthresh`, at which point it switches to linear growth|

---

### TCP Congestion Control: Retrospective — AIMD and the Sawtooth

Ignoring the initial slow-start phase, and assuming losses are signaled by triple duplicate ACKs rather than timeouts, classic TCP's steady-state behavior boils down to: **linear (additive) increase** of `cwnd` by one MSS per RTT, followed by **halving (multiplicative decrease)** of `cwnd` on a triple-duplicate-ACK event. This pattern earns the name **AIMD — Additive-Increase, Multiplicative-Decrease** congestion control, and it produces the characteristic **"sawtooth"** shape when `cwnd` is plotted against time:

![[Pasted image 20260627123252.png]]

This sawtooth is the visual signature of TCP's "probing for bandwidth" intuition: the window climbs linearly (probing upward) until a loss event occurs, drops sharply by a factor of two, and immediately starts climbing again to see whether more bandwidth has become available. Remarkably, although AIMD was developed largely from engineering insight and real-network experimentation rather than from first-principles theory, later analysis (roughly a decade after deployment) showed that TCP's congestion-control algorithm behaves like a distributed asynchronous optimization procedure, simultaneously optimizing several important aspects of both individual and aggregate network performance — a result that gave rise to a substantial body of subsequent congestion-control theory.

### Macroscopic Throughput: How Fast Does a Long-Lived Reno Connection Really Go?

Ignoring the brief slow-start phases (which the sender grows out of exponentially fast, so they contribute little to the overall average), it's possible to derive the **average throughput** of a long-lived TCP Reno connection purely from its sawtooth shape.

During any given round-trip interval, the connection's sending rate is roughly `w/RTT`, where `w` is the current window size in bytes. TCP probes for more bandwidth by increasing `w` by 1 MSS every RTT, until a loss event occurs. Let `W` denote the value of `w` at the _moment_ a loss event occurs. Assuming RTT and `W` stay roughly constant over the connection's life, the sending rate therefore ranges between two extremes:

```
W/(2·RTT)   ──────────────────────────────►   W/RTT
 (right after a halving)              (right before the next loss)
```

This idealized picture — the network drops a packet exactly when the rate hits `W/RTT`; the rate is cut in half, then climbs back up by `MSS/RTT` every RTT until it reaches `W/RTT` again, repeating indefinitely — is a highly simplified macroscopic model of TCP's steady-state behavior. Because the throughput increases _linearly_ between these two extreme values, the **average throughput of a connection** works out to:

```
average throughput of a connection = 0.75 · W / RTT
```

This idealized model can be extended further to derive a relationship between a connection's _loss rate_ and the bandwidth it can sustain — a useful (if simplified) link between two quantities that, at first glance, seem like they should be independent.

---

## 3.7.2 Recent End-End TCP Congestion Control Algorithms

Over roughly the past 25 years, many new end-to-end congestion-control algorithms have been developed and deployed. Measurements of the congestion-control flavors actually used by the most popular web sites on the Internet show that classic TCP — the algorithm just described in painstaking detail — has, in practice, been **largely replaced** by newer flavors, especially **BBR** and, before that, **CUBIC**. Some of these newer approaches only tweak classic TCP's loss-as-a-congestion-signal philosophy slightly; others abandon that philosophy altogether.

### TCP CUBIC: Improving on Additive Increase

TCP Reno's AIMD approach raises a natural question: is _always_ increasing additively the best way to probe for a sending rate just below the threshold that triggers packet loss? If the congestion state of the link hasn't actually changed much since the last loss, cautiously climbing back up one MSS at a time may be needlessly slow — it might be smarter to **quickly** ramp the rate back up close to the pre-loss sending rate, and only _then_ switch to cautious probing for anything beyond that. This insight is the core idea behind **TCP CUBIC** (and its predecessor, BIC).

CUBIC differs from Reno only in how it behaves during the congestion-avoidance phase (the congestion window is still only increased on ACK receipt). Define:

|Variable|Meaning|
|---|---|
|**`W_max`**|The size of the congestion window at the moment loss was most recently detected|
|**`K`**|A future point in time at which CUBIC's congestion window would again reach `W_max`, assuming no further losses occur|

CUBIC increases the congestion window as a function of the **cube** of the distance between the current time `t` and `K`:

|Regime|Behavior|
|---|---|
|**`t` is much less than `K`** (far from `K`, i.e., right after a loss)|The cubic term is large, so the window increases **rapidly** — CUBIC quickly ramps the sending rate back up close to the pre-loss rate, `W_max`|
|**`t` is close to `K`**|The cubic term shrinks toward zero, so window growth is **small and cautious** — exactly the regime where the link's congestion level likely hasn't changed much|
|**`t` is greater than `K`**|The cubic rule implies window increases stay small while `t` is still near `K` (cautious probing near the known danger point), but then **increase rapidly** again as `t` moves further past `K` — letting CUBIC quickly discover a _new_ operating point if the link's congestion level has genuinely changed|

![[Pasted image 20260627123446.png]]

The net effect: CUBIC spends more of its time hovering near (just below) the link's actual, unknown congestion threshold than Reno does — Reno's purely linear climb spends a lot of time well below `W_max` on its way back up after every halving, while CUBIC gets there much faster and lingers there longer, enjoying more overall throughput.

> **Adoption history, by the numbers:** Around the year 2000, nearly all popular web servers ran classic TCP. By 2014, measurements of the most popular web servers found that roughly half had switched to a CUBIC variant, which also became the default TCP congestion-control algorithm in the Linux kernel. By 2024, however, **BBR usage had surpassed CUBIC usage — and indeed surpassed the combined usage of every other flavor of TCP congestion control put together.**

### TCP Vegas: Delay-Based Congestion Control

Classic TCP, CUBIC, and most other flavors covered so far all share one trait: they rely on **segment loss** — inferred via a timeout or a triple duplicate ACK — as their _only_ indication of congestion. **TCP Vegas** takes a fundamentally different, _proactive_ approach: it uses measured **RTT delay** to detect the _onset_ of congestion, ideally **before** any packet loss actually occurs.

|Step|Mechanism|
|---|---|
|1|The sender measures the RTT of the source-to-destination path for every acknowledged packet|
|2|Let `RTT_min` be the **minimum** of these measurements — this occurs when the path is uncongested and packets experience minimal queueing delay|
|3|If the connection's congestion window is `cwnd`, the _uncongested_ throughput rate would be `cwnd / RTT_min`|
|4|**If the sender's actual measured throughput is close to this uncongested rate**, the path is (by definition and measurement) not yet congested — the sending rate can be safely increased|
|5|**If actual measured throughput is significantly less than the uncongested rate**, the path is becoming congested — the sender proactively decreases its sending rate, _without waiting for an actual loss event_|

> **Analogy — Watching Your Commute Time Creep Up:** Imagine judging traffic not by whether you eventually crash (loss-based), but by noticing your commute is quietly taking longer than your personal best time on this route (delay-based). A creeping commute time is an early warning that congestion is building, well before gridlock (packet loss) actually happens — exactly the signal Vegas exploits.

Vegas was among the first Internet congestion-control protocols to make use of measured delay this way; several later protocols have since adopted similar delay-based approaches.

### BBR: Bottleneck Bandwidth and Round-Trip Propagation Time

**BBR** (Bottleneck Bandwidth and RTT) operates on the intuition of **"keep the pipe just full, but no fuller."** "Keeping the pipe full" means the bottleneck link limiting the connection's throughput should always be kept busy doing useful work. "But no fuller" means there's nothing to gain — except _increased delay_ — from letting large queues build up at that bottleneck once it's already fully utilized. Developed initially by Google and now in its third version (BBRv3), BBR has, as of 2024, become **the single most widely deployed flavor of TCP congestion control.**

#### The Key Quantity: In-Flight Packets

Recall the notion of a bottleneck link. As more connections pass through a shared bottleneck, each is progressively allowed more unacknowledged ("in-flight") packets between sender and receiver. BBR tracks this directly via a quantity called **`n_inflight`** — the number of in-flight packets — which is subtly different from a connection's raw transmission rate:

|Quantity|What It Measures|
|---|---|
|**Transmission rate**|A strict measure of _how fast_ data is being sent|
|**`n_inflight`**|A _count_ of packets currently in transit — implicitly also a function of the path's RTT, since a longer pipe naturally needs to hold more packets to stay "full" than a shorter one|

#### Three Considerations Behind BBR's Design

|Consideration|Reasoning|
|---|---|
|**1. Keep the pipe full**|The bottleneck link should be kept continuously busy doing useful work — under-filling it wastes available capacity|
|**2. Keep queues short ("no fuller")**|Some buffering is needed to absorb statistical fluctuations in arrival rate, but _persistent_ queues at the bottleneck should be avoided — a persistent queue only adds delay, not throughput|
|**3. Account for the bandwidth-delay product**|A connection's path can hold more in-flight data, without loss, the larger its link capacity _and_ the larger its RTT — long, high-bandwidth "long, fat pipes" can support more in-flight packets than short, low-bandwidth ones. The **bandwidth · delay product** approximates the average amount of in-flight data a connection's path can sustain _before_ a queue starts forming|

![[Pasted image 20260627123717.png]]

As `n_inflight` increases from zero, throughput initially climbs while RTT stays flat (the in-flight data is distributed evenly through the pipe, with RTT consisting purely of propagation delay). Once in-flight data reaches the **bandwidth·delay product**, the pipe is "just full enough" — increasing `n_inflight` further no longer increases throughput (since there's already a persistent queue forming at the bottleneck) but _does_ start increasing RTT, due to that growing queueing delay. The point where `n_inflight` exactly equals the bandwidth·delay product is, in this idealized picture, the **ideal operating point** — full, but not over-full. (In practice, statistical traffic variation means this is an average-case approximation rather than an exact line.)

#### Three Operating Phases

BBR estimates the bandwidth·delay product directly by measuring RTT and setting `n_inflight` accordingly, operating at two different timescales — at large intervals, it periodically reduces `n_inflight` to _drain_ the pipeline and check whether a new, lower RTT has become achievable; at shorter intervals, it cycles through three phases:

|Phase|Behavior|
|---|---|
|**Acceleration**|BBR increases its sending rate (raising `n_inflight`) until reaching a throughput plateau — saturating the available per-flow capacity|
|**Cruising**|BBR sends at the rate the network is actually delivering data, as evidenced by received ACKs|
|**Deceleration**|BBR deliberately sends slower than the network is delivering, reducing `n_inflight` and easing queue pressure — specifically to check for a lower RTT|

BBR also includes a **transmission pacing mechanism**, ensuring large bursts of segments aren't sent back-to-back-to-back, smoothing out how traffic actually hits the network. A known concern with earlier BBR versions was **fairness** when competing against traditional loss-based flows (CUBIC or Reno) at the same bottleneck — a concern that BBRv3 explicitly works to address (more on fairness generally in 3.7.4).

---

## 3.7.3 Network-Assisted Explicit Congestion Notification

Every approach covered so far is **end-to-end**: the sender infers congestion purely from observations it can make itself (packet loss, RTT, ACK arrival patterns) — the network never explicitly tells it anything. **Explicit Congestion Notification (ECN)** is the alternative: extensions to both IP and TCP (RFC 3168) that let the network _directly_ signal congestion to a TCP sender and receiver, rather than leaving the sender to guess.

### How the Signal Travels

At the network layer, **two bits** (with four possible value combinations) inside the IP datagram header's Type-of-Service field are used for ECN. One setting of these bits is used by a router to indicate that it is **experiencing congestion**, marking the IP datagram in flight. This marked datagram carries the congestion indication all the way to the destination host. (RFC 3168 doesn't specify _when_ exactly a router should consider itself congested — that threshold is a configuration choice made by the router's vendor and decided by the network operator. The general intuition is to set the bit to signal the _onset_ of congestion, ideally before any actual loss occurs.) A second ECN-bit setting is used by the _sending_ host to tell routers along the path that both the sender and receiver are ECN-capable, and therefore able to take action in response to ECN-indicated congestion.

![[Pasted image 20260627123917.png]]

### Closing the Loop: What Each Side Does

|Step|What Happens|
|---|---|
|**1. Router marks**|A congested router sets the ECN bits in the IP header of a datagram passing through it|
|**2. Receiving host informs sender**|When the TCP at the receiving host receives this ECN-marked datagram, it informs the TCP at the _sending_ host by setting the **ECE (Explicit Congestion Notification Echo) bit** in a receiver-to-sender TCP ACK segment|
|**3. Sender reacts**|On receiving an ACK carrying the congestion indication, the sender reacts exactly as it would react to a lost segment detected via fast retransmit — **halving its congestion window** — and sets the **CWR (Congestion Window Reduced) bit** in the header of its next outgoing segment, confirming it has indeed responded|

> Other transport-layer protocols beyond TCP make use of network-layer-signaled ECN too: the **Datagram Congestion Control Protocol (DCCP)** provides a low-overhead, congestion-controlled, UDP-like unreliable service that utilizes ECN; **DCTCP** (Data Center TCP) and **DCQCN** (Data Center Quantized Congestion Notification), both designed specifically for data-center networks, also rely on it. Recent Internet measurements show that **more than 80% of popular web servers**, and the paths from those servers to their clients, support ECN capabilities — a sign of just how mainstream this network-assisted approach has become.

---

## 3.7.4 Fairness

### What "Fair" Actually Means Here

Consider `K` TCP connections, each following a different end-to-end path, but all happening to pass through the same shared **bottleneck link** of transmission capacity `R` bps. (Recall: a bottleneck link is one where, for a given connection, every _other_ link along its path has abundant capacity by comparison.) Suppose each connection is transferring a large file, and no UDP traffic shares the bottleneck. A congestion-control mechanism is called **fair** if the average transmission rate of each of the `K` connections works out to approximately `R/K` — an equal slice of the shared link's capacity.

![[Pasted image 20260627124049.png]]

### A Worked Two-Connection Case

Consider two TCP connections sharing one bottleneck link of capacity `R`, with the same MSS and RTT (so equal congestion windows translate to equal throughput), both with plenty of data to send, and no other competing traffic. Plot the realized throughput of connection 2 against connection 1's throughput. If TCP shares the link equally, the realized throughputs should sit along the **45-degree "equal capacity share" line** emanating from the origin; ideally, the _sum_ of the two throughputs should also equal `R` (the **full-utilization line**) — the actual goal is for the achieved throughputs to land somewhere near where these two lines intersect.

![[Pasted image 20260627131219.png]]

**Walking the dynamics:** Suppose at some point in time the two connections' windows realize throughputs at point **A**. Because the _total_ link capacity jointly consumed is less than `R`, no loss occurs, and _both_ connections increase their windows by 1 MSS per RTT (congestion avoidance) — so the joint throughput climbs along a 45-degree line (equal increase for both) starting from A. Eventually the link's full capacity is exceeded and packet loss occurs — say, when throughputs reach point **B**. Both connections then halve their windows, landing at point **C** — exactly halfway along the line from B back toward the origin. Because joint capacity is now well under `R` again, both connections resume climbing along a new 45-degree line from C, eventually triggering loss again at point **D**, halving again, and so on.

> **The takeaway, made concrete:** Regardless of where the two connections start in this two-dimensional space, their realized throughputs will eventually **converge** to fluctuating along the equal-capacity-share line — an elegant, almost geometric explanation for why TCP's AIMD behavior tends to equalize bandwidth among competing connections sharing a bottleneck, even though neither connection is explicitly coordinating with the other.

### Where This Idealization Breaks Down

The scenario above assumed only TCP traffic traverses the bottleneck, that all connections share the _same_ RTT, and that each connection corresponds to exactly one host-destination pair. In practice, none of these conditions reliably hold, and real connections can end up with very unequal slices of capacity. In particular, when multiple connections share a bottleneck, **sessions with a smaller RTT** are able to grab available capacity more quickly — because they open their congestion windows faster, simply by virtue of completing more RTT cycles per unit time — and consequently enjoy higher throughput than connections with larger RTTs, all else equal.

### Fairness and UDP

TCP's congestion control regulates an application's transmission rate entirely through the congestion-window mechanism — but this regulation only applies to applications that actually _use_ TCP. Many real-time multimedia applications — Internet phone calls, video conferencing — deliberately **avoid** running over TCP for exactly this reason: they don't want their transmission rate throttled, even when the network is genuinely congested. Instead, these applications often run over **UDP**, which has no built-in congestion control whatsoever, and can therefore pump audio/video into the network at a constant rate, occasionally losing packets, rather than reducing rate to "fair" levels during congestion and dropping nothing.

> **From TCP's perspective, this is a real fairness problem.** TCP congestion control will decrease its sending rate in the face of increasing congestion, while UDP sources have no obligation to do the same — so it's entirely possible for UDP traffic to crowd out, and effectively starve, well-behaved TCP traffic sharing the same path. A number of congestion-control mechanisms have been proposed for the Internet specifically to prevent unchecked UDP traffic from bringing overall network throughput to a grinding halt.

### Fairness and Parallel TCP Connections

Even if UDP traffic could somehow be forced to behave fairly, the fairness problem still wouldn't be fully solved — because **nothing stops a TCP-based application from opening multiple parallel TCP connections at once.** Web browsers, for instance, routinely use several parallel connections to fetch the multiple objects embedded within a single web page (the exact number is configurable in most browsers).

**Worked example:** Consider a bottleneck link of rate `R` currently supporting nine ongoing applications, each using a single TCP connection. If a tenth application arrives and _also_ uses just one TCP connection, it gets approximately the same fair share, `R/10`. But if this new application instead opens **11 parallel TCP connections**, it ends up claiming more than `R/2` of the link's capacity — a deeply unfair allocation, achieved purely by gaming the _count_ of connections rather than sending data any faster per connection. Because parallel-connection usage is genuinely common on the modern Web, this isn't a hypothetical edge case — it's a real, persistent gap in how "fairly" capacity actually gets divided in practice.

> This wraps up the discussion of TCP congestion control — arguably no other single Internet protocol has drawn as much sustained research attention, both in the literature and in continued real-world deployment refinement, as this one.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Congestion control is entirely self-reported by the sender, based only on ACK behavior**|A receiver that lies — for example, by acknowledging segments faster or more generously than it actually received them (an "ACK-splitting" or "optimistic ACK" pattern) — can trick a well-behaved sender into growing its congestion window far faster than the network can actually support, congesting the path for everyone else|TCP's congestion control assumes a cooperative receiver; this is a known class of attack precisely because nothing in the protocol itself cryptographically verifies that an ACK reflects honest receipt|
|**UDP traffic has no congestion control and can crowd out TCP**|An attacker (or simply an uncooperative application) can flood a shared link with UDP traffic at a constant rate, forcing well-behaved TCP flows to back off again and again while the UDP traffic never does — a fairness violation that, taken to an extreme, functions as a denial-of-service vector against everyone else on the link|Network operators deploy traffic-shaping and rate-limiting mechanisms specifically to prevent unthrottled UDP traffic from starving TCP flows; never assume every flow on a shared link is behaving cooperatively|
|**Opening many parallel TCP connections claims a disproportionate share of bandwidth**|An application (or attacker) can intentionally open dozens of parallel connections to a single bottleneck to claim far more than its "fair" `R/K` share, at the direct expense of single-connection users sharing that link|This is a known, largely unsolved gap in TCP's fairness model; some networks impose per-source connection limits or fair-queuing disciplines at routers specifically to counteract it|

---

## Questions I Still Have

- [ ] CUBIC's window-growth function depends on tunable parameters that determine exactly how quickly `K` is approached — what determines reasonable default values for those parameters in real deployments, and how sensitive is CUBIC's fairness-vs-throughput tradeoff to getting them wrong?
- [ ] BBR explicitly tries to avoid the persistent queueing that loss-based algorithms (Reno/CUBIC) tolerate by design — does this mean a BBR flow competing against a CUBIC flow at the same bottleneck will systematically under-claim bandwidth, since CUBIC will happily keep filling the queue BBR is trying to avoid? Is this exactly the BBRv3 fairness concern mentioned, and how does BBRv3 actually address it mechanically?
- [ ] TCP Vegas detects congestion via rising RTT _before_ loss — but if it's sharing a bottleneck with loss-based flows (Reno/CUBIC) that don't back off until loss actually occurs, doesn't Vegas systematically lose out on bandwidth to its more "aggressive," loss-tolerant competitors? Is this why Vegas never became as widely deployed as Reno/CUBIC/BBR?
- [ ] For ECN, since the threshold for a router marking a packet as "congested" is left as a vendor/operator configuration choice rather than a hard spec value — how much does ECN's real-world effectiveness vary based on how conservatively or aggressively different network operators set that threshold?
- [ ] The fairness section shows RTT-based unfairness (lower-RTT connections grab more bandwidth) as an inherent property of AIMD — do any of the newer flavors (CUBIC, BBR) meaningfully reduce this particular unfairness, or does it persist across all of them since it's rooted in how often a connection gets to act per unit time, not in the specific increase/decrease rule used?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Congestion window (`cwnd`)**|Sender-side variable constraining how much unacknowledged data may be outstanding, based on perceived network congestion|
|**Loss event**|Either a timeout or the receipt of three duplicate ACKs — classic TCP's two signals that a segment has likely been lost|
|**Self-clocking**|TCP's property of using the arrival of ACKs themselves to trigger growth in `cwnd`, rather than relying on an independent clock|
|**Slow start**|The phase where `cwnd` starts at 1 MSS and doubles every RTT, ending at a loss event or when `cwnd` reaches `ssthresh`|
|**`ssthresh`**|"Slow-start threshold" — set to half of `cwnd` at the most recent loss event; marks where TCP switches from exponential to linear growth on the way back up|
|**Congestion avoidance**|The phase where `cwnd` grows linearly, by roughly one MSS per RTT, rather than doubling|
|**Fast recovery**|An optional phase following a triple-duplicate-ACK loss event, increasing `cwnd` by 1 MSS per further duplicate ACK rather than collapsing to slow start|
|**TCP Tahoe**|An early TCP version that always resets `cwnd` to 1 MSS on any loss event, regardless of type|
|**TCP Reno**|A later TCP version that halves (rather than collapses) `cwnd` specifically on triple-duplicate-ACK loss events|
|**AIMD**|Additive-Increase, Multiplicative-Decrease — the overall pattern of classic TCP's steady-state congestion control, producing a "sawtooth" `cwnd` graph|
|**TCP CUBIC**|A congestion-control flavor that grows `cwnd` as a cubic function of time-distance from `K` (the projected point of return to `W_max`), ramping quickly near a loss and cautiously near the last-known danger point|
|**`W_max`**|The congestion-window size at the most recent loss event, used by CUBIC as a target/reference point|
|**TCP Vegas**|A delay-based congestion-control flavor that proactively detects congestion via rising RTT, rather than waiting for actual packet loss|
|**`RTT_min`**|In TCP Vegas, the smallest RTT observed for the connection, representing the uncongested baseline|
|**BBR**|Bottleneck Bandwidth and RTT — a congestion-control protocol that estimates the bandwidth-delay product directly and tries to keep the bottleneck pipe "full, but no fuller"|
|**`n_inflight`**|In BBR, the number of currently in-flight (unacknowledged) packets — the quantity BBR directly manages|
|**Bandwidth-delay product**|An approximation of how much data a path can hold "in flight" without forming a queue, equal to the bottleneck's bandwidth multiplied by the path's RTT|
|**ECN (Explicit Congestion Notification)**|A network-assisted congestion-control extension where routers mark IP headers to signal congestion directly, rather than relying on the sender to infer it|
|**ECE bit**|TCP flag used by a receiver to echo an ECN congestion indication back to the sender|
|**CWR bit**|TCP flag used by a sender to confirm it has reduced its congestion window in response to an ECN signal|
|**Fairness**|A congestion-control mechanism is fair if, among `K` connections sharing a bottleneck of capacity `R`, each gets approximately `R/K`|
|**Bottleneck link**|The single link along a connection's path with the least relative spare capacity compared to the connection's other links|

---

## Related Concepts

---

→ Next: [[TRANSPORT LAYER FUNCTIONALITY]]