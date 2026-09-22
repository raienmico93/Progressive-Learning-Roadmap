# React Conditional Rendering — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

Conditional rendering is the technique of displaying different UI elements or components based on specific conditions, such as state values, props, or application logic.

**Technical Definition**

In React, conditional rendering is achieved using standard JavaScript control flow mechanisms—`if` statements, the ternary operator (`? :`), and the logical AND operator (`&&`)—within JSX expressions. React does not have a special template syntax for conditions; instead, it relies on JavaScript's native conditional constructs to determine which elements to render. When a condition is false, React can return `null` to render nothing, or return an alternative JSX tree .

**Beginner-Friendly Explanation**

Conditional rendering is how you show different things on screen depending on what's happening. For example, if a user is logged in, show their dashboard; if not, show the login form. If data is still loading, show a spinner. If there's an error, show an error message. It's like a traffic light—different lights for different situations.

### Key Characteristics

- **JavaScript-Native**: React uses ordinary JavaScript conditional logic (`if`, `&&`, `? :`), not a special template language .
- **Declarative**: You describe what should appear under each condition, and React handles the DOM updates.
- **Early Return**: Components can return different JSX trees entirely using `if` statements before the main return .
- **Null for Nothing**: Returning `null` from a component renders nothing without breaking the component tree .

### Prerequisites

- React Components and JSX
- State and Props
- Basic JavaScript conditionals (`if/else`, `&&`, `? :`)

### Related Programming Areas

- **Loading States**: Conditional rendering of spinners and skeletons during data fetching .
- **Error Boundaries**: Components that catch errors and render fallback UI .
- **Permission Systems**: Conditional rendering based on user roles and permissions .
- **Empty States**: User-friendly UI when lists or data are empty .

### Core Concepts / Features

- `if` Statements
- Ternary Expressions
- Logical `&&`
- Conditional Component Selection
- Loading States
- Empty States
- Error States
- Permission-Based Rendering

---

## 1. `if` Statements

### Definitions

**Core Definition**

`if` statements are JavaScript conditional constructs used to return entirely different JSX trees from a component based on conditions.

**Technical Definition**

Before the `return` statement in a component, developers can use `if`, `else if`, and `else` to branch logic and assign different JSX to a variable, or use early returns to exit the function with different JSX. This is the most verbose but most flexible conditional rendering technique .

**Beginner-Friendly Explanation**

An `if` statement is like a fork in the road. If something is true, take one path (return one thing); otherwise, take another path. It's the clearest way to handle multiple conditions.

### Purposes

- To return completely different JSX trees based on multiple conditions.
- To handle complex branching logic that cannot be expressed inline.
- To use early returns for cleaner guard clauses.
- To assign JSX to variables before returning.

### Syntax Rules and Structure

**Complete General Syntax (Early Return)**

```jsx
function Component({ condition }) {
  if (condition) {
    return <FirstView />;
  }
  return <SecondView />;
}
```

**Complete General Syntax (Variable Assignment)**

