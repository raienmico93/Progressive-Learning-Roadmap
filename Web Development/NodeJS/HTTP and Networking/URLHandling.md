# URL, URI, and Query String Engineering — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** URL, URI, and query string engineering in Node.js refers to the parsing, manipulation, construction, and security validation of web addresses and their components using the `node:url` module and its two APIs: the legacy Node.js-specific API and the modern WHATWG-compliant API.

**Technical Definition:** The `node:url` module provides utilities for URL resolution and parsing. It offers two APIs: a legacy API that is Node.js specific (`url.parse()`, `url.format()`, `url.resolve()`) and a newer API that implements the same WHATWG URL Standard used by web browsers (the `URL` class and `URLSearchParams` class). The WHATWG `URL` class is browser-compatible and is also available as a property on the global object. A URL string is a structured string containing multiple meaningful components; when parsed, a URL object is returned containing properties for each of these components.

**Beginner-Friendly Explanation:** A URL is like a postal address for the internet. It tells you how to get somewhere (protocol), where to go (hostname and port), what part of the site you want (pathname), what extra instructions you're sending (query string), and what specific spot on the page you're aiming for (hash). Node.js gives you two toolboxes for working with URLs: an old one (legacy API) that's deprecated and a new one (WHATWG API) that works the same way as in web browsers.

### Key Characteristics

- **Two APIs:** Legacy (Node.js-specific, deprecated) and WHATWG (browser-compatible, recommended).
- **Global availability:** The `URL` and `URLSearchParams` classes are available on the global object and can also be imported from `node:url`.
- **Automatic encoding:** The WHATWG `URL` class automatically percent-encodes special characters in components like the pathname, query, and hash.
- **Punycode conversion:** Unicode characters in hostnames are automatically converted to ASCII using the Punycode algorithm.
- **Structured decomposition:** Every URL object exposes properties for each component (protocol, hostname, port, pathname, search, hash).
- **Security-relevant:** The legacy `url.parse()` is deprecated due to security implications (DEP0169).

### Prerequisites

- **Node.js runtime:** The `url` module is built into Node.js; no external installation is required. It has been stable since v0.10.0.
- **Basic JavaScript knowledge:** Understanding of objects, strings, and classes.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Basic understanding of URLs:** How URLs are structured (scheme, host, path, query, fragment).

### Related Programming Areas

- **HTTP Fundamentals & Web Protocols:** Request/response anatomy, methods, status codes, headers.
- **Path Management:** The `node:path` module for file system path manipulation.
- **Query String Engineering:** Parsing and building query parameters for APIs.
- **Security:** Preventing injection attacks, SSRF, and open redirects via proper URL validation.
- **Web Development:** Constructing URLs for API calls, redirects, and links.

### Core Concepts

1. **Modern URL Parsing** — legacy `url.parse()` vs. the WHATWG `URL` class.
2. **URL Manipulation & Deconstruction** — resolving relative paths and extracting components.
3. **Query & Path Parameters** — `URLSearchParams` and input sanitisation.
4. **Encoding & Security Standards** — `encodeURIComponent()` and `decodeURIComponent()`.

---

## Core Concept 1: Modern URL Parsing

### Definitions

**Core Definition:** Modern URL parsing in Node.js uses the WHATWG-compliant `URL` class, which replaces the deprecated legacy `url.parse()` method and provides browser-compatible, standardised URL parsing.

**Technical Definition:** The WHATWG `URL` class is a browser-compatible URL class, implemented by following the WHATWG URL Standard. It is available as a property on the global object and can also be imported from `node:url`. The legacy `url.parse()` method is deprecated (DEP0116 and DEP0169) because its behaviour is not standardized and is prone to errors that have security implications. The WHATWG API should be used instead for all new code.

**Beginner-Friendly Explanation:** The old way of parsing URLs in Node.js (`url.parse()`) is like using an old map that doesn't always match the real roads. The new way (`new URL()`) is the standard that all web browsers use, so it's reliable, secure, and consistent with how the rest of the web works.

### Purposes

- To parse URL strings into structured objects with named components.
- To ensure consistent URL parsing across Node.js and web browsers.
- To avoid security vulnerabilities associated with the legacy parser.
- To enable construction and manipulation of URLs using a standard API.

