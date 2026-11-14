# `useId` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useId` is a React Hook for generating unique IDs that can be passed to accessibility attributes, ensuring each component instance has its own distinct identifier.

**Technical Definition:** `useId` is a React Hook that returns a unique ID string associated with a particular `useId` call within a particular component instance. The ID is stable across re-renders of the same component instance and is designed to be consistent between server and client renders, making it safe for use in server-side rendering (SSR) and hydration. It is primarily intended for generating IDs for HTML accessibility attributes such as `id`, `htmlFor`, `aria-describedby`, `aria-labelledby`, and `aria-controls`. `useId` does not take any parameters and must be called at the top level of a component or custom Hook.

**Beginner-Friendly Explanation:** `useId` is a React tool that gives you a unique string, like a special serial number, for each component that uses it. You use this string to connect HTML elements to each other for accessibility. For example, you can connect a `<label>` to its `<input>` so that screen readers and users know they belong together. The best part is that it works perfectly with server-side rendering: the ID generated on the server matches the one on the client, so React does not get confused during hydration.

### Key Characteristics

- **Unique per instance:** Every component instance that calls `useId` receives its own distinct ID string.
- **Stable across renders:** The ID remains the same for a component instance across all its re-renders, preventing accessibility relationships from breaking.
- **SSR-safe:** React generates matching IDs on the server and the client during hydration, eliminating hydration mismatches that occur with random or counter-based IDs.
- **Colon-containing:** The generated string includes a colon (`:`), which makes it unique but means it is not a valid CSS selector without a prefix.
- **Accessibility-focused:** Designed specifically for DOM relationships, not for React keys or cache keys.
- **React 18+:** The Hook was introduced in React 18 to solve the problem of generating stable, unique IDs for SSR.

### Prerequisites

- React function components and Hooks.
- Basic HTML and form accessibility (labels, `htmlFor`, `id`, ARIA attributes).
- Understanding of server-side rendering (SSR) and hydration.
- Familiarity with React 18+ features.

### Related Programming Areas

- **Accessibility (a11y)** — ARIA attributes and form label associations.
- **Server-Side Rendering (SSR)** — hydration consistency.
- **Form Components** — reusable form fields with unique IDs.
- **React Concurrent Rendering** — ID stability across interruptible renders.

---

## 1. `useId`

### Definitions

**Core Definition:** `useId` is a React Hook that generates a unique ID string for use in accessibility attributes.

**Technical Definition:** `const id = useId()` is called at the top level of a component or custom Hook. It takes no parameters and returns a string that is unique to that specific `useId` call within that specific component instance. The ID is generated during rendering but is guaranteed to be stable for the lifetime of the component instance and consistent between server and client renders. React maintains an internal counter or seed to ensure uniqueness, and the colon format (e.g., `:r0:`) prevents collisions with user-provided IDs.

**Beginner-Friendly Explanation:** `useId` is a function you call at the top of your component. It gives you back a unique string that you can use as an ID for HTML elements. You do not give it any arguments. It just works.

### Purposes

- To generate a unique ID for each component instance, avoiding duplicate IDs when a component is reused.
- To create stable IDs that do not change across re-renders.
- To ensure IDs match between server and client during SSR hydration.
- To connect HTML elements via `id`, `htmlFor`, `aria-describedby`, `aria-labelledby`, and `aria-controls` attributes for accessibility.
- To replace error-prone manual ID generation (random strings, global counters).

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useId } from 'react';

const id = useId();
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `useId()` | Hook call; takes no parameters. |
| `id` | A unique string generated for this component instance. |

**Syntax Rules:**

1. Call `useId` at the top level of a component or custom Hook; never inside loops or conditions.
2. `useId` takes no parameters.
3. The returned ID is stable across re-renders of the same component instance.
4. Pass the ID to accessibility attributes: `id`, `htmlFor`, `aria-describedby`, `aria-labelledby`, `aria-controls`.
5. To generate multiple related IDs, call `useId` once and append suffixes: `` `${id}-firstName` ``, `` `${id}-lastName` ``.

**Constraints and Limitations:**

- `useId` cannot be used in async Server Components (as of React 19).
- The generated ID contains a colon (`:`), so it is not a valid CSS selector without a prefix.
- `useId` should not be used for list keys or `use()` cache keys.
- With SSR, the component tree must be identical on server and client for IDs to match.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessible Password Field

