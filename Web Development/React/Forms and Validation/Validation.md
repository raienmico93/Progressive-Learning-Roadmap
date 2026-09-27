# React Validation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React validation is the practice of evaluating user input against defined rules—required fields, format constraints, cross-field dependencies, and server-side checks—and presenting the results as accessible, timely feedback within a React form.

**Technical Definition:** React validation encompasses the strategies, APIs, and patterns used to verify that form input conforms to expected constraints before it is submitted to a server. It operates at multiple layers: **client-side validation** (using HTML5 constraint validation attributes and/or custom React logic) and **server-side validation** (using API checks for uniqueness, business rules, and data integrity). The browser's Constraint Validation API provides native validation through attributes such as `required`, `pattern`, `min`, `max`, `minlength`, and `maxlength`, along with methods like `checkValidity()`, `reportValidity()`, and `setCustomValidity()` for programmatic control. React validation logic can be **synchronous** (evaluated during render or in event handlers) or **asynchronous** (requiring server round trips with debouncing and request cancellation). Validation state includes errors (blocking), warnings (non-blocking), and pending states (async checks in progress). Libraries like React Hook Form and TanStack Form provide structured validation APIs with configurable modes (`onChange`, `onBlur`, `onSubmit`, `onTouched`, `all`), schema resolvers (Zod, Yup), and built-in accessibility support. The Web Content Accessibility Guidelines (WCAG) require that error messages be programmatically associated with their inputs via `aria-describedby`, that invalid inputs be marked with `aria-invalid`, and that focus be managed to the first error on submission failure.

**Beginner-Friendly Explanation:** Validation is like a spell-checker for your form. Some checks are instant—"this field is required," "this email doesn't have an @ symbol." Others take time—"is this username already taken?" For instant checks, you can show errors as the user types or when they leave a field. For slow checks, you need to wait for the server and show a "checking..." message. You also need to make sure screen readers announce the errors and that the user's focus is moved to the first problem so they can fix it quickly.

### Key Characteristics

- **Layered Defence:** Client-side validation provides immediate feedback; server-side validation provides security and business-rule enforcement. Both are necessary.
- **Trigger Flexibility:** Validation can run on change, on blur, on submit, or on a combination, configurable per form or per field.
- **Constraint API Integration:** Native HTML5 validation attributes work alongside custom React logic, with `setCustomValidity()` bridging the two.
- **Asynchronous Awareness:** Server-side checks require debouncing, request cancellation, and stale-result guards to prevent race conditions.
- **Cross-Field Dependency:** Rules like password confirmation and date-range validation require access to other field values.
- **Accessibility by Default:** `aria-invalid`, `aria-describedby`, `role="alert"`, and focus management are essential for screen-reader users.
- **Progressive Enhancement:** Native HTML validation works without JavaScript; React validation enhances it when JavaScript is available.

### Prerequisites

- Solid understanding of React function components, JSX, and props.
- Familiarity with the `useState`, `useEffect`, and `useRef` Hooks.
- Working knowledge of HTML form elements and their attributes.
- Basic understanding of asynchronous JavaScript (promises, `async`/`await`).
- Awareness of the browser's Constraint Validation API and `FormData`.

### Related Programming Areas

- **Form Architecture:** Controlled vs. uncontrolled inputs, `FormData`, submission lifecycle.
- **State Management:** Form state, validation state, and submission state.
- **Accessibility (a11y):** ARIA attributes, focus management, live regions.
- **Server Integration:** Async validation, API error mapping, server actions.
- **Schema Validation:** Zod, Yup, and other validation libraries.

### Core Concepts / Features

1. Inline Validation vs. On-Submit Validation Triggers
2. Built-in HTML5 Constraint Validation vs. Custom React Validation Logic
3. Basic Constraints (Required, Type, Range, Pattern/Regex)
4. Dependent and Cross-Field Validation (Password Confirmation, Date Ranges)
5. Async and Server-Side Validation Integration
6. Accessibility (a11y) in Validation

---

## Core Concept 1: Inline Validation vs. On-Submit Validation Triggers

### Definitions

**Core Definition:** Inline validation evaluates input as the user interacts with a field (on change or blur), while on-submit validation evaluates all fields when the user attempts to submit the form.

**Technical Definition:** Validation triggers determine when validation rules are evaluated. The **`onChange`** mode validates on every keystroke, providing the most immediate feedback but causing the most re-renders. The **`onBlur`** mode validates when the user leaves a field, balancing responsiveness with performance. The **`onSubmit`** mode (the default in React Hook Form) validates all fields only when the user attempts to submit, minimising re-renders but providing no feedback until submission. The **`onTouched`** mode validates on the first blur and then on every change thereafter. The **`all`** mode validates on both blur and change. The **`reValidateMode`** option controls validation behaviour after the first submission attempt (default: `onChange`). A common UX pattern is to combine `onBlur` for initial validation with `onChange` for re-validation once an error has been shown, so the error clears as the user corrects it.

**Beginner-Friendly Explanation:** Think of validation triggers like when a teacher checks your homework. **On-submit** is like checking everything at the end—you get all the feedback at once, but you might have made the same mistake ten times. **On-blur** is like checking each answer as you finish it—you get feedback after each question but not while you're still writing. **On-change** is like the teacher looking over your shoulder every time you write a letter—very immediate, but potentially annoying and distracting. The best approach is usually a combination: check on blur (when you finish a field) and then re-check on change (as you fix the error).

### Purposes

- **On-submit:** To minimise re-renders and validate only when necessary (default for short forms).
- **On-blur:** To provide feedback after the user finishes a field without distracting them while typing.
- **On-change:** To provide immediate feedback for fields where real-time validation is critical (e.g., password strength, character limits).
- **On-touched:** To validate on first blur and then on subsequent changes, balancing initial and ongoing feedback.
- **Re-validation:** To clear errors as the user corrects them, rather than waiting for the next submission.

### Syntax Rules and Structure

**React Hook Form `mode` and `reValidateMode`:**
```jsx
const {
  register,
  handleSubmit,
  formState: { errors },
} = useForm({
  mode: 'onBlur',           // Validate on blur before submission
  reValidateMode: 'onChange', // Re-validate on change after submission
  defaultValues: { email: '', password: '' },
});
```

**Component Breakdown:**
- `mode: 'onBlur'`: Validation runs when the user leaves a field.
- `reValidateMode: 'onChange'`: After the first submission attempt, validation runs on every change.
- `mode` options: `'onSubmit'` (default), `'onBlur'`, `'onChange'`, `'onTouched'`, `'all'`.

