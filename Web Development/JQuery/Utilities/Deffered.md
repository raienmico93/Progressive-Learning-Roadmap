# jQuery Deferred Objects — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
A jQuery Deferred Object is a chainable utility object that manages asynchronous operations by tracking their state (pending, resolved, or rejected) and providing methods to register callbacks, invoke callbacks, and relay state to derived promises.

**Technical Definition:**
`jQuery.Deferred()` returns a Deferred object implementing a superset of the Promises/A+ interface. It exposes state-changing methods (`.resolve()`, `.reject()`, `.notify()` and their `With` variants) and callback-registration methods (`.done()`, `.fail()`, `.progress()`, `.always()`, `.then()`, `.pipe()`). The `.promise()` method returns a restricted Promise view that exposes only callback-registration methods, preventing external state changes. Deferreds were introduced in jQuery 1.5 as part of a major Ajax rewrite following the CommonJS Promises/A design, and were made Promises/A+ compliant in jQuery 3.0. Deferreds power jQuery's `$.ajax()` (via the `jqXHR` object) and `$.when()` composition utility.

**Beginner-Friendly Explanation:**
A Deferred object is like a promise you control. Imagine you're waiting for a package. A Deferred is the tracking number — you can check its status, attach callbacks that say "tell me when it arrives" or "tell me if it gets lost," and when the package arrives or is lost, you "resolve" or "reject" the Deferred. Other people can listen to the tracking number, but only you can update its status.

### Key Characteristics

- **Tri-State Model:** A Deferred is always in one of three states: `"pending"`, `"resolved"`, or `"rejected"`. Transitions are one-way and permanent.
- **Producer/Consumer Separation:** The Deferred (producer) exposes state-changing methods; the Promise (consumer) exposes only callback registration. `.promise()` returns the restricted view.
- **Multiple Callbacks:** Any number of callbacks can be attached via `.done()`, `.fail()`, `.always()`, and `.progress()`; they fire in registration order.
- **Late Binding:** Callbacks attached after the Deferred is resolved or rejected fire immediately.
- **Promises/A+ Compliance:** Since jQuery 3.0, `.then()` is Promises/A+ compliant, making Deferreds interoperable with native Promises.
- **Composition:** `$.when()` accepts multiple Deferreds and returns a single Promise that resolves when all resolve, or rejects when any rejects.
- **Deprecated Methods:** `.pipe()` is deprecated (use `.then()`); `.notify()` and `.progress()` are jQuery-specific and not part of Promises/A+.

### Prerequisites

- Solid understanding of JavaScript functions, callbacks, and asynchronous concepts
- Familiarity with jQuery AJAX methods (`$.ajax()`, `$.get()`, `$.post()`)
- Basic knowledge of Promises and asynchronous programming
- Understanding of the `this` context in JavaScript
- Experience with browser developer tools (Console)

### Related Programming Areas

- **Promises/A+ Specification:** The standard that jQuery 3.0+ Deferreds comply with.
- **Native ES6 Promises:** Modern JavaScript's built-in Promise implementation.
- **AJAX:** `jqXHR` is a Deferred-backed Promise returned by `$.ajax()`.
- **Asynchronous Coordination:** `$.when()` composes multiple Deferreds.
- **Event-Driven Programming:** Deferreds complement events by providing a linear, chainable alternative.
- **Async/Await:** Native syntax sugar over Promises; can interoperate with jQuery Deferreds (jQuery Deferreds are thenable).

---

## 1. Deferred Objects

### Definitions

**Core Definition:** A Deferred object is a stateful, chainable utility that represents a unit of asynchronous work and allows callbacks to be registered for its success, failure, or progress.

**Technical Definition:** `jQuery.Deferred()` returns an object with methods for state management (`.resolve()`, `.reject()`, `.notify()`) and callback registration (`.done()`, `.fail()`, `.progress()`, `.always()`, `.then()`). It is the producer side of the Promise pattern. The `.promise()` method extracts the consumer-only view.

**Beginner-Friendly Explanation:** A Deferred is a promise you can control. You decide when it succeeds or fails, and others can listen.

### Purposes

- To create custom asynchronous operations with controllable state.
- To provide a standard interface for callback management.
- To decouple the code that produces an async result from the code that consumes it.
- To enable chaining and composition of asynchronous operations.

### Syntax Rules and Structure

```javascript
var deferred = $.Deferred();

// Register callbacks
deferred.done(function(value) { /* success */ });
deferred.fail(function(reason) { /* failure */ });
deferred.always(function() { /* always */ });

// Change state
deferred.resolve(value);   // or deferred.reject(reason);

// Get restricted promise
var promise = deferred.promise();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `.done(fn)` | Register success callback. |
| `.fail(fn)` | Register failure callback. |
| `.always(fn)` | Register callback for both outcomes. |
| `.resolve(value)` | Transition to resolved state. |
| `.reject(reason)` | Transition to rejected state. |
| `.promise()` | Return restricted Promise view. |

**Syntax Rules:**
1. A Deferred starts in `"pending"` state.
2. `.resolve()` or `.reject()` transitions permanently.
3. Callbacks attached after settlement fire immediately.
4. `.promise()` returns an object without state-changing methods.

**Constraints and Limitations:**
- Deferreds are mutable; the producer can change state.
- Pre-jQuery 3.0 not fully Promises/A+ compliant.
- `.notify()` / `.progress()` are not part of Promises/A+.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Deferred Usage**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Deferred Objects</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Create a Deferred
        var deferred = $.Deferred();

        // Step 2: Register callbacks
        deferred.done(function(value) {
            $("#output").append("Done: " + value + "<br>");
        });
        deferred.fail(function(reason) {
            $("#output").append("Failed: " + reason + "<br>");
        });
        deferred.always(function() {
            $("#output").append("Always runs<br>");
        });

        // Step 3: Resolve after 1 second
        setTimeout(function() {
            deferred.resolve("Operation complete");
            console.log("State:", deferred.state()); // "resolved"
        }, 1000);
    </script>
</body>
</html>
```

