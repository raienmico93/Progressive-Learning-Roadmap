# C++ Serialization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Serialization is the process of converting an object or a group of objects into a flat, one-dimensional sequence of bits (or text) so that it can be stored on disk, sent over a network, or transmitted across processes, and later reconstructed (deserialized) into the original object(s). In C++, this typically involves flattening complex data structures — including nested objects, containers, and pointers — into a format that can be written to a file or stream, and then reversing the process to resurrect the original objects on possibly another machine.

**Technical Definition**
In C++, serialization is the transformation of an in-memory object graph into a byte stream (binary) or a character stream (textual) according to a well-defined format specification. The reverse process, deserialization (or unserialization), reconstructs the object graph from the stream. The C++ standard library does not provide a built-in serialization framework, so developers rely on third-party libraries (nlohmann/json, Protocol Buffers, FlatBuffers, etc.) or custom implementations. The choice of format and library depends on factors such as human readability, cross-language interoperability, performance, schema evolution requirements, and memory constraints.

**Beginner-Friendly Explanation**
Imagine you have a complex LEGO model (your C++ objects) and you want to mail it to a friend. You cannot mail the assembled model directly — you need to take it apart, flatten all the pieces into a box, and ship it. When your friend receives the box, they reassemble the model exactly as it was. Serialization is the process of “taking apart” your objects into a stream of data; deserialization is “reassembling” them. Different serialization libraries are like different packing strategies: some use bubble wrap (JSON, XML — human-readable but bulky), some use vacuum-sealed bags (binary formats like Protobuf — compact and fast), and some use flat-pack furniture (FlatBuffers — no assembly required).

### Key Characteristics

- **Format choice** — textual (JSON, XML, YAML) vs. binary (Protocol Buffers, FlatBuffers, MessagePack, CBOR).
- **Schema-driven vs. schemaless** — Protobuf and FlatBuffers require a schema (`.proto`, `.fbs`); JSON and MessagePack are schemaless.
- **Zero-copy access** — FlatBuffers allows reading data directly from the serialized buffer without deserialization.
- **Human readability** — JSON, XML, and YAML are human-readable; binary formats are not.
- **Cross-language support** — most libraries support multiple programming languages.
- **Performance spectrum** — from high-speed binary formats (simdjson for parsing, FlatBuffers for zero-copy) to ergonomic but slower textual formats (nlohmann/json).
- **Schema evolution** — Protobuf and FlatBuffers are designed for forward/backward compatibility as data schemas change over time.

### Prerequisites

- Solid understanding of C++ classes, structs, and STL containers.
- Familiarity with file I/O (`<fstream>`) and string streams (`<sstream>`).
- Basic knowledge of build systems (CMake) and package managers (vcpkg, Conan).
- Awareness of memory management and performance trade-offs.
- (For schema-driven formats) basic understanding of schema definition languages.

### Related Programming Areas

- **Network programming** — serializing messages for wire transmission.
- **Persistent storage** — saving application state to disk.
- **Inter-process communication (IPC)** — sharing data between processes.
- **Web APIs** — JSON/XML serialization for REST and SOAP services.
- **Game development** — asset and save-game serialization.
- **Distributed systems** — cross-language data exchange.

### Core Concepts / Features

1. Serialization (general concepts)
2. JSON libraries (nlohmann/json, RapidJSON, simdjson)
3. Protocol Buffers
4. FlatBuffers
5. MessagePack
6. CBOR
7. XML libraries (pugixml, TinyXML, libxml2)
8. YAML libraries (yaml-cpp)
9. Binary serialization (general)
10. Serialization best practices

---

## 1. Serialization (General Concepts)

### Definitions

**Core Definition**
Serialization is the umbrella term for converting in-memory data structures into a storable or transmittable format, and deserialization is the inverse operation.

**Technical Definition**
Serialization transforms an object graph into a linear byte or character stream, encoding both the data and its structure according to a format specification. Key design decisions include: whether the format is binary or textual, whether it is schema-driven or self-describing, how it handles pointers and references (object identity, cycles), and how it supports versioning and schema evolution. Common serialization patterns in C++ include custom `serialize`/`deserialize` methods, free functions, and library-provided adapters (e.g., `to_json`/`from_json` for nlohmann/json, `YAML::convert<T>` for yaml-cpp).

**Beginner-Friendly Explanation**
Serialization is the general idea of “flattening” your data so it can be saved or sent. Deserialization is “unflattening” it. Different formats and libraries are just different ways of doing this flattening.

### Purposes

