---
title: FILE TRANSFER - FTP
date: 2026-06-04
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.3 File Transfer: FTP

> **One-Line Summary:** FTP transfers files between a local and remote host using **two separate TCP connections** — a persistent control connection (port 21) for commands and a non-persistent data connection (port 20) for actual file bytes — making it **stateful** and fundamentally different from HTTP.

---

## Core Idea

**FTP (File Transfer Protocol)** is one of the oldest application-layer protocols on the Internet — predating the Web entirely. While HTTP transfers objects embedded in web pages, FTP's job is exactly what it sounds like: moving raw files from one host's filesystem to another's.

The defining architectural choice of FTP — one that makes it unique among the protocols in this chapter — is using **two separate TCP connections simultaneously**: one for control (commands), one for data (files). This split is the key to understanding everything else about how FTP works.

From a cybersecurity perspective, FTP is a textbook example of a legacy protocol that was designed without security in mind. Everything — usernames, passwords, file contents — travels in **cleartext**. Modern usage has largely replaced it with SFTP (SSH File Transfer Protocol) and FTPS (FTP over TLS), but understanding plain FTP first is essential.

---

# 2.3.0 FTP — Overview and Session Setup

## How an FTP Session Begins

> In a typical FTP session, the user sits in front of one host (the **local host**) and wants to transfer files to or from a **remote host**.

The flow of a session from start to first file transfer:

```
Step 1 → User provides hostname of the remote host
         → FTP client process on local host initiates a
           TCP connection with the FTP server process on remote host

Step 2 → User provides user identification and password
         → Sent over the TCP connection as FTP commands

Step 3 → Server authorises the user

Step 4 → User copies files between local and remote filesystems
         (in either direction)
```

The user interacts with FTP through an **FTP user agent** — this could be a GUI application (like FileZilla), a command-line `ftp` program, or a browser's built-in FTP client.

![[Pasted image 20260604105842.png]] _(Figure 2.14 — FTP moves files between local and remote file systems: the user interacts through an FTP user interface → FTP client → FTP server, with file transfer in both directions; each side has its own local file system)_

---

# 2.3.1 FTP's Two TCP Connections — Control and Data

## The Defining Feature of FTP

> **FTP uses two parallel TCP connections to transfer a file: a control connection and a data connection.**

This is the most striking difference between FTP and HTTP. HTTP uses a single TCP connection for everything. FTP splits the job across two:

|Connection|Port|Purpose|Persistence|
|---|---|---|---|
|**Control connection**|**21**|Sends commands (USER, PASS, LIST, RETR, STOR) and receives replies|**Persistent** — stays open for the entire session|
|**Data connection**|**20**|Transfers actual file bytes|**Non-persistent** — opened for one file, then closed|

![[Pasted image 20260604110018.png]] _(Figure 2.15 — Control and data connections: the FTP client maintains a persistent TCP control connection on port 21 throughout the session; a separate TCP data connection on port 20 is opened and closed for each individual file transfer)_

---

## Control Connection — The Long-Lived Pipe

When a user starts an FTP session with a remote host:

1. The **client side initiates a control TCP connection** with the server on **port 21**
2. The client sends the user identification and password **over this control connection**
3. The client also sends commands to **change the remote directory** over this connection
4. The control connection **remains open for the entire duration of the session**

The control connection carries nothing but **commands and replies** — no file data ever flows through it. Every piece of file data gets its own dedicated data connection.

---

## Data Connection — The Short-Lived Pipe

When the server receives a command for a file transfer over the control connection:

```
Server side initiates a TCP data connection to the client side
  → File is sent over the data connection
  → Data connection is closed immediately after

If another file needs to be transferred:
  → A brand new data connection is opened
  → File is sent
  → Data connection is closed again
```

> **Each file gets exactly one data connection — opened fresh, used once, then closed.** Data connections are non-persistent.

---

## Out-of-Band vs In-Band Control

> Because FTP uses a **separate** control connection, FTP is said to send its control information **out-of-band**.

> HTTP, by contrast, sends request and response headers on the **same** TCP connection that carries the transferred file — HTTP sends control information **in-band**.

This is a fundamental protocol design distinction:

