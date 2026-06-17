---
title: DNS
date: 2026-06-17
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.5 DNS — The Internet's Directory Service

> **One-Line Summary:** DNS is a **distributed, hierarchical database** plus an application-layer protocol that translates human-friendly **hostnames** into machine-friendly **IP addresses** — and it does this through a delegated tree of root, TLD, and authoritative servers, made fast and scalable by aggressive **caching**.

---

## Core Idea

Human beings can be identified in many ways — by name, by Social Security number, by driver's license number. Each identifier suits a different context: the IRS prefers your fixed-length SSN because it's easy for computers to process and guaranteed unique; ordinary people prefer your name because it's memorable.

> **Analogy:** Imagine introducing yourself at a party by saying "Hi, my name is 132-67-9875." It works, technically — every number is unique — but nobody will remember it. Now imagine a government database trying to file paperwork using "John Smith" when there are ten thousand John Smiths. Both identifiers are valid; they're just optimized for different audiences.

Internet hosts have exactly the same dual-identity problem:

|Identifier|Optimized for|Example|
|---|---|---|
|**Hostname**|Humans — mnemonic, memorable|`cnn.com`, `www.yahoo.com`, `gaia.cs.umass.edu`|
|**IP address**|Routers — fixed-length, hierarchical, easy to process|`121.7.106.83`|

> An IP address consists of **four bytes** and has a **rigid hierarchical structure** — as you scan it left to right, you get progressively more specific information about where the host is located in the network of networks. This is similar to scanning a postal address from bottom to top: city tells you less than street address, which tells you less than apartment number.

**DNS exists to bridge these two worlds** — it is the directory service that translates the hostname you type into the IP address your computer actually needs to open a TCP connection.

---

# 2.5.1 Services Provided by DNS

> DNS is **(1)** a distributed database implemented in a hierarchy of **DNS servers**, and **(2)** an application-layer protocol that allows hosts to query that distributed database.

DNS servers commonly run **BIND** (Berkeley Internet Name Domain) software on UNIX machines. **DNS runs over UDP and uses port 53** — notably, this is one of the few major application-layer protocols that does _not_ run over TCP, because DNS queries are typically small and a single lost packet can simply be retried rather than requiring a full reliable connection.

---

## Service 1 — Hostname-to-IP Translation (The Core Job)

DNS is invisibly invoked every single time you use HTTP, SMTP, or FTP with a hostname instead of a raw IP address. Here's exactly what happens when your browser requests `www.someschool.edu/index.html`:

```
Step 1 → The browser extracts the hostname, www.someschool.edu, from the URL
          and passes it to the client side of the DNS application

Step 2 → The DNS client sends a query containing the hostname
          to a DNS server

Step 3 → The DNS client eventually receives a reply,
          which includes the IP address for the hostname

Step 4 → Once the browser receives the IP address from DNS,
          it can initiate a TCP connection to the HTTP server
          process located at port 80 at that IP address
```

