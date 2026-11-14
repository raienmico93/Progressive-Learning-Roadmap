# React Form Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A form in React is a collection of interactive input elements — text fields, checkboxes, selects, and buttons — that collect user data, validate it, and submit it to an application or server.

**Technical Definition:** React forms are built on native HTML form elements (`<form>`, `<input>`, `<textarea>`, `<select>`, `<button>`) whose values are either **controlled** by React state (the `value` prop plus an `onChange` handler) or **uncontrolled** (the DOM owns the value, read via `FormData` or a ref). React normalises form events into SyntheticEvents, so `onChange` fires on every keystroke for text inputs (unlike the native DOM `change` event, which fires on blur). React 19 adds **form actions**: the `<form action={fn}>` prop accepts a function that receives `FormData`, integrates with `useActionState` for pending/error state, and integrates with `useFormStatus` for submit-button status. Validation can be layered: native HTML constraint validation, client-side JavaScript validation, and server-side validation.

**Beginner-Friendly Explanation:** A form is how users talk to your app — typing their name, picking a date, choosing an option, and clicking "Submit." In React, you usually let React remember what the user typed (this is called a **controlled** component) so you can validate it, show errors, and format it as they type. Sometimes you let the browser remember the value instead (an **uncontrolled** component) and read everything at once when the form is submitted. React also gives you events for every important moment: when the user types, when they leave a field, when they submit, and when the form resets. Getting forms right matters because forms are where users enter data — and where bugs and accessibility problems are most costly.

### Key Characteristics

- **Controlled vs. uncontrolled:** Controlled inputs are driven by React state (`value` + `onChange`); uncontrolled inputs are driven by the DOM (`defaultValue`, read via `FormData` or refs).
- **`onChange` fires per keystroke:** React's `onChange` behaves like the native `input` event for text inputs, not the native `change` event.
- **Single source of truth:** Controlled forms keep one authoritative state object, making validation and conditional UI predictable.
- **Synthetic events:** React wraps native events; `e.preventDefault()` and `e.stopPropagation()` work as expected.
- **Native validation is free:** `required`, `type="email"`, `pattern`, `min`, `max`, and `minLength` work without JavaScript.
- **React 19 actions:** `<form action={fn}>` receives `FormData` directly and integrates with `useActionState` and `useFormStatus`.
- **Accessibility is structural:** Labels, `aria-describedby`, `aria-invalid`, and focus management are not optional extras.

### Prerequisites

- React function components, state, props, and event handling.
- JSX syntax and the `className` / `htmlFor` conventions.
- Basic HTML form elements and attributes.
- Familiarity with `useState`, `useRef`, and `useEffect`.

### Related Programming Areas

- **State Management** — lifting form state, form libraries, and server state.
- **Accessibility** — labels, ARIA attributes, and focus management.
- **Data Fetching** — submitting forms to APIs and handling server responses.
- **Validation** — schema validation (Zod, Yup) and constraint validation.
- **Rendering Performance** — controlled inputs re-render on every keystroke.

---

## 1. Forms

### Definitions

**Core Definition:** A form is a container element that groups input controls and handles their submission as a single unit.

**Technical Definition:** The `<form>` element groups interactive controls and defines submission behaviour. In React, `<form>` supports `onSubmit`, `onReset`, and (React 19+) `action`. Submission gathers the form's controls into a `FormData` object, which can be read in the submit handler or passed directly to a form action function. React's `<form>` supports the native `action` attribute (a URL) and, in React 19, a function action that receives `FormData`.

**Beginner-Friendly Explanation:** A form is the box that holds all the fields and the submit button. It ties them together so the browser knows what to send when the user submits.

### Purposes

- To group related input controls into a single submission unit.
- To provide native submission, reset, and validation behaviour.
- To collect user data into a `FormData` object for processing.
- To anchor accessible structure (fieldsets, legends, error summaries).

### Syntax Rules and Structure

**General Syntax:**

```jsx
<form onSubmit={handleSubmit}>
  {/* controls */}
  <button type="submit">Submit</button>
</form>
```

**Component Breakdown:**

| Element / Attribute | Description |
|---------------------|-------------|
| `<form>` | Groups controls and handles submission |
| `onSubmit` | Handler that receives a SyntheticEvent |
| `action` | URL (native) or function (React 19+) |
| `type="submit"` | Button that triggers submission |
| `type="reset"` | Button that triggers reset |

**Syntax Rules:**

1. Call `e.preventDefault()` inside `onSubmit` to stop the full-page reload.
2. Use `className` (not `class`) and `htmlFor` (not `for`).
3. In React 19+, `action` can be a function that receives `FormData`; React resets uncontrolled fields after a successful action.
4. A submit button must have `type="submit"` (the default inside a form).
5. Buttons that should not submit must have `type="button"`.

**Constraints and Limitations:**

- A function `action` cannot be used together with an `onSubmit` handler in the same form.
- Native reset does not reset React controlled state unless you handle `onReset`.
- Nested forms are invalid HTML.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Controlled Form

