# Node.js Operating System (`os`) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `node:os` module provides operating system-related utility methods and properties, allowing Node.js programs to query low-level details about the host environment.

**Technical Definition:** The `node:os` module provides operating system-related utility methods and properties. It can be accessed using `import os from 'node:os'` or `const os = require('node:os')`. It exposes functions for CPU architecture, memory statistics, platform identification, host details, and system load, with a subset of methods being thin wrappers around libuv's cross-platform abstractions.

**Beginner-Friendly Explanation:** Think of `os` as Node.js's "system information panel." Just like your computer's settings app shows you how much RAM you have, how many CPU cores are available, and what operating system you're running, the `os` module lets your JavaScript code ask those same questions — so your program can adapt its behaviour based on the machine it's running on.

### Key Characteristics

- **Cross-platform:** Abstracts over POSIX and Windows differences, returning consistent data structures.
- **Synchronous and immediate:** All methods are synchronous and return values immediately; there is no callback or Promise-based variant.
- **Read-only:** The module provides information about the system but does not modify any system settings.
- **Implementation-dependent:** Some methods (notably `os.loadavg()`) return meaningful values only on POSIX systems; Windows returns `[0, 0, 0]`.
- **Thin wrapper over libuv:** Many methods delegate to libuv's cross-platform C APIs, ensuring consistent behaviour.

### Prerequisites

- **Node.js runtime:** The `os` module is built into Node.js; no external installation is required.
- **Basic JavaScript knowledge:** Understanding of objects, arrays, and console output.
- **Familiarity with `require`/`import`:** Knowing how to import built-in Node.js modules.
- **Basic understanding of hardware concepts:** CPU cores, memory, operating systems.

### Related Programming Areas

- **Process management:** Combining `os` with `process` to build monitoring tools.
- **Cluster and worker threads:** Using `os.availableParallelism()` to decide worker pool sizes.
- **Performance monitoring:** Tracking memory and load averages for health checks.
- **Cross-platform applications:** Detecting the OS to choose platform-specific code paths.
- **Network utilities:** Inspecting network interfaces for server binding decisions.

### Core Concepts

The following core concepts are covered in this cheat sheet:

1. **OS API Utilities** — the module import and general usage.
2. **CPU Architecture, Core Counts, and Usage** — `os.arch()`, `os.cpus()`, `os.availableParallelism()`.
3. **System Memory** — `os.totalmem()`, `os.freemem()`.
4. **Platform Information** — `os.platform()`, `os.type()`, `os.release()`.
5. **Host Information** — `os.hostname()`, `os.homedir()`, `os.tmpdir()`, `os.networkInterfaces()`.
6. **System Uptime and Load Averages** — `os.uptime()`, `os.loadavg()`.

---

## Core Concept 1: OS API Utilities

### Definitions

**Core Definition:** The `node:os` module is a built-in Node.js module that provides operating system-related utility methods and properties.

**Technical Definition:** The `node:os` module provides operating system-related utility methods and properties. It can be accessed using `import os from 'node:os'` (ESM) or `const os = require('node:os')` (CommonJS). The module has been stable since Node.js v0.10.0 and exposes both methods (functions) and properties (values such as `os.EOL` and `os.constants`).

**Beginner-Friendly Explanation:** You import `os` just like any other Node.js module. Once imported, it's a toolbox of functions you can call to ask your computer questions: "How much memory is free?", "What OS are you running?", "How many CPU cores do you have?"

### Purposes

- To query the host operating system's configuration and state at runtime.
- To enable adaptive behaviour based on the platform, CPU count, or available memory.
- To provide diagnostic information for logging and monitoring.
- To support cross-platform application logic without hardcoding platform assumptions.

### Syntax Rules and Structure

**CommonJS import:**
```js
const os = require('node:os');
```
| Component | Breakdown |
|-----------|-----------|
| `require` | CommonJS import function. |
| `'node:os'` | Built-in module specifier (the `node:` prefix is recommended). |
| Returns | The `os` module object. |

