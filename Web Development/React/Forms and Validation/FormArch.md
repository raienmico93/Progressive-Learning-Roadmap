# React Form Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React form architecture is the discipline of designing, structuring, and managing forms in React applications—covering input control, state synchronisation, validation, submission lifecycle, and the integration of native browser form capabilities with React's declarative model.

**Technical Definition:** React form architecture encompasses the patterns, APIs, and trade-offs involved in building forms in React. It distinguishes between **controlled inputs** (where React state drives the input's value and every keystroke triggers a state update and re-render) and **uncontrolled inputs** (where the DOM owns the value, and React reads it via refs or the `FormData` API on submission). It covers form state management (values, dirty/pristine status, touched fields), validation state tracking (synchronous rules, asynchronous checks, error and warning states), submission lifecycle management (submitting, success, error), and the integration of native HTML form elements with the `FormData` API. React 19 introduced **form Actions**, which allow `<form action={fn}>` to accept a function that receives `FormData`, and new hooks (`useActionState`, `useFormStatus`, `useOptimistic`) that manage pending states, results, and optimistic updates declaratively. The architectural choice between controlled and uncontrolled inputs, and between manual state management and a form library (React Hook Form, Formik, TanStack Form), determines the performance, complexity, and maintainability of the form.

**Beginner-Friendly Explanation:** A form is how users type information into your app—their name, email, password, search query. React gives you two ways to handle what they type. In the **controlled** approach, React holds the value in state and updates it on every keystroke, like a teacher watching every letter you write. In the **uncontrolled** approach, the browser holds the value, and React reads it only when you need it (on submit), like a teacher who collects your paper at the end. React 19 adds a third, more modern approach: **Actions**, where you hand the whole form to a function and React manages the pending state for you. Each approach has trade-offs, and this cheat sheet explains when to use which.

### Key Characteristics

- **Two Primary Input Models:** Controlled (state-driven) and uncontrolled (DOM-driven) inputs, with distinct performance and ergonomics trade-offs.
- **Native FormData Integration:** Modern React (especially React 19 Actions) embraces the browser's `FormData` API, allowing forms to work without JavaScript in some cases (progressive enhancement).
- **State Granularity:** Form state includes values, dirty/pristine status, touched fields, and validation state—each tracked separately.
- **Validation Layering:** Synchronous validation (required, pattern) and asynchronous validation (server checks, uniqueness) require different handling.
- **Submission Lifecycle:** Forms have distinct states—idle, submitting, success, error—each requiring UI feedback.
- **React 19 Actions:** Form submissions can be handled by async functions, with automatic pending states via `useFormStatus` and result states via `useActionState`.
- **Performance Considerations:** Controlled inputs re-render on every keystroke; uncontrolled inputs and libraries like React Hook Form minimise re-renders.

### Prerequisites

- Solid understanding of React function components, JSX, and props.
- Familiarity with the `useState`, `useRef`, and `useEffect` Hooks.
- Working knowledge of HTML form elements (`<form>`, `<input>`, `<select>`, `<textarea>`).
- Basic understanding of asynchronous JavaScript (promises, `async`/`await`).
- Awareness of the `FormData` API and browser form validation.

### Related Programming Areas

- **State Management:** Managing form values, validation, and submission state.
- **Validation:** Schema validation with Zod, Yup, or custom rules.
- **Accessibility:** Labelling inputs, announcing errors, managing focus.
- **Server Actions:** Handling form submissions on the server (Next.js, Remix).
- **UX Design:** Loading states, error messages, optimistic updates.

### Core Concepts / Features

1. Controlled Inputs vs. Uncontrolled Inputs
2. Native Form Elements and the FormData API
3. Form State Management (Values, Dirty/Pristine, Touched)
4. Validation State Tracking (Errors, Warnings, Async Pending)
5. Submission State Management (Submitting, Success, Error, `useFormStatus`)
6. React 19 Action Integration (`action`, `useActionState`, `useTransition`)

---

## Core Concept 1: Controlled Inputs vs. Uncontrolled Inputs

### Definitions

**Core Definition:** Controlled inputs are form elements whose value is driven by React state and updated via `onChange` handlers, while uncontrolled inputs are form elements whose value is owned by the DOM and accessed via refs or the `FormData` API.

**Technical Definition:** In a **controlled input**, React is the "single source of truth" for the input's value. The `value` prop is set from state, and an `onChange` handler updates that state on every keystroke. This causes a re-render on every keystroke but gives React full control over the value, enabling instant validation, conditional disabling, and value transformation. In an **uncontrolled input**, the DOM owns the value. React sets an initial value via `defaultValue` (or `defaultChecked` for checkboxes), and reads the current value on demand via a `ref` (`inputRef.current.value`) or by reading `FormData` on submission. Uncontrolled inputs do not re-render on keystrokes, making them more performant for large forms. React's documentation notes that in most cases, controlled components are recommended because they allow React to manage the form's state; however, uncontrolled components can be simpler when the form is simple and you only need the value on submission.

**Beginner-Friendly Explanation:** A controlled input is like a remote-controlled car—React holds the controller and decides exactly where the car goes on every keystroke. An uncontrolled input is like a wind-up toy—you set it going, and it runs on its own until you pick it up and read its position (on submit). Controlled gives you more control but requires more work; uncontrolled is simpler and faster but gives you less control while typing.

### Purposes

- **Controlled:** To synchronise input values with React state for instant validation, formatting, and conditional rendering.
- **Controlled:** To implement features like live character counts, input masking, and dependent fields.
- **Controlled:** To programmatically set or reset input values at any time.
- **Uncontrolled:** To reduce re-renders in forms with many fields or high-frequency typing.
- **Uncontrolled:** To leverage the browser's native form behaviour, including `FormData` and progressive enhancement.
- **Uncontrolled:** To integrate with third-party libraries that manipulate the DOM directly.

### Syntax Rules and Structure

**Controlled Input:**
```jsx
function ControlledForm() {
  const [name, setName] = useState('');

  function handleChange(e) {
    setName(e.target.value); // React owns the value
  }

  return (
    <form>
      <input value={name} onChange={handleChange} />
      <p>Hello, {name}!</p>
    </form>
  );
}
```

**Component Breakdown:**
- `value={name}`: The input's value is driven by React state.
- `onChange={handleChange}`: Every keystroke updates state.
- `setName(e.target.value)`: The state update triggers a re-render.

**Uncontrolled Input:**
```jsx
function UncontrolledForm() {
  const inputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Submitted: ${inputRef.current.value}`); // Read on demand
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Component Breakdown:**
- `ref={inputRef}`: A ref attached to the DOM element.
- `defaultValue=""`: The initial value (not controlled).
- `inputRef.current.value`: The current value read on submit.

