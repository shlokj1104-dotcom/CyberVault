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

> **One-Line Summary:** HTTP is the Web's application-layer protocol. It has historically run over TCP — non-persistent in HTTP/1.0, persistent (with pipelining) in HTTP/1.1, and multiplexed over a single connection via framing in HTTP/2 — but HTTP/3 breaks from this lineage and runs over **QUIC**, a UDP-based protocol that absorbs TLS into its handshake and gives each HTTP message its own independently-reliable stream. HTTP itself remains stateless by design; state is layered back on top using cookies, and performance is improved using browser caching and the conditional GET mechanism.

---

## Core Idea

Until the early 1990s, the Internet was used primarily by researchers, academics, and university students to log in to remote hosts, transfer files, exchange news, and send electronic mail. These applications were (and remain) extremely useful, but the Internet itself was essentially unknown outside the academic and research communities.

Then, in the early 1990s, a major new application arrived on the scene — the **World Wide Web** [Berners-Lee 1994]. The Web was the first Internet application to catch the general public's eye. It dramatically changed how people interact both inside and outside their work environments, elevating the Internet from being just one of many data networks to essentially **the** one and only data network.

Perhaps what appeals most to users is that the Web operates **on demand**. Users receive what they want, when they want it — unlike traditional broadcast radio and television, which force users to tune in whenever the content provider happens to make the content available.

**Why people love the Web:**

- It is enormously easy for any individual to make information available over the Web — everyone can become a publisher at extremely low cost
- Hyperlinks and search engines help users navigate through an ocean of information
- Photos and videos stimulate the senses
- Forms, JavaScript, video, and many other devices enable interaction with pages and sites
- The Web and its protocols serve as a **platform** for generative AI, YouTube, Web-based e-mail (such as Gmail), and most mobile Internet applications, including Instagram and Google Maps

The Web's application-layer protocol is **HTTP (HyperText Transfer Protocol)**. HTTP is what your browser speaks to every web server on the planet. Understanding HTTP is foundational for both web development and cybersecurity — every web attack ultimately operates at this layer.

---

# 2.2.1 Overview of HTTP

**HyperText Transfer Protocol (HTTP)** is defined across several RFCs: [RFC 1945], [RFC 7230], [RFC 7540], and [RFC 9114]. Today the vast majority of applications run over HTTP — Web browsing, video streaming, social media, and even email between clients and mail servers. Smartphone apps also typically communicate with their servers over HTTP.

HTTP is implemented in **two programs**: a client program and a server program, executing on different end systems, talking to each other by exchanging HTTP messages. HTTP defines the structure of these messages and how the client and server exchange them.

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

**Concrete example:** A Web page that has an HTML file and five JPEG images = **6 objects total**. The base HTML file references the other objects with the objects' URLs. When your browser fetches the page, it first downloads the HTML, then parses it to discover all the referenced objects, then fetches those too.

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
|**Web browser**|Implements the **client side** of HTTP — sends requests, renders responses|Chrome, Edge|
|**Web server**|Implements the **server side** of HTTP — houses objects, listens for requests, sends responses|Apache, Nginx, Microsoft Internet Information Server (IIS)|

> Because Web browsers implement the client side of HTTP, in the context of the Web the terms _browser_ and _client_ are used interchangeably. The same goes for _Web server_ and _server_.

---

## What HTTP Actually Does

> **HTTP defines how Web clients request Web pages from Web servers, and how servers transfer Web pages to clients.**

The fundamental interaction is **request → response**:

1. User requests a Web page (for example, clicks a hyperlink)
2. Browser sends **HTTP request messages** for the objects in the page to the server
3. Server receives the requests and responds with **HTTP response messages** that contain the objects

![[Pasted image 20260619213308.png]] _(Figure 2.6 — HTTP request-response behavior: a server running Apache exchanges HTTP request and response messages with a PC running Microsoft Edge and an Android smartphone running Google Chrome)_

This is why HTTP is called a **request-response protocol** — every interaction is a client asking for something and a server answering.

---

## HTTP Runs Over TCP — or Over QUIC/UDP in HTTP/3

> Depending on the HTTP version, HTTP can use either **TCP** or **UDP** as its underlying transport protocol. Using UDP is a relatively recent development — covered in §2.2.7. For now we discuss HTTP assuming **TCP** is the underlying transport protocol, which is the case for HTTP/1.0, HTTP/1.1, and HTTP/2.

**The sequence (TCP case):**

1. HTTP client initiates a **TCP connection** with the server (default port **80** for HTTP, **443** for HTTPS)
2. Once the connection is established, the browser and server processes access TCP through their **socket interfaces** — on the client side the socket interface is the door between the client process and the TCP connection; on the server side it is the door between the server process and the TCP connection
3. The client sends HTTP request messages into its socket interface and receives HTTP response messages from its socket interface
4. The server receives request messages from its socket interface and sends response messages into its socket interface

> Once the client sends a message into its socket interface, the message is out of the client's hands and "in the hands" of TCP. Recall that TCP provides a **reliable data transfer service** to HTTP — each HTTP request message sent by a client eventually arrives intact at the server, and each HTTP response message sent by the server eventually arrives intact at the client.

**Why TCP and not UDP for HTTP/1.x and HTTP/2?**

HTTP needs **every single byte** of every object to arrive correctly. If even one byte of an HTML file or image is lost, the page is broken. TCP's reliable data transfer guarantees correct, in-order delivery — you cannot render a partially received image.

> Here we see one of the great advantages of a layered architecture — HTTP need not worry about lost data or the details of how TCP recovers from loss or reordering of data within the network. That is the job of TCP and the protocols in the lower layers of the protocol stack.

---

## HTTP is Stateless — By Design

> HTTP is a **stateless protocol** — the server maintains **no information whatsoever** about the clients between requests.

