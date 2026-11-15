# React State Management Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** State management is the practice of classifying, placing, and updating the data an application remembers — server responses, UI flags, URL parameters, form drafts, and persisted preferences — using the mechanism that matches each category.

**Technical Definition:** State management in React is not a single problem but a **taxonomy problem**. Every piece of state has three axes: **owner** (who is the authority — the server, the client, the URL, the browser, or a form), **lifetime** (one render, one mount, one session, or across sessions), and **scope** (one component, one subtree, one route, or the entire application). Classify a value on those axes and the correct mechanism follows almost mechanically: server-owned data belongs in a query cache, shareable view state belongs in the URL, a modal flag belongs in the component that owns the modal, and anything computable is derived rather than stored. The pain teams attribute to "state management" is almost always a **category error**: server data copied into a store, view state hidden in a component, or a global store holding things that belong to one screen.

**Beginner-Friendly Explanation:** State is everything your app needs to remember. But "remembering" is not one job — it is several very different jobs wearing the same name. Remembering what the server said is different from remembering whether a dropdown is open; remembering a filter is different from remembering a draft form; remembering a total is different from remembering a login token. Most state bugs come from putting one kind of thing in the wrong kind of storage — like putting server data into a global store and then fighting cache invalidation forever. This cheat sheet teaches you to **classify first, choose a tool second**.

### Key Characteristics

- **Classification precedes tooling:** Name the owner, lifetime, and scope of a piece of state in one sentence; if you cannot, it is probably two things that should be split.
- **Server state is a cache, not owned state:** It is stale on arrival, shared across components, and changes without the client knowing.
- **Shareable state belongs in the URL:** Filters, sort, page, tab, selected ID, and search query should survive reload and be linkable.
- **Local state is the default:** It is the cheapest to reason about and delete; lift and globalize only on evidence of a real second consumer.
- **Derived state is not state:** A total, a filtered list, a count, or an `isValid` flag should be computed, never stored.
- **One source of truth per fact:** Two places that both claim to know the current user will eventually disagree.
- **The 2026 consensus is clear:** Push server state to specialised libraries (TanStack Query, SWR), keep global client state minimal (Zustand, Jotai), let URLs carry shareable state, and use React Hook Form for performance-critical forms.

### Prerequisites

- React function components, `useState`, `useEffect`, and `useReducer`.
- JavaScript Promises and `async`/`await`.
- HTTP fundamentals (methods, status codes, headers) for server state.
- Basic understanding of URL structure and browser storage APIs.
- Familiarity with React Context (for shared state).

### Related Programming Areas

- **Server State Management** — TanStack Query, SWR, Apollo Client.
- **Global Client State** — Zustand, Jotai, Redux Toolkit, Context.
- **URL State Management** — React Router, TanStack Router, `nuqs`.
- **Form State Management** — React Hook Form, Formik, `useActionState`.
- **Performance** — selectors, memoization, and re-render control.

---

## 1. State Management Fundamentals & Categories

### Definitions

**Core Definition:** State categories are the distinct buckets into which application state is classified — server, client (UI), URL, form, persistent, and derived — each with its own characteristics, lifecycle, and correct tool.

**Technical Definition:** State divides along three axes: **owner** (server, client, URL, browser/platform, form), **lifetime** (this render, this mount, this session, across sessions), and **scope** (component, subtree, route, application). The categories are: **Server state** (authority lives on a server; the client holds a stale cache), **Client state** (the client is the authority: modal visibility, wizard step, selected row, unsaved draft), **URL state** (what the user is looking at: route parameters, filters, sort, page, open tab), **Browser/platform state** (persisted preferences and tokens in `localStorage`, `sessionStorage`, cookies, or IndexedDB, plus platform facts like online status and viewport size), **Form state** (a special case of client state with its own vocabulary of values, dirty, touched, and validity), and **Derived state** (not state at all — values computed from any of the above).

**Beginner-Friendly Explanation:** There are six kinds of state. Server state is what the server knows. Client state is what the user is doing in the app right now. URL state is what should be shareable. Persistent state is what should survive a reload. Form state is what the user is typing. Derived state is what you can calculate from the others. Each one belongs in a different place.

### Purposes

- To prevent category errors that cause cache-invalidation nightmares, unshareable URLs, and stale derived values.
- To choose the mechanism that provides the right guarantees for free (caching, staleness, shareability, persistence).
- To make state decisions reviewable by two engineers instead of a matter of taste.
- To avoid the compounding maintenance cost of misclassification.

### Syntax Rules and Structure

**Classification decision flow** (ask in order, stop when the state is placed):

