---
title: VIDEO STREAMING & CDN
date: 2026-06-18
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.5 Video Streaming and Content Distribution Networks

> **One-Line Summary:** Streaming video is the single biggest consumer of Internet bandwidth, and the entire engineering challenge boils down to two problems solved together: encoding video so quality can flex with available bandwidth (DASH), and physically positioning copies of that video close to every user on Earth so no single link or data center becomes a bottleneck (CDNs).

---

## Core Idea: Two Problems, One Section

Streaming video — Netflix, YouTube, Prime Video, and similar services — now accounts for more than 65% of all global Internet traffic, and nearly 75% of mobile traffic. A single 7 Mbps video stream, watched for one hour, consumes about 3 gigabytes of data. Multiply that by hundreds of millions of simultaneous viewers worldwide, and "just put the video on a server and let people download it" breaks down almost immediately.

Two genuinely separate problems need solving, and the rest of this section is organized around them:

1. **The encoding problem** — users have wildly different available bandwidth (fiber vs. 3G vs. congested hotel WiFi). How do you serve the _right_ quality to each user, and keep adjusting as their bandwidth changes mid-stream? → Solved by **DASH** (2.5.2).
2. **The distribution problem** — even with perfect encoding, streaming from one data center to millions of simultaneous global viewers saturates links and creates huge delay for distant users. → Solved by **Content Distribution Networks** (2.5.3).

---

## 2.5.1 Internet Video — The Medium Itself

### What Is a Video, From a Networking Point of View?

Forget compression for a second. At the most basic level, **a video is just a sequence of still images ("frames"), displayed one after another at a constant rate** — typically 24 or 30 images per second. Each uncompressed image is an array of pixels, and each pixel is represented by some number of bits encoding its color and brightness (luminance).

> **Analogy — A Flipbook:** A video is exactly like a children's flipbook — a stack of slightly different drawings that, flipped quickly, _appear_ to move. The "frame rate" is just how fast you flip the pages. Networking doesn't care about the illusion of motion — it only cares how many bits it has to push through the network to deliver each page in time.

### The Property That Matters Most: Bit Rate

An uncompressed video would be far too large to stream practically. This is solved with **compression**: modern algorithms can compress a video down to nearly any target bit rate, with a direct trade-off:

> **higher bit rate → better quality, more bandwidth required** **lower bit rate → worse quality, less bandwidth required**

This trade-off is the foundation of everything else in this section. From a networking perspective, **a video's bit rate — not its resolution, codec, or length — is its single most important property.** Compressed Internet video today ranges from about 100 kbps (low quality) up to 25 Mbps (ultra-HD).

|Quality Tier|Typical Bit Rate|
|---|---|
|Low quality|~100 kbps|
|Standard / HD|1 – 5 Mbps|
|Ultra HD|up to 25 Mbps|

### Why Multiple Versions of the Same Video Exist

Because compression can target _any_ bit rate, providers don't encode a video once — they encode it **multiple times**, at multiple bit rates (say, 300 kbps, 1 Mbps, and 3 Mbps versions of the same movie). The user's device then picks whichever version best matches its current network conditions.

> **Analogy — Print Sizes of the Same Photo:** A photo print shop can print the same photograph as a postage-stamp thumbnail, a postcard, or a wall poster — same content, different amounts of "ink" (data) used. You pick the size that fits where you're going to display it (your bandwidth).

This idea — multiple encoded versions of one video, ready to be picked from — is the single most important prerequisite for understanding DASH next.

---

## 2.5.2 HTTP Streaming and DASH

### Step 1: Plain HTTP Streaming (the naive approach)

_(Quick refresher: an HTTP GET request asks a web server for a specific resource at a specific URL, over a TCP connection — covered earlier in this chapter.)_

In the simplest form of video streaming, **the video is just an ordinary file sitting on an HTTP server at a URL** — nothing special about it from the server's point of view.

```
1. User clicks "play"
2. Client opens a TCP connection to the server
3. Client sends an HTTP GET request for the video's URL
4. Server streams the video file back inside the HTTP response,
   as fast as the network and TCP will allow
5. Client collects incoming bytes into a buffer
6. Once buffer exceeds a threshold → playback begins
7. Player continuously grabs frames from the buffer, decompresses
   them, and displays them — while more bytes keep arriving
```

