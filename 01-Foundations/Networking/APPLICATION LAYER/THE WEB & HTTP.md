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

> **One-Line Summary:** HTTP is the Web's application-layer protocol — it runs over TCP, is stateless by design, and comes in persistent and non-persistent variants; state is layered back on top using cookies, and performance is improved using Web caches (proxy servers).

---

## Core Idea

The **World Wide Web** is an application that operates **on demand** — users get what they want when they want it, unlike broadcast TV or radio. It has captured the public imagination and transformed how humans communicate, work, and consume information.

The Web's application-layer protocol is **HTTP (HyperText Transfer Protocol)**.

---

# 2.2.1 Overview of HTTP

## Web Pages and Objects

> A **Web page** (also called a document) consists of **objects**.

- An **object** is a file — an HTML file, a JPEG image, a Java applet, a video clip — addressable by a single URL
- Most Web pages consist of:
    - A **base HTML file**
    - Several **referenced objects** embedded in the base HTML

**Example:** A Web page with HTML text and five JPEG images has **six objects** — the base HTML file plus five images. The base HTML file references the others via their URLs.

**URL structure:**

```
http://www.someSchool.edu/someDepartment/picture.gif
       ↑                  ↑
       Hostname            Path name
```

Each URL has two components: the **hostname** of the server that houses the object and the **path name** of the object on that server.

---

## Browsers and Servers

|Role|Description|Examples|
|---|---|---|
|**Web browsers**|Implement the **client side** of HTTP|Internet Explorer, Firefox, Chrome|
|**Web servers**|Implement the **server side** of HTTP — house Web objects, each addressable by a URL|Apache, Microsoft IIS|

> We use the words _browser_ and _client_ interchangeably in the context of the Web.

---

## What HTTP Does

> **HTTP defines how Web clients request Web pages from Web servers and how servers transfer Web pages to clients.**

![[HTTP_request_response.png]] _(Figure 2.6 — HTTP request-response behavior: browser sends HTTP request messages for objects; server responds with HTTP response messages containing the objects)_

**The interaction:**

- When a user requests a Web page (e.g. clicks a hyperlink), the browser sends **HTTP request messages** for the objects in that page to the server
- The server receives the requests and responds with **HTTP response messages** containing the objects

---

## HTTP and TCP

> HTTP uses **TCP** as its underlying transport protocol (not UDP).

**How it works:**

1. The HTTP client first **initiates a TCP connection** with the server
2. Once the connection is established, the browser and server processes access TCP through their **socket interfaces**
3. The client-side socket is the door between the client process and the TCP connection; the server-side socket is the door between the server process and the TCP connection
4. HTTP messages flow between sockets in both directions

**Why TCP?**

- HTTP is a reliable-transfer application — you need all the bytes of an HTML page and all embedded objects to arrive correctly
- TCP's reliable data transfer guarantees this; UDP would not

---

## HTTP is Stateless

> HTTP is a **stateless protocol** — an HTTP server maintains **no information** about the clients.

- The server sends requested files to clients without storing any client state
- If a client asks for the same object twice in a few seconds, the server does not notice — it just sends it again
- **Stateful protocols** are complex: servers must maintain state, handle crashes that corrupt state, etc.
- **Stateless protocols** are simpler and more scalable

> This is a deliberate design decision. State is added back on top using **cookies** (Section 2.2.4) when needed.

---

# 2.2.2 Non-Persistent and Persistent Connections

A key design question: should each request/response pair go over a **separate TCP connection**, or should they all flow over the **same TCP connection**?

- **Non-persistent connections** — each request/response pair uses a separate TCP connection
- **Persistent connections** — all requests and responses use the same TCP connection

HTTP/1.0 used non-persistent connections. **HTTP/1.1 uses persistent connections by default.**

---

## Non-Persistent HTTP

**Step-by-step — requesting a Web page with 10 JPEG objects:**

```
1. HTTP client initiates TCP connection to server (port 80)
2. HTTP client sends HTTP request message (containing URL) into TCP socket
3. HTTP server receives request, retrieves object from storage,
   encapsulates it in an HTTP response message, sends it into socket
4. HTTP server tells TCP to close the connection
   (TCP waits until client has received the response reliably)
5. HTTP client receives response. TCP connection terminates.
   Client extracts the base HTML file, finds 10 JPEG object references.
6. Steps 1–4 repeated for each of the 10 JPEG objects
```

> Each TCP connection transports exactly **one request message and one response message** — hence "non-persistent."

**Problems with non-persistent HTTP:**

