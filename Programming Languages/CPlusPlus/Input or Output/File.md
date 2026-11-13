# C++ File I/O — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
C++ File I/O is the part of the C++ Standard Library, declared in `<fstream>`, that provides facilities for reading from and writing to files on persistent storage using stream abstractions.

**Technical Definition**
The `<fstream>` header defines the class templates `std::basic_ifstream`, `std::basic_ofstream`, and `std::basic_fstream`, which combine a file-based stream buffer (`std::basic_filebuf`) with the high-level formatted and unformatted I/O interfaces of `std::basic_istream`, `std::basic_ostream`, and `std::basic_iostream`, respectively. These classes manage the opening, closing, reading, writing, and positioning of files through a well-defined set of member functions and open modes.

**Beginner-Friendly Explanation**
File I/O lets your program save data to a file and read it back later. Think of a file as a long strip of characters on disk. `std::ofstream` is like a pen that writes to the strip; `std::ifstream` is like a scanner that reads from it; `std::fstream` can do both. You open the file, move a “cursor” around, read or write data, and then close it.

### Key Characteristics

- **RAII-based** — file streams automatically close their associated file when destroyed.
- **Type-safe** — the compiler enforces correct usage of `<<` and `>>`.
- **Buffered** — file streams buffer reads and writes for efficiency.
- **Mode-driven** — behaviour is controlled by a bitmask of open modes (`in`, `out`, `app`, `binary`, `trunc`, `ate`).
- **Stateful** — each stream tracks error state (`goodbit`, `failbit`, `badbit`, `eofbit`) that must be checked.
- **Positionable** — separate “get” and “put” pointers can be moved with `seekg`/`seekp` and queried with `tellg`/`tellp`.

### Prerequisites

- Basic C++ syntax (`#include`, variables, functions).
- Familiarity with standard I/O streams (`std::cin`, `std::cout`).
- Understanding of the stream state model (`good()`, `fail()`, `eof()`).
- Basic knowledge of string handling (`std::string`, `std::getline`).

### Related Programming Areas

- **Standard I/O** — `<iostream>` for console streams.
- **String streams** — `<sstream>` for in-memory streams.
- **Serialization** — persisting object state to files.
- **Resource management (RAII)** — automatic cleanup of file handles.
- **Error handling** — exception safety and state validation.

### Core Concepts / Features

1. `<fstream>`
2. `std::ifstream`
3. `std::ofstream`
4. `std::fstream`
5. File modes
6. File operations
7. File reading
8. File writing
9. File positioning
10. File state
11. File I/O best practices

---

## 1. `<fstream>`

### Definitions

**Core Definition**
`<fstream>` is the standard library header that declares the file stream class templates and their convenience typedefs.

**Technical Definition**
The header defines `std::basic_filebuf`, `std::basic_ifstream`, `std::basic_ofstream`, and `std::basic_fstream`, along with the typedefs `std::ifstream` (`basic_ifstream<char>`), `std::ofstream` (`basic_ofstream<char>`), `std::fstream` (`basic_fstream<char>`), and their `wchar_t` counterparts (`wifstream`, `wofstream`, `wfstream`).

**Beginner-Friendly Explanation**
`<fstream>` is the toolbox header you include to work with files. It gives you `ifstream` for reading, `ofstream` for writing, and `fstream` for both.

### Purposes

- To provide the declarations needed for file stream objects.
- To expose the three primary file stream classes.
- To enable RAII-based file handling.
- To support both narrow and wide character file I/O.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <fstream>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<fstream>` | Standard header for file streams. |

**Syntax Rules**
- Must be included before using any file stream type.
- The header is part of the `<iostream>` family and may indirectly include other I/O headers.

**Constraints and Limitations**
- On some implementations, including `<fstream>` alone may not provide `std::string`; include `<string>` explicitly when needed.

### Annotated Code Example

```cpp
#include <fstream>    // enables file stream types
#include <iostream>

