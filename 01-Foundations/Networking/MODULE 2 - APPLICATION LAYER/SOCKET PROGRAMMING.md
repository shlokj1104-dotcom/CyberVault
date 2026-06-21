---
title: SOCKET PROGRAMMING
date: 2026-06-20
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.6 Socket Programming: Creating Network Applications

> **One-Line Summary:** A socket is the programming interface between an application and the network — every networked program you have ever written or used ultimately reads from and writes to a socket. This section shows how to actually build both ends of a client-server application: first in UDP, then in TCP, and finally a high-level look at how an application talks to a QUIC API — all demonstrated in Python 3.

---

## Core Idea: What Is a Socket?

Recall from §2.1 that a typical network application consists of a pair of programs — a client program and a server program — residing in two different end systems. When these two programs are executed, a client process and a server process are created, and these processes communicate with each other by **reading from, and writing to, sockets**. When creating a network application, the developer's main task is therefore to write the code for both the client and the server programs.

> **Analogy — House and Door:** Each process is analogous to a house, and the process's socket is analogous to a door. The application resides on one side of the door, inside the house; the transport-layer protocol resides on the other side of the door, in the outside world. The application developer has control of everything on the application-layer side of the socket; however, it has little control of the transport-layer side.

When a network application runs, the operating system creates a pair of socket objects — one in each process — and data flows between them. The developer's job is to write the logic on both sides: what the client sends, how the server processes it, and what it sends back.

---

## Two Types of Network Applications

Before writing a single line of code, there's a design decision that's already been made for you by context: is your application **open** (defined by a public RFC standard) or **proprietary**?

|Type|What it means|Who can interoperate|Example|
|---|---|---|---|
|**Open / RFC-defined**|The protocol's operation is specified in a protocol standard such as an RFC; the rules specifying its operation are known to all, and client/server programs must conform to those rules|Any independent developer who follows the spec|A Google Chrome browser communicating with an Apache Web server (the client side and server side of HTTP are both precisely defined in RFC 2616), or a BitTorrent client communicating with a BitTorrent tracker|
|**Proprietary**|The client and server programs employ an application-layer protocol that has **not** been openly published in an RFC or elsewhere. A single developer (or development team) creates both programs and has complete control over the code|Only that developer's own code — other independent developers cannot build code that interoperates with the application|A proprietary IoT sensor talking to its vendor's own cloud backend|

