# PHP Cookies & Transport Security — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP Cookies & Transport Security is the practice of using HTTP cookies to store small pieces of state on the client's browser, while applying security attributes and transport-layer protections (HTTPS/TLS) to prevent theft, tampering, cross-site leakage, and privacy violations.

**Technical Definition**  
HTTP cookies are name/value pairs defined by RFC 6265 and transmitted via the `Set-Cookie` response header and `Cookie` request header. In PHP, cookies are created with `setcookie()` or `setrawcookie()`. The security posture of a cookie is determined by a set of attributes embedded in the `Set-Cookie` header: `Expires`/`Max-Age` (lifespan), `Secure` (TLS-only transmission), `HttpOnly` (blocking JavaScript access), `SameSite` (`Strict`, `Lax`, `None` — controlling cross-site attachment), and `Partitioned` (CHIPS — isolating the cookie to a specific top-level site). Transport security is provided by HTTPS, which encrypts the cookie in transit and prevents network eavesdropping and man-in-the-middle (MITM) tampering.

**Beginner-Friendly Explanation**  
A cookie is like a name tag the server gives your browser. Every time you visit the same website, your browser shows the name tag, and the server knows who you are. But name tags can be stolen or misused. Security flags are stickers on the name tag that say: "only show this over a locked (HTTPS) connection" (`Secure`), "don't let JavaScript read it" (`HttpOnly`), "don't show it to other websites" (`SameSite`), and "keep it separate for each website" (`Partitioned`). Using these flags, together with HTTPS, is how you keep cookies safe.

---

### Key Characteristics

- Cookies are stored client-side and sent automatically with matching requests.
- `setcookie()` URL-encodes the value; `setrawcookie()` does not.
- Cookies must be set before any output is sent (unless output buffering is active).
- The `$_COOKIE` superglobal contains cookies sent by the client on the current request.
- `Secure` cookies are only transmitted over HTTPS.
- `HttpOnly` cookies are invisible to JavaScript's `document.cookie`.
- `SameSite` controls whether cookies are attached to cross-site requests.
- `Partitioned` (CHIPS) cookies are isolated per top-level site.
- Cookie prefixes (`__Secure-`, `__Host-`) enforce additional security constraints.
- Cookies are limited to ~4 KB per cookie and ~50 cookies per domain.
- Cookies are not encrypted by default; HTTPS protects them in transit.
- Deprecated/unsafe: `setrawcookie()` with untrusted values, cookies without `SameSite`, cookies without `Secure` over HTTPS.

---

### Prerequisites

- Basic PHP syntax: functions, superglobals, headers.
- Basic HTTP: request/response headers, status codes.
- HTML forms and sessions.
- A web server with PHP and HTTPS/TLS configured.
- Understanding of cookies and browser storage.

---

### Related Programming Areas

- HTTP protocol and RFC 6265.
- TLS/HTTPS and certificate management.
- Session management (`session_start()`, `session_set_cookie_params()`).
- CSRF protection and SameSite.
- Browser privacy features (CHIPS, third-party cookie restrictions).
- Content Security Policy (CSP) as defense-in-depth.

---

## Core Concepts / Features

## 1. Stateful Placement: Setting Cookies with `setcookie()` and `setrawcookie()`

### Definitions

**Core Definition**  
`setcookie()` and `setrawcookie()` are PHP functions that send a `Set-Cookie` HTTP header to the browser, instructing it to store a name/value pair with optional attributes.

**Technical Definition**  
`setcookie(string $name, string $value = "", array|int $expires_or_options = 0, string $path = "", string $domain = "", bool $secure = false, bool $httponly = false)` defines a cookie to be sent with the HTTP response. The value is automatically URL-encoded. `setrawcookie()` is identical except it does not URL-encode the value. As of PHP 7.3, both functions accept an options array as the third argument, enabling `expires`, `path`, `domain`, `secure`, `httponly`, and `samesite` keys. Cookies are stored by the browser and sent back in the `Cookie` request header on subsequent matching requests; PHP populates `$_COOKIE` from that header.

**Beginner-Friendly Explanation**  
`setcookie()` is how PHP tells the browser "remember this little note." You give it a name, a value, and optionally how long it should last, which pages it applies to, and how secure it should be. `setrawcookie()` does the same thing but does not encode the value, which is useful when you need exact control over special characters.

### Purposes

- To store small pieces of state on the client (preferences, tokens, identifiers).
- To implement "remember me" functionality.
- To track sessions via session cookies.
- To store language, theme, and locale preferences.
- To carry CSRF tokens or nonces.
- To support analytics and A/B testing (with privacy considerations).
- To persist data between requests without server-side session storage.

### Syntax Rules and Structure

