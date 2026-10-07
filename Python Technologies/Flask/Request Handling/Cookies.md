# Flask Cookies: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A cookie is a small piece of data (typically no more than 4KB) that a server sends to a client's web browser, which the browser stores and sends back with subsequent requests to the same domain. In Flask, cookies are read via `request.cookies` and set via `Response.set_cookie()`.

**Technical Definition:** Cookies are transmitted as HTTP headers: the server sends `Set-Cookie` response headers containing name-value pairs along with attributes (expiry, domain, path, `Secure`, `HttpOnly`, `SameSite`), and the client returns them in the `Cookie` request header. Flask's `Response.set_cookie()` method builds the `Set-Cookie` header with the specified attributes, while `request.cookies` exposes an `ImmutableMultiDict` of parsed cookie values. Flask's session system builds on top of cookies by serializing session data, signing it with a `SECRET_KEY` via the `itsdangerous` library, and storing it in a cookie named `session` by default.

**Beginner-Friendly Explanation:** Cookies are like name tags that a website gives your browser. When you visit a site, the server can give your browser a cookie containing some information (like your preferred language or a session ID). Every time you revisit that site, your browser automatically shows the cookie to the server, so the server remembers who you are.

### Key Characteristics

- **Client-side storage:** Cookies are stored on the client's machine and sent with every request to the matching domain.
- **Size-limited:** Typically limited to 4KB per cookie and a maximum number of cookies per domain.
- **Domain and path scoped:** Cookies are sent only to the domain and path they were set for.
- **Expiry-controlled:** Cookies can be session-based (deleted when the browser closes) or persistent (with an explicit `max_age` or `expires`).
- **Security attributes:** `Secure`, `HttpOnly`, and `SameSite` control how and when cookies are transmitted.
- **Signed sessions:** Flask's session cookie is cryptographically signed to prevent tampering but not encrypted, so data is visible to the client.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of HTTP requests, responses, and headers.
- Familiarity with Flask's `request` and `Response` objects.
- For signed cookies: understanding of cryptographic signing and the `SECRET_KEY` configuration.

### Related Programming Areas

- **Session management:** Cookies are the transport mechanism for Flask's default session implementation.
- **Authentication:** Cookies store session identifiers and remember-me tokens.
- **Security:** `Secure`, `HttpOnly`, and `SameSite` attributes defend against interception, XSS, and CSRF.
- **User experience:** Cookies store preferences (theme, language) across visits.
- **Privacy and compliance:** Cookie usage is regulated by GDPR, ePrivacy, and CCPA.

### Core Concepts / Features

1. Reading Cookies (`request.cookies`)
2. Setting Cookies (`Response.set_cookie()`)
3. Cookie Attributes (`Secure`, `HttpOnly`, `SameSite`, `Domain`, `Path`, `Max-Age`, `Expires`)
4. Deleting Cookies (`Response.delete_cookie()`)
5. Signed and Encrypted Cookies (Flask Session Middleware and Custom Cryptography)

---

## 1. Reading Cookies (`request.cookies`)

### Definitions

**Core Definition:** `request.cookies` is a dictionary-like object that contains all cookies sent by the client in the `Cookie` request header, with cookie names as keys and cookie values as strings.

**Technical Definition:** `request.cookies` is an instance of `werkzeug.datastructures.ImmutableMultiDict`. When a request arrives, Werkzeug parses the `Cookie` header (a semicolon-separated list of `name=value` pairs) and populates the `MultiDict`. The `.get()` method provides safe access with an optional default, while `[]` raises `KeyError` if the cookie is missing. Cookie values are always strings; type conversion must be performed manually.

**Beginner-Friendly Explanation:** `request.cookies` is like a Python dictionary that Flask fills with all the cookies your browser sent. You read them with `request.cookies.get("cookie_name")`, and it safely returns `None` if the cookie doesn't exist.

### Purposes

- To read session identifiers stored in cookies.
- To remember user preferences (language, theme) across visits.
- To read authentication tokens stored in cookies.
- To implement tracking or analytics (with appropriate privacy considerations).
- To access custom application data stored in cookies.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import request

# Safe access with default
value = request.cookies.get('cookie_name', default=None)

# Dictionary-style access (raises KeyError if missing)
value = request.cookies['cookie_name']

# Check if a cookie exists
if 'cookie_name' in request.cookies:
    ...

# Iterate all cookies
for name, value in request.cookies.items():
    print(name, value)
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `.get(name, default)` | Returns the cookie value or default |
| `[name]` | Returns the value; raises `KeyError` if missing |
| `.items()` | Iterates over cookie name-value pairs |
| `in` operator | Checks if a cookie name exists |

**Syntax Rules:**