```jsx
import { useId } from 'react';

function PasswordField() {
  // Step 1: Generate a unique ID
  const passwordHintId = useId();

  return (
    <>
      {/* Step 2: Connect input to description via aria-describedby */}
      <label>
        Password:
        <input type="password" aria-describedby={passwordHintId} />
      </label>
      {/* Step 3: Give the description element the same ID */}
      <p id={passwordHintId}>
        The password should contain at least 18 characters.
      </p>
    </>
  );
}
```

**Expected Output:** The password input is programmatically associated with its description paragraph. Screen readers announce the hint when the input is focused. If `PasswordField` is rendered multiple times, each instance has its own unique IDs.

**Why this output:** `useId()` generates a unique ID for each `PasswordField` instance. The same ID is used in `aria-describedby` on the input and `id` on the paragraph, creating the accessibility relationship without hardcoding or duplication.

#### Example 2: Multiple Related IDs with Suffixes

```jsx
import { useId } from 'react';

function LoginForm() {
  const id = useId();

  return (
    <form>
      <label htmlFor={id + '-username'}>Username:</label>
      <input id={id + '-username'} type="text" />

      <label htmlFor={id + '-password'}>Password:</label>
      <input id={id + '-password'} type="password" />
    </form>
  );
}
```

**Expected Output:** Two form fields with unique IDs (`:r0:-username` and `:r0:-password`), correctly associated with their labels.

**Why this output:** A single `useId()` call generates one base ID. Suffixes create distinct IDs for multiple related elements, avoiding multiple Hook calls and ensuring all IDs are unique within the component.

### Real-World Cases

- **Reusable form components:** Text fields, checkboxes, radio groups, and selects that need unique IDs for labels and ARIA relationships.
- **Design systems:** Component libraries where the same component is rendered many times on a page.
- **SSR frameworks (Next.js, Remix):** Generating IDs that survive hydration without mismatches.
- **Modal and dialog components:** Connecting `aria-labelledby` and `aria-describedby` to the correct elements.

### References

- React — useId (Official) — https://react.dev/reference/react/useId
- React — useId (React 18) — https://18.react.dev/reference/react/useId

---

## 2. Unique IDs

### Definitions

**Core Definition:** A unique ID is a string that identifies an element uniquely within a document, ensuring that HTML `id` attributes do not collide.

**Technical Definition:** In HTML, `id` attributes must be unique within the entire document. When React components are reused, hardcoded IDs lead to duplicates, which break accessibility associations (labels, ARIA relationships) and can cause unpredictable browser behaviour. `useId` generates IDs that are unique per component instance, even when the same component is rendered multiple times on the same page.

**Beginner-Friendly Explanation:** A unique ID is a name that belongs to only one element on the page. If two elements have the same ID, things break. `useId` makes sure every component gets its own unique name.

### Purposes

- To prevent duplicate IDs when components are reused.
- To ensure accessibility relationships (label-input, ARIA) remain correct.
- To avoid browser quirks caused by duplicate IDs.
- To allow components to be rendered multiple times without ID collisions.

### Syntax Rules and Structure

```jsx
const id = useId();
```

**Syntax Rules:**

1. Use `useId` instead of hardcoded strings for IDs in reusable components.
2. Use `useId` instead of random strings (`Math.random()`) or global counters, which break SSR.
3. Each `useId` call generates one unique base ID; use suffixes for multiple related IDs.

### Annotated Code Examples

```jsx
function TextField({ label, ...props }) {
  const id = useId();

  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input id={id} {...props} />
    </div>
  );
}

function RegistrationForm() {
  return (
    <form>
      <TextField label="Username" />
      <TextField label="Email" type="email" />
      <TextField label="Password" type="password" />
    </form>
  );
}
```

**Expected Output:** Each `TextField` instance receives a unique ID. The labels are correctly associated with their respective inputs. Clicking a label focuses the correct input.

**Why this output:** `useId` generates a different ID for each `TextField` instance. The `htmlFor` and `id` attributes use the same ID within each instance, creating correct label-input associations without collisions.

### Real-World Cases

- **Form component libraries:** Every field component needs a unique ID for its label.
- **Dynamic forms:** Rendering the same field multiple times based on user input.
- **Lists of form fields:** Each item in a list needs its own ID.

### References