**General Syntax: `setcookie()` (Modern Options Array)**
```php
setcookie(
    'cookie_name',                  // name (required)
    'cookie_value',                 // value (optional)
    [
        'expires'  => time() + 3600, // expiration Unix timestamp
        'path'     => '/',            // path scope
        'domain'   => 'example.com',  // domain scope
        'secure'   => true,           // HTTPS only
        'httponly' => true,           // block JavaScript access
        'samesite' => 'Lax'           // SameSite policy
    ]
);
```

**Component Breakdown**

- `cookie_name` — the cookie name; cannot contain `=`, `,`, `;`, whitespace, or control characters.
- `cookie_value` — the value; automatically URL-encoded by `setcookie()`.
- `expires` — Unix timestamp; `0` creates a session cookie (deleted when browser closes).
- `path` — URL path the cookie applies to; `/` means the entire site.
- `domain` — domain scope; empty means the current host only.
- `secure` — if `true`, only sent over HTTPS.
- `httponly` — if `true`, inaccessible to JavaScript.
- `samesite` — `'Strict'`, `'Lax'`, or `'None'`.

**General Syntax: `setcookie()` (Legacy Positional Parameters)**
```php
setcookie(
    'cookie_name',            // name
    'cookie_value',           // value
    time() + 3600,            // expires
    '/',                      // path
    'example.com',            // domain
    true,                     // secure
    true                      // httponly
);
```

**Component Breakdown**

- Positional arguments map to the same attributes as the options array.
- No `samesite` parameter in the legacy signature; use the options array or `header()` for `SameSite`.

**General Syntax: `setrawcookie()`**
```php
setrawcookie(
    'cookie_name',
    'raw value with spaces & symbols',
    [
        'expires'  => time() + 3600,
        'path'     => '/',
        'secure'   => true,
        'httponly' => true,
        'samesite' => 'Strict'
    ]
);
```

**Component Breakdown**

- Identical to `setcookie()` except the value is not URL-encoded.
- The browser may interpret certain characters as delimiters; use with caution.
- `$_COOKIE` will contain the raw (undecoded) value.

**General Syntax: Deleting a Cookie**
```php
setcookie('cookie_name', '', [
    'expires'  => time() - 3600,
    'path'     => '/',
    'domain'   => 'example.com',
    'secure'   => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);
```

**Component Breakdown**

- Setting an expired timestamp deletes the cookie.
- Attributes must match the original cookie for the browser to identify it.

**Syntax Rules**

- `setcookie()` must be called before any output is sent to the browser (unless output buffering is active).
- Cookies are sent as HTTP headers; use `headers_sent()` to check.
- The value is automatically URL-encoded by `setcookie()`; `setrawcookie()` skips encoding.
- Cookie names cannot contain `=`, `,`, `;`, whitespace, or control characters.
- Cookies with the same name, path, and domain overwrite each other.
- Multiple values can be stored under one name using array syntax (`name[]`).
- The options array syntax requires PHP 7.3 or later.
- `SameSite=None` requires `Secure=true` in modern browsers.
- Cookies are not available in `$_COOKIE` until the next request after being set.

**Constraints and Limitations**

- Cookies are limited to ~4 KB per cookie and ~50 cookies per domain.
- Cookies are stored client-side and can be viewed, modified, or deleted by the user.
- Cookies are sent on every matching request, adding to request size.
- `setcookie()` cannot be used after headers are sent.
- `setrawcookie()` can produce invalid headers if the value contains newlines or control characters.
- Cookies are not encrypted; HTTPS is required to protect them in transit.
- Some browsers cap cookie lifetime (e.g., 7 days for certain types).
- Third-party cookie restrictions are increasingly strict.

### Annotated Code Examples and Expected Outputs

#### Example 1: Setting a Secure Preference Cookie

**File: `set_preference.php`**
```php
<?php
// Set a theme preference cookie with security attributes.
setcookie('theme', 'dark', [
    'expires'  => time() + 60 * 60 * 24 * 30, // 30 days
    'path'     => '/',
    'secure'   => true,     // HTTPS only
    'httponly' => true,     // JavaScript cannot read it
    'samesite' => 'Lax'     // restrict cross-site sending
]);

echo 'Preference cookie set.';
?>
```

**How to Run**

1. Save as `set_preference.php` on an HTTPS server.
2. Visit the page in a browser.

**Expected Output (rendered)**
```
Preference cookie set.
```

**HTTP Response Header (simplified)**
```
Set-Cookie: theme=dark; Expires=...; Max-Age=2592000; Path=/; Secure; HttpOnly; SameSite=Lax
```

**Why This Output Occurs**

- `setcookie()` sends a `Set-Cookie` header with the specified attributes.
- The browser stores the cookie and will send it back on future HTTPS requests to `/`.
- `Secure` ensures the cookie is not sent over plain HTTP.
- `HttpOnly` prevents JavaScript access.
- `SameSite=Lax` restricts cross-site transmission.

