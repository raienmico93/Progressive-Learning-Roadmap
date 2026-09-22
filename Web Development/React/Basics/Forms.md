# React Forms — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

React forms are interactive HTML form elements whose values are managed by React state, enabling predictable data flow and validation within React components.

**Technical Definition**

React forms use "controlled components" where form element values are driven by React state rather than the DOM's internal state. Each form element receives its current value through a prop (`value` or `checked`) and notifies React of user input through an `onChange` handler. This makes React state the "single source of truth" for all form data, enabling real-time validation, conditional rendering, and programmatic control .

**Beginner-Friendly Explanation**

In a regular HTML form, the input fields remember what you type all by themselves. In React, you take control: you tell the input what to display (from your state), and the input tells you when the user types something new. You're in charge of remembering the data, which lets you validate it, change it, or reset it whenever you want.

### Key Characteristics

- **Single Source of Truth**: Form data lives in React state, not the DOM .
- **Two-Way Binding**: Value flows from state to input (`value` prop) and from input back to state (`onChange` handler).
- **Immediate Feedback**: `onChange` fires on every keystroke, enabling real-time validation .
- **Predictable**: The displayed value is always exactly what is in state.

### Prerequisites

- React State Fundamentals (`useState`)
- Event Handling (`onChange`, `onSubmit`)
- JSX attributes and expressions

### Related Programming Areas

- **Validation Libraries**: React Hook Form, Formik, Yup, Zod .
- **Server Actions**: React 19's `action` prop for form submission .
- **Accessibility**: Proper labeling with `<label htmlFor>` and `<select id>` .

### Core Concepts / Features

- Controlled Components
- Input State
- Text Inputs
- Checkboxes
- Radio Buttons
- Select Elements
- Textareas
- Form Submission
- Client-Side Validation
- Form Reset

---

## 1. Controlled Components

### Definitions

**Core Definition**

A controlled component is a form element whose value is controlled by React state, with the displayed value always driven by a `value` or `checked` prop and updated via an `onChange` handler.

**Technical Definition**

In a controlled component, React state serves as the "single source of truth." The form element's `value` (or `checked`) prop is set to a state variable, and the `onChange` handler updates that state on every user interaction. Because the displayed value comes from state, React fully controls what the user sees and can programmatically modify it .

**Beginner-Friendly Explanation**

Think of a controlled input like a puppet. The puppet (input field) only does what you (React state) tell it to do. When the user types, the puppet tells you, you update the script (state), and the puppet displays the new script. You're always in control.

### Purposes

- To make React state the authoritative source of form data.
- To enable real-time validation and transformation of input.
- To allow programmatic control over form values (reset, prefill, disable).
- To synchronize form data with other UI elements.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const [value, setValue] = useState('');

<input 
  value={value} 
  onChange={e => setValue(e.target.value)} 
/>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `value={value}` | Binds input value to state. |
| `onChange={...}` | Updates state on every keystroke. |
| `e.target.value` | The current input value from the event. |

**Syntax Rules**

1. **`value` requires `onChange`**: Without `onChange`, the input is read-only .
2. **State drives display**: The input always shows the current state value.
3. **Update synchronously**: `onChange` must update state immediately for typing to work.

**Constraints and Limitations**

- More boilerplate than uncontrolled components.
- Every keystroke triggers a re-render (may affect performance in large forms).
- Cannot switch between controlled and uncontrolled .

### Annotated Complete Code Examples

**Example 1: Basic Controlled Input**

```jsx
import { useState } from 'react';

function NameForm() {
  const [name, setName] = useState('');

  return (
    <div>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
        placeholder="Enter name"
      />
      <p>You typed: {name}</p>
    </div>
  );
}
```

**Expected Output (after typing "Alice")**

```
You typed: Alice
```

**Why This Output Occurs**

`value={name}` binds the input to state. `onChange` fires on each keystroke, calling `setName` with the new value. React re-renders, updating both the input display and the paragraph.

### Real-World Cases

**Case: Search Filter**

A search input is controlled so the filtered list updates in real time as the user types.

### References