- Cookie values are always strings; convert types manually (e.g., `int(request.cookies.get('count', 0))`).
- Use `.get()` for optional cookies to avoid `KeyError`.
- Cookies are sent only to the domain and path they were set for.
- The `Cookie` header can contain multiple cookies separated by `; `.

**Constraints and Limitations:**

- Cookies are limited in size (typically 4KB per cookie); do not store large data.
- Cookies are sent with every request, increasing bandwidth.
- Cookie data is visible to the client and can be modified; never trust cookie data for security-critical decisions without validation or signing.

### Annotated Code Examples

**Example 1: Reading a Cookie**

```python
from flask import Flask, request, make_response

app = Flask(__name__)

@app.route("/set-cookie")
def set_cookie():
    resp = make_response("Cookie set")
    resp.set_cookie("theme", "dark", max_age=3600)
    return resp

@app.route("/read-cookie")
def read_cookie():
    theme = request.cookies.get("theme", "light")
    return f"Your theme is: {theme}"

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- Visiting `/set-cookie` sets the `theme` cookie to `dark`.
- Visiting `/read-cookie` afterwards → `"Your theme is: dark"`.
- Visiting `/read-cookie` without the cookie → `"Your theme is: light"`.

**Why this output:** `response.set_cookie()` instructs the browser to store the cookie. On subsequent requests, the browser sends the cookie in the `Cookie` header, and `request.cookies.get()` retrieves it. The default `"light"` is used when the cookie is absent.

**Example 2: Iterating All Cookies**

```python
@app.route("/all-cookies")
def all_cookies():
    cookies = {name: value for name, value in request.cookies.items()}
    return cookies
```

**Expected Output:**
- `GET /all-cookies` with cookies `theme=dark; lang=en` → `{"theme": "dark", "lang": "en"}`

**Why this output:** `request.cookies.items()` yields all cookie name-value pairs. The view builds a dictionary and returns it as JSON.

### Real-World Cases

- **Session management:** Reading the `session` cookie to restore user state.
- **Language preferences:** Reading a `lang` cookie to serve content in the user's preferred language.
- **Analytics:** Reading tracking cookies to identify returning visitors.

### References

- Flask Quickstart: Cookies — https://flask.palletsprojects.com/en/stable/quickstart/#cookies
- Flask API: `request.cookies` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.cookies
- Werkzeug `ImmutableMultiDict` — https://werkzeug.palletsprojects.com/en/stable/datastructures/#werkzeug.datastructures.ImmutableMultiDict

---

## 2. Setting Cookies (`Response.set_cookie()`)

### Definitions

**Core Definition:** `Response.set_cookie()` is a method on Flask's `Response` object that adds a `Set-Cookie` header to the HTTP response, instructing the client's browser to store a cookie.

**Technical Definition:** `Response.set_cookie(key, value='', max_age=None, expires=None, path='/', domain=None, secure=False, httponly=False, samesite=None, partitioned=False)` constructs a `Set-Cookie` header with the specified name-value pair and attributes. The `max_age` parameter specifies the cookie's lifetime in seconds; `expires` specifies an absolute expiry date. The `secure`, `httponly`, and `samesite` parameters control the corresponding cookie attributes. Flask's `Response` class inherits this method from Werkzeug's `BaseResponse`.

**Beginner-Friendly Explanation:** `set_cookie()` is how your Flask app tells the browser to remember something. You call it on the response object, providing a name, a value, and optional settings like how long the cookie should last.

### Purposes

- To store session identifiers and authentication tokens.
- To remember user preferences across visits.
- To implement "remember me" functionality.
- To track user behavior (with consent).
- To store CSRF tokens and other security-related data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
from flask import make_response

resp = make_response("Response body")
resp.set_cookie(
    key,                 # Cookie name (required)
    value='',            # Cookie value (default: empty string)
    max_age=None,        # Lifetime in seconds
    expires=None,        # Absolute expiry (datetime or timestamp)
    path='/',            # URL path the cookie applies to
    domain=None,         # Domain the cookie applies to
    secure=False,        # HTTPS-only transmission
    httponly=False,      # Block JavaScript access
    samesite=None,       # 'Strict', 'Lax', or 'None'
    partitioned=False    # Partitioned (CHIPS) attribute
)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `key` | Cookie name (required) |
| `value` | Cookie value (optional; defaults to empty string) |
| `max_age` | Lifetime in seconds (e.g., `3600` for 1 hour) |
| `expires` | Absolute expiry date (alternative to `max_age`) |
| `path` | URL path scope (default: `"/"`) |
| `domain` | Domain scope (default: current domain) |
| `secure` | If `True`, cookie sent only over HTTPS |
| `httponly` | If `True`, cookie inaccessible to JavaScript |
| `samesite` | `'Strict'`, `'Lax'`, or `'None'` |
| `partitioned` | If `True`, enables CHIPS partitioning |

**Syntax Rules:**

- The `key` parameter is required and must be a valid cookie name.
- The `value` parameter should be URL-safe; Flask encodes it automatically.
- `max_age` and `expires` are mutually exclusive; `max_age` is preferred in modern browsers.
- `samesite='None'` requires `secure=True` in modern browsers.
- The `path` parameter defaults to `"/"`, making the cookie available across the entire site.

**Constraints and Limitations:**

- Cookie values are limited to approximately 4KB.
- The total number of cookies per domain is limited (typically 50 in modern browsers).
- Setting cookies does not take effect until the response is sent to the client.
- Cookies set without `max_age` or `expires` are session cookies (deleted when the browser closes).

### Annotated Code Examples

**Example 1: Setting a Simple Cookie**

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route("/set-theme")
def set_theme():
    resp = make_response("Theme set to dark")
    resp.set_cookie("theme", "dark", max_age=3600)
    return resp

if __name__ == "__main__":
    app.run(debug=True)
```