It is important to note that the server sends requested files to clients **without storing any state information about the client**. If a particular client asks for the same object twice within a few seconds, the server does not respond by saying it just served that object — instead, the server resends the object, as it has completely forgotten what it did earlier.

What this means in practice:

- The server has no memory of the first request when processing the second
- No session tables, no client history, nothing stored between requests

**Why stateless?**

Stateful protocols are complex and fragile:

- Servers must maintain state tables for every client
- If a server crashes, those tables are lost — clients and server are now out of sync
- State is expensive to maintain at scale (millions of concurrent users)

> Statelessness simplifies server design and has permitted engineers to develop high-performance Web servers that can handle thousands of simultaneous TCP connections.

We also note that the Web uses the **client-server application architecture** described in §2.1 — a Web server is always on, with a fixed IP address, and services requests from potentially millions of different browsers.

> The tradeoff: statelessness makes HTTP simple but means you cannot do things like "shopping carts" or "login sessions" natively. The solution is **cookies** (§2.2.4) — state layered on top of a stateless protocol.

---

## HTTP Versions

HTTP has gone through many versions:

|Version|RFC|Era|Transport|
|---|---|---|---|
|**HTTP/1.0**|RFC 1945|Early 1990s|TCP|
|**HTTP/1.1**|RFC 7230|1997|TCP|
|**HTTP/2**|RFC 7540|2015|TCP|
|**HTTP/3**|RFC 9114|2022|UDP (via QUIC)|

HTTP/1.0, HTTP/1.1, and HTTP/2 all run on top of TCP. More recently, **HTTP/3** has been standardized and is gaining traction — it runs over UDP, discussed in §2.2.7.

---

# 2.2.2 Non-Persistent and Persistent Connections

In many Internet applications, the client and server communicate over an extended period, with the client making a series of requests and the server responding to each. When this interaction takes place over TCP, the application developer needs to make an important decision — should each request/response pair be sent over a **separate** TCP connection, or should all requests and responses be sent over the **same** TCP connection?

|Type|Description|
|---|---|
|**Non-persistent connections**|Separate TCP connection for **each** request/response pair — open, use once, close|
|**Persistent connections**|**Same** TCP connection reused for multiple request/response pairs|

> **HTTP/1.0** → non-persistent (default) **HTTP/1.1 and HTTP/2** → persistent (default)

---

## Non-Persistent HTTP — How It Works

Imagine requesting a web page that has **1 base HTML file + 10 JPEG images** (11 objects total), all residing on the same server, with the base HTML file's URL being `http://www.someSchool.edu/someDepartment/home.index`. With non-persistent HTTP:

```
For the base HTML file:
  Step 1 → Client initiates TCP connection to www.someSchool.edu on port 80
            (a socket is created at the client and at the server)
  Step 2 → Client sends HTTP GET request via its socket, naming the
            path /someDepartment/home.index
  Step 3 → Server receives the request via its socket, retrieves the
            object from storage (RAM or disk), encapsulates it in an
            HTTP response message, and sends it to the client via its socket
  Step 4 → Server tells TCP to close the connection
            (TCP doesn't actually terminate until it knows the client
            has received the response message intact)
  Step 5 → Client receives the response; the TCP connection terminates.
            The client extracts the HTML file, examines it, and finds
            references to the 10 JPEG objects.

For each of the 10 JPEGs:
  Repeat Steps 1–4 for each image (10 separate TCP connections)

Total TCP connections opened = 11 (one per object)
```

> Each non-persistent TCP connection transports exactly **one request message and one response message**. Two different browsers may interpret (display) a received Web page differently — HTTP has nothing to do with how a page is rendered; the HTTP specs ([RFC 1945] and [RFC 7540]) define only the communication protocol between the client and server HTTP programs.

**Problems:**

1. **Overhead per object** — A brand-new TCP connection must be established and maintained for **each requested object**. For each connection, TCP buffers must be allocated and TCP variables kept in both client and server — a significant burden on a server handling hundreds of clients simultaneously.
    
2. **2 RTTs per object** — Each object suffers a delivery delay of two RTTs: one RTT to establish the TCP connection, and one RTT to request and receive the object.
    

---

## RTT — Round-Trip Time

> The **round-trip time (RTT)** is the time it takes for a small packet to travel from client to server and then **back** to the client.

RTT includes:

- **Packet-propagation delays**
- **Packet-queuing delays** in intermediate routers and switches
- **Packet-processing delays**

When a user clicks a hyperlink, the browser initiates a TCP connection involving a **three-way handshake**: the client sends a small TCP segment, the server acknowledges and responds with a small TCP segment, and finally the client acknowledges back to the server.

![[Pasted image 20260619213546.png]] _(Figure 2.7 — Back-of-the-envelope calculation for the time needed to request and receive an HTML file: RTT 1 for the TCP three-way handshake, RTT 2 for the HTTP request and the start of the response, plus the file transmission time at the end)_

**Response time formula for non-persistent HTTP (one object):**

```
Response time = 2 × RTT + file transmission time

Where:
  RTT 1 → The first two parts of the three-way handshake
  RTT 2 → The client sends the HTTP request combined with the third
           part of the handshake (the ACK); the server's HTML file
           transmission begins once the request arrives
  + file transmission time → time to push all bytes of the file
```

Thus, roughly, the total response time for one object is **two RTTs plus the transmission time at the server**.

---

## HTTP with Persistent Connections

Non-persistent connections have shortcomings:

- A brand-new connection must be established and maintained for **each requested object** — placing a significant burden on the server
- Each object suffers a delivery delay of **two RTTs**

> With **persistent connections**, as employed in **HTTP/1.1 and HTTP/2**, the server leaves the TCP connection open after sending a response. Subsequent requests and responses between the same client and server can be sent over the **same** connection.