| Question | Answer | Category |
|----------|--------|----------|
| Who is the authority? | A server | **Server state** → query cache (cache, don't own) |
| Who is the authority? | The client | → next question |
| Should the user be able to link, bookmark, or reload into it? | Yes | **URL state** → route params / search params |
| Can it be computed from something you already have? | Yes | **Derived state** → compute on read, never store |
| How many consumers? | One component | **Local state** |
| How many consumers? | One subtree | Lift to closest common owner |
| How many consumers? | App-wide, cross-route | **Global store** |
| Must survive a reload? | Yes | **Persistent state** → storage + explicit hydration |

**Syntax Rules:**

1. **Authority beats scope.** A piece of server data read by one component is still server state; putting it in `useState` because "only one component needs it" is the most common category error.
2. **Ask about actual consumers, not anticipated ones.** A second consumer that does not exist yet is not a reason to globalize.
3. **Lifetime is the forgotten axis.** State that must survive a reload needs an explicit persistence and hydration story, including versioning and fallback.
4. **State that must survive only a mount** is often better held outside React entirely (a ref) if it does not affect rendering.
5. **Model state so invalid combinations cannot exist.** `isLoading`, `isError`, and `data` as three independent fields admits nonsense; a single discriminated value does not.

**Constraints and Limitations:**

- The taxonomy is a thinking tool, not a runtime cost; there is never a reason to skip it.
- Cross-cutting state (e.g., a value that is both server-owned and URL-driven) must be split into its constituent categories rather than forced into one.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Classifying State Before Writing Code

```tsx
// A checkout screen has four pieces of state:
// 1. The cart items (from the server)
// 2. Whether the promo-code drawer is open (UI)
// 3. The applied promo code (URL + server)
// 4. The order total (derived)

// ❌ Wrong: everything in one global store
const useCheckoutStore = create((set) => ({
  cartItems: [],
  isPromoDrawerOpen: false,
  promoCode: null,
  total: 0,
  setTotal: (total) => set({ total }), // Stored derivation
}));

// ✅ Correct: each category in its own home
// Server state → TanStack Query
const { data: cartItems } = useQuery({ queryKey: ['cart'], queryFn: fetchCart });

// UI state → local useState
const [isPromoDrawerOpen, setPromoDrawerOpen] = useState(false);

// URL state → search params
const [searchParams] = useSearchParams();
const promoCode = searchParams.get('promo');

// Derived state → computed during render
const total = cartItems.reduce((sum, item) => sum + item.price * item.qty, 0);
```

**Expected Output:** The cart is cached and revalidated by TanStack Query; the drawer flag never causes global re-renders; the promo code survives reload and is shareable; the total cannot go stale because it is recomputed from `cartItems`.

**Why this output:** Each piece of state is placed according to its owner. Server data in a store would require reimplementing caching, deduplication, and background sync. A stored total would need synchronization code and would eventually disagree with the items.

#### Example 2: The Category-Error Diagnostic

```tsx
// Symptom: two screens disagree after a mutation, nothing refetches on tab return
// Diagnosis: server data is copied into useState inside a provider
function BadInvoicesProvider({ children }) {
  const [invoices, setInvoices] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    fetchInvoices()
      .then(setInvoices)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, []);

  // Two screens read this copy; mutations elsewhere never update it
  return <Ctx.Provider value={{ invoices, isLoading, error }}>{children}</Ctx.Provider>;
}

// Fix: treat server data as a cache, not as owned state
function InvoicesList() {
  const { data, isPending, error } = useQuery({
    queryKey: ['invoices'],
    queryFn: fetchInvoices,
  });
  // The query cache is the single source of truth; every consumer stays in sync
}
```

**Expected Output:** After the fix, every screen reading invoices sees the same cache entry, mutations invalidate it automatically, and returning to the tab triggers background revalidation.

**Why this output:** Copying server data into `useState` creates a second source of truth that no mutation can reach. A query cache provides deduplication, staleness, background revalidation, retry, and cancellation for free.

### Real-World Cases

- **E-commerce checkout:** Cart (server), drawer state (UI), promo code (URL + server), total (derived).
- **Dashboards:** Widget data (server), filter panel state (URL), active tab (URL), loading flags (local).
- **Multi-step wizards:** Step index (local/URL), draft values (form/persistent), validation errors (derived).
- **Settings pages:** Preferences (persistent + server), unsaved-changes flag (local/form).

### References

- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1
- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md
- MAO Sovereign Architect — State Management Design — https://raw.githubusercontent.com/javimontano/mao-sovereign-architect/refs/heads/main/skills/state-management-design/SKILL.md

---

## 2. Local State

### Definitions

**Core Definition:** Local state is state that belongs to a single component and is not shared with siblings or distant parts of the tree.

**Technical Definition:** Local component state is state owned by one component instance. It is not shared with siblings and is managed with `useState` (for simple, independent values) or `useReducer` (for complex transitions with multiple sub-values). React preserves local state as long as the component renders in the same position in the UI tree; if the component is removed or a different component renders in the same position, React discards the state.

**Beginner-Friendly Explanation:** Local state is a component's private memory — the dropdown's open flag, the input's current value, the accordion's expansion. Nobody else needs it, so nobody else should see it.

### Purposes

- To manage UI concerns that no other component needs.
- To keep state as close as possible to where it is used, minimising re-renders and coupling.
- To provide the cheapest state to reason about, test, and delete.
- To serve as the starting point before lifting or globalizing on evidence.

### Syntax Rules and Structure

**`useState` for independent values:**

```tsx
const [isOpen, setIsOpen] = useState(false);
const [query, setQuery] = useState('');
```

**`useReducer` for complex transitions:**

```tsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { ...state, count: state.count + state.step };
    case 'setStep': return { ...state, step: action.payload };
    default: return state;
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
```

**Syntax Rules:**

1. Start with `useState`; escalate to `useReducer` when the next state depends on the previous and transitions follow patterns.
2. Use functional updates (`setCount(c => c + 1)`) to avoid stale state.
3. Use lazy initialization (`useState(() => expensive())`) for expensive initial values.
4. Do not store values that can be computed from other local state.
5. Lift state only when a second component genuinely needs it.

**Constraints and Limitations:**

- Local state is lost when the component unmounts or its position in the tree changes.
- Local state cannot be shared with siblings without lifting.

### Annotated Code Examples

```tsx
function Accordion({ items }) {
  const [openIndex, setOpenIndex] = useState(null);

  return items.map((item, index) => (
    <section key={item.id}>
      <button
        aria-expanded={openIndex === index}
        onClick={() => setOpenIndex(openIndex === index ? null : index)}
      >
        {item.title}
      </button>
      {openIndex === index && <div>{item.content}</div>}
    </section>
  ));
}
```

**Expected Output:** Only one accordion section is open at a time; the state is private to `Accordion`.

**Why this output:** `openIndex` is local because no sibling or distant component needs to know which section is open. Lifting it would add coupling with no benefit.

### Real-World Cases

- **Accordions:** Tracking the open section.
- **Toggles:** Modal, drawer, and dropdown visibility.
- **Inputs:** Individual field values in a single component.
- **Tabs:** The active tab within a tab group.

### References

- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md
- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management

---

## 3. UI State

### Definitions

**Core Definition:** UI state is synchronous, ephemeral state that controls how the interface looks and behaves — independent of any server data.

**Technical Definition:** UI state is owned by the component tree and is synchronous, ephemeral, and scoped. It includes modal and drawer visibility, dropdown selections, accordion expansion, animation states, hover and focus flags, and component-specific toggles. UI state should live as close to where it is used as possible; lifting it to a global store creates unnecessary coupling and re-renders.

**Beginner-Friendly Explanation:** UI state is "what the interface is doing right now" — is the menu open, which tab is active, is the tooltip showing. It has nothing to do with the server.

### Purposes

- To control visibility, selection, expansion, and animation.
- To provide immediate, synchronous feedback for user interactions.
- To keep interface concerns isolated from server and URL concerns.
- To avoid unnecessary global re-renders from local UI flags.

### Syntax Rules and Structure

| UI Concern | Typical Mechanism |
|------------|-------------------|
| Modal / drawer open | `useState` in the owning component |
| Active tab | `useState` or URL (if shareable) |
| Hover / focus | CSS pseudo-classes or local state |
| Animation phase | Local state or animation library |
| Cross-component UI flag | Context or Zustand (only if genuinely shared) |

**Syntax Rules:**

1. UI state belongs in the component that renders the UI it controls.
2. Use CSS (`:hover`, `:focus`) for purely visual states rather than JavaScript state.
3. Use URL state instead of UI state when the UI concern should be shareable (e.g., active tab in a linkable dashboard).
4. Use a global store for UI state only when multiple distant components must read or change it (e.g., sidebar open state read by layout and header).

**Constraints and Limitations:**

- Global UI state causes unrelated re-renders; use selectors if a store is unavoidable.
- Ephemeral UI state is lost on reload by design.

### Annotated Code Examples

```tsx
function Tooltip({ content, children }) {
  const [isVisible, setIsVisible] = useState(false);

  return (
    <span
      onMouseEnter={() => setIsVisible(true)}
      onMouseLeave={() => setIsVisible(false)}
      onFocus={() => setIsVisible(true)}
      onBlur={() => setIsVisible(false)}
    >
      {children}
      {isVisible && <span role="tooltip">{content}</span>}
    </span>
  );
}
```

**Expected Output:** The tooltip appears on hover and focus, and disappears on leave and blur.

**Why this output:** The visibility flag is local because only the `Tooltip` component renders the tooltip. No other component needs to know.

### Real-World Cases

- **Modals and drawers:** Open/close state local to the trigger or layout.
- **Dropdowns:** Selected item and open state local to the dropdown.
- **Tabs:** Active tab (local or URL depending on shareability).
- **Hover cards:** Visibility local to the card.

### References

- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md

---

## 4. Shared State

### Definitions

**Core Definition:** Shared state is state that multiple components at different locations in the tree need to read or modify.

**Technical Definition:** Shared client state is state owned by the client but consumed by more than one component or subtree — for example, a shopping cart, a theme, a session, or an "active workspace" selection. It is the category that most often tempts teams toward a global store, but the correct first step is to **lift state to the closest common owner**. Only when the state crosses routes or has no natural common owner does a store become justified.

**Beginner-Friendly Explanation:** Shared state is data more than one component cares about — like the cart count shown in the header and changed on the product page. The first fix is to move it up to their common parent, not to reach for a global store.

### Purposes

- To synchronise related components that must agree on a value.
- To avoid duplicating state across siblings.
- To provide a single source of truth for cross-component facts.
- To enable composition without prop drilling.

### Syntax Rules and Structure

**Escalation ladder:**

1. **Local state** → default.
2. **Lift to closest common parent** → when two siblings need the same value.
3. **Context** → when the value is read by many components at different depths and changes infrequently.
4. **Global store (Zustand, Jotai, Redux Toolkit)** → when the state crosses routes, has no natural common owner, or must be read/changed from outside React (WebSocket, tests).

**Syntax Rules:**

1. Exhaust lifting and Context before adding a store.
2. Context carries a value; it does not manage state efficiently for high-frequency changes because every consumer re-renders.
3. Use a store with selectors when the value changes often and only some readers should re-render.
4. Keep shared state minimal; most "global" state is actually server state or URL state.

**Constraints and Limitations:**

- Over-globalizing spreads unrelated re-renders and couples components that should be independent.
- Context value changes re-render all consumers even if they only care about part of the value.

### Annotated Code Examples

```tsx
// Step 1: Lift to the closest common owner
function ProductPage() {
  const [cartCount, setCartCount] = useState(0);
  return (
    <>
      <Header cartCount={cartCount} />
      <ProductCard onAdd={() => setCartCount((c) => c + 1)} />
    </>
  );
}

// Step 2: Only when the cart is read across routes and changed from many places,
// move to a store with selectors
const useCartStore = create((set) => ({
  count: 0,
  increase: () => set((s) => ({ count: s.count + 1 })),
}));

function Header() {
  const count = useCartStore((s) => s.count); // Selector: only re-renders on count change
  return <span>{count}</span>;
}
```

**Expected Output:** In the first version, lifting solves sharing with no dependency. In the second, the store is justified only because the cart is app-wide and high-frequency.

**Why this output:** Lifting is a small, local refactor; narrowing a global store is not. A store is justified by cross-route scope and frequent updates, not by anticipation.

### Real-World Cases

- **Shopping carts:** Count in header, changes on product pages and cart page.
- **Themes:** Light/dark mode read by many components, changed in settings.
- **Authentication session:** Current user read across the app.
- **Multi-step forms:** Shared draft across step components.

### References

- When you need a store (virastack) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/zustand/when-you-need-a-store.md
- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1

---

## 5. Global State

### Definitions

**Core Definition:** Global state is client state that is intentionally made available across routes and distant components, typically through a store or a top-level Context provider.

**Technical Definition:** Global client state is the small set of values that genuinely belong to the whole application — theme, authentication session, notification queue, drawer open state, feature flags, user preferences, and active workspace. It is the last rung of the escalation ladder, justified only when state crosses routes or has no natural common owner. Global stores are **not** for server data; duplicating server state into a global store and manually syncing it is a canonical anti-pattern.

**Beginner-Friendly Explanation:** Global state is the handful of things that truly belong to the whole app — like the theme or the logged-in user. It is not "everything the app knows."

### Purposes

- To provide app-wide values that change infrequently (theme, session, locale).
- To allow distant components to read and change the same client state without prop drilling.
- To support reading and writing state from outside React (WebSocket listeners, tests).
- To centralise state that must survive route changes.

### Syntax Rules and Structure

**Tool selection:**

| Scenario | Recommended tool |
|----------|-----------------|
| Small app, simple global state | Zustand or Jotai |
| Large app, complex global state | Redux Toolkit |
| Atomic, fine-grained updates | Jotai |
| Heavy server interaction | TanStack Query + light client store |
| Infrequently changing app-wide value | Context (no store needed) |

**Syntax Rules:**

1. Keep global state minimal; after server state moves to TanStack Query, what remains is usually a handful of values such as theme and sidebar state.
2. Use selectors to prevent unnecessary re-renders.
3. Never duplicate server state into a global store.
4. Use Context to share a **store instance**, not a high-frequency value; the store's selectors do the picking.
5. Choose Redux Toolkit only when the team needs strict conventions and devtools at scale.

**Constraints and Limitations:**

- Global state is overkill for most UI state; it creates coupling and re-renders that local state would not.
- Global stores can become "global state soup" when used for everything.

### Annotated Code Examples

```tsx
import { create } from 'zustand';

// Global client state: theme and sidebar visibility
const useAppStore = create((set) => ({
  theme: 'light',
  isSidebarOpen: true,
  toggleTheme: () =>
    set((s) => ({ theme: s.theme === 'light' ? 'dark' : 'light' })),
  toggleSidebar: () => set((s) => ({ isSidebarOpen: !s.isSidebarOpen })),
}));

function Header() {
  const theme = useAppStore((s) => s.theme); // Only re-renders on theme change
  const toggleTheme = useAppStore((s) => s.toggleTheme);
  return <button onClick={toggleTheme}>{theme}</button>;
}
```

**Expected Output:** The header re-renders only when `theme` changes; toggling the sidebar does not re-render the header.

**Why this output:** Selectors subscribe to specific slices, so unrelated global changes do not cause re-renders.

### Real-World Cases

- **Theming:** Light/dark mode read by many components.
- **Authentication session:** Current user and permissions.
- **Feature flags:** App-wide toggles.
- **UI shell:** Sidebar open/closed, notification queue.

### References

- When you need a store (virastack) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/zustand/when-you-need-a-store.md
- Asymmetric-al — React State Management — https://github.com/asymmetric-al/core/blob/bddbe6a3bcd6c8bc4ee082696bd5f751e1d54434/.agents/skills/react-state-management/SKILL.md
- MAO Sovereign Architect — State Management Design — https://raw.githubusercontent.com/javimontano/mao-sovereign-architect/refs/heads/main/skills/state-management-design/SKILL.md

---

## 6. Server State

### Definitions

**Core Definition:** Server state is data whose source of truth lives on a server — users, products, orders, invoices — and is cached by the client, stale from the moment it arrives.

**Technical Definition:** Server state is asynchronous, shared, cacheable, and owned remotely. The client holds a cache, not the truth. It requires specialised handling: caching, deduplication, background refetching, staleness windows, retries, cancellation, and cache invalidation after mutations. Managing it with `useState` or Redux forces you to reimplement all of these manually. TanStack Query and SWR solve them out of the box.

**Beginner-Friendly Explanation:** Server state is what the server knows. Your app keeps a temporary copy, but that copy can be out of date the moment it arrives. You need a tool that understands staleness, refetching, and invalidation.

### Purposes

- To display remote data with loading, error, and empty states.
- To cache data and avoid redundant requests.
- To keep data fresh with background revalidation.
- To synchronise the UI after mutations via invalidation.
- To deduplicate concurrent requests for the same data.

### Syntax Rules and Structure

**TanStack Query basics:**

```tsx
const { data, isPending, isError, error } = useQuery({
  queryKey: ['user', userId],
  queryFn: ({ signal }) => fetchUser(userId, { signal }),
  staleTime: 5 * 60 * 1000,
  gcTime: 30 * 60 * 1000,
});
```

**Syntax Rules:**

1. Include every parameter that affects the result in the query key.
2. Set `staleTime` based on data volatility (0 for real-time, minutes/hours for stable data).
3. Use `gcTime` to control memory retention of inactive queries.
4. Never copy server data into a global store; let the query cache be the single source of truth.
5. Invalidate relevant query keys after mutations.

**Constraints and Limitations:**

- Server state is stale by definition; do not treat cached data as authoritative.
- Query keys must be serialisable; non-serialisable values break caching.

### Annotated Code Examples

```tsx
function InvoiceList() {
  const { data, isPending, isError } = useQuery({
    queryKey: ['invoices'],
    queryFn: fetchInvoices,
    staleTime: 30_000,
  });

  if (isPending) return <InvoiceSkeleton />;
  if (isError) return <ErrorBanner />;
  if (!data?.length) return <EmptyState />;
  return <InvoiceTable invoices={data} />;
}
```

**Expected Output:** A skeleton on first load, an error on failure, an empty state if there are no invoices, and the table on success. Returning to the tab within 30 seconds serves cached data without a request; after 30 seconds, it serves cached data and revalidates in the background.

**Why this output:** `useQuery` handles caching, deduplication, and background revalidation. The three state branches cover pending, error, and empty before rendering success data.

### Real-World Cases

- **User profiles:** Fetching user data by ID.
- **Product catalogs:** Caching lists and details.
- **Dashboards:** Stats and widgets with short staleness.
- **Invoices and orders:** Data that must be revalidated after mutations.

### References

- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- MAO Sovereign Architect — State Management Design — https://raw.githubusercontent.com/javimontano/mao-sovereign-architect/refs/heads/main/skills/state-management-design/SKILL.md
- When you need a store (virastack) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/zustand/when-you-need-a-store.md

---

## 7. URL State

### Definitions

**Core Definition:** URL state is application state encoded in the address bar — route parameters, search params, filters, sort, page, and tab — so it can be linked, bookmarked, shared, and restored.

**Technical Definition:** URL state is owned by the browser address bar and is shareable and SSR-safe. It includes route parameters (`/products/42`), query strings (`?category=electronics&sort=price&page=2`), filters, sort order, pagination, the open tab, and the selected record. Anything a user should be able to link to a colleague, bookmark, or reload into belongs in the URL. In Next.js App Router, Server Components read `searchParams` and Client Components read `useSearchParams`.

**Beginner-Friendly Explanation:** URL state is the part of your app's state that belongs in the address bar — the filters, the page number, the selected item. If you would want to share the current view with a friend, it is URL state.

### Purposes

- To make filtered and paginated views shareable and bookmarkable.
- To survive reloads and make the back button work correctly.
- To enable server-side rendering of initial state without hydration mismatch.
- To make state visible and debuggable in the address bar.
- To avoid unshareable `useState` filters that lose state on reload.

### Syntax Rules and Structure

**Reading and writing search params:**

```tsx
const [searchParams, setSearchParams] = useSearchParams();
const page = Number(searchParams.get('page') ?? '1');

setSearchParams({ page: '2', sort: 'price' }, { replace: true });
```

**TanStack Router typed search:**

```tsx
const search = useSearch({ from: '/products' }); // Typed from validateSearch
```

**Syntax Rules:**

1. Store filters, sort, page, tab, and selected ID in search params or route params.
2. Parse and validate params before use (they are strings).
3. Use `replace: true` for filter changes that should not create history entries.
4. Keep resource identity in the path and view options in the query string.
5. Avoid putting sensitive data in URLs.

**Constraints and Limitations:**

- URLs are logged, shared, and stored in history — never put secrets in them.
- Long URLs are impractical; browsers rate-limit frequent URL updates.
- Complex nested state does not fit well in URLs; use a serialisation strategy.

### Annotated Code Examples

```tsx
function ProductFilters() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get('category') ?? 'all';
  const sort = searchParams.get('sort') ?? 'relevance';

  function updateFilter(key, value) {
    setSearchParams(
      (prev) => {
        const next = new URLSearchParams(prev);
        next.set(key, value);
        next.set('page', '1');
        return next;
      },
      { replace: true }
    );
  }

  return (
    <>
      <select value={category} onChange={(e) => updateFilter('category', e.target.value)}>
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
      </select>
      <select value={sort} onChange={(e) => updateFilter('sort', e.target.value)}>
        <option value="relevance">Relevance</option>
        <option value="price">Price</option>
      </select>
    </>
  );
}
```

**Expected Output:** Changing the category or sort updates the URL, resets the page to 1, and survives reload. The current view is shareable.

**Why this output:** Filters are in the URL, so a reload restores them and a shared link opens the same view. `replace: true` avoids polluting history with every filter change.

### Real-World Cases

- **E-commerce filters:** Category, price range, sort, and page.
- **Search results:** Query and page number.
- **Dashboards:** Date range, tab, and view mode.
- **Documentation:** Version and section.

### References

- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- When you need a store (virastack) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/zustand/when-you-need-a-store.md
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md

---

## 8. Form State

### Definitions

**Core Definition:** Form state is a specialised category of client state that tracks input values, dirty status, touched fields, validation errors, and submission status.

**Technical Definition:** Form state is a special case of client state with its own vocabulary: values, dirty, touched, validity, submission state, and field-level errors. It requires dirty tracking (to warn about unsaved changes), field-level and form-level validation, submission state (to disable buttons and show loading), error mapping (to associate errors with fields), and array field support (dynamic add/remove rows). Native `useState` form handling triggers excessive re-renders on every keystroke; libraries like React Hook Form use uncontrolled inputs to batch updates.

**Beginner-Friendly Explanation:** Form state is what the user is typing, whether they have changed anything, whether it is valid, and whether it is being submitted. It is different enough from other state that it deserves its own library.

### Purposes

- To track input values and changes.
- To validate fields and map errors to the correct inputs.
- To manage submission lifecycle (idle, submitting, success, error).
- To warn about unsaved changes.
- To support dynamic arrays of fields.

### Syntax Rules and Structure

**React Hook Form (recommended for performance-critical forms):**

```tsx
const { register, handleSubmit, formState: { errors, isSubmitting } } =
  useForm({ resolver: zodResolver(schema) });
```

**React 19 `useActionState` (for Server Actions):**

```tsx
const [state, formAction, isPending] = useActionState(action, initialState);
```

**Syntax Rules:**

1. Use a form library for anything beyond trivial forms; native `useState` causes per-keystroke re-renders.
2. Use uncontrolled inputs with React Hook Form to minimise re-renders.
3. Validate on blur or submit; avoid premature errors.
4. Map server validation errors (422) back to fields.
5. Use `useId` for unique input IDs and `aria-describedby` for error association.

**Constraints and Limitations:**

- Form state should not be globalised; it belongs to the form.
- Persisting form drafts requires explicit storage handling and versioning.

### Annotated Code Examples

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'At least 8 characters'),
});

function SignupForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } =
    useForm({ resolver: zodResolver(schema) });

  return (
    <form onSubmit={handleSubmit(async (data) => { await api.signup(data); })}>
      <input {...register('email')} />
      {errors.email && <span role="alert">{errors.email.message}</span>}
      <input {...register('password')} type="password" />
      {errors.password && <span role="alert">{errors.password.message}</span>}
      <button disabled={isSubmitting}>
        {isSubmitting ? 'Submitting…' : 'Sign up'}
      </button>
    </form>
  );
}
```

**Expected Output:** Typing does not re-render the form; validation runs on submit; errors appear under the correct fields; the button disables while submitting.

**Why this output:** React Hook Form uses uncontrolled inputs and refs, so values live in the DOM and only subscribed fields re-render. `zodResolver` validates the entire form on submit.

### Real-World Cases

- **Signup and login:** Email, password, and validation.
- **Checkout:** Address and payment forms with field-level errors.
- **Settings:** Profile forms with dirty tracking.
- **Surveys:** Dynamic arrays of questions.

### References

- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- MAO Sovereign Architect — State Management Design — https://raw.githubusercontent.com/javimontano/mao-sovereign-architect/refs/heads/main/skills/state-management-design/SKILL.md
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md

---

## 9. Derived State

### Definitions

**Core Definition:** Derived state is not state at all — it is a value computed from existing props or state, and storing it creates a second source of truth that will eventually drift.

**Technical Definition:** If a value can be computed from current props or state, it should be **derived during render**, not stored in state or updated in an effect. Examples include totals, filtered lists, counts, indexes, formatted values, and `isValid` flags. Storing derived state requires synchronization code, increases memory usage, and causes unnecessary re-renders. If the computation is genuinely expensive, memoize it with `useMemo` or a selector rather than storing it in state.

**Beginner-Friendly Explanation:** Derived state is anything you can calculate from what you already have — the total of a cart, the filtered list, whether a form is valid. Do not write it down; calculate it when you need it.

### Purposes

- To avoid synchronization bugs from duplicated data.
- To reduce state surface area and memory usage.
- To prevent extra renders caused by storing and updating derived values.
- To keep a single source of truth for every fact.

### Syntax Rules and Structure

**✅ Derive during render:**

```tsx
const [items, setItems] = useState([]);
const total = items.reduce((sum, i) => sum + i.price, 0); // Computed, not stored
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const fullName = `${firstName} ${lastName}`;
```

**❌ Stored derivation (anti-pattern):**

```tsx
const [total, setTotal] = useState(0);
useEffect(() => {
  setTotal(items.reduce((sum, i) => sum + i.price, 0)); // Stored and synced
}, [items]);
```

**Syntax Rules:**

1. Compute during render for simple values.
2. Use `useMemo` for expensive computations that run on every render.
3. Use selectors in stores to compute derived values on demand.
4. Never store what you can derive; if a value must exist in two shapes, one is derived.

**Constraints and Limitations:**

- `useMemo` is a performance optimisation, not a semantic guarantee.
- Over-memoizing cheap computations adds overhead without benefit.

### Annotated Code Examples

```tsx
function Cart({ items }) {
  const [items, setItems] = useState([]);

  // Derived during render — never stored
  const total = items.reduce((sum, i) => sum + i.price * i.qty, 0);
  const itemCount = items.reduce((sum, i) => sum + i.qty, 0);
  const hasItems = items.length > 0;

  return (
    <div>
      <p>{itemCount} items</p>
      <p>Total: ${total.toFixed(2)}</p>
      {!hasItems && <EmptyCart />}
    </div>
  );
}
```

**Expected Output:** `total`, `itemCount`, and `hasItems` are always correct because they are recomputed from `items` on every render. They can never disagree with the items.

**Why this output:** Derived values computed during render cannot go stale. Storing them would require synchronization and would eventually drift.

### Real-World Cases

- **Cart totals:** Summing item prices.
- **Filtered lists:** Filtering todos by status.
- **Form validity:** `!errors.email && !errors.password`.
- **Formatted display:** Dates, currency, and names.

### References

- pproenca/dot-skills — Derive, Don't Store Computed — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/same-results-less-code/references/derive-dont-store-computed.md
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md
- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1
- React Blog — You Probably Don't Need Derived State — https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html

---

## 10. Persistent State

### Definitions

**Core Definition:** Persistent state is state that survives a page reload or browser close, stored in browser storage such as `localStorage`, `sessionStorage`, cookies, or IndexedDB.

**Technical Definition:** Persistent state is owned by browser storage and must survive reload, with careful handling across the hydration boundary. Storage options differ in capacity and lifetime: `localStorage` (~5 MB per origin, permanent), `sessionStorage` (~5 MB, tab lifetime), IndexedDB (browser-managed, up to ~60% of disk, permanent), Cache API (permanent), and cookies (~4 KB per cookie, configurable). **Never read `localStorage` during render** — it does not exist on the server and will cause hydration mismatches. Persisted state should be versioned and validated on read, with a fallback when the stored shape is from an older version of the app.

**Beginner-Friendly Explanation:** Persistent state is the state that should still be there after you refresh the page — like your theme choice or a saved draft. You store it in the browser, but you have to be careful: the server cannot see it, so reading it too early breaks hydration.

### Purposes

- To preserve user preferences across sessions (theme, language, layout).
- To save drafts so users do not lose work.
- To persist authentication tokens (with security considerations).
- To support offline-first experiences with IndexedDB.

### Syntax Rules and Structure

**Custom `useLocalStorage` hook:**

```tsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