- To persist object state across program runs.
- To transmit data between processes or machines.
- To enable cross-language data exchange.
- To support caching and checkpointing.
- To facilitate testing by comparing serialized representations.

### Syntax Rules and Structure

**General Syntax** (custom serialization pattern)
```cpp
// Member function style
class MyClass {
public:
    template <typename Archive>
    void serialize(Archive& ar) {
        ar & field1 & field2;
    }
};

// Free function style
void to_json(json& j, const MyClass& obj);
void from_json(const json& j, MyClass& obj);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `serialize` / `to_json` / `encode` | Converts object to format. |
| `deserialize` / `from_json` / `decode` | Reconstructs object from format. |
| Archive / stream | Target for serialized data. |

**Syntax Rules**
- The serialization format must define how each type is encoded.
- Object identity and cycles must be handled explicitly for pointer-rich graphs.
- Schema evolution must be considered (adding/removing fields).

**Constraints and Limitations**
- Raw binary dumps of memory are non-portable (endianness, padding).
- Pointers require special handling (graph serialization).

### Annotated Code Example (conceptual)

```cpp
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;

    // Simple textual serialization
    std::string serialize() const {
        return name + "|" + std::to_string(age);
    }

    static Person deserialize(const std::string& s) {
        auto pos = s.find('|');
        return { s.substr(0, pos), std::stoi(s.substr(pos + 1)) };
    }
};

int main() {
    Person p{"Alice", 30};
    std::string data = p.serialize();
    Person q = Person::deserialize(data);
    std::cout << q.name << " " << q.age << '\n';   // Alice 30
}
```

**Expected Output**
```
Alice 30
```

**Why this output?**  
The `serialize` method flattens the object into a `name|age` string, and `deserialize` reconstructs it.

### Real-World Cases

- **Save games** — persisting player state.
- **Configuration files** — storing application settings.
- **Network protocols** — encoding messages.

### References

- Serialization and Unserialization (ISOCPP FAQ) — https://isocpp.org/wiki/faq/serialization

---

## 2. JSON Libraries

### Definitions

**Core Definition**
JSON (JavaScript Object Notation) is a lightweight, human-readable, text-based data interchange format. C++ JSON libraries provide facilities for parsing JSON text into C++ data structures and serializing C++ objects back into JSON.

**Technical Definition**
JSON represents data as objects (key-value maps), arrays, strings, numbers, booleans, and null. C++ JSON libraries differ in their API style (DOM vs. SAX), performance characteristics, memory allocation strategies, and support for features like JSON Pointer, JSON Schema, and ordered objects. The three libraries covered here — nlohmann/json, RapidJSON, and simdjson — represent the ergonomic, balanced, and high-performance extremes of the spectrum.

**Beginner-Friendly Explanation**
JSON looks like a neatly organised list of key-value pairs. A JSON library lets you turn your C++ objects into JSON text (so you can save or send them) and turn JSON text back into C++ objects.

### Purposes

- To exchange data with web services and APIs.
- To store configuration and structured data in a human-readable format.
- To enable cross-language data interchange.
- To provide a schemaless, flexible data format.

### Key Characteristics

| Library | API Style | Speed | Ease of Use | Memory Model |
|---------|-----------|-------|-------------|--------------|
| nlohmann/json | DOM, STL-like | ~400 MB/s | Excellent | Dynamic allocation |
| RapidJSON | SAX + DOM | Fast (comparable to `strlen`) | Moderate | 16 bytes/value, custom allocator |
| simdjson | DOM + On-Demand | 4× RapidJSON, 25× nlohmann | Moderate | SIMD-accelerated, zero-allocation possible |

### Syntax Rules and Structure

**nlohmann/json general syntax**
```cpp
#include <nlohmann/json.hpp>
using json = nlohmann::json;

json j = { {"name", "Alice"}, {"age", 30} };
std::string s = j.dump();
auto parsed = json::parse(s);
```

**RapidJSON general syntax**
```cpp
#include "rapidjson/document.h"
rapidjson::Document d;
d.Parse(json_str);
```

**simdjson general syntax**
```cpp
#include <simdjson.h>
simdjson::dom::parser parser;
simdjson::dom::element doc = parser.parse(json_str);
```

### Annotated Code Examples

**nlohmann/json example**
```cpp
#include <nlohmann/json.hpp>
#include <iostream>

using json = nlohmann::json;