> **Important performance note:** DNS adds an **additional delay** to every Internet application that uses it — sometimes substantial. Fortunately, the desired IP address is often **cached** in a nearby DNS server, which helps reduce both network traffic and average delay (we'll see how shortly).

---

## Service 2 — Host Aliasing

> A host with a complicated hostname can have one or more **alias names**.

For example, `relay1.west-coast.enterprise.com` is hard to remember and exposes internal infrastructure naming conventions. The company can register friendlier aliases like `enterprise.com` or `www.enterprise.com`.

|Term|Meaning|
|---|---|
|**Canonical hostname**|The "real," complicated underlying hostname — e.g. `relay1.west-coast.enterprise.com`|
|**Alias hostname**|The mnemonic, public-facing name that points to the canonical one — e.g. `enterprise.com`|

> **Analogy:** This is exactly like a company having an official, jargon-heavy legal registration name ("Enterprise Holdings International Logistics Division Server 1") while everyone outside just calls it "Enterprise." DNS can be invoked by an application to obtain the canonical hostname for a supplied alias, as well as the IP address of the host.

---

## Service 3 — Mail Server Aliasing

> For obvious reasons, it is highly desirable that e-mail addresses be mnemonic.

If Bob has a Hotmail account, his e-mail address should be the simple `bob@hotmail.com` — not something exposing Hotmail's actual internal server name, which might be `relay1.west-coast.hotmail.com`.

DNS solves this the same way it solves host aliasing: a mail application can invoke DNS to get the canonical hostname for a supplied alias.

> **A neat consequence:** the **MX record** (covered in 2.5.3) permits a company's **mail server** and **Web server** to have **identical aliased hostnames**. Both could be called `enterprise.com` even though, under the hood, they are two completely different physical machines with different canonical names. A client asking "where do I send mail for enterprise.com?" gets a different answer (via an MX lookup) than a client asking "where do I send a Web request for enterprise.com?" (via an A-record lookup) — same alias, two different underlying machines.

---

## Service 4 — Load Distribution

> DNS is also used to perform **load distribution** among replicated servers, such as replicated Web servers.

Busy sites like `cnn.com` are replicated across multiple servers — each running on a different physical machine with a different IP address. A _set_ of IP addresses is associated with one canonical hostname.

**How DNS spreads the load:**

```
DNS database stores a SET of IP addresses for cnn.com:
  [157.166.226.25, 157.166.226.26, 157.166.226.27, ...]

Client A queries cnn.com
  → DNS server returns the full set, ROTATED: [...26, ...27, ...25]
  → Client A's browser uses the FIRST address in the list → ...26

Client B queries cnn.com a moment later
  → DNS server returns the set again, rotated differently: [...27, ...25, ...26]
  → Client B's browser uses the first address → ...27
```

> Because a client typically sends its HTTP request to the IP address that is **listed first** in the set, **DNS rotation** distributes traffic among the replicated servers automatically — no special load balancer hardware is required at the DNS layer itself.

> **Analogy:** Picture a popular restaurant with five identical branches across town. Instead of a single hostess directing everyone to Branch 1 every time, the hostess randomly shuffles which branch she recommends first to each new customer. Over thousands of customers, the load naturally spreads evenly across all five branches.

DNS rotation is also used for **e-mail**, so multiple mail servers can share the same alias name. Content distribution companies like **Akamai** use DNS in even more sophisticated ways to direct users to nearby content servers (covered in Chapter 7).

> DNS is specified in **RFC 1034** and **RFC 1035**, and updated in several additional RFCs since.

---

# 2.5.2 Overview of How DNS Works

## DNS as a Black Box

From an application's point of view, DNS looks deceptively simple:

```
Application invokes the client side of DNS,
  specifying the hostname to be translated
  (on UNIX machines, the function call is gethostbyname())

DNS query and reply messages travel inside UDP datagrams to port 53

After a delay (milliseconds to seconds),
  the user's host receives a DNS reply with the mapping

This mapping is passed back to the invoking application
```

> From the perspective of the invoking application, **DNS is a black box** providing a simple, straightforward translation service. But the black box that implements this service is actually complex — a large number of DNS servers distributed around the globe, plus an application-layer protocol specifying how they all talk to each other.

---

## Why Not Just Use One Big Centralized DNS Server?

It's tempting to imagine a simpler design: one giant DNS server holding every mapping in the world, with all clients querying it directly. This design is attractive for its simplicity — but it falls apart at Internet scale for **four concrete reasons**:

|Problem|Explanation|
|---|---|
|**Single point of failure**|If the one DNS server crashes, the **entire Internet** effectively stops working — nobody can resolve any hostname|
|**Traffic volume**|A single server would have to handle every DNS query generated by **hundreds of millions of hosts** — every HTTP request, every e-mail, everything|
|**Distant centralized database**|You cannot be "close" to every client simultaneously. Put the server in New York, and queries from Australia must cross half the globe over slow, congested links — adding serious delay|
|**Maintenance**|The database would have to track records for **every host on the Internet**, updated constantly as new hosts appear — an unmanageable single point of administration|

> **Analogy:** Imagine if the entire world had exactly **one library**, located in one city, containing every book ever written, and every single person on Earth had to travel there in person to borrow any book. It would collapse instantly under its own weight — both physically (the building) and logistically (the queues). A distributed system of local libraries, each holding a relevant subset and able to request books from others, is the only design that scales.

> In summary: a centralized database in a single DNS server **simply doesn't scale**. DNS is, consequently, distributed by design — and is held up as a textbook example of how a distributed database can be successfully implemented across the Internet.

---

## The Distributed, Hierarchical Database

To deal with scale, DNS uses a **large number of servers**, organized hierarchically and distributed worldwide. No single DNS server holds all the mappings for all hosts — the mappings are spread across the hierarchy.

> There are, to a first approximation, **three classes of DNS servers**: **root DNS servers**, **top-level domain (TLD) DNS servers**, and **authoritative DNS servers**.

![[Pasted image 20260617221520.png]] _(Figure 2.19 — Portion of the hierarchy of DNS servers: Root DNS servers at the top, branching into TLD servers for .com, .org, .edu, which in turn branch into authoritative servers for specific domains like yahoo.com, amazon.com, pbs.org, poly.edu, umass.edu)_

> **Analogy for the three-tier hierarchy:** Think of it like asking for directions in an unfamiliar country. You first ask a **national tourist information desk** ("Which region handles addresses ending in this postal code zone?") — that's the **root server**. The tourist desk doesn't know your exact address, but it points you to the right **regional office** — that's the **TLD server**, e.g. the office that handles all `.com` addresses. The regional office, in turn, doesn't know your exact house either, but it knows exactly which **local post office** is authoritative for your specific street — that's the **authoritative server**, which finally hands over the precise address (IP).

---

### Root DNS Servers

> There are **13 root DNS servers** (labeled A through M) in the Internet, most located in North America.

![[Pasted image 20260617221839.png]] _(Figure 2.20 — DNS root servers in 2026)_

> Although we refer to each of the 13 root DNS servers as if it were a single server, each "server" is actually a **network of replicated servers**, for both security and reliability purposes. All together, there were **247 root servers as of fall 2011**.

> **Why replicate?** A single physical machine, however well-protected, is a single point of failure and a juicy DDoS target. By replicating each of the 13 logical root servers across dozens of physical machines worldwide (using a technique called anycast), an attack or outage at one location doesn't take down the whole system.

### Top-Level Domain (TLD) Servers

> TLD servers are responsible for top-level domains such as `com`, `org`, `net`, `edu`, and `gov`, as well as country-level top-level domains such as `uk`, `fr`, `ca`, and `jp`.

- The company **Verisign Global Registry Services** maintains the TLD servers for the `.com` top-level domain
- The company **Educause** maintains the TLD servers for the `.edu` top-level domain

### Authoritative DNS Servers

> Every organization with publicly accessible hosts (Web servers, mail servers) on the Internet **must** provide publicly accessible DNS records mapping the names of those hosts to IP addresses. An organization's **authoritative DNS server** houses these records.

An organization has two options:

1. **Implement its own** authoritative DNS server to hold these records
2. **Pay** to have the records hosted in an authoritative DNS server run by a service provider

Most universities and large companies implement and maintain their **own primary and secondary (backup)** authoritative DNS server.

---

### Local DNS Servers — The Hidden Fourth Player

> There is another important type of DNS server called the **local DNS server**. A local DNS server does **not** strictly belong to the hierarchy of servers, but is nevertheless central to the DNS architecture.

Each ISP — a university, an academic department, a company, or a residential ISP — has a local DNS server (also called a **default name server**). When a host connects to an ISP, the ISP provides the host with the IP addresses of one or more of its local DNS servers (typically via DHCP).

> A host's local DNS server is typically "**close**" to the host. For an institutional ISP, the local DNS server may be on the same LAN as the host; for a residential ISP, it is typically separated by no more than a few routers.

When a host makes a DNS query, the query is sent to the **local DNS server**, which acts as a **proxy**, forwarding the query into the server hierarchy on the client's behalf.

> **Analogy:** The local DNS server is like your personal travel agent. You don't personally call up the national tourist office, the regional office, and the local post office every single time you need directions — you tell your travel agent the destination, and _they_ make all the necessary calls on your behalf, then hand you back the final answer.

---

## Worked Example — Resolving gaia.cs.umass.edu

Suppose host `cis.poly.edu` desires the IP address of `gaia.cs.umass.edu`. Suppose Poly's local DNS server is called `dns.poly.edu`, and the authoritative DNS server for `gaia.cs.umass.edu` is `dns.umass.edu`.

![[Pasted image 20260617222055.png]] _(Figure 2.21 — Interaction of the various DNS servers: requesting host cis.poly.edu queries its local DNS server dns.poly.edu ①⑧, which queries the root server ②③, then the TLD server ④⑤, then the authoritative server dns.umass.edu ⑥⑦, before returning the answer to the requesting host)_

```
① cis.poly.edu sends a DNS query to its local DNS server, dns.poly.edu
   (the query asks for: gaia.cs.umass.edu)

② dns.poly.edu forwards the query to a root DNS server

③ The root DNS server notes the .edu suffix and returns a list of
   IP addresses for TLD servers responsible for edu

④ dns.poly.edu resends the query to one of these edu TLD servers

⑤ The TLD server notes the umass.edu suffix and responds with the
   IP address of the authoritative server for umass.edu — i.e. dns.umass.edu

⑥ dns.poly.edu resends the query directly to dns.umass.edu

⑦ dns.umass.edu responds with the IP address of gaia.cs.umass.edu

⑧ dns.poly.edu finally returns the answer to the requesting host, cis.poly.edu
```

> **Total: 8 DNS messages** were sent just to resolve one hostname! (Four queries out, four replies back.)

**But it can get even deeper.** Suppose that the University of Massachusetts has its own DNS server for the university (`dns.umass.edu`), and each _department_ additionally has its own departmental DNS server, authoritative for hosts in that department. In this case, when the intermediate server `dns.umass.edu` receives a query for a host ending in `cs.umass.edu`, it doesn't have the final answer either — it returns the address of `dns.cs.umass.edu` (the department's own authoritative server) to `dns.poly.edu`, which must then query _that_ server too.

