---
title: ELECTRONIC MAIL
date: 2026-06-04
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.3 Electronic Mail in the Internet

> **One-Line Summary:** Internet e-mail is an **asynchronous, store-and-forward system** built from three components — user agents, mail servers, and SMTP — where SMTP **pushes** mail between servers, and a separate access protocol (IMAP or HTTP) **pulls** it down to the recipient's device.

---

## Core Idea

Electronic mail has existed since the very beginning of the Internet — it was the most popular application when the Internet was in its infancy and remains one of the most important and utilized applications today.

The key property of e-mail that sets it apart from a phone call or a video chat is that it is **asynchronous**: people send and read messages when it is convenient for them, without having to coordinate with the other person's schedule. You send a message at 2 AM; your colleague reads it at 9 AM. Neither of you needed to be available at the same time.

Understanding how e-mail actually works under the hood is critical for cybersecurity. Spam, phishing, spoofing, and business email compromise (BEC) attacks all exploit weaknesses in the e-mail protocol stack that we'll see in this section.

---

## The Three Major Components

> The Internet mail system has three major components: **user agents**, **mail servers**, and **SMTP** (Simple Mail Transfer Protocol).

![[Pasted image 20260619221442.png]] _(Figure 2.12 — A high-level view of the Internet e-mail system: user agents sit at the edges, mail servers (each with an outgoing message queue and per-user mailboxes) sit in the middle, and SMTP connects the mail servers to each other)_

Think of the Internet mail system like a physical postal network:

|Mail System Component|Physical Post Office Analogy|
|---|---|
|**User agent**|You, writing a letter at home and dropping it in the postbox|
|**Mail server**|Your local post office — it receives, sorts, holds, and forwards mail|
|**SMTP**|The mail trucks and postal routes connecting post offices to each other|
|**Mailbox**|Your personal PO box at the post office — holds mail until you pick it up|
|**Message queue**|A tray of undelivered letters the post office keeps retrying|

---

### User Agents

> **User agents** allow users to read, reply to, forward, save, and compose messages.

Examples: Microsoft Outlook, Apple Mail, Thunderbird, web-based Gmail, the Gmail App running on a smartphone.

- Today, many users send and receive mail through **HTTP-based user agents** such as Gmail and Yahoo! Mail, rather than a traditional desktop mail client.
- When **Alice** finishes composing her message, her user agent sends it to **her mail server**, where it is placed in the **outgoing message queue**
- When **Bob** wants to read a message, his user agent **retrieves** the message from his mailbox on **his mail server**

---

### Mail Servers

> **Mail servers form the core of the e-mail infrastructure.**

Each recipient (e.g. Bob) has a **mailbox** located in one of the mail servers. The mailbox manages and maintains the messages that have been sent to him.

A typical message journey:

```
Alice's user agent
  → Alice's mail server (placed in outgoing message queue)
    → Bob's mail server (delivered via SMTP)
      → Bob's mailbox
        → Bob's user agent (retrieved via IMAP / HTTP)
```

When Bob wants to access the messages sitting in his mailbox, **the mail server containing his mailbox authenticates him** using his username and password.

**What happens when delivery fails?**