**Uncontrolled with FormData:**
```jsx
function FormDataForm() {
  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    alert(`Name: ${formData.get('name')}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" defaultValue="" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Component Breakdown:**
- `name="name"`: The form field name used by `FormData`.
- `new FormData(e.target)`: Reads all named fields from the form.
- `formData.get('name')`: Retrieves the value of the `name` field.

**Syntax Rules:**
- Controlled inputs require both `value` and `onChange` (or `readOnly`).
- Uncontrolled inputs use `defaultValue` (not `value`) and a `name` attribute for `FormData`.
- Never mix: an input with `value` but no `onChange` is read-only (React will warn).
- Use `defaultChecked` for uncontrolled checkboxes and `checked` + `onChange` for controlled.
- Use `<textarea>` with `value` (controlled) or `defaultValue` (uncontrolled); never use `children`.
- Use `<select value={...}>` for controlled, `<select defaultValue={...}>` for uncontrolled.

**Constraints and Limitations:**
- Controlled inputs re-render on every keystroke, which can be slow for large forms.
- Uncontrolled inputs cannot easily implement live validation, character counts, or dependent fields.
- Uncontrolled inputs require refs or `FormData` to read values, which is less declarative.
- File inputs (`<input type="file" />`) are always uncontrolled; their value cannot be set programmatically.
- Mixing controlled and uncontrolled inputs in the same form is allowed but requires discipline.

### Annotated Code Examples

**Example 1: Controlled Form with Live Validation**

```jsx
import { useState } from 'react';

function ControlledSignup() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  // Live validation derived from state
  const emailValid = email.includes('@');
  const passwordValid = password.length >= 8;
  const formValid = emailValid && passwordValid;

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Signing up with ${email}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Email</label>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        {email && !emailValid && <span style={{ color: 'red' }}>Invalid email</span>}
      </div>

      <div>
        <label>Password</label>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        {password && !passwordValid && <span style={{ color: 'red' }}>Too short</span>}
      </div>

      <button type="submit" disabled={!formValid}>Sign Up</button>
    </form>
  );
}
```

**Expected Output:** As the user types, validation messages appear below each field if the input is invalid. The "Sign Up" button is disabled until both fields are valid.

**Why This Output Occurs:** Because the inputs are controlled, React re-renders on every keystroke. The validation logic runs during render, deriving `emailValid` and `passwordValid` from the current state. The button's `disabled` prop reflects `formValid`. No `useEffect` is needed—validation is derived, not synced.

**Example 2: Uncontrolled Form with FormData**

```jsx
function UncontrolledSignup() {
  function handleSubmit(e) {
    e.preventDefault();

    const formData = new FormData(e.target);
    const email = formData.get('email');
    const password = formData.get('password');
    
    alert(`Signing up with ${email}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Email</label>
        <input type="email" name="email" required />
      </div>

      <div>
        <label>Password</label>
        <input type="password" name="password" minLength={8} required />
      </div>

      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Expected Output:** The form uses the browser's native validation (`required`, `minLength`). Submitting with invalid data shows browser-native validation messages. Submitting with valid data reads the values via `FormData` and shows an alert.

**Why This Output Occurs:** The inputs are uncontrolled—the DOM owns their values. No React state is involved, so no re-renders occur during typing. On submit, `new FormData(e.target)` reads all named fields. The browser's native validation runs before the submit handler if the form has `required` or `minLength` attributes.

### Real-World Cases

- **Controlled:** Live search inputs, character counters, masked inputs (phone, credit card), dependent dropdowns.
- **Uncontrolled:** Simple login forms, forms with many fields where performance matters, forms that rely on native browser validation.
- **Hybrid:** Large forms where most fields are uncontrolled but a few (e.g., a live preview field) are controlled.

### References

- React Official Documentation – Reacting to Input with State: https://react.dev/learn/reacting-to-input-with-state
- React Official Documentation – `<input>`: https://react.dev/reference/react-dom/components/input
- React Official Documentation – `<select>`: https://react.dev/reference/react-dom/components/select
- React Official Documentation – `<textarea>`: https://react.dev/reference/react-dom/components/textarea
- React Official Documentation – Manipulating the DOM with Refs: https://react.dev/learn/manipulating-the-dom-with-refs

---

## Core Concept 2: Native Form Elements and the FormData API

### Definitions

**Core Definition:** The FormData API is a browser interface that constructs a set of key/value pairs representing form fields and their values, providing a standard way to read form data without manually querying each input.

**Technical Definition:** `FormData` is a Web API that collects form field names and values into an iterable set of key/value pairs. Constructing `new FormData(formElement)` reads all form controls with a `name` attribute, including text inputs, checkboxes, radio buttons, selects, textareas, and file inputs. `FormData` supports `get(name)`, `getAll(name)`, `has(name)`, `set(name, value)`, `append(name, value)`, `delete(name)`, and `entries()` for reading and manipulating data. It is the native format for `fetch()` request bodies (with `Content-Type: multipart/form-data` or `application/x-www-form-urlencoded`). React 19 integrates `FormData` directly into form Actions: when a `<form action={fn}>` is submitted, `fn` receives a `FormData` instance as its first argument. This makes `FormData` the bridge between native HTML forms and React's declarative model, enabling progressive enhancement—forms work without JavaScript if the server handles the submission.

**Beginner-Friendly Explanation:** `FormData` is like a clipboard that collects everything from a form. Instead of asking each input "what's your value?" one by one, you hand the form to `FormData`, and it reads every named field automatically. It's especially useful when you have many fields or file uploads, and it's the standard way to send form data to a server.

### Purposes

- To read all form field values in a single operation without querying each input individually.
- To handle file uploads natively via `<input type="file" />`.
- To send form data to a server via `fetch()` with the correct content type.
- To enable progressive enhancement—forms that work without JavaScript.
- To integrate with React 19 form Actions, which receive `FormData` automatically.
- To support multiple values for the same field name (e.g., checkboxes, multi-selects).

### Syntax Rules and Structure

**Reading FormData on Submit:**
```jsx
function FormDataExample() {
  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);

    // Read single values
    const name = formData.get('name');        // string | null
    const email = formData.get('email');

    // Read multiple values (checkboxes, multi-selects)
    const interests = formData.getAll('interests'); // string[]

    // Iterate all entries
    for (const [key, value] of formData.entries()) {
      console.log(key, value);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" />
      <input name="email" type="email" />
      <label><input type="checkbox" name="interests" value="coding" /> Coding</label>
      <label><input type="checkbox" name="interests" value="music" /> Music</label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Component Breakdown:**
- `new FormData(e.target)`: Constructs FormData from the form element.
- `formData.get('name')`: Returns the first value for the key, or `null`.
- `formData.getAll('interests')`: Returns all values for the key as an array.
- `formData.entries()`: Returns an iterator of `[key, value]` pairs.

**Sending FormData to a Server:**
```jsx
async function handleSubmit(e) {
  e.preventDefault();
  const formData = new FormData(e.target);

  const response = await fetch('/api/submit', {
    method: 'POST',
    body: formData, // Content-Type is set automatically
  });

  const result = await response.json();
}
```

**Component Breakdown:**
- `body: formData`: `fetch` sets `Content-Type: multipart/form-data` with the boundary automatically.
- No need to manually serialise or set headers.

**React 19 Form Action:**
```jsx
async function submitAction(formData) {
  const name = formData.get('name');
  await saveToServer({ name });
}

