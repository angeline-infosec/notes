# _HTTP in Detail_
_Learn about how you request content from a web server using the HTTP protocol_


## Task 1: What is HTTP(S)?

### **HTTP (HyperText Transfer Protocol)** 

HTTP stands for Hypertext Transfer Protocol. It is the foundational application-layer protocol used to transfer all types of data across the World Wide Web (HTML, images, videos, API responses, and more). It was developed by Tim Berners-Lee and his team between 1989–1991.

**Key features of HTTP:**
- **Client-Server Model**: A client (usually your browser) sends a request; a web server sends back a response.
- **Stateless Protocol**: Each request is handled independently. The server doesn't inherently remember previous requests from the same client. (This is exactly why cookies exist)
- **Data Delivery**: HTTP fetches all kinds of resources like HTML documents, images, videos, and including data from APIs - structured data (often JSON or XML) that applications exchange with each other, as opposed to a full rendered webpage for a human to view.

**API (Application Programming Interface)**: A set of rules and definitions that lets different software systems communicate with each other — what requests can be made, what data format to send/expect, and what responses will look like. It's essentially a contract between two pieces of software, rather than between a human and an interface.

An analogy about how this interaction happens: 
API = a waiter in a restaurant.

You (the client/app) don't walk into the kitchen (the server) and cook your own food. You tell the waiter (the API) what you want from the menu. The waiter takes your order to the kitchen using a specific format the kitchen understands, and brings back exactly what you asked for.

* You never see how the kitchen actually works; you just interact through the waiter.
* The menu is like the API's documentation: it tells you what you're allowed to ask for.
* If you ask for something not on the menu (a malformed request), the waiter comes back with "sorry, we don't have that," and that's your 400/404 error.

### **HTTPS (HyperText Transfer Protocol Secure)** 

HTTPS stands for HyperText Transfer Protocol Secure. It is the secure version of HTTP. HTTPS data is encrypted so it not only stops people from seeing the data you are receiving and sending, but it also gives you assurances that you're talking to the correct web server and not something impersonating it.

It provides two things HTTP alone doesn't:
1. **Confidentiality**: data in transit is encrypted, so it can't be read if intercepted.
2. **Authentication**: a certificate confirms you're actually talking to the real server, not an impersonator.


### HTTP vs. HTTPS

HTTP: Data is sent in plain text, making it vulnerable to interception.

HTTPS: The secure version of HTTP. It uses encryption (SSL/TLS) to protect sensitive data. 

> **Important nuance**: HTTPS being "secure" only means the *connection* is encrypted and the server's identity is verified. It does **not** make a website immune to attacks. Vulnerabilities like SQL injection, XSS, weak authentication, or a misconfigured/expired certificate can all still exist on a site served over HTTPS. Encryption in transit ≠ a secure application.

### A note on SSL/TLS and where they actually sit
SSL/TLS are often loosely called "transport layer" protocols, but that's not quite accurate in the OSI model sense — they don't replace TCP. Instead, they sit **between the application layer and the transport layer**: your HTTP data gets encrypted by TLS first, then handed down to TCP for actual delivery. TLS is the modern, secure successor to the older (now deprecated) SSL protocol.

### Servers — the other half of the conversation
A **server** is hardware/software that processes requests from clients and returns data, resources, or services over a network. Common types:

| Server Type | Purpose |
|---|---|
| **Web Server** | Stores website files and serves them to users via HTTP/HTTPS |
| **File Server** | Stores and shares files across multiple users securely |
| **Database Server** | Runs a DBMS to store, manage, and retrieve data for client applications |
| **Mail Server** | Sends, receives, and stores email |

**Q&A**
- What does HTTP stand for? **Hypertext Transfer Protocol**
- What does the S in HTTPS stand for? **Secure**
- Challenge flag from the mock webpage's certificate issue: **THM{INVALID_HTTP_CERT}**

---

## Task 2: Requests and Responses

### URLs (Uniform Resource Locator)
A URL tells the browser exactly how and where to find a resource. Breaking down `http://user:password@tryhackme.com:80/view-room?id=1#task3`:

| Component | Meaning | From example |
|---|---|---|
| **Scheme** | Protocol to use (HTTP, HTTPS, FTP) | `http` |
| **User** | Optional credentials embedded in the URL | `user:password` |
| **Host/Domain** | Domain name or IP of the server | `tryhackme.com` |
| **Port** | Port to connect on (default 80 for HTTP, 443 for HTTPS; can technically be 1–65535) | `80` |
| **Path** | Location/file being requested | `/view-room` |
| **Query String** | Extra parameters sent to the path | `?id=1` |
| **Fragment** | Jumps to a specific section of the loaded page | `#task3` |

### Anatomy of a request
A minimal request can be a single line: `GET / HTTP/1.1` — meaning "retrieve the resource at the root path, using HTTP version 1.1." In practice, requests carry additional **headers** for context (covered fully in Task 5).

```http
GET / HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Referer: https://tryhackme.com/
```
- Line 1: Method (`GET`), path (`/`), HTTP version (`1.1`)
- Line 2: Which website is being requested
- Line 3: The client's browser/version
- Line 4: The page that linked here
- A blank line always signals the end of the request

### Anatomy of a response
```http
HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Fri, 09 Apr 2021 13:34:03 GMT
Content-Type: text/html
Content-Length: 98

<html>...</html>
```
- Line 1: HTTP version + **status code** (`200 OK` = success — see Task 4)
- Line 2: Server software/version
- Line 3: Server's current date/time
- Line 4: Type of content being returned
- Line 5: Length of the response body, so the client can confirm nothing's missing
- Blank line signals end of headers, followed by the actual response body