```jsx
function Component({ role }) {
  let content;
  if (role === 'admin') {
    content = <AdminPanel />;
  } else if (role === 'moderator') {
    content = <ModeratorPanel />;
  } else {
    content = <UserPanel />;
  }
  return <div>{content}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `if (condition)` | JavaScript conditional check. |
| `return <FirstView />` | Early return with JSX. |
| `let content;` | Variable to hold the JSX to render. |
| `{content}` | Embedding the variable in JSX. |

**Syntax Rules**

1. **`if` cannot be used directly inside JSX**: It is a statement, not an expression. Use it before the `return` .
2. **Early return pattern**: Return the first matching condition immediately.
3. **Variable assignment pattern**: Assign JSX to a variable inside `if/else` blocks, then embed the variable in JSX.

**Constraints and Limitations**

- More verbose than inline options.
- Cannot be used inside JSX curly braces.
- For simple two-way conditions, ternary is more concise.

### Annotated Complete Code Examples

**Example 1: Early Return for Authentication**

```jsx
function LoginMessage({ isLoggedIn, username }) {
  // Early return if logged in
  if (isLoggedIn) {
    return <h1>Welcome back, {username}!</h1>;
  }
  // Fallback if not logged in
  return <h1>Please sign in.</h1>;
}
```

**Expected Output (isLoggedIn = true)**

```
Welcome back, Alice!
```

**Why This Output Occurs**

The `if (isLoggedIn)` condition is true, so the component returns the welcome message immediately. The second return is never reached .

**Example 2: Multiple Conditions with Variable Assignment**

```jsx
function Dashboard({ user }) {
  let content;

  if (user.role === 'admin') {
    content = <AdminPanel />;
  } else if (user.role === 'moderator') {
    content = <ModeratorPanel />;
  } else {
    content = <UserPanel />;
  }

  return (
    <div className="dashboard">
      <h1>Dashboard</h1>
      {content}
    </div>
  );
}
```

**Expected Output (user.role = 'moderator')**

```
Dashboard
[Moderator Panel]
```

**Why This Output Occurs**

The `else if` chain checks each role. `moderator` matches the second condition, so `content` is assigned `<ModeratorPanel />`. The variable is then embedded in the returned JSX .

### Real-World Cases

**Case: Multi-Role Application**

An enterprise application uses `if/else if/else` to render entirely different navigation menus, dashboards, and feature sets based on user roles (admin, manager, employee, guest).

### References

- React — Conditional Rendering (Official) - https://react.dev/learn/conditional-rendering
- React (Russian) — Условная отрисовка - https://ru.react.js.org/docs/conditional-rendering.html

---

## 2. Ternary Expressions

### Definitions

**Core Definition**

The ternary operator (`? :`) is a JavaScript expression that returns one of two values based on a condition, commonly used for inline conditional rendering in JSX.

**Technical Definition**

The conditional (ternary) operator `condition ? trueValue : falseValue` is an expression, meaning it can be embedded directly inside JSX curly braces. React commonly uses it to choose between two JSX elements or strings without breaking out of the JSX context .

**Beginner-Friendly Explanation**

A ternary is like a quick question: "Is this true? If yes, show this; if no, show that." It's a compact way to choose between two options right in the middle of your JSX.

### Purposes

- To render one of two JSX elements inline based on a condition.
- To conditionally apply class names, styles, or text content.
- To keep JSX concise for simple two-way conditions.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{condition ? <TrueComponent /> : <FalseComponent />}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Any JavaScript expression evaluating to truthy/falsy. |
| `?` | Separator between condition and true value. |
| `:` | Separator between true and false values. |

**Syntax Rules**

1. **Both branches are required**: Unlike `&&`, the ternary always needs a false branch.
2. **Can return strings or JSX**: Both branches can be any expression, including strings and elements .
3. **Avoid deep nesting**: More than two levels of nested ternaries harms readability .

**Constraints and Limitations**

- Nested ternaries become difficult to read beyond two levels.
- Both branches must be expressions; statements cannot be used.

### Annotated Complete Code Examples

**Example 1: Inline Text Toggle**

```jsx
function UserStatus({ isOnline }) {
  return (
    <span className={isOnline ? 'status-online' : 'status-offline'}>
      {isOnline ? '🟢 Online' : '🔴 Offline'}
    </span>
  );
}
```

**Expected Output (isOnline = true)**

```
🟢 Online
```

**Why This Output Occurs**

The ternary `isOnline ? '🟢 Online' : '🔴 Offline'` evaluates to the string '🟢 Online' when `isOnline` is true. The same condition also selects the CSS class .

**Example 2: Conditional Component Rendering**

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
    </div>
  );
}
```

**Explanation**

The ternary chooses between two components. When `isLoggedIn` is true, `<UserGreeting />` is rendered; otherwise, `<GuestGreeting />` .

### Real-World Cases

**Case: Theme Toggle**

A component uses a ternary to switch between light and dark mode icons: `{isDark ? <MoonIcon /> : <SunIcon />}`.

### References

- React — Conditional Rendering (Ternary) - https://react.dev/learn/conditional-rendering
- React (Russian) — Встроенный if-else с тернарным оператором - https://ru.react.js.org/docs/conditional-rendering.html

---

## 3. Logical `&&`

### Definitions

**Core Definition**

The logical AND operator (`&&`) renders a JSX element only when a condition is true, rendering nothing (or the falsy value) when false.

**Technical Definition**

In JavaScript, `true && expression` evaluates to `expression`, while `false && expression` evaluates to `false`. React ignores `false`, `null`, and `undefined` in JSX, so `condition && <Element />` renders the element only when the condition is truthy, and renders nothing otherwise .