int main() {
    std::ofstream out("example.txt");   // std::ofstream available
    out << "Hello, file!\n";
}
```

**Expected Output**  
(A file `example.txt` is created containing `Hello, file!`.)

**Why this output?**  
`<fstream>` declares `std::ofstream`, which is used to write to the file.

### Real-World Cases

- Any program that persists data between runs.
- Logging systems that write to files.
- Configuration file readers.

### References

- `<fstream>` — https://en.cppreference.com/w/cpp/header/fstream

---

## 2. `std::ifstream`

### Definitions

**Core Definition**
`std::ifstream` is a class that provides high-level input operations on file-based streams.

**Technical Definition**
`std::basic_ifstream<CharT, Traits>` inherits from `std::basic_istream<CharT, Traits>` and holds a `std::basic_filebuf` as its underlying stream buffer. It is the standard type for reading from files.

**Beginner-Friendly Explanation**
`std::ifstream` is your “file reader”. You create one, give it a filename, and then use `>>`, `getline`, or `read` to pull data out of the file.

### Purposes

- To read formatted data from a file.
- To read raw bytes or lines of text.
- To support RAII-based file reading.
- To enable state-based error detection during reads.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::ifstream in("filename");
// or
std::ifstream in;
in.open("filename");
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| Constructor | Opens the file (with optional mode). |
| `open()` | Opens a file if not already open. |
| `is_open()` | Checks whether a file is associated. |
| `close()` | Closes the file. |
| `>>`, `getline`, `read` | Input operations inherited from `istream`. |

**Syntax Rules**
- The default open mode is `std::ios::in`.
- `ifstream` is movable but not copyable.
- The file is automatically closed when the `ifstream` object is destroyed.

**Constraints and Limitations**
- Cannot write to the file.
- Reading past the end sets `eofbit` and `failbit`.

### Annotated Code Examples

**Example 1: Reading words**

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ifstream in("data.txt");
    std::string word;
    while (in >> word) {               // reads whitespace-separated tokens
        std::cout << word << '\n';
    }
}
```

**Expected Output**  
(Prints each word from `data.txt` on its own line.)

**Why this output?**  
`operator>>` extracts tokens separated by whitespace.

**Example 2: Reading lines**

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ifstream in("data.txt");
    std::string line;
    while (std::getline(in, line)) {   // reads line by line
        std::cout << line << '\n';
    }
}
```

**Expected Output**  
(Prints each line of `data.txt`.)

**Why this output?**  
`getline` reads until the newline character.

### Real-World Cases

- **Configuration parsing** — reading key-value pairs from text files.
- **Data analysis** — loading CSV or log files.
- **Save-game loaders** — restoring game state.

### References

- `std::basic_ifstream` — https://en.cppreference.com/w/cpp/io/basic_ifstream

---

## 3. `std::ofstream`

### Definitions

**Core Definition**
`std::ofstream` is a class that provides high-level output operations on file-based streams.

**Technical Definition**
`std::basic_ofstream<CharT, Traits>` inherits from `std::basic_ostream<CharT, Traits>` and holds a `std::basic_filebuf`. It is the standard type for writing to files.

**Beginner-Friendly Explanation**
`std::ofstream` is your “file writer”. You create one with a filename and then use `<<`, `write`, or `put` to send data into the file.

### Purposes

- To write formatted data to a file.
- To write raw bytes or binary data.
- To create or overwrite files.
- To append data to existing files (with `app` mode).

### Syntax Rules and Structure

**General Syntax**
```cpp
std::ofstream out("filename");
// or
std::ofstream out;
out.open("filename");
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| Constructor | Opens the file for writing. |
| `open()` | Opens a file if not already open. |
| `is_open()` | Checks whether a file is associated. |
| `close()` | Closes the file. |
| `<<`, `write`, `put` | Output operations inherited from `ostream`. |

**Syntax Rules**
- The default open mode is `std::ios::out | std::ios::trunc` for `ofstream` constructors (truncates existing files unless `app` is specified).
- `ofstream` is movable but not copyable.
- The file is automatically closed on destruction.

**Constraints and Limitations**
- Cannot read from the file.
- Without `app`, opening an existing file truncates it.

### Annotated Code Examples

**Example 1: Writing text**

```cpp
#include <fstream>

int main() {
    std::ofstream out("output.txt");
    out << "Hello, file!\n";
    out << 42 << '\n';
}
```

**Expected Output**  
(A file `output.txt` containing `Hello, file!` and `42`.)

**Why this output?**  
Each `<<` inserts data into the file stream.

**Example 2: Appending**

```cpp
#include <fstream>

int main() {
    std::ofstream out("log.txt", std::ios::app);   // append mode
    out << "New log entry\n";
}
```

**Expected Output**  
(The line `New log entry` is appended to `log.txt` without erasing existing content.)

**Why this output?**  
`std::ios::app` seeks to the end before each write.

### Real-World Cases

