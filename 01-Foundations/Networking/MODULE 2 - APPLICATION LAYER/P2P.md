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

> **One-Line Summary:** P2P architecture removes the always-on infrastructure server from the picture entirely — ordinary user machines ("peers") directly serve each other, which makes the system **self-scaling** (more users automatically means more total capacity, not just more load), but also makes it harder to control, secure, and police, since there is no single throat to choke.

---

## Core Idea

Every application studied so far in this chapter — the Web, e-mail, DNS — is fundamentally **client-server**: a small number of powerful, always-on machines (Web servers, mail servers, DNS servers) sit at the center, and a much larger number of clients connect to them, take what they need, and disconnect. The server does essentially all the heavy lifting; the clients are mostly passive consumers.

> Recall from Section 2.1.1 that with a P2P architecture, there is **minimal (or no) reliance on always-on infrastructure servers**. Instead, pairs of intermittently connected hosts, called peers, communicate directly with each other.

The single most important structural fact about P2P is this: the peers are not owned by a service provider, but are instead desktops and laptops controlled by users. Nobody is paying a cloud bill to keep your laptop running 24/7 — you turn it on and off whenever you like, which is precisely what "intermittently connected" means.

> **Analogy — Potluck vs. Catered Dinner:** A client-server application is like a catered party: one company (the caterer) buys all the ingredients, cooks everything in a big professional kitchen, and serves every single guest from that one kitchen. If 10x more guests show up, the caterer needs 10x more ovens, staff, and ingredients — the burden scales entirely with the host. A P2P application is a potluck: every guest who shows up _also_ brings a dish to share. The more guests arrive, the more food arrives too. The "serving capacity" of the party grows automatically with the number of people attending it — nobody needs to rent a bigger kitchen.

|Property|Client-Server|Peer-to-Peer|
|---|---|---|
|**Who serves data**|A small number of dedicated, always-on servers|Every peer can both consume and supply data|
|**Ownership of infrastructure**|Service provider owns and maintains the servers|Ordinary users own the peers (their own PCs)|
|**Availability**|Servers are always-on by design|Peers are **intermittently connected** — here today, offline tomorrow|
|**How load grows with users**|Server bears the entire burden; more users = more server strain|Self-scaling — more users bring more combined capacity|
|**Examples covered so far**|Web (HTTP), e-mail (SMTP/IMAP), DNS|File distribution (BitTorrent), Distributed Hash Tables (DHT)|

This section examines two P2P applications that showcase this self-scalability in two very different ways:

1. **File distribution** — getting one large file out to a large number of hosts (BitTorrent is the worked example).
2. **A distributed database** spread across millions of peers, where each peer holds only a tiny slice of the whole — this is the **Distributed Hash Table (DHT)**.

---

# 2.6.1 P2P File Distribution

## The Problem: Getting One Big File to Everyone

> We begin our foray into P2P by considering a very natural application, namely, distributing a large file from a single server to a large number of hosts (called peers).

Think of a new Linux distro ISO, a major OS patch, an MP3 album, or an MPEG video file. In the client-server world, this is a brutal problem: in client-server file distribution, the server must send a copy of the file to each of the peers — placing an enormous burden on the server and consuming a large amount of server bandwidth. If a million people want the file, the server's access link must somehow push the _entire file_ a million separate times.

P2P flips this completely on its head: in P2P file distribution, each peer can redistribute any portion of the file it has received to any other peers, thereby assisting the server in the distribution process. As of 2012, the most popular P2P file distribution protocol is BitTorrent.

> **Analogy — Photocopying a Handout:** Imagine a professor has one master copy of a 500-page handout and 60 students need it. Client-server is the professor personally photocopying and handing a full copy to every single student one at a time — the professor (the "server") does all 60 copy-jobs alone. P2P is the professor photocopying the first 10 pages for the first few students, who immediately start photocopying _those_ pages for other students while the professor moves on to the next chunk. Within minutes, dozens of students are simultaneously feeding each other pages, and the professor's photocopier is only one of many machines now doing the work.

## Quantifying Scalability — How Long Does Distribution Actually Take?

To compare the two architectures rigorously rather than just intuitively, the textbook builds a simple mathematical model.

**Variables used throughout this model:**

|Symbol|Meaning|
|---|---|
|**F**|Size of the file to be distributed, in bits|
|**N**|Number of peers that want a copy of the file|
|**u_s**|Upload rate of the _server's_ access link|
|**u_i**|Upload rate of the _i_-th peer's access link|
|**d_i**|Download rate of the _i_-th peer's access link|
|**d_min**|The _slowest_ download rate among all peers, i.e. `min{d_1, d_2, ..., d_N}`|

