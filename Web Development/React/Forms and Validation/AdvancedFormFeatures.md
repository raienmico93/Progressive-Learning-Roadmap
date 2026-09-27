# React Advanced Form Features: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React advanced form features encompass the patterns and techniques used to build production-grade forms that go beyond simple input collection—including dynamic field arrays, multi-step wizards, file uploads, asynchronous validation, draft persistence, and accessible error handling.

**Technical Definition:** React advanced form features are the architectural patterns and library APIs that address the complex requirements of real-world forms. These include `useFieldArray` for dynamic list management with stable identity keys, wizard orchestration with parent-owned state and per-step validation gates, file upload pipelines with drag-and-drop and progress tracking, debounced asynchronous field validation with request cancellation, storage-backed draft persistence with restore-on-mount, and error UX patterns that prioritise accessibility through `aria-invalid`, `aria-describedby`, focus management, and error summaries. Each feature addresses a distinct failure mode: lost field identity during reordering, lost user progress during step transitions, no feedback during slow uploads, race conditions from rapid typing, data loss on refresh, and inaccessibility for screen-reader users.

**Beginner-Friendly Explanation:** Basic forms let users type text and click submit. Advanced forms handle everything that makes real-world forms hard: letting users add and reorder multiple items (like line items on an invoice), guiding them through multi-step processes without losing their work, uploading files with a progress bar, checking with the server whether a username is taken without spamming it, saving drafts so a refresh doesn't wipe everything, and making sure screen-reader users can find and fix errors.

### Key Characteristics

- **Stable Identity:** Dynamic fields use auto-generated `id` keys (not array indices) so React can track items correctly during reordering, insertion, and removal.
- **Parent-Owned Draft State:** Multi-step wizards keep all form state in a persistent parent shell, so step transitions never discard data.
- **Asynchronous Safety:** Debounced validation and request cancellation prevent race conditions and server overload.
- **Storage-Backed Recovery:** Draft persistence writes to `localStorage` or `sessionStorage` on significant changes and restores on mount.
- **Accessibility-First Errors:** Error summaries, focus-on-first-error, and ARIA attributes ensure errors are perceivable and operable by all users.
- **Performance Isolation:** Field arrays and large forms use isolated subscriptions (`useFieldArray`, `useFormState`) to prevent cascade re-renders.

### Prerequisites

- Solid understanding of React function components, JSX, and Hooks.
- Working knowledge of React Hook Form (`useForm`, `register`, `handleSubmit`, `formState`).
- Familiarity with controlled vs. uncontrolled inputs and the `useRef` Hook.
- Basic understanding of Zod schema validation.
- Awareness of the Web Storage API (`localStorage`, `sessionStorage`).

### Related Programming Areas

- **Form Architecture:** Controlled vs. uncontrolled inputs, submission lifecycle.
- **State Management:** Form state, validation state, persistence.
- **Accessibility (a11y):** ARIA attributes, focus management, live regions.
- **Performance Optimisation:** Isolated subscriptions, memoisation, stable keys.
- **Server Integration:** Async validation, API error mapping, file uploads.

### Core Concepts / Features

1. Dynamic Field Arrays (`useFieldArray`)
2. Multi-Step / Wizard Forms
3. File Uploads (Drag-and-Drop, Progress, Validation)
4. Debounced and Async Field-Level Validation
5. Form State Persistence and Draft Recovery
6. Error UX (Error Summaries, Focus-on-Error, Field-Level Messaging)

---

## Core Concept 1: Dynamic Field Arrays (`useFieldArray`)

### Definitions

**Core Definition:** `useFieldArray` is a React Hook Form hook that manages dynamic lists of form fields, providing operations to add, remove, insert, swap, and move items while maintaining stable identity and form state synchronisation.

**Technical Definition:** The `useFieldArray` hook provides functionality for managing dynamic lists of form fields in React Hook Form. It enables adding, removing, reordering, and updating array-based form data while maintaining proper validation, form state synchronization, and field registration. The hook returns a `fields` array whose entries contain a stable auto-generated `id` (separate from the array index) plus `defaultValue` metadata, and mutation methods: `append`, `prepend`, `insert`, `remove`, `swap`, `move`, `update`, and `replace`. The `id` is critical: using the array index as a React key causes React to lose track of component identity when items are reordered, removed, or inserted, leading to input state corruption and focus loss. The hook integrates with React Hook Form's core control system via `control._subjects.array` to stay in sync with form-wide resets or value changes.

**Beginner-Friendly Explanation:** `useFieldArray` is like a dynamic list on a form where users can add rows ("Add another phone number"), remove rows, or reorder them. The tricky part is that React needs to know which row is which when the order changes. `useFieldArray` gives each row a unique ID that stays with the row even when it moves, so React doesn't get confused and your inputs don't lose their values.

### Purposes

- To manage dynamic lists of repeating fields (phone numbers, line items, beneficiaries).
- To add items at the end (`append`), at the beginning (`prepend`), or at a specific index (`insert`).
- To remove single, multiple, or all items (`remove`).
- To reorder items by swapping two positions (`swap`) or moving an item to a new position (`move`).
- To replace a single item (`update`) or the entire array (`replace`).
- To maintain stable React keys for correct reconciliation during reordering.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { useForm, useFieldArray } from 'react-hook-form';

