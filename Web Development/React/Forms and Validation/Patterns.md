# React Form Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React form patterns are the reusable architectural solutions for managing form state, submission, validation, dynamic structure, and accessibility across the component tree.

**Technical Definition:** Form patterns in React address the full lifecycle of user input: **capturing** values (controlled vs. uncontrolled), **validating** them (client, server, schema), **structuring** them (flat, nested, arrays, conditional), **submitting** them (event handlers, Server Actions, `useActionState`), **persisting** them (storage, drafts), and **presenting** errors accessibly. The patterns are shaped by three architectural decisions: (1) **where state lives** — React state, the DOM, a form library's store, or the URL; (2) **when validation runs** — on blur, on submit, or debounced; and (3) **how the form scales** — multi-step wizards, dynamic arrays, and conditional fields each require distinct state and validation strategies. The 2026 consensus is clear: use React Hook Form (uncontrolled, minimal re-renders) for performance-critical forms, TanStack Form for headless type-safe forms, and React 19's `useActionState` with Server Actions for progressive-enhancement-friendly submission. Rolling your own form state past trivial complexity is a maintenance trap .

**Beginner-Friendly Explanation:** Form patterns are the proven ways to build forms that stay manageable as they grow. A simple contact form with two fields is easy. But a checkout form with 20 fields, a multi-step onboarding wizard, dynamic line items, and conditional address fields is a different problem entirely. The patterns in this cheat sheet tell you where to put the state, when to validate, how to break a long form into steps without losing data, how to let users add and remove rows, how to show fields only when they are relevant, and how to make sure screen-reader users can find and fix errors. The key insight is that **form state is its own category** — it is not server state, not UI state, and not URL state — and mixing the tools for these categories is the root cause of most “our forms are a mess” complaints .

### Key Characteristics

- **Form state is a distinct category:** It needs dirty/validation/submission lifecycle management, separate from server cache or global UI state .
- **Uncontrolled is the performance default:** React Hook Form's ref-based approach avoids per-keystroke re-renders; TanStack Form uses granular subscriptions .
- **Validation is layered:** Client-side for UX, server-side for integrity, schema (Zod/Yup) as the shared source of truth .
- **State placement determines scalability:** Local `useState` breaks at multi-step; `FormProvider` + `useFormContext` shares form state across step components .
- **Dynamic structure requires dedicated primitives:** `useFieldArray` manages repeatable rows with stable keys, append/remove/swap/move operations .
- **Conditional and dependent fields are subscription problems:** `watch` or `form.Subscribe` drives rendering without re-rendering the whole form .
- **Accessibility is structural, not cosmetic:** `aria-invalid`, `aria-describedby`, `role="alert"`, and focus management are baseline requirements .

### Prerequisites

- React function components, `useState`, `useReducer`, and event handling.
- Controlled vs. uncontrolled components.
- Basic knowledge of `FormData`, `useRef`, and `useEffect`.
- Familiarity with a schema validator (Zod or Yup).
- Understanding of accessibility (labels, ARIA, focus management).

### Related Programming Areas

- **Form Libraries** — React Hook Form, TanStack Form, Formik.
- **Form Validation** — client, server, schema, async.
- **State Management** — server state vs. form state vs. URL state.
- **React 19 Server Actions** — `useActionState`, `useFormStatus`, progressive enhancement.
- **Accessibility** — WCAG error identification, ARIA live regions.

---

## 1. Form State Management

### Definitions

**Core Definition:** Form state management is the practice of deciding where form values, errors, touched flags, and submission status live, and how they flow between components.

**Technical Definition:** Form state management in React spans a spectrum: **local `useState`** (simple, controlled, re-renders per keystroke), **`useReducer`** (complex transitions), **form libraries** (React Hook Form's ref-based store, TanStack Form's subscription store), and **React 19 `useActionState`** (server-driven state for actions). The critical architectural rule is **match the tool to the state category**: form state is not server state (use TanStack Query), not global UI state (use Zustand), and not URL state (use `nuqs`) . For forms with more than trivial complexity, a form library is strongly recommended over hand-rolled state .

**Beginner-Friendly Explanation:** Form state management is about deciding who keeps track of what the user typed. For a tiny form, React's `useState` is fine. For anything bigger, a form library like React Hook Form keeps the values without re-rendering the whole page on every keystroke, which keeps things fast.

### Purposes

- To choose the right state location for the form's complexity and performance needs.
- To keep values, errors, and submission status synchronised.
- To avoid redundant or duplicated state (e.g., storing `fullName` when it can be derived) .
- To scale from a single input to multi-step wizards and dynamic arrays.
- To integrate with server state and URL state without conflating categories.

### Syntax Rules and Structure

**State Management Selection Guide** :

| State Type | Tool | Rationale |
|------------|------|-----------|
| UI Local | `useState`, `useReducer` | Component-internal |
| Server State | TanStack Query | Caching, refetch, optimistic |
| Global Client | Zustand | Concise, minimal boilerplate |
| URL State | `nuqs` / `useSearchParams` | Filters, pagination |
| Form State | React Hook Form + Zod | Integrated validation |
| Theme/i18n | Context + Provider | Low change frequency |

**Syntax Rules:**

1. Use `useState` only for trivial forms; escalate to a form library for anything with validation, dynamic fields, or multi-step flows .
2. Never store derived values in form state; compute them during render .
3. Use `FormProvider` + `useFormContext` to share form state across step components without prop drilling .
4. Use `useWatch` or `form.Subscribe` to isolate subscriptions and avoid full-form re-renders .
5. Centralise schemas in a shared module (e.g., `lib/schemas`) for client and server .

**Constraints and Limitations:**

- Controlled `useState` forms re-render the entire component on every keystroke, which becomes noticeable at scale .
- Hand-rolled form state past trivial complexity is a maintenance trap .
- Conflating form state with server state (e.g., putting API data in the form) causes cache invalidation and synchronisation bugs .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Trivial Form with `useState`

```jsx
import { useState } from 'react';

function ContactForm() {
  const [name, setName] = useState('');

  return (
    <form>
      <label htmlFor="name">Name</label>
      <input id="name" value={name} onChange={(e) => setName(e.target.value)} />
    </form>
  );
}
```