In particular, an entire Web page (the base HTML file and all 10 images) can be sent over a single persistent TCP connection. Moreover, multiple Web pages residing on the same server can be sent from the server to the same client over a single persistent TCP connection. These requests for objects can be made using **"pipelining"** — that is, back-to-back, without waiting for replies to pending requests.

|Mode|Behaviour|Cost|
|---|---|---|
|**Persistent without pipelining**|Client waits for each response before sending the next request|~1 RTT per object — still sequential, but without repeated handshake cost|
|**Persistent with pipelining** _(default)_|Client sends requests back-to-back without waiting for any response|~1 RTT total for all objects on the same server|

Typically, the HTTP server closes a connection when it isn't used for a certain time (a configurable **timeout interval**). When the server receives the back-to-back requests, it sends the objects back-to-back.

---

# 2.2.3 HTTP Message Format

The HTTP specifications ([RFC 1945]; [RFC 7230]; [RFC 7540]; [RFC 9114]) define the HTTP message formats. There are exactly two types of HTTP messages: **request messages** (client → server) and **response messages** (server → client).

---

## HTTP Request Message

**A typical HTTP request message:**

```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

We can learn a lot from this simple message. First, it is written in ordinary **ASCII text**, so an ordinary computer-literate human can read it. Second, the message consists of five lines, each followed by a carriage return and a line feed; the last line is followed by an additional carriage return and line feed. A request message can have many more lines, or as few as one.

The first line is called the **request line**; the subsequent lines are called the **header lines**.

**Breaking down the request line** (`GET /somedir/page.html HTTP/1.1`):

- **Method field** → `GET` — can take on several values: `GET`, `POST`, `HEAD`, `PUT`, `DELETE`. The great majority of HTTP requests use `GET`.
- **URL field** → `/somedir/page.html` — the object being requested
- **HTTP version field** → `HTTP/1.1`

**Breaking down each header line:**

|Header|Value in example|What it means|
|---|---|---|
|`Host:`|`www.someschool.edu`|Specifies the host on which the object resides. You might think this is unnecessary since there's already a TCP connection in place — but this information is required by **Web proxy caches** (§2.2.5)|
|`Connection:`|`close`|Tells the server the browser doesn't want to bother with persistent connections — it wants the connection closed after the requested object is sent|
|`User-agent:`|`Mozilla/5.0`|Specifies the user agent — the browser type making the request (here, Firefox). Useful because a server can send different versions of the same object to different user agents (each addressed by the same URL)|
|`Accept-language:`|`fr`|Indicates the user prefers a French version of the object if one exists; otherwise the server should send its default version. One of many **content negotiation** headers available in HTTP|

**General format of a request message:**

```
Request line   →  method  SP  URL  SP  HTTP-version  CRLF
Header line 1  →  field-name: value  CRLF
Header line 2  →  field-name: value  CRLF
...
(blank line)   →  CRLF
Entity body    →  (empty for GET; present for POST)
```

`SP` = a space character, `CRLF` = carriage return + line feed (`\r\n`)

![[Pasted image 20260619214701.png]]
_(Figure 2.8 — General format of an HTTP request message)_

The general format closely follows the example above. After the header lines (and the additional CRLF) there is an **entity body**. The entity body is empty with the `GET` method but is used with `POST`. An HTTP client often uses `POST` when the user fills out a form — for example, providing search words to a search engine. With a `POST` message, the user is still requesting a Web page, but the specific contents of the page depend on what was entered into the form fields; if the method field is `POST`, the entity body contains what the user entered into the form fields.

> **GET with form data:** A request generated from a form does not necessarily have to use `POST`. HTML forms often use `GET` and include the inputted data in the requested URL. For example, if a form uses `GET`, has two fields, and the inputs to the two fields are `monkeys` and `bananas`, the URL has the structure `www.somesite.com/animalsearch?monkeys&bananas`. In day-to-day Web surfing you have probably noticed extended URLs of this sort.

> **POST vs GET for forms:** POST puts form data in the entity body (invisible in URL); GET puts it in the URL (visible, logged by servers and proxies). **Never use GET for sensitive data like passwords.**

---

### HTTP Methods — Complete Reference

|Method|Body?|Description|Use case|
|---|---|---|---|
|**GET**|No|Request the object at the specified URL|Fetching any web resource (pages, images, APIs)|
|**POST**|Yes|Submit data to the server; data is in the entity body|HTML form submissions (login forms, search, etc.)|
|**HEAD**|No|Similar to GET, but the server's response leaves out the requested object|Debugging — application developers often use `HEAD` for this purpose|
|**PUT**|Yes|Upload an object to a specific path on a specific Web server|Used in conjunction with Web-publishing tools, and by applications that need to upload objects to Web servers|
|**DELETE**|No|Delete the object at the specified URL|Allows a user, or an application, to delete an object on a Web server|

---

## HTTP Response Message

**A typical HTTP response message** (the response to the example request above):

```
HTTP/1.1 200 OK
Connection: close
Date: Mon, 21 Oct 2024 18:58:21 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Sun, 20 Oct 2024 13:20:46 GMT
Content-Length: 6821
Content-Type: text/html
  (data data data data data ...)
