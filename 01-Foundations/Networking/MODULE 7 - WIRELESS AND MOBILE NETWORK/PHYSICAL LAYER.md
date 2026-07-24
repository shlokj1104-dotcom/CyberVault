---
title: PHYSICAL LAYER
date: 2026-07-24
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 7.2 The Physical Layer in Wireless Networks

> **One-Line Summary:** The **physical layer** of a wireless network converts digital bits into, and recovers them from, **electromagnetic (radio) waves** propagating through open space — a process governed by the wave's **amplitude, frequency, and phase**; degraded by **noise, path loss, and multipath**; capped by **Shannon's capacity theorem**; and made robust through **MIMO antennas**, **beamforming**, and **adaptive modulation** (e.g., **QPSK** and **QAM**) that trade bit rate for reliability as channel conditions fluctuate.

---

## Core Idea: Why the Physical Layer Finally Gets Its Due

Throughout Chapters 2 through 6, an understanding of just a few simple physical-layer characteristics — bit transmission rate, bit error rate, and propagation delay — was **sufficient** (adequate, nothing more required) to make sense of the material. Wireless networking is an aberration (a marked departure from the norm) in this regard: it is simply not possible to reason well about WiFi, cellular, or satellite networks without first getting genuinely comfortable with the physical layer's mechanics. Accordingly, this section (and the next) dive down into wireless physical-layer details, adopting a **computer-science-flavored, intuitive** treatment of the wireless channel rather than a rigorous, equation-heavy EE (electrical engineering) description.

The material builds up in two stages:

1. **Characteristics of wireless channels** (7.2.1) — the physical properties (amplitude, frequency, phase, power, bandwidth, noise, path loss, multipath) that make a radio channel behave so differently, and so much more capriciously (unpredictably, prone to sudden change), than a wired one.
2. **Coding and modulation: from bits to symbols to waveforms** (7.2.2) — the concrete pipeline by which the link layer's raw data bits are actually transformed into, and later recovered from, transmitted radio waveforms.

---

## 7.2.1 Characteristics of Wireless Channels

### Radio Waves: Amplitude, Frequency, and Phase

At the most rudimentary (basic, foundational) level, a time-varying electric current in a transmitting antenna creates **electromagnetic (radio) waves** that propagate through space and induce a corresponding current in one or more receiving antennas elsewhere. The transmitter encodes information by **modulating** — that is, systematically changing — characteristics of this wave. Four properties are foundational:

- **Amplitude:** the height of the wave at a given point in space and time.
- **Wavelength (λ):** the distance between two successive wave peaks.
- **Cycle time:** the time between two successive wave peaks.
- **Frequency:** the reciprocal of cycle time (1/cycle time), measured in Hertz (Hz) — a one-megahertz wave completes 1 × 10⁶ cycles per second.
- **Phase:** the fraction of a cycle covered up until time _t_, measured from a reference point (typically where the wave's amplitude is zero and increasing), expressed either as a fraction of a cycle or in degrees (0–360°). A phase 75 percent of the way through a cycle corresponds to 270 degrees.

```
Fig 7.2 -- EM Wave: Amplitude, Wavelength, Phase
────────────────────────────────────────
 Amplitude
   ^      ___         ___         ___
   |    /     \     /     \     /
   |  /         \ /         \ /
   +--------------------------------> t
   |              \         / \
   |                \_____/     \___
   |<--- lambda (wavelength) --->|
   Phase(deg):  0   90  180  270  360
```

![[Pasted image 20260724152421.png]]

Electromagnetic waves have a **direction** and propagate at approximately the speed of light, 3 × 10⁸ m/sec.

> **Analogy — Two Playground Swings:** Amplitude, frequency, and phase are perhaps most intuitively grasped by picturing two children on adjacent playground swings. **Amplitude** is how high each child swings — a taller arc means a stronger signal. **Frequency** is how many times per second each swing completes a full back-and-forth cycle — swing faster, and the frequency rises. **Phase** is simply whether the two children are swinging **in sync** or **offset** from one another — if one child reaches the highest point of their arc exactly when the other reaches the lowest, the two swings are 180 degrees out of phase, precisely as the receiver in Figure 7.12 later distinguishes a 0-bit curve from a 1-bit curve purely by this kind of timing offset.

---

### Radio Transmission: Power, Bandwidth

**Power** is the amount of energy per unit time that a transmitter puts into its emitted electromagnetic waves — informally, the signal's "strength." Power is measured in watts, typically milliwatts (mW) in wireless networks; electrical engineers often prefer the logarithmic **dBm** (decibel-milliwatts) scale instead, though this document, like the source chapter, sticks to the more intuitive linear mW measure.

An idealized wave concentrates all its radiated energy at a single frequency. In practice, however, a transmitter's power is spread across more than one frequency. The **power spectral density** function characterizes how much power is present in a signal as a function of frequency, and is typically centered around a so-called **carrier frequency** (1 GHz in the textbook's example).