**Expected Output:** The input is controlled by `name` state; every keystroke re-renders the component.

**Why this output:** `useState` is the simplest state primitive. It is appropriate for a single field with no validation or dynamic structure.

#### Example 2: Form State with React Hook Form

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

function SignupForm() {
  const { register, handleSubmit, formState: { errors } } =
    useForm({ resolver: zodResolver(schema) });

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <input {...register('email')} />
      {errors.email && <span role="alert">{errors.email.message}</span>}
      <input {...register('password')} type="password" />
      {errors.password && <span role="alert">{errors.password.message}</span>}
      <button type="submit">Sign up</button>
    </form>
  );
}
```

**Expected Output:** Typing does not re-render the form. Validation runs on submit, and errors appear under each field.

**Why this output:** React Hook Form stores values in its own store via refs, avoiding per-keystroke re-renders. `zodResolver` runs the Zod schema on submit.

### Real-World Cases

- **Simple contact forms:** `useState` is sufficient.
- **Large registration forms:** React Hook Form + Zod is the default stack.
- **Multi-step wizards:** `FormProvider` + `useFormContext` shares state across steps.
- **Server-driven forms:** React 19 `useActionState` for progressive enhancement.

### References

- sujeet-pro — Client State Management: Form State — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/client-state-management/README.md#1
- modu-ai — State Management Selection Guide — https://github.com/modu-ai/moai-adk/blob/4fa2aa9db2e0add0f1299e35407141b2b41c5ec0/internal/template/templates/.claude/skills/moai-ref-react-patterns/SKILL.md
- React — Managing State — https://uk.react.dev/learn/managing-state

---

## 2. Form Submission

### Definitions

**Core Definition:** Form submission is the process of gathering validated form data and sending it to an application handler or server endpoint.

**Technical Definition:** React form submission has three primary patterns: **`onSubmit` + state** (read values from state or `FormData`, call `e.preventDefault()`, send a fetch request), **React 19 `action`** (a function passed to `<form action={fn}>` that receives `FormData` directly; React manages pending state and resets uncontrolled fields after success), and **`useActionState`** (wraps an async action and returns `[state, formAction, isPending]`, integrating with Server Actions for progressive enhancement) . Submission must handle validation failure, server errors, and pending states without losing the user's input.

**Beginner-Friendly Explanation:** Submission is what happens when the user clicks “Submit.” The modern React way is to give the form an action function that receives all the data at once, handles the server request, and tells you whether it is pending or failed.

### Purposes

- To collect validated data and send it to the server.
- To manage pending, success, and error states.
- To support progressive enhancement (forms work without JavaScript when using Server Actions).
- To reset or preserve form state appropriately after submission.
- To map server errors back to the correct fields.

### Syntax Rules and Structure

**`onSubmit` with `FormData`:**

```tsx
function handleSubmit(e) {
  e.preventDefault();
  const data = new FormData(e.currentTarget);
  fetch('/api/submit', { method: 'POST', body: data });
}
```

**React 19 `useActionState`:**

```tsx
const [state, formAction, isPending] = useActionState(asyncAction, initialState);

<form action={formAction}>
  <input name="email" />
  {state?.error && <p role="alert">{state.error}</p>}
  <button disabled={isPending}>{isPending ? 'Saving…' : 'Save'}</button>
</form>
```

**Syntax Rules:**

1. Call `e.preventDefault()` in `onSubmit` to avoid a full-page reload.
2. Use `new FormData(e.currentTarget)` to read uncontrolled values; named controls are required.
3. In React 19, `<form action={fn}>` receives `FormData` and manages pending state for the form.
4. Use `useActionState` for async actions; it returns the latest state, a form action, and a pending flag .
5. Use `useFormStatus` in a child of the form to read `pending` for submit buttons.

**Constraints and Limitations:**

- A function `action` and `onSubmit` cannot both be used on the same form.
- Server Actions require a server runtime (Next.js, Remix, or a compatible setup).
- Native submission causes a page reload unless prevented or handled by an action.

### Annotated Code Examples

```tsx
import { useActionState } from 'react';

async function subscribeAction(prevState, formData) {
  const email = formData.get('email');
  if (!email) return { error: 'Email is required' };
  await api.subscribe(email);
  return { success: true };
}

function NewsletterForm() {
  const [state, formAction, isPending] = useActionState(subscribeAction, null);

  return (
    <form action={formAction}>
      <input name="email" type="email" />
      {state?.error && <p role="alert">{state.error}</p>}
      <button disabled={isPending}>{isPending ? 'Subscribing…' : 'Subscribe'}</button>
    </form>
  );
}
```

**Expected Output:** Submitting calls `subscribeAction` with `FormData`. `isPending` disables the button during the request. Errors are announced with `role="alert"`.

**Why this output:** `useActionState` wraps the async action, provides the pending flag, and returns the action's state. React manages the submission lifecycle.

### Real-World Cases

- **Newsletter signups:** Simple async actions with pending state.
- **Authentication:** Login and signup with server validation and error mapping.
- **Checkout:** Order submission with server-side payment validation.
- **Settings forms:** Profile updates with optimistic feedback.

### References

- RESEARCH: ADVANCED SHADCN/UI & NEXT.JS 16 PATTERNS — https://github.com
- React — useActionState — https://react.dev/reference/react/useActionState
- React — `<form>` — https://react.dev/reference/react-dom/components/form

---

## 3. Form Reset

### Definitions

**Core Definition:** Form reset is the process of restoring form fields to their initial values and clearing validation and touched state.

**Technical Definition:** React forms have two reset mechanisms: **native reset** (`<button type="reset">` or `form.reset()`, which restores DOM values to their `defaultValue` but does **not** reset React controlled state), and **library reset** (React Hook Form's `reset()` restores the form to its `defaultValues` and clears errors/touched state). For multi-step forms, reset must also restore the current step and any persisted draft. Resetting after submission is a common pattern; resetting on validation failure must be avoided to preserve user input.

**Beginner-Friendly Explanation:** Reset means clearing the form back to the way it started. If you are using React to control the fields, the browser's reset button does not work — you have to call the form library's reset function.

### Purposes

- To clear the form after a successful submission.
- To restore initial values when editing is cancelled.
- To clear validation errors and touched state.
- To provide a “Start over” or “Clear” action.
- To reset multi-step forms to the first step.

### Syntax Rules and Structure

**React Hook Form Reset:**

```tsx
const { reset } = useForm({ defaultValues: { email: '' } });

