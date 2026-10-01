# Low-Level Network Concepts & TCP/IP — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Low-level network programming in Node.js refers to the use of the built-in `net`, `dns`, and `tls` modules to build TCP/IP servers and clients, resolve domain names, and establish encrypted connections, operating below the abstraction layer of HTTP.

**Technical Definition:** Node.js provides three core modules for low-level networking. The `node:net` module provides an asynchronous network API for creating stream-based TCP or IPC servers and clients, exposing the `net.Server` and `net.Socket` classes. The `node:dns` module enables name resolution, with `dns.lookup()` using the operating system's `getaddrinfo(3)` facilities and `dns.resolve()` performing actual DNS protocol queries over the network. The `node:tls` module provides an implementation of the Transport Layer Security (TLS) and Secure Socket Layer (SSL) protocols built on top of OpenSSL, exposing `tls.createServer()` and `tls.TLSSocket` for secure communication.

**Beginner-Friendly Explanation:** When you use HTTP, Node.js handles all the messy details of networking for you. Low-level networking is about understanding and controlling those details yourself — how TCP connections are established (the handshake), how domain names are translated to IP addresses (DNS), how data is encrypted (TLS), and how connections are kept alive and reused. Think of it as the difference between driving an automatic car (HTTP) and understanding what happens under the hood (TCP/IP, DNS, TLS).

### Key Characteristics

- **Stream-based TCP:** `net.Socket` is a Duplex stream, supporting `write()`, `'data'` events, and backpressure like any other Node.js stream.
- **Asynchronous DNS:** DNS resolution is non-blocking, but `dns.lookup()` runs on libuv's threadpool (limited to 4 threads by default), while `dns.resolve()` uses c-ares and does not block the threadpool.
- **TLS/SSL encryption:** `tls.createServer()` provides TLS termination with support for certificates, CAs, SNI, ALPN, and mutual TLS (mTLS).
- **Connection lifecycle management:** TCP connections go through a well-defined state machine (LISTEN, SYN-SENT, ESTABLISHED, etc.), and Node.js exposes hooks for timeouts, keep-alive, and graceful shutdown.
- **Backlog control:** `server.listen(port, host, backlog)` controls the maximum number of pending connections the OS will queue.
- **Keep-alive optimisation:** `socket.setKeepAlive()` and `http.Agent` with `keepAlive: true` enable TCP connection reuse, reducing handshake overhead.

### Prerequisites

- **Node.js runtime:** The `net`, `dns`, and `tls` modules are built into Node.js.
- **Basic JavaScript knowledge:** Understanding of functions, callbacks, and the `EventEmitter` pattern.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Basic networking concepts:** IP addresses, ports, TCP vs. UDP, and the client-server model.
- **Stream fundamentals:** Understanding of Readable, Writable, and Duplex streams.

### Related Programming Areas

- **HTTP Fundamentals:** `http.Server` extends `net.Server`, and HTTP clients use `net.Socket` internally.
- **Streams:** TCP sockets are Duplex streams; TLS sockets wrap TCP sockets.
- **Security:** TLS/SSL, certificate management, and mTLS authentication.
- **Performance:** Keep-alive, connection pooling, and backlog tuning.
- **DNS:** Name resolution, caching, and threadpool considerations.

### Core Concepts

1. **The Transport Layer** — TCP/IP stack fundamentals, `net.createServer`/`net.Socket`, ports, and backlog.
2. **Domain Name System (DNS)** — `dns.resolve()` vs. `dns.lookup()`, caching, and threadpool behaviour.
3. **Network Security & Encryption** — TLS/SSL architecture, `tls.createServer()`, and mTLS.
4. **Connection Lifecycles** — Keep-Alive and connection reuse.

---

## Core Concept 1: The Transport Layer

### Sub-Feature 1.1: TCP/IP Stack Fundamentals (Handshakes, Packets, Sliding Windows)

#### Definitions

**Core Definition:** TCP (Transmission Control Protocol) is a connection-oriented, reliable transport protocol that provides ordered, error-checked delivery of data between applications over an IP network.

**Technical Definition:** TCP/IP is the foundation of internet communications. TCP provides reliable, ordered, and error-checked delivery of data between applications running on hosts connected via an IP network. Key characteristics include: connection-oriented protocol (requires a three-way handshake), full-duplex communication, reliable delivery with acknowledgments and retransmissions, flow control via a windowing mechanism, and congestion control. The TCP three-way handshake consists of: the client sends a SYN segment, the server responds with SYN-ACK, and the client sends ACK. TCP uses a sliding window algorithm for flow control: the receiver advertises a window size, and the sender can have multiple unacknowledged packets in flight up to that window size, increasing throughput.

**Beginner-Friendly Explanation:** TCP is like a phone call — you dial (SYN), the other person answers (SYN-ACK), and you say "hello" (ACK). Once connected, you can both talk at the same time (full-duplex). The sliding window is like a conveyor belt that can hold multiple items at once — instead of sending one packet and waiting for confirmation before sending the next, TCP sends several packets and only waits if the receiver's buffer is full.

#### Purposes

- To provide reliable, ordered data delivery over an unreliable IP network.
- To enable full-duplex communication between two endpoints.
- To manage flow control so a fast sender does not overwhelm a slow receiver.
- To detect and recover from packet loss through retransmission.

#### Syntax Rules and Structure

TCP is not directly "syntax" in the programming sense; it is a protocol specification (RFC 793, RFC 9293). However, Node.js exposes TCP behaviour through the `net` module:

```js
const net = require('node:net');

// Create a TCP server
const server = net.createServer((socket) => {
  socket.on('data', (data) => { /* handle data */ });
  socket.on('end', () => { /* handle disconnect */ });
});

server.listen(8000, () => {
  console.log('TCP server listening on port 8000');
});
```