- Epic React — Improving React Accessibility with useId — https://www.epicreact.dev/improving-react-accessibility-with-use-id-knljs
- web-accessibility-guide — React/Next.js accessibility — https://github.com/arzu-guney/web-accessibility-guide

---

## 3. Server-Side Rendering (SSR)

### Definitions

**Core Definition:** Server-side rendering (SSR) is the process of rendering React components to HTML on the server, which is then sent to the browser and hydrated into an interactive application.

**Technical Definition:** In SSR, React renders the component tree to HTML on the server. The browser receives the HTML and displays it immediately, then React hydrates the HTML on the client, attaching event handlers and making it interactive. For hydration to succeed, the server-rendered HTML must match what the client would render. `useId` is designed for this: it generates the same ID string on the server and the client, ensuring that accessibility attributes and DOM relationships remain consistent. Random IDs or global counters fail in SSR because the server and client generate different values.

**Beginner-Friendly Explanation:** SSR means the server builds the HTML first, so users see content faster. Later, the browser makes it interactive ("hydration"). `useId` makes sure the IDs used on the server are the same as the ones the browser expects, so nothing breaks.

### Purposes

- To generate IDs that are consistent between server and client renders.
- To prevent hydration mismatches caused by unstable IDs.
- To ensure accessibility relationships survive hydration.
- To support streaming SSR and concurrent rendering without ID conflicts.

### Syntax Rules and Structure

```jsx
const id = useId(); // Same ID on server and client
```

**Syntax Rules:**

1. `useId` is SSR-safe; it generates matching IDs on both server and client.
2. The component tree must be identical on server and client for IDs to match.
3. Do not use random IDs or global counters in SSR; they cause hydration mismatches.
4. In React 18+, `useId` works with streaming SSR and concurrent rendering.

**Constraints and Limitations:**

- If the component tree differs between server and client (e.g., conditional rendering), `useId` values may mismatch.
- `useId` cannot be used in async Server Components.
- When rendering multiple independent React apps on one page, use `identifierPrefix` to prevent ID clashes.

### Annotated Code Examples

```jsx
// Server and client both call useId and get the same ID
function PasswordField() {
  const passwordHintId = useId();

  return (
    <>
      <label>
        Password:
        <input type="password" aria-describedby={passwordHintId} />
      </label>
      <p id={passwordHintId}>Password hint</p>
    </>
  );
}
```

**Expected Output:** The server-rendered HTML and the client-rendered HTML use the same ID for `aria-describedby` and `id`. Hydration succeeds without warnings.

**Why this output:** `useId` generates a deterministic ID based on the component's position in the tree. Since the tree is the same on server and client, the IDs match.

### Real-World Cases

- **Next.js applications:** Pages rendered on the server with form components using `useId`.
- **Remix applications:** SSR with accessible form fields.
- **Any SSR framework:** Stable IDs for hydration.

### References

- React — useId: Caveats (SSR requirement) — https://react.dev/reference/react/useId
- DEV Community — The Definitive React 19 useId Guide — https://dev.to/a1guy/the-definitive-react-19-useid-guide-patterns-pitfalls-and-pro-tips-40ia
- React — useId (Mintlify, SSR stable) — https://mintlify.wiki/facebook/react/api/hooks/use-id

---

## 4. Hydration

### Definitions

**Core Definition:** Hydration is the process by which React attaches event handlers and state to server-rendered HTML on the client, making it interactive.

**Technical Definition:** During hydration, React compares the server-rendered HTML with the output of the client render. If they match, React reuses the existing DOM and attaches event handlers. If they differ, React logs a hydration mismatch error. `useId` ensures that IDs match between server and client, preventing mismatches for accessibility attributes. Without `useId`, using random IDs or counters that differ between server and client causes hydration errors and breaks DOM relationships.

**Beginner-Friendly Explanation:** Hydration is when React "brings the page to life" in the browser after the server has sent the HTML. For this to work smoothly, the IDs on the server and client must be the same. `useId` guarantees that.

### Purposes

- To ensure ID attributes match between server and client during hydration.
- To prevent hydration mismatch errors and warnings.
- To preserve accessibility relationships after hydration.
- To allow the server-rendered HTML to remain valid and interactive.

### Syntax Rules and Structure

```jsx
const id = useId(); // Stable across SSR and hydration
```

**Syntax Rules:**

1. Use `useId` for any ID that will appear in both server and client renders.
2. Avoid random IDs and counters; they cause hydration mismatches.
3. Ensure the component tree is identical on server and client.
4. If rendering multiple apps, use `identifierPrefix` to scope IDs.