- **Logging** — appending messages to log files.
- **Report generation** — writing formatted output.
- **Data export** — saving computed results.

### References

- `std::basic_ofstream` — https://en.cppreference.com/w/cpp/io/basic_ofstream

---

## 4. `std::fstream`

### Definitions

**Core Definition**
`std::fstream` is a class that provides high-level input and output operations on file-based streams.

**Technical Definition**
`std::basic_fstream<CharT, Traits>` inherits from `std::basic_iostream<CharT, Traits>` and holds a `std::basic_filebuf`. It combines the functionality of `ifstream` and `ofstream`.

**Beginner-Friendly Explanation**
`std::fstream` is a “file reader-writer”. It can both read from and write to the same file, which is useful for updating records in place.

### Purposes

- To read from and write to the same file.
- To support random-access file updates.
- To simplify code that requires both input and output.
- To enable in-place modification of file content.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::fstream fs("filename", std::ios::in | std::ios::out);
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| Constructor | Opens the file with the given mode. |
| `open()` | Opens a file if not already open. |
| `is_open()` | Checks whether a file is associated. |
| `close()` | Closes the file. |
| `seekg`, `seekp`, `tellg`, `tellp` | Positioning operations. |

**Syntax Rules**
- The default open mode for `fstream` is `std::ios::in | std::ios::out`.
- Combining `in` and `out` requires the file to exist (unless `trunc` or `app` is also used).
- `fstream` is movable but not copyable.

**Constraints and Limitations**
- When switching between reading and writing, a seek operation may be required between them (depending on the implementation).
- File must exist for `in | out` mode unless `trunc` or `app` is specified.

### Annotated Code Examples

**Example 1: Read-write access**

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::fstream fs("data.txt", std::ios::in | std::ios::out);
    if (!fs) { std::cerr << "Failed to open\n"; return 1; }
    std::string line;
    std::getline(fs, line);            // read first line
    fs.seekp(0, std::ios::end);        // move put pointer to end
    fs << "\nAppended by fstream";     // write
    std::cout << "Read: " << line << '\n';
}
```

**Expected Output**  
(Prints the first line of `data.txt` and appends a new line.)

**Why this output?**  
`fstream` supports both `getline` (input) and `<<` (output) on the same file.

**Example 2: Updating a character in place**

```cpp
#include <fstream>

int main() {
    std::fstream fs("data.txt", std::ios::in | std::ios::out);
    fs.seekp(0);                       // go to beginning
    fs.put('X');                       // overwrite first character
}
```

**Expected Output**  
(The first character of `data.txt` becomes `X`.)

**Why this output?**  
`seekp` positions the put pointer, and `put` writes a single character.

### Real-World Cases

- **Database-like record updates** — modifying fixed-size records.
- **Configuration editors** — reading and rewriting settings.
- **File patchers** — in-place byte modifications.

### References

- `std::basic_fstream` — https://en.cppreference.com/w/cpp/io/basic_fstream

---

## 5. File Modes

### Definitions

**Core Definition**
File modes are bitmask flags that specify how a file stream should open and interact with a file.

**Technical Definition**
The `std::ios_base::openmode` type is a bitmask of the constants `app`, `binary`, `in`, `out`, `trunc`, and `ate` (and, since C++23, `noreplace`). They are combined with the bitwise OR operator `|`.

**Beginner-Friendly Explanation**
File modes tell the stream what you intend to do with the file: read it, write it, append to it, treat it as binary, etc. You can combine modes with `|`.

### Purposes

- To control whether a file is opened for reading, writing, or both.
- To specify append vs. truncate behaviour.
- To enable binary (raw byte) I/O.
- To position the file pointer at the beginning or end.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::ios::in | std::ios::out | std::ios::binary
```

**Component Breakdown**

| Mode | Meaning |
|------|---------|
| `in` | Open for reading. |
| `out` | Open for writing. |
| `app` | Seek to end before each write (append). |
| `trunc` | Discard existing contents on open. |
| `ate` | Seek to end immediately after open. |
| `binary` | Open in binary mode (no text translation). |
| `noreplace` (C++23) | Open in exclusive mode (fail if file exists). |

**Syntax Rules**
- Modes are combined with `|`.
- Default modes depend on the stream type: `ifstream` uses `in`; `ofstream` uses `out` (and implicitly `trunc` unless `app` is given); `fstream` uses `in | out`.
- `trunc` is ignored if `app` is also specified.

