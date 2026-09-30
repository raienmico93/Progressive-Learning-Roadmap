# Node.js Process & Resource Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Node.js process and resource management refers to the set of built-in modules and global objects that provide information about and control over the current Node.js process, its environment, its I/O streams, and its ability to spawn additional processes and threads.

**Technical Definition:** The `process` object is a global that provides information about, and control over, the current Node.js process. As a global, it is always available to Node.js applications without using `require()`. The `process` object is an instance of `EventEmitter`. The `node:child_process` module provides the ability to spawn child processes, while the `node:worker_threads` module enables the use of threads with message channels between them for parallel JavaScript execution.

**Beginner-Friendly Explanation:** When you run a Node.js program, it becomes a "process" — a running program with its own memory, environment variables, and standard input/output streams. Node.js gives you tools to inspect and control this process (like reading environment variables or handling signals), and also to create additional processes or threads when you need to do many things at once — like having multiple workers in a kitchen instead of just one cook.

### Key Characteristics

- **Global availability:** The `process` object is a global — no `require()` is needed to access it.
- **Event-driven:** `process` is an `EventEmitter` and emits lifecycle events such as `'exit'`, `'beforeExit'`, `'uncaughtException'`, and signal events.
- **Environment bridge:** `process.env` provides access to environment variables, and Node.js natively supports loading `.env` files via `process.loadEnvFile()` and CLI flags.
- **Graceful shutdown support:** Signal handlers for `SIGINT` and `SIGTERM` enable orderly resource cleanup before termination.
- **Multi-processing and multi-threading:** `child_process` spawns independent processes; `worker_threads` enables true parallelism within a single process.
- **Stream-based I/O:** `process.stdin`, `process.stdout`, and `process.stderr` are streams, providing non-blocking I/O capabilities.

### Prerequisites

- **Node.js runtime:** The `process` object is built into Node.js. `child_process` and `worker_threads` are built-in modules.
- **Basic JavaScript knowledge:** Understanding of functions, callbacks, streams, and the event loop.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Basic understanding of operating system concepts:** Processes, signals, environment variables, and I/O streams.
- **Asynchronous programming concepts:** The event loop, callbacks, and Promises.

### Related Programming Areas

- **File System (`fs`):** `process.cwd()` provides the current working directory for relative path resolution.
- **Streams:** `process.stdin`, `process.stdout`, and `process.stderr` are stream instances.
- **Events:** The `process` object extends `EventEmitter`; signal handling uses event listeners.
- **Cluster module:** Built on top of `child_process.fork()` for multi-core scaling.
- **Performance monitoring:** `process.memoryUsage()` and `process.cpuUsage()` provide resource statistics.
- **CLI applications:** `process.argv` and `process.exitCode` are fundamental to command-line tools.

### Core Concepts

The following core concepts are covered in this cheat sheet:

1. **Process Global Object Configuration** — the `process` object and its key properties.
2. **Environment Variables (`process.env`) and Native `.env` File Loading** — environment configuration.
3. **System Exit Codes, Unhandled Rejections, and Uncaught Exceptions** — error handling and termination.
4. **POSIX Signal Handling (SIGINT, SIGTERM for Graceful Shutdown)** — orderly termination.
5. **Standard I/O Pipelines (`process.stdin`, `process.stdout`, `process.stderr`)** — stream-based I/O.
6. **Multi-Processing Using `child_process` and `worker_threads`** — parallelism and concurrency.

---

## Core Concept 1: Process Global Object Configuration

### Definitions

**Core Definition:** The `process` object is a global object that provides information about and control over the current Node.js process.

**Technical Definition:** The `process` object provides information about, and control over, the current Node.js process. It can be imported as `import process from 'node:process'` or `const process = require('node:process')`. The `process` object is an instance of `EventEmitter` and exposes properties such as `process.argv` (command-line arguments), `process.execPath` (the absolute pathname of the executable that started the Node.js process), and `process.pid` (the process ID).

**Beginner-Friendly Explanation:** The `process` object is like a control panel for your running Node.js program. It tells you things like what arguments were passed when the program started, what directory the program is running in, and how much memory it's using. You can also use it to tell the program to exit, or to listen for signals like Ctrl+C.

### Purposes