**ESM import:**
```js
import os from 'node:os';
```
| Component | Breakdown |
|-----------|-----------|
| `import os` | Default import of the os module. |
| `from 'node:os'` | Module specifier. |

**Key properties:**
| Property | Description |
|----------|-------------|
| `os.EOL` | The OS-specific end-of-line marker (`\n` on POSIX, `\r\n` on Windows). |
| `os.constants` | Commonly used OS-specific constants (error codes, signals). |
| `os.devNull` | The platform-specific null device path (`/dev/null` or `\\.\nul`). |

**Constraints and Limitations:**
- All `os` methods are synchronous and return values immediately.
- Some values (e.g., `os.freemem()`) may be misleading on certain platforms; on Linux, `os.freemem()` reports only completely free memory, not "available" memory.
- The module does not modify system state; it is purely informational.

### Annotated Code Example

```js
// os-import.js
const os = require('node:os');

// Basic import check
console.log('os module loaded:', typeof os === 'object');
console.log('End-of-line marker:', JSON.stringify(os.EOL));
console.log('Null device:', os.devNull);
console.log('Platform:', os.platform());
```

**Expected Output (POSIX):**
```
os module loaded: true
End-of-line marker: "\n"
Null device: /dev/null
Platform: linux
```

**Why this output:** The `os` module is imported and its properties are accessed directly. `os.EOL` is `\n` on POSIX and `\r\n` on Windows. `os.devNull` returns the platform's null device path. This confirms the module is loaded and the platform is correctly detected.

### Real-World Cases

- **Cross-platform line endings:** Using `os.EOL` when writing files that must have native line endings.
- **Signal handling:** Using `os.constants.signals.SIGTERM` for process signal constants.
- **Startup diagnostics:** Logging `os.platform()` and `os.arch()` at application startup for debugging.

---

## Core Concept 2: CPU Architecture, Core Counts, and Usage Information

### Definitions

**Core Definition:** These methods report the CPU architecture the Node.js binary was compiled for, the number of logical CPU cores, and per-core usage statistics.

**Technical Definition:** `os.arch()` returns a string identifying the operating system CPU architecture for which the Node.js binary was compiled, with possible values including `'arm'`, `'arm64'`, `'ia32'`, `'loong64'`, `'mips'`, `'mipsel'`, `'ppc64'`, `'riscv64'`, `'s390x'`, and `'x64'`. `os.cpus()` returns an array of objects containing information about each logical CPU core, with properties including `model`, `speed` (in MHz), and `times` (an object with `user`, `nice`, `sys`, `idle`, and `irq` in milliseconds). `os.availableParallelism()` returns an estimate of the default amount of parallelism a program should use, always greater than zero.

**Beginner-Friendly Explanation:** `os.arch()` tells you what kind of processor your computer has (like "x64" for most modern Intel/AMD chips, or "arm64" for Apple Silicon). `os.cpus()` gives you a detailed report card for each CPU core — its model name, speed, and how much time it has spent doing different types of work. `os.availableParallelism()` gives you a simple number: how many things can your program do at once.

### Purposes

- To determine the CPU architecture for downloading or loading architecture-specific binaries.
- To count logical CPU cores for sizing worker thread pools or cluster processes.
- To monitor per-core CPU utilisation for performance diagnostics.
- To obtain a recommended parallelism level without counting cores manually.

### Syntax Rules and Structure

**`os.arch()`:**
```js
const arch = os.arch();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | A string: `'arm'`, `'arm64'`, `'ia32'`, `'x64'`, etc. |
| Equivalent | `process.arch`. |

**`os.cpus()`:**
```js
const cpus = os.cpus();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | An array of objects, one per logical CPU core. |
| `model` | CPU model string. |
| `speed` | Clock speed in MHz. |
| `times.user` | Milliseconds spent in user mode. |
| `times.nice` | Milliseconds spent in nice mode. |
| `times.sys` | Milliseconds spent in sys mode. |
| `times.idle` | Milliseconds spent idle. |
| `times.irq` | Milliseconds spent servicing interrupts. |