**Expected Output (after 1 second):**
```
Done: Operation complete
Always runs
```

**Why This Code Produces That Result:** The Deferred starts pending. When `.resolve()` is called, it transitions to `"resolved"`, firing all `.done()` and `.always()` callbacks with the resolved value.

### Real-World Cases
- **Custom Async Operations:** Wrapping WebSocket, timer, or animation completion.
- **Plugin APIs:** Returning Promises from plugin methods.
- **Workflow Coordination:** Managing multi-step async processes.

---

## 2. `$.Deferred()`

### Definitions

**Core Definition:** `$.Deferred()` is the constructor function that creates a new Deferred object.

**Technical Definition:** `jQuery.Deferred([beforeStart])` returns a new Deferred. The optional `beforeStart` callback receives the Deferred and can attach callbacks before the constructor returns.

**Beginner-Friendly Explanation:** `$.Deferred()` creates a new promise you can control.

### Purposes
- To create a new Deferred object.
- To optionally initialize callbacks before the Deferred is returned.

### Syntax Rules and Structure

```javascript
var deferred = $.Deferred();
var deferred = $.Deferred(function(d) {
    // Attach callbacks before returning
    d.done(function() { /* ... */ });
});
```

**Syntax Rules:**
1. Returns a new Deferred in `"pending"` state.
2. Optional `beforeStart` callback receives the Deferred.
3. `new` is optional.

**Constraints and Limitations:**
- Each call creates an independent Deferred.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Deferred with beforeStart**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.Deferred() Constructor</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Create Deferred with beforeStart
        var deferred = $.Deferred(function(d) {
            d.done(function(value) {
                $("#output").append("Pre-registered: " + value + "<br>");
            });
        });

        // Step 2: Add another callback
        deferred.done(function(value) {
            $("#output").append("Post-registered: " + value);
        });

        // Step 3: Resolve
        deferred.resolve("Hello");
    </script>
</body>
</html>
```

**Expected Output:**
```
Pre-registered: Hello
Post-registered: Hello
```

**Why This Code Produces That Result:** The `beforeStart` callback registers a handler before the constructor returns. Both handlers fire in registration order when resolved.

### Real-World Cases
- **Library Initialization:** Setting up default callbacks.
- **Testing:** Creating Deferreds with pre-registered assertions.

---

## 3. Deferred States

### Definitions

**Core Definition:** A Deferred state is one of three mutually exclusive conditions — pending, resolved, or rejected — that describes the current lifecycle stage of the Deferred.

**Technical Definition:** The state is queryable via `.state()`, which returns `"pending"`, `"resolved"`, or `"rejected"`. Transitions from pending to resolved/rejected are permanent; once settled, the Deferred cannot change state.

**Beginner-Friendly Explanation:** A Deferred has three states: waiting (pending), succeeded (resolved), or failed (rejected). Once it succeeds or fails, it stays that way.

### Purposes
- To track the lifecycle of an asynchronous operation.
- To branch logic based on whether the operation has completed.
- To prevent state changes after settlement.

---

## 4. Pending State

### Definitions

**Core Definition:** The pending state is the initial state of a Deferred, indicating that the asynchronous operation has not yet completed.

**Technical Definition:** A Deferred starts in the `"pending"` state. Callbacks registered during this state are queued and fire when the Deferred is resolved or rejected. `.state()` returns `"pending"`.

**Beginner-Friendly Explanation:** Pending means "still waiting."

### Purposes
- To indicate that the operation is in progress.
- To queue callbacks for later execution.

### Syntax Rules and Structure

```javascript
var deferred = $.Deferred();
console.log(deferred.state()); // "pending"
```

**Syntax Rules:**
1. Initial state for all Deferreds.
2. Callbacks are queued until settlement.

**Constraints and Limitations:**
- Cannot be returned to once settled.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Observing the Pending State**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Pending State</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();
        $("#output").append("Initial state: " + deferred.state() + "<br>");

        deferred.done(function() {
            $("#output").append("After resolve: " + deferred.state());
        });

        setTimeout(function() {
            deferred.resolve();
        }, 500);
    </script>
</body>
</html>
```

**Expected Output:**
```
Initial state: pending
After resolve: resolved
```

**Why This Code Produces That Result:** The Deferred starts pending. After `resolve()`, the state is `"resolved"`.

### Real-World Cases
- **Loading Indicators:** Showing a spinner while pending.
- **Queue Management:** Queuing operations until a resource is ready.

---

## 5. Resolved State

### Definitions

**Core Definition:** The resolved state indicates that the asynchronous operation completed successfully.

**Technical Definition:** A Deferred transitions to `"resolved"` when `.resolve()` or `.resolveWith()` is called. `.done()` callbacks fire with the resolved values. `.state()` returns `"resolved"`.

**Beginner-Friendly Explanation:** Resolved means "it worked."

### Purposes
- To signal successful completion.
- To trigger success callbacks.

### Syntax Rules and Structure

```javascript
deferred.resolve(value1, value2);
// .done(function(v1, v2) { ... })
```

**Syntax Rules:**
1. `.resolve()` accepts any number of values.
2. Values are passed to `.done()` callbacks.
3. State cannot be changed after resolution.

**Constraints and Limitations:**
- One-way transition; cannot be undone.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Resolving with Multiple Values**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Resolved State</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();
        deferred.done(function(a, b, c) {
            $("#output").text("Resolved with: " + a + ", " + b + ", " + c);
        });
        deferred.resolve("alpha", "beta", "gamma");
        console.log("State:", deferred.state()); // "resolved"
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Resolved with: alpha, beta, gamma".