> **Why buffer before playing?** Network speed is never perfectly steady — there will be brief slowdowns. The buffer is a safety cushion: as long as it doesn't run dry, small network hiccups stay invisible to the viewer. This is the same reason a video call shows "buffering" before connecting.

### The Shortcoming of Plain HTTP Streaming

This was the original way YouTube streamed video, and it has one major flaw: **every client receives the exact same encoding of the video, no matter their current bandwidth — and no matter how that bandwidth changes mid-stream.** A user on fiber and a user on spotty hotel WiFi get treated identically. If bandwidth drops below the video's bit rate, the buffer runs dry and the video freezes.

This shortcoming is exactly what DASH was invented to fix.

### Step 2: DASH — Dynamic Adaptive Streaming over HTTP

**DASH** solves the "one-size-fits-all" problem by combining two ideas already introduced:

1. The video is encoded into **several different versions**, each at a different bit rate (2.5.1).
2. Each version is chopped into small **chunks** (segments), typically just a few seconds long.

```
Movie (2 hours)
  │
  ├── Version A (300 kbps)  → chunk1, chunk2, chunk3, ... chunkN
  ├── Version B (1 Mbps)    → chunk1, chunk2, chunk3, ... chunkN
  └── Version C (3 Mbps)    → chunk1, chunk2, chunk3, ... chunkN
```

Each version lives on the HTTP server under its own URL. A special file called the **manifest file** lists every available version, its bit rate, and its corresponding URL — the client's "menu," downloaded before any actual video data.

### How a DASH Client Actually Plays a Video

```
1. Client requests the manifest file from the server
2. Client parses the manifest → learns the available versions + URLs
3. Client requests chunk 1, often starting from a low-bitrate version
4. While downloading each chunk, the client MEASURES the achieved
   throughput (bytes received per second)
5. A rate-determination algorithm runs:
       - lots of video buffered + high measured bandwidth →
            request the NEXT chunk from a HIGHER-bitrate version
       - little video buffered + low measured bandwidth →
            request the NEXT chunk from a LOWER-bitrate version
6. Repeat from step 3 for every chunk, continuously re-adapting
```

> **Analogy — A Buffet, Not a Fixed Menu:** Plain HTTP streaming is a restaurant serving everyone the exact same fixed portion regardless of appetite. DASH is a buffet: the client looks at the manifest ("here's everything on offer, with prices"), and serves itself a few seconds' worth of food at a time — bigger portions (higher bitrate) when it's hungry and the kitchen is fast, smaller portions when the kitchen is slow. It re-evaluates its choice every single time it goes back.

This is why a DASH stream can visibly soften and then sharpen again mid-playback — that's the rate-adaptation algorithm reacting to a real bandwidth change, live.

### Why DASH Is "Just HTTP" — and Why That's a Feature

Every request in DASH — the manifest, and every chunk of every version — is an ordinary HTTP GET request, often using the **byte-range header** to ask for just part of a file. This means DASH needs **no special network infrastructure**: any standard web server, cache, firewall, or CDN already knows how to handle plain HTTP GET requests. DASH inherits HTTP's entire existing ecosystem for free.

|Property|Plain HTTP Streaming|DASH|
|---|---|---|
|Versions of video stored|One|Multiple, at different bit rates|
|Adapts to changing bandwidth|No|Yes — continuously, chunk by chunk|
|Special server software needed|No — ordinary HTTP server|No — still an ordinary HTTP server|
|Decision-maker|Server just sends as fast as it can|**Client** decides which chunk/version to request next|
|What's requested first|The whole movie file|A manifest file, then many small chunks|

---

## 2.5.3 Content Distribution Networks (CDNs)

### The Problem: Why One Data Center Isn't Enough

Imagine a video company takes the simplest possible approach: build **one massive data center**, store every video there, and stream directly from it to every user worldwide. This breaks in three distinct ways:

|Problem|Why It Happens|
|---|---|
|**1. Long, congested paths**|A client far from the data center has packets crossing many links and many ISPs. If _any single link_ on that path has less throughput than the video's bit rate, the user freezes — end-to-end throughput is governed by the _bottleneck_ link, not the average one.|
|**2. Wasted, repeated bandwidth**|A popular video gets sent, again and again, over the _same_ links to many different users. The video company pays its provider ISP for every one of those repeated transmissions.|
|**3. Single point of failure**|If the one data center — or just its connection to the Internet — goes down, the entire service goes down everywhere on Earth simultaneously. No redundancy.|