**Beginner-Friendly Explanation**

`&&` is a shorthand for "show this only if the condition is true." It's like saying "If it's sunny, bring an umbrella" — but you only bring the umbrella when it's sunny, and do nothing when it's not.

### Purposes

- To render an element only when a condition is true, with no alternative.
- To conditionally include optional UI elements (badges, messages, buttons).
- To keep JSX concise for single-branch conditions.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{condition && <Element />}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Boolean expression (must be boolean, not a number). |
| `&&` | Logical AND operator. |
| `<Element />` | JSX to render if condition is true. |

**Syntax Rules**

1. **Use boolean conditions**: Always use `count > 0 &&` not `count &&` to avoid rendering `0` .
2. **No else branch**: `&&` only handles the true case; use ternary for two-way conditions.
3. **Left side must be boolean**: Non-boolean falsy values (0, '') will be rendered by React .

**Constraints and Limitations**

- `0 && <Element />` renders `0`, not nothing. This is a common pitfall .
- `'' && <Element />` renders empty string (invisible, but still in the DOM).
- Cannot provide a fallback for the false case.

### Annotated Complete Code Examples

**Example 1: Notification Badge**

```jsx
function Notifications({ messages }) {
  return (
    <div>
      <h2>Inbox</h2>
      {messages.length > 0 && (
        <p>You have {messages.length} unread messages.</p>
      )}
    </div>
  );
}
```

**Expected Output (messages = ['a', 'b'])**

```
Inbox
You have 2 unread messages.
```

**Expected Output (messages = [])**

```
Inbox
```

**Why This Output Occurs**

`messages.length > 0` evaluates to `true` when there are messages, so the `<p>` renders. When empty, it evaluates to `false`, and React renders nothing for that expression .

**Example 2: The `0` Pitfall**

```jsx
function ItemCount({ count }) {
  return (
    <div>
      {/* ❌ Wrong: renders "0" when count is 0 */}
      {count && <p>{count} items</p>}
      
      {/* ✅ Correct: explicit boolean check */}
      {count > 0 && <p>{count} items</p>}
    </div>
  );
}
```

**Explanation**

When `count` is `0`, `0 && <p>` evaluates to `0`, which React renders as the number `0`. Always use a boolean comparison like `count > 0` .

### Real-World Cases

**Case: Premium Feature Badge**

A "Pro" badge is rendered only for premium users: `{user.isPremium && <ProBadge />}`. Non-premium users see nothing.

### References

- React — Conditional Rendering (`&&`) - https://react.dev/learn/conditional-rendering
- React (Russian) — Встроенный оператор if с логическим && - https://ru.react.js.org/docs/conditional-rendering.html

---

## 4. Conditional Component Selection

### Definitions

**Core Definition**

Conditional component selection is the technique of choosing which component to render from a set of possibilities based on a state or prop value.

**Technical Definition**

Instead of using `if/else` or ternaries to switch between components, developers can use a variable holding a component reference, a `switch` statement, or a mapping object to select and render the appropriate component. This is useful when there are more than two possible components to render .

**Beginner-Friendly Explanation**

Conditional component selection is like a TV remote—you press a button (change state) and the channel (component) changes. You have many channels, but only one shows at a time.

### Purposes

- To render different components based on a selection state.
- To handle tabbed interfaces or multi-step wizards.
- To avoid deeply nested ternaries or `if/else` chains.

### Syntax Rules and Structure

**Complete General Syntax (Variable + Switch)**

```jsx
function TabContent({ activeTab }) {
  let content;
  switch (activeTab) {
    case 'profile':
      content = <Profile />;
      break;
    case 'settings':
      content = <Settings />;
      break;
    default:
      content = <Home />;
  }
  return <div>{content}</div>;
}
```

**Complete General Syntax (Mapping Object)**