**Why This Code Produces That Result:** `.resolve()` passes three values to the `.done()` callback in order.

### Real-World Cases
- **Data Loading:** Signaling that data has been fetched.
- **Animation Complete:** Signaling that an animation finished.

---

## 6. Rejected State

### Definitions

**Core Definition:** The rejected state indicates that the asynchronous operation failed.

**Technical Definition:** A Deferred transitions to `"rejected"` when `.reject()` or `.rejectWith()` is called. `.fail()` callbacks fire with the rejection reason. `.state()` returns `"rejected"`.

**Beginner-Friendly Explanation:** Rejected means "it failed."

### Purposes
- To signal failure.
- To trigger error callbacks.

### Syntax Rules and Structure

```javascript
deferred.reject(reason);
// .fail(function(reason) { ... })
```

**Syntax Rules:**
1. `.reject()` accepts any number of values.
2. Values are passed to `.fail()` callbacks.
3. State cannot be changed after rejection.

**Constraints and Limitations:**
- One-way transition; cannot be undone.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Rejecting with a Reason**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Rejected State</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();
        deferred.fail(function(error) {
            $("#output").text("Rejected: " + error.message);
        });
        deferred.reject({ message: "Network timeout", code: 504 });
        console.log("State:", deferred.state()); // "rejected"
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Rejected: Network timeout".

**Why This Code Produces That Result:** `.reject()` passes the object to the `.fail()` callback, which accesses `error.message`.

### Real-World Cases
- **Error Handling:** Signaling failed API calls.
- **Validation:** Rejecting invalid operations.

---

## 7. `.resolve()` / `.resolveWith()`

### Definitions

**Core Definition:** `.resolve()` transitions a Deferred to the resolved state, calling all `.done()` callbacks with the provided arguments. `.resolveWith()` does the same while explicitly setting the `this` context.

**Technical Definition:** `deferred.resolve([args])` resolves the Deferred with the given arguments. `deferred.resolveWith(context, [args])` resolves it with `this` set to `context` and the given arguments.

**Beginner-Friendly Explanation:** `.resolve()` says "it worked" and passes values to success handlers. `.resolveWith()` also sets what `this` means in those handlers.

### Purposes
- To signal successful completion.
- To pass result values to callbacks.
- To control the `this` context of callbacks.

### Syntax Rules and Structure

```javascript
deferred.resolve(value1, value2);
deferred.resolveWith(context, [value1, value2]);
```

**Syntax Rules:**
1. Values are passed as separate arguments.
2. `.resolveWith()` requires an array for arguments.
3. State becomes `"resolved"` permanently.

**Constraints and Limitations:**
- Can only be called once (subsequent calls ignored).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: resolve vs. resolveWith**

```html
<!DOCTYPE html>
<html>
<head>
    <title>resolve vs resolveWith</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: resolveWith sets `this`
        var context = { name: "MyContext" };
        var d1 = $.Deferred();
        d1.done(function(val) {
            $("#output").append("resolveWith this.name: " + this.name + ", val: " + val + "<br>");
        });
        d1.resolveWith(context, ["value1"]);

        // Step 2: resolve does not set `this` (defaults to promise)
        var d2 = $.Deferred();
        d2.done(function(val) {
            $("#output").append("resolve val: " + val + "<br>");
        });
        d2.resolve("value2");
    </script>
</body>
</html>
```

**Expected Output:**
```
resolveWith this.name: MyContext, val: value1
resolve val: value2
```

**Why This Code Produces That Result:** `.resolveWith(context, [...])` sets `this` to the context object inside the callback. `.resolve()` passes values without a custom context.

### Real-World Cases
- **Object-Oriented Deferreds:** Resolving with a class instance as context.
- **Plugin APIs:** Controlling callback context.

---

## 8. `.reject()` / `.rejectWith()`

### Definitions

**Core Definition:** `.reject()` transitions a Deferred to the rejected state, calling all `.fail()` callbacks with the provided arguments. `.rejectWith()` does the same while explicitly setting the `this` context.

**Technical Definition:** `deferred.reject([args])` rejects the Deferred with the given arguments. `deferred.rejectWith(context, [args])` rejects it with `this` set to `context` and the given arguments.

**Beginner-Friendly Explanation:** `.reject()` says "it failed" and passes error details to failure handlers. `.rejectWith()` also sets `this`.

### Purposes
- To signal failure.
- To pass error information to callbacks.
- To control the `this` context of failure callbacks.

### Syntax Rules and Structure

```javascript
deferred.reject(reason);
deferred.rejectWith(context, [reason]);
```

**Syntax Rules:**
1. Values are passed as separate arguments.
2. `.rejectWith()` requires an array.
3. State becomes `"rejected"` permanently.

**Constraints and Limitations:**
- Can only be called once.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: reject vs. rejectWith**

```html
<!DOCTYPE html>
<html>
<head>
    <title>reject vs rejectWith</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: rejectWith sets `this`
        var ctx = { source: "API" };
        var d1 = $.Deferred();
        d1.fail(function(err) {
            $("#output").append("rejectWith from: " + this.source + ", err: " + err + "<br>");
        });
        d1.rejectWith(ctx, ["Connection failed"]);

        // Step 2: reject without context
        var d2 = $.Deferred();
        d2.fail(function(err) {
            $("#output").append("reject err: " + err);
        });
        d2.reject("Timeout");
    </script>
</body>
</html>
```

**Expected Output:**
```
rejectWith from: API, err: Connection failed
reject err: Timeout
```

**Why This Code Produces That Result:** `.rejectWith(ctx, [...])` sets `this` to `ctx` inside the failure callback. `.reject()` passes the reason without a custom context.