```jsx
import { useState } from 'react';

function ContactForm() {
  const [name, setName] = useState('');

  function handleSubmit(e) {
    e.preventDefault(); // Stop full-page reload
    console.log('Submitted name:', name);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="name">Name</label>
      <input
        id="name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** Typing updates the input on every keystroke. Clicking Submit logs the name and does not reload the page.

**Why this output:** `value={name}` makes the input controlled by React state. `onChange` updates state on every keystroke. `e.preventDefault()` stops the browser's default navigation.

#### Example 2: React 19 Form Action

```jsx
function ContactForm() {
  function handleAction(formData) {
    const name = formData.get('name'); // Read from FormData
    console.log('Submitted name:', name);
  }

  return (
    <form action={handleAction}>
      <label htmlFor="name">Name</label>
      <input id="name" name="name" defaultValue="" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** Clicking Submit passes the form's `FormData` to `handleAction`. The uncontrolled field is reset automatically after the action completes.

**Why this output:** React 19's function `action` receives `FormData` directly, removing the need for a controlled state object. The `name` attribute is required so the field appears in `FormData`.

### Real-World Cases

- **Authentication:** Login and signup forms with validation.
- **Checkout:** Shipping address and payment forms.
- **Settings:** Profile and preference forms.
- **Search:** Filter and query forms.

### References

- React — `<form>` — https://react.dev/reference/react-dom/components/form
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- MDN — Web forms — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms

---

## 2. Form Elements

### Definitions

**Core Definition:** Form elements are the interactive controls — inputs, textareas, selects, checkboxes, radios, and buttons — that collect user data inside a form.

**Technical Definition:** React supports the native form controls with controlled and uncontrolled patterns. Each control type has a specific value prop: `value` for text inputs, textareas, and selects; `checked` for checkboxes and radios; `files` for file inputs (always uncontrolled). Default values use `defaultValue` and `defaultChecked`. React requires `onChange` alongside `value` to avoid read-only warnings, and uses `htmlFor` for label association.

**Beginner-Friendly Explanation:** Form elements are the actual fields users interact with — the text box, the dropdown, the checkbox. React lets you either watch every keystroke (controlled) or read the values when the form is submitted (uncontrolled).

### Purposes

- To collect specific types of user input (text, numbers, dates, files, choices).
- To provide accessible, labelled controls with appropriate semantics.
- To enable controlled validation and formatting as the user types.
- To support uncontrolled, form-action-based submission for simpler forms.

### Syntax Rules and Structure

**Control Patterns:**

| Control | Controlled prop | Uncontrolled prop | Event |
|---------|-----------------|-------------------|-------|
| `<input type="text">` | `value` | `defaultValue` | `onChange` |
| `<textarea>` | `value` | `defaultValue` | `onChange` |
| `<select>` | `value` | `defaultValue` | `onChange` |
| `<input type="checkbox">` | `checked` | `defaultChecked` | `onChange` |
| `<input type="radio">` | `checked` | `defaultChecked` | `onChange` |
| `<input type="file">` | — | (always uncontrolled) | `onChange` |

**Syntax Rules:**

1. Never pass `value` without `onChange`; React warns and the field becomes read-only.
2. Use `checked` (not `value`) for checkboxes and radios.
3. Use `defaultValue` / `defaultChecked` for uncontrolled controls.
4. File inputs are always uncontrolled; read files from `e.target.files` or `FormData`.
5. `<textarea>` uses `value`, not children, in React.
6. `<select>` uses `value` on the `<select>` element, and `<option>` children use `value`.
7. For `<select multiple>`, `value` must be an array.

**Constraints and Limitations:**

- A controlled input's value cannot be changed by the browser without React state updating.
- Radio groups require matching `name` attributes and a shared state value.
- Checkbox groups require array state and mapping over values.

### Annotated Code Examples

```jsx
import { useState } from 'react';

function ProfileForm() {
  const [form, setForm] = useState({
    name: '',
    bio: '',
    role: 'developer',
    newsletter: false,
  });

  function update(field) {
    return (e) => {
      const value =
        e.target.type === 'checkbox' ? e.target.checked : e.target.value;
      setForm((prev) => ({ ...prev, [field]: value }));
    };
  }

  return (
    <form>
      <label htmlFor="name">Name</label>
      <input id="name" value={form.name} onChange={update('name')} />

      <label htmlFor="bio">Bio</label>
      <textarea id="bio" value={form.bio} onChange={update('bio')} />

      <label htmlFor="role">Role</label>
      <select id="role" value={form.role} onChange={update('role')}>
        <option value="developer">Developer</option>
        <option value="designer">Designer</option>
      </select>

      <label>
        <input
          type="checkbox"
          checked={form.newsletter}
          onChange={update('newsletter')}
        />
        Subscribe to newsletter
      </label>
    </form>
  );
}
```

**Expected Output:** All four controls stay in sync with the single `form` state object. The checkbox stores a boolean; the select stores a string; text and textarea store strings.

**Why this output:** A single updater function handles all control types by branching on `type === 'checkbox'`. Each control uses the correct value prop (`value` vs `checked`) and is bound to the matching state field.

### Real-World Cases

- **Registration forms:** Text, email, password, and checkbox fields.
- **Surveys:** Radio groups, selects, and textareas.
- **Upload forms:** File inputs and multiple file selection.
- **Settings panels:** Toggles (checkboxes) and dropdowns.

### References

- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — `<textarea>` — https://react.dev/reference/react-dom/components/textarea
- React — `<select>` — https://react.dev/reference/react-dom/components/select
- MDN — HTML forms elements — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form

---

## 3. Form Submission

### Definitions

**Core Definition:** Form submission is the process of gathering the form's data and sending it to an application handler or server endpoint.

**Technical Definition:** Submission is triggered by a submit button or by pressing Enter in a single-line input. In React, submission is handled by `onSubmit` (which receives a SyntheticEvent and requires `e.preventDefault()` to stop the native navigation) or, in React 19, by a function `action` prop that receives a `FormData` object. The `FormData` object contains all named controls. Server submission typically uses `fetch` with a `FormData` or JSON body; React 19 actions can be async functions, and `useActionState` manages the returned state.

**Beginner-Friendly Explanation:** Submission is what happens when the user clicks "Submit" or presses Enter. Your code grabs all the values, checks them, and sends them to the server.

### Purposes

- To collect all form values into a single `FormData` object or state snapshot.
- To validate data before sending it to the server.
- To send data to an API and handle success, failure, and pending states.
- To prevent full-page reloads and provide a smooth single-page experience.
- To reset or preserve form state based on the submission outcome.

### Syntax Rules and Structure

**Submission Patterns:**

| Pattern | Mechanism |
|---------|-----------|
| `onSubmit` + state | Read controlled state, `e.preventDefault()`, send request |
| `onSubmit` + FormData | `new FormData(e.currentTarget)`, read values |
| React 19 `action` | `<form action={async (formData) => {}}>` |
| React 19 `useActionState` | `[state, formAction, isPending] = useActionState(action, initial)` |
| React 19 `useFormStatus` | `pending` from a child of the form |

**Syntax Rules:**

1. Call `e.preventDefault()` in `onSubmit` to avoid a full-page reload.
2. Use `new FormData(e.currentTarget)` to read uncontrolled values.
3. Named controls (`name="..."`) are required for `FormData` to capture them.
4. In React 19, `action` receives `FormData` and can be async; React manages pending state for the form.
5. Use `useFormStatus()` in a child component of the form, not in the form component itself.
6. Handle errors and return field-level error state from `useActionState`.

**Constraints and Limitations:**

- Function `action` and `onSubmit` cannot both be used on the same form.
- `useFormStatus` only reports the status of the nearest parent `<form>`.
- Native submission causes a page reload unless prevented.

### Annotated Code Examples

**`onSubmit` with `FormData`:**

```jsx
function SearchForm() {
  async function handleSubmit(e) {
    e.preventDefault();
    const data = new FormData(e.currentTarget);
    const query = data.get('query');

    const res = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
    const results = await res.json();
    console.log(results);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="query" type="search" />
      <button type="submit">Search</button>
    </form>
  );
}
```

**Expected Output:** Submitting reads `query` from `FormData`, sends the request, and logs the results without reloading the page.

**Why this output:** `e.preventDefault()` stops navigation. `new FormData(e.currentTarget)` captures named controls. The `name="query"` attribute makes the field available in `FormData`.

**React 19 `useActionState`:**

```jsx
import { useActionState } from 'react';

async function submitForm(prevState, formData) {
  const email = formData.get('email');
  if (!email) {
    return { error: 'Email is required' };
  }
  await fetch('/api/subscribe', { method: 'POST', body: formData });
  return { success: true };
}

function NewsletterForm() {
  const [state, formAction, isPending] = useActionState(submitForm, null);

  return (
    <form action={formAction}>
      <input name="email" type="email" />
      {state?.error && <p role="alert">{state.error}</p>}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Submitting…' : 'Subscribe'}
      </button>
    </form>
  );
}
```

**Expected Output:** Submitting calls `submitForm` with `FormData`. `isPending` disables the button while the request runs. Errors are rendered with `role="alert"`.

**Why this output:** `useActionState` wraps the action and returns the latest state, a form action, and a pending flag. React manages the submission lifecycle.

### Real-World Cases

- **Authentication:** Login and signup submissions with error feedback.
- **Checkout:** Multi-step order submission with validation.
- **Newsletter:** Simple email subscription forms.
- **Search:** Query submission with URL updates.

### References

- React — `<form>` — https://react.dev/reference/react-dom/components/form
- React — useActionState — https://react.dev/reference/react/useActionState
- React — useFormStatus — https://react.dev/reference/react-dom/hooks/useFormStatus
- MDN — Sending form data — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data

---

## 4. Form Validation

### Definitions

**Core Definition:** Form validation is the process of checking that user input meets the required rules before it is accepted or submitted.

**Technical Definition:** Validation can occur at three layers. **Native constraint validation** uses HTML attributes (`required`, `type`, `pattern`, `min`, `max`, `minLength`, `maxLength`, `step`) and is enforced by the browser on submission. **Client-side JavaScript validation** checks values in React state or `FormData` and displays custom error messages. **Server-side validation** is the authoritative check performed on the server, which must never be skipped because client-side checks can be bypassed. Validation state is typically stored alongside the form data and surfaced through `aria-invalid` and `aria-describedby`.

**Beginner-Friendly Explanation:** Validation is checking whether the user filled in the form correctly. The browser can check some things for free (like "this field is required"), your React code can check more complex rules, and the server must always check again because users can get around client-side checks.

### Purposes

- To prevent invalid or incomplete data from being submitted.
- To provide immediate, helpful feedback to the user.
- To enforce business rules (password strength, valid dates, required fields).
- To meet accessibility requirements by associating errors with fields.
- To protect the server from malformed or malicious input.

### Syntax Rules and Structure

**Validation Layers:**

| Layer | Mechanism | Trust level |
|-------|-----------|-------------|
| Native | HTML attributes | Convenience only |
| Client-side JS | State checks, schema validation | UX only |
| Server-side | Server checks | Authoritative |

**Native Constraint Attributes:**

| Attribute | Applies to |
|-----------|------------|
| `required` | Text, select, checkbox, radio |
| `type="email"` / `type="url"` | Format validation |
| `pattern` | Regex pattern |
| `min` / `max` | Number and date ranges |
| `minLength` / `maxLength` | Text length |
| `step` | Numeric increments |

**Syntax Rules:**

1. Use native attributes as the first line of defence; they work without JavaScript.
2. Validate on `onBlur` (field-level) and on `onSubmit` (whole form).
3. Display errors with `aria-invalid="true"` and `aria-describedby` pointing to the error message.
4. Associate errors with fields using `useId`.
5. Never rely on client-side validation alone; always validate on the server.
6. Focus the first invalid field on submit failure.

**Constraints and Limitations:**

- Native validation UI is inconsistent across browsers.
- Client-side validation can be bypassed.
- Validating on every keystroke can feel noisy; prefer `onBlur` or debounced validation.

### Annotated Code Examples

```jsx
import { useId, useState } from 'react';

function SignupForm() {
  const emailId = useId();
  const emailErrorId = useId();
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  function validate(value) {
    if (!value) return 'Email is required';
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return 'Enter a valid email';
    return '';
  }

  function handleBlur() {
    setError(validate(email));
  }

  function handleSubmit(e) {
    e.preventDefault();
    const message = validate(email);
    if (message) {
      setError(message);
      document.getElementById(emailId)?.focus();
      return;
    }
    // Submit to server
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      <label htmlFor={emailId}>Email</label>
      <input
        id={emailId}
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        onBlur={handleBlur}
        aria-invalid={error ? 'true' : undefined}
        aria-describedby={error ? emailErrorId : undefined}
        required
      />
      {error && <p id={emailErrorId} role="alert">{error}</p>}
      <button type="submit">Sign up</button>
    </form>
  );
}
```

**Expected Output:** Leaving the field empty or entering an invalid email shows an error message. On submit, the first invalid field is focused and announced to assistive technology.

**Why this output:** `onBlur` triggers field-level validation. `aria-invalid` and `aria-describedby` connect the error to the input. `noValidate` disables the browser's native popup so custom messages are shown consistently. Focus moves to the invalid field on submit.

### Real-World Cases

- **Signup forms:** Email format and password strength.
- **Checkout:** Required shipping fields and valid payment details.
- **Booking forms:** Date ranges and availability checks.
- **Settings:** Profile fields with length and format constraints.

### References

- MDN — Client-side form validation — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation
- MDN — Constraint validation — https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation
- W3C WAI — Forms Tutorial: Validating Input — https://www.w3.org/WAI/tutorials/forms/validation/
- react-hook-form — https://react-hook-form.com/

---

## 5. Form State

### Definitions

**Core Definition:** Form state is the data that tracks the current values, validation errors, touched fields, and submission status of a form.

**Technical Definition:** Form state is typically a single object holding field values, plus derived validation state (errors, validity) and interaction state (touched, dirty, submitting). It can live in `useState`, `useReducer`, a form library (`react-hook-form`, Formik), or the DOM itself (uncontrolled). Controlled forms use one source of truth for values; derived values like `isValid` are computed during render rather than stored. Lifting form state to a parent component enables cross-field validation and sharing.

**Beginner-Friendly Explanation:** Form state is your app's memory of what the user has typed, which fields they have touched, and whether there are errors. Keeping it organized makes validation and conditional UI much easier.

### Purposes

- To track current field values as the user types.
- To track interaction state (touched, dirty) for better error timing.
- To derive validation errors and overall validity.
- To enable conditional UI (disable submit, show hints).
- To lift state for cross-field validation and multi-step forms.

### Syntax Rules and Structure

**State Shape:**

```js
{
  values: { name: '', email: '' },
  errors: { email: 'Enter a valid email' },
  touched: { email: true },
  isSubmitting: false,
}
```

**Syntax Rules:**

1. Group related field values into one object rather than many separate `useState` calls for a single form.
2. Derive `isValid` from values during render; do not store it.
3. Store errors only for fields that have been touched or submitted.
4. Use `useReducer` when form transitions become complex (add field, reset, validate).
5. Lift state to the parent when siblings need to read or change it.

**Constraints and Limitations:**

- Storing derived validity in state causes synchronisation bugs.
- Separate `useState` calls per field make cross-field validation and reset harder.
- Form libraries add bundle size but reduce boilerplate.

### Annotated Code Examples

```jsx
import { useState } from 'react';

function SignupForm() {
  const [values, setValues] = useState({ email: '', password: '' });
  const [touched, setTouched] = useState({});

  // Derived during render — never stored in state
  const errors = {
    email: !values.email ? 'Email is required' : '',
    password: values.password.length < 8 ? 'At least 8 characters' : '',
  };
  const isValid = !errors.email && !errors.password;

  function handleChange(field) {
    return (e) => setValues((v) => ({ ...v, [field]: e.target.value }));
  }

  function handleBlur(field) {
    setTouched((t) => ({ ...t, [field]: true }));
  }

  return (
    <form>
      <input
        value={values.email}
        onChange={handleChange('email')}
        onBlur={() => handleBlur('email')}
        aria-invalid={touched.email && !!errors.email}
      />
      {touched.email && errors.email && <p role="alert">{errors.email}</p>}

      <input
        type="password"
        value={values.password}
        onChange={handleChange('password')}
        onBlur={() => handleBlur('password')}
      />
      {touched.password && errors.password && <p role="alert">{errors.password}</p>}

      <button disabled={!isValid}>Sign up</button>
    </form>
  );
}
```

**Expected Output:** Errors appear only after a field is touched. The submit button is disabled until both fields are valid.

**Why this output:** `values` is the single source of truth. `errors` and `isValid` are derived during render, so they never go stale. `touched` delays error display until the user leaves a field.

### Real-World Cases

- **Multi-step wizards:** Lifting state to a parent across steps.
- **Cross-field validation:** Password confirmation, date ranges.
- **Conditional forms:** Showing fields based on earlier answers.
- **Draft saving:** Persisting form state to storage.

### References

- React — Managing State: Reacting to input with state — https://react.dev/learn/reacting-to-input-with-state
- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure

---

## 6. Form Data

### Definitions

**Core Definition:** Form data is the collection of name-value pairs produced by a form's controls at submission time.

**Technical Definition:** The browser's `FormData` API constructs a `Set`-like collection of entries from a form's named controls. In React, `new FormData(e.currentTarget)` reads uncontrolled values in an `onSubmit` handler; in React 19, a function `action` receives `FormData` directly. `FormData` supports files, multiple values for the same name, and conversion to a plain object via `Object.fromEntries` (with caveats for repeated names and files).

**Beginner-Friendly Explanation:** Form data is the bundle of everything the user entered, ready to send to the server. React can give it to you as a `FormData` object.

### Purposes

- To collect all named controls in one object without controlled state.
- To support file uploads and repeated field names.
- To pass data directly to a React 19 form action.
- To serialise form values for API requests.

### Syntax Rules and Structure

**`FormData` APIs:**

| Method | Purpose |
|--------|---------|
| `formData.get(name)` | First value for a name |
| `formData.getAll(name)` | All values for a name |
| `formData.entries()` | Iterate name-value pairs |
| `Object.fromEntries(formData)` | Convert to plain object |

**Syntax Rules:**

1. Controls must have a `name` attribute to appear in `FormData`.
2. `getAll` is required for repeated names (checkbox groups, multi-select).
3. File inputs appear as `File` objects in `FormData`.
4. `Object.fromEntries` keeps only the last value for repeated names.
5. In React 19, `action` receives `FormData` and React resets uncontrolled fields after a successful action.

### Annotated Code Examples

```jsx
function PreferencesForm() {
  function handleAction(formData) {
    const topics = formData.getAll('topics'); // Repeated name
    const avatar = formData.get('avatar');    // File
    const email = formData.get('email');      // Single value

    console.log({ email, topics, avatarName: avatar?.name });
  }

  return (
    <form action={handleAction}>
      <input name="email" type="email" />

      <label>
        <input type="checkbox" name="topics" value="react" />
        React
      </label>
      <label>
        <input type="checkbox" name="topics" value="css" />
        CSS
      </label>

      <input name="avatar" type="file" />

      <button type="submit">Save</button>
    </form>
  );
}
```

**Expected Output:** Checking both topics returns `['react', 'css']` from `getAll('topics')`. The file input returns a `File` object. The email returns a string.

**Why this output:** Multiple controls sharing `name="topics"` produce multiple entries, so `getAll` is required. File inputs always produce `File` objects. Single-name fields return the first value with `get`.

### Real-World Cases

- **File uploads:** Avatars, documents, and attachments.
- **Multi-select filters:** Categories, tags, and permissions.
- **Uncontrolled forms:** Simple forms that do not need per-keystroke state.
- **React 19 actions:** Passing `FormData` directly to server actions.

### References

- MDN — FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData
- React — `<form>` — https://react.dev/reference/react-dom/components/form
- MDN — Sending form data — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data

---

## 7. Form Events

### Definitions

**Core Definition:** Form events are the React SyntheticEvents fired by form controls and the form element in response to user interaction and submission.

**Technical Definition:** React attaches event listeners at the root and dispatches SyntheticEvents along the fiber tree. Form-related events include `onSubmit`, `onChange`, `onInput`, `onBlur`, `onFocus`, and `onReset`. React normalises `onChange` to fire on every input event for text controls, matching the native `input` event rather than the native `change` event. Each event receives a SyntheticEvent whose `target` is the element that triggered the event.

**Beginner-Friendly Explanation:** Form events are the moments React tells you something happened — the user typed, left a field, submitted, or reset the form. You attach functions to these moments to respond.

### Form Events Overview

| Event | Fires when | Common use |
|-------|-----------|------------|
| `onSubmit` | Form is submitted | Validate and send data |
| `onChange` | Value changes (per keystroke for text) | Update controlled state |
| `onInput` | Input value changes | Same as `onChange` in React for text |
| `onBlur` | Element loses focus | Field-level validation |
| `onFocus` | Element gains focus | Show hints, track focus |
| `onReset` | Form is reset | Reset controlled state |

---

### 7.1 `onSubmit`

**Definitions:**

- **Core Definition:** `onSubmit` is the event handler that runs when a form is submitted.
- **Technical Definition:** React's `onSubmit` receives a SyntheticEvent whose `currentTarget` is the `<form>` element. Calling `e.preventDefault()` stops the browser's default navigation. It fires when a submit button is clicked or Enter is pressed in a single-line input.
- **Beginner-Friendly Explanation:** `onSubmit` is what happens when the user clicks "Submit." It is where you validate and send the data.

**Purposes:**

- To validate the entire form before submission.
- To prevent the default page reload with `e.preventDefault()`.
- To gather form data and send it to an API.
- To manage submission state (loading, errors, success).

**Syntax Rules and Structure:**

```jsx
<form onSubmit={handleSubmit}>
  <button type="submit">Submit</button>
</form>
```

1. Call `e.preventDefault()` to avoid navigation.
2. Read data from state or `new FormData(e.currentTarget)`.
3. Only `type="submit"` buttons trigger submission.
4. A function `action` cannot be combined with `onSubmit`.

**Annotated Example:**

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');

  async function handleSubmit(e) {
    e.preventDefault();
    if (!email) return;
    await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email }),
      headers: { 'Content-Type': 'application/json' },
    });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button type="submit">Log in</button>
    </form>
  );
}
```

**Expected Output:** Clicking "Log in" prevents the page reload, validates the email, and sends a POST request.

**Why this output:** `e.preventDefault()` stops the native submission. The controlled `email` state provides the payload. The `type="submit"` button triggers `onSubmit`.

**Real-World Cases:** Login, signup, checkout, and search forms.

**References:** React — `<form>` — https://react.dev/reference/react-dom/components/form

---

### 7.2 `onChange`

**Definitions:**

- **Core Definition:** `onChange` is the event handler that fires when the value of a form control changes.
- **Technical Definition:** In React, `onChange` fires on every keystroke for text inputs, textareas, and selects, matching the native `input` event. This differs from the native DOM `change` event, which fires on blur. For checkboxes and radios, `onChange` fires when the checked state changes. React requires `onChange` alongside `value` for controlled controls.
- **Beginner-Friendly Explanation:** `onChange` runs every time the user types or changes a field. It is how you keep React state in sync with what the user enters.

**Purposes:**

- To update controlled state on every value change.
- To enable real-time validation, formatting, or filtering.
- To keep the UI in sync with user input.
- To drive conditional rendering based on input values.

**Syntax Rules and Structure:**

```jsx
<input value={value} onChange={(e) => setValue(e.target.value)} />
```

1. Always pair `value` with `onChange` for controlled inputs.
2. Use `e.target.value` for text, select, and textarea.
3. Use `e.target.checked` for checkboxes and radios.
4. Use the functional updater form for state that depends on previous state.

**Annotated Example:**

```jsx
function NameInput() {
  const [name, setName] = useState('');

  return (
    <>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <p>Hello, {name || 'stranger'}!</p>
    </>
  );
}
```

**Expected Output:** The greeting updates on every keystroke.

**Why this output:** `onChange` fires per keystroke, updating `name`. React re-renders with the new value and the greeting reflects it.

**Real-World Cases:** Live search, character counters, password strength meters.

**References:** React — `<input>` — https://react.dev/reference/react-dom/components/input

---

### 7.3 `onInput`

**Definitions:**

- **Core Definition:** `onInput` is the event handler that fires when the value of an input element changes.
- **Technical Definition:** React supports `onInput`, which maps to the native `input` event. For text inputs, `onInput` and `onChange` behave similarly in React, but `onInput` is the native event and fires before `onChange` in some cases. React 19 uses `onChange` as the idiomatic controlled-input handler; `onInput` is useful when interacting with native APIs or third-party libraries that expect the native event.
- **Beginner-Friendly Explanation:** `onInput` is very similar to `onChange` in React. It fires when the user types. Most React code uses `onChange`; `onInput` is for special cases.

**Purposes:**

- To respond to native input events for interoperability.
- To handle input types where native `input` semantics are required.
- To integrate with non-React libraries listening for `input`.

**Syntax Rules and Structure:**

```jsx
<input onInput={(e) => handle(e.nativeEvent)} />
```

1. `onInput` receives a SyntheticEvent; use `e.nativeEvent` for the native event.
2. Prefer `onChange` for standard controlled inputs.
3. Use `onInput` when a library or API requires the native `input` event.

**Annotated Example:**

```jsx
function SearchInput({ onQueryChange }) {
  return (
    <input
      type="search"
      onInput={(e) => onQueryChange(e.target.value)}
      placeholder="Search…"
    />
  );
}
```

**Expected Output:** Every keystroke calls `onQueryChange` with the current value.

**Why this output:** `onInput` fires on native input events, forwarding the value to the parent callback.

**Real-World Cases:** Integrating with native form libraries, custom elements, or third-party widgets.

**References:** MDN — input event — https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event

---

### 7.4 `onBlur`

**Definitions:**

- **Core Definition:** `onBlur` is the event handler that fires when an element loses focus.
- **Technical Definition:** `onBlur` maps to the native `blur` event, which does not bubble in the DOM but does propagate through React's synthetic event system. In forms, `onBlur` is commonly used to mark a field as "touched" and trigger field-level validation after the user leaves it.
- **Beginner-Friendly Explanation:** `onBlur` runs when the user clicks away from a field. It is the perfect moment to check whether what they typed is valid.

**Purposes:**

- To trigger field-level validation when the user leaves a field.
- To mark fields as touched for error display timing.
- To format or normalise values on blur.
- To close autocomplete or suggestion menus.

**Syntax Rules and Structure:**

```jsx
<input onBlur={() => setTouched(true)} />
```

1. Use `onBlur` for validation timing; use `onChange` for value updates.
2. Combine with a `touched` state map to display errors only after interaction.
3. `onBlur` fires before focus moves to the next element.

**Annotated Example:**

```jsx
function EmailField() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  function handleBlur() {
    setError(email.includes('@') ? '' : 'Enter a valid email');
  }

  return (
    <>
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        onBlur={handleBlur}
      />
      {error && <p role="alert">{error}</p>}
    </>
  );
}
```

**Expected Output:** The error appears only after the user leaves the field with an invalid email.

**Why this output:** `onBlur` runs when focus leaves the input, triggering validation without nagging the user on every keystroke.

**Real-World Cases:** Email, password, and username validation; formatting phone numbers.

**References:** MDN — blur event — https://developer.mozilla.org/en-US/docs/Web/API/Element/blur_event

---

### 7.5 `onFocus`

**Definitions:**

- **Core Definition:** `onFocus` is the event handler that fires when an element gains focus.
- **Technical Definition:** `onFocus` maps to the native `focus` event. In forms, it is used to show hints, highlight the active field, or open suggestion menus. It fires when the user clicks or tabs into a field.
- **Beginner-Friendly Explanation:** `onFocus` runs when the user clicks into or tabs to a field. It is a good moment to show help text or highlight the field.

**Purposes:**

- To show contextual hints or help text when a field is focused.
- To highlight the active field visually.
- To open autocomplete or suggestion menus.
- To track which field is active for keyboard navigation.

**Syntax Rules and Structure:**

```jsx
<input onFocus={() => setFocused(true)} />
```

1. Use `onFocus` for non-destructive UI changes (hints, highlights).
2. Pair with `onBlur` to reset focus-related UI.
3. Do not use `onFocus` for validation; use `onBlur` instead.

**Annotated Example:**

```jsx
function PasswordField() {
  const [focused, setFocused] = useState(false);

  return (
    <>
      <input
        type="password"
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
      />
      {focused && <p>Use at least 8 characters.</p>}
    </>
  );
}
```

**Expected Output:** The hint appears while the field is focused and disappears when focus leaves.

**Why this output:** `onFocus` and `onBlur` toggle the `focused` state, showing and hiding the hint.

**Real-World Cases:** Password requirement hints, search suggestions, field highlighting.

**References:** MDN — focus event — https://developer.mozilla.org/en-US/docs/Web/API/Element/focus_event

---

### 7.6 `onReset`

**Definitions:**

- **Core Definition:** `onReset` is the event handler that fires when a form is reset.
- **Technical Definition:** `onReset` maps to the native `reset` event, triggered by a `<button type="reset">` or `form.reset()`. Native reset restores DOM values to their default values but does **not** reset React controlled state, so a controlled form appears unchanged unless `onReset` also resets state.
- **Beginner-Friendly Explanation:** `onReset` runs when the form is cleared. Because React controls the values, you must also clear React state in this handler; otherwise, the fields stay filled.

**Purposes:**

- To reset controlled form state when the form is reset.
- To clear validation errors and touched state.
- To return the form to its initial values.
- To provide a "Clear" button that works with controlled inputs.

**Syntax Rules and Structure:**

```jsx
<form onReset={handleReset}>
  <button type="reset">Clear</button>
