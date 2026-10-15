# JavaScript Typed Arrays — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Typed arrays are array-like objects that provide a mechanism for reading and writing raw binary data in memory buffers, with each type representing a specific numeric format and fixed element size.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §25), a TypedArray is an exotic object that presents an array-like view over an `ArrayBuffer` (or `SharedArrayBuffer`). Unlike standard arrays, typed arrays have a fixed element type, a fixed length, and store data in a contiguous block of memory. There are eleven typed array constructors: `Int8Array`, `Uint8Array`, `Uint8ClampedArray`, `Int16Array`, `Uint16Array`, `Int32Array`, `Uint32Array`, `Float32Array`, `Float64Array`, `BigInt64Array`, and `BigUint64Array`. All share `%TypedArray%` as their common prototype, and each is backed by an `ArrayBuffer` accessible via the `.buffer` property. The `DataView` interface provides a lower-level, endianness-aware view over an `ArrayBuffer`.

**Beginner-Friendly Explanation:** Regular JavaScript arrays are flexible — they can hold any type of value, grow and shrink, and each element takes up as much memory as it needs. Typed arrays are different: they hold only numbers of one specific type (like 8-bit integers or 64-bit floats), have a fixed size, and pack the data tightly in memory. This makes them much faster and more memory-efficient for numeric work, and they're essential for handling binary data like images, audio, files, and network protocols.

### Key Characteristics

- **Fixed type:** Each typed array holds only one numeric type.
- **Fixed length:** Length is set at creation and cannot change.
- **Contiguous memory:** Data is stored in a single `ArrayBuffer`.
- **Raw binary access:** Direct byte-level manipulation.
- **Shared buffer:** Multiple views can share one `ArrayBuffer`.
- **Platform-independent:** Sizes are fixed regardless of platform.
- **Not a subclass of Array:** `Array.isArray()` returns `false`.
- **Endianness:** `DataView` exposes endianness control; typed arrays use platform endianness.

### Prerequisites

- Basic JavaScript syntax (arrays, numbers, constructors).
- Understanding of binary representation and bits/bytes.
- Familiarity with array-like objects.
- Awareness of `ArrayBuffer` and memory.
- Basic knowledge of numeric types (integers, floats).

### Related Programming Areas

- **Binary data:** Files, network protocols, images, audio.
- **WebGL:** Vertex buffers, textures.
- **WebAssembly:** Linear memory.
- **Canvas:** Pixel manipulation via `ImageData`.
- **Cryptography:** Byte-level operations.
- **Performance:** Numerical computation.

---

## Typed Arrays

### Definitions

**Core Definition:** A typed array is an array-like view over a contiguous block of binary memory, with a fixed element type.

**Technical Definition:** Per ECMA-262 §25.1, a TypedArray is an exotic object with integer-indexed properties and a `[[ViewedArrayBuffer]]` internal slot. Each element type has a fixed byte size: `Int8Array` (1 byte), `Int16Array` (2 bytes), `Int32Array` (4 bytes), `Float32Array` (4 bytes), `Float64Array` (8 bytes), and BigInt variants (8 bytes). Elements are accessed via integer indices in the range `[0, length - 1]`. The `buffer` property exposes the underlying `ArrayBuffer`.

**Beginner-Friendly Explanation:** A typed array is a special kind of array that holds only one type of number and stores data in a compact binary block. It's fast and efficient but less flexible than a regular array.

### Purposes

- To store and manipulate binary data efficiently.
- To work with WebGL, WebAssembly, and Canvas.
- To process files and network data.
- To perform numerical computation with high performance.
- To interoperate with native code and hardware.

### Syntax Rules and Structure

**Basic creation:**
```js
new Int8Array(length)         // From length
new Int8Array(array)          // From array
new Int8Array(buffer)         // From ArrayBuffer
new Int8Array(buffer, offset, length) // View over buffer
Int8Array.from(iterable)      // From iterable
Int8Array.of(1, 2, 3)         // From arguments
```