> In this deeper scenario, a total of **10 DNS messages** are sent — DNS resolution can chain through as many layers of delegation as an organization chooses to set up.

---

## Recursive vs Iterative Queries

The example above uses **both** kinds of queries simultaneously:

|Query Type|How it behaves|Who does the work|
|---|---|---|
|**Recursive query**|The querying server asks another server to **obtain the mapping on its behalf** and report back the final answer|The _queried_ server takes on the burden of chasing down the chain|
|**Iterative query**|The reply is sent **directly back** to the original requester, who must then make the next query itself|The _original querier_ does all the chasing|

In the worked example: the query sent from `cis.poly.edu` to `dns.poly.edu` is **recursive** (poly.edu asked the local server to "go get me the full answer, whatever it takes"). All three subsequent queries — local server to root, local server to TLD, local server to authoritative — are **iterative**, because each server replies directly back to `dns.poly.edu` rather than chasing the next hop itself.

![[Pasted image 20260617222226.png]] _(Figure 2.22 — Recursive queries in DNS: an alternative query chain where every single query in the chain — including the root server's and TLD server's onward requests — is recursive rather than iterative)_

> **In theory**, any DNS query can be either iterative or recursive. **In practice**, queries typically follow the pattern shown earlier: the query from the requesting host to its local DNS server is **recursive**, while the remaining queries (local server onward into the hierarchy) are **iterative**.