```
Fig 7.3a -- Power Spectral Density
────────────────────────────────────────
 Power
   |         ▄▄▄
   |        █████
   |       ███████
   +-------------------------> Freq
        .9975  1.00  1.0025 (GHz)
             carrier (center) freq
             
Fig 7.3b -- Signal Bandwidth (22 MHz channel)
────────────────────────────────────────
 Power
   |      ┌─────────────┐
   |      │  22 MHz BW  │
   |      │             │
   +------┴─────────────┴----> Freq
        2.427   2.438   2.449 (GHz)
```

![[Pasted image 20260724152646.png]]

Armed with the concept of power spectral density, we can now precisely define a radio signal's **bandwidth** — the width (measured in Hz) of the range of frequencies occupied by the signal. A 22-MHz-wide channel, spread evenly between 2.427 and 2.429 GHz, corresponds to WiFi channel 6 (covered in Section 7.6).

**A crucial source of confusion, worth dwelling on:** this precise physical-layer notion of "bandwidth" (measured in Hertz, describing the width of a frequency band) is genuinely **different** from the far more informal, everyday networking usage of "bandwidth" to mean a link's bit transmission rate or capacity (measured in bits per second). The two are **related** — a wider radio channel generally supports a higher bit rate — but the relationship, as later sections make clear, is rather intricate (elaborate, not a simple one-to-one correspondence).

> **Analogy — A Highway's Width vs. Its Traffic Throughput:** Radio bandwidth (Hz) is analogous to how many **lanes wide** a highway is; link bit-rate (bps) is analogous to how many **cars per hour** actually flow past a given point. A wider highway (more Hz) generally permits more cars per hour, but the precise relationship also depends on how fast the cars travel and how tightly they're packed — exactly as a channel's bit rate depends not just on its Hz-bandwidth but on the modulation scheme layered on top of it (a theme this document returns to in Section 7.2.2).

---

### Sources of Noise

Radio signals, familiarly from everyday experience with AM/FM/satellite radio, can be clear or filled with "static" — that is, **noise**. There are several typical sources:

- **Interfering transmitters:** unlicensed frequency bands (e.g., the 2.4–2.5 GHz band) permit many industrial and consumer devices — baby monitors, garage door openers, WiFi, Bluetooth, Zigbee — to transmit in an uncoordinated manner; one's own network's signal is, from a neighboring network's perspective, indistinguishable from noise.
- **EM (electromagnetic) radiators:** electric motors and microwave ovens, particularly older and less-shielded models, can emit stray radio waves.
- **Thermal and electronic noise:** natural thermal variations and inherent imperfections in the transmitter's or receiver's own electronics introduce noise into the signal, entirely independent of any external interferer.

![[Pasted image 20260724152938.png]]

---

### Radio Channel: Noise, Signal-to-Noise (SNR) Ratio, and Capacity

The ratio of signal power to noise power is the **signal-to-noise ratio (SNR)**, typically measured in decibels (dB):

```
SNR(dB) = 10 · log10( received signal power / noise power )
```

An SNR value of 0 dB means signal and noise are present in **equal** amounts. For WiFi, a minimum acceptable SNR is around 20 dB; for LTE, the minimum acceptable value ranges from −5 to 18 dB depending on the modulation technique used.

With power, bandwidth, and noise now in hand, **Shannon's capacity theorem** — one of the most celebrated (widely admired) results in all of communication theory — relates communication capacity _C_ (the maximum possible error-free data rate, in bits per second) to a channel's bandwidth _B_ and its received signal and noise powers:

```
C = B · log2( 1 + received signal power / noise power )
```

Shannon's theorem gives only an **upper bound** on channel capacity — no matter how cleverly data is coded and modulated onto the wireless signal, the achieved rate can never exceed _C_ for a given bandwidth and SNR. Two salient (noteworthy) practical corollaries (natural, follow-on consequences) fall directly out of this formula:

- At a fixed SNR, capacity scales **linearly** with bandwidth — doubling the Hz doubles the theoretical ceiling on bit rate.
- Because capacity grows only **logarithmically** (sublinearly) with SNR, there are **diminishing returns** to boosting SNR once it is already high — an enormous, costly increase in transmit power buys only a comparatively modest gain in achievable rate.

> **Analogy — A Lecture Hall with a Noisy Air Conditioner:** Imagine straining to understand a lecturer while a nearby air conditioner hums in the background. Speaking a little louder (raising SNR) helps meaningfully at first — but once the lecturer is already shouting over the hum, speaking still louder yields only marginal, incremental improvements in intelligibility. Shannon's theorem formalizes precisely this everyday intuition: an ever-diminishing return on ever-greater signal power once the noise is already comfortably drowned out.

---

### Radio Channel Properties: Path Loss, Multipath, and Hidden Terminals

A radio signal changes in important ways as it propagates from transmitter to receiver. Most fundamentally, a wave **loses strength** as it propagates — a phenomenon called **path loss** or **attenuation** (weakening). For an unobstructed line-of-sight path, the ratio of transmitted to received power decreases according to an **inverse-square law**:

```
P_received / P_transmitted  ~  1 / (f · d)^2
```

where _f_ is the signal's frequency and _d_ is the transmitter–receiver distance. The intuition: since the surface area of a sphere grows as the **square** of its radius, the same fixed quantity of radiated energy is spread over an area four times larger at distance 2_d_, and nine times larger at distance 3_d_, than at distance _d_. A path-loss **exponent** of 2 describes free-space loss in open air; an exponent around 3 is typical of an outdoor urban environment, and an exponent of 4 or higher is common **within buildings** — making path loss an even more pernicious (harmful, insidious) concern indoors.