**Common properties:**
```js
ta.length         // Number of elements
ta.byteLength     // Size in bytes
ta.byteOffset     // Offset in the buffer
ta.buffer         // Underlying ArrayBuffer
ta.BYTES_PER_ELEMENT // Size of each element
```

**Constraints and Limitations:**
- Fixed length; cannot grow or shrink.
- Only numbers (or BigInts) can be stored; other values are coerced.
- Out-of-bounds reads return `undefined`; writes are ignored.
- No `push`, `pop`, `splice`, or `concat` (immutable-length methods).
- `Array.isArray()` returns `false`.

### Annotated Code Examples

#### Example 1: Creating Typed Arrays

```js
// Step 1: From length.
const i8 = new Int8Array(4);
console.log(i8); // Int8Array(4) [0, 0, 0, 0]

// Step 2: From array.
const u8 = new Uint8Array([10, 20, 30]);
console.log(u8); // Uint8Array(3) [10, 20, 30]

// Step 3: From ArrayBuffer.
const buffer = new ArrayBuffer(8);
const i32 = new Int32Array(buffer);
console.log(i32.length); // 2 (8 bytes / 4 bytes per element)

// Step 4: Properties.
console.log(u8.length);            // 3
console.log(u8.byteLength);        // 3
console.log(u8.byteOffset);        // 0
console.log(u8.BYTES_PER_ELEMENT); // 1
```

**Expected Output:**
```
Int8Array(4) [ 0, 0, 0, 0 ]
Uint8Array(3) [ 10, 20, 30 ]
2
3
3
0
1
```

**Why This Output:** Typed arrays expose element count, byte size, and the underlying buffer.

#### Example 2: Shared Buffer

```js
const buffer = new ArrayBuffer(8);

// Step 1: Two views over the same buffer.
const i8 = new Int8Array(buffer);
const i32 = new Int32Array(buffer);

// Step 2: Write via i32; read via i8.
i32[0] = 0x01020304;
console.log(i8[0]); // 4 (little-endian)
console.log(i8[1]); // 3
console.log(i8[2]); // 2
console.log(i8[3]); // 1
```

**Expected Output:**
```
4
3
2
1
```

**Why This Output:** Both views share the same memory; writing via `i32` affects the bytes read via `i8` (little-endian on most platforms).

### Real-World Cases

- **Image processing:** `ImageData` uses `Uint8ClampedArray`.
- **WebGL:** Vertex and index buffers.
- **WebAssembly:** Linear memory is an `ArrayBuffer`.
- **Audio:** `AudioBuffer` uses `Float32Array`.

---

## `Int8Array`

### Definitions

**Core Definition:** `Int8Array` is a typed array of 8-bit signed integers (range −128 to 127).

**Technical Definition:** Per ECMA-262 §25.2.4, `Int8Array` elements are 1 byte each, interpreted as two's complement signed integers in the range [−128, 127].

**Beginner-Friendly Explanation:** `Int8Array` holds small signed numbers, from −128 to 127.

### Annotated Code Examples

```js
const arr = new Int8Array([127, -128, 0]);
console.log(arr); // Int8Array(3) [127, -128, 0]

// Values wrap on overflow.
const wrapped = new Int8Array([128, 200]);
console.log(wrapped); // Int8Array(2) [-128, -56]
```

**Expected Output:**
```
Int8Array(3) [ 127, -128, 0 ]
Int8Array(2) [ -128, -56 ]
```

**Why This Output:** Values outside [−128, 127] wrap modulo 256.

### Real-World Cases

- **Audio samples:** Small signed PCM values.
- **Compact storage:** When range fits in 8 bits.

---

## `Uint8Array`

### Definitions

**Core Definition:** `Uint8Array` is a typed array of 8-bit unsigned integers (range 0 to 255).

**Technical Definition:** Per ECMA-262 §25.2.5, `Uint8Array` elements are 1 byte each, interpreted as unsigned integers in [0, 255].

**Beginner-Friendly Explanation:** `Uint8Array` holds bytes — numbers from 0 to 255. It's the most common typed array for binary data.

### Annotated Code Examples