**`os.availableParallelism()`:**
```js
const parallelism = os.availableParallelism();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | An integer ≥ 1. |
| Added in | Node.js v18.14.0 / v19.4.0. |
| Wrapper | libuv's `uv_available_parallelism()`. |

**Constraints and Limitations:**
- `os.cpus()` returns an empty array if CPU information is unavailable (e.g., `/proc` not mounted).
- `os.cpus().length` should not be used to calculate available parallelism; use `os.availableParallelism()` instead, as CPU affinity and cgroup limits may reduce the effective count.
- `nice` times are always `0` on Windows.

### Multiple Annotated Code Examples

#### Example 1: Inspecting CPU Information

```js
// cpu-info.js
const os = require('node:os');

// Architecture
console.log('Architecture:', os.arch());

// Number of logical CPU cores
const cpus = os.cpus();
console.log('Logical CPU cores:', cpus.length);

// Details of the first core
if (cpus.length > 0) {
  const first = cpus[0];
  console.log('Model:', first.model);
  console.log('Speed:', first.speed, 'MHz');
  console.log('Times:', JSON.stringify(first.times));
}

// Recommended parallelism
console.log('Available parallelism:', os.availableParallelism());
```

**Expected Output (varies by machine):**
```
Architecture: x64
Logical CPU cores: 8
Model: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
Speed: 2592 MHz
Times: {"user":123456,"nice":0,"sys":78901,"idle":9876543,"irq":0}
Available parallelism: 8
```

**Why this output:** `os.arch()` returns the architecture for which the Node.js binary was compiled (here `x64`). `os.cpus()` returns one object per logical core (8 in this example). The `times` object accumulates milliseconds spent in each mode since boot. `os.availableParallelism()` returns a recommended parallelism value.

#### Example 2: Calculating CPU Usage Percentage

```js
// cpu-usage.js
const os = require('node:os');

// Take a snapshot, wait, then take another to compute usage
function getCpuUsage() {
  const cpus = os.cpus();
  let totalIdle = 0, totalTick = 0;

  for (const cpu of cpus) {
    for (const type in cpu.times) {
      totalTick += cpu.times[type];
    }
    totalIdle += cpu.times.idle;
  }

  return { idle: totalIdle / cpus.length, total: totalTick / cpus.length };
}

const start = getCpuUsage();

setTimeout(() => {
  const end = getCpuUsage();
  const idleDiff = end.idle - start.idle;
  const totalDiff = end.total - start.total;
  const usage = 100 - (100 * idleDiff / totalDiff);
  console.log('CPU Usage: ~' + usage.toFixed(2) + '%');
}, 1000);
```

**Expected Output (varies):**
```
CPU Usage: ~12.34%
```

**Why this output:** CPU usage is calculated by comparing the change in idle and total tick counts over a one-second interval. A higher idle difference means lower usage. This is the same technique used by system monitoring tools.

### Real-World Cases

- **Worker pool sizing:** `const poolSize = os.availableParallelism();` to create an optimal number of worker threads.
- **Cluster forking:** `const numCPUs = os.cpus().length;` to fork one process per core in a cluster.
- **Architecture-specific downloads:** Selecting the correct prebuilt binary based on `os.arch()`.
- **Health checks:** Reporting CPU load in a `/health` endpoint of a monitoring service.

---

## Core Concept 3: System Memory (Total vs. Free Memory Tracking)

### Definitions

**Core Definition:** `os.totalmem()` returns the total amount of system memory in bytes, and `os.freemem()` returns the amount of free system memory in bytes.

**Technical Definition:** `os.totalmem()` returns the total amount of system memory in bytes. `os.freemem()` returns the amount of free system memory in bytes. On Linux, `os.freemem()` reports only completely free memory (as shown by `MemFree` in `/proc/meminfo`), not "available" memory (which includes reclaimable buffers and cache).

**Beginner-Friendly Explanation:** `os.totalmem()` tells you how much RAM your computer has in total. `os.freemem()` tells you how much of it is currently sitting unused. Think of it as looking at a car's fuel gauge: totalmem is the tank size, freemem is how much fuel is left.

### Purposes

- To monitor system memory pressure for health checks.
- To decide whether to proceed with memory-intensive operations.
- To log memory statistics for diagnostics and capacity planning.
- To calculate memory usage percentages for dashboards.

### Syntax Rules and Structure

**`os.totalmem()`:**
```js
const total = os.totalmem();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | Total system memory in bytes (integer). |