#### Example 2: Using `setrawcookie()` for a Raw Value

**File: `raw_cookie.php`**
```php
<?php
// Set a raw cookie (value is not URL-encoded).
setrawcookie('raw_data', 'a b&c=d', [
    'expires'  => time() + 3600,
    'path'     => '/',
    'secure'   => true,
    'httponly' => true
]);

// Read it back on the next request.
if (isset($_COOKIE['raw_data'])) {
    echo 'Raw cookie value: ' . htmlspecialchars($_COOKIE['raw_data'], ENT_QUOTES, 'UTF-8');
}
?>
```

**Expected Output (second request)**
```
Raw cookie value: a b&c=d
```

**Why This Output Occurs**

- `setrawcookie()` sends the value without URL-encoding.
- The browser stores the raw value.
- `$_COOKIE['raw_data']` contains the raw value on the next request.
- `htmlspecialchars()` escapes it for HTML output.

### Real-World Cases

- **Theme/language preferences:** Store user choices.
- **Remember-me tokens:** Store a long-lived token.
- **Analytics IDs:** Store a visitor identifier (with privacy consent).
- **Shopping cart IDs:** Store a cart identifier.
- **A/B testing:** Store a variant identifier.

---

## 2. The Security Parameter Flags

### 2.1 `expires` / `max-age` — Tracking Lifespan Rules

#### Definitions

**Core Definition**  
`expires` and `max-age` control how long a cookie remains stored in the browser. `expires` specifies an absolute date/time; `max-age` specifies a relative number of seconds.

**Technical Definition**  
The `Expires` attribute is an HTTP-date (e.g., `Wed, 21 Oct 2026 07:28:00 GMT`) after which the cookie is invalid. The `Max-Age` attribute is a non-zero decimal integer representing the cookie's lifetime in seconds from the time of receipt. If both are present, `Max-Age` takes precedence. A value of `0` for `expires` or a negative `max-age` deletes the cookie. Session cookies (no `expires`/`max-age`) are deleted when the browser session ends.

**Beginner-Friendly Explanation**  
`expires` is like writing a specific "best before" date on a carton of milk. `max-age` is like saying "good for 30 days from now." The browser uses whichever is present; if both are present, `max-age` wins. If you set either to a past value, the browser throws the cookie away immediately.

#### Purposes

- To persist cookies across browser sessions.
- To set temporary cookies for short-lived tokens.
- To delete cookies programmatically.
- To control cookie lifetime for security (shorter is safer for sensitive data).
- To comply with privacy regulations (e.g., consent expiry).
- To implement "remember me" with a defined duration.

#### Syntax Rules and Structure

**General Syntax: `expires` (Unix Timestamp)**
```php
setcookie('name', 'value', [
    'expires' => time() + 60 * 60 * 24 * 30  // 30 days
]);
```

**Component Breakdown**

- `time()` — current Unix timestamp.
- `+ 60 * 60 * 24 * 30` — 30 days in seconds.
- `0` — session cookie (deleted on browser close).
- Past timestamp — deletes the cookie.

**General Syntax: `max-age`**
```php
setcookie('name', 'value', [
    'max_age' => 3600  // 1 hour
]);
```

**Component Breakdown**

- `max_age` — relative lifetime in seconds.
- `0` or negative — deletes the cookie.
- Supported in PHP 7.3+ options array.

**General Syntax: Legacy `expires`**
```php
setcookie('name', 'value', time() + 3600);
```

**Component Breakdown**

- Third positional argument is the Unix timestamp.

**Syntax Rules**

- `expires` is an absolute Unix timestamp; `max-age` is a relative number of seconds.
- If both are present, `max-age` takes precedence.
- Session cookies omit both; they are deleted when the browser closes.
- Setting `expires` to `0` creates a session cookie.
- Setting `expires` to a past timestamp deletes the cookie.
- The maximum expiration is 03:14:07 UTC on 19 January 2038 (32-bit timestamp limit).
- Some browsers cap cookie lifetime (e.g., 7 days for certain cookie types).
- `max-age` is more robust across time zones and clock skew.

**Constraints and Limitations**

- Browser policies may cap the lifetime of certain cookies.
- Clock skew between client and server can affect `expires` accuracy.
- `max-age` is supported by all modern browsers but not by some older ones.
- Session cookies may be restored by browsers after crash recovery.
- Long-lived cookies increase the window of exposure if stolen.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Session Cookie vs. Persistent Cookie

```php
<?php
// Session cookie: deleted when browser closes.
setcookie('session_pref', 'blue', [
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);

// Persistent cookie: 30 days.
setcookie('persistent_pref', 'dark', [
    'expires' => time() + 60 * 60 * 24 * 30,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);

// Delete a cookie: set expiry in the past.
setcookie('old_cookie', '', [
    'expires' => time() - 3600,
    'path' => '/'
]);
?>
```