**Expected Output:**
- `GET /set-theme` → response header `Set-Cookie: theme=dark; Max-Age=3600; Path=/`

**Why this output:** `set_cookie()` adds the `Set-Cookie` header to the response. The browser stores the cookie for 1 hour (`max_age=3600`) and sends it back with subsequent requests.

**Example 2: Setting a Session Cookie (No Expiry)**

```python
@app.route("/set-session-cookie")
def set_session_cookie():
    resp = make_response("Session cookie set")
    resp.set_cookie("temp", "value")
    return resp
```

**Expected Output:**
- `GET /set-session-cookie` → response header `Set-Cookie: temp=value; Path=/`

**Why this output:** Without `max_age` or `expires`, the cookie is a session cookie, which the browser deletes when closed.

### Real-World Cases

- **Authentication:** Setting a session cookie after successful login.
- **Preferences:** Storing the user's selected theme or language.
- **Shopping carts:** Storing cart identifiers for anonymous users.
- **Consent:** Recording cookie consent preferences.

### References

- Flask API: `Response.set_cookie` — https://flask.palletsprojects.com/en/stable/api/#flask.Response.set_cookie
- Flask Quickstart: Cookies — https://flask.palletsprojects.com/en/stable/quickstart/#cookies
- Werkzeug `BaseResponse.set_cookie` — https://werkzeug.palletsprojects.com/en/stable/wrappers/#werkzeug.wrappers.Response.set_cookie

---

## 3. Cookie Attributes

### Definitions

**Core Definition:** Cookie attributes are additional parameters that control a cookie's behavior, including its lifetime, scope, and security restrictions.

**Technical Definition:** Cookie attributes are appended to the `Set-Cookie` header after the name-value pair, separated by semicolons. The standard attributes are `Expires`, `Max-Age`, `Domain`, `Path`, `Secure`, `HttpOnly`, and `SameSite`. Each attribute controls a specific aspect of the cookie's behavior, as defined by RFC 6265 and its extensions.

**Beginner-Friendly Explanation:** Cookie attributes are like settings on a cookie. You can control how long it lasts (`Max-Age`), which parts of your site it applies to (`Path`), and how secure it is (`Secure`, `HttpOnly`, `SameSite`).

### Purposes

- To control how long a cookie persists on the client.
- To restrict which URLs and domains receive the cookie.
- To enforce security policies (HTTPS-only, no JavaScript access, CSRF protection).
- To comply with browser privacy policies (CHIPS partitioning).
- To prevent cookie leakage across origins.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
resp.set_cookie(
    "name", "value",
    max_age=3600,           # Lifetime in seconds
    expires=datetime(...),  # Absolute expiry
    path="/",               # Path scope
    domain=".example.com",  # Domain scope
    secure=True,            # HTTPS-only
    httponly=True,          # No JavaScript access
    samesite="Lax",         # CSRF protection
    partitioned=True        # CHIPS partitioning
)
```

**Component Breakdown:**

| Attribute | Header | Purpose |
|-----------|--------|---------|
| `Max-Age` | `Max-Age=3600` | Lifetime in seconds |
| `Expires` | `Expires=Wed, 21 Oct 2026 07:28:00 GMT` | Absolute expiry |
| `Domain` | `Domain=.example.com` | Domain scope |
| `Path` | `Path=/app` | Path scope |
| `Secure` | `Secure` | HTTPS-only transmission |
| `HttpOnly` | `HttpOnly` | Block JavaScript access |
| `SameSite` | `SameSite=Lax` | Cross-site request policy |
| `Partitioned` | `Partitioned` | CHIPS partitioning |

**Syntax Rules:**

- `Max-Age` takes precedence over `Expires` in modern browsers.
- `Domain` should start with a dot (`.example.com`) to include subdomains.
- `Path` restricts the cookie to a URL path prefix.
- `Secure` requires the connection to be HTTPS.
- `HttpOnly` prevents access via `document.cookie` in JavaScript.
- `SameSite` accepts `'Strict'`, `'Lax'`, or `'None'`; `'None'` requires `Secure`.

**Constraints and Limitations:**

- `SameSite=None` without `Secure` is rejected by modern browsers.
- `Partitioned` cookies require `Secure` and `SameSite=None` in Chrome.
- The `Domain` attribute cannot be set to a different domain than the one that set the cookie.
- The `Path` attribute must be a prefix of the request path.

### Annotated Code Examples

**Example 1: Secure, HttpOnly, SameSite Cookie**

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route("/secure-cookie")
def secure_cookie():
    resp = make_response("Secure cookie set")
    resp.set_cookie(
        "session_id", "abc123",
        max_age=3600,
        secure=True,        # HTTPS only
        httponly=True,      # No JavaScript access
        samesite="Lax"      # CSRF protection
    )
    return resp
```