### Real-World Cases
- **Service Errors:** Rejecting with service-specific context.
- **Validation Failures:** Rejecting with field information.

---

## 9. `.notify()` / `.notifyWith()`

### Definitions

**Core Definition:** `.notify()` sends progress notifications to `.progress()` callbacks without changing the Deferred's state. `.notifyWith()` does the same while setting the `this` context.

**Technical Definition:** `deferred.notify([args])` calls all `.progress()` callbacks with the given arguments. The Deferred remains in `"pending"` state. `.notifyWith(context, [args])` sets `this` to `context`. These methods are jQuery-specific and not part of Promises/A+.

**Beginner-Friendly Explanation:** `.notify()` sends a progress update — like "50% done" — without resolving or rejecting the Deferred.

### Purposes
- To report progress during a long-running operation.
- To update UI (progress bars) while the operation is pending.

### Syntax Rules and Structure

```javascript
deferred.notify(percentComplete);
deferred.notifyWith(context, [percentComplete]);
// .progress(function(percent) { ... })
```

**Syntax Rules:**
1. Does not change state.
2. Can be called multiple times while pending.
3. `.progress()` callbacks receive the notification values.

**Constraints and Limitations:**
- Not part of Promises/A+.
- `.progress()` is only available on jQuery Deferreds, not native Promises.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Progress Notifications**

```html
<!DOCTYPE html>
<html>
<head>
    <title>notify Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="progress"></div>
    <script>
        var deferred = $.Deferred();

        deferred.progress(function(percent) {
            $("#progress").text("Progress: " + percent + "%");
            console.log("Progress:", percent + "%");
        });

        var percent = 0;
        var interval = setInterval(function() {
            percent += 20;
            deferred.notify(percent); // Send progress
            if (percent >= 100) {
                clearInterval(interval);
                deferred.resolve("Done");
            }
        }, 500);

        deferred.done(function(msg) {
            $("#progress").append("<br>" + msg);
        });
    </script>
</body>
</html>
```

**Expected Output:** The progress div updates from 0% to 100% every 500ms, then displays "Done".

**Why This Code Produces That Result:** `.notify()` fires `.progress()` callbacks while the Deferred remains pending. When the operation completes, `.resolve()` fires `.done()`.

### Real-World Cases
- **File Uploads:** Reporting upload progress.
- **Batch Processing:** Reporting items processed.
- **Long-Running Tasks:** Showing completion percentage.

---

## 10. `.done()`

### Definitions

**Core Definition:** `.done()` registers a callback that fires when the Deferred is resolved.

**Technical Definition:** `deferred.done(callback)` adds the callback to the success callback list. Multiple `.done()` calls can be chained. Callbacks fire in registration order with the resolved values.

**Beginner-Friendly Explanation:** `.done()` says "when this succeeds, do this."

### Purposes
- To handle successful completion.
- To process resolved values.

### Syntax Rules and Structure

```javascript
deferred.done(function(value1, value2) { /* ... */ });
```

**Syntax Rules:**
1. Multiple `.done()` calls allowed.
2. Returns the Deferred for chaining.
3. Fires immediately if already resolved.

**Constraints and Limitations:**
- Only fires on resolution.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Multiple done Handlers**

```html
<!DOCTYPE html>
<html>
<head>
    <title>done Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();
        deferred.done(function(v) { $("#output").append("Handler 1: " + v + "<br>"); });
        deferred.done(function(v) { $("#output").append("Handler 2: " + v + "<br>"); });
        deferred.done(function(v) { $("#output").append("Handler 3: " + v); });
        deferred.resolve("Success");
    </script>
</body>
</html>
```

**Expected Output:**
```
Handler 1: Success
Handler 2: Success
Handler 3: Success
```

**Why This Code Produces That Result:** All three `.done()` handlers fire in registration order when the Deferred resolves.

### Real-World Cases
- **UI Updates:** Multiple handlers updating different parts of the page.
- **Logging:** Logging success alongside UI updates.

---

## 11. `.fail()`

### Definitions

**Core Definition:** `.fail()` registers a callback that fires when the Deferred is rejected.

**Technical Definition:** `deferred.fail(callback)` adds the callback to the failure callback list. Callbacks fire with the rejection reason(s).

**Beginner-Friendly Explanation:** `.fail()` says "if this fails, do this."

### Purposes
- To handle failures.
- To display error messages or log errors.

### Syntax Rules and Structure

```javascript
deferred.fail(function(reason) { /* ... */ });
```

**Syntax Rules:**
1. Multiple `.fail()` calls allowed.
2. Fires immediately if already rejected.

**Constraints and Limitations:**
- Only fires on rejection.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Multiple fail Handlers**

```html
<!DOCTYPE html>
<html>
<head>
    <title>fail Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();
        deferred.fail(function(r) { $("#output").append("Fail 1: " + r + "<br>"); });
        deferred.fail(function(r) { $("#output").append("Fail 2: " + r); });
        deferred.reject("Error occurred");
    </script>
</body>
</html>
```

**Expected Output:**
```
Fail 1: Error occurred
Fail 2: Error occurred
```

**Why This Code Produces That Result:** Both `.fail()` handlers fire when the Deferred is rejected.

### Real-World Cases
- **Error Logging:** Logging failures alongside UI errors.
- **Fallback Logic:** Trying alternative operations on failure.

---

## 12. `.progress()`

### Definitions

**Core Definition:** `.progress()` registers a callback that fires when the Deferred receives a notification via `.notify()`.

**Technical Definition:** `deferred.progress(callback)` adds the callback to the progress callback list. Progress callbacks fire only while the Deferred is pending.

**Beginner-Friendly Explanation:** `.progress()` says "tell me how it's going while I wait."

### Purposes
- To receive progress updates during long operations.
- To update progress bars and status messages.