**Q&A**
- What HTTP protocol is being used in the above example? **HTTP/1.1**
- What response header tells the browser how much data to expect? **Content-Length**

---

## Task 3: HTTP Methods

HTTP methods are standardized actions a client uses to tell the server what it wants to do with a resource. The four you'll use constantly:

| Method | Purpose | Example |
|---|---|---|
| **GET** | Retrieve information — no data is changed on the server | `GET /articles/5` → fetch article #5 |
| **POST** | Submit data to create a new record | `POST /users` with body `name=alice` → creates a new user |
| **PUT** | Submit data to update an existing record | `PUT /users/5` with body `email=new@x.com` → updates user 5's email |
| **DELETE** | Remove a record | `DELETE /users/5` → deletes user 5 |

**Q&A**
- Create a new user account: **POST**
- Update your email address: **PUT**
- Remove a picture you've uploaded: **DELETE**
- View a news article: **GET**

---

## Task 4: HTTP Status Codes

Status codes are returned in the first line of every response and fall into five ranges:

| Range | Category | Meaning |
|---|---|---|
| 100–199 | Informational | Initial part of the request accepted; continue sending (rarely seen today) |
| 200–299 | Success | Request completed successfully |
| 300–399 | Redirection | Client should look elsewhere for the resource |
| 400–499 | Client Error | Something wrong with the client's request |
| 500–599 | Server Error | Something went wrong on the server's end |

**Common codes:**

| Code | Name | Meaning |
|---|---|---|
| 200 | OK | Request succeeded |
| 201 | Created | A new resource (user, post, etc.) was created |
| 301 | Moved Permanently | Resource permanently relocated |
| 302 | Found | Resource temporarily relocated |
| 400 | Bad Request | Request was malformed or missing required data |
| 401 | Not Authorised | Authentication required |
| 403 | Forbidden | Authenticated or not, you don't have permission |
| 404 | Page Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | Wrong method used for this resource |
| 500 | Internal Server Error | Server hit an error it couldn't handle |
| 503 | Service Unavailable | Server overloaded or down for maintenance |

(A fun visual reference for remembering these: [http.cat](https://http.cat).)

**Q&A**
- Created a new user/blog post: **201**
- Tried to access a page that doesn't exist: **404**
- Server can't reach its database and crashes: **503**
- Tried to edit your profile without logging in: **401**

---

## Task 5: Headers

Headers carry extra metadata alongside a request or response. None are strictly *required*, but without them a website won't function or render properly.

**Common request headers (client → server):**

| Header | Purpose |
|---|---|
| `Host` | Which site to serve, when one server hosts multiple sites |
| `User-Agent` | Browser/software + version, so the server can format content appropriately |
| `Content-Length` | Size of the data being sent (e.g. a form submission) |
| `Accept-Encoding` | Compression methods the client supports |
| `Cookie` | Data sent back to help the server "remember" the client (see Task 6) |

**Common response headers (server → client):**

| Header | Purpose |
|---|---|
| `Set-Cookie` | Instructs the client to store data, sent back on future requests |
| `Cache-Control` | How long to cache this content before re-requesting |
| `Content-Type` | What kind of data is being returned (HTML, JSON, image, etc.) |
| `Content-Encoding` | Compression method used on the response body |

**Q&A**
- Tells the server what browser is being used: **User-Agent**
- Tells the browser what type of data is being returned: **Content-Type**
- Tells the server which website is being requested: **Host**

---

## Task 6: Cookies

Because HTTP is **stateless**, the server has no built-in memory of who you are between requests. Cookies solve this: small pieces of data the server asks your browser to store (via `Set-Cookie`), which your browser then sends back automatically on every subsequent request (via the `Cookie` header). This is how a site "remembers" you're logged in, your preferences, or that you've visited before.

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: GET / (no cookie yet)
    Server-->>Client: 200 OK + form asking for name
    Client->>Server: POST / (name=adam)
    Server-->>Client: 200 OK + Set-Cookie: name=adam
    Client->>Server: GET / (Cookie: name=adam)
    Server-->>Client: 200 OK — "Welcome back, adam"
```

Cookie values used for authentication are typically not plain-text passwords but **tokens** — random, hard-to-guess secret strings that identify a session.

**A note on privacy**: cookies are a meaningful contributor to your *passive* online data trail — sites can track behavior across visits (and sometimes across other sites, via third-party cookies) without you actively providing information each time.

**Q&A**
- Header used to save cookies to your computer: **Set-Cookie**

---

## Task 7: Making Requests (Practical)

| Action | Result / Flag |
|---|---|
| `GET /room` | `THM{YOU'RE_IN_THE_ROOM}` |
| `GET /blog?id=1` | `THM{YOU_FOUND_THE_BLOG}` |
| `DELETE /user/1` | `THM{USER_IS_DELETED}` |
| `PUT /user/2` with `username=admin` | `THM{USER_HAS_UPDATED}` |
| `POST /login` with `username=thm&password=letmein` | `THM{HTTP_REQUEST_MASTER}` |

---

## Key Takeaways
- HTTP = the protocol for transferring web data; stateless by design; client-server model.
- HTTPS = HTTP + TLS encryption + server identity verification — but "secure connection" ≠ "secure application."
- URLs encode everything needed to locate a resource: scheme, host, port, path, query string, fragment.
- Four core methods: GET (read), POST (create), PUT (update), DELETE (remove).
- Status codes: 2xx success, 3xx redirect, 4xx client error, 5xx server error.
- Headers carry metadata both ways; cookies (via `Set-Cookie`/`Cookie`) patch over HTTP's statelessness to enable sessions.