- To access command-line arguments passed to the Node.js script.
- To determine the current working directory and executable path.
- To retrieve the process ID (PID) for logging or inter-process communication.
- To access Node.js version and platform information.
- To register event listeners for process lifecycle events.

### Syntax Rules and Structure

**Accessing the process object:**
```js
import process from 'node:process';   // ESM
const process = require('node:process'); // CommonJS
```
| Component | Breakdown |
|-----------|-----------|
| `process` | Global object; no import strictly required. |
| `'node:process'` | Module specifier for explicit import. |

**Key properties:**
| Property | Description |
|----------|-------------|
| `process.argv` | Array of command-line arguments. `argv[0]` is the Node.js executable, `argv[1]` is the script path. |
| `process.execPath` | Absolute pathname of the Node.js executable. |
| `process.pid` | The process ID of the current process. |
| `process.cwd()` | Returns the current working directory. |
| `process.version` | Node.js version string (e.g., `'v24.0.0'`). |
| `process.platform` | Platform identifier (`'linux'`, `'win32'`, `'darwin'`). |

**Constraints and Limitations:**
- `process.argv` contents vary based on how Node.js is invoked (e.g., with `-e` flag).
- `process.cwd()` returns the directory from which the Node.js process was launched, not the directory of the script file.
- Some properties (e.g., `process.pid`) are read-only.

### Annotated Code Example

```js
// process-info.js
const process = require('node:process');

// Command-line arguments
console.log('Script path:', process.argv[1]);
console.log('All args:', process.argv);

// Process identification
console.log('PID:', process.pid);
console.log('Executable:', process.execPath);

// Environment
console.log('Platform:', process.platform);
console.log('Node version:', process.version);
console.log('CWD:', process.cwd());
```

**Expected Output (run as `node process-info.js hello world`):**
```
Script path: /home/user/process-info.js
All args: [ '/usr/bin/node', '/home/user/process-info.js', 'hello', 'world' ]
PID: 12345
Executable: /usr/bin/node
Platform: linux
Node version: v24.0.0
CWD: /home/user
```

**Why this output:** `process.argv[0]` is the Node.js executable, `argv[1]` is the script path, and subsequent elements are the user-supplied arguments. The `pid` is the OS-assigned process ID. `cwd()` returns the directory from which the command was run.

### Real-World Cases

- **CLI argument parsing:** Reading `process.argv` to handle user-supplied flags and file paths.
- **Logging:** Including `process.pid` in log messages to distinguish between multiple process instances.
- **Platform detection:** Using `process.platform` to choose platform-specific code paths.

---

## Core Concept 2: Environment Variables (`process.env`) and Native `.env` File Loading

### Definitions

**Core Definition:** `process.env` is an object containing the user environment variables, and Node.js natively supports loading variables from `.env` files via `process.loadEnvFile()` and the `--env-file` CLI flag.

**Technical Definition:** The basic API for interacting with environment variables is `process.env`, which consists of an object with pre-populated user environment variables that can be modified and expanded. `.env` files (also known as dotenv files) are files that define environment variables, which Node.js applications can then interact with. Node.js defines its own specification for `.env` files: a `.env` file is a file that contains key-value pairs, each pair represented by a variable name followed by the equal sign (`=`) followed by a variable value. The `process.loadEnvFile()` method loads an `.env` file and populates `process.env` with its variables. Node.js v20.6.0 introduced the `--env-file` CLI option to natively load `.env` files.

**Beginner-Friendly Explanation:** Environment variables are like sticky notes that you attach to your program before it runs — they carry configuration information like API keys, database passwords, or the port number to listen on. Instead of hardcoding these values in your code, you store them in a `.env` file and Node.js can read them into `process.env`.

### Purposes

- To store configuration and secrets outside of source code.
- To differentiate behaviour between development, staging, and production environments.
- To avoid hardcoding sensitive values like API keys and database credentials.
- To provide a standard, portable mechanism for application configuration.
- To replace the third-party `dotenv` package with native Node.js functionality (Node.js v20.6.0+).

### Syntax Rules and Structure

**Accessing environment variables:**
```js
const value = process.env.MY_VAR; // returns undefined if not set
```
| Component | Breakdown |
|-----------|-----------|
| `process.env` | Object containing environment variables. |
| `MY_VAR` | Variable name (must match `^[a-zA-Z_]+[a-zA-Z0-9_]*$`). |
| Returns | String value, or `undefined` if not set. |