- React — Forms (Controlled Components) - https://react.dev/reference/react-dom/components/input

---

## 2. Input State

### Definitions

**Core Definition**

Input state is the React state that stores the current values of form inputs, serving as the source of truth for what the user has entered.

**Technical Definition**

Input state is typically a single state object (for multiple fields) or multiple `useState` calls (one per field). For forms with many fields, a single object with a generic `onChange` handler that uses the input's `name` attribute is the recommended pattern .

**Beginner-Friendly Explanation**

Input state is like a notebook where you write down everything the user types. You look at the notebook to know what's in each field, and you update it every time they type.

### Purposes

- To store all form data in one place.
- To enable validation and submission using current values.
- To simplify handling multiple form fields.

### Syntax Rules and Structure

**Complete General Syntax (Multiple Fields)**

```jsx
const [formData, setFormData] = useState({
  firstName: '',
  lastName: '',
  email: ''
});

const handleChange = (e) => {
  const { name, value } = e.target;
  setFormData(prev => ({ ...prev, [name]: value }));
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `formData` | Object holding all field values. |
| `name` | Input's name attribute (matches key in formData). |
| `[name]: value` | Computed property updates the correct field. |

**Syntax Rules**

1. **Use `name` attribute**: Each input needs a `name` matching its state key .
2. **Spread previous state**: `setFormData(prev => ({ ...prev, [name]: value }))` preserves other fields.
3. **Single handler**: One handler works for all inputs.

### Annotated Complete Code Examples

**Example 1: Multiple Inputs with Single State**

```jsx
import { useState } from 'react';

function RegistrationForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  return (
    <form>
      <input name="username" value={formData.username} onChange={handleChange} placeholder="Username" />
      <input name="email" value={formData.email} onChange={handleChange} placeholder="Email" />
      <input name="password" type="password" value={formData.password} onChange={handleChange} placeholder="Password" />
      <pre>{JSON.stringify(formData, null, 2)}</pre>
    </form>
  );
}
```

**Expected Output (after filling fields)**

```json
{
  "username": "alice",
  "email": "[email protected]",
  "password": "secret123"
}
```

**Why This Output Occurs**

Each input has a `name` matching a key in `formData`. The generic `handleChange` uses `e.target.name` to determine which field to update, preserving all other values with the spread operator .

### Real-World Cases

**Case: Multi-Field Registration**

A registration form stores username, email, password, and preferences in a single state object, with one change handler managing all fields.

### References

- GitHub — Handling Multiple Inputs - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/react/6-react-forms/0-react-controlled-components.mdx

---

## 3. Text Inputs

### Definitions

**Core Definition**

Text inputs are single-line form fields (`<input type="text">`) whose values are controlled by React state via the `value` and `onChange` props.

**Technical Definition**

In React, text inputs are controlled by setting the `value` prop to a state variable and updating that state in the `onChange` handler. React's `onChange` behaves like the native `input` event, firing on every keystroke .

**Beginner-Friendly Explanation**

A text input is the most basic form field. In React, you tell it what to show (from state) and update the state whenever the user types.

### Purposes

- To collect single-line text data.
- To enable controlled input for validation.
- To transform input (e.g., uppercase, filter characters).

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<input 
  type="text"
  value={value} 
  onChange={e => setValue(e.target.value)} 
/>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type="text"` | Default input type (can be omitted). |
| `value` | Current value from state. |
| `onChange` | Handler updating state. |

**Syntax Rules**

1. **Always provide `onChange`** with `value` .
2. **Use `e.target.value`** to access the new value.

### Annotated Complete Code Examples

**Example 1: Uppercase Transformation**

```jsx
function UppercaseInput() {
  const [value, setValue] = useState('');

  function handleChange(e) {
    // Transform to uppercase as user types
    setValue(e.target.value.toUpperCase());
  }

  return <input value={value} onChange={handleChange} />;
}
```

**Expected Output (typing "hello")**

```
HELLO
```

**Why This Output Occurs**

The handler calls `.toUpperCase()` on the input value before setting state. The displayed value is always uppercase, demonstrating React's control over the input .

### Real-World Cases