**Constraints and Limitations**
- `in` alone on a non-existent file fails to open.
- `out` without `app` or `trunc` on an existing file is implementation-defined.
- `noreplace` is C++23 and may not be available on older compilers.

### Annotated Code Examples

**Example 1: Binary append**

```cpp
#include <fstream>

int main() {
    std::ofstream out("data.bin", std::ios::binary | std::ios::app);
    int value = 42;
    out.write(reinterpret_cast<const char*>(&value), sizeof(value));
}
```

**Expected Output**  
(The 4-byte binary representation of `42` is appended to `data.bin`.)

**Why this output?**  
`binary` prevents text translation; `app` appends without truncation.

**Example 2: Read-write without truncation**

```cpp
#include <fstream>

int main() {
    std::fstream fs("data.txt", std::ios::in | std::ios::out);
    // existing content is preserved
}
```

**Expected Output**  
(The file is opened for both reading and writing without being cleared.)

**Why this output?**  
`in | out` without `trunc` preserves the file contents.

### Real-World Cases

- **Binary file formats** — images, executables, serialised data.
- **Log rotation** — append mode for continuous logging.
- **Read-modify-write cycles** — `in | out` without truncation.

### References

- `std::ios_base::openmode` — https://en.cppreference.com/w/cpp/io/ios_base/openmode

---

## 6. File Operations

### Definitions

**Core Definition**
File operations are the member functions used to open, close, and query the status of a file stream.

**Technical Definition**
The key operations are `open()`, `close()`, `is_open()`, and the state-query functions (`good()`, `fail()`, `bad()`, `eof()`). These are provided by `basic_fstream` and its base classes.

**Beginner-Friendly Explanation**
File operations are the basic actions: open a file, close it, and check whether everything went okay.

### Purposes

- To explicitly open and close files.
- To check whether a file is currently associated with a stream.
- To detect errors during I/O.
- To release file handles deterministically.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream.open(filename, mode);
stream.close();
stream.is_open();
stream.good();   // also fail(), bad(), eof()
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `open()` | Associates a file with the stream. |
| `close()` | Closes the associated file. |
| `is_open()` | Returns `true` if a file is open. |
| State queries | Report error conditions. |

**Syntax Rules**
- `open()` fails if the stream is already open.
- `close()` is automatically called by the destructor.
- `is_open()` is the preferred way to check successful opening.

**Constraints and Limitations**
- Calling `open()` on an already-open stream sets `failbit`.
- After `close()`, the stream can be reopened with `open()`.

### Annotated Code Examples

**Example 1: Explicit open and close**

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream in;
    in.open("data.txt");
    if (!in.is_open()) {
        std::cerr << "Could not open file\n";
        return 1;
    }
    // ... use the file ...
    in.close();
}
```

**Expected Output**  
(No output on success; error message if the file cannot be opened.)

**Why this output?**  
`is_open()` verifies that the association succeeded.

**Example 2: RAII automatic closing**

```cpp
#include <fstream>

int main() {
    std::ofstream out("temp.txt");
    out << "Temporary data\n";
    // out closes automatically at end of scope
}
```

**Expected Output**  
(`temp.txt` contains `Temporary data`; the file is closed when `main` returns.)

**Why this output?**  
The `ofstream` destructor closes the file.

### Real-World Cases

- **Conditional file access** — checking `is_open()` before reading.
- **Resource-constrained systems** — explicitly closing files to free handles.
- **Long-running applications** — reopening files for log rotation.

### References

- `std::basic_ifstream` — https://en.cppreference.com/w/cpp/io/basic_ifstream
- `std::basic_ofstream` — https://en.cppreference.com/w/cpp/io/basic_ofstream

---

## 7. File Reading

### Definitions

**Core Definition**
File reading is the process of extracting data from a file into program variables using an input file stream.

**Technical Definition**
Reading from an `ifstream` or `fstream` uses the formatted extraction operator `>>`, unformatted functions like `get()`, `getline()`, and `read()`, all inherited from `std::basic_istream`.

**Beginner-Friendly Explanation**
File reading means pulling data out of a file: you can read words with `>>`, whole lines with `getline`, or raw blocks with `read`.

### Purposes

- To load data from persistent storage.
- To parse structured text files.
- To read binary records.
- To process files line by line or token by token.

### Syntax Rules and Structure

**General Syntax**
```cpp
in >> value;                 // formatted extraction
std::getline(in, line);      // read a line
in.read(buffer, size);       // read raw bytes
in.get(ch);                  // read one character
```

**Component Breakdown**

| Function | Description |
|----------|-------------|
| `>>` | Skips whitespace, extracts formatted value. |
| `getline` | Reads until newline (or delimiter). |
| `read` | Reads a fixed number of bytes. |
| `get` | Reads a single character. |

**Syntax Rules**
- `>>` skips leading whitespace by default.
- `getline` extracts and discards the delimiter.
- `read` sets `failbit` if fewer than `size` characters are read.

**Constraints and Limitations**
- Mixing `>>` and `getline` requires care because `>>` leaves the newline in the buffer.
- Reading past EOF sets `eofbit` and `failbit`.

### Annotated Code Examples

**Example 1: Reading numbers**

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream in("numbers.txt");
    int x;
    while (in >> x) {
        std::cout << x * 2 << '\n';
    }
}
```