**`os.freemem()`:**
```js
const free = os.freemem();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | Free system memory in bytes (integer). |

**Constraints and Limitations:**
- On Linux, `os.freemem()` may report a much lower value than what system monitors show because it excludes reclaimable cache.
- On macOS, `os.freemem()` may not reflect "available" memory accurately.
- Values are in bytes; conversion to GB/MB is the caller's responsibility.
- The value changes constantly and is a snapshot at the moment of the call.

### Annotated Code Example

```js
// memory-info.js
const os = require('node:os');

// Convert bytes to a human-readable string
function formatBytes(bytes) {
  const units = ['B', 'KiB', 'MiB', 'GiB', 'TiB'];
  let i = 0;
  while (bytes >= 1024 && i < units.length - 1) {
    bytes /= 1024;
    i++;
  }
  return bytes.toFixed(2) + ' ' + units[i];
}

const total = os.totalmem();
const free = os.freemem();
const used = total - free;

console.log('Total memory:', formatBytes(total));
console.log('Free memory: ', formatBytes(free));
console.log('Used memory: ', formatBytes(used));
console.log('Usage: ' + ((used / total) * 100).toFixed(1) + '%');
```

**Expected Output (varies by machine):**
```
Total memory: 15.98 GiB
Free memory:  2.34 GiB
Used memory:  13.64 GiB
Usage: 85.3%
```

**Why this output:** `os.totalmem()` and `os.freemem()` return byte counts, which are converted to gibibytes (GiB) using 1024-based division. The "used" value is derived by subtracting free from total. On Linux, the "free" value may be lower than what system monitors report because it excludes cache memory.

### Real-World Cases

- **Memory guard rails:** Refusing to start a memory-intensive job if `os.freemem() < threshold`.
- **Monitoring dashboards:** Reporting memory usage alongside CPU load.
- **Container awareness:** Logging memory stats to detect when a container is approaching its limit.
- **Graceful degradation:** Switching to a lighter processing mode when free memory is low.

---

## Core Concept 4: Platform Information (OS Type, Release Version, and Platform String)

### Definitions

**Core Definition:** These methods identify the operating system: `os.platform()` for the platform identifier, `os.type()` for the OS name, and `os.release()` for the kernel version.

**Technical Definition:** `os.platform()` returns a string identifying the operating system platform for which the Node.js binary was compiled; the value is set at compile time and is equivalent to `process.platform`. Possible values include `'darwin'`, `'freebsd'`, `'linux'`, `'openbsd'`, `'sunos'`, and `'win32'`. `os.type()` returns the operating system name as returned by `uname` — for example, `'Linux'` on Linux, `'Darwin'` on macOS, and `'Windows_NT'` on Windows. `os.release()` returns the operating system release version string.

**Beginner-Friendly Explanation:** `os.platform()` tells you the broad family of the OS using a short keyword like `'linux'` or `'win32'` — useful for writing code that behaves differently on Windows versus Linux. `os.type()` gives a more human-readable name like `'Linux'` or `'Darwin'`. `os.release()` gives the specific version number of the OS kernel, like `'5.15.0-91-generic'`.

### Purposes

- To write platform-specific code paths (e.g., different path separators or shell commands).
- To log the OS version for support and debugging.
- To detect the OS family before installing platform-specific native modules.
- To display system information in a user-facing "About" dialog.

### Syntax Rules and Structure

**`os.platform()`:**
```js
const platform = os.platform();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | A string: `'darwin'`, `'freebsd'`, `'linux'`, `'openbsd'`, `'sunos'`, `'win32'`. |
| Equivalent | `process.platform`. |