```js
const bytes = new Uint8Array([0, 128, 255]);
console.log(bytes); // Uint8Array(3) [0, 128, 255]

// Values wrap.
console.log(new Uint8Array([256, -1])); // Uint8Array(2) [0, 255]
```

**Expected Output:**
```
Uint8Array(3) [ 0, 128, 255 ]
Uint8Array(2) [ 0, 255 ]
```

**Why This Output:** Values wrap modulo 256.

### Real-World Cases

- **File I/O:** Reading binary files.
- **Network:** Parsing binary protocols.
- **ImageData:** Pixel data (RGBA).
- **TextEncoder:** UTF-8 encoding.

---

## `Uint8ClampedArray`

### Definitions

**Core Definition:** `Uint8ClampedArray` is a typed array of 8-bit unsigned integers that clamps values instead of wrapping.

**Technical Definition:** Per ECMA-262 §25.2.6, `Uint8ClampedArray` behaves like `Uint8Array` except that values outside [0, 255] are clamped (not wrapped), and non-integer values are rounded using round-half-to-even.

**Beginner-Friendly Explanation:** `Uint8ClampedArray` is like `Uint8Array`, but instead of wrapping, it clamps. `300` becomes `255`, and `-5` becomes `0`.

### Annotated Code Examples

```js
const clamped = new Uint8ClampedArray([-10, 300, 128.5]);
console.log(clamped); // Uint8ClampedArray(3) [0, 255, 128]

// Compare with Uint8Array.
console.log(new Uint8Array([-10, 300, 128.5])); // Uint8Array(3) [246, 44, 128]
```

**Expected Output:**
```
Uint8ClampedArray(3) [ 0, 255, 128 ]
Uint8Array(3) [ 246, 44, 128 ]
```

**Why This Output:** Clamping avoids overflow artifacts; rounding uses banker's rounding (`128.5` → `128`).

### Real-World Cases

- **Canvas ImageData:** Pixel values are clamped.
- **CSS colour values:** Clamping to [0, 255].

---

## `Int16Array`, `Uint16Array`, `Int32Array`, `Uint32Array`

### Definitions

**Core Definition:** These typed arrays hold 16-bit and 32-bit integers, signed and unsigned.

**Technical Definition:** `Int16Array` and `Uint16Array` use 2 bytes per element; `Int32Array` and `Uint32Array` use 4 bytes. Ranges: `Int16Array` [−32768, 32767], `Uint16Array` [0, 65535], `Int32Array` [−2³¹, 2³¹ − 1], `Uint32Array` [0, 2³² − 1].

**Beginner-Friendly Explanation:** These typed arrays hold larger integers. Use 16-bit for audio samples and 32-bit for general integer work.

### Annotated Code Examples

```js
const i16 = new Int16Array([-32768, 32767]);
console.log(i16); // Int16Array(2) [-32768, 32767]

const u32 = new Uint32Array([0, 4294967295]);
console.log(u32); // Uint32Array(2) [0, 4294967295]

// Byte length.
console.log(i16.byteLength); // 4 (2 elements × 2 bytes)
console.log(u32.byteLength); // 8 (2 elements × 4 bytes)
```

**Expected Output:**
```
Int16Array(2) [ -32768, 32767 ]
Uint32Array(2) [ 0, 4294967295 ]
4
8
```

### Real-World Cases

- **Audio:** 16-bit PCM samples.
- **Networking:** 32-bit packet fields.
- **Image processing:** 16-bit-per-channel images.

---

## `Float32Array`, `Float64Array`

### Definitions

**Core Definition:** `Float32Array` and `Float64Array` hold single-precision (32-bit) and double-precision (64-bit) floating-point numbers.

**Technical Definition:** Per ECMA-262 §25.2.9 and §25.2.10, `Float32Array` uses IEEE 754 single-precision (4 bytes), and `Float64Array` uses IEEE 754 double-precision (8 bytes). `Float64Array` matches the precision of JavaScript `Number`.

**Beginner-Friendly Explanation:** `Float32Array` holds smaller floating-point numbers; `Float64Array` holds the same precision as regular JavaScript numbers.

