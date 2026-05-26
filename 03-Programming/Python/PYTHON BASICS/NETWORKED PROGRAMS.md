---
title: NETWORKED PROGRAMS
date: 2026-05-26
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# NETWORKED PROGRAMS 

## One-Line Summary

**Networked programs** use Python's `socket` library (low-level) or `urllib` library (high-level) to _talk to web servers over the HTTP protocol_ — fetching text, images, and HTML pages, then parsing them with **regex** or **BeautifulSoup**.

---

## PART 1 — HOW THE WEB WORKS (Conceptual)

### The Problem Without Networking Knowledge

Before you understand sockets and HTTP, web access feels like magic. Once you understand the layers, you realise it's just structured text flying back and forth over a two-way pipe — and Python gives you full control of that pipe.

> Think of a **socket** like a phone call between two programs. Once the call connects, both sides can talk and listen. The rules of the conversation (who goes first, what to say) is called the **protocol**.

### What Is a Socket?

A **socket** is a two-way network connection between two programs. Unlike a file (one-way read), you can **both send and receive** on the same socket. Key behaviour:

- If you try to **read** from a socket and nothing has been sent yet, your program just **waits** (blocks).
- If both sides wait without sending first, you get a deadlock — they'd wait forever.
- This is why protocols exist: they define who speaks first and in what format.

### What Is a Protocol?

A **protocol** is a set of precise rules that determine:

- Who sends first
- What format the message must be in
- What the valid responses are