- A **brand-new TCP connection** must be established for each requested object — buffers must be allocated and TCP variables maintained at both client and server → significant burden for a Web server handling requests from hundreds of clients simultaneously
- **Each object suffers a delivery delay of two RTTs** (one RTT to establish TCP, one RTT to request and receive the object) plus transmission time

---

## RTT and Non-Persistent HTTP Response Time

> The **round-trip time (RTT)** is the time it takes for a small packet to travel from client to server and then back to the client.

RTT includes:

- Packet-propagation delays
- Packet-queuing delays in intermediate routers and switches
- Packet-processing delays

![[RTT_diagram.png]] _(Figure 2.7 — Back-of-the-envelope calculation for time needed to request and receive an HTML file)_

**Non-persistent HTTP response time per object:**

```
Response time = 2 RTT + file transmission time

Breakdown:
  RTT 1 → TCP three-way handshake (first two parts: SYN + SYN-ACK)
  RTT 2 → HTTP request + first few bytes of HTTP response to arrive
  + file transmission time
```

**Total for a page with N objects (serially):**

```
Total = (2 RTT + transmission time for base HTML)
      + N × (2 RTT + transmission time per object)
```

Browsers often use **parallel TCP connections** (e.g., open 5–10 connections simultaneously) to reduce this wall-clock time.

---

## Persistent HTTP (HTTP/1.1 Default)

> With **persistent connections**, the server **leaves the TCP connection open** after sending a response. Subsequent requests and responses between the same client and server are sent over the same connection.

**Variants:**

|Variant|How it works|
|---|---|
|**Without pipelining**|Client issues a new request only when it has received the previous response — one RTT per object after the initial connection|
|**With pipelining** (HTTP/1.1 default)|Client sends requests back-to-back without waiting for responses — all referenced objects can be fetched in roughly **one RTT** total|

**Why persistent + pipelining is much better:**

- No repeated TCP handshake overhead
- No TCP slow-start per object (TCP has warmed up)
- Pipelining means only ~1 RTT needed for multiple objects on the same server

The server closes a persistent connection when it isn't used for a configurable **timeout interval**.

---

# 2.2.3 HTTP Message Format

Two types of HTTP messages: **request** messages and **response** messages.

---

## HTTP Request Message

**Example request:**

```
GET /somedir/page.html HTTP/1.1\r\n
Host: www.someschool.edu\r\n
Connection: close\r\n
User-agent: Mozilla/5.0\r\n
Accept-language: fr\r\n
\r\n
```

**Structure:**

```
┌─────────────────────────────────────────────────┐
│  Request line    → method  SP  URL  SP  version │
├─────────────────────────────────────────────────┤
│  Header line 1   → field-name: value            │
│  Header line 2   → field-name: value            │
│  ...                                            │
├─────────────────────────────────────────────────┤
│  Blank line      → \r\n                         │
├─────────────────────────────────────────────────┤
│  Entity body     → (used with POST)             │
└─────────────────────────────────────────────────┘
```

**The request line has three fields:**

- **Method field** — GET, POST, HEAD, PUT, DELETE
- **URL field** — path to the requested object
- **HTTP version field** — HTTP/1.0, HTTP/1.1

**Important header lines:**

|Header|Meaning|
|---|---|
|`Host:`|Specifies the host on which the object resides — required by Web proxy caches|
|`Connection: close`|Non-persistent — close after sending response|
|`User-agent:`|Browser type making the request — server can send different versions of the same object to different browsers|
|`Accept-language:`|Preferred language version of the object|

---

### HTTP Methods

|Method|Description|Has Body?|
|---|---|---|
|**GET**|Request an object identified by the URL|No (for simple GETs)|
|**POST**|Submit form data — entity body contains user input|Yes|
|**HEAD**|Like GET but server responds with only an HTTP message (no object) — used for debugging|No|
|**PUT**|Upload an object to a specific path on the server|Yes|
|**DELETE**|Delete an object on the server|No|

> **GET with form data:** A browser can also send form data via GET by encoding it in the URL: `www.somesite.com/animalsearch?monkeys&banana` — the `?` separates the URL from the form data and `&` separates fields.

---

## HTTP Response Message

**Example response:**

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

**Structure:**

```
┌─────────────────────────────────────────────────────┐
│  Status line   → version  SP  status code  SP phrase│
├─────────────────────────────────────────────────────┤
│  Header lines  → field-name: value                  │
│  ...                                                │
├─────────────────────────────────────────────────────┤
│  Blank line    → \r\n                               │
├─────────────────────────────────────────────────────┤
│  Entity body   → the requested object               │
└─────────────────────────────────────────────────────┘
```

