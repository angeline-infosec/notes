# _DNS in Detail_
_Learn how DNS works and how it helps you access internet services._

---

## Background Concepts

Before getting into DNS itself, a few underlying concepts make the rest of this easier to follow.

**IP Address**

An IP (Internet Protocol) address is a unique string of numbers assigned to every device (like a phone, computer, or router) that connects to a network or the internet; it's the actual destination DNS resolves a domain name to.

- **IPv4**: 32-bit, written as four decimal octets (0–255) separated by periods, e.g. `104.26.10.229`. ~4.3 billion possible addresses, which is why IPv6 exists.
- 
- **IPv6**: 128-bit, written as eight groups of hexadecimal digits separated by colons, e.g. `2606:4700:20::681a:be5`. Designed to solve IPv4 address exhaustion.


**TCP vs UDP**

DNS's choice of transport protocol matters for how it behaves on the network:

- **TCP (Transmission Control Protocol)**: connection-oriented, reliable protocol, ordered delivery, but higher overhead due to the three-way handshake. DNS falls back to TCP when a response is too large to fit in a single UDP packet (traditionally >512 bytes without EDNS0) or for **zone transfers** (AXFR) between DNS servers. (eg. Web browsing, emails, file transfer, and online banking where missing data causes errors.)
  
- **UDP (User Datagram Protocol)**: connectionless, no handshake, no delivery guarantee but fast, low overhead. DNS uses this by default for standard lookups because speed matters more than guaranteed delivery for a quick query/response. (eg. Live video streaming, online multiplayer gaming, and voice calls where a dropped frame matters less than a delay)

- TCP focuses on accuracy and reliable delivery, while UDP trades away those checks to maximize raw speed.
  
---

## Task 1: What is DNS?

DNS, which stands for Domain Name System, is an application-layer protocol that runs on port 53 and primarily uses UDP for speed, falling back to TCP for large data transfers. It translates human-readable domain names into machine-readable IP addresses and vice versa, allowing network devices to locate and route traffic to internet resources. 

**Q: What does DNS stand for?**

A: Domain Name System

---

## Task 2: Domain Hierarchy

DNS names are structured as a hierarchy, read right to left in terms of authority:

| Component | Definition | Example |
|---|---|---|
| **Root Domain** | The unnamed top of the DNS tree, represented by `.` | `.` |
| **TLD (Top-Level Domain)** | Rightmost label of a domain. Split into **gTLD** (generic - `.com`, `.org`, `.edu`, `.gov`, historically purpose-based) and **ccTLD** (country code - `.ca`, `.co.uk`, geography-based) | `.com` |
| **Second-Level Domain** | Sits directly left of the TLD. Max 63 characters + TLD; only `a-z`, `0-9`, hyphens (no leading/trailing or consecutive hyphens) | `tryhackme` in `tryhackme.com` |
| **Subdomain** | Sits left of the second-level domain, separated by a period. Same character/length rules as second-level domains (63 chars max per label); unlimited number of subdomains allowed, but full domain name capped at 253 characters | `admin` in `admin.tryhackme.com` |

**Diagram: domain hierarchy tree:**

```mermaid
graph TD
    Root(("Root Domain ."))
    Root --> edu[".edu"]
    Root --> com[".com"]
    Root --> gov[".gov"]
    Root --> mil[".mil"]

    edu --> MIT["MIT (2nd-level)"]
    com --> THM["Tryhackme (2nd-level)"]
    com --> Google["Google (2nd-level)"]
    gov --> USA["USA (2nd-level)"]
    gov --> NASA["NASA (2nd-level)"]
    mil --> Army["Army (2nd-level)"]
```

**Q&A**
- What is the maximum length of a subdomain? **63**
- Which of the following characters cannot be used in a subdomain (`3`, `b`, `_`, `-`)? **`_`**
- What is the maximum length of a domain name? **253**
- What type of TLD is `.co.uk`? **ccTLD**

