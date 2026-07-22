---
title: ERROR DETECTION & CORRECTION TECHNIQUE
date: 2026-07-18
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 6.2 Error-Detection and -Correction Techniques

> **One-Line Summary:** Because bit-level corruption is an unavoidable physical reality on any link, the link layer augments data **D** with **error-detection and -correction bits (EDC)** before transmission, using one of three increasingly sophisticated techniques — **parity checks** (simple, low overhead, weak protection), **checksumming** (used at the transport layer, moderate protection, software-friendly), and **cyclic redundancy checks (CRC)** (used at the link layer, strong protection, hardware-friendly) — each trading off computational/overhead cost against the probability of an error slipping through **undetected**.

---

## Core Idea: You Can't Prevent Bit Errors, So You Detect (or Correct) Them

Section 6.1 established that bit-level error detection and correction — catching and fixing the corruption of bits in a link-layer frame as it travels between two physically connected neighboring nodes — is a service often provided at the link layer. (Chapter 3 showed this same fundamental idea also shows up at the transport layer.) A full treatment of the underlying theory is itself the subject of entire textbooks; the goal here is narrower and more practical: build an **intuitive feel** for what error-detection and -correction techniques can actually do, and walk through a few simple techniques that are genuinely used in practice.

### The General Setup

Figure 6.3 illustrates the setting for this whole section. At the **sending node**, data **D** — the information to be protected against bit errors — is **augmented** with error-detection and -correction bits, **EDC**. Critically, **D** is not just the datagram passed down from the network layer; it typically also includes link-level addressing information, sequence numbers, and other fields from the link-frame header. Both **D** and **EDC** are sent together to the receiving node in a single link-level frame.

At the **receiving node**, a sequence of bits is received: **D′** and **EDC′**. Because of in-transit bit flips, **D′** and **EDC′** may **differ** from the original **D** and **EDC** that were sent.

```
Fig -- EDC Setup (Sender-to-Receiver)
────────────────────────────────────────────
 SENDER                      RECEIVER
 Datagram                    Datagram
    │                            ▲
    ▼                            │
 ┌─────┬─────┐   bit-error   ┌─────┬─────┐
 │  D  │ EDC │──── prone ───▶│ D'  │EDC' │
 └─────┴─────┘     link      └─────┴─────┘
 d data bits                  received bits
                              (may differ
                               from D,EDC)

 Receiver check: are all bits in D' OK?
   YES -> no error detected
   NO  -> error detected
```

![[Pasted image 20260722203658.png]]

> **Analogy — A Sealed, Tamper-Evident Package:** Think of **D** as the actual contents of a package, and **EDC** as a special **tamper-evident seal** applied at the factory based on the package's exact weight and contents. When the package arrives, the receiver doesn't need to know exactly _what's_ inside to check whether it was tampered with — they just re-verify the seal against what arrived. If the seal doesn't match, _something_ changed in transit. Crucially — and this is the subtle part — a clever enough tamperer could conceivably alter the contents in a way that still produces a seal that looks valid (this is exactly the **undetected error** problem discussed below).

### The Receiver's Real Challenge (and a Subtle Wording Trap)

The receiver's job is to determine whether or not **D′** is the same as the original **D**, given that it has only received **D′** and **EDC′** — it never gets to see the original **D** directly for comparison. The **exact wording** of this decision matters a great deal: the receiver asks whether **an error is detected**, _not_ whether **an error has occurred**. These are not the same question!

- **Error-detection and -correction techniques allow the receiver to** _sometimes_, **but not always**, **detect that bit errors have occurred.** Even with error-detection bits in use, the received information can still contain **undetected bit errors** — meaning the receiver may be entirely unaware that the datagram it delivers up to the network layer is corrupted, or unaware that a field in the frame's header has been corrupted.
- The practical design goal, therefore, is simply to choose an error-detection scheme that keeps the **probability** of such undetected occurrences **small** — not to eliminate that probability entirely, which is generally infeasible.
- As a general rule: **more sophisticated error-detection and -correction techniques** (i.e., ones with a smaller probability of allowing undetected bit errors) **incur a larger overhead** — more computation is needed, and a larger number of error-detection and -correction bits must be computed and transmitted.

This section examines three techniques for detecting errors in transmitted data, in increasing order of sophistication: **parity checks** (to illustrate the basic ideas), **checksumming methods** (more typically used at the transport layer), and **cyclic redundancy checks** (more typically used at the link layer, in an adapter).

---

## 6.2.1 Parity Checks

### The Single (One-Bit) Parity Scheme