### Syntax Rules and Structure

**WHATWG URL parsing:**
```js
const myURL = new URL(input, base?);
```

| Component | Breakdown |
|-----------|-----------|
| `input` | The absolute or relative URL string to parse. |
| `base` | Optional base URL for resolving relative inputs. |
| Returns | A `URL` object with component properties. |
| Throws | `TypeError` if the input or base is not a valid URL. |

**Legacy URL parsing (deprecated):**
```js
const url = require('node:url');
const parsed = url.parse(urlString, parseQueryString?, slashesDenoteHost?);
```

| Component | Breakdown |
|-----------|-----------|
| `urlString` | The URL string to parse. |
| `parseQueryString` | If `true`, the query property is set to an object. |
| `slashesDenoteHost` | If `true`, `//foo/bar` is treated as `//foo/bar`. |
| Returns | A legacy `Url` object. |
| Status | Deprecated (DEP0116, DEP0169). |

**Constraints and Limitations:**
- `new URL()` throws a `TypeError` if the input is not a valid URL.
- Relative URLs require a `base` parameter; without it, `new URL('/path')` throws.
- The legacy `url.parse()` is deprecated and should not be used in new code.
- Unicode hostnames are converted to Punycode automatically (requires ICU).

### Multiple Annotated Code Examples

#### Example 1: Parsing with the WHATWG URL Class

```js
// whatwg-url.js
const myURL = new URL('https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash');

console.log('href:', myURL.href);
// → 'https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash'
console.log('protocol:', myURL.protocol);
// → 'https:'
console.log('username:', myURL.username);
// → 'user'
console.log('password:', myURL.password);
// → 'pass'
console.log('hostname:', myURL.hostname);
// → 'sub.example.com'
console.log('port:', myURL.port);
// → '8080'
console.log('pathname:', myURL.pathname);
// → '/p/a/t/h'
console.log('search:', myURL.search);
// → '?query=string'
console.log('hash:', myURL.hash);
// → '#hash'
console.log('origin:', myURL.origin);
// → 'https://sub.example.com:8080'
```

**Expected Output:**
```
href: https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash
protocol: https:
username: user
password: pass
hostname: sub.example.com
port: 8080
pathname: /p/a/t/h
search: ?query=string
hash: #hash
origin: https://sub.example.com:8080
```

**Why this output:** The `URL` constructor parses the URL string and exposes each component as a property. The `origin` property includes protocol and host but excludes username, password, and path.

#### Example 2: Parsing Relative URLs with a Base

```js
// relative-url.js
const myURL = new URL('/foo', 'https://example.org/');
console.log(myURL.href);
// → 'https://example.org/foo'

const myURL2 = new URL('bar', 'https://example.org/foo/baz');
console.log(myURL2.href);
// → 'https://example.org/foo/bar'

// Without a base, relative URLs throw
try {
  new URL('/foo');
} catch (err) {
  console.log('Error:', err.message);
  // → 'Invalid URL'
}
```

**Expected Output:**
```
https://example.org/foo
https://example.org/foo/bar
Error: Invalid URL
```

**Why this output:** When the input is a relative path (starting with `/` or a segment), the `base` parameter is required. The URL class resolves the relative path against the base URL, producing the absolute URL. Without a base, a relative URL cannot be resolved and a `TypeError` is thrown.

#### Example 3: Demonstrating the Legacy API (Deprecated)

```js
// legacy-url.js
const url = require('node:url');

const parsed = url.parse('https://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash');

console.log('protocol :', parsed.protocol);  // → 'https:'
console.log('host     :', parsed.host);      // → 'sub.example.com:8080'
console.log('pathname :', parsed.pathname);  // → '/p/a/t/h'
console.log('query    :', parsed.query);     // → 'query=string'
console.log('hash     :', parsed.hash);      // → '#hash'
```

**Expected Output:**
```
protocol : https:
host     : sub.example.com:8080
pathname : /p/a/t/h
query    : query=string
hash     : #hash
```

**Why this output:** The legacy `url.parse()` returns a `Url` object with similar properties but different naming conventions (e.g., `host` instead of `hostname` and `port`, `query` instead of `search`). This API is deprecated and should be replaced with `new URL()` in new code.

### Real-World Cases