**Expected Output**  
(Each number in `numbers.txt` is doubled and printed.)

**Why this output?**  
`>>` extracts integers one by one.

**Example 2: Reading binary data**

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream in("data.bin", std::ios::binary);
    int value;
    in.read(reinterpret_cast<char*>(&value), sizeof(value));
    if (in) std::cout << value << '\n';
}
```

**Expected Output**  
(The integer stored in `data.bin` is printed.)

**Why this output?**  
`read` extracts the raw bytes into `value`.

### Real-World Cases

- **CSV parsers** — reading rows with `getline` and fields with `>>`.
- **Binary asset loaders** — reading textures, models.
- **Log analysers** — processing lines of log data.

### References

- `std::basic_istream` — https://en.cppreference.com/w/cpp/io/basic_istream

---

## 8. File Writing

### Definitions

**Core Definition**
File writing is the process of sending data from program variables into a file using an output file stream.

**Technical Definition**
Writing to an `ofstream` or `fstream` uses the formatted insertion operator `<<`, or unformatted functions like `put()` and `write()`, inherited from `std::basic_ostream`.

**Beginner-Friendly Explanation**
File writing means putting data into a file: `<<` writes formatted values, `put` writes one character, and `write` writes raw bytes.

### Purposes

- To persist data between program runs.
- To generate reports and logs.
- To save binary structures.
- To export computed results.

### Syntax Rules and Structure

**General Syntax**
```cpp
out << value;                  // formatted insertion
out.write(buffer, size);       // write raw bytes
out.put(ch);                   // write one character
```

**Component Breakdown**

| Function | Description |
|----------|-------------|
| `<<` | Inserts formatted data. |
| `write` | Writes a block of bytes. |
| `put` | Writes a single character. |

**Syntax Rules**
- `<<` applies stream formatting (e.g., `std::hex`, `std::setprecision`).
- `write` does not perform any formatting.
- The stream must be open and in a good state.

**Constraints and Limitations**
- Writing to a read-only stream fails.
- Buffering means data may not be immediately on disk until flushed or the stream is closed.

### Annotated Code Examples

**Example 1: Writing formatted text**

```cpp
#include <fstream>
#include <iomanip>

int main() {
    std::ofstream out("report.txt");
    out << "Value: " << std::setw(8) << 42 << '\n';
}
```

**Expected Output**  
(`report.txt` contains `Value:       42`.)

**Why this output?**  
`setw` pads the number to width 8.

**Example 2: Writing binary**

```cpp
#include <fstream>