**Constraints and Limitations:**

- Conditional rendering that changes the tree between server and client can cause ID mismatches.
- `useId` is not available in async Server Components.

### Annotated Code Examples

```jsx
// ✅ Correct: useId ensures hydration match
function FormField() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Name:</label>
      <input id={id} type="text" />
    </>
  );
}

// ❌ Incorrect: random ID causes hydration mismatch
function BadFormField() {
  const id = Math.random().toString(36).slice(2); // Different on server and client!
  return (
    <>
      <label htmlFor={id}>Name:</label>
      <input id={id} type="text" />
    </>
  );
}
```

**Expected Output:** The `useId` version hydrates without errors. The random ID version triggers a hydration mismatch warning and breaks the label-input association.

**Why this output:** `useId` generates deterministic IDs based on the tree position. Random IDs differ between server and client, causing React's hydration comparison to fail.

### Real-World Cases

- **SSR form components:** Text fields, checkboxes, and radio buttons that need stable IDs.
- **Accessible modals:** `aria-labelledby` and `aria-describedby` IDs must survive hydration.
- **Design systems with SSR:** Components used across server-rendered pages.

### References

- DEV Community — useId and SSR hydration — https://dev.to/a1guy/the-definitive-react-19-useid-guide-patterns-pitfalls-and-pro-tips-40ia
- GitHub Issue #33779 — useId hydration mismatch in conditional loops — https://github.com/facebook/react/issues/33779
- Epic React — useId works seamlessly with SSR — https://www.epicreact.dev/improving-react-accessibility-with-use-id-knljs

---

## 5. Accessibility

### Definitions

**Core Definition:** Accessibility (a11y) is the practice of designing and building web applications that can be used by people with disabilities, including those who rely on screen readers, keyboard navigation, and other assistive technologies.

**Technical Definition:** In HTML, accessibility relationships are created by connecting elements with shared IDs. A `<label>` is connected to an `<input>` via the label's `htmlFor` attribute and the input's `id` attribute. ARIA attributes like `aria-describedby`, `aria-labelledby`, and `aria-controls` also use IDs to reference other elements. `useId` generates these IDs uniquely and stably, ensuring that assistive technologies can correctly interpret the relationships. Duplicate or unstable IDs break these relationships and make the application inaccessible.

**Beginner-Friendly Explanation:** Accessibility is about making sure everyone can use your app, including people who use screen readers or cannot use a mouse. `useId` helps by creating unique names for elements so that labels and descriptions are correctly connected to the inputs they belong to.

### Purposes

- To connect labels to inputs with unique, stable IDs.
- To create ARIA relationships (`aria-describedby`, `aria-labelledby`, `aria-controls`) that remain correct.
- To prevent duplicate IDs that confuse screen readers.
- To ensure accessibility attributes survive re-renders and hydration.

### Syntax Rules and Structure

**Label-Input Association:**

```jsx
const id = useId();
<label htmlFor={id}>Email</label>
<input id={id} type="email" />
```

**ARIA Relationships:**

```jsx
const id = useId();
<input aria-describedby={id + '-help'} />
<p id={id + '-help'}>Enter your email address</p>
```

**Syntax Rules:**

1. Use `useId` for any `id` used in accessibility attributes.
2. Use `htmlFor` on labels and `id` on inputs with the same value.
3. Use `aria-describedby` for hints and error messages.
4. Use `aria-labelledby` when the label text comes from another element.
5. Use `aria-controls` to indicate a controlling relationship.

### Annotated Code Examples

```jsx
function AccessibleInput({ label, helpText, error }) {
  const id = useId();
  const helpId = id + '-help';
  const errorId = id + '-error';

  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input
        id={id}
        aria-describedby={`${helpId}${error ? ` ${errorId}` : ''}`}
        aria-invalid={!!error}
      />
      <p id={helpId}>{helpText}</p>
      {error && <p id={errorId} role="alert">{error}</p>}
    </div>
  );
}
```

**Expected Output:** The input is associated with its label and its help text. If there is an error, the error message is also associated via `aria-describedby`. Screen readers announce the label, help text, and error together.

**Why this output:** A single `useId` base is used with suffixes to create `helpId` and `errorId`. The `aria-describedby` attribute references both the help text and the error message, creating a complete accessibility relationship.