> **Simplifying assumption used throughout:** the Internet core has abundant bandwidth, so **all bottlenecks are in the access links** (the "last mile" connecting server/peers to the Internet) — not inside the network itself. Also, no other applications are competing for bandwidth at the same time.

![[Screenshot_2026-06-18_144143.png]] _(Figure 2.24 — An illustrative file distribution problem: one server holding file F connects to the Internet at upload rate u_s; N peers connect with their own upload rates u_1...u_N and download rates d_1...d_N)_

### Client-Server Distribution Time (D_cs)

In the client-server case, the peers contribute nothing — the server alone must push every bit to every peer. Two independent lower bounds apply simultaneously:

- The server must transmit one copy of the file to each of the N peers. Thus the server must transmit NF bits. Since the server's upload rate is u_s, the time to distribute the file must be at least NF/u_s.
- d_min denotes the download rate of the peer with the lowest download rate, that is, d_min = min{d_1, d_2, ..., d_N}. The peer with the lowest download rate cannot obtain all F bits of the file in less than F/d_min seconds. Thus the minimum distribution time is at least F/d_min.

Combining both constraints (the actual time can never be faster than _either_ bottleneck) gives:

```
D_cs = max { NF / u_s ,  F / d_min }                       (Equation 2.1)
```

> **The key takeaway:** for large N, the **first term dominates** (`NF/u_s`), so distribution time grows **linearly with N**. Double the number of peers wanting the file, and the server-bound delivery time roughly doubles too — the server's single upload pipe is the permanent bottleneck, no matter how many fast-downloading peers show up.

### P2P Distribution Time (D_p2p)

This is more subtle, because now peers redistribute pieces of the file to each other as they receive them, so the "who serves whom" pattern can vary. Three independent lower bounds apply:

- At the beginning of the distribution, only the server has the file. To get this file into the community of peers, the server must send each bit of the file at least once into its access link. Thus, the minimum distribution time is at least F/u_s. (Unlike client-server, a bit sent once by the server doesn't need to be re-sent by the server — peers can re-circulate it among themselves.)
- As before, the slowest downloader caps things at **F/d_min**.
- Observe that the total upload capacity of the system as a whole is equal to the upload rate of the server plus the upload rates of each of the individual peers, that is, u_total = u_s + u_1 + ... + u_N. The system must deliver (upload) F bits to each of the N peers, thus delivering a total of NF bits. This cannot be done at a rate faster than u_total. Thus, the minimum distribution time is also at least NF/(u_s + u_1 + ... + u_N).

Combining all three constraints:

```
D_p2p = max { F/u_s ,  F/d_min ,  NF / (u_s + u_1 + ... + u_N) }      (Equation 2.2 → refined as 2.3)
```

![[Screenshot_2026-06-18_144203.png]] _(Figure 2.25 — Distribution time for P2P and client-server architectures: client-server grows linearly without bound as N increases, while the P2P curve flattens out and stays under one hour for any N)_

> **Why the P2P term is so much friendlier:** in client-server, the file has to be uploaded _N separate times by one machine_. In P2P, the same `NF` bits of total delivery work get spread across the **combined upload capacity of every peer plus the server** — and every new peer that joins doesn't just add demand, it also adds its own upload rate to the denominator. This is the literal mathematical definition of _self-scaling_.

### Worked Numeric Example (mirrors Figure 2.25's assumptions)

Assume `F/u_s = 1 hour` (the server alone could push the whole file out in an hour to one recipient), `u_s = 10u` (the server is 10x faster than any single peer's upload), and every peer has the same upload rate `u`, with download rates fast enough to never be the bottleneck (so `d_min` drops out of the comparison).

|N (peers)|D_cs (client-server)|D_p2p|
|---|---|---|
|5|30 min|20 min|
|10|1 hr|30 min|
|20|2 hr|40 min|
|30|3 hr|45 min|
|1,000|~100 hr (4+ days)|~59.4 min|

> **What this table is really saying:** as N gets large, client-server distribution time explodes without bound — a thousand-fold increase in N (a thousand peers to a million) increases distribution time by roughly a thousand-fold too, exactly as the textbook warns. P2P distribution time, by contrast, **converges toward F/u_s (one hour) and never exceeds it** — it is mathematically impossible for the P2P scheme to take longer than the time the server alone would need to push out a single copy, no matter how many million peers join.

---

## BitTorrent — The Protocol That Made P2P File Sharing Mainstream

> BitTorrent is a popular P2P protocol for file distribution. In BitTorrent lingo, the collection of all peers participating in the distribution of a particular file is called a torrent.

|BitTorrent Term|Meaning|
|---|---|
|**Torrent**|The collection of all peers participating in the distribution of a particular file|
|**Chunk**|Peers in a torrent download equal-size chunks of the file from one another, with a typical chunk size of 256 KBytes|
|**Tracker**|An infrastructure node that registers peers joining a torrent and keeps track of which peers are currently participating|
|**Neighboring peers**|The subset of peers a given peer has succeeded in establishing a TCP connection with, out of a larger randomly-given list|

### How a Peer Joins and Trades

When a peer first joins a torrent, it has no chunks. Over time it accumulates more and more chunks. While it downloads chunks it also uploads chunks to other peers. A peer is even free to leave the moment it has the complete file (selfishly) or stick around afterward purely to keep seeding chunks to others (altruistically).

![[Screenshot_2026-06-18_144220.png]] _(Figure 2.26 — File distribution with BitTorrent: Alice obtains a list of peers from the tracker, establishes TCP connections with a subset of them ("neighboring peers"), and trades chunks back and forth across the mesh)_

> **Worked example — Alice joins a torrent:** when a new peer, Alice, joins the torrent, the tracker randomly selects a subset of peers (for concreteness, say 50) from the set of participating peers, and sends the IP addresses of these 50 peers to Alice. Alice attempts to open TCP connections with all 50 — whichever ones actually succeed become her **neighboring peers**, and that set keeps shifting as peers come and go.

### Two Big Decisions Every Peer Must Make

At any instant, a peer has _some_ chunks and is missing others, and its neighbors each have a different subset too. This creates exactly two strategic questions:

#### 1. Which chunk to request next? → **Rarest First**

> In deciding which chunks to request, Alice uses a technique called rarest first. The idea is to determine, from among the chunks she does not have, the chunks that are the rarest among her neighbors and then request those rarest chunks first. In this manner, the rarest chunks get more quickly redistributed, aiming to (roughly) equalize the numbers of copies of each chunk in the torrent.

> **Analogy — Trading Cards:** Imagine a classroom trading rare baseball/Pokémon cards. If everyone already has five copies of "Common Card #1" but only one kid in the whole class has "Legendary Card #99," that legendary card is one accidental spill of a backpack away from disappearing from circulation entirely. Smart traders prioritize getting _and_ spreading the rarest cards first, so that even if one kid stays home sick, the card is no longer a single point of failure. BitTorrent's rarest-first rule does exactly this for file chunks — it deliberately spreads scarce data wide _before_ a peer holding a rare chunk has a chance to disconnect and take it with them.

#### 2. Whose requests to honor? → **Tit-for-Tat (the Choking Algorithm)**

> BitTorrent uses a clever trading algorithm. The basic idea is that Alice gives priority to the neighbors that are currently supplying her data at the highest rate. Specifically, for each of her neighbors, Alice continually measures the rate at which she receives bits and determines the four peers that are feeding her bits at the highest rate. She then reciprocates by sending chunks to these same four peers. Every 10 seconds, she recalculates the rates and possibly modifies the set of four peers. In BitTorrent lingo, these four peers are said to be unchoked.

> Every 30 seconds, she also picks one additional neighbor at random and sends it chunks — this peer is said to be optimistically unchoked. Because Alice is sending data to Bob, he may become one of Bob's top four uploaders, in which case Bob would start to send data to Alice. All other neighboring peers besides these five peers (four "top" peers and one probing peer) are "choked" — that is, they do not receive any chunks from Alice.

> The incentive mechanism for trading just described is often referred to as tit-for-tat.

> **Analogy — The Networking Reciprocity Rule:** Tit-for-tat is exactly the unwritten rule of professional favor-trading: "I'll keep doing favors for the people who reliably do favors back for me — but every so often, I'll take a chance and help out a stranger, just in case they turn out to be even more generous than my current circle." The "optimistic unchoke" is the crucial bit, because without it, the network would calcify: two strangers who'd actually trade great rates with each other would never discover that fact, since neither would ever take the first risk-free step to find out.

|Mechanism|Purpose|Frequency|
|---|---|---|
|**Unchoking top 4**|Reward the peers currently giving you the best rate, so good traders keep finding each other|Re-evaluated every 10 seconds|
|**Optimistic unchoke**|Randomly probe one new peer in case they offer an even better rate, preventing the network from stagnating into static cliques|Re-chosen every 30 seconds|
|**Choking**|Refuse to send chunks to anyone outside the top-4 + 1-probe set|All other neighbors, by default|

### Why the Incentive Design Actually Matters

> It has been shown that this incentive scheme can be circumvented. Nevertheless, the BitTorrent ecosystem is wildly successful, with millions of simultaneous peers actively sharing files in hundreds of thousands of torrents. If BitTorrent had been designed without tit-for-tat (or a variant), but otherwise exactly the same, BitTorrent would likely not even exist now, as the majority of users would have been freeriders.

This is the single biggest lesson of the BitTorrent design: a P2P system is not just a networking protocol, it's an **economic system**. Without a built-in incentive to upload, rational selfish users would all just download and never reciprocate — a phenomenon called **free-riding** or **leeching**. Tit-for-tat makes selfishness and cooperation point in the same direction: the _only_ way to get fast downloads is to also upload generously.

---

# 2.6.2 Distributed Hash Tables (DHT)

## Starting Point: A Centralized Key-Value Store

Before distributing anything, it helps to understand the _centralized_ version of the problem being solved. Let's begin by describing a centralized version of this simple database, which will simply contain (key, value) pairs.

> For example, the keys could be social security numbers and the values could be the corresponding human names; in this case, an example key-value pair is (156-45-7081, Johnny Wu). Or the keys could be content names (e.g., names of movies, albums, and software), and the value could be the IP address at which the content is stored; in this case an example key-value pair is (Led Zeppelin IV, 128.17.123.38).

> We query the database with a key. If there are one or more key-value pairs in the database that match the query key, the database returns the corresponding values.

> **Analogy — A Library Card Catalog:** A centralized key-value store is exactly like the old library card catalog: you hand the librarian a "key" (a book title or author name), and she hands back the "value" (the shelf location). It works beautifully as long as there's one librarian and one catalog cabinet. Building such a database is straightforward with a client-server architecture that stores all the (key, value) pairs in one central server. But just like the single-centralized-DNS-server idea from Section 2.5, this collapses under Internet-scale load and becomes a single point of failure.

So in this section, we'll instead consider how to build a distributed, P2P version of this database that will store the (key, value) pairs over millions of peers. In the P2P system, each peer will only hold a small subset of the totality of the (key, value) pairs. We'll allow any peer to query the distributed database with a particular key. The distributed database will then locate the peers that have the corresponding (key, value) pairs and return the key-value pairs to the querying peer. Any peer will also be allowed to insert new key-value pairs into the database. Such a distributed database is referred to as a distributed hash table (DHT).

### A Concrete DHT Example — Finding the Latest Linux ISO

> In this case, a key is the content name and the value is the IP address of a peer that has a copy of the content. So, if Bob and Charlie each have a copy of the latest Linux distribution, then the DHT database will include the following two key-value pairs: (Linux, IP_Bob) and (Linux, IP_Charlie). More specifically, since the DHT database is distributed over the peers, some peer, say Dave, will be responsible for the key "Linux" and will have the corresponding key-value pairs.

> Now suppose Alice wants to obtain a copy of Linux. She first needs to know which peers have a copy of Linux before she can begin to download it. To this end, she queries the DHT with "Linux" as the key. The DHT then determines that the peer Dave is responsible for the key "Linux." The DHT then contacts peer Dave, obtains from Dave the key-value pairs (Linux, IP_Bob) and (Linux, IP_Charlie), and passes them on to Alice. Alice can then download the latest Linux distribution from either IP_Bob or IP_Charlie.

### Why the Naïve Approach Fails

> One naïve approach to building a DHT is to randomly scatter the (key, value) pairs across all the peers and have each peer maintain a list of the IP addresses of all participating peers. In this design, the querying peer sends its query to all other peers, and the peers containing the (key, value) pairs that match the key can respond with their matching pairs. Such an approach is completely unscalable, of course, as it would require each peer to not only know about all other peers (possibly millions of such peers!) but even worse, have each query sent to all peers.

> **Analogy — Shouting Into a Crowd:** This naïve scheme is like trying to find one specific person's phone number by standing in the middle of Times Square and shouting their name, hoping the one person who happens to know it shouts back. It technically _could_ work with ten people nearby. With a million people, it's pure noise — and worse, every single question anyone ever asks gets broadcast to every single person in the crowd, all the time.

### A Smarter Design: Hashing Peers and Keys Into the Same Number Space

> We now describe an elegant approach to designing a DHT. To this end, let's first assign an identifier to each peer, where each identifier is an integer in the range [0, 2^n − 1] for some fixed n. Let's also require each key to be an integer in the same range.

But real-world keys (SSNs, content names like "Linux" or "Led Zeppelin IV") aren't naturally integers, so: we will use a hash function that maps each key (e.g., a social security number) to an integer in the range [0, 2^n − 1]. A hash function is a many-to-one function for which two different inputs can have the same output (same integer), but the likelihood of having the same output is extremely small. When we refer to "the key," we are referring to the hash of the original key. So, for example, if the original key is "Led Zeppelin IV," the key used in the DHT will be the integer that equals the hash of "Led Zeppelin IV." This is why "Hash" is used in the term "Distributed Hash Function."

> **Analogy — Coat Check Tickets:** Think of a hash function like the numbering system at a coat-check counter. You hand over your coat (the "key," e.g., "Led Zeppelin IV"), and instead of writing down your whole name, the attendant slaps a numbered ticket on it — say, ticket #47. Now any coat-check attendant in the building can look at the number 47 and know exactly which rack and peg to check, without ever needing to remember your name. Two different coats _could_, in extremely rare bad luck, get the same ticket number — that's a hash collision — but a well-designed system makes that astronomically unlikely.

### Assigning Keys to Peers — "Closest Successor"

> Now we consider the problem of storing the (key, value) pairs in the DHT. The central issue here is defining a rule for assigning keys to peers. Given that each peer has an integer identifier and that each key is also an integer in the same range, a natural approach is to assign each (key, value) pair to the peer whose identifier is closest to the key.

> To gain some insight here, let's take a look at a specific example. Suppose n = 4 so that all the peer and key identifiers are in the range [0, 15]. Further suppose that there are eight peers in the system with identifiers 1, 3, 4, 5, 8, 10, 12, and 15. Finally, suppose we want to store the (key, value) pair (11, Johnny Wu) in one of the eight peers. Using our closest convention, since peer 12 is the closest successor of key 11, we therefore store the pair (11, Johnny Wu) in the peer 12.

> **Tie-breaking rule:** if the key is exactly equal to one of the peer identifiers, we store the (key, value) pair in that matching peer; and if the key is larger than all the peer identifiers, we use a modulo-2^n convention, storing the (key, value) pair in the peer with the smallest identifier. (Picture the number line wrapping around into a circle — key 15 looking for a successor when the largest peer ID is 12 simply wraps back around to peer 1.)

> **Why this is still impractical as described so far:** if Alice were to keep track of all the peers in the system (peer IDs and corresponding IP addresses), she could locally determine the closest peer. But such an approach requires each peer to keep track of all other peers in the DHT — which is completely impractical for a large-scale system with millions of peers.

## Circular DHT — Organizing Peers Without Global Knowledge

> To address this problem of scale, let's now consider organizing the peers into a circle. In this circular arrangement, each peer only keeps track of its immediate successor and immediate predecessor (modulo 2^n).

![[Screenshot_2026-06-18_144245.png]] _(Figure 2.27 — (a) A circular DHT with peers at identifiers 1, 3, 4, 5, 8, 10, 12, 15; peer 3 wants to determine who is responsible for key 11. (b) The same circular DHT with shortcut links added for faster routing)_

> Each peer is only aware of its immediate successor and predecessor; for example, peer 5 knows the IP address and identifier for peers 8 and 4 but does not necessarily know anything about any other peers that may be in the DHT.

### Overlay Networks — A Network Drawn on Top of a Network

> This circular arrangement of the peers is a special case of an overlay network. In an overlay network, the peers form an abstract logical network which resides above the "underlay" computer network consisting of physical links, routers, and hosts. The links in an overlay network are not physical links, but are simply virtual liaisons between pairs of peers. In the overlay in Figure 2.27(a), there are eight peers and eight overlay links; in the overlay in Figure 2.27(b) there are eight peers and 16 overlay links. A single overlay link typically uses many physical links and physical routers in the underlay network.

> **Analogy — A Group Chat vs. the Phone Network:** Think of a WhatsApp group of eight friends as the "overlay." Each friend only directly "knows" the friends they're chatting with in that group — that's the logical overlay link. But under the hood, each message actually travels through dozens of physical cell towers, fiber routes, and ISPs (the "underlay") that the friends never think about at all. The overlay is the _map of who-talks-to-whom_; the underlay is the actual plumbing that makes it physically possible.

### Worked Routing Example — Resolving Key 11 in the Circular DHT

> Now suppose that peer 3 wants to determine which peer in the DHT is responsible for key 11. Using the circular overlay, the origin peer (peer 3) creates a message saying "Who is responsible for key 11?" and sends this message clockwise around the circle. Whenever a peer receives such a message, because it knows the identifier of its successor and predecessor, it can determine whether it is responsible for (that is, closest to) the key in question.

```
Peer 3 asks: "Who is responsible for key 11?" → sends clockwise

Peer 4 receives it → not responsible (its successor, 5, is closer to 11)
                   → passes the message along to peer 5

Peer 5 receives it → not responsible → passes along to peer 8

Peer 8 receives it → not responsible → passes along to peer 10

Peer 10 receives it → not responsible → passes along to peer 12

Peer 12 receives it → IS the closest peer to key 11
                     → sends a reply directly back to peer 3,
                       saying "peer 12 is responsible for key 11"
```

> So, for example, when peer 4 receives the message asking about key 11, it determines that it is not responsible for the key (because its successor is closer to the key), so it just passes the message along to peer 5. This process continues until the message arrives at peer 12, who determines that it is the closest peer to key 11. At this point, peer 12 can send a message back to the querying peer, peer 3, indicating that it is responsible for key 11.

> **Analogy — Passing a Note Around a Circular Classroom:** Imagine eight students seated in a circle, each only allowed to talk to the classmate immediately to their left and right. To find out who's holding a particular library book, one student writes "Who has book #11?" on a note and passes it clockwise. Every student who receives it checks "is it me, or is the person to my right closer?" and just passes it on if it isn't them. Eventually the note reaches the right student, who replies directly back to whoever originally asked — instead of every student standing up and shouting at once.

### The Fundamental Tradeoff: Neighbors-Per-Peer vs. Messages-Per-Query

> The circular DHT provides a very elegant solution for reducing the amount of overlay information each peer must manage. In particular, each peer needs only to be aware of two peers, its immediate successor and its immediate predecessor. But this solution introduces yet another problem. Although each peer is only aware of two neighboring peers, to find the node responsible for a key (in the worst case), all N nodes in the DHT will have to forward a message around the circle; N/2 messages are sent on average.

> Thus, in designing a DHT, there is tradeoff between the number of neighbors each peer has to track and the number of messages that the DHT needs to send to resolve a single query. On one hand, if each peer tracks all other peers (mesh overlay), then only one message is sent per query, but each peer has to keep track of N peers. On the other hand, with a circular DHT, each peer is only aware of two peers, but N/2 messages are sent on average for each query.

|Overlay Design|Neighbors tracked per peer|Messages per query (average)|Tradeoff|
|---|---|---|---|
|**Mesh** (everyone knows everyone)|N (every other peer)|1|Fast lookups, but each peer's address book grows with the whole network — unscalable for millions of peers|
|**Plain Circular**|2 (successor + predecessor)|N/2|Tiny memory footprint per peer, but painfully slow lookups at scale|
|**Circular + Shortcuts**|O(log N)|O(log N)|The sweet spot — both costs grow only logarithmically with N|

### Adding Shortcuts

> One refinement is to use the circular overlay as a foundation, but add "shortcuts" so that each peer not only keeps track of its immediate successor and predecessor, but also a relatively small number of shortcut peers scattered about the circle. Shortcuts are used to expedite the routing of query messages. Specifically, when a peer receives a message that is querying for a key, it forwards the message to the neighbor (successor neighbor or one of the shortcut neighbors) which is the closest to the key.

> Thus, in Figure 2.27(b), when peer 4 receives the message asking about key 11, it determines that the closest peer to the key (among its neighbors) is its shortcut neighbor 10 and then forwards the message directly to peer 10. Instead of crawling node-by-node (4→5→8→10→12), the message leapfrogs straight to 10, then on to 12 — far fewer hops for the same query.

> **Analogy — Express Trains vs. Local Trains:** Plain circular routing is like a subway line where every train stops at every single station — reliable, but slow across a long line. Adding shortcuts is like adding an express train that skips ahead to major stations and only switches to "local" stops near the destination. You still have the full local line as a fallback, but most journeys get dramatically faster.

> The next natural question is "How many shortcut neighbors should a peer have, and which peers should be these shortcut neighbors?" This question has received significant attention in the research community. Importantly, it has been shown that the DHT can be designed so that both the number of neighbors per peer as well as the number of messages per query is O(log N), where N is the number of peers. Such designs strike a satisfactory compromise between the extreme solutions of using mesh and circular overlay topologies.

---

## Peer Churn — Designing for a World Where Anyone Can Vanish

> In P2P systems, a peer can come or go without warning. Thus, when designing a DHT, we also must be concerned about maintaining the DHT overlay in the presence of such peer churn.

To survive this, the design is hardened with a small amount of redundancy: to handle peer churn, we will now require each peer to track (that is, know the IP address of) its first and second successors; for example, peer 4 now tracks both peer 5 and peer 8. We also require each peer to periodically verify that its two successors are alive (for example, by periodically sending ping messages to them and asking for responses).

### Worked Example 1 — A Peer Leaves Abruptly

> Let's now consider how the DHT is maintained when a peer abruptly leaves. For example, suppose peer 5 in Figure 2.27(a) abruptly leaves. In this case, the two peers preceding the departed peer (4 and 3) learn that 5 has departed, since it no longer responds to ping messages. Peers 4 and 3 then need to update their successor state information.

```
Peer 4 detects peer 5 is unreachable (failed ping)

Step 1 → Peer 4 replaces its first successor (peer 5) with
          its second successor (peer 8)

Step 2 → Peer 4 then asks its new first successor (peer 8) for
          the identifier and IP address of its immediate
          successor (peer 10). Peer 4 then makes peer 10
          its new second successor.
```

> Peer 4 replaces its first successor (peer 5) with its second successor (peer 8). Peer 4 then asks its new first successor (peer 8) for the identifier and IP address of its immediate successor (peer 10). Peer 4 then makes peer 10 second successor.

> **Analogy — Emergency Contact Chains:** This is exactly the logic of a phone tree or relay-race baton handoff. Each runner doesn't just know who's directly in front of them — they also know who's _two_ positions ahead, just in case the runner directly ahead trips and drops out. If that happens, you don't panic and stop the whole race; you simply step forward, take the baton from the next runner in line, and quietly ask _them_ who's now running after them, repairing the chain on the fly.

### Worked Example 2 — A New Peer Joins

> Let's say a peer with identifier 13 wants to join the DHT, and at the time of joining, it only knows about peer 1's existence in the DHT. Peer 13 would first send peer 1 a message, saying "what will be 13's predecessor and successor?" This message gets forwarded through the DHT until it reaches peer 12, who realizes that it will be 13's predecessor and that its current successor, peer 15, will become 13's successor. Next, peer 12 sends this predecessor and successor information to peer 13. Peer 13 can now join the ring between peer 12 and peer 15, slotting neatly into the circle without any peer needing global knowledge of the whole network.

> **Analogy — Inserting Yourself Into a Conga Line:** Joining a circular DHT is like a new dancer trying to insert themselves into a moving conga line at a wedding, knowing only that "someone near the front" exists. They ask that one known dancer to relay a question down the line — "who should be in front of and behind me, given my position number?" — until the right gap is found, and then they simply tap into place between those two dancers, who now both know about the newcomer.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Malware distribution via torrents**|Pirated software, "cracked" games, and ROM/ISO torrents are an extremely common malware delivery vector — an attacker repackages a popular file with a trojan or ransomware payload and seeds it into a popular torrent's swarm, relying on the file's popularity to spread the infection widely and fast|Hash-verify downloads against known-good checksums (SHA-256); rely on signed packages from official repositories rather than P2P swarms for executable content; sandbox/AV-scan anything pulled from an untrusted torrent before execution|
|**Free-riding / leeching**|A selfish peer disables uploading entirely (configures a client to download-only) to consume bandwidth without contributing any back, degrading the swarm's overall health|BitTorrent's **tit-for-tat choking algorithm** structurally punishes this — peers who don't upload simply don't get unchoked by anyone else, so their download speed collapses naturally, without needing a central enforcer|
|**Sybil attacks on a DHT**|An attacker creates a huge number of fake peer identities (Sybils) and floods the DHT's identifier space, attempting to surround a specific target key so that the attacker's own (malicious) peers become "responsible" for it — letting them return forged values for that key|Identity costs (e.g., requiring proof-of-work or a verified bootstrap node to obtain a peer ID), reputation systems, and cross-checking responses against multiple independent peers rather than trusting the first reply|
|**Eclipse attacks**|A variant of the Sybil attack where the attacker specifically surrounds a victim peer's neighbor set in the overlay (its successor/predecessor and shortcut links), effectively isolating the victim and feeding it only attacker-controlled routing information|Diverse, randomized neighbor/shortcut selection so an attacker cannot predictably control which peers end up adjacent to a given victim in the overlay|
|**DHT poisoning / index poisoning**|An attacker inserts large numbers of bogus (key, value) pairs — e.g., associating a popular movie or software's content-name key with IP addresses that don't actually serve that file, or that serve a malicious substitute — to disrupt or hijack content discovery|Reputation/voting on inserted records; requiring some peers to actually attempt a download and validate file hashes before trusting a (key, value) mapping|
|**Tracker takedown as a legal chokepoint**|Centralized trackers are the one component of classic BitTorrent that _does_ resemble infrastructure, making them a visible legal target — shutting down a tracker historically crippled torrents that depended on it|Modern BitTorrent largely sidesteps this with **trackerless** operation via a DHT (peers discover each other through the DHT instead of a central tracker), making the swarm far more resilient to takedown of any single node — the same self-scaling property that helps performance also helps evade centralized control|
|**Botnet command-and-control over P2P/DHT**|Real-world malware families (e.g., the Storm worm, Conficker) have used P2P-style overlay networks and DHT-like lookups specifically _because_ they have no single server to take down — law enforcement seizing one "node" doesn't kill the botnet|Behavioral/traffic analysis to detect anomalous DHT-style traffic patterns on a host; network segmentation and egress filtering to prevent infected hosts from participating in such overlays at all|
|**Loss of anonymity / IP exposure**|Because P2P file distribution is fundamentally peer-to-_peer_, every participant in a torrent's swarm can see the real IP addresses of every other participant trading that file — this is exactly how anti-piracy firms identify and send takedown/legal notices to individual downloaders|VPNs/proxies to mask the originating IP before joining a swarm (note: legality and terms-of-service implications vary and this is a factual/technical point, not legal advice)|

---

## Questions I Still Have

- [ ] The textbook shows BitTorrent's tit-for-tat "can be circumvented" — what do real exploit techniques (e.g., the BitTyrant client) actually do to game the choking algorithm, and why hasn't the protocol been patched to close that loophole?
- [ ] How exactly does a _trackerless_ BitTorrent swarm bootstrap a brand-new peer's very first connection if there's no tracker to ask — is it purely via the DHT, magnet links, and a hardcoded list of well-known bootstrap nodes?
- [ ] The chapter says DHT lookup and neighbor count can both be made O(log N) — is this the same underlying structure as **Chord** or **Kademlia** (the DHTs actually used by real BitTorrent clients today), or a simplified teaching model?
- [ ] For peer churn, the book only describes the leave/join procedure for the immediate successor side — what's the symmetric procedure for repairing the _predecessor_ pointers, and does it require the same number of round trips?
- [ ] How do modern decentralized systems (e.g., IPFS, blockchain peer discovery) build on this exact DHT model, and what do they change to handle adversarial peers more robustly than the textbook's basic Sybil-naive design?
- [ ] Given that swarm participants can see each other's IP addresses by design, how do anonymity-focused P2P networks (e.g., I2P, Tor's hidden services) restructure the overlay to prevent this kind of exposure entirely?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**P2P (Peer-to-Peer)**|An architecture where intermittently-connected, user-owned hosts ("peers") communicate directly with each other, with minimal reliance on always-on infrastructure servers|
|**Peer**|An ordinary user-controlled host (desktop/laptop) that both consumes and supplies data in a P2P system|
|**Self-scaling**|The property that a system's total capacity grows automatically as more users join, rather than placing all the added burden on a fixed server|
|**Distribution time (D)**|The time required to get a complete copy of a file to all N peers requesting it|
|**Torrent**|The full collection of peers participating in distributing one particular file|
|**Chunk**|A fixed-size piece of a file (typically 256 KB) that peers download from, and upload to, one another|
|**Tracker**|An infrastructure node that registers peers in a torrent and tracks which are currently active|
|**Neighboring peers**|The subset of peers a given peer has an active TCP connection with|
|**Rarest first**|BitTorrent's chunk-selection strategy: request the chunks that are least common among your neighbors first, to keep all chunks well distributed|
|**Tit-for-tat / choking algorithm**|BitTorrent's incentive mechanism: reciprocate uploads only to the neighbors currently giving you the best download rate, plus one randomly probed peer|
|**Unchoked**|A neighbor currently being sent chunks (one of the top-4 reciprocal traders, or the current optimistic probe)|
|**Optimistically unchoked**|A randomly chosen neighbor given chunks on a trial basis, to discover potentially better trading partners|
|**Free-riding / leeching**|Consuming a P2P system's resources (downloading) without contributing back (uploading)|
|**DHT (Distributed Hash Table)**|A P2P database where (key, value) pairs are spread across many peers, each peer holding only a small subset, with a defined rule for which peer is responsible for which key|
|**Hash function**|A many-to-one function mapping an arbitrary key into a fixed-range integer identifier, used to place both peers and keys onto the same numeric space|
|**Closest successor**|The rule used to assign a (key, value) pair to a peer: the peer whose identifier is numerically closest to (and at or above) the key's hashed identifier|
|**Circular DHT**|A DHT overlay where peers are arranged in a ring, each tracking only its immediate successor and predecessor|
|**Overlay network**|An abstract, logical network of virtual links between peers, layered on top of the actual physical ("underlay") network|
|**Shortcuts**|Additional long-range links added to a circular DHT so queries can skip ahead rather than crawling node-by-node, reducing lookup cost to O(log N)|
|**Peer churn**|The constant, unpredictable joining and leaving of peers in a P2P system, which any robust DHT design must tolerate|
|**Sybil attack**|An attack where one adversary creates many fake peer identities to gain disproportionate influence over a P2P overlay or DHT|
|**Eclipse attack**|An attack that surrounds a specific victim peer's neighbors with attacker-controlled peers, isolating it from honest routing information|

---

## Related Concepts

- [[2.5 - DNS - The Internet's Directory Service]] — another textbook example of a distributed database, but hierarchical/delegated rather than peer-organized; useful to contrast DNS's tree structure against the DHT's ring structure
- Chapter 7 — hash functions revisited in more depth (referenced directly by this section)
- Content Distribution Networks (CDNs) — a hybrid approach where infrastructure providers like Akamai replicate content geographically rather than relying purely on user peers

---

→ Next: [[2.7 - Video Streaming and Content Distribution Networks]]