> **Analogy:** Recursive is like asking your assistant, "Find out the CEO's direct phone number — I don't care how many calls you have to make, just bring me back the final number." Iterative is like the assistant calling the company's main switchboard and being told, "I don't know that, call this regional office instead" — and having to dial that number themselves, rather than the switchboard making the next call.

> **Why does the hierarchy mostly use iterative queries for the upper levels?** Root and TLD servers handle an enormous volume of traffic; if every single query forced them to recursively chase the whole answer down on behalf of the requester, that load would be far heavier than simply pointing the requester to the next server and stepping away.

---

## DNS Caching — The Key to Making This Fast

> Our discussion thus far has ignored **DNS caching**, a critically important feature of the DNS system. In truth, **DNS extensively exploits caching** to improve delay performance and reduce the number of DNS messages ricocheting around the Internet.

**The idea is simple:** In a query chain, when a DNS server receives a DNS reply (containing, say, a mapping from hostname to IP address), it can **cache** that mapping in its local memory.

```
apricot.poly.edu queries dns.poly.edu for the IP address of cnn.com
  → dns.poly.edu doesn't have it cached → goes through the full chain
  → gets the answer → CACHES the mapping locally → returns it

A few hours later, kiwi.poly.fr also queries dns.poly.edu for cnn.com
  → dns.poly.edu finds the cached mapping IMMEDIATELY
  → returns it WITHOUT contacting any other DNS server at all
```