int main() {
    json j;
    j["name"] = "Alice";
    j["age"] = 30;
    std::string s = j.dump(2);       // pretty print
    std::cout << s << '\n';
}
```

**Expected Output**
```json
{
  "age": 30,
  "name": "Alice"
}
```

**Why this output?**  
nlohmann/json stores keys sorted alphabetically by default (using `std::map`).

**RapidJSON example**
```cpp
#include "rapidjson/document.h"
#include <iostream>

int main() {
    const char* json = R"({"name":"Alice","age":30})";
    rapidjson::Document d;
    d.Parse(json);
    std::cout << d["name"].GetString() << '\n';
}
```

**Expected Output**
```
Alice
```

**Why this output?**  
RapidJSON parses the JSON into a DOM and provides accessors like `GetString()`.

**simdjson example**
```cpp
#include <simdjson.h>
#include <iostream>

int main() {
    simdjson::dom::parser parser;
    auto doc = parser.parse(R"({"name":"Alice","age":30})"_padded);
    std::cout << doc["name"] << '\n';
}
```

**Expected Output**
```
Alice
```

**Why this output?**  
simdjson uses SIMD instructions to parse JSON at high speed.

### Real-World Cases

- **Web API clients** — parsing JSON responses.
- **Configuration** — reading JSON config files.
- **Log processing** — structured logging in JSON.

### References

- nlohmann/json — https://json.nlohmann.me/
- RapidJSON — https://rapidjson.org/
- simdjson — https://simdjson.org/

---

## 3. Protocol Buffers

### Definitions

**Core Definition**
Protocol Buffers (Protobuf) is a language-neutral, platform-neutral, extensible mechanism for serializing structured data, developed by Google.

**Technical Definition**
Protobuf uses a `.proto` schema file to define message types. The `protoc` compiler generates C++ classes with getters/setters for fields, plus `SerializeToString()` and `ParseFromString()` methods. The wire format is a compact binary encoding that supports schema evolution — fields can be added or deprecated without breaking compatibility, provided field numbers are not reused.

**Beginner-Friendly Explanation**
Protobuf is like a highly efficient binary JSON with a strict schema. You define your data structure in a `.proto` file, and Protobuf generates C++ code that can pack your data into a tiny binary blob and unpack it back. It is designed so that old code can read data written by new code (and vice versa), as long as you follow the field-numbering rules.

### Purposes

- To enable efficient, compact binary serialization.
- To support schema evolution and backward compatibility.
- To provide cross-language data interchange.
- To reduce payload sizes compared to JSON/XML.

### Syntax Rules and Structure

**General Syntax** (`.proto` file)
```protobuf
syntax = "proto3";

message Person {
    string name = 1;
    int32 age = 2;
    repeated string emails = 3;
}
```

**Generated C++ usage**
```cpp
#include "person.pb.h"

Person p;
p.set_name("Alice");
p.set_age(30);

std::string binary;
p.SerializeToString(&binary);

Person q;
q.ParseFromString(binary);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `.proto` file | Schema definition. |
| `protoc` | Compiler that generates C++ code. |
| Generated class | Provides getters/setters and serialization methods. |
| `SerializeToString()` | Serializes to binary string. |
| `ParseFromString()` | Deserializes from binary string. |

**Syntax Rules**
- Field numbers must be unique and never reused after deletion.
- `proto3` is the current recommended syntax.
- Fields can be `optional`, `repeated`, or part of `oneof`.

**Constraints and Limitations**
- Requires a build step (`protoc`).
- Generated code can be large.
- Not human-readable.

### Annotated Code Example

```cpp
// person.proto
syntax = "proto3";
message Person {
    string name = 1;
    int32 age = 2;
}
```

```cpp
// main.cpp
#include "person.pb.h"
#include <iostream>

int main() {
    Person p;
    p.set_name("Alice");
    p.set_age(30);

    std::string binary;
    p.SerializeToString(&binary);
    std::cout << "Bytes: " << binary.size() << '\n';

    Person q;
    q.ParseFromString(binary);
    std::cout << q.name() << ' ' << q.age() << '\n';
}
```

**Expected Output**
```
Bytes: 8
Alice 30
```

**Why this output?**  
The binary encoding of `"Alice"` (5 bytes + tag) and `30` (1 byte + tag) totals 8 bytes; deserialization reconstructs the object.

### Real-World Cases

- **gRPC** — default serialization format for gRPC services.
- **Microservices** — inter-service communication.
- **Data pipelines** — efficient storage and transmission.

### References

- Protocol Buffers C++ Tutorial — https://protobuf.dev/getting-started/cpptutorial/
- Protocol Buffers Language Guide — https://protobuf.dev/programming-guides/proto3/

---

## 4. FlatBuffers

### Definitions