```

This message has three sections: an initial **status line**, six **header lines**, and then the **entity body**. The entity body is the meat of the message — it contains the requested object itself.

**Breaking down the status line:**

- `HTTP/1.1` → server's protocol version
- `200` → status code
- `OK` → corresponding status phrase — together these indicate that the server found, and is sending, the requested object

**Breaking down each header line:**

|Header|Example value|What it means|
|---|---|---|
|`Connection:`|`close`|Server will close the TCP connection after sending the message|
|`Date:`|`Mon, 21 Oct 2024 18:58:21 GMT`|The time and date the server **created and sent** the response — not when the object was created/last modified, but when the server retrieved the object from its file system and inserted it into the response|
|`Server:`|`Apache/2.2.3 (CentOS)`|Indicates the message was generated by an Apache Web server — analogous to `User-agent:` in the request. A **privacy/security risk** in production (tells attackers what software/version to target)|
|`Last-Modified:`|`Sun, 20 Oct 2024 13:20:46 GMT`|Time and date the object was created or last modified. **Critical for object caching**, both in the local browser and in network cache servers (proxy servers) — see §2.2.5|
|`Content-Length:`|`6821`|Number of bytes in the object being sent|
|`Content-Type:`|`text/html`|Indicates the object in the entity body is HTML text. The object type is officially indicated by `Content-Type:`, **not** the file extension|

**General format of a response message:**

```
Status line    →  HTTP-version  SP  status-code  SP  phrase  CRLF
Header line 1  →  field-name: value  CRLF
Header line 2  →  field-name: value  CRLF
...
(blank line)   →  CRLF
Entity body    →  the requested object (HTML, JPEG, etc.)
```

![[Pasted image 20260619214717.png]]
_(Figure 2.9 — General format of an HTTP response message)_

---

### HTTP Status Codes — Complete Reference

|Range|Category|Meaning|
|---|---|---|
|**1xx**|Informational|Request received, continuing process|
|**2xx**|Success|Request was successfully received, understood, and accepted|
|**3xx**|Redirection|Further action needed to complete the request|
|**4xx**|Client Error|Request has bad syntax or cannot be fulfilled — **client's fault**|
|**5xx**|Server Error|Server failed to fulfil an apparently valid request — **server's fault**|

**Some common status codes and phrases:**

|Code|Phrase|Meaning|
|---|---|---|
|**200**|OK|Request succeeded and the information is returned in the response|
|**301**|Moved Permanently|Requested object has been permanently moved; the new URL is specified in the response's `Location:` header — the client software will automatically retrieve the new URL|
|**400**|Bad Request|Generic error code indicating the request could not be understood by the server|
|**404**|Not Found|The requested document does not exist on this server|
|**505**|HTTP Version Not Supported|The requested HTTP protocol version is not supported by the server|

> `304 Not Modified` is a special status code used specifically in response to a **conditional GET** — covered in §2.2.5.

> **Practical exercise — see a real HTTP request and response:** From a command-line prompt, enter:

 
```
 curl -v http://gaia.cs.umass.edu/kurose_ross/index.php
```

> This sends an HTTP GET request to the `gaia.cs.umass.edu` Web server to retrieve `/kurose_ross/index.php` — the authors' homepage for this textbook. The verbose option (`-v`) displays the text content of both the HTTP GET request and the response received, as text. To see the response rendered, just enter the URL into a browser instead.

The HTTP specification defines many more header lines than the ones covered here — a browser generates header lines as a function of the browser type/version, the user's browser configuration, and whether the browser already has a cached (but possibly out-of-date) version of the object. Web servers behave similarly: different products, versions, and configurations all influence which header lines are included in response messages.

---

# 2.2.4 User-Server Interaction — Cookies

We mentioned above that an HTTP server is stateless. This simplifies server design and has permitted engineers to build high-performance Web servers that handle thousands of simultaneous TCP connections. However, it is often desirable for a Web site to identify users — either because it wishes to restrict access, or because it wants to serve content as a function of user identity.

**The solution:** **Cookies**, defined in [RFC 6265], allow sites to track users. Most major commercial Web sites use cookies today.

---

## The Four Components of the Cookie System

1. A `Set-cookie:` header line in the **HTTP response** (server → client)
2. A `Cookie:` header line in the **HTTP request** (client → server)
3. A **cookie file** stored on the user's end system, managed by the browser
4. A **back-end database** at the Web site

All four must be in place for the system to work.

---

## How Cookies Work — Step by Step

**Scenario:** Susan, who always accesses the Web using Google Chrome from her home PC, contacts Amazon.com for the first time. Suppose she has already visited eBay in the past.

```
Visit 1 — No cookie yet:
  Susan's browser → Amazon server
  (usual HTTP request, no Cookie: header)

  Amazon server:
  → Creates a unique identification number for Susan: 1678
  → Creates an entry in its back-end database, indexed by that number
  → Sends an HTTP response INCLUDING:
      Set-cookie: 1678

  Susan's browser:
  → Sees the Set-cookie: header
  → Appends a line to its cookie file: hostname + identification number
    (the cookie file already has an entry for eBay, since Susan visited
    that site in the past)
```

```
Every subsequent request to Amazon:
  Susan's browser → Amazon server
  → Browser consults its cookie file, extracts the identification
    number for this site, and includes:
      Cookie: 1678

  Amazon server:
  → Reads Cookie: 1678, looks it up in its back-end database
  → Is able to track Susan's activity at the site — though Amazon
    doesn't necessarily know her name, it knows exactly which pages
    user 1678 visited, in which order, and at what times
```

```
One week later:
  Susan's browser still includes Cookie: 1678 in every request
  Amazon's database still has the matching entry
  Amazon can recommend products based on pages Susan visited in the past