| TCP Concept | Node.js Exposure |
|-------------|------------------|
| Three-way handshake | `'connect'` event on socket |
| Sliding window | Managed by the OS kernel; not directly exposed |
| Packet retransmission | Managed by the OS kernel |
| Connection state | `socket.readyState` (`'opening'`, `'open'`, `'readOnly'`, `'writeOnly'`, `'closed'`) |

**Constraints and Limitations:**
- TCP is a stream protocol; message boundaries are not preserved. Application-level framing is required.
- The OS kernel manages buffering, retransmission, and window sizing; Node.js cannot directly control these.
- TCP connections are identified by the 4-tuple (source IP, source port, destination IP, destination port).

#### Annotated Code Example

```js
// tcp-server.js
const net = require('node:net');

const server = net.createServer((socket) => {
  console.log('Client connected from:', socket.remoteAddress, socket.remotePort);

  // 'data' fires when TCP segments are received and reassembled
  socket.on('data', (data) => {
    console.log('Received:', data.toString().trim());
    socket.write(`Echo: ${data}`);
  });

  socket.on('end', () => {
    console.log('Client disconnected');
  });

  socket.on('error', (err) => {
    console.error('Socket error:', err.message);
  });
});

server.listen(8000, '127.0.0.1', () => {
  console.log('TCP server listening on 127.0.0.1:8000');
});
```

```js
// tcp-client.js
const net = require('node:net');

const client = net.createConnection({ port: 8000, host: '127.0.0.1' }, () => {
  console.log('Connected to server (TCP handshake complete)');
  client.write('Hello, TCP!');
});

client.on('data', (data) => {
  console.log('Server response:', data.toString().trim());
  client.end();
});

client.on('end', () => {
  console.log('Disconnected from server');
});
```

**Expected Output (server):**
```
TCP server listening on 127.0.0.1:8000
Client connected from: 127.0.0.1 54321
Received: Hello, TCP!
Client disconnected
```

**Expected Output (client):**
```
Connected to server (TCP handshake complete)
Server response: Echo: Hello, TCP!
Disconnected from server
```

**Why this output:** The `net.createServer()` creates a TCP server bound to `127.0.0.1:8000`. When the client connects, the OS completes the three-way handshake and Node.js emits the `'connect'` event on the client and the `'connection'` event on the server. Data written by the client is received as a Buffer and echoed back. The `'end'` event fires when the client calls `client.end()`.

#### Real-World Cases

- **Custom protocols:** Building proprietary TCP-based protocols (e.g., database wire protocols).
- **Chat servers:** Implementing real-time bidirectional communication.
- **IoT devices:** Communicating with embedded devices over TCP.
- **Proxies:** Building TCP proxies and load balancers.

---

### Sub-Feature 1.2: Building Raw TCP Servers and Clients Using `net.createServer` and `net.Socket`

#### Definitions

**Core Definition:** `net.createServer()` creates a TCP server, and `net.Socket` represents a TCP socket (connection) that can be used by both servers and clients.

**Technical Definition:** The `net` module provides an asynchronous network API for creating stream-based TCP or IPC servers and clients. The `net.Server` class is used to create a TCP or IPC server. The `net.Socket` class is an abstraction of a TCP socket or a streaming IPC endpoint; it is a Duplex stream that implements the `'connect'`, `'data'`, `'end'`, `'error'`, `'timeout'`, `'drain'`, and `'close'` events. `net.createConnection()` is a factory function that creates a new `net.Socket` and immediately initiates a connection.

**Beginner-Friendly Explanation:** `net.createServer()` is like opening a shop — it listens for customers (clients) to arrive. `net.Socket` is the individual phone line to each customer. The server uses one socket per client, and each socket is a two-way stream: you can read data from it and write data to it.

#### Purposes

- To create TCP servers that listen for and accept incoming connections.
- To create TCP clients that connect to remote servers.
- To implement custom application-level protocols over TCP.
- To handle multiple concurrent connections efficiently.

#### Syntax Rules and Structure

**`net.createServer()`:**
```js
net.createServer(options?, connectionListener?);
```

| Option | Description |
|--------|-------------|
| `allowHalfOpen` | If `false`, the socket automatically ends when the other end sends FIN. Default: `false`. |
| `pauseOnConnect` | If `true`, the socket is paused on connection. Default: `false`. |
| `keepAlive` | Enable TCP keep-alive. |
| `keepAliveInitialDelay` | Initial delay before first keep-alive probe. |

**`net.createConnection()`:**
```js
net.createConnection(options, connectListener?);
```

| Option | Description |
|--------|-------------|
| `port` | Port to connect to. |
| `host` | Host to connect to. |
| `localAddress` | Local address to bind to. |
| `timeout` | Socket timeout in milliseconds. |
| `family` | IP family (4, 6, or 0 for both). |

**Constraints and Limitations:**
- TCP sockets are limited by OS file descriptor limits.
- `allowHalfOpen: true` requires explicit `socket.end()` to close the connection.
- The `'data'` event emits Buffers by default; use `setEncoding()` for strings.

#### Annotated Code Example

```js
// tcp-server-client.js
const net = require('node:net');

// Server
const server = net.createServer({ allowHalfOpen: false }, (socket) => {
  const clientId = `${socket.remoteAddress}:${socket.remotePort}`;
  console.log(`Client connected: ${clientId}`);

  socket.setEncoding('utf8');

  socket.on('data', (data) => {
    console.log(`[${clientId}] ${data.trim()}`);
    socket.write(`ACK: ${data.trim()}\n`);
  });

  socket.on('close', () => {
    console.log(`Client disconnected: ${clientId}`);
  });
});

server.listen(9000, () => {
  console.log('Server listening on port 9000');

  // Client
  const client = net.createConnection({ port: 9000, host: 'localhost' }, () => {
    console.log('Client connected');
    client.write('Message 1\n');
    client.write('Message 2\n');
  });

  client.setEncoding('utf8');
  client.on('data', (data) => {
    console.log('Client received:', data.trim());
  });

  client.on('end', () => {
    console.log('Client disconnected from server');
  });

  setTimeout(() => client.end(), 1000);
});
```