</form>
```

1. Native reset does not reset controlled state; handle `onReset` to reset state manually.
2. Call `e.preventDefault()` if you want to manage the reset entirely in React.
3. Reset errors and touched state alongside values.

**Annotated Example:**

```jsx
function SearchForm() {
  const initial = { query: '' };
  const [values, setValues] = useState(initial);
  const [touched, setTouched] = useState({});

  function handleReset(e) {
    e.preventDefault();
    setValues(initial);
    setTouched({});
  }

  return (
    <form onReset={handleReset}>
      <input
        value={values.query}
        onChange={(e) => setValues({ query: e.target.value })}
      />
      <button type="reset">Clear</button>
    </form>
  );
}
```

**Expected Output:** Clicking "Clear" resets the controlled input and clears touched state.

**Why this output:** `onReset` prevents the native reset (which would not affect controlled state) and resets the React state, causing the input to re-render empty.

**Real-World Cases:** Search forms with clear buttons, filters with reset controls, multi-field forms with a "Start over" action.

**References:** MDN — reset event — https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/reset_event
- React — `<form>` — https://react.dev/reference/react-dom/components/form

---

## 8. Form Accessibility

### Definitions

**Core Definition:** Form accessibility is the practice of building forms that can be understood and operated by everyone, including people using screen readers, keyboards, and other assistive technologies.

**Technical Definition:** Accessible forms require: a programmatic label for every control (`<label htmlFor>` or `aria-labelledby`); error messages associated via `aria-describedby`; invalid state exposed with `aria-invalid`; required state exposed with `aria-required` or the native `required` attribute; grouping with `<fieldset>` and `<legend>`; focus management on validation failure; and live announcements for dynamic errors (`role="alert"` or `aria-live`). Colour alone must never convey state.

**Beginner-Friendly Explanation:** Accessible forms work for everyone. Every field has a real label, errors are announced to screen readers, keyboard users can reach everything, and you never rely on colour alone to show a problem.

### Purposes

- To ensure every control has a programmatic label.
- To make error messages discoverable by screen readers.
- To support keyboard-only navigation and operation.
- To communicate required and invalid states programmatically.
- To group related controls with semantic fieldsets.

### Syntax Rules and Structure

**Accessibility Checklist:**

| Requirement | Implementation |
|-------------|----------------|
| Label | `<label htmlFor={id}>` with matching `id` |
| Error association | `aria-describedby={errorId}` |
| Invalid state | `aria-invalid="true"` |
| Required state | `required` + `aria-required="true"` |
| Grouping | `<fieldset>` + `<legend>` |
| Error announcement | `role="alert"` or `aria-live="assertive"` |
| Focus management | Focus first invalid field on submit |
| Unique IDs | `useId()` |

**Syntax Rules:**

1. Every input must have a visible, programmatic label.
2. Use `useId` to generate unique IDs for labels and error messages.
3. Associate errors with `aria-describedby` and mark fields with `aria-invalid`.
4. Use `<fieldset>` and `<legend>` for radio and checkbox groups.
5. Announce dynamic errors with `role="alert"`.
6. Move focus to the first invalid field on submit.
7. Do not rely on placeholder text as a label.

### Annotated Code Examples

```jsx
import { useId, useState } from 'react';