function Form() {
  const { control, register, handleSubmit } = useForm({
    defaultValues: { phoneNumbers: [{ value: '' }] },
  });

  const { fields, append, remove, swap, move, insert, prepend, update, replace } =
    useFieldArray({
      control,           // From useForm (optional if using FormProvider)
      name: 'phoneNumbers', // Unique name for the field array
    });

  return (
    <form onSubmit={handleSubmit(data => console.log(data))}>
      {fields.map((field, index) => (
        <div key={field.id}> {/* ✅ Use field.id, NOT index */}
          <input {...register(`phoneNumbers.${index}.value`)} />
          <button type="button" onClick={() => remove(index)}>Remove</button>
          <button type="button" onClick={() => swap(index, index + 1)}>↓</button>
          <button type="button" onClick={() => move(index, 0)}>Top</button>
        </div>
      ))}
      <button type="button" onClick={() => append({ value: '' })}>Add</button>
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Component Breakdown:**
- `useFieldArray({ control, name })`: Initialises the field array manager.
- `fields`: An array of objects, each containing a stable `id` and `defaultValue`.
- `register(\`phoneNumbers.${index}.value\`)`: Registers each field with the array path syntax.
- `key={field.id}`: Uses the stable ID as the React key.
- `append({ value: '' })`: Adds a new item to the end (data is required, cannot be partial).
- `remove(index)`: Removes the item at the specified index.
- `swap(indexA, indexB)`: Exchanges two items.
- `move(from, to)`: Moves an item to a new position.

**Nested Field Arrays:**
```jsx
// Invoice with line items, each with sub-items
const { fields: lineItems } = useFieldArray({ control, name: 'lineItems' });

// Inside each line item:
const { fields: subItems } = useFieldArray({
  control,
  name: `lineItems.${index}.subItems`,
});
```

**Validation with Zod:**
```jsx
const schema = z.object({
  emails: z.array(z.object({
    value: z.string().email('Invalid email'),
  })).min(1, 'At least one email required'),
});
```

**Syntax Rules:**
- Always use `field.id` as the React key, never the array index.
- `append` data is required and cannot be partial; provide a complete default object.
- Use `register(\`arrayName.${index}.fieldName\`)` for each field.
- Use `swap` for adjacent reordering; use `move` for arbitrary repositioning.
- Nested field arrays are supported for multi-level dynamic forms.
- The `keyName` prop can be set if your field objects already contain an `id` property (it will be overwritten otherwise).
- Since v7.79.0, a `disabled` prop disables the entire array and makes all mutation methods no-ops.

**Constraints and Limitations:**
- `useFieldArray` issues a stable `id` separate from the index because reordering and removal would otherwise corrupt React's reconciliation.
- `append` cannot accept partial data; every field in the new item must be provided.
- Dynamic array names (computed at runtime) are not supported.
- Using the array index as key causes focus loss and input corruption when items are reordered.
- For very large arrays (600–700 inputs), batch operations like `replace` can be slow without the latest RHF optimisations; a fix reduced `replace` time from 84 seconds to 0.087 seconds for 100 fields.

### Annotated Code Example: Dynamic Phone Number List

```jsx
import { useForm, useFieldArray } from 'react-hook-form';

function PhoneNumbersForm() {
  const { control, register, handleSubmit, formState: { errors } } = useForm({
    defaultValues: {
      phoneNumbers: [{ number: '', type: 'mobile' }],
    },
  });

  const { fields, append, remove, swap, move } = useFieldArray({
    control,
    name: 'phoneNumbers',
  });

  return (
    <form onSubmit={handleSubmit(data => alert(JSON.stringify(data)))}>
      <h2>Phone Numbers</h2>

      {fields.map((field, index) => (
        <div key={field.id} style={{ display: 'flex', gap: 8, marginBottom: 8 }}>
          <input
            placeholder="Number"
            {...register(`phoneNumbers.${index}.number`, {
              required: 'Number is required',
              pattern: { value: /^\+?[1-9]\d{1,14}$/, message: 'Invalid phone number' },
            })}
            aria-invalid={!!errors.phoneNumbers?.[index]?.number}
          />

          <select {...register(`phoneNumbers.${index}.type`)}>
            <option value="mobile">Mobile</option>
            <option value="home">Home</option>
            <option value="work">Work</option>
          </select>

          <button type="button" onClick={() => remove(index)}>✕</button>
          {index > 0 && (
            <button type="button" onClick={() => swap(index, index - 1)}>↑</button>
          )}
          {index < fields.length - 1 && (
            <button type="button" onClick={() => swap(index, index + 1)}>↓</button>
          )}
        </div>
      ))}

      <button type="button" onClick={() => append({ number: '', type: 'mobile' })}>
        Add Phone Number
      </button>
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** A form with a list of phone number rows. Each row has a number input, a type dropdown, a remove button, and up/down arrows for reordering. Clicking "Add Phone Number" appends a new row. Reordering preserves the values in each row (because `field.id` keeps React's reconciliation stable). Removing a row deletes it.

**Why This Output Occurs:** `useFieldArray` assigns each row a stable `id`. When `swap` is called, the array order changes but the `id`s move with the data, so React knows which DOM node corresponds to which item. The `register` calls use the index to map to the correct array position. The `errors.phoneNumbers?.[index]?.number` path reads the validation error for the specific row.

### Real-World Cases

- **E-commerce line items:** An order form where users add multiple products, each with quantity and price.
- **Beneficiary forms:** Insurance or banking forms with multiple beneficiaries.
- **Contact lists:** CRM forms with multiple phone numbers, emails, or addresses.
- **Invoice builders:** Line items with nested sub-items (taxes, discounts).
- **Survey builders:** Dynamic question arrays with add/remove/reorder.

### References

- React Hook Form – useFieldArray: https://react-hook-form.com/docs/usefieldarray
- DeepWiki – useFieldArray Hook: https://deepwiki.com/react-hook-form/react-hook-form/5-usefieldarray-hook
- CoddyKit – Dynamic Fields with useFieldArray: https://www.coddykit.com/courses/react/dynamic-fields-with-usefieldarray-3707453

---

## Core Concept 2: Multi-Step / Wizard Forms

### Definitions

**Core Definition:** Multi-step or wizard forms are forms that guide users through a sequence of logical steps, where each step collects a portion of the data, and all data is preserved across step transitions.

**Technical Definition:** A multi-step form is not a sequence of pages that replace each other; it is one continuous interaction with checkpoints. The critical architectural principle is that **all form state lives in a persistent parent shell**, not in individual step components. Conditional rendering that unmounts a step component destroys its local state, which is why step-local state causes data loss during Back/Next navigation. The correct approach is to treat a wizard as a coordination problem: which UI is currently committed, which work is preparing the next step, what data must persist regardless of which step is visible, and what validation rules must be satisfied before a new step is allowed to commit. React 19 provides `useTransition` for non-blocking step transitions, `useActionState` for submission as a coordinated transaction, and `useOptimistic` for optimistic receipt rendering. Validation gates ensure the "Next" button is only enabled when the current step's fields are valid, and partial validation via `trigger(fieldNames)` validates only the current step's fields.

**Beginner-Friendly Explanation:** A multi-step form is like a guided interview. The interviewer (parent shell) holds all your answers in a folder. Each question (step) is shown one at a time, but the interviewer never throws away your previous answers. When you go back, the folder still has everything. When you move forward, the interviewer checks that you answered the current question before moving on. The key is that the folder is the single source of truth—not the individual questions.

### Purposes

- To break long forms into manageable steps, reducing cognitive load.
- To preserve all data across step transitions (Back/Next).
- To validate only the current step's fields before allowing progression.
- To provide a review step that summarises all prior data.
- To support conditional branching (skip steps based on answers).
- To enable draft persistence across sessions.

### Syntax Rules and Structure

**Parent-Owned State with Step Rendering:**
```jsx
import { useState } from 'react';
import { useForm, FormProvider } from 'react-hook-form';

const steps = [
  { id: 'personal', label: 'Personal Info' },
  { id: 'address', label: 'Address' },
  { id: 'review', label: 'Review' },
];

function Wizard() {
  const [currentStep, setCurrentStep] = useState(0);
  const methods = useForm({
    mode: 'onBlur',
    defaultValues: { firstName: '', lastName: '', street: '', city: '' },
  });

  // Validate only the current step's fields before advancing
  async function handleNext() {
    const fields = steps[currentStep].fields;
    const valid = await methods.trigger(fields);
    if (valid) setCurrentStep(prev => prev + 1);
  }

  function handleBack() {
    setCurrentStep(prev => prev - 1);
  }

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(data => alert(JSON.stringify(data)))}>
        <Stepper steps={steps} currentStep={currentStep} />

        {/* All steps are rendered but only the current one is visible */}
        <div style={{ display: currentStep === 0 ? 'block' : 'none' }}>
          <PersonalStep />
        </div>
        <div style={{ display: currentStep === 1 ? 'block' : 'none' }}>
          <AddressStep />
        </div>
        <div style={{ display: currentStep === 2 ? 'block' : 'none' }}>
          <ReviewStep />
        </div>

        <div>
          {currentStep > 0 && <button type="button" onClick={handleBack}>Back</button>}
          {currentStep < steps.length - 1 && (
            <button type="button" onClick={handleNext}>Next</button>
          )}
          {currentStep === steps.length - 1 && (
            <button type="submit">Submit</button>
          )}
        </div>
      </form>
    </FormProvider>
  );
}
```

**Per-Step Validation Schema (Zod):**
```jsx
const stepSchemas = {
  personal: z.object({
    firstName: z.string().min(1, 'First name required'),
    lastName: z.string().min(1, 'Last name required'),
  }),
  address: z.object({
    street: z.string().min(1, 'Street required'),
    city: z.string().min(1, 'City required'),
  }),
};
```

**React 19 Transaction-Safe Submission:**
```jsx
import { useActionState, useOptimistic, useTransition } from 'react';