**Case: Email Input**

An email input uses `type="email"` with controlled state for validation before submission.

### References

- React — Forms (Controlled Components) - https://17.reactjs.org/docs/forms.html

---

## 4. Checkboxes

### Definitions

**Core Definition**

Checkboxes are form controls (`<input type="checkbox">`) that represent a boolean state (checked/unchecked) or a selection in a group.

**Technical Definition**

In React, checkboxes use the `checked` prop (not `value`) to reflect state. The `onChange` handler updates state using `e.target.checked` (boolean). For groups, an array of selected values is used .

**Beginner-Friendly Explanation**

A checkbox is a toggle. In React, you tell it whether it's checked (from state), and when the user clicks it, you update the state with the new checked value.

### Purposes

- To represent boolean options (agree to terms).
- To allow multiple selections from a list.
- To toggle features on/off.

### Syntax Rules and Structure

**Complete General Syntax (Single)**

```jsx
<input 
  type="checkbox"
  checked={isChecked} 
  onChange={e => setIsChecked(e.target.checked)} 
/>
```

**Complete General Syntax (Group)**

```jsx
<input
  type="checkbox"
  checked={selected.includes('react')}
  onChange={e => {
    if (e.target.checked) setSelected([...selected, 'react']);
    else setSelected(selected.filter(s => s !== 'react'));
  }}
/>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `checked` | Boolean reflecting state. |
| `e.target.checked` | New boolean value from event. |

**Syntax Rules**

1. **Use `checked`, not `value`** .
2. **Access `e.target.checked`** for boolean state.
3. **For groups**: Use an array and add/remove based on checked.

### Annotated Complete Code Examples

**Example 1: Boolean Checkbox**

```jsx
function TermsCheckbox() {
  const [agreed, setAgreed] = useState(false);

  return (
    <label>
      <input
        type="checkbox"
        checked={agreed}
        onChange={e => setAgreed(e.target.checked)}
      />
      I agree to the terms
    </label>
  );
}
```

**Expected Behavior**

Clicking the checkbox toggles `agreed` between `true` and `false`.

**Why This Occurs**

`checked={agreed}` binds the checkbox to state. `onChange` updates state with `e.target.checked` .

### Real-World Cases

**Case: Newsletter Subscription**

A checkbox controls whether the user is subscribed to a newsletter, stored as a boolean in state.

### References

- CoreUI — How to handle checkbox inputs in React - https://coreui.io/answers/how-to-handle-checkbox-inputs-in-react/

---

## 5. Radio Buttons

### Definitions

**Core Definition**

Radio buttons are form controls (`<input type="radio">`) that allow selecting exactly one option from a group.

**Technical Definition**

Radio buttons in a group share the same `name` attribute. React controls the selected option by comparing the current state value with each radio's `value` prop, setting `checked={state === value}` .

**Beginner-Friendly Explanation**

Radio buttons are like a multiple-choice question where you can only pick one answer. In React, you store the selected value in state and check which radio matches that value.

### Purposes

- To select one option from a set (payment method, gender).
- To enforce single selection within a group.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<input
  type="radio"
  name="groupName"
  value="option1"
  checked={selected === 'option1'}
  onChange={e => setSelected(e.target.value)}
/>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `name` | Shared across group members. |
| `value` | The option's value. |
| `checked` | Boolean comparing state to value. |
| `onChange` | Updates state with selected value. |

**Syntax Rules**

1. **Shared `name`**: All radios in a group must share the same `name` .
2. **`checked` comparison**: `checked={state === value}`.
3. **`e.target.value`**: The selected radio's value.

### Annotated Complete Code Examples

**Example 1: Payment Method Selection**

```jsx
function PaymentForm() {
  const [method, setMethod] = useState('credit');

  return (
    <form>
      <label>
        <input
          type="radio"
          name="payment"
          value="credit"
          checked={method === 'credit'}
          onChange={e => setMethod(e.target.value)}
        />
        Credit Card
      </label>
      <label>
        <input
          type="radio"
          name="payment"
          value="paypal"
          checked={method === 'paypal'}
          onChange={e => setMethod(e.target.value)}
        />
        PayPal
      </label>
      <p>Selected: {method}</p>
    </form>
  );
}
```

**Expected Output (after selecting PayPal)**

```
Selected: paypal
```

**Why This Output Occurs**

Both radios share `name="payment"`. `checked` compares state to each value. `onChange` updates state with the selected value .

### Real-World Cases

**Case: Shipping Speed**

A form uses radio buttons to select shipping speed (standard, express, overnight).

### References

- CoreUI — How to handle radio buttons in React - https://coreui.io/answers/how-to-handle-radio-buttons-in-react/

---

## 6. Select Elements

### Definitions

**Core Definition**

Select elements (`<select>`) are dropdown lists where the user selects one (or multiple) options from a list.

**Technical Definition**

In React, `<select>` uses the `value` prop on the root element (not `selected` on options) to control which option is displayed. The `onChange` handler updates state with `e.target.value`. For multi-select, `value` is an array and `multiple` is set .

**Beginner-Friendly Explanation**

A select dropdown is a list where you pick an item. In React, you tell the select which value is chosen (from state), and when the user picks a new one, you update the state.

### Purposes

- To select one option from many (country, category).
- To select multiple options (tags, permissions).

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<select value={selected} onChange={e => setSelected(e.target.value)}>
  <option value="apple">Apple</option>
  <option value="banana">Banana</option>
</select>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `value` | Controls selected option. |
| `onChange` | Updates state with selected value. |
| `<option value>` | Each option's value. |

**Syntax Rules**

1. **`value` on `<select>`**: Not `selected` on `<option>` .
2. **Every `<option>` needs a `value`**.
3. **For multiple**: `value` is an array, `multiple={true}`.

### Annotated Complete Code Examples

**Example 1: Fruit Selector**

```jsx
function FruitPicker() {
  const [fruit, setFruit] = useState('apple');

  return (
    <label>
      Pick a fruit:
      <select value={fruit} onChange={e => setFruit(e.target.value)}>
        <option value="apple">Apple</option>
        <option value="banana">Banana</option>
        <option value="orange">Orange</option>
      </select>
      <p>Selected: {fruit}</p>
    </label>
  );
}
```

**Expected Output (after selecting Banana)**

```
Selected: banana
```

**Why This Output Occurs**

The `<select>` `value` prop is bound to `fruit` state. `onChange` updates state with the selected option's value .

### Real-World Cases

**Case: Country Selector**

A form uses a select element to choose a country from a long list, with the value stored in state.

### References

- React — `<select>` Reference - https://18.react.dev/reference/react-dom/components/select

---

## 7. Textareas

### Definitions

**Core Definition**

Textareas are multi-line text input fields (`<textarea>`) whose value is controlled by React state.

**Technical Definition**

In React, `<textarea>` uses the `value` prop (not children) to control its content. The `onChange` handler updates state on every keystroke, identical to text inputs .

**Beginner-Friendly Explanation**

A textarea is like a text input but bigger. In React, you control it the same way: value from state, onChange updates state.

### Purposes

- To collect multi-line text (comments, messages, descriptions).
- To control and validate longer text content.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<textarea 
  value={text} 
  onChange={e => setText(e.target.value)} 
  rows={4}
/>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `value` | Controls textarea content. |
| `onChange` | Updates state on input. |

**Syntax Rules**

1. **Use `value`, not children** .
2. **`onChange` required** for controlled behavior.
3. **Do not use `<textarea>content</textarea>`**.

### Annotated Complete Code Examples

**Example 1: Comment Box**

```jsx
function CommentBox() {
  const [comment, setComment] = useState('');

  return (
    <div>
      <textarea
        value={comment}
        onChange={e => setComment(e.target.value)}
        placeholder="Write a comment..."
        rows={4}
      />
      <p>Characters: {comment.length}</p>
    </div>
  );
}
```

**Expected Output (after typing "Hello")**

```
Characters: 5
```

**Why This Output Occurs**

`value={comment}` binds the textarea to state. `onChange` updates state on each keystroke. The character count reflects the current state length .

### Real-World Cases

**Case: Blog Post Editor**

A textarea collects the post content, with state used for character limits and preview.

### References

- React — `<textarea>` Reference - https://vi.react.dev/reference/react-dom/components/textarea

---

## 8. Form Submission

### Definitions

**Core Definition**

Form submission is the process of handling the `onSubmit` event to process form data when the user submits a form.

**Technical Definition**

React's `onSubmit` prop on `<form>` fires when the form is submitted. The handler typically calls `e.preventDefault()` to stop page reload, then processes the state data (validation, API call) . React 19 also supports the `action` prop for automatic form handling .

**Beginner-Friendly Explanation**

Form submission is what happens when you click the submit button. You write a handler that reads the form data and does something with it—like sending it to a server.

### Purposes

- To process form data when the user submits.
- To validate all fields before submission.
- To send data to a server or update application state.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
function handleSubmit(e) {
  e.preventDefault();
  // Process formData state
}

<form onSubmit={handleSubmit}>
  {/* fields */}
  <button type="submit">Submit</button>
</form>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `onSubmit` | Form event handler. |
| `e.preventDefault()` | Stops page reload. |
| `type="submit"` | Button that triggers submission. |

**Syntax Rules**

1. **Always `e.preventDefault()`** to stop page reload .
2. **Access data from state** (controlled) or `FormData` (uncontrolled).
3. **Validate before submission**.

### Annotated Complete Code Examples

**Example 1: Basic Submit Handler**

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Logging in with: ${email}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={email}
        onChange={e => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Log In</button>
    </form>
  );
}
```