### Annotated Code Examples

```js
const f32 = new Float32Array([1.5, Math.PI]);
const f64 = new Float64Array([1.5, Math.PI]);

console.log(f32[1]); // 3.1415927410125732 (single precision)
console.log(f64[1]); // 3.141592653589793 (double precision)

console.log(f32.byteLength); // 8
console.log(f64.byteLength); // 16
```

**Expected Output:**
```
3.1415927410125732
3.141592653589793
8
16
```

**Why This Output:** `Float32Array` has less precision than `Float64Array`.

### Real-World Cases

- **Graphics:** Vertex positions, matrices.
- **Audio:** `AudioBuffer` uses `Float32Array`.
- **Machine learning:** Weight matrices.
- **Scientific computing:** High-precision calculations.

---

## `BigInt64Array`, `BigUint64Array`

### Definitions

**Core Definition:** `BigInt64Array` and `BigUint64Array` hold 64-bit signed and unsigned BigInt values.

**Technical Definition:** Per ECMA-262 §25.2.7 and §25.2.8, these typed arrays use 8 bytes per element and store `BigInt` values. Ranges: `BigInt64Array` [−2⁶³, 2⁶³ − 1], `BigUint64Array` [0, 2⁶⁴ − 1].

**Beginner-Friendly Explanation:** These hold very large integers that exceed the safe range of JavaScript `Number`.

### Annotated Code Examples

```js
const b64 = new BigInt64Array([1n, -1n, 9007199254740993n]);
console.log(b64); // BigInt64Array(3) [1n, -1n, 9007199254740993n]

const bu64 = new BigUint64Array([0n, 18446744073709551615n]);
console.log(bu64); // BigUint64Array(2) [0n, 18446744073709551615n]
```

**Expected Output:**
```
BigInt64Array(3) [ 1n, -1n, 9007199254740993n ]
BigUint64Array(2) [ 0n, 18446744073709551615n ]
```

### Real-World Cases

- **Cryptography:** 64-bit keys and hashes.
- **Databases:** BigInt IDs.
- **Scientific computing:** High-precision integers.

---

## `ArrayBuffer`

### Definitions

**Core Definition:** `ArrayBuffer` is a fixed-length container for raw binary data that cannot be accessed directly; it must be viewed through a typed array or `DataView`.

**Technical Definition:** Per ECMA-262 §25.1.3, `ArrayBuffer` is an object with an `[[ArrayBufferData]]` internal slot holding a block of bytes, and a `[[ArrayBufferByteLength]]` slot. Its content cannot be read or written directly; typed arrays and `DataView` provide access. `ArrayBuffer.prototype.slice()` creates a copy.

**Beginner-Friendly Explanation:** `ArrayBuffer` is the "raw memory block." You can't touch its bytes directly — you create a typed array or `DataView` to read and write them.

### Purposes

- To allocate raw memory for binary data.
- To serve as the backing store for typed arrays.
- To transfer data between threads (with `SharedArrayBuffer`).
- To slice and copy binary data.

### Annotated Code Examples

```js
// Step 1: Allocate.
const buffer = new ArrayBuffer(16);
console.log(buffer.byteLength); // 16

// Step 2: Create a view.
const view = new Uint8Array(buffer);
view[0] = 255;
console.log(view[0]); // 255

// Step 3: Slice (copy).
const slice = buffer.slice(0, 8);
console.log(slice.byteLength); // 8

// Step 4: Check if detached.
console.log(buffer.detached); // false
```

**Expected Output:**
```
16
255
8
false
```

### Real-World Cases

- **WebGL buffers:** Upload vertex data.
- **File reading:** `FileReader.readAsArrayBuffer()`.
- **Networking:** `XMLHttpRequest.responseType = "arraybuffer"`.
- **WebAssembly:** Linear memory is an `ArrayBuffer`.

---

## `DataView`

### Definitions

**Core Definition:** `DataView` is a low-level interface for reading and writing multiple numeric types in an `ArrayBuffer` with explicit endianness control.