### Syntax Rules and Structure

```javascript
deferred.progress(function(percent) { /* ... */ });
```

**Syntax Rules:**
1. Fires only while pending.
2. Multiple `.progress()` calls allowed.
3. Not part of Promises/A+.

**Constraints and Limitations:**
- Does not fire after resolution/rejection.
- Not available on native Promises.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Progress Updates**

```html
<!DOCTYPE html>
<html>
<head>
    <title>progress Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="bar"></div>
    <script>
        var deferred = $.Deferred();
        deferred.progress(function(p) {
            $("#bar").css("width", p + "%").text(p + "%");
        });
        deferred.done(function() {
            $("#bar").text("Complete!");
        });

        var p = 0;
        var timer = setInterval(function() {
            p += 10;
            deferred.notify(p);
            if (p >= 100) { clearInterval(timer); deferred.resolve(); }
        }, 200);
    </script>
</body>
</html>
```

**Expected Output:** The progress bar grows from 10% to 100%, then displays "Complete!".

**Why This Code Produces That Result:** `.notify()` fires `.progress()` callbacks while pending, updating the bar. `.resolve()` fires `.done()` at the end.

### Real-World Cases
- **File Uploads:** Progress bars.
- **Data Processing:** Reporting items processed.

---

## 13. `.always()`

### Definitions

**Core Definition:** `.always()` registers a callback that fires regardless of whether the Deferred is resolved or rejected.

**Technical Definition:** `deferred.always(callback)` adds the callback to both the success and failure lists. It fires with the same arguments as `.done()` or `.fail()`, depending on the outcome.

**Beginner-Friendly Explanation:** `.always()` says "no matter what happens, do this."

### Purposes
- To perform cleanup regardless of outcome.
- To hide loading indicators.
- To log completion.

### Syntax Rules and Structure

```javascript
deferred.always(function() { /* ... */ });
```

**Syntax Rules:**
1. Fires for both success and failure.
2. Multiple `.always()` calls allowed.
3. Arguments vary by outcome.

**Constraints and Limitations:**
- Does not receive consistent arguments.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Cleanup with always**

```html
<!DOCTYPE html>
<html>
<head>
    <title>always Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="loading">Loading...</div>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();
        deferred.always(function() {
            $("#loading").hide();
            $("#output").append("Cleanup done<br>");
        });
        deferred.done(function(v) {
            $("#output").append("Success: " + v);
        });
        setTimeout(function() { deferred.resolve("Data loaded"); }, 1000);
    </script>
</body>
</html>
```

**Expected Output (after 1 second):** "Cleanup done" and "Success: Data loaded" appear; the loading indicator hides.

**Why This Code Produces That Result:** `.always()` fires after resolution, hiding the loading indicator. Order: `.done()` fires first, then `.always()`.

### Real-World Cases
- **Loading Indicators:** Hiding spinners.
- **Resource Cleanup:** Closing connections.

---

## 14. `.then()`

### Definitions

**Core Definition:** `.then()` registers handlers for both success and failure and returns a new Promise, enabling chaining and transformation.

**Technical Definition:** `deferred.then(onFulfilled, onRejected, onProgress)` returns a new Promise. If the handler returns a value, the new Promise resolves with it. If it returns a Promise, the new Promise adopts its state. If it throws, the new Promise rejects. Since jQuery 3.0, `.then()` is Promises/A+ compliant.

**Beginner-Friendly Explanation:** `.then()` is the modern, chainable version of `.done()` and `.fail()`. It transforms the result and passes it to the next step.

### Purposes
- To transform resolved values.
- To chain asynchronous operations.
- To handle both success and failure in one method.
- To recover from errors.

### Syntax Rules and Structure

```javascript
deferred.then(
    function(value) { return transformedValue; },
    function(reason) { /* handle error */ }
).then(function(value) { /* next step */ });
```

**Syntax Rules:**
1. Returns a new Promise.
2. Returning a value resolves the new Promise with it.
3. Returning a Promise adopts its state.
4. Throwing rejects the new Promise.

**Constraints and Limitations:**
- Pre-jQuery 3.0 not fully Promises/A+ compliant.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Chaining with then**

```html
<!DOCTYPE html>
<html>
<head>
    <title>then Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var deferred = $.Deferred();

        deferred
            .then(function(value) {
                $("#output").append("Step 1: " + value + "<br>");
                return value * 2;
            })
            .then(function(value) {
                $("#output").append("Step 2 (doubled): " + value + "<br>");
                return value + 10;
            })
            .then(function(value) {
                $("#output").append("Step 3 (plus 10): " + value);
            });

        deferred.resolve(5);
    </script>
</body>
</html>
```

**Expected Output:**
```
Step 1: 5
Step 2 (doubled): 10
Step 3 (plus 10): 20
```

**Why This Code Produces That Result:** Each `.then()` transforms the value and passes it to the next. Step 1 returns 10, Step 2 returns 20, Step 3 receives 20.

### Real-World Cases
- **Data Pipelines:** Transforming API responses.
- **Dependent Requests:** Chaining AJAX calls.
- **Error Recovery:** Returning fallback values.

---

## 15. `.pipe()`

### Definitions

**Core Definition:** `.pipe()` is a deprecated method for filtering and chaining Deferreds. **Deprecated since jQuery 1.8** — use `.then()` instead.

**Technical Definition:** `deferred.pipe(onFulfilled, onRejected, onProgress)` was the pre-1.8 way to chain Deferreds. It has been superseded by `.then()`, which is Promises/A+ compliant.

**Beginner-Friendly Explanation:** `.pipe()` was the old way to chain promises. Use `.then()` instead.

### Purposes
- (Historical) To chain Deferreds before `.then()` was compliant.

### Syntax Rules and Structure