<form action={submitAction}>
  <input name="name" />
  <button type="submit">Submit</button>
</form>
```

**Component Breakdown:**
- `action={submitAction}`: React passes the FormData to the action.
- `formData.get('name')`: Reads the value inside the action.

**Syntax Rules:**
- Every form control that should be included in `FormData` must have a `name` attribute.
- Use `formData.get(name)` for single values; `formData.getAll(name)` for multiple.
- Checkboxes and radio buttons only appear in `FormData` if they are checked.
- File inputs are supported natively; `formData.get('file')` returns a `File` object.
- `FormData` values are always strings or `File` objects; parse numbers and booleans explicitly.
- When passing `FormData` to `fetch`, do not set `Content-Type` manually—the browser sets it with the correct boundary.

**Constraints and Limitations:**
- `FormData` only includes form controls with a `name` attribute.
- Disabled fields are excluded from `FormData`.
- `FormData` cannot represent nested objects; flatten or serialise manually if needed.
- `FormData` values are strings; type coercion is the developer's responsibility.
- `get()` returns `null` for missing keys; `getAll()` returns an empty array.

### Annotated Code Example: Full FormData Usage

```jsx
function RegistrationForm() {
  async function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);

    // Extract and type-cast values
    const data = {
      name: formData.get('name'),
      email: formData.get('email'),
      age: Number(formData.get('age')),
      newsletter: formData.get('newsletter') === 'on',
      interests: formData.getAll('interests'),
    };

    console.log('Submitting:', data);

    // Send to server
    const response = await fetch('/api/register', {
      method: 'POST',
      body: formData,
    });

    if (response.ok) {
      alert('Registered successfully!');
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name</label>
        <input name="name" required />
      </div>

      <div>
        <label>Email</label>
        <input name="email" type="email" required />
      </div>

      <div>
        <label>Age</label>
        <input name="age" type="number" />
      </div>

      <div>
        <label>
          <input type="checkbox" name="newsletter" /> Subscribe to newsletter
        </label>
      </div>

      <fieldset>
        <legend>Interests</legend>
        <label><input type="checkbox" name="interests" value="coding" /> Coding</label>
        <label><input type="checkbox" name="interests" value="music" /> Music</label>
        <label><input type="checkbox" name="interests" value="sports" /> Sports</label>
      </fieldset>

      <button type="submit">Register</button>
    </form>
  );
}
```

**Expected Output:** The form displays name, email, age, newsletter checkbox, and interest checkboxes. Submitting logs an object with typed values and sends the `FormData` to the server. The `interests` field is an array of checked values.

**Why This Output Occurs:** `FormData` reads all named fields. `get()` returns strings, so `Number()` converts age, and the newsletter checkbox is checked by comparing to `'on'` (the default value for checked checkboxes). `getAll('interests')` returns an array of all checked interest values. The `FormData` object is sent directly to `fetch`, which sets the correct multipart content type.

### Real-World Cases

- **File uploads:** Sending images or documents to a server without manually reading file contents.
- **Multi-field forms:** Registration, checkout, and survey forms with many fields.
- **Progressive enhancement:** Forms that work without JavaScript when submitted to a server action.
- **React 19 Actions:** Passing `FormData` to async actions for server-side processing.
- **Multi-value fields:** Checkbox groups, multi-selects, and tag inputs.

### References

- MDN Web Docs – FormData: https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs – Using FormData Objects: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects
- React Official Documentation – `<form>`: https://react.dev/reference/react-dom/components/form
- React Official Documentation – Form Actions: https://react.dev/reference/react-dom/components/form#form-actions

---

## Core Concept 3: Form State Management (Values, Dirty/Pristine, Touched)

### Definitions

**Core Definition:** Form state management is the tracking of a form's values, dirty/pristine status (whether the form has been modified), and touched fields (whether a field has been focused and blurred), enabling validation timing and submission UX.

**Technical Definition:** Form state consists of several distinct pieces. **Values** are the current content of each field. **Dirty/pristine** status indicates whether the form's values differ from their initial values (`isDirty` is `true` if any field has changed). **Touched fields** track which fields the user has interacted with (focused and then blurred), enabling "validate on blur" strategies that avoid showing errors before the user has finished typing. **Dirty fields** track which specific fields have changed from their initial values, useful for partial updates. Libraries like React Hook Form expose these via `formState` (`isDirty`, `dirtyFields`, `touchedFields`, `isValid`, `isSubmitting`). Manual management with `useState` requires separate state for values, dirty flags, and touched flags, which becomes verbose as the form grows. The recommended approach is to use a form library for anything beyond trivial forms, or to derive dirty/pristine status from a comparison with initial values during render.

**Beginner-Friendly Explanation:** Imagine filling out a paper form. **Values** are what you've written in each box. **Dirty** means you've changed something from the original (e.g., a pre-filled address you edited). **Touched** means you've interacted with a field—you clicked into it and then clicked away. Knowing whether a field is touched helps you decide when to show an error: showing "email is required" before the user has even clicked the email field is annoying; showing it after they've typed and left the field is helpful.

### Purposes

- To track the current value of every form field.
- To determine whether the form has been modified (`isDirty`) and warn users before losing unsaved changes.
- To determine which specific fields have been modified (`dirtyFields`) for partial updates.
- To track which fields have been interacted with (`touchedFields`) to control validation timing.
- To enable "validate on blur" or "validate on submit" strategies that improve UX.
- To derive form validity during render without storing redundant state.

### Syntax Rules and Structure

**Manual State Management (Simple Forms):**
```jsx
function ManualForm() {
  const [values, setValues] = useState({ email: '', password: '' });
  const [touched, setTouched] = useState({ email: false, password: false });

  const initialValues = { email: '', password: '' };
  const isDirty = JSON.stringify(values) !== JSON.stringify(initialValues);

  function handleChange(e) {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  }

  function handleBlur(e) {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  }

  const errors = {
    email: values.email.includes('@') ? null : 'Invalid email',
    password: values.password.length >= 8 ? null : 'Too short',
  };

  return (
    <form>
      <input
        name="email"
        value={values.email}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {touched.email && errors.email && <span>{errors.email}</span>}

      <input
        name="password"
        type="password"
        value={values.password}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {touched.password && errors.password && <span>{errors.password}</span>}

      <button disabled={!isDirty}>Save</button>
    </form>
  );
}
```

**Component Breakdown:**
- `values`: An object holding all field values.
- `touched`: An object tracking which fields have been blurred.
- `isDirty`: Derived by comparing `values` to `initialValues`.
- `errors`: Derived from `values` during render.
- Errors are shown only when the field is touched.

**React Hook Form (Recommended for Complex Forms):**
```jsx
import { useForm } from 'react-hook-form';