int main() {
    std::ofstream out("data.bin", std::ios::binary);
    int value = 12345;
    out.write(reinterpret_cast<const char*>(&value), sizeof(value));
}
```

**Expected Output**  
(The 4-byte binary representation of `12345` is written.)

**Why this output?**  
`write` outputs raw bytes without formatting.

### Real-World Cases

- **Logging** — writing timestamped messages.
- **Serialization** — saving object state.
- **File format export** — generating images or documents.

### References

- `std::basic_ostream` — https://en.cppreference.com/w/cpp/io/basic_ostream

---

## 9. File Positioning

### Definitions

**Core Definition**
File positioning is the ability to move and query the read (“get”) and write (“put”) pointers within a file stream.

**Technical Definition**
Input streams provide `seekg()` and `tellg()`; output streams provide `seekp()` and `tellp()`. `seekg`/`seekp` accept either an absolute position or an offset relative to `beg`, `cur`, or `end`. `tellg`/`tellp` return the current position as `pos_type`.

**Beginner-Friendly Explanation**
Think of the file as a long tape with a cursor. `seekg` moves the read cursor, `tellg` tells you where it is; `seekp` moves the write cursor, `tellp` tells you where it is.

### Purposes

- To read or write at a specific location.
- To implement random access.
- To rewind or skip portions of a file.
- To determine the size of a file (by seeking to the end and calling `tellg`).

### Syntax Rules and Structure

**General Syntax**
```cpp
in.seekg(offset, std::ios::beg);   // input
out.seekp(offset, std::ios::end);  // output
auto pos = in.tellg();             // input position
auto pos = out.tellp();            // output position
```

**Component Breakdown**

| Function | Stream | Description |
|----------|--------|-------------|
| `seekg` | `istream` | Sets input position. |
| `tellg` | `istream` | Returns input position. |
| `seekp` | `ostream` | Sets output position. |
| `tellp` | `ostream` | Returns output position. |

**Syntax Rules**
- `seekdir` constants: `std::ios::beg`, `std::ios::cur`, `std::ios::end`.
- `seekg` clears `eofbit` before seeking.
- `tellg`/`tellp` return `pos_type(-1)` on failure.

**Constraints and Limitations**
- Seeking in a text stream is limited (only absolute positions obtained from `tellg` are portable).
- Switching between reading and writing in an `fstream` may require an intervening seek.

### Annotated Code Examples

**Example 1: Rewinding an input file**

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ifstream in("data.txt");
    std::string line;
    std::getline(in, line);
    in.seekg(0, std::ios::beg);        // rewind to start
    std::getline(in, line);            // reads first line again
    std::cout << line << '\n';
}
```

**Expected Output**  
(The first line of `data.txt` is printed.)

**Why this output?**  
`seekg(0, beg)` moves the get pointer back to the beginning.

**Example 2: Getting file size**

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream in("data.bin", std::ios::binary | std::ios::ate);
    auto size = in.tellg();            // position at end = size
    std::cout << "Size: " << size << " bytes\n";
}
```

**Expected Output**  
(`Size: N bytes` where N is the file size.)

**Why this output?**  
`ate` seeks to the end on open; `tellg` returns that position.

### Real-World Cases

- **Database files** — locating fixed-size records.
- **File patching** — writing at specific offsets.
- **Media files** — seeking within streams.

### References

- `seekg` — https://en.cppreference.com/w/cpp/io/basic_istream/seekg
- `tellg` — https://en.cppreference.com/w/cpp/io/basic_istream/tellg
- `seekp` — https://en.cppreference.com/w/cpp/io/basic_ostream/seekp
- `tellp` — https://en.cppreference.com/w/cpp/io/basic_ostream/tellp

---

## 10. File State

### Definitions

**Core Definition**
File state is the set of error flags that indicate the condition of a file stream after an operation.

**Technical Definition**
Every stream maintains an `iostate` bitmask with `goodbit`, `failbit`, `badbit`, and `eofbit`. These can be queried with `good()`, `fail()`, `bad()`, `eof()`, and reset with `clear()`.

**Beginner-Friendly Explanation**
File state tells you whether the last operation succeeded or whether something went wrong (e.g., you tried to read past the end of the file).

### Purposes

- To detect I/O errors.
- To control read loops (read until EOF).
- To recover from failed operations.
- To ensure robust file handling.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream.good();    // no error
stream.fail();    // operation failed
stream.bad();     // irrecoverable error
stream.eof();     // end of file
stream.clear();   // reset state
```

**Component Breakdown**

| Flag | Meaning |
|------|---------|
| `goodbit` | No error. |
| `eofbit` | End of file reached. |
| `failbit` | Logical error (e.g., type mismatch). |
| `badbit` | Irrecoverable stream error. |

**Syntax Rules**
- `fail()` returns `true` if `failbit` **or** `badbit` is set.
- `clear()` with no arguments resets all flags to `goodbit`.
- After `clear()`, the stream can be used again (if the underlying device is still valid).

**Constraints and Limitations**
- `eofbit` alone does not indicate an error; it is normal at end of input.
- Once `badbit` is set, the stream may be unusable without reopening.

### Annotated Code Examples