![[Pasted image 20260724153139.png]]

This **quadratic (squared)** relationship holds in both distance **and** frequency, with a direct, practical corollary: low-frequency (e.g., kHz) radio tends toward long-distance, low-data-rate applications such as radio navigation, whereas high frequencies (e.g., GHz) tend toward shorter-distance terrestrial applications such as cellular and WiFi.

**The hidden terminal problem.** The exponential decline of received power with distance gives rise to a genuinely thorny (difficult, prickly) scenario: nodes A and B might be within range of each other, and B and C might likewise be within range of each other, and yet A and C are **too far apart** — below the signal-detection threshold — to hear one another directly. A and C are said to be **hidden** from each other. If both A and C wish to transmit to B, they may each sense an apparently idle channel (since neither hears the other transmitting), yet their simultaneous transmissions will nonetheless **interfere** with one another precisely at B, corrupting what B receives.

![[Pasted image 20260724153340.png]]

A second, geometrically distinct variant of this same problem arises when A's and B's signals are **physically blocked** from each other — say, by an intervening mountain — even though both A and B can independently reach C; their transmissions to C still interfere with each other at C, notwithstanding that A and B never sense each other at all. Later, in Section 7.3.1, this physical-layer reality will be shown to directly motivate wireless link-layer multiple-access protocols such as WiFi's CSMA/CA.

> **Analogy — Two Campers Behind a Ridge:** Picture two campers, A and C, on opposite sides of a tall ridge, each able to shout loud enough to be heard by a third camper, B, standing between them on open ground — but too far, or too obstructed, to hear each other's shouts directly. If A and C both decide to shout to B at the same moment (each believing, reasonably, that the "channel is free" since neither hears the other), their voices will nonetheless **collide** and garble at B's ears — the essence of the hidden terminal problem.

**Multipath.** Beyond simple attenuation, part of a transmitted signal follows a direct **line-of-sight (LOS) path** to the receiver, while other parts of the very same signal **reflect** off intervening objects (buildings, terrain) and arrive at the receiver at slightly **later** points in time, having traveled a longer physical distance. These reflected, delayed copies are called **multipath** signals.

![[Pasted image 20260724153525.png]]

Because of multipath, the reception of a signal transmitted at one single instant is smeared (spread out) over time at the receiver — considerably complicating the receiver's job of reconstructing the originally transmitted signal. Multipath also constrains the maximum rate at which a transmitter can safely change its transmitted signal: the sender must space successive signal transitions far enough apart that the LOS pulse **and** all of its multipath reflections are received before the _next_ LOS pulse arrives — otherwise, successive pulses (and their reflections) blur together at the receiver. This receiver-dictated minimum spacing between signal changes is called the receiver's **coherence time**.

![[Pasted image 20260724153718.png]]

---

### Antennas: MIMO

The antennas deployed in most of today's wireless cellular and WiFi devices bear little resemblance to those of even a generation ago. Beginning in the early 2000s, wireless networks began migrating from single-antenna-at-each-end systems toward **multiple-input and multiple-output (MIMO)** antennas — multiple antennas at **both** the transmitter and the receiver. An immediate, first-order advantage of _N_ receiver antennas is that the total received signal carries roughly _N_ times the power — but MIMO's true benefit runs considerably deeper, exploiting the **multiple physical paths** between sending and receiving antennas in one of two fundamentally different ways:

- **MIMO spatial diversity** sends **redundant** (duplicated) streams of the _same_ information from a transmitter to a receiver, exploiting the fact that signals traveling different physical paths experience different degradations (e.g., different multipath fading). By combining several differently degraded copies of the same signal, MIMO systems maximize signal strength and minimize error rates — a technique essential for combating the adverse effects of multipath fading, even when antennas are separated by mere centimeters (as on a single cell phone).

```
Fig 7.9a -- MIMO Spatial Diversity
────────────────────────────────────────
              H1,1
        x ───────────┬──> y1 = H1,1(x)
         \
          \   H1,2
           └──────────> y2 = H1,2(x)
 (ONE signal x sent, arrives via TWO
  differently-faded paths/antennas)
```

![[Pasted image 20260724154104.png]]

- **MIMO spatial multiplexing**, by contrast, sends **different, independent** streams of information in parallel along the different paths between transmitter and receiver antennas. Each receiving antenna picks up a signal containing **both** transmitted streams, combined "in the air" — e.g., y₁ = H₁,₁(x₁) + H₂,₁(x₂). The receiver's task is then to disentangle (separate out) the original signals x₁ and x₂ from the received mixtures y₁ and y₂ — a signal-processing problem that, given two equations (one per receive antenna) and two unknowns (x₁ and x₂), is at least conceptually tractable, though the true channel functions H must be jointly estimated by sender and receiver in practice.

