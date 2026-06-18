---
title: P2P
date: 2026-06-18
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.6 Peer-to-Peer (P2P) Applications — When the Users Become the Infrastructure

> **One-Line Summary:** P2P architecture removes the always-on infrastructure server from the picture — ordinary user machines ("peers") serve each other directly. This makes the system **self-scaling** (more users means more total capacity, not just more load), but also makes it harder to control, secure, and police, since there's no single throat to choke.

---

## Core Idea

Every application covered earlier in this chapter — the Web, e-mail, DNS — is client-server at its core: a small number of powerful, always-on machines do nearly all the work, while a much larger number of clients connect, take what they need, and disconnect.

P2P architecture works the opposite way. There's minimal or no reliance on always-on infrastructure servers; instead, pairs of intermittently connected hosts — peers — talk directly to each other. The peers aren't owned or maintained by any service provider; they're just regular desktops and laptops, switched on and off whenever their owners feel like it. That's what "intermittently connected" really means: nobody is paying to keep your laptop running 24/7, and the system has to be designed around the assumption that any given peer might vanish at any moment.

> **Analogy — Potluck vs. Catered Dinner:** A client-server application is a catered party — one company buys the ingredients, cooks in one kitchen, and serves every guest from that single kitchen. Ten times more guests means the caterer needs ten times more ovens and staff; the entire burden scales with the host. A P2P application is a potluck — every guest who shows up also brings a dish to share. The more guests arrive, the more food arrives too. Serving capacity grows automatically with the number of people attending, with nobody needing to rent a bigger kitchen.

|Property|Client-Server|Peer-to-Peer|
|---|---|---|
|**Who serves data**|A small number of dedicated, always-on servers|Every peer both consumes and supplies data|
|**Ownership of infrastructure**|Service provider owns and maintains the servers|Ordinary users own the peers — their own PCs|
|**Availability**|Servers are always-on by design|Peers come and go unpredictably|
|**How load grows with users**|Server bears the entire burden; more users = more server strain|Self-scaling — more users bring more combined capacity|
|**Examples already covered**|Web (HTTP), e-mail (SMTP/IMAP), DNS|File distribution (BitTorrent), distributed databases (DHT)|

Two P2P applications illustrate this self-scalability in two very different ways: distributing one large file to many hosts (BitTorrent), and spreading an entire database across millions of peers so that no single one holds the whole thing (a Distributed Hash Table, or DHT).

---

# 2.6.1 P2P File Distribution

## The Problem: Getting One Big File to Everyone

Picture a new Linux ISO, a major OS patch, an album, or a video file that a huge number of people all want at once. Under client-server distribution, the server has to personally transmit a full copy of the file to every single requester — if a million people want it, the server's access link has to push the entire file a million separate times. The server's bandwidth becomes the hard ceiling on how fast anyone gets the file, no matter how fast their own connections are.

P2P flips that completely: once a peer has downloaded even part of the file, it can immediately start redistributing that part to other peers, taking load directly off the original server. BitTorrent is the dominant protocol built around this idea.

> **Analogy — Photocopying a Handout:** A professor has one master copy of a 500-page handout and sixty students need it. Client-server is the professor personally photocopying and handing a full copy to each of the sixty students, one at a time — the professor does all sixty jobs alone. P2P is the professor photocopying the first ten pages for a few students, who immediately start copying those same pages for other students while the professor moves on to the next chunk. Within minutes, dozens of students are feeding each other pages, and the professor's photocopier is just one of many machines doing the work now.

## Quantifying Scalability — How Long Does Distribution Actually Take?

A simple mathematical model makes the comparison precise rather than just intuitive.

**Variables used throughout:**

|Symbol|Meaning|
|---|---|
|**F**|Size of the file to be distributed, in bits|
|**N**|Number of peers that want a copy of the file|
|**u_s**|Upload rate of the server's access link|
|**u_i**|Upload rate of the _i_-th peer's access link|
|**d_i**|Download rate of the _i_-th peer's access link|
|**d_min**|The slowest download rate among all peers: `min{d_1, d_2, ..., d_N}`|