function RHFForm() {
  const {
    register,
    handleSubmit,
    formState: {
      isDirty,
      dirtyFields,
      touchedFields,
      errors,
      isValid,
    },
  } = useForm({
    defaultValues: { email: '', password: '' },
    mode: 'onBlur', // Validate on blur
  });

  return (
    <form onSubmit={handleSubmit(data => console.log(data))}>
      <input {...register('email', { required: 'Email is required' })} />
      {touchedFields.email && errors.email && <span>{errors.email.message}</span>}

      <input
        type="password"
        {...register('password', { minLength: { value: 8, message: 'Too short' } })}
      />
      {touchedFields.password && errors.password && <span>{errors.password.message}</span>}

      <button type="submit" disabled={!isDirty || !isValid}>Save</button>
    </form>
  );
}
```

**Component Breakdown:**
- `isDirty`: `true` if any field has changed from its default value.
- `dirtyFields`: An object with `true` for each changed field.
- `touchedFields`: An object with `true` for each blurred field.
- `errors`: Validation errors keyed by field name.
- `isValid`: `true` if the form passes all validation rules.
- `mode: 'onBlur'`: Validation runs on blur, not on every keystroke.

**Syntax Rules:**
- Always provide `defaultValues` so the library can determine dirty status.
- Use `dirtyFields` for partial updates (only send changed fields).
- Use `touchedFields` to decide when to show errors (avoid showing errors before interaction).
- Derive `isValid` from validation rules; do not store it in separate state.
- Choose a validation mode: `onSubmit` (default), `onBlur`, `onChange`, `onTouched`, or `all`.
- For manual forms, derive dirty status by comparing to a stable `initialValues` reference.

**Constraints and Limitations:**
- `JSON.stringify` comparison is fragile for objects with different key orders or non-serialisable values.
- Manual state management becomes verbose as the form grows; use a library for complex forms.
- `touchedFields` only becomes `true` after a blur event; fields focused and never blurred remain untouched.
- `isDirty` does not reset automatically; call `reset()` after successful submission.
- React Hook Form's `formState` is a Proxy; reading properties subscribes to them, so read only what you need.

### Annotated Code Example: React Hook Form with Dirty and Touched Tracking

```jsx
import { useForm } from 'react-hook-form';

function ProfileForm() {
  const {
    register,
    handleSubmit,
    reset,
    formState: { isDirty, dirtyFields, touchedFields, errors, isSubmitting },
  } = useForm({
    defaultValues: { name: 'Alice', email: 'alice@example.com' },
    mode: 'onBlur',
  });

  async function onSubmit(data) {
    await new Promise(r => setTimeout(r, 1000)); // Simulate API call
    alert(`Saved: ${JSON.stringify(data)}`);
    reset(data); // Reset dirty state after save
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Name</label>
        <input {...register('name', { required: 'Name is required' })} />
        {touchedFields.name && errors.name && (
          <span style={{ color: 'red' }}>{errors.name.message}</span>
        )}
      </div>

      <div>
        <label>Email</label>
        <input
          {...register('email', {
            required: 'Email is required',
            pattern: { value: /@/, message: 'Invalid email' },
          })}
        />
        {touchedFields.email && errors.email && (
          <span style={{ color: 'red' }}>{errors.email.message}</span>
        )}
      </div>

      <p>Dirty: {isDirty ? 'Yes' : 'No'}</p>
      <p>Dirty fields: {Object.keys(dirtyFields).join(', ') || 'none'}</p>

      <button type="submit" disabled={!isDirty || isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save'}
      </button>
    </form>
  );
}
```

**Expected Output:** A profile form pre-filled with "Alice" and "alice@example.com". Editing a field marks it as dirty and enables the Save button. Blurring an invalid field shows its error. Saving shows "Saving...", then alerts the data and resets the dirty state.

**Why This Output Occurs:** `defaultValues` provides the initial values, so React Hook Form knows when a field changes. `dirtyFields` tracks which specific fields changed. `touchedFields` becomes `true` after a blur, controlling when errors appear. `reset(data)` updates the baseline to the saved data, so `isDirty` becomes `false` after saving.

### Real-World Cases

- **Profile editing:** Pre-filled forms where only changed fields are sent to the server.
- **Unsaved changes warning:** Using `isDirty` to block navigation or show a confirmation dialog.
- **Multi-step forms:** Tracking which steps are complete and which fields are dirty.
- **Settings panels:** Tracking touched fields to decide when to auto-save.
- **Admin CRUD forms:** Partial updates using `dirtyFields`.

### References

- React Hook Form – formState: https://react-hook-form.com/docs/useform/formstate
- React Hook Form – useForm: https://react-hook-form.com/docs/useform
- React Official Documentation – Reacting to Input with State: https://react.dev/learn/reacting-to-input-with-state

---

## Core Concept 4: Validation State Tracking (Errors, Warnings, Async Pending)

### Definitions

**Core Definition:** Validation state tracking is the process of evaluating form inputs against rules (required, pattern, range, custom), tracking the resulting error and warning messages, and handling asynchronous validation checks with pending states.

**Technical Definition:** Validation state in a form consists of **errors** (blocking issues that prevent submission), **warnings** (non-blocking advisories), and **pending** states (asynchronous validation in progress). Validation can be **synchronous** (required fields, regex patterns, min/max length, custom synchronous functions) or **asynchronous** (server-side uniqueness checks, username availability, CAPTCHA verification). Synchronous validation runs during render or on change/blur and produces immediate feedback. Asynchronous validation requires debouncing, cancellation of stale requests (via `AbortController` or request IDs), and a pending state to inform the user that a check is in progress. Libraries like React Hook Form support both synchronous rules (via `register` options) and asynchronous validation (via async `validate` functions or schema resolvers like Zod). The validation mode (`onChange`, `onBlur`, `onSubmit`, `onTouched`, `all`) determines when validation runs. Errors should be displayed accessibly with `aria-invalid` and `aria-describedby`, and focus should move to the first invalid field on submission failure.

**Beginner-Friendly Explanation:** Validation is like a spell-checker for your form. Some checks are instant—"this field is required," "this email doesn't have an @ symbol." Others take time—"is this username already taken?" For the instant checks, you can show errors as the user types or when they leave a field. For the slow checks, you need to wait for the server and show a "checking..." message. You also need to be careful: if the user types quickly, an old check might come back after a new one, so you need to ignore stale results.

### Purposes

- To prevent invalid data from being submitted to the server.
- To provide immediate, helpful feedback to users as they fill out the form.
- To handle asynchronous validation (server checks) without blocking the UI.
- To distinguish between blocking errors and non-blocking warnings.
- To track pending validation state so users know a check is in progress.
- To display errors accessibly for screen-reader users.

### Syntax Rules and Structure

**Synchronous Validation with React Hook Form:**
```jsx
<input
  {...register('email', {
    required: 'Email is required',
    pattern: { value: /^\S+@\S+$/, message: 'Invalid email format' },
    minLength: { value: 5, message: 'Too short' },
  })}
/>
```

**Custom Synchronous Validation:**
```jsx
<input
  {...register('password', {
    validate: {
      hasUppercase: (v) => /[A-Z]/.test(v) || 'Must contain an uppercase letter',
      hasNumber: (v) => /\d/.test(v) || 'Must contain a number',
      notCommon: (v) =>
        !['password', '12345678'].includes(v) || 'Too common',
    },
  })}
/>
```

**Asynchronous Validation with React Hook Form:**
```jsx
<input
  {...register('username', {
    validate: {
      available: async (value) => {
        const response = await fetch(`/api/check-username?u=${value}`);
        const { available } = await response.json();
        return available || 'Username is already taken';
      },
    },
  })}
/>
```

**Schema Validation with Zod:**
```jsx
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'At least 8 characters'),
  confirm: z.string(),
}).refine((data) => data.password === data.confirm, {
  message: 'Passwords do not match',
  path: ['confirm'],
});