```jsx
const components = {
  profile: Profile,
  settings: Settings,
  home: Home
};

function TabContent({ activeTab }) {
  const Component = components[activeTab] || Home;
  return <Component />;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `switch (activeTab)` | JavaScript switch statement. |
| `case 'profile':` | Match condition. |
| `content = <Profile />` | Assign the component to render. |
| `components[activeTab]` | Lookup component by key. |

**Syntax Rules**

1. **Capitalize component variables**: `const Component = components[key]` then `<Component />` (not `<component />`).
2. **Use `default`**: Always provide a fallback for unknown values.
3. **Mapping object is concise**: For many options, a key-to-component map is cleaner than `switch`.

**Constraints and Limitations**

- The component variable must be capitalized when used in JSX.
- Dynamic component selection can complicate static analysis and code splitting.

### Annotated Complete Code Examples

**Example 1: Tab Selection with State**

```jsx
function AboutUs() {
  const [selected, setSelected] = useState('bio');

  function showSelectedOption() {
    switch (selected) {
      case 'details':
        return <PersonalDetails />;
      case 'bio':
        return <ProfileBio />;
      case 'qualif':
        return <ProfileQualifications />;
      default:
        return <ProfileBio />;
    }
  }

  return (
    <div>
      <button onClick={() => setSelected('details')}>Personal Details</button>
      <button onClick={() => setSelected('bio')}>Bio</button>
      <button onClick={() => setSelected('qualif')}>Qualifications</button>
      <div>{showSelectedOption()}</div>
    </div>
  );
}
```

**Expected Output (clicking "Personal Details")**

```
[Personal Details component rendered]
```

**Why This Output Occurs**

Clicking the button calls `setSelected('details')`. The `switch` statement matches `'details'` and returns `<PersonalDetails />`. React re-renders with the new component .

**Example 2: Mapping Object for Cleaner Code**

```jsx
const TAB_COMPONENTS = {
  details: PersonalDetails,
  bio: ProfileBio,
  qualif: ProfileQualifications
};

