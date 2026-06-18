---
title: SOCKET PROGRAMMING - UDP & TCP
date: 2026-06-18
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# 2.7 Socket Programming — Creating Network Applications

> **One-Line Summary:** A socket is the programming interface between an application and the network — every networked program you have ever written or used ultimately reads from and writes to a socket. This section shows how to actually build both ends of a client-server application, first in UDP and then in TCP, using Python.

---

## Core Idea: What Is a Socket?

Every network application, at the code level, is two programs: one running on the client machine, one running on the server machine. These two processes communicate by sending bytes into, and reading bytes out of, **sockets**. A socket is the door between your application code and the operating system's networking stack — your code lives on the inside, the Internet lives on the outside, and the socket is the only opening between them.

> **Analogy — House and Door:** Think of a process as a house and its socket as the front door. Your application code controls everything inside the house — what data to send, how to process what arrives. But once data leaves through the door, it's in the transport layer's hands, and you have almost no control over how it actually travels across the network to its destination.

When a network application runs, the operating system creates a pair of socket objects — one in each process — and data flows between them. The developer's job is to write the logic on both sides: what the client sends, how the server processes it, and what it sends back.

---

## Two Types of Network Applications

Before writing a single line of code, there's a design decision that's already been made for you by context: is your application **open** (defined by a public RFC standard) or **proprietary**?

|Type|What it means|Who can interoperate|Example|
|---|---|---|---|
|**Open / RFC-defined**|The protocol is publicly specified in an RFC or similar standard document; anyone can read the spec and write a conforming implementation|Any independent developer who follows the spec|A Firefox browser talking to an Apache server — two programs written by different companies, working together because both follow HTTP as defined in RFC 7230|
|**Proprietary**|The client and server programs are written by the same developer (or team) who defines the protocol themselves; it's never published|Only that developer's own code — no third party can build a compatible client or server without reverse-engineering it|A proprietary IoT sensor talking to its vendor's own cloud backend|

The practical consequence for development: if your application is RFC-defined, you must conform to the RFC's port numbers and message formats. If it's proprietary, you have total freedom over the protocol design, but you're also solely responsible for building both ends.

---

## First Decision: UDP or TCP?

The single most important choice when designing a networked application is whether to use UDP or TCP as the underlying transport. The difference is deep:

|Property|UDP|TCP|
|---|---|---|
|**Connection**|Connectionless — each packet sent independently, no setup required|Connection-oriented — a three-way handshake must complete before any data flows|
|**Reliability**|None — the sending process explicitly attaches a destination address to every packet; no guarantee the packet arrives, arrives once, or arrives in order|Reliable byte-stream — TCP guarantees every byte arrives, arrives exactly once, and arrives in order|
|**Overhead**|Minimal — fire and forget|Higher — connection setup, acknowledgements, congestion control all add overhead|
|**Use case**|Speed matters more than guaranteed delivery — streaming, gaming, DNS|Correctness matters — web browsing, email, file transfer|
|**Port number handling**|Developer explicitly attaches destination IP + port to every packet sent; source port is attached automatically by the OS|Port numbers are baked into the connection at setup time; after that, just write bytes into the pipe|

There's also a rule for port numbers: if your application implements a publicly defined RFC protocol, you must use the well-known port number that RFC specifies (e.g., HTTP is 80, SMTP is 25). If you're writing a proprietary application, choose a port number that doesn't collide with any of those reserved ones.

---

# 2.7.1 Socket Programming with UDP

## How UDP Sockets Work

The flow for a single UDP exchange is:

```
Client                                      Server
------                                      ------
1. Create a socket                          1. Create a socket
                                            2. Bind socket to a known port number
                                            3. Wait for a packet to arrive

2. Attach destination (IP + port) to msg
3. Send packet via socket ──────────────► 4. Read packet, note the client's
                                               source address (IP + port)
                                            5. Process data
                                            6. Attach client's address to reply
4. Wait for reply                          7. Send reply ◄────────────────────

5. Read reply from socket
6. Close socket
```

Key point about addressing: the client has to explicitly state where it's sending each packet (server IP + server port). The server learns where to send its reply by reading the source address automatically embedded in the incoming packet by the OS — the server never needs to know the client's address in advance.