### The Solution: Content Distribution Networks

A **CDN** is a network of servers spread across many geographically distributed locations, each holding copies of content (video, images, documents, etc.). Instead of every user reaching back to one central server, **each user's request is redirected to a nearby CDN server** that already has a copy.

> **Analogy — Regional Warehouses vs. One Central Factory:** A single data center is like a factory shipping every order, to every customer on Earth, from one warehouse — slow and expensive for distant customers. A CDN is like a network of regional fulfillment centers: copies of popular products are pre-positioned close to customers, so most orders ship from a nearby warehouse instead of crossing the country.

### Private vs. Third-Party CDNs

|Type|Who Owns It|Example|
|---|---|---|
|**Private CDN**|The content provider itself|Netflix's "Open Connect"; Google's CDN for YouTube|
|**Third-party CDN**|A separate company, hired by multiple content providers|Akamai, Cloudflare, Amazon CloudFront|

### Two Server-Placement Philosophies

|Philosophy|Strategy|Trade-off|Pioneered By|
|---|---|---|---|
|**Enter Deep**|Push server clusters deep _inside_ the access networks of ISPs, in thousands of locations worldwide|Very close to end users → lower delay, higher throughput, fewer hops — but a highly distributed design is hard to maintain and manage|Akamai|
|**Bring Home**|Build fewer, larger clusters, typically at Internet Exchange Points (IXPs) instead of inside individual ISPs|Lower maintenance/management overhead — but possibly higher delay and lower throughput, since servers sit farther from end users|Limelight (and many others)|

> **Analogy — Corner Shops vs. Regional Malls:** Enter Deep is opening a small store on nearly every street corner — maximally close to customers, but a logistical headache to stock thousands of tiny locations. Bring Home is building a handful of huge regional malls at major highway interchanges (IXPs) — easier to run, but customers travel a bit farther to reach one.

### How a CDN Actually Redirects a Request (DNS-Based Redirection)

This leans on something already studied: **DNS**. Concrete example — content provider "NetCinema" uses third-party "KingCDN" to distribute its videos, with each video assigned a URL like `http://video.netcinema.com/6Y7B23V`.

![[Pasted image 20260621135433.png]] _(Figure 2.20 — DNS redirects a user's request to a CDN server: the user's host, NetCinema's authoritative DNS server, KingCDN's authoritative DNS server, the Local DNS server, and a KingCDN content distribution server, connected by six numbered arrows)_

```
USER'S HOST
   │ (1) clicks link → triggers a DNS query for video.netcinema.com
   ▼
LOCAL DNS SERVER (LDNS)
   │ (2) relays the query to NetCinema's authoritative DNS server
   ▼
NETCINEMA AUTHORITATIVE DNS
   │ (3) does NOT return an IP — observes the string "video" in the
   │     hostname and hands off by returning a hostname inside
   │     KingCDN's own domain instead, e.g. a1105.kingcdn.com
   ▼
LOCAL DNS SERVER (LDNS)
   │ (4) sends a SECOND query, now for a1105.kingcdn.com —
   │     entering KingCDN's own private DNS infrastructure
   ▼
KINGCDN AUTHORITATIVE DNS
   │ returns the IP address of a specific, nearby KingCDN
   │ content server
   ▼
LOCAL DNS SERVER (LDNS)
   │ (5) forwards that IP address back to the user's host
   ▼
USER'S HOST
   │ (6) establishes a direct TCP connection to that IP and issues
   │     an HTTP GET for the video. If DASH is used, the server
   │     first sends back the manifest file.
   ▼
KINGCDN CONTENT SERVER → streams the video
```

The key insight: **the hand-off happens entirely inside DNS, before a single byte of video is requested.** NetCinema's own DNS server never returns an IP for the video — it deliberately hands the query to KingCDN's _own_ DNS infrastructure, which is the system that actually decides which physical KingCDN server (which cluster, which location) the user should be sent to.

> **Analogy — A Receptionist Who Transfers You:** It's like calling a company's switchboard, and instead of connecting you directly, the receptionist says "let me transfer you to our courier partner's own dispatch line" — and that dispatch line is the one that actually decides which delivery driver (server) is nearest to you.