**Expected Output:**
- `GET /secure-cookie` over HTTPS → response header `Set-Cookie: session_id=abc123; Max-Age=3600; Path=/; Secure; HttpOnly; SameSite=Lax`

**Why this output:** The `secure=True`, `httponly=True`, and `samesite="Lax"` parameters add the corresponding attributes to the `Set-Cookie` header, enforcing security policies.

### Real-World Cases

- **Authentication cookies:** Always use `Secure`, `HttpOnly`, and `SameSite=Lax` or `Strict`.
- **Cross-site OAuth:** Use `SameSite=None; Secure` for cookies that must be sent in cross-origin redirects.
- **CSRF tokens:** Use `SameSite=Lax` or `Strict` to prevent cross-site submission.

### References

- RFC 6265: HTTP State Management Mechanism — https://www.rfc-editor.org/rfc/rfc6265
- MDN: Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- Flask API: `Response.set_cookie` — https://flask.palletsprojects.com/en/stable/api/#flask.Response.set_cookie

---

### 3.1 Secure (Enforcing HTTPS-Only Transmission)

#### Definitions

**Core Definition:** The `Secure` attribute instructs the browser to send the cookie only over HTTPS connections, preventing interception over unencrypted HTTP.

**Technical Definition:** When the `Secure` attribute is set, the browser includes the cookie in requests only when the connection uses HTTPS (or `wss://` for WebSockets). This prevents cookies from being transmitted over plaintext HTTP, where they could be intercepted by network attackers. The attribute is defined in RFC 6265 and is supported by all modern browsers.

**Beginner-Friendly Explanation:** The `Secure` flag makes sure your cookie is only sent over an encrypted connection (HTTPS), so nobody can steal it by listening to your network traffic.

#### Purposes

- To prevent cookie theft via network interception (man-in-the-middle attacks).
- To protect session identifiers and authentication tokens from eavesdropping.
- To comply with security best practices and regulatory requirements.
- To prevent cookie leakage when a user visits the HTTP version of a site.

#### Syntax Rules and Structure

```python
resp.set_cookie("name", "value", secure=True)
```

| Parameter | Description |
|-----------|-------------|
| `secure=True` | Adds the `Secure` attribute to the cookie |
| `secure=False` | No `Secure` attribute (default) |

**Syntax Rules:**

- The `Secure` attribute has no value; it is a flag.
- Browsers silently drop `Secure` cookies set over HTTP.
- `SameSite=None` requires `Secure=True` in modern browsers.

**Constraints and Limitations:**

- During development, `Secure` cookies are not sent over `http://localhost` unless the browser is configured to allow it.
- Setting `Secure` on a cookie does not encrypt its contents; it only controls transmission.

#### Annotated Code Examples

```python
@app.route("/login")
def login():
    resp = make_response("Logged in")
    resp.set_cookie("session_id", "abc123", secure=True, httponly=True)
    return resp
```

**Expected Output:**
- Over HTTPS → `Set-Cookie: session_id=abc123; Path=/; Secure; HttpOnly`
- Over HTTP → the browser rejects the cookie and does not store it.

**Why this output:** The `Secure` attribute ensures the cookie is only stored and sent when the connection is HTTPS. Over HTTP, the browser discards the cookie.

### 3.2 HttpOnly (Mitigating XSS by Blocking Client-Side JavaScript Access)

#### Definitions

**Core Definition:** The `HttpOnly` attribute prevents client-side JavaScript from accessing the cookie via `document.cookie`, mitigating cookie theft through cross-site scripting (XSS) attacks.