**Important response header lines:**

|Header|Meaning|
|---|---|
|`Connection: close`|Server will close TCP connection after sending message|
|`Date:`|Time and date when the HTTP response was created and sent by the server|
|`Server:`|Server software that generated the message (analogous to `User-agent`)|
|`Last-Modified:`|Time and date the object was created or last modified — critical for object caching|
|`Content-Length:`|Number of bytes in the object being sent|
|`Content-Type:`|Object type in the entity body (text/html, image/jpeg, etc.)|

---

### HTTP Status Codes

The status line's status code and phrase tell the client what happened.

**Common status codes:**

|Code|Phrase|Meaning|
|---|---|---|
|**200**|OK|Request succeeded; object is in the response body|
|**301**|Moved Permanently|Object has moved to a new URL given in `Location:` header — client automatically retrieves the new URL|
|**304**|Not Modified|Used in conditional GET — object not modified; client can use its cached copy|
|**400**|Bad Request|Generic error — request could not be understood by server|
|**404**|Not Found|Requested document does not exist on this server|
|**505**|HTTP Version Not Supported|Requested HTTP protocol version not supported|

> **Practical tip:** You can see a real HTTP response message by telnetting to a server:
> 
> ```
> telnet cis.poly.edu 80
> GET /~ross/ HTTP/1.1
> Host: cis.poly.edu
> ```
> 
> (Press Enter twice.) This opens a TCP connection to port 80 and sends an HTTP request. Use `HEAD` instead of `GET` to see only the headers without the object body.

---

# 2.2.4 User-Server Interaction — Cookies

HTTP is stateless — but many Web applications need to identify users (shopping carts, personalization, session management). The mechanism: **cookies**.

> Cookies are defined in RFC 6265.

**Four components of the cookie technology:**

1. A `Set-cookie:` header line in the HTTP **response** message
2. A `Cookie:` header line in the HTTP **request** message
3. A cookie file kept on the user's end system, managed by the browser
4. A back-end database at the Web site

**How cookies work (example — first visit to Amazon):**

```
Step 1: User visits Amazon for the first time
        → HTTP request arrives at Amazon server (no cookie header)
        → Server creates a unique ID (e.g., 1678) for the user
        → Server creates an entry in its back-end database for ID 1678
        → Server sends HTTP response with: Set-cookie: 1678

Step 2: Browser appends the cookie to its cookie file
        → Subsequent requests to Amazon include: Cookie: 1678

Step 3: On future visits (even a week later)
        → Browser extracts cookie for Amazon, puts Cookie: 1678 in request
        → Server looks up 1678 in database → knows who you are
        → Can serve personalized content (shopping cart, recommendations)
```

![[cookies_diagram.png]] _(Figure 2.10 — Keeping user state with cookies: Set-cookie header creates the association; Cookie header maintains it on subsequent visits)_

**What cookies can be used for:**

- Shopping carts
- Session authentication (logging in)
- User preferences and personalization
- Tracking browsing behavior (by third-party sites)

**Privacy concern:**

> Cookies can be used to learn a lot about a user. Combined with user-provided account information, a Web site can know a great deal — name, email, credit card, browsing history.

This is a significant privacy issue — "cookies and privacy" is an ongoing debate in the Internet community.

---

# 2.2.5 Web Caching

> A **Web cache** — also called a **proxy server** — is a network entity that satisfies HTTP requests on behalf of an origin Web server.

The Web cache has its own disk storage and keeps copies of recently requested objects.

![[web_cache_diagram.png]] _(Figure 2.11 — Clients requesting objects through a Web cache: the cache sits between clients and origin servers)_

**How a Web cache works (step by step):**

```
1. Browser establishes TCP connection to the Web cache, sends HTTP request

2. Web cache checks if it has a copy of the object in local storage
   → If yes: returns the object in an HTTP response → done

3. If no: Web cache opens TCP connection to the origin server
          Sends HTTP request for the object into cache-to-server connection
          Origin server sends object in HTTP response back to Web cache

4. Web cache stores a copy of the object in local storage
   Sends the copy (in HTTP response) to the client browser
   over the existing TCP connection between client and cache
```

> **Note:** A cache is both a **server** (for browsers making requests) and a **client** (when it fetches from origin servers) at the same time.

---

## Who Installs Web Caches?

> Typically a Web cache is purchased and installed by an **ISP**.

- A university might install a cache on its campus network and configure all campus browsers to point to it
- A major residential ISP might install caches in its network and preconfigure shipped browsers to point to them

---