function AccessibleSignup() {
  const emailId = useId();
  const emailErrorId = useId();
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    if (!email.includes('@')) {
      setError('Enter a valid email address');
      document.getElementById(emailId)?.focus();
    }
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      <label htmlFor={emailId}>Email address</label>
      <input
        id={emailId}
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        aria-invalid={error ? 'true' : undefined}
        aria-describedby={error ? emailErrorId : undefined}
        aria-required="true"
        required
      />
      {error && (
        <p id={emailErrorId} role="alert">
          {error}
        </p>
      )}
      <button type="submit">Sign up</button>
    </form>
  );
}
```

**Expected Output:** The input has a visible label, announces its invalid state, associates the error message, and focuses the invalid field on submit.

**Why this output:** `htmlFor`/`id` connect the label. `aria-invalid` communicates validity. `aria-describedby` connects the error text. `role="alert"` announces the error. Focus moves to the invalid field.

### Real-World Cases

- **Signup and login:** Labelled fields with inline errors.
- **Checkout:** Grouped address fields with fieldsets.
- **Surveys:** Accessible radio and checkbox groups.
- **Search:** Labelled search inputs with clear buttons.

### References

- W3C WAI — Forms Tutorial — https://www.w3.org/WAI/tutorials/forms/
- WebAIM — Creating Accessible Forms — https://webaim.org/techniques/forms/
- React — useId — https://react.dev/reference/react/useId
- MDN — ARIA: aria-describedby — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby

---

## 9. Form Best Practices

### Definitions

**Core Definition:** Form best practices are the guidelines for building forms that are correct, accessible, performant, and maintainable.

**Technical Definition:** Best practices combine controlled/uncontrolled strategy, validation timing, accessible error presentation, submission state management, and library choices. The key decisions are: whether to use controlled state or `FormData`/actions; when to validate (on blur, on submit, or debounced); how to display and announce errors; how to manage pending and server-error states; and when a form library is justified.

**Beginner-Friendly Explanation:** Good forms are predictable, easy to use, and easy to maintain. They validate at the right time, show clear errors, work with keyboards and screen readers, and do not lose the user's work.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Choose controlled or uncontrolled deliberately** | Controlled for dynamic validation and formatting; uncontrolled/actions for simpler forms. |
| **Label every field** | Visible `<label>` with `htmlFor` and a matching `id`. |
| **Validate on blur and submit** | Avoid noisy per-keystroke errors; validate the whole form on submit. |
| **Show errors next to fields** | Associate errors with `aria-describedby` and `role="alert"`. |
| **Never rely on client validation alone** | Always validate on the server. |
| **Manage pending state** | Disable submit while submitting; use `useFormStatus` or `isPending`. |
| **Preserve user input** | Do not reset fields on validation failure. |
| **Use `useId` for IDs** | Avoid duplicate IDs in reusable components. |
| **Group related controls** | Use `<fieldset>` and `<legend>` for radio/checkbox groups. |
| **Support keyboard navigation** | Logical tab order; visible focus styles. |
| **Use a form library when justified** | `react-hook-form` for complex forms; avoid for simple ones. |
| **Test with assistive technology** | Verify screen-reader announcements and keyboard flows. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
import { useId, useState } from 'react';

function GoodForm() {
  const id = useId();
  const [values, setValues] = useState({ email: '' });
  const [errors, setErrors] = useState({});
  const [pending, setPending] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    const nextErrors = {};
    if (!values.email.includes('@')) nextErrors.email = 'Invalid email';
    if (Object.keys(nextErrors).length) {
      setErrors(nextErrors);
      return;
    }
    setPending(true);
    try {
      await fetch('/api/subscribe', {
        method: 'POST',
        body: JSON.stringify(values),
        headers: { 'Content-Type': 'application/json' },
      });
    } finally {
      setPending(false);
    }
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      <label htmlFor={id}>Email</label>
      <input
        id={id}
        value={values.email}
        onChange={(e) => setValues({ email: e.target.value })}
        aria-invalid={errors.email ? 'true' : undefined}
        aria-describedby={errors.email ? `${id}-error` : undefined}
      />
      {errors.email && (
        <p id={`${id}-error`} role="alert">{errors.email}</p>
      )}
      <button disabled={pending}>{pending ? 'Saving…' : 'Subscribe'}</button>
    </form>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ No label, placeholder only
<input placeholder="Email" />

// ❌ Controlled input without onChange
<input value={email} />

// ❌ Native reset without resetting React state
<button type="reset">Clear</button>

// ❌ Client-only validation
if (email.includes('@')) submit(); // No server check

// ❌ Colour-only error indication
<input style={{ borderColor: error ? 'red' : 'gray' }} />
```