| Protocol | Control Channel             | Data Channel           | Same Connection?     |
| -------- | --------------------------- | ---------------------- | -------------------- |
| **FTP**  | Port 21 (separate TCP)      | Port 20 (separate TCP) | No — **out-of-band** |
| **HTTP** | Embedded in same TCP stream | Same TCP stream        | Yes — **in-band**    |
| **SMTP** | In-band (same connection)   | In-band                | Yes — **in-band**    |

The out-of-band approach gives FTP more flexibility (you can keep issuing commands while a transfer is in progress on the data connection) but adds complexity by requiring two simultaneous connections.

---

# 2.3.2 FTP is Stateful — Unlike HTTP

> **Throughout a session, the FTP server must maintain state about the user.**

Specifically, the server must:

- Associate the **control connection** with a specific user account
- Keep track of the **user's current directory** as the user navigates the remote file system

This is in sharp contrast to HTTP:

| Protocol | State Maintained?                                           | Impact                                                                                    |
| -------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **HTTP** | **None** — stateless by design                              | Trivially scalable; crash recovery is simple                                              |
| **FTP**  | **Yes** — current directory, authenticated user per session | Significantly constrains the total number of simultaneous sessions the server can support |

> Keeping track of this state information for each ongoing user session significantly constrains the total number of sessions that FTP can maintain simultaneously. HTTP, on the other hand, is stateless — it does not have to keep track of any user state.

---

# 2.3.3 FTP Commands and Replies

## Commands (Client → Server)

FTP commands are sent across the **control connection** in **7-bit ASCII format** — just like HTTP commands, they are human-readable. Each command consists of **four uppercase ASCII characters** (some with optional arguments). Commands are delimited by a carriage return and line feed (`\r\n`).

**The most common FTP commands:**

| Command         | What it does                                                                                                                                                                        |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `USER username` | Sends the user identification to the server                                                                                                                                         |
| `PASS password` | Sends the user password to the server                                                                                                                                               |
| `LIST`          | Asks the server to send back a list of all files in the current remote directory. The file list is sent over a **new, non-persistent data connection** (not the control connection) |
| `RETR filename` | Retrieves (gets) a file from the current directory of the remote host. Causes the remote host to initiate a data connection and send the file over it                               |
| `STOR filename` | Stores (puts) a file into the current directory of the remote host                                                                                                                  |

> There is typically a **one-to-one correspondence** between the command the user issues and the FTP command sent across the control connection.

---

## Replies (Server → Client)

Each FTP command sent from client to server is followed by a reply sent **back from server to client** over the control connection.

> **FTP replies are three-digit numbers**, optionally followed by a human-readable message. This structure is similar to the status code and phrase in the HTTP response status line.

**Common FTP replies:**

|Reply Code|Meaning|
|---|---|
|`331`|Username OK, password required|
|`125`|Data connection already open; transfer starting|
|`425`|Can't open data connection|
|`452`|Error writing file|

The numeric code is what software acts on; the message is for human operators reading logs.

> For the complete list of FTP commands and replies, see **RFC 959**.

---

# FTP vs HTTP — Side-by-Side Comparison

