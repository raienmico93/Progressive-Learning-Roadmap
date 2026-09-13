Here's a comprehensive set of study notes for **"Web Fundamentals Prerequisites"** — the essential background knowledge you need before diving into HTML, CSS, and JavaScript.

---

# Web Fundamentals Prerequisites

## 1. Internet and World Wide Web Concepts

### The Internet vs. the Web

These terms are often used interchangeably, but they are **not the same thing**.

| Concept | Definition |
|---|---|
| **Internet** | A global network of interconnected computers that communicate using standardized protocols (TCP/IP). |
| **World Wide Web (WWW)** | A service that runs *on top of* the Internet — a system of interlinked hypertext documents accessed via browsers. |

**Analogy:** The Internet is the road system; the Web is the delivery trucks that use those roads. Email, VoIP, and file transfer also use the Internet but are not part of the Web.

### Key Characteristics of the Web
- **Hypertext**: Documents linked to one another via hyperlinks.
- **Distributed**: No single central server owns the Web.
- **Open standards**: Built on publicly documented protocols (HTTP, HTML, URLs).
- **Platform-independent**: Any device with a browser can access it.

---

## 2. Client-Server Architecture

The Web follows a **client-server model**.

```
┌─────────────┐         Request          ┌─────────────┐
│   CLIENT    │ ───────────────────────► │   SERVER    │
│ (Browser)   │                          │ (Web Server)│
│             │ ◄─────────────────────── │             │
└─────────────┘         Response         └─────────────┘
```

### Client
- Usually a **web browser** (Chrome, Firefox, Safari, Edge).
- Initiates requests for resources.
- Renders the received content for the user.

### Server
- A computer (or software) that **listens for requests** and **returns responses**.
- Stores and serves web resources (HTML, CSS, JS, images, data).
- Examples: Apache, Nginx, IIS, Node.js servers.

### Key Points
- Communication is **request-driven** — the client always initiates.
- Servers are typically **stateless** (each request is independent unless sessions/cookies are used).
- One server can serve many clients simultaneously.

---

## 3. Web Browsers

A **web browser** is software that retrieves, interprets, and displays web content.

### Major Browsers
- Google Chrome (Blink engine)
- Mozilla Firefox (Gecko engine)
- Safari (WebKit engine)
- Microsoft Edge (Blink engine)
- Opera (Blink engine)

### Core Components of a Browser

| Component | Role |
|---|---|
| **User Interface** | Address bar, tabs, buttons |
| **Rendering Engine** | Parses HTML/CSS and paints pixels (e.g., Blink, Gecko) |
| **JavaScript Engine** | Executes JS (e.g., V8, SpiderMonkey) |
| **Networking** | Handles HTTP requests/responses |
| **UI Backend** | Draws widgets and windows |
| **Data Storage** | Cookies, localStorage, IndexedDB, cache |

### What Browsers Do
1. Accept a URL from the user.
2. Resolve the domain via DNS.
3. Send an HTTP request to the server.
4. Receive and parse the response.
5. Render the page and execute scripts.
6. Handle user interaction and dynamic updates.

---

## 4. Web Servers

A **web server** is software (and often the machine running it) that serves web content to clients.

### Responsibilities
- Listen for incoming HTTP/HTTPS requests.
- Process requests (static files or dynamic content via backend code).
- Return appropriate responses (HTML, JSON, images, error codes).
- Manage security, logging, and performance.

### Popular Web Servers
- **Apache HTTP Server** — mature, widely used.
- **Nginx** — high performance, often used as a reverse proxy.
- **IIS** — Microsoft's server.
- **Node.js (with Express)** — JavaScript-based server.
- **Caddy** — modern, automatic HTTPS.