function AboutUs() {
  const [selected, setSelected] = useState('bio');
  const Component = TAB_COMPONENTS[selected] || ProfileBio;

  return (
    <div>
      <button onClick={() => setSelected('details')}>Personal Details</button>
      <button onClick={() => setSelected('bio')}>Bio</button>
      <button onClick={() => setSelected('qualif')}>Qualifications</button>
      <div><Component /></div>
    </div>
  );
}
```

**Explanation**

The mapping object replaces the `switch` statement. `Component` is looked up by key and rendered. This pattern scales better for many options and is easier to read.

### Real-World Cases

**Case: Settings Page Tabs**

A settings page has tabs for Account, Notifications, Privacy, and Billing. Clicking each tab sets state, and the mapping object selects the corresponding component.

### References

- Stack Overflow — Conditional render 1 of 3 components - https://stackoverflow.com/posts/67644596/revisions
- npm — react-context-switch - https://www.npmjs.com/package/react-context-switch

---

## 5. Loading States

### Definitions

**Core Definition**

Loading states are conditional UI displays shown while asynchronous operations (data fetching, form submission) are in progress, providing feedback to users.

**Technical Definition**

A loading state is tracked with a boolean `useState` variable (`loading`). Before an async operation, `setLoading(true)` is called; in the `finally` block (or after completion), `setLoading(false)` is called. The component conditionally renders a spinner, skeleton, or loading message while `loading` is true .

**Beginner-Friendly Explanation**

A loading state is like a "Please wait" sign. While the app is fetching data from the server, it shows a spinner so the user knows something is happening.

### Purposes

- To provide visual feedback during async operations.
- To improve perceived performance by showing activity.
- To prevent users from interacting with incomplete data.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const [loading, setLoading] = useState(false);

async function fetchData() {
  setLoading(true);
  try {
    const data = await fetch('/api/data');
    // process data
  } catch (error) {
    // handle error
  } finally {
    setLoading(false); // Always runs
  }
}

if (loading) return <Spinner />;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `loading` | Boolean state. |
| `setLoading(true)` | Start loading before async call. |
| `finally { setLoading(false) }` | Stop loading regardless of success/failure. |

**Syntax Rules**

1. **Use `finally`**: Ensures `loading` is set to `false` even if the request fails .
2. **Early return**: Return the loading UI before rendering the main content.
3. **Skeletons over spinners**: Consider skeleton loaders for better perceived performance .

**Constraints and Limitations**

- Multiple concurrent requests need separate loading states or a counter.
- Loading states add boilerplate; libraries like React Query manage them automatically.

### Annotated Complete Code Examples

**Example 1: Loading Spinner Pattern**

```jsx
function DataFetcher() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchData() {
      try {
        const response = await fetch('/api/data');
        if (!response.ok) throw new Error('Failed to fetch');
        setData(await response.json());
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    fetchData();
  }, []);

  if (loading) {
    return (
      <div className="spinner-container">
        <div className="spinner"></div>
        <p>Loading...</p>
      </div>
    );
  }

  if (error) {
    return <div className="error">Error: {error}</div>;
  }

  return <div>{data.title}</div>;
}
```

**Expected Behavior**

While fetching, a spinner with "Loading..." is shown. After success, the data title appears. On error, the error message appears.

**Why This Occurs**

`loading` starts as `true`. The async function runs, and in `finally`, `setLoading(false)` triggers a re-render. The component then renders either the error or the data .

### Real-World Cases

**Case: Dashboard Data Loading**

A dashboard shows a skeleton layout while fetching statistics, metrics, and recent activity from multiple API endpoints.

### References

- CoreUI — How to handle loading spinners in React - https://coreui.io/answers/how-to-handle-loading-spinners-in-react/
- CoreUI — How to show loading state in React - https://coreui.io/answers/how-to-show-loading-state-in-react/

---

## 6. Empty States

### Definitions

**Core Definition**

An empty state is a UI display shown when a list, collection, or data set contains no items, providing context and guidance to the user.

**Technical Definition**

An empty state is rendered when a condition like `items.length === 0` is true and no loading or error condition is active. It typically includes an icon, a message explaining the emptiness, and an action button to help the user get started .

**Beginner-Friendly Explanation**

An empty state is what you show when there's nothing to show. Instead of a blank page, you show a friendly message like "No products yet" with a button to add your first product.

### Purposes

- To provide context when data is absent.
- To guide users toward taking action (creating their first item).
- To avoid confusing blank screens.
- To improve first-time user experience.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
function ItemList({ items, loading }) {
  if (loading) return <Skeleton />;
  
  if (items.length === 0) {
    return (
      <EmptyState
        icon={<PackageIcon />}
        title="No items yet"
        description="Create your first item to get started"
        action={<button>Add Item</button>}
      />
    );
  }
  
  return items.map(item => <Item key={item.id} {...item} />);
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `items.length === 0` | Condition for empty state. |
| `<EmptyState />` | Component displaying the empty UI. |
| `action` | Button or link to guide next steps. |

**Syntax Rules**

1. **Check `!loading` first**: Only show empty state after loading completes .
2. **Include actionable guidance**: Tell users what to do next.
3. **Use appropriate icons**: Visual context reinforces the message.

**Constraints and Limitations**

- Distinguish between "no data" and "data not yet loaded."
- Empty states should not be shown while loading or on error.

### Annotated Complete Code Examples

**Example 1: Empty Product List**

```jsx
function ProductList({ products, loading }) {
  if (loading) {
    return Array.from({ length: 5 }).map((_, i) => (
      <ProductSkeleton key={i} />
    ));
  }

  if (products.length === 0) {
    return (
      <div className="empty-state">
        <PackageIcon size={48} />
        <h3>No products yet</h3>
        <p>Create your first product to get started with invoicing.</p>
        <Link to="/products/new">
          <PlusIcon /> Create product
        </Link>
      </div>
    );
  }

  return products.map(p => <ProductItem key={p.id} {...p} />);
}
```

**Expected Output (empty products)**

```
[Package icon]
No products yet
Create your first product to get started with invoicing.
[Create product button]
```

**Why This Output Occurs**

`products.length === 0` is true and `loading` is false, so the empty state renders. The message guides the user to create their first product .

### Real-World Cases

**Case: Invoice Management**

An invoice app shows an empty state on the Clients, Products, and Invoices pages when no items exist, each with a tailored message and "Add" button .

### References

- GitHub Issue — Add Empty State Component to List Pages - https://github.com/stephane71/invoices-manager/issues/63
- GitHub — PositronList Empty State Renderer - https://github.com/posit-dev/positron/blob/052d0b32/src/vs/workbench/browser/positronList/positronList.tsx

---

## 7. Error States

### Definitions

**Core Definition**

An error state is a UI display shown when an operation fails, providing the user with information about what went wrong and possibly how to recover.

**Technical Definition**

Error states are managed with a `useState` variable (`error`). When an async operation throws or returns an error, `setError(message)` is called. The component conditionally renders an error message, often with a retry button. For component crashes, React Error Boundaries catch errors and render fallback UI .

**Beginner-Friendly Explanation**

An error state is what you show when something goes wrong. Instead of a broken page, you show a friendly message like "Something went wrong" with a button to try again.

### Purposes

- To inform users when operations fail.
- To provide recovery options (retry, go back).
- To prevent complete application crashes.
- To log errors for debugging.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const [error, setError] = useState(null);

try {
  await fetchData();
} catch (err) {
  setError(err.message);
}

if (error) {
  return <ErrorDisplay message={error} onRetry={handleRetry} />;
}
```