**Technical Definition:** Per ECMA-262 §25.3, `DataView` provides `getInt8`, `getUint8`, `getInt16`, `getUint16`, `getInt32`, `getUint32`, `getFloat32`, `getFloat64`, `getBigInt64`, `getBigUint64` and corresponding setters. Each method takes a byte offset and an optional `littleEndian` boolean (default `false` for big-endian).

**Beginner-Friendly Explanation:** `DataView` lets you read and write bytes at any position, in any format, and choose the endianness. It's ideal for parsing binary file formats and network protocols.

### Purposes

- To parse binary formats with mixed types.
- To control endianness explicitly.
- To read/write at arbitrary offsets.
- To handle protocol headers.

### Annotated Code Examples

#### Example 1: Reading Different Types

```js
const buffer = new ArrayBuffer(8);
const view = new DataView(buffer);

view.setUint8(0, 255);
view.setInt16(1, -1000, true); // little-endian
view.setFloat32(3, Math.PI, true);

console.log(view.getUint8(0));          // 255
console.log(view.getInt16(1, true));    // -1000
console.log(view.getFloat32(3, true));  // 3.1415927410125732
```

**Expected Output:**
```
255
-1000
3.1415927410125732
```

**Why This Output:** Each setter/getter reads/writes a specific type at a specific offset.

#### Example 2: Endianness

```js
const buffer = new ArrayBuffer(4);
const view = new DataView(buffer);

view.setUint32(0, 0x01020304, false); // big-endian
console.log(view.getUint8(0)); // 1
console.log(view.getUint8(1)); // 2
console.log(view.getUint8(2)); // 3
console.log(view.getUint8(3)); // 4

view.setUint32(0, 0x01020304, true); // little-endian
console.log(view.getUint8(0)); // 4
console.log(view.getUint8(3)); // 1
```

**Expected Output:**
```
1
2
3
4
4
1
```

**Why This Output:** `DataView` allows explicit endianness control.

### Real-World Cases

- **PNG/JPEG parsing:** Reading headers.
- **Network protocols:** TCP/UDP headers.
- **File formats:** Custom binary formats.
- **Cryptography:** Byte-level operations.

---

## Typed Array Operations

### Definitions

**Core Definition:** Typed array operations are the methods and patterns available on typed arrays, which are a subset of `Array.prototype` methods plus binary-specific operations.

**Technical Definition:** Typed arrays implement most `Array.prototype` methods (`map`, `filter`, `reduce`, `forEach`, `slice`, `subarray`, `set`, `fill`, `copyWithin`, `sort`, `reverse`, `indexOf`, `includes`, `find`, `findIndex`, `join`, `every`, `some`), but not `push`, `pop`, `shift`, `unshift`, `splice`, or `concat` (length is fixed). `set()` copies elements from another array; `subarray()` creates a view (not a copy); `slice()` creates a copy.

**Beginner-Friendly Explanation:** Typed arrays have many of the same methods as regular arrays, plus a few special ones for working with binary data.

### Common Operations

#### 1. `set()` — Copy from Another Array

```js
const ta = new Uint8Array(4);
ta.set([1, 2, 3, 4]);
console.log(ta); // Uint8Array(4) [1, 2, 3, 4]
```

#### 2. `subarray()` — View (Not Copy)

```js
const ta = new Uint8Array([1, 2, 3, 4, 5]);
const sub = ta.subarray(1, 4);
console.log(sub); // Uint8Array(3) [2, 3, 4]
sub[0] = 99;
console.log(ta); // Uint8Array(5) [1, 99, 3, 4, 5]
```

#### 3. `slice()` — Copy

```js
const ta = new Uint8Array([1, 2, 3, 4, 5]);
const copy = ta.slice(1, 4);
copy[0] = 99;
console.log(ta);   // Uint8Array(5) [1, 2, 3, 4, 5]
console.log(copy); // Uint8Array(3) [99, 3, 4]
```

#### 4. `map`, `filter`, `reduce`