**Expected Output:**
```
Server listening on port 9000
Client connected
Client connected: ::1:54322
[::1:54322] Message 1
Client received: ACK: Message 1
[::1:54322] Message 2
Client received: ACK: Message 2
Client disconnected: ::1:54322
Client disconnected from server
```

**Why this output:** The server accepts a client connection and logs the client's address. The client sends two messages. The server echoes each message back with an `ACK:` prefix. The `allowHalfOpen: false` setting causes the socket to automatically end when the client calls `client.end()`.

#### Real-World Cases

- **Database drivers:** Implementing PostgreSQL, MySQL, or Redis wire protocols.
- **Game servers:** Low-latency TCP communication for multiplayer games.
- **Message brokers:** Custom pub/sub systems over TCP.
- **Remote shell:** Implementing SSH-like protocols.

---

### Sub-Feature 1.3: Port Allocation, Binding Errors (EADDRINUSE), and Internal OS Backlogs

#### Definitions

**Core Definition:** Port binding is the process of associating a server with a specific port number. `EADDRINUSE` is an error that occurs when another process is already using the requested port. The backlog is the maximum number of pending connections the OS will queue before refusing new connections.

**Technical Definition:** When `server.listen(port, host, backlog)` is called, the OS binds the server socket to the specified port. If another process is already bound to that port, the `'error'` event is emitted with `code: 'EADDRINUSE'`. The `backlog` parameter controls the maximum length of the queue of pending connections; if the queue is full, the OS may refuse new connections (behaviour varies by OS). The default backlog is typically 511 on Linux.

**Beginner-Friendly Explanation:** A port is like a door number in a large building (the IP address). If two servers try to use the same door, one gets an `EADDRINUSE` error — "this door is already in use." The backlog is like a waiting room: if the receptionist (the OS) is busy, new clients wait in the room until the server is ready to accept them. If the room is full, new clients are turned away.

#### Purposes

- To bind a server to a specific port and host.
- To handle port conflicts gracefully.
- To tune the backlog for high-traffic servers.
- To detect and recover from binding errors.

#### Syntax Rules and Structure

```js
server.listen(port, host, backlog, callback);
```

| Parameter | Description |
|-----------|-------------|
| `port` | Port number (0 for random allocation). |
| `host` | Hostname or IP address. |
| `backlog` | Maximum pending connections (default: 511 on Linux). |
| `callback` | Invoked when the server starts listening. |

**Common binding errors:**
| Error Code | Meaning | Resolution |
|------------|---------|------------|
| `EADDRINUSE` | Port already in use. | Use a different port or wait for the port to be released. |
| `EACCES` | Permission denied (privileged port). | Use a port above 1024 or run with elevated privileges. |
| `EADDRNOTAVAIL` | Address not available. | Check the host/IP binding. |

**Constraints and Limitations:**
- `EADDRINUSE` can occur even after a server is stopped if the port is in `TIME_WAIT` state.
- The backlog is a hint to the OS; the actual behaviour depends on the OS kernel.
- Setting `backlog` too high may consume kernel memory; setting it too low may cause connection refusals.

#### Annotated Code Example

```js
// port-binding.js
const net = require('node:net');

function createServer(port, retries = 3) {
  const server = net.createServer((socket) => {
    socket.end('Hello!\n');
  });

  server.on('error', (err) => {
    if (err.code === 'EADDRINUSE') {
      console.error(`Port ${port} is already in use.`);
      if (retries > 0) {
        console.log(`Retrying in 1 second (${retries} retries left)...`);
        setTimeout(() => {
          server.close();
          createServer(port, retries - 1);
        }, 1000);
      } else {
        console.error('Exhausted retries. Exiting.');
        process.exit(1);
      }
    } else {
      throw err;
    }
  });

  server.listen(port, '127.0.0.1', 511, () => {
    console.log(`Server listening on port ${port}`);
  });

  return server;
}

// Start two servers on the same port to demonstrate EADDRINUSE
const server1 = createServer(3000);
// server2 will fail with EADDRINUSE
setTimeout(() => {
  const server2 = createServer(3000);
}, 500);
```

**Expected Output:**
```
Server listening on port 3000
Port 3000 is already in use.
Retrying in 1 second (3 retries left)...
Port 3000 is already in use.
Retrying in 1 second (2 retries left)...
...
```

**Why this output:** The first server binds successfully to port 3000. The second server attempts the same port and receives `EADDRINUSE`. The error handler retries with a delay. The `backlog` parameter is set to 511, which is the default on Linux.

#### Real-World Cases

- **Development servers:** Handling port conflicts when a previous process hasn't fully exited.
- **Microservices:** Dynamically allocating ports for inter-service communication.
- **High-traffic servers:** Tuning the backlog for production workloads.
- **Containerised applications:** Binding to `0.0.0.0` to accept connections from outside the container.

---

## Core Concept 2: Domain Name System (DNS)

### Sub-Feature 2.1: Resolving Domains Using `dns.resolve()` and `dns.lookup()`

#### Definitions

**Core Definition:** `dns.lookup()` resolves a hostname to an IP address using the operating system's resolver, while `dns.resolve()` performs an actual DNS protocol query over the network using c-ares.

**Technical Definition:** `dns.lookup(hostname[, options], callback)` resolves a hostname (e.g., `'example.com'`) into the first found A (IPv4) or AAAA (IPv6) record. It uses the same operating system facilities as most other programs and is implemented as a synchronous call to `getaddrinfo(3)` that runs on libuv's threadpool. `dns.resolve(hostname[, rrtype], callback)` uses the DNS protocol to resolve a hostname into an array of resource records. Unlike `dns.lookup()`, it does not use `getaddrinfo(3)` and always performs a DNS query on the network, which is done asynchronously without using libuv's threadpool.