**Complete General Syntax (Error Boundary)**

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    logError(error, info);
  }
  
  render() {
    if (this.state.hasError) {
      return <FallbackUI />;
    }
    return this.props.children;
  }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `error` | State holding error message or null. |
| `ErrorBoundary` | Class component that catches rendering errors. |
| `getDerivedStateFromError` | Updates state when error occurs. |
| `componentDidCatch` | Logs error details. |

**Syntax Rules**

1. **Error boundaries catch render errors**: They do not catch errors in event handlers or async code .
2. **Use `try/catch` for async**: Error boundaries do not catch async errors; use `try/catch` in async functions .
3. **Provide retry**: Give users a way to recover from transient errors .

**Constraints and Limitations**

- Error boundaries only catch errors during rendering, lifecycle, and constructors.
- Event handler errors and async errors need `try/catch`.
- Error boundaries must be class components (no Hook equivalent).

### Annotated Complete Code Examples

**Example 1: API Error State**

```jsx
function DataFetcher() {
  const [error, setError] = useState(null);

  async function fetchData() {
    try {
      const res = await fetch('/api/data');
      if (!res.ok) throw new Error('Failed to fetch');
      // ...
    } catch (err) {
      setError(err.message);
    }
  }

  if (error) {
    return (
      <div className="error">
        <p>Error: {error}</p>
        <button onClick={() => { setError(null); fetchData(); }}>
          Retry
        </button>
      </div>
    );
  }

  // ...
}
```

**Expected Output (on fetch failure)**

```
Error: Failed to fetch
[Retry button]
```

**Why This Output Occurs**

The `catch` block sets the error message. The component conditionally renders the error UI. Clicking Retry clears the error and re-fetches .