### Real-World Cases

- **Form validation:** Connecting error messages to fields with `aria-describedby`.
- **Password fields:** Associating password requirements with the input.
- **Custom selects and comboboxes:** Using `aria-controls` and `aria-labelledby`.
- **Modal dialogs:** Using `aria-labelledby` for the modal title.

### References

- React — useId: Generating unique IDs for accessibility attributes — https://react.dev/reference/react/useId
- Mintlify — useId: ARIA relationships — https://mintlify.wiki/facebook/react/api/hooks/use-id
- web-accessibility-guide — useId for label association — https://github.com/arzu-guney/web-accessibility-guide
- Epic React — Improving React Accessibility with useId — https://www.epicreact.dev/improving-react-accessibility-with-use-id-knljs

---

## 6. `useId` Best Practices

### Definitions

**Core Definition:** `useId` best practices are the guidelines for using the Hook correctly — generating unique IDs for accessibility, avoiding misuse, and ensuring SSR compatibility.

**Technical Definition:** Best practices centre on using `useId` exclusively for DOM accessibility relationships, calling it at the top level of components, using suffixes for related IDs, avoiding list keys and cache keys, ensuring the component tree is identical between server and client, and using `identifierPrefix` when multiple apps share a page. The Hook should be called once per component and reused with suffixes rather than called multiple times. It should not be used for values that need to be stable across unmounts or for data-derived keys.