**Expected HTTP Headers**
```
Set-Cookie: session_pref=blue; Path=/; Secure; HttpOnly; SameSite=Lax
Set-Cookie: persistent_pref=dark; Expires=...; Max-Age=2592000; Path=/; Secure; HttpOnly; SameSite=Lax
Set-Cookie: old_cookie=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Path=/
```

**Why This Output Occurs**

- The session cookie has no `Expires`/`Max-Age`, so it is session-scoped.
- The persistent cookie has an `Expires` timestamp 30 days in the future.
- The deleted cookie has an expired timestamp.

#### Real-World Cases

- **Remember me:** Persistent cookie with a 30-day `Max-Age`.
- **Shopping cart:** Session cookie for the cart ID.
- **Consent banner:** Cookie with a 6-month expiry.
- **Temporary tokens:** Short `Max-Age` (e.g., 5 minutes).
- **Logout:** Delete the session cookie by setting a past expiry.

---

### 2.2 `Secure` — Restricting Transmission to TLS/HTTPS

#### Definitions

**Core Definition**  
The `Secure` flag instructs the browser to send the cookie only over HTTPS (TLS-encrypted) connections, never over plain HTTP.

**Technical Definition**  
When `Secure` is set, the browser includes the cookie in requests only when the connection uses a secure protocol (HTTPS). This prevents the cookie from being transmitted in cleartext over HTTP, where it could be intercepted by a network eavesdropper or MITM attacker. The `Secure` attribute is defined in RFC 6265 §4.1.2.5. It does not encrypt the cookie itself; it relies on TLS for confidentiality and integrity.

**Beginner-Friendly Explanation**  
`Secure` is like a "sealed envelope only" rule. The browser will only show the cookie over a locked (HTTPS) connection. If someone tries to send it over an unlocked (HTTP) connection, the browser refuses. This keeps the cookie safe from spies on the network.

#### Purposes

- To prevent cookie theft via network eavesdropping.
- To prevent MITM attacks from reading or modifying cookies.
- To protect session identifiers and authentication tokens.
- To comply with security standards (OWASP, PCI DSS).
- To ensure cookies are not sent over insecure HTTP.
- To work with `SameSite=None`, which requires `Secure`.

#### Syntax Rules and Structure

**General Syntax: `Secure` Flag**
```php
setcookie('name', 'value', [
    'secure' => true
]);
```

**Component Breakdown**

- `secure => true` — adds the `Secure` attribute to the `Set-Cookie` header.
- `secure => false` (default) — cookie can be sent over HTTP and HTTPS.

**General Syntax: Session Cookie Secure Flag**
```php
session_set_cookie_params([
    'secure' => true
]);
session_start();
```

**Component Breakdown**

- `session_set_cookie_params()` configures the session cookie attributes.
- `secure => true` applies to the session cookie.

**General Syntax: `php.ini` Configuration**
```ini
session.cookie_secure = 1
```

**Component Breakdown**

- `session.cookie_secure = 1` makes the session cookie `Secure` globally.
- Applies to all session cookies set by PHP.

**Syntax Rules**

- `Secure` cookies are only sent over HTTPS.
- If the site is served over plain HTTP, `Secure` cookies are never sent, breaking functionality.
- `SameSite=None` requires `Secure=true` in modern browsers.
- `Secure` does not protect against XSS or CSRF.
- `Secure` is ignored by some very old browsers; modern browsers support it universally.
- The cookie is still stored on the client; `Secure` only restricts transmission.

**Constraints and Limitations**

- Requires HTTPS; plain HTTP sites cannot use `Secure` cookies effectively.
- `Secure` does not encrypt the cookie; TLS does.
- `Secure` does not prevent client-side access (use `HttpOnly`).
- Mixed content (HTTPS page loading HTTP resources) may cause `Secure` cookies to be sent incorrectly.
- Some corporate proxies may strip `Secure` attributes.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Setting a Secure Cookie

```php
<?php
// Only set if the connection is HTTPS.
if (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off') {
    setcookie('secure_token', bin2hex(random_bytes(16)), [
        'expires'  => time() + 3600,
        'path'     => '/',
        'secure'   => true,
        'httponly' => true,
        'samesite' => 'Strict'
    ]);
    echo 'Secure cookie set.';
} else {
    echo 'HTTPS required.';
}
?>
```

**Expected Output (HTTPS)**
```
Secure cookie set.
```
**Expected Output (HTTP)**
```
HTTPS required.
```

**Why This Output Occurs**

- The script checks `$_SERVER['HTTPS']` before setting the cookie.
- The cookie is only set over HTTPS, where `Secure` is meaningful.
- The `Secure` attribute prevents transmission over HTTP.