The practical consequence for development: if the application implements a protocol defined by an RFC, it should use the **well-known port number** associated with that protocol. If the application is proprietary, the developer must be careful to **avoid** using those well-known port numbers. (Port numbers were briefly introduced in §2.1; they're covered in more detail in Chapter 3.)

---

## First Decision: UDP or TCP?

During the development phase, one of the first decisions a developer must make is whether the application is to run over **TCP** or **UDP**.

|Property|UDP|TCP|
|---|---|---|
|**Connection**|Connectionless — sends independent packets of data from one end system to the other, without any guarantees about delivery|Connection-oriented — provides a reliable byte-stream channel through which data flows between two end systems; client and server must handshake before sending any data|
|**Reliability**|None — the sending process explicitly attaches a destination address to every packet; no guarantee the packet arrives, arrives once, or arrives in order|Reliable byte-stream — TCP guarantees every byte arrives, exactly once, and in order|
|**Overhead**|Minimal — fire and forget|Higher — connection setup, acknowledgements, congestion control all add overhead|
|**Use case**|Speed matters more than guaranteed delivery — streaming, gaming, DNS|Correctness matters — web browsing, email, file transfer|
|**Port number handling**|Developer explicitly attaches destination IP + port to every packet sent; source port is attached automatically by the OS|Port numbers are baked into the connection at setup time; after that, just write bytes into the pipe|

> **Why Python?** Simple UDP and TCP applications can be presented in Python, Java, C, or C++ — this section uses **Python 3**. Python is chosen because it clearly exposes the key socket concepts: there are fewer lines of code, and each line can be explained to a novice programmer without difficulty. There's no need to worry if you're unfamiliar with Python — the code is easy to follow with experience in Java, C, or C++. (Java versions of all examples and associated labs are available on the textbook's companion website; for readers interested in C, see [Donahoo 2001; Stevens 1997; Frost 1994] — the Python examples below have a similar look and feel to C.)

**The demo application used throughout this section:**

```
1. Client reads a line of characters (data) from its keyboard and
   sends the data to the server.
2. Server receives the data and converts the characters to uppercase.
3. Server sends the modified data to the client.
4. Client receives the modified data and displays the line on its screen.
```

This is minimal by design — the goal is to expose the socket mechanics as clearly as possible, not to build something useful. For this application, the server port number is (arbitrarily) chosen to be **12000**.

---

# 2.6.1 Socket Programming with UDP

## How UDP Sockets Work

Before the sending process can push a packet of data out the socket door, when using UDP it must first **attach a destination address** to the packet. After the packet passes through the sender's socket, the Internet uses this destination address to route the packet to the socket in the receiving process. When the packet arrives at the receiving socket, the receiving process retrieves it through the socket and inspects its contents.

What goes into that destination address?

- The **destination host's IP address** — so routers in the Internet can route the packet to the destination host
- The **destination socket's port number** — since a host may run many network applications, each with one or more sockets, the port number identifies the _particular_ socket on that host. A port number is assigned to a socket when the socket is created.

> In summary, the sending process attaches a destination address — destination host's IP address + destination socket's port number — to the packet. The sender's **source address** (source host's IP address + source socket's port number) is also attached to the packet, but attaching the source address is typically **not** done by the application code — it's automatically done by the underlying operating system.

![[Pasted image 20260620161711.png]]_(Figure 2.22 — The client-server application using UDP: the client creates a socket, attaches a destination address, and sends a datagram; the server reads the datagram from its socket, writes its reply back to the client's address, and stays in its loop waiting for the next packet)_

> **Analogy — A Walkie-Talkie Dispatcher:** UDP socket communication is like walkie-talkie radio. Each transmission is independent — the sender announces the recipient's channel (destination address), sends the message, and waits to hear back. Nobody "connects" to anyone; every transmission stands alone. The dispatcher (server) just sits on their channel all day, writing down who called in (`clientAddress`) so they can radio back.

The client program is called `UDPClient.py`, and the server program is called `UDPServer.py`. The code below is intentionally minimal, to emphasize the key issues — "good code" would have a few more auxiliary lines, particularly for handling error cases.

---

## UDPClient.py — Line by Line

```python
from socket import *
serverName = 'hostname'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input('Input lowercase sentence:')
clientSocket.sendto(message.encode(),(serverName, serverPort))
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())
clientSocket.close()
```

**`from socket import *`** Imports Python's socket module. By including this line, the program can create sockets.

**`serverName = 'hostname'` and `serverPort = 12000`** `serverName` is a string containing either the IP address of the server (e.g., `'128.138.32.126'`) or the server's hostname (e.g., `'cis.poly.edu'`). If a hostname is given, a **DNS lookup** is automatically performed to obtain the IP address. `serverPort` is set to 12000.

**`clientSocket = socket(AF_INET, SOCK_DGRAM)`** This line creates the client's socket, `clientSocket`. The first parameter, `AF_INET`, indicates the underlying network is using **IPv4**. The second parameter, `SOCK_DGRAM`, means this socket is of type **UDP** (rather than TCP). Note that the port number of the client socket is _not_ specified when it's created — the operating system does this automatically.

**`message = input('Input lowercase sentence:')`** `input()` is a built-in Python function. The user at the client is prompted with the words "Input lowercase sentence:" and types a line, which is put into the variable `message`.

**`clientSocket.sendto(message.encode(),(serverName, serverPort))`** First, the message is converted from string type to byte type using the **`.encode()`** method — bytes (not strings) must be sent into a socket. The method `sendto()` then attaches the destination address `(serverName, serverPort)` to the message and sends the resulting packet into the client's socket. (The source address is also attached, but automatically, by the OS rather than explicitly by the code.) Sending a client-to-server message via a UDP socket is that simple.

**`modifiedMessage, serverAddress = clientSocket.recvfrom(2048)`** Blocks here, waiting for a packet to arrive. When a packet arrives, its data is put into `modifiedMessage` and its source address is put into `serverAddress` (which contains both the server's IP address and port number — not actually needed by `UDPClient`, since it already knows the server's address, but provided nevertheless). `2048` is the buffer size in bytes — sufficient for most purposes.

**`print(modifiedMessage.decode())`** Prints `modifiedMessage` on the user's display, after converting it from bytes back to a string with **`.decode()`**. It should be the original line the user typed, now capitalized.

**`clientSocket.close()`** Closes the socket. The process then terminates.

---

## UDPServer.py — Line by Line

```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))
print("The server is ready to receive")
while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```

The beginning of `UDPServer` is similar to `UDPClient` — it imports the socket module, sets `serverPort` to 12000, and creates a `SOCK_DGRAM` (UDP) socket. The first line of code significantly different from `UDPClient` is:

**`serverSocket.bind(('', serverPort))`** This **binds** (assigns) port number 12000 to the server's socket. Thus, in `UDPServer`, the code explicitly assigns a port number to the socket — when anyone sends a packet to port 12000 at the server's IP address, that packet is directed to this socket.

**`while True:`** `UDPServer` then enters this loop, which allows it to receive and process packets from clients **indefinitely**.

**`message, clientAddress = serverSocket.recvfrom(2048)`** When a packet arrives at the server's socket, its data is put into `message` and its source address into `clientAddress` (containing both the client's IP address and port number). Here, `UDPServer` _will_ make use of this address information — it provides a return address, similar to the return address on ordinary postal mail, telling the server where to direct its reply.

**`modifiedMessage = message.decode().upper()`** This is the heart of the application — it converts the received bytes to a string with `.decode()`, then capitalizes it with `.upper()`.

**`serverSocket.sendto(modifiedMessage.encode(), clientAddress)`** This converts the capitalized string back to bytes with `.encode()`, attaches the client's address to it, and sends the resulting packet into the server's socket. The Internet then delivers the packet to that client address. After sending, the server remains in the `while` loop, waiting for another UDP packet to arrive — from any client running on any host.

---

## Testing and Extending the UDP Application

To test the pair of programs, run `UDPClient.py` on one host and `UDPServer.py` on another (being sure to include the proper hostname or IP address of the server in `UDPClient.py`). Run `UDPServer.py` first, so the server process idles, waiting to be contacted; then run `UDPClient.py` and type a sentence followed by a carriage return.

To develop your own UDP client-server application, you can begin by slightly modifying these programs. For example, instead of converting all the letters to uppercase, the server could count the number of times the letter `s` appears and return that number — or you could modify the client so that after receiving a capitalized sentence, the user can continue sending more sentences to the server.

---

# 2.6.2 Socket Programming with TCP

Unlike UDP, **TCP is a connection-oriented protocol**. This means that before the client and server can start sending data to each other, they must first **handshake and establish a TCP connection**. One end of the connection is attached to the client socket, the other to a server socket. When creating the connection, we associate with it the client socket address (IP + port) and the server socket address (IP + port). With the connection established, when one side wants to send data to the other, it just **drops the data into the TCP connection** via its socket — this is different from UDP, where the application must attach a destination address to the packet before dropping it into the socket.

## The Client Initiates; the Server Must Be Ready

The client has the job of **initiating** contact with the server. For the server to be able to react to that contact, it must already be ready. This implies two things:

1. As with UDP, the TCP server must be running as a process **before** the client attempts to initiate contact
2. The server program must have a special door — more precisely, a special socket — that **welcomes** some initial contact from a client process running on an arbitrary host

Using the house/door analogy, we will sometimes refer to the client's initial contact as **"knocking on the welcoming door."**

With the server process running, the client process can initiate a TCP connection by creating a TCP socket. When the client creates its socket, it specifies the address of the welcoming socket in the server — the IP address of the server host and the port number of that socket. After creating its socket, the client initiates a **three-way handshake** and establishes a TCP connection with the server. The three-way handshake, which takes place within the transport layer, is completely **invisible** to the client and server programs.

During the handshake, the client process knocks on the welcoming door of the server process. When the server "hears" the knock, it creates a **new door** — more precisely, a _new_ socket dedicated to that particular client.

> **Common point of confusion:** Students encountering TCP sockets for the first time sometimes confuse the **welcoming socket** (the initial point of contact for _all_ clients wanting to communicate with the server) and the **connection socket** that is newly created for communicating with _each_ client.

|Socket|Name in code|Purpose|
|---|---|---|
|**Welcoming socket**|`serverSocket`|Sits open permanently, waiting for new clients to knock. It handles _only_ connection setup — it never carries actual data.|
|**Connection socket**|`connectionSocket`|Created fresh for each specific client that successfully connects. All actual data exchange (request, reply) flows through this one. Closed when the conversation with that client ends.|

From the application's perspective, the client's socket and the server's connection socket are directly connected by a pipe. The client process can send arbitrary bytes into its socket, and TCP guarantees that the server process will receive (through the connection socket) each byte in the order sent — TCP thus provides a **reliable service** between the client and server processes. Furthermore, just as people can go in and out the same door, the client process not only sends bytes into but also receives bytes from its socket; similarly, the server process not only receives bytes from but also sends bytes into its connection socket.

![[Pasted image 20260620161930.png]]
_(Figure 2.23 — The TCPServer process has two sockets: a welcoming socket that handles the three-way handshake, and a connection socket created fresh for each client that connects, through which all actual data flows in both directions)_

> **Analogy — A Hotel Reception Desk:** The welcoming socket is the hotel's main front door and reception desk — always open, designed to greet anyone who arrives and check them in. The connection socket is the specific room key issued to that guest: a dedicated, private channel just for them. The front desk (welcoming socket) stays open for the next guest, while the guest uses their room (connection socket) to do the actual stay.

---

## TCPClient.py — Line by Line

We use the same simple client-server application to demonstrate TCP socket programming: the client sends one line of data to the server, the server capitalizes it and sends it back.

```python
from socket import *
serverName = 'servername'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName,serverPort))
sentence = input('Input lowercase sentence:')
clientSocket.send(sentence.encode())
modifiedSentence = clientSocket.recv(1024)
print('From Server: ', modifiedSentence.decode())
clientSocket.close()
```

Let's focus on the lines that differ significantly from `UDPClient`.

**`clientSocket = socket(AF_INET, SOCK_STREAM)`** This creates the client's socket. The first parameter again indicates IPv4; the second, `SOCK_STREAM`, means this is a **TCP** socket (rather than UDP). As before, the port number of the client socket is not specified — the OS assigns it automatically.

**`clientSocket.connect((serverName,serverPort))`** Recall that before the client can send data to the server (or vice versa) using a TCP socket, a TCP connection must first be established. This line **initiates** that connection — its parameter is the address of the server side of the connection. After this line executes, the three-way handshake has been performed and a TCP connection is established between client and server.

**`sentence = input('Input lowercase sentence:')`** Obtains a sentence from the user, as with `UDPClient`.

**`clientSocket.send(sentence.encode())`** Sends `sentence` through the client's socket and into the TCP connection. Note that the program does **not** explicitly create a packet and attach a destination address, as was the case with UDP sockets — the client program simply **drops the bytes into the TCP connection**. The client then waits to receive bytes from the server.

**`modifiedSentence = clientSocket.recv(1024)`** When characters arrive from the server, they're placed into `modifiedSentence`.

**`clientSocket.close()`** Closes the socket and, hence, closes the TCP connection between client and server. This causes TCP in the client to send a TCP message to TCP in the server (covered in §3.5).

---

## TCPServer.py — Line by Line

```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET,SOCK_STREAM)
serverSocket.bind(('',serverPort))
serverSocket.listen(1)
print('The server is ready to receive')
while True:
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    connectionSocket.close()
```

**`serverSocket = socket(AF_INET,SOCK_STREAM)`** Creates a TCP socket, as with `TCPClient`.

**`serverSocket.bind(('',serverPort))`** Associates the server port number with this socket — similar to `UDPServer`.

**`serverSocket.listen(1)`** With TCP, `serverSocket` is our **welcoming** socket. This line has the server listen for TCP connection requests; the parameter specifies the maximum number of queued connections (at least 1).

**`connectionSocket, addr = serverSocket.accept()`** This is the most important line in the server. The program blocks here until a client knocks on the door. When a client calls `connect()`, the `accept()` method creates a **brand-new socket** in the server, called `connectionSocket`, dedicated entirely to that particular client. The client and server then complete the handshaking, establishing a TCP connection between the client's `clientSocket` and the server's `connectionSocket`. With the connection established, the client and server can send bytes to each other — all bytes sent from one side are guaranteed to arrive at the other, and in order. `serverSocket` remains open, and will accept the next client's connection request on the next loop iteration.

**`sentence = connectionSocket.recv(1024).decode()`** Reads the client's message from the _connection_ socket, not the welcoming socket, converting the received bytes to a string.

**`capitalizedSentence = sentence.upper()` and `connectionSocket.send(capitalizedSentence.encode())`** Capitalizes the message and sends it back, after converting back to bytes, through the same connection socket.

**`connectionSocket.close()`** After sending the modified sentence to the client, closes the connection socket — that conversation is done. `serverSocket` is intentionally **not** closed here; it stays open so the `while True:` loop can call `accept()` again and serve the next client.

![[Pasted image 20260620162127.png]]
_(Figure 2.24 — The client-server application using TCP: unlike UDP, the client must call `connect()` before any data flows, the server must call `listen()` and `accept()`, and data is exchanged through `connectionSocket` — not `serverSocket`)_

---

## UDP vs TCP Socket Programming — Side-by-Side Comparison

|Step|UDP|TCP|
|---|---|---|
|**Socket type constant**|`SOCK_DGRAM`|`SOCK_STREAM`|
|**Client: before sending**|Nothing — just create socket and go|Must call `.connect()` to establish a connection first|
|**Client: sending data**|`clientSocket.sendto(msg.encode(), (serverName, serverPort))` — destination attached per packet|`clientSocket.send(msg.encode())` — destination already known from connection setup|
|**Server: extra setup step**|`bind()` only|`bind()` + `listen()` + `accept()` (which creates a new `connectionSocket`)|
|**Server: two sockets?**|No — one socket handles both connection and data|Yes — `serverSocket` (welcoming) and `connectionSocket` (per-client data)|
|**Data delivery guarantee**|None — may be lost, duplicated, or reordered|Guaranteed in-order, exactly once|
|**Addressing per message**|Explicit — source and destination addresses attached to every packet|Implicit — baked into the connection; the byte-stream pipe handles it|
|**Close behavior**|`clientSocket.close()` just frees the local resource|`clientSocket.close()` sends a TCP message to the other side's TCP — a network event|
|**Bytes vs strings**|Application explicitly `.encode()`s strings to bytes before sending and `.decode()`s bytes to strings after receiving|Same — `.encode()`/`.decode()` required on both client and server sides|

You are encouraged to run the UDP and TCP program pairs on two separate hosts, modify them to achieve slightly different goals, and compare how they differ. Many of the socket programming assignments at the end of Chapters 2 and 3 build directly on these two programs.

---

# 2.6.3 Socket Programming with QUIC

This section closes with a high-level discussion of how an application developer would use a **QUIC API** to send messages between a client and a server.

## HTTP/3 Client Side

Consider what happens when a Web browser uses the **HTTP/3** protocol to request a Web page from a server also running HTTP/3. Instead of initiating a traditional TCP handshake, the client initiates a **QUIC handshake on top of UDP**.

- QUIC incorporates the **TLS 1.3** handshake into the QUIC handshake — meaning connection setup and encryption setup are done **in parallel**, not serially as with HTTP/2 (TLS-over-TCP)
- Even though UDP is connectionless, **QUIC is connection-oriented** — it allows the client and server to retain connection information across different HTTP requests, for the same or different Web pages

## HTTP/3 Server Side

Now consider the HTTP/3 server, with a connection already established between client and server. After receiving request(s) over the QUIC connection, the server needs to send HTTP messages — each encapsulating an object — back to the client.

```
HTTP/3 Server, given an established QUIC connection:

  For each object the server needs to send:
    → Ask the QUIC API to create a separate STREAM with its own
      stream ID

  Example for one Web page:
    Stream 1  →  HTML base page
    Stream 2  →  CSS object
    Stream 3  →  image

  All three streams' messages can then be sent back-to-back into
  the SAME QUIC connection.
```

When HTTP/3 wants to send a message, it specifies to the QUIC API the **QUIC connection ID** and the **stream ID**. Everything below that is handled internally by the QUIC sub-layer:

- **Encryption** of the data
- Breaking the streams (messages) into smaller **frames**
- **Multiplexing** the frames from different streams over the connection

The HTTP/3 server can also tell the QUIC API to **prioritize** streams based on the importance of the resource — for instance, prioritizing the HTML content (since it forms the basic structure of the Web page) so it is sent before other resources like images or JavaScript.

> **Important:** The HTTP/3 code interacts with the **QUIC API**, not with the underlying UDP socket API directly. So from the perspective of the HTTP/3 application developer, **HTTP/3 runs over the QUIC transport protocol** — even though, underneath, QUIC is itself built as a sub-layer running over plain UDP sockets.

---

# Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**UDP port scanning**|Because UDP is connectionless, a scanner can spray packets at all 65535 UDP ports on a target machine to discover which ones are listening (respond or don't respond). Unlike TCP, there's no handshake to complete — you just send and observe what comes back (or what's silently dropped)|Close all unused UDP ports via firewall rules; rate-limit UDP traffic from unknown sources; know exactly which services are supposed to have open UDP ports on your machines|
|**UDP spoofing**|The source address on a UDP packet is filled in by the OS automatically, but an attacker with raw socket access can overwrite it with a fake source IP. This is the foundation of UDP-based amplification DDoS attacks (DNS amplification, NTP amplification) — the attacker forges the victim's IP as the source so the large server response gets sent to the victim|Source address validation (BCP 38/RFC 2827) at ISPs; configure DNS/NTP servers to only respond to trusted IP ranges|
|**TCP SYN flood**|TCP's three-way handshake requires the server to allocate resources (a half-open connection entry) after receiving the first SYN packet. An attacker floods a server with SYN packets from spoofed IPs — the server allocates state for each "connection" but the handshake never completes, exhausting the connection table and blocking legitimate clients|SYN cookies — a technique where the server doesn't allocate state until the third packet of the handshake arrives, making half-open entries effectively free; rate limiting SYNs per IP|
|**Port numbers as attack surface information**|Well-known port numbers reveal exactly what service is running on a machine. A server listening on port 22 is almost certainly running SSH; port 3306 is MySQL. Attackers use this to target known vulnerabilities in specific services|Use non-standard ports for sensitive internal services where possible (security through obscurity — not a complete defense, but raises the bar); hide services behind a firewall or VPN|
|**Unencrypted sockets**|The code shown here sends data in plaintext — any attacker who can intercept traffic on the path between client and server can read the entire conversation trivially (a man-in-the-middle or passive wiretap)|Wrap sockets in TLS using Python's `ssl.wrap_socket()` — this is how HTTPS, SMTPS, and every other "-S" variant of a protocol works. TLS encrypts the byte-stream without changing the socket programming model much|
|**Binding to 0.0.0.0 vs. a specific IP**|`serverSocket.bind(('', serverPort))` binds to all available network interfaces. If the server has both a public IP and a private/admin IP, the service is now reachable on both — potentially exposing an admin service to the public Internet unintentionally|Bind to a specific interface IP instead of `''` when a service is only meant to be reachable from a specific network|
|**Raw sockets and privilege escalation**|Raw sockets (which allow manually crafting packet headers, including the source address) require root/admin privileges in most operating systems. A program that opens a raw socket has essentially full control over outgoing network traffic — a malware capability|Audit which processes on a system have opened raw sockets; run servers as unprivileged users, never as root, so that even if an exploit compromises the process, it can't open raw sockets|
|**Proprietary protocol weakness**|Proprietary application protocols designed by non-security-specialists often skip authentication, have no encryption, and assume "nobody outside the company knows the protocol." But protocols are routinely reverse-engineered — Wireshark captures, fuzzing, and disassembly reveal undocumented protocols quickly|Even proprietary protocols must be designed with authentication and encryption from the start; obscurity is not a security boundary|
|**QUIC connection ID / stream ID confusion**|Since QUIC multiplexes many logical streams over one connection (identified by connection ID + stream ID), a bug or vulnerability in how an application or library validates these IDs could allow one client's stream to be confused with another's, or allow injection into an unrelated stream|Rely on a well-vetted QUIC library's API rather than manually parsing or constructing connection/stream IDs; keep QUIC implementations patched, since this is still a relatively young protocol stack|

---

## Questions I Still Have

- [ ] `bind(('', serverPort))` binds to all interfaces — is there ever a reason for a _client_ to call `bind()` too, rather than letting the OS choose its port automatically?
- [ ] The TCP server here handles one client at a time — `accept()` blocks until the current client disconnects. How do real servers handle thousands of concurrent clients? (Threading, async I/O, event loops — to be explored.)
- [ ] `clientSocket.close()` on the TCP side sends a closing message — but what's the difference between a graceful FIN-based teardown and a RST (reset)? When does a socket produce a RST instead? (§3.5 promises more detail.)
- [ ] Python's `ssl.wrap_socket()` is mentioned as the path to encrypted TCP sockets — how much of the socket code shown here actually changes when you wrap in TLS, and what's the performance overhead?
- [ ] `SOCK_DGRAM` and `SOCK_STREAM` are the two types shown here — what is `SOCK_RAW`, when would it appear, and why does it require elevated OS privileges?
- [ ] Since most operating systems don't natively expose a "QUIC socket" the way they expose TCP/UDP sockets, what does a QUIC API actually look like at the library level in practice — is it always a user-space library (like `aioquic` in Python) sitting on top of a regular UDP socket underneath?
- [ ] The UDP server's `while True:` loop handles one packet at a time, sequentially. Is there a situation where a slow `.upper()` operation (or more realistically, a slow database query) would cause the server to fall badly behind and start dropping UDP packets?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Socket**|The programming interface between an application and the OS's networking stack — a process writes data into its socket to send, and reads from it to receive|
|**AF_INET**|Address family constant specifying IPv4 as the underlying network protocol|
|**SOCK_DGRAM**|Socket type constant specifying UDP (datagram — connectionless, unreliable, addressed per-packet)|
|**SOCK_STREAM**|Socket type constant specifying TCP (stream — connection-oriented, reliable, ordered byte stream)|
|**`.encode()` / `.decode()`**|Python 3 string/bytes conversion methods — sockets send and receive raw bytes, so strings must be `.encode()`d before sending and `.decode()`d after receiving|
|**`bind()`**|Assigns a specific port number to a server socket so the OS knows which process to route incoming packets to|
|**`sendto()`**|UDP-specific send method — requires an explicit destination address (IP + port) on every call|
|**`recvfrom()`**|Receive method that also returns the sender's address — used in UDP where there is no persistent connection to tell you who sent the data|
|**`connect()`**|TCP-specific call that initiates the three-way handshake and establishes a connection before any data flows|
|**`send()`**|TCP-specific send method — no destination address needed, as the connection already encodes the destination|
|**`listen()`**|Switches a TCP server socket into listening mode; takes a parameter setting the maximum number of queued connection requests|
|**`accept()`**|Blocks until a client completes the three-way handshake, then returns a brand-new connection socket dedicated to that client — the key step that creates the two-socket model|
|**Welcoming socket**|The persistent server socket that handles only connection setup (listen + accept); stays open for the lifetime of the server|
|**Connection socket**|The per-client socket created by `accept()` — used for all actual data exchange with that specific client, then closed when done|
|**Three-way handshake**|The TCP connection-establishment procedure, completed before any data flows; handled entirely by TCP/the OS, invisible to the application code|
|**Well-known port numbers**|Standardized port numbers defined by RFCs for specific protocols (e.g., 80 for HTTP, 25 for SMTP); must be used if implementing an RFC-defined protocol, and avoided otherwise|
|**Open protocol**|A protocol whose operation is specified in a public RFC or standards document; any developer who reads the spec can build a compatible implementation|
|**Proprietary protocol**|A protocol defined and controlled by a single developer or company; not publicly specified, so no third party can build compatible software without reverse-engineering|
|**QUIC API**|The interface an application (e.g., an HTTP/3 implementation) uses to create QUIC connections and streams; sits above the underlying UDP socket and is what application code actually talks to|
|**Stream / Stream ID**|An independent, reliably-delivered logical channel within a single QUIC connection, identified by a stream ID — used to carry one HTTP/3 message (e.g., the HTML base page, a CSS file, an image)|
|**QUIC connection ID**|An identifier for a QUIC connection, specified alongside a stream ID whenever an application sends a message through the QUIC API|
|**SYN flood**|A TCP DoS attack that exploits the three-way handshake by sending large numbers of SYN packets from spoofed source IPs, exhausting the server's half-open connection table|
|**SYN cookies**|A defense against SYN floods: the server encodes connection state into the initial sequence number instead of allocating a table entry, so half-open connections consume no server resources|
|**Raw socket**|A special socket type that allows the application to manually craft packet headers including the source address; requires root/admin privileges|

---

## Related Concepts

---

→ Next: [[INTRODUCTION & SERVICES]]