> A local DNS server can even cache the IP addresses of **TLD servers**, allowing it to bypass the **root servers** entirely in a query chain — this happens often in practice, and is a big reason root servers aren't overwhelmed despite handling billions of devices' worth of traffic.

**The catch — staleness:** Because hosts and hostname/IP mappings are **by no means permanent** (a server might change IP address, or shut down), DNS servers **discard cached information after a period of time** — often set to **two days**. This time-to-live behaviour is governed by the **TTL** field, covered next.

> **Analogy:** This is exactly like memorizing a friend's phone number after looking it up once instead of checking your contacts app every single time you want to call them. You trust the number for a while — but if you haven't talked to them in months, you might want to double-check it's still valid, because people do change numbers.

---

# 2.5.3 DNS Records and Messages

## Resource Records (RRs)

> The DNS servers that together implement the distributed database store **resource records (RRs)**, including RRs that provide hostname-to-IP address mappings. Each DNS reply message carries one or more resource records.

A resource record is a **four-tuple**:

```
(Name, Value, Type, TTL)
```

> **TTL** is the **time to live** of the resource record — it determines when a resource should be removed from a cache. (We'll ignore TTL in the examples below for simplicity.)

The meaning of `Name` and `Value` **depends entirely on `Type`**:

|Type|Name field means...|Value field means...|Example record|
|---|---|---|---|
|**A**|A hostname|The IP address for that hostname|`(relay1.bar.foo.com, 145.37.93.126, A)`|
|**NS**|A domain (e.g. `foo.com`)|The hostname of an authoritative DNS server that knows how to obtain IPs for hosts in that domain|`(foo.com, dns.foo.com, NS)`|
|**CNAME**|An alias hostname|The **canonical** hostname for that alias|`(foo.com, relay1.bar.foo.com, CNAME)`|
|**MX**|An alias hostname|The canonical name of a **mail server** that has that alias|`(foo.com, mail.bar.foo.com, MX)`|

> **The NS record is the glue that builds the hierarchy.** It doesn't answer "what's the IP address" — it answers "which server should I ask next." This is what allows a TLD server to route you toward an authoritative server without itself knowing the final answer.

> **MX vs CNAME — a subtle but important distinction:** Both deal with aliases, but for **different kinds of services**. By using the MX record, a company can have the **same** aliased hostname for both its mail server _and_ one of its other servers (such as its Web server) — e.g. both could be `enterprise.com`. To obtain the canonical name for the _mail server_ specifically, a DNS client queries for an **MX record**; to obtain the canonical name for the _other server_ (e.g. Web), the client queries for a **CNAME record**. Same alias name, different record type, different underlying machine.

### How Authoritative-ness Affects What's Stored

> If a DNS server is **authoritative** for a particular hostname, it will contain a **Type A record** for that hostname (even non-authoritative servers may contain a Type A record, simply cached).

> If a server is **not authoritative** for a hostname, it instead contains a **Type NS record** for the domain that includes the hostname — plus a Type A record giving the IP address of _that_ NS server, so the chain of delegation can continue.

**Worked example:** Suppose an `edu` TLD server is not authoritative for `gaia.cs.umass.edu`. It will instead contain:

```
(umass.edu, dns.umass.edu, NS)     ← "I don't know it, but ask this server"
(dns.umass.edu, 128.119.40.111, A) ← "...and here's that server's IP address"
```

---

## DNS Messages

There are exactly two kinds of DNS messages — **query** and **reply** — and both share the **same format**.

![[1781714206182_image.png]] _(Figure 2.23 — DNS message format: a 12-byte header section (identification, flags, and four count fields) followed by four variable-length sections — questions, answers, authority, and additional information)_

### The Header Section (First 12 Bytes)

|Field|Purpose|
|---|---|
|**Identification**|A 16-bit number identifying the query. Copied into the reply so the client can match received replies with sent queries|
|**Flags**|Multiple 1-bit flags: query/reply flag, authoritative flag (set when the answering server is authoritative for the queried name), recursion-desired flag (set by the client when it wants the server to perform recursion), recursion-available flag (set by the server if it supports recursion)|
|**Number of questions / answer RRs / authority RRs / additional RRs**|Four count fields indicating how many entries follow in each of the four sections below|

### The Four Variable-Length Sections

|Section|Contents|
|---|---|
|**Question**|The name being queried, plus a type field (Type A for a host address, Type MX for a mail server, etc.)|
|**Answer**|The resource record(s) for the name that was originally queried. Can contain **multiple** RRs — e.g. a replicated Web server has multiple IP addresses|
|**Authority**|Records of other authoritative servers|
|**Additional**|Other helpful records. Example: an MX query's answer field contains the canonical hostname of the mail server; the additional section then contains a **Type A record** providing the IP address for _that_ canonical hostname — saving the client a second round trip|

---

## Querying DNS Yourself — nslookup

> The **`nslookup`** program, available on most Windows and UNIX platforms, lets you send a DNS query message directly to any DNS server (root, TLD, or authoritative) and see the human-readable reply.

```bash
nslookup
```

You can also use one of many websites that let you remotely run `nslookup` if you don't want to use your own terminal.

---

## Inserting Records into the DNS Database — Registering a Domain

How do records get into the DNS database in the first place? Suppose you've started a company called **Network Utopia** and want to register `networkutopia.com`.

> A **registrar** is a commercial entity that verifies the uniqueness of the domain name, enters the domain name into the DNS database, and collects a small fee from you for its services.

Prior to 1999, a single registrar (Network Solutions) had a monopoly on `.com`, `.net`, and `.org` domain registration. Today, many registrars compete for customers, and **ICANN** (Internet Corporation for Assigned Names and Numbers) accredits them all.

**What you must provide the registrar:**

When registering `networkutopia.com`, you must provide the names and IP addresses of your **primary and secondary authoritative DNS servers** — say, `dns1.networkutopia.com` (212.212.212.1) and `dns2.networkutopia.com` (212.212.212.2).

**What the registrar does with that information:**

For each of your two authoritative DNS servers, the registrar inserts a **Type NS** and a **Type A** record into the **TLD `com` servers**:

```
(networkutopia.com, dns1.networkutopia.com, NS)
(dns1.networkutopia.com, 212.212.212.1, A)
```

You'll also need to ensure that the **Type A** record for your Web server (`www.networkutopia.com`) and the **Type MX** record for your mail server (`mail.networkutopia.com`) are entered into **your own authoritative DNS servers** (not the TLD servers — that's your job to maintain, not the registrar's).

> **Historically static, now dynamic:** Until fairly recently, the contents of each DNS server were configured statically — e.g. from a configuration file created by a system administrator by hand. More recently, an **UPDATE** option has been added to the DNS protocol (**RFC 2136**, **RFC 3007**) to allow data to be **dynamically** added or deleted via DNS messages themselves — no manual file editing required.

---

## Full Worked Example — Putting It All Together

Once registration is complete, people can visit `www.networkutopia.com` and send e-mail to employees there. Let's trace exactly what happens when **Alice in Australia** views the web page:

```
1. Alice's host sends a DNS query to her LOCAL DNS server

2. The local DNS server contacts a TLD com server
   (after first contacting a root server, unless that root's address is cached)

3. The TLD com server contains the NS and A records the registrar
   inserted earlier → it replies to Alice's local DNS server with
   BOTH resource records

4. Alice's local DNS server sends a DNS query to 212.212.212.1
   (dns1.networkutopia.com), asking specifically for the Type A
   record corresponding to www.networkutopia.com

5. This record provides the IP address of the desired Web server,
   say 212.212.71.4, which the local DNS server passes back to
   Alice's host

6. Alice's browser can now initiate a TCP connection to
   212.212.71.4 and send an HTTP request over that connection
```

> **There's a lot more going on than meets the eye when you surf the Web!** Every single hostname-based request — every link click, every e-mail send — silently triggers this entire chain of delegation, caching, and lookups, almost always completed in well under a second.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**DNS cache poisoning**|An attacker injects a **forged** resource record into a DNS server's cache (e.g. tricking it into caching `bank.com → attacker's IP`). Every subsequent query for that hostname from that server's clients gets redirected to the attacker|**DNSSEC** (DNS Security Extensions) cryptographically signs resource records, allowing resolvers to verify authenticity and reject forged responses|
|**DNS spoofing / Man-in-the-middle on UDP**|Because DNS runs over **UDP** (connectionless, unauthenticated), an attacker who can intercept or race a DNS query can send a forged reply before the real one arrives, redirecting the victim to a malicious IP|Source port randomization, query ID randomization (raising the bar for blind-guessing the 16-bit Identification field), and DNSSEC for cryptographic verification|
|**DDoS attacks on root/TLD servers**|The 13 root server "addresses" are a tempting target — if attackers could overwhelm all 247 physical root server instances simultaneously, hostname resolution worldwide would degrade|**Anycast** replication (hundreds of physical machines sharing the same logical address) and aggressive **caching** at lower levels mean even sustained attacks on roots rarely cause global outages — most queries never even reach a root server thanks to caching|
|**Typosquatting / lookalike domains**|Attackers register domains that are one character off from a legitimate one (`gooogle.com`, `paypa1.com`) and rely on users mistyping or misreading|Browser warnings, trademark-based registrar takedown policies, user vigilance|
|**DNS tunneling**|Because DNS queries are rarely blocked by firewalls (everything needs DNS to function), attackers encode **stealth command-and-control traffic or exfiltrated data** inside DNS query/response fields, bypassing network monitoring focused on HTTP/FTP|Deep packet inspection of DNS traffic for abnormal query patterns, query length, and entropy; restricting outbound DNS to approved resolvers only|
|**Open DNS resolvers — amplification attacks**|A DNS server configured to answer recursive queries from **anyone** on the Internet (not just its own clients) can be abused: an attacker sends a small spoofed query (with the victim's IP as the source) requesting a large record type, and the resolver blasts a much larger reply at the victim — a classic **DNS amplification DDoS**|Configure resolvers to only accept recursive queries from trusted/internal IP ranges; rate-limit responses|
|**Registrar account hijacking**|If an attacker compromises your domain registrar account, they can change your authoritative NS records to point anywhere — effectively hijacking your entire domain (mail, web, everything) without ever touching your actual servers|Strong registrar account security: 2FA, registry locks, monitoring for unauthorized DNS record changes|
|**MX record manipulation**|If an attacker can alter a domain's MX record (via DNS poisoning or registrar compromise), all incoming e-mail for that domain silently routes to the attacker's mail server instead|DNSSEC validation of MX lookups; monitoring DNS records for unexpected changes; SPF/DKIM/DMARC (from Section 2.4) provide additional layers even if MX is compromised|

---

## Questions I Still Have

- [ ] **DNSSEC** keeps coming up as the fix for cache poisoning and spoofing — but how exactly does it chain trust from the root zone down to an individual domain's records? Is there a "root key" that everything ultimately anchors to?
- [ ] If DNS runs over **UDP** for speed, what happens when a reply is too large to fit in a single UDP datagram (e.g. DNSSEC-signed responses with large cryptographic signatures)? Does DNS fall back to TCP in that case?
- [ ] How does **anycast routing** actually work to let 247 physical machines all answer to the same 13 logical root server addresses? Is this purely a BGP routing trick?
- [ ] TTL is often set to **two days** for caching — but for load-balanced or rapidly-changing services (like CDN edge nodes), TTLs are often set to mere **seconds or minutes**. What's the tradeoff curve between freshness and DNS query volume here?
- [ ] How does **DHCP** (mentioned briefly as the mechanism that tells a host its local DNS server's address) actually communicate that information at connection time — what's in the DHCP handshake that carries it?
- [ ] Modern resolvers increasingly use **DNS over HTTPS (DoH)** or **DNS over TLS (DoT)** to encrypt queries — how does this interact with the local DNS server / ISP model described here? Does it bypass the ISP's local DNS server entirely?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**DNS**|Domain Name System — a distributed database plus application-layer protocol that translates hostnames to IP addresses|
|**Hostname**|A mnemonic, human-friendly identifier for an Internet host (e.g. `www.yahoo.com`)|
|**IP address**|A 32-bit (four-byte), hierarchically structured numeric identifier for a host (e.g. `121.7.106.83`)|
|**Canonical hostname**|The "real," underlying hostname of a server|
|**Alias hostname**|A mnemonic, public-facing name pointing to a canonical hostname|
|**DNS rotation**|Returning a set of IP addresses for one hostname in rotated order across different queries, to distribute load among replicated servers|
|**Root DNS server**|Top of the DNS hierarchy; there are 13 logical root servers (A–M), each replicated across many physical machines worldwide|
|**TLD (Top-Level Domain) server**|Responsible for domains like `.com`, `.org`, `.edu`, and country codes like `.uk`, `.jp`|
|**Authoritative DNS server**|Holds the actual, definitive DNS records for an organization's publicly accessible hosts|
|**Local DNS server**|An ISP's "default name server" that acts as a proxy, forwarding a host's queries into the DNS hierarchy; not formally part of the hierarchy itself|
|**Recursive query**|A query where the receiving server obtains the full answer on behalf of the requester before replying|
|**Iterative query**|A query where the receiving server replies directly with a pointer to the next server, leaving the requester to make the next query itself|
|**DNS caching**|Storing previously resolved mappings locally to avoid repeating the full resolution chain; subject to a TTL-based expiration|
|**TTL (Time to Live)**|A field on a resource record dictating how long it may remain cached before being discarded|
|**Resource Record (RR)**|A four-tuple `(Name, Value, Type, TTL)` — the basic unit of data stored and exchanged in DNS|
|**Type A record**|Maps a hostname directly to an IP address|
|**Type NS record**|Maps a domain to the hostname of an authoritative server for that domain — used for delegation|
|**Type CNAME record**|Maps an alias hostname to its canonical hostname|
|**Type MX record**|Maps an alias hostname to the canonical name of a mail server|
|**DNS message**|The query/reply message format shared by both DNS query and reply messages — header + question + answer + authority + additional sections|
|**nslookup**|A command-line tool for sending DNS queries directly to a specified DNS server|
|**Registrar**|A commercial entity (accredited by ICANN) that verifies domain name uniqueness and enters it into the DNS database|
|**ICANN**|Internet Corporation for Assigned Names and Numbers — accredits domain registrars|
|**DNSSEC**|DNS Security Extensions — cryptographically signs DNS records to prevent spoofing and cache poisoning|
|**DNS cache poisoning**|An attack that injects forged resource records into a DNS server's cache to redirect victims|
|**DNS amplification attack**|A DDoS technique exploiting open recursive resolvers to send oversized replies to a spoofed victim address|

---

## Related Concepts

- [[2.2 - The Web and HTTP]] — DNS resolution is the prerequisite step before any HTTP request can be sent; the hostname in a URL must be resolved before the TCP connection in Section 2.2 can even begin
- [[2.4 - Electronic Mail in the Internet]] — Mail server aliasing (Type MX records) directly extends the mail server architecture discussed there; SMTP servers rely on DNS MX lookups to find each other
- [[2.3 - File Transfer FTP]] — FTP clients also resolve the remote host's hostname via DNS before establishing the control connection

---

→ Next: [[2.6 - Peer-to-Peer Applications]]