#### Real-World Cases

- **Session cookies:** Always set `Secure` on HTTPS sites.
- **Authentication tokens:** Protect tokens from network theft.
- **Payment forms:** Protect payment-related cookies.
- **OAuth tokens:** Secure transmission is mandatory.
- **API keys in cookies:** Protect from interception.

---

### 2.3 `HttpOnly` — Mitigating Token Extraction via JavaScript

#### Definitions

**Core Definition**  
The `HttpOnly` flag instructs the browser to make the cookie inaccessible to JavaScript through the `document.cookie` API, mitigating cookie theft via Cross-Site Scripting (XSS).

**Technical Definition**  
When `HttpOnly` is set, the browser excludes the cookie from the `document.cookie` property and other JavaScript-accessible cookie APIs. The cookie is still sent with HTTP requests and is available in `$_COOKIE`. This prevents an attacker who injects JavaScript (via XSS) from reading and exfiltrating the cookie. The `HttpOnly` attribute is defined in RFC 6265 §4.1.2.6 and is supported by all modern browsers.

**Beginner-Friendly Explanation**  
`HttpOnly` is like putting your name tag in a locked box that only the server can open. JavaScript running in the page — including malicious scripts injected by an attacker — cannot read it. This means even if an attacker finds an XSS bug, they cannot steal the cookie.

#### Purposes

- To prevent XSS attacks from stealing session cookies.
- To protect authentication tokens from JavaScript-based exfiltration.
- To reduce the impact of XSS vulnerabilities.
- To comply with OWASP and NIST security guidelines.
- To protect sensitive cookies from browser extensions and injected scripts.
- To complement CSRF tokens and other defenses.

#### Syntax Rules and Structure

**General Syntax: `HttpOnly` Flag**
```php
setcookie('name', 'value', [
    'httponly' => true
]);
```

**Component Breakdown**

- `httponly => true` — adds the `HttpOnly` attribute.
- `httponly => false` (default) — cookie is accessible to JavaScript.

**General Syntax: Session Cookie HttpOnly**
```php
session_set_cookie_params([
    'httponly' => true
]);
session_start();
```

**Component Breakdown**

- Applies `HttpOnly` to the session cookie.

**General Syntax: `php.ini` Configuration**
```ini
session.cookie_httponly = 1
```

**Component Breakdown**

- `session.cookie_httponly = 1` makes the session cookie `HttpOnly` globally.

**Syntax Rules**

- `HttpOnly` cookies are not accessible via `document.cookie`.
- The cookie is still sent with HTTP requests.
- `HttpOnly` does not prevent CSRF; use `SameSite` and CSRF tokens.
- `HttpOnly` does not prevent network eavesdropping; use `Secure`.
- `HttpOnly` is supported by all modern browsers.
- The flag is set in the `Set-Cookie` header and cannot be changed by JavaScript.

**Constraints and Limitations**

- `HttpOnly` does not protect against other attack vectors (e.g., network sniffing, CSRF).
- Some legitimate JavaScript functionality may require access to the cookie; use alternative mechanisms (e.g., meta tags).
- `HttpOnly` does not prevent XSS from performing actions on behalf of the user.
- Browser extensions may still access `HttpOnly` cookies in some cases.
- `HttpOnly` is not a substitute for input validation and output encoding.

#### Annotated Code Examples and Expected Outputs

##### Example 1: HttpOnly Session Cookie

```php
<?php
// Configure session cookie with HttpOnly.
session_set_cookie_params([
    'lifetime' => 0,
    'path'     => '/',
    'secure'   => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);
session_start();

$_SESSION['user_id'] = 42;
echo 'Session started with HttpOnly cookie.';
?>
```

**Expected HTTP Header**
```
Set-Cookie: PHPSESSID=...; Path=/; Secure; HttpOnly; SameSite=Lax
```

**Why This Output Occurs**

- `session_set_cookie_params()` configures the session cookie with `HttpOnly`.
- The browser stores the cookie but does not expose it to JavaScript.
- XSS scripts cannot read the session ID.

#### Real-World Cases

- **Session cookies:** Always set `HttpOnly`.
- **Authentication tokens:** Protect from XSS theft.
- **CSRF tokens:** Store in `HttpOnly` cookies when possible.
- **API tokens:** Protect from JavaScript exfiltration.
- **Admin panels:** High-value targets for XSS.

---

### 2.4 `SameSite` — Configuring Strict Transport Barriers

#### Definitions

**Core Definition**  
The `SameSite` attribute controls whether a cookie is attached to cross-site requests. It accepts three values: `Strict`, `Lax`, and `None`, each offering a different balance between security and functionality.

