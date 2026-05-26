---
title: THE WEB & HTTP
date: 2026-05-26
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.2 The Web and HTTP

> **One-Line Summary:** HTTP is the Web's application-layer protocol — it runs over TCP, is stateless by design, and comes in persistent and non-persistent variants; state is layered back on top using cookies, and performance is improved using Web caches (proxy servers) and the conditional GET mechanism.

---

## Core Idea

The **World Wide Web** is an application that operates **on demand** — unlike broadcast TV or radio, the user pulls exactly what they want, exactly when they want it. It has arguably become the defining application of the modern Internet.

The Web's application-layer protocol is **HTTP (HyperText Transfer Protocol)**. HTTP is what your browser speaks to every web server on the planet. Understanding HTTP is foundational for both web development and cybersecurity — every web attack ultimately operates at this layer.

---

# 2.2.1 Overview of HTTP

## Web Pages and Objects

> A **Web page** (also called a document) consists of **objects**.

An **object** is simply a file — it could be:

- An HTML file
- A JPEG or PNG image
- A JavaScript file
- A CSS stylesheet
- A Java applet or a video clip

Each object is **addressable by a single URL**. Most Web pages have the following structure:

```
Web Page
├── base HTML file        ← the skeleton; references everything else
├── object 1 (JPEG)
├── object 2 (JPEG)
├── object 3 (CSS)
└── object 4 (JS)
```

**Concrete example:** A Web page that has an HTML file and five JPEG images = **6 objects total**. The base HTML file contains the URLs pointing to each image. When your browser fetches the page, it first downloads the HTML, then parses it to discover all the referenced objects, then fetches those too.

**URL Structure:**

```
http://www.someSchool.edu/someDepartment/picture.gif
       ──────────────────  ──────────────────────────
          Hostname                 Path name
```

- **Hostname** → identifies which server holds the object
- **Path name** → identifies which specific file on that server

---

## Browsers and Servers

|Role|What it does|Examples|
|---|---|---|
|**Web browser**|Implements the **client side** of HTTP — sends requests, renders responses|Firefox, Chrome, Internet Explorer, Safari|
|**Web server**|Implements the **server side** of HTTP — stores objects, listens for requests, sends responses|Apache, Microsoft IIS, Nginx|

> In the context of HTTP, the terms _browser_ and _client_ are used interchangeably. Same for _web server_ and _server_.

---

## What HTTP Actually Does

> **HTTP defines how Web clients request Web pages from Web servers, and how servers transfer Web pages to clients.**

The fundamental interaction is **request → response**:

1. User clicks a link (or types a URL)
2. Browser sends an **HTTP request message** to the server asking for an object
3. Server receives the request, finds the object, wraps it in an **HTTP response message**, and sends it back
4. Browser renders the received object

![[Pasted image 20260526213043.png]] _(Figure 2.6 — HTTP request-response behavior: a PC running Internet Explorer and a Linux machine running Firefox both talk to an Apache web server via HTTP request and response messages)_

This is why HTTP is called a **request-response protocol** — every interaction is a client asking for something and a server answering.

---

## HTTP Runs Over TCP

> HTTP uses **TCP** as its underlying transport protocol — not UDP.

**The sequence:**

1. HTTP client initiates a **TCP connection** with the server (default port **80** for HTTP, **443** for HTTPS)
2. Once the TCP connection is established, both sides access it through their **socket interfaces** — the socket is the door between the application process and the TCP connection
3. The client pushes an HTTP request through its socket into the TCP connection
4. The server reads the HTTP request from its socket, processes it, and pushes the HTTP response back through its socket
5. The response travels through TCP and arrives at the client's socket

**Why TCP and not UDP?**

HTTP needs **every single byte** of every object to arrive correctly. If even one byte of an HTML file or image is lost, the page is broken. TCP's **reliable data transfer** guarantees correct, in-order delivery. UDP offers no such guarantee and would be completely inappropriate here — you cannot render a partially received image.

---

## HTTP is Stateless — By Design

> HTTP is a **stateless protocol** — the server maintains **no information whatsoever** about the clients between requests.

What this means in practice:

- If the same client requests the same object twice in 5 seconds, the server treats both as completely fresh, independent requests
- The server has no memory of the first request when processing the second
- No session tables, no client history, nothing stored between requests

**Why stateless?**

Stateful protocols are complex and fragile:

- Servers must maintain state tables for every client
- If a server crashes, those tables are lost — clients and server are now out of sync
- State is expensive to maintain at scale (millions of concurrent users)

Statelessness makes servers **simpler, faster, and easier to scale**.

> The tradeoff: statelessness makes HTTP simple but means you cannot do things like "shopping carts" or "login sessions" natively. The solution is **cookies** (Section 2.2.4) — state layered on top of a stateless protocol.

---

# 2.2.2 Non-Persistent and Persistent Connections

A fundamental design question when building an application-layer protocol: should each request/response pair be sent over its own **separate TCP connection**, or should all requests and responses flow over the **same TCP connection**?

|Type|Description|
|---|---|
|**Non-persistent connections**|Separate TCP connection for **each** request/response pair — open, use once, close|
|**Persistent connections**|**Same** TCP connection reused for multiple request/response pairs|

> **HTTP/1.0** → non-persistent (default) **HTTP/1.1** → persistent (default)

---

## Non-Persistent HTTP — How It Works

Imagine requesting a web page that has **1 base HTML file + 10 JPEG images** (11 objects total). With non-persistent HTTP:

```
For the base HTML file:
  Step 1 → Client initiates TCP connection to server on port 80
  Step 2 → Client sends HTTP GET request for the HTML file
  Step 3 → Server receives request, finds the HTML file,
            wraps it in HTTP response, sends it back
  Step 4 → Server signals TCP to close the connection
            (TCP waits until client has fully received the data)
  Step 5 → Client receives the response. TCP connection terminates.
            Client parses the HTML, discovers 10 JPEG references.

For each of the 10 JPEGs:
  Repeat Steps 1–4 for each image (10 separate TCP connections)

Total TCP connections opened = 11 (one per object)
```

> Each TCP connection transports exactly **one request message and one response message** — that is what "non-persistent" means.

**Problems:**

1. **Overhead per object** — A brand-new TCP connection must be set up for each object. This means buffers allocated, TCP variables initialized, and the three-way handshake performed — at both client and server — for every single object. A web server handling thousands of clients simultaneously feels this pain acutely.
    
2. **2 RTTs per object** — Each object costs at minimum two full round trips before delivery begins (one RTT for TCP setup, one RTT for the HTTP request+response). On a high-latency connection, this is brutal.
    

---

## RTT — Round-Trip Time

> The **round-trip time (RTT)** is the time it takes for a small packet to travel from client to server and then **back** to the client.

RTT captures all the delays in between:

- **Propagation delay** — speed of light over the physical medium
- **Queuing delay** — waiting in router buffers along the path
- **Processing delay** — time routers take to examine packet headers

![[Pasted image 20260526213247.png]] _(Figure 2.7 — Timeline showing 2 RTTs needed to fetch a single file with non-persistent HTTP: RTT 1 for TCP handshake, RTT 2 for the HTTP request and first bytes of response, plus the file transmission time at the end)_

**Response time formula for non-persistent HTTP (one object):**

```
Response time = 2 × RTT + file transmission time

Where:
  RTT 1 → TCP 3-way handshake (SYN from client → SYN-ACK from server)
           The ACK + HTTP request are combined in the third message
  RTT 2 → HTTP request travels to server, response begins arriving
  + file transmission time → time to push all bytes of the file
```

**For a full page with N objects (serial):**

```
Total time = (2 RTT + HTML transmission time)
           + N × (2 RTT + object transmission time)
```

For a page with 10 images, that's **22 RTTs** worth of handshake overhead alone, before counting any actual file transmission. Browsers partially mitigate this by opening **multiple parallel TCP connections** (typically 5–10), but non-persistent HTTP is still fundamentally inefficient.

---

## Persistent HTTP (HTTP/1.1 Default)

> With **persistent connections**, the server **leaves the TCP connection open** after sending a response. All subsequent requests and responses between the same client-server pair go over the same TCP connection.

**The key insight:** once the TCP connection is warmed up and the three-way handshake is paid for, why throw it away?