### Cluster Selection Strategies

Once a request reaches the CDN's own DNS system, the CDN still has to decide **which of its many clusters** should serve this particular user.

|Strategy|How It Works|Advantage|Disadvantage|
|---|---|---|---|
|**Geographically closest**|Use a commercial geo-location database to map the requesting LDNS's IP address to a physical location, and pick the nearest cluster "as the bird flies"|Simple, works reasonably well for most clients|Geographic distance ≠ network distance (hops/delay); breaks down if the user is configured to use a remotely-located LDNS; ignores real-time congestion|
|**Real-time measurement**|CDN clusters periodically probe LDNSs worldwide (e.g., ping messages or DNS queries) to measure actual delay/loss, and route based on _current_ conditions|Reflects real network conditions, not just map distance; adapts as conditions change|Many LDNSs are configured to silently ignore such probes, limiting how many clients can actually be measured this way|

### CDN Caching: How Content Actually Gets Onto Each Server

A CDN cannot store _everything_ at _every_ location — too expensive in storage and distribution cost, especially for rarely-watched content. Each location stores only a subset, collectively called a **cache**, managed with one of two strategies:

|Strategy|How It Works|Used By|
|---|---|---|
|**Push**|The CDN _forecasts_ which videos will be in high demand at each location, and proactively sends ("pushes") those videos to the relevant clusters ahead of time|Netflix|
|**Pull**|Content is _not_ pre-positioned. On a **cache miss**, the cluster retrieves the video from a CDN central server (or another cluster), stores a local copy, and streams it to the user — all in parallel, to keep latency low|YouTube (Akamai mixes both push and pull)|

When a cluster's storage fills up, it removes videos that are no longer frequently requested, to make room for fresher or more popular content.

> **Analogy — Restocking a Vending Machine:** Push caching is a vending machine supplier who studies sales patterns and restocks each machine with predicted best-sellers _before_ anyone asks. Pull caching is a machine that's empty until the first customer requests a specific snack — the machine "phones" the warehouse, gets one delivered, keeps a few extra in stock, and only restocks proactively once it sees repeated demand.

### Case Study Box — Akamai

Akamai pioneered the **Enter Deep** philosophy and has run CDN infrastructure since the late 1990s. As of late 2023, Akamai operated more than 360,000 servers across more than 4,200 edge locations in over 1,350 networks, delivering more than 175 terabits per second of data to end users. Akamai's core technology has two parts: (1) content-replication algorithms that dynamically scale the number of copies of content in response to demand spikes ("flash crowds"), placing copies close to requesting users, and (2) a DNS-based redirection service — the same general mechanism walked through above — that directs each user's request to a nearby available server. Akamai also offers general edge-computing services beyond video, letting companies run computation physically close to their customers.

### Case Study Box — Google's Data Center and Network Infrastructure

Google's infrastructure has **two tiers**, interconnected by Google's own private global wide-area network, called **B4**:

```
                ┌──────────────────────────┐
                │   Google's Private WAN    │
                │      (network B4)         │
                └────────────┬──────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                                            │
 ┌──────▼───────┐                          ┌─────────▼──────────┐
 │ Large-Scale   │                          │ Network Edge        │
 │ Data Centers  │                          │ Sites (GGC, etc.)   │
 │ ~23 "mega"    │                          │ placed inside ISPs  │
 │ centers,      │                          │                     │
 │ ~100,000      │                          │ → static/cacheable  │
 │ servers each  │                          │   content (e.g.     │
 │ → dynamic,    │                          │   YouTube videos)   │
 │   personalized│                          │                     │
 │   content     │                          │                     │
 └───────────────┘                          └─────────────────────┘
```

- **Large-scale data centers** — roughly 23 "mega data centers" across North America, Europe, and Asia, each with on the order of 100,000 servers. These serve dynamic, often personalized content — search results, Gmail messages, and so on.
- **Network edge sites** — more than 100 Google Global Cache (GGC) sites plus 140+ network edge sites, placed _inside_ ISPs, serving relatively static content (like YouTube videos) close to end users; this keeps a high-quality experience for the user while freeing the ISP from carrying that traffic in from outside its own network.
- **Network connectivity** — Google's Edge Points of Presence (POPs) connect to ISPs and private networks at more than 200 IXPs and over 100 additional interconnection facilities worldwide.
- **Google Cloud** — in addition to serving Google's own products, Google also hosts third-party services; for example, Spotify runs its own services on Google Cloud, which (as of late 2024) had data centers across 40 regions globally.