**Core Definition**
FlatBuffers is an efficient cross-platform serialization library that enables zero-copy access to serialized data without a parsing/unpacking step.

**Technical Definition**
FlatBuffers uses a `.fbs` schema file compiled by the `flatc` compiler to generate C++ accessor classes. The serialized buffer is a single contiguous block of memory that can be read directly — accessors return pointers into the buffer without deserialization. This is achieved by storing data in a format that allows direct memory mapping, with offsets instead of pointers. It is designed for latency-sensitive applications such as games and real-time systems.

**Beginner-Friendly Explanation**
FlatBuffers is like a “flat-pack” serialization format: the data is laid out in memory in exactly the form it will be read, so you can access any field directly from the buffer without unpacking anything. This makes it extremely fast for reading, at the cost of slightly slower writes.

### Purposes

- To provide zero-copy deserialization.
- To minimise latency in read-heavy workloads.
- To support schema evolution with backward compatibility.
- To enable memory-mapped file access.

### Syntax Rules and Structure

**General Syntax** (`.fbs` file)
```flatbuffers
table Monster {
    name: string;
    hp: short;
    mana: short;
}
root_type Monster;
```

**Generated C++ usage**
```cpp
#include "monster_generated.h"

// Writing
flatbuffers::FlatBufferBuilder builder;
auto name = builder.CreateString("MyMonster");
auto monster = CreateMonster(builder, name, 80, 150);
builder.Finish(monster);

// Reading (no deserialization)
auto m = GetMonster(builder.GetBufferPointer());
std::cout << m->name()->c_str() << '\n';
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `.fbs` schema | Defines data structures. |
| `flatc` | Compiles schema to C++ code. |
| `FlatBufferBuilder` | Constructs the buffer. |
| `GetMonster()` | Returns a pointer into the buffer. |

**Syntax Rules**
- All fields are optional with defaults.
- Schema evolution supports adding/removing fields while preserving compatibility.
- Buffers are immutable once built (mutable tables are possible with `flatc`).

**Constraints and Limitations**
- Building the buffer is slower than Protobuf.
- Not human-readable.
- Larger buffer size than Protobuf for some data.

### Annotated Code Example

```cpp
// monster.fbs
table Monster {
    name: string;
    hp: short;
    mana: short;
}
root_type Monster;
```

```cpp
// main.cpp
#include "monster_generated.h"
#include <iostream>

int main() {
    flatbuffers::FlatBufferBuilder builder;
    auto name = builder.CreateString("MyMonster");
    auto monster = CreateMonster(builder, name, 80, 150);
    builder.Finish(monster);

    auto m = GetMonster(builder.GetBufferPointer());
    std::cout << m->name()->c_str() << ' '
              << m->hp() << ' ' << m->mana() << '\n';
}
```

**Expected Output**
```
MyMonster 80 150
```

**Why this output?**  
The data is read directly from the buffer without parsing; accessors return values from the serialized layout.

### Real-World Cases

- **Game engines** — asset serialization and network messages.
- **Real-time systems** — low-latency data access.
- **Mobile applications** — memory-mapped data files.

### References

- FlatBuffers C++ Usage — https://flatbuffers.dev/languages/cpp/
- FlatBuffers Tutorial — https://flatbuffers.dev/tutorial/

---

## 5. MessagePack

### Definitions

**Core Definition**
MessagePack is an efficient binary serialization format that is like JSON but smaller and faster.

**Technical Definition**
MessagePack encodes data types (integers, floats, strings, binary, arrays, maps) into a compact binary format. Small integers occupy a single byte, and short strings require only one extra byte beyond the string content itself. The C++ implementation (`msgpack-c`) is header-only for C++03/C++11 and provides `msgpack::pack` and `msgpack::unpack` functions, along with type adaptors for standard containers.

**Beginner-Friendly Explanation**
MessagePack is “binary JSON” — it stores the same kinds of data as JSON but in a compact binary form. It is faster and smaller than JSON, at the cost of being unreadable by humans.

### Purposes

- To reduce payload size compared to JSON.
- To improve serialization/deserialization speed.
- To provide a schemaless binary format.
- To enable cross-language data exchange.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <msgpack.hpp>

// Pack
std::stringstream buffer;
msgpack::pack(buffer, std::make_tuple(1, true, "example"));

// Unpack
std::string str = buffer.str();
msgpack::object_handle oh = msgpack::unpack(str.data(), str.size());
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `msgpack::pack` | Serializes objects to a buffer. |
| `msgpack::unpack` | Deserializes a buffer into `msgpack::object`. |
| `convert()` | Converts `msgpack::object` to a typed value. |

**Syntax Rules**
- Supports all standard containers and tuples.
- Custom types can be adapted via `MSGPACK_DEFINE`.
- The deserialized `object_handle` must outlive the object.

**Constraints and Limitations**
- Not human-readable.
- No schema; validation is application-dependent.

### Annotated Code Example

```cpp
#include <msgpack.hpp>
#include <iostream>
#include <sstream>