**Two flavours of persistent HTTP:**

|Mode|Behaviour|Cost|
|---|---|---|
|**Persistent without pipelining**|Client waits for each response before sending the next request|~1 RTT per object (vs 2 for non-persistent) — still sequential|
|**Persistent with pipelining** _(HTTP/1.1 default)_|Client sends all requests back-to-back without waiting for any response — requests are "pipelined"|~1 RTT total for all objects on same server|

**Why pipelining is a big deal:**

Without pipelining, fetching 10 objects still takes 10 sequential RTTs (just without the TCP setup cost each time). With pipelining, the client fires all 10 GET requests in quick succession and the server streams back all 10 responses — the whole batch costs roughly 1 RTT regardless of N.

The server closes a persistent connection when it has been idle for a configurable **timeout interval** (typically around 75 seconds for Apache). The client can also explicitly request closure with the `Connection: close` header.

---

# 2.2.3 HTTP Message Format

There are exactly two types of HTTP messages: **request messages** (client → server) and **response messages** (server → client). Both are written in **ASCII text** — you can read them with your eyes, which makes HTTP easy to debug.

---

## HTTP Request Message

### Structure

![[Pasted image 20260526213422.png]] _(HTTP request message general format: Request line at top, followed by Header lines, then a blank line (\r\n), then the optional Entity body used with POST)_

**Every HTTP request message has this exact layout:**

```
Request line   →  method  SP  URL  SP  HTTP-version  CRLF
Header line 1  →  field-name: value  CRLF
Header line 2  →  field-name: value  CRLF
...
(blank line)   →  CRLF
Entity body    →  (only present for POST; empty for GET)
```

`SP` = a space character, `CRLF` = carriage return + line feed (`\r\n`)

**Concrete example of a real HTTP request:**

```
GET /somedir/page.html HTTP/1.1\r\n
Host: www.someschool.edu\r\n
Connection: close\r\n
User-agent: Mozilla/5.0\r\n
Accept-language: fr\r\n
\r\n
```

**Breaking down the request line:**

- `GET` → the method (what operation the client wants)
- `/somedir/page.html` → the URL (which object)
- `HTTP/1.1` → the protocol version

**Breaking down each header line:**

|Header|Value in example|What it means|
|---|---|---|
|`Host:`|`www.someschool.edu`|Hostname of the server. **Required by HTTP/1.1.** Used by proxy caches and servers hosting multiple sites on one IP|
|`Connection:`|`close`|Tells server: "don't bother keeping this connection alive — close it after the response." Overrides HTTP/1.1's default persistent behaviour|
|`User-agent:`|`Mozilla/5.0`|Which browser is making the request. Servers can use this to send browser-specific versions of a page (e.g. mobile-optimised HTML for a mobile UA)|
|`Accept-language:`|`fr`|Client prefers a French version of the object if available; if not, send the default. This is content negotiation|

---

### HTTP Methods — Complete Reference

|Method|Body?|Description|Use case|
|---|---|---|---|
|**GET**|No|Request the object at the specified URL|Fetching any web resource (pages, images, APIs)|
|**POST**|Yes|Submit data to the server; data is in the entity body|HTML form submissions (login forms, search, etc.)|
|**HEAD**|No|Like GET but server only sends the headers — no object body|Debugging; checking if an object exists/was modified without downloading it|
|**PUT**|Yes|Upload an object to a specific path on the server|REST APIs; uploading a resource|
|**DELETE**|No|Delete the object at the specified URL|REST APIs; removing a resource|

> **GET with form data:** When a browser submits a form using GET (not POST), form data is appended to the URL: `www.site.com/search?query=networking&lang=en`. The `?` separates the URL from form data; `&` separates fields. This is why search queries appear in your browser's address bar.

> **POST vs GET for forms:** POST puts form data in the entity body (invisible in URL); GET puts it in the URL (visible, logged by servers and proxies). **Never use GET for sensitive data like passwords.**

---

## HTTP Response Message

### Structure

![[Pasted image 20260526213610.png]] _(HTTP response message general format: Status line at top with version, status code, and phrase; followed by Header lines; then a blank line (\r\n); then the Entity body containing the actual requested object)_

**Every HTTP response message follows this layout:**