**Beginner-Friendly Explanation:** `dns.lookup()` is like asking the local phone book operator — they might have a cached answer or use the OS's built-in directory. It's convenient but can get slow if many people ask at once (because there are only 4 operators). `dns.resolve()` is like calling the DNS company directly — it's more explicit, faster for bulk queries, and doesn't tie up the local operators.

#### Purposes

- To convert human-readable domain names into IP addresses.
- To resolve specific DNS record types (MX, TXT, SRV, etc.).
- To perform reverse DNS lookups (IP → hostname).
- To choose the right resolution method for performance-critical applications.

#### Syntax Rules and Structure

**`dns.lookup()`:**
```js
const dns = require('node:dns');
dns.lookup('example.com', (err, address, family) => {
  console.log(address, family);
});
```

| Option | Description |
|--------|-------------|
| `family` | `4`, `6`, or `0` (both). |
| `hints` | `dns.ADDRCONFIG`, `dns.V4MAPPED`, `dns.ALL`. |
| `all` | If `true`, returns all addresses. |

**`dns.resolve()`:**
```js
dns.resolve('example.com', 'A', (err, records) => {
  console.log(records); // ['93.184.216.34']
});
```

| Record Type | Method |
|-------------|--------|
| `'A'` | `dns.resolve4()` |
| `'AAAA'` | `dns.resolve6()` |
| `'MX'` | `dns.resolveMx()` |
| `'TXT'` | `dns.resolveTxt()` |
| `'SRV'` | `dns.resolveSrv()` |

**Constraints and Limitations:**
- `dns.lookup()` runs on libuv's threadpool (default 4 threads); >4 parallel lookups block other threadpool operations (fs, crypto, zlib).
- `dns.resolve()` does not use `/etc/hosts` and may return different results than `dns.lookup()`.
- `dns.resolve()` requires an actual DNS server to be reachable.

#### Annotated Code Example

```js
// dns-resolution.js
const dns = require('node:dns');

// dns.lookup() — uses OS resolver (threadpool)
console.log('--- dns.lookup() ---');
dns.lookup('example.com', { all: true }, (err, addresses) => {
  if (err) throw err;
  console.log('Addresses:', addresses);
  // → [{ address: '93.184.216.34', family: 4 }, { address: '2606:2800:...', family: 6 }]
});

// dns.resolve4() — uses c-ares (no threadpool)
console.log('--- dns.resolve4() ---');
dns.resolve4('example.com', (err, addresses) => {
  if (err) throw err;
  console.log('IPv4 addresses:', addresses);
  // → ['93.184.216.34']
});

// dns.resolveMx() — specific record type
dns.resolveMx('gmail.com', (err, records) => {
  if (err) throw err;
  console.log('MX records:', records);
  // → [{ exchange: 'gmail-smtp-in.l.google.com', priority: 5 }, ...]
});
```

**Expected Output:**
```
--- dns.lookup() ---
Addresses: [ { address: '93.184.216.34', family: 4 }, ... ]
--- dns.resolve4() ---
IPv4 addresses: [ '93.184.216.34' ]
MX records: [ { exchange: 'gmail-smtp-in.l.google.com', priority: 5 }, ... ]
```

**Why this output:** `dns.lookup()` returns all addresses (IPv4 and IPv6) because `all: true` was specified. `dns.resolve4()` returns only IPv4 addresses as strings. `dns.resolveMx()` returns MX records with exchange and priority fields.

#### Real-World Cases

- **HTTP clients:** Resolving API hostnames before making requests.
- **Email servers:** Looking up MX records for SMTP delivery.
- **Service discovery:** Using SRV records to find microservice instances.
- **Security tools:** Reverse DNS lookups for logging and threat analysis.

---

### Sub-Feature 2.2: OS-Level Caching vs. Node's Thread-Pool DNS Behaviour

#### Definitions

**Core Definition:** `dns.lookup()` uses the operating system's caching and resolution facilities, while `dns.resolve()` performs network DNS queries that bypass OS caching. `dns.lookup()` runs on libuv's threadpool, which is limited to 4 threads by default.

**Technical Definition:** Under the hood, `dns.lookup()` uses the same operating system facilities as most other programs — for instance, it will almost always resolve a given name the same way as the `ping` command. On most POSIX-like operating systems, the behaviour of `dns.lookup()` can be modified by changing settings in `nsswitch.conf(5)` and/or `resolv.conf(5)`. Though the call to `dns.lookup()` will be asynchronous from JavaScript's perspective, it is implemented as a synchronous call to `getaddrinfo(3)` that runs on libuv's threadpool. `dns.resolve*()` functions do not use `getaddrinfo(3)` and always perform a DNS query on the network; this network communication is always asynchronous and does not use libuv's threadpool. As a result, `dns.resolve*()` functions cannot have the same negative impact on other processing that happens on libuv's threadpool that `dns.lookup()` can have.

**Beginner-Friendly Explanation:** Imagine you have 4 assistants (threadpool threads). Every time you ask `dns.lookup()` to find an address, one assistant has to stop what they're doing and call the OS phone book. If you make 5 lookup requests at once, one has to wait — and so does every file read or crypto operation that also needs an assistant. `dns.resolve()` doesn't use assistants at all; it makes the call itself, so the assistants stay free for other work.

#### Purposes

- To understand the performance implications of DNS resolution on the threadpool.
- To choose the appropriate DNS method for high-throughput applications.
- To avoid blocking the threadpool with DNS lookups.
- To leverage OS-level caching for frequently resolved domains.

#### Syntax Rules and Structure

| Aspect | `dns.lookup()` | `dns.resolve()` |
|--------|----------------|-----------------|
| Resolution method | `getaddrinfo(3)` (OS) | c-ares (DNS protocol) |
| Threadpool | Yes (blocks libuv threadpool) | No (network I/O) |
| `/etc/hosts` | Yes | No |
| OS caching | Yes | No |
| Parallel limit | 4 (default threadpool) | Unlimited |
| Best for | General use, OS consistency | High-performance, bulk queries |