```

![[Pasted image 20260619214943.png]] _(Figure 2.10 — Keeping user state with cookies: the server creates the ID and sets the cookie in the response; all subsequent requests carry the Cookie header; the server's backend database maps ID to user data)_

**Registration and "one-click shopping":** If Susan also registers with Amazon — providing her full name, e-mail address, postal address, and credit card information — Amazon can associate this information with her identification number (and with every page she has ever visited at the site). This is how Amazon and other e-commerce sites provide **"one-click shopping"**: when Susan chooses to purchase an item on a subsequent visit, she doesn't need to re-enter her name, card number, or address.

**Cookies for session state:** Cookies can be used to identify a user across an entire session. The first time a user visits a site, they may provide an identification (possibly their name). During subsequent requests in that session, the browser passes the cookie header to the server, identifying the user throughout. For example, when a user logs in to a Web-based e-mail application, the browser sends cookie information to the server, permitting the server to identify the user throughout the session — this creates a **user session layer on top of stateless HTTP**.

**The privacy concern:**

Although cookies often simplify the Internet shopping experience, they are controversial because they can also be considered an invasion of privacy. Using a combination of cookies and user-supplied account information, a Web site can learn a great deal about a user — and potentially sell this information to a third party.

---

# 2.2.5 Browser Caching

We learned earlier (Figure 2.7) that the minimum delay from when your browser first requests an object until it receives that object is **2RTT** — one RTT to establish the TCP connection, and another RTT to send the HTTP request and receive the response. (We'll see shortly that HTTP/3 has been optimized to require just **one RTT** to request a Web page in many cases.) Can this delay be cut to something even _less_ than one RTT — even to **zero**? Although this might seem to defy the laws of physics, it can indeed happen with **browser caching**, a technique implemented in all modern Web browsers and used extensively across the Web. One estimate is that **75% of Web requests** receive HTTP responses carrying explicit browser-caching instructions [Web Almanac 2021].

With browser caching, a Web browser **locally** stores the content of recently received Web objects in its **browser cache**. When a user requests a Web object, the browser first checks whether that object is stored in its local cache. If so, it may immediately display the object — without making a new request to the Web server, and without incurring the request/response delay — since the browser already requested and received that object earlier.

**The challenge:** how does the browser know whether the object currently housed on the Web server has been modified since the copy was cached locally? Certain objects (images, JS, CSS) change infrequently; others (text) may change often. HTTP provides fields in both the GET and response messages to help manage browser caching [RFC 7232]:

- **`Cache-Control`** — A field in an HTTP **response** message that lets the server specify how the content should be cached.
    - `Cache-Control: no-store` instructs the browser **not** to cache the content locally — the browser will always have to re-request it from the server.
    - `Cache-Control: max-age=3600` tells the browser the content can remain cached for **3600 seconds**, after which it must be re-requested. Many other `Cache-Control` directives are defined in [RFC 7232].
- **`If-Modified-Since`** — HTTP provides a mechanism in the client-to-server message, known as the **conditional GET message**, to facilitate caching.

> An HTTP request is a **conditional GET** if: (1) it uses the `GET` method AND (2) it includes an `If-Modified-Since:` header line.

With a conditional GET, the client explicitly requests an object but lets the server know it already has that object in its browser cache, and when that object was added to the cache. The server will reply with the full object only if it has changed since; otherwise it simply confirms that the cached copy is still current — without sending the object again.

---

## How the Conditional GET Works — Full Example

Suppose a browser has previously requested an object, say `kiwi.jpg`, has that object in its cache, and has recorded the time it was added to the cache. When the user later requests this object again, the browser is uncertain whether the cached copy is still current — this can happen because the server never provided cache-control information, a previously provided `max-age` directive has expired, or the server directed the browser to always explicitly check freshness. In this case, the browser sends a conditional GET:

```
Browser → Server:
  GET /fruit/kiwi.jpg HTTP/1.1
  Host: www.exotiquecuisine.com
  If-modified-since: Fri, 13 Sep 2024 09:23:24
```

The `If-modified-since:` value indicates the cached object was received on 13 Sep 2024 at 9:23:24 — this conditional GET is telling the server to send the object **only if** it has been modified since that date.

**Case A — Object has NOT been modified since that date:**

```
Server → Browser:
  HTTP/1.1 304 Not Modified
  Date: Fri, 8 Nov 2024 12:07:29
  Server: Apache/2.4.62 (Unix)
  (empty entity body)
```

The Web server still sends a response message but does **not** include the requested object. The `304 Not Modified` status line tells the browser its cached copy is current and can be used. Including the object here would only waste bandwidth and increase user-perceived response time — particularly for large objects.

**Case B — Object HAS been modified since that date:**

```
Server → Browser:
  HTTP/1.1 200 OK
  (new object body — updated image bytes)
```

The browser replaces its old cached copy with the new object and updates its locally-recorded modification time.

> **Key insight:** The `304 Not Modified` response has an **empty body** — no object bytes are transmitted. The browser already has the object; the server is just confirming "yours is still good." This saves bandwidth while ensuring correctness.

---

# 2.2.6 HTTP/2

**HTTP/2** [RFC 7540], standardized in 2015, was the first new version of HTTP since HTTP/1.1 (standardized in 1997). About **35% of the top 10 million websites** supported HTTP/2 in 2024 [W3Techs], and most browsers — including Chrome, Edge, Safari, Opera, and Firefox — support it. Usage has declined somewhat in recent years due to the emergence of HTTP/3 (§2.2.7).

**Primary goals of HTTP/2:**

- Reduce perceived latency by enabling request and response **multiplexing** over a **single** TCP connection
- Provide **request prioritization** and **server push**
- Provide efficient **compression** of HTTP header fields

> HTTP/2 does **not** change HTTP methods, status codes, URLs, or header fields. Instead, it changes **how data is formatted and transported** between client and server.

---

## The Motivation — Head of Line (HOL) Blocking

Recall that HTTP/1.1 uses persistent TCP connections, allowing a Web page to be sent from server to client over a **single** TCP connection. Having only one TCP connection per page reduces the number of sockets at the server and ensures each transported page gets a fair share of network bandwidth.

But Web browser developers quickly discovered that sending all of a page's objects over a single TCP connection causes a **Head of Line (HOL) blocking** problem.

> Consider a Web page that includes an HTML base page, a large video clip near the top of the page, and many small objects below the video. Suppose there is a low-to-medium-speed bottleneck link (e.g. a low-speed wireless link) between server and client. With a single TCP connection, the video clip will take a long time to cross the bottleneck link, while the small objects are delayed waiting behind it — the video at the head of the line **blocks** the small objects behind it.

HTTP/1.1 browsers typically work around this by opening **multiple parallel TCP connections**, so objects on the same page are sent in parallel — small objects can then arrive and render much faster.

**A second, sneakier reason browsers do this:** TCP congestion control (Chapter 3) aims to give each TCP connection sharing a bottleneck link an equal share of available bandwidth — roughly 1/n for _n_ connections. By opening multiple parallel connections for a single page, a browser can "cheat" and grab a larger portion of the link's bandwidth. Many HTTP/1.1 browsers open **up to six parallel TCP connections**, both to circumvent HOL blocking and to obtain more bandwidth.

> One of the primary goals of HTTP/2 is to get rid of (or reduce) parallel TCP connections for transporting a single Web page — reducing the number of sockets that need to be open and maintained at servers, and letting TCP congestion control operate as intended. But with only **one** TCP connection per page, HTTP/2 requires carefully designed mechanisms to avoid HOL blocking.

---

## HTTP/2 Framing

HTTP/2's solution to HOL blocking is to break each message into small **frames**, and **interleave** the request and response messages on the same TCP connection.

**Example:** A Web page consisting of one large video clip and 8 smaller objects. The server receives 9 concurrent requests and needs to send 9 competing HTTP response messages over the same TCP connection. Suppose all frames are of fixed length, the video clip consists of **1000 frames**, and each small object consists of **2 frames**.

```
WITHOUT interleaving (frames sent in order, object by object):
  Video clip's 1000 frames sent first → small objects must wait
  → Small objects fully delivered only after 1016 frames sent total