```javascript
// Deprecated
deferred.pipe(fn).pipe(fn2);
```

**Syntax Rules:**
1. Deprecated since jQuery 1.8.
2. Use `.then()` instead.

**Constraints and Limitations:**
- Deprecated; may be removed in future versions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: pipe vs then**

```html
<!DOCTYPE html>
<html>
<head>
    <title>pipe Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var d = $.Deferred();
        // Deprecated pipe (still works)
        d.pipe(function(v) { return v * 2; })
         .done(function(v) { $("#output").text("Pipe result: " + v); });
        d.resolve(5);

        // Preferred: then
        var d2 = $.Deferred();
        d2.then(function(v) { return v * 3; })
          .done(function(v) { $("#output").append("<br>Then result: " + v); });
        d2.resolve(5);
    </script>
</body>
</html>
```

**Expected Output:**
```
Pipe result: 10
Then result: 15
```

**Why This Code Produces That Result:** Both `.pipe()` and `.then()` transform the resolved value. `.then()` is preferred because it is Promises/A+ compliant.

### Real-World Cases
- **Legacy Code:** Migrating from `.pipe()` to `.then()`.

---

## 16. `.promise()`

### Definitions

**Core Definition:** `.promise()` returns a restricted Promise view of a Deferred, exposing only callback-registration methods and hiding state-changing methods.

**Technical Definition:** `deferred.promise([target])` returns a Promise object with `.done()`, `.fail()`, `.always()`, `.then()`, `.progress()`, `.state()`, and `.pipe()` but without `.resolve()`, `.reject()`, `.notify()`, or their `With` variants. If `target` is provided, the Promise methods are added to the target object.

**Beginner-Friendly Explanation:** `.promise()` gives others a "read-only" view of your Deferred — they can listen but not change the state.

### Purposes
- To share a Deferred with consumers without allowing them to change its state.
- To add Promise methods to existing objects.

### Syntax Rules and Structure

```javascript
var promise = deferred.promise();
promise.done(fn);  // works
// promise.resolve(); // undefined — not exposed

// Attach to an object
var obj = {};
deferred.promise(obj);
obj.done(fn);
```

**Syntax Rules:**
1. Returns a restricted view.
2. Does not expose state-changing methods.
3. Optional `target` receives the Promise methods.

**Constraints and Limitations:**
- The restricted view is not a full Deferred.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Exposing a Promise**

```html
<!DOCTYPE html>
<html>
<head>
    <title>promise Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Create a Deferred
        var deferred = $.Deferred();

        // Step 2: Expose only the Promise
        var promise = deferred.promise();
        promise.done(function(v) {
            $("#output").text("Promise resolved: " + v);
        });

        // Step 3: Resolve via the Deferred (not the Promise)
        deferred.resolve("Data ready");

        // promise.resolve is undefined
        console.log("promise.resolve:", typeof promise.resolve); // "undefined"
    </script>
</body>
</html>
```

**Expected Output:** The output div displays "Promise resolved: Data ready". Console logs `promise.resolve: undefined`.

**Why This Code Produces That Result:** `.promise()` returns a restricted view without state-changing methods. The Deferred resolves it, firing the Promise's `.done()` handler.

### Real-World Cases
- **API Design:** Returning Promises from functions while keeping resolution private.
- **Plugin APIs:** Exposing read-only Promise interfaces.

---

## 17. `.state()`

### Definitions

**Core Definition:** `.state()` returns the current state of the Deferred as a string.

**Technical Definition:** `deferred.state()` returns `"pending"`, `"resolved"`, or `"rejected"`.

**Beginner-Friendly Explanation:** `.state()` tells you if the Deferred is still waiting, succeeded, or failed.

### Purposes
- To query the current state.
- To branch logic based on state.

### Syntax Rules and Structure

```javascript
deferred.state(); // "pending", "resolved", or "rejected"
```

**Syntax Rules:**
1. Returns a string.
2. Useful for debugging.

**Constraints and Limitations:**
- Read-only.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Querying State**

```html
<!DOCTYPE html>
<html>
<head>
    <title>state Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        var d = $.Deferred();
        $("#output").append("1: " + d.state() + "<br>"); // pending
        d.resolve();
        $("#output").append("2: " + d.state()); // resolved
    </script>
</body>
</html>
```

**Expected Output:**
```
1: pending
2: resolved
```

**Why This Code Produces That Result:** `.state()` returns the current state, which changes from pending to resolved after `.resolve()`.

### Real-World Cases
- **Debugging:** Checking state during troubleshooting.
- **Conditional Logic:** Branching based on state.

---

## 18. Promise Objects

### Definitions

**Core Definition:** A Promise object is a restricted view of a Deferred that exposes only callback-registration methods, providing a read-only interface for consumers.

**Technical Definition:** Created via `deferred.promise()`, a Promise exposes `.done()`, `.fail()`, `.always()`, `.then()`, `.progress()`, `.state()`, and `.pipe()`. It does not expose `.resolve()`, `.reject()`, or `.notify()`. The `jqXHR` object returned by `$.ajax()` is a Promise.

**Beginner-Friendly Explanation:** A Promise is a "read-only" version of a Deferred. You can listen to it but not change its state.

### Purposes
- To share asynchronous results without exposing control.
- To provide a standard interface for consumers.

### Syntax Rules and Structure

```javascript
var promise = deferred.promise();
promise.done(fn).fail(fn).always(fn);
```

**Syntax Rules:**
1. Only callback-registration methods.
2. State-changing methods are hidden.
3. Can be shared safely.