- **API clients:** Parsing API endpoint URLs to extract the base URL and path.
- **Redirect handling:** Validating and resolving redirect targets safely.
- **Web scraping:** Extracting and normalising links from HTML pages.

---

## Core Concept 2: URL Manipulation & Deconstruction

### Definitions

**Core Definition:** URL manipulation involves extracting individual components from a URL object (protocol, hostname, port, pathname, hash) and resolving relative paths against a base URL to construct absolute URLs.

**Technical Definition:** A parsed WHATWG `URL` object exposes read/write properties for each component: `protocol`, `username`, `password`, `hostname`, `port`, `pathname`, `search`, `hash`, and `origin`. Relative URLs are resolved using the `base` parameter of the `URL` constructor, which behaves similarly to how a web browser resolves an anchor tag. The legacy `url.resolve()` method provided similar functionality but is deprecated.

**Beginner-Friendly Explanation:** Think of a URL as a multi-part form. You can read each part individually (what protocol? what host? what path?) and you can change each part independently. You can also take a partial address (like "room 302") and combine it with a base address (like "123 Main Street") to get the full address ("123 Main Street, room 302").

### Purposes

- To extract specific components of a URL for processing or display.
- To resolve relative URLs against a known base URL.
- To construct absolute URLs from component parts.
- To modify individual URL components without rebuilding the entire URL string.

### Syntax Rules and Structure

**Extracting components:**
```js
const myURL = new URL('https://example.com:8080/path?query=value#hash');

myURL.protocol;  // 'https:'
myURL.hostname;  // 'example.com'
myURL.port;      // '8080'
myURL.pathname;  // '/path'
myURL.search;    // '?query=value'
myURL.hash;      // '#hash'
myURL.origin;    // 'https://example.com:8080'
```

**Resolving relative paths:**
```js
const absolute = new URL('../images/logo.png', 'https://example.com/blog/posts/');
// → 'https://example.com/blog/images/logo.png'
```

| Property | Description |
|----------|-------------|
| `protocol` | The URL scheme (e.g., `'https:'`). |
| `hostname` | The host name without port. |
| `port` | The port number. |
| `pathname` | The path portion. |
| `search` | The query string including `?`. |
| `hash` | The fragment including `#`. |
| `origin` | Protocol + host (no credentials). |

**Constraints and Limitations:**
- Some properties are read-only (`origin`, `href`).
- Changing `hostname` does not automatically update `origin` in all cases.
- The legacy `url.resolve()` is deprecated; use `new URL(relative, base)` instead.

### Annotated Code Example

```js
// url-manipulation.js
const base = new URL('https://api.example.com/v1/users/42?fields=name,email#profile');

// Extract components
console.log('Protocol     :', base.protocol);    // 'https:'
console.log('Hostname     :', base.hostname);    // 'api.example.com'
console.log('Port         :', base.port);        // '' (default for https)
console.log('Pathname     :', base.pathname);    // '/v1/users/42'
console.log('Search       :', base.search);      // '?fields=name,email'
console.log('Hash         :', base.hash);        // '#profile'

// Modify components
base.pathname = '/v1/posts/99';
base.searchParams.set('fields', 'title,body');
console.log('Modified URL :', base.href);
// → 'https://api.example.com/v1/posts/99?fields=title%2Cbody#profile'

// Resolve a relative URL
const relative = new URL('../comments', base);
console.log('Resolved     :', relative.href);
// → 'https://api.example.com/v1/comments'
```

**Expected Output:**
```
Protocol     : https:
Hostname     : api.example.com
Port         : 
Pathname     : /v1/users/42
Search       : ?fields=name,email
Hash         : #profile
Modified URL : https://api.example.com/v1/posts/99?fields=title%2Cbody#profile
Resolved     : https://api.example.com/v1/comments
```

**Why this output:** The `URL` object exposes each component as a readable and writable property. Modifying `pathname` and `searchParams` updates the URL. The `URL` constructor resolves the relative path `../comments` against the modified base URL, producing the correct absolute URL.

### Real-World Cases

- **API client construction:** Building endpoint URLs by modifying a base URL.
- **Link normalisation:** Resolving relative links in web scraping.
- **Redirect handling:** Validating that a redirect target is within the same origin.

---

## Core Concept 3: Query & Path Parameters