function Wizard() {
  const [isPending, startTransition] = useTransition();
  const [state, formAction] = useActionState(submitWizard, null);
  const [optimisticReceipt, addOptimisticReceipt] = useOptimistic(
    state?.receipt,
    (current, newReceipt) => newReceipt
  );

  async function handleSubmit(formData) {
    addOptimisticReceipt({ status: 'pending', message: 'Submitting...' });
    startTransition(async () => {
      await formAction(formData);
    });
  }
}
```

**Syntax Rules:**
- Keep all form state in a parent component using `FormProvider`.
- Render all steps but toggle visibility (or use `display: none`) rather than unmounting them.
- Validate only the current step's fields before advancing (`trigger(fieldNames)`).
- Use `getValues()` to read the complete form state for the review step.
- Use `useTransition` for non-urgent step transitions so the UI remains responsive.
- Use `useActionState` for transaction-safe submission with pending state.
- Persist the draft to `localStorage` or `sessionStorage` on significant changes.

**Constraints and Limitations:**
- Conditional rendering that unmounts step components destroys step-local state; parent-owned state is mandatory.
- `trigger(fieldNames)` returns a promise; await it before advancing.
- React 19's `useTransition` is required for non-blocking step transitions with async work.
- A wizard is experienced as one continuous interaction, not page navigation; treating it as pages causes flicker and data loss.

### Annotated Code Example: Three-Step Wizard with Validation Gates

```jsx
import { useState } from 'react';
import { useForm, FormProvider, useFormContext } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const fullSchema = z.object({
  firstName: z.string().min(1, 'First name required'),
  lastName: z.string().min(1, 'Last name required'),
  street: z.string().min(1, 'Street required'),
  city: z.string().min(1, 'City required'),
});

function PersonalStep() {
  const { register, formState: { errors } } = useFormContext();
  return (
    <fieldset>
      <legend>Personal Info</legend>
      <label>First Name <input {...register('firstName')} /></label>
      {errors.firstName && <span role="alert">{errors.firstName.message}</span>}
      <label>Last Name <input {...register('lastName')} /></label>
      {errors.lastName && <span role="alert">{errors.lastName.message}</span>}
    </fieldset>
  );
}

function AddressStep() {
  const { register, formState: { errors } } = useFormContext();
  return (
    <fieldset>
      <legend>Address</legend>
      <label>Street <input {...register('street')} /></label>
      {errors.street && <span role="alert">{errors.street.message}</span>}
      <label>City <input {...register('city')} /></label>
      {errors.city && <span role="alert">{errors.city.message}</span>}
    </fieldset>
  );
}

function ReviewStep() {
  const { getValues } = useFormContext();
  const data = getValues();
  return (
    <fieldset>
      <legend>Review</legend>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </fieldset>
  );
}

export default function Wizard() {
  const [step, setStep] = useState(0);
  const methods = useForm({
    resolver: zodResolver(fullSchema),
    mode: 'onBlur',
    defaultValues: { firstName: '', lastName: '', street: '', city: '' },
  });

  const stepFields = [
    ['firstName', 'lastName'],
    ['street', 'city'],
    [],
  ];

  async function next() {
    const valid = await methods.trigger(stepFields[step]);
    if (valid) setStep(s => s + 1);
  }

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(d => alert(JSON.stringify(d)))}>
        <p>Step {step + 1} of 3</p>

        <div style={{ display: step === 0 ? 'block' : 'none' }}><PersonalStep /></div>
        <div style={{ display: step === 1 ? 'block' : 'none' }}><AddressStep /></div>
        <div style={{ display: step === 2 ? 'block' : 'none' }}><ReviewStep /></div>

        <div>
          {step > 0 && <button type="button" onClick={() => setStep(s => s - 1)}>Back</button>}
          {step < 2 && <button type="button" onClick={next}>Next</button>}
          {step === 2 && <button type="submit">Submit</button>}
        </div>
      </form>
    </FormProvider>
  );
}
```

**Expected Output:** A three-step wizard with Personal Info, Address, and Review. Clicking "Next" validates only the current step's fields; if invalid, it stays on the step. Going "Back" preserves all entered data. The Review step shows all values via `getValues()`. Submitting shows an alert with all data.

**Why This Output Occurs:** All fields are registered with the same `FormProvider` context, so their values persist in the parent form state regardless of which step is visible. The steps are rendered with `display: none` rather than being unmounted, so their registered inputs remain in the DOM and their values are preserved. `trigger(stepFields[step])` validates only the fields belonging to the current step, enabling the validation gate.

### Real-World Cases

- **Insurance quote forms:** Multi-step applications with personal, vehicle, coverage, and review steps.
- **Onboarding flows:** Account setup with profile, preferences, and confirmation steps.
- **Checkout processes:** Cart, shipping, payment, and review steps.
- **Loan applications:** Employment, income, assets, and review steps.
- **Survey tools:** Multi-section questionnaires with conditional branching.

### References

- Educative – Multi-Step and Wizard Forms: https://www.educative.io/courses/learn-react/lta/multi-step-and-wizard-forms
- Educative – Solution: Multi-Step Order Form: https://www.educative.io/courses/learn-react/solution-multi-step-order-form
- AppSignal – Smooth Async Transitions in React 19: https://blog.appsignal.com/2025/08/27/smooth-async-transitions-in-react-19.html

---

## Core Concept 3: File Uploads (Drag-and-Drop, Progress, Validation)

### Definitions

**Core Definition:** File upload management in React is the practice of allowing users to select or drag files into a form, validating them against constraints (size, type, count), uploading them with progress tracking, and handling errors and retries.

**Technical Definition:** File uploads in React involve three layers. **Selection** uses `<input type="file" />` (always uncontrolled; its value cannot be set programmatically) or a drag-and-drop zone built with `react-dropzone` or native `dragover`/`drop` events. **Validation** checks file size (`maxSize`), MIME type (`accept`), and count (`maxFiles`) before upload. **Upload** uses `fetch` with `FormData` (which supports files natively) or Axios with `onUploadProgress` for real-time progress tracking. Axios provides `onUploadProgress` in both browser and Node environments to capture upload progress. The `react-file-upload-kit` library offers drag-and-drop, multi-file support, progress tracking, validation, retry, pause/resume, chunked upload, and React Hook Form integration out of the box. Key edge cases include cancelled file selections (the `FileList` object is read-only), duplicate detection, and progress persistence across page refreshes.

**Beginner-Friendly Explanation:** A file upload component lets users drag a file onto a drop zone or click to browse. It checks that the file is the right size and type before uploading. During upload, it shows a progress bar so the user knows something is happening. If the upload fails, it offers a retry. For multiple files, it handles them independently so one failure doesn't block the rest.

### Purposes

- To allow users to select one or multiple files via drag-and-drop or file picker.
- To validate file size, type, and count before upload.
- To show real-time upload progress per file.
- To handle upload errors with retry, cancel, and pause/resume.
- To integrate with React Hook Form for form-level validation and submission.
- To support chunked uploads for large files.

### Syntax Rules and Structure

**File Selection with Drag-and-Drop (react-dropzone):**
```jsx
import { useDropzone } from 'react-dropzone';

