# React Controlled Components — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A controlled component is a form element whose displayed value is driven entirely by React state, with every user change routed through an `onChange` handler that updates that state.

**Technical Definition:** In a controlled component, React state is the **single source of truth** for the input's value. The element receives its current value through the `value` prop (or `checked` for checkboxes and radios), and the `onChange` handler receives a SyntheticEvent whose `target` exposes the proposed new value. The handler updates state, which triggers a re-render, which passes the new value back to the element. The element never holds its own value independently. React's `onChange` for text inputs fires on every native `input` event — that is, on every keystroke — not on blur as the native DOM `change` event does. The React documentation states: *"In most cases, we recommend using controlled components to implement forms. In a controlled component, form data is handled by a React component."* File inputs are the sole exception: their `value` cannot be set programmatically, so they remain uncontrolled.

**Beginner-Friendly Explanation:** A controlled component is a form field where **React** decides what the field shows, not the browser. When you type, React catches the change, saves it in state, and then tells the field to display that saved value. It feels like a round trip: type → React saves it → React shows it. Because React holds the value, you can instantly validate it, format it, clear it, or enable/disable other UI based on it. The trade-off is that you must write the `value` and `onChange` wiring yourself — if you forget `onChange`, the field becomes read-only, and React will warn you.

### Key Characteristics

- **Single source of truth:** React state owns the value; the DOM reflects state, never the reverse.
- **`value` + `onChange` are a pair:** Providing `value` without `onChange` makes the input read-only and triggers a React warning.
- **Per-keystroke updates:** React's `onChange` fires on every input event for text controls, enabling real-time validation and formatting.
- **`checked` for checkboxes and radios:** Boolean controls use the `checked` prop, not `value`.
- **Array values for multi-select:** A `<select multiple>` uses an array as its `value`.
- **File inputs are always uncontrolled:** React cannot set a file input's `value`; you must read `e.target.files` or `FormData`.
- **Predictable and testable:** Because state drives the UI, tests can assert on state transitions and rendered output without simulating browser internals.

### Prerequisites

- React function components, `useState`, and event handling.
- JSX syntax, `className`, and `htmlFor`.
- Native HTML form elements and their attributes.
- Understanding of the render cycle and re-renders.

### Related Programming Areas

- **Form Fundamentals** — submission, validation, accessibility.
- **Uncontrolled Components** — the DOM-managed alternative.
- **Rendering Performance** — controlled inputs re-render on every keystroke.
- **Form Libraries** — `react-hook-form` uses uncontrolled inputs with refs for performance.
- **React 19 Actions** — `<form action={fn}>` with `FormData` as an alternative for simple forms.

---

## 1. Controlled Components

### Definitions

**Core Definition:** A controlled component is a form element whose value is controlled by React state rather than by the DOM.

**Technical Definition:** A controlled component is a React element (`<input>`, `<textarea>`, `<select>`) that receives its current value via a prop (`value` or `checked`) and notifies React of proposed changes via `onChange`. React state is updated in the handler, and the new state is passed back as the value prop on the next render. The component is therefore fully predictable: at any render, the element's displayed value is exactly the state value. React's documentation notes that in a controlled component, form data is handled by a React component.

**Beginner-Friendly Explanation:** A controlled component is a field where React is in charge. You always know exactly what is in the field because it is stored in React state.

### Purposes

- To maintain a single source of truth for form values.
- To enable instant validation, formatting, and conditional UI as the user types.
- To make form state predictable, debuggable, and testable.
- To allow programmatic control over the value (reset, prefill, transform).
- To synchronise multiple fields and derived UI with the same state.

### Syntax Rules and Structure

**General Syntax:**

```jsx
const [value, setValue] = useState('');

<input value={value} onChange={(e) => setValue(e.target.value)} />
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `value={value}` | The current value from React state. |
| `onChange` | Handler that receives the proposed change and updates state. |
| `setValue` | State setter that triggers a re-render with the new value. |

**Syntax Rules:**

1. Every controlled element must have both a value prop and an `onChange` handler.
2. Use `value` for text inputs, textareas, and selects; use `checked` for checkboxes and radios.
3. Read the new value from `e.target.value` (or `e.target.checked` for booleans).
4. Update state with the setter; React re-renders with the new value.
5. Use the functional updater form when the new state depends on the previous state.

**Constraints and Limitations:**

- Providing `value` without `onChange` produces a read-only input and a React warning.
- Passing `undefined` or `null` as `value` makes the input uncontrolled (and may warn).
- File inputs cannot be controlled; their `value` is read-only.
- Every keystroke triggers a re-render, which can be costly in large forms.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Controlled Text Input

```jsx
import { useState } from 'react';