|Feature|HTTP|FTP|
|---|---|---|
|**Transport**|TCP|TCP|
|**Number of TCP connections**|1 (or 1 persistent + reuse)|2 (control + data)|
|**Control port**|80 (HTTP) / 443 (HTTPS)|**21**|
|**Data port**|Same as control|**20**|
|**Control channel style**|In-band|**Out-of-band**|
|**Stateful?**|**No** — stateless|**Yes** — tracks directory, user|
|**Data connection persistence**|Persistent (HTTP/1.1)|Non-persistent (one per file)|
|**Authentication**|None built-in (handled by cookies/sessions/TLS)|Built-in (`USER`/`PASS` commands)|
|**Encryption (plain)**|None (cleartext)|None (cleartext)|
|**Secure variant**|HTTPS (HTTP + TLS)|SFTP (SSH) / FTPS (FTP + TLS)|
|**Commands readable?**|Yes (ASCII)|Yes (ASCII)|
|**Modern usage**|Dominant — every web request|Largely replaced by SFTP/cloud storage|

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Cleartext credentials**|`USER` and `PASS` commands travel in plaintext over the control connection — a passive sniffer (Wireshark on the same network segment) captures the username and password trivially|Never use plain FTP for anything sensitive. Use **SFTP** (SSH-based, encrypts everything) or **FTPS** (FTP + TLS, encrypts both control and data channels)|
|**Cleartext file content**|Every byte of every file transferred over the data connection is readable by anyone on the path — corporate documents, source code, config files|Same solution: SFTP or FTPS. At minimum, encrypt the files before FTP if legacy infrastructure forces plain FTP|
|**PORT command injection (Bounce Attack)**|In active FTP mode, the client specifies an IP:port for the server to connect to. A malicious client can specify someone else's IP — using the FTP server as a proxy to port-scan or attack third parties|Use **passive FTP mode** (PASV), where the server specifies the data connection endpoint instead. Modern firewalls and clients default to passive mode|
|**Anonymous FTP**|Many FTP servers allow `USER anonymous` + any email as password — intended for public downloads but misconfigured servers expose internal files|Restrict anonymous access strictly. Anonymous FTP should have read-only access to a dedicated public directory only|
|**FTP port 21 exposure**|Port 21 on a public-facing server is a frequent automated scan target. Brute-force login attacks are common against FTP servers|Rate-limit login attempts, use fail2ban, move SFTP to a non-standard port, use key-based authentication instead of passwords|
|**Out-of-band data channel**|The dynamic data connection on port 20 (or a PASV-negotiated port) can complicate stateful firewall rules, potentially creating gaps in coverage|Proper stateful firewalling handles FTP-aware inspection (FTP ALG — Application Layer Gateway) to track data connections correctly|

---

## Questions I Still Have

- [ ] In **active FTP mode**, the server initiates the data connection back to the client — this breaks behind NAT (router doesn't know to forward port 20 to the right client). How exactly does **passive mode (PASV)** solve this, and what port range does it use?
- [ ] SFTP is actually SSH File Transfer Protocol — it has nothing to do with FTP internally, it's a completely separate protocol over SSH port 22. How does it differ architecturally from FTPS (which is genuinely FTP + TLS)?
- [ ] FTP is said to be non-persistent for data connections — but wouldn't the overhead of setting up a new TCP data connection for every file be significant for bulk transfers? Is there an FTP extension that reuses the data connection?
- [ ] The `LIST` command sends its output over a **data connection** — so listing a remote directory actually opens a temporary TCP connection just to receive the file listing? What is the overhead in practice?
- [ ] How do modern browsers handle `ftp://` URLs? Have major browsers (Chrome, Firefox) completely dropped FTP support, and if so, when and why?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**FTP**|File Transfer Protocol — application-layer protocol for transferring files between a local and remote host|
|**FTP user agent**|The interface through which the user interacts with FTP (e.g. FileZilla, command-line `ftp`, browser)|
|**Control connection**|The persistent TCP connection on port 21 used to send FTP commands and receive replies throughout a session|
|**Data connection**|The non-persistent TCP connection on port 20 opened for each individual file transfer, then closed|
|**Out-of-band**|Control information sent on a **separate** channel from the data — FTP's approach|
|**In-band**|Control information sent on the **same** channel as the data — HTTP's and SMTP's approach|
|**Active FTP**|FTP mode where the **server** initiates the data connection back to the client — problematic with NAT and firewalls|
|**Passive FTP (PASV)**|FTP mode where the **client** initiates the data connection to a port the server specifies — NAT-friendly|
|**USER**|FTP command to send user identification to the server|
|**PASS**|FTP command to send user password to the server|
|**LIST**|FTP command to retrieve the current remote directory listing (sent over a new data connection)|
|**RETR**|FTP command to retrieve (download) a file from the remote host|
|**STOR**|FTP command to store (upload) a file to the remote host|
|**331**|FTP reply: Username OK, password required|
|**125**|FTP reply: Data connection already open, transfer starting|
|**Stateful protocol**|Protocol where the server maintains information about each client across interactions (contrast with HTTP's statelessness)|
|**SFTP**|SSH File Transfer Protocol — a completely separate, secure protocol over SSH port 22; not FTP + encryption|
|**FTPS**|FTP Secure — genuine FTP extended with TLS encryption on both the control and data connections|
|**RFC 959**|The RFC defining the FTP standard|

---

## Related Concepts

-

---

→ Next: [[ELECTRONIC MAIL]]