## Why Web Caching? — Two Reasons

### Reason 1 — Reduce Response Time

> A Web cache can **substantially reduce the response time** for a client request, particularly if the bottleneck bandwidth between client and origin server is much less than the bottleneck bandwidth between client and cache.

If there's a high-speed connection between client and cache, and the cache has the requested object, delivery is near-instant.

### Reason 2 — Reduce Traffic on Access Links

> Web caches can **substantially reduce traffic** on an institution's access link to the Internet.

![[bottleneck_diagram.png]] _(Figure 2.12 — Bottleneck between institutional network and Internet: 15 Mbps access link connecting 100 Mbps LAN to origin servers)_

**Back-of-the-envelope example:**

Given:

- Average object size: 1 Mbits
- Average request rate from institution: 15 requests/second
- Average data rate to browsers: 15 Mbps
- Access link capacity: 15 Mbps
- Internet delay (origin server to access link): ~2 seconds

**Without a cache:**

```
Traffic intensity on access link = (15 requests/sec × 1 Mbit) / 15 Mbps = 1.0
→ Traffic intensity approaching 1 → delays approach infinity
→ Total response time ≈ minutes (unacceptable)
```

**Option 1 — Upgrade access link to 100 Mbps:**

```
Traffic intensity = 15 Mbps / 100 Mbps = 0.15 → small queuing delay
→ Total delay ≈ 2 seconds + (milliseconds queuing) ≈ 2 seconds
Cost: Significant — paying for a much faster Internet connection
```

**Option 2 — Install a Web cache:**

![[cache_institutional.png]] _(Figure 2.13 — Adding a cache to the institutional network: cache sits on the 100 Mbps LAN side)_

Suppose the cache has a **hit rate** of 0.4 (40% of requests satisfied from cache).

```
40% of requests:
  Satisfied by cache over LAN → delay ≈ milliseconds

60% of requests:
  Must go to origin server over access link
  Traffic intensity on access link = (0.6 × 15 Mbps) / 15 Mbps = 0.6
  → Queuing delay is small (traffic intensity 0.6, not 1.0)
  → Delay ≈ 2 seconds + small queuing delay ≈ 2 seconds

Total average delay:
  = 0.4 × (milliseconds) + 0.6 × (2 seconds)
  ≈ 0.4 × 0 + 0.6 × 2
  = 1.2 seconds
```

> Result: Average delay ≈ 1.2 seconds — better than upgrading the link, at **far lower cost**. Web cache software runs on inexpensive PCs.

---

## Content Distribution Networks (CDNs)

> Through the use of **Content Distribution Networks (CDNs)**, Web caches are increasingly playing an important role in the Internet.

- A CDN company installs many **geographically distributed caches** throughout the Internet
- This **localizes much of the traffic**
- Types: **shared CDNs** (e.g. Akamai, Limelight) and **dedicated CDNs** (e.g. Google, Microsoft)

CDNs are covered in more detail in Chapter 7.

---

# 2.2.6 The Conditional GET

**The problem with caching:** the cached copy may be **stale** — the object on the origin server may have been modified after it was cached.

> HTTP has a mechanism to verify that cached objects are up to date: the **conditional GET**.

**An HTTP request message is a conditional GET if:**

1. It uses the GET method
2. It includes an `If-Modified-Since:` header line

**How it works:**

```
Step 1 — Proxy cache sends request to origin server, gets response:
  HTTP/1.1 200 OK
  Date: Sat, 08 Oct 2011 15:39:29
  Last-Modified: Wed, 05 Oct 2011 09:23:24
  Content-Type: image/gif
  (object body...)

  → Cache stores the object and the Last-Modified date

Step 2 — One week later, browser requests same object again
  → Cache sends conditional GET to origin server:
  GET /fruit/kiwi.gif HTTP/1.1
  Host: www.exotiquecuisine.com
  If-Modified-Since: Wed, 05 Oct 2011 09:23:24

Step 3a — Object NOT modified:
  HTTP/1.1 304 Not Modified
  Date: Sat, 15 Oct 2011 15:39:29
  (empty body)
  → Cache forwards its stored copy to the browser
  → No wasted bandwidth transmitting the object again

Step 3b — Object WAS modified:
  HTTP/1.1 200 OK
  (new object in body)
  → Cache stores new copy, sends it to browser
```