The simplest possible form of error detection is the use of a **single parity bit**. Suppose the information to be sent, **D**, has **d** bits.

- In an **even parity scheme**, the sender includes **one additional bit** and chooses its value such that the **total number of 1s** in the resulting **d + 1** bits (original information plus the parity bit) is **even**.
- For **odd parity schemes**, the parity bit value is chosen such that there is an **odd** number of 1s.

```
Fig -- Single (Even) Parity Bit
────────────────────────────────────────
   d data bits          Parity bit
 ┌───────────────────┐ ┌───┐
 │ 0111001101010111  │ │ 1 │
 └───────────────────┘ └───┘
   count of 1s in d bits = 11 (odd)
   parity bit = 1 -> total 1s = 12 (even)

 Receiver: count 1s in received d+1 bits.
   Odd count  -> error detected
   Even count -> assumed no error
                 (but CAN miss even
                  numbers of bit flips!)
```


**Receiver operation** with a single parity bit is equally simple: the receiver need only count the number of 1s in the received **d + 1** bits. If an **odd** number of 1-valued bits is found under an even-parity scheme, the receiver knows that **at least one** bit error has occurred. More precisely, it knows that some **odd** number of bit errors has occurred.

### The Critical Weakness: What About an Even Number of Errors?

What happens if an **even** number of bit errors occurs? You should convince yourself this would result in an **undetected error** — the parity check would come back looking perfectly valid, even though the data was corrupted!

- **If** bit errors are rare and can be assumed to occur **independently** from one bit to the next, the probability of multiple bit errors in a single packet would be extremely small, and a single parity bit might suffice.
- **However**, measurements have shown that rather than occurring independently, errors are often **clustered together in "bursts."** Under burst-error conditions, the probability of undetected errors in a frame protected by single-bit parity can approach **50 percent** [Spragins 1991]. This is a serious real-world weakness, motivating the need for a more robust scheme.

> **Analogy — Counting Raised Hands at a Crowded Meeting:** Single-bit parity is like a meeting organizer who only checks _whether an odd or even number of people raised their hands_ — never _who_ specifically raised a hand. If exactly one person's hand-raise status flips (one attendee who should have raised their hand didn't, or vice versa), the odd/even count changes and the organizer notices something's off. But if **two** people's hand-raise statuses flip simultaneously (say, due to a distracting noise — a "burst" of confusion), the total count can come out looking exactly the same as if nothing happened at all — the mix-up goes completely unnoticed.

### Two-Dimensional Parity: A More Robust Generalization

Figure 6.5 shows a **two-dimensional generalization** of the single-bit parity scheme, which provides insight into true error-**correction** techniques (not just detection). Here, the **d** bits of **D** are divided into **i** rows and **j** columns. A parity value is computed for **each row** and for **each column**. The resulting **i + j + 1** parity bits comprise the link-layer frame's error-detection bits.

```
Fig -- Two-Dimensional Parity
────────────────────────────────────────
             Row parity ─────▶
        d1,1  d1,2 ... d1,j | d1,j+1
        d2,1  d2,2 ... d2,j | d2,j+1
   Col     :    :        :  |   :
  parity di,1  di,2 ... di,j| di,j+1
   │   d(i+1),1 ...   ...   | d(i+1),j+1
   ▼

 i rows x j cols of data + i+j+1 parity bits

 Single-bit error -> its ROW parity AND
 its COLUMN parity both flag error, so
 receiver can locate AND correct it.
```

**Why this works for correction, not just detection:** Suppose a single bit error occurs somewhere in the original **d** bits of information. With this two-dimensional parity scheme, the parity of **both** the column **and** the row containing the flipped bit will be in error. The receiver can therefore not only **detect** that a single-bit error has occurred, but can use the **column and row indices** with parity errors to actually **identify the exact bit** that was corrupted — and **correct** that error! This is a genuine, if limited, example of the receiver fixing an error on its own, without needing a retransmission.

- Although the discussion here has focused on a single bit of information, the parity bits themselves are also detectable and correctable.
- **Any combination of two errors** in a packet can also be **detected** (but _not_ corrected) by two-dimensional parity.

> **Analogy — A Spreadsheet with Row and Column Checksums:** Imagine a grid of numbers where every row has a running total in the margin, and every column also has a running total at the bottom — like a classic accounting ledger cross-check. If exactly one cell gets corrupted, **both** its row's total _and_ its column's total will be off — and since only one cell sits at that specific row/column intersection, you know **exactly which cell** to fix, without needing to re-request the entire ledger. But if two separate cells are corrupted, you'll notice the totals are wrong (detection), but you generally can't pinpoint _which two_ cells without more information (no correction).