**Constraints and Limitations:**
- The default libuv threadpool size is 4 (`UV_THREADPOOL_SIZE` environment variable can change it, up to 1024).
- `dns.lookup()` blocking the threadpool affects file I/O, crypto, and zlib operations.
- `dns.resolve()` may not respect `/etc/hosts` entries, causing inconsistencies.
- Some networking APIs (e.g., `socket.connect()`) allow the default resolver to be replaced.

#### Annotated Code Example

```js
// dns-threadpool.js
const dns = require('node:dns');

// Demonstrate the threadpool limit with dns.lookup()
console.log('--- dns.lookup() with parallel requests ---');
const hosts = ['example.com', 'google.com', 'github.com', 'nodejs.org', 'npmjs.com'];

let completed = 0;
const start = Date.now();

hosts.forEach((host, i) => {
  dns.lookup(host, (err, address) => {
    if (err) {
      console.error(`lookup ${host} failed:`, err.message);
    } else {
      console.log(`lookup ${host}: ${address} (${Date.now() - start}ms)`);
    }
    completed++;
  });
});

// With only 4 threads, one lookup waits for the others
// The 5th lookup will take noticeably longer

// dns.resolve() does not use the threadpool
setTimeout(() => {
  console.log('--- dns.resolve4() with parallel requests ---');
  hosts.forEach((host) => {
    dns.resolve4(host, (err, addresses) => {
      if (err) {
        console.error(`resolve ${host} failed:`, err.message);
      } else {
        console.log(`resolve ${host}: ${addresses[0]}`);
      }
    });
  });
}, 2000);
```

**Expected Output (varies by machine):**
```
--- dns.lookup() with parallel requests ---
lookup example.com: 93.184.216.34 (45ms)
lookup google.com: 142.250.72.14 (48ms)
lookup github.com: 140.82.112.4 (52ms)
lookup nodejs.org: 104.20.42.25 (55ms)
lookup npmjs.com: 104.16.26.35 (110ms)  ← 5th lookup waits for a thread
--- dns.resolve4() with parallel requests ---
resolve example.com: 93.184.216.34
resolve google.com: 142.250.72.14
resolve github.com: 140.82.112.4
resolve nodejs.org: 104.20.42.25
resolve npmjs.com: 104.16.26.35
```

**Why this output:** The first four `dns.lookup()` calls complete in parallel (using the 4 threadpool threads). The fifth call waits for a thread to become available, so it takes significantly longer. `dns.resolve4()` calls do not use the threadpool and all complete in parallel.

#### Real-World Cases

- **High-throughput API clients:** Using `dns.resolve()` to avoid threadpool contention.
- **HTTP servers:** Understanding why `dns.lookup()` can cause latency spikes under load.
- **Containerised applications:** Adjusting `UV_THREADPOOL_SIZE` for DNS-heavy workloads.
- **Caching DNS resolvers:** Using OS-level caching for frequently accessed domains.

---

## Core Concept 3: Network Security & Encryption

### Sub-Feature 3.1: Cryptographic Handshakes, Certificates, and TLS/SSL Architecture

#### Definitions

**Core Definition:** TLS (Transport Layer Security) is a cryptographic protocol that provides secure communication over a computer network by encrypting data and authenticating endpoints using digital certificates.

**Technical Definition:** The `node:tls` module provides an implementation of the Transport Layer Security (TLS) and Secure Socket Layer (SSL) protocols built on top of OpenSSL. TLS/SSL protocols rely on a public key infrastructure (PKI) to enable secure communication between client and server. Key components include: private keys (secret keys used for decryption and signing), certificates (public keys signed by a Certificate Authority), and Certificate Authorities (trusted entities that sign certificates). TLS uses a handshake protocol to negotiate cipher suites, authenticate the server (and optionally the client), and establish a shared session key. Perfect Forward Secrecy (PFS) ensures that session keys are not compromised even if the server's private key is compromised; ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) is enabled by default.

**Beginner-Friendly Explanation:** TLS is like sealing a letter in a tamper-proof envelope. The certificate is like a notarised ID card that proves the server is who it claims to be. The handshake is the process of checking IDs, agreeing on a secret code, and then sealing the envelope. Once the handshake is done, all data is encrypted and no one can read it in transit.

#### Purposes

- To encrypt data in transit, preventing eavesdropping.
- To authenticate the server (and optionally the client) using certificates.
- To ensure data integrity (detecting tampering).
- To provide perfect forward secrecy, protecting past sessions even if the private key is compromised.

#### Syntax Rules and Structure

**`tls.createServer()`:**
```js
const tls = require('node:tls');
const server = tls.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
  ca: [fs.readFileSync('ca-cert.pem')],
  requestCert: true,
  rejectUnauthorized: true,
}, (socket) => { /* handle secure connection */ });
```

| Option | Description |
|--------|-------------|
| `key` | Private key in PEM format. |
| `cert` | Certificate in PEM format. |
| `ca` | Array of trusted CA certificates. |
| `requestCert` | Request client certificate. Default: `false`. |
| `rejectUnauthorized` | Reject unauthorized clients. Default: `true`. |
| `minVersion` | Minimum TLS version (e.g., `'TLSv1.2'`). |
| `maxVersion` | Maximum TLS version (e.g., `'TLSv1.3'`). |
| `ciphers` | Cipher suite specification. |
| `dhparam` | Diffie-Hellman parameters (`'auto'` recommended). |
| `ecdhCurve` | ECDH curves to use. |
| `ALPNProtocols` | ALPN protocols (e.g., `['h2', 'http/1.1']`). |

**Constraints and Limitations:**
- TLS requires a valid certificate chain; self-signed certificates are not trusted by browsers.
- `rejectUnauthorized: false` disables certificate validation and is a security risk.
- TLS 1.0 and 1.1 are deprecated; use TLS 1.2 or 1.3.
- Certificate expiration must be monitored.

#### Annotated Code Example

