# React Uncontrolled Components — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An uncontrolled component is a form element whose value is managed by the DOM itself rather than by React state, with the current value read on demand through a ref or a `FormData` object.

**Technical Definition:** In an uncontrolled component, form data is handled by the DOM. Instead of wiring `value` and `onChange` to React state for every keystroke, you give the element a `defaultValue` (or `defaultChecked`) for its initial value and attach a `ref` to read its current value when needed. The DOM is the single source of truth between renders. Submission typically reads values from `new FormData(e.currentTarget)` or from `ref.current.value`. React's documentation recommends controlled components in most cases, but notes that uncontrolled components can be easier when integrating with non-React code and can be slightly less code for simple forms. File inputs (`<input type="file">`) are always uncontrolled because their value can only be set by the user, not programmatically.

**Beginner-Friendly Explanation:** An uncontrolled component is a form field where the **browser** remembers what the user typed, not React. You give the field a starting value with `defaultValue`, and when you need to know what is in it — usually when the form is submitted — you look it up with a `ref` or by reading the form's data. It is like a paper form on a clipboard: you write on it yourself, and someone reads it later. Controlled components are the opposite: React watches every keystroke and keeps a copy of the value in state. Uncontrolled is less code for simple forms and easier to mix with non-React libraries, but it gives you less real-time control over validation and formatting.

### Key Characteristics

- **DOM-owned state:** The browser holds the value between renders; React reads it on demand.
- **`defaultValue` / `defaultChecked`:** Initial values are set with these props, not `value` / `checked`.
- **Refs or `FormData`:** Current values are read via a `ref` (`ref.current.value`) or at submit time via `new FormData(e.currentTarget)`.
- **No per-keystroke re-renders:** Typing does not update React state, so the component does not re-render on every keystroke.
- **File inputs are always uncontrolled:** Their `value` is read-only and can only be set by the user.
- **Less code for simple forms:** No state variables, no `onChange` handlers, no controlled round trip.
- **Harder to validate live:** Real-time validation, formatting, and conditional UI require extra work compared with controlled components.

### Prerequisites

- React function components and JSX.
- `useRef` and the concept of DOM refs.
- Native HTML form elements and attributes.
- Basic understanding of `FormData` and form submission.
- Familiarity with controlled components for comparison.

### Related Programming Areas

- **Controlled Components** — the state-driven alternative for real-time validation and formatting.
- **Form Fundamentals** — submission, validation, accessibility.
- **Refs** — reading DOM values imperatively.
- **React 19 Actions** — `<form action={fn}>` with `FormData` as a modern uncontrolled-friendly pattern.
- **Form Libraries** — `react-hook-form` uses uncontrolled inputs with refs for performance.

---

## 1. Uncontrolled Components

### Definitions

**Core Definition:** An uncontrolled component is a form element whose value is managed by the DOM rather than by React state.

**Technical Definition:** An uncontrolled component does not receive a `value` prop bound to React state. Instead, it receives a `defaultValue` (or `defaultChecked`) for initialisation and is read imperatively via a `ref` or declaratively via `FormData` on submission. React's documentation describes uncontrolled components as the alternative to controlled components, where form data is handled by the DOM itself. Because React does not track the value, changes do not trigger re-renders, and the component's behaviour is closer to a plain HTML form.

**Beginner-Friendly Explanation:** An uncontrolled component is a field where the browser keeps track of what the user enters, and React only looks at the value when it needs it.

### Purposes

- To reduce boilerplate for simple forms that do not need per-keystroke state.
- To integrate easily with non-React code and native DOM APIs.
- To avoid re-rendering on every keystroke for performance-sensitive forms.
- To handle file inputs, which cannot be controlled.
- To leverage native form features (`FormData`, `form.reset()`, constraint validation).

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useRef } from 'react';