```
Fig 7.9b -- MIMO Spatial Multiplexing
────────────────────────────────────────
 x1 ──H1,1──┬──────> y1 = H1,1(x1)
     \      │             + H2,1(x2)
      \H1,2 │H2,1
       \    │
 x2 ────┴H2,2┴─────> y2 = H1,2(x1)
                          + H2,2(x2)
 (TWO different signals sent at once,
  mixed together at each receive ant.)
```

![[Pasted image 20260724154326.png]]

|Property|Spatial Diversity|Spatial Multiplexing|
|---|---|---|
|**What is sent**|Redundant copies of the **same** stream|**Different**, independent streams in parallel|
|**Primary goal**|Reliability — combat multipath fading|Throughput — increase effective data rate|
|**Underlying trick exploited**|Different paths fade differently|Distinct paths are (largely) separable/solvable equations|
|**Receiver's job**|Combine copies (e.g., select strongest, or coherently add)|Disentangle superimposed streams algebraically|

> **Analogy — Two Messengers vs. Two Chapters:** Spatial diversity is like sending the **same** important letter via two different couriers, each taking a different route — if one courier is delayed by traffic (fading), the other likely still arrives promptly, so the message reliably gets through. Spatial multiplexing, by contrast, is like splitting a book into two chapters and having two couriers deliver **different halves simultaneously** — nothing is duplicated, so the total delivered content doubles, but now _both_ couriers must successfully arrive for the whole book to be reconstructed.

---

### Beamforming and Directional Antennas

**Beamforming** involves directing a transmitted signal toward a specific user or area, instead of radiating it in all directions indiscriminately (without careful distinction). It is typically implemented using an **array** of antennas: by adjusting the phase and amplitude of the signal emitted at each individual antenna, the array creates **constructive interference** in the desired direction and **destructive interference** elsewhere, forming a directional "beam" of radio energy. Beamforming can be dynamic, continuously adjusting the beam's direction to track a moving user. Its benefits include:

1. **Increased signal strength** — focusing energy in a specific direction raises the signal received by the intended user, enabling higher data rates and more reliable connections.
2. **Reduced interference** — less signal energy leaks into undesired directions, improving performance in dense environments with many competing devices.
3. **Improved coverage** — directional beams can extend the effective coverage area, improving connectivity at the fringes (outer edges) of a cell's normal range.
4. **Enhanced capacity** — by using the available spectrum more efficiently, beamforming supports more simultaneous users.

Beamforming is used widely in WiFi, modern cellular networks, and satellite communications.

> **Analogy — A Flashlight vs. a Bare Light Bulb:** An omnidirectional antenna is like a bare, unshaded light bulb — illumination is scattered in every direction, much of it wasted on empty space no one occupies. Beamforming is like swapping that bulb for a flashlight whose beam can be aimed precisely at a reader's book — the same amount of "energy" (power) is now concentrated exactly where it is genuinely needed, brightening that one spot considerably more than the diffuse bulb ever could.

---

### Multi-user MIMO (MU-MIMO)

The MIMO discussion above implicitly focused on **single-user MIMO (SU-MIMO)** — multiple antennas at transmitter and/or receiver serving a **single** device (typically 2 to 4 antennas in a WiFi base station). **Multi-user MIMO (MU-MIMO)**, sometimes called **Massive MIMO**, instead uses different subsets of a large antenna array to transmit to **multiple different devices simultaneously**. Each subset employs MIMO spatial diversity to reliably deliver its signal to one device, and MU-MIMO commonly layers **beamforming** on top, steering an independent beam toward each recipient at the same time. A modern MU-MIMO antenna for a 5G base station can contain an array of up to **64 beamforming transmitter elements**, with a comparable number of receiving elements.

![[Pasted image 20260724154545.png]]

|Feature|SU-MIMO|MU-MIMO / Massive MIMO|
|---|---|---|
|**Recipients per transmission**|One device|Many devices concurrently|
|**Typical antenna count**|2–4 antennas|Up to ~64 antenna elements|
|**Beamforming typically used?**|Optional|Commonly, yes|
|**Primary win**|Reliability/throughput to one user|Aggregate network capacity across users|

---

### Radio Spectrum

The **radio spectrum** is the range of the electromagnetic spectrum from 3 Hz to 3 THz. It is widely regarded as a **national asset**, and its use is regulated by individual countries, with loose coordination (but no binding, enforceable global regulation) provided by the International Telecommunication Union (ITU), a United Nations agency. There are two broad classes of spectrum-use types:

- **Licensed spectrum.** A government-issued license must be obtained to operate a transmitter in a licensed band — this includes the bands used by commercial cellular network providers. In many countries, such spectrum is allocated via **auctions** that have generated tens to hundreds of billions of dollars in licensing fees. Because commercial operators must recoup (recover) these steep licensing, tower, and land-access costs, cellular service pricing reflects this considerable overhead — unlike, say, WiFi providers operating in unlicensed spectrum.
- **Unlicensed spectrum.** Portions of the spectrum are available for use without obtaining any license at all — institutions and individuals may purchase and use devices (garage door openers, baby monitors, microwave ovens, and, notably, WiFi networks) here, subject only to constraints such as transmission power limits. Because many uncoordinated devices can transmit in unlicensed spectrum, they can — and routinely do — interfere with one another.