**Example 2: Error Boundary**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error('Caught error:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

**Explanation**

If `MyComponent` throws during rendering, the Error Boundary catches it and renders the fallback UI instead of crashing the entire app .

### Real-World Cases

**Case: Payment Processing**

A checkout page wraps the payment form in an Error Boundary. If the payment component crashes, a fallback UI with a retry option is shown, preventing the whole page from breaking .

### References

- CoreUI — How to show error state in React - https://coreui.io/answers/how-to-show-error-state-in-react/
- Sentry Blog — Guide to Error & Exception Handling in React - https://sentry-blog.sentry.dev/guide-to-error-and-exception-handling-in-react/
- GitHub PR — Error Handling Boundaries - https://github.com/ALIPHATICHYD/Soroban-Registry/pull/209

---

## 8. Permission-Based Rendering

### Definitions

**Core Definition**

Permission-based rendering is the technique of showing or hiding UI elements based on the user's role, permissions, or access level.

**Technical Definition**

Permission-based rendering uses a permission provider (context) that evaluates the user's permissions against required permissions. Components or hooks check permissions and conditionally render UI. Common patterns include a `<Can>` component, a `usePermission` hook, or a `PermissionGate` wrapper .

**Beginner-Friendly Explanation**

Permission-based rendering is like a VIP section at a club. Only people with the right wristband (permission) can see or use certain features. If you're not an admin, you don't see the admin button.

### Purposes

- To hide UI elements the user is not authorized to use.
- To show admin-only controls.
- To enforce access control at the UI level.
- To provide a consistent permission model across the application.

### Syntax Rules and Structure

**Complete General Syntax (Hook)**

```jsx
const { can } = usePermissions();

return (
  <div>
    {can('delete', 'posts') && <DeleteButton />}
    {can('edit', 'posts') && <EditButton />}
  </div>
);
```

**Complete General Syntax (Component)**

```jsx
<Can do="delete" on="posts" fallback={<p>Access denied</p>}>
  <DeleteButton />
</Can>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `usePermissions()` | Hook providing permission checks. |
| `can(action, resource)` | Returns boolean. |
| `<Can do on>` | Declarative permission wrapper. |

**Syntax Rules**

1. **Centralize permissions**: Define roles and permissions in a config file .
2. **Use a provider**: Wrap the app in `PermissionsProvider` with the user's role and permissions .
3. **Check at the right level**: Hide the UI element, not just disable it, for security-sensitive features.

**Constraints and Limitations**

- UI-level permissions are not security; server-side checks are still required.
- Over-hiding can confuse users; consider showing disabled states with explanations.

### Annotated Complete Code Examples

**Example 1: Hook-Based Permission Check**

```jsx
import { usePermissions } from 'react-permissions-solution';

function PostActions() {
  const { can } = usePermissions();

  return (
    <div>
      {can('read', 'posts') && <PostList />}
      {can('edit', 'posts') && <EditButton />}
      {can('delete', 'posts') && <DeleteButton />}
      {can('approve', 'posts') && <ApproveButton />}
    </div>
  );
}
```

**Expected Output (user has read and edit, not delete/approve)**

```
[PostList] [EditButton]
```

**Why This Output Occurs**

`can()` returns true only for permissions the user has. Elements for missing permissions are not rendered .

**Example 2: Declarative `<Can>` Component**

```jsx
import { Can } from 'react-permissions-solution';

function AdminPanel() {
  return (
    <div>
      <Can do="delete" on="posts" fallback={<p>You don't have permission.</p>}>
        <DeleteButton />
      </Can>
      
      <Can do={['edit', 'publish']} on="posts" mode="any">
        <PublishButton />
      </Can>
      
      <Can do="*" role="admin">
        <AdminBadge />
      </Can>
    </div>
  );
}
```

**Explanation**

The `<Can>` component checks permissions and renders its children only if authorized. The `fallback` prop provides alternative UI. The `mode` prop controls whether all or any permissions are required .

### Real-World Cases

**Case: Admin Dashboard**

An admin dashboard uses permission-based rendering to show different widgets for admins, moderators, and regular users. The `usePermissions` hook checks each widget's required permission.

### References

- npm — react-permissions-solution - https://www.npmjs.com/package/react-permissions-solution
- npm — permission-gate - https://www.npmjs.com/package/permission-gate
- Socket — @ldauth/react Permissions - https://socket.dev/npm/package/%40ldauth%2Freact

---

## References

- React Official Documentation — Conditional Rendering - https://react.dev/learn/conditional-rendering
- React (Russian) — Условная отрисовка - https://ru.react.js.org/docs/conditional-rendering.html
- CoreUI — How to handle loading spinners in React - https://coreui.io/answers/how-to-handle-loading-spinners-in-react/
- CoreUI — How to show loading state in React - https://coreui.io/answers/how-to-show-loading-state-in-react/
- CoreUI — How to show error state in React - https://coreui.io/answers/how-to-show-error-state-in-react/
- Sentry Blog — Guide to Error & Exception Handling in React - https://sentry-blog.sentry.dev/guide-to-error-and-exception-handling-in-react/
- npm — react-permissions-solution - https://www.npmjs.com/package/react-permissions-solution
- npm — permission-gate - https://www.npmjs.com/package/permission-gate
- Socket — @ldauth/react - https://socket.dev/npm/package/%40ldauth%2Freact
- GitHub Issue — Add Empty State Component to List Pages - https://github.com/stephane71/invoices-manager/issues/63
- GitHub — ListViewState Component - https://github.com/jfolcini/agaric/blob/b7a8f4a0b4c3984109160f8f2cf6bb4e9c1a03fb/src/components/common/ListViewState.tsx
- GitHub PR — Error Handling Boundaries - https://github.com/ALIPHATICHYD/Soroban-Registry/pull/209
- Stack Overflow — Conditional render 1 of 3 components - https://stackoverflow.com/posts/67644596/revisions
- npm — react-context-switch - https://www.npmjs.com/package/react-context-switch
- GitHub — KODR Week 6 Conditional Rendering - https://github.com/Bloivating-Major/KODR/blob/master/Week%206/Week%206.2/Readme.md
- GitHub — okf-bundles Conditional Rendering Summary - https://github.com/saschb2b/okf-bundles/blob/main/bundles/react/describing-the-ui/conditional-rendering.md