---

## Task 3: Record Types

| Record | Resolves to | Example |
|---|---|---|
| **A** | IPv4 address | `104.26.10.229` |
| **AAAA** | IPv6 address | `2606:4700:20::681a:be5` |
| **CNAME** | Another domain name (alias) | `store.tryhackme.com` → `shops.shopify.com` |
| **MX** | Mail server(s) for the domain, with a priority value determining try-order (lower = higher priority) | `alt1.aspmx.l.google.com` |
| **TXT** | Free-text field; used for domain ownership verification, SPF/DKIM/DMARC anti-spoofing records, etc. | `"v=spf1 ip4:192.0.2.0/24 ~all"` |

**Q&A**
- What type of record would be used to advise where to send email? **MX**
- What type of record handles IPv6 addresses? **AAAA**

---

## Task 4: Making a Request (DNS Resolution Process)

### The DNS server hierarchy

- **Recursive DNS Server**: This is the "middleman" that does the searching for you. Usually run by your internet provider (or one you pick yourself, like `1.1.1.1`). When you ask for a website, this server checks if it already knows the answer from a recent search(it checks its local cache). If it does, it answers right away. If not, it goes and finds out for you.

- **Root DNS Server**: Think of this as the starting point of every search. It doesn't know the actual answer, but it knows which direction to send you based on the ending of the domain (like `.com` or `.gov`). (DNS backbone of the internet. It recognizes the TLD of .com(example) and gives the correct TLD server that deals with .com addresses)

- **TLD Server**: This server knows which specific server is actually in charge of a domain. It doesn't have the final answer either; it just tells you exactly where to go next to get it.

- **Authoritative DNS Server (Nameserver)**: This is the final stop. The server that actually owns and stores the real answer for a domain. Any changes to a domain's DNS info happen here first. Most domains have more than one of these as a backup, in case one goes down. (Stores DNS records for particular domain names)


### Resolution flow

```mermaid
sequenceDiagram
    participant Client
    participant Recursive as Recursive DNS Server
    participant Root as Root DNS Server
    participant TLD as TLD Server
    participant Auth as Authoritative DNS Server

    Client->>Recursive: 1. Query (check local cache first)
    Recursive->>Root: 2. Cache miss — ask root
    Root-->>Recursive: 3. Refers to correct TLD server
    Recursive->>TLD: Query TLD server
    TLD-->>Recursive: 4. Refers to authoritative nameserver
    Recursive->>Auth: Query authoritative server
    Auth-->>Recursive: Returns DNS record + TTL
    Recursive-->>Client: 5. Returns answer (caches it locally)
```

Easier to remember:





**TTL (Time To Live)**: every DNS record includes a TTL value, in seconds, specifying how long the record can be cached before it must be looked up again. This is what makes caching and the "cache hit" shortcut at step 2 possible, reducing repeated lookups for popular domains.

**Q&A**
- What field specifies how long a DNS record should be cached for? **TTL**
- What type of DNS Server is usually provided by your ISP? **Recursive DNS Server**
- What type of server holds all the records for a domain? **Authoritative DNS Server**

---

## Task 5: Practical

**Q&A**
- CNAME of `shop.website.thm`: **shops.myshopify.com**
- TXT record value of `website.thm`: **THM{7012BBA60997F35A9516C2E16D2944FF}**
- Numerical priority value for the MX record: **30**
- IP address for the A record of `www.website.thm`: **10.10.10.10**

---

## Key Takeaways
- DNS = phonebook for the internet; port 53, UDP by default, TCP for large responses/zone transfers.
- Hierarchy: Root → TLD → Second-Level Domain → Subdomain.
- Five core record types to know cold: A, AAAA, CNAME, MX, TXT.
- Resolution path: Client (cache) → Recursive Resolver (cache) → Root → TLD → Authoritative → back down the chain, cached along the way per TTL.