**Loading a `.env` file natively:**
```js
process.loadEnvFile('.env'); // loads .env in cwd
// or
node --env-file=.env index.js
```
| Component | Breakdown |
|-----------|-----------|
| `process.loadEnvFile(path?)` | Loads an `.env` file and populates `process.env`. |
| `--env-file=.env` | CLI flag to load an `.env` file at startup. |

**`.env` file syntax rules:**
| Rule | Example |
|------|---------|
| Variable names: letters, digits, underscores; cannot start with a digit. | `MY_VAR=value` |
| Values can be quoted with single or double quotes. | `MY_VAR="hello world"` |
| Quoted values can span multiple lines. | `MY_VAR="line1\nline2"` |
| `#` denotes a comment. | `# This is a comment` |
| Leading/trailing whitespace around keys and values is ignored. | `MY_VAR = value` |

**Constraints and Limitations:**
- All values in `.env` files are interpreted as strings, even if they look like numbers or booleans.
- `.env` files have no formal specification; Node.js defines its own.
- The `--env-file` flag was introduced in Node.js v20.6.0; `process.loadEnvFile()` in v20.12.0.
- Environment variables are visible to the process and any child processes it spawns.

### Multiple Annotated Code Examples

#### Example 1: Reading Environment Variables

```js
// env-read.js
// Assume environment: MY_VAR=hello, PORT=3000 set externally
console.log('MY_VAR:', process.env.MY_VAR);       // 'hello'
console.log('PORT:', process.env.PORT);           // '3000'
console.log('MISSING:', process.env.MISSING);     // undefined

// Provide a default value using the nullish coalescing operator
const port = process.env.PORT ?? 8080;
console.log('Port:', port); // '3000' (string) or 8080 (number)
```

**Expected Output:**
```
MY_VAR: hello
PORT: 3000
MISSING: undefined
Port: 3000
```

**Why this output:** `process.env` returns strings for set variables and `undefined` for unset ones. The `??` operator provides a fallback when the variable is unset.

#### Example 2: Loading a `.env` File Natively

```bash
# .env file
DB_HOST=localhost
DB_PORT=5432
DB_USER=admin
DB_PASS="secret123"
```

```js
// env-load.js
const process = require('node:process');

// Load .env from the current directory
process.loadEnvFile('.env');

console.log('DB host:', process.env.DB_HOST);   // 'localhost'
console.log('DB port:', process.env.DB_PORT);   // '5432' (string, not number)
console.log('DB user:', process.env.DB_USER);   // 'admin'
console.log('DB pass:', process.env.DB_PASS);   // 'secret123'
```

**Expected Output:**
```
DB host: localhost
DB port: 5432
DB user: admin
DB pass: secret123
```

**Why this output:** `process.loadEnvFile('.env')` reads the file and populates `process.env`. All values are strings, even `5432`, which would need `parseInt()` for numeric use. Quoted values are automatically unquoted.

### Real-World Cases

- **Database configuration:** Storing connection strings and credentials in `.env` files.
- **API keys:** Keeping third-party API keys out of version control.
- **Multi-environment deployment:** Using different `.env` files for development, staging, and production.
- **Docker/Kubernetes:** Injecting environment variables into containers via `process.env`.

---

## Core Concept 3: System Exit Codes, Unhandled Rejections, and Uncaught Exceptions

### Definitions

**Core Definition:** `process.exit()` terminates the process with a specified exit code, `process.exitCode` sets the exit code for graceful termination, `'uncaughtException'` handles exceptions that bubble to the event loop, and `'unhandledRejection'` handles Promise rejections without error handlers.

**Technical Definition:** The `process.exit([code])` method instructs Node.js to terminate the process synchronously with an exit status of `code`. If `code` is omitted, exit uses either the 'success' code `0` or the value of `process.exitCode` if it has been set. The `'uncaughtException'` event is emitted when an uncaught JavaScript exception bubbles all the way back to the event loop. By default, Node.js handles such exceptions by printing the stack trace to stderr and exiting with code 1, overriding any previously set `process.exitCode`. The `'unhandledRejection'` event is emitted whenever a Promise is rejected and no error handler is attached to the promise within a turn of the event loop. If an `'unhandledRejection'` event is emitted but not handled, it will be raised as an uncaught exception.