const { register, formState: { errors } } = useForm({
  resolver: zodResolver(schema),
});
```

**Displaying Errors Accessibly:**
```jsx
<div>
  <label htmlFor="email">Email</label>
  <input
    id="email"
    aria-invalid={!!errors.email}
    aria-describedby={errors.email ? 'email-error' : undefined}
    {...register('email', { required: 'Email is required' })}
  />
  {errors.email && (
    <span id="email-error" role="alert" style={{ color: 'red' }}>
      {errors.email.message}
    </span>
  )}
</div>
```

**Async Pending State (Manual):**
```jsx
function useAsyncValidation(value) {
  const [state, setState] = useState({ status: 'idle', message: null });

  useEffect(() => {
    if (!value) return;
    let ignore = false;
    setState({ status: 'checking', message: null });

    const timer = setTimeout(async () => {
      const res = await fetch(`/api/check?u=${value}`);
      const { available } = await res.json();
      if (!ignore) {
        setState({
          status: available ? 'valid' : 'invalid',
          message: available ? null : 'Already taken',
        });
      }
    }, 500); // Debounce

    return () => {
      ignore = true;
      clearTimeout(timer);
    };
  }, [value]);

  return state;
}
```

**Component Breakdown:**
- `status`: `'idle'`, `'checking'`, `'valid'`, or `'invalid'`.
- `ignore` flag: Prevents stale results from updating state.
- `setTimeout`: Debounces the request by 500ms.
- `clearTimeout`: Cancels the pending request on cleanup.

**Syntax Rules:**
- Use synchronous validation for required, pattern, length, and range checks.
- Use asynchronous validation for server-side checks (uniqueness, availability).
- Always debounce asynchronous validation to avoid a request per keystroke.
- Always cancel or ignore stale asynchronous results using `AbortController` or an `ignore` flag.
- Use `aria-invalid` and `aria-describedby` for accessible error display.
- Use `role="alert"` on error messages so screen readers announce them.
- Move focus to the first invalid field on submit failure.
- Use a schema resolver (Zod, Yup) for complex cross-field validation.

**Constraints and Limitations:**
- Asynchronous validation adds latency; show a pending indicator.
- Race conditions are common with async validation; always guard against stale results.
- `validate` functions in React Hook Form run on every validation cycle; keep them fast.
- Schema validation adds a dependency but provides type safety and reusable rules.
- Warnings are not natively supported by all libraries; implement them as custom validation that does not block submission.

### Annotated Code Example: Async Username Validation with Pending State

```jsx
import { useState, useEffect } from 'react';
import { useForm } from 'react-hook-form';