**Per-Field Validation Trigger (TanStack Form):**
```jsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) =>
      value < 13 ? 'You must be 13 to make an account' : undefined,
    onBlur: ({ value }) =>
      value < 0 ? 'Invalid value' : undefined,
  }}
>
  {(field) => (
    <>
      <label htmlFor={field.name}>Age:</label>
      <input
        id={field.name}
        name={field.name}
        type="number"
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.valueAsNumber)}
        onBlur={field.handleBlur}
      />
      {!field.state.meta.isValid && (
        <em role="alert">{field.state.meta.errors.join(', ')}</em>
      )}
    </>
  )}
</form.Field>
```

**Component Breakdown:**
- `validators.onChange`: Runs on every keystroke; useful for immediate feedback.
- `validators.onBlur`: Runs when the field loses focus; useful for less urgent checks.
- Different rules can run at different times on the same field.

**Syntax Rules:**
- Use `onSubmit` (default) for short forms where re-renders are costly.
- Use `onBlur` for most fields; it provides feedback without distracting the user while typing.
- Use `onChange` only for fields where real-time feedback is essential (password strength, character counters).
- Set `reValidateMode: 'onChange'` so errors clear as the user corrects them.
- In TanStack Form, validation is configured per-field via the `validators` prop.
- Avoid validating on change for all fields in large forms; it causes excessive re-renders.

**Constraints and Limitations:**
- `onChange` validation causes a re-render on every keystroke, which can be slow for large forms.
- `onBlur` validation does not provide feedback while the user is still typing, which can be frustrating if the error is obvious.
- `onSubmit` validation can produce a wall of errors at once, overwhelming the user.
- The `all` mode validates on both blur and change, which can cause duplicate validations.
- Per-field triggers in TanStack Form require explicit configuration for each field.

### Annotated Code Examples

**Example 1: On-Blur with Re-validation on Change (React Hook Form)**

```jsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, touchedFields },
  } = useForm({
    mode: 'onBlur',           // Validate when the user leaves a field
    reValidateMode: 'onChange', // Re-validate on change after first submission
    defaultValues: { email: '', password: '' },
  });

  async function onSubmit(data) {
    await login(data);
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
          {...register('email', {
            required: 'Email is required',
            pattern: { value: /^\S+@\S+$/, message: 'Invalid email' },
          })}
        />
        
        {touchedFields.email && errors.email && (
          <span id="email-error" role="alert" style={{ color: 'red' }}>
            {errors.email.message}
          </span>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          aria-invalid={!!errors.password}
          aria-describedby={errors.password ? 'password-error' : undefined}
          {...register('password', {
            required: 'Password is required',
            minLength: { value: 8, message: 'At least 8 characters' },
          })}
        />

        {touchedFields.password && errors.password && (
          <span id="password-error" role="alert" style={{ color: 'red' }}>
            {errors.password.message}
          </span>
        )}
      </div>

      <button type="submit">Log In</button>
    </form>
  );
}
```

**Expected Output:** Errors appear when the user leaves a field with invalid input. After the first submission attempt, errors clear as the user types a correction. No error is shown before the user has interacted with a field.

**Why This Output Occurs:** `mode: 'onBlur'` delays validation until the user leaves the field, preventing premature error messages. `touchedFields.email` and `touchedFields.password` are `true` only after the field has been blurred, so errors are shown only for fields the user has interacted with. `reValidateMode: 'onChange'` ensures that once the user has attempted submission, errors clear as they type.

**Example 2: On-Change with Debounce (Manual)**

```jsx
import { useState, useEffect } from 'react';

function UsernameField() {
  const [username, setUsername] = useState('');
  const [error, setError] = useState(null);
  const [checking, setChecking] = useState(false);

  useEffect(() => {
    if (!username) {
      setError(null);
      return;
    }

    setChecking(true);
    const timer = setTimeout(async () => {
      const res = await fetch(`/api/check-username?u=${username}`);
      const { available } = await res.json();
      setError(available ? null : 'Username is taken');
      setChecking(false);
    }, 500); // Debounce 500ms

    return () => clearTimeout(timer);
  }, [username]);

  return (
    <div>
      <label htmlFor="username">Username</label>
      <input
        id="username"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        aria-invalid={!!error}
        aria-describedby={error ? 'username-error' : undefined}
      />
      {checking && <span>Checking...</span>}
      {error && (
        <span id="username-error" role="alert" style={{ color: 'red' }}>
          {error}
        </span>
      )}
    </div>
  );
}
```

**Expected Output:** As the user types, "Checking..." appears after a brief pause, then either the error "Username is taken" or no error. The check is debounced by 500ms.

**Why This Output Occurs:** The `useEffect` runs on every change to `username`, but the `setTimeout` delays the actual request by 500ms. If the user types another character within that window, the cleanup function clears the previous timer and starts a new one. This prevents a request per keystroke.

### Real-World Cases

- **Login forms:** `onBlur` validation for email and password; `onSubmit` for final validation.
- **Signup forms:** `onChange` for password strength; `onBlur` for email format.
- **Search inputs:** `onChange` with debouncing for live suggestions.
- **Multi-step wizards:** `onBlur` per field, `onSubmit` to advance to the next step.
- **Checkout forms:** `onBlur` for address and payment fields; `onSubmit` for final confirmation.

### References

- React Hook Form – useForm (`mode`, `reValidateMode`): https://react-hook-form.com/docs/useform#mode
- TanStack Form – Form and Field Validation: https://tanstack.com/form/latest/docs/framework/react/guides/validation
- Stack Overflow – How to submit with React Hook Form when having mode onBlur: https://stackoverflow.com/questions/75672987

---

## Core Concept 2: Built-in HTML5 Constraint Validation vs. Custom React Validation Logic

### Definitions

**Core Definition:** HTML5 constraint validation uses native browser attributes and the Constraint Validation API to validate form inputs, while custom React validation logic uses JavaScript functions, state, and libraries to evaluate input against arbitrary rules.

**Technical Definition:** The **Constraint Validation API** is a browser API that provides native validation for form controls. It defines validity states (e.g., `valueMissing`, `typeMismatch`, `patternMismatch`, `tooLong`, `tooShort`, `rangeUnderflow`, `rangeOverflow`, `stepMismatch`, `badInput`, `customError`) and methods (`checkValidity()`, `reportValidity()`, `setCustomValidity()`). HTML5 attributes (`required`, `type="email"`, `pattern`, `min`, `max`, `minlength`, `maxlength`, `step`) declare constraints declaratively. Custom React validation logic uses JavaScript to evaluate input against rules that are not expressible via HTML attributes—cross-field dependencies, business rules, or server-side checks. The `setCustomValidity()` method bridges the two: it allows custom React logic to set a native error message that the browser displays. React Hook Form's `shouldUseNativeValidation` option enables browser-native validation, and the `progressive` option forwards validation attributes to the DOM for progressive enhancement.

