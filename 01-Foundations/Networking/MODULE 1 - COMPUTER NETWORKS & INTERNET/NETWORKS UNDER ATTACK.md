---
title: NETWORKS UNDER ATTACK
date: 2026-05-17
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 1.6 Networks Under Attack

> **One-Line Summary:** The Internet was originally designed for mutual trust — not security. Today's four major attack categories are malware infection, denial-of-service (DoS/DDoS), packet sniffing, and IP spoofing — each exploiting fundamental properties of the network architecture we studied in sections 1.1 through 1.5.

---

## Why Security Matters

![[Pasted image 20260619143205.png]]

The Internet is now **mission critical** for:

- Large and small companies
- Universities
- Government agencies
- Individual professional, social, and personal activities
- **Billions of "things"** — wearables and home/IoT devices are increasingly being connected to the Internet as well

Behind all this utility is a dark side — "bad guys" who attempt to:

- **Damage** Internet-connected computers
- **Violate** our privacy
- **Render inoperable** the Internet services we depend on

> Network security is about two things:
> 
> 1. How bad guys can **attack** computer networks.
> 2. How we can **defend** against those attacks — or better yet, design architectures **immune** to attacks in the first place.

Network security has become a **central topic** in computer networking because of the frequency and variety of existing attacks, as well as the threat of new and more destructive future attacks. Since we don't yet have deep expertise in networking protocols at this point in the course, this section is meant only to survey today's more prevalent security-related problems and whet our appetite for the more substantial security discussions later in the book (Chapter 8 in particular).

**Why the Internet is insecure by design:** The original Internet was designed on the model of:

> _"A group of mutually trusting users attached to a transparent network"_

In this model:

- No need for authentication — user identity taken at declared face value, rather than being authenticated by default.
- Anyone can send a packet to anyone — default capability, not a requested/granted privilege.
- The network is transparent — no built-in encryption or access control.

Today's Internet **does not** involve mutually trusting users. Users:

- Don't necessarily trust each other.
- May wish to communicate anonymously.
- May communicate indirectly through third parties — e.g. **Web caches** (Chapter 2) or **mobility-assisting agents** (Chapter 7).
- May distrust the hardware, software, and even the air through which they communicate.

> **Keep this in mind throughout the course:** Communication among mutually trusted users is the **exception**, not the rule. Welcome to the world of modern computer networking.

---

# Attack Category 1 — Malware

## What Is Malware?

![[Pasted image 20260619143409.png]]

When we attach devices to the Internet, we receive all kinds of good stuff — **Instagram posts, search results, streaming music, video conference calls, streaming movies**, and so on. But along with the good stuff comes **malicious software** — collectively known as **malware** — that can enter and infect our devices.

**What malware can do once it infects a host:**

- Delete files
- Install **spyware** — collects private information:
    - Social security numbers
    - Passwords
    - Keystrokes
    - Sends all of it back to the attacker over the Internet
- Enroll the compromised host into a **botnet**

---

## Botnets

![[Pasted image 20260619143628.png]]

> A **botnet** is a network of thousands of similarly compromised devices that the bad guys control and leverage for:
> 
> - Spam email distribution
> - Distributed denial-of-service (DDoS) attacks against targeted hosts

---

## Self-Replicating Malware

Much of today's malware is **self-replicating**:

1. Infects one host.
2. From that host, seeks entry into other hosts over the Internet.
3. From newly infected hosts, seeks entry into yet more hosts.
4. Spreads **exponentially fast**.

Two main forms:

| Feature          | Virus                                              | Worm                                                           |
| ---------------- | -------------------------------------------------- | -------------------------------------------------------------- |
| User interaction | Required                                           | None needed                                                    |
| Entry method     | User opens malicious email attachment              | Exploits vulnerable network application autonomously           |
| Classic example  | Email attachment → user runs executable → executes | Attacker sends malware to open port → app accepts it           |
| Self-replication | Sends itself to every address in user's contacts   | Infected host scans Internet for same vulnerable app → spreads |
![[Pasted image 20260619143911.png]]

> Today malware is **pervasive and costly** to defend against. Key design question: What can network designers do to defend Internet-attached devices from malware attacks?

---

# Attack Category 2 — Denial-of-Service (DoS) Attacks

## What Is a DoS Attack?