**`os.type()`:**
```js
const type = os.type();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | `'Linux'`, `'Darwin'`, `'Windows_NT'`, etc. |

**`os.release()`:**
```js
const release = os.release();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | The OS release version string. |

**Constraints and Limitations:**
- `os.platform()` values are compile-time constants; they do not change at runtime.
- `os.type()` returns the kernel name, which may differ from the marketing name (e.g., `'Darwin'` for macOS).
- `os.release()` format varies by OS and is not standardised.

### Annotated Code Example

```js
// platform-info.js
const os = require('node:os');

console.log('platform():', os.platform());
console.log('type():    ', os.type());
console.log('release(): ', os.release());

// Conditional logic based on platform
if (os.platform() === 'win32') {
  console.log('Running on Windows — using Windows-specific behaviour.');
} else if (os.platform() === 'darwin') {
  console.log('Running on macOS — using macOS-specific behaviour.');
} else {
  console.log('Running on a POSIX system (Linux/BSD/Solaris).');
}
```

**Expected Output (Linux):**
```
platform(): linux
type():     Linux
release():  5.15.0-91-generic
Running on a POSIX system (Linux/BSD/Solaris).
```

**Expected Output (Windows):**
```
platform(): win32
type():     Windows_NT
release():  10.0.22631
Running on Windows — using Windows-specific behaviour.
```

**Why this output:** `os.platform()` returns the Node.js compile-time platform identifier (`'win32'` on Windows, `'linux'` on Linux). `os.type()` returns the kernel name from `uname`. `os.release()` returns the kernel version. The conditional demonstrates how to branch logic based on platform.

### Real-World Cases

- **Installing native dependencies:** Choosing the correct prebuilt binary for the platform.
- **Shell command selection:** Using `cmd.exe` on Windows and `/bin/sh` on POSIX.
- **Path handling:** Combining `os.platform()` with `path.sep` for platform-safe path construction.
- **Diagnostic logging:** Recording `os.type()` and `os.release()` at startup for bug reports.

---

## Core Concept 5: Host Information (Hostname, Home Directory, Temporary Directories, and Network Interfaces)

### Definitions

**Core Definition:** These methods provide information about the host machine: its network name, the current user's home directory, the system temporary directory, and the details of all network interfaces.

**Technical Definition:** `os.hostname()` returns the hostname of the operating system. `os.homedir()` returns the string path of the current user's home directory. `os.tmpdir()` returns the operating system's default directory for temporary files. `os.networkInterfaces()` returns an object containing network interfaces that have been assigned a network address, with each key being the interface name and each value an array of address objects.

**Beginner-Friendly Explanation:** `os.hostname()` tells you your computer's name on the network (like "my-laptop"). `os.homedir()` gives you your user folder (`/home/alice` or `C:\Users\Alice`). `os.tmpdir()` gives you a place to put temporary files that can be cleaned up later. `os.networkInterfaces()` lists all your network connections — Wi-Fi, Ethernet, localhost — and their IP addresses.

### Purposes

- To construct paths relative to the user's home directory without hardcoding usernames.
- To obtain a safe temporary directory for scratch files.
- To identify the host in distributed systems and logging.
- To enumerate network addresses for binding servers or discovering the local IP.

### Syntax Rules and Structure

**`os.hostname()`:**
```js
const hostname = os.hostname();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | The hostname string. |

**`os.homedir()`:**
```js
const home = os.homedir();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | The home directory path string. |

**`os.tmpdir()`:**
```js
const tmp = os.tmpdir();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | The temporary directory path string. |

**`os.networkInterfaces()`:**
```js
const interfaces = os.networkInterfaces();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | An object keyed by interface name. |
| Each entry | An array of address objects. |
| `address` | The IP address. |
| `netmask` | The subnet mask. |
| `family` | `'IPv4'` or `'IPv6'`. |
| `mac` | The MAC address. |
| `internal` | `true` if the interface is loopback. |

**Constraints and Limitations:**
- `os.homedir()` uses the `$HOME` environment variable on POSIX and `USERPROFILE` on Windows; if these are unset, it falls back to platform-specific lookups.
- `os.tmpdir()` returns a path but does not create the directory; it is assumed to exist.
- `os.networkInterfaces()` returns only interfaces that have at least one assigned address; interfaces without addresses are omitted.
- The order of interface keys and address arrays is not guaranteed.