**Beginner-Friendly Explanation:** HTML5 validation is like a pre-printed form with checkboxes and format rules built in—the browser checks them automatically. Custom React validation is like having a personal assistant who reads your form and applies rules the browser doesn't know about—like "this password must match the one above it" or "this username must not already exist." The two work together: the browser handles the basics, and React handles the rest.

### Purposes

- **HTML5:** To provide zero-JavaScript validation for basic constraints (required, format, range).
- **HTML5:** To leverage the browser's built-in error UI (popovers) for immediate feedback.
- **HTML5:** To enable progressive enhancement—validation works before JavaScript loads.
- **Custom React:** To implement validation rules that HTML5 cannot express (cross-field, business logic).
- **Custom React:** To control error message content, placement, and styling.
- **Custom React:** To integrate async validation (server checks) and complex validation libraries.

### Syntax Rules and Structure

**HTML5 Constraint Validation Attributes:**
```html
<input type="email" required />                          <!-- Required + email format -->
<input type="number" min="18" max="120" />               <!-- Range validation -->
<input type="text" pattern="[A-Za-z]{3,}" />            <!-- Pattern validation -->
<input type="password" minlength="8" maxlength="64" />   <!-- Length validation -->
<input type="url" />                                     <!-- URL format -->
```

**Constraint Validation API (Programmatic):**
```jsx
const inputRef = useRef(null);

// Check validity
const isValid = inputRef.current.checkValidity();

// Show the browser's validation message
inputRef.current.reportValidity();

// Set a custom error message
inputRef.current.setCustomValidity('This username is already taken');

// Clear the custom error
inputRef.current.setCustomValidity('');
```

**Custom React Validation with React Hook Form:**
```jsx
const { register, formState: { errors } } = useForm();

<input
  {...register('email', {
    required: 'Email is required',
    pattern: { value: /^\S+@\S+$/, message: 'Invalid email format' },
    validate: {
      notCommon: (v) =>
        !['test@test.com', 'admin@admin.com'].includes(v) ||
        'This email is not allowed',
    },
  })}
/>
```

**Component Breakdown:**
- `required`, `pattern`: Built-in rules with custom messages.
- `validate.notCommon`: A custom synchronous validation function.
- The function returns `true` (valid) or a string (error message).

**Bridging Custom Logic to Native Validation:**
```jsx
function useNativeCustomValidation(inputRef, errorMessage) {
  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.setCustomValidity(errorMessage || '');
    }
  }, [errorMessage, inputRef]);
}
```

**Component Breakdown:**
- `setCustomValidity('')`: Clears the custom error.
- `setCustomValidity(message)`: Sets a native error message that the browser displays.

**Syntax Rules:**
- Use HTML5 attributes for basic constraints (required, type, min, max, minlength, maxlength, pattern).
- Use custom `validate` functions for rules HTML5 cannot express.
- Use `setCustomValidity()` to integrate custom errors with the browser's native validation UI.
- Set `noValidate` on the `<form>` to disable native validation when using a library.
- Use `shouldUseNativeValidation` in React Hook Form to enable browser-native validation.

**Constraints and Limitations:**
- HTML5 validation messages are browser-controlled and inconsistent across browsers.
- HTML5 validation does not support cross-field rules or async checks.
- Native validation popovers can be difficult to style or position.
- `setCustomValidity()` requires manual clearing; if not cleared, the field remains invalid.
- HTML5 validation does not work in older browsers; always validate on the server as a fallback.

### Annotated Code Example: Combining HTML5 and Custom Validation

```jsx
import { useForm } from 'react-hook-form';

function RegistrationForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({ mode: 'onBlur' });

  return (
    <form onSubmit={handleSubmit(data => alert(JSON.stringify(data)))}>
      {/* HTML5: required + email type */}
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          required
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
          {...register('email', {
            required: 'Email is required',
            pattern: { value: /^\S+@\S+$/, message: 'Invalid email' },
          })}
        />
        {errors.email && (
          <span id="email-error" role="alert" style={{ color: 'red' }}>
            {errors.email.message}
          </span>
        )}
      </div>

      {/* Custom: business rule not expressible in HTML5 */}
      <div>
        <label htmlFor="username">Username</label>
        <input
          id="username"
          aria-invalid={!!errors.username}
          aria-describedby={errors.username ? 'username-error' : undefined}
          {...register('username', {
            required: 'Username is required',
            minLength: { value: 3, message: 'At least 3 characters' },
            validate: {
              notReserved: (v) =>
                !['admin', 'root', 'system'].includes(v.toLowerCase()) ||
                'This username is reserved',
            },
          })}
        />
        {errors.username && (
          <span id="username-error" role="alert" style={{ color: 'red' }}>
            {errors.username.message}
          </span>
        )}
      </div>

      <button type="submit">Register</button>
    </form>
  );
}
```

**Expected Output:** The email field uses both HTML5 `type="email"` and `required` attributes (which React Hook Form respects) plus a custom pattern message. The username field uses a custom `validate.notReserved` function to reject reserved names. Errors appear on blur with custom messages.

**Why This Output Occurs:** The `type="email"` and `required` attributes provide native browser validation, but React Hook Form overrides the native messages with its own. The `validate.notReserved` function is pure custom logic that HTML5 cannot express. Both validation layers run, and the first error (in order of registration) is displayed.

### Real-World Cases

- **HTML5:** Simple contact forms, search inputs, and any form where basic constraints are sufficient.
- **Custom React:** Signup forms with reserved usernames, checkout forms with coupon validation, and any form with business rules.
- **Combined:** Login forms (HTML5 for email format, custom for password strength), registration forms (HTML5 for required, custom for uniqueness).

### References

- MDN Web Docs – Constraint Validation: https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation
- MDN Web Docs – Constraint Validation API: https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation
- GitNation – Replacing Form Libraries With Native Web APIs: https://gitnation.com/contents/replacing-form-libraries-with-native-web-apis
- React Hook Form – shouldUseNativeValidation: https://react-hook-form.com/docs/useform#shouldUseNativeValidation

---

## Core Concept 3: Basic Constraints (Required, Type, Range, Pattern/Regex)

### Definitions

**Core Definition:** Basic constraints are the fundamental validation rules applied to individual form fields—required fields, type validation (email, number, URL), range validation (min/max for numbers and dates), and pattern/regex validation for custom formats.

