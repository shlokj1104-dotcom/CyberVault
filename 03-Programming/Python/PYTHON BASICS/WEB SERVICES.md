---
title: WEB SERVICES
date: 2026-05-26
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# USING WEB SERVICES

## One-Line Summary

**Web services** allow programs to exchange structured data over HTTP using two common formats — **XML** (tree-structured, verbose, self-describing) and **JSON** (compact, maps directly to Python dicts/lists) — which are consumed via **APIs** and parsed using Python's built-in `xml.etree.ElementTree` and `json` libraries.

---

## PART 1 — WHY WEB SERVICES EXIST (Conceptual)

### The Problem Without Web Services

After programs could retrieve HTML over HTTP, the next evolution was producing documents **designed to be consumed by other programs** — not displayed in a browser. HTML is for humans; web service data is for machines.

> Think of a web service like a restaurant's kitchen hatch. The browser (dining room) and the API (kitchen) have a defined format for orders and responses — regardless of how the kitchen is internally organised.

### The Two Dominant Formats

There are two common formats used when exchanging data across the web:

- **XML** (eXtensible Markup Language) — has been in use for a long time; best suited for exchanging **document-style** data. Verbose but self-describing.
- **JSON** (JavaScript Object Notation) — best when programs want to exchange **dictionaries, lists, or internal information** with each other. Simpler and maps directly to native Python structures.

Both formats travel over HTTP — JSON is quickly becoming the format of choice for nearly all data exchange between applications due to its relative simplicity compared to XML.

---

## PART 2 — XML: eXtensible Markup Language

### What XML Looks Like

XML looks very similar to HTML, but is **more structured**. Here is a sample XML document:

```xml
<person>
  <name>Chuck</name>
  <phone type="intl">
    +1 734 303 4456
  </phone>
  <email hide="yes" />
</person>
```

### Key XML Concepts

- Each pair of opening (`<person>`) and closing (`</person>`) tags represents an **element** or **node**, named after the tag.
- Elements can have **text content**, **attributes** (e.g., `hide`), and **nested child elements**.
- If an element is **empty** (no content), it may use a **self-closing tag**: `<email hide="yes" />`.
- An XML document is best thought of as a **tree structure** — there is a top element (here: `person`), and other tags are drawn as _children_ of their _parent_ elements.

### XML as a Tree

```
         person
        /   |   \
     name  phone  email
            |
          +1 734 303 4456
       (attribute: type=intl)   (attribute: hide=yes)
```

_(See Figure 13.1 in the textbook for the visual tree diagram)_

---

## PART 3 — PARSING XML WITH ElementTree

### Basic XML Parsing

Python's built-in `xml.etree.ElementTree` library parses XML into a navigable tree. Using an XML parser has the advantage that even when the XML has quirks, `ElementTree` lets you extract data without worrying about the rules of XML syntax.

```python
import xml.etree.ElementTree as ET

data = '''
<person>
  <name>Chuck</name>
  <phone type="intl">
    +1 734 303 4456
  </phone>
  <email hide="yes" />
</person>'''

tree = ET.fromstring(data)
print('Name:', tree.find('name').text)      # Chuck
print('Attr:', tree.find('email').get('hide'))  # yes
```

**How it works:**

- `ET.fromstring(data)` converts the string into a **tree** of XML elements.
- `.find('tag')` searches the tree and retrieves the **first element** matching the specified tag.
- `.text` returns the text content between the opening and closing tags.
- `.get('attr')` retrieves the value of an attribute (like `hide`).

Triple single quotes (`'''`) — as well as triple double quotes (`"""`) — allow creation of strings that span multiple lines.

### Looping Through Multiple Nodes

Often XML has **multiple nodes of the same type** and you need to loop through all of them:

```python
import xml.etree.ElementTree as ET

input = '''
<stuff>
  <users>
    <user x="2">
      <id>001</id>
      <name>Chuck</name>
    </user>
    <user x="7">
      <id>009</id>
      <name>Brent</name>
    </user>
  </users>
</stuff>'''

stuff = ET.fromstring(input)
lst = stuff.findall('users/user')
print('User count:', len(lst))          # User count: 2

for item in lst:
    print('Name', item.find('name').text)
    print('Id', item.find('id').text)
    print('Attribute', item.get('x'))
```

**Output:**

```
User count: 2
Name Chuck
Id 001
Attribute 2
Name Brent
Id 009
Attribute 7
```