**Technical Definition**  
`SameSite=Strict` attaches the cookie only to requests originating from the same site (same registrable domain). `SameSite=Lax` attaches the cookie to same-site requests and top-level cross-site navigations with safe methods (GET). `SameSite=None` attaches the cookie to all requests, including cross-site, but requires `Secure=true`. The attribute is defined in RFC 6265bis and is a primary defense against CSRF, XSSI, and cross-site tracking.

**Beginner-Friendly Explanation**  
`SameSite` is like a rule about who can show your name tag. `Strict` says "only show it when you're on my website." `Lax` says "show it when you're on my website or when you click a link to come here." `None` says "show it everywhere" (but only over HTTPS). `Strict` is the safest, `Lax` is a good balance, and `None` is needed for some legitimate cross-site features.

#### Purposes

- To prevent CSRF attacks by blocking cross-site cookie attachment.
- To mitigate cross-site script inclusion (XSSI) attacks.
- To reduce cross-site tracking and privacy leakage.
- To control cookie behavior for embedded content.
- To comply with browser privacy defaults (Chrome, Firefox).
- To balance security with legitimate cross-site use cases.

#### Syntax Rules and Structure

**General Syntax: SameSite Values**
```php
setcookie('name', 'value', [
    'samesite' => 'Strict'   // or 'Lax' or 'None'
]);
```

**Component Breakdown**

- `'Strict'` — cookie sent only on same-site requests.
- `'Lax'` — cookie sent on same-site requests and top-level GET navigations.
- `'None'` — cookie sent on all requests; requires `Secure`.

**General Syntax: Session Cookie SameSite**
```php
session_set_cookie_params([
    'samesite' => 'Lax'
]);
session_start();
```

**Component Breakdown**

- Applies `SameSite` to the session cookie.

**General Syntax: `php.ini` Configuration**
```ini
session.cookie_samesite = "Lax"
```

**Component Breakdown**

- `session.cookie_samesite = "Lax"` sets the default for session cookies.
- Available since PHP 7.3.

**SameSite Value Comparison**

| Value | Same-site requests | Cross-site GET navigation | Cross-site POST/iframe | CSRF protection | Use case |
|---|---|---|---|---|---|
| `Strict` | Sent | Not sent | Not sent | Strongest | High-security, no cross-site needs |
| `Lax` | Sent | Sent | Not sent | Good | Default for most sites |
| `None` | Sent | Sent | Sent | None (relies on other defenses) | Cross-site embeds, OAuth |

**Syntax Rules**

- `SameSite=None` requires `Secure=true` in modern browsers.
- `Lax` is the default in Chrome 80+ and Firefox for cookies without an explicit `SameSite`.
- `Strict` is the most secure but may break legitimate cross-site flows (e.g., OAuth callbacks).
- `Lax` allows top-level GET navigations; it does not send cookies on cross-site POST, iframes, or AJAX.
- `SameSite` is ignored by older browsers; use CSRF tokens as defense-in-depth.
- The attribute is set in the `Set-Cookie` header.

**Constraints and Limitations**

- `SameSite=Strict` can break login flows from external links (user appears logged out until they navigate).
- `SameSite=Lax` still sends cookies on top-level GET navigations, which can be exploited in some scenarios.
- `SameSite=None` provides no CSRF protection; use CSRF tokens.
- Browser behavior varies; some browsers have different defaults and enforcement.
- `SameSite` does not protect against XSS.
- The attribute is not supported by very old browsers.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Comparing SameSite Values

```php
<?php
// Strict: most secure, breaks external links.
setcookie('strict_cookie', 'value1', [
    'samesite' => 'Strict',
    'secure'   => true,
    'httponly' => true
]);

// Lax: balanced, allows top-level GET.
setcookie('lax_cookie', 'value2', [
    'samesite' => 'Lax',
    'secure'   => true,
    'httponly' => true
]);

// None: cross-site, requires Secure.
setcookie('none_cookie', 'value3', [
    'samesite' => 'None',
    'secure'   => true,
    'httponly' => true
]);
?>
```

**Expected HTTP Headers**
```
Set-Cookie: strict_cookie=value1; Secure; HttpOnly; SameSite=Strict
Set-Cookie: lax_cookie=value2; Secure; HttpOnly; SameSite=Lax
Set-Cookie: none_cookie=value3; Secure; HttpOnly; SameSite=None
```

**Why This Output Occurs**

- Each cookie has a different `SameSite` value.
- `Secure` is required for `None`.
- The browser enforces the cross-site attachment rules.

#### Real-World Cases

- **Banking:** `SameSite=Strict` for maximum CSRF protection.
- **E-commerce:** `SameSite=Lax` for a balance of security and usability.
- **OAuth providers:** `SameSite=None` for cross-site redirects, with CSRF tokens.
- **Embedded widgets:** `SameSite=None` for third-party iframes.
- **Admin panels:** `SameSite=Strict` for high security.