For a single YouTube video request, the pieces are stitched together: the static video content may come from a nearby "bring-home" edge cache, the surrounding web page content from a nearby "enter-deep" cache, and the personalized recommendations/ads from one of the mega data centers.

---

## 2.5.4 Case Studies: Netflix and YouTube

Both companies solve the same underlying problem — encode and distribute video at scale — but with notably different architectures.

### Netflix

As of 2025, Netflix is one of the leading streaming services in North America. Its distribution system has **two major components**: the Amazon cloud (control plane) and its own private CDN, **Open Connect** (data plane).

![[Pasted image 20260621135901.png]] _(Figure 2.21 — Netflix video streaming platform: the Amazon Cloud uploads versions to multiple CDN servers, sends the manifest file to the client, and CDN servers stream video chunks via DASH directly to the client)_

**What runs in the Amazon Cloud (control plane):**

|Function|What It Does|
|---|---|
|**Web site / user-facing services**|User registration, login, billing, the movie catalog, and the recommendation system — run entirely on Amazon servers|
|**Content ingestion**|Before any movie can stream, Netflix receives the studio's master version and processes it|
|**Content processing**|Creates many different formats per movie — one for each target device (desktop, smartphone, smart TV, game console) — each at multiple bit rates, enabling DASH-style adaptive streaming|
|**Uploading to CDN**|Once all versions of a movie are ready, the Amazon-cloud hosts upload them to Open Connect|

**What runs on Open Connect (data plane):** Netflix rolled out its streaming service in 2007 initially using three third-party CDN companies. It has since built **Open Connect**, its own private CDN, and now streams all of its video through it. Netflix has installed its own server racks both **inside IXPs** and **inside residential ISPs themselves** — currently in over 200 IXP locations, plus hundreds of ISP locations. Each rack has multiple 10 Gbps Ethernet ports and over 100 terabytes of storage. Larger IXP installations hold the _entire_ Netflix streaming library, in all its DASH bit-rate versions; smaller racks (which cannot hold the entire library) hold only the most popular videos, determined on a day-to-day basis.

**How a Netflix stream actually starts, step by step:**

```
1. User selects a movie on the website (served from Amazon Cloud)
2. Netflix's Amazon-cloud control software determines which of its
   CDN servers currently hold a copy of that movie
3. Among those, it picks the "best" server for THIS client:
       - prefers a Netflix rack inside the client's own residential
         ISP, if that rack has the movie
       - otherwise, a rack at a nearby IXP
4. The Amazon cloud sends the client:
       - the IP address of the chosen CDN server
       - a manifest file with the URLs for the different DASH versions
5. Client connects DIRECTLY to that CDN server and starts requesting
   ~4-second-long DASH chunks (via the HTTP byte-range header),
   adapting bit rate as it goes (2.5.2)
```

**Key architectural difference from a generic CDN:** Netflix does **not** use DNS redirection to pick a server (unlike 2.5.3 above) — the Netflix control software _directly tells_ the client which CDN server to use. Because Netflix's private CDN distributes _only_ video (never web pages), it can simplify and tailor its design specifically for that one job. Netflix also uses **push caching exclusively**, not pull: content is pushed into CDN racks on a schedule, during off-peak hours, rather than fetched reactively on a cache miss.

### YouTube

YouTube began service in April 2005 and was acquired by Google in November 2006. Its design and protocols are largely **proprietary** — what's known about its internals comes mainly from independent measurement studies, not official documentation.

Like Netflix, YouTube relies heavily on CDN technology — specifically **Google's own private CDN**, with server clusters in many hundreds of different IXP and ISP locations, plus Google's own huge data centers.