**HTTP** (Hypertext Transfer Protocol) is the protocol of the web. It lives at: `https://www.w3.org/Protocols/rfc2616/rfc2616.txt` (a 176-page document — you don't need to read it all).

### The HTTP Request Format

To request a document from a web server over HTTP, you connect to port 80 and send a line like:

```
GET http://data.pr4e.org/romeo.txt HTTP/1.0

```

The format is: `GET <url> HTTP/1.0` followed by a **blank line** (two `\r\n` sequences). The blank line signals the end of the request.

The web server then responds with **headers** first, then a **blank line**, then the **actual content**.

---

## PART 2 — SOCKETS: THE LOW-LEVEL WAY

### Importing and Creating a Socket

```python
import socket

mysock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

- `socket.AF_INET` → IPv4 address family
- `socket.SOCK_STREAM` → TCP (reliable, ordered byte stream)

### The World's Simplest Web Browser

```python
import socket

mysock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
mysock.connect(('data.pr4e.org', 80))           # connect to server on port 80

cmd = 'GET http://data.pr4e.org/romeo.txt HTTP/1.0\r\n\r\n'.encode()
mysock.send(cmd)                                  # send the HTTP GET request

while True:
    data = mysock.recv(512)                       # receive up to 512 bytes at a time
    if len(data) < 1:
        break                                     # empty response = server is done
    print(data.decode(), end='')

mysock.close()
```

**Why `.encode()` and `.decode()`?**

- HTTP requires sending **bytes**, not strings.
- `str.encode()` converts a string → bytes object.
- `bytes.decode()` converts bytes → string for printing.
- `b'Hello world'` and `'Hello world'.encode()` are equivalent.

### What the Server Sends Back

The server response looks like this:

```
HTTP/1.1 200 OK
Date: Wed, 11 Apr 2018 18:52:55 GMT
Server: Apache/2.4.7 (Ubuntu)
Last-Modified: Sat, 13 May 2017 11:22:22 GMT
Content-Length: 167
Content-Type: text/plain

But soft what light through yonder window breaks
It is the east and Juliet is the sun
...
```

The structure is:

1. **Status line** (`HTTP/1.1 200 OK`)
2. **Headers** (key: value pairs)
3. **Blank line** (signals end of headers)
4. **Body** (the actual content you asked for)

> **Java comparison:** In Java you'd use `java.net.Socket`, `BufferedReader`, `PrintWriter`. Python's `socket` module is much more concise — no type declarations or stream wrappers.

---

## PART 3 — RETRIEVING AN IMAGE OVER HTTP

Images are binary files. The program accumulates all the bytes in memory, strips the headers, and saves the raw image data to disk.

```python
import socket
import time

HOST = 'data.pr4e.org'
PORT = 80

mysock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
mysock.connect((HOST, PORT))
mysock.sendall(b'GET http://data.pr4e.org/cover3.jpg HTTP/1.0\r\n\r\n')

count = 0
picture = b""                    # accumulate bytes (binary string)

while True:
    data = mysock.recv(5120)     # get up to 5120 bytes at a time
    if len(data) < 1: break
    count = count + len(data)
    picture = picture + data     # concatenate bytes

mysock.close()

# Find where the headers end (blank line = \r\n\r\n)
pos = picture.find(b"\r\n\r\n")
print('Header length', pos)
print(picture[:pos].decode())    # print headers as text

# Skip past the header and save image data only
picture = picture[pos+4:]        # +4 to skip past the \r\n\r\n itself
fhand = open("stuff.jpg", "wb")  # open in WRITE BINARY mode
fhand.write(picture)
fhand.close()
```

### Key Points About `recv()`

- `recv(n)` returns **at most** `n` bytes — but may return fewer if less data is available at that moment (depends on network speed).
- The program loops until `recv()` returns an empty byte string `b''`, which means the server has closed the connection.
- If you **slow down** your recv() calls with `time.sleep(0.25)`, the server can buffer up more data, so you tend to get the full `n` bytes each time (called **flow control** — the sending side pauses when the buffer fills up).

---

## PART 4 — `urllib`: THE HIGH-LEVEL WAY

### Why `urllib` Exists

Doing all this manually with sockets is tedious. Python's `urllib` library handles the entire HTTP protocol for you — connection, headers, encoding — and just gives you the content.

> Think of sockets as driving manually with a gear stick. `urllib` is the automatic transmission — same roads, much less effort.

### Basic `urllib` Usage

```python
import urllib.request

fhand = urllib.request.urlopen('http://data.pr4e.org/romeo.txt')
for line in fhand:
    print(line.decode().strip())   # each line comes as bytes, so decode it
```

`urlopen()` returns a **file-like object** — you can loop over it line by line, just like a local file opened with `open()`. The headers are consumed internally; you only see the body.

### Word Frequency Counter Using `urllib`

```python
import urllib.request, urllib.parse, urllib.error

fhand = urllib.request.urlopen('http://data.pr4e.org/romeo.txt')

counts = dict()
for line in fhand:
    words = line.decode().split()
    for word in words:
        counts[word] = counts.get(word, 0) + 1

print(counts)
```

Once the URL is opened, you can treat it exactly like a local file — `split()`, loops, dictionaries, everything works the same way.

### Reading Binary Files with `urllib`

**Small files (fit in memory):**

```python
import urllib.request, urllib.parse, urllib.error

img = urllib.request.urlopen('http://data.pr4e.org/cover3.jpg').read()
fhand = open('cover3.jpg', 'wb')   # write binary
fhand.write(img)
fhand.close()
```

**Large files (stream in blocks to avoid running out of memory):**

```python
import urllib.request, urllib.parse, urllib.error

img = urllib.request.urlopen('http://data.pr4e.org/cover3.jpg')
fhand = open('cover3.jpg', 'wb')
size = 0

while True:
    info = img.read(100000)     # read 100,000 bytes at a time
    if len(info) < 1: break
    size = size + len(info)
    fhand.write(info)           # write each block to disk immediately

print(size, 'characters copied.')
fhand.close()
```

> ⚠️ **Always stream large files.** If you `.read()` a 4GB video file into memory at once, your program crashes. Use the chunked approach above.

---

## PART 5 — PARSING HTML AND WEB SCRAPING

### What Is Web Scraping?

**Web scraping** = writing a program that pretends to be a web browser, retrieves pages, and extracts data from the HTML.

Google does this at massive scale — it "**spiders**" the web by following links on every page it finds, then uses the number of inbound links to a page as a measure of its importance (PageRank).

### Method 1 — Parsing HTML with Regular Expressions

For simple, well-structured HTML, regex works fine:

```python
# Extract all HTTP/HTTPS links from a web page
import urllib.request, urllib.parse, urllib.error
import re
import ssl

# Ignore SSL certificate errors (for testing only)
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter - ')
html = urllib.request.urlopen(url, context=ctx).read()    # returns bytes

links = re.findall(b'href="(http[s]?://.*?)"', html)     # b'' = bytes pattern
for link in links:
    print(link.decode())
```

Breaking down the regex `href="(http[s]?://.*?)"`:

|Part|Meaning|
|---|---|
|`href="`|Match the literal attribute name|
|`(`|Start capture group (what we want to extract)|
|`http`|Literal "http"|
|`[s]?`|Optionally followed by "s" (for https)|
|`://`|Literal "://"|
|`.*?`|Any characters, **non-greedy** (stops at first `"`)|
|`)"`|End capture group, then closing double-quote|

**Problem with regex on HTML:** Real-world HTML is messy and broken. Regex can miss valid links or grab invalid ones when the HTML isn't perfectly formatted.

### Method 2 — Parsing HTML with BeautifulSoup (The Right Way)

**BeautifulSoup** is a Python library specifically designed to parse even badly-formed HTML gracefully. Install it with:

```bash
pip install beautifulsoup4
```

#### Basic Link Extraction with BeautifulSoup

```python
import urllib.request, urllib.parse, urllib.error
from bs4 import BeautifulSoup
import ssl

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter - ')
html = urllib.request.urlopen(url, context=ctx).read()
soup = BeautifulSoup(html, 'html.parser')

# Get all <a> anchor tags
tags = soup('a')
for tag in tags:
    print(tag.get('href', None))    # extract the href attribute
```

#### Extracting Multiple Parts of a Tag

```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
import ssl

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter - ')
html = urlopen(url, context=ctx).read()
soup = BeautifulSoup(html, "html.parser")

tags = soup('a')
for tag in tags:
    print('TAG:', tag)
    print('URL:', tag.get('href', None))
    print('Contents:', tag.contents[0])   # text between the tags
    print('Attrs:', tag.attrs)            # dictionary of all attributes
```

**Sample output:**

```
TAG: <a href="http://www.dr-chuck.com/page2.htm">Second Page</a>
URL: http://www.dr-chuck.com/page2.htm
Contents: Second Page
Attrs: [('href', 'http://www.dr-chuck.com/page2.htm')]
```

### Regex vs BeautifulSoup — When to Use Which

|Scenario|Use|
|---|---|
|Well-formed, predictable HTML|Regex (simpler)|
|Real-world messy HTML|BeautifulSoup (robust)|
|Need tag attributes, contents, structure|BeautifulSoup|
|Quick one-liner extraction|Regex|

> 💡 **Rule of thumb:** If regex gives you bad data on even 1% of pages, switch to BeautifulSoup.

---

## PART 6 — SSL AND HTTPS

Many websites use **HTTPS** (HTTP + TLS encryption). Python's `ssl` module handles this, but in test/learning environments you often want to disable certificate verification:

```python
import ssl

# Create a context that ignores certificate errors
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

# Pass it to urlopen
html = urllib.request.urlopen(url, context=ctx).read()
```

> ⚠️ **Never disable SSL verification in production code.** Only use this for learning/testing. Real code should use the default SSL context which verifies certificates.

---

## PART 7 — UNIX / LINUX BONUS: `curl` AND `wget`

Linux/Mac users have command-line tools that do the same thing as `urllib`:

```bash
# curl = "copy URL" — downloads a file
curl -O http://www.py4e.com/cover.jpg

# wget — similar functionality
wget http://www.py4e.com/cover.jpg
```

The Python files `curl1.py` and `curl2.py` in the py4e codebase implement equivalent functionality to these commands — they're named intentionally.

---

## PART 8 — COMPLETE CODE PATTERNS (Quick Reference)

### Pattern 1 — Raw Socket HTTP Request

```python
import socket

mysock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
mysock.connect(('hostname.com', 80))
mysock.send('GET /path HTTP/1.0\r\n\r\n'.encode())

while True:
    data = mysock.recv(512)
    if len(data) < 1: break
    print(data.decode(), end='')

mysock.close()
```

### Pattern 2 — Simple urllib Text Fetch

```python
import urllib.request

fhand = urllib.request.urlopen('http://url.com/file.txt')
for line in fhand:
    print(line.decode().strip())
```

### Pattern 3 — Stream Binary File to Disk

```python
import urllib.request

url = 'http://url.com/image.jpg'
img = urllib.request.urlopen(url)
fhand = open('local_copy.jpg', 'wb')
size = 0
while True:
    chunk = img.read(100000)
    if len(chunk) < 1: break
    size += len(chunk)
    fhand.write(chunk)
fhand.close()
print(size, 'bytes copied.')
```

### Pattern 4 — Scrape Links with BeautifulSoup

```python
import urllib.request
from bs4 import BeautifulSoup
import ssl

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter URL: ')
html = urllib.request.urlopen(url, context=ctx).read()
soup = BeautifulSoup(html, 'html.parser')

for tag in soup('a'):
    link = tag.get('href', None)
    if link: print(link)
```

### Pattern 5 — Scrape Links with Regex

```python
import urllib.request, re, ssl

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter URL: ')
html = urllib.request.urlopen(url, context=ctx).read()
links = re.findall(b'href="(http[s]?://.*?)"', html)
for link in links:
    print(link.decode())
```

---

## PART 9 — ENCODE / DECODE REFERENCE

HTTP operates at the byte level, not the string level. This is one of Python 3's big changes from Python 2.

|Operation|Code|Use case|
|---|---|---|
|String → bytes|`'hello'.encode()`|Sending over socket|
|Bytes with `b''`|`b'GET /...'`|Same as `.encode()`|
|Bytes → string|`data.decode()`|Printing received data|
|Bytes → string, strip whitespace|`line.decode().strip()`|Processing urllib lines|

```python
# These are identical:
b'Hello world'
'Hello world'.encode()

# Default encoding is UTF-8:
'café'.encode()          # b'caf\xc3\xa9'
b'caf\xc3\xa9'.decode()  # 'café'
```

---

## PART 10 — FLOW CONTROL (Important Concept)

When you call `recv(n)`, Python doesn't always give you `n` bytes — it gives you however many have arrived so far. This is because there's a **buffer** between the server's `send()` and your `recv()`.

- **Without delay:** Server can send faster than you consume. Buffer fills up, server pauses. You might get inconsistent chunk sizes (3200, 5120, 3167, etc.).
- **With `time.sleep(0.25)` delay:** Server fills the buffer while you wait. When you call `recv(5120)` you reliably get 5120 bytes each time (until the last chunk).
- **Pausing either side due to buffer** = **flow control** — a built-in TCP mechanism.

---

## PART 11 — GLOSSARY

|Term|Definition|
|---|---|
|**socket**|A two-way network connection between two applications; can both send and receive|
|**port**|A number that identifies which application on a server you're contacting; web = 80, email = 25|
|**protocol**|A set of rules governing the format and order of messages between two programs|
|**HTTP**|Hypertext Transfer Protocol — the protocol of the web|
|**GET request**|The HTTP command to request a document from a server|
|**headers**|Key-value metadata the server sends before the actual content (Content-Type, Content-Length, etc.)|
|**bytes object**|Python type `bytes`/`b''` — raw binary data; required for network I/O|
|**encode()**|Converts a Python string to bytes (default UTF-8)|
|**decode()**|Converts bytes back to a Python string|
|**urllib**|Python's built-in library for making HTTP requests at a higher level than sockets|
|**scrape**|A program that pretends to be a browser, fetches pages, and extracts data from HTML|
|**spider**|A web scraper that follows links from page to page (like Google's crawler)|
|**BeautifulSoup**|A Python library that parses HTML (even broken HTML) and lets you extract data easily|
|**flow control**|TCP mechanism where the sender pauses when the receiver's buffer is full|
|**SSL/TLS**|Encryption layer on top of HTTP (makes it HTTPS); Python's `ssl` module handles this|
|**curl**|Unix command-line tool for fetching URLs (`curl -O url`)|
|**wget**|Another Unix tool similar to curl|

---

## PART 12 — JAVA vs PYTHON COMPARISON

|Concept|Java|Python|
|---|---|---|
|Create socket|`new Socket("host", 80)`|`socket.socket(socket.AF_INET, socket.SOCK_STREAM)`|
|Connect|`socket.connect(...)`|`mysock.connect(('host', 80))`|
|Send data|`out.println("GET ...")`|`mysock.send('GET ...'.encode())`|
|Receive data|`in.read(buf)`|`mysock.recv(512)`|
|HTTP request (high-level)|`HttpURLConnection`|`urllib.request.urlopen(url)`|
|Parse HTML|Jsoup library|BeautifulSoup (bs4)|
|Bytes vs strings|`byte[]` vs `String`|`bytes` vs `str`; use `.encode()`/`.decode()`|
|SSL context|`SSLContext`, `TrustManager`|`ssl.create_default_context()`|

---

## PART 13 — COMMON MISTAKES AND DEBUGGING

**1. Sending a string instead of bytes to a socket:**

```python
mysock.send('GET /file HTTP/1.0\r\n\r\n')     # ❌ TypeError
mysock.send('GET /file HTTP/1.0\r\n\r\n'.encode())  # ✅
```

**2. Forgetting the blank line at the end of the HTTP request:**

```python
cmd = 'GET http://data.pr4e.org/romeo.txt HTTP/1.0\r\n'   # ❌ server waits forever
cmd = 'GET http://data.pr4e.org/romeo.txt HTTP/1.0\r\n\r\n'  # ✅ blank line signals end
```

**3. Printing bytes without decoding:**

```python
for line in urllib.request.urlopen(url):
    print(line)          # ❌ prints: b'But soft what light...\n'
    print(line.decode()) # ✅ prints: But soft what light...
```

**4. Opening a binary file in text mode:**

```python
fhand = open('image.jpg', 'w')   # ❌ text mode — corrupts binary data
fhand = open('image.jpg', 'wb')  # ✅ binary write mode
```

**5. Reading the entire response into memory for huge files:**

```python
img = urllib.request.urlopen(url).read()  # ❌ crashes on 4GB video
# ✅ use the chunked while-loop pattern from Part 8
```

**6. Not handling SSL errors on HTTPS sites:**

```python
html = urllib.request.urlopen('https://site.com').read()  # ❌ may fail with SSL error
# ✅ pass ssl context — see Part 6
```

---

## PART 14 — EXERCISES

**Basic Socket:**

- [ ] Modify `socket1.py` to prompt the user for the URL instead of hardcoding it. Use `split('/')` to extract the hostname.
- [ ] Modify the socket program to count received characters and stop displaying after 3000. Still retrieve the full document and print total character count at the end.

**urllib:**

- [ ] Use `urllib` to replicate exercise 2 above — retrieve a URL, display first 3000 characters, and count the total.
- [ ] Modify `urllinks.py` to extract and **count** the number of `<p>` (paragraph) tags in an HTML page. Print just the count.

**Advanced:**

- [ ] Modify the socket program to only display content **after** the blank line (headers stripped). Remember `recv()` gives you raw bytes including newlines.
- [ ] Write a program that reads a web page and counts how many times each word appears (like the word-frequency exercise but sourced from a URL).

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[REGEX]]
- [[FILE HANDLING]]
- [[STRINGS]]
- [[DICTIONARIES, TUPLES & SETS]]
- [[OBJECT-ORIENTED PROGRAMMING (OOP)]]