| Spectrum Band                             | Network Type                                          | Licensed / Unlicensed       |
| ----------------------------------------- | ----------------------------------------------------- | --------------------------- |
| 2.4 GHz (2.40–2.49 GHz)                   | WiFi, Bluetooth, Zigbee                               | Unlicensed                  |
| 5 GHz (5.17–5.83 GHz)                     | WiFi                                                  | Unlicensed                  |
| 6 GHz (5.9–7.12 GHz)                      | WiFi                                                  | Unlicensed                  |
| < 1 GHz ("low-band" cellular)             | 2G/3G/4G/5G — long range, lower rate (50–250 Mbps)    | Licensed                    |
| 1.8, 3.3–3.8, 6 GHz ("mid-band" cellular) | 2G/3G/4G/5G — the "sweet spot" (~5 mi, 100–900 Mbps)  | Licensed                    |
| 3.5 GHz CBRS                              | Cellular / "Private 5G"                               | Licensed **and** Unlicensed |
| 26, 40, 50, 66 GHz ("mmWave", high-band)  | 5G — short range (< 1 mi), very high speed (< 3 Gbps) | Licensed                    |
| 915 MHz (902–928 MHz)                     | LoRa (IoT)                                            | Unlicensed                  |


New spectrum-sharing methods are also being developed to let non-incumbent devices (those not associated with a spectrum's licensee) opportunistically use spectrum the licensee isn't currently using — the CBRS band, for instance, permits cellular base stations to transmit provided no naval radar (the licensed incumbent) is actively present.

> **Analogy — Private Property vs. a Public Park:** Licensed spectrum is akin to owning a plot of private land with a recorded deed, purchased at auction — exclusive, expensive, and legally defensible against trespassers. Unlicensed spectrum is more like a public park: free to enter and use, but shared with everyone else who shows up, with no guarantee that some other visitor's boombox (a competing device) won't drown out your own picnic conversation.

---

## 7.2.2 Coding and Modulation: From Bits to Symbols to Waveforms

Having covered the characteristics of the wireless channel, this subsection moves "up" the stack, while remaining squarely within the physical layer, to discuss how a wireless transmitter groups data bits into **symbols**, encodes those symbols into transmitted **waveforms**, and how a receiver reverses this process at the other end.

```
Fig 7.11 -- Physical-Layer Transmission Pipeline
────────────────────────────────────────
 Data   Coding    Modulation    RF
 bits ->(bits)-> (symbols, -> transmit
                  waveform)      │
                                 ▼
                          Noisy channel
                                 │
 Data <- Decoding <- Demodulation<- RF
 bits                             recv
 (dashed arrows: channel-quality feed-
  back flows from receiver to sender)
```

![[Pasted image 20260724154801.png]]

Data bits are presented to the physical layer at the transmitter, processed into electromagnetic waveforms that propagate over the (noisy) channel to the receiver, and are there transformed back into bits, which are then passed up to the link layer. Ideally, this entire pipeline proceeds as swiftly (quickly) as possible.

### Physical-Layer Coding

In the first pipeline step ("coding"), the original data bits from the link layer are **expanded** — in the simplest case, redundant copies of the original bits are added to the bitstream, protecting against the corruption of some of the transmitted bits by channel noise. It is worth emphasizing that bits added here, at the physical layer, are entirely **in addition to** any checksum or CRC bits already appended at the link layer (Chapter 6) — a link-layer frame's checksum/CRC bits are simply treated, at this stage, exactly like any other bits in the frame, and so may themselves be further protected by these added physical-layer bits.

The transmitted bits may also be **reordered** — spreading redundant copies of a given bit widely across the transmitted bit sequence — so that a single **burst** of noise doesn't corrupt every copy of the same bit at once. For example, eight original data bits d₁ d₂ d₃ d₄ d₅ d₆ d₇ d₈ might first be expanded (where dᵢ′ denotes a copy of the i-th bit) to sixteen bits, then **shuffled**:

```
d1 d5' d2 d6' d3 d7' d4 d8' d5 d1' d6 d2' d7 d3' d8 d4'
```

If three consecutive received bits are lost to a burst of interference (denoted x), the receiver — after **unshuffling** the bits back into their original order — still finds enough surviving redundant copies to reconstruct all eight original bits, even in the presence of a genuine burst error.

> **Analogy — An Accordion-Folded Letter:** Imagine writing an important letter twice for redundancy, then folding the two copies **accordion-style**, interleaving their sentences line by line, before mailing the single folded packet. If a careless courier spills coffee across one contiguous section of the folded packet (a burst error), the stain destroys only scattered, alternating lines — never both copies of the _same_ sentence at once — leaving enough intact lines, once the letter is unfolded, to reconstruct the entire original message.

---

### Modulation: Amplitude, Frequency, and Phase

**Modulation** lies at the heart of the wireless transmitter's physical layer: it takes coded bits and creates waveforms that encode them onto a **carrier signal**. There are three common approaches, each modulating one of the fundamental wave characteristics introduced earlier in 7.2.1:

- **Amplitude Modulation (ASK — amplitude shift keying):** bit values are encoded by setting the carrier's **amplitude** to one of several discrete (distinct, separate) values during each time interval — e.g., a 0-bit is sent at low amplitude, a 1-bit at high amplitude.
- **Frequency Modulation (FSK — frequency shift keying):** bit values are encoded by changing the carrier's **frequency** across different time intervals — e.g., one frequency represents a 0-bit, a different frequency a 1-bit.
- **Phase Modulation (PSK — phase shift keying):** bit values are encoded by changing the carrier's **phase** across different time intervals — e.g., a signal that is zero-and-increasing at the start of an interval encodes a 0-bit, while one that is zero-and-decreasing (180 degrees out of phase) encodes a 1-bit.

![[Pasted image 20260724155006.png]]

In each scheme, the receiver need only measure the amplitude, frequency, or phase of the incoming analog radio signal to correctly infer whether a 0-bit or 1-bit was sent — though reliably detecting frequency and phase shifts does require that sender and receiver clocks be carefully synchronized with one another.

|Scheme|Property Varied|Simplicity|Noise Sensitivity|
|---|---|---|---|
|**ASK**|Amplitude|Simplest to implement|Most vulnerable — noise directly perturbs amplitude|
|**FSK**|Frequency|Moderate|More robust than ASK to amplitude-based noise|
|**PSK**|Phase|Requires precise clock sync|Generally the most noise-resilient of the three|

> **Analogy — A Lighthouse Signaling Ships:** Picture a lighthouse trying to send a coded message to a distant ship using only its beam. It could vary how **bright** the beam shines (ASK), how **fast** it blinks (FSK), or exactly **when** within a fixed blinking rhythm it flashes (PSK) — three entirely different physical "knobs," any one of which is sufficient, on its own, to carry the same underlying binary message.

---

### Quadrature Phase Shift Keying (QPSK)

The examples above show only **two** values of amplitude, frequency, or phase encoding a **single** bit at a time. But why stop at two values, and why not modulate multiple properties at once to pack in still more information?

**Quadrature Phase Shift Keying (QPSK)** groups the serial bitstream **two bits at a time** into a single **symbol**, using **four** distinct phase-shift values (45°, 135°, 225°, 315°) to represent the four possible two-bit combinations: "00", "01", "10", "11". All four QPSK signals share the same amplitude — only their phase differs.

![[Pasted image 20260724155223.png]]

A crucial consequence: because each transmitted symbol now carries **two** bits at once, the rate at which symbols must be processed is only **half** the underlying bit rate — leaving correspondingly more time, per symbol, for signal processing than pure bit-by-bit signaling would allow.

The four QPSK symbols are often depicted on a **constellation diagram** — a plot where a point's angular position (measured counterclockwise from due east, i.e., the 3:00 position) represents the symbol's phase, and its distance from the origin represents amplitude.

![[Pasted image 20260724155507.png]]

> **Analogy — A Clock Face as a Mailbox Grid:** Imagine a clock face with exactly four mail slots positioned at the 1:30, 4:30, 7:30, and 10:30 marks — every letter dropped in must land in exactly one of these four slots, and which slot a letter lands in instantly conveys **two bits' worth** of information (since there are four possible slots) rather than the mere one bit conveyed by a simple "slot present / no slot" binary system.

---

### Quadrature Amplitude Modulation (QAM)

In QPSK, only the signal's **phase** is modulated, while its amplitude stays fixed. **Quadrature Amplitude Modulation (QAM)** modulates **both** phase **and** amplitude simultaneously, allowing many more distinct symbols to be packed onto a single constellation diagram. The number preceding "-QAM" names how many distinct symbols that version of QAM uses: 4-QAM is essentially identical to QPSK (4 symbols, 2 bits/symbol); 16-QAM uses 16 symbols (4 bits/symbol); 64-QAM uses 64 symbols (6 bits/symbol).

![[Pasted image 20260724155755.png]]

16-QAM, 64-QAM, and 256-QAM are all used in 4G/5G and WiFi networks today, with even higher-order QAMs (1024-QAM, 4096-QAM) appearing in the WiFi-6 and WiFi-7 standards.

A constellation diagram also offers direct insight into how noise produces symbol errors. Suppose a sender transmits the "1111" symbol of a 16-QAM constellation. Changing electromagnetic conditions and measurement noise at the receiver mean that the **received** amplitude and phase for that same "1111" symbol will differ slightly **each time** it is sent — scattering a cloud of possible received measurements around the true transmitted point. Most of these land closer to "1111" than to any other symbol, so the receiver correctly infers "1111" was sent — but occasionally a received measurement lands closer to a **neighboring** symbol (e.g., "1011"), and the receiver incorrectly infers the wrong symbol was transmitted, resulting in a **symbol error**.

![[Pasted image 20260724155944.png]]

|QAM Order|Bits per Symbol|Constellation Density|Noise Sensitivity|
|---|---|---|---|
|**4-QAM (≈ QPSK)**|2|Sparse (4 points)|Most robust — most tolerant of a noisy channel|
|**16-QAM**|4|Moderate (16 points)|Middling|
|**64-QAM**|6|Dense (64 points)|Least robust — requires a comparatively clean channel|

> **Analogy — A Crowded vs. a Spacious Parking Lot:** A sparse 4-QAM constellation is like a parking lot with only four widely spaced spots — even a driver who parks somewhat carelessly (noise) can't easily mistake which spot they're in. A dense 64-QAM constellation is like the same lot painted with 64 tightly packed spots — a driver's honest parking error (noise) now has a real chance of landing them in the **wrong** spot altogether, which is precisely why higher-order QAM is more error-prone under identical noise conditions.

---

### Adaptive Modulation

Figures 7.15 and 7.16 together suggest an important, somewhat counterintuitive (contrary to naive expectation) tension: as a constellation diagram grows more densely packed at higher-order QAMs, the effects of noise and receiver measurement error become more **pronounced** (noticeable), resulting in higher symbol-error rates and hence higher **bit error rates (BER)**. While higher transmission rates might always **seem** desirable, blindly reaching for a higher-order QAM is not always wise if the resulting error rate proves too high.

![[Pasted image 20260724160146.png]]

Suppose the goal is to keep BER below 10⁻⁴ while achieving the highest bit rate possible subject to that constraint. Figure 7.17 indicates a natural, layered decision rule:

|SNR (dB)|Modulation Scheme to Use|Rationale|
|---|---|---|
|**< 13**|4-QAM|Channel too noisy for denser constellations to stay under the BER target|
|**13 – 17**|16-QAM|Cleaner channel supports a higher bit rate at an acceptable BER|
|**> 17**|64-QAM|Cleanest channel; highest bit rate achievable within the BER budget|

As the SNR varies over time — due to a device moving, obstacles appearing, or interference changing — the modulation technique used by the transmitter and receiver can be changed **dynamically** to always realize the highest throughput possible at that moment, subject to the BER constraint. This, of course, requires the sender and receiver to continually **coordinate**: measuring and reporting BER and SNR to one another, and negotiating changes in modulation technique — precisely the mechanism used in 4G/5G wireless networks.

> **Analogy — Shifting Gears on a Bicycle:** Adaptive modulation is much like a cyclist shifting gears in response to terrain. On a steep, bumpy hill (a noisy, low-SNR channel), the cyclist drops into a **low gear** — slower, but far more reliable and less likely to stall (analogous to robust 4-QAM). On a smooth, flat road (a clean, high-SNR channel), the same cyclist shifts into a **high gear** — considerably faster, though it demands more precise, error-free pedaling (analogous to dense 64-QAM). A skilled cyclist doesn't fix a single gear for the whole ride; they continually adapt it to the terrain actually underfoot, exactly as an adaptive-modulation radio continually adapts to its actual, currently measured channel conditions.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**The wireless medium is inherently broadcast**|Anyone with a receiver within range can passively eavesdrop on transmitted radio waves, since — unlike a wired link — there is no physical cable to tap into; interception requires no physical access at all|Encryption at higher layers (e.g., WPA at the link layer, TLS at the transport layer) is essential, since the physical layer itself offers no inherent confidentiality|
|**Known modulation schemes make jamming tractable (feasible)**|An attacker who knows a channel's frequency and modulation scheme can transmit deliberate noise or interfering signals precisely engineered to drive the effective SNR below the threshold needed for reliable demodulation, causing a targeted denial of service|Spread-spectrum techniques, frequency hopping, and simply operating across multiple bands/channels can make deliberate jamming markedly more costly and less effective for an attacker|
|**Unlicensed spectrum has essentially no access control**|Because unlicensed bands (2.4/5/6 GHz) require no license and only modest power limits, an attacker can transmit interfering signals in those bands largely without fear of legal or technical barriers|Licensed spectrum, by contrast, offers at least a legal (if not physical) deterrent against unauthorized transmission — a genuine trade-off institutions weigh when choosing WiFi versus private cellular (e.g., CBRS) deployments|
|**Beamforming incidentally narrows the eavesdropping "surface"**|An attacker positioned outside a beamformed signal's narrow directional path receives a substantially weaker signal than one positioned within it, somewhat raising the bar for casual eavesdropping|Beamforming should nonetheless never be treated as a security mechanism in its own right — it is a physical-layer optimization for throughput and coverage, not a substitute for genuine cryptographic protection|
|**Adaptive modulation's feedback channel is itself a potential attack surface**|Because sender and receiver must exchange BER/SNR measurements to coordinate modulation changes, a man-in-the-middle attacker able to forge this feedback could potentially force a victim into an unnecessarily low-order, low-throughput modulation scheme — a wireless analog of a protocol "downgrade attack"|Authenticating the control/feedback channel used for adaptive-modulation coordination (rather than trusting it implicitly) helps guard against such forced-downgrade manipulation|

---

## Questions I Still Have

- [ ] Given that Shannon's capacity theorem provides only an _upper bound_, how close do real-world 5G and WiFi-6/7 coding-and-modulation pipelines typically come to that theoretical ceiling in practice — is the gap a matter of a few percent, or something far larger?
- [ ] The text notes the path-loss exponent varies considerably by environment (~2 free space, ~3 outdoor urban, 4+ indoors) — how is this exponent actually measured or estimated for a specific real-world deployment, and how often would it need re-measurement as an environment's physical layout changes (new furniture, foot traffic, seasonal foliage)?
- [ ] MU-MIMO base stations can field up to 64 beamforming antenna elements — how does the system decide how many independent, simultaneous spatial streams it can actually support at once, and what happens when more devices request service than there are available independent streams?
- [ ] Adaptive modulation depends on sender and receiver continuously exchanging BER/SNR feedback — how frequently does this renegotiation actually happen in a live system, and what's the bandwidth/latency overhead of that coordination signaling itself?
- [ ] Since unlicensed spectrum is shared and fundamentally uncoordinated, what mechanisms — beyond the link-layer multiple-access protocols covered later in the chapter — do real WiFi deployments use to stay usable in the increasingly crowded 2.4/5/6 GHz bands, especially alongside Bluetooth and Zigbee traffic?
- [ ] The hidden terminal problem is introduced here purely as a _physical-layer_ geometry issue (signal strength falling below a detection threshold) — how precisely does a link-layer protocol like 802.11's CSMA/CA (Section 7.3.1) actually resolve a problem whose root cause lies entirely outside the link layer's own visibility?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Amplitude**|The height of an electromagnetic wave at a given point in space and time|
|**Wavelength (λ)**|The distance between two successive peaks of a wave|
|**Cycle time**|The time between two successive peaks of a wave|
|**Frequency**|The reciprocal of cycle time (1/cycle time), measured in Hertz (Hz)|
|**Phase**|The fraction of a wave's cycle covered up to a given time, expressed in degrees (0–360°)|
|**Power**|Energy per unit time emitted by a transmitter, informally the signal's "strength," measured in mW|
|**Power spectral density**|A function characterizing how a transmitter's power is distributed across frequencies|
|**Carrier frequency**|The center frequency around which a signal's power spectral density is concentrated|
|**Bandwidth (radio, physical-layer sense)**|The width, in Hz, of the range of frequencies occupied by a signal — distinct from a link's bit-rate ("bandwidth" in the everyday networking sense)|
|**Noise**|Unwanted disturbance corrupting a transmitted signal, from interfering transmitters, EM radiators, or thermal/electronic sources|
|**Signal-to-noise ratio (SNR)**|The ratio of signal power to noise power, typically measured in decibels (dB)|
|**Shannon's capacity theorem**|A formula giving the maximum error-free data rate (capacity) a channel can support, as a function of its bandwidth and SNR|
|**Path loss / attenuation**|The weakening of a radio signal's strength as it propagates from transmitter to receiver|
|**Path-loss exponent**|The power to which distance is raised in a path-loss model (≈2 free space, ≈3 outdoor urban, 4+ indoors)|
|**Hidden terminal problem**|A scenario where two nodes, each in range of a common third node, cannot hear each other and so may transmit simultaneously, causing interference at the shared node|
|**Multipath**|The phenomenon whereby reflected copies of a transmitted signal arrive at the receiver via longer paths and later times than the direct line-of-sight signal|
|**Coherence time**|The minimum time a receiver needs between signal changes so that a line-of-sight signal and its multipath reflections don't blur together|
|**MIMO (multiple-input, multiple-output)**|The use of multiple antennas at both transmitter and receiver to exploit multiple physical signal paths|
|**Spatial diversity**|A MIMO technique sending redundant copies of the same signal over multiple paths, to improve reliability|
|**Spatial multiplexing**|A MIMO technique sending different, independent signal streams in parallel over multiple paths, to improve throughput|
|**Beamforming**|Directing a transmitted signal toward a specific direction using a phased antenna array, rather than radiating omnidirectionally|
|**MU-MIMO / Massive MIMO**|The use of a large antenna array's subsets to transmit to multiple different devices simultaneously, often combined with beamforming|
|**Radio spectrum**|The portion of the electromagnetic spectrum from 3 Hz to 3 THz used for wireless communication|
|**Licensed spectrum**|Spectrum requiring a government-issued license (often obtained via auction) to transmit in|
|**Unlicensed spectrum**|Spectrum available for use without a license, subject to power and usage constraints, and shared by uncoordinated devices|
|**Modulation**|The process of encoding coded bits onto a carrier signal by varying its amplitude, frequency, or phase|
|**ASK (amplitude shift keying)**|Modulation encoding bit values as discrete carrier-amplitude levels|
|**FSK (frequency shift keying)**|Modulation encoding bit values as discrete carrier-frequency values|
|**PSK (phase shift keying)**|Modulation encoding bit values as discrete carrier-phase values|
|**QPSK (quadrature phase shift keying)**|A PSK scheme using four phase values to encode 2-bit symbols|
|**Constellation diagram**|A diagram plotting modulation symbols by phase (angle) and amplitude (distance from origin)|
|**QAM (quadrature amplitude modulation)**|Modulation encoding symbols using both phase and amplitude, allowing many symbols (e.g., 16-QAM, 64-QAM)|
|**Bit error rate (BER)**|The fraction of received bits that are in error, generally rising with higher-order modulation and falling with higher SNR|
|**Adaptive modulation**|Dynamically changing the modulation scheme in use to match current channel (SNR) conditions, maximizing throughput subject to a BER constraint|
|**Physical-layer coding**|The addition and reordering (e.g., interleaving) of redundant bits at the physical layer, to protect against channel noise and burst errors|

---

## Related Concepts

---

→ Next: [[WIRELESS ACCESS NETWORK]]