```js
const ta = new Int32Array([1, 2, 3, 4]);
const doubled = ta.map((x) => x * 2);
console.log(doubled); // Int32Array(4) [2, 4, 6, 8]
```

#### 5. `sort` and `reverse`

```js
const ta = new Int32Array([3, 1, 4, 1, 5]);
ta.sort();
console.log(ta); // Int32Array(5) [1, 1, 3, 4, 5]
```

### Annotated Code Examples

```js
// Step 1: Create and fill.
const ta = new Uint8Array(5);
ta.fill(7);
console.log(ta); // Uint8Array(5) [7, 7, 7, 7, 7]

// Step 2: Set from array.
ta.set([1, 2, 3], 1);
console.log(ta); // Uint8Array(5) [7, 1, 2, 3, 7]

// Step 3: Subarray (view).
const sub = ta.subarray(1, 4);
console.log(sub); // Uint8Array(3) [1, 2, 3]

// Step 4: Copy with slice.
const copy = ta.slice(1, 4);
copy[0] = 99;
console.log(ta[1]);   // 1 (unchanged)
console.log(copy[0]); // 99
```

**Expected Output:**
```
Uint8Array(5) [ 7, 7, 7, 7, 7 ]
Uint8Array(5) [ 7, 1, 2, 3, 7 ]
Uint8Array(3) [ 1, 2, 3 ]
1
99
```

### Real-World Cases

- **Image processing:** Copying pixel data with `set`.
- **WebGL:** Creating views with `subarray`.
- **Audio:** Applying effects with `map`.
- **Networking:** Parsing packets with `slice`.

---

## Typed Array Performance

### Definitions

**Core Definition:** Typed array performance refers to the speed and memory efficiency of typed arrays compared to regular arrays.

**Technical Definition:** Typed arrays store data in contiguous memory with fixed element size, allowing the engine to use highly optimised operations. They avoid the overhead of boxed values, hidden classes, and dynamic resizing. For numeric computation, typed arrays are typically faster and use less memory than regular arrays.

**Beginner-Friendly Explanation:** Typed arrays are faster and use less memory than regular arrays for numeric data because they don't have the overhead of flexible types and dynamic sizing.

### Performance Characteristics

| Aspect | Typed Arrays | Regular Arrays |
|---|---|---|
| Memory per element | Fixed (1–8 bytes) | ~8–16 bytes (boxed) |
| Type checking | None (fixed type) | Dynamic |
| Resizing | Not allowed | Dynamic |
| Iteration | Very fast | Fast |
| Numeric operations | Very fast | Slower |
| General values | Numbers/BigInt only | Any value |

### Annotated Code Examples

```js
const N = 1_000_000;

// Regular array
let start = performance.now();
const arr = [];
for (let i = 0; i < N; i++) arr.push(i);
let sum = 0;
for (let i = 0; i < N; i++) sum += arr[i];
console.log(`Array: ${(performance.now() - start).toFixed(2)} ms`);

// Typed array
start = performance.now();
const ta = new Int32Array(N);
for (let i = 0; i < N; i++) ta[i] = i;
let sum2 = 0;
for (let i = 0; i < N; i++) sum2 += ta[i];
console.log(`Int32Array: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
Array: 15.40 ms
Int32Array: 5.20 ms
```

**Why This Output:** Typed arrays avoid boxing and resizing overhead.

### Real-World Cases

- **Graphics:** Vertex buffers.
- **Audio:** Sample buffers.
- **ML:** Tensor operations.
- **WebAssembly:** Linear memory.

---

## Binary Data Handling

### Definitions

**Core Definition:** Binary data handling is the practice of reading, writing, and manipulating raw bytes using typed arrays, `ArrayBuffer`, and `DataView`.

**Technical Definition:** Binary handling involves converting between JavaScript values and byte sequences, parsing binary formats, and serialising data. `TextEncoder`/`TextDecoder` handle string ↔ bytes; `DataView` handles multi-byte numeric types; `Blob` and `File` wrap binary data.

**Beginner-Friendly Explanation:** Binary data handling means working directly with the bytes that make up files, images, and network packets.

### Common Patterns

#### 1. String to Bytes (UTF-8)

```js
const encoder = new TextEncoder();
const bytes = encoder.encode("Hello");
console.log(bytes); // Uint8Array(5) [72, 101, 108, 108, 111]
```

#### 2. Bytes to String

```js
const decoder = new TextDecoder("utf-8");
const text = decoder.decode(new Uint8Array([72, 101, 108, 108, 111]));
console.log(text); // "Hello"
```

#### 3. Reading a Binary File

```js
// In a browser:
const fileInput = document.querySelector("input[type=file]");
fileInput.addEventListener("change", async (e) => {
  const file = e.target.files[0];
  const buffer = await file.arrayBuffer();
  const view = new DataView(buffer);
  console.log(view.getUint8(0));
});
```

#### 4. Parsing a Custom Binary Format

```js
// Format: [1 byte type][2 bytes length][length bytes payload]
function parse(buffer) {
  const view = new DataView(buffer);
  const type = view.getUint8(0);
  const length = view.getUint16(1);
  const payload = new Uint8Array(buffer, 3, length);
  return { type, length, payload };
}