**Technical Definition:** When `HttpOnly` is set, the browser stores the cookie but does not expose it to JavaScript APIs such as `document.cookie`. The cookie is still sent in HTTP requests to the server. This attribute is defined in RFC 6265 and is supported by all modern browsers.

**Beginner-Friendly Explanation:** `HttpOnly` makes the cookie invisible to JavaScript, so if an attacker injects malicious scripts into your page, they can't steal the cookie.

#### Purposes

- To mitigate XSS attacks that attempt to steal session cookies.
- To protect authentication tokens from client-side script access.
- To enforce the principle of least privilege for client-side code.
- To comply with security best practices for session management.

#### Syntax Rules and Structure

```python
resp.set_cookie("session_id", "abc123", httponly=True)
```

| Parameter | Description |
|-----------|-------------|
| `httponly=True` | Adds the `HttpOnly` attribute |
| `httponly=False` | No `HttpOnly` attribute (default) |

**Syntax Rules:**

- `HttpOnly` has no value; it is a flag.
- Cookies with `HttpOnly` are still accessible via HTTP headers (server-side).
- `HttpOnly` does not prevent the cookie from being sent with cross-site requests; use `SameSite` for that.

**Constraints and Limitations:**

- `HttpOnly` does not protect against all XSS attacks; it only prevents direct cookie theft via `document.cookie`.
- Some legitimate JavaScript functionality may require access to cookies; use alternative mechanisms (e.g., CSRF tokens in meta tags) in such cases.

#### Annotated Code Examples

```python
@app.route("/login")
def login():
    resp = make_response("Logged in")
    resp.set_cookie("session_id", "abc123", httponly=True, secure=True)
    return resp
```

**Expected Output:**
- `Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure`
- In the browser, `document.cookie` does not include `session_id`.

**Why this output:** The `HttpOnly` attribute prevents JavaScript from reading the cookie, while the server can still read it from the `Cookie` request header.

### 3.3 SameSite (Configuring Strict, Lax, or None to Defend Against CSRF)

#### Definitions

**Core Definition:** The `SameSite` attribute controls whether a cookie is sent with cross-site requests, providing defense-in-depth against cross-site request forgery (CSRF) attacks.

**Technical Definition:** The `SameSite` attribute accepts three values: `'Strict'` (cookie sent only for same-site requests), `'Lax'` (cookie sent for same-site requests and top-level cross-site navigations with safe methods like GET), and `'None'` (cookie sent for all requests; requires `Secure`). The attribute is defined in RFC 6265bis and is supported by all modern browsers.

**Beginner-Friendly Explanation:** `SameSite` tells the browser when to send your cookie. `Strict` means only when you're on the same site. `Lax` means also when you click a link from another site. `None` means always send it (but only over HTTPS).

#### Purposes

- To mitigate CSRF attacks by preventing cookies from being sent with cross-site requests.
- To provide defense-in-depth alongside CSRF tokens.
- To control cookie behavior in cross-origin scenarios (OAuth, embedded content).
- To comply with browser default policies (Chrome defaults to `Lax`).

#### Syntax Rules and Structure

```python
resp.set_cookie("session_id", "abc123", samesite="Lax")
```

| Value | Behavior |
|-------|----------|
| `'Strict'` | Cookie sent only for same-site requests |
| `'Lax'` | Cookie sent for same-site requests and top-level cross-site navigations (GET) |
| `'None'` | Cookie sent for all requests; requires `Secure` |

**Syntax Rules:**

- `SameSite=None` requires `Secure=True`; otherwise, the browser rejects the cookie.
- `SameSite=Lax` is the default in modern browsers if the attribute is not specified.
- `SameSite=Strict` may break OAuth flows and embedded content that rely on cookies.

**Constraints and Limitations:**

- `SameSite` does not replace CSRF tokens; use both for defense-in-depth.
- Older browsers may not support `SameSite=None`; provide fallback behavior.
- `SameSite=Strict` can prevent users from being logged in when they arrive via an external link.

#### Annotated Code Examples

```python
@app.route("/login")
def login():
    resp = make_response("Logged in")
    resp.set_cookie(
        "session_id", "abc123",
        samesite="Strict",  # Only same-site requests
        secure=True,
        httponly=True
    )
    return resp

@app.route("/oauth-callback")
def oauth_callback():
    resp = make_response("OAuth complete")
    resp.set_cookie(
        "oauth_state", "xyz",
        samesite="None",    # Allow cross-site (OAuth redirect)
        secure=True         # Required with SameSite=None
    )
    return resp
```