**Syntax Rules:**

1. Read storage in an effect or lazy initializer, never during render on the server.
2. Version the storage key and validate the stored shape on read.
3. Use `localStorage` for preferences and non-sensitive tokens; use `sessionStorage` for tab-scoped wizard state.
4. Use cookies (`HttpOnly`, `Secure`, `SameSite`) for authentication tokens instead of `localStorage` when possible.
5. Persist only what is necessary; do not persist server state — let the query cache rehydrate it.

**Constraints and Limitations:**

- Storage is synchronous and limited (~5 MB for `localStorage`).
- Persisted data can become stale if the schema changes; version and validate.
- Persisting sensitive data in `localStorage` is an XSS risk.

### Annotated Code Examples

```tsx
function usePersistedTheme() {
  const [theme, setTheme] = useState(() => {
    if (typeof window === 'undefined') return 'light';
    const stored = localStorage.getItem('theme:v1');
    return stored === 'dark' ? 'dark' : 'light';
  });

  useEffect(() => {
    localStorage.setItem('theme:v1', theme);
  }, [theme]);

  return [theme, setTheme];
}
```

**Expected Output:** The theme is restored from `localStorage` on mount (client-side only) and persisted whenever it changes. Server rendering uses `'light'` as the fallback, avoiding hydration mismatches.