```js
// tls-server.js
const tls = require('node:tls');
const fs = require('node:fs');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
  minVersion: 'TLSv1.2',
  maxVersion: 'TLSv1.3',
};

const server = tls.createServer(options, (socket) => {
  console.log('TLS connection established');
  console.log('Cipher:', socket.getCipher());
  console.log('Protocol:', socket.getProtocol());
  console.log('Authorized:', socket.authorized);

  socket.write('Hello from TLS server!\n');
  socket.pipe(socket); // Echo
});

server.listen(8443, () => {
  console.log('TLS server listening on port 8443');
});
```

```js
// tls-client.js
const tls = require('node:tls');
const fs = require('node:fs');

const options = {
  ca: [fs.readFileSync('ca-cert.pem')], // Trust our CA
  rejectUnauthorized: true,
};

const client = tls.connect(8443, 'localhost', options, () => {
  console.log('TLS handshake complete');
  console.log('Authorized:', client.authorized);
  client.write('Hello from TLS client!\n');
});

client.on('data', (data) => {
  console.log('Server response:', data.toString().trim());
  client.end();
});

client.on('error', (err) => {
  console.error('TLS error:', err.message);
});
```

**Expected Output (server):**
```
TLS server listening on port 8443
TLS connection established
Cipher: { name: 'ECDHE-RSA-AES256-GCM-SHA384', version: 'TLSv1.2' }
Protocol: TLSv1.2
Authorized: false
```

**Expected Output (client):**
```
TLS handshake complete
Authorized: true
Server response: Hello from TLS server!
```

**Why this output:** The TLS handshake negotiates the cipher suite and protocol version. The server's `socket.authorized` is `false` because the client did not present a certificate (mutual TLS is not configured). The client's `authorized` is `true` because it validated the server's certificate against the trusted CA.

#### Real-World Cases

- **HTTPS servers:** TLS termination for web applications.
- **Database connections:** Encrypted connections to PostgreSQL, MySQL, etc.
- **Message queues:** TLS-secured communication with Kafka, RabbitMQ.
- **API gateways:** TLS termination at the edge.

---

### Sub-Feature 3.2: Implementing Secure TCP Communication Using the Native `tls` Module

#### Definitions

**Core Definition:** Secure TCP communication is achieved by wrapping a TCP socket in a TLS layer using `tls.createServer()` (server-side) and `tls.connect()` (client-side).

**Technical Definition:** `tls.createServer([options][, secureConnectionListener])` creates a TLS server. The `secureConnectionListener` is called with a `tls.TLSSocket` object, which extends `net.Socket` and adds TLS-specific methods such as `getCipher()`, `getProtocol()`, `getPeerCertificate()`, and `authorized`. `tls.connect(options[, callback])` creates a TLS client connection and returns a `tls.TLSSocket`.

**Beginner-Friendly Explanation:** A TLS socket is like a regular TCP socket but with a secure tunnel built around it. You can use it exactly like a TCP socket — `write()`, `'data'` events, `pipe()` — but everything that goes through it is encrypted.

#### Purposes

- To add encryption to existing TCP-based protocols.
- To authenticate servers and clients using certificates.
- To leverage ALPN for protocol negotiation (HTTP/2, etc.).
- To implement secure custom protocols.

#### Syntax Rules and Structure

**`tls.connect()`:**
```js
const client = tls.connect(port, host, options, callback);
```

| Option | Description |
|--------|-------------|
| `host` | Host to connect to. |
| `port` | Port to connect to. |
| `ca` | Trusted CA certificates. |
| `cert` | Client certificate. |
| `key` | Client private key. |
| `rejectUnauthorized` | Validate server certificate. Default: `true`. |
| `servername` | SNI server name. |
| `ALPNProtocols` | ALPN protocols. |

**TLS socket methods:**
| Method | Description |
|--------|-------------|
| `socket.getCipher()` | Returns the negotiated cipher. |
| `socket.getProtocol()` | Returns the TLS protocol version. |
| `socket.getPeerCertificate()` | Returns the peer's certificate. |
| `socket.authorized` | Whether the peer is authorized. |
| `socket.authorizationError` | Reason for authorization failure. |

**Constraints and Limitations:**
- `tls.connect()` does not work with `net.createConnection()`; it creates its own socket.
- TLS handshake adds latency (1-2 round trips).
- Session resumption can reduce handshake overhead for repeated connections.

#### Annotated Code Example

```js
// tls-secure-echo.js
const tls = require('node:tls');
const fs = require('node:fs');

// Server
const server = tls.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
}, (socket) => {
  console.log('Secure connection established');
  socket.setEncoding('utf8');
  socket.on('data', (data) => {
    console.log('Received:', data.trim());
    socket.write(`Secure echo: ${data}`);
  });
});

server.listen(8443, () => {
  console.log('TLS echo server on port 8443');

  // Client
  const client = tls.connect(8443, 'localhost', {
    rejectUnauthorized: false, // For self-signed certs in dev
  }, () => {
    console.log('Client connected securely');
    client.write('Secret message\n');
  });

  client.setEncoding('utf8');
  client.on('data', (data) => {
    console.log('Client received:', data.trim());
    client.end();
  });
});
```

**Expected Output:**
```
TLS echo server on port 8443
Client connected securely
Secure connection established
Received: Secret message
Client received: Secure echo: Secret message
```

**Why this output:** The TLS handshake completes before any application data is exchanged. The client sends an encrypted message, the server decrypts and echoes it back, and the client receives the response. The `rejectUnauthorized: false` setting is used only for development with self-signed certificates.

#### Real-World Cases

- **Secure chat applications:** Encrypted messaging over TLS.
- **IoT device communication:** TLS-secured MQTT.
- **Database proxies:** TLS termination for database connections.
- **Custom secure protocols:** Building proprietary encrypted protocols.

---

### Sub-Feature 3.3: Managing CAs (Certificate Authorities) and Mutual TLS (mTLS) Authentication