> A **DoS (Denial-of-Service) attack** renders a network, host, or other piece of infrastructure **unusable by legitimate users**.

**Targets include:**

- Web servers
- Email servers
- DNS servers (discussed in Chapter 2)
- Institutional networks

Internet DoS attacks are extremely common — thousands occur every year.

---

## Three Categories of DoS Attacks

![[Pasted image 20260619144100.png]]

### 1. Vulnerability Attack

```
Attacker → few well-crafted messages/packets
         → targets vulnerable app or OS
         → right sequence triggers the flaw
         → service stops / host crashes
```

- Doesn't require flooding.
- Exploits a specific software bug.
- Even one or two packets can take down a server.
- Example: sending a malformed packet that causes a buffer overflow → remote code execution or crash.

---

### 2. Bandwidth Flooding

Attacker → deluge of packets → clogs access link → legitimate traffic can't get through

**Key connection to Section 1.4 (delay and throughput):**

- If server's access link rate = **R bps**
- Attacker must send traffic at approximately **R bps** to cause damage
- If R is very large → a single attacker may not generate enough traffic
- Solution for attacker: use a **botnet** (DDoS)

---

### 3. Connection Flooding

```
Attacker → floods target with half-open or fully open TCP connections
         → (TCP connections are discussed in Chapter 3)
         → host bogs down managing bogus connections
         → legitimate connections refused
```

- Exploits the TCP three-way handshake.
- Attacker sends SYN packets but never completes the handshake.
- Server allocates resources for each half-open connection.
- Server's connection table fills up → legitimate users get refused.
- Also known as a **SYN flood attack**.

> Different defenses are needed for each of the three DoS categories — there's no single fix that covers vulnerability attacks, bandwidth flooding, and connection flooding all at once.

---

## DoS → DDoS (Distributed Denial-of-Service)