**Expected Output:**
- `/login` → `Set-Cookie: session_id=abc123; Path=/; Secure; HttpOnly; SameSite=Strict`
- `/oauth-callback` → `Set-Cookie: oauth_state=xyz; Path=/; Secure; SameSite=None`

**Why this output:** The `samesite` parameter controls the cookie's cross-site behavior. `Strict` is used for session cookies, while `None` (with `Secure`) is used for OAuth state cookies that must survive cross-site redirects.

### Real-World Cases

- **Session cookies:** Use `SameSite=Lax` or `Strict` to prevent CSRF.
- **OAuth state cookies:** Use `SameSite=None; Secure` to allow cross-site redirects.
- **Third-party widgets:** Use `SameSite=None; Secure` for cookies used in embedded iframes.

### References

- RFC 6265: HTTP State Management Mechanism — https://www.rfc-editor.org/rfc/rfc6265
- RFC 6265bis: SameSite — https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis/
- MDN: SameSite Cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- web.dev: SameSite Cookies Explained — https://web.dev/articles/samesite-cookies-explained

---

## 4. Deleting Cookies (`Response.delete_cookie()`)

### Definitions

**Core Definition:** `Response.delete_cookie()` removes a cookie from the client's browser by setting its value to an empty string and its expiry to a time in the past.

**Technical Definition:** `Response.delete_cookie(key, path='/', domain=None, secure=False, httponly=False, samesite=None)` adds a `Set-Cookie` header with the specified cookie name, an empty value, and an `Expires` attribute set to `Thu, 01 Jan 1970 00:00:00 GMT` (or a `Max-Age` of `0`). Because HTTP has no dedicated "delete cookie" mechanism, the browser is instructed to remove the cookie by overwriting it with an expired one.

**Beginner-Friendly Explanation:** To delete a cookie, you tell the browser that the cookie has already expired. The browser then removes it. Flask's `delete_cookie()` method does this for you.

### Purposes

- To log users out by removing session cookies.
- To clear user preferences and tracking data.
- To reset application state stored in cookies.
- To comply with user requests to delete their data.
- To remove cookies after they are no longer needed.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
resp = make_response("Cookie deleted")
resp.delete_cookie(
    key,                # Cookie name (required)
    path='/',           # Must match the path used when setting
    domain=None,        # Must match the domain used when setting
    secure=False,
    httponly=False,
    samesite=None
)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `key` | Cookie name to delete (required) |
| `path` | Path scope; must match the original cookie's path |
| `domain` | Domain scope; must match the original cookie's domain |
| `secure` | Must match the original cookie's `Secure` attribute |
| `httponly` | Must match the original cookie's `HttpOnly` attribute |
| `samesite` | Must match the original cookie's `SameSite` attribute |

**Syntax Rules:**

- The `key` parameter is required; it must be the name of the cookie to delete.
- The `path` and `domain` parameters must match those used when the cookie was set; otherwise, the deletion will not work.
- `delete_cookie()` sets the cookie's expiry to a past date, causing the browser to remove it.
- The response body can be anything; the deletion happens via the `Set-Cookie` header.

**Constraints and Limitations:**

- If the `path` or `domain` does not match the original cookie, the browser will not delete it.
- Cookies set with `Secure` or `HttpOnly` must be deleted with the same attributes to ensure the browser matches them.
- Some browsers may not delete cookies immediately; the change takes effect on the next request.

### Annotated Code Examples

**Example 1: Deleting a Cookie**

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route("/delete-theme")
def delete_theme():
    resp = make_response("Theme cookie deleted")
    resp.delete_cookie("theme")
    return resp
```

**Expected Output:**
- `GET /delete-theme` → response header `Set-Cookie: theme=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Max-Age=0; Path=/`

**Why this output:** `delete_cookie()` sends a `Set-Cookie` header with an empty value and an expired date. The browser interprets this as a request to remove the cookie.

**Example 2: Deleting a Cookie with Path and Domain**

```python
@app.route("/delete-session")
def delete_session():
    resp = make_response("Session deleted")
    resp.delete_cookie("session_id", path="/app", domain=".example.com")
    return resp