**Beginner-Friendly Explanation:** When your program finishes, it sends a "report card" to the operating system — a number called an exit code. `0` means "everything went fine," and any other number means something went wrong. If an error isn't caught anywhere in your code, Node.js prints it and exits with code `1`. You can listen for these uncaught errors to log them properly before the process exits.

### Purposes

- To communicate success or failure to the shell or parent process.
- To ensure clean resource cleanup before termination.
- To handle unexpected errors gracefully without crashing silently.
- To distinguish between different failure modes through specific exit codes.
- To prevent unhandled Promise rejections from crashing the process unexpectedly.

### Syntax Rules and Structure

**`process.exit(code?)`:**
```js
process.exit(0);  // success
process.exit(1);  // failure
```
| Component | Breakdown |
|-----------|-----------|
| `code` | Optional integer (or integer string). Default: `0`. |
| Behaviour | Terminates synchronously. |

**`process.exitCode`:**
```js
process.exitCode = 1; // set code, let process exit naturally
```
| Component | Breakdown |
|-----------|-----------|
| Type | Number or integer string. |
| Behaviour | Process exits with this code when the event loop empties. |

**Common exit codes:**
| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Uncaught Fatal Exception |
| `2` | Unused (reserved by Bash) |
| `5` | Fatal Error (V8) |
| `9` | Invalid Argument |
| `12` | Invalid Debug Argument |