---

### 2.5 `Partitioned` (CHIPS) — Isolating Cookies by Top-Level Site

#### Definitions

**Core Definition**  
The `Partitioned` attribute (Cookies Having Independent Partitioned State, CHIPS) isolates a cookie to a specific top-level site, creating a separate cookie jar per top-level site. This prevents cross-site tracking while allowing legitimate embedded services to function.

**Technical Definition**  
CHIPS introduces a `Partitioned` attribute in the `Set-Cookie` header. When set, the browser stores the cookie under a partition key derived from the top-level site's origin (scheme + registrable domain). The cookie is only sent when the request's top-level site matches the partition key. `Partitioned` requires `Secure=true`. It is designed to replace third-party cookies for legitimate embedded use cases while preventing cross-site tracking. The attribute is defined in the IETF draft and implemented in Chromium-based browsers.

**Beginner-Friendly Explanation**  
Imagine a shared apartment building. Without CHIPS, all tenants share one mailbox, so anyone can read your mail. With CHIPS, each apartment gets its own locked mailbox. A third-party service (like an embedded chat widget) can store a cookie, but it is isolated to the website you are currently visiting. The same service on a different website gets a different cookie. This prevents the service from tracking you across sites while still letting it work.

#### Purposes

- To enable legitimate cross-site cookies without enabling tracking.
- To isolate embedded third-party services per top-level site.
- To comply with privacy regulations and browser third-party cookie restrictions.
- To provide a privacy-preserving alternative to third-party cookies.
- To support embedded authentication, payments, and widgets.
- To prevent cross-site correlation of user activity.

#### Syntax Rules and Structure

**General Syntax: `Partitioned` Attribute**
```php
setcookie('name', 'value', [
    'secure'      => true,
    'partitioned' => true
]);
```

**Component Breakdown**

- `secure => true` — required; `Partitioned` cookies must be Secure.
- `partitioned => true` — adds the `Partitioned` attribute.
- The browser stores the cookie in a partition keyed by the top-level site.

**General Syntax: Session Cookie Partitioned**
```php
session_set_cookie_params([
    'secure'      => true,
    'partitioned' => true
]);
session_start();
```

**Component Breakdown**

- Applies `Partitioned` to the session cookie.
- Requires PHP 8.6+ or the RFC implementation.

**General Syntax: Manual Header (Workaround)**
```php
header('Set-Cookie: name=value; Secure; SameSite=None; Partitioned');
```

**Component Breakdown**

- Use `header()` if `setcookie()` does not yet support `Partitioned`.
- Must include `Secure` and usually `SameSite=None`.

**Syntax Rules**

- `Partitioned` requires `Secure=true`.
- The cookie is isolated per top-level site.
- The partition key is based on the top-level site's origin.
- `Partitioned` cookies are not sent in unpartitioned contexts.
- Domains may use up to 10 KB or 180 cookies per partition.
- `Clear-Site-Data: "cookies"` deletes only the partition in which the header was received.
- The attribute is opt-in; cookies without it remain unpartitioned.

**Constraints and Limitations**

- Browser support is limited to Chromium-based browsers (Chrome, Edge) as of 2026.
- PHP support depends on version; `setcookie()` may not yet support the `partitioned` key.
- `Partitioned` does not prevent all tracking; it reduces cross-site correlation.
- The partition key may include a cross-site ancestor bit in some implementations.
- Cookies with the same name in different partitions are independent.
- `Partitioned` is not a substitute for `SameSite`; use both.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Setting a Partitioned Cookie

```php
<?php
// Set a partitioned cookie (CHIPS).
setcookie('embed_session', 'abc123', [
    'expires'     => time() + 3600,
    'path'        => '/',
    'secure'      => true,
    'httponly'    => true,
    'samesite'    => 'None',
    'partitioned' => true
]);

echo 'Partitioned cookie set.';
?>
```

**Expected HTTP Header**
```
Set-Cookie: embed_session=abc123; Expires=...; Max-Age=3600; Path=/; Secure; HttpOnly; SameSite=None; Partitioned
```

**Why This Output Occurs**

- `partitioned => true` adds the `Partitioned` attribute.
- `Secure` and `SameSite=None` are required for cross-site partitioned cookies.
- The browser stores the cookie in a partition keyed by the top-level site.
- The cookie is only sent when the top-level site matches the partition.

#### Real-World Cases

- **Embedded chat widgets:** Store session per top-level site.
- **Third-party payment iframes:** Isolate payment session per merchant.
- **Embedded authentication:** Support SSO without cross-site tracking.
- **Analytics:** Measure per-site engagement without cross-site correlation.
- **Content recommendations:** Personalize per-site without cross-site tracking.

---

## Security Considerations