The model assumes every bottleneck sits in the access links (the "last mile" connecting a server or peer to the Internet), not inside the Internet's core, and that no other traffic is competing for that bandwidth at the same time.

![[Pasted image 20260618152734.png]] _(Figure 2.24 — A server holding file F connects to the Internet at upload rate u_s; N peers connect with their own upload rates u_1...u_N and download rates d_1...d_N)_

### Client-Server Distribution Time (D_cs)

In client-server, the peers contribute nothing — the server alone pushes every bit to every peer. Two independent constraints set the floor on how fast this can possibly go:

- The server has to transmit a full copy of the file to each of N peers, meaning NF total bits leave the server. At upload rate u_s, that alone takes at least NF/u_s.
- No peer can receive the whole file faster than its own download rate allows. The slowest peer caps the process at F/d_min.

Because the real distribution time can't beat either bound, it's the larger of the two:

```
D_cs = max { NF / u_s ,  F / d_min }                       (Equation 2.1)
```

For any reasonably large N, the first term dominates, so distribution time grows **linearly with the number of peers**. Double the number of people wanting the file, and the server-bound delivery time roughly doubles too — the server's single upload pipe is a permanent bottleneck regardless of how fast individual peers can download.

### P2P Distribution Time (D_p2p)

This is more subtle because peers redistribute pieces to each other as they receive them. Three independent constraints apply:

- The server only needs to push each bit out of its own access link once — after that, peers can recirculate it among themselves. So the floor here is F/u_s, not NF/u_s.
- The slowest downloader still caps things at F/d_min, exactly as before.
- The system's total upload capacity is the server's upload rate plus every peer's upload rate combined: u_s + u_1 + ... + u_N. Delivering F bits to each of N peers means NF bits of total work, and that work can't go faster than the combined capacity available to do it. That gives a third floor: NF / (u_s + u_1 + ... + u_N).

```
D_p2p = max { F/u_s ,  F/d_min ,  NF / (u_s + u_1 + ... + u_N) }      (Equation 2.3)
```

![[Pasted image 20260618152831.png]] _(Figure 2.25 — Client-server distribution time grows linearly without bound as N increases; the P2P curve flattens out and stays under one hour for any N)_

The P2P formula is friendlier because the same NF bits of delivery work get spread across the combined upload capacity of every peer plus the server — and every new peer that joins adds its own upload rate to that combined capacity at the same time it adds demand. That's the literal mathematical definition of self-scaling.

### Worked Numeric Example