### Definitions

**Core Definition:** Query parameters are key-value pairs appended to a URL after the `?` character, while path parameters are dynamic segments within the URL path. The `URLSearchParams` API provides methods for reading, writing, and manipulating query parameters.

**Technical Definition:** The `URLSearchParams` API provides read and write access to the query of a URL. It can be used standalone with one of four constructors: from a string, an object, an iterable, or another `URLSearchParams` object. The `URL` object exposes a `searchParams` property that is an instance of `URLSearchParams`. Path parameters are extracted by matching the pathname against a pattern (e.g., a regular expression).

**Beginner-Friendly Explanation:** Query parameters are like the extra details you give a search engine — "I want page 2, sorted by date." Path parameters are like saying "I want the user with ID 42" — the ID is part of the address itself. The `URLSearchParams` API makes it easy to add, remove, and read these details without manually manipulating strings.

### Purposes

- To read individual query parameters from a URL.
- To add, remove, or modify query parameters programmatically.
- To extract dynamic path parameters from a route.
- To safely handle complex data structures (arrays, nested objects) in query strings.

### Syntax Rules and Structure

**Reading query parameters:**
```js
const myURL = new URL('https://example.com?name=Alice&age=30');
myURL.searchParams.get('name');    // 'Alice'
myURL.searchParams.get('missing'); // null
myURL.searchParams.has('age');     // true
```

**Modifying query parameters:**
```js
myURL.searchParams.set('age', '31');   // Update
myURL.searchParams.append('tag', 'a'); // Add
myURL.searchParams.delete('name');     // Remove
```

**Iterating query parameters:**
```js
for (const [key, value] of myURL.searchParams) {
  console.log(key, value);
}
```

| Method | Description |
|--------|-------------|
| `get(key)` | Returns the first value for a key, or `null`. |
| `getAll(key)` | Returns all values for a key. |
| `has(key)` | Returns `true` if the key exists. |
| `set(key, value)` | Sets the value, replacing all existing. |
| `append(key, value)` | Adds a value without removing existing. |
| `delete(key)` | Removes all values for a key. |
| `sort()` | Sorts keys in-place. |
| `forEach(callback)` | Iterates key-value pairs. |

**Constraints and Limitations:**
- `URLSearchParams` values are always strings.
- Complex data (arrays, nested objects) must be serialised (e.g., JSON) or encoded using conventions (e.g., `key[]=value`).
- The `searchParams` property is live: modifying it updates the URL immediately.

### Multiple Annotated Code Examples

#### Example 1: Reading and Modifying Query Parameters

```js
// search-params.js
const myURL = new URL('https://api.example.com/search?q=nodejs&page=1&limit=10');

// Read parameters
console.log('Query:', myURL.searchParams.get('q'));    // 'nodejs'
console.log('Page:', myURL.searchParams.get('page'));  // '1'

// Add and modify
myURL.searchParams.set('page', '2');
myURL.searchParams.append('sort', 'date');
myURL.searchParams.delete('limit');

console.log('Modified URL:', myURL.href);
// → 'https://api.example.com/search?q=nodejs&page=2&sort=date'
```

**Expected Output:**
```
Query: nodejs
Page: 1
Modified URL: https://api.example.com/search?q=nodejs&page=2&sort=date
```

**Why this output:** `get()` retrieves individual values. `set()` replaces the `page` value. `append()` adds a new `sort` parameter without removing existing ones. `delete()` removes the `limit` parameter. The `href` property reflects all changes immediately.

#### Example 2: Handling Arrays in Query Strings

```js
// array-params.js
const myURL = new URL('https://example.com/filter');

// Append multiple values for the same key
myURL.searchParams.append('tag', 'javascript');
myURL.searchParams.append('tag', 'nodejs');
myURL.searchParams.append('tag', 'backend');

console.log('URL:', myURL.href);
// → 'https://example.com/filter?tag=javascript&tag=nodejs&tag=backend'

console.log('All tags:', myURL.searchParams.getAll('tag'));
// → ['javascript', 'nodejs', 'backend']
```

**Expected Output:**
```
URL: https://example.com/filter?tag=javascript&tag=nodejs&tag=backend
All tags: [ 'javascript', 'nodejs', 'backend' ]
```