If Alice's server cannot deliver mail to Bob's server (e.g. Bob's server is temporarily down), Alice's server **holds the message in a message queue** and retries periodically — typically every **30 minutes**. If repeated attempts over **several days** all fail, the server removes the message and notifies Alice with a bounce e-mail.

> This is why you sometimes get a "Mail delivery failed" notification hours or days after sending — your mail server was retrying in the background the whole time.

**Mail servers wear two hats:**

When a mail server sends mail to other servers, it acts as an **SMTP client**. When it receives mail from other servers, it acts as an **SMTP server**. Every mail server runs both sides simultaneously.

---

# 2.3.1 SMTP

> **SMTP is the principal application-layer protocol for Internet electronic mail.** It uses TCP's reliable data transfer to move mail from the sender's mail server to the recipient's mail server.

SMTP is defined in **RFC 5321** and is **much older than HTTP** — the original SMTP RFC dates back to **1982**. This age shows: SMTP carries some archaic design constraints that were reasonable in 1982 but are painful today.

---

## The 7-bit ASCII Constraint — SMTP's Biggest Quirk

> SMTP restricts the **body** (not just the headers) of all mail messages to **7-bit ASCII**.

This was fine in 1982 when everyone typed plain English text and no one e-mailed images. Today it is a real constraint:

- **Binary data** (images, PDFs, executables) must be **encoded into ASCII** before being sent over SMTP
- The ASCII-encoded message must be **decoded back to binary** after SMTP transport at the destination
- This is what **MIME (Multipurpose Internet Mail Extensions)** and **Base64 encoding** do — they're the solution to SMTP's ASCII limitation
- HTTP, by contrast, has **no such restriction** — it can carry arbitrary binary data natively

> **Analogy:** Imagine a post office that only accepts letters written in English, no matter where they're going. If you want to send a photograph, you have to first translate the photograph into a description written in English, ship it, and have the recipient reconstruct the photograph from the description. That's exactly what Base64 encoding does for binary attachments over SMTP.

---

## How SMTP Delivers a Message — Six Steps

**Scenario:** Alice wants to send Bob a simple ASCII message. Alice's address: `alice@crepes.fr`. Bob's address: `bob@hamburger.edu`.

![[Pasted image 20260619221649.png]] _(Figure 2.13 — Alice sends a message to Bob: ① Alice composes in her user agent, ② user agent sends to Alice's mail server (message queue), ③ SMTP client on Alice's server opens TCP connection to Bob's server, ④ SMTP handshake then message sent, ⑤ Bob's server places message in Bob's mailbox, ⑥ Bob reads at his convenience via his user agent)_

```
Step 1 → Alice invokes her user agent, types bob@hamburger.edu,
          composes the message, instructs the agent to send

Step 2 → Alice's user agent sends the message to Alice's mail server
          → placed in the outgoing message queue

Step 3 → The SMTP client side (running on Alice's mail server)
          sees the message in the queue
          → opens a TCP connection to the SMTP server
            running on Bob's mail server (port 25)

Step 4 → After the initial SMTP handshake,
          the SMTP client sends Alice's message into the TCP connection

Step 5 → At Bob's mail server, the SMTP server side receives the message
          → places it in Bob's mailbox

Step 6 → Bob invokes his user agent to read the message at his convenience
```

---

## A Critical Design Detail — No Intermediate Servers

> **SMTP does not normally use intermediate mail servers for sending mail**, even when the two servers are on opposite sides of the world.

If Alice's server is in Hong Kong and Bob's server is in St. Louis, the TCP connection is a **direct, end-to-end** connection between Hong Kong and St. Louis — no relay through some intermediate server in between.

**What if Bob's server is down?**

> The message **remains in Alice's mail server** and waits for a new attempt. It does **not** get placed in some intermediate mail server.

This is why Alice's mail server is essential in the architecture — it is the **responsible party** for retrying delivery to Bob's server. If Alice's user agent tried to deliver directly to Bob's server and Bob's server was down, Alice would have no mechanism for retrying.

---

## SMTP Handshake and Transcript — The Full Dialogue

When the TCP connection is established between the two SMTP servers, they do not immediately start transferring the message. They first **introduce themselves** in an **application-layer handshake** — just as humans introduce themselves before doing business.

During this handshake, the SMTP client tells the server:

- The e-mail address of the **sender** (who generated the message)
- The e-mail address of the **recipient**

Only then does the client send the message body.

**Annotated example transcript** (C = client `crepes.fr`, S = server `hamburger.edu`):

```
S: 220 hamburger.edu               ← Server announces itself (220 = ready)

C: HELO crepes.fr                  ← Client introduces itself
S: 250 Hello crepes.fr, pleased to meet you  ← Server acknowledges

C: MAIL FROM: <alice@crepes.fr>    ← Who is sending?
S: 250 alice@crepes.fr ... Sender ok

C: RCPT TO: <bob@hamburger.edu>    ← Who is the recipient?
S: 250 bob@hamburger.edu ... Recipient ok

C: DATA                            ← "I'm about to send the message body"
S: 354 Enter mail, end with "." on a line by itself

C: Do you like ketchup?            ← Message body begins
C: How about pickles?
C: .                               ← A single period on its own line = end of message
S: 250 Message accepted for delivery

C: QUIT                            ← Closing the connection
S: 221 hamburger.edu closing connection
```

**Breaking down the five commands:**

|Command|What it means|
|---|---|
|`HELO`|Client introduces itself (abbreviation of HELLO)|
|`MAIL FROM:`|Specifies the sender's address|
|`RCPT TO:`|Specifies the recipient's address|
|`DATA`|Signals the start of the message body; server prompts for it|
|`QUIT`|End the session and close the TCP connection|

**The period terminator:** The message body ends with `CRLF.CRLF` — a line containing a single period. In ASCII jargon, CR = carriage return, LF = line feed. This is how the SMTP server knows the message is complete without needing a `Content-Length:` header.

---

## Persistent Connections in SMTP

> **SMTP uses persistent connections.** If the sending mail server has several messages to send to the same receiving mail server, it can send all messages over the **same TCP connection**.

For each new message on the same connection, the client starts with a fresh `MAIL FROM:` command and only issues `QUIT` after all messages have been sent.

> **Try it yourself:** Open a terminal and run `telnet serverName 25`. You will immediately receive the `220` reply. Then issue `HELO`, `MAIL FROM`, `RCPT TO`, `DATA`, `CRLF.CRLF`, and `QUIT` in sequence to send an actual e-mail through a live SMTP server.

_(The textbook's Programming Assignment 3 walks through exactly this — building a simple user agent that implements the client side of SMTP, capable of sending a message to an arbitrary recipient via a local mail server.)_

---

# 2.3.2 Mail Message Formats

> Just like postal mail has a peripheral header at the top of the letter (addresses, date), an e-mail message has **header lines** that precede the body.

Mail message headers are defined in **RFC 5322**. The format:

```
From: alice@crepes.fr
To: bob@hamburger.edu
Subject: Searching for the meaning of life.
                              ← blank line (CRLF) separates header from body
(message body in ASCII follows here)
```

**Key header lines:**

|Header|Required?|What it contains|
|---|---|---|
|`From:`|**Yes**|Sender's e-mail address|
|`To:`|**Yes**|Recipient's e-mail address|
|`Subject:`|Optional|A description of the message|
|`CC:`|Optional|Carbon copy recipients|
|`Date:`|Optional|When the message was composed|

> **Critical distinction:** These header lines (`From:`, `To:`, `Subject:`) are part of the **mail message itself** — they are different from the SMTP commands (`MAIL FROM:`, `RCPT TO:`) used during the SMTP handshake. The SMTP commands are part of the **protocol exchange**; the header lines are part of the **message content**. They happen to contain some of the same words ("from" and "to") which can be confusing — but they serve entirely different purposes at different layers.

---

# 2.3.3 Mail Access Protocols

> Once SMTP delivers a message into Bob's mailbox, Bob's user agent still can't fetch it using SMTP — obtaining messages is a **pull** operation, and SMTP is strictly a **push** protocol. A separate access protocol is needed.

### Why Mail Servers Don't Live on Your Laptop

It might seem natural to just run a mail server directly on Bob's phone or PC, so Alice's server could hand the message straight to Bob's device. The catch: a mail server has to stay **always on and connected** to the Internet, since mail can arrive at any moment — impractical for a device that gets shut down, put to sleep, or carried out of signal range.

Instead, the standard setup is:

- Bob runs a lightweight **user agent** on his own device
- His **mailbox** lives on an **always-on, shared mail server** (run by his ISP, university, or webmail provider)

### Why the Two-Step Relay (Alice's Agent → Alice's Server → Bob's Server)

> Alice's user agent doesn't dialogue directly with Bob's mail server. Instead, it uses **SMTP or HTTP** to hand the message to **her own** mail server first, which then uses SMTP (as an SMTP client) to relay it on to Bob's mail server.

Why not skip the middleman? Without first depositing the message on her own server, Alice's user agent would have **no recourse** if Bob's mail server happened to be unreachable at that exact moment. By depositing it locally first:

- Alice's mail server can keep retrying delivery to Bob's server (e.g. every 30 minutes) until it comes back online
- If Alice's own mail server is the one that's down, that's now a problem for her system administrator — not something her user agent has to solve

![[Pasted image 20260619221903.png]] _(Figure 2.14 — E-mail protocols and their communicating entities: Alice's agent reaches Alice's mail server via SMTP or HTTP; Alice's mail server relays to Bob's mail server via SMTP; Bob's agent pulls from Bob's mail server via HTTP or IMAP)_

**The three mail access protocols, today:**

|Protocol|Port|RFC|Used by|
|---|---|---|---|
|**HTTP**|**80 / 443**|—|Web-based mail and smartphone apps (Gmail, Yahoo! Mail)|
|**IMAP** (Internet Mail Access Protocol)|**143**|RFC 3501|Desktop mail clients (Microsoft Outlook)|

Both approaches let Bob manage **folders that live on his mail server** — create folders, move messages between them, delete messages, mark messages as important — and see the exact same state from any device he logs in from.

---

## HTTP-Based Access (Web-Mail & Smartphone Apps)

> If Bob is using web-based e-mail or a smartphone app, his user agent uses **HTTP** to retrieve his e-mail. This requires Bob's mail server to run **both** an HTTP interface (for his user agent) **and** an SMTP interface (to keep receiving mail from other mail servers).

**The protocol split for web-mail:**

|Leg of the journey|Protocol used|
|---|---|
|Bob's browser → Bob's mail server (reading)|**HTTP**|
|Alice's browser → Alice's mail server (sending)|**HTTP**|
|Alice's mail server → Bob's mail server|**SMTP** (unchanged)|

The key insight: HTTP only handles the **last mile** between the user's browser and their own mail server. Server-to-server transport is always SMTP, regardless of whether the users are accessing mail via browser, Outlook, or a phone app.

> **Analogy:** Think of the web mail server as a **bilingual post office employee** who speaks English (HTTP) with customers at the counter, and speaks the internal postal language (SMTP) when sending packages to other post offices. The customer never sees or speaks the postal language.

---

## IMAP — Internet Mail Access Protocol

> IMAP (RFC 3501) maintains a **folder hierarchy on the server** that is accessible, in the exact same state, from any device.

**The analogy for IMAP:**

> Think of IMAP as a **smart, always-on filing cabinet** that lives at the post office and that you can access from any branch. When mail arrives, it goes into your INBOX folder in the cabinet. You can create folders (Work, Personal, Receipts), move messages between them, search across all folders, and read individual messages — all while the mail stays **on the server**. Whether you access it from your phone, laptop, or a computer at an airport, you see the exact same folders and messages in the exact same state.

**IMAP capabilities:**

|Feature|Details|
|---|---|
|**Remote folders**|Folders live on the server, not just locally on the client|
|**Move messages between folders**|Fully supported, and reflected on every device|
|**Search remote folders**|Search by sender, subject, content — server-side|
|**State across sessions**|Server remembers folder structure and which messages are in which folders|
|**Fetch partial messages**|Can fetch just a header, or one part of a multipart message|
|**Nomadic/multi-device use**|Built in by design — log in anywhere, see the same state|

**IMAP's folder model:**

When a message first arrives at the IMAP server, it is automatically placed in the recipient's **INBOX** folder. The recipient can then:

- Move it to a user-created folder (e.g. "Work Projects")
- Read, delete, or reply to it
- Search remote folders for messages matching specific criteria

All of this state — which messages are in which folders — is **maintained by the IMAP server across sessions**. Log in from your phone after reading email on your laptop, and the phone sees exactly the same folder state.

**Partial message fetching — IMAP's killer feature for slow connections:**

> IMAP has commands that permit a user agent to obtain just one **component** of a message — for example, just the message header, or just one part of a multipart MIME message.

This is hugely valuable on **low-bandwidth connections** (e.g. a slow mobile network). Instead of downloading a 15 MB message with a video attachment just to check who sent it, your phone can fetch only the headers. You decide whether to download the full message.

---

## HTTP vs IMAP — Quick Reference

|Dimension|HTTP-based|IMAP|
|---|---|---|
|**Typical client**|Browser, smartphone app (Gmail, Yahoo!)|Desktop mail client (Outlook)|
|**Server interface required**|HTTP interface + SMTP interface|IMAP interface (port 143) + SMTP interface|
|**Folder management**|On the server|On the server|
|**State across sessions**|Maintained server-side|Maintained server-side|
|**Partial fetch**|Depends on client implementation|Native protocol feature (headers / MIME parts)|

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**SMTP sender spoofing**|The `MAIL FROM:` in the SMTP handshake and the `From:` in the message header are **completely independent** — an attacker can put any address in either field. There is no built-in authentication of the sender's identity|**SPF** (Sender Policy Framework) — DNS records specifying which servers are allowed to send mail for a domain. **DKIM** (DomainKeys Identified Mail) — cryptographic signature in a header proving the message came from the domain's mail server. **DMARC** — policy tying SPF and DKIM together|
|**Phishing via spoofed headers**|`From: ceo@yourcompany.com` in the message header looks real to any client; the actual `MAIL FROM:` in the SMTP exchange may be `attacker@evil.com` — most users never see SMTP-level details|User training; e-mail clients that display the full sender address; DMARC enforcement|
|**SMTP cleartext (port 25)**|Port 25 (plain SMTP) sends everything — including message content — in cleartext. Anyone on the path can read your messages|Use **STARTTLS** (upgrades the SMTP connection to TLS) or **SMTPS** (SMTP over TLS, port 465). TLS encrypts the connection between mail servers|
|**IMAP cleartext credentials**|Plain IMAP (port 143) sends login credentials unencrypted — trivially sniffable on a shared network|Use **IMAPS** (IMAP over TLS, port 993). All modern mail clients support this|
|**Message queue as attack surface**|If Alice's mail server is compromised, her entire outgoing message queue — including messages to high-value targets — is exposed. Queue also reveals who she corresponds with|Encrypt mail servers; restrict access to the queue directory; use end-to-end encryption (PGP/GPG) so even a compromised server can't read message content|
|**MIME encoding abuse**|Attackers encode malicious executables as Base64 MIME attachments to bypass naive content filters that only inspect plaintext. The 7-bit ASCII constraint that forced MIME into existence is the same constraint attackers exploit|Modern mail gateways decode and scan MIME parts before delivery. Content filtering, sandboxing, file-type blocking for dangerous extensions (`.exe`, `.vbs`, `.js`)|
|**Open relays**|A mail server that relays messages for any sender without authentication is an **open relay** — spammers exploit these to send millions of messages using someone else's infrastructure|Mail servers must require **SMTP AUTH** (authentication) before accepting messages for relay. Modern MTAs default to closed relay|
|**Web-based mail XSS**|HTML-formatted e-mail rendered in a browser is a vector for **Cross-Site Scripting (XSS)** — malicious JavaScript embedded in an HTML e-mail executes in the victim's browser context|Mail clients sanitise HTML before rendering; Content Security Policy headers; disabling JavaScript in mail rendering contexts|

---

## Questions I Still Have

- [ ] The `From:` header in the mail message and the `MAIL FROM:` in the SMTP handshake are different — which one does my mail client (Gmail, Outlook) actually display as "From"? And when SPF/DKIM fail, what does the user actually see?
- [ ] IMAP maintains folder state on the server — but what happens when two clients (phone + laptop) both modify the same message simultaneously? Is there a locking mechanism, or last-write-wins?
- [ ] SMTP uses port 25 between mail servers, but modern mail clients submit outgoing mail on port **587** (SMTP Submission) — what is the difference between the two ports architecturally, and why was submission separated from relay?
- [ ] End-to-end encrypted mail (PGP/GPG, S/MIME) encrypts the message body — but does it also encrypt the headers (`To:`, `From:`, `Subject:`)? Or can intermediate servers still read the metadata?
- [ ] How exactly does MIME handle a message with both HTML and plaintext bodies — the `multipart/alternative` type? Which version does a client display, and how does it decide?
- [ ] If both HTTP-based and IMAP-based access let Bob manage folders on the server, is there ever a real practical reason to prefer one over the other — or is it purely a client-software choice (browser vs. Outlook)?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**E-mail / Electronic mail**|Asynchronous communication medium — messages sent and read at each person's convenience|
|**User agent**|Application through which users compose, read, reply to, and manage e-mail (Outlook, Apple Mail, Gmail browser)|
|**Mail server**|Core infrastructure component — stores mailboxes, manages outgoing message queues, runs both SMTP client and server|
|**Mailbox**|Per-user storage on a mail server that holds incoming messages until retrieved; access requires authentication|
|**Message queue**|Outgoing mail on a server awaiting successful delivery; retried periodically if delivery fails|
|**SMTP**|Simple Mail Transfer Protocol — the application-layer protocol that pushes e-mail between mail servers; defined in RFC 5321|
|**HELO**|SMTP command with which the client introduces itself to the server|
|**MAIL FROM:**|SMTP command specifying the sender's address (part of the SMTP handshake — distinct from the `From:` message header)|
|**RCPT TO:**|SMTP command specifying the recipient's address|
|**DATA**|SMTP command signalling the start of the message body|
|**QUIT**|SMTP command closing the connection|
|**Push protocol**|Protocol where the sender initiates delivery — SMTP|
|**Pull protocol**|Protocol where the receiver initiates retrieval — HTTP, IMAP|
|**7-bit ASCII constraint**|SMTP restriction requiring all message bodies to be 7-bit ASCII; binary attachments must be Base64-encoded|
|**MIME**|Multipurpose Internet Mail Extensions — encoding standard that allows binary data and non-ASCII text to travel over SMTP|
|**RFC 5322**|RFC defining the format of e-mail messages (header lines and body)|
|**IMAP**|Internet Mail Access Protocol — mail access protocol on port 143 (RFC 3501); manages folders on the server; maintains state across sessions; supports partial message fetch|
|**Web-based e-mail**|E-mail accessed through a browser using HTTP to communicate with the mail server (Gmail, Yahoo! Mail, Outlook.com); requires the mail server to run an HTTP interface alongside its SMTP interface|
|**SPF**|Sender Policy Framework — DNS-based mechanism specifying which servers may send mail for a domain|
|**DKIM**|DomainKeys Identified Mail — cryptographic header signature proving message authenticity|
|**DMARC**|Policy framework tying SPF and DKIM together to handle failures|
|**STARTTLS**|SMTP extension that upgrades a plaintext connection to TLS mid-session|
|**Open relay**|A misconfigured mail server that forwards messages for any sender — exploited by spammers|

---

## Related Concepts

---

→ Next: [[DNS]]