**Expected Output**

```
When submitted: Alert displays "Logging in with: [email]"
```

**Why This Output Occurs**

`onSubmit` fires on form submission. `preventDefault()` stops reload. The handler reads current `email` state .

### Real-World Cases

**Case: Contact Form**

A contact form uses `onSubmit` to validate fields and send data via API without page reload.

### References

- CoreUI — How to handle form submission in React - https://coreui.io/answers/how-to-handle-form-submission-in-react/

---

## 9. Client-Side Validation

### Definitions

**Core Definition**

Client-side validation is the process of checking form data against rules in the browser before submission, providing immediate feedback to users.

**Technical Definition**

In React, validation is typically performed in the `onSubmit` handler (or on `onChange` for real-time feedback). An `errors` state object stores validation messages. If errors exist, submission is prevented and messages are displayed .

**Beginner-Friendly Explanation**

Validation is checking if the user filled out the form correctly. If they forgot a required field or entered an invalid email, you show them a message before they submit.

### Purposes

- To ensure data quality before submission.
- To provide immediate feedback to users.
- To reduce server-side validation load.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const [errors, setErrors] = useState({});

const validate = () => {
  const newErrors = {};
  if (!email) newErrors.email = 'Email is required';
  if (!/\S+@\S+\.\S+/.test(email)) newErrors.email = 'Invalid email';
  setErrors(newErrors);
  return Object.keys(newErrors).length === 0;
};