### Static vs. Dynamic Serving
- **Static**: Returns pre-existing files (e.g., `index.html`, `logo.png`).
- **Dynamic**: Generates content on-the-fly (e.g., a user's dashboard from a database).

---

## 5. HTTP and HTTPS Fundamentals

### What is HTTP?
**HyperText Transfer Protocol** — the protocol that defines how messages are formatted and transmitted between clients and servers.

- **Stateless**: Each request is independent.
- **Text-based** (HTTP/1.1 and HTTP/2 use binary framing but are conceptually textual).
- Runs over **TCP** (and now **QUIC** for HTTP/3).

### What is HTTPS?
**HTTP Secure** — HTTP layered over **TLS/SSL encryption**.

- Encrypts data in transit.
- Verifies server identity via certificates.
- Prevents eavesdropping and tampering.
- Now the **default** for virtually all websites.

### HTTP Methods

| Method | Purpose |
|---|---|
| GET | Retrieve a resource |
| POST | Submit data (create) |
| PUT | Replace a resource |
| PATCH | Partially update a resource |
| DELETE | Remove a resource |
| HEAD | Like GET but returns only headers |
| OPTIONS | Query supported methods |

### HTTP Status Codes

| Range | Meaning | Examples |
|---|---|---|
| 1xx | Informational | 100 Continue |
| 2xx | Success | 200 OK, 201 Created |
| 3xx | Redirection | 301 Moved Permanently, 302 Found |
| 4xx | Client Error | 404 Not Found, 403 Forbidden |
| 5xx | Server Error | 500 Internal Server Error |

### Common HTTP Headers
- **Request**: `Host`, `User-Agent`, `Accept`, `Cookie`, `Authorization`
- **Response**: `Content-Type`, `Content-Length`, `Set-Cookie`, `Cache-Control`

---

## 6. URLs and URIs

### URI vs. URL vs. URN

| Term | Meaning |
|---|---|
| **URI** | Uniform Resource Identifier — a string identifying a resource. The broadest term. |
| **URL** | Uniform Resource Locator — a URI that specifies *how to locate* a resource (includes protocol, host, path). |
| **URN** | Uniform Resource Name — a URI that names a resource without specifying location (e.g., `urn:isbn:0451450523`). |

**Every URL is a URI, but not every URI is a URL.**

### Anatomy of a URL

```
https://www.example.com:443/path/page.html?query=value#section
└─┬─┘   └──────┬───────┘└┬┘└─────┬─────┘└────┬────┘└──┬──┘
scheme      host       port    path       query   fragment
```

| Part | Description |
|---|---|
| **Scheme** | Protocol (`http`, `https`, `ftp`, `mailto`) |
| **Host** | Domain name or IP address |
| **Port** | Optional; default 80 (HTTP) or 443 (HTTPS) |
| **Path** | Location of the resource on the server |
| **Query String** | Key-value pairs after `?`, separated by `&` |
| **Fragment** | Anchor within the page after `#` (not sent to server) |

### Types of URLs
- **Absolute**: Full URL with scheme and host (`https://example.com/page`).
- **Relative**: Path relative to the current document (`/about`, `../img/logo.png`).

---

## 7. Domain Names

A **domain name** is a human-readable label that maps to an IP address.

### Structure

```
www.example.com.
└┬─┘ └──┬──┘└─┬─┘
subdomain  SLD  TLD
```

| Part | Meaning |
|---|---|
| **TLD** (Top-Level Domain) | `.com`, `.org`, `.net`, `.ph`, `.io` |
| **SLD** (Second-Level Domain) | `example` — the name you register |
| **Subdomain** | `www`, `blog`, `api` — optional, configurable |
| **Root** | The trailing dot (usually omitted) |

### Types of TLDs
- **gTLD**: Generic (`.com`, `.org`, `.app`)
- **ccTLD**: Country code (`.ph`, `.uk`, `.jp`)
- **Sponsored**: Restricted (`.gov`, `.edu`, `.mil`)

### Domain Registration
- Domains are registered through **registrars** (e.g., Namecheap, GoDaddy).
- Managed under **ICANN** (global) and **registry operators** per TLD.
- Registration grants **use rights**, not ownership, for a period of time.

---

## 8. DNS Fundamentals

**DNS (Domain Name System)** is the Internet's "phonebook" — it translates domain names into IP addresses.

### Why DNS Exists
Humans remember names (`google.com`); computers route to numbers (`142.250.190.78`).

### The DNS Resolution Process

```
1. Browser cache → OS cache → Router cache
2. Recursive Resolver (ISP or public like 8.8.8.8)
3. Root Nameserver        → directs to TLD server
4. TLD Nameserver         → directs to authoritative server
5. Authoritative Nameserver → returns the IP address
```

### Common DNS Record Types

| Record | Purpose |
|---|---|
| **A** | Maps a name to an IPv4 address |
| **AAAA** | Maps a name to an IPv6 address |
| **CNAME** | Alias to another domain name |
| **MX** | Mail server for the domain |
| **TXT** | Arbitrary text (used for verification, SPF, DKIM) |
| **NS** | Nameservers for the domain |
| **SOA** | Start of Authority — zone metadata |

### Key Concepts
- **Propagation**: DNS changes may take hours to spread globally (TTL-based).
- **TTL (Time To Live)**: How long a record is cached.
- **DNSSEC**: Security extension to prevent spoofing.

---

## 9. Request-Response Lifecycle

This is the **full journey** of a single web request. Understanding it is critical.

### Step-by-Step

```
1.  User enters URL in browser
2.  Browser checks caches (browser, OS, DNS)
3.  DNS lookup → resolve domain to IP
4.  TCP handshake (SYN → SYN-ACK → ACK)
5.  TLS handshake (for HTTPS)
6.  Browser sends HTTP request
7.  Server processes request
8.  Server sends HTTP response
9.  Browser receives response
10. Browser parses HTML → builds DOM
11. Browser requests additional resources (CSS, JS, images)
12. Browser renders the page
13. JavaScript executes, page becomes interactive
```

### Diagram

```
Browser                Network                Server
   │                      │                      │
   │──DNS query──────────►│                      │
   │◄─IP address──────────│                      │
   │──TCP handshake──────►│─────────────────────►│
   │──TLS handshake──────►│─────────────────────►│
   │──HTTP request───────►│─────────────────────►│
   │                      │                      │ process
   │◄─HTTP response───────│◄─────────────────────│
   │──requests for CSS/JS►│─────────────────────►│
   │◄─resources───────────│◄─────────────────────│
   │  render + execute    │                      │
```

### Latency Contributors
- DNS resolution time
- TCP/TLS handshake rounds
- Network distance (RTT)
- Server processing time
- Payload size
- Number of additional resource requests

---

## 10. Browser Rendering Fundamentals

Once the browser receives HTML, it must turn it into pixels. This is the **critical rendering path**.

### The Rendering Pipeline

```
HTML ──► DOM Tree
             │
CSS  ──► CSSOM Tree
             │
        Render Tree ──► Layout ──► Paint ──► Composite
             ▲
JavaScript ──┘ (can modify DOM/CSSOM at any time)
```

### Stages Explained

| Stage | Description |
|---|---|
| **1. Parsing HTML** | Browser reads HTML and builds the **DOM** (Document Object Model) tree. |
| **2. Parsing CSS** | Browser reads CSS and builds the **CSSOM** (CSS Object Model) tree. |
| **3. Render Tree** | DOM + CSSOM combine — only visible elements included. |
| **4. Layout (Reflow)** | Browser calculates position and size of each element. |
| **5. Paint** | Browser fills in pixels (colors, text, images, borders). |
| **6. Composite** | Layers are combined and drawn to the screen. |

### Key Terms

- **DOM**: Tree representation of the HTML document, manipulable via JavaScript.
- **CSSOM**: Tree representation of styles.
- **Reflow**: Recalculating layout (expensive).
- **Repaint**: Redrawing pixels without changing layout (less expensive).
- **Critical Rendering Path**: The sequence from HTML bytes → pixels.
- **Blocking Resources**:
  - **CSS** is render-blocking — the page won't render until CSSOM is built.
  - **JavaScript** is parser-blocking by default — it pauses HTML parsing unless `async` or `defer` is used.

### Performance Implications
- Minimize render-blocking resources.
- Inline critical CSS.
- Defer non-critical JavaScript.
- Optimize images and reduce payload size.
- Use modern formats (WebP, AVIF) and lazy loading.

---

## Summary Table: How Everything Connects

| Layer | Purpose | Example |
|---|---|---|
| **Internet** | Global network infrastructure | TCP/IP, routers, cables |
| **DNS** | Name → IP translation | `google.com` → `142.250.190.78` |
| **URL/URI** | Identifies resources | `https://example.com/page` |
| **HTTP/HTTPS** | Transfers resources | Request/response messages |
| **Client** | Requests and renders | Chrome, Firefox |
| **Server** | Serves resources | Nginx, Apache |
| **Browser Rendering** | Turns bytes into pixels | DOM → CSSOM → Render Tree |

---

## Key Takeaways

1. The **Internet** is infrastructure; the **Web** is a service running on it.
2. The **client-server model** drives all web communication.
3. **HTTP/HTTPS** define how requests and responses are exchanged.
4. **DNS** translates human-friendly names into machine-routable IPs.
5. A **URL** is a structured address pointing to a resource.
6. The **request-response lifecycle** is the complete journey from click to render.
7. **Browser rendering** follows a precise pipeline: DOM → CSSOM → Render Tree → Layout → Paint → Composite.
8. Understanding these fundamentals makes learning **HTML, CSS, and JavaScript** far more meaningful.

---

Would you like me to follow this with **"2. HTML Document Structure"** or expand any section (e.g., DNS deep-dive, HTTP/2 vs HTTP/3, or rendering performance)?