**Beginner-Friendly Explanation:** Use `useId` for accessibility IDs and nothing else. Call it at the top of your component. If you need several related IDs, call it once and add suffixes. Never use it for list keys. Make sure your server and client render the same tree.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use only for accessibility** | IDs for `htmlFor`, `aria-*`, `id` — not for keys or cache keys. |
| **Call at the top level** | Never inside loops, conditions, or nested functions. |
| **Call once, use suffixes** | One `useId()` per component; append suffixes for related IDs. |
| **Never use for list keys** | Keys must come from data, not `useId`. |
| **Never use for `use()` cache keys** | Cache keys must be data-derived. |
| **Ensure matching trees for SSR** | Server and client must render the same component tree. |
| **Use `identifierPrefix` for multiple apps** | Prevents ID clashes across independent React roots. |
| **Prefix for CSS selectors** | The colon in the ID makes it invalid for CSS; add a prefix if needed. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
function Form() {
  const id = useId(); // One call per component

  return (
    <form>
      <label htmlFor={id + '-firstName'}>First Name</label>
      <input id={id + '-firstName'} type="text" />

      <label htmlFor={id + '-lastName'}>Last Name</label>
      <input id={id + '-lastName'} type="text" />
    </form>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ Using useId for list keys
const key = useId();
{items.map(item => <div key={key}>{item}</div>)}

// ❌ Calling useId inside a condition
if (isVisible) {
  const id = useId(); // Breaks Hook rules
}

// ❌ Using random IDs instead of useId in SSR
const id = Math.random().toString(36); // Hydration mismatch!
```

**Expected Output:** The good practice produces unique, stable IDs for accessibility without SSR issues. The bad practices cause hydration mismatches, duplicate keys, and Hook violations.

**Why this output:** `useId` is designed for accessibility IDs, not keys. Conditional calls violate Hook rules. Random IDs differ between server and client.

### Real-World Cases

- **Design system form components:** Use `useId` for label and ARIA IDs.
- **SSR applications:** Ensure matching trees and use `identifierPrefix` if needed.
- **Micro-frontends:** Multiple React apps on one page use different `identifierPrefix` values.

### References

- React — useId: Caveats — https://react.dev/reference/react/useId
- Mintlify — useId: identifierPrefix — https://mintlify.wiki/facebook/react/api/hooks/use-id
- DEV Community — The Definitive React 19 useId Guide — https://dev.to/a1guy/the-definitive-react-19-useid-guide-patterns-pitfalls-and-pro-tips-40ia
- Epic React — When Not to Use useId — https://www.epicreact.dev/improving-react-accessibility-with-use-id-knljs

---

## 7. `useId` Pitfalls

### Definitions

**Core Definition:** `useId` pitfalls are common mistakes that undermine the Hook's purpose, cause hydration errors, or break accessibility.

**Technical Definition:** Common pitfalls include using `useId` for list keys (keys must come from data), using `useId` for `use()` cache keys (cache keys must be data-derived), calling `useId` conditionally (violates Hook rules), generating random IDs instead of `useId` in SSR (causes hydration mismatches), using `useId` inside async Server Components (unsupported), and assuming the ID is a valid CSS selector (the colon makes it invalid). Additional pitfalls include mismatched component trees between server and client and clashing IDs across multiple React roots without `identifierPrefix`.

**Beginner-Friendly Explanation:** These are the traps to avoid. Do not use `useId` for keys. Do not call it inside `if` statements. Do not use random IDs for SSR. Do not assume you can use the ID in CSS without a prefix.

### Common Pitfalls Table

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Using `useId` for list keys** | Keys must be data-derived; `useId` changes per instance. | Use `item.id` from data. |
| **Using `useId` for `use()` cache keys** | Cache keys must be data-derived. | Generate cache keys from data. |
| **Calling `useId` conditionally** | Violates Rules of Hooks. | Always call at the top level. |
| **Random IDs in SSR** | Hydration mismatch. | Use `useId`. |
| **`useId` in async Server Components** | Unsupported. | Use in client components or synchronous server components. |
| **Assuming ID is a valid CSS selector** | The colon makes it invalid. | Add a prefix: `id-${useId()}`. |
| **Mismatched trees in SSR** | IDs differ between server and client. | Ensure identical component trees. |
| **Clashing IDs across multiple roots** | Independent apps generate overlapping IDs. | Use `identifierPrefix`. |

### Annotated Code Examples

**Pitfall 1: `useId` for List Keys**

```jsx
// ❌ Wrong: useId is not for keys
{items.map(item => {
  const key = useId();
  return <div key={key}>{item}</div>;
})}

// ✅ Correct: key from data
{items.map(item => <div key={item.id}>{item}</div>)}
```

**Pitfall 2: Random IDs in SSR**

```jsx
// ❌ Wrong: hydration mismatch
const id = Math.random().toString(36);

// ✅ Correct: stable across server and client
const id = useId();
```

**Pitfall 3: `useId` in a CSS Selector**

```css
/* ❌ Wrong: colon is invalid in CSS selector */
#:r0: { color: red; }

/* ✅ Correct: prefix the ID */
#id-\:r0\: { color: red; }
```

**Expected Output:** The correct versions produce stable IDs, valid keys, and valid CSS selectors. The incorrect versions cause hydration errors, duplicate keys, or invalid CSS.

**Why this output:** `useId` is designed for accessibility IDs, not keys or cache keys. Random IDs break SSR. The colon in the generated ID requires escaping or prefixing for CSS.

### Real-World Cases

- **SSR frameworks:** Random IDs are a common source of hydration warnings.
- **Design systems:** Misusing `useId` for keys causes React reconciliation bugs.
- **Multiple React roots:** Missing `identifierPrefix` causes ID clashes.

### References

- React — useId: Caveats — https://react.dev/reference/react/useId
- React — useId: Pitfall (list keys) — https://react.dev/reference/react/useId
- DEV Community — The Definitive React 19 useId Guide — https://dev.to/a1guy/the-definitive-react-19-useid-guide-patterns-pitfalls-and-pro-tips-40ia
- Mintlify — useId: CSS selector limitation — https://mintlify.wiki/facebook/react/api/hooks/use-id
- GitHub Issue #33779 — useId hydration mismatch — https://github.com/facebook/react/issues/33779

---

## References

- React — useId (Official) — https://react.dev/reference/react/useId
- React — useId (Markdown) — https://react.dev/reference/react/useId.md
- React — useId (React 18) — https://18.react.dev/reference/react/useId
- React — useId (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-id
- React — useId (React 19) — https://19.reactjs.ac.cn/reference/react/useId
- Epic React — Improving React Accessibility with `useId` — https://www.epicreact.dev/improving-react-accessibility-with-use-id-knljs
- DEV Community — The Definitive React 19 useId Guide — https://dev.to/a1guy/the-definitive-react-19-useid-guide-patterns-pitfalls-and-pro-tips-40ia
- web-accessibility-guide — React/Next.js accessibility — https://github.com/arzu-guney/web-accessibility-guide
- GitHub Issue #33779 — useId hydration mismatch — https://github.com/facebook/react/issues/33779
- Telerik — The React useId Hook — https://www.telerik.com/blogs/the-react-useid-hook
- okf-bundles — useId reference — https://github.com/saschb2b/okf-bundles