function NameForm() {
  const [name, setName] = useState('');

  function handleChange(e) {
    setName(e.target.value); // Update state on every keystroke
  }

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Submitted: ${name}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="name">Name</label>
      <input id="name" value={name} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** The input displays the current `name` state. Typing updates the input on every keystroke. Submitting alerts the current value.

**Why this output:** `value={name}` binds the input to state. `onChange` fires on every keystroke, calling `setName`. React re-renders with the new value. Without `onChange`, the input would be read-only.

#### Example 2: Controlled Form with Multiple Field Types

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

**Expected Output:** All four controls stay synchronised with the single `form` state object. Checkbox changes store a boolean; text, textarea, and select store strings.

**Why this output:** A generic `update` function branches on `e.target.type` to read the correct value (`checked` vs `value`). Each control uses the correct prop (`value` vs `checked`) and the correct state field.

### Real-World Cases

- **Authentication forms:** Email and password fields with live validation.
- **Checkout forms:** Address fields with formatting and validation.
- **Search interfaces:** Query inputs with instant filtering.
- **Settings panels:** Preference forms with toggles and selects.

### References

- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — Forms — https://18.react.dev/reference/react-dom/components/input#controlling-an-input-with-a-state-variable
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- MDN — HTML forms — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms

---

## 2. `value` Prop

### Definitions

**Core Definition:** The `value` prop is the React prop that sets a form element's displayed value from state.

**Technical Definition:** The `value` prop binds a form control to a React state value. For text inputs, textareas, and selects, it is a string (or an array for multi-select). For checkboxes and radios, the boolean state of the control is controlled by the `checked` prop, while `value` defines the value submitted with the form. Passing `value` makes the element controlled; omitting it makes it uncontrolled. React warns if `value` is provided without `onChange`, or if `value` is `null`/`undefined` (which switches to uncontrolled mode).

**Beginner-Friendly Explanation:** The `value` prop is how you tell the input what to display. It comes from your React state.

### Purposes

- To bind a form control's displayed value to React state.
- To control the value programmatically (reset, prefill, transform).
- To keep the UI in sync with state at all times.
- To enable conditional rendering and validation based on the value.

### Syntax Rules and Structure

**Value Prop by Control Type:**

| Control | Prop | Value type |
|---------|------|------------|
| `<input type="text">` | `value` | String |
| `<textarea>` | `value` | String |
| `<select>` | `value` | String |
| `<select multiple>` | `value` | Array of strings |
| `<input type="checkbox">` | `checked` | Boolean |
| `<input type="radio">` | `checked` | Boolean (plus `value` for the submitted value) |
| `<input type="file">` | — | Cannot be set |

**Syntax Rules:**

1. Always pair `value` with `onChange` for controlled inputs.
2. Use `value` for text, textarea, and select; use `checked` for checkboxes and radios.
3. For radio groups, use `checked={selected === option.value}` on each radio.
4. For multi-select, pass an array and read `Array.from(e.target.selectedOptions, o => o.value)`.
5. Avoid `value={null}` or `value={undefined}`; they switch the control to uncontrolled mode.

**Constraints and Limitations:**

- `value` without `onChange` produces a read-only warning.
- `value` cannot be set on file inputs.
- `value` on a `<select multiple>` must be an array.

### Annotated Code Examples

```jsx
import { useState } from 'react';

function ColourPicker() {
  const [colour, setColour] = useState('blue');

  return (
    <>
      <select value={colour} onChange={(e) => setColour(e.target.value)}>
        <option value="red">Red</option>
        <option value="green">Green</option>
        <option value="blue">Blue</option>
      </select>
      <p>Selected: {colour}</p>
    </>
  );
}
```

**Expected Output:** The select displays `"blue"` initially. Changing the option updates state and the paragraph.

**Why this output:** `value={colour}` tells the `<select>` which option is selected. `onChange` updates `colour`. The paragraph reads the same state.

### Real-World Cases

- **Prefilled forms:** Setting `value` from server data.
- **Controlled transforms:** Forcing uppercase or masked input.
- **Reset flows:** Setting `value` back to an empty string.
- **Conditional selects:** Disabling options based on state.

### References

- React — `<input>`: value — https://react.dev/reference/react-dom/components/input#props
- React — `<select>`: value — https://react.dev/reference/react-dom/components/select#props
- React — `<textarea>`: value — https://react.dev/reference/react-dom/components/textarea#props

---

## 3. `onChange` Handler

### Definitions

**Core Definition:** The `onChange` handler is the React event handler that receives proposed changes to a controlled element and updates the corresponding state.

**Technical Definition:** React's `onChange` maps to the native `input` event for text controls, firing on every keystroke (unlike the native `change` event, which fires on blur). For checkboxes and radios, it fires when the checked state changes. The handler receives a SyntheticEvent; `e.target.value` (or `e.target.checked`) holds the proposed new value. Updating state in the handler causes React to re-render with the new value, completing the controlled round trip.

**Beginner-Friendly Explanation:** `onChange` is the function that runs every time the user changes a field. It is where you save the new value into React state.

### Purposes

- To capture every change to a controlled element.
- To update the state that drives the element's value.
- To perform real-time validation, formatting, and conditional logic.
- To synchronise multiple related fields.

### Syntax Rules and Structure

```jsx
<input value={value} onChange={(e) => setValue(e.target.value)} />
<input type="checkbox" checked={checked} onChange={(e) => setChecked(e.target.checked)} />
```

**Syntax Rules:**

1. `onChange` must be provided for every controlled element.
2. Use `e.target.value` for text, textarea, select, and radio groups.
3. Use `e.target.checked` for checkboxes and individual radio state.
4. Use the functional updater form when the new state depends on the previous state: `setForm(prev => ({ ...prev, name: e.target.value }))`.
5. Debounce or defer expensive work inside `onChange` if it runs on every keystroke.

**Constraints and Limitations:**

- `onChange` fires per keystroke for text inputs, which can be costly for expensive validation.
- Avoid heavy synchronous work in `onChange`; it runs on the main thread during input.
- Do not mutate state; always create new objects/arrays.

### Annotated Code Examples

```jsx
function SearchField({ onQueryChange }) {
  return (
    <input
      value={query}
      onChange={(e) => {
        const value = e.target.value;
        setQuery(value);
        onQueryChange(value); // Notify parent
      }}
    />
  );
}
```

**Expected Output:** Every keystroke updates local state and calls the parent callback with the new value.

**Why this output:** `onChange` fires on every input event, updating state and propagating the value to the parent.

### Real-World Cases

- **Live search:** Filtering results on each keystroke.
- **Character counters:** Counting characters as the user types.
- **Password strength meters:** Evaluating strength in real time.
- **Input masking:** Formatting phone numbers or credit cards.

### References

- React — `<input>`: onChange — https://react.dev/reference/react-dom/components/input#props
- React — Responding to Events — https://react.dev/learn/responding-to-events
- MDN — input event — https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event

---

## 4. Controlled Inputs

### Definitions

**Core Definition:** A controlled input is an `<input>` element whose value is driven by React state.

**Technical Definition:** A controlled input receives its `value` from state and notifies React of changes via `onChange`. The state is updated in the handler, and the new value is passed back as the `value` prop on the next render. This creates a predictable, unidirectional data flow: state → input → event → state.

**Beginner-Friendly Explanation:** A controlled input is a text box where React decides what is shown. You always know its value because it is in state.

### Purposes

- To keep input values synchronised with React state.
- To enable real-time validation and formatting.
- To support programmatic control (reset, prefill, disable).
- To make form data easy to submit and test.

### Syntax Rules and Structure

```jsx
<input value={value} onChange={(e) => setValue(e.target.value)} />
```

**Syntax Rules:**

1. Provide both `value` and `onChange`.
2. Use `e.target.value` to read the new value.
3. Update state synchronously in the handler.
4. Use `type="text"`, `type="email"`, `type="password"`, etc., as needed.

### Annotated Code Examples

```jsx
function EmailInput() {
  const [email, setEmail] = useState('');
  const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

  return (
    <>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      {!isValid && email.length > 0 && <p>Enter a valid email</p>}
    </>
  );
}
```

**Expected Output:** The input stays in sync with state, and the validation message appears while the email is invalid.

**Why this output:** `value={email}` binds the input to state. `onChange` updates state on every keystroke. The validation derives from the same state during render.

### Real-World Cases

- **Login forms:** Email and password inputs.
- **Search bars:** Query inputs with live filtering.
- **Profile forms:** Name, bio, and contact fields.

### References

- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state

---

## 5. Controlled Text Inputs

### Definitions

**Core Definition:** A controlled text input is a `<input>` of type text, email, password, search, or similar, whose string value is driven by React state.

**Technical Definition:** Text inputs use the `value` prop (a string) and `onChange` (reading `e.target.value`). React's `onChange` fires on every native `input` event, so state updates on every keystroke. This enables real-time validation, formatting, and conditional rendering. The `type` attribute provides native validation and keyboard affordances but does not change the controlled pattern.

**Beginner-Friendly Explanation:** A controlled text input is the classic example: a text box where React knows every character the user types.

### Purposes

- To keep text values in React state on every keystroke.
- To validate or format text as the user types.
- To enable conditional UI based on text length or content.
- To prefill or reset text programmatically.

### Syntax Rules and Structure

```jsx
<input type="text" value={text} onChange={(e) => setText(e.target.value)} />
```

**Syntax Rules:**

1. Use `value` (string) and `onChange`.
2. Use `type` for native validation and keyboard hints (`email`, `password`, `search`, `tel`, `url`, `number`).
3. Avoid `value={undefined}`; it makes the input uncontrolled.
4. For numeric inputs, convert `e.target.value` to a number if state must be numeric, and handle empty strings.

### Annotated Code Examples

```jsx
function UppercaseInput() {
  const [text, setText] = useState('');

  return (
    <input
      value={text}
      onChange={(e) => setText(e.target.value.toUpperCase())}
    />
  );
}
```

**Expected Output:** Typing lowercase letters displays uppercase characters.

**Why this output:** `onChange` transforms the value before storing it. The input re-renders with the transformed value.

### Real-World Cases

- **Email inputs:** `type="email"` with validation.
- **Password inputs:** `type="password"` with strength meters.
- **Search inputs:** `type="search"` with instant filtering.
- **Phone inputs:** Input masking as the user types.

### References

- React — `<input>`: Controlling an input with a state variable — https://react.dev/reference/react-dom/components/input#controlling-an-input-with-a-state-variable
- MDN — `<input>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input

---

## 6. Controlled Checkboxes

### Definitions

**Core Definition:** A controlled checkbox is a checkbox whose checked state is driven by a boolean in React state.

**Technical Definition:** Checkboxes use the `checked` prop (a boolean) and `onChange` (reading `e.target.checked`). The `value` attribute defines the value submitted with the form but does not control the checkbox's visual state. For groups of checkboxes, state is typically an array of selected values or an object of booleans.

**Beginner-Friendly Explanation:** A controlled checkbox is a tick box where React knows whether it is ticked. You use `checked` instead of `value`.

### Purposes

- To keep checked state in React state.
- To enable conditional logic based on checked values.
- To support groups of checkboxes with array state.
- To prefill or reset checkbox state programmatically.

### Syntax Rules and Structure

**Single Checkbox:**

```jsx
<input type="checkbox" checked={agreed} onChange={(e) => setAgreed(e.target.checked)} />
```

**Checkbox Group (Array State):**

```jsx
const [selected, setSelected] = useState([]);

function toggle(value) {
  setSelected(prev =>
    prev.includes(value)
      ? prev.filter(v => v !== value)
      : [...prev, value]
  );
}
```

**Syntax Rules:**

1. Use `checked` (boolean), not `value`, to control the checkbox state.
2. Read `e.target.checked` in `onChange`.
3. For groups, use an array or object in state and toggle on change.
4. The `value` attribute is still useful for form submission.

### Annotated Code Examples

```jsx
function ToppingPicker() {
  const [selected, setSelected] = useState([]);

  function toggle(topping) {
    setSelected(prev =>
      prev.includes(topping)
        ? prev.filter(t => t !== topping)
        : [...prev, topping]
    );
  }

  return (
    <>
      {['cheese', 'olives', 'mushrooms'].map(topping => (
        <label key={topping}>
          <input
            type="checkbox"
            checked={selected.includes(topping)}
            onChange={() => toggle(topping)}
            value={topping}
          />
          {topping}
        </label>
      ))}
      <p>Selected: {selected.join(', ') || 'none'}</p>
    </>
  );
}
```

**Expected Output:** Checking a topping adds it to the array; unchecking removes it. The paragraph reflects the selected toppings.

**Why this output:** `checked={selected.includes(topping)}` derives the checkbox state from the array. `onChange` toggles the value in the array immutably.

### Real-World Cases

- **Terms acceptance:** Single checkbox for agreement.
- **Filter panels:** Multiple checkboxes for categories.
- **Preference settings:** Toggles for options.
- **Bulk selection:** Selecting multiple rows in a table.

### References

- React — `<input type="checkbox">` — https://react.dev/reference/react-dom/components/input#props
- MDN — `<input type="checkbox">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox

---

## 7. Controlled Radio Buttons

### Definitions

**Core Definition:** A controlled radio button is a radio whose selected state is driven by React state, with only one option in a group selected at a time.

**Technical Definition:** Radios use the `checked` prop to indicate selection and `onChange` to update state. A radio group shares the same `name` attribute; each radio has a `value` that identifies it. State stores the selected value, and each radio's `checked` is derived by comparing state to that radio's value. Selection is mutually exclusive by nature.

**Beginner-Friendly Explanation:** Controlled radios let React decide which option in a group is selected. You store the selected value in state and mark each radio as checked if its value matches.

### Purposes

- To keep the selected radio value in React state.
- To enforce single-selection semantics.
- To enable conditional UI based on the selected option.
- To prefill or reset the selected option programmatically.

### Syntax Rules and Structure

```jsx
const [size, setSize] = useState('medium');

<input
  type="radio"
  name="size"
  value="small"
  checked={size === 'small'}
  onChange={(e) => setSize(e.target.value)}
/>
```

**Syntax Rules:**

1. All radios in a group must share the same `name`.
2. Each radio has a unique `value`.
3. Use `checked={state === option.value}` to derive selection.
4. Read `e.target.value` in `onChange` to update the selected value.
5. Only one radio per group can be `checked`.

### Annotated Code Examples

```jsx
function SizePicker() {
  const [size, setSize] = useState('medium');

  return (
    <>
      {['small', 'medium', 'large'].map(option => (
        <label key={option}>
          <input
            type="radio"
            name="size"
            value={option}
            checked={size === option}
            onChange={(e) => setSize(e.target.value)}
          />
          {option}
        </label>
      ))}
      <p>Selected: {size}</p>
    </>
  );
}
```

**Expected Output:** Selecting a radio updates the state and the paragraph. Only one option is selected at a time.

**Why this output:** Each radio’s `checked` is derived from `size === option`. Changing a radio calls `setSize` with the selected value, which updates all radios on re-render.

### Real-World Cases

- **Shipping options:** Standard, express, overnight.
- **Payment methods:** Card, PayPal, bank transfer.
- **Survey questions:** Single-choice answers.
- **Theme selection:** Light, dark, system.

### References

- React — `<input type="radio">` — https://react.dev/reference/react-dom/components/input#props
- MDN — `<input type="radio">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/radio

---

## 8. Controlled Selects

### Definitions

**Core Definition:** A controlled select is a `<select>` element whose selected option is driven by React state.

**Technical Definition:** A controlled `<select>` uses the `value` prop on the `<select>` element (not on the options) and `onChange` to update state. Each `<option>` has a `value` attribute. For multi-select (`<select multiple>`), the `value` is an array of selected option values. React sets the selected options based on the `value` prop, ensuring the selection is always synchronised with state.

**Beginner-Friendly Explanation:** A controlled select is a dropdown where React decides which option is selected. You put `value` on the `<select>`, not on the options.

### Purposes

- To keep the selected option in React state.
- To enable conditional UI based on the selected option.
- To support multi-select with array state.
- To prefill or reset the selection programmatically.

### Syntax Rules and Structure

**Single Select:**

```jsx
<select value={fruit} onChange={(e) => setFruit(e.target.value)}>
  <option value="apple">Apple</option>
  <option value="banana">Banana</option>
</select>
```

**Multi-Select:**

```jsx
<select
  multiple
  value={selected}
  onChange={(e) =>
    setSelected(Array.from(e.target.selectedOptions, o => o.value))
  }
>
  {/* options */}
</select>
```

**Syntax Rules:**

1. Put `value` on the `<select>`, not on the `<option>`.
2. Each `<option>` has a `value` attribute.
3. For multi-select, `value` must be an array and `onChange` reads `selectedOptions`.
4. An empty string `value=""` selects the placeholder option.

### Annotated Code Examples

```jsx
function FruitSelect() {
  const [fruit, setFruit] = useState('apple');

  return (
    <>
      <select value={fruit} onChange={(e) => setFruit(e.target.value)}>
        <option value="apple">Apple</option>
        <option value="banana">Banana</option>
        <option value="cherry">Cherry</option>
      </select>
      <p>Selected: {fruit}</p>
    </>
  );
}
```

**Expected Output:** The select shows "Apple" initially. Choosing another option updates the paragraph.

**Why this output:** `value={fruit}` tells React which option is selected. `onChange` updates `fruit`, and React re-renders the select with the new selection.

### Real-World Cases

- **Country selectors:** Dropdowns with many options.
- **Category filters:** Single or multi-select filters.
- **Settings forms:** Selecting a preference from a list.
- **Multi-select tags:** Choosing multiple labels.

### References

- React — `<select>` — https://react.dev/reference/react-dom/components/select
- MDN — `<select>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select

---

## 9. Controlled Textareas

### Definitions

**Core Definition:** A controlled textarea is a `<textarea>` element whose multi-line text value is driven by React state.

**Technical Definition:** In React, `<textarea>` is controlled with the `value` prop, not by children. The `onChange` handler reads `e.target.value`, which contains the full text content. React normalises textarea behaviour so it matches the controlled input pattern, despite the native HTML convention of placing text between the opening and closing tags.

**Beginner-Friendly Explanation:** A controlled textarea is a multi-line text box where React knows the content. Unlike HTML, you use `value` instead of putting text inside the tags.

### Purposes

- To keep multi-line text in React state.
- To enable character counting and validation.
- To support auto-resizing based on content.
- To prefill or reset text programmatically.

### Syntax Rules and Structure

```jsx
<textarea value={bio} onChange={(e) => setBio(e.target.value)} />
```

**Syntax Rules:**

1. Use `value` on the `<textarea>`, not children.
2. Read `e.target.value` in `onChange`.
3. Do not use `defaultValue` alongside `value`; they conflict.
4. For auto-resizing, measure `scrollHeight` in an effect.

### Annotated Code Examples

```jsx
function BioField() {
  const [bio, setBio] = useState('');
  const maxLength = 200;

  return (
    <>
      <textarea
        value={bio}
        onChange={(e) => setBio(e.target.value)}
        maxLength={maxLength}
      />
      <p>{bio.length}/{maxLength} characters</p>
    </>
  );
}
```

**Expected Output:** Typing updates the textarea and the character counter. Input stops at 200 characters.

**Why this output:** `value={bio}` binds the textarea to state. `onChange` updates state on every keystroke. `maxLength` enforces the limit natively.

### Real-World Cases

- **Comment boxes:** Multi-line user comments.
- **Bio fields:** Profile descriptions with character limits.
- **Message composers:** Chat and email composition.
- **Code editors:** Simple textarea-based editors.

### References

- React — `<textarea>` — https://react.dev/reference/react-dom/components/textarea
- MDN — `<textarea>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea

---

## 10. Controlled File Inputs

### Definitions

**Core Definition:** File inputs cannot be controlled in React; their `value` is read-only, so they are always uncontrolled.

**Technical Definition:** The HTML `<input type="file">` element has a read-only `value` that the browser controls for security reasons. React cannot set it programmatically. Passing a `value` prop to a file input does not work (and React will warn if you try to control it with state). The correct pattern is to use an `onChange` handler to read `e.target.files` (a `FileList`) or use `FormData` on submission. To reset a file input, you must either use a `key` to remount it or set `inputRef.current.value = ''` via a ref.

**Beginner-Friendly Explanation:** File inputs are the one exception: React cannot control them. Instead of a `value` prop, you read the selected files from `e.target.files` when the user picks them. To clear the input, you have to use a trick like changing its `key`.

### Purposes

- To read user-selected files through `e.target.files`.
- To support file uploads with `FormData`.
- To validate file type and size on change.
- To reset the file input using a `key` or ref.

### Syntax Rules and Structure

**Reading Files:**

```jsx
<input type="file" onChange={(e) => setFile(e.target.files?.[0] ?? null)} />
```

**Resetting with a Key:**

```jsx
<input key={resetKey} type="file" onChange={handleFileChange} />
```

**Resetting with a Ref:**

```jsx
inputRef.current.value = '';
```

**Syntax Rules:**

1. Never pass `value` to a file input; it is read-only and React cannot control it.
2. Read files from `e.target.files` (a `FileList`).
3. Use `multiple` to allow multiple files; read all entries from `files`.
4. To reset, change the `key` or set `ref.current.value = ''`.
5. Use `FormData` for submission; file inputs appear as `File` objects.

**Constraints and Limitations:**

- File inputs cannot be controlled with state.
- `value` is read-only; setting it programmatically throws or is ignored.
- The `files` list is live; copy it if you need to retain it.

### Annotated Code Examples

```jsx
import { useRef, useState } from 'react';

function AvatarUpload() {
  const [file, setFile] = useState(null);
  const [error, setError] = useState('');
  const inputRef = useRef(null);

  function handleChange(e) {
    const selected = e.target.files?.[0] ?? null;
    if (selected && selected.size > 2 * 1024 * 1024) {
      setError('File must be under 2MB');
      return;
    }
    setError('');
    setFile(selected);
  }

  function reset() {
    setFile(null);
    setError('');
    if (inputRef.current) inputRef.current.value = '';
  }

  return (
    <>
      <input
        ref={inputRef}
        type="file"
        accept="image/*"
        onChange={handleChange}
      />
      {error && <p role="alert">{error}</p>}
      {file && <p>Selected: {file.name}</p>}
      <button type="button" onClick={reset}>Clear</button>
    </>
  );
}
```

**Expected Output:** Selecting a file stores it in state and displays its name. Selecting a file over 2MB shows an error. Clicking Clear resets both state and the file input.

**Why this output:** The file input is uncontrolled; the selected file is read from `e.target.files`. State tracks the file for display and validation. `inputRef.current.value = ''` resets the native input.

### Real-World Cases

- **Avatar uploads:** Profile picture selection with validation.
- **Document uploads:** PDF or image attachments.
- **Bulk uploads:** Multiple file selection.
- **Form submissions:** Sending files via `FormData`.

### References

- React — `<input type="file">` — https://react.dev/reference/react-dom/components/input#props
- MDN — `<input type="file">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file
- MDN — FileList — https://developer.mozilla.org/en-US/docs/Web/API/FileList

---

## 11. Controlled Components Benefits

### Definitions

**Core Definition:** Controlled components benefit applications by making form state explicit, predictable, and fully synchronised with React.

**Technical Definition:** Controlled components provide a single source of truth, instant reactivity, and programmatic control over form values. Because state drives the UI, validation, formatting, conditional rendering, and submission all derive from the same data. React's documentation notes that controlled components support instant field validation, conditionally disabling the submit button, and enforcing input formats.

**Beginner-Friendly Explanation:** Controlled components make forms easier to reason about because React always knows the current values.

### Purposes

- To provide a single source of truth for form data.
- To enable instant validation and formatting.
- To support conditional UI (disable submit, show hints).
- To allow programmatic control (reset, prefill, transform).
- To make forms testable and debuggable.
- To synchronise multiple fields and derived state.

### Benefits Table

| Benefit | Description |
|---------|-------------|
| **Single source of truth** | State is the authoritative value at all times. |
| **Instant validation** | Validate on every keystroke or on blur. |
| **Conditional UI** | Disable submit, show errors, enable fields. |
| **Formatting** | Transform input as the user types. |
| **Programmatic control** | Reset, prefill, or transform values from code. |
| **Testability** | Assert on state and rendered output without browser internals. |
| **Synchronisation** | Multiple fields derive from the same state. |
| **Predictability** | The UI always reflects state. |

### Annotated Code Examples

```jsx
function PasswordForm() {
  const [password, setPassword] = useState('');
  const [confirm, setConfirm] = useState('');

  const tooShort = password.length < 8;
  const mismatch = confirm.length > 0 && password !== confirm;
  const canSubmit = !tooShort && !mismatch && confirm.length > 0;

  return (
    <form>
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      {tooShort && <p>At least 8 characters</p>}

      <input
        type="password"
        value={confirm}
        onChange={(e) => setConfirm(e.target.value)}
      />
      {mismatch && <p>Passwords do not match</p>}

      <button disabled={!canSubmit}>Submit</button>
    </form>
  );
}
```

**Expected Output:** The submit button stays disabled until the password is long enough and matches the confirmation. Error messages update live.

**Why this output:** All derived UI (`tooShort`, `mismatch`, `canSubmit`) is computed from the same state during render. No separate flags or synchronisation logic is needed.

### Real-World Cases

- **Signup forms:** Password strength and confirmation.
- **Checkout:** Address validation and conditional fields.
- **Search:** Live filtering and result counts.
- **Settings:** Dependent options and disabled states.

### References

- React — `<input>`: Benefits of controlled inputs — https://react.dev/reference/react-dom/components/input#props
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure

---

## 12. Controlled Components Pitfalls

### Definitions

**Core Definition:** Controlled component pitfalls are the common mistakes that cause read-only inputs, lost focus, stale values, or performance problems.

**Technical Definition:** Pitfalls arise from the controlled round trip: providing `value` without `onChange`; passing `undefined` or `null` as `value`; using `value` instead of `checked` for checkboxes; attempting to control file inputs; mutating state instead of creating new objects; and causing unnecessary re-renders on every keystroke. Additional pitfalls include losing focus when values are transformed asynchronously, and re-rendering large forms on every keystroke without optimisation.

**Beginner-Friendly Explanation:** These are the traps that make controlled components behave badly. The most common is forgetting `onChange`, which makes the field read-only.

### Common Pitfalls Table

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **`value` without `onChange`** | Input is read-only; React warns | Always pair `value` with `onChange` |
| **`value={undefined}`** | Switches to uncontrolled mode | Use `''` for empty controlled values |
| **`value` on checkbox** | Does not control checked state | Use `checked` |
| **Controlling file inputs** | `value` is read-only | Read `e.target.files`; reset with `key` or ref |
| **Mutating state** | Changes not detected; stale UI | Create new objects/arrays |
| **New object props** | Breaks memoization | Stabilise with `useMemo`/`useCallback` |
| **Async transformations** | Lost cursor position/focus | Transform synchronously or manage selection |

### Annotated Code Examples

**Pitfall 1: `value` without `onChange`**

```jsx
// ❌ Read-only input; React warns
<input value={name} />

// ✅ Controlled input
<input value={name} onChange={(e) => setName(e.target.value)} />
```

**Pitfall 2: `value={undefined}`**

```jsx
// ❌ Uncontrolled mode, may warn
<input value={undefined} onChange={...} />

// ✅ Controlled empty string
<input value={name ?? ''} onChange={...} />
```

**Pitfall 3: Mutating state**

```jsx
// ❌ Mutation: React may not re-render
form.name = e.target.value;
setForm(form);

// ✅ Immutable update
setForm(prev => ({ ...prev, name: e.target.value }));
```

**Expected Output:** The correct versions produce predictable controlled behaviour. The incorrect versions cause read-only inputs, uncontrolled mode, or missed re-renders.

**Why this output:** `value` requires `onChange` to complete the controlled round trip. `undefined` signals an uncontrolled input. Mutation does not create a new state reference.

### Real-World Cases

- **Read-only inputs:** The most common controlled-component bug.
- **Uncontrolled fallbacks:** Accidentally passing `undefined` from optional props.
- **State mutation:** A frequent source of stale controlled values.

### References

- React — `<input>`: Caveats — https://react.dev/reference/react-dom/components/input#caveats
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state

---

## 13. Controlled Components Performance

### Definitions

**Core Definition:** Controlled component performance is the cost of re-rendering the component tree on every value change, and the strategies for mitigating it.

**Technical Definition:** Controlled components re-render on every keystroke because `onChange` updates state. For a single input, this is cheap. For large forms, complex validation, or expensive sibling subtrees, the per-keystroke render can become a bottleneck. Mitigations include: isolating state to the leaf input; memoizing expensive siblings (`React.memo`, `useMemo`); debouncing expensive validation; using uncontrolled inputs for fields that do not need real-time UI; and using form libraries like `react-hook-form` that use uncontrolled inputs with refs and optimise re-renders.

**Beginner-Friendly Explanation:** Controlled inputs re-render on every keystroke. For a simple form, that is fine. For a big form or a heavy page, it can slow things down. The fixes are to keep state local, memoize heavy parts, or use uncontrolled inputs where you do not need live updates.

### Purposes

- To understand the cost of per-keystroke re-renders.
- To isolate state to the components that need it.
- To prevent expensive siblings from re-rendering.
- To choose between controlled, uncontrolled, and library-based approaches.

### Syntax Rules and Structure

**Performance Strategies:**

| Strategy | Mechanism |
|----------|-----------|
| Colocate state | Keep form state in the smallest component |
| Memoize siblings | `React.memo` for expensive components |
| Memoize derived data | `useMemo` for expensive validation |
| Debounce validation | Delay expensive checks |
| Uncontrolled inputs | Use `defaultValue` + `FormData` for simple forms |
| Form libraries | `react-hook-form` uses refs and optimised re-renders |

**Syntax Rules:**

1. Keep controlled state as close to the input as possible.
2. Avoid passing new objects or callbacks to memoized siblings on every keystroke.
3. Debounce expensive async validation; do not run it on every keystroke.
4. Use uncontrolled inputs for fields where live UI is unnecessary.
5. Measure with the React DevTools Profiler before optimising.

### Annotated Code Examples

**✅ Isolated state and memoized sibling:**

```jsx
import { memo, useState } from 'react';

const ExpensiveChart = memo(function ExpensiveChart({ data }) {
  return <Chart data={data} />;
});

function SearchPage({ data }) {
  return (
    <>
      <SearchField />
      <ExpensiveChart data={data} /> {/* Does not re-render on keystrokes */}
    </>
  );
}

function SearchField() {
  const [query, setQuery] = useState('');
  return (
    <input value={query} onChange={(e) => setQuery(e.target.value)} />
  );
}
```

**Expected Output:** Typing in the search field does not re-render `ExpensiveChart`.

**Why this output:** The controlled state is isolated in `SearchField`. `ExpensiveChart` is memoized and receives a stable `data` prop, so it skips re-rendering.

**✅ Uncontrolled for simple forms:**

```jsx
function SimpleForm() {
  function handleSubmit(e) {
    e.preventDefault();
    const data = new FormData(e.currentTarget);
    console.log(data.get('email'));
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" defaultValue="" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** The email input does not re-render on keystrokes; its value is read at submit time.

**Why this output:** Uncontrolled inputs do not update React state on every keystroke, so the component does not re-render until submission.

### Real-World Cases

- **Large forms:** Uncontrolled inputs or form libraries reduce re-renders.
- **Search with heavy results:** Isolate the input state; memoize the results.
- **Dashboard filters:** Keep filter state local and memoize chart components.
- **Async validation:** Debounce API checks instead of validating on every keystroke.

### References

- React — `<input>`: Optimizing re-renders — https://react.dev/reference/react-dom/components/input
- React — memo — https://react.dev/reference/react/memo
- React — useMemo — https://react.dev/reference/react/useMemo
- react-hook-form — https://react-hook-form.com/
- Steve Kinney — Colocation of State — https://stevekinney.com/courses/react-performance/colocation-of-state

---

## References

- React — `<input>` — https://react.dev/reference/react-dom/components/input
- React — `<textarea>` — https://react.dev/reference/react-dom/components/textarea
- React — `<select>` — https://react.dev/reference/react-dom/components/select
- React — Reacting to Input with State — https://react.dev/learn/reacting-to-input-with-state
- React — Responding to Events — https://react.dev/learn/responding-to-events
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state
- React — memo — https://react.dev/reference/react/memo
- React — useMemo — https://react.dev/reference/react/useMemo
- MDN — `<input>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input
- MDN — `<input type="checkbox">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox
- MDN — `<input type="radio">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/radio
- MDN — `<input type="file">` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file
- MDN — `<select>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select
- MDN — `<textarea>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea
- MDN — FileList — https://developer.mozilla.org/en-US/docs/Web/API/FileList
- MDN — input event — https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event
- react-hook-form — https://react-hook-form.com/
- Steve Kinney — Colocation of State — https://stevekinney.com/courses/react-performance/colocation-of-state