### Forward Error Correction (FEC)

The general **ability of the receiver to both detect and correct errors** — as illustrated by two-dimensional parity — is known as **forward error correction (FEC)**. These techniques are commonly used in **audio storage and playback devices**, such as audio CDs.

In a networking setting, FEC techniques can be used either **by themselves**, or **in conjunction with link-layer ARQ** (Automatic Repeat reQuest) techniques similar to those examined for reliable data transfer in Chapter 3. FEC techniques are valuable because they:

1. **Decrease the number of sender retransmissions required.**
2. **Allow for immediate correction of errors at the receiver** — this avoids having to wait for the **round-trip propagation delay** needed for the sender to receive a NAK packet, and for the retransmitted packet to then propagate back to the receiver.

This second benefit is a **potentially significant advantage** for:

- **Real-time network applications** [Rubenstein 1998], where retransmission delays are especially costly.
- **Links with long propagation delays** (such as **deep-space links**), where the round-trip cost of a retransmission cycle can be enormous.

---

## 6.2.2 Checksumming Methods

In **checksumming techniques**, the **d bits of data** (from Figure 6.4's setup) are treated as a sequence of **k-bit integers**. One simple checksumming method is to simply **sum these k-bit integers** and use the resulting sum as the error-detection bits.

### The Internet Checksum

**The Internet checksum** is based on exactly this approach: bytes of data are treated as **16-bit integers** and summed. The **1s complement** of this sum forms the Internet checksum that is carried in the segment header. (This was discussed in detail in Section 3.3.)

**Receiver-side check:** the receiver checks the checksum by taking the **1s complement sum** of the received data (**including** the checksum) and checking whether the result is all-1 bits.

- If **any** of the bits are 0, an error is indicated.
- RFC 1071 discusses the Internet checksum algorithm and its implementation in detail.

### Where the Checksum Is Computed

|Protocol|Checksum Coverage|
|---|---|
|**TCP and UDP**|Checksum is computed over **all fields** — both the header and the data fields — of the segment|
|**IP**|Checksum is computed **only over the IP header** (since the encapsulated UDP or TCP segment already carries its own separate checksum)|
|**Other protocols (e.g., XTP)**|One checksum may be computed over the header, and a **separate** checksum computed over the entire packet|

### Why Checksumming — and Why Not at the Link Layer?

Checksumming methods require **relatively little packet overhead**. For example, the checksums used in TCP and UDP take up only **16 bits**. However, they provide **relatively weak protection** against errors compared with the **cyclic redundancy check**, which is discussed next and which is more often used at the link layer.

This raises a natural question: **why is checksumming used at the transport layer, while CRC is used at the link layer?**

- The **transport layer** is typically implemented **in software**, as part of the host's operating system. Because transport-layer error detection is implemented in software, it's important to have a **simple and fast** error-detection scheme — hence, checksumming.
- The **link layer**, by contrast, has error detection implemented in **dedicated hardware** in adapters, which can **rapidly perform the more complex CRC operations** that would be too slow to compute purely in software.

> **Analogy — Mental Math vs. a Dedicated Calculator:** Checksumming is like doing quick **mental addition** — fast, low-effort, "good enough" for a rough sanity check, and something you can do on the fly without any special tools (i.e., in software, on a general-purpose CPU). CRC is like using a **dedicated calculator built for one specific complex calculation** — it can chew through much more sophisticated math (polynomial division) far faster than a human doing it in their head, but it requires purpose-built hardware to get that speed. It makes sense to use "mental math" (checksumming) where software flexibility matters more than raw error-catching power (the transport layer), and to use the "dedicated calculator" (CRC, in hardware) where speed and strength of error-catching matter more (the link layer).

---

## 6.2.3 Cyclic Redundancy Check (CRC)

### Why CRC — "Polynomial Codes"

An error-detection technique used **widely** in today's computer networks is based on **cyclic redundancy check (CRC)** codes. CRC codes are also known as **polynomial codes**, since it's possible to view the bit string to be sent as **coefficients of a polynomial** whose operations are done using **polynomial arithmetic** on the bit string.

### How CRC Codes Operate

Consider the **d-bit** piece of data, **D**, that the sending node wants to send to the receiving node. The sender and receiver **must first agree on** an **r + 1** bit pattern, known as a **generator**, which is denoted **G**. It's required that the **most significant (leftmost)** bit of **G** be a **1**.

```
Fig -- CRC: Sender and Receiver View
────────────────────────────────────────
      d bits         r bits
 ┌──────────────┐ ┌────────┐
 │ D: data bits │ │R:CRCbits│
 └──────────────┘ └────────┘

 Sender chooses R so that:
   D · 2^r XOR R   is exactly
   divisible by generator G  (no remainder)

 Transmits (D · 2^r XOR R), i.e. D
 followed by R, over the link.

 Receiver divides received (d+r) bits
 by G:
   remainder = 0  -> no error detected
   remainder ≠ 0  -> error detected
```

**The key idea** behind CRC codes: for a given piece of data, **D**, the sender chooses **r** additional bits, **R**, and appends them to **D** such that the resulting **d + r** bit pattern (interpreted as a binary number) is **exactly divisible by G** (i.e., has **no remainder**), using **modulo-2 arithmetic**. The process of error checking is thus simple: **the receiver divides the received d + r bits by G.** If the remainder is **nonzero**, the receiver knows an error has occurred; otherwise, the data is accepted as being correct.

### Modulo-2 Arithmetic (No Carries, No Borrows)

All CRC calculations are done in **modulo-2 arithmetic**, without carries in addition or borrows in subtraction. This means addition and subtraction are **identical**, and both are equivalent to the **bitwise exclusive-OR (XOR)** of the operands. For example:

```
1011 XOR 0101 = 1110
1001 XOR 1101 = 0100
```

We similarly have:

```
1011 - 0101 = 1110
1001 - 1101 = 0100
```

**Multiplication and division** work the same as in regular base-2 arithmetic, except that any required addition or subtraction is done without carries or borrows. As in regular binary arithmetic, **multiplication by 2^k left-shifts a bit pattern by k places.** Thus, given **D** and **R**, the quantity **D · 2^r XOR R** yields the **d + r** bit pattern being transmitted.

### How the Sender Actually Computes R

The crucial question is: how does the sender compute **R**? The goal is to find **R** such that there is an integer **n** where:

```
D · 2^r XOR R = nG
```

That is, choose **R** such that **G divides evenly into (D · 2^r XOR R)**, with no remainder. If we XOR (add, modulo-2, without carry) **R** to both sides of the above equation:

```
D · 2^r = nG XOR R
```

This equation tells us that if we **divide D · 2^r by G**, the value of the remainder is **precisely R**. In other words:

```
R = remainder ( D · 2^r / G )
```

**Worked example** (from Figure 6.7): for the case of **D = 101110**, **d = 6**, **G = 1001**, and **r = 3**, the resulting **9 bits transmitted** in this case are **101 110 011**.

### CRC Standards in Practice

International standards have been defined for **8-, 12-, 16-, and 32-bit generators, G**. The **CRC-32** 32-bit standard, which has been adopted in a number of link-level IEEE protocols, uses a generator of:

```
G_CRC-32 = 100000100110000010001110110110111
```

**Detection guarantee:** Each of the CRC standards can detect **burst errors of fewer than r + 1 bits.** (This means that **all consecutive bit errors of r bits or fewer** will be detected.) Furthermore, under appropriate assumptions:

- A burst of length **greater than r + 1** is detected with probability **1 − 0.5^r**.
- Each of the CRC standards can also detect **any odd number of bit errors**.

> **Analogy — A Long Division "Divisibility Password":** Think of the generator **G** as a secret **divisibility rule** that both sender and receiver agree on in advance — like agreeing that "the final number, including a few extra digits we'll tack on, must be evenly divisible by 1001." The sender does the algebra needed to pick exactly the right extra digits (**R**) so that the padded number satisfies the rule perfectly. When the receiver gets the number, they don't need to know _what the original data was supposed to be_ — they simply check the divisibility rule themselves. If it still divides evenly, the data almost certainly arrived intact; if it doesn't, **something** got scrambled in transit. Because division is sensitive to _where_ in the number a digit gets changed, this "password" catches a much wider range of corruption patterns — including whole clusters (bursts) of consecutive flipped bits — than simply checking a sum (checksum) or a single parity count ever could.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**EDC schemes protect against noise, not malice**|A sophisticated on-path attacker can deliberately craft a modified frame whose bits still satisfy the parity, checksum, or CRC check — meaning EDC mechanisms provide **no cryptographic integrity guarantee** and can be bypassed by anyone who understands the (publicly documented) algorithm in use|EDC (parity/checksum/CRC) must never be treated as a security mechanism; genuine tamper-detection requires cryptographic integrity mechanisms (e.g., MACs, digital signatures) layered on top, since CRC/checksum are designed only to catch accidental, not intentional, corruption|
|**Single-bit parity's ~50% blind spot under burst errors**|An attacker capable of injecting a burst of bit errors (e.g., via signal jamming timed to a frame) into a link protected only by single-bit parity has close to even odds of the tampering going completely undetected|Because burst errors are common in real channels (not just from attackers, but from ordinary interference), single-bit parity is inadequate for any serious deployment — this is precisely why CRC, which explicitly guarantees detection of burst errors up to r bits, is the practical standard at the link layer|
|**CRC generator polynomials are standardized and public**|Since standard CRC generators (e.g., CRC-32) are publicly known, an attacker with the ability to modify a frame in transit can, in principle, compute a replacement R that keeps the CRC check passing for their modified data — this is a well-known technique in tampering with files/protocols that rely solely on CRC for "integrity"|Systems that need real integrity protection against intentional tampering (not just accidental link noise) should pair CRC (for catching transmission noise) with a separate, keyed cryptographic check the attacker cannot forge without knowing a secret key|
|**Checksumming's software implementation surface**|Because Internet checksums are computed in host software (OS network stack) rather than hardware, bugs in checksum computation/validation code are a potential (if narrow) target for exploitation via deliberately malformed segments|Keeping checksum-handling code in the OS network stack well-tested and patched reduces this narrow but real attack surface, especially given how foundational and widely-invoked this code path is|

---

## Questions I Still Have

- [ ] The text says two-dimensional parity can detect (but not correct) any combination of two bit errors — what's the underlying mathematical reason it fails to correct in the two-error case specifically, and how does this generalize (or not) to three or more simultaneous errors?
- [ ] For CRC, the standards guarantee detection of any burst error of length ≤ r bits, and detection with probability 1 − 0.5^r for longer bursts — intuitively, why does the detection probability formula involve exactly 0.5^r, and what's the "appropriate assumption" the book mentions that this formula depends on?
- [ ] Given that checksumming is described as weaker than CRC, why do TCP and UDP still use the (weaker) Internet checksum rather than adopting CRC at the transport layer now that CPUs are so much faster than when these protocols were designed?
- [ ] The text mentions FEC can be used "by itself" or "in conjunction with" ARQ — in a real protocol, how is the decision made about which errors get handled via FEC (local correction) versus which trigger a full ARQ retransmission cycle?
- [ ] For the CRC-32 generator given as a 33-bit pattern, how was this specific bit pattern chosen/standardized — is it selected for particular mathematical properties (e.g., being a primitive polynomial), or was it more empirically tuned?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**EDC (Error-Detection and -Correction bits)**|Bits appended to data D by the sender, used by the receiver to check whether D was corrupted in transit|
|**Undetected bit error**|A transmission error that occurs but is not caught by the error-detection scheme in use — the receiver believes the data to be correct when it is not|
|**Parity bit**|A single extra bit chosen so that the total number of 1s among the data bits (plus the parity bit) is even (even parity) or odd (odd parity)|
|**Two-dimensional parity**|A generalization computing a parity bit for every row and every column of a data grid, enabling both detection and correction of any single-bit error|
|**Forward error correction (FEC)**|The general ability of a receiver to both detect and correct bit errors without requiring retransmission from the sender|
|**Checksumming**|An error-detection method that sums k-bit integer segments of data and uses the resulting sum (or its complement) as the error-detection bits|
|**Internet checksum**|The specific checksum method used by TCP, UDP, and IP: treats data as 16-bit integers, sums them, and uses the 1s complement of the sum as the checksum|
|**Cyclic redundancy check (CRC)**|A polynomial-based error-detection technique where the sender appends r bits, R, chosen so the full (d+r)-bit pattern is exactly divisible (mod-2) by an agreed-upon generator, G|
|**Generator (G)**|An (r+1)-bit pattern, agreed upon in advance by sender and receiver, used as the divisor in CRC calculations; must have a leading 1 bit|
|**Polynomial codes**|Another name for CRC codes, reflecting the fact that the bit string can be interpreted as the coefficients of a polynomial under modulo-2 (polynomial) arithmetic|
|**Modulo-2 arithmetic**|Arithmetic without carries or borrows, in which addition and subtraction are both equivalent to bitwise XOR|
|**Burst error**|A cluster of consecutive bit errors, as opposed to independently/randomly scattered single-bit errors; a known real-world weakness of simple schemes like single-bit parity|
|**CRC-32**|A standardized 32-bit CRC generator adopted in numerous link-level IEEE protocols, guaranteeing detection of all burst errors of length ≤ 32 bits and any odd number of bit errors|

---

## Related Concepts

---

→ Next: [[MULTIPLE ACCESS LINK & PROTOCOL]]