**Technical Definition:** Basic constraints are the most common validation rules and are expressible in both HTML5 and React Hook Form. **Required** ensures a field is not empty (`required` attribute; `required: 'message'` in RHF). **Type validation** uses the `type` attribute (`email`, `number`, `url`, `tel`, `date`) to enforce format expectations. **Range validation** uses `min` and `max` attributes for numbers, dates, and other types with a range (`min: 18, max: 120`). **Length validation** uses `minlength` and `maxlength` for strings. **Pattern validation** uses the `pattern` attribute or RHF's `pattern` option with a regular expression. React Hook Form maps these attributes to its own validation rules, allowing custom error messages for each. The rules can be combined on a single field, and RHF evaluates them in order.

**Beginner-Friendly Explanation:** Basic constraints are like the rules on a paper form: "This field is required," "Enter a valid email address," "Age must be between 18 and 120," "Phone number must be 10 digits." They are the everyday checks that catch most mistakes before the form is submitted.

### Purposes

- **Required:** To ensure critical fields (email, password, name) are not left empty.
- **Type:** To ensure the input format matches the expected data type (email, URL, number).
- **Range:** To enforce minimum and maximum values for numbers, dates, and lengths.
- **Pattern/Regex:** To enforce custom formats (phone numbers, postal codes, credit card numbers).
- **Combined:** To apply multiple rules to a single field, each with its own error message.

### Syntax Rules and Structure

**HTML5 Attributes:**
```html
<input type="email" required />
<input type="number" min="18" max="120" />
<input type="text" minlength="3" maxlength="20" />
<input type="text" pattern="[A-Za-z]{3,}" />
<input type="url" />
```

**React Hook Form Registration:**
```jsx
<input
  {...register('email', {
    required: 'Email is required',
    pattern: { value: /^\S+@\S+$/, message: 'Invalid email format' },
  })}
/>

<input
  type="number"
  {...register('age', {
    required: 'Age is required',
    min: { value: 18, message: 'Must be at least 18' },
    max: { value: 120, message: 'Must be at most 120' },
    valueAsNumber: true,
  })}
/>

<input
  {...register('password', {
    required: 'Password is required',
    minLength: { value: 8, message: 'At least 8 characters' },
    maxLength: { value: 64, message: 'At most 64 characters' },
  })}
/>

<input
  {...register('phone', {
    required: 'Phone is required',
    pattern: {
      value: /^\+?[1-9]\d{1,14}$/,
      message: 'Invalid phone number',
    },
  })}
/>
```

**Component Breakdown:**
- `required`: The field must not be empty.
- `pattern`: The value must match the regular expression.
- `min` / `max`: The value must be within the range.
- `minLength` / `maxLength`: The string length must be within the range.
- `valueAsNumber`: Converts the string value to a number for numeric validation.

**Syntax Rules:**
- Use `required` for essential fields.
- Use `type="email"` and `type="url"` for format validation, or a `pattern` for custom formats.
- Use `min` and `max` for numeric ranges; use `minLength` and `maxLength` for string lengths.
- Use `valueAsNumber: true` for numeric fields so RHF validates numbers, not strings.
- Provide custom error messages for every rule to give users specific feedback.
- Combine rules on a single field; RHF evaluates them in order and shows the first error by default.

**Constraints and Limitations:**
- `pattern` is anchored automatically (the entire value must match), so `^` and `$` are not needed.
- `min`/`max` work on strings for numeric types only if `valueAsNumber` is used; otherwise, string comparison is used.
- `required` does not work with `type="checkbox"` in the same way; use `validate` for checkbox groups.
- HTML5 `pattern` does not support lookaheads or other advanced regex features in all browsers.
- Overly strict patterns can reject valid input (e.g., international phone numbers); test thoroughly.

### Annotated Code Example: Multi-Rule Field

```jsx
import { useForm } from 'react-hook-form';

function ProfileForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({ mode: 'onBlur' });

  return (
    <form onSubmit={handleSubmit(data => alert(JSON.stringify(data)))}>
      {/* Required + pattern */}
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
          {...register('email', {
            required: 'Email is required',
            pattern: { value: /^\S+@\S+$/, message: 'Invalid email format' },
          })}
        />
        {errors.email && (
          <span id="email-error" role="alert" style={{ color: 'red' }}>
            {errors.email.message}
          </span>
        )}
      </div>

      {/* Range validation */}
      <div>
        <label htmlFor="age">Age</label>
        <input
          id="age"
          type="number"
          aria-invalid={!!errors.age}
          aria-describedby={errors.age ? 'age-error' : undefined}
          {...register('age', {
            required: 'Age is required',
            min: { value: 18, message: 'You must be at least 18' },
            max: { value: 120, message: 'Please enter a realistic age' },
            valueAsNumber: true,
          })}
        />
        {errors.age && (
          <span id="age-error" role="alert" style={{ color: 'red' }}>
            {errors.age.message}
          </span>
        )}
      </div>

      {/* Length validation */}
      <div>
        <label htmlFor="username">Username</label>
        <input
          id="username"
          aria-invalid={!!errors.username}
          aria-describedby={errors.username ? 'username-error' : undefined}
          {...register('username', {
            required: 'Username is required',
            minLength: { value: 3, message: 'At least 3 characters' },
            maxLength: { value: 20, message: 'At most 20 characters' },
          })}
        />
        {errors.username && (
          <span id="username-error" role="alert" style={{ color: 'red' }}>
            {errors.username.message}
          </span>
        )}
      </div>

      <button type="submit">Save Profile</button>
    </form>
  );
}
```

**Expected Output:** Submitting with an empty email shows "Email is required". Entering an invalid email shows "Invalid email format". Entering an age below 18 shows "You must be at least 18". Entering a username shorter than 3 characters shows "At least 3 characters". All errors appear on blur.

**Why This Output Occurs:** Each field is registered with multiple validation rules. React Hook Form evaluates them in order and displays the first error message. `valueAsNumber: true` ensures the age field is validated as a number, not a string. The `aria-invalid` and `aria-describedby` attributes make the errors accessible.

### Real-World Cases

- **Signup forms:** Email (type + required), password (minLength + pattern), age (min/max).
- **Checkout forms:** Postal code (pattern), phone (pattern), credit card (pattern + length).
- **Profile forms:** Username (required + minLength), bio (maxLength).
- **Admin forms:** ID fields (pattern), date fields (min/max), numeric fields (range).
- **Surveys:** Rating scales (min/max), text responses (required + maxLength).

### References

- React Hook Form – register (validation rules): https://react-hook-form.com/docs/useform/register
- MDN Web Docs – Client-side form validation: https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation
- AbstractAPI – How to Validate Forms in React: https://www.abstractapi.com/guides/email-validation/react-form-validation

---

## Core Concept 4: Dependent and Cross-Field Validation (Password Confirmation, Date Ranges)

### Definitions