function DropZone({ onFiles }) {
  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop: onFiles,
    accept: { 'image/*': ['.png', '.jpg', '.jpeg'] },
    maxSize: 5 * 1024 * 1024, // 5MB
    maxFiles: 5,
  });

  return (
    <div {...getRootProps()} style={{
      border: '2px dashed #ccc',
      padding: 20,
      background: isDragActive ? '#f0f0f0' : 'white',
    }}>
      <input {...getInputProps()} />
      {isDragActive ? 'Drop files here' : 'Drag files here or click to browse'}
    </div>
  );
}
```

**Upload with Progress (Axios):**
```jsx
import axios from 'axios';

async function uploadFile(file, onProgress) {
  const formData = new FormData();
  formData.append('file', file);

  return axios.post('/api/upload', formData, {
    onUploadProgress: (event) => {
      const percent = Math.round((event.loaded * 100) / event.total);
      onProgress(percent);
    },
  });
}
```

**Component Breakdown:**
- `onUploadProgress`: Fires repeatedly during upload, providing `loaded` and `total` bytes.
- `Math.round((event.loaded * 100) / event.total)`: Converts bytes to a percentage.
- `FormData`: Sends the file as `multipart/form-data`.

**Validation (Zod + React Hook Form):**
```jsx
const fileSchema = z.object({
  avatar: z
    .instanceof(File)
    .refine((file) => file.size <= 5 * 1024 * 1024, 'Max 5MB')
    .refine(
      (file) => ['image/jpeg', 'image/png'].includes(file.type),
      'Only JPEG and PNG allowed'
    ),
});
```

**Syntax Rules:**
- Use `react-dropzone` for drag-and-drop; it handles edge cases like duplicate drops and invalid types.
- Always validate file size and type on both client and server.
- Use `FormData` to send files; `fetch` sets the correct `Content-Type` automatically.
- Use Axios's `onUploadProgress` for progress tracking; it works in browser and Node.
- File inputs are always uncontrolled; use `Controller` or a custom handler for React Hook Form integration.
- Show per-file progress, not a single aggregate bar, for multi-file uploads.

**Constraints and Limitations:**
- File inputs cannot be programmatically set; users must select files manually.
- `FileList` objects are read-only; converting to an array is required for manipulation.
- Progress events may not fire for small files (the upload completes too quickly).
- `onUploadProgress` does not work in React Native for Axios.
- Large files may require chunked uploads to avoid timeouts and browser memory limits.

### Annotated Code Example: Multi-File Uploader with Progress and Validation

```jsx
import { useState, useCallback } from 'react';
import { useDropzone } from 'react-dropzone';
import axios from 'axios';

export default function FileUploader() {
  const [files, setFiles] = useState([]);

  const onDrop = useCallback((accepted, rejected) => {
    if (rejected.length > 0) {
      alert(`Rejected: ${rejected.map(r => r.file.name).join(', ')}`);
    }
    setFiles(prev => [
      ...prev,
      ...accepted.map(file => ({
        file,
        progress: 0,
        status: 'pending',
        error: null,
      })),
    ]);
  }, []);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: { 'image/*': ['.png', '.jpg', '.jpeg'] },
    maxSize: 5 * 1024 * 1024,
    maxFiles: 10,
  });

  async function uploadAll() {
    for (const item of files) {
      if (item.status === 'done') continue;
      setFiles(prev => prev.map(f =>
        f.file === item.file ? { ...f, status: 'uploading' } : f
      ));

      try {
        const formData = new FormData();
        formData.append('file', item.file);

        await axios.post('/api/upload', formData, {
          onUploadProgress: (event) => {
            const percent = Math.round((event.loaded * 100) / event.total);
            setFiles(prev => prev.map(f =>
              f.file === item.file ? { ...f, progress: percent } : f
            ));
          },
        });

        setFiles(prev => prev.map(f =>
          f.file === item.file ? { ...f, status: 'done', progress: 100 } : f
        ));
      } catch (err) {
        setFiles(prev => prev.map(f =>
          f.file === item.file
            ? { ...f, status: 'error', error: err.message }
            : f
        ));
      }
    }
  }

  return (
    <div>
      <div {...getRootProps()} style={{
        border: '2px dashed #ccc', padding: 20,
        background: isDragActive ? '#f0f0f0' : 'white',
      }}>
        <input {...getInputProps()} />
        {isDragActive ? 'Drop here' : 'Drag files or click to browse'}
      </div>

      <ul>
        {files.map((item, i) => (
          <li key={i}>
            {item.file.name} — {item.status}
            {item.status === 'uploading' && <progress value={item.progress} max={100} />}
            {item.error && <span style={{ color: 'red' }}>{item.error}</span>}
          </li>
        ))}
      </ul>

      <button onClick={uploadAll} disabled={files.length === 0}>
        Upload All
      </button>
    </div>
  );
}
```

**Expected Output:** A drop zone that accepts PNG and JPEG files up to 5MB, max 10 files. Dropping files adds them to a list with "pending" status. Clicking "Upload All" uploads each file sequentially, showing a progress bar per file. Successful files show "done"; failures show the error message.

**Why This Output Occurs:** `useDropzone` handles drag-and-drop and validates files against `accept`, `maxSize`, and `maxFiles`. Rejected files are shown in an alert. Each accepted file is tracked in state with `progress`, `status`, and `error`. Axios's `onUploadProgress` updates the progress percentage in real time. The sequential upload loop prevents overwhelming the server with concurrent requests.

### Real-World Cases

- **Profile avatars:** Uploading a profile picture with size and type validation.
- **E-commerce product images:** Multi-file upload with image preview and reordering.
- **Document management:** Uploading PDFs and Office documents with progress tracking.
- **Resume submissions:** Uploading PDF resumes with size limits.
- **Chat attachments:** Drag-and-drop file sharing with progress indicators.

### References

- react-file-upload-kit – npm: https://www.npmjs.com/package/react-file-upload-kit
- DEV Community – How to Build a Reliable File Upload with Progress in React and Node.js: https://dev.to/ar_abid_bcfb3a80671e2a134/how-to-build-a-reliable-file-upload-with-progress-in-react-and-nodejs-11hk
- Syncfusion – React File Upload: https://ej2.syncfusion.com/react/documentation/file-upload/getting-started
- Axios – Progress Capturing: https://axios-http.com/docs/req_config

---

## Core Concept 4: Debounced and Async Field-Level Validation

### Definitions

**Core Definition:** Debounced asynchronous field validation is the practice of delaying a server-side validation check (e.g., username availability) until the user pauses typing, and cancelling in-flight requests when the input changes to prevent race conditions.

**Technical Definition:** Async field validation is used when correctness depends on the server: username or email availability, coupon validity, or any uniqueness check. The implementation has three critical components. **Debouncing** delays the request by 300–500ms after the last keystroke, preventing a request per keystroke. **Request cancellation** uses `AbortController` to cancel the previous in-flight request when a new keystroke arrives, so a slow response for an older value can never overwrite a newer one. **Race condition guards** check `err.name === 'AbortError'` to ignore the rejected promise from an aborted request. In React Hook Form, async validation is implemented via a `validate` function that returns `Promise<true | string>`; the resolved string becomes `errors.username.message`, and `true` clears it. A validator factory that owns the debounce timer and `AbortController` ensures the state persists across renders and is cleaned up properly. Keep purely syntactic rules (format, length) synchronous in the schema; only the network round-trip belongs in the async validator.

**Beginner-Friendly Explanation:** Imagine checking if a username is available. You don't want to call the server on every letter the user types ("j", "jo", "joh", "john")—that's 4 requests for one name. Instead, you wait until the user stops typing (debounce). If they start typing again before the server responds, you cancel the previous request (abort) so the old answer doesn't overwrite the new one. The user sees "Checking availability..." and then either "Available" or "Username is taken."

### Purposes

- To check username, email, or domain availability without spamming the server.
- To validate coupon codes, referral codes, or gift card balances.
- To prevent race conditions where a slow older response overwrites a newer result.
- To provide a pending indicator ("Checking...") while the server responds.
- To keep syntactic validation (format, length) synchronous and fast.

### Syntax Rules and Structure

**Validator Factory with Debounce and AbortController:**
```tsx
import { useMemo } from 'react';