**Why this output:** The lazy initializer checks `typeof window` to avoid server access. The key is versioned (`theme:v1`). The fallback handles missing or invalid values.

### Real-World Cases

- **Theme preference:** Light/dark mode persisted in `localStorage`.
- **Wizard progress:** `sessionStorage` for tab-scoped multi-step flow state.
- **Drafts:** Persisting form drafts so users do not lose work.
- **Authentication:** `HttpOnly` cookies for tokens, not `localStorage`.

### References

- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- sujeet.pro — Client State Management (RSC boundary) — https://raw.githubusercontent.com/sujeet-pro/sujeet.pro/refs/heads/main/content/articles/client-state-management/README.md
- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1

---

## 11. State Categories (Consolidated Reference)

### Definitions

**Core Definition:** The state categories table is the consolidated mapping of each category to its owner, lifetime, scope, and correct mechanism.

**Technical Definition:** The categories are distinguished by **owner** (server, client, URL, browser/platform, form), **lifetime** (one render, one mount, one session, across sessions), and **scope** (component, subtree, route, application). The table below consolidates the mapping from the decision flow.

### Category Mapping Table

| Category | Owner | Lifetime | Typical Scope | Correct Mechanism |
|----------|-------|----------|---------------|-------------------|
| **Server state** | Server | Stale on arrival | App-wide (cached) | TanStack Query, SWR |
| **Local / UI state** | Client | One mount | One component | `useState`, `useReducer` |
| **Shared client state** | Client | One session | Subtree / route | Lift, Context, Zustand, Jotai |
| **Global client state** | Client | One session | App-wide | Zustand, Redux Toolkit, Jotai |
| **URL state** | Browser address bar | Across reloads | Route | Router search/route params, `nuqs` |
| **Form state** | Form | One mount | Form subtree | React Hook Form, Formik |
| **Persistent state** | Browser storage | Across sessions | Client only | `localStorage`, `sessionStorage`, IndexedDB |
| **Derived state** | Computed | N/A | Any | Compute during render, `useMemo`, selectors |