const handleSubmit = (e) => {
  e.preventDefault();
  if (validate()) { /* submit */ }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `errors` | Object mapping field names to messages. |
| `validate()` | Function checking all rules. |
| `noValidate` | Disables browser validation (optional). |

**Syntax Rules**

1. **Validate on submit** (or on change for real-time).
2. **Store errors in state**.
3. **Clear errors when field is edited** .

### Annotated Complete Code Examples

**Example 1: Required Field Validation**

```jsx
function ValidatedForm() {
  const [email, setEmail] = useState('');
  const [errors, setErrors] = useState({});

  function validate() {
    const newErrors = {};
    if (!email.trim()) newErrors.email = 'Email is required';
    else if (!/\S+@\S+\.\S+/.test(email)) newErrors.email = 'Invalid email';
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }

  function handleSubmit(e) {
    e.preventDefault();
    if (validate()) {
      alert('Form submitted!');
    }
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      <input
        value={email}
        onChange={e => {
          setEmail(e.target.value);
          if (errors.email) setErrors({ ...errors, email: null });
        }}
        placeholder="Email"
      />
      {errors.email && <p style={{ color: 'red' }}>{errors.email}</p>}
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output**

```
When submitting empty: "Email is required"
When submitting "abc": "Invalid email"
When submitting valid email: Alert "Form submitted!"
```

**Why This Output Occurs**

`validate()` checks the email against rules and populates `errors`. The error message renders conditionally. Typing clears the error for that field .

### Real-World Cases

**Case: Password Strength**

A form validates password length and complexity on submit, showing specific error messages for each failure.

### References

- Telerik — React Forms Validation FAQ - https://www.telerik.com/faqs/react/react-forms-validation
- GitHub — Form Validation - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/react/6-react-forms/6-react-form-submission.mdx

---

## 10. Form Reset

### Definitions

**Core Definition**

Form reset is the process of returning all form fields to their initial or empty state, typically after successful submission.

**Technical Definition**

For controlled components, reset is achieved by setting the form state back to its initial values. This can be done by storing the initial state in a constant and calling `setFormData(initialState)` or by using a `key` prop to remount the form .

**Beginner-Friendly Explanation**

Form reset is like erasing a whiteboard. After the user submits the form, you clear all the fields so they can start fresh.

### Purposes

- To clear form data after successful submission.
- To allow users to start over.
- To provide a clean state for new entries.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const initialState = { email: '', password: '' };
const [formData, setFormData] = useState(initialState);

function handleSubmit(e) {
  e.preventDefault();
  // ... process
  setFormData(initialState); // Reset
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `initialState` | Constant holding initial values. |
| `setFormData(initialState)` | Resets state to initial. |

**Syntax Rules**

1. **Store initial state in a constant** for reuse .
2. **Reset after successful submission**.
3. **Alternative**: Use `key` prop to remount (resets all state).

### Annotated Complete Code Examples

**Example 1: Reset After Submission**

```jsx
const initialState = { name: '', email: '' };

function ResetForm() {
  const [formData, setFormData] = useState(initialState);

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Submitted: ${formData.name}, ${formData.email}`);
    setFormData(initialState); // Reset to initial
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={e => setFormData({ ...formData, name: e.target.value })}
        placeholder="Name"
      />
      <input
        name="email"
        value={formData.email}
        onChange={e => setFormData({ ...formData, email: e.target.value })}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Behavior**

After submission, both fields are cleared.

**Why This Occurs**

`setFormData(initialState)` sets state back to the initial empty object, causing both controlled inputs to display empty values .

### Real-World Cases

**Case: Chat Input**

After sending a message, the chat input is reset to empty, allowing the user to type a new message.

### References

- CoreUI — How to reset form in React - https://coreui.io/answers/how-to-reset-form-in-react/

---

## References

- React Official Documentation — Forms (Controlled Components) - https://react.dev/reference/react-dom/components/input
- React Official Documentation — `<select>` Reference - https://18.react.dev/reference/react-dom/components/select
- React Official Documentation — `<textarea>` Reference - https://vi.react.dev/reference/react-dom/components/textarea
- React Official Documentation — `<form>` Reference - https://react.dev/reference/react-dom/components/form
- React Legacy Documentation — Forms - https://17.reactjs.org/docs/forms.html
- CoreUI — How to handle checkbox inputs in React - https://coreui.io/answers/how-to-handle-checkbox-inputs-in-react/
- CoreUI — How to handle radio buttons in React - https://coreui.io/answers/how-to-handle-radio-buttons-in-react/
- CoreUI — How to handle select dropdown in React - https://coreui.io/answers/how-to-handle-select-dropdown-in-react/
- CoreUI — How to handle form submission in React - https://coreui.io/answers/how-to-handle-form-submission-in-react/
- CoreUI — How to reset form in React - https://coreui.io/answers/how-to-reset-form-in-react/
- Telerik — React Forms Validation FAQ - https://www.telerik.com/faqs/react/react-forms-validation
- GitHub — Compile-N-Run React Forms Documentation - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/react/6-react-forms/0-react-controlled-components.mdx
- GitHub — React 19 Controlled Checkboxes Issue - https://github.com/react/react/issues/31695