int main() {
    msgpack::type::tuple<int, bool, std::string> src(1, true, "example");
    std::stringstream buffer;
    msgpack::pack(buffer, src);

    std::string str = buffer.str();
    msgpack::object_handle oh = msgpack::unpack(str.data(), str.size());
    msgpack::object deserialized = oh.get();

    msgpack::type::tuple<int, bool, std::string> dst;
    deserialized.convert(dst);
    std::cout << dst.get<0>() << ' ' << dst.get<1>()
              << ' ' << dst.get<2>() << '\n';
}
```

**Expected Output**
```
1 1 example
```

**Why this output?**  
The tuple is packed into a compact binary buffer and unpacked back into the original types.

### Real-World Cases

- **Real-time messaging** — compact message payloads.
- **Game state synchronization** — low-latency network updates.
- **Embedded systems** — efficient binary encoding.

### References

- msgpack-c GitHub — https://github.com/msgpack/msgpack-c
- MessagePack specification — https://msgpack.org/

---

## 6. CBOR

### Definitions

**Core Definition**
CBOR (Concise Binary Object Representation) is a binary data serialization format designed for small encoded sizes and extensibility without version negotiation.

**Technical Definition**
CBOR is defined in RFC 8949. It is a superset of the JSON data model, supporting additional data types (byte strings, tags, indefinite-length items) and custom type definitions via tags. The C++ ecosystem offers several libraries, including `cbor_tags` (C++20, with automatic reflection) and `LibCppBor` (C++17, DOM-style). CBOR is an IETF standard and is designed for interoperability with other languages and systems.

**Beginner-Friendly Explanation**
CBOR is a binary format that is more compact than JSON but supports the same basic data types plus some extras (like binary blobs and tags). It is an official internet standard, which makes it suitable for interoperability.

### Purposes

- To provide a compact binary encoding of JSON-like data.
- To support extensibility via tags.
- To enable interoperability with IETF-standard protocols (e.g., COSE, CWT).
- To reduce message sizes in constrained environments.

### Syntax Rules and Structure

**General Syntax** (using `cbor_tags`)
```cpp
#include <cbor_tags/cbor_encoder.h>
#include <cbor_tags/cbor_decoder.h>

std::vector<std::byte> data;
auto enc = cbor::tags::make_encoder(data);
enc(2, 3.14, "Hello, World!"sv);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Encoder | `make_encoder(buffer)` creates an encoder. |
| Decoder | `make_decoder(buffer)` creates a decoder. |
| Tags | Extend the data model with custom types. |

**Syntax Rules**
- CBOR supports integers, floats, strings, byte strings, arrays, maps, tags, and simple values.
- Indefinite-length items allow streaming.
- Tags provide extensibility without version negotiation.

**Constraints and Limitations**
- Less human-readable than JSON.
- Library support in C++ is less mature than JSON or Protobuf.

### Annotated Code Example (using `cbor_tags`)

```cpp
#include "cbor_tags/cbor_encoder.h"
#include "cbor_tags/cbor_decoder.h"
#include <iostream>
#include <vector>

using namespace cbor::tags;

int main() {
    std::vector<std::byte> data;
    auto enc = make_encoder(data);
    enc(42, 3.14, "CBOR"sv);

    auto dec = make_decoder(data);
    int i; double d; std::string_view s;
    dec(i, d, s);
    std::cout << i << ' ' << d << ' ' << s << '\n';
}
```

**Expected Output**
```
42 3.14 CBOR
```

**Why this output?**  
The values are encoded into CBOR binary and decoded back into their original types.

### Real-World Cases

- **IoT protocols** — COSE (CBOR Object Signing and Encryption).
- **Constrained devices** — compact binary messages.
- **WebAuthn** — CBOR-encoded attestation objects.

### References

- CBOR RFC 8949 — https://datatracker.ietf.org/doc/html/rfc8949
- cbor_tags GitHub — https://github.com/jkammerland/cbor_tags
- LibCppBor — https://android.googlesource.com/platform/external/libcppbor/

---

## 7. XML Libraries

### Definitions