**`'uncaughtException'` handler:**
```js
process.on('uncaughtException', (err, origin) => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `err` | The uncaught `Error` object. |
| `origin` | `'uncaughtException'` or `'unhandledRejection'`. |

**`'unhandledRejection'` handler:**
```js
process.on('unhandledRejection', (reason, promise) => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `reason` | The rejection reason (typically an `Error`). |
| `promise` | The Promise that was rejected. |

**Constraints and Limitations:**
- Calling `process.exit()` may truncate asynchronous writes to `stdout`/`stderr`; prefer setting `process.exitCode` and letting the process exit naturally.
- Adding an `'uncaughtException'` handler overrides the default behaviour of printing the stack trace and exiting with code 1. Use with caution — the application may be in an undefined state after an uncaught exception.
- `'unhandledRejection'` behaviour can be changed via the `--unhandled-rejections` flag.

### Multiple Annotated Code Examples

#### Example 1: Setting Exit Codes

```js
// exit-code.js
const process = require('node:process');

const success = true;

if (success) {
  console.log('Operation succeeded.');
  process.exitCode = 0;
} else {
  console.error('Operation failed.');
  process.exitCode = 1;
}

// Process exits naturally with the set code after the event loop empties
```

**Expected Output (with shell `$?`):**
```
Operation succeeded.
$ echo $?
0
```

**Why this output:** Instead of calling `process.exit()`, the code sets `process.exitCode`. The process exits naturally when the event loop has no more work, using the assigned code. This ensures stdout writes are not truncated.

#### Example 2: Handling Uncaught Exceptions

```js
// uncaught-exception.js
const process = require('node:process');

process.on('uncaughtException', (err, origin) => {
  console.error(`Caught exception: ${err.message}`);
  console.error(`Origin: ${origin}`);
  process.exitCode = 1;
});

// Intentionally cause an uncaught exception
setTimeout(() => {
  throw new Error('Something went wrong!');
}, 100);
```

**Expected Output:**
```
Caught exception: Something went wrong!
Origin: uncaughtException
```

**Why this output:** The `'uncaughtException'` handler intercepts the thrown error before the default crash behaviour occurs. The handler logs the error and sets `process.exitCode = 1` for a graceful failure exit.

#### Example 3: Handling Unhandled Promise Rejections

```js
// unhandled-rejection.js
const process = require('node:process');

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection:', reason.message);
  process.exitCode = 1;
});

// A Promise that rejects with no catch handler
Promise.reject(new Error('Promise failed'));
```

**Expected Output:**
```
Unhandled rejection: Promise failed
```

**Why this output:** The `'unhandledRejection'` event is emitted because the Promise rejection was not caught within a turn of the event loop. The handler receives the reason and the Promise, logs it, and sets the exit code.

### Real-World Cases

- **CLI tools:** Setting exit codes for shell scripting and CI pipelines.
- **Server processes:** Logging uncaught exceptions before exit for post-mortem debugging.
- **Graceful degradation:** Handling unhandled rejections without crashing the process.

---

## Core Concept 4: POSIX Signal Handling (SIGINT, SIGTERM for Graceful Shutdown)

### Definitions

**Core Definition:** Node.js can listen for POSIX signals such as `SIGINT` (Ctrl+C) and `SIGTERM` (termination request) to perform graceful shutdown operations before the process exits.

**Technical Definition:** Node.js establishes signal handlers for `SIGINT` and `SIGTERM`, and Node.js processes will not terminate immediately due to receipt of those signals. Instead, Node.js will perform a series of cleanup operations and then re-raise the handled signal. The most reliable approach is listening for `SIGTERM` and `SIGINT` signals to perform cleanup operations before process termination.

**Beginner-Friendly Explanation:** When you press Ctrl+C in a terminal, the operating system sends a `SIGINT` signal to the program. Normally this kills the program immediately. But if your Node.js program is listening for that signal, it can first finish what it's doing — close database connections, complete pending requests, save state — and then exit cleanly. This is called a "graceful shutdown."

### Purposes

- To ensure pending requests are completed before the server shuts down.
- To close database connections and release resources cleanly.
- To remove temporary files and perform final logging.
- To prevent data corruption by avoiding abrupt termination.
- To handle container orchestrator termination signals (SIGTERM from Kubernetes, Docker).

### Syntax Rules and Structure

**Registering a signal handler:**
```js
process.on('SIGINT', () => { /* handle */ });
process.on('SIGTERM', () => { /* handle */ });
```
| Component | Breakdown |
|-----------|-----------|
| `'SIGINT'` | Signal sent by Ctrl+C on POSIX systems. |
| `'SIGTERM'` | Signal sent by `kill` command or container orchestrators. |
| Listener | Callback invoked when the signal is received. |

**Supported signals:**
| Signal | Description |
|--------|-------------|
| `SIGINT` | Interrupt from keyboard (Ctrl+C). |
| `SIGTERM` | Termination signal (default for `kill`). |
| `SIGHUP` | Hangup; often used to reload configuration. |
| `SIGUSR1` | User-defined signal 1; starts the debugger. |
| `SIGUSR2` | User-defined signal 2; used by nodemon. |

**Constraints and Limitations:**
- On Windows, most POSIX signals are not supported; `SIGINT` and `SIGBREAK` are the exceptions.
- Once a signal handler is installed, the process will not terminate on that signal unless the handler calls `process.exit()`.
- Signal handlers should be synchronous; asynchronous cleanup requires manual coordination.
- It is not possible to install a handler for `SIGKILL` or `SIGSTOP`.

### Annotated Code Example

```js
// graceful-shutdown.js
const http = require('node:http');

const server = http.createServer((req, res) => {
  res.end('Hello, world!\n');
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});

const gracefulShutdown = (signal) => {
  console.log(`Received ${signal}, starting graceful shutdown...`);

  // Stop accepting new connections
  server.close(() => {
    console.log('HTTP server closed.');
    // Close database connections, clean up temp files, etc.
    console.log('Cleanup completed, exiting.');
    process.exit(0);
  });

  // Force shutdown if cleanup takes too long
  setTimeout(() => {
    console.error('Forced shutdown after timeout.');
    process.exit(1);
  }, 10000);
};

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

console.log('Press Ctrl+C to trigger graceful shutdown.');
```

**Expected Output (when Ctrl+C is pressed):**
```
Server running on port 3000
Press Ctrl+C to trigger graceful shutdown.
^C
Received SIGINT, starting graceful shutdown...
HTTP server closed.
Cleanup completed, exiting.
```

**Why this output:** The `SIGINT` signal triggers the `gracefulShutdown` function. `server.close()` stops the server from accepting new connections and invokes its callback once all existing connections are finished. Then the process exits with code 0. A timeout ensures the process doesn't hang indefinitely.

### Real-World Cases

- **Web servers:** Closing HTTP servers gracefully during deployments or scaling events.
- **Database applications:** Closing connection pools before exit.
- **Containerised applications:** Handling Kubernetes `SIGTERM` for zero-downtime rolling updates.
- **Message queues:** Finishing in-flight messages before shutting down consumers.

---

## Core Concept 5: Standard I/O Pipelines (`process.stdin`, `process.stdout`, `process.stderr`)

### Definitions

**Core Definition:** `process.stdin`, `process.stdout`, and `process.stderr` are streams connected to the standard input, output, and error streams of the process, respectively.

**Technical Definition:** The `process.stdout` property returns a stream connected to `stdout` (fd 1). It is a `net.Socket` (which is a Duplex stream) unless fd 1 refers to a file. When Node.js detects that it is being run with a text terminal ("TTY") attached, `process.stdin` will, by default, be initialized as an instance of `tty.ReadStream` and both `process.stdout` and `process.stderr` will, by default, be instances of `tty.WriteStream`. The preferred method of determining whether Node.js is being run within a TTY context is to check that the value of the `process.stdout.isTTY` property is `true`.

**Beginner-Friendly Explanation:** Think of `process.stdin` as a pipe that carries data into your program (what the user types), `process.stdout` as a pipe that carries normal output out (what your program prints), and `process.stderr` as a separate pipe for error messages. Keeping normal output and error output separate is important because it lets you redirect them independently in the shell.

### Purposes

- To read user input interactively from the command line.
- To write program output to the console or to files via redirection.
- To write error messages separately from normal output.
- To pipe data between processes.
- To detect whether the process is running in an interactive terminal (TTY).

### Syntax Rules and Structure

**Reading from stdin:**
```js
process.stdin.on('data', (chunk) => {
  // chunk is a Buffer
});
```
| Component | Breakdown |
|-----------|-----------|
| `'data'` | Event emitted when data is available. |
| `chunk` | A `Buffer` containing the input data. |
| `process.stdin.setEncoding('utf8')` | Converts chunks to strings. |

**Writing to stdout:**
```js
process.stdout.write('Hello\n');
```
| Component | Breakdown |
|-----------|-----------|
| `write(data)` | Writes data to the stream. Returns `true` if the buffer is not full. |
| Encoding | Uses the stream's default encoding (utf8 for strings). |

**Writing to stderr:**
```js
process.stderr.write('Error\n');
```
| Component | Breakdown |
|-----------|-----------|
| `write(data)` | Writes error data. |

**Key properties:**
| Property | Description |
|----------|-------------|
| `process.stdout.isTTY` | `true` if stdout is a TTY. |
| `process.stdin.isTTY` | `true` if stdin is a TTY. |
| `process.stderr.isTTY` | `true` if stderr is a TTY. |

**Constraints and Limitations:**
- `process.stdout` and `process.stderr` differ from other Node.js streams: they cannot be closed (`end()` will throw).
- Writes to `stdout` may be asynchronous when connected to a pipe or file, but synchronous when connected to a TTY.
- `process.exit()` can truncate pending stdout writes; set `process.exitCode` instead.

### Annotated Code Example

```js
// stdio-example.js
const process = require('node:process');

// Check if running in a TTY
console.log('stdout is TTY:', process.stdout.isTTY);

// Write to stdout
process.stdout.write('This goes to stdout.\n');

// Write to stderr
process.stderr.write('This goes to stderr.\n');

// Read from stdin line by line
process.stdin.setEncoding('utf8');
process.stdin.on('data', (chunk) => {
  process.stdout.write(`Received: ${chunk}`);
});

// End stdin after a moment (for demonstration)
setTimeout(() => process.stdin.pause(), 3000);
```

**Expected Output (run interactively, typing "hello" then pressing Enter):**
```
stdout is TTY: true
This goes to stdout.
This goes to stderr.
hello
Received: hello
```

**Why this output:** `process.stdout.write()` and `process.stderr.write()` send data to their respective streams. The `'data'` event on `process.stdin` fires when the user types input. The `Received: hello` line demonstrates reading from stdin and echoing back to stdout.

### Real-World Cases

- **Interactive CLI tools:** Reading user input and displaying prompts.
- **Logging:** Writing structured logs to stdout and errors to stderr.
- **Shell pipelines:** Piping data between Node.js processes and other command-line tools.
- **Progress indicators:** Checking `isTTY` to decide whether to display progress bars.

---

## Core Concept 6: Multi-Processing Using `child_process` and `worker_threads`

### Definitions

**Core Definition:** `child_process` spawns independent operating system processes, while `worker_threads` creates true threads within a single process, both enabling parallel execution of JavaScript.

**Technical Definition:** The `child_process` module provides the ability to spawn child processes in a manner that is similar, but not identical, to `popen(3)`. This capability is primarily provided by the `child_process.spawn()` function. `child_process.fork()` spawns a new Node.js process and invokes a specified module with an IPC communication channel established that allows sending messages between parent and child. The `worker_threads` module enables the use of threads with message channels between them. Workers (threads) are useful for performing CPU-intensive JavaScript operations. They will not help much with I/O-intensive work.

**Beginner-Friendly Explanation:** Node.js normally runs your JavaScript code in a single thread — like one person doing all the work. For I/O tasks (like reading files or waiting for network requests), this is fine because the person can switch tasks while waiting. But for CPU-heavy tasks (like video encoding or complex calculations), one person isn't enough. `child_process` lets you hire completely separate workers (processes), while `worker_threads` lets you add extra hands within the same workshop (process).

### Purposes

- To run CPU-intensive JavaScript without blocking the main event loop.
- To utilise multiple CPU cores for parallel computation.
- To spawn external commands and shell scripts.
- To isolate risky or unstable code in separate processes.
- To build multi-process servers using the cluster module.

### Syntax Rules and Structure

**`child_process.spawn()`:**
```js
const { spawn } = require('node:child_process');
const child = spawn('ls', ['-lh', '/usr']);
```
| Component | Breakdown |
|-----------|-----------|
| `command` | The command to execute. |
| `args` | Array of arguments. |
| `options` | Optional configuration (cwd, env, stdio). |
| Returns | A `ChildProcess` instance with `stdout`, `stderr`, `stdin`. |

**`child_process.exec()`:**
```js
const { exec } = require('node:child_process');
exec('ls -lh /usr', (err, stdout, stderr) => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `command` | Shell command string. |
| `callback` | `(err, stdout, stderr) => {}`. |
| Returns | Buffered output (not streams). |

**`child_process.fork()`:**
```js
const { fork } = require('node:child_process');
const child = fork('worker.js');
child.send({ message: 'hello' });
child.on('message', (msg) => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `modulePath` | Path to the Node.js module to run. |
| `options` | Optional configuration. |
| Returns | `ChildProcess` with IPC channel (`send`, `'message'` event). |

**`worker_threads.Worker`:**
```js
const { Worker, isMainThread, workerData } = require('node:worker_threads');
if (isMainThread) {
  const worker = new Worker(__filename, { workerData: 'Hello' });
} else {
  console.log(workerData);
}
```
| Component | Breakdown |
|-----------|-----------|
| `Worker(filename)` | Creates a new worker thread. |
| `workerData` | Data passed to the worker (cloned). |
| `isMainThread` | `true` if running on the main thread. |

**Comparison:**

| Feature | `child_process.spawn` | `child_process.exec` | `child_process.fork` | `worker_threads` |
|---------|----------------------|---------------------|---------------------|------------------|
| New V8 instance | No | No | Yes | No (separate isolate) |
| IPC channel | Optional | No | Yes | Yes |
| Memory overhead | Low | Medium | High | Medium |
| Best for | Streaming, large data | Shell commands | Node.js modules | CPU-intensive JS |
| Shared memory | No | No | No | Yes (SharedArrayBuffer) |

**Constraints and Limitations:**
- `child_process.exec()` buffers all output in memory; for large outputs, use `spawn()`.
- `child_process.fork()` spawns a full Node.js process, which has significant memory overhead.
- `worker_threads` cannot directly share memory unless using `SharedArrayBuffer` or `MessageChannel`.
- Worker threads are not a replacement for child processes for I/O-bound work.
- `process.exit()` in a worker thread stops only that thread, not the process.

### Multiple Annotated Code Examples

#### Example 1: Using `child_process.spawn` for Streaming

```js
// spawn-example.js
const { spawn } = require('node:child_process');

const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`Child process exited with code ${code}`);
});
```

**Expected Output:**
```
stdout: total 0
drwxr-xr-x   2 root root    6 Jan  1 00:00 bin
...
Child process exited with code 0
```

**Why this output:** `spawn()` starts the `ls` command and returns a `ChildProcess` with streams for `stdout` and `stderr`. The `'data'` events fire as data becomes available, allowing streaming of large outputs. The `'close'` event signals that the child process has exited.

#### Example 2: Using `child_process.fork` with IPC

```js
// parent.js
const { fork } = require('node:child_process');

const child = fork('./child.js');

child.on('message', (msg) => {
  console.log('Parent received:', msg);
});

child.send({ task: 'compute', value: 42 });

// child.js
const process = require('node:process');

process.on('message', (msg) => {
  console.log('Child received:', msg);
  const result = msg.value * 2;
  process.send({ result });
});
```

**Expected Output:**
```
Child received: { task: 'compute', value: 42 }
Parent received: { result: 84 }
```

**Why this output:** `fork()` creates a new Node.js process and establishes an IPC channel. The parent sends a message with `child.send()`; the child receives it via `process.on('message')` and sends back a result with `process.send()`. This enables bidirectional communication between processes.

#### Example 3: Using `worker_threads` for CPU-Intensive Work

```js
// worker-main.js
const { Worker, isMainThread, parentPort, workerData } = require('node:worker_threads');

if (isMainThread) {
  // Main thread: spawn a worker
  const worker = new Worker(__filename, { workerData: { n: 40 } });

  worker.on('message', (result) => {
    console.log('Fibonacci result:', result);
  });

  worker.on('error', (err) => {
    console.error('Worker error:', err);
  });

  worker.on('exit', (code) => {
    console.log('Worker exited with code:', code);
  });
} else {
  // Worker thread: perform CPU-intensive computation
  function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
  }

  const result = fibonacci(workerData.n);
  parentPort.postMessage(result);
}
```

**Expected Output:**
```
Fibonacci result: 102334155
Worker exited with code: 0
```

**Why this output:** The main thread creates a `Worker` running the same file. Inside the worker, `isMainThread` is `false`, so the worker computes the Fibonacci number and sends the result back via `parentPort.postMessage()`. The main thread receives it via the `'message'` event. The worker exits cleanly with code 0.

### Real-World Cases

- **Image/video processing:** Offloading CPU-intensive encoding to worker threads.
- **Web scraping:** Spawning child processes to run external tools.
- **Cluster servers:** Using `cluster.fork()` (built on `child_process.fork()`) to utilise all CPU cores.
- **Data pipelines:** Streaming large datasets through `spawn()` without buffering.
- **Cryptocurrency mining:** Using worker threads for parallel hash computation.

---

## References

- Node.js Documentation — Process — https://nodejs.org/api/process.html
- Node.js Documentation — Environment Variables — https://nodejs.org/api/environment_variables.html
- Node.js Documentation — `process.exit()` — https://nodejs.org/api/process.html#processexitcode
- Node.js Documentation — `'uncaughtException'` — https://nodejs.org/api/process.html#event-uncaughtexception
- Node.js Documentation — `'unhandledRejection'` — https://nodejs.org/api/process.html#event-unhandledrejection
- Node.js Documentation — `process.stdin` — https://nodejs.org/api/process.html#processstdin
- Node.js Documentation — `process.stdout` — https://nodejs.org/api/process.html#processstdout
- Node.js Documentation — `process.stderr` — https://nodejs.org/api/process.html#processstderr
- Node.js Documentation — Child Process — https://nodejs.org/api/child_process.html
- Node.js Documentation — `child_process.spawn()` — https://nodejs.org/api/child_process.html#child_processspawncommand-args-options
- Node.js Documentation — `child_process.exec()` — https://nodejs.org/api/child_process.html#child_processexeccommand-options-callback
- Node.js Documentation — `child_process.fork()` — https://nodejs.org/api/child_process.html#child_processforkmodulepath-args-options
- Node.js Documentation — Worker Threads — https://nodejs.org/api/worker_threads.html
- Node.js Documentation — `worker_threads.Worker` — https://nodejs.org/api/worker_threads.html#class-worker
- Node.js Documentation — TTY — https://nodejs.org/api/tty.html
- Node.js Documentation — `process.loadEnvFile()` — https://nodejs.org/api/process.html#processloadenvfilepath
- CoreUI — How to Handle Process Signals in Node.js — https://coreui.io/answers/how-to-handle-process-signals-in-nodejs/
- SitePoint — An Introduction to Node.js Multithreading — https://www.sitepoint.com/node-js-multithreading/
- Stack Overflow — Difference between Child_process and Worker Threads — https://stackoverflow.com/questions/56312692/what-is-the-difference-between-child-process-and-worker-threads