#### Definitions

**Core Definition:** Mutual TLS (mTLS) is a TLS configuration where both the server and the client present certificates to authenticate each other, using a shared Certificate Authority (CA) to validate certificates.

**Technical Definition:** In mTLS, the server is configured with `requestCert: true` and `rejectUnauthorized: true` (or `false` for optional client certs), and the client is configured with a `cert` and `key`. Both sides trust the same CA (or a chain of CAs). The server can then inspect `socket.authorized` and `socket.getPeerCertificate()` to verify the client's identity. mTLS is commonly used in zero-trust architectures, service meshes, and API security.

**Beginner-Friendly Explanation:** Regular TLS is like a bouncer checking your ID at a club — the club proves it's legitimate, but you don't have to prove who you are. mTLS is like a high-security building where both you and the building have to show ID. The building proves it's the real building, and you prove you're an authorized visitor.

#### Purposes

- To authenticate both ends of a connection (zero-trust security).
- To prevent unauthorized clients from connecting to a server.
- To implement service-to-service authentication in microservices.
- To secure APIs without relying on tokens or passwords.

#### Syntax Rules and Structure

**Server-side mTLS:**
```js
const server = tls.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
  ca: [fs.readFileSync('ca-cert.pem')],
  requestCert: true,
  rejectUnauthorized: true,
}, (socket) => {
  if (socket.authorized) {
    console.log('Client certificate valid');
  } else {
    console.log('Client certificate invalid:', socket.authorizationError);
  }
});
```

**Client-side mTLS:**
```js
const client = tls.connect(8443, 'localhost', {
  ca: [fs.readFileSync('ca-cert.pem')],
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  rejectUnauthorized: true,
});
```

| Option | Server | Client |
|--------|--------|--------|
| `ca` | Trusted CA for client certs. | Trusted CA for server cert. |
| `cert` | Server certificate. | Client certificate. |
| `key` | Server private key. | Client private key. |
| `requestCert` | `true` to request client cert. | N/A |
| `rejectUnauthorized` | `true` to reject invalid clients. | `true` to reject invalid servers. |

**Constraints and Limitations:**
- Both sides must trust the same CA (or a chain of CAs).
- Client certificates must be distributed securely.
- `rejectUnauthorized: true` requires valid certificates; use `false` only for development.
- Certificate rotation requires careful coordination.

#### Annotated Code Example

```js
// mtls-server.js
const tls = require('node:tls');
const fs = require('node:fs');

const server = tls.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
  ca: [fs.readFileSync('ca-cert.pem')],
  requestCert: true,
  rejectUnauthorized: true,
}, (socket) => {
  const cert = socket.getPeerCertificate();

  if (socket.authorized) {
    console.log('Authorized client:', cert.subject.CN);
    socket.write(`Welcome, ${cert.subject.CN}!\n`);
  } else {
    console.log('Unauthorized client:', socket.authorizationError);
    socket.write('Unauthorized\n');
  }

  socket.on('data', (data) => {
    console.log(`[${cert.subject.CN}] ${data.toString().trim()}`);
  });
});

server.listen(8443, () => {
  console.log('mTLS server on port 8443');
});
```

```js
// mtls-client.js
const tls = require('node:tls');
const fs = require('node:fs');

const client = tls.connect(8443, 'localhost', {
  ca: [fs.readFileSync('ca-cert.pem')],
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  rejectUnauthorized: true,
}, () => {
  console.log('mTLS handshake complete');
  console.log('Authorized:', client.authorized);

  const cert = client.getPeerCertificate();
  console.log('Server CN:', cert.subject.CN);

  client.write('Hello from mTLS client!\n');
});

client.on('data', (data) => {
  console.log('Server:', data.toString().trim());
  client.end();
});

client.on('error', (err) => {
  console.error('mTLS error:', err.message);
});
```

**Expected Output (server):**
```
mTLS server on port 8443
Authorized client: my-client
[my-client] Hello from mTLS client!
```

**Expected Output (client):**
```
mTLS handshake complete
Authorized: true
Server CN: my-server
Server: Welcome, my-client!
```

**Why this output:** Both the server and client present certificates signed by the same CA. The server validates the client's certificate (`socket.authorized` is `true`) and extracts the Common Name (`my-client`). The client validates the server's certificate and extracts the server's CN (`my-server`). Both sides are authenticated.

#### Real-World Cases

- **Service mesh:** mTLS between microservices (Istio, Linkerd).
- **API security:** Zero-trust API authentication without tokens.
- **IoT:** Device authentication using client certificates.
- **Banking:** Mutual authentication for high-security transactions.

---

## Core Concept 4: Connection Lifecycles

### Sub-Feature 4.1: Keep-Alive Mechanisms and HTTP Connection Reuse Optimisation

#### Definitions

**Core Definition:** Keep-alive is a mechanism that allows a single TCP connection to be reused for multiple HTTP requests, reducing the overhead of establishing a new connection for each request.

**Technical Definition:** An `Agent` is responsible for managing connection persistence and reuse for HTTP clients. It maintains a queue of pending requests for a given host and port, reusing a single socket connection for each until the queue is empty, at which time the socket is either destroyed or put into a pool where it is kept to be used again for requests to the same host and port. Whether it is destroyed or pooled depends on the `keepAlive` option. Pooled connections have TCP Keep-Alive enabled for them, but servers may still close idle connections, in which case they will be removed from the pool and a new connection will be made when a new HTTP request is made for that host and port. The `socket.setKeepAlive([enable][, initialDelay])` method enables/disables keep-alive functionality and optionally sets the initial delay before the first keepalive probe is sent on an idle socket.

**Beginner-Friendly Explanation:** Keep-alive is like keeping a phone call open instead of hanging up and redialing for every sentence. The HTTP Agent is the phone operator who decides when to hang up (destroy the socket) and when to keep the line open (pool the socket) for the next call.

#### Purposes