// After successful submit
reset(); // Restores defaultValues and clears errors/touched

// Reset to new values
reset({ email: 'new@example.com' });
```

**Native Reset (uncontrolled):**

```tsx
<form ref={formRef}>
  <button type="reset">Clear</button>
</form>
// formRef.current.reset() restores defaultValue
```

**Syntax Rules:**

1. Use the form library's `reset()` for controlled forms; native reset does not affect React state.
2. For uncontrolled forms, `form.reset()` restores `defaultValue`.
3. Do not reset on validation failure; preserve user input.
4. Reset multi-step forms by resetting the step state and the form state together.
5. Clear persisted drafts on reset if the draft represents incomplete work.

### Annotated Code Examples

```tsx
function SignupForm() {
  const { register, handleSubmit, reset, formState: { isSubmitSuccessful } } =
    useForm({ defaultValues: { email: '', password: '' } });

  useEffect(() => {
    if (isSubmitSuccessful) reset();
  }, [isSubmitSuccessful, reset]);

  return (
    <form onSubmit={handleSubmit(async (data) => { await api.signup(data); })}>
      <input {...register('email')} />
      <input {...register('password')} type="password" />
      <button type="submit">Sign up</button>
    </form>
  );
}
```

**Expected Output:** After a successful submission, the form fields are cleared and errors/touched state are reset.

**Why this output:** `isSubmitSuccessful` becomes `true` after a successful submit, and the effect calls `reset()` to restore `defaultValues`.

### Real-World Cases

- **Post-submission clearing:** Contact and newsletter forms.
- **Cancel editing:** Profile edit forms that revert to the original values.
- **Multi-step wizards:** Starting a new application from step one.

### References

- React Hook Form — reset — https://react-hook-form.com/docs/useform/reset
- React — `<form>` — https://react.dev/reference/react-dom/components/form

---

## 4. Form Persistence

### Definitions

**Core Definition:** Form persistence is the practice of saving form state to browser storage or a server draft so that users do not lose their input across page reloads, tab switches, or sessions.

**Technical Definition:** Form persistence saves form values (and optionally touched/error state) to `localStorage` (survives sessions), `sessionStorage` (per-tab, cleared when the tab closes), or a server draft. The choice depends on the data's sensitivity and lifetime: `sessionStorage` is ideal for wizard steps and temporary form data; `localStorage` is for longer-lived drafts that should survive tab closure . Libraries like `react-hook-form-storage` provide a `useFormStorage` hook with debounced auto-save, field inclusion/exclusion, and SSR-safe restoration . Sensitive fields (passwords, payment data) should be excluded from persistence.

**Beginner-Friendly Explanation:** Persistence means saving what the user typed so they do not have to start over if they accidentally refresh the page. You save the draft to the browser's storage — just be careful not to save passwords or credit card numbers.

### Purposes

- To prevent data loss from accidental reloads or navigation.
- To support long forms and multi-step wizards that span sessions.
- To restore drafts when the user returns.
- To improve completion rates for complex forms.
- To provide a “resume where you left off” experience.

### Syntax Rules and Structure

**`useFormStorage` (React Hook Form):**

```tsx
import { useFormStorage } from 'react-hook-form-storage';

const { isRestored, isLoading } = useFormStorage('my-form', form, {
  storage: sessionStorage,       // or localStorage
  exclude: ['password', 'cardNumber'], // Never persist sensitive fields
  debounce: 500,                 // Auto-save debounce
});
```

**Manual Persistence:**

```tsx
useEffect(() => {
  const saved = sessionStorage.getItem('draft');
  if (saved) reset(JSON.parse(saved));
}, []);

useEffect(() => {
  const subscription = watch((values) => {
    sessionStorage.setItem('draft', JSON.stringify(values));
  });
  return () => subscription.unsubscribe();
}, [watch]);
```

**Syntax Rules:**

1. Use `sessionStorage` for tab-scoped drafts (wizard steps); `localStorage` for cross-session drafts .
2. Exclude sensitive fields (passwords, payment data) from persistence.
3. Debounce auto-save to avoid writing on every keystroke .
4. Handle SSR safely: do not read `window` storage during server rendering .
5. Clear the draft after successful submission.

**Constraints and Limitations:**

- Storage is synchronous and limited (~5–10MB).
- Persisted data can become stale if the schema changes; version the storage key.
- Persisting sensitive data is a security risk.

### Annotated Code Examples

```tsx
function WizardStep() {
  const form = useForm({ defaultValues: { firstName: '', email: '' } });
  const { isLoading } = useFormStorage('wizard-draft', form, {
    storage: sessionStorage,
    exclude: ['password'],
    debounce: 500,
  });

  if (isLoading) return <Spinner />;
  return <form>{/* fields */}</form>;
}
```

**Expected Output:** The form's values are saved to `sessionStorage` with a 500ms debounce and restored when the component mounts.

**Why this output:** `useFormStorage` watches the form, saves values to the chosen storage, and restores them on mount. `exclude` prevents sensitive fields from being saved.

### Real-World Cases

- **Multi-step onboarding:** Saving progress across steps and sessions.
- **Long application forms:** Loan applications, insurance quotes.
- **Content editors:** Draft posts and comments.
- **Checkout recovery:** Restoring a cart after accidental navigation.

### References

- react-hook-form-storage — npm — https://www.npmjs.com/package/react-hook-form-storage
- @sncrr/react-hook-form-persist — npm — https://www.npmjs.com/package/@sncrr/react-hook-form-persist
- React useSessionStorage Hook — https://github.com

---

## 5. Multi-Step Forms

### Definitions

**Core Definition:** A multi-step form is a single form split across multiple screens, where the user progresses through steps and the data is preserved across steps.

**Technical Definition:** Multi-step forms use a **step state** (`currentStep`) and a **shared form state** that persists across step transitions. The two dominant patterns are: **one form, multiple step components** (using `FormProvider` + `useFormContext` so every step reads the same form instance) and **one form per step with lifted data** (each step is a separate form, and data is merged into a parent state on “Next”). The first pattern is preferred because it preserves validation state and avoids re-mounting the form instance . Validation is typically **per-step** (`trigger(fieldsForStep)`) before advancing, with full-form validation on final submit .

**Beginner-Friendly Explanation:** A multi-step form is a long form broken into pages. You fill in step one, click “Next,” fill in step two, and so on. The data from step one is still there when you go back.

### Purposes

- To break long forms into manageable chunks.
- To validate incrementally, reducing cognitive load.
- To allow users to go back and correct earlier steps.
- To show progress and guide completion.
- To preserve data across steps without re-mounting the form.

### Syntax Rules and Structure

**`FormProvider` + `useFormContext` Pattern** :

```tsx
const methods = useForm({ resolver: zodResolver(wizardSchema), mode: 'onBlur' });