**Expected Output:** The good practices produce an accessible, predictable form with managed pending state and server-ready validation. The bad practices cause inaccessible fields, read-only inputs, stale reset behaviour, and unannounced errors.

**Why this output:** Labels and ARIA attributes make the form accessible. `onChange` is required for controlled inputs. React state must be reset explicitly. Server validation is authoritative. Errors must be announced, not just coloured.

### Real-World Cases

- **All production forms:** These practices apply universally.
- **Accessibility audits:** Label, error, and focus requirements are common audit findings.
- **Design systems:** Form components encode these practices once for reuse.

### References

- W3C WAI — Forms Tutorial — https://www.w3.org/WAI/tutorials/forms/
- WebAIM — Creating Accessible Forms — https://webaim.org/techniques/forms/
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- react-hook-form — https://react-hook-form.com/
- MDN — Client-side form validation — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation

---

## References

- React — `<form>` — https://react.dev/reference/react-dom/components/form
- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — `<textarea>` — https://react.dev/reference/react-dom/components/textarea
- React — `<select>` — https://react.dev/reference/react-dom/components/select
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- React — Managing State — https://react.dev/learn/managing-state
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- React — useActionState — https://react.dev/reference/react/useActionState
- React — useFormStatus — https://react.dev/reference/react-dom/hooks/useFormStatus
- React — useId — https://react.dev/reference/react/useId
- MDN — Web forms — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms
- MDN — Client-side form validation — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation
- MDN — Constraint validation — https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation
- MDN — FormData — https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN — Sending form data — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data
- MDN — input event — https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event
- MDN — blur event — https://developer.mozilla.org/en-US/docs/Web/API/Element/blur_event
- MDN — focus event — https://developer.mozilla.org/en-US/docs/Web/API/Element/focus_event
- MDN — reset event — https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/reset_event
- MDN — ARIA: aria-describedby — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby
- W3C WAI — Forms Tutorial — https://www.w3.org/WAI/tutorials/forms/
- WebAIM — Creating Accessible Forms — https://webaim.org/techniques/forms/
- react-hook-form — https://react-hook-form.com/