```
Status line    →  HTTP-version  SP  status-code  SP  phrase  CRLF
Header line 1  →  field-name: value  CRLF
Header line 2  →  field-name: value  CRLF
...
(blank line)   →  CRLF
Entity body    →  the requested object (HTML, JPEG, etc.)
```

**Concrete example of a real HTTP response:**

```
HTTP/1.1 200 OK\r\n
Connection: close\r\n
Date: Tue, 09 Aug 2011 15:44:04 GMT\r\n
Server: Apache/2.2.3 (CentOS)\r\n
Last-Modified: Tue, 09 Aug 2011 15:11:03 GMT\r\n
Content-Length: 6821\r\n
Content-Type: text/html\r\n
\r\n
(data data data ...)
```

**Breaking down the status line:**

- `HTTP/1.1` → server's protocol version
- `200` → status code (numeric result of the request)
- `OK` → human-readable phrase explaining the code

**Breaking down each header line:**

|Header|Example value|What it means|
|---|---|---|
|`Connection:`|`close`|Server will close TCP connection after this response|
|`Date:`|`Tue, 09 Aug 2011 15:44:04 GMT`|The **exact time the server created and sent this response** — not when the object was created, but when this specific message was assembled|
|`Server:`|`Apache/2.2.3 (CentOS)`|Server software version. Useful for debugging; a **privacy/security risk** in production (tells attackers what to target)|
|`Last-Modified:`|`Tue, 09 Aug 2011 15:11:03 GMT`|When this object was last modified on the server. **Critical for caching** — used by the conditional GET mechanism|
|`Content-Length:`|`6821`|Size of the entity body in bytes. Tells the client exactly how many bytes to expect|
|`Content-Type:`|`text/html`|MIME type of the entity body — tells the browser what kind of object it received and how to render it|

---

### HTTP Status Codes — Complete Reference

The **three-digit status code** is the most important part of the response for programmatic processing. Grouped by category:

|Range|Category|Meaning|
|---|---|---|
|**1xx**|Informational|Request received, continuing process|
|**2xx**|Success|Request was successfully received, understood, and accepted|
|**3xx**|Redirection|Further action needed to complete the request|
|**4xx**|Client Error|Request has bad syntax or cannot be fulfilled — **client's fault**|
|**5xx**|Server Error|Server failed to fulfil an apparently valid request — **server's fault**|

**Most important codes to know:**

|Code|Phrase|Meaning|When you see it|
|---|---|---|---|
|**200**|OK|Request succeeded. Object is in the entity body|Normal successful fetch|
|**301**|Moved Permanently|Object has permanently moved to a new URL given in the `Location:` header. Browser automatically follows the redirect|Site has changed its URL structure|
|**304**|Not Modified|Sent in response to conditional GET. Object unchanged — client should use its cached copy. **No entity body**|Cache validation (see 2.2.6)|
|**400**|Bad Request|Server could not understand the request — malformed syntax|Buggy client code; malformed request|
|**404**|Not Found|Requested object does not exist on this server|Broken links, deleted pages, typos in URL|
|**505**|HTTP Version Not Supported|Server does not support the HTTP version in the request|Version mismatch between client and server|

> **Practical exercise — see a real HTTP response:** Open a terminal and type:

```
telnet cis.poly.edu 80
GET /~ross/ HTTP/1.1
Host: cis.poly.edu
```
 
> Press Enter twice. You will see a live HTTP response message — status line, all headers, then the HTML body. Use `HEAD` instead of `GET` to see only headers without downloading the page body.

---

# 2.2.4 User-Server Interaction — Cookies

**The problem:** HTTP is stateless, but the real Web needs state. Amazon needs to know who you are to show your shopping cart. Gmail needs to know you're logged in. A news site wants to remember your reading preferences.

**The solution:** **Cookies** — a mechanism for layering state on top of a stateless protocol.

> Cookies are standardised in RFC 6265.

---

## The Four Components of the Cookie System

1. A `Set-cookie:` header line in the **HTTP response** (server → client)
2. A `Cookie:` header line in the **HTTP request** (client → server)
3. A **cookie file** stored on the user's machine, managed by the browser
4. A **back-end database** at the Web site, indexed by cookie value