<FormProvider {...methods}>
  <form onSubmit={methods.handleSubmit(onSubmit)}>
    {currentStep === 1 && <Step1 />}
    {currentStep === 2 && <Step2 />}
    <button onClick={handleNext}>Next</button>
  </form>
</FormProvider>
```

**Per-Step Validation:**

```tsx
async function handleNext() {
  const fields = getFieldsForStep(currentStep);
  const isValid = await methods.trigger(fields);
  if (isValid) setCurrentStep((s) => s + 1);
}
```

**Syntax Rules:**

1. Use `FormProvider` to share the form instance across step components .
2. Validate only the current step's fields before advancing .
3. Validate the entire form on final submit.
4. Show a progress indicator (`Step X of Y`) with accessible `aria-current="step"` .
5. Persist step state if the wizard should survive reloads.

### Annotated Code Examples

```tsx
function Wizard() {
  const [step, setStep] = useState(1);
  const methods = useForm({ defaultValues: { name: '', email: '', plan: 'basic' } });

  const next = async () => {
    const fields = step === 1 ? ['name', 'email'] : ['plan'];
    if (await methods.trigger(fields)) setStep((s) => s + 1);
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit((data) => api.submit(data))}>
        {step === 1 && <Step1 />}
        {step === 2 && <Step2 />}
        <button type="button" onClick={next}>Next</button>
      </form>
    </FormProvider>
  );
}
```

**Expected Output:** The user advances only when the current step's fields are valid. The form instance is shared across steps.

**Why this output:** `FormProvider` makes the form instance available to all step components. `trigger(fields)` validates only the current step's fields before advancing.

### Real-World Cases

- **Onboarding flows:** Account, profile, preferences.
- **Checkout:** Shipping, payment, review.
- **Loan applications:** Personal, employment, financial details.
- **Job applications:** Personal info, experience, references.

### References

- React Hook Form — Multi-Step Wizard — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-forms-react-hook-form/examples/wizard.md
- Form UX Patterns — Multi-Step Wizard — https://github.com/bbeierle12/skill-mcp-claude/blob/a35223a33b13a8742e3a8b06ae833fc3e7ed05b3/skills/forms-router/references/ux-patterns.md
- React Hook Form — FormProvider — https://react-hook-form.com/docs/formprovider

---

## 6. Wizard Forms

### Definitions

**Core Definition:** A wizard form is a multi-step form with explicit navigation controls (Back, Next, Submit) and step-aware validation and progress indication.

**Technical Definition:** Wizard forms extend multi-step forms with a **state machine** for step transitions: `currentStep`, `completedSteps`, and optional step-level validation gates. The wizard state model tracks which steps are complete and allows navigation only to completed steps or the next incomplete step . Wizards are common in onboarding, setup, and checkout flows; they often include progress indicators and may persist step state to storage. React 19's `useActionState` can be used for server-driven wizard submissions.

**Beginner-Friendly Explanation:** A wizard form is a multi-step form with a visible path — a progress bar showing where you are, a “Back” button, and a “Next” button that only works when the current step is valid.

### Purposes

- To guide users through a linear sequence of steps.
- To enforce step completion before advancing.
- To provide clear navigation and progress feedback.
- To handle complex, branching flows.
- To persist progress across sessions.

### Syntax Rules and Structure

**Wizard State Model** :

```ts
interface WizardState {
  currentStep: number;
  totalSteps: number;
  data: Record<string, unknown>;
  completedSteps: Set<number>;
}
```

**Navigation Rules:**

1. `next()` marks the current step complete and advances.
2. `prev()` moves back without marking incomplete.
3. `goTo(step)` only allows completed steps or the next incomplete step.

**Syntax Rules:**

1. Track `currentStep`, `completedSteps`, and form data in a single wizard state.
2. Validate the current step before allowing `next()`.
3. Render a progress indicator with `aria-current="step"`.
4. Preserve form data across steps (shared form instance or lifted state).
5. Allow backward navigation without losing data.

### Annotated Code Examples

```tsx
function WizardForm() {
  const [state, setState] = useState({ step: 1, completed: new Set() });

  const next = async () => {
    if (await methods.trigger(fieldsForStep(state.step))) {
      setState((s) => ({
        step: s.step + 1,
        completed: new Set([...s.completed, s.step]),
      }));
    }
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(submitAll)}>
        <nav aria-label="Progress">
          <ol>
            {[1, 2, 3].map((n) => (
              <li key={n} aria-current={state.step === n ? 'step' : undefined}>
                Step {n}
              </li>
            ))}
          </ol>
        </nav>
        {state.step === 1 && <Step1 />}
        {state.step === 2 && <Step2 />}
        {state.step === 3 && <Step3 />}
        <button type="button" onClick={next}>Next</button>
      </form>
    </FormProvider>
  );
}
```

**Expected Output:** The progress indicator highlights the current step. “Next” advances only when the current step is valid.

**Why this output:** The wizard state tracks the step and completed steps. The progress indicator uses `aria-current="step"` for accessibility. Validation gates the transition.

### Real-World Cases

- **Onboarding:** Account setup, profile completion, preferences.
- **Checkout:** Shipping, payment, review and confirm.
- **Setup wizards:** Initial configuration for SaaS products.
- **Data import tools:** Upload, mapping, validation, import.

### References

- Form UX Patterns — Multi-Step Wizard State — https://github.com/bbeierle12/skill-mcp-claude/blob/a35223a33b13a8742e3a8b06ae833fc3e7ed05b3/skills/forms-router/references/ux-patterns.md
- smartstepper — npm — https://www.npmjs.com/package/smartstepper
- React Hook Form — Multi-Step Wizard — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-forms-react-hook-form/examples/wizard.md

---

## 7. Dynamic Forms

### Definitions

**Core Definition:** A dynamic form is a form whose structure — fields, field groups, or entire sections — can change at runtime based on user actions or data.

**Technical Definition:** Dynamic forms add, remove, reorder, or replace fields in response to user input. They require a form library or state structure that can manage **dynamic field registration** and **stable keys** for reconciliation. React Hook Form provides `useFieldArray` for repeatable field groups ; TanStack Form provides `FieldArray` with similar capabilities. Dynamic forms must handle validation for fields that may not exist at validation time, and must unregister removed fields to avoid stale values in the submitted data.

**Beginner-Friendly Explanation:** A dynamic form lets the user add or remove parts of the form — like adding another phone number, another line item, or another passenger.

### Purposes

- To let users add or remove repeatable sections.
- To support forms whose shape depends on data (e.g., invoice line items, survey questions).
- To avoid rendering fields that are not relevant.
- To handle variable-length arrays of values.

### Syntax Rules and Structure

**`useFieldArray` (React Hook Form)** :

```tsx
const { fields, append, remove, swap, move, insert } = useFieldArray({
  control,
  name: 'lineItems',
});
```

**Rules:**

1. Use `field.id` (not `index`) as the React key for each row .
2. Use `append` to add, `remove` to delete, `swap`/`move` to reorder .
3. Fields removed with `remove` are unregistered; their values are excluded from submission.
4. Validate arrays with `rules` (e.g., `minLength`) on the field array itself .

**Constraints and Limitations:**

- Dynamic field names are not supported; the array name must be static.
- Primitive values in arrays may have limitations in some versions; keep rows as objects .
- Removed fields must be unregistered to avoid stale data.

### Annotated Code Examples

```tsx
function InvoiceForm() {
  const { control, register, handleSubmit } = useForm({
    defaultValues: { lineItems: [{ description: '', quantity: 1 }] },
  });
  const { fields, append, remove } = useFieldArray({ control, name: 'lineItems' });

  return (
    <form onSubmit={handleSubmit((data) => api.createInvoice(data))}>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`lineItems.${index}.description`)} />
          <input type="number" {...register(`lineItems.${index}.quantity`, { valueAsNumber: true })} />
          <button type="button" onClick={() => remove(index)}>Remove</button>
        </div>
      ))}
      <button type="button" onClick={() => append({ description: '', quantity: 1 })}>
        Add line item
      </button>
      <button type="submit">Create invoice</button>
    </form>
  );
}
```

**Expected Output:** Users can add and remove line items. Each row uses a stable `field.id` key. Removed rows are excluded from the submitted data.

**Why this output:** `useFieldArray` manages the array state and stable keys. `append` and `remove` mutate the array without breaking reconciliation.

### Real-World Cases

- **Invoice line items:** Products, quantities, prices.
- **Survey questions:** Repeatable question groups.
- **Passenger details:** Multiple travellers on a booking.
- **Emergency contacts:** Repeatable contact rows.

### References

- React Hook Form — useFieldArray — https://react-hook-form.com/docs/usefieldarray
- React Hook Form — Repeatable Rows — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-forms-react-hook-form/examples/arrays.md
- React Hook Form — FieldArray Component — https://react-hook-form.com/docs/usefieldarray/fieldarray

---

## 8. Nested Forms

### Definitions

**Core Definition:** A nested form is a form structure where field values are organised into nested objects (e.g., `address.city`) rather than a flat list.

**Technical Definition:** Nested forms use **path-based field names** (`register('address.city')`, `register('user.profile.name')`) to bind deeply nested values. The form library traverses the object path to read and write the value. Validation schemas must mirror the nested structure (`z.object({ address: z.object({ city: z.string() }) })`). Nested forms are common for structured data like addresses, user profiles, and configuration objects.

**Beginner-Friendly Explanation:** A nested form has groups inside groups — like an address with a street, city, and postal code all grouped under “address.”

### Purposes

- To model structured data naturally (addresses, profiles, settings).
- To keep related fields grouped in state and validation.
- To support complex schemas with nested objects.
- To enable conditional logic on nested groups.

### Syntax Rules and Structure

**Path-Based Registration:**

```tsx
<input {...register('address.street')} />
<input {...register('address.city')} />
<input {...register('address.postalCode')} />
```

**Nested Schema:**

```ts
const schema = z.object({
  address: z.object({
    street: z.string().min(1),
    city: z.string().min(1),
    postalCode: z.string().regex(/^\d{5}$/),
  }),
});
```

**Syntax Rules:**

1. Use dot notation for nested paths: `register('user.profile.name')`.
2. Mirror the nested structure in the validation schema.
3. Use `FormProvider` or `useFormContext` to avoid passing `control` through many layers.
4. For nested arrays, cast the field array name to the correct path type .

### Annotated Code Examples

```tsx
function AddressForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input {...register('address.street')} />
      {errors.address?.street && <span role="alert">{errors.address.street.message}</span>}
      <input {...register('address.city')} />
      {errors.address?.city && <span role="alert">{errors.address.city.message}</span>}
      <button type="submit">Save</button>
    </form>
  );
}
```

**Expected Output:** The form reads and validates nested `address` values. Errors appear under the correct nested field.

**Why this output:** `register('address.street')` binds the input to the nested path. The Zod schema validates the nested object, and `errors.address?.street` accesses the nested error.

### Real-World Cases

- **Shipping addresses:** Street, city, postal code, country.
- **User profiles:** Name, contact, preferences.
- **Company settings:** Billing, technical, notification settings.
- **Event registration:** Attendee info, ticket type, add-ons.

### References

- React Hook Form — register — https://react-hook-form.com/docs/useform/register
- React Hook Form + Zod Skill — https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/fellipeutaka/leon/react-hook-form-zod/SKILL.md

---

## 9. Array Fields

### Definitions

**Core Definition:** Array fields are form fields that hold an ordered list of values or objects, managed by a dedicated array primitive.

**Technical Definition:** Array fields are the data structure behind dynamic forms. The form library must provide operations to **append**, **prepend**, **insert**, **remove**, **swap**, **move**, **update**, and **replace** items while maintaining stable React keys and correct validation state . React Hook Form's `useFieldArray` and TanStack Form's `FieldArray` are the primary implementations. Each row must use the library-generated `id` as its React key, not the array index .

**Beginner-Friendly Explanation:** Array fields are lists inside your form — a list of phone numbers, a list of products, a list of guests.

### Purposes

- To manage repeatable field groups with stable identity.
- To support add/remove/reorder operations without breaking React reconciliation.
- To validate array-level rules (minimum items, maximum items).
- To map array data to and from the server.

### Syntax Rules and Structure

**`useFieldArray` Operations** :

| Method | Purpose |
|--------|---------|
| `append(obj)` | Add to the end |
| `prepend(obj)` | Add to the start |
| `insert(index, obj)` | Insert at a position |
| `remove(index)` | Remove at a position |
| `swap(a, b)` | Swap two rows |
| `move(from, to)` | Move a row |
| `update(index, obj)` | Replace a row |
| `replace(array)` | Replace the whole array |

**Syntax Rules:**

1. Always use `field.id` as the React key .
2. Use `append`/`remove` for add/remove; `swap`/`move` for reordering.
3. Validate array length with `rules: { minLength: { value: 1, message: '...' } }` .
4. Removed rows are unregistered and excluded from submission.

### Annotated Code Examples

```tsx
const { fields, append, remove, swap } = useFieldArray({
  control,
  name: 'contacts',
  rules: { minLength: { value: 1, message: 'At least one contact is required' } },
});