### Annotated Code Example

```js
// host-info.js
const os = require('node:os');

// Basic host information
console.log('Hostname:', os.hostname());
console.log('Home directory:', os.homedir());
console.log('Temp directory:', os.tmpdir());

// Network interfaces
const interfaces = os.networkInterfaces();

for (const [name, addresses] of Object.entries(interfaces)) {
  for (const addr of addresses) {
    // Filter to IPv4 non-internal addresses for brevity
    if (addr.family === 'IPv4' && !addr.internal) {
      console.log(`[${name}] ${addr.address} (MAC: ${addr.mac})`);
    }
  }
}
```

**Expected Output (varies by machine):**
```
Hostname: dev-machine
Home directory: /home/alice
Temp directory: /tmp
[eth0] 192.168.1.42 (MAC: 06:00:00:02:0e:00)
[wlan0] 192.168.1.55 (MAC: 0a:00:00:11:22:33)
```

**Why this output:** `os.hostname()` returns the machine's network name. `os.homedir()` and `os.tmpdir()` return platform-specific paths. The network interface loop iterates over all interfaces and their addresses, filtering to show only external IPv4 addresses. Internal (loopback) interfaces are excluded.

### Real-World Cases

- **User configuration files:** Storing settings in `path.join(os.homedir(), '.myapp', 'config.json')`.
- **Temporary file creation:** Writing scratch files to `os.tmpdir()` for automatic cleanup.
- **Service discovery:** Using `os.hostname()` to identify the machine in a cluster.
- **Local IP detection:** Finding the machine's LAN IP for server binding or peer-to-peer communication.
- **Container hostname:** Using the hostname as a unique identifier in containerised deployments.

---

## Core Concept 6: System Uptime and Load Averages

### Definitions

**Core Definition:** `os.uptime()` returns the number of seconds the system has been running, while `os.loadavg()` returns an array of the 1, 5, and 15-minute load averages.

**Technical Definition:** `os.uptime()` returns the system uptime in seconds. `os.loadavg()` returns an array containing the 1, 5, and 15-minute load averages. The load average is a measure of system activity calculated by the operating system and expressed as a fractional number. As a rule of thumb, the load average should ideally be less than the number of logical CPUs in the system. The load average is a very UNIX-y concept; there is no real equivalent on Windows platforms, so the function always returns `[0, 0, 0]` on Windows.

**Beginner-Friendly Explanation:** `os.uptime()` tells you how long your computer has been running since it was last booted — like a stopwatch. `os.loadavg()` tells you how "busy" the computer is. A load average of 1.0 on a single-core machine means the CPU is fully busy. A load average of 4.0 on a 4-core machine means all cores are fully busy. If it goes above that, the system is overloaded.

### Purposes

- To monitor system health and detect overloaded machines.
- To log uptime for diagnostics (e.g., detecting unexpected reboots).
- To make scheduling decisions based on current system load.
- To display system status in monitoring dashboards.

### Syntax Rules and Structure

**`os.uptime()`:**
```js
const uptime = os.uptime();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | System uptime in seconds (integer). |

**`os.loadavg()`:**
```js
const loads = os.loadavg();
```
| Component | Breakdown |
|-----------|-----------|
| Returns | An array of three numbers: `[1min, 5min, 15min]`. |
| POSIX | Meaningful values. |
| Windows | Always `[0, 0, 0]`. |

**Constraints and Limitations:**
- `os.loadavg()` is only meaningful on POSIX systems (Linux, macOS, BSD); on Windows it always returns `[0, 0, 0]`.
- On Linux, load average includes processes waiting for I/O, not just CPU contention.
- The load average is a system-wide metric and is not affected by CPU affinity or cgroups.
- Values may differ slightly from the `uptime` command if sampled at different moments.

### Annotated Code Example

```js
// uptime-loadavg.js
const os = require('node:os');