```

**Expected Output:**
- `GET /delete-session` → `Set-Cookie: session_id=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Max-Age=0; Path=/app; Domain=.example.com`

**Why this output:** The `path` and `domain` parameters must match the original cookie's scope for the browser to delete it. If they differ, the deletion will not affect the intended cookie.

### Real-World Cases

- **Logout:** Deleting the session cookie when the user logs out.
- **Cookie consent:** Deleting tracking cookies when the user withdraws consent.
- **Preference reset:** Deleting theme or language cookies when the user resets settings.

### References

- Flask API: `Response.delete_cookie` — https://flask.palletsprojects.com/en/stable/api/#flask.Response.delete_cookie
- Stack Overflow: How to remove cookies in Flask — https://stackoverflow.com/questions/14349155/how-to-remove-cookies-in-flask
- MDN: Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie

---

## 5. Signed and Encrypted Cookies (Flask Session Middleware and Custom Cryptography)

### Definitions

**Core Definition:** Signed cookies are cookies whose contents are cryptographically signed to detect tampering, while encrypted cookies are both signed and encrypted to prevent the client from reading their contents.

**Technical Definition:** Flask's default session implementation uses `itsdangerous` to sign (not encrypt) session data with the application's `SECRET_KEY`. The session data is serialized, compressed, and signed; the resulting token is stored in a cookie named `session` by default. The client can read the data (it is Base64-encoded) but cannot modify it without invalidating the signature. For encryption, developers can use libraries like `cryptography` or `Flask-Session` with server-side storage.

**Beginner-Friendly Explanation:** A signed cookie is like a sealed envelope—the client can see what's inside, but if they change anything, the seal breaks and the server rejects it. An encrypted cookie is like a locked box—the client can't see inside at all. Flask's session cookie is signed (not encrypted), so never store sensitive data in it.

### Purposes

- To store session data on the client without risk of tampering.
- To implement stateless authentication and state management.
- To protect session data from unauthorized modification.
- To provide a lightweight alternative to server-side session storage.
- To enable encrypted cookie storage for sensitive data using custom cryptography.

### Syntax Rules and Structure

**Flask Session (Signed Cookies):**

```python
from flask import Flask, session

app = Flask(__name__)
app.secret_key = "your-secret-key"

@app.route("/set-session")
def set_session():
    session["username"] = "alice"
    session["role"] = "admin"
    return "Session set"

@app.route("/get-session")
def get_session():
    username = session.get("username", "guest")
    return f"Hello, {username}!"
```

**Custom Encrypted Cookies:**

```python
from cryptography.fernet import Fernet

key = Fernet.generate_key()
cipher = Fernet(key)

@app.route("/set-encrypted")
def set_encrypted():
    token = cipher.encrypt(b"secret data")
    resp = make_response("Encrypted cookie set")
    resp.set_cookie("encrypted", token.decode(), secure=True, httponly=True)
    return resp

@app.route("/get-encrypted")
def get_encrypted():
    token = request.cookies.get("encrypted")
    if token:
        data = cipher.decrypt(token.encode()).decode()
        return f"Decrypted: {data}"
    return "No cookie"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `app.secret_key` | Secret key for signing the session cookie |
| `session` | Flask's session object (a `SecureCookieSession`) |
| `itsdangerous` | Library used by Flask for signing |
| `Fernet` | Symmetric encryption from the `cryptography` library |
| `cipher.encrypt()` | Encrypts data into a token |
| `cipher.decrypt()` | Decrypts a token back to data |

**Syntax Rules:**

- A `SECRET_KEY` must be set for Flask's session system to work.
- Session data is stored in a cookie named `session` by default.
- The session cookie is signed (not encrypted); never store sensitive data in it.
- For encryption, use a library like `cryptography` (Fernet) or `Flask-Session` with a server-side backend.
- `SESSION_COOKIE_SECURE`, `SESSION_COOKIE_HTTPONLY`, and `SESSION_COOKIE_SAMESITE` configure the session cookie's attributes.

**Constraints and Limitations:**

- Flask's session cookie is limited to 4KB; store only minimal data (e.g., user ID).
- The `SECRET_KEY` must be kept secret; if compromised, attackers can forge sessions.
- Signed cookies are not encrypted; session data is visible to the client.
- Server-side sessions (e.g., Redis) are required for large or sensitive session data.

### Annotated Code Examples

**Example 1: Flask Session (Signed Cookie)**

```python
from flask import Flask, session, redirect, url_for

app = Flask(__name__)
app.secret_key = "a-very-secret-key"

@app.route("/login")
def login():
    session["username"] = "alice"
    session["role"] = "admin"
    return redirect(url_for("profile"))

@app.route("/profile")
def profile():
    if "username" not in session:
        return redirect(url_for("login"))
    return f"Hello, {session['username']}! Role: {session['role']}"

@app.route("/logout")
def logout():
    session.clear()
    return "Logged out"
```

**Expected Output:**
- `GET /login` → redirects to `/profile`; the session cookie is set.
- `GET /profile` → `"Hello, alice! Role: admin"`
- `GET /logout` → session cleared.

**Why this output:** Flask signs the session data with the `SECRET_KEY` and stores it in a cookie. The data is visible to the client (Base64-encoded) but cannot be modified without invalidating the signature.

**Example 2: Custom Encrypted Cookie**