- To reduce latency by avoiding TCP handshake overhead for repeated requests.
- To reduce CPU usage on both client and server by reusing connections.
- To improve throughput for high-volume HTTP clients.
- To detect dead connections using TCP keep-alive probes.

#### Syntax Rules and Structure

**HTTP Agent with keep-alive:**
```js
const http = require('node:http');

const agent = new http.Agent({
  keepAlive: true,
  keepAliveMsecs: 1000,  // Initial delay for TCP keep-alive probes
  maxSockets: 50,        // Max sockets per host
  maxFreeSockets: 10,    // Max idle sockets to keep
  timeout: 60000,        // Socket timeout
});

http.get({ hostname: 'example.com', agent }, (res) => {
  // ...
});
```

| Option | Description | Default |
|--------|-------------|---------|
| `keepAlive` | Keep sockets around for reuse. | `false` |
| `keepAliveMsecs` | Initial delay for TCP keep-alive probes. | 1000 |
| `maxSockets` | Maximum sockets per host. | Infinity |
| `maxFreeSockets` | Maximum idle sockets to keep. | 256 |
| `timeout` | Socket timeout in milliseconds. | — |

**Socket-level keep-alive:**
```js
socket.setKeepAlive(true, 1000);
```

| Parameter | Description |
|-----------|-------------|
| `enable` | Enable/disable keep-alive. |
| `initialDelay` | Delay before first keepalive probe (ms). |

**Constraints and Limitations:**
- `keepAliveMsecs` values below 1000 ms are silently truncated to 1000 ms on some platforms.
- Servers may close idle connections; the Agent handles this by removing them from the pool.
- `agent: false` disables connection reuse for a single request.
- Pooled sockets are `unref()`ed so they don't keep the process alive.

#### Annotated Code Example

```js
// keep-alive.js
const http = require('node:http');

// Create a keep-alive agent
const agent = new http.Agent({
  keepAlive: true,
  keepAliveMsecs: 1000,
  maxSockets: 10,
  maxFreeSockets: 5,
});

// Make multiple requests to the same host
const urls = [
  'http://example.com/',
  'http://example.com/',
  'http://example.com/',
];

let completed = 0;

urls.forEach((url) => {
  const req = http.get(url, { agent }, (res) => {
    console.log(`Response ${++completed}: ${res.statusCode}`);
    res.resume(); // Consume response

    res.on('end', () => {
      if (completed === urls.length) {
        console.log('All requests completed with connection reuse.');
        agent.destroy(); // Clean up
      }
    });
  });

  // Check if the socket was reused
  req.on('socket', (socket) => {
    console.log('Socket reused:', socket.reused || false);
  });
});
```

**Expected Output:**
```
Socket reused: false
Response 1: 200
Socket reused: true
Response 2: 200
Socket reused: true
Response 3: 200
All requests completed with connection reuse.
```

**Why this output:** The first request creates a new socket (not reused). The second and third requests reuse the pooled socket from the first request. The `agent.destroy()` call cleans up the pool when all requests are complete.

#### Real-World Cases

- **API clients:** Reusing connections to a backend API for high-throughput calls.
- **Web scrapers:** Managing connections to multiple domains with per-host pools.
- **Microservice gateways:** Tuning connection pools to match backend capacity.
- **Server-to-server communication:** Reducing latency in distributed systems.

---

## References

- Node.js Documentation — Net — https://nodejs.org/api/net.html
- Node.js Documentation — `net.createServer()` — https://nodejs.org/api/net.html#netcreateserveroptions-connectionlistener
- Node.js Documentation — `net.Socket` — https://nodejs.org/api/net.html#class-netsocket
- Node.js Documentation — DNS — https://nodejs.org/api/dns.html
- Node.js Documentation — DNS Implementation Considerations — https://nodejs.org/api/dns.html#implementation-considerations
- Node.js Documentation — `dns.lookup()` — https://nodejs.org/api/dns.html#dnslookuphostname-options-callback
- Node.js Documentation — `dns.resolve()` — https://nodejs.org/api/dns.html#dnsresolvehostname-rrtype-callback
- Node.js Documentation — TLS (SSL) — https://nodejs.org/api/tls.html
- Node.js Documentation — `tls.createServer()` — https://nodejs.org/api/tls.html#tlscreateserveroptions-secureconnectionlistener
- Node.js Documentation — `tls.connect()` — https://nodejs.org/api/tls.html#tlsconnectoptions-callback
- Node.js Documentation — `http.Agent` — https://nodejs.org/api/http.html#class-httpagent
- Node.js Documentation — `socket.setKeepAlive()` — https://nodejs.org/api/net.html#socketsetkeepaliveenable-initialdelay
- Node.js Documentation — `UV_THREADPOOL_SIZE` — https://nodejs.org/api/cli.html#uv_threadpool_sizesize
- RFC 793 — Transmission Control Protocol — https://www.rfc-editor.org/rfc/rfc793
- RFC 9293 — Transmission Control Protocol (TCP) — https://www.rfc-editor.org/rfc/rfc9293
- RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3 — https://www.rfc-editor.org/rfc/rfc8446
- RFC 5246 — The Transport Layer Security (TLS) Protocol Version 1.2 — https://www.rfc-editor.org/rfc/rfc5246
- DeepWiki — TCP/IP and Socket Programming — https://deepwiki.com/ElemeFE/node-interview/5.1-tcpip-and-socket-programming
- Node.js Stream Processing for Large Datasets — Grizzly Peak Software — https://grizzlypeaksoftware.com/library/nodejs-stream-processing-for-large-datasets-efm75efe
- DNS Queries in Node.js: dns.lookup vs dns.resolve Explained — https://dnschkr.com
- How to handle EADDRINUSE errors — Socket.IO — https://socket.io/how-to/handle-eaddrinuse
- Node.js Socket Options: Keep-Alive, Nagle, and Backlog — https://www.thenodebook.com
- Mutual TLS (mTLS) in Node.js — https://shattered.io