**Core Definition**
XML (eXtensible Markup Language) is a text-based markup language for encoding structured data in a human-readable format. C++ XML libraries provide parsing, manipulation, and generation of XML documents.

**Technical Definition**
XML documents consist of elements (tags), attributes, text content, and nested structures. C++ XML libraries vary in their parsing model (DOM vs. SAX), feature set (XPath, schema validation), and resource footprint. The three libraries covered here — pugixml, TinyXML-2, and libxml2 — represent the lightweight DOM, minimalist DOM, and full-featured (DOM + SAX + validation) ends of the spectrum.

**Beginner-Friendly Explanation**
XML looks like HTML with custom tags. A C++ XML library lets you read XML files into a tree you can navigate, or build XML trees and write them to files.

### Purposes

- To parse configuration files and data exchange documents.
- To generate XML for web services (SOAP, RSS).
- To validate documents against schemas (DTD, XSD).
- To query XML trees with XPath.

### Key Characteristics

| Library | Model | XPath | Schema Validation | Best For |
|---------|-------|-------|-------------------|----------|
| pugixml | DOM | Yes (XPath 1.0) | No | Fast DOM parsing with queries |
| TinyXML-2 | DOM | No | No | Simple config files, small payloads |
| libxml2 | DOM, SAX, pull | Yes | DTD, XSD, RelaxNG | Large files, validation |

### Syntax Rules and Structure

**pugixml general syntax**
```cpp
#include "pugixml.hpp"
pugi::xml_document doc;
doc.load_string(xml_str);
auto node = doc.child("root").child("item");
```

**TinyXML-2 general syntax**
```cpp
#include "tinyxml2.h"
tinyxml2::XMLDocument doc;
doc.Parse(xml_str);
auto elem = doc.FirstChildElement("root");
```

**libxml2 general syntax**
```cpp
#include <libxml/parser.h>
xmlDocPtr doc = xmlParseFile("file.xml");
xmlNodePtr root = xmlDocGetRootElement(doc);
```

### Annotated Code Examples

**pugixml example**
```cpp
#include "pugixml.hpp"
#include <iostream>

int main() {
    pugi::xml_document doc;
    doc.load_string("<root><item id='1'>Hello</item></root>");
    auto item = doc.child("root").child("item");
    std::cout << item.text().as_string() << '\n';
}
```

**Expected Output**
```
Hello
```

**Why this output?**  
pugixml parses the XML into a DOM and provides `text()` for element content.

**TinyXML-2 example**
```cpp
#include "tinyxml2.h"
#include <iostream>

int main() {
    tinyxml2::XMLDocument doc;
    doc.Parse("<root><item id='1'>Hello</item></root>");
    auto item = doc.FirstChildElement("root")->FirstChildElement("item");
    std::cout << item->GetText() << '\n';
}
```

**Expected Output**
```
Hello
```

**Why this output?**  
TinyXML-2 builds a DOM and provides `GetText()` for element content.

**libxml2 example**
```cpp
#include <libxml/parser.h>
#include <iostream>

int main() {
    xmlDocPtr doc = xmlParseMemory("<root><item>Hello</item></root>", 35);
    xmlNodePtr root = xmlDocGetRootElement(doc);
    xmlNodePtr item = root->children;
    std::cout << (const char*)item->children->content << '\n';
    xmlFreeDoc(doc);
}
```

**Expected Output**
```
Hello
```

**Why this output?**  
libxml2 parses the XML into a tree of `xmlNode` structures.

### Real-World Cases

- **Configuration files** — application settings.
- **SOAP web services** — XML-based RPC.
- **Document formats** — SVG, RSS, Office Open XML.

### References

- pugixml — https://pugixml.org/
- TinyXML-2 — https://github.com/leethomason/tinyxml2
- libxml2 — https://gitlab.gnome.org/GNOME/libxml2

---

## 8. YAML Libraries (yaml-cpp)

### Definitions