**Constraints and Limitations:**
- Cannot resolve or reject.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Sharing a Promise**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Promise Objects</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        function fetchData() {
            var deferred = $.Deferred();
            // Simulate async work
            setTimeout(function() {
                deferred.resolve({ name: "Result" });
            }, 500);
            // Return only the Promise
            return deferred.promise();
        }

        var promise = fetchData();
        promise.done(function(data) {
            $("#output").text("Received: " + data.name);
        });

        // promise.resolve() is undefined
        console.log("Can resolve?", typeof promise.resolve); // "undefined"
    </script>
</body>
</html>
```

**Expected Output (after 500ms):** The output div displays "Received: Result".

**Why This Code Produces That Result:** `fetchData()` returns only the Promise view, so the caller cannot resolve it. The Deferred resolves it internally, firing the Promise's `.done()` handler.

### Real-World Cases
- **API Functions:** Returning Promises while keeping control internal.
- **Plugin APIs:** Exposing read-only async interfaces.

---

## 19. Deferred Chaining

### Definitions

**Core Definition:** Deferred chaining is the practice of connecting multiple asynchronous operations in sequence using `.then()`, where each operation starts after the previous one completes.

**Technical Definition:** Each `.then()` call returns a new Promise. If the handler returns a value, the next Promise resolves with it. If it returns a Promise, the next Promise adopts its state. This enables sequential execution without callback nesting.

**Beginner-Friendly Explanation:** Chaining lets you say "do this, then when it's done, do that, then the next thing."

### Purposes
- To execute asynchronous operations in sequence.
- To transform values between steps.
- To avoid callback hell.

### Syntax Rules and Structure

```javascript
deferred
    .then(function(v) { return nextDeferred; })
    .then(function(v) { return anotherDeferred; })
    .then(function(final) { /* ... */ });
```

**Syntax Rules:**
1. Each `.then()` returns a new Promise.
2. Returning a Promise adopts its state.
3. Errors propagate until caught.

**Constraints and Limitations:**
- Each `.then()` adds minor overhead.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Sequential AJAX Chain**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Deferred Chaining</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // Step 1: Get a post
        $.get("https://jsonplaceholder.typicode.com/posts/1")
            .then(function(post) {
                $("#output").append("Post: " + post.title + "<br>");
                // Step 2: Use userId to get the user
                return $.get("https://jsonplaceholder.typicode.com/users/" + post.userId);
            })
            .then(function(user) {
                $("#output").append("User: " + user.name + "<br>");
                return user.email;
            })
            .then(function(email) {
                $("#output").append("Email: " + email);
            })
            .fail(function(jqXHR) {
                $("#output").append("Error: " + jqXHR.status);
            });
    </script>
</body>
</html>
```

**Expected Output:**
```
Post: sunt aut facere...
User: Leanne Graham
Email: Sincere@april.biz
```

**Why This Code Produces That Result:** Each `.then()` returns a new Promise (either an AJAX request or a value). The chain executes sequentially, and the final `.then()` receives the email. The `.fail()` catches any error.

### Real-World Cases
- **Multi-Step Workflows:** Validate → Submit → Redirect.
- **Data Pipelines:** Fetch → Transform → Render.
- **Dependent API Calls:** Fetch related resources in order.

---

## 20. Deferred Composition

### Definitions

**Core Definition:** Deferred composition is the practice of combining multiple Deferreds into a single Promise using `$.when()`, enabling coordination of parallel asynchronous operations.

**Technical Definition:** `$.when(deferred1, deferred2, ...)` returns a Promise that resolves when all input Deferreds resolve, with resolved values passed as separate arguments. If any rejects, the combined Promise rejects immediately.

**Beginner-Friendly Explanation:** Composition lets you wait for multiple things to finish before proceeding.

### Purposes
- To coordinate parallel asynchronous operations.
- To aggregate results from multiple sources.
- To implement parallel loading with a single completion handler.

### Syntax Rules and Structure

```javascript
$.when(d1, d2, d3)
    .done(function(r1, r2, r3) { /* ... */ })
    .fail(function(reason) { /* ... */ });
```

**Syntax Rules:**
1. Accepts any number of Deferreds.
2. Resolved values passed as separate arguments.
3. Rejects if any input rejects.