WITH HTTP/2 frame interleaving:
  Frame 1 of video  → Frame 1 of each of the 8 small objects sent
  Frame 2 of video  → Frame 2 (last) of each of the 8 small objects sent
  → ALL small objects fully delivered after just 18 frames sent total
```

> The HTTP/2 framing mechanism can significantly decrease user-perceived delay. The ability to break an HTTP message into independent frames, interleave them, and reassemble them on the other end is the single most important enhancement of HTTP/2.

**How framing works mechanically:**

- Framing is done by the **framing sub-layer** of the HTTP/2 protocol
- When a server sends an HTTP response, it is processed by the framing sub-layer, where it is broken into frames: the header field becomes one frame, and the body is broken into one or more additional frames
- The frames of the response are interleaved by the framing sub-layer with the frames of other responses, and sent over the single persistent TCP connection
- As frames arrive at the client, they are first reassembled into the original response messages at the framing sub-layer, then processed by the browser as usual
- A client's HTTP requests are similarly broken into frames and interleaved
- The framing sub-layer also **binary-encodes** the frames — binary protocols are more efficient to parse, lead to slightly smaller frames, and are less error-prone

---

## Response Message Prioritization and Server Pushing

**Message prioritization** allows developers to customize the relative priority of requests to better optimize application performance. The framing sub-layer organizes messages into parallel streams of data destined to the same requestor. When a client sends concurrent requests to a server, it can assign a **weight between 1 and 256** to each message — the higher the number, the higher the priority — so the server can send the frames for the highest-priority responses first. The client can also state each message's **dependency** on other messages, by specifying the ID of the message it depends on.

**Server push:** Another feature of HTTP/2 is the ability for a server to send **multiple responses** for a single client request. In addition to responding to the original request, the server can **push** additional objects to the client — without the client having to request each one. This is possible because the HTML base page indicates which objects will be needed to fully render the page. Instead of waiting for explicit HTTP requests for these objects, the server can analyze the HTML page, identify the needed objects, and send them to the client **before** receiving explicit requests for them. Server push eliminates the extra latency caused by waiting for those requests.

---

# 2.2.7 HTTP/3 and QUIC

HTTP/1.0, HTTP/1.1, and HTTP/2 all share one important common characteristic — they all run over **TCP**. But that doesn't mean HTTP will always run over TCP! **HTTP/3**, standardized in 2022 [RFC 9114], has hit the Internet community by storm — it is rapidly gaining market share for email, video/audio streaming, and social media, and has become the default HTTP version for many popular Web browsers and smartphone applications. Remarkably, **HTTP/3 does not run over TCP — it runs over UDP.**

---

## The QUIC Transport Protocol

HTTP/3 runs over the **Quick UDP Internet Connections (QUIC)** transport protocol, standardized in 2021 [RFC 9000].

You may now be thinking: "Hold on — haven't we been told the Internet has exactly two transport protocols, UDP and TCP? Is there now a third one called QUIC?"

> Strictly speaking, the Internet continues to have only **two** transport protocols, TCP and UDP. QUIC is **not** actually a transport-layer protocol — it is a **sub-layer in the application layer** that uses UDP to send and receive packets over the Internet. Hence the "UDP" in the name, Quick **UDP** Internet Connections.

However, even though QUIC is not, strictly speaking, a transport-layer protocol, **from the application developer's perspective it is one**. Network application developers can write applications by creating QUIC sockets, similar to how they create TCP and UDP sockets. When an application establishes a QUIC socket, the QUIC sublayer creates a UDP socket beneath it. The application can then send and receive messages through the QUIC socket without concerning itself with QUIC's inner workings.

![[Pasted image 20260619215149.png]]
_(Figure 2.11 — a. Traditional secure HTTP protocol stack: HTTP/2 over TLS over TCP over IP; b. Secure QUIC-based HTTP/3 protocol stack: HTTP/2 (slimmed) over QUIC over UDP over IP — TLS is absorbed into QUIC, and together QUIC + HTTP/2-slimmed make up HTTP/3)_

---

## Why a New "Transport Protocol"?

QUIC is a generic protocol usable by many applications, but it was primarily designed to improve **Web performance**.

In the distant past (up to about 2005), most Web transactions ran directly over TCP without any encryption. Over the years, encryption and authentication became default requirements for Web surfing, Web search, and Web-based email — typically provided via **TLS** (§2.1.4, covered in more detail in Chapter 8).

**The "double handshake" problem:** Today's HTTP/1.1 and HTTP/2 Web transactions typically employ TLS running on top of TCP — commonly known as **HTTPS**. This design requires **two handshake phases**: one to set up the TCP connection, and a subsequent one to share encryption keys between client and server. This introduces undesirable delays for interactive HTTP applications, including Web browsing.

> One of the most important features of QUIC is that it gets rid of this double handshake by **absorbing the TLS handshake phase into the initial connection-setup handshake** — hence the word "Quick."

Specifically, QUIC integrates a recent version of TLS, called **TLS 1.3**, directly into the protocol, eliminating the need for a separate TLS handshake. QUIC performs both connection setup and encryption setup in **one round trip**, reducing latency. On reconnecting, QUIC can reuse previously stored session and encryption parameters, allowing **0-RTT** data transmission without waiting for a full handshake.

**QUIC also re-solves HOL blocking — better than HTTP/2 did.** Recall that HTTP/2 breaks messages into small frames and multiplexes frames from different HTTP messages over a **single TCP connection**. But this doesn't fully resolve HOL blocking: in HTTP/2, if a packet is lost, **TCP requires the lost packet to be retransmitted and processed before any other packets can be delivered** — even packets from messages that don't depend on the lost one. This is a different form of HOL blocking, especially severe in wireless environments where packet loss is common.

> QUIC solves this by using UDP instead of TCP, and applying **reliable data transfer to each message separately** within the same QUIC connection. QUIC uses the term **"streams"** for the different data streams (different HTTP messages, in an HTTP context) sharing a single QUIC connection. Each stream is managed independently — if a packet from one stream is lost, only that stream is affected; other streams continue sending and receiving data without waiting for the lost packet to be retransmitted, eliminating the HOL blocking problem that plagues single-connection TCP.

> The inner workings of QUIC are discussed in greater detail in Section 3.7.

---

## QUIC Services

When developing a new networking application, should you run it over UDP, TCP, or QUIC? From the application developer's perspective, **QUIC services are more similar to TCP than to UDP:**

- **Connection-oriented**, like TCP — a QUIC connection is created between client and server before data is sent, established on top of the bare-bones connectionless UDP
- **Reliable data transfer**, like TCP — the application knows every message sent will eventually arrive at the other side
- **Congestion and flow control**, like TCP

**But QUIC offers additional services that go well beyond TCP:**

|Service|What it provides|
|---|---|
|**Built-in Encryption**|Integrates TLS 1.3 directly, providing encrypted communication by default — eliminates the need for separate TLS-over-TCP handshakes, making secure connections faster and more efficient|
|**Independent Data Streams**|Allows multiple streams (e.g., multiple HTTP messages) to be sent simultaneously over a single QUIC connection. Streams are independent, so packet loss in one stream doesn't affect others — unlike TCP. Each stream can also be managed/prioritized individually (video, text, control messages)|
|**0-RTT Handshakes**|For returning clients, allows data to be sent immediately without waiting for a full connection handshake — speeds up reconnecting, benefiting latency-sensitive applications like Web surfing and gaming|
|**Connection Migration**|Allows connections to remain active even if the client's IP address changes (e.g., switching from Wi-Fi to cellular) — improves reliability for mobile users|

---

## HTTP/3: Running the Web Over QUIC

HTTP/3, the latest version of HTTP, is designed to address several limitations of previous versions — particularly performance, security, and reliability over modern networks. Unlike previous versions, HTTP/3 operates over the QUIC protocol, gaining all the QUIC services described above.

Major browsers — Chrome, Firefox, Edge, and Safari — now support HTTP/3 and have adopted it as their default protocol when connecting to HTTP/3-compatible servers. Leading Internet companies such as Google have implemented HTTP/3 to improve user experience across search, video distribution, and video conferencing.

> HTTP/3 does **not** change the message formats of earlier HTTP versions. Its only major change compared with HTTP/2 is that, instead of using a persistent **TCP** connection between client and server, it uses a persistent **QUIC** connection.

In summary, HTTP/3 clients and servers use QUIC's features — fast connection establishment, encryption, multiplexing of independent streams, prioritization, and connection migration — to efficiently and securely deliver multiple objects of a Web page in parallel, while reducing latency and ensuring smooth, uninterrupted transmission.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**HTTP is cleartext**|All data — including passwords, cookies, and session tokens — is visible to anyone on the same network (e.g. public WiFi). Passive sniffing with tools like Wireshark trivially captures it|Always use **HTTPS** (HTTP over TLS, or natively encrypted via QUIC in HTTP/3). The `Secure` flag on cookies ensures they are never sent over plain HTTP|
|**Cookies — Session Hijacking**|If an attacker steals a user's `Cookie:` header (via sniffing, XSS, etc.), they can replay it and impersonate that user entirely|`HttpOnly` flag (JavaScript cannot access the cookie), `Secure` flag (only sent over HTTPS), `SameSite=Strict` (not sent in cross-origin requests)|
|**Cookies — CSRF**|A malicious website can trick a logged-in user's browser into sending a forged request to another site — the browser automatically includes the cookie|CSRF tokens (random secret embedded in forms), `SameSite` cookie attribute|
|**HTTP Methods (PUT / DELETE)**|If a server exposes PUT or DELETE without proper authentication, an attacker can overwrite or delete server-side content|All non-GET/HEAD methods must require authentication and authorisation. REST APIs must validate permissions on every request|
|**Cache-Control Manipulation**|A malicious intermediary or compromised server response can set aggressive caching directives to force browsers to serve stale or poisoned content long-term|Use conservative `max-age` values for sensitive content; use `no-store` for anything containing personal data or tokens|
|**HTTP/2 Server Push Abuse**|A malicious or compromised server could push unrequested, oversized, or malicious resources to the client before they're asked for|Browsers validate and can reject pushed resources that violate same-origin policy; servers should only push resources genuinely needed for the page|
|**Information Leakage via Headers**|The `Server:` header reveals software and version (e.g. `Apache/2.2.3`) — attackers target known CVEs for that version|Strip or spoof the `Server:` header in production. Return generic error pages for 404/400/505 (don't expose directory structure)|
|**GET with Sensitive Form Data**|Passwords or tokens in GET URLs are stored in browser history, server access logs, proxy logs, and leaked via `Referer` headers to third-party resources|Use POST for any sensitive form data. HTTPS encrypts URLs in transit, but they still appear in server-side logs — never put secrets in URLs|
|**QUIC/UDP Amplification & Reflection**|Because HTTP/3 runs over UDP, it inherits UDP's susceptibility to spoofed-source amplification/reflection attacks against servers|QUIC requires source-address validation tokens before committing significant server resources, mitigating naive UDP reflection|
|**QUIC 0-RTT Replay Attacks**|0-RTT data sent on reconnection doesn't have the same forward-secrecy guarantees as a full handshake — an attacker who captures a 0-RTT request can potentially replay it|Servers should never apply 0-RTT data to non-idempotent operations (e.g., financial transactions) without additional safeguards|

---

## Questions I Still Have

- [ ] HTTP/1.1 persistent connections — if the server's timeout fires while the client is mid-think-time, does the client get an error on its next request, or does it silently reconnect?
- [ ] Does HTTPS (TLS-over-TCP) prevent browser caching from working with intermediary proxies, since the content is encrypted end-to-end? Does QUIC's built-in TLS 1.3 change this picture at all?
- [ ] What exactly is the `Referer` [sic] header — when is it sent, what does it contain, and why is it a privacy risk for users browsing sensitive pages?
- [ ] Section 3.7 promises a deeper look at QUIC's inner workings — specifically, how does per-stream reliable delivery actually get implemented over a fundamentally unreliable UDP substrate?
- [ ] How exactly does QUIC's **connection migration** keep a connection alive when the client's IP changes mid-stream — what identifies the same logical connection across two different IP addresses?
- [ ] CDNs are mentioned as being covered later (Section 2.5/2.6, in the context of video streaming) — how do they interact with HTTP/3 and QUIC's connection model, especially with Anycast-style routing to the nearest edge node?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**HTTP**|HyperText Transfer Protocol — the application-layer protocol of the Web|
|**Web page**|A document consisting of a base HTML file and referenced objects|
|**Object**|Any file addressable by a single URL (HTML, JPEG, CSS, JS, video, etc.)|
|**URL**|Uniform Resource Locator — hostname + path name, together identifying one object|
|**Web browser**|Client-side implementation of HTTP (Chrome, Edge, etc.)|
|**Web server**|Server-side implementation of HTTP — stores objects, responds to requests|
|**Stateless protocol**|Protocol that maintains zero information about past client interactions|
|**Non-persistent HTTP**|Each request/response pair uses its own TCP connection — open, use once, close|
|**Persistent HTTP**|Multiple requests/responses share the same TCP connection — default in HTTP/1.1 and HTTP/2|
|**RTT (Round-Trip Time)**|Time for a small packet to travel from client to server and back|
|**Pipelining**|Sending multiple HTTP requests back-to-back without waiting for prior responses|
|**Request line**|First line of HTTP request — method, URL, HTTP version|
|**Status line**|First line of HTTP response — HTTP version, status code, phrase|
|**Entity body**|The payload of an HTTP message (POST request data; response object bytes)|
|**GET / POST / HEAD / PUT / DELETE**|The five HTTP methods covered in this section|
|**200 OK**|Request succeeded; object is in the entity body|
|**301 Moved Permanently**|Object has a new permanent URL; browser should follow the `Location:` header|
|**304 Not Modified**|Conditional GET result — cached copy is still valid; no body transmitted|
|**404 Not Found**|Requested object does not exist on this server|
|**Cookie**|A small piece of state stored by the browser, sent in every request to the issuing site|
|**Set-cookie**|Response header instructing the browser to store a cookie value|
|**Browser cache**|Local storage in the browser holding recently received Web objects|
|**Cache-Control**|Response header letting the server specify caching behaviour (`no-store`, `max-age`, etc.)|
|**Conditional GET**|HTTP GET with an `If-Modified-Since:` header — asks the server if the object has changed|
|**If-Modified-Since**|Request header carrying the date of the cached copy for freshness checking|
|**Last-Modified**|Response header indicating when the server object was last changed|
|**HTTP/2**|Version of HTTP (RFC 7540) that multiplexes messages over a single TCP connection via framing|
|**HOL (Head of Line) blocking**|A slow or lost object/packet at the front of a connection delays unrelated objects/packets behind it|
|**Framing sub-layer**|HTTP/2 component that breaks messages into binary frames and interleaves them|
|**Server push**|HTTP/2 feature letting the server send objects the client hasn't explicitly requested yet|
|**HTTP/3**|Version of HTTP (RFC 9114) that runs over QUIC instead of TCP|
|**QUIC**|Quick UDP Internet Connections (RFC 9000) — an application-layer sub-layer over UDP that behaves like a transport protocol: connection-oriented, reliable, congestion-controlled, with built-in TLS 1.3 and independent streams|
|**Stream**|An independent, reliably-delivered data flow (e.g., one HTTP message) within a single QUIC connection|
|**0-RTT**|Connection-establishment mode where a returning client sends data immediately, reusing prior session parameters instead of performing a full handshake|
|**Connection migration**|QUIC's ability to keep a connection alive across a change in the client's IP address|

---

## Related Concepts

---

→ Next: [[FTP]]