**Why this output:** Multiple calls to `append()` with the same key produce repeated query parameters. `getAll()` retrieves all values as an array. This is the standard way to represent arrays in query strings.

#### Example 3: Extracting Path Parameters

```js
// path-params.js
function matchRoute(pathname) {
  // Match /users/:id
  const userMatch = pathname.match(/^\/users\/(\d+)$/);
  if (userMatch) {
    return { type: 'user', id: parseInt(userMatch[1], 10) };
  }

  // Match /posts/:slug
  const postMatch = pathname.match(/^\/posts\/([a-z0-9-]+)$/);
  if (postMatch) {
    return { type: 'post', slug: postMatch[1] };
  }

  return { type: 'unknown' };
}

console.log(matchRoute('/users/42'));
// → { type: 'user', id: 42 }
console.log(matchRoute('/posts/hello-world'));
// → { type: 'post', slug: 'hello-world' }
```

**Expected Output:**
```
{ type: 'user', id: 42 }
{ type: 'post', slug: 'hello-world' }
```

**Why this output:** The `match()` method applies a regular expression to the pathname. Capture groups extract the dynamic segments (the user ID or post slug). The numeric ID is parsed with `parseInt()` for safe use.

### Real-World Cases

- **Search APIs:** Building and parsing search queries with filters, pagination, and sorting.
- **REST APIs:** Extracting resource IDs and slugs from URL paths.
- **Analytics:** Tracking campaign parameters (UTM tags) in URLs.

---

## Core Concept 4: Encoding & Security Standards

### Definitions

**Core Definition:** URL encoding converts special characters into a percent-encoded format (e.g., space becomes `%20`) to ensure they are transmitted safely, while decoding reverses the process. `encodeURIComponent()` and `decodeURIComponent()` are the standard JavaScript functions for this purpose.

**Technical Definition:** `encodeURIComponent()` takes a string and converts it into a URL-friendly format by replacing each instance of certain characters with one, two, three, or four escape sequences representing the UTF-8 encoding of the character. `decodeURIComponent()` reverses this process. These functions are essential for preventing injection attacks where malicious input could alter the structure of a URL — for example, an unencoded `&` in a parameter value could be interpreted as the start of a new parameter.

**Beginner-Friendly Explanation:** URL encoding is like putting fragile items in bubble wrap before shipping. Special characters like `&`, `=`, `?`, and spaces could break the URL structure if sent as-is. `encodeURIComponent()` wraps them in percent-encoded form so they arrive intact. `decodeURIComponent()` unwraps them on the other end.

### Purposes

- To safely include user input in URL query parameters.
- To prevent injection attacks (XSS, SSRF, open redirects).
- To ensure special characters are transmitted correctly.
- To handle Unicode characters in URLs.

### Syntax Rules and Structure

**Encoding:**
```js
encodeURIComponent('Jack & Jill');
// → 'Jack%20%26%20Jill'
```

**Decoding:**
```js
decodeURIComponent('Jack%20%26%20Jill');
// → 'Jack & Jill'
```

**Using with URLSearchParams (automatic encoding):**
```js
const params = new URLSearchParams();
params.set('name', 'Jack & Jill');
console.log(params.toString());
// → 'name=Jack+%26+Jill'
```

| Function | Encodes | Use Case |
|----------|---------|----------|
| `encodeURIComponent()` | All special characters including `&`, `=`, `?`, `/` | Query parameter values |
| `encodeURI()` | Only characters not allowed in URLs | Full URL encoding |
| `URLSearchParams` | Automatically encodes values | Building query strings |

**Constraints and Limitations:**
- `encodeURIComponent()` does not encode `A-Z a-z 0-9 - _ . ! ~ * ' ( )`.
- Double-encoding can occur if `URLSearchParams` is used on already-encoded values.
- `decodeURIComponent()` throws `URIError` on malformed input.

### Multiple Annotated Code Examples

#### Example 1: Safe Query Parameter Construction

```js
// encoding-security.js
const userInput = 'Jack & Jill <script>alert("xss")</script>';

// Unsafe: concatenating directly
const unsafeURL = `https://example.com/search?q=${userInput}`;
console.log('Unsafe:', unsafeURL);
// → 'https://example.com/search?q=Jack & Jill <script>alert("xss")</script>'