![[Pasted image 20260619144316.png]] _(This corresponds to Figure 1.26 in the textbook — note that the diagram's compromised attack sources are now labeled **"zombies"** rather than "slaves.")_

**The single-source problem:**

- If R is very large, a single attack source may not generate enough traffic.
- An upstream router may detect and block traffic from a single source before it reaches the server.

**The DDoS solution:**

```
Attacker ──"start attack"──→ zombie ──┐
                             zombie ──┤
                             zombie ──┼──→ VICTIM
                             zombie ──┤
                             zombie ──┘
         aggregate rate ≈ R → access link clogged
```

- Attacker controls **multiple sources** (botnet "zombies").
- Each zombie blasts traffic at the target simultaneously.
- The aggregate rate across all controlled sources needs to be ≈ R.
- DDoS attacks leveraging botnets with thousands of hosts are a **common occurrence today**.
- There are roughly **100 major DDoS attacks** on Internet infrastructure **every year** [Sitaraman 2022].

> **Why DDoS is harder to defend against than DoS:**
> 
> - Traffic comes from many different IP addresses.
> - Hard to distinguish zombie traffic from legitimate traffic.
> - Upstream routers can't easily block it — source IPs keep changing.
> - Requires specialized DDoS scrubbing infrastructure.

---

# Attack Category 3 — Packet Sniffing

## What Is a Packet Sniffer?

![[Pasted image 20260619144512.png]]

> A **packet sniffer** is a passive receiver that records a copy of every packet that flies by.

**How it works:**

- Attacker places a passive receiver in the **vicinity of a wireless transmitter** (or on a wired broadcast medium).
- The receiver **copies every packet** transmitted on that channel.
- The attacker then **analyzes captured packets offline** for sensitive information — sometimes even reconstructing application-layer data from the captured packets.

**What sniffed packets can contain:**

- Passwords
- Social security numbers
- Trade secrets
- Private personal messages
- Session tokens and cookies
- Any unencrypted data

---

## Where Sniffing Works

**Wireless environments:**

- WiFi-connected laptops, smartphones, tablets.
- Handheld devices with **cellular Internet connections** (mobility and cellular networking are covered in Chapter 7).
- Any device within range of the wireless transmitter.
- Attacker just needs to be in the vicinity — no physical access needed.

**Wired broadcast environments:**

- In wired Ethernet LANs (broadcast medium) — packets are sent to all devices on the segment.
- A sniffer on any device in the LAN sees all packets.
- Cable access networks (HFC) — also broadcast → vulnerable.
- A bad guy who gains access to an institution's access router or access link can plant a sniffer that copies **every packet** going to/from the organization.

---

## Why Sniffing Is Hard to Detect

> Because packet sniffers are **passive** — they do not inject any packets into the channel — they are **very difficult to detect**.

```
Normal device  → receives only packets addressed to it
Sniffer device → receives ALL packets on the link
               → copies them silently
               → generates no extra traffic
               → leaves no anomaly to detect
```

**Tools used for legitimate sniffing (also usable maliciously):**

- **Wireshark** [Wireshark 2025] — the industry-standard packet analysis tool.
    - Freely available.
    - Used by networking students, engineers, and attackers alike.
    - The Wireshark labs associated with the textbook use exactly this tool for hands-on packet capture and application-layer data reconstruction exercises.

---

## Defense Against Sniffing

> Some of the best defenses against packet sniffing involve **cryptography**.

- **Encrypt all traffic** — even if the attacker captures every packet, the contents are unreadable without the key.
- **TLS/HTTPS** — encrypts web traffic end-to-end.
- **VPN** — encrypts all traffic between device and VPN server.
- **WPA3** — latest WiFi encryption standard for wireless networks.
- **End-to-end encryption** — ensures only sender and receiver can read the content, regardless of who captures packets in between.
- Cryptography as applied to network security is explored in depth in Chapter 8.

---

# Attack Category 4 — IP Spoofing

## The Bad Guys Can Masquerade as Someone You Trust

![[Pasted image 20260619144712.png]]

It is surprisingly easy to create a packet with:

- An **arbitrary source IP address**
- Any packet content
- Any destination address

...and then transmit this hand-crafted packet into the Internet, which will dutifully forward it to its destination.

> The ability to inject packets into the Internet with a **false source address** is known as **IP spoofing** — and is but one of many ways in which one user can masquerade as another user.

---

## How IP Spoofing Works

**Legitimate:** Real Source ──[real IP]──→ Router ──→ Destination Destination trusts source IP → acts accordingly

**Spoofed:** Attacker ──[fake IP]──→ Router ──→ Victim Victim (e.g. an Internet router) trusts the false source IP as truthful → executes embedded command, e.g. modifies its own forwarding table

**Why routers don't catch this:**

- Recall from Section 1.3: routers only look at the **destination IP** to forward packets. They do NOT verify whether the source IP is genuine.
- The Internet was designed for efficiency, not authentication.

---

## What an Attacker Can Do With IP Spoofing

|Attack|How Spoofing Enables It|
|---|---|
|**Masquerade as trusted host**|Send packets appearing to come from a trusted IP (e.g., internal network address)|
|**Amplification attacks**|Send requests with victim's IP as source → servers flood victim with responses|
|**Bypass IP-based access controls**|Systems that trust based on IP address can be fooled|
|**Hide attack source**|Make DDoS traffic appear to come from many legitimate IPs|
|**Man-in-the-middle**|Poison routing tables by injecting spoofed routing protocol messages|

---

## Defense Against IP Spoofing

> The solution requires **end-point authentication** — a mechanism that allows us to determine with certainty if a message originates from where we think it does. End-point authentication mechanisms are explored in Chapter 8.

**Defenses:**

- **End-point authentication** — cryptographically verify the source of every message (Chapter 8).
- **Ingress filtering** — ISPs and routers check that packets arriving on a link actually have a source address consistent with that link's network. Packets with spoofed addresses are dropped.
- **IPSec** — adds authentication and encryption at the network layer.
- **Cryptographic signatures** — packets signed with a private key cannot be forged without the key.

---

# All Four Attacks — Comparison Table

|Attack|What It Does|Key Property Exploited|Detection Difficulty|Defense|
|---|---|---|---|---|
|**Malware**|Infects host, steals data, creates bots|Users run untrusted code|Medium|Antivirus, sandboxing, patches|
|**DoS/DDoS**|Overwhelms target with traffic or connections|Finite buffer + bandwidth|Medium (DDoS is hard)|Rate limiting, scrubbing, over-provisioning|
|**Packet Sniffing**|Passively copies all passing packets|Broadcast/wireless media are shared|Very hard (passive)|Encryption (TLS, VPN, WPA3)|
|**IP Spoofing**|Forges source IP to impersonate others|Routers don't verify source address|Hard|End-point authentication, ingress filtering|

---

## How Each Attack Maps to Concepts from 1.1–1.5

|Concept Learned|Attack That Exploits It|
|---|---|
|**Packets travel through shared links** (1.2)|Sniffing — shared broadcast medium|
|**Output buffers are finite** (1.3)|DoS bandwidth flooding — fill buffer → packet loss|
|**Traffic intensity La/R → 1 = collapse** (1.4)|DoS/DDoS — push La/R above 1 at victim's access link|
|**TCP connection state** (Chapter 3, briefly previewed in 1.3)|Connection flooding — exhaust connection table|
|**Routers only check destination IP** (1.3)|IP spoofing — source address never verified|
|**Physical/link layer is broadcast** (1.2)|Packet sniffing on Ethernet or WiFi|
|**Self-replicating code spreads via protocols** (1.1)|Worms spreading via vulnerable network applications|
|**Internet designed for mutual trust** (1.6)|All attacks — the original design assumption was wrong|

---

## The Big Picture — Why the Internet Is Insecure

**Original Assumptions** [Blumenthal 2001]:

- All users are mutually trusting
- Network is transparent — no need to hide
- Identity can be taken at declared face value
- Anyone can send to anyone (open by default)

**Reality Today:**

- Users do NOT trust each other
- Privacy is essential
- Identity must be verified cryptographically
- Open-by-default is a major attack surface
- Users often communicate indirectly through third parties (Web caches, mobility-assisting agents) and may distrust the hardware, software, or even the air through which they communicate

**Result:** All four attack types flow naturally from these broken assumptions. Security was bolted on after the fact — TLS, IPSec, firewalls — rather than built in from the start.

**Security challenges to keep in mind going forward (per the textbook's closing framing of this section):** defenses against sniffing, end-point masquerading, man-in-the-middle attacks, DDoS attacks, malware, and more.

---

## Questions I Still Have

- [ ] How exactly does a SYN flood exhaust the TCP connection table — what data structure does the server maintain per connection?
- [ ] Ingress filtering sounds simple — why isn't it universally deployed to stop IP spoofing?
- [ ] Can WPA3 WiFi encryption completely prevent packet sniffing, or can an attacker still capture and decrypt packets?
- [ ] In a DDoS, how do botnet zombies receive the "start attack" command without being detected by defenders?
- [ ] What is the difference between a DoS vulnerability attack and an exploit? Are all exploits DoS attacks?
- [ ] Why did the diagram terminology shift from "slave" to "zombie" for DDoS attack sources — is "zombie" now the more standard industry term?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Malware**|Malicious software that infects and harms Internet-connected devices|
|**Spyware**|Malware that collects private info and sends it to the attacker|
|**Botnet**|Network of thousands of compromised devices controlled by an attacker|
|**Self-replicating malware**|Malware that automatically spreads to new hosts after infection|
|**Virus**|Malware requiring user interaction to spread (e.g., opening email attachment)|
|**Worm**|Malware that enters and spreads without any user interaction|
|**DoS**|Denial-of-Service — makes a resource unusable by legitimate users|
|**DDoS**|Distributed DoS — DoS using many coordinated attack sources (a botnet of zombies)|
|**Vulnerability attack**|DoS via well-crafted messages/packets exploiting a specific software bug|
|**Bandwidth flooding**|DoS via overwhelming the target's access link with traffic|
|**Connection flooding**|DoS via exhausting server's TCP connection table with bogus connections|
|**SYN flood**|Type of connection flooding — sends many SYN packets, never completes handshake|
|**Zombie**|A compromised host under an attacker's control, used as a DDoS attack source (older texts call this a "slave")|
|**Packet sniffer**|Passive receiver that copies every packet on a shared medium|
|**Wireshark**|Industry-standard packet capture and analysis tool|
|**IP spoofing**|Injecting packets with a forged source IP address|
|**End-point authentication**|Cryptographic mechanism to verify message origin|
|**Ingress filtering**|ISP/router drops packets with source IPs inconsistent with their arrival link|

---

## Related Concepts

---

→ Next: [[PRINCIPLES OF NETWORK APPLICATIONS]]