const buf = new ArrayBuffer(8);
const v = new DataView(buf);
v.setUint8(0, 1);
v.setUint16(1, 3);
new Uint8Array(buf, 3, 3).set([10, 20, 30]);

console.log(parse(buf));
// { type: 1, length: 3, payload: Uint8Array(3) [10, 20, 30] }
```

### Annotated Code Examples

#### Example 1: Encoding and Decoding

```js
const encoder = new TextEncoder();
const decoder = new TextDecoder();

const bytes = encoder.encode("Hello, 世界");
console.log(bytes.byteLength); // 13 (UTF-8: 7 ASCII + 3×2 for CJK)

const text = decoder.decode(bytes);
console.log(text); // "Hello, 世界"
```

**Expected Output:**
```
13
Hello, 世界
```

**Why This Output:** UTF-8 encodes ASCII as 1 byte and CJK characters as 3 bytes each.

#### Example 2: Hex Dump

```js
function hexDump(buffer) {
  const bytes = new Uint8Array(buffer);
  const hex = [...bytes].map((b) => b.toString(16).padStart(2, "0"));
  return hex.join(" ");
}

const buf = new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f]).buffer;
console.log(hexDump(buf)); // "48 65 6c 6c 6f"
```

**Expected Output:**
```
48 65 6c 6c 6f
```

**Why This Output:** Each byte is converted to a two-digit hex string.

### Real-World Cases

- **File readers:** Parsing images, PDFs.
- **Network protocols:** Parsing TCP/UDP packets.
- **Cryptography:** Hashing, encryption.
- **WebAssembly:** Memory manipulation.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Indexed Collections — https://tc39.es/ecma262/multipage/indexed-collections.html
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – TypedArray Objects — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-typedarray-objects
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – DataView Objects — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-dataview-objects
- MDN Web Docs – JavaScript typed arrays — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Typed_arrays
- MDN Web Docs – TypedArray — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray
- MDN Web Docs – Int8Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Int8Array
- MDN Web Docs – Uint8Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array
- MDN Web Docs – Uint8ClampedArray — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray
- MDN Web Docs – Int16Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Int16Array
- MDN Web Docs – Uint16Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint16Array
- MDN Web Docs – Int32Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Int32Array
- MDN Web Docs – Uint32Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint32Array
- MDN Web Docs – Float32Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Float32Array
- MDN Web Docs – Float64Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Float64Array
- MDN Web Docs – BigInt64Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt64Array
- MDN Web Docs – BigUint64Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigUint64Array
- MDN Web Docs – ArrayBuffer — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer
- MDN Web Docs – DataView — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView
- MDN Web Docs – TextEncoder — https://developer.mozilla.org/en-US/docs/Web/API/TextEncoder
- MDN Web Docs – TextDecoder — https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds
- V8 Blog – Typed Arrays in V8 — https://v8.dev/blog/elements-kinds
- JavaScript.info – ArrayBuffer, binary arrays — https://javascript.info/arraybuffer-binary-arrays