function createUsernameValidator(delayMs = 400) {
  let timer: ReturnType<typeof setTimeout> | null = null;
  let controller: AbortController | null = null;

  return (value: string): Promise<true | string> =>
    new Promise((resolve) => {
      if (value.length < 3) return resolve('At least 3 characters');

      if (timer) clearTimeout(timer);
      controller?.abort();
      controller = new AbortController();

      timer = setTimeout(async () => {
        try {
          const res = await fetch(
            `/api/username-available?u=${encodeURIComponent(value)}`,
            { signal: controller!.signal }
          );
          const { available } = await res.json();
          resolve(available ? true : 'That username is taken');
        } catch (err) {
          if ((err as Error).name === 'AbortError') return;
          resolve('Could not check availability — try again');
        }
      }, delayMs);
    });
}
```

**Component Breakdown:**
- `timer`: Holds the debounce timer ID.
- `controller`: Holds the current `AbortController`.
- `if (timer) clearTimeout(timer)`: Cancels the pending debounce from the previous keystroke.
- `controller?.abort()`: Cancels the in-flight request from the previous keystroke.
- `if (err.name === 'AbortError') return`: Ignores the rejected promise from an aborted request.
- `resolve(true)`: Clears the error; `resolve(string)`: Sets the error message.

**Integration with React Hook Form:**
```tsx
const validateUsername = useMemo(() => createUsernameValidator(), []);
const { register, formState: { errors, isValidating, isSubmitting } } = useForm({
  mode: 'onChange',
});

<input
  {...register('username', { validate: validateUsername })}
  aria-invalid={!!errors.username}
  aria-describedby={errors.username ? 'username-error' : undefined}
/>
<p id="username-error" role="alert" aria-live="polite">
  {isValidating ? 'Checking availability…' : errors.username?.message}
</p>
```

**Syntax Rules:**
- Debounce async validation by 300–500ms to avoid a request per keystroke.
- Use `AbortController` to cancel the previous in-flight request.
- Check `err.name === 'AbortError'` and return early; do not resolve an error for aborted requests.
- Use `useMemo` to create the validator once per component instance.
- Show `isValidating` as a pending indicator with `aria-live="polite"`.
- Keep syntactic rules (minLength, pattern) synchronous; only the network call is async.
- Disable the submit button while `isValidating` or `isSubmitting` is true.

**Constraints and Limitations:**
- Async validation adds latency; the user must wait for the server response.
- Without debouncing, the server receives a request per keystroke.
- Without cancellation, stale responses can overwrite newer results.
- The validator factory's timer and controller are per-component-instance; multiple instances have independent state.
- React Hook Form's `isValidating` is `true` while any async validation is running.

### Annotated Code Example: Username Availability with Full Race Condition Guard

```tsx
import { useMemo } from 'react';
import { useForm } from 'react-hook-form';

function createUsernameValidator(delayMs = 400) {
  let timer: ReturnType<typeof setTimeout> | null = null;
  let controller: AbortController | null = null;

  return (value: string): Promise<true | string> =>
    new Promise((resolve) => {
      if (value.length < 3) return resolve('At least 3 characters');

      if (timer) clearTimeout(timer);
      controller?.abort();
      controller = new AbortController();

      timer = setTimeout(async () => {
        try {
          const res = await fetch(
            `/api/username-available?u=${encodeURIComponent(value)}`,
            { signal: controller!.signal }
          );
          const { available } = (await res.json()) as { available: boolean };
          resolve(available ? true : 'That username is taken');
        } catch (err) {
          if ((err as Error).name === 'AbortError') return;
          resolve('Could not check availability — try again');
        }
      }, delayMs);
    });
}