### The `findall` Path Rule — Critical Gotcha

> ⚠️ You **must include all parent-level elements** in the `findall` path, except for the top-level element.

```python
lst  = stuff.findall('users/user')   # ✅  finds both user nodes (nested under 'users')
lst2 = stuff.findall('user')         # ❌  returns 0 — 'user' is not a direct child of 'stuff'
```

`lst` stores all `user` elements nested within their `users` parent. `lst2` looks for `user` elements that are direct children of `stuff` — there are none, so it returns 0.

### ElementTree Method Summary

|Method|What it does|
|---|---|
|`ET.fromstring(data)`|Parse XML string → tree|
|`tree.find('tag')`|First matching child element|
|`tree.findall('path/tag')`|List of all matching elements|
|`element.text`|Text content between tags|
|`element.get('attr')`|Value of an attribute|

---

## PART 4 — JSON: JavaScript Object Notation

### What JSON Looks Like

The JSON format was inspired by the object and array format used in JavaScript. Since Python was invented before JavaScript, Python's syntax for dictionaries and lists _influenced_ the syntax of JSON — so JSON is nearly identical to a combination of Python lists and dictionaries.

Here is a JSON encoding roughly equivalent to the XML person example above:

```json
{
  "name" : "Chuck",
  "phone" : {
    "type" : "intl",
    "number" : "+1 734 303 4456"
  },
  "email" : {
    "hide" : "yes"
  }
}
```

### XML vs JSON — Key Differences

- In XML you can add **attributes** like `type="intl"` directly to a tag. In JSON, you simply use **key-value pairs** — no concept of attributes.
- The XML `<person>` wrapper tag disappears in JSON, replaced by outer curly braces `{}`.
- JSON structures are **simpler** than XML because JSON has fewer capabilities.
- But JSON maps _directly_ to combinations of dictionaries and lists — since nearly all programming languages have equivalents, JSON is very natural for programs to exchange data.
- XML is **more self-describing** than JSON — the tags tell you what the data means. JSON is more succinct (an advantage) but also less self-describing (a disadvantage).

> 💡 **Industry trend:** The field is moving away from XML towards JSON for web services because JSON parsing code is simpler and more direct. However, XML retains an advantage for document-style data — most word processors store documents internally using XML rather than JSON.

---

## PART 5 — PARSING JSON

### Basic JSON Parsing

JSON is parsed using Python's built-in `json` library. `json.loads()` converts a JSON string into native Python structures (lists and dicts) — after which you use standard Python syntax to access the data, no special library needed.

```python
import json

data = '''
[
  { "id" : "001",
    "x" : "2",
    "name" : "Chuck"
  } ,
  { "id" : "009",
    "x" : "7",
    "name" : "Brent"
  }
]'''

info = json.loads(data)
print('User count:', len(info))       # User count: 2

for item in info:
    print('Name', item['name'])
    print('Id', item['id'])
    print('Attribute', item['x'])
```

**Output:**

```
User count: 2
Name Chuck
Id 001
Attribute 2
Name Brent
Id 009
Attribute 7
```

### How JSON Parsing Works

- `json.loads()` returns a **Python list** (because the JSON starts with `[`).
- Each item in the list is a **Python dictionary** (because each JSON object is `{...}`).
- You traverse the list with a `for` loop and use the **Python index operator `[]`** to extract values.
- There is no need to use the `json` library after parsing — the returned data is **simply native Python structures**.

The output is identical to the XML version above — the choice of format is about convenience of encoding, not the data itself.

---

## PART 6 — XML vs JSON COMPARISON

|Feature|XML|JSON|
|---|---|---|
|Verbosity|More verbose|More compact|
|Attributes|Supported natively|No attributes — use key-value pairs|
|Self-describing|Yes — tags name the data|Less so|
|Maps to Python|Requires ElementTree parsing|Directly maps to dicts/lists|
|Best for|Document-style data|Dictionaries, lists, API responses|
|Trend|Legacy/document use|Preferred for new web services|
|Parser|`xml.etree.ElementTree`|`json` (built-in)|

---

## PART 7 — APPLICATION PROGRAMMING INTERFACES (APIs)

### What Is an API?

With the ability to exchange HTTP data in XML or JSON format, the next step is to define and document **"contracts"** between applications. These application-to-application contracts are called **Application Program Interfaces (APIs)**.