**Constraints and Limitations:**
- Rejects as soon as one fails.
- Use `.apply()` for dynamic arrays.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Parallel Requests with $.when()**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Deferred Composition</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        $.when(
            $.get("https://jsonplaceholder.typicode.com/posts/1"),
            $.get("https://jsonplaceholder.typicode.com/posts/2"),
            $.get("https://jsonplaceholder.typicode.com/posts/3")
        ).done(function(p1, p2, p3) {
            $("#output").append(
                "1: " + p1[0].title + "<br>" +
                "2: " + p2[0].title + "<br>" +
                "3: " + p3[0].title
            );
        }).fail(function(jqXHR) {
            $("#output").text("One request failed: " + jqXHR.status);
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
1: sunt aut facere...
2: qui est esse
3: ea molestias quasi...
```

**Why This Code Produces That Result:** All three requests run in parallel. When all succeed, `.done()` fires with all three results.

### Real-World Cases
- **Dashboard Loading:** Multiple widgets loading simultaneously.
- **Batch Operations:** Submitting multiple forms.
- **Parallel Data Fetching:** Loading related resources.

---

## 21. Deferred Best Practices

### Definitions

**Core Definition:** Deferred Best Practices are empirically established guidelines for creating, using, and composing Deferred objects to ensure clean, maintainable, and Promises/A+-compliant asynchronous code.

**Technical Definition:** Best practices encompass returning Promises (not Deferreds) from functions, using `.then()` over `.done()`/`.fail()` for transformation and chaining, using `.catch()` for error handling, avoiding the deferred anti-pattern, cleaning up with `.always()` or `.finally()`, being aware of jQuery version differences, and composing with `$.when()`.

### Purposes

- To write clean, maintainable asynchronous code.
- To ensure compatibility with native Promises.
- To avoid memory leaks and race conditions.
- To make code testable.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Return Promises from functions | `return deferred;` | `return deferred.promise();` |
| 2 | Use `.then()` for transformation | `.done(fn)` | `.then(fn)` |
| 3 | Use `.catch()` for errors | `.fail(fn)` | `.catch(fn)` |
| 4 | Avoid the deferred anti-pattern | Wrap existing Promise | Return existing Promise |
| 5 | Single error handler at end | `.fail()` on each step | `.catch()` at chain end |
| 6 | Use `$.when()` for composition | Manual counting | `$.when(p1, p2)` |
| 7 | Clean up with `.always()` | Cleanup in both done/fail | `.always(cleanup)` |
| 8 | Be version-aware | Assume pre-3.0 behavior | Test with jQuery 3.x |
| 9 | Avoid `.pipe()` | `.pipe(fn)` | `.then(fn)` |
| 10 | Use `.finally()` for cleanup | `.always()` with no return | `.finally(cleanup)` |

**Syntax Rules:**
1. Always return `deferred.promise()`, not the Deferred itself.
2. Use `.then()` for transformation and chaining.
3. Place a single `.catch()` at the end of a chain.
4. Avoid wrapping existing Promises in new Deferreds.
5. Use `$.when()` for parallel coordination.
6. Prefer `.then()` over deprecated `.pipe()`.

**Constraints and Limitations:**
- jQuery Deferreds are not identical to native Promises (pre-3.0).
- `.progress()` and `.notify()` are jQuery-specific.
- Mixing jQuery and native Promises requires care.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Applying Deferred Best Practices**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Deferred Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>
    <script>
        // ✅ BEST PRACTICE 1: Return a Promise, not a Deferred
        function fetchPost(id) {
            var deferred = $.Deferred();
            $.get("https://jsonplaceholder.typicode.com/posts/" + id)
                .done(function(data) { deferred.resolve(data); })
                .fail(function(err) { deferred.reject(err); });
            return deferred.promise();  // ✅ Restricted view
        }

        // ✅ BEST PRACTICE 2: Use .then() for transformation
        fetchPost(1)
            .then(function(post) {
                $("#output").append("Post: " + post.title + "<br>");
                return post.userId;
            })
            .then(function(userId) {
                // ✅ BEST PRACTICE 3: Return a new Promise
                return $.get("https://jsonplaceholder.typicode.com/users/" + userId);
            })
            .then(function(user) {
                $("#output").append("User: " + user.name);
            })
            // ✅ BEST PRACTICE 4: Single catch at the end
            .catch(function(err) {
                $("#output").append("Error: " + (err.status || err));
            })
            // ✅ BEST PRACTICE 5: finally for cleanup
            .finally(function() {
                console.log("Chain complete");
            });

        // ✅ BEST PRACTICE 6: Composition with $.when()
        $.when(fetchPost(2), fetchPost(3))
            .done(function(p2, p3) {
                $("#output").append("<br>Parallel: " + p2.title + " & " + p3.title);
            });
    </script>
</body>
</html>
```

**Expected Output:**
```
Post: sunt aut facere...
User: Leanne Graham
Parallel: qui est esse & ea molestias quasi...
```
Console logs "Chain complete".

**Why This Code Produces That Result:** The code applies multiple best practices: returning Promises, using `.then()` for transformation, `.catch()` for centralized error handling, `.finally()` for cleanup, and `$.when()` for composition. The result is clean, maintainable, Promises/A+-compatible asynchronous code.

### Real-World Cases
- **API Client Libraries:** Returning Promises from wrapper functions.
- **SPA Data Layers:** Chaining and composing data fetches.
- **Testing:** Returning Promises makes functions easy to test.

---

## References

- jQuery API Documentation — Deferred Object - https://api.jquery.com/category/deferred-object/
- jQuery API Documentation — jQuery.Deferred() - https://api.jquery.com/jQuery.Deferred/
- jQuery API Documentation — deferred.resolve() - https://api.jquery.com/deferred.resolve/
- jQuery API Documentation — deferred.reject() - https://api.jquery.com/deferred.reject/
- jQuery API Documentation — deferred.notify() - https://api.jquery.com/deferred.notify/
- jQuery API Documentation — deferred.resolveWith() - https://api.jquery.com/deferred.resolveWith/
- jQuery API Documentation — deferred.rejectWith() - https://api.jquery.com/deferred.rejectWith/
- jQuery API Documentation — deferred.notifyWith() - https://api.jquery.com/deferred.notifyWith/
- jQuery API Documentation — deferred.done() - https://api.jquery.com/deferred.done/
- jQuery API Documentation — deferred.fail() - https://api.jquery.com/deferred.fail/
- jQuery API Documentation — deferred.progress() - https://api.jquery.com/deferred.progress/
- jQuery API Documentation — deferred.always() - https://api.jquery.com/deferred.always/
- jQuery API Documentation — deferred.then() - https://api.jquery.com/deferred.then/
- jQuery API Documentation — deferred.pipe() - https://api.jquery.com/deferred.pipe/
- jQuery API Documentation — deferred.promise() - https://api.jquery.com/deferred.promise/
- jQuery API Documentation — deferred.state() - https://api.jquery.com/deferred.state/
- jQuery API Documentation — jQuery.when() - https://api.jquery.com/jQuery.when/
- jQuery Learning Center — jQuery Deferreds - https://learn.jquery.com/code-organization/deferreds/jquery-deferreds/
- jQuery 3.0 Upgrade Guide - https://jquery.com/upgrade-guide/3.0/
- Promises/A+ Specification - https://promisesaplus.com/
- MDN Web Docs — Promise - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- jQuery Blog — jQuery 3.0 Final Released - https://blog.jquery.com/2016/06/09/jquery-3-0-final-released/