export default function UsernameForm() {
  const validateUsername = useMemo(() => createUsernameValidator(), []);
  const {
    register,
    handleSubmit,
    formState: { errors, isValidating, isSubmitting },
  } = useForm({ mode: 'onChange', defaultValues: { username: '' } });

  const err = errors.username;

  return (
    <form noValidate onSubmit={handleSubmit(async (v) => { alert(`Welcome, ${v.username}!`); })}>
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

**Expected Output:** As the user types a username, after a 400ms pause, "Checking availability…" appears, then either the error "That username is taken" or no error. If the user types again during the check, the previous request is aborted and the new check starts. The submit button is disabled while checking or submitting.

**Why This Output Occurs:** The `createUsernameValidator` factory owns the debounce timer and `AbortController`, both of which persist across renders via `useMemo`. When the user types, `clearTimeout` cancels the pending debounce, and `controller.abort()` cancels the in-flight request. The `AbortError` is caught and ignored, allowing the newer request to own the result. `isValidating` drives the pending indicator, and `aria-live="polite"` ensures screen readers announce it without interrupting.

### Real-World Cases

- **Signup forms:** Username and email availability checks.
- **E-commerce:** Coupon code and gift card validation.
- **Domain registration:** Domain name availability checks.
- **Banking:** Account number and routing number validation.
- **Multi-tenant apps:** Subdomain availability checks.

### References

- JS Form Validation – React Hook Form Async Field Validation: https://js-form-validation.com/framework-integration-patterns/react-hook-form-validation/react-hook-form-async-field-validation/
- KendoReact – Form Validation: https://www.telerik.com/kendo-react-ui/components/form/validation
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController

---

## Core Concept 5: Form State Persistence and Draft Recovery

### Definitions

**Core Definition:** Form state persistence is the practice of writing form data to browser storage (`localStorage` or `sessionStorage`) so that a draft survives page refreshes, tab closures, and browser restarts, and can be restored when the user returns.

**Technical Definition:** Persistence involves two halves: **writing** state to storage on every significant change, and **reading** from storage on mount to restore a previous session. The implementation serialises the values object, the current step indicator, and a timestamp to JSON, then writes to storage with a consistent key. The timestamp is critical—it enables expiry on the read side. Reading on mount involves checking whether a saved entry exists, whether the timestamp is within the expiry window, and if so, initialising the form state from the saved values rather than the empty defaults. In React, the natural pattern is a `useState` initialiser function that checks for saved state synchronously before the component renders, avoiding a flash of empty fields followed by a re-render with restored values. With React Hook Form, use `watch()` to subscribe to all field values and trigger a persistence effect, and use `reset()` on mount to initialise the form with restored values—calling `reset()` after the form initialises, not before, ensures the refs are populated correctly.

**Beginner-Friendly Explanation:** Imagine you're filling out a long insurance application and you accidentally close the tab. Without persistence, all your answers are gone. With persistence, the form automatically saves your progress to your browser's storage. When you come back, it offers to restore your draft. You can pick up exactly where you left off. The difference between `localStorage` and `sessionStorage` is how long the draft lives: `localStorage` survives browser restarts; `sessionStorage` is cleared when the tab closes.

### Purposes

- To prevent data loss when users accidentally refresh, close, or navigate away from a long form.
- To support multi-session workflows where users return days later.
- To provide a "Resume Draft" banner when a saved draft is detected.
- To clear the draft after successful submission to avoid stale data.
- To expire drafts after a configurable window (e.g., 24 hours).

### Syntax Rules and Structure

**Custom Hook: `useFormDraft`**
```tsx
import { useEffect, useRef } from 'react';
import { useFormContext } from 'react-hook-form';

const STORAGE_KEY = 'form-draft';
const MAX_AGE = 24 * 60 * 60 * 1000; // 24 hours

export function useFormDraft() {
  const { watch, reset, formState: { isSubmitSuccessful } } = useFormContext();
  const initialized = useRef(false);

  // Restore on mount
  useEffect(() => {
    if (initialized.current) return;
    initialized.current = true;

    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) return;

    try {
      const { values, timestamp } = JSON.parse(saved);
      if (Date.now() - timestamp > MAX_AGE) {
        localStorage.removeItem(STORAGE_KEY);
        return;
      }
      reset(values);
    } catch {
      localStorage.removeItem(STORAGE_KEY);
    }
  }, [reset]);

  // Persist on change (debounced)
  useEffect(() => {
    const subscription = watch((values) => {
      const timer = setTimeout(() => {
        localStorage.setItem(STORAGE_KEY, JSON.stringify({
          values,
          timestamp: Date.now(),
        }));
      }, 1000);
      return () => clearTimeout(timer);
    });
    return () => subscription.unsubscribe();
  }, [watch]);

  // Clear on successful submit
  useEffect(() => {
    if (isSubmitSuccessful) {
      localStorage.removeItem(STORAGE_KEY);
    }
  }, [isSubmitSuccessful]);
}
```

**Component Breakdown:**
- `initialized`: A ref to ensure restore runs only once.
- `watch(callback)`: Subscribes to all field changes; the callback receives the current values.
- `setTimeout(..., 1000)`: Debounces the write by 1 second to avoid writing on every keystroke.
- `localStorage.setItem(STORAGE_KEY, JSON.stringify({ values, timestamp }))`: Writes the draft with a timestamp.
- `reset(values)`: Restores the form with the saved values.
- `isSubmitSuccessful`: Clears the draft after a successful submission.

**Storage Choice:**

| Storage | Lifetime | Use Case |
|---|---|---|
| `localStorage` | Until cleared or browser data cleared | Multi-session forms (insurance, loan, B2B onboarding) |
| `sessionStorage` | Until tab closes; not shared between tabs | High-security forms, checkout with expiring cart |

**Syntax Rules:**
- Store a timestamp with the draft to enable expiry.
- Debounce the write (1 second) to avoid excessive storage operations.
- Use a `useState` initialiser or a `useEffect` with an `initialized` ref to restore once on mount.
- Call `reset(values)` after the form initialises, not before.
- Clear the draft on successful submission.
- Never store sensitive data (passwords, tokens, personal information) in `localStorage`.
- Use `sessionStorage` for short-lived, tab-specific drafts.

**Constraints and Limitations:**
- `localStorage` is synchronous and blocks the main thread; keep drafts small.
- `localStorage` has a ~5–10 MB limit; do not store files or large datasets.
- Drafts can become stale; always include a timestamp and expiry window.
- Multiple tabs can overwrite each other's drafts; use a unique key per form instance if needed.
- File inputs cannot be persisted (the `FileList` object is not serialisable).

### Annotated Code Example: Multi-Step Wizard with Draft Recovery

```tsx
import { useState, useEffect, useRef } from 'react';
import { useForm, FormProvider, useFormContext } from 'react-hook-form';

const STORAGE_KEY = 'wizard-draft';
const MAX_AGE = 24 * 60 * 60 * 1000;

function useWizardDraft() {
  const { watch, reset, formState: { isSubmitSuccessful } } = useFormContext();
  const initialized = useRef(false);
  const [hasDraft, setHasDraft] = useState(false);

  useEffect(() => {
    if (initialized.current) return;
    initialized.current = true;

    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) return;

    try {
      const { values, timestamp } = JSON.parse(saved);
      if (Date.now() - timestamp > MAX_AGE) {
        localStorage.removeItem(STORAGE_KEY);
        return;
      }
      reset(values);
      setHasDraft(true);
    } catch {
      localStorage.removeItem(STORAGE_KEY);
    }
  }, [reset]);

  useEffect(() => {
    const subscription = watch((values) => {
      const timer = setTimeout(() => {
        localStorage.setItem(STORAGE_KEY, JSON.stringify({
          values,
          timestamp: Date.now(),
        }));
      }, 1000);
      return () => clearTimeout(timer);
    });
    return () => subscription.unsubscribe();
  }, [watch]);

  useEffect(() => {
    if (isSubmitSuccessful) {
      localStorage.removeItem(STORAGE_KEY);
      setHasDraft(false);
    }
  }, [isSubmitSuccessful]);

  function discardDraft() {
    localStorage.removeItem(STORAGE_KEY);
    reset();
    setHasDraft(false);
  }

  return { hasDraft, discardDraft };
}

function WizardContent() {
  const { register, handleSubmit } = useFormContext();
  const { hasDraft, discardDraft } = useWizardDraft();

  return (
    <form onSubmit={handleSubmit(d => alert(JSON.stringify(d)))}>
      {hasDraft && (
        <div role="status" style={{ background: '#fffbe6', padding: 12, marginBottom: 16 }}>
          <p>A draft was found. Your progress has been restored.</p>
          <button type="button" onClick={discardDraft}>Discard draft</button>
        </div>
      )}
      <input {...register('firstName')} placeholder="First Name" />
      <input {...register('lastName')} placeholder="Last Name" />
      <button type="submit">Submit</button>
    </form>
  );
}

export default function Wizard() {
  const methods = useForm({ defaultValues: { firstName: '', lastName: '' } });
  return (
    <FormProvider {...methods}>
      <WizardContent />
    </FormProvider>
  );
}
```

**Expected Output:** When the user types into the form and refreshes the page, the draft is restored and a "Draft found" banner appears. Clicking "Discard draft" clears the saved data and resets the form. After a successful submission, the draft is cleared automatically.

**Why This Output Occurs:** The `useWizardDraft` hook restores the draft on mount via `reset(values)`, debounces writes via `watch`, and clears the draft on successful submission. The `hasDraft` flag controls the banner. The `initialized` ref ensures the restore runs only once, preventing an infinite loop. The timestamp and `MAX_AGE` prevent stale drafts from being restored.

### Real-World Cases

- **Insurance applications:** Long forms that users complete over multiple sessions.
- **Loan applications:** Multi-step forms with financial data.
- **B2B onboarding:** Company setup forms with many fields.
- **Survey tools:** Long questionnaires that users pause and resume.
- **E-commerce checkout:** Cart and shipping details that survive a refresh.

### References

- DEV Community – How to Persist Multi-Step Form State With localStorage and sessionStorage: https://dev.to/137foundry/how-to-persist-multi-step-form-state-with-localstorage-and-sessionstorage-224m
- use-form-draft – npm: https://www.npmjs.com/package/use-form-draft
- react-usedrafty – Snyk: https://security.snyk.io/package/npm/react-usedrafty

---

## Core Concept 6: Error UX (Error Summaries, Focus-on-Error, Field-Level Messaging)

### Definitions

**Core Definition:** Error UX in forms is the practice of presenting validation errors in a way that is perceivable, understandable, and actionable for all users—including screen-reader and keyboard users—through error summaries, focus management, and accessible field-level messaging.

**Technical Definition:** Accessible error UX requires four elements. First, every error message must be programmatically associated with its input via `aria-describedby`. Second, invalid inputs must be marked with `aria-invalid="true"` (or `"false"` when valid). Third, an **error summary** at the top of the form provides a single place to see all errors, with each error linking to its field; the summary receives focus after submission, ensuring screen readers announce it. Fourth, **focus management** moves keyboard focus to the first invalid field (or the error summary) after submission failure, so keyboard users are not stranded. The `useForm` hook's `setFocus` method can focus a specific field programmatically, but there is a known race condition when `setFocus` is called immediately after `setError`; the fix is to call `setFocus` in a `useEffect` or after a microtask. Live regions (`aria-live="assertive"` for the summary, `aria-live="polite"` for pending indicators) announce dynamic changes without interrupting the user.

**Beginner-Friendly Explanation:** Imagine a blind user filling out a form. When they submit and there are errors, they need to hear: "There are 3 errors in this form." Then they need to navigate to each error and hear which field it belongs to. `aria-describedby` makes the screen reader read the error when the field is focused. `aria-invalid` says "this field has a problem." The error summary gives an overview. And moving focus to the first error means the user doesn't have to hunt through the form to find what went wrong.

### Purposes

- To ensure screen-reader users can perceive and understand validation errors.
- To provide a summary of all errors at the top of the form for quick scanning.
- To move keyboard focus to the first error so users can fix it immediately.
- To mark invalid fields with `aria-invalid` for assistive technology.
- To associate error messages with inputs via `aria-describedby`.
- To announce dynamic errors via live regions without interrupting.

### Syntax Rules and Structure

**Accessible Field Pattern:**
```jsx
<div>
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    aria-invalid={errors.email ? 'true' : 'false'}
    aria-describedby={errors.email ? 'email-error' : 'email-hint'}
    aria-required="true"
    {...register('email')}
  />
  <span id="email-hint">We'll never share your email</span>
  {errors.email && (
    <span id="email-error" role="alert" style={{ color: 'red' }}>
      {errors.email.message}
    </span>
  )}
</div>
```

**Component Breakdown:**
- `aria-invalid="true"`: Announces the field as invalid.
- `aria-describedby="email-error"`: Associates the input with its error message.
- `aria-required="true"`: Indicates the field is required.
- `role="alert"`: Announces the error message when it appears.

**Error Summary Component:**
```jsx
function ErrorSummary({ errors, focusOnMount = true }) {
  const summaryRef = useRef(null);
  const errorEntries = Object.entries(errors).filter(([, v]) => v);

  useEffect(() => {
    if (focusOnMount && errorEntries.length > 0 && summaryRef.current) {
      summaryRef.current.focus();
    }
  }, [errorEntries.length, focusOnMount]);

  if (errorEntries.length === 0) return null;

  return (
    <div
      ref={summaryRef}
      role="alert"
      aria-labelledby="error-summary-title"
      tabIndex={-1}
      style={{ border: '2px solid red', padding: 16, marginBottom: 16 }}
    >
      <h2 id="error-summary-title">
        There {errorEntries.length === 1 ? 'is 1 error' : `are ${errorEntries.length} errors`} in your submission
      </h2>
      <ul>
        {errorEntries.map(([field, message]) => (
          <li key={field}>
            <a
              href={`#${field}`}
              onClick={(e) => {
                e.preventDefault();
                document.getElementById(field)?.focus();
              }}
            >
              {message}
            </a>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Component Breakdown:**
- `tabIndex={-1}`: Makes the summary programmatically focusable without adding it to the tab order.
- `role="alert"`: Announces the summary to screen readers.
- `useEffect`: Focuses the summary when errors appear.
- Each error is a link that focuses its field when clicked.

**Focus Management Hook:**
```jsx
function useFormFocus() {
  const formRef = useRef(null);

  const focusFirstError = useCallback(() => {
    if (!formRef.current) return;
    const firstErrorField = formRef.current.querySelector('[aria-invalid="true"]');
    if (firstErrorField) {
      firstErrorField.focus();
      firstErrorField.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }
  }, []);

  const focusErrorSummary = useCallback(() => {
    formRef.current?.querySelector('[role="alert"]')?.focus();
  }, []);

  return { formRef, focusFirstError, focusErrorSummary };
}
```

**Component Breakdown:**
- `querySelector('[aria-invalid="true"]')`: Finds the first invalid field.
- `focus()`: Moves keyboard focus to it.
- `scrollIntoView()`: Scrolls the field into view.

**Syntax Rules:**
- Use `aria-invalid` on every input, with `"true"` when invalid and `"false"` when valid.
- Use `aria-describedby` to link inputs to their error messages.
- Use `role="alert"` on error messages for immediate announcement.
- Provide an error summary at the top with links to each field.
- Focus the error summary after submission failure, then focus the first error field.
- Use `tabIndex={-1}` on the summary to make it focusable without adding to tab order.
- Use `aria-live="polite"` for pending indicators ("Checking availability…") to avoid interrupting.
- Never rely on colour alone to indicate errors; include text and icons.

**Constraints and Limitations:**
- `setFocus` immediately after `setError` can fail due to a race condition; use `useEffect` or a microtask delay.
- `role="alert"` is disruptive if overused; reserve it for important errors.
- `aria-live` regions must be present in the DOM before content is inserted.
- The error summary should not be the only way to perceive errors; field-level messages are also required.
- Focus management can be jarring if overused; only move focus when the user submits.

### Annotated Code Example: Complete Accessible Form with Error Summary

```jsx
import { useRef, useEffect } from 'react';
import { useForm } from 'react-hook-form';

function ErrorSummary({ errors }) {
  const ref = useRef(null);
  const entries = Object.entries(errors).filter(([, v]) => v);

  useEffect(() => {
    if (entries.length > 0 && ref.current) ref.current.focus();
  }, [entries.length]);

  if (entries.length === 0) return null;

  return (
    <div ref={ref} role="alert" tabIndex={-1}
         style={{ border: '2px solid red', padding: 16, marginBottom: 16 }}>
      <h2>There {entries.length === 1 ? 'is 1 error' : `are ${entries.length} errors`}</h2>
      <ul>
        {entries.map(([field, err]) => (
          <li key={field}>
            <a href={`#${field}`} onClick={(e) => {
              e.preventDefault();
              document.getElementById(field)?.focus();
            }}>
              {err.message}
            </a>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default function AccessibleForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({ mode: 'onBlur' });

  return (
    <form onSubmit={handleSubmit(d => alert(JSON.stringify(d)))} noValidate>
      <ErrorSummary errors={errors} />

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error' : undefined}
          {...register('email', {
            required: 'Email is required',
            pattern: { value: /@/, message: 'Invalid email' },
          })}
        />
        {errors.email && (
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
          aria-invalid={errors.password ? 'true' : 'false'}
          aria-describedby={errors.password ? 'password-error' : undefined}
          {...register('password', {
            required: 'Password is required',
            minLength: { value: 8, message: 'At least 8 characters' },
          })}
        />
        {errors.password && (
          <span id="password-error" role="alert" style={{ color: 'red' }}>
            {errors.password.message}
          </span>
        )}
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** Submitting with empty fields shows an error summary at the top ("There are 2 errors") that receives focus, plus error messages below each field. Clicking an error in the summary focuses the corresponding field. Screen readers announce the summary and each field error.

**Why This Output Occurs:** The `ErrorSummary` component focuses itself when errors appear (`useEffect`), so screen readers announce the summary. Each error is a link that focuses its field. The inputs have `aria-invalid` and `aria-describedby` linking them to their error messages. The `role="alert"` on error messages announces them when they appear. The `noValidate` attribute disables native browser validation so the custom accessible messages are used.

### Real-World Cases

- **Login forms:** Accessible error summaries and focus management for email and password.
- **Signup forms:** Accessible validation with multiple fields and cross-field rules.
- **Checkout forms:** Accessible payment and address validation.
- **Admin panels:** Accessible CRUD forms with server-side error mapping.
- **Surveys:** Accessible error summaries for long multi-page forms.

### References

- GitHub – React Hook Form + Zod Accessibility Best Practices: https://github.com/secondsky/claude-skills/blob/main/plugins/react-hook-form-zod/skills/react-hook-form-zod/references/accessibility.md
- OneUptime – How to Implement Accessible Forms in React with ARIA Attributes: https://oneuptime.com/blog/post/2026-01-15-accessible-forms-react-aria/view
- OneUptime – How to Implement Focus Management in React Single Page Applications: https://oneuptime.com/blog/post/2026-01-15-focus-management-react-spa/view
- JS Form Validation – Managing Focus After Validation Failure: https://www.js-form-validation.com/advanced-javascript-validation-logic-patterns/managing-focus-after-validation-failure/

---

## Comparison and Decision Guidance

| Feature | Primary API/Tool | Key Pattern | Common Pitfall |
|---|---|---|---|
| **Dynamic Field Arrays** | `useFieldArray` | `field.id` as key; `append`/`remove`/`swap`/`move` | Using index as key; `append` with partial data |
| **Multi-Step Wizard** | `FormProvider` + `useFormContext` | Parent-owned state; `trigger(fieldNames)` per step | Unmounting step components; step-local state |
| **File Uploads** | `react-dropzone` + Axios `onUploadProgress` | Validate size/type; per-file progress | No cancellation; aggregate progress only |
| **Async Validation** | Validator factory with `AbortController` | Debounce 400ms; abort previous request | No debounce; stale response overwrites |
| **Draft Persistence** | `localStorage` / `sessionStorage` | Debounced write; restore on mount; timestamp | Writing on every keystroke; no expiry |
| **Error UX** | `aria-invalid` + `aria-describedby` + error summary | Focus summary, then first error | Missing ARIA; focus not moved; colour-only |

**Decision Guidance:**
- **Start with `useFieldArray`** for any repeating fields; never manage dynamic lists with manual `useState` arrays.
- **Use `FormProvider` and parent-owned state** for all multi-step wizards; never store step data in step components.
- **Use `react-dropzone` and Axios** for file uploads; validate on both client and server.
- **Use a validator factory with `AbortController`** for async validation; never validate on every keystroke without debouncing.
- **Use `localStorage` with a timestamp** for multi-session drafts; use `sessionStorage` for short-lived, tab-specific drafts.
- **Always implement error summaries and focus management**; `aria-invalid` and `aria-describedby` are not optional.
- **Debounce persistence writes** (1 second) to avoid excessive storage operations.
- **Test with screen readers** (NVDA, VoiceOver) and keyboard-only navigation.

---

## References

- React Hook Form – useFieldArray: https://react-hook-form.com/docs/usefieldarray
- React Hook Form – useForm: https://react-hook-form.com/docs/useform
- React Hook Form – useFormState: https://react-hook-form.com/docs/useformstate
- DeepWiki – useFieldArray Hook: https://deepwiki.com/react-hook-form/react-hook-form/5-usefieldarray-hook
- Educative – Multi-Step and Wizard Forms: https://www.educative.io/courses/learn-react/lta/multi-step-and-wizard-forms
- Educative – Solution: Multi-Step Order Form: https://www.educative.io/courses/learn-react/solution-multi-step-order-form
- AppSignal – Smooth Async Transitions in React 19: https://blog.appsignal.com/2025/08/27/smooth-async-transitions-in-react-19.html
- JS Form Validation – React Hook Form Async Field Validation: https://js-form-validation.com/framework-integration-patterns/react-hook-form-validation/react-hook-form-async-field-validation/
- JS Form Validation – Managing Focus After Validation Failure: https://www.js-form-validation.com/advanced-javascript-validation-logic-patterns/managing-focus-after-validation-failure/
- DEV Community – How to Persist Multi-Step Form State With localStorage and sessionStorage: https://dev.to/137foundry/how-to-persist-multi-step-form-state-with-localstorage-and-sessionstorage-224m
- DEV Community – How to Build a Reliable File Upload with Progress in React and Node.js: https://dev.to/ar_abid_bcfb3a80671e2a134/how-to-build-a-reliable-file-upload-with-progress-in-react-and-nodejs-11hk
- react-file-upload-kit – npm: https://www.npmjs.com/package/react-file-upload-kit
- Syncfusion – React File Upload: https://ej2.syncfusion.com/react/documentation/file-upload/getting-started
- OneUptime – How to Implement Accessible Forms in React with ARIA Attributes: https://oneuptime.com/blog/post/2026-01-15-accessible-forms-react-aria/view
- OneUptime – How to Implement Focus Management in React Single Page Applications: https://oneuptime.com/blog/post/2026-01-15-focus-management-react-spa/view
- GitHub – React Hook Form + Zod Accessibility Best Practices: https://github.com/secondsky/claude-skills/blob/main/plugins/react-hook-form-zod/skills/react-hook-form-zod/references/accessibility.md
- use-form-draft – npm: https://www.npmjs.com/package/use-form-draft
- react-usedrafty – Snyk: https://security.snyk.io/package/npm/react-usedrafty
- CoddyKit – Dynamic Fields with useFieldArray: https://www.coddykit.com/courses/react/dynamic-fields-with-usefieldarray-3707453