{fields.map((field, index) => (
  <div key={field.id}>
    <input {...register(`contacts.${index}.name`)} />
    <button type="button" onClick={() => swap(index, index - 1)} disabled={index === 0}>
      Move up
    </button>
    <button type="button" onClick={() => remove(index)}>Remove</button>
  </div>
))}
```

**Expected Output:** Users can add, remove, and reorder contacts. The array-level `minLength` rule prevents removing all contacts.

**Why this output:** `useFieldArray` manages the array state and stable keys. `swap` reorders rows, and `remove` unregisters the removed row.

### Real-World Cases

- **Contact lists:** Multiple phone numbers or emails.
- **Order line items:** Products and quantities.
- **Attendee lists:** Multiple guests.
- **Education history:** Multiple degrees or certifications.

### References

- React Hook Form — useFieldArray — https://react-hook-form.com/docs/usefieldarray
- React Hook Form — FieldArray Component — https://react-hook-form.com/docs/usefieldarray/fieldarray
- React Hook Form — Repeatable Rows — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-forms-react-hook-form/examples/arrays.md

---

## 10. Conditional Fields

### Definitions

**Core Definition:** Conditional fields are fields that are rendered, required, or enabled only when a specific condition is met.

**Technical Definition:** Conditional fields use a **watched value** (the controlling field) to decide whether to render a dependent field. The pattern requires three things: (1) a subscription to the controlling value (`watch` or `form.Subscribe`), (2) conditional rendering of the dependent field, and (3) conditional validation or `shouldUnregister` to ensure hidden fields do not submit stale values . Zod's `discriminatedUnion` is the idiomatic way to model conditional field sets with type-safe validation .

**Beginner-Friendly Explanation:** A conditional field shows up only when it makes sense. If you choose “Credit Card” as the payment method, the card number field appears. If you choose “PayPal,” it disappears.

### Purposes

- To show only relevant fields, reducing cognitive load.
- To enforce conditional requiredness (e.g., “State is required if Country is US”).
- To avoid submitting stale values from hidden fields.
- To model branching form flows.

### Syntax Rules and Structure

**Conditional Rendering:**

```tsx
const paymentMethod = watch('paymentMethod');