![[Pasted image 20260618232633.png]] _(Figure 2.28 — The client-server application using UDP: the client creates a socket, attaches a destination address, and sends; the server reads the incoming segment, processes it, writes the reply back to the client's address, and stays in its loop waiting for the next packet)_

---

## UDPClient.py — Line by Line

```python
from socket import *
serverName = 'hostname'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = raw_input('Input lowercase sentence:')
clientSocket.sendto(message, (serverName, serverPort))
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print modifiedMessage
clientSocket.close()
```

**`from socket import *`** Imports Python's socket module — everything needed to create and use sockets lives here.

**`serverName = 'hostname'` and `serverPort = 12000`** `serverName` can be either a raw IP string like `'128.138.32.126'` or a hostname like `'cis.poly.edu'`. If a hostname is given, Python automatically performs a DNS lookup to resolve it to an IP before sending. Port 12000 is an arbitrary choice here — any unused non-reserved port works.

**`clientSocket = socket(AF_INET, SOCK_DGRAM)`** This is where the socket is actually created. `AF_INET` means the underlying network uses IPv4 (address family IPv4). `SOCK_DGRAM` means it's a UDP socket (datagram socket). Notice that no port number is specified here — the OS automatically assigns an available port to this client socket, which becomes the source port number embedded in every outgoing packet.

**`message = raw_input('Input lowercase sentence:')`** Prompts the user to type a line at the terminal. Whatever they type is stored in `message`.

**`clientSocket.sendto(message, (serverName, serverPort))`** `sendto()` does two things at once: it attaches the destination address `(serverName, serverPort)` to the message, then pushes the whole thing through `clientSocket` and out to the network. The OS automatically appends the client's source IP and port in the packet header — the code doesn't need to do that manually.

**`modifiedMessage, serverAddress = clientSocket.recvfrom(2048)`** Blocks here, waiting for a packet to arrive. When one does, the packet's payload goes into `modifiedMessage` and the sender's address (IP + port) goes into `serverAddress`. The `2048` is the buffer size in bytes — more than large enough for a text sentence.

**`print modifiedMessage` and `clientSocket.close()`** Prints the capitalized reply and closes the socket, terminating the process.

---

## UDPServer.py — Line by Line

```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))
print "The server is ready to receive"
while 1:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.upper()
    serverSocket.sendto(modifiedMessage, clientAddress)
```

**`serverSocket = socket(AF_INET, SOCK_DGRAM)`** Same socket creation as the client — IPv4, UDP. The critical difference comes in the next line.

**`serverSocket.bind(('', serverPort))`** This is what makes a server different from a client at the socket level. `bind()` permanently attaches port 12000 to this socket. After this line, any packet arriving at this machine's port 12000 gets routed by the OS directly to `serverSocket`. Without bind, the OS wouldn't know which process should receive packets on that port.

**`while 1:`** The server runs forever — it has no reason to stop. It loops indefinitely, handling one client request per iteration.

**`message, clientAddress = serverSocket.recvfrom(2048)`** Blocks until a packet arrives. The packet's data goes into `message`; the sender's IP and port go into `clientAddress`. This address is crucial — it's the server's only way to know where to send the reply, since UDP is connectionless and the server never established any prior link with the client.

**`modifiedMessage = message.upper()`** The whole "business logic" of this server: capitalize the message. In a real server, this would be where meaningful processing happens.

**`serverSocket.sendto(modifiedMessage, clientAddress)`** Sends the capitalized message back to exactly the address the request came from. Then the loop immediately goes back to `recvfrom()`, waiting for the next client — could be the same client, could be a completely different one.

> **Analogy — A Walkie-Talkie Dispatcher:** UDP socket communication is like walkie-talkie radio. Each transmission is independent — the sender announces the recipient's channel (destination address), sends the message, and waits to hear back. Nobody "connects" to anyone; every transmission stands alone. The dispatcher (server) just sits on their channel all day, writing down who called in (clientAddress) so they can radio back.

---

# 2.7.2 Socket Programming with TCP

## How TCP Sockets Work — and Why There Are Two Server Sockets

TCP fundamentally changes the model: before any data can flow, a connection must be established. This requires a **three-way handshake** between the client and server before either side can send actual application data. Only after the handshake completes does a reliable byte-stream pipe exist between the two processes.

This creates a distinction that confuses newcomers: a TCP server actually uses **two different sockets**, not one.

|Socket|Name in code|Purpose|
|---|---|---|
|**Welcoming socket**|`serverSocket`|Sits open permanently, waiting for new clients to knock. It handles _only_ connection setup — it never carries actual data.|
|**Connection socket**|`connectionSocket`|Created fresh for each specific client that successfully connects. All actual data exchange (request, reply) flows through this one. Closed when the conversation with that client ends.|

> **Analogy — A Hotel Reception Desk:** The welcoming socket is the hotel's main front door and reception desk — always open, designed to greet anyone who arrives and check them in. The connection socket is the specific room key issued to that guest: it's a dedicated, private channel just for them. The front desk (welcoming socket) stays open for the next guest, while the guest uses their room (connection socket) to do the actual stay. You'd never use the front desk to order room service — that's what the in-room phone (connection socket) is for.

![[Pasted image 20260618232804.png]] _(Figure 2.29 — The TCPServer process has two sockets: a welcoming socket that handles the three-way handshake, and a connection socket created fresh for each client that connects, through which all actual data flows in both directions)_

The full TCP flow:

```
Client                                         Server
------                                         ------
                                               1. Create welcoming socket (serverSocket)
                                               2. Bind to port number
                                               3. Listen for incoming connections
                                               4. Block on accept() — wait for a knock

1. Create socket (clientSocket)
2. Connect to server (triggers 3-way handshake) ──►
                                               5. accept() returns, creating a new
                                                  connectionSocket dedicated to this client
                                               ◄── TCP connection is now established

3. Send data via clientSocket ─────────────────────────────────────────► 
                                               6. Read from connectionSocket
                                               7. Process (capitalize)
                                               8. Send reply ◄──────────────────────────

4. Read reply from clientSocket
5. Close clientSocket (sends TCP FIN) ─────────────────────────────────►
                                               9. Close connectionSocket
                                               10. Loop back to accept() for next client
```

![[Pasted image 20260618232939.png]] _(Figure 2.30 — The client-server application using TCP: unlike UDP, the client must call connect() before any data flows, the server must call listen() and accept(), and data is exchanged through connectionSocket — not serverSocket)_

---

## TCPClient.py — Line by Line

```python
from socket import *
serverName = 'servername'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName, serverPort))
sentence = raw_input('Input lowercase sentence:')
clientSocket.send(sentence)
modifiedSentence = clientSocket.recv(1024)
print 'From Server:', modifiedSentence
clientSocket.close()
```

**`clientSocket = socket(AF_INET, SOCK_STREAM)`** `SOCK_STREAM` instead of `SOCK_DGRAM` — this is a TCP socket. Everything else about socket creation is the same.

**`clientSocket.connect((serverName, serverPort))`** This is the line that doesn't exist in the UDP version at all. Calling `connect()` triggers the three-way TCP handshake between this client and the server. By the time this line returns, the connection is fully established and ready to carry data. The parameters are the server's address and port — same information UDP would have attached to each individual packet, but here it's set once for the whole connection.

**`clientSocket.send(sentence)`** No destination address attached — unlike `sendto()` in UDP, `send()` just drops bytes into the TCP connection. TCP itself is responsible for getting them to the other end reliably. The application code doesn't think about packets, addresses, or acknowledgements at all.

**`modifiedSentence = clientSocket.recv(1024)`** Reads bytes arriving from the server. Characters accumulate in `modifiedSentence` until a carriage return is received.

**`clientSocket.close()`** Closes the socket and terminates the TCP connection. Under the hood, this causes TCP in the client to send a FIN message to TCP in the server — a proper connection teardown, not just silently dropping the link.

---

## TCPServer.py — Line by Line

```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort))
serverSocket.listen(1)
print 'The server is ready to receive'
while 1:
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024)
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence)
    connectionSocket.close()
```

**`serverSocket = socket(AF_INET, SOCK_STREAM)` and `serverSocket.bind(('', serverPort))`** Creates a TCP socket and binds it to port 12000 — same as the UDP server setup, just with `SOCK_STREAM` instead of `SOCK_DGRAM`.

**`serverSocket.listen(1)`** Switches the socket into listening mode — it's now the welcoming socket, ready to accept connection requests. The parameter `1` specifies the maximum number of connection requests that can be queued while the server is busy handling an existing one.

**`connectionSocket, addr = serverSocket.accept()`** This is the most important line in the server. The program blocks here until a client calls `connect()`. When the TCP handshake completes, `accept()` returns **a brand-new socket object** — `connectionSocket` — dedicated entirely to this one client. `serverSocket` remains open and will accept the next client's connection request on the next loop iteration. This is where the two-socket model becomes concrete in the code.

**`sentence = connectionSocket.recv(1024)`** Reads the client's message from the _connection_ socket, not the welcoming socket. This is where all data exchange happens.

**`capitalizedSentence = sentence.upper()` and `connectionSocket.send(capitalizedSentence)`** Capitalizes the message and sends it back through the same connection socket.

**`connectionSocket.close()`** Closes the connection socket for this client — their conversation is done. `serverSocket` is intentionally not closed here; it stays open so the `while 1:` loop can call `accept()` again and serve the next client.

---

## UDP vs TCP Socket Programming — Side-by-Side Comparison

|Step|UDP|TCP|
|---|---|---|
|**Socket type constant**|`SOCK_DGRAM`|`SOCK_STREAM`|
|**Client: before sending**|Nothing — just create socket and go|Must call `.connect()` to establish a connection first|
|**Client: sending data**|`clientSocket.sendto(msg, (serverName, serverPort))` — destination attached per packet|`clientSocket.send(msg)` — destination already known from connection setup|
|**Server: extra setup step**|`bind()` only|`bind()` + `listen()` + `accept()` (which creates a new connectionSocket)|
|**Server: two sockets?**|No — one socket handles both connection and data|Yes — `serverSocket` (welcoming) and `connectionSocket` (per-client data)|
|**Data delivery guarantee**|None — may be lost, duplicated, or reordered|Guaranteed in-order, exactly once|
|**Addressing per message**|Explicit — source and dest addresses attached to every packet|Implicit — baked into the connection; the byte-stream pipe handles it|
|**Close behavior**|`clientSocket.close()` just frees the local resource|`clientSocket.close()` sends a TCP FIN to the other side — a network event|

---

## The Demo Application in Both Versions

Both the UDP and TCP programs implement the same trivial application:

```
1. Client reads a line of text from the keyboard
2. Client sends it to the server
3. Server receives it and converts it to uppercase
4. Server sends the capitalized version back
5. Client receives and prints the result
```

This is minimal by design — the goal is to expose the socket mechanics as clearly as possible, not to build something useful. The interesting parts are the socket setup, not the `.upper()` call.

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

---

## Questions I Still Have

- [ ] `bind(('', serverPort))` binds to all interfaces — is there ever a reason for a _client_ to call bind() too, rather than letting the OS choose its port automatically?
- [ ] The TCP server here handles one client at a time — `accept()` blocks until the current client disconnects. How do real servers handle thousands of concurrent clients? (Threading, async I/O, event loops — to be explored.)
- [ ] `clientSocket.close()` on the TCP side sends a FIN — but what's the difference between a graceful FIN-based teardown and a RST (reset)? When does a socket produce a RST instead?
- [ ] Python's `ssl.wrap_socket()` is mentioned as the path to encrypted sockets — how much of the socket code shown here actually changes when you wrap in TLS, and what's the performance overhead?
- [ ] `SOCK_DGRAM` and `SOCK_STREAM` are the two types shown here — what is `SOCK_RAW`, when would it appear, and why does it require elevated OS privileges?
- [ ] The UDP server's `while 1:` loop handles one packet at a time, sequentially. Is there a situation where a slow `.upper()` operation (or more realistically, a slow database query) would cause the server to fall badly behind and start dropping UDP packets?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Socket**|The programming interface between an application and the OS's networking stack — a process writes data into its socket to send, and reads from it to receive|
|**AF_INET**|Address family constant specifying IPv4 as the underlying network protocol|
|**SOCK_DGRAM**|Socket type constant specifying UDP (datagram — connectionless, unreliable, addressed per-packet)|
|**SOCK_STREAM**|Socket type constant specifying TCP (stream — connection-oriented, reliable, ordered byte stream)|
|**`bind()`**|Assigns a specific port number to a server socket so the OS knows which process to route incoming packets to|
|**`sendto()`**|UDP-specific send method — requires an explicit destination address (IP + port) on every call|
|**`recvfrom()`**|Receive method that also returns the sender's address — used in UDP where there is no persistent connection to tell you who sent the data|
|**`connect()`**|TCP-specific call that initiates the three-way handshake and establishes a connection before any data flows|
|**`send()`**|TCP-specific send method — no destination address needed, as the connection already encodes the destination|
|**`listen()`**|Switches a TCP server socket into listening mode; takes a backlog parameter setting the queue size for incoming connections|
|**`accept()`**|Blocks until a client completes the three-way handshake, then returns a brand-new connection socket dedicated to that client — the key step that creates the two-socket model|
|**Welcoming socket**|The persistent server socket that handles only connection setup (listen + accept); stays open for the lifetime of the server|
|**Connection socket**|The per-client socket created by `accept()` — used for all actual data exchange with that specific client, then closed when done|
|**Three-way handshake**|The TCP connection-establishment procedure: SYN → SYN-ACK → ACK, completed before any data flows; handled entirely by TCP/the OS, invisible to the application code|
|**Well-known port numbers**|Standardized port numbers defined by RFCs for specific protocols (e.g., 80 for HTTP, 25 for SMTP, 53 for DNS); must be used if implementing an RFC-defined protocol|
|**Open protocol**|A protocol defined in a public RFC or standard; any developer who reads the spec can build a compatible implementation|
|**Proprietary protocol**|A protocol defined and controlled by a single developer or company; not publicly specified, so no third party can build compatible software without reverse-engineering|
|**SYN flood**|A TCP DoS attack that exploits the three-way handshake by sending large numbers of SYN packets from spoofed source IPs, exhausting the server's half-open connection table|
|**SYN cookies**|A defense against SYN floods: the server encodes connection state into the initial sequence number instead of allocating a table entry, so half-open connections consume no server resources|
|**Raw socket**|A special socket type that allows the application to manually craft packet headers including the source address; requires root/admin privileges|

---

## Related Concepts

- 
---

→ Next: [[3.1 - Introduction and Transport-Layer Services]]