```python
from flask import Flask, make_response, request
from cryptography.fernet import Fernet

app = Flask(__name__)
key = Fernet.generate_key()
cipher = Fernet(key)

@app.route("/set-secret")
def set_secret():
    token = cipher.encrypt(b"sensitive-data")
    resp = make_response("Secret cookie set")
    resp.set_cookie("secret", token.decode(), secure=True, httponly=True, samesite="Strict")
    return resp

@app.route("/get-secret")
def get_secret():
    token = request.cookies.get("secret")
    if not token:
        return "No secret cookie"
    try:
        data = cipher.decrypt(token.encode()).decode()
        return f"Secret: {data}"
    except Exception:
        return "Invalid cookie"
```

**Expected Output:**
- `GET /set-secret` → sets an encrypted cookie.
- `GET /get-secret` → `"Secret: sensitive-data"`.

**Why this output:** The `Fernet` cipher encrypts the data, so the client cannot read it. The server decrypts the token when the cookie is received. The `secure`, `httponly`, and `samesite` attributes provide additional protection.

### Real-World Cases

- **Stateless authentication:** Storing user ID and role in a signed session cookie.
- **Remember-me tokens:** Storing a signed token in a persistent cookie.
- **Sensitive data:** Encrypting cookies that contain PII or tokens.
- **Multi-server deployments:** Using signed cookies to share session state across servers without a central store.

### References

- Flask Sessions — https://flask.palletsprojects.com/en/stable/quickstart/#sessions
- Flask Configuration: `SECRET_KEY` — https://flask.palletsprojects.com/en/stable/config/#SECRET_KEY
- Flask Configuration: `SESSION_COOKIE_SECURE` — https://flask.palletsprojects.com/en/stable/config/#SESSION_COOKIE_SECURE
- Flask Configuration: `SESSION_COOKIE_HTTPONLY` — https://flask.palletsprojects.com/en/stable/config/#SESSION_COOKIE_HTTPONLY
- Flask Configuration: `SESSION_COOKIE_SAMESITE` — https://flask.palletsprojects.com/en/stable/config/#SESSION_COOKIE_SAMESITE
- Werkzeug SecureCookie — https://werkzeug.palletsprojects.com/en/stable/contrib/securecookie/
- itsdangerous Documentation — https://itsdangerous.palletsprojects.com/
- cryptography (Fernet) — https://cryptography.io/en/latest/fernet/
- Flask-Session — https://flask-session.readthedocs.io/

---

## References

- Flask Quickstart: Cookies — https://flask.palletsprojects.com/en/stable/quickstart/#cookies
- Flask Quickstart: Sessions — https://flask.palletsprojects.com/en/stable/quickstart/#sessions
- Flask API: `request.cookies` — https://flask.palletsprojects.com/en/stable/api/#flask.Request.cookies
- Flask API: `Response.set_cookie` — https://flask.palletsprojects.com/en/stable/api/#flask.Response.set_cookie
- Flask API: `Response.delete_cookie` — https://flask.palletsprojects.com/en/stable/api/#flask.Response.delete_cookie
- Flask Configuration: `SECRET_KEY` — https://flask.palletsprojects.com/en/stable/config/#SECRET_KEY
- Flask Configuration: `SESSION_COOKIE_SECURE` — https://flask.palletsprojects.com/en/stable/config/#SESSION_COOKIE_SECURE
- Flask Configuration: `SESSION_COOKIE_HTTPONLY` — https://flask.palletsprojects.com/en/stable/config/#SESSION_COOKIE_HTTPONLY
- Flask Configuration: `SESSION_COOKIE_SAMESITE` — https://flask.palletsprojects.com/en/stable/config/#SESSION_COOKIE_SAMESITE
- Flask Security Best Practices (2026 Guide) — https://safeguard.sh/resources/blog/flask-security-best-practices
- RFC 6265: HTTP State Management Mechanism — https://www.rfc-editor.org/rfc/rfc6265
- RFC 6265bis: SameSite — https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis/
- MDN: Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- MDN: SameSite Cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- web.dev: SameSite Cookies Explained — https://web.dev/articles/samesite-cookies-explained
- Werkzeug SecureCookie — https://werkzeug.palletsprojects.com/en/stable/contrib/securecookie/
- itsdangerous Documentation — https://itsdangerous.palletsprojects.com/
- cryptography (Fernet) — https://cryptography.io/en/latest/fernet/
- Flask-Session — https://flask-session.readthedocs.io/
- Stack Overflow: How to remove cookies in Flask — https://stackoverflow.com/questions/14349155/how-to-remove-cookies-in-flask
- Stack Overflow: Flask session cookie security — https://stackoverflow.com/questions/34902378/where-do-i-get-a-secret-key-for-flask