// Safe: using URLSearchParams (automatic encoding)
const safeURL = new URL('https://example.com/search');
safeURL.searchParams.set('q', userInput);
console.log('Safe:', safeURL.href);
// → 'https://example.com/search?q=Jack+%26+Jill+%3Cscript%3Ealert%28%22xss%22%29%3C%2Fscript%3E'

// Safe: using encodeURIComponent manually
const encoded = encodeURIComponent(userInput);
console.log('Encoded:', encoded);
// → 'Jack%20%26%20Jill%20%3Cscript%3Ealert(%22xss%22)%3C%2Fscript%3E'
```

**Expected Output:**
```
Unsafe: https://example.com/search?q=Jack & Jill <script>alert("xss")</script>
Safe: https://example.com/search?q=Jack+%26+Jill+%3Cscript%3Ealert%28%22xss%22%29%3C%2Fscript%3E
Encoded: Jack%20%26%20Jill%20%3Cscript%3Ealert(%22xss%22)%3C%2Fscript%3E
```

**Why this output:** The unsafe URL contains raw user input with `&`, spaces, and `<` characters that could break the URL structure or be interpreted as HTML. `URLSearchParams.set()` automatically percent-encodes the value. `encodeURIComponent()` achieves the same result manually. Both prevent injection attacks.

#### Example 2: Decoding and Validating Redirect Targets

```js
// redirect-validation.js
function safeRedirect(target, allowedHosts) {
  let decoded;
  try {
    decoded = decodeURIComponent(target);
  } catch (err) {
    throw new Error('Invalid encoding in redirect target');
  }

  let url;
  try {
    url = new URL(decoded, 'https://example.com');
  } catch (err) {
    throw new Error('Invalid redirect URL');
  }

  if (!allowedHosts.includes(url.hostname)) {
    throw new Error(`Redirect to ${url.hostname} is not allowed`);
  }

  return url.href;
}

// Safe redirect within the same host
console.log(safeRedirect('/dashboard', ['example.com']));
// → 'https://example.com/dashboard'

// Blocked redirect to an external host
try {
  safeRedirect('https://evil.com/phishing', ['example.com']);
} catch (err) {
  console.log('Blocked:', err.message);
  // → 'Blocked: Redirect to evil.com is not allowed'
}
```

**Expected Output:**
```
https://example.com/dashboard
Blocked: Redirect to evil.com is not allowed
```

**Why this output:** The function first decodes the redirect target (in case it was percent-encoded). It then resolves it against the base URL and validates the hostname against an allowlist. External hosts are rejected, preventing open redirect attacks.

### Real-World Cases

- **Form submissions:** Encoding user input before adding it to a query string.
- **Redirect handling:** Validating redirect targets to prevent open redirects.
- **API clients:** Ensuring query parameters with special characters are transmitted correctly.

---

## References

- Node.js Documentation — URL — https://nodejs.org/api/url.html
- Node.js Documentation — WHATWG URL API — https://nodejs.org/api/url.html#the-whatwg-url-api
- Node.js Documentation — Legacy URL API — https://nodejs.org/api/url.html#legacy-url-api
- Node.js Documentation — `URLSearchParams` — https://nodejs.org/api/url.html#class-urlsearchparams
- Node.js Documentation — `new URL()` — https://nodejs.org/api/url.html#new-urlinput-base
- Node.js Documentation — DEP0116: Legacy URL API — https://nodejs.org/api/deprecations.html#dep0116-legacy-url-api
- Node.js Documentation — DEP0169: Insecure `url.parse()` — https://nodejs.org/api/deprecations.html#dep0169-urlparse
- WHATWG URL Standard — https://url.spec.whatwg.org/
- MDN Web Docs — `encodeURIComponent()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent
- MDN Web Docs — `decodeURIComponent()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/decodeURIComponent
- MDN Web Docs — URL — https://developer.mozilla.org/en-US/docs/Web/API/URL
- MDN Web Docs — URLSearchParams — https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams
- OWASP — URL Encoding — https://owasp.org/www-community/attacks/xss/
- OWASP — Open Redirect — https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html
- Safely Validating Untrusted URLs in Node.js — https://safeguard.sh/resources/blog/validating-untrusted-urls-nodejs