function Form() {
  const inputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    console.log(inputRef.current.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="Initial" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `ref={inputRef}` | Attaches a ref to the DOM node. |
| `defaultValue` | The initial value; subsequent changes are not tracked by React. |
| `inputRef.current.value` | Reads the current DOM value when needed. |

**Syntax Rules:**

1. Use `defaultValue` (or `defaultChecked`), not `value` (or `checked`), for initial values.
2. Attach a `ref` to read the value imperatively, or read `FormData` on submit.
3. Do not pass both `value` and `defaultValue`; `value` would make the component controlled.
4. File inputs are always uncontrolled; read `e.target.files`.
5. Call `e.preventDefault()` in `onSubmit` to avoid a page reload (unless using a native `action` URL).

**Constraints and Limitations:**

- React cannot synchronise the value with state; reading it requires a ref or `FormData`.
- Live validation and formatting require manual event handlers or additional state.
- Resetting an uncontrolled input requires `form.reset()`, a key change, or a ref assignment.
- Uncontrolled components do not re-render on change, so derived UI will not update automatically.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Uncontrolled Text Input with a Ref

```jsx
import { useRef } from 'react';

function NameForm() {
  const nameRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Submitted: ${nameRef.current.value}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="name">Name</label>
      <input id="name" ref={nameRef} defaultValue="Alice" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** The input starts with "Alice". Typing does not re-render the component. Clicking Submit alerts the current value.

**Why this output:** `defaultValue` sets the initial DOM value. The `ref` gives direct access to the DOM node. `handleSubmit` reads `nameRef.current.value` at submit time. No React state is involved, so no re-renders occur on typing.

#### Example 2: Uncontrolled Form with `FormData`

```jsx
function SignupForm() {
  function handleSubmit(e) {
    e.preventDefault();
    const data = new FormData(e.currentTarget);
    const email = data.get('email');
    const newsletter = data.get('newsletter') === 'on';
    console.log({ email, newsletter });
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" type="email" defaultValue="" />

      <label>
        <input name="newsletter" type="checkbox" defaultChecked />
        Subscribe
      </label>

      <button type="submit">Sign up</button>
    </form>
  );
}
```

**Expected Output:** Submitting reads `email` as a string and `newsletter` as `true` (from the `"on"` value) without any React state.

**Why this output:** `new FormData(e.currentTarget)` collects all named controls. `defaultChecked` sets the checkbox's initial state. The `name` attributes are required for `FormData` to capture the values.

### Real-World Cases

- **Simple contact forms:** Name, email, and message fields without live validation.
- **File uploads:** Always uncontrolled because file inputs cannot be controlled.
- **Legacy form integration:** Mixing React with non-React form libraries.
- **Performance-sensitive forms:** Large forms where per-keystroke re-renders are costly.
- **React 19 form actions:** Passing `FormData` directly to an `action` function.

### References

- React — Uncontrolled Components (Legacy) — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input>`: Uncontrolled inputs — https://react.dev/reference/react-dom/components/input
- React — `<form>` — https://react.dev/reference/react-dom/components/form
- React — Manipulating the DOM with Refs — https://react.dev/learn/manipulating-the-dom-with-refs
- MDN — FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData

---

## 2. `defaultValue` Prop

### Definitions

**Core Definition:** The `defaultValue` prop sets the initial value of an uncontrolled form element without making it controlled.

**Technical Definition:** `defaultValue` is a React prop that sets the initial value of an `<input>`, `<textarea>`, or `<select>` during the first render. React writes it to the DOM node's initial value, but does not update the DOM when the prop changes on subsequent renders. In the React rendering lifecycle, the `value` attribute on form elements overrides the value in the DOM; `defaultValue` is used instead when React should specify an initial value but leave subsequent updates to the DOM. For `<select>` and `<textarea>`, `defaultValue` serves the same initialisation role.

**Beginner-Friendly Explanation:** `defaultValue` is the starting value of an uncontrolled field. After the user starts typing, React no longer controls it.

### Purposes

- To set the initial value of an uncontrolled input, textarea, or select.
- To prefill a form with server data without making it controlled.
- To leave subsequent updates to the DOM for uncontrolled behaviour.
- To avoid the read-only warning that comes from passing `value` without `onChange`.

### Syntax Rules and Structure

```jsx
<input defaultValue="Initial" />
<textarea defaultValue="Initial text" />
<select defaultValue="option-2">{/* options */}</select>
```

**Syntax Rules:**

1. Use `defaultValue` instead of `value` for uncontrolled elements.
2. `defaultValue` is applied on the initial render; later changes to the prop do not update the DOM.
3. For `<select>`, `defaultValue` must match an `<option>` value.
4. For `<textarea>`, `defaultValue` sets the text content.
5. Do not combine `value` and `defaultValue` on the same element.

**Constraints and Limitations:**

- Changing `defaultValue` after mount does not update the displayed value.
- `defaultValue` does not provide the per-keystroke reactivity of controlled components.
- For file inputs, `defaultValue` cannot be used to set a file.

### Annotated Code Examples

```jsx
function ProfileForm({ user }) {
  const nameRef = useRef(null);

  return (
    <form>
      <label htmlFor="name">Name</label>
      <input
        id="name"
        ref={nameRef}
        defaultValue={user.name} // Initial value from server
      />

      <label htmlFor="bio">Bio</label>
      <textarea
        id="bio"
        defaultValue={user.bio}
      />

      <label htmlFor="role">Role</label>
      <select id="role" defaultValue={user.role}>
        <option value="developer">Developer</option>
        <option value="designer">Designer</option>
      </select>
    </form>
  );
}
```

**Expected Output:** All three fields start with the user's server-provided values. Typing changes the DOM values, and React does not re-render.

**Why this output:** `defaultValue` initialises each control once from `user`. Subsequent edits are handled by the DOM, keeping the component uncontrolled.

### Real-World Cases

- **Edit forms:** Prefilling fields with existing server data.
- **Settings pages:** Loading current preferences as initial values.
- **Checkout forms:** Prefilling shipping details from the user's account.

### References

- React — Uncontrolled Components: Default Values (Legacy) — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input>`: defaultValue — https://react.dev/reference/react-dom/components/input
- React — `<textarea>`: defaultValue — https://react.dev/reference/react-dom/components/textarea
- React — `<select>`: defaultValue — https://react.dev/reference/react-dom/components/select

---

## 3. `defaultChecked` Prop

### Definitions

**Core Definition:** The `defaultChecked` prop sets the initial checked state of an uncontrolled checkbox or radio button.

**Technical Definition:** `defaultChecked` is the uncontrolled counterpart of `checked`. It initialises a checkbox or radio button's checked state on the first render. React does not update the DOM when the prop changes afterwards, leaving subsequent checked-state changes to the user and the DOM. React's documentation notes that `<input type="checkbox">` and `<input type="radio">` support `defaultChecked` for uncontrolled usage.

**Beginner-Friendly Explanation:** `defaultChecked` is the starting tick state of an uncontrolled checkbox or radio. After the user clicks it, the browser remembers the state, not React.

### Purposes

- To set the initial checked state of an uncontrolled checkbox or radio.
- To prefill boolean options from server data without controlled state.
- To support uncontrolled forms with boolean fields.
- To avoid the read-only warning from `checked` without `onChange`.

### Syntax Rules and Structure

```jsx
<input type="checkbox" defaultChecked={true} />
<input type="radio" name="size" value="large" defaultChecked />
```

**Syntax Rules:**

1. Use `defaultChecked` instead of `checked` for uncontrolled checkboxes and radios.
2. For radio groups, only one radio in the group should have `defaultChecked`.
3. `defaultChecked` applies on the initial render only.
4. Read the current checked state from `e.target.checked` in an event handler or via `FormData`.

**Constraints and Limitations:**

- Changing `defaultChecked` after mount does not update the checkbox.
- Uncontrolled checkboxes do not trigger re-renders, so derived UI will not update.
- `defaultChecked` cannot be used with `checked` on the same element.

### Annotated Code Examples

```jsx
function PreferencesForm({ initialPrefs }) {
  function handleSubmit(e) {
    e.preventDefault();
    const data = new FormData(e.currentTarget);
    console.log('Newsletter:', data.get('newsletter') === 'on');
    console.log('Theme:', data.get('theme'));
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        <input
          type="checkbox"
          name="newsletter"
          defaultChecked={initialPrefs.newsletter}
        />
        Subscribe to newsletter
      </label>

      <label>
        <input
          type="radio"
          name="theme"
          value="light"
          defaultChecked={initialPrefs.theme === 'light'}
        />
        Light
      </label>
      <label>
        <input
          type="radio"
          name="theme"
          value="dark"
          defaultChecked={initialPrefs.theme === 'dark'}
        />
        Dark
      </label>

      <button type="submit">Save</button>
    </form>
  );
}
```

**Expected Output:** The checkbox and radio initial states reflect `initialPrefs`. Submitting reads the current checked values from `FormData`.

**Why this output:** `defaultChecked` initialises the checked state. The DOM manages subsequent changes. `FormData` captures the submitted values using the `name` attributes.

### Real-World Cases

- **Preference forms:** Newsletter opt-ins and theme choices.
- **Settings panels:** Boolean toggles prefilled from user data.
- **Survey forms:** Default-selected radio options.

### References

- React — Uncontrolled Components: Default Values (Legacy) — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input type="checkbox">`: defaultChecked — https://react.dev/reference/react-dom/components/input
- React — `<input type="radio">`: defaultChecked — https://react.dev/reference/react-dom/components/input

---

## 4. Refs

### Definitions

**Core Definition:** A ref is a mutable object that holds a reference to a DOM node, used to read or manipulate an uncontrolled element directly.

**Technical Definition:** In React, a ref created with `useRef(null)` and attached to a DOM element via the `ref` prop receives the DOM node in its `current` property after mount. Refs are the imperative escape hatch for reading uncontrolled values (`ref.current.value`), calling DOM methods (`focus()`, `select()`), and resetting inputs (`ref.current.value = ''`). React 19 allows `ref` as a regular prop on function components and supports cleanup functions in callback refs. Refs do not trigger re-renders when mutated.

**Beginner-Friendly Explanation:** A ref is a handle to an actual DOM element. You attach it to an input, and later you can ask it "what is your value?" or tell it "clear yourself."

### Purposes

- To read the current value of an uncontrolled input on demand.
- To focus, select, or scroll to an element.
- To reset an uncontrolled input programmatically.
- To integrate with third-party DOM libraries that need the raw node.
- To avoid re-renders while still accessing DOM values.

### Syntax Rules and Structure

```jsx
import { useRef } from 'react';

const inputRef = useRef(null);

<input ref={inputRef} defaultValue="" />

// Read: inputRef.current.value
// Reset: inputRef.current.value = ''
// Focus: inputRef.current.focus()
```

**Syntax Rules:**

1. Create the ref with `useRef(null)`.
2. Attach it with `ref={inputRef}`.
3. Access the node via `inputRef.current`.
4. Read values in event handlers or effects, not during render.
5. In TypeScript, type the ref for the specific element (`useRef<HTMLInputElement>(null)`).

**Constraints and Limitations:**

- `ref.current` is `null` before mount and after unmount.
- Do not read or write refs during render (except initialization).
- Mutating `ref.current` does not trigger re-renders.

### Annotated Code Examples

```jsx
import { useRef } from 'react';

function SearchForm() {
  const inputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    const query = inputRef.current.value.trim();
    if (!query) {
      inputRef.current.focus(); // Focus the invalid field
      return;
    }
    console.log('Searching for:', query);
    inputRef.current.value = ''; // Reset the uncontrolled input
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} type="search" defaultValue="" />
      <button type="submit">Search</button>
    </form>
  );
}
```

**Expected Output:** Submitting with an empty field focuses the input. Submitting with a query logs it and clears the input.

**Why this output:** `inputRef.current` gives direct access to the DOM node. The handler can read, focus, and reset the input imperatively without React state.

### Real-World Cases

- **Search bars:** Reading the query and clearing the input after submission.
- **Focus management:** Focusing the first invalid field.
- **File inputs:** Reading `inputRef.current.files`.
- **Third-party integration:** Passing DOM nodes to libraries.

### References

- React — useRef — https://react.dev/reference/react/useRef
- React — Manipulating the DOM with Refs — https://react.dev/learn/manipulating-the-dom-with-refs
- React — `<input>`: Reading a ref — https://react.dev/reference/react-dom/components/input

---

## 5. Uncontrolled Inputs

### Definitions

**Core Definition:** An uncontrolled input is an `<input>` (or `<textarea>` / `<select>`) whose value is not bound to React state.

**Technical Definition:** Uncontrolled inputs use `defaultValue` / `defaultChecked` for initialisation and rely on the DOM to store the current value. Reading the value requires a ref or `FormData`. File inputs are inherently uncontrolled because their `value` is read-only. Uncontrolled inputs do not re-render on change, making them suitable for simple forms and performance-sensitive scenarios where per-keystroke state is unnecessary.

**Beginner-Friendly Explanation:** An uncontrolled input is a field where the browser holds the value. You set a starting value, and you read the current value when you need it.

### Purposes

- To avoid state management for simple fields.
- To reduce re-renders in large or performance-sensitive forms.
- To handle file inputs.
- To integrate with native form submission and `FormData`.
- To prefill values without maintaining controlled state.

### Syntax Rules and Structure

```jsx
<input defaultValue="Initial" ref={inputRef} name="field" />
<textarea defaultValue="Initial" name="bio" />
<select defaultValue="a" name="choice">{/* options */}</select>
```

**Syntax Rules:**

1. Use `defaultValue` for initial values.
2. Use `name` for `FormData` capture.
3. Use `ref` for imperative reads and resets.
4. Do not pass `value` (that would make the input controlled).
5. For checkboxes and radios, use `defaultChecked`.

**Constraints and Limitations:**

- No per-keystroke validation or formatting without extra handlers.
- Derived UI does not update automatically.
- Resetting requires `form.reset()`, a key change, or a ref assignment.

### Annotated Code Examples

```jsx
import { useRef } from 'react';

function CommentForm() {
  const formRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    const data = new FormData(e.currentTarget);
    console.log({
      author: data.get('author'),
      comment: data.get('comment'),
    });
    formRef.current.reset(); // Reset all uncontrolled inputs
  }

  return (
    <form ref={formRef} onSubmit={handleSubmit}>
      <input name="author" defaultValue="" placeholder="Your name" />
      <textarea name="comment" defaultValue="" placeholder="Comment" />
      <button type="submit">Post</button>
    </form>
  );
}
```

**Expected Output:** Submitting logs the author and comment values and resets the form. No React state or re-renders are involved.

**Why this output:** `FormData` collects the named controls. `formRef.current.reset()` restores all controls to their default values, which is the native reset mechanism.

### Real-World Cases

- **Comment forms:** Simple text and textarea inputs.
- **File uploads:** Reading `files` from a ref.
- **Large forms:** Avoiding per-keystroke re-renders.
- **Native form submission:** Using `action` and `FormData`.

### References

- React — Uncontrolled Components (Legacy) — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — `<textarea>` — https://react.dev/reference/react-dom/components/textarea
- React — `<select>` — https://react.dev/reference/react-dom/components/select
- MDN — HTMLFormElement.reset — https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/reset

---

## 6. Uncontrolled Components Benefits

### Definitions

**Core Definition:** Uncontrolled components benefit applications by reducing boilerplate, avoiding per-keystroke re-renders, and integrating naturally with DOM and non-React APIs.

**Technical Definition:** Uncontrolled components provide performance and simplicity advantages: no state variables or `onChange` handlers, no re-renders on each keystroke, native `FormData` and `form.reset()` support, and easier integration with non-React code. File inputs are only possible in uncontrolled form. React's documentation notes that since an uncontrolled component keeps the source of truth in the DOM, it is sometimes easier to integrate React and non-React code, and can be slightly less code for quick implementations.

**Beginner-Friendly Explanation:** Uncontrolled components are simpler and faster for forms that do not need live validation. They also work better with plain HTML features and outside libraries.

### Purposes

- To reduce boilerplate for simple forms.
- To avoid re-rendering on every keystroke.
- To integrate easily with native form APIs and non-React code.
- To support file uploads.
- To leverage `FormData` and native reset behaviour.

### Benefits Table

| Benefit | Description |
|---------|-------------|
| **Less code** | No state variables, `onChange` handlers, or controlled round trip. |
| **Fewer re-renders** | Typing does not update React state. |
| **Native form APIs** | `FormData`, `form.reset()`, and constraint validation work naturally. |
| **File uploads** | The only viable approach for `<input type="file">`. |
| **Non-React integration** | Easier to mix with plain DOM or third-party libraries. |
| **Simpler mental model** | The DOM is the source of truth for field values. |

### Annotated Code Examples

```jsx
function QuickContactForm() {
  function handleSubmit(e) {
    e.preventDefault();
    const data = new FormData(e.currentTarget);
    fetch('/api/contact', { method: 'POST', body: data });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="Name" defaultValue="" />
      <input name="email" type="email" placeholder="Email" defaultValue="" />
      <textarea name="message" placeholder="Message" defaultValue="" />
      <button type="submit">Send</button>
    </form>
  );
}
```

**Expected Output:** Submitting posts the form data directly to the API with no React state. The component does not re-render on typing.

**Why this output:** `FormData` captures all named controls. The `fetch` body accepts `FormData` directly. No state means no per-keystroke re-renders.

### Real-World Cases

- **Quick prototypes:** Fast forms without state boilerplate.
- **File uploads:** Avatars, documents, attachments.
- **Large forms:** Reducing render cost.
- **Non-React widgets:** Integrating with legacy DOM code.

### References

- React — Uncontrolled Components (Legacy): Benefits — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<form>` — https://react.dev/reference/react-dom/components/form
- MDN — FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData

---

## 7. Uncontrolled Components Pitfalls

### Definitions

**Core Definition:** Uncontrolled component pitfalls are the common mistakes that cause lost values, stale UI, incorrect resets, or integration failures.

**Technical Definition:** Pitfalls include: reading `ref.current` before mount (null); expecting re-renders on change; failing to reset values after submission; mixing `value` and `defaultValue`; missing `name` attributes so `FormData` captures nothing; attempting to control file inputs; and using `defaultValue` for values that need to update after mount. These issues arise because the DOM, not React, owns the value.

**Beginner-Friendly Explanation:** These are the traps that make uncontrolled components behave unexpectedly. The biggest one is forgetting that React does not track the value, so the UI will not update automatically.

### Common Pitfalls Table

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **`ref.current` is null** | Reading before mount | Read in event handlers or effects |
| **Expecting re-renders** | Derived UI does not update | Add state or an event handler if live UI is needed |
| **Missing `name`** | `FormData` is empty | Add `name` attributes |
| **Mixing `value` and `defaultValue`** | Becomes controlled or warns | Use one or the other |
| **Controlling file inputs** | `value` is read-only | Read `e.target.files` |
| **Stale `defaultValue`** | Changing the prop does not update the DOM | Use a key or controlled state |
| **No reset after submit** | Form retains old values | `form.reset()`, key change, or ref assignment |

### Annotated Code Examples

**Pitfall 1: Reading refs during render**

```jsx
// ❌ ref.current is null on first render
function Bad() {
  const ref = useRef(null);
  console.log(ref.current.value); // Error
  return <input ref={ref} />;
}

// ✅ Read in an event handler
function Good() {
  const ref = useRef(null);
  const handleClick = () => console.log(ref.current.value);
  return <input ref={ref} onClick={handleClick} />;
}
```

**Pitfall 2: Missing `name`**

```jsx
// ❌ FormData is empty
<input defaultValue="" />

// ✅ Named control
<input name="email" defaultValue="" />
```

**Pitfall 3: Mixing `value` and `defaultValue`**

```jsx
// ❌ Controlled and uncontrolled conflict
<input value={value} defaultValue="x" />

// ✅ Controlled
<input value={value} onChange={...} />

// ✅ Uncontrolled
<input defaultValue="x" />
```

**Expected Output:** The correct versions produce predictable reads and submissions. The incorrect versions throw errors, submit empty data, or trigger React warnings.

**Why this output:** Refs are null before mount. `FormData` only includes named controls. Mixing controlled and uncontrolled props is unsupported.

### Real-World Cases

- **FormData bugs:** Missing `name` attributes are a common cause of empty submissions.
- **Stale prefills:** `defaultValue` not updating after async data loads.
- **Reset issues:** Forms retaining values after submission.

### References

- React — Uncontrolled Components (Legacy) — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input>`: Caveats — https://react.dev/reference/react-dom/components/input
- MDN — FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData

---

## 8. Controlled vs Uncontrolled

### Definitions

**Core Definition:** Controlled and uncontrolled components differ in who owns the form value: React state or the DOM.

**Technical Definition:** Controlled components bind `value` (or `checked`) to React state and update it via `onChange`, making React the single source of truth. Uncontrolled components use `defaultValue` (or `defaultChecked`) and read the DOM value via refs or `FormData`, making the DOM the source of truth. Controlled components enable real-time validation, formatting, and conditional UI; uncontrolled components reduce boilerplate and re-renders and integrate more naturally with native form APIs.

**Beginner-Friendly Explanation:** Controlled means React remembers the value. Uncontrolled means the browser remembers it. Controlled gives you more control; uncontrolled gives you less code.

### Comparison Table

| Aspect | Controlled | Uncontrolled |
|--------|------------|--------------|
| Source of truth | React state | DOM |
| Value prop | `value` / `checked` | `defaultValue` / `defaultChecked` |
| Change handling | `onChange` updates state | DOM handles changes |
| Reading values | From state | From refs or `FormData` |
| Re-renders on typing | Yes | No |
| Real-time validation | Easy | Manual |
| Formatting as you type | Easy | Hard |
| Boilerplate | More | Less |
| File inputs | Not possible | Required |
| Reset | Set state | `form.reset()`, key, or ref |
| Best for | Dynamic, validated forms | Simple forms, files, performance |

### Annotated Code Examples

**Controlled:**

```jsx
const [email, setEmail] = useState('');
<input value={email} onChange={(e) => setEmail(e.target.value)} />
```

**Uncontrolled:**

```jsx
const emailRef = useRef(null);
<input ref={emailRef} defaultValue="" />
```

**Expected Output:** Both capture the email. The controlled version re-renders on each keystroke and can validate live; the uncontrolled version reads the value on demand.

**Why this output:** The controlled version stores the value in state. The uncontrolled version stores it in the DOM and reads it through the ref.

### Real-World Cases

- **Controlled:** Signup forms with validation, formatting, and dependent fields.
- **Uncontrolled:** File uploads, simple contact forms, large forms without live validation.

### References

- React — Uncontrolled Components (Legacy): Controlled vs uncontrolled — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input>`: Controlled vs uncontrolled — https://react.dev/reference/react-dom/components/input
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state

---

## 9. When to Use Each

### Definitions

**Core Definition:** Choosing between controlled and uncontrolled components depends on whether the form needs real-time React-driven behaviour or simple DOM-managed values.

**Technical Definition:** Controlled components are recommended in most cases because they give React full control over form state, enabling validation, formatting, conditional UI, and predictable submission. Uncontrolled components are appropriate for simple forms without live validation, file inputs (which cannot be controlled), integration with non-React code, performance-sensitive forms where per-keystroke re-renders are costly, and React 19 form actions that consume `FormData`.

**Beginner-Friendly Explanation:** Use controlled components when you need to react to what the user types. Use uncontrolled components for simple forms or files, or when you want less code and fewer re-renders.

### Decision Guide

| Scenario | Recommendation |
|----------|----------------|
| Live validation or formatting | Controlled |
| Dependent fields or conditional UI | Controlled |
| Prefilling from server data with live updates | Controlled |
| Simple contact form | Uncontrolled |
| File uploads | Uncontrolled |
| Large form with no live validation | Uncontrolled |
| Integration with non-React DOM code | Uncontrolled |
| React 19 form action with `FormData` | Uncontrolled |
| Performance-sensitive forms | Uncontrolled (or form library) |
| Most general cases | Controlled |

### Annotated Code Examples

**Controlled: live validation**

```jsx
function EmailForm() {
  const [email, setEmail] = useState('');
  const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

  return (
    <>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      {!isValid && email && <p>Enter a valid email</p>}
      <button disabled={!isValid}>Submit</button>
    </>
  );
}
```

**Uncontrolled: simple file upload**

```jsx
function FileUpload() {
  function handleSubmit(e) {
    e.preventDefault();
    const file = e.target.elements.avatar.files[0];
    const data = new FormData();
    data.append('avatar', file);
    fetch('/api/upload', { method: 'POST', body: data });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="avatar" type="file" />
      <button type="submit">Upload</button>
    </form>
  );
}
```

**Expected Output:** The controlled form validates in real time and disables the button when invalid. The uncontrolled form reads the file at submit time.

**Why this output:** The controlled form needs per-keystroke state to drive validation and button state. The file input cannot be controlled, so it is read via the form's elements at submission.

### Real-World Cases

- **Controlled:** Authentication, checkout, and any form with live validation.
- **Uncontrolled:** File uploads, quick prototypes, and large forms without live validation.
- **Hybrid:** A form may mix controlled and uncontrolled fields, e.g., controlled validation for email and an uncontrolled file input.

### References

- React — Uncontrolled Components (Legacy) — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — `<form>` — https://react.dev/reference/react-dom/components/form
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- react-hook-form — https://react-hook-form.com/

---

## References

- React — Uncontrolled Components (Legacy) — https://legacy.reactjs.org/docs/uncontrolled-components.html
- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — `<textarea>` — https://react.dev/reference/react-dom/components/textarea
- React — `<select>` — https://react.dev/reference/react-dom/components/select
- React — `<form>` — https://react.dev/reference/react-dom/components/form
- React — useRef — https://react.dev/reference/react/useRef
- React — Manipulating the DOM with Refs — https://react.dev/learn/manipulating-the-dom-with-refs
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- React — useActionState — https://react.dev/reference/react/useActionState
- React — useFormStatus — https://react.dev/reference/react-dom/hooks/useFormStatus
- MDN — FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN — HTMLFormElement.reset — https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/reset
- MDN — FileList — https://developer.mozilla.org/en-US/docs/Web/API/FileList
- MDN — `<input type="file">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file
- react-hook-form — https://react-hook-form.com/