{paymentMethod === 'card' && <CardFields />}
{paymentMethod === 'bank' && <BankFields />}
```

**Conditional Schema (Zod `discriminatedUnion`)** :

```ts
const schema = z.discriminatedUnion('paymentMethod', [
  z.object({ paymentMethod: z.literal('card'), cardNumber: z.string() }),
  z.object({ paymentMethod: z.literal('bank'), bankAccount: z.string() }),
]);
```

**Syntax Rules:**

1. Subscribe to the controlling value with `watch` or `form.Subscribe` .
2. Render the dependent field conditionally.
3. Use `shouldUnregister: true` to remove hidden fields from submission, or use a discriminated union schema.
4. Validate conditionally required fields in the schema, not just in the UI.

### Annotated Code Examples

```tsx
function PaymentForm() {
  const { register, watch, handleSubmit } = useForm();
  const method = watch('paymentMethod');

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <select {...register('paymentMethod')}>
        <option value="card">Credit Card</option>
        <option value="paypal">PayPal</option>
      </select>

      {method === 'card' && (
        <>
          <input {...register('cardNumber', { required: 'Card number is required' })} />
          <input {...register('cardExpiry', { required: 'Expiry is required' })} />
        </>
      )}

      {method === 'paypal' && (
        <input {...register('paypalEmail', { required: 'PayPal email is required' })} />
      )}

      <button type="submit">Pay</button>
    </form>
  );
}
```

**Expected Output:** Only the fields for the selected payment method are rendered and validated.

**Why this output:** `watch('paymentMethod')` subscribes to the controlling field. Conditional JSX renders the correct field set. Validation rules are attached to each conditional field.

### Real-World Cases

- **Payment methods:** Card vs. bank vs. PayPal.
- **Shipping options:** Different address fields for domestic vs. international.
- **Contact preferences:** Email vs. phone.
- **Business type:** Individual vs. company registration fields.

### References

- Conditional Field Rendering in React Hook Form — https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-forms/references/templates/conditional-fields.md
- Form UX Patterns — Conditional Fields — https://github.com/bbeierle12/skill-mcp-claude/blob/a35223a33b13a8742e3a8b06ae833fc3e7ed05b3/skills/forms-router/references/ux-patterns.md

---

## 11. Dependent Fields

### Definitions

**Core Definition:** Dependent fields are fields whose options, values, or validity depend on the value of another field.

**Technical Definition:** Dependent fields create **inter-field dependencies**: cascading selects (country → province), conditional enabling (submit disabled until terms accepted), and cross-field validation (password === confirmPassword). The pattern requires a **subscription** to the controlling field and a **derived update** to the dependent field (e.g., clearing the province when the country changes, or filtering options). Zod's `.refine()` handles cross-field validation; `watch` drives UI dependencies .

**Beginner-Friendly Explanation:** A dependent field changes based on another field. Pick a country, and the province dropdown updates. Type a password, and the “confirm password” field checks that they match.

### Purposes

- To implement cascading selects and filtered options.
- To enforce cross-field validation (password confirmation, date ranges).
- To enable or disable controls based on other values.
- To clear stale values when a parent field changes.

### Syntax Rules and Structure

**Cascading Selects:**

```tsx
const country = watch('country');
const provinces = provincesByCountry[country] ?? [];