**Example 1: Reading until EOF**

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream in("data.txt");
    std::string line;
    while (std::getline(in, line)) {
        std::cout << line << '\n';
    }
    if (in.eof()) std::cout << "(End of file)\n";
}
```

**Expected Output**  
(All lines followed by `(End of file)`.)

**Why this output?**  
`getline` fails at EOF, setting `eofbit` and `failbit`; `eof()` confirms the cause.

**Example 2: Clearing a failed state**

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream in("data.txt");
    int x;
    in >> x;
    if (in.fail()) {
        in.clear();                    // reset flags
        std::cout << "Recovered\n";
    }
}
```

**Expected Output**  
(`Recovered` if the first extraction fails.)

**Why this output?**  
`clear()` resets the error flags so the stream can be used again.

### Real-World Cases

- **Robust file parsers** — recovering from malformed input.
- **Data validation** — checking that reads succeeded.
- **Multi-pass processing** — clearing flags before rewinding.

### References

- `std::basic_ios::clear` — https://en.cppreference.com/w/cpp/io/basic_ios/clear
- `std::ios_base::iostate` — https://en.cppreference.com/w/cpp/io/ios_base/iostate

---

## 11. File I/O Best Practices

### Definitions

**Core Definition**
File I/O best practices are guidelines for using file streams safely, efficiently, and idiomatically.

**Technical Definition**
The C++ Core Guidelines (SL.io series) recommend preferring iostreams for type safety and extensibility (SL.io.3), always considering ill-formed input (SL.io.2), and using RAII to manage file handles (R.1). Additional practices include checking `is_open()`, preferring `'\n'` over `std::endl` in loops, and using binary mode for raw data.

**Beginner-Friendly Explanation**
Best practices are good habits: always check that a file opened successfully, use RAII so files close automatically, and be careful with input validation.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **SL.io.2** | Always consider ill-formed input when reading. |
| **SL.io.3** | Prefer iostreams for I/O. |
| **R.1** | Manage resources automatically using RAII. |
| **Check `is_open()`** | Verify the file opened before using it. |
| **Prefer `'\n'`** | Avoid `std::endl` in performance-critical loops. |
| **Use binary mode** | For non-text data, use `std::ios::binary`. |
| **Flush or close before reuse** | Ensure data is written before another process reads it. |
| **Avoid mixing `>>` and `getline`** | Handle the leftover newline carefully. |

### Annotated Code Example

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ofstream out("log.txt", std::ios::app);   // RAII: auto-close
    if (!out.is_open()) {                           // check success
        std::cerr << "Cannot open log\n";
        return 1;
    }
    out << "Entry" << '\n';                         // '\n', not endl
}   // out closes automatically
```

**Expected Output**  
(`Entry` is appended to `log.txt`.)

**Why this output?**  
RAII ensures closing; `is_open()` validates; `'\n'` avoids unnecessary flushing.

### Real-World Cases

- **Production logging** — safe, efficient file writes.
- **Data pipelines** — robust input validation.
- **Cross-platform tools** — consistent binary handling.

### References

- C++ Core Guidelines — SL.io: Iostream — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-io
- C++ Core Guidelines — R.1: Manage resources automatically using RAII — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-raii

---

## References

- `<fstream>` header — https://en.cppreference.com/w/cpp/header/fstream
- `std::basic_ifstream` — https://en.cppreference.com/w/cpp/io/basic_ifstream
- `std::basic_ofstream` — https://en.cppreference.com/w/cpp/io/basic_ofstream
- `std::basic_fstream` — https://en.cppreference.com/w/cpp/io/basic_fstream
- `std::ios_base::openmode` — https://en.cppreference.com/w/cpp/io/ios_base/openmode
- `std::basic_istream::seekg` — https://en.cppreference.com/w/cpp/io/basic_istream/seekg
- `std::basic_istream::tellg` — https://en.cppreference.com/w/cpp/io/basic_istream/tellg
- `std::basic_ostream::seekp` — https://en.cppreference.com/w/cpp/io/basic_ostream/seekp
- `std::basic_ostream::tellp` — https://en.cppreference.com/w/cpp/io/basic_ostream/tellp
- `std::basic_ios::clear` — https://en.cppreference.com/w/cpp/io/basic_ios/clear
- `std::ios_base::iostate` — https://en.cppreference.com/w/cpp/io/ios_base/iostate
- C++ Core Guidelines — SL.io: Iostream — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-io
- C++ Core Guidelines — R.1: Manage resources automatically using RAII — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-raii

---

This cheat sheet provides a comprehensive, structured reference for C++ File I/O, covering every requested component individually, with definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references.