Assume F/u_s = 1 hour (the server alone could push the whole file to one recipient in an hour), u_s = 10u (the server is ten times faster than any single peer's upload), and every peer shares the same upload rate u, with download rates fast enough to never be the bottleneck.

|N (peers)|D_cs (client-server)|D_p2p|
|---|---|---|
|5|30 min|20 min|
|10|1 hr|30 min|
|20|2 hr|40 min|
|30|3 hr|45 min|
|1,000|~100 hr (4+ days)|~59.4 min|

Client-server distribution time explodes without bound as N grows — a thousand-fold increase in N produces a roughly thousand-fold increase in distribution time. P2P distribution time converges toward F/u_s (one hour) and mathematically can never exceed it, no matter how many peers join.

---

## BitTorrent — The Protocol That Made P2P File Sharing Mainstream

In BitTorrent terminology, all the peers participating in distributing one particular file together form a **torrent**. Peers trade fixed-size pieces of the file called **chunks**, typically 256 KB each. A new peer starts with zero chunks and accumulates more over time, simultaneously downloading new chunks and uploading the ones it already has to other peers. Once a peer has the complete file, it can leave immediately or keep seeding chunks to others.

|BitTorrent Term|Meaning|
|---|---|
|**Torrent**|All the peers currently participating in distributing one specific file|
|**Chunk**|A fixed-size piece of the file (typically 256 KB) traded between peers|
|**Tracker**|An infrastructure node that registers peers joining a torrent and tracks which peers are currently active|
|**Neighboring peers**|The subset of peers a given peer has actually succeeded in opening a TCP connection with, out of a larger candidate list|

### How a Peer Joins and Trades

When a new peer joins a torrent, the tracker hands it a randomly selected subset of currently-participating peers — fifty is a typical example. The new peer attempts to open TCP connections with all fifty; whichever connections succeed become its neighboring peers, and that set keeps shifting as other peers join and leave over time.

![[Pasted image 20260618152938.png]] _(Figure 2.26 — A new peer obtains a list of candidate peers from the tracker, establishes TCP connections with a subset of them, and trades chunks back and forth across that mesh)_

### Two Big Decisions Every Peer Must Make

At any given moment, a peer has some chunks and is missing others, and each neighbor has a different mix. That creates exactly two ongoing decisions.

#### 1. Which chunk to request next? → **Rarest First**

A peer requests chunks that are currently the _least_ common among its neighbors before requesting more abundant ones. Spreading scarce chunks first keeps the overall supply of every chunk roughly balanced across the swarm.

> **Analogy — Trading Cards:** In a classroom trading rare cards, if everyone already owns five copies of a common card, that card is in no danger of disappearing. But if only one kid in the whole class owns a legendary card, it's one lost backpack away from vanishing from circulation entirely. Smart traders prioritize spreading the rarest cards first, so a single dropout never wipes something out. Rarest-first does exactly that for file chunks — it deliberately gets scarce data copied wide before the one peer holding it has a chance to disconnect for good.

#### 2. Whose requests to honor? → **Tit-for-Tat (the Choking Algorithm)**

Each peer continuously measures the rate at which every neighbor is currently sending it data, and reciprocates by sending chunks back to whichever four neighbors are currently the fastest suppliers — these four are said to be **unchoked**. That ranking gets recalculated roughly every ten seconds. On top of that, every thirty seconds the peer also picks one additional neighbor at random and sends it chunks too — an **optimistic unchoke**. If that randomly probed peer turns out to reciprocate well, it can earn a permanent spot among the top four; if not, it gets dropped back to being choked, meaning it receives nothing. Everyone outside the top four plus the one probe is choked.

> **Analogy — The Networking Reciprocity Rule:** Tit-for-tat is the unwritten rule of professional favor-trading: keep doing favors for the people who reliably do favors back, but every so often take a chance and help a stranger, in case they turn out to be even more generous than your current circle. The optimistic unchoke matters because without it, the network would calcify — two strangers who would actually trade great rates with each other would never discover that fact, since neither would ever take the first risk-free step to find out.

|Mechanism|Purpose|Frequency|
|---|---|---|
|**Unchoking top 4**|Reward the peers currently giving the best rate, so good traders keep finding each other|Re-evaluated ~every 10 seconds|
|**Optimistic unchoke**|Randomly probe one new peer in case it offers a better rate, preventing stagnation into static cliques|Re-chosen ~every 30 seconds|
|**Choking**|Withhold chunks from everyone outside the top-4 + 1-probe set|All other neighbors, by default|

### Why the Incentive Design Actually Matters

Tit-for-tat can be gamed, but it's still the single most important reason BitTorrent works at scale. A P2P system isn't just a networking protocol — it's an economic system. Without a built-in incentive to upload, a rational selfish user would download and never reciprocate, a behavior usually called **free-riding** or **leeching**. Tit-for-tat makes selfishness and cooperation point the same direction: the only way to download fast is to also upload generously.

---

# 2.6.2 Distributed Hash Tables (DHT)

## Starting Point: A Centralized Key-Value Store

Strip away the distribution problem for a moment and think about what's actually being stored: a set of (key, value) pairs, looked up by key. The keys might be social security numbers paired with names, or content names (a song, an OS image) paired with the IP address of a peer that has a copy. Querying with a key returns whatever value or values match it.

> **Analogy — A Library Card Catalog:** A centralized key-value store is the old library card catalog — hand the librarian a "key" (a title or author), and she hands back the "value" (a shelf location). It works perfectly with one librarian and one cabinet. Building this with one central server is straightforward; the problem, exactly as with a single centralized DNS server in Section 2.5, is that it collapses under Internet-scale load and becomes a single point of failure.

The goal here is the distributed version: spread the (key, value) pairs across millions of peers so each one holds only a small slice, while still letting any peer query any key and get back the correct value, and insert new key-value pairs of its own. That structure is called a **Distributed Hash Table (DHT)**.

### A Concrete DHT Example — Finding the Latest Linux ISO

If Bob and Charlie each have a copy of the latest Linux distribution, the DHT holds two key-value pairs: (Linux, IP_Bob) and (Linux, IP_Charlie). Some single peer in the system — say, Dave — is responsible for the key "Linux" and stores those pairs. When Alice wants a copy, she queries the DHT with "Linux" as the key; the DHT determines that Dave is responsible for that key, contacts Dave, retrieves both IP addresses, and hands them back to Alice, who can now download from either Bob or Charlie directly.

### Why the Naïve Approach Fails

The most obvious design — scatter the pairs randomly across all peers, and have every peer keep a list of every other peer's address — falls apart immediately at scale. A query would need to be broadcast to every single peer in the system, and every peer would need to maintain a roster of every other peer, possibly millions of them.

> **Analogy — Shouting Into a Crowd:** This naïve scheme is standing in the middle of Times Square shouting a name, hoping the one person who knows the answer happens to shout back. It could work with ten people nearby. With a million people, it's pure noise — and every single question anyone ever asks gets broadcast to the entire crowd, every time.

### A Smarter Design: Hashing Peers and Keys Into the Same Number Space

The elegant fix is to give every peer an integer identifier in the range [0, 2^n − 1] for some fixed n, and require every key to live in that same numeric range. Real-world keys like "Led Zeppelin IV" or a social security number aren't naturally integers, so a **hash function** converts each key into an integer in that range. A hash function is many-to-one — different inputs can in theory collide to the same output — but the odds of that happening are made astronomically small by design. From here on, "the key" really means the hashed integer, not the original string.

> **Analogy — Coat Check Tickets:** A hash function works like a coat-check counter. Hand over your coat (the key, e.g. "Led Zeppelin IV"), and instead of writing your name, the attendant slaps a numbered ticket on it — say, #47. Any attendant in the building can now look at #47 and know exactly which rack to check, without ever needing your name. Two coats could in theory get the same number — a hash collision — but a well-designed system makes that vanishingly unlikely.

### Assigning Keys to Peers — "Closest Successor"

With every peer and every key living in the same integer range, the natural rule is: assign each (key, value) pair to the peer whose identifier is numerically closest to the key, specifically the closest one _at or above_ it — its closest successor.

For example, with n = 4 (so identifiers run from 0 to 15) and eight peers present at identifiers 1, 3, 4, 5, 8, 10, 12, and 15, storing the pair (11, Johnny Wu) goes to peer 12, since 12 is the closest successor of 11. If a key exactly matches a peer's identifier, it's stored there directly; if a key is larger than every peer identifier present, the numbering wraps around modulo 2^n and the pair lands on the peer with the smallest identifier instead — picture the number line bent into a circle.

This rule only works cheaply if a peer can actually figure out who the closest successor is, and that's the catch: if every peer tracked every other peer's ID and address, it could compute the closest successor locally — but that's exactly the same all-peers-know-all-peers problem that made the naïve design unscalable in the first place.

## Circular DHT — Organizing Peers Without Global Knowledge

The fix is to arrange the peers themselves into a ring, where each peer tracks only its immediate successor and immediate predecessor around that ring (modulo 2^n).

![[Pasted image 20260618153018.png]] _(Figure 2.27 — (a) A circular DHT with peers at identifiers 1, 3, 4, 5, 8, 10, 12, 15, where peer 3 is trying to find who's responsible for key 11. (b) The same ring with shortcut links added for faster routing)_

In this arrangement, peer 5 knows the address of peers 8 and 4 — its successor and predecessor — and nothing more. It has no idea who else exists in the DHT.

### Overlay Networks — A Network Drawn on Top of a Network

This ring is a specific case of an **overlay network**: an abstract logical network of peers, layered on top of the actual physical "underlay" network of routers and links. The connections in the overlay aren't physical wires — they're virtual relationships between pairs of peers. A ring of eight peers like Figure 2.27(a) has eight such overlay links; adding shortcuts as in 2.27(b) brings that up to sixteen. Each individual overlay link is, underneath, riding on top of many real physical hops.

> **Analogy — A Group Chat vs. the Phone Network:** A WhatsApp group of eight friends is the overlay — each friend only directly "knows" the others in that chat. But every message actually travels through dozens of physical cell towers and ISP routes underneath, none of which the friends ever think about. The overlay is the map of who-talks-to-whom; the underlay is the plumbing that makes it physically work.

### Worked Routing Example — Resolving Key 11 in the Circular DHT

```
Peer 3 asks: "Who is responsible for key 11?" → sends clockwise

Peer 4 receives it → not responsible (its successor, 5, is closer to 11)
                   → passes the message along to peer 5

Peer 5 receives it → not responsible → passes along to peer 8

Peer 8 receives it → not responsible → passes along to peer 10

Peer 10 receives it → not responsible → passes along to peer 12

Peer 12 receives it → is the closest peer to key 11
                     → sends a reply directly back to peer 3,
                       saying "peer 12 is responsible for key 11"
```

Every peer along the way only needs to check one thing: is its own successor closer to the target key than it is? If yes, the message gets passed on; if a peer turns out to be the closest match, it replies directly back to the original asker instead of continuing the relay.

> **Analogy — Passing a Note Around a Circular Classroom:** Eight students sit in a circle, each only allowed to talk to the classmate immediately to their left and right. To find who's holding a particular book, one student writes "Who has book #11?" and passes it clockwise. Every student who receives it checks whether they or their right-hand neighbor is closer, and just passes it on if it isn't them. Eventually the note reaches the right student, who replies directly back to whoever asked — instead of every student shouting at once.

### The Fundamental Tradeoff: Neighbors-Per-Peer vs. Messages-Per-Query

Tracking only two neighbors per peer is wonderfully cheap in memory, but it comes at a real cost: resolving a query in the worst case may require forwarding the message all the way around the ring, an average of N/2 messages per query. Compare that to a mesh design where every peer knows every other peer directly — that resolves any query in a single message, but each peer's address book now has to scale with the entire network.

|Overlay Design|Neighbors tracked per peer|Messages per query (average)|Tradeoff|
|---|---|---|---|
|**Mesh** (everyone knows everyone)|N (every other peer)|1|Fast lookups, but each peer's address book grows with the whole network — unscalable for millions of peers|
|**Plain circular**|2 (successor + predecessor)|N/2|Tiny memory footprint per peer, but slow lookups at scale|
|**Circular + shortcuts**|O(log N)|O(log N)|The sweet spot — both costs grow only logarithmically with N|

### Adding Shortcuts

The refinement is to keep the ring as the backbone but give each peer a small number of additional long-range "shortcut" links scattered around the circle. When a peer receives a query, it forwards it to whichever neighbor — successor or shortcut — is closest to the target key. In the shortcut version of Figure 2.27(b), peer 4 can jump straight to its shortcut neighbor at 10 instead of crawling through 5 and 8 first, cutting the number of hops dramatically for the same query.

> **Analogy — Express Trains vs. Local Trains:** Plain ring routing is a subway line where every train stops at every station — reliable, but slow across a long line. Shortcuts add an express train that skips ahead to major stations and only switches to local stops near the destination. The full local line still exists as a fallback, but most journeys get much faster.

Choosing the right number of shortcuts, and which peers should hold them, is an active design question, but it's been shown that a DHT can be built so that both the number of neighbors per peer and the number of messages per query scale as O(log N) — a satisfying middle ground between a fully meshed overlay and a bare ring.

---

## Peer Churn — Designing for a World Where Anyone Can Vanish

Peers in a P2P system come and go without warning, so the overlay has to keep working even as that happens. The standard hardening is redundancy: each peer tracks not just its first successor but its second successor as well, and periodically pings both to confirm they're still alive.

### Worked Example 1 — A Peer Leaves Abruptly

Suppose peer 5 in the ring drops offline without warning. The two peers immediately preceding it — 4 and 3 — eventually notice this because peer 5 stops responding to pings, and they need to repair their own state:

```
Peer 4 detects peer 5 is unreachable (failed ping)

Step 1 → Peer 4 replaces its first successor (peer 5) with
          its second successor (peer 8)

Step 2 → Peer 4 then asks its new first successor (peer 8) for
          the identifier and IP address of its immediate
          successor (peer 10). Peer 4 then makes peer 10
          its new second successor.
```

> **Analogy — Emergency Contact Chains:** This is the logic of a relay-race baton handoff. Each runner doesn't just know who's directly ahead — they also know who's two positions ahead, in case the runner directly in front trips and drops out. If that happens, the next runner simply steps forward, takes the baton, and asks that person who's now running after them, repairing the chain on the fly without stopping the race.

### Worked Example 2 — A New Peer Joins

Suppose a peer with identifier 13 wants to join, knowing only the existence of peer 1. Peer 13 sends a message asking what its predecessor and successor should be; that message gets relayed around the ring until it reaches peer 12, which realizes it will become 13's predecessor and that its current successor, peer 15, will become 13's successor. Peer 12 sends that predecessor/successor information back to peer 13, which can now slot itself into the ring between 12 and 15 — without any single peer ever needing a complete picture of the network.

> **Analogy — Inserting Yourself Into a Conga Line:** Joining a circular DHT is like a new dancer inserting themselves into a moving conga line, knowing only that one dancer near the front exists. They relay a question down the line — who should be in front of and behind me — until the right gap is found, then simply tap into place between those two dancers, who now both know about the newcomer.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Malware distribution via torrents**|Pirated software, "cracked" games, and ISO/ROM torrents are a common malware delivery vector — an attacker repackages a popular file with a trojan or ransomware payload and seeds it into a popular swarm, relying on the file's popularity to spread the infection widely and fast|Hash-verify downloads against known-good checksums (SHA-256); prefer signed packages from official repositories over P2P swarms for executable content; sandbox or scan anything pulled from an untrusted torrent before running it|
|**Free-riding / leeching**|A selfish peer disables uploading entirely to consume bandwidth without contributing back, degrading the swarm's overall health|The tit-for-tat choking algorithm structurally punishes this — a peer that never uploads simply never gets unchoked by anyone else, so its download speed collapses naturally without needing a central enforcer|
|**Sybil attacks on a DHT**|An attacker creates a large number of fake peer identities and floods the identifier space around a specific target key, trying to make the attacker's own peers responsible for it and able to return forged values|Identity costs (e.g., requiring proof-of-work or a verified bootstrap node before issuing a peer ID), reputation systems, and cross-checking responses against multiple independent peers rather than trusting the first reply|
|**Eclipse attacks**|A targeted variant of the Sybil attack that specifically surrounds a victim peer's successor/predecessor and shortcut links, isolating it and feeding it only attacker-controlled routing information|Diverse, randomized neighbor and shortcut selection, so an attacker can't predictably control which peers end up adjacent to a given victim in the overlay|
|**DHT poisoning / index poisoning**|An attacker inserts bogus (key, value) pairs — e.g. mapping a popular software's content-name key to IP addresses that don't actually serve it, or serve a malicious substitute — to disrupt or hijack content discovery|Reputation or voting on inserted records; validating file hashes before trusting a (key, value) mapping rather than acting on the first result|
|**Tracker takedown as a legal chokepoint**|A centralized tracker is the one component of classic BitTorrent that does resemble infrastructure, making it a visible legal target — taking down a tracker historically crippled any torrent that depended on it|Modern BitTorrent largely sidesteps this with trackerless operation, where peers discover each other through a DHT instead of a central tracker — the same self-scaling property that helps performance also helps evade centralized takedown|
|**Botnet command-and-control over P2P/DHT**|Real malware families (the Storm worm, Conficker) have used P2P-style overlays for command-and-control precisely because there's no single server to seize|Behavioral and traffic analysis to detect anomalous DHT-style traffic on a host; network segmentation and egress filtering to keep infected hosts from joining such overlays in the first place|
|**Loss of anonymity / IP exposure**|Because file distribution is fundamentally peer-to-peer, every participant in a swarm can see the real IP addresses of every other participant trading that file — this is exactly how anti-piracy firms identify individual downloaders and send takedown notices|VPNs or proxies to mask the originating IP before joining a swarm (legality and terms-of-service implications vary by jurisdiction and platform — this is a technical point, not legal advice)|

---

## Questions I Still Have

- [ ] Tit-for-tat is known to be gameable — what do real exploit techniques (e.g. the BitTyrant client) actually do to manipulate the choking algorithm, and why hasn't the protocol been redesigned to close that loophole?
- [ ] How does a trackerless BitTorrent swarm bootstrap a brand-new peer's very first connection if there's no tracker to ask — purely via the DHT, magnet links, and a hardcoded list of well-known bootstrap nodes?
- [ ] Is the O(log N) circular-DHT-with-shortcuts design described here the same structure as Chord or Kademlia (the DHTs real BitTorrent clients actually use today), or a simplified teaching model?
- [ ] Peer-churn handling here only walks through repairing successor pointers — what's the symmetric process for repairing predecessor pointers, and does it need the same number of round trips?
- [ ] How do modern decentralized systems (IPFS, blockchain peer discovery) build on this same DHT model, and what do they change to handle adversarial peers more robustly than this basic, Sybil-naive design?
- [ ] Given that swarm participants can see each other's IP addresses by design, how do anonymity-focused P2P networks (I2P, Tor hidden services) restructure the overlay to prevent that exposure entirely?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**P2P (Peer-to-Peer)**|An architecture where intermittently-connected, user-owned hosts communicate directly with each other, with minimal reliance on always-on infrastructure servers|
|**Peer**|An ordinary user-controlled host that both consumes and supplies data in a P2P system|
|**Self-scaling**|The property that a system's total capacity grows automatically as more users join, rather than placing all the added burden on a fixed server|
|**Distribution time (D)**|The time required to get a complete copy of a file to all N peers requesting it|
|**Torrent**|The full collection of peers participating in distributing one particular file|
|**Chunk**|A fixed-size piece of a file (typically 256 KB) traded between peers|
|**Tracker**|An infrastructure node that registers peers in a torrent and tracks which are currently active|
|**Neighboring peers**|The subset of peers a given peer has an active TCP connection with|
|**Rarest first**|BitTorrent's chunk-selection strategy: request the chunks least common among your neighbors first, to keep all chunks well distributed|
|**Tit-for-tat / choking algorithm**|BitTorrent's incentive mechanism: reciprocate uploads only to the neighbors currently giving you the best download rate, plus one randomly probed peer|
|**Unchoked**|A neighbor currently being sent chunks — one of the top-4 reciprocal traders, or the current optimistic probe|
|**Optimistically unchoked**|A randomly chosen neighbor given chunks on a trial basis, to discover potentially better trading partners|
|**Free-riding / leeching**|Consuming a P2P system's resources without contributing back|
|**DHT (Distributed Hash Table)**|A P2P database where (key, value) pairs are spread across many peers, each holding only a small subset, with a defined rule for which peer is responsible for which key|
|**Hash function**|A many-to-one function mapping an arbitrary key into a fixed-range integer identifier, used to place both peers and keys onto the same numeric space|
|**Closest successor**|The rule for assigning a (key, value) pair to a peer: the peer whose identifier is numerically closest to (and at or above) the key's hashed identifier|
|**Circular DHT**|A DHT overlay where peers are arranged in a ring, each tracking only its immediate successor and predecessor|
|**Overlay network**|An abstract, logical network of virtual links between peers, layered on top of the actual physical "underlay" network|
|**Shortcuts**|Additional long-range links in a circular DHT that let queries skip ahead instead of crawling node-by-node, cutting lookup cost to O(log N)|
|**Peer churn**|The constant, unpredictable joining and leaving of peers, which any robust DHT design must tolerate|
|**Sybil attack**|An attack where one adversary creates many fake peer identities to gain disproportionate influence over a P2P overlay or DHT|
|**Eclipse attack**|An attack surrounding a specific victim peer's neighbors with attacker-controlled peers, isolating it from honest routing information|

---

## Related Concepts

- 

---

→ Next: [[2.7 - Video Streaming and Content Distribution Networks]]