When we use an API, generally one program makes a set of _services_ available for use by other applications, and publishes the rules (i.e., the "rules") that must be followed to access those services.

### Service-Oriented Architecture (SOA)

When programs are built where their functionality includes **accessing services provided by other programs**, this approach is called **Service-Oriented Architecture (SOA)**.

- **SOA approach:** The overall application makes use of the services of other applications. One copy of data lives on the service provider; consumers access it via API.
- **Non-SOA approach:** A single standalone application contains all the code necessary to implement itself.

**Real-world SOA example:** Booking a flight on a travel website. The site contacts hotel services, airline services, and car rental services through their respective APIs — the hotel data is NOT stored on the airline's computers. When you pay by credit card, still other services become involved. _(See Figure 13.2 — Service-oriented architecture diagram)_

**SOA advantages:**

- You always maintain **only one copy of data** (critical for things like hotel reservations where you don't want to over-commit).
- The **owners of the data** can set the rules about the use of their data.

When an application makes its API available over the web, the services provided are called **web services**.

---

## PART 8 — SECURITY AND API KEYS

### Why API Keys Exist

It is very common to need an **API key** to use a vendor's API. The general idea is that the vendor wants to know **who is using their services** and **how much** each user is using — for rate-limiting, billing, or preventing abuse.

Vendors may have free and pay tiers, or policies that limit the number of requests a single individual can make during a particular time period.

### Two Common Authentication Approaches

**1. API Key as URL parameter or POST data (simple):**

```
https://api.example.com/data?key=YOUR_API_KEY&query=something
```

You simply include the key as part of the URL or POST body.

**2. OAuth (cryptographic signing):**

For increased assurance of the source of requests, vendors may require **cryptographically signed messages** using shared keys and secrets. The very common technology for this is called **OAuth** (see `www.oauth.net`).

There are convenient and free OAuth libraries available, so you don't need to implement OAuth from scratch. These libraries vary in complexity and richness.

> 💡 Never hard-code API keys in source files you share publicly. Use environment variables or config files that are excluded from version control.

---

## PART 9 — COMPLETE CODE PATTERNS (Quick Reference)

### Pattern 1 — Parse Simple XML

```python
import xml.etree.ElementTree as ET

tree = ET.fromstring(xml_string)
value = tree.find('tag_name').text
attr  = tree.find('tag_name').get('attribute_name')
```

### Pattern 2 — Loop Through Multiple XML Nodes

```python
import xml.etree.ElementTree as ET

stuff = ET.fromstring(xml_string)
items = stuff.findall('parent/child')    # must include parent path!

for item in items:
    print(item.find('name').text)
    print(item.get('attribute'))
```

### Pattern 3 — Parse JSON into Python Structures

```python
import json

data = json.loads(json_string)      # returns list or dict
for item in data:                   # if JSON was a list
    print(item['key'])              # access like a normal dict
```

### Pattern 4 — Fetch and Parse JSON from a URL

```python
import urllib.request, urllib.parse, urllib.error
import json
import ssl

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = 'https://api.example.com/data?key=YOUR_KEY'
response = urllib.request.urlopen(url, context=ctx).read()
data = json.loads(response)

for item in data:
    print(item['name'])
```

### Pattern 5 — Fetch and Parse XML from a URL

```python
import urllib.request
import xml.etree.ElementTree as ET
import ssl

ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = 'https://api.example.com/data.xml'
response = urllib.request.urlopen(url, context=ctx).read()
tree = ET.fromstring(response)

for node in tree.findall('parent/child'):
    print(node.find('name').text)
```

---

## PART 10 — GLOSSARY

|Term|Definition|
|---|---|
|**XML**|eXtensible Markup Language — a format that allows for the markup of structured data; uses opening/closing tags; best for document-style data|
|**JSON**|JavaScript Object Notation — a format for markup of structured data based on JavaScript Object syntax; maps directly to Python dicts and lists|
|**element / node**|A named unit in an XML document defined by an opening and closing tag (e.g., `<name>Chuck</name>`)|
|**attribute**|A key-value pair inside an XML opening tag (e.g., `type="intl"`); has no direct equivalent in JSON|
|**self-closing tag**|An XML tag for an element with no content (e.g., `<email hide="yes" />`)|
|**ElementTree**|A built-in Python library used to parse XML data into a navigable tree structure|
|**`ET.fromstring()`**|Parses an XML string into an ElementTree object|
|**`.find()`**|Returns the first child element matching a tag name|
|**`.findall()`**|Returns a list of all matching elements at a given path|
|**`.text`**|The text content between a tag's opening and closing markers|
|**`.get()`**|Retrieves the value of a named attribute from an XML element|
|**`json.loads()`**|Parses a JSON string and returns native Python structures (lists/dicts)|
|**API**|Application Program Interface — a contract between applications that defines the patterns of interaction between two application components|
|**SOA**|Service-Oriented Architecture — when an application is made of components connected across a network, each exposing services via APIs|
|**web service**|An API whose services are made available over the web (HTTP)|
|**API key**|A secret token sent with API requests so the vendor knows who is calling and can enforce rate limits or billing|
|**OAuth**|A protocol for cryptographically signing API requests using shared keys and secrets; used when simple key-passing isn't secure enough|

---

## PART 11 — COMMON MISTAKES AND DEBUGGING

**1. Forgetting the parent path in `findall`:**

```python
lst = stuff.findall('user')          # ❌ returns 0 if 'user' isn't a direct child
lst = stuff.findall('users/user')    # ✅ include all parent levels
```

**2. Confusing `.find()` (returns element) with `.findall()` (returns list):**

```python
item = tree.find('name')             # returns one Element or None
items = tree.findall('users/user')   # returns a Python list (possibly empty)
```

**3. Calling `.text` on a missing element (returns None → AttributeError):**

```python
val = tree.find('missing').text      # ❌ AttributeError: 'NoneType' object has no attribute 'text'

# ✅ safe version:
el = tree.find('missing')
val = el.text if el is not None else 'default'
```

**4. Using `json.load()` vs `json.loads()`:**

```python
json.load(file_object)     # reads from a file handle
json.loads(string)         # parses a string — use this when you have urllib response text
```

**5. Forgetting to decode bytes before passing to `json.loads()` or `ET.fromstring()`:**

```python
raw = urllib.request.urlopen(url).read()    # returns bytes
json.loads(raw)                             # ✅ json.loads() accepts bytes in Python 3.6+
ET.fromstring(raw)                          # ✅ ElementTree also accepts bytes
ET.fromstring(raw.decode())                 # also fine
```

**6. Treating parsed JSON as something other than native Python:**

```python
data = json.loads(json_str)
data.find('name')           # ❌ 'dict' object has no attribute 'find' — that's XML
data['name']                # ✅ it's just a dict now
```

---

## PART 12 — XML vs JSON CODE COMPARISON (Side-by-Side)

The same data — two users — represented and parsed both ways:

**XML version:**

```python
import xml.etree.ElementTree as ET

xml_input = '''
<stuff>
  <users>
    <user x="2"><id>001</id><name>Chuck</name></user>
    <user x="7"><id>009</id><name>Brent</name></user>
  </users>
</stuff>'''

stuff = ET.fromstring(xml_input)
lst = stuff.findall('users/user')

for item in lst:
    print(item.find('name').text, item.find('id').text, item.get('x'))
```

**JSON version:**

```python
import json

json_input = '''
[
  {"id":"001","x":"2","name":"Chuck"},
  {"id":"009","x":"7","name":"Brent"}
]'''

info = json.loads(json_input)

for item in info:
    print(item['name'], item['id'], item['x'])
```

**Both produce the same output.** The JSON version is more concise; the XML version is more explicit about structure.

---

## PART 13 — EXERCISES

**XML:**

- [ ] Modify the `xml1.py` example to also print the phone number from the XML tree using `.find().text`.
- [ ] Write a program that reads the multi-user XML, counts how many users have the `x` attribute set to a value greater than 5.

**JSON:**

- [ ] Rewrite the XML multi-user example entirely using JSON input — produce the same output using `json.loads()`.
- [ ] Write a program that fetches JSON from a public API (e.g., `https://jsonplaceholder.typicode.com/users`) and prints the name and email of each user.

**APIs:**

- [ ] Sign up for a free API key from a public service (e.g., OpenWeatherMap). Write a program using `urllib` to fetch current weather for your city and parse the JSON response.
- [ ] Find a public XML-based web service and write a Python script to extract three fields from its response using `ElementTree`.

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[NETWORKED PROGRAMS]]
- [[REGEX]]
- [[DICTIONARIES, TUPLES & SETS]]
- [[FILE HANDLING]]
- [[OBJECT-ORIENTED PROGRAMMING (OOP)]]