// Uptime in seconds and human-readable
const uptimeSeconds = os.uptime();
const days = Math.floor(uptimeSeconds / 86400);
const hours = Math.floor((uptimeSeconds % 86400) / 3600);
const minutes = Math.floor((uptimeSeconds % 3600) / 60);
console.log(`System uptime: ${days}d ${hours}h ${minutes}m (${uptimeSeconds}s)`);

// Load averages
const [one, five, fifteen] = os.loadavg();
console.log('Load averages (1m, 5m, 15m):', one.toFixed(2), five.toFixed(2), fifteen.toFixed(2));

// Compare with CPU count
const cpus = os.availableParallelism();
console.log(`CPU cores available: ${cpus}`);
console.log(`1-minute load per core: ${(one / cpus).toFixed(2)}`);
```

**Expected Output (Linux, 8 cores):**
```
System uptime: 3d 14h 22m (311040s)
Load averages (1m, 5m, 15m): 2.34 3.12 3.45
CPU cores available: 8
1-minute load per core: 0.29
```

**Expected Output (Windows):**
```
System uptime: 1d 2h 15m (94320s)
Load averages (1m, 5m, 15m): 0.00 0.00 0.00
CPU cores available: 8
1-minute load per core: 0.00
```

**Why this output:** `os.uptime()` returns seconds, which are converted to a human-readable format. `os.loadavg()` returns the three load averages; on Linux these are meaningful, while on Windows they are always `[0, 0, 0]`. Dividing the 1-minute load by the CPU count gives a per-core load figure, which is more interpretable than the raw value.

### Real-World Cases

- **Health checks:** Reporting uptime and load average in a `/health` endpoint to detect overloaded instances.
- **Load balancer decisions:** Using load average to decide whether to accept new requests.
- **Diagnostic logging:** Recording uptime at startup to detect unexpected restarts.
- **Capacity planning:** Tracking load averages over time to decide when to scale horizontally.
- **Graceful shutdown:** Detecting system overload and shedding non-critical work.

---

## References

- Node.js Documentation — OS — https://nodejs.org/api/os.html
- Node.js Documentation — `os.arch()` — https://nodejs.org/api/os.html#osarch
- Node.js Documentation — `os.availableParallelism()` — https://nodejs.org/api/os.html#osavailableparallelism
- Node.js Documentation — `os.cpus()` — https://nodejs.org/api/os.html#oscpus
- Node.js Documentation — `os.freemem()` — https://nodejs.org/api/os.html#osfreemem
- Node.js Documentation — `os.totalmem()` — https://nodejs.org/api/os.html#ostotalmem
- Node.js Documentation — `os.homedir()` — https://nodejs.org/api/os.html#oshomedir
- Node.js Documentation — `os.hostname()` — https://nodejs.org/api/os.html#oshostname
- Node.js Documentation — `os.loadavg()` — https://nodejs.org/api/os.html#osloadavg
- Node.js Documentation — `os.networkInterfaces()` — https://nodejs.org/api/os.html#osnetworkinterfaces
- Node.js Documentation — `os.platform()` — https://nodejs.org/api/os.html#osplatform
- Node.js Documentation — `os.release()` — https://nodejs.org/api/os.html#osrelease
- Node.js Documentation — `os.tmpdir()` — https://nodejs.org/api/os.html#ostmpdir
- Node.js Documentation — `os.type()` — https://nodejs.org/api/os.html#ostype
- Node.js Documentation — `os.uptime()` — https://nodejs.org/api/os.html#osuptime
- Node.js Documentation — OS Constants — https://nodejs.org/api/os.html#os-constants
- libuv Documentation — `uv_available_parallelism()` — https://docs.libuv.org/en/v1.x/misc.html#c.uv_available_parallelism
- Stack Overflow — `os.freemem()` vs system monitors — https://stackoverflow.com/questions/51911849/node-js-os-freemem-do-not-work-correctly
- Stack Overflow — `os.loadavg()` vs `uptime` — https://stackoverflow.com/questions/28437135/os-loadavg-returns-different-values-than-uptime