**Core Definition:** Dependent and cross-field validation evaluates a field's validity based on the value of one or more other fields, such as confirming that a password and its confirmation match, or that an end date is after a start date.

**Technical Definition:** Cross-field validation rules depend on multiple fields' values. In React Hook Form, custom `validate` functions receive the field's value as the first argument and the entire form values object as the second argument, enabling rules like `validate: (value, formValues) => value === formValues.password || 'Passwords do not match'`. In schema validation (Zod, Yup), cross-field rules are expressed using `.refine()` (Zod) or `.ref()` (Yup), which have access to the entire schema. For date ranges, the rule checks that the end date is after the start date. React Hook Form's `trigger` method can manually trigger validation on dependent fields when one field changes. The pattern is essential for password confirmation, date-range validation, conditional required fields ("required if another field is enabled"), and dynamic limits.

**Beginner-Friendly Explanation:** Cross-field validation is like checking that two puzzle pieces fit together. The "confirm password" field is only valid if it matches the "password" field. The "end date" is only valid if it comes after the "start date." These rules cannot be checked by looking at one field alone—they need to compare multiple fields.

### Purposes

- **Password confirmation:** To ensure the user typed the same password twice.
- **Date ranges:** To ensure the end date is after the start date.
- **Conditional required:** To make a field required only if another field has a specific value.
- **Dynamic limits:** To enforce a maximum based on another field's value.
- **Dependent selects:** To validate that a selected option is valid for the chosen category.

### Syntax Rules and Structure

**React Hook Form with `validate` Function:**
```jsx
<input
  {...register('confirmPassword', {
    validate: (value, formValues) =>
      value === formValues.password || 'Passwords do not match',
  })}
/>
```

**Component Breakdown:**
- `value`: The current value of `confirmPassword`.
- `formValues`: The entire form values object.
- The function returns `true` (valid) or a string (error message).

**Date Range Validation:**
```jsx
<input
  type="date"
  {...register('startDate', { required: 'Start date is required' })}
/>

<input
  type="date"
  {...register('endDate', {
    required: 'End date is required',
    validate: (value, formValues) =>
      !formValues.startDate ||
      new Date(value) > new Date(formValues.startDate) ||
      'End date must be after start date',
  })}
/>
```

**Zod Schema with `.refine()`:**
```jsx
const schema = z.object({
  password: z.string().min(8, 'At least 8 characters'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'], // Attach error to confirmPassword field
});
```

**Component Breakdown:**
- `.refine()`: Adds a custom validation rule to the schema.
- `path: ['confirmPassword']`: Attaches the error message to the `confirmPassword` field, not the form root.

**Manual Trigger for Dependent Fields:**
```jsx
const { register, trigger, formState: { errors } } = useForm();

<input
  {...register('password', {
    onChange: () => trigger('confirmPassword'), // Re-validate confirmPassword when password changes
  })}
/>

<input
  {...register('confirmPassword', {
    validate: (value, formValues) =>
      value === formValues.password || 'Passwords do not match',
  })}
/>
```

**Syntax Rules:**
- Use `validate` with the second `formValues` argument for cross-field rules in React Hook Form.
- Use `.refine()` in Zod or `.ref()` in Yup for schema-level cross-field validation.
- Use `path` in Zod's `.refine()` to attach the error to the correct field.
- Use `trigger('fieldName')` to manually re-validate dependent fields when the source field changes.
- Provide clear error messages that identify which field caused the error.

**Constraints and Limitations:**
- Cross-field validation cannot be expressed with HTML5 attributes alone.
- Validation order matters: if the source field is invalid, the dependent field's validation may also fail.
- In React Hook Form, `validate` functions run on every validation cycle for the field; keep them fast.
- Zod's `.refine()` runs after all field-level rules pass; if a field is invalid, `.refine()` may not run.
- Date comparisons require careful handling of time zones and invalid date objects.

### Annotated Code Example: Password Confirmation with Zod

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  password: z.string().min(8, 'At least 8 characters'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});

function PasswordForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: zodResolver(schema),
    mode: 'onBlur',
  });

  return (
    <form onSubmit={handleSubmit(data => alert('Passwords match!'))}>
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          aria-invalid={!!errors.password}
          aria-describedby={errors.password ? 'password-error' : undefined}
          {...register('password')}
        />
        {errors.password && (
          <span id="password-error" role="alert" style={{ color: 'red' }}>
            {errors.password.message}
          </span>
        )}
      </div>

      <div>
        <label htmlFor="confirmPassword">Confirm Password</label>
        <input
          id="confirmPassword"
          type="password"
          aria-invalid={!!errors.confirmPassword}
          aria-describedby={errors.confirmPassword ? 'confirm-error' : undefined}
          {...register('confirmPassword')}
        />
        {errors.confirmPassword && (
          <span id="confirm-error" role="alert" style={{ color: 'red' }}>
            {errors.confirmPassword.message}
          </span>
        )}
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** Submitting with a password shorter than 8 characters shows "At least 8 characters". Submitting with mismatched passwords shows "Passwords do not match" on the confirm field. Submitting with matching passwords shows an alert.

**Why This Output Occurs:** The Zod schema validates `password` with `min(8)` and then uses `.refine()` to check that `password` equals `confirmPassword`. The `path: ['confirmPassword']` attaches the refinement error to the `confirmPassword` field, so it appears below that input. The `zodResolver` integrates the schema with React Hook Form.

### Real-World Cases

- **Signup forms:** Password confirmation, email confirmation.
- **Booking forms:** Check-out date after check-in date.
- **Financial forms:** Transfer amount not exceeding account balance.
- **Admin forms:** End date after start date for campaigns or promotions.
- **Surveys:** Conditional questions based on previous answers.

### References

- React Hook Form – Custom Validation Functions: https://deepwiki.com/react-hook-form/react-hook-form/4.3-custom-validation-functions
- React Hook Form – trigger: https://react-hook-form.com/docs/useform/trigger
- Zod – Refinements: https://zod.dev/?id=refine
- AirLib – Cross-Field Matching: https://airlib.dev/posts/building-forms-with-air-form

---

## Core Concept 5: Async and Server-Side Validation Integration (Database Checks and API Error Mapping)

### Definitions

**Core Definition:** Async and server-side validation integration is the practice of validating form input against remote resources (database uniqueness, API business rules) and mapping server-returned errors back to the appropriate form fields.

**Technical Definition:** Asynchronous validation requires a network round trip to a server. Common use cases include checking username or email availability, validating coupon codes, and verifying account balances. The implementation must handle **debouncing** (delaying the request until the user pauses typing), **request cancellation** (aborting in-flight requests when the input changes), and **race condition guards** (ignoring stale responses that arrive after a newer response). React Hook Form supports async validation via `validate` functions that return a `Promise<true | string>`. Server-side validation occurs after the form is submitted; the server validates the data and returns a structured error response (e.g., `{ errors: { email: 'Email already exists' } }`), which the client maps back to the form fields. React Hook Form's `setError` method programmatically sets field errors, and the `errors` prop can be used to inject server errors. Zod and Yup schemas can be shared between client and server for consistent validation.

**Beginner-Friendly Explanation:** Async validation is like calling a librarian to check if a book is available. You don't want to call on every keystroke—that would be annoying and slow. So you wait until the user pauses typing (debounce), and if they start typing again, you hang up and call again (cancel the previous request). Once the server responds, you show the result. Server-side validation is like the librarian checking the shelf after you submit your request—if the book is already checked out, they tell you, and you show that error next to the book title.

### Purposes

- **Username/email availability:** To check if a username or email is already taken.
- **Coupon validation:** To verify a coupon code is valid and applicable.
- **Account checks:** To verify an account number or routing number.
- **Server-side business rules:** To enforce rules that cannot be checked on the client (inventory, permissions).
- **Error mapping:** To display server-returned errors next to the correct form fields.

### Syntax Rules and Structure

**Async Validation with React Hook Form (Debounce + AbortController):**
```jsx
import { useForm } from 'react-hook-form';
import { useMemo, useId } from 'react';

function createUsernameValidator(delayMs = 400) {
  let timer = null;
  let controller = null;

  return (value) => new Promise((resolve) => {
    if (value.length < 3) return resolve('At least 3 characters');

    if (timer) clearTimeout(timer);
    controller?.abort();
    controller = new AbortController();

    timer = setTimeout(async () => {
      try {
        const res = await fetch(
          `/api/username-available?u=${encodeURIComponent(value)}`,
          { signal: controller.signal }
        );
        const { available } = await res.json();
        resolve(available ? true : 'That username is taken');
      } catch (err) {
        if (err.name === 'AbortError') return; // Newer keystroke cancelled us
        resolve('Could not check availability — try again');
      }
    }, delayMs);
  });
}

function UsernameForm() {
  const validateUsername = useMemo(() => createUsernameValidator(), []);
  const {
    register,
    handleSubmit,
    formState: { errors, isValidating, isSubmitting },
  } = useForm({ mode: 'onChange', defaultValues: { username: '' } });

  const err = errors.username;

  return (
    <form onSubmit={handleSubmit(data => alert(`Welcome, ${data.username}!`))}>
      <div>
        <label htmlFor="username">Username</label>
        <input
          id="username"
          aria-invalid={err ? 'true' : undefined}
          aria-describedby={err ? 'username-error' : undefined}
          {...register('username', { validate: validateUsername })}
        />
        <p id="username-error" role="alert" aria-live="polite">
          {isValidating ? 'Checking availability…' : err?.message}
        </p>
      </div>

      <button type="submit" disabled={isSubmitting || isValidating}>
        Sign Up
      </button>
    </form>
  );
}
```

**Component Breakdown:**
- `createUsernameValidator`: A factory that owns the debounce timer and `AbortController`.
- `controller?.abort()`: Cancels the previous in-flight request.
- `err.name === 'AbortError'`: Ignores the error from an aborted request.
- `isValidating`: `true` while the async validation is running.
- `aria-live="polite"`: Announces the "Checking availability…" message to screen readers.

**Server-Side Error Mapping with `setError`:**
```jsx
const { setError, handleSubmit } = useForm();

async function onSubmit(data) {
  const response = await fetch('/api/register', {
    method: 'POST',
    body: JSON.stringify(data),
    headers: { 'Content-Type': 'application/json' },
  });

  if (!response.ok) {
    const { errors } = await response.json();
    // Map server errors to form fields
    Object.entries(errors).forEach(([field, message]) => {
      setError(field, { type: 'server', message });
    });
    return;
  }

  // Success
}
```

**Component Breakdown:**
- `setError(field, { type, message })`: Programmatically sets an error on a field.
- `errors`: The server's error object, keyed by field name.
- The client maps server errors to the correct fields.

**Syntax Rules:**
- Debounce async validation by 300–500ms to avoid a request per keystroke.
- Use `AbortController` to cancel in-flight requests when the input changes.
- Use an `ignore` flag or check `err.name === 'AbortError'` to ignore stale results.
- Show a pending indicator (`isValidating`) while the async check is running.
- Use `setError` to map server-returned errors to form fields.
- Share Zod/Yup schemas between client and server for consistent validation.
- Always validate on the server; client-side validation is for UX, not security.

**Constraints and Limitations:**
- Async validation adds latency; the user must wait for the server response.
- Without debouncing, a request fires on every keystroke, overwhelming the server.
- Without cancellation, stale responses can overwrite newer results (race condition).
- Server error formats vary; the client must know the expected shape.
- `setError` does not clear automatically; call `clearErrors` or re-validate after correction.

### Annotated Code Example: Async Email Availability with Race Condition Guard

```jsx
import { useState, useEffect, useRef } from 'react';

function EmailField() {
  const [email, setEmail] = useState('');
  const [status, setStatus] = useState('idle'); // idle | checking | available | taken
  const [error, setError] = useState(null);
  const abortRef = useRef(null);

  useEffect(() => {
    if (!email || !email.includes('@')) {
      setStatus('idle');
      setError(null);
      return;
    }

    setStatus('checking');
    setError(null);

    const timer = setTimeout(async () => {
      abortRef.current?.abort();
      const controller = new AbortController();
      abortRef.current = controller;

      try {
        const res = await fetch(`/api/check-email?e=${encodeURIComponent(email)}`, {
          signal: controller.signal,
        });
        const { available } = await res.json();
        setStatus(available ? 'available' : 'taken');
        setError(available ? null : 'This email is already registered');
      } catch (err) {
        if (err.name === 'AbortError') return;
        setStatus('idle');
        setError('Could not check email — try again');
      }
    }, 500); // Debounce 500ms

    return () => {
      clearTimeout(timer);
      abortRef.current?.abort();
    };
  }, [email]);

  return (
    <div>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        aria-invalid={!!error}
        aria-describedby={error ? 'email-error' : 'email-status'}
      />
      {status === 'checking' && <span id="email-status">Checking availability…</span>}
      {status === 'available' && <span id="email-status" style={{ color: 'green' }}>Available</span>}
      {error && <span id="email-error" role="alert" style={{ color: 'red' }}>{error}</span>}
    </div>
  );
}
```

**Expected Output:** As the user types a valid email, after a 500ms pause, "Checking availability…" appears, then either "Available" or "This email is already registered". If the user types again during the check, the previous request is cancelled.

**Why This Output Occurs:** The `useEffect` runs on every change to `email`. The `setTimeout` debounces the request by 500ms. The cleanup function clears the timer and aborts any in-flight request. The `AbortController` ensures that stale responses are ignored. The status state drives the UI feedback.

### Real-World Cases

- **Signup forms:** Username and email availability checks.
- **E-commerce:** Coupon code validation and gift card balance checks.
- **Banking:** Account number and routing number validation.
- **Booking systems:** Availability checks for dates and rooms.
- **Multi-tenant apps:** Subdomain availability checks.

### References

- React Hook Form – Async Field Validation: https://js-form-validation.com/framework-integration-patterns/react-hook-form-validation/react-hook-form-async-field-validation/
- React Hook Form – setError: https://react-hook-form.com/docs/useform/seterror
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- TanStack Form – Server Error Mapping: https://github.com/TanStack/form/discussions/1708

---

## Core Concept 6: Accessibility (a11y) in Validation

### Definitions

**Core Definition:** Accessibility in validation is the practice of ensuring that validation errors are perceivable, understandable, and operable by all users, including those using screen readers, keyboards, and other assistive technologies.

**Technical Definition:** Accessible validation requires four key elements. First, every input must have a programmatically associated label (`<label htmlFor="id">` or `aria-label`). Second, error messages must be associated with their inputs via `aria-describedby`, which points to the error message's `id`. Third, invalid inputs must be marked with `aria-invalid="true"` (or `"false"` when valid), which screen readers announce. Fourth, focus must be managed so that on submission failure, focus moves to the first invalid field, and error messages should be announced via `role="alert"` or a live region (`aria-live="assertive"`). Required fields should be marked with `aria-required="true"` or the native `required` attribute. Fieldsets and legends group related fields. The WCAG 2.2 AA guidelines require error identification (3.3.1), labels or instructions (3.3.2), and error suggestion (3.3.3).

**Beginner-Friendly Explanation:** Imagine you cannot see the screen and are filling out a form using a screen reader. When you make a mistake, you need to hear that there is an error and what the error is. `aria-invalid` tells the screen reader "this field has a problem." `aria-describedby` tells it "read the error message associated with this field." `role="alert"` makes the error message announce itself immediately. And moving focus to the first error means you don't have to hunt through the form to find what went wrong.

### Purposes

- To ensure screen-reader users can perceive and understand validation errors.
- To ensure keyboard users can navigate to and correct invalid fields.
- To comply with WCAG 2.2 AA accessibility guidelines.
- To provide a consistent, predictable error experience for all users.
- To announce errors dynamically without requiring the user to re-read the entire form.

### Syntax Rules and Structure

**Accessible Error Pattern (React Hook Form):**
```jsx
import { useForm } from 'react-hook-form';
import { useId } from 'react';

function AccessibleField() {
  const {
    register,
    formState: { errors },
  } = useForm({ mode: 'onBlur' });

  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div>
      <label htmlFor={id}>Email</label>
      <input
        id={id}
        type="email"
        aria-required="true"
        aria-invalid={errors.email ? 'true' : 'false'}
        aria-describedby={errors.email ? errorId : undefined}
        {...register('email', {
          required: 'Email is required',
          pattern: { value: /^\S+@\S+$/, message: 'Invalid email' },
        })}
      />
      {errors.email && (
        <span id={errorId} role="alert" style={{ color: 'red' }}>
          {errors.email.message}
        </span>
      )}
    </div>
  );
}
```

**Component Breakdown:**
- `aria-required="true"`: Indicates the field is required.
- `aria-invalid="true" | "false"`: Indicates whether the field is currently invalid.
- `aria-describedby={errorId}`: Associates the input with its error message.
- `role="alert"`: Announces the error message when it appears.
- `id` and `htmlFor`: Associates the label with the input.

**Focus Management on Submit Failure:**
```jsx
import { useEffect, useRef } from 'react';

const firstErrorRef = useRef(null);

useEffect(() => {
  if (Object.keys(errors).length > 0) {
    firstErrorRef.current?.focus();
  }
}, [errors]);

// In JSX: attach ref to the first field that has an error
<input
  ref={Object.keys(errors)[0] === 'email' ? firstErrorRef : undefined}
  {...register('email')}
/>
```

**Component Breakdown:**
- `firstErrorRef`: A ref attached to the first field with an error.
- `useEffect`: Runs when `errors` changes; focuses the first error field.

**Live Region for Form-Level Errors:**
```jsx
{Object.keys(errors).length > 0 && (
  <div role="alert" aria-live="assertive" aria-atomic="true">
    Form has {Object.keys(errors).length} errors. Please review.
  </div>
)}
```

**Component Breakdown:**
- `role="alert"`: Announces the message immediately.
- `aria-live="assertive"`: Interrupts the screen reader to announce.
- `aria-atomic="true"`: Announces the entire message, not just the change.

**Syntax Rules:**
- Every input must have a label (`<label htmlFor>`, `aria-label`, or `aria-labelledby`).
- Use `aria-invalid="true"` (and `"false"` when valid) on invalid inputs.
- Use `aria-describedby` to link inputs to their error messages.
- Use `role="alert"` or `aria-live="assertive"` for error messages that appear dynamically.
- Move focus to the first invalid field on submission failure.
- Use `aria-required="true"` or the native `required` attribute for required fields.
- Group related fields with `<fieldset>` and `<legend>`.
- Do not rely on colour alone to indicate errors; include text and icons.

**Constraints and Limitations:**
- `role="alert"` can be disruptive if used excessively; reserve it for important errors.
- `aria-live` regions must be present in the DOM before content is inserted; dynamically created live regions may not announce.
- Focus management can be jarring if not handled carefully; focus should only move when the user submits.
- Screen readers handle `aria-describedby` differently; test with multiple screen readers.
- Some accessibility attributes (`aria-invalid`) are not supported by all older browsers.

### Annotated Code Example: Fully Accessible Login Form

```jsx
import { useForm } from 'react-hook-form';
import { useId, useEffect, useRef } from 'react';

function AccessibleLoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({ mode: 'onBlur' });

  const emailId = useId();
  const passwordId = useId();
  const emailErrorId = `${emailId}-error`;
  const passwordErrorId = `${passwordId}-error`;
  const firstErrorRef = useRef(null);

  // Focus the first error field on submission failure
  useEffect(() => {
    if (Object.keys(errors).length > 0 && firstErrorRef.current) {
      firstErrorRef.current.focus();
    }
  }, [errors]);

  const firstErrorField = Object.keys(errors)[0];

  return (
    <form onSubmit={handleSubmit(data => alert('Logged in!'))} noValidate>
      {/* Form-level error summary */}
      {Object.keys(errors).length > 0 && (
        <div role="alert" aria-live="assertive" aria-atomic="true"
             style={{ color: 'red', marginBottom: 16 }}>
          Form has {Object.keys(errors).length} error(s). Please review.
        </div>
      )}

      {/* Email field */}
      <div style={{ marginBottom: 16 }}>
        <label htmlFor={emailId}>
          Email <span aria-hidden="true">*</span>
        </label>
        <input
          id={emailId}
          type="email"
          ref={firstErrorField === 'email' ? firstErrorRef : undefined}
          aria-required="true"
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? emailErrorId : undefined}
          {...register('email', {
            required: 'Email is required',
            pattern: { value: /^\S+@\S+$/, message: 'Invalid email format' },
          })}
        />
        {errors.email && (
          <span id={emailErrorId} role="alert" style={{ color: 'red' }}>
            {errors.email.message}
          </span>
        )}
      </div>

      {/* Password field */}
      <div style={{ marginBottom: 16 }}>
        <label htmlFor={passwordId}>
          Password <span aria-hidden="true">*</span>
        </label>
        <input
          id={passwordId}
          type="password"
          ref={firstErrorField === 'password' ? firstErrorRef : undefined}
          aria-required="true"
          aria-invalid={errors.password ? 'true' : 'false'}
          aria-describedby={errors.password ? passwordErrorId : undefined}
          {...register('password', {
            required: 'Password is required',
            minLength: { value: 8, message: 'At least 8 characters' },
          })}
        />
        {errors.password && (
          <span id={passwordErrorId} role="alert" style={{ color: 'red' }}>
            {errors.password.message}
          </span>
        )}
      </div>

      <button type="submit">Log In</button>
    </form>
  );
}
```

**Expected Output:** Submitting with empty fields shows a form-level error summary, an error message below each invalid field, and moves focus to the first invalid field. Screen readers announce the error summary and the field-specific errors.

**Why This Output Occurs:** `aria-invalid` tells the screen reader the field is invalid. `aria-describedby` links the input to its error message. `role="alert"` on the error message announces it. The `useEffect` focuses the first error field. The `noValidate` attribute disables native browser validation so the custom accessible messages are used. `aria-required` indicates required fields.

### Real-World Cases

- **Login forms:** Accessible email and password validation with focus management.
- **Signup forms:** Accessible password confirmation and username availability.
- **Checkout forms:** Accessible payment and address validation.
- **Surveys:** Accessible error summaries for long forms.
- **Admin panels:** Accessible CRUD forms with server-side error mapping.

### References

- React Hook Form + Zod – Accessibility Best Practices: https://github.com/secondsky/claude-skills/blob/main/plugins/react-hook-form-zod/skills/react-hook-form-zod/references/accessibility.md
- WCAG 2.2 – Error Identification (3.3.1): https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html
- WCAG 2.2 – Labels or Instructions (3.3.2): https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html
- WCAG 2.2 – Error Suggestion (3.3.3): https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html
- WAI-ARIA – Alert Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/alert/

---

## Comparison and Decision Guidance

| Validation Approach | Trigger | Best For | Performance | Accessibility |
|---|---|---|---|---|
| **HTML5 Constraint Validation** | On submit (browser-controlled) | Simple forms, progressive enhancement | Excellent (native) | Native browser UI |
| **React Hook Form `onSubmit`** | On submit | Short forms, wizards | Excellent (minimal re-renders) | Requires manual ARIA |
| **React Hook Form `onBlur`** | On blur | Most forms (balanced) | Good | Requires manual ARIA |
| **React Hook Form `onChange`** | On change | Password strength, character counters | Poor (re-renders per keystroke) | Requires manual ARIA |
| **Async Validation** | Debounced on change | Username/email availability | Depends on debounce | Requires `aria-live` |
| **Server-Side Validation** | On submit | Business rules, security | N/A (server-side) | Requires `setError` mapping |
| **Cross-Field Validation** | On blur or submit | Password confirmation, date ranges | Good | Requires manual ARIA |

**Decision Guidance:**
- **Start with HTML5 constraints** for basic rules (required, email, min/max); they work without JavaScript.
- **Use React Hook Form** for all forms beyond trivial cases; it provides structured validation, error state, and performance.
- **Use `onBlur` as the default trigger** — it balances feedback with performance.
- **Use `onChange` only for fields where real-time feedback is critical** (password strength, character counters).
- **Use `validate` functions** for rules HTML5 cannot express (cross-field, business logic).
- **Use Zod or Yup** for schema validation when rules are complex or need to be shared between client and server.
- **Debounce and cancel async validation** to avoid race conditions and server overload.
- **Map server errors with `setError`** so they appear next to the correct fields.
- **Always implement accessibility** — `aria-invalid`, `aria-describedby`, `role="alert"`, and focus management are not optional.
- **Never rely on client-side validation alone** — always validate on the server for security.

---

## References

- React Hook Form – useForm (`mode`, `reValidateMode`): https://react-hook-form.com/docs/useform#mode
- React Hook Form – register (validation rules): https://react-hook-form.com/docs/useform/register
- React Hook Form – setError: https://react-hook-form.com/docs/useform/seterror
- React Hook Form – trigger: https://react-hook-form.com/docs/useform/trigger
- React Hook Form – Custom Validation Functions: https://deepwiki.com/react-hook-form/react-hook-form/4.3-custom-validation-functions
- React Hook Form – Async Field Validation: https://js-form-validation.com/framework-integration-patterns/react-hook-form-validation/react-hook-form-async-field-validation/
- React Hook Form + Zod – Accessibility Best Practices: https://github.com/secondsky/claude-skills/blob/main/plugins/react-hook-form-zod/skills/react-hook-form-zod/references/accessibility.md
- TanStack Form – Form and Field Validation: https://tanstack.com/form/latest/docs/framework/react/guides/validation
- TanStack Form – Server Error Mapping: https://github.com/TanStack/form/discussions/1708
- MDN Web Docs – Constraint Validation: https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation
- MDN Web Docs – Client-side form validation: https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- GitNation – Replacing Form Libraries With Native Web APIs: https://gitnation.com/contents/replacing-form-libraries-with-native-web-apis
- AbstractAPI – How to Validate Forms in React: https://www.abstractapi.com/guides/email-validation/react-form-validation
- Zod – Refinements: https://zod.dev/?id=refine
- WCAG 2.2 – Error Identification (3.3.1): https://www.w3.org/WAI/WCAG22/Understanding/error-identification.html
- WCAG 2.2 – Labels or Instructions (3.3.2): https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html
- WCAG 2.2 – Error Suggestion (3.3.3): https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion.html
- WAI-ARIA – Alert Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/alert/