{provinces.length > 0 && (
  <select {...register('province')}>
    {provinces.map((p) => <option key={p} value={p}>{p}</option>)}
  </select>
)}
```

**Cross-Field Validation (Zod):**

```ts
z.object({ password: z.string().min(8), confirm: z.string() })
  .refine((d) => d.password === d.confirm, {
    message: "Passwords don't match",
    path: ['confirm'],
  });
```

**Syntax Rules:**

1. Subscribe to the controlling field with `watch` or `form.Subscribe`.
2. Derive dependent options or validity from the controlling value.
3. Clear dependent field values when the controlling value changes to avoid stale submissions.
4. Use `.refine()` with `path` for cross-field errors .

### Annotated Code Examples

```tsx
function PasswordForm() {
  const { register, watch, formState: { errors } } = useForm({
    resolver: zodResolver(passwordSchema),
  });
  const password = watch('password');

  return (
    <form>
      <input {...register('password')} type="password" />
      <input {...register('confirm')} type="password" />
      {errors.confirm && <span role="alert">{errors.confirm.message}</span>}
    </form>
  );
}
```

**Expected Output:** The confirmation field is validated against the password field. The error appears on the confirm field.

**Why this output:** The Zod schema's `.refine()` checks equality and routes the error to `confirm`. The form validates both fields together.

### Real-World Cases

- **Address forms:** Country → province/state.
- **Password flows:** Password → confirmation.
- **Date ranges:** Start date → end date.
- **Product configurators:** Model → options → price.

### References

- TanStack Form Linked Fields Patterns — https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-forms/references/templates/linked-fields-form.md
- Form UX Patterns — Conditional Fields — https://github.com/bbeierle12/skill-mcp-claude/blob/a35223a33b13a8742e3a8b06ae833fc3e7ed05b3/skills/forms-router/references/ux-patterns.md

---

## 12. Form Accessibility

### Definitions

**Core Definition:** Form accessibility is the practice of building forms that can be understood and operated by everyone, including screen-reader users and keyboard-only users.

**Technical Definition:** Accessible forms require four error-related channels: **visual** (the message is rendered near the field), **state** (`aria-invalid="true"` on the input), **association** (`aria-describedby` pointing to the message's `id`), and **announcement** (`role="alert"` or an `aria-live` region) . On submit failure, focus should move to the first invalid field . Every input needs a visible `<label>` with a matching `id` and `htmlFor`; missing label associations are the most common accessibility issue in code review . For grouped controls (radios, checkboxes), `<fieldset>` and `<legend>` provide semantic grouping .

**Beginner-Friendly Explanation:** An accessible form works for everyone. Every field has a label, every error is announced to screen readers, keyboard users can reach everything, and on submit failure the focus jumps to the first problem.

### Purposes

- To ensure every control has a programmatic label.
- To make error messages discoverable and announced.
- To support keyboard-only navigation and operation.
- To communicate required and invalid states programmatically.
- To meet WCAG 2.1 requirements.

### Syntax Rules and Structure

**Required ARIA Wiring** :

| Channel | Implementation |
|---------|----------------|
| Label | `<label htmlFor={id}>` + `id` on input |
| Invalid state | `aria-invalid="true"` |
| Error association | `aria-describedby={errorId}` |
| Announcement | `role="alert"` on the error message |
| Focus | Focus first invalid field on submit |

**Syntax Rules:**

1. Give every input a visible `<label>` with a matching `htmlFor`/`id` .
2. Render errors with a stable `id` and point `aria-describedby` at it when present .
3. Set `aria-invalid` to reflect the real error state .
4. Use `role="alert"` for dynamically appearing field errors .
5. Move focus to the first invalid field on submit failure .
6. Use `<fieldset>` and `<legend>` for radio and checkbox groups .

### Annotated Code Examples

```tsx
function AccessibleField({ id, label, error, ...props }) {
  const errorId = `${id}-error`;
  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input
        id={id}
        aria-invalid={error ? 'true' : undefined}
        aria-describedby={error ? errorId : undefined}
        {...props}
      />
      {error && <span id={errorId} role="alert">{error}</span>}
    </div>
  );
}
```

**Expected Output:** Screen readers announce the error when it appears. The input is marked invalid and associated with the message.

**Why this output:** `aria-invalid` communicates validity; `aria-describedby` associates the message; `role="alert"` announces it dynamically .

### Real-World Cases

- **All forms with validation:** Accessible error display is a baseline requirement.
- **WCAG audits:** Error identification (3.3.1) and suggestion (3.3.3) are common findings.
- **Screen-reader testing:** A frequent source of inaccessible forms.

### References

- ualiyou/frontend-engineering — Error Messaging — https://github.com/ualiyou/frontend-engineering/blob/v0.1.0/docs/03-application-architecture/forms-validation/error-messaging.md
- Frontend Accessibility Patterns — https://github.com/mblode/agent-skills
- W3C WAI — Error Identification (WCAG 3.3.1) — https://www.w3.org/WAI/WCAG21/Understanding/error-identification.html

---

## 13. Form Best Practices

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Match the tool to the state category** | Form state → React Hook Form; server state → TanStack Query; URL state → `nuqs` . |
| **Use a form library past trivial complexity** | Hand-rolled form state is a maintenance trap . |
| **Validate on blur and submit** | Avoid premature errors; debounce real-time validation (300ms) . |
| **Share one schema** | Use Zod on both client and server to eliminate drift . |
| **Use `FormProvider` for multi-step** | Share the form instance across steps without prop drilling . |
| **Use `field.id` as the key in `useFieldArray`** | Never use the array index . |
| **Exclude sensitive fields from persistence** | Never save passwords or payment data . |
| **Wire errors accessibly** | `aria-invalid`, `aria-describedby`, `role="alert"`, focus management . |
| **Prefer `sessionStorage` for wizard drafts** | Tab-scoped data; `localStorage` for cross-session drafts . |
| **Use `useActionState` for server-driven forms** | Progressive enhancement and pending state . |
| **Centralise schemas** | Keep schemas in `lib/schemas` for reuse . |
| **Test with assistive technology** | Verify screen-reader announcements and keyboard flows. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Shared schema, accessible errors, isolated state
const schema = z.object({
  email: z.string().email('Email must include an @ and a domain'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

const { register, handleSubmit, formState: { errors } } = useForm({
  resolver: zodResolver(schema),
  mode: 'onBlur',
});

<input
  aria-invalid={!!errors.email}
  aria-describedby={errors.email ? 'email-error' : undefined}
  {...register('email')}
/>
{errors.email && <span id="email-error" role="alert">{errors.email.message}</span>}
```