### Cookie Prefixes (`__Secure-` and `__Host-`)

- **`__Secure-` prefix:** Cookies with names starting with `__Secure-` must be set with the `Secure` flag; otherwise, the browser rejects them.
- **`__Host-` prefix:** Cookies with names starting with `__Host-` must be set with `Secure`, must not have a `Domain` attribute, and must have `Path=/`. This prevents subdomain injection and path-based attacks.
- **PHP parsing vulnerability:** PHP's parsing of encoded cookie names can allow attackers to forge `__Host-`/`__Secure-` cookies; ensure input validation.

### Transport Security (HTTPS/TLS)

- **HTTPS is mandatory** for `Secure`, `SameSite=None`, and `Partitioned` cookies.
- **TLS encrypts** cookies in transit, preventing eavesdropping and MITM tampering.
- **HSTS** (HTTP Strict Transport Security) enforces HTTPS and prevents downgrade attacks.
- **Certificate validation** must be correct; invalid certificates break HTTPS.
- **Mixed content** (HTTPS page loading HTTP resources) can leak cookies; avoid it.

### Session Cookie Security

- Use `session_set_cookie_params()` or `php.ini` to set `Secure`, `HttpOnly`, and `SameSite` on session cookies.
- PHP RFC "Secure Session Configuration Defaults" (PHP 8.6) proposes secure defaults for `session.use_strict_mode`, `session.cookie_httponly`, and `session.cookie_samesite`.
- Regenerate session IDs after login (`session_regenerate_id(true)`).
- Set `session.use_only_cookies = 1` and `session.use_strict_mode = 1`.

### Common Mistakes

- **Not setting `Secure` on HTTPS sites:** Cookies can leak over HTTP.
- **Not setting `HttpOnly`:** XSS can steal session cookies.
- **Using `SameSite=None` without `Secure`:** Browser rejects the cookie.
- **Using `SameSite=Strict` on login pages:** External links appear logged out.
- **Storing sensitive data in cookies:** Cookies are visible to the user.
- **Not validating cookie values:** Cookies can be tampered with.
- **Using `setrawcookie()` with untrusted input:** Header injection risk.
- **Ignoring cookie prefixes:** Missed protection against subdomain attacks.

### Deprecated, Unsafe, or Version-Specific Features

| Feature | Status | Replacement |
|---|---|---|
| `setcookie()` without `SameSite` | Unsafe (relies on browser default) | Set `SameSite` explicitly |
| `setrawcookie()` with untrusted values | Unsafe | Use `setcookie()` or validate |
| `session.cookie_samesite` empty default | Unsafe | Set `SameSite=Lax` or `Strict` |
| `session.cookie_httponly` default off | Unsafe | Set to `1` |
| `session.use_strict_mode` default off | Unsafe | Set to `1` |
| `SameSite=None` without `Secure` | Rejected by browsers | Add `Secure` |
| `Partitioned` without `Secure` | Rejected | Add `Secure` |
| Legacy positional `setcookie()` | Still supported | Use options array |
| `setcookie()` with `expires` only | Works | Consider `max_age` for robustness |

### References Links

- PHP: `setcookie()` — https://www.php.net/manual/en/function.setcookie.php
- PHP: `setrawcookie()` — https://www.php.net/manual/en/function.setrawcookie.php
- PHP: Cookies — https://www.php.net/manual/en/features.cookies.php
- PHP: `session_set_cookie_params()` — https://www.php.net/manual/en/function.session-set-cookie-params.php
- PHP: Session Configuration — https://www.php.net/manual/en/session.configuration.php
- PHP RFC: Secure Session Configuration Defaults — https://wiki.php.net/rfc/session_security_defaults
- PHP RFC: Cookies Having Independent Partitioned State (CHIPS) — https://wiki.php.net/rfc/chips
- MDN: `Set-Cookie` — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie
- MDN: `SameSite` cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie/SameSite
- MDN: Cookie prefixes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie/Prefixes
- MDN: `HttpOnly` — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie#httponly
- MDN: `Secure` — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie#secure
- RFC 6265: HTTP State Management Mechanism — https://datatracker.ietf.org/doc/html/rfc6265
- RFC 6265bis: Cookies: HTTP State Management Mechanism (draft) — https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis
- IETF: Cookies Having Independent Partitioned State (CHIPS) — https://datatracker.ietf.org/meeting/115/materials/slides-115-httpbis-sessa-partitioned-cookies-00
- Chrome Developers: CHIPS — https://developer.chrome.com/docs/privacy-sandbox/chips/
- OWASP: Session Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP: CSRF Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP: Secure Cookie Attribute — https://owasp.org/www-community/controls/SecureCookieAttribute
- NIST SP 800-63B: Digital Identity Guidelines — https://pages.nist.gov/800-63-4/sp800-63b/session/