**Core Definition**
YAML (YAML Ain't Markup Language) is a human-friendly data serialization language. yaml-cpp is the de facto standard C++ library for parsing and emitting YAML.

**Technical Definition**
yaml-cpp models YAML documents as a tree of `YAML::Node` objects. Parsing is performed by `YAML::Load` or `YAML::LoadFile`, which returns a `Node`. Values are accessed via `node["key"].as<T>()`. Serialization is done by streaming a `Node` to an output stream. Custom types are supported by specialising `YAML::convert<T>` with `encode` and `decode` functions. yaml-cpp supports YAML 1.2 and is MIT-licensed.

**Beginner-Friendly Explanation**
yaml-cpp is a library that reads YAML files (which look like indented lists) into a tree structure you can navigate, and writes C++ data structures out as YAML. It is especially good for configuration files.

### Purposes

- To read and write YAML configuration files.
- To support human-readable data serialization.
- To enable custom type conversion via `YAML::convert<T>`.
- To integrate with C++ applications via CMake.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <yaml-cpp/yaml.h>

YAML::Node doc = YAML::Load("key: value");
std::string value = doc["key"].as<std::string>();

YAML::Node out;
out["key"] = "value";
std::cout << out;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `YAML::Load` / `LoadFile` | Parses YAML into a `Node`. |
| `node["key"].as<T>()` | Extracts a typed value. |
| `YAML::convert<T>` | Custom encode/decode for user types. |
| `operator<<` | Emits YAML to a stream. |

**Syntax Rules**
- Missing keys return an undefined node (tests `false`).
- `as<T>()` throws `YAML::BadConversion` on type mismatch.
- Custom types require `YAML::convert<T>` specialisation.

**Constraints and Limitations**
- yaml-cpp is not header-only; it must be linked.
- YAML is indentation-sensitive, which can be error-prone.

### Annotated Code Example

```cpp
#include <yaml-cpp/yaml.h>
#include <iostream>
#include <string>

struct Server {
    std::string host;
    int port;
};

namespace YAML {
template <>
struct convert<Server> {
    static Node encode(const Server& s) {
        Node n;
        n["host"] = s.host;
        n["port"] = s.port;
        return n;
    }
    static bool decode(const Node& n, Server& s) {
        if (!n["host"] || !n["port"]) return false;
        s.host = n["host"].as<std::string>();
        s.port = n["port"].as<int>();
        return true;
    }
};
}

int main() {
    YAML::Node doc = YAML::Load("host: localhost\nport: 8080");
    Server s = doc.as<Server>();
    std::cout << s.host << ':' << s.port << '\n';
}
```

**Expected Output**
```
localhost:8080
```

**Why this output?**  
The `convert<Server>` specialisation tells yaml-cpp how to decode the YAML node into a `Server` struct.

### Real-World Cases

- **Configuration management** — Kubernetes, Docker Compose, CI/CD.
- **Application settings** — user-editable config files.
- **Data serialization** — human-readable data interchange.

### References

- yaml-cpp GitHub — https://github.com/jbeder/yaml-cpp
- yaml-cpp Tutorial — https://github.com/jbeder/yaml-cpp/wiki/Tutorial

---

## 9. Binary Serialization (General)

### Definitions

**Core Definition**
Binary serialization encodes data into a compact binary format (as opposed to text), typically for performance and size efficiency.

**Technical Definition**
Binary serialization formats store data in its native binary representation or a packed binary encoding, avoiding the overhead of text encoding and parsing. Examples include Protocol Buffers, FlatBuffers, MessagePack, CBOR, and custom binary formats. Performance benchmarks show that binary formats (CBOR, MessagePack, Protobuf) achieve substantial advantages over JSON in encoding speed and payload compactness, with minimal heap overhead — CBOR and MessagePack incur ~144 bytes of first-call container allocation then 0B incremental per-message, while Protobuf achieves true zero via arena pre-allocation. For numeric data, the size differences are dramatic: a `std::vector<float>` of 10K elements serializes to 124.11 KB in JSON, 39.07 KB in BEVE, 48.83 KB in MessagePack, 39.07 KB in CBOR, and 39.07 KB in Protobuf.

**Beginner-Friendly Explanation**
Binary serialization is like compressing your data into a tight binary blob. It is much smaller and faster to process than text formats, but you cannot read it with a text editor.

### Purposes

- To minimise payload size for storage or transmission.
- To maximise serialization/deserialization speed.
- To reduce CPU overhead in high-throughput systems.
- To avoid the ambiguity of text-based formats.

### Key Characteristics

| Format | Size (vector<float>, 10K) | Speed | Schema |
|--------|---------------------------|-------|--------|
| JSON | 124.11 KB | Slower | No |
| MessagePack | 48.83 KB | Fast | No |
| CBOR | 39.07 KB | Fast | No |
| Protobuf | 39.07 KB | Fast | Yes (schema) |

### Annotated Code Example (custom binary)

```cpp
#include <fstream>
#include <cstdint>
#include <iostream>

struct Record {
    uint32_t id;
    double value;
};

int main() {
    Record r{42, 3.14};
    std::ofstream out("data.bin", std::ios::binary);
    out.write(reinterpret_cast<const char*>(&r), sizeof(r));
    out.close();

    Record r2;
    std::ifstream in("data.bin", std::ios::binary);
    in.read(reinterpret_cast<char*>(&r2), sizeof(r2));
    std::cout << r2.id << ' ' << r2.value << '\n';
}
```

**Expected Output**
```
42 3.14
```

**Why this output?**  
The raw bytes of the struct are written and read back. Note: this is not portable across different architectures due to endianness and padding.

### Real-World Cases

- **High-frequency trading** — minimal latency.
- **Game engines** — fast asset loading.
- **Embedded systems** — constrained memory and bandwidth.

### References

- Binary serialization performance benchmarks — https://github.com/stephenberry/binary_performance
- FlatBuffers benchmarks — https://flatbuffers.dev/benchmarks/

---

## 10. Serialization Best Practices

### Definitions

**Core Definition**
Serialization best practices are guidelines for choosing the right format and library, designing serializable types, and handling versioning, security, and performance.

**Technical Definition**
Best practices encompass: selecting between text and binary formats based on requirements (human readability vs. performance), using schema-driven formats (Protobuf, FlatBuffers) when cross-language interoperability and schema evolution are important, handling object graphs and pointers correctly (avoiding cycles, preserving identity), validating untrusted input, and considering security implications (deserialization attacks). The C++ Core Guidelines and ISOCPP FAQ provide foundational guidance on serialization design.

**Beginner-Friendly Explanation**
Best practices are good habits: choose the right format for the job (JSON for readability, Protobuf for efficiency), always validate data you receive, and design your classes so they can be serialized cleanly.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Choose the right format** | Text for readability, binary for performance. |
| **Use schemas for evolution** | Protobuf/FlatBuffers for forward/backward compatibility. |
| **Handle pointers carefully** | Object graphs with cycles require special handling. |
| **Validate untrusted input** | Never trust deserialized data. |
| **Version your formats** | Include version fields for backward compatibility. |
| **Benchmark before optimising** | Measure serialization performance in context. |
| **Avoid raw memory dumps** | Use well-defined formats for portability. |
| **Prefer zero-copy when latency matters** | FlatBuffers for read-heavy workloads. |

### Annotated Code Example (validating deserialized data)

```cpp
#include <nlohmann/json.hpp>
#include <iostream>

using json = nlohmann::json;

int main() {
    std::string input = R"({"age": "not a number"})";
    try {
        json j = json::parse(input);
        int age = j.at("age").get<int>();   // throws on type mismatch
        std::cout << age << '\n';
    } catch (const json::exception& e) {
        std::cerr << "Invalid data: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Invalid data: [json.exception.type_error.302] type must be number, but is string
```

**Why this output?**  
`get<int>()` throws `type_error` because the JSON value is a string, not a number. Always validate.

### Real-World Cases

- **API security** — preventing deserialization attacks.
- **Microservices** — ensuring schema compatibility.
- **Data pipelines** — reliable data exchange.

### References

- Serialization and Unserialization (ISOCPP FAQ) — https://isocpp.org/wiki/faq/serialization
- C++ Core Guidelines — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

## References

- Serialization and Unserialization (ISOCPP FAQ) — https://isocpp.org/wiki/faq/serialization
- nlohmann/json — https://json.nlohmann.me/
- RapidJSON — https://rapidjson.org/
- simdjson — https://simdjson.org/
- Protocol Buffers C++ Tutorial — https://protobuf.dev/getting-started/cpptutorial/
- FlatBuffers C++ Usage — https://flatbuffers.dev/languages/cpp/
- MessagePack (msgpack-c) — https://github.com/msgpack/msgpack-c
- CBOR RFC 8949 — https://datatracker.ietf.org/doc/html/rfc8949
- cbor_tags GitHub — https://github.com/jkammerland/cbor_tags
- pugixml — https://pugixml.org/
- TinyXML-2 — https://github.com/leethomason/tinyxml2
- libxml2 — https://gitlab.gnome.org/GNOME/libxml2
- yaml-cpp — https://github.com/jbeder/yaml-cpp
- yaml-cpp Tutorial — https://github.com/jbeder/yaml-cpp/wiki/Tutorial
- Binary serialization performance benchmarks — https://github.com/stephenberry/binary_performance
- FlatBuffers benchmarks — https://flatbuffers.dev/benchmarks/
- C++ Core Guidelines — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

This cheat sheet provides a comprehensive, structured reference for C++ serialization, covering every requested format and library individually, with definitions, purposes, syntax rules, annotated code examples, real-world cases, performance considerations, and authoritative references.