**❌ Bad Practices:**

```tsx
// ❌ Hand-rolled state for a complex form
const [values, setValues] = useState({});

// ❌ Index as key in useFieldArray
{fields.map((field, index) => <div key={index}>...)}

// ❌ Generic error message
{error && <span>Error</span>}

// ❌ Persisting sensitive fields
useFormStorage('form', form, { include: ['password'] });
```

**Expected Output:** The good practices produce accessible, scalable, and maintainable forms. The bad practices cause maintenance debt, reconciliation bugs, inaccessible errors, and security risks.

**Why this output:** Form libraries handle complex state correctly. Stable keys prevent reconciliation bugs. Specific, associated errors are accessible. Sensitive fields must never be persisted.

### Real-World Cases

- **All production forms:** These practices apply universally.
- **Design systems:** Form components encode these patterns once for reuse.
- **Accessibility audits:** ARIA wiring and focus management are common findings.

### References

- sujeet-pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/client-state-management/README.md#1
- ualiyou/frontend-engineering — Error Messaging — https://github.com/ualiyou/frontend-engineering/blob/v0.1.0/docs/03-application-architecture/forms-validation/error-messaging.md
- Form UX Patterns — https://github.com/bbeierle12/skill-mcp-claude/blob/a35223a33b13a8742e3a8b06ae833fc3e7ed05b3/skills/forms-router/references/ux-patterns.md

---

## References

- sujeet-pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/client-state-management/README.md#1
- modu-ai — React Patterns Reference — https://github.com/modu-ai/moai-adk/blob/4fa2aa9db2e0add0f1299e35407141b2b41c5ec0/internal/template/templates/.claude/skills/moai-ref-react-patterns/SKILL.md
- FreeCodeCamp — A Guide to Modern React Form Architecture: TanStack Form + Zod + Shadcn — https://www.freecodecamp.org/news/a-guide-to-modern-react-form-architecture-tanstack-form-zod-shadcn
- RESEARCH: ADVANCED SHADCN/UI & NEXT.JS 16 PATTERNS — https://github.com
- React Hook Form — useFieldArray — https://react-hook-form.com/docs/usefieldarray
- React Hook Form — FieldArray Component — https://react-hook-form.com/docs/usefieldarray/fieldarray
- React Hook Form — Repeatable Rows — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-forms-react-hook-form/examples/arrays.md
- React Hook Form — Multi-Step Wizard — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-forms-react-hook-form/examples/wizard.md
- Form UX Patterns — https://github.com/bbeierle12/skill-mcp-claude/blob/a35223a33b13a8742e3a8b06ae833fc3e7ed05b3/skills/forms-router/references/ux-patterns.md
- react-hook-form-storage — npm — https://www.npmjs.com/package/react-hook-form-storage
- @sncrr/react-hook-form-persist — npm — https://www.npmjs.com/package/@sncrr/react-hook-form-persist
- TanStack Form Linked Fields Patterns — https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-forms/references/templates/linked-fields-form.md
- Conditional Field Rendering in React Hook Form — https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-forms/references/templates/conditional-fields.md
- ualiyou/frontend-engineering — Error Messaging — https://github.com/ualiyou/frontend-engineering/blob/v0.1.0/docs/03-application-architecture/forms-validation/error-messaging.md
- Frontend Accessibility Patterns — https://github.com/mblode/agent-skills
- React — Managing State — https://uk.react.dev/learn/managing-state
- React Hook Form — FormProvider — https://react-hook-form.com/docs/formprovider
- React — useActionState — https://react.dev/reference/react/useActionState
- React — `<form>` — https://react.dev/reference/react-dom/components/form
- W3C WAI — Error Identification (WCAG 3.3.1) — https://www.w3.org/WAI/WCAG21/Understanding/error-identification.html