### Syntax Rules

1. **Authority beats scope:** server data read by one component is still server state.
2. **Shareability beats convenience:** if a user would link or reload into it, it is URL state.
3. **Computability beats storage:** if you can compute it, do not store it.
4. **Consumers beat anticipation:** do not globalize for a second consumer that does not exist yet.
5. **Persistence needs an explicit story:** version, validate, and fall back.

### Real-World Cases

- **E-commerce:** Server (products, cart), URL (filters, sort), local (drawer), derived (total).
- **SaaS dashboard:** Server (stats), URL (date range, tab), local (widget hover), persistent (theme).
- **Wizard:** Form (draft values), URL (step if shareable), persistent (draft across sessions), derived (progress percentage).

### References

- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1
- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- MAO Sovereign Architect — State Management Design — https://raw.githubusercontent.com/javimontano/mao-sovereign-architect/refs/heads/main/skills/state-management-design/SKILL.md
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md

---

## 12. State Management Approaches

### Definitions

**Core Definition:** State management approaches are the mechanisms and libraries used to implement each state category — from React built-ins to specialised server-state and client-state libraries.

**Technical Definition:** Approaches span three families: **React built-ins** (`useState`, `useReducer`, Context), **specialised server-state libraries** (TanStack Query, SWR, Apollo Client), and **client-state libraries** (Zustand, Jotai, Redux Toolkit, Valtio). The 2026 consensus is to **match the tool to the category**, not to pick one universal library: server state goes to TanStack Query, UI state starts with `useState`, shared client state uses Zustand or Jotai, forms use React Hook Form, and URLs carry shareable state.