function UsernameForm() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors, isValidating },
  } = useForm({ mode: 'onChange' });

  const username = watch('username');

  const { register: registerField, ...rest } = register('username', {
    required: 'Username is required',
    minLength: { value: 3, message: 'At least 3 characters' },
    validate: {
      available: async (value) => {
        const res = await fetch(`/api/check-username?u=${value}`);
        const { available } = await res.json();
        return available || 'Username is already taken';
      },
    },
  });

  return (
    <form onSubmit={handleSubmit(data => alert(`Welcome, ${data.username}!`))}>
      <div>
        <label htmlFor="username">Username</label>
        <input
          id="username"
          aria-invalid={!!errors.username}
          aria-describedby={errors.username ? 'username-error' : undefined}
          {...registerField}
        />

        {isValidating && <span>Checking availability...</span>}

        {errors.username && (
          <span id="username-error" role="alert" style={{ color: 'red' }}>
            {errors.username.message}
          </span>
        )}
      </div>

      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Expected Output:** As the user types a username, after a brief delay, "Checking availability..." appears, then either the error "Username is already taken" or no error. Submitting with a valid username shows an alert.

**Why This Output Occurs:** React Hook Form's `validate.available` function runs asynchronously. While it is running, `isValidating` is `true`, showing the pending indicator. When the promise resolves, either the error is cleared (available) or set (taken). The `aria-invalid` and `aria-describedby` attributes make the error accessible.

### Real-World Cases

- **Signup forms:** Checking username and email availability.
- **Checkout:** Validating coupon codes and shipping addresses.
- **Banking:** Validating account numbers and routing numbers.
- **Surveys:** Conditional validation based on previous answers.
- **Multi-step forms:** Validating each step before allowing progression.

### References

- React Hook Form – Validation: https://react-hook-form.com/get-started#Applyvalidation
- React Hook Form – register: https://react-hook-form.com/docs/useform/register
- React Hook Form – Resolvers: https://react-hook-form.com/docs/useform#resolver
- Zod – Documentation: https://zod.dev/
- WAI-ARIA Authoring Practices – Form Validation: https://www.w3.org/WAI/ARIA/apg/

---

## Core Concept 5: Submission State Management (Submitting, Success, Error, `useFormStatus`)

### Definitions

**Core Definition:** Submission state management is the tracking and presentation of a form's lifecycle during submission—idle, submitting, success, and error—including disabling inputs, showing spinners, and displaying success or error messages.

**Technical Definition:** Form submission involves several states. **Idle** is the initial state before submission. **Submitting** is active while the submission request is in flight; during this state, the submit button should be disabled to prevent double submission, and a spinner or "Submitting..." message should be shown. **Success** indicates the submission completed successfully; the form may be reset, a success message shown, or the user redirected. **Error** indicates the submission failed; error messages should be displayed, and the form should remain editable. React 19's `useFormStatus` Hook provides `{ pending, data, method, action }` for child components of a form, allowing submit buttons to read the pending state without prop drilling. `useActionState` provides `[state, formAction, isPending]` for managing the action's result state. In React Hook Form, `formState.isSubmitting`, `isSubmitted`, `isSubmitSuccessful`, and `submitCount` provide equivalent information. A common pattern is to disable the submit button during submission, show a spinner, and display a success or error banner after completion.

**Beginner-Friendly Explanation:** Submitting a form is like mailing a letter. While it's in transit (submitting), you can't do much—the button is disabled so you don't send it twice. When it arrives (success), you get a confirmation. If it bounces back (error), you get a message and can try again. `useFormStatus` is like a tracking number that any part of the form can check to see if the letter is still in transit.

### Purposes

- To prevent double submission by disabling the submit button while the request is in flight.
- To provide visual feedback (spinner, "Submitting...") during the request.
- To display success messages and reset the form after successful submission.
- To display error messages and keep the form editable after failed submission.
- To track submission count and reset state for repeated submissions.
- To enable progressive enhancement with `useFormStatus` in React 19.

### Syntax Rules and Structure

**React Hook Form Submission State:**
```jsx
const {
  handleSubmit,
  formState: { isSubmitting, isSubmitted, isSubmitSuccessful, submitCount, errors },
} = useForm();

<form onSubmit={handleSubmit(onSubmit)}>
  <button type="submit" disabled={isSubmitting}>
    {isSubmitting ? 'Submitting...' : 'Submit'}
  </button>
  {isSubmitted && isSubmitSuccessful && <p>Success!</p>}
</form>
```

**Component Breakdown:**
- `isSubmitting`: `true` while the async submit handler is running.
- `isSubmitted`: `true` after the form has been submitted (even if invalid).
- `isSubmitSuccessful`: `true` if the last submission succeeded.
- `submitCount`: The number of submission attempts.

**React 19 `useFormStatus`:**
```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

function MyForm() {
  async function handleSubmit(formData) {
    await saveToServer(formData);
  }

  return (
    <form action={handleSubmit}>
      <input name="email" />
      <SubmitButton />
    </form>
  );
}
```

**Component Breakdown:**
- `useFormStatus()`: Must be called inside a component rendered within a `<form>`.
- `pending`: `true` while the action is running.
- `data`: The `FormData` being submitted (or `null`).
- `method`: `'get'` or `'post'`.
- `action`: The action function or URL.

**React 19 `useActionState`:**
```jsx
import { useActionState } from 'react';

async function submitAction(prevState, formData) {
  const email = formData.get('email');
  if (!email) {
    return { error: 'Email is required' };
  }
  await saveToServer({ email });
  return { success: true, message: 'Subscribed!' };
}

function NewsletterForm() {
  const [state, formAction, isPending] = useActionState(submitAction, null);

  return (
    <form action={formAction}>
      <input name="email" />
      {state?.error && <p style={{ color: 'red' }}>{state.error}</p>}
      {state?.success && <p style={{ color: 'green' }}>{state.message}</p>}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Subscribing...' : 'Subscribe'}
      </button>
    </form>
  );
}
```

**Component Breakdown:**
- `useActionState(action, initialState)`: Returns `[state, formAction, isPending]`.
- `state`: The action's return value (starts as `initialState`).
- `formAction`: The function to pass to `<form action={...}>`.
- `isPending`: `true` while the action is running.

**Syntax Rules:**
- Use `isSubmitting` (React Hook Form) or `pending`/`isPending` (React 19) to disable the submit button.
- Always disable the submit button during submission to prevent double submission.
- Display a spinner or "Submitting..." message during submission.
- Reset the form after successful submission (React Hook Form's `reset()`, or React 19's automatic reset for uncontrolled forms).
- Display success and error messages after submission completes.
- `useFormStatus` must be called in a child component of the `<form>`, not in the component that renders the `<form>`.
- `useActionState` replaces the deprecated `useFormState` from React 18.

**Constraints and Limitations:**
- `isSubmitting` in React Hook Form only works with async `onSubmit` handlers; synchronous handlers do not trigger it.
- `useFormStatus` reads the status of the nearest parent `<form>`, so it cannot be used in the same component that renders the `<form>`.
- React 19 automatically resets uncontrolled forms after a successful action; controlled forms are not reset automatically.
- Errors from actions must be returned as state, not thrown, unless using an Error Boundary.

### Annotated Code Example: React 19 Form with useActionState and useFormStatus

```jsx
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

// Submit button reads pending state via useFormStatus
function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Subscribe'}
    </button>
  );
}

// Action function
async function subscribeAction(prevState, formData) {
  const email = formData.get('email');

  if (!email || !email.includes('@')) {
    return { error: 'Please enter a valid email address.' };
  }

  // Simulate API call
  await new Promise((r) => setTimeout(r, 1000));

  return { success: true, message: `Subscribed ${email}!` };
}

export default function NewsletterForm() {
  const [state, formAction, isPending] = useActionState(subscribeAction, null);

  return (
    <form action={formAction}>
      <h2>Newsletter</h2>

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          aria-invalid={!!state?.error}
          aria-describedby={state?.error ? 'email-error' : undefined}
        />
      </div>

      {state?.error && (
        <p id="email-error" role="alert" style={{ color: 'red' }}>
          {state.error}
        </p>
      )}

      {state?.success && (
        <p role="status" style={{ color: 'green' }}>
          {state.message}
        </p>
      )}

      <SubmitButton />
    </form>
  );
}
```

**Expected Output:** A newsletter form with an email input and "Subscribe" button. Submitting with an invalid email shows "Please enter a valid email address." Submitting with a valid email shows "Submitting..." on the button, then "Subscribed [email]!" and resets the form.

**Why This Output Occurs:** `useActionState` wraps the `subscribeAction`, providing `state` (the action's return value) and `isPending`. The `SubmitButton` child reads `pending` via `useFormStatus` and disables itself. After a successful action, React automatically resets the uncontrolled form. The `state` object is updated with either `error` or `success`, which drives the conditional rendering of messages.

### Real-World Cases

- **Newsletter signup:** Simple form with email validation and success message.
- **Contact forms:** Name, email, message with submission feedback.
- **Login forms:** Email and password with error display and redirect on success.
- **Multi-step wizards:** Per-step submission with pending and error states.
- **File uploads:** Progress indication during upload (using `useFormStatus`).

### References

- React Official Documentation – `useFormStatus`: https://react.dev/reference/react-dom/hooks/useFormStatus
- React Official Documentation – `useActionState`: https://react.dev/reference/react/useActionState
- React Official Documentation – `<form>` Actions: https://react.dev/reference/react-dom/components/form
- React Hook Form – formState: https://react-hook-form.com/docs/useform/formstate

---

## Core Concept 6: React 19 Action Integration (`action`, `useActionState`, `useTransition`)

### Definitions

**Core Definition:** React 19 Actions are functions passed to the `<form action={...}>` prop (or to `useActionState`) that handle form submissions, receiving `FormData` and returning a result, with React managing pending states, automatic form resets, and optimistic updates.

**Technical Definition:** React 19 introduced **Actions** as a first-class way to handle form submissions and other asynchronous transitions. An Action is an async function passed to the `action` prop of a `<form>` element. When the form is submitted, React calls the Action with the `FormData` and manages the submission lifecycle: it sets the form's pending state (`useFormStatus().pending`), disables the form during submission (if using an uncontrolled form), and resets the form after a successful Action. The `useActionState` Hook wraps an Action to provide a result state (`state`), a wrapped action (`formAction`), and a pending flag (`isPending`). The `useTransition` Hook provides `[isPending, startTransition]` for marking non-urgent updates, and `useOptimistic` allows optimistic UI updates during Action execution. Actions can be **client actions** (functions defined in the client) or **server actions** (functions marked with `'use server'` in frameworks like Next.js). Actions integrate with `<form>`, `<button formAction>`, and `<input formAction>` for multiple submission targets. React 19's Actions work with progressive enhancement: if the form is submitted before hydration, the browser submits it natively to the server.

**Beginner-Friendly Explanation:** An Action is like giving your form a to-do list. Instead of writing an `onSubmit` handler that manually sets `isSubmitting` to `true`, calls the API, sets it back to `false`, and handles errors, you just write a function that does the work, and React handles the rest. React knows when the function is running (so it can disable the button), knows when it succeeds (so it can reset the form), and knows if it fails (so it can show an error). It's like having a personal assistant for form submissions.

### Purposes

- To simplify form submission by eliminating manual pending-state management.
- To integrate forms with server actions for full-stack data mutations.
- To enable progressive enhancement—forms work without JavaScript.
- To manage the result of an action (success, error, data) with `useActionState`.
- To provide pending states for buttons and inputs with `useFormStatus`.
- To perform optimistic updates during action execution with `useOptimistic`.
- To coordinate multiple actions on the same form with `formAction` on buttons.

### Syntax Rules and Structure

**Basic Form Action:**
```jsx
async function handleSubmit(formData) {
  const email = formData.get('email');
  await subscribe(email);
}

<form action={handleSubmit}>
  <input name="email" />
  <button type="submit">Subscribe</button>
</form>
```

**Component Breakdown:**
- `action={handleSubmit}`: React calls `handleSubmit` with `FormData` on submit.
- React manages the pending state automatically.
- The form is reset after a successful submission.

**`useActionState` for Result State:**
```jsx
import { useActionState } from 'react';

async function action(prevState, formData) {
  const email = formData.get('email');
  if (!email) return { error: 'Email required' };
  await save(email);
  return { success: true };
}

function Form() {
  const [state, formAction, isPending] = useActionState(action, null);

  return (
    <form action={formAction}>
      <input name="email" />
      {state?.error && <p>{state.error}</p>}
      <button disabled={isPending}>{isPending ? 'Saving...' : 'Save'}</button>
    </form>
  );
}
```

**Component Breakdown:**
- `useActionState(action, initialState)`: Returns `[state, formAction, isPending]`.
- `state`: The action's return value (starts as `initialState`).
- `formAction`: The wrapped action to pass to `<form action={formAction}>`.
- `isPending`: `true` while the action is running.

**`useFormStatus` for Submit Buttons:**
```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Saving...' : 'Save'}</button>;
}
```

**Component Breakdown:**
- Must be a child of the `<form>`.
- `pending`: `true` while the form's action is running.

**`useTransition` for Non-Form Actions:**
```jsx
import { useTransition } from 'react';

function SearchBox() {
  const [isPending, startTransition] = useTransition();
  const [query, setQuery] = useState('');

  function handleSearch() {
    startTransition(async () => {
      await search(query);
    });
  }

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <button onClick={handleSearch} disabled={isPending}>
        {isPending ? 'Searching...' : 'Search'}
      </button>
    </div>
  );
}
```

**Component Breakdown:**
- `useTransition()`: Returns `[isPending, startTransition]`.
- `startTransition(async () => { ... })`: Marks the async work as a transition.
- `isPending`: `true` while the transition is running.

**`useOptimistic` for Optimistic Updates:**
```jsx
import { useOptimistic } from 'react';

function TodoList({ todos, addTodo }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, { ...newTodo, pending: true }]
  );

  async function handleAdd(formData) {
    const text = formData.get('text');
    addOptimisticTodo({ text }); // Instant UI update
    await addTodo(text);          // Server call
  }

  return (
    <div>
      <ul>
        {optimisticTodos.map((todo, i) => (
          <li key={i} style={{ opacity: todo.pending ? 0.5 : 1 }}>
            {todo.text}
          </li>
        ))}
      </ul>
      <form action={handleAdd}>
        <input name="text" />
        <button type="submit">Add</button>
      </form>
    </div>
  );
}
```

**Component Breakdown:**
- `useOptimistic(todos, reducer)`: Returns the optimistic state and a function to add optimistic updates.
- `addOptimisticTodo({ text })`: Immediately adds the todo to the UI.
- `await addTodo(text)`: The actual server call; if it succeeds, the optimistic update is replaced by the real data.

**Syntax Rules:**
- Pass an async function to `<form action={fn}>`; React calls it with `FormData`.
- Use `useActionState` when you need the action's return value (error, success, data).
- Use `useFormStatus` in a child component of the form for pending state.
- Use `useTransition` for non-form async operations that should not block the UI.
- Use `useOptimistic` to show immediate feedback before the server responds.
- Server actions must be marked with `'use server'` and defined in a separate file or at the top of a module.
- The `action` prop also accepts a URL string for progressive enhancement.

**Constraints and Limitations:**
- React 19 Actions require React 19+; older versions use `onSubmit` handlers.
- `useFormStatus` must be called in a child of the `<form>`, not in the same component.
- `useActionState` replaces the deprecated `useFormState` from React 18.
- Uncontrolled forms are reset automatically after a successful action; controlled forms are not.
- Optimistic updates must be reconciled with the server's response; handle rollback on error.
- Server actions require a framework that supports them (Next.js, Remix) or a server runtime.

### Annotated Code Example: Complete React 19 Form with Action, `useActionState`, and `useFormStatus`

```jsx
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

// Submit button: reads pending state from the form
function SubmitButton({ label }) {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : label}
    </button>
  );
}

// Server action (in a real app, marked with 'use server')
async function createUserAction(prevState, formData) {
  const name = formData.get('name');
  const email = formData.get('email');

  // Validation
  const errors = {};
  if (!name) errors.name = 'Name is required';
  if (!email || !email.includes('@')) errors.email = 'Valid email required';

  if (Object.keys(errors).length > 0) {
    return { errors };
  }

  // Simulate API call
  await new Promise((r) => setTimeout(r, 1000));

  return { success: true, message: `Created user ${name}` };
}

export default function CreateUserForm() {
  const [state, formAction, isPending] = useActionState(createUserAction, null);

  return (
    <form action={formAction}>
      <h2>Create User</h2>

      <div>
        <label htmlFor="name">Name</label>
        <input
          id="name"
          name="name"
          aria-invalid={!!state?.errors?.name}
          aria-describedby={state?.errors?.name ? 'name-error' : undefined}
        />
        {state?.errors?.name && (
          <span id="name-error" role="alert" style={{ color: 'red' }}>
            {state.errors.name}
          </span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          aria-invalid={!!state?.errors?.email}
          aria-describedby={state?.errors?.email ? 'email-error' : undefined}
        />
        {state?.errors?.email && (
          <span id="email-error" role="alert" style={{ color: 'red' }}>
            {state.errors.email}
          </span>
        )}
      </div>

      {state?.success && (
        <p role="status" style={{ color: 'green' }}>
          {state.message}
        </p>
      )}

      <SubmitButton label="Create User" />
    </form>
  );
}
```

**Expected Output:** A "Create User" form with name and email fields. Submitting with empty fields shows validation errors. Submitting with valid data shows "Submitting...", then "Created user [name]" and resets the form.

**Why This Output Ocurs:** `useActionState` wraps `createUserAction`, providing `state` (the action's return value) and `isPending`. The action validates the `FormData` and returns either `{ errors }` or `{ success, message }`. The `SubmitButton` child reads `pending` via `useFormStatus` and disables itself. After a successful action, React resets the uncontrolled form. The `state` object drives conditional rendering of errors and the success message.

### Real-World Cases

- **Server-side mutations:** Creating, updating, and deleting records via server actions.
- **Newsletter signups:** Simple forms with validation and success feedback.
- **Contact forms:** Forms with server-side email sending and error handling.
- **Multi-step forms:** Each step submits via an action, with the result guiding the next step.
- **Optimistic updates:** Liking, bookmarking, or adding to cart with immediate UI feedback.

### References

- React Official Documentation – `<form>`: https://react.dev/reference/react-dom/components/form
- React Official Documentation – `useActionState`: https://react.dev/reference/react/useActionState
- React Official Documentation – `useFormStatus`: https://react.dev/reference/react-dom/hooks/useFormStatus
- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `useOptimistic`: https://react.dev/reference/react/useOptimistic
- React Official Documentation – Server Actions: https://react.dev/reference/rsc/server-functions

---

## Comparison and Decision Guidance

| Approach | Best For | State Ownership | Re-renders | Pending State | Progressive Enhancement |
|---|---|---|---|---|---|
| **Controlled inputs** | Live validation, conditional fields, formatting | React | Every keystroke | Manual | No |
| **Uncontrolled inputs** | Simple forms, performance-critical forms | DOM | None | Manual | Yes |
| **React Hook Form** | Complex forms, large forms, schema validation | Library (refs) | Minimal (on validation) | `isSubmitting` | Partial |
| **React 19 Actions** | Server mutations, progressive enhancement | React (action) | On action | `useFormStatus` / `isPending` | Yes |
| **TanStack Form** | Framework-agnostic, type-safe forms | Library | Configurable | `isSubmitting` | Partial |

**Decision Guidance:**
- **Start with React Hook Form** for complex forms; it minimises re-renders and provides dirty/touched/validation state out of the box.
- **Use controlled inputs** for fields that need instant validation, formatting, or dependent logic.
- **Use uncontrolled inputs** for simple forms or when performance is critical and you only need values on submit.
- **Use React 19 Actions** when your app uses React 19+ and you want progressive enhancement or server actions.
- **Use `useActionState`** when you need the action's return value (error, success, data).
- **Use `useFormStatus`** in submit buttons to read the form's pending state without prop drilling.
- **Use `useOptimistic`** for actions where immediate feedback matters (likes, toggles, adds).
- **Use `FormData`** for file uploads and multi-field forms; it is the native way to read form data.
- **Never mix controlled and uncontrolled** for the same input.

---

## References

- React Official Documentation – Reacting to Input with State: https://react.dev/learn/reacting-to-input-with-state
- React Official Documentation – `<form>`: https://react.dev/reference/react-dom/components/form
- React Official Documentation – `<input>`: https://react.dev/reference/react-dom/components/input
- React Official Documentation – `<select>`: https://react.dev/reference/react-dom/components/select
- React Official Documentation – `<textarea>`: https://react.dev/reference/react-dom/components/textarea
- React Official Documentation – `useActionState`: https://react.dev/reference/react/useActionState
- React Official Documentation – `useFormStatus`: https://react.dev/reference/react-dom/hooks/useFormStatus
- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `useOptimistic`: https://react.dev/reference/react/useOptimistic
- React Official Documentation – Server Functions: https://react.dev/reference/rsc/server-functions
- React Official Documentation – Manipulating the DOM with Refs: https://react.dev/learn/manipulating-the-dom-with-refs
- React Hook Form – Documentation: https://react-hook-form.com/
- React Hook Form – formState: https://react-hook-form.com/docs/useform/formstate
- React Hook Form – register: https://react-hook-form.com/docs/useform/register
- React Hook Form – Resolvers: https://react-hook-form.com/docs/useform#resolver
- MDN Web Docs – FormData: https://developer.mozilla.org/en-US/docs/Web/API/FormData
- MDN Web Docs – Using FormData Objects: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects
- MDN Web Docs – Client-side form validation: https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation
- Zod – Documentation: https://zod.dev/
- WAI-ARIA Authoring Practices – Form Validation: https://www.w3.org/WAI/ARIA/apg/
- Refine – Essentials of Managing Form State with React Hook Form: https://refine.dev/blog/react-hook-form/