|Aspect|Netflix|YouTube|
|---|---|---|
|**CDN ownership**|Private (Open Connect)|Private (Google's CDN)|
|**Caching strategy**|Push only|**Pull**|
|**Server-selection method**|Control software directly assigns a server — no DNS redirect|**DNS redirect** (2.5.3) — most of the time, directs the client to the cluster with the lowest RTT|
|**Load balancing**|Not needed — assignment is content-aware|Sometimes deliberately directs a client to a more distant cluster via DNS, specifically to balance load across clusters|
|**Content scope of CDN**|Video only (web pages served separately, from Amazon Cloud)|Video only (web pages served from Google's other infrastructure)|
|**Content arrival path**|Studio masters ingested once, centrally, by Netflix|Several million videos uploaded daily, client→server over HTTP; uploads are converted to YouTube format at multiple bit rates entirely within Google's own data centers|

> **Why would YouTube deliberately pick a farther cluster?** A real engineering trade-off: always picking the absolute closest cluster (lowest RTT) can overload that one cluster when many nearby users hit it at once. Occasionally routing some users to a slightly farther — but less congested — cluster keeps overall system throughput higher, even though it's individually a bit worse for those redirected users.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**DNS-based CDN redirection is an attack surface**|Since cluster selection happens via DNS (2.5.3), anyone who can poison or spoof DNS responses can redirect users to an attacker-controlled "CDN server" instead of the legitimate one — serving malware or stealing credentials under a trusted-looking domain|Use DNSSEC to cryptographically validate DNS responses; CDNs should monitor for DNS resolutions pointing to unexpected IP ranges|
|**Manifest files as a target**|A DASH manifest lists every chunk URL for a video. If it isn't integrity-protected, an attacker who tampers with it could redirect chunk requests to malicious URLs, or inject fake low-quality chunks to degrade service|Serve manifest files over HTTPS/TLS; sign manifests where the streaming protocol supports it|
|**CDN as a man-in-the-middle by design**|A third-party CDN sits directly between every user and the origin content, often terminating TLS on the CDN's own servers — meaning the CDN provider technically _can_ see decrypted traffic, a major trust dependency attackers could exploit if the CDN's infrastructure is compromised|Choose CDN providers with strong security track records and contractual data-handling guarantees; for highly sensitive content, consider end-to-end encryption the CDN cannot decrypt, accepting the loss of CDN-side optimizations|
|**Geo-location database spoofing for cluster selection**|"Geographically closest" selection relies on mapping an LDNS's IP to a location via a commercial geo-database. Attackers controlling DNS infrastructure for a victim network can influence which LDNS appears to make a request, subtly steering which cluster traffic gets routed through|Cross-validate geo-IP results against real-time RTT/loss measurements rather than trusting geo-databases alone|
|**Encrypted video traffic still leaks information (side-channel)**|Even when DASH chunks travel over HTTPS, the distinctive _pattern_ of chunk sizes and timing for a given video — since DASH segments video into variable-sized chunks based on content complexity — can act as a fingerprint. An observer monitoring only encrypted traffic metadata can sometimes infer _which specific video_ a user is watching, without decrypting anything|Pad chunk sizes to obscure the underlying content-complexity pattern; use more uniform segment sizes — this remains an active area of privacy research, not a fully solved problem|
|**DDoS resilience as both a benefit and a risk of CDN architecture**|Because CDNs distribute load across thousands of servers, a volumetric DDoS aimed at a single origin is much harder to execute — but the CDN itself becomes a high-value target, since compromising the CDN provider's infrastructure can affect every customer behind it simultaneously|CDNs invest heavily in their own DDoS-mitigation infrastructure; content providers should understand they're trusting their CDN's security posture as much as their own|
|**Push-caching content integrity (Netflix-style)**|If an attacker compromises the content-ingestion or transcoding pipeline upstream, poisoned content could be _pushed_ out to thousands of CDN racks proactively, before anyone notices — a much larger blast radius than a pull-based system, where caches only fetch content reactively as users actually request it|Strong integrity checks (hashing/signing) at every stage of the ingestion → processing → CDN-push pipeline; treat the control plane as a high-value target requiring strict access control|

---

## Questions I Still Have

- [ ] DASH's rate-adaptation algorithm is described here only at a high level ("if buffer is full and bandwidth is high, go up a tier") — what do _real_ algorithms (YouTube's, Netflix's) actually optimize for, and how do they avoid oscillating rapidly between quality levels?
- [ ] The DNS hand-off in Figure 2.20 — is this technically implemented via a CNAME record, or some other DNS record type? Worth connecting back to whatever the DNS section covered about record types.
- [ ] For "real-time measurement" cluster selection, many LDNSs ignore probes — how big a fraction, in practice? Does this mean large CDNs end up relying mostly on geo-location instead?
- [ ] Netflix's control software tells the client which server to use directly, with no DNS redirection at all. Does that make Netflix's CDN selection fully immune to the DNS-spoofing risks above, or does the initial Amazon-cloud connection introduce a different point of trust?
- [ ] How does a CDN cache actually decide a video has become unpopular and should be evicted — simple least-recently-used (LRU), or something that predicts future demand?
- [ ] The chunk-size side-channel risk mentioned in the security table — how practical is this in reality? Has it been demonstrated against real streaming services, or is it mostly theoretical?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Bit rate**|The number of bits per second needed to represent a video at a given quality; the single most important network-relevant property of a video|
|**Compression**|Algorithms that reduce a video's bit rate while trading off some image quality; lets the same video be encoded at essentially any target bit rate|
|**HTTP streaming**|The simplest form of video delivery: the video is an ordinary file at a URL, served via a normal HTTP GET, with the client buffering before playback|
|**Buffering**|Collecting incoming video bytes in a client-side memory buffer before/during playback, so brief network slowdowns stay invisible to the viewer|
|**DASH (Dynamic Adaptive Streaming over HTTP)**|A streaming protocol where the video is encoded at multiple bit rates, divided into small chunks, and the client dynamically requests whichever chunk/version best matches current measured bandwidth and buffer state|
|**Representation / version**|One specific bit-rate encoding of a video within a DASH deployment|
|**Chunk / segment**|A short (often a few-second) piece of one version of a video, individually requestable via its own URL|
|**Manifest file**|A file listing all available versions of a video, their bit rates, and corresponding URLs — the first thing a DASH client downloads|
|**Rate-determination algorithm**|Client-side logic deciding which bit-rate version's chunk to request next, based on measured throughput and buffer level|
|**Byte-range header**|An HTTP header letting a client request only part of a file — used by DASH clients to fetch a specific chunk|
|**Content Distribution Network (CDN)**|A network of geographically distributed servers that store copies of content and redirect each user's request to a nearby server, reducing delay, reducing repeated bandwidth use, and avoiding single points of failure|
|**Private CDN**|A CDN owned and operated by the content provider itself (e.g., Netflix's Open Connect, Google's CDN for YouTube)|
|**Third-party CDN**|A CDN company distributing content on behalf of multiple unrelated content providers (e.g., Akamai, Cloudflare, Amazon CloudFront)|
|**Enter Deep**|A CDN placement philosophy: many small server clusters deep inside ISP access networks, close to end users (pioneered by Akamai)|
|**Bring Home**|A CDN placement philosophy: fewer, larger clusters at a small number of sites, typically IXPs, instead of inside individual ISPs|
|**Cluster selection strategy**|The mechanism a CDN uses to decide which of its server clusters should handle a given client's request|
|**Geographically-closest selection**|A cluster-selection strategy using a geo-location database to map the requester's LDNS IP to a location and pick the nearest cluster|
|**Real-time measurement selection**|A cluster-selection strategy where clusters actively probe LDNSs (ping/DNS queries) and select based on actual current delay/loss|
|**Cache (CDN context)**|The subset of content stored at a particular CDN location; not every location holds every piece of content|
|**Push caching**|Proactively sending content to clusters in anticipation of demand, before any user requests it (used by Netflix)|
|**Pull caching**|Fetching content into a cluster reactively, only after a cache miss (used by YouTube)|
|**Cache miss**|When a CDN cluster receiving a request does not have a local copy of the requested content, and must retrieve it elsewhere|
|**DNS-based redirection**|The mechanism by which a content provider's authoritative DNS server hands a query off to a CDN's own private DNS infrastructure, which resolves it to a specific CDN server's IP|
|**Open Connect**|Netflix's own private CDN, with server racks installed inside both IXPs and residential ISPs|
|**B4**|Google's own private global wide-area network, interconnecting its data centers|
|**Google Global Cache (GGC)**|Google's network-edge cache sites, installed inside ISPs, serving relatively static content like YouTube videos close to end users|

---

## Related Concepts

---

→ Next: [[SOCKET PROGRAMMING]]