All four must be in place for the system to work.

---

## How Cookies Work — Step by Step

**Scenario:** Susan visits Amazon for the first time from her home PC.

```
Visit 1 — No cookie yet:
  Susan's browser → Amazon server
  (usual HTTP request, no Cookie: header)

  Amazon server:
  → Notices: no cookie in request = new user
  → Creates a unique ID for Susan: 1678
  → Creates an entry in its database: { 1678: Susan's activity }
  → Sends HTTP response INCLUDING:
      Set-cookie: 1678

  Susan's browser:
  → Reads the Set-cookie header
  → Appends to Susan's cookie file: amazon.com → 1678
```

```
Visit 2 (and all subsequent visits):
  Susan's browser → Amazon server
  → Browser checks cookie file, finds entry for amazon.com: 1678
  → Automatically adds to request:
      Cookie: 1678

  Amazon server:
  → Reads Cookie: 1678
  → Looks up 1678 in database
  → Knows this is Susan, retrieves her history, preferences, cart
  → Returns personalised content
```

```
One week later:
  Same thing — browser still has the cookie, still sends Cookie: 1678
  Amazon still has the database entry
  Susan gets her shopping cart back
```

![[Pasted image 20260526213931.png]] _(Figure 2.10 — Keeping user state with cookies: the server creates the ID and sets the cookie in the response; all subsequent requests carry the Cookie header; the server's backend database maps ID to user data)_

**What websites use cookies for:**

|Use case|How cookies enable it|
|---|---|
|Session authentication|Cookie value identifies a logged-in session — server knows you don't need to log in again|
|Shopping carts|Cart contents stored in database, keyed by cookie ID|
|Personalisation|Preferences, language, layout stored server-side, retrieved by cookie|
|Recommendations|Purchase/browse history tracked in database against cookie ID|
|Third-party tracking|Ad networks set their own cookies across many sites to build cross-site browsing profiles|

**The privacy concern:**

Cookies allow websites to build detailed profiles of users across time and even across sites (via third-party cookies from ad networks). A site can combine cookie-tracked browsing behaviour with account information (name, email, address) to create an extremely detailed profile. This is the fundamental tension between personalisation and privacy — and it drives ongoing debates about cookie consent laws (GDPR, etc.).

---

# 2.2.5 Web Caching

> A **Web cache** — also called a **proxy server** — is a network entity that satisfies HTTP requests on behalf of an origin Web server.

The Web cache has its own local disk storage. When it receives a request for an object, it checks if it has a stored copy. If yes, it serves it locally. If no, it fetches it from the origin server, stores a copy, and forwards it to the client.

![[Pasted image 20260526214140.png]] _(Figure 2.11 — Clients requesting objects through a Web cache: both clients talk to the proxy server; the proxy fetches from origin servers only when its local copy is missing or stale)_

---

## How a Web Cache Satisfies a Request — Step by Step

```
Step 1:
  Browser sends HTTP request to the Web cache
  (browser is configured to route all requests through the cache)

Step 2:
  Web cache checks its local disk storage
  → HIT: cache has a copy → sends it directly to browser in HTTP response
          ✓ Origin server not contacted. Fast.
  → MISS: cache does not have the object → proceed to Step 3

Step 3 (cache miss only):
  Web cache opens a TCP connection to the origin server
  Sends an HTTP request for the object to the origin server
  Origin server sends back an HTTP response with the object

Step 4:
  Web cache receives the object
  → Stores a copy in local disk storage
  → Forwards the object to the browser in an HTTP response
    (over the existing TCP connection between browser and cache)
```

> **Important:** A Web cache is simultaneously a **server** (it responds to browsers' requests) and a **client** (it makes requests to origin servers). It plays both roles, which is why it is also called a "proxy."

---

## Who Installs and Operates Web Caches?

> Typically a Web cache is purchased and installed by an **ISP**.

- A **university** installs a cache on the campus network → all campus browsers are configured to point to it. Students browsing popular content hit the campus cache instead of the Internet.
- A **residential ISP** (like a cable company) installs caches in its regional infrastructure → reduces traffic on expensive backbone links.
- Both cases reduce costs and improve speed for users.

---

## Why Web Caching? — Two Quantitative Reasons

### Reason 1 — Reduce Client Response Time

> A cache can **substantially reduce response time**, especially when the bottleneck bandwidth between client and origin server is much lower than the bandwidth between client and cache.

If a high-speed LAN connects the client to the cache, and the cache has the object, the client gets it at LAN speed (~100 Mbps or 1 Gbps) instead of waiting for it to cross a congested 15 Mbps Internet link. The difference is dramatic.

### Reason 2 — Reduce Traffic on Access Links (and Cost)

![[Pasted image 20260526214407.png]] _(Figure 2.12 — Bottleneck scenario: institutional network with 100 Mbps LAN connected to the public Internet via a 15 Mbps access link. Origin servers are scattered across the Internet.)_

**Given parameters:**

- Average object size: **1 Mbit**
- Average browser request rate: **15 requests/second**
- Average data rate to browsers: **15 Mbps**
- Access link capacity: **15 Mbps**
- Average Internet delay (router on Internet side → origin server → back): **2 seconds**

**Scenario A — No cache:**

```
Traffic intensity on access link:
  = Data rate / Link capacity
  = 15 Mbps / 15 Mbps
  = 1.0

Traffic intensity = 1.0 → queuing delay approaches INFINITY
→ Total response time ≈ minutes (completely unusable)
```

**Scenario B — Upgrade access link to 100 Mbps:**

```
Traffic intensity on access link:
  = 15 Mbps / 100 Mbps
  = 0.15

→ Queuing delay ≈ milliseconds (negligible)
→ Total delay ≈ 2 sec (Internet delay) + milliseconds
             ≈ ~2 seconds

Cost: Significant — leasing a 100 Mbps Internet connection
      is much more expensive than a 15 Mbps one
```

**Scenario C — Install a local Web cache (hit rate = 0.4):**

![[Pasted image 20260526214557.png]] _(Figure 2.13 — Web cache added to institutional network: cache sits inside the 100 Mbps LAN. 40% of requests are served locally; 60% go out over the 15 Mbps access link.)_

```
40% of requests → satisfied by cache (LAN speed)
  Delay ≈ a few milliseconds ≈ ~0 seconds

60% of requests → must fetch from origin server over access link
  Traffic intensity on access link:
    = (0.6 × 15 Mbps) / 15 Mbps = 0.6
    → Queuing delay is small (0.6 << 1.0)
    → Delay ≈ 2 sec Internet delay + small queuing ≈ ~2 seconds

Average total delay:
  = 0.4 × (0 sec) + 0.6 × (2 sec)
  = 0 + 1.2
  = 1.2 seconds

Cost: An inexpensive PC running open-source cache software
      (e.g. Squid) — far cheaper than upgrading the access link
```

> **Result: 1.2 seconds average response time at a fraction of the cost of the 100 Mbps link upgrade.** Web caches are one of the most cost-effective performance improvements in networking.

---

## Content Distribution Networks (CDNs)

> Through **Content Distribution Networks (CDNs)**, the concept of web caching is deployed at Internet scale.

A CDN company installs many **geographically distributed caches** (sometimes hundreds of locations worldwide) throughout the Internet. When a user in Kolkata requests a YouTube video, they are served by a CDN node in India rather than a server in California — drastically reducing latency and backbone traffic.

|Type|Examples|
|---|---|
|**Shared CDN** (serves multiple companies' content)|Akamai, Limelight, Cloudflare|
|**Dedicated CDN** (serves one company's own content)|Google (YouTube), Microsoft (Azure CDN), Netflix (Open Connect)|

CDNs are covered in much greater detail in Chapter 7.

---

# 2.2.6 The Conditional GET

**The problem:** Caching introduces a new issue. The cached copy of an object might be **stale** — the origin server might have updated the object since the cache stored it. Serving a stale copy to the user is wrong.

**The solution:** The **conditional GET** — a mechanism for a cache to verify whether its stored copy is still fresh without unnecessarily re-downloading the entire object if it hasn't changed.

> An HTTP request is a **conditional GET** if: (1) it uses the `GET` method AND (2) it includes an `If-Modified-Since:` header line.

---

## How the Conditional GET Works — Full Example

**Step 1 — Cache fetches object for the first time (no cached copy yet):**

```
Cache → Origin server:
  GET /fruit/kiwi.gif HTTP/1.1
  Host: www.exotiquecuisine.com

Origin server → Cache:
  HTTP/1.1 200 OK
  Date: Sat, 08 Oct 2011 15:39:29 GMT
  Last-Modified: Wed, 05 Oct 2011 09:23:24 GMT
  Content-Type: image/gif
  (object body — the actual image bytes)

Cache stores:
  → A copy of kiwi.gif in local storage
  → The Last-Modified date: Wed, 05 Oct 2011 09:23:24 GMT
```

**Step 2 — One week later, a browser requests the same object:**

The cache has a copy, but is it still fresh? It asks the origin server using a conditional GET:

```
Cache → Origin server:
  GET /fruit/kiwi.gif HTTP/1.1
  Host: www.exotiquecuisine.com
  If-Modified-Since: Wed, 05 Oct 2011 09:23:24 GMT
                     ↑ this is the date the cache received it
```

**Step 3a — Object has NOT been modified since that date:**

```
Origin server → Cache:
  HTTP/1.1 304 Not Modified
  Date: Sat, 15 Oct 2011 15:39:29 GMT
  (empty entity body — no object bytes transmitted)

Cache → Browser:
  Forwards its locally stored copy of kiwi.gif
```

**Step 3b — Object HAS been modified since that date:**

```
Origin server → Cache:
  HTTP/1.1 200 OK
  Date: Sat, 15 Oct 2011 15:39:29 GMT
  Last-Modified: Mon, 10 Oct 2011 14:22:00 GMT
  (new object body — updated image bytes)

Cache:
  → Replaces old copy with the new one
  → Updates the stored Last-Modified date
  → Forwards the new object to the browser
```

> **Key insight:** The `304 Not Modified` response has an **empty body** — no object bytes are transmitted. The cache already has the object; the server is just confirming "yours is still good." This saves bandwidth while ensuring correctness.

**Summary of conditional GET:**

```
Last-Modified (from 200 OK)
  → Cache stores this date alongside the object

If-Modified-Since (in conditional GET)
  → Cache sends this date back to server on next check

304 Not Modified (response with empty body)
  → Object unchanged → use cached copy → no bandwidth wasted

200 OK (response with full body)
  → Object changed → receive new copy → update cache
```

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**HTTP is cleartext**|All data — including passwords, cookies, and session tokens — is visible to anyone on the same network (e.g. public WiFi). Passive sniffing with tools like Wireshark trivially captures it|Always use **HTTPS** (HTTP over TLS). The `Secure` flag on cookies ensures they are never sent over HTTP|
|**Cookies — Session Hijacking**|If an attacker steals a user's `Cookie: 1678` header (via sniffing, XSS, etc.), they can replay it and impersonate that user entirely|`HttpOnly` flag (JavaScript cannot access the cookie), `Secure` flag (only sent over HTTPS), `SameSite=Strict` (not sent in cross-origin requests)|
|**Cookies — CSRF**|A malicious website can trick a logged-in user's browser into sending a forged request to another site — the browser automatically includes the cookie|CSRF tokens (random secret embedded in forms), `SameSite` cookie attribute|
|**HTTP Methods (PUT / DELETE)**|If a server exposes PUT or DELETE without proper authentication, an attacker can overwrite or delete server-side content|All non-GET/HEAD methods must require authentication and authorisation. REST APIs must validate permissions on every request|
|**Proxy / Cache Poisoning**|A malicious proxy can serve modified or malicious objects to all clients routed through it. Cache poisoning attacks inject crafted HTTP responses into shared caches, where they get served to many victims|HTTPS end-to-end encryption prevents a proxy from reading or modifying content. HTTPS with certificate validation prevents MITM via rogue proxies|
|**Information Leakage via Headers**|The `Server:` header reveals software and version (e.g. `Apache/2.2.3 (CentOS)`) — attackers target known CVEs for that version. `Last-Modified` can leak when internal files were changed|Strip or spoof the `Server:` header in production. Return generic error pages for 404/400/505 (don't expose directory structure)|
|**GET with Sensitive Form Data**|Passwords or tokens in GET URLs are stored in browser history, server access logs, proxy logs, and leaked in `Referer` headers to third-party resources on the target page|Use POST for any sensitive form data. HTTPS encrypts URLs in transit but they still appear in server-side logs — never put secrets in URLs|
|**CDN Supply Chain Attack**|Compromise of a CDN (or a shared CDN's customer) can serve malicious JavaScript to millions of users globally|**Subresource Integrity (SRI)** — browsers verify a cryptographic hash of fetched scripts. HTTPS everywhere. Certificate pinning for critical resources|

---

## Questions I Still Have

- [ ] HTTP/1.1 persistent connections — if the server's timeout fires while the client is mid-think-time, does the client get an error on its next request, or does it silently reconnect?
- [ ] Pipelining sounds great, but HTTP/1.1 still has **head-of-line blocking** (a slow response blocks later responses in the queue). How does **HTTP/2 multiplexing** solve this at the framing layer?
- [ ] Does HTTPS break web caching entirely? If traffic is encrypted end-to-end, the proxy can't read or cache the content — how do CDNs work with HTTPS?
- [ ] What exactly is the `Referer` [sic] header — when is it sent, what does it contain, and why is it a privacy risk for users browsing sensitive pages?
- [ ] CDNs route users to the nearest cache — is this done via **Anycast DNS** (same IP, multiple locations) or something else entirely?
- [ ] HTTP/2 and HTTP/3 (QUIC over UDP) — how do they change the persistent/non-persistent connection model? Does RTT analysis still apply the same way?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**HTTP**|HyperText Transfer Protocol — the application-layer protocol of the Web|
|**Web page**|A document consisting of a base HTML file and referenced objects|
|**Object**|Any file addressable by a single URL (HTML, JPEG, CSS, JS, video, etc.)|
|**URL**|Uniform Resource Locator — hostname + path name, together identifying one object|
|**Web browser**|Client-side implementation of HTTP (Firefox, Chrome, Safari, IE)|
|**Web server**|Server-side implementation of HTTP — stores objects, responds to requests|
|**Stateless protocol**|Protocol that maintains zero information about past client interactions|
|**Non-persistent HTTP**|Each request/response pair uses its own TCP connection — open, use once, close|
|**Persistent HTTP**|Multiple requests/responses share the same TCP connection — HTTP/1.1 default|
|**RTT (Round-Trip Time)**|Time for a small packet to travel from client to server and back|
|**Pipelining**|Sending multiple HTTP requests back-to-back without waiting for prior responses|
|**Request line**|First line of HTTP request — method, URL, HTTP version|
|**Status line**|First line of HTTP response — HTTP version, status code, phrase|
|**Entity body**|The payload of an HTTP message (POST request data; response object bytes)|
|**GET**|HTTP method to retrieve an object|
|**POST**|HTTP method to submit data to the server (data in entity body)|
|**HEAD**|HTTP method that retrieves only response headers, not the object body|
|**PUT**|HTTP method to upload an object to a specific URL on the server|
|**200 OK**|Request succeeded; object is in the entity body|
|**301 Moved Permanently**|Object has a new permanent URL; browser should follow the `Location:` header|
|**304 Not Modified**|Conditional GET result — cached copy is still valid; no body transmitted|
|**404 Not Found**|Requested object does not exist on this server|
|**Cookie**|A small piece of state stored by the browser, sent in every request to the issuing site|
|**Set-cookie**|Response header instructing the browser to store a cookie value|
|**Web cache / Proxy server**|Network entity that stores recently requested objects and serves them locally|
|**Hit rate**|Fraction of all requests satisfied by a Web cache from its local storage|
|**CDN**|Content Distribution Network — geographically distributed caches serving content closer to users|
|**Conditional GET**|HTTP GET with `If-Modified-Since:` header — asks server if object has changed|
|**If-Modified-Since**|Request header carrying the date of the cached copy for freshness checking|
|**Last-Modified**|Response header indicating when the server object was last changed|
|**304 Not Modified**|Response to conditional GET confirming cached copy is current — empty body|

---

## Related Concepts

-

---

→ Next: [[2.3 - File Transfer FTP]]