> The `304 Not Modified` response tells the cache: "your copy is still good, go ahead and forward it." The response body is empty — no wasted bandwidth.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**HTTP is cleartext**|HTTP sends everything unencrypted — passwords, cookies, session tokens visible to anyone sniffing the wire (e.g. on a shared WiFi)|Always use HTTPS (HTTP over TLS/SSL); see Chapter 8|
|**Cookies**|Cookie theft (session hijacking) — if you steal someone's `Cookie: 1678` header, you can impersonate them. Also: CSRF attacks use cookies to forge authenticated requests|`HttpOnly` flag (JS can't read cookie), `Secure` flag (cookie only over HTTPS), `SameSite` attribute|
|**HTTP methods**|PUT and DELETE can modify/delete server content if access control is weak|Authenticate and authorize all non-GET methods; REST APIs must validate permissions|
|**Proxy/cache**|A malicious proxy can serve stale or modified objects; cache poisoning attacks inject bad responses into shared caches|HTTPS end-to-end encryption prevents proxy from reading or modifying content|
|**Conditional GET / Last-Modified**|Timing attacks — `Last-Modified` header reveals when content was last changed, leaking server activity|Not high-risk but worth noting in threat models|
|**404/400/505 responses**|Error messages can reveal server software version, directory structure, or internal paths|Configure servers to return generic error pages; strip `Server:` header|
|**URL-encoded form data (GET)**|Form data in URL is logged in server logs, proxy logs, browser history — leaked in `Referer` headers to third parties|Use POST for sensitive data; HTTPS encrypts URL in transit (but not logs)|
|**CDNs**|CDN compromise can serve malicious content to millions of users (supply-chain attack)|Subresource Integrity (SRI) hashes, HTTPS everywhere, certificate pinning|

---

## Questions I Still Have

- [ ] HTTP/1.1 uses persistent connections by default — does the server ever close them proactively, or does it always wait for the timeout?
- [ ] Pipelining in HTTP/1.1 still has **head-of-line blocking** — how does HTTP/2 solve this with multiplexing?
- [ ] If a cache stores HTTPS objects, can it cache them at all? Does HTTPS fundamentally break caching?
- [ ] What exactly is the `Referer` header and why is it a privacy risk?
- [ ] CDNs like Akamai — how do they route users to the nearest cache? Is it DNS-based?
- [ ] Why is the `Date:` field the time the response was _sent_ and not the time the object was last _modified_ — what's the distinction used for?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**HTTP**|HyperText Transfer Protocol — the Web's application-layer protocol|
|**Web page**|A document consisting of objects (HTML file + referenced objects)|
|**Object**|A file addressable by a single URL (HTML, JPEG, video, etc.)|
|**URL**|Uniform Resource Locator — hostname + path name identifying an object|
|**Web browser**|Client-side implementation of HTTP (Firefox, Chrome, IE)|
|**Web server**|Server-side implementation of HTTP — houses Web objects|
|**Stateless protocol**|Protocol that maintains no information about past client requests|
|**Non-persistent HTTP**|Each request/response pair uses a separate TCP connection|
|**Persistent HTTP**|All requests/responses between same client-server use one TCP connection|
|**RTT**|Round-Trip Time — time for a small packet to go from client to server and back|
|**Pipelining**|Client sends multiple HTTP requests back-to-back without waiting for responses|
|**HTTP request message**|Message sent by client with method, URL, headers, optional body|
|**HTTP response message**|Message sent by server with status code, headers, and object body|
|**GET**|HTTP method to request an object|
|**POST**|HTTP method to submit form data (body contains user input)|
|**HEAD**|Like GET but server sends only headers, no object body|
|**Status code**|3-digit number in HTTP response indicating the outcome (200, 404, etc.)|
|**Cookie**|Mechanism to maintain user state on top of stateless HTTP|
|**Set-cookie**|Response header that tells browser to store a cookie|
|**Web cache / Proxy server**|Intermediary that stores copies of recently requested objects|
|**Hit rate**|Fraction of requests satisfied by a Web cache from its local storage|
|**CDN**|Content Distribution Network — geographically distributed caches|
|**Conditional GET**|HTTP GET with `If-Modified-Since` header to check if cached copy is fresh|
|**304 Not Modified**|Response telling cache its copy is still valid — no object in body|
|**Last-Modified**|Response header indicating when the object was last changed on the server|

---

## Related Concepts

- [[2.1 - Principles of Network Applications]]
- [[TCP Three-Way Handshake — Chapter 3 Preview]]
- [[2.3 - File Transfer FTP]]
- [[2.4 - DNS]]
- [[SSL and TLS — Chapter 8 Preview]]
- [[CDNs — Chapter 7 Preview]]
- [[Socket Programming — Section 2.7]]

---

→ Next: [[2.3 - File Transfer FTP]]