**Beginner-Friendly Explanation:** There is no single "best" state library. There are tools for server state, tools for client state, and tools for forms — and using the right tool for each job is the whole game.

### Tool Comparison Table

| Tool | Category | Bundle (gzip) | Learning Curve | Best For |
|------|----------|---------------|----------------|----------|
| **useState / useReducer** | Local / UI | 0 | Low | Component-local state |
| **Context** | Shared (low-frequency) | 0 | Low | Theme, session, store instance |
| **TanStack Query** | Server | ~13–16 KB | Medium | Server state with caching and revalidation |
| **SWR** | Server | ~4–5.5 KB | Low | Lightweight server state |
| **Zustand** | Shared / global client | ~1.2 KB | Low | UI state, settings, auth |
| **Jotai** | Atomic / derived client | ~3.2 KB | Medium | Fine-grained reactive atoms |
| **Redux Toolkit** | Global client | ~12 KB | High | Large teams with strict conventions |
| **React Hook Form** | Form | Small | Low | Performance-critical forms |

### Syntax Rules

1. Use `useState` for simple local state; escalate to `useReducer` for complex transitions.
2. Use Context to share an **instance** (a store), not a high-frequency value.
3. Use TanStack Query or SWR for server state; never duplicate server data into a client store.
4. Use Zustand as the default for shared client state; Jotai for fine-grained atoms; Redux Toolkit for large teams.
5. Use React Hook Form for performance-critical forms.
6. Use router search params or `nuqs` for shareable URL state.

### Real-World Cases

- **Small app, simple state:** Zustand or Jotai.
- **Large app, complex state:** Redux Toolkit.
- **Heavy server interaction:** TanStack Query + light client state.
- **Atomic/granular updates:** Jotai.

### References

- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- Asymmetric-al — React State Management — https://github.com/asymmetric-al/core/blob/bddbe6a3bcd6c8bc4ee082696bd5f751e1d54434/.agents/skills/react-state-management/SKILL.md
- PkgPulse — The State of React State Management in 2026 — https://www.pkgpulse.com

---

## 13. State Management Selection

### Definitions

**Core Definition:** State management selection is the process of choosing the right mechanism for each piece of state based on its category, scope, and lifetime — not on library preference.

**Technical Definition:** Selection follows the **bucket-first, library-second** rule: decide the category before reaching for a tool. The decision flow asks: who is the authority → should it be shareable → can it be derived → how many consumers → must it survive a reload. The 2026 consensus is to push server state to specialised libraries, keep global state minimal, and let URLs carry shareable state.

**Beginner-Friendly Explanation:** To choose a tool, first ask what kind of state you have. Server state? URL state? Local state? The answer tells you the tool.

### Selection Decision Table

| Question | Answer | Tool |
|----------|--------|------|
| Who is the authority? | Server | TanStack Query / SWR |
| Who is the authority? | Client | → next question |
| Should it be shareable/reloadable? | Yes | Router search params / `nuqs` |
| Can it be computed? | Yes | Derive during render / `useMemo` |
| How many consumers? | One component | `useState` |
| How many consumers? | One subtree | Lift / Context |
| How many consumers? | App-wide | Zustand / Jotai / Redux Toolkit |
| Must survive reload? | Yes | `localStorage` / `sessionStorage` / IndexedDB + versioning |
| Is it a form? | Yes | React Hook Form / Formik |

### Syntax Rules

1. Classify first; the tool follows.
2. Start local and move outward on evidence.
3. Never store what you can derive.
4. Treat server data as a cache, never as owned state.
5. Give persisted state a version and a fallback.

### Real-World Cases

- **New project:** Start with `useState` + TanStack Query; add Zustand only when shared client state appears.
- **Legacy Redux app:** Move server state to TanStack Query; keep Redux for truly global client state.
- **Dashboard:** TanStack Query (stats), URL (filters), local (widget UI), derived (totals).

### References

- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1
- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- Toptal — Top React State Management Libraries and How to Choose — https://www.toptal.com

---

## 14. State Management Best Practices

### Definitions

**Core Definition:** State management best practices are the guidelines for classifying, placing, and updating state to minimise bugs, re-renders, and maintenance cost.

**Technical Definition:** Best practices centre on **classification before tooling**, **server data as cache**, **URL for shareable state**, **local by default**, **derived not stored**, **one source of truth per fact**, **invalid combinations impossible**, **persisted state versioned**, and **server state never duplicated into a global store**. In React Server Components, UI and persistent state must live behind a `'use client'` boundary, and `localStorage` must never be read during render.

**Beginner-Friendly Explanation:** Good state management means putting each thing in the right place, not storing what you can calculate, and not globalizing what can stay local.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Classify before choosing** | Name owner, lifetime, and scope in one sentence |
| **Treat server data as cache** | Never own it or duplicate it into a store |
| **Put shareable state in the URL** | Filters, sort, page, tab, selected ID |
| **Start local and move outward on evidence** | Lift when a second consumer appears; globalize only cross-route |
| **Never store what you can derive** | Compute totals, counts, filtered lists, validity |
| **Keep one source of truth per fact** | Two owners will eventually disagree |
| **Make invalid combinations impossible** | Use discriminated unions instead of independent flags |
| **Version persisted state** | Validate on read and fall back |
| **Don't put platform facts in maintained state** | Online status, viewport size — read them, don't mirror them |
| **Use selectors in stores** | Prevent unnecessary re-renders |
| **Colocate state** | Keep it as close to its use as possible |
| **Separate concerns** | Server state (TanStack Query) vs client state (Zustand) |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Server state → TanStack Query
const { data: invoices } = useQuery({ queryKey: ['invoices'], queryFn: fetchInvoices });

// URL state → search params
const [searchParams] = useSearchParams();
const status = searchParams.get('status') ?? 'all';

// UI state → local
const [isFilterOpen, setFilterOpen] = useState(false);

// Derived state → computed
const filtered = invoices.filter((i) => status === 'all' || i.status === status);
const total = filtered.reduce((sum, i) => sum + i.amount, 0);
```

**❌ Bad Practices:**

```tsx
// ❌ Server data in a global store with manual sync
const useStore = create((set) => ({ invoices: [], setInvoices: (i) => set({ invoices: i }) }));

// ❌ Stored derived state
const [total, setTotal] = useState(0);

// ❌ Modal flag globalized
const useUIStore = create((set) => ({ isModalOpen: false }));

// ❌ Unshareable filters in useState
const [status, setStatus] = useState('all');
```

**Expected Output:** The good practices produce correct caching, shareable views, and no stale derived values. The bad practices cause cache-invalidation work, synchronization bugs, unnecessary global re-renders, and unshareable URLs.

**Why this output:** Server data in a store forces manual invalidation; stored totals drift; global modal flags couple unrelated components; `useState` filters cannot be shared.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **RSC applications:** UI and persistent state behind `'use client'`; `localStorage` only in effects.
- **Large teams:** Classification makes state decisions reviewable and consistent.

### References

- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1
- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- When you need a store (virastack) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/zustand/when-you-need-a-store.md
- Asymmetric-al — React State Management — https://github.com/asymmetric-al/core/blob/bddbe6a3bcd6c8bc4ee082696bd5f751e1d54434/.agents/skills/react-state-management/SKILL.md

---

## References

- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md#1
- sujeet.pro — Client State Management — https://github.com/sujeet-pro/sujeet.pro/tree/main/content/articles/client-state-management
- sujeet.pro — Client State Management (RSC boundary) — https://raw.githubusercontent.com/sujeet-pro/sujeet.pro/refs/heads/main/content/articles/client-state-management/README.md
- React State Management Reference — https://github.com/rand/cc-polymath/blob/baa2df114416570f5e0e2961143d20e3716a2fc8/skills/frontend/react-state-management/resources/REFERENCE.md
- MAO Sovereign Architect — State Management Design — https://raw.githubusercontent.com/javimontano/mao-sovereign-architect/refs/heads/main/skills/state-management-design/SKILL.md
- When you need a store (virastack) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/zustand/when-you-need-a-store.md
- Asymmetric-al — React State Management — https://github.com/asymmetric-al/core/blob/bddbe6a3bcd6c8bc4ee082696bd5f751e1d54434/.agents/skills/react-state-management/SKILL.md
- pproenca/dot-skills — Derive, Don't Store Computed — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/same-results-less-code/references/derive-dont-store-computed.md
- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state
- React Blog — You Probably Don't Need Derived State — https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html
- Toptal — Top React State Management Libraries and How to Choose — https://www.toptal.com
- PkgPulse — The State of React State Management in 2026 — https://www.pkgpulse.com