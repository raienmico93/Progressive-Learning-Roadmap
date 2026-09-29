# Deferred Rendering in React: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Deferred Rendering is React's concurrent rendering technique that allows a fast-changing value to be passed to a component as a deferred (lagging) copy, enabling React to keep high-priority updates (like user input) responsive while deferring lower-priority updates (like heavy list re-renders) to the background.

**Technical Definition:** Deferred Rendering is implemented through the `useDeferredValue` Hook, which returns a deferred version of a value. During urgent renders, the deferred value lags behind the current value. React first re-renders with the old deferred value (keeping the UI responsive), then schedules a background re-render with the new value. The background re-render is interruptible: if a new update occurs, React discards the in-progress background render and restarts. This mechanism is built on React's concurrent rendering architecture and the Fiber reconciler, which allows rendering work to be paused, resumed, or discarded.

**Beginner-Friendly Explanation:** Imagine you're typing in a search box that filters a huge list. Every keystroke updates the input, but filtering the list takes time. Without deferred rendering, the input might feel laggy because React is busy filtering. `useDeferredValue` lets you give the list a "delayed" copy of the search query—the input updates instantly, and the list catches up when React has spare time. If you keep typing, React keeps delaying the list update and only renders the final result.

### Key Characteristics

- **Adaptive Deferral:** Unlike debouncing, which uses a fixed delay, `useDeferredValue` defers only when React is busy with more important work. When the app is idle, deferred values update immediately.
- **Interruptible Background Renders:** If a new update occurs during a background render, React restarts the render with the latest value.
- **No Fixed Delay:** There is no built-in millisecond delay; deferral is based on rendering priority, not time.
- **Stale Value Preservation:** The component receiving the deferred value continues to display the previous value until the background render completes.
- **Integration with Suspense:** If a background update suspends, the user sees the old deferred value rather than a fallback.
- **Initial Render Equality:** During the initial render, the deferred value equals the current value; deferral only kicks in during updates.

### Prerequisites

- Solid understanding of React Hooks (`useState`, `useMemo`, `React.memo`).
- Familiarity with React's rendering lifecycle and concurrent rendering concepts.
- Basic understanding of the difference between urgent and non-urgent updates.
- Knowledge of `startTransition` and `useTransition` (for comparison).

### Related Programming Areas

- **Concurrent Rendering:** The underlying architecture enabling deferred rendering.
- **Transitions:** `startTransition` and `useTransition` for marking updates as non-urgent.
- **Performance Optimisation:** Memoisation, virtualisation, and rendering strategies.
- **User Experience:** Perceived performance and responsive interfaces.
- **State Management:** Coordinating local and derived state.

### Core Concepts / Features

1. The `useDeferredValue` Hook
2. High-Frequency Input Optimization
3. Stale Value Indications
4. Comparison with Debounce/Throttle

---

## Core Concept 1: The `useDeferredValue` Hook

### Definitions

**Core Definition:** `useDeferredValue` is a React Hook that returns a deferred version of a value, allowing React to defer updating a part of the UI while keeping the rest responsive.

**Technical Definition:** `useDeferredValue(value, initialValue?)` is a Hook that must be called at the top level of a component. It returns the current deferred value. During the initial render, the returned value is `initialValue` (if provided) or the same as `value`. During updates, React first re-renders with the old deferred value, then schedules a background re-render with the new value. The background re-render is interruptible and will restart if a new update to the value occurs. The Hook is integrated with `<Suspense>`: if the background update suspends, the user sees the old deferred value until the data loads.

**Beginner-Friendly Explanation:** `useDeferredValue` lets you create a "delayed copy" of a value. You give it the real value (like a search query), and it gives you back a version that updates later. You use the real value for the parts of the UI that need to be instant (like the input field) and the deferred value for the parts that can wait (like the results list).

### Purposes

- To keep high-priority updates (like typing) responsive while deferring expensive re-renders.
- To create a lagging copy of a value for use in slow-rendering components.
- To defer updates without introducing a fixed delay (unlike debouncing).
- To integrate with Suspense to show stale content instead of a fallback.
- To optimise rendering of large lists, charts, and other expensive components.
- To work with values received as props or from custom Hooks where the setter is not accessible.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');

  // Create a deferred version of the query
  const deferredQuery = useDeferredValue(query);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {/* SearchResults receives the deferred value */}
      <SearchResults query={deferredQuery} />
    </div>
  );
}
```

**Component Breakdown:**
- `useState('')`: Manages the urgent state (the input value).
- `useDeferredValue(query)`: Returns a deferred version of `query`.
- `<input value={query}>`: Uses the urgent value for instant input responsiveness.
- `<SearchResults query={deferredQuery}>`: Uses the deferred value for the expensive component.

**Syntax with `initialValue` (React 19+):**
```jsx
const deferredQuery = useDeferredValue(query, '');
```

**Component Breakdown:**
- The second argument is an `initialValue` used during the initial render.
- This is useful for server-side rendering, where the `initialValue` is always used.
- If omitted, the deferred value equals the current value during the initial render.

**Syntax Rules:**
- Call `useDeferredValue` at the top level of a component or custom Hook.
- The `value` can be any type: string, number, object, etc.
- For objects, the value should be created outside of rendering (e.g., in state or props) to avoid unnecessary background re-renders.
- `useDeferredValue` does not by itself prevent network requests; pair it with request cancellation if needed.
- When the update is inside a transition, `useDeferredValue` always returns the new value and does not spawn a deferred render.
- The background re-render does not fire Effects until it is committed to the screen.

**Constraints and Limitations:**
- `useDeferredValue` does not make rendering faster; it only delays it.
- For effective re-render skipping, the expensive component must be wrapped in `React.memo`.
- If the deferred value is used without `useMemo` for expensive computations, the component may still re-render unnecessarily.
- `useDeferredValue` is a React 18+ feature; the `initialValue` argument is React 19+.
- It does not work with text inputs for the urgent value; the input must use the live value.

### Annotated Code Examples

**Example 1: Basic Search with Deferred Value**

```jsx
import React, { useState, useDeferredValue, useMemo } from 'react';

// A component that renders a large, filtered list
const SearchResults = React.memo(function SearchResults({ query }) {
  // Generate a large list and filter it based on the query
  const items = useMemo(() => {
    return Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`)
      .filter(item => item.toLowerCase().includes(query.toLowerCase()));
  }, [query]);

  return (
    <ul>
      {items.slice(0, 50).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
});

function SearchApp() {
  const [input, setInput] = useState('');
  // Create a deferred version of the input
  const deferredInput = useDeferredValue(input);

  function handleChange(e) {
    // Urgent update: keep the input responsive
    setInput(e.target.value);
  }

  return (
    <div>
      <input
        value={input}
        onChange={handleChange}
        placeholder="Type to search..."
      />
      {/* SearchResults receives the deferred value */}
      <SearchResults query={deferredInput} />
    </div>
  );
}

export default SearchApp;
```

**Expected Output:** As the user types, the input field updates instantly with every keystroke. The search results list updates a moment later. If the user types quickly, intermediate filter operations are skipped, and only the final query's results are rendered. The input never feels laggy.

**Why This Output Occurs:** The `setInput` call is an urgent update, so React processes it immediately, keeping the input responsive. `useDeferredValue` returns a deferred version of `input` that lags behind during urgent renders. React first re-renders with the old deferred value (keeping the list visible), then schedules a background re-render with the new deferred value. If the user types again before the background render completes, React discards it and restarts with the latest value. `React.memo` prevents `SearchResults` from re-rendering unless the deferred value actually changes.

**Example 2: Deferred Value with `initialValue` for SSR**

```jsx
import React, { useState, useDeferredValue, useMemo } from 'react';

const SearchResults = React.memo(function SearchResults({ query }) {
  const items = useMemo(() => {
    return Array.from({ length: 5000 }, (_, i) => `Result ${i + 1}`)
      .filter(item => item.toLowerCase().includes(query.toLowerCase()));
  }, [query]);

  return (
    <ul>
      {items.slice(0, 30).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
});

function App() {
  const [query, setQuery] = useState('');
  // Use initialValue for SSR hydration consistency
  const deferredQuery = useDeferredValue(query, '');

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <SearchResults query={deferredQuery} />
    </div>
  );
}

export default App;
```

**Expected Output:** During the initial render (including server-side rendering), `deferredQuery` is `''` (the `initialValue`), so the results list renders all items. After hydration, as the user types, the input updates instantly, and the results list updates when React has spare time.

**Why This Output Occurs:** The `initialValue` argument ensures that the server-rendered HTML and the initial client render use the same value, preventing hydration mismatches. Without `initialValue`, the deferred value would equal `query` during the initial render, which could differ between server and client. After hydration, `useDeferredValue` behaves normally, deferring updates during urgent renders.

### Real-World Cases

- **Search autocomplete:** Keeping the input responsive while filtering large datasets.
- **Filtering and sorting:** Applying filters to large lists without blocking the UI.
- **Data visualisation:** Updating charts with new data while keeping controls responsive.
- **Live previews:** Showing a deferred preview of formatted content while the user types.
- **Dashboard widgets:** Updating slow-rendering widgets without affecting other controls.
- **Code editors:** Deferring syntax highlighting updates while the user types.

---

## Core Concept 2: High-Frequency Input Optimization

### Definitions

**Core Definition:** High-frequency input optimisation is the practice of using `useDeferredValue` to keep fast-changing inputs (search boxes, sliders, filters) responsive while deferring the expensive downstream re-renders they trigger.

**Technical Definition:** High-frequency inputs generate events at 30–60+ per second (e.g., keystrokes, slider movements). If each event triggers an expensive re-render (filtering thousands of items, recalculating a chart), the main thread becomes blocked, causing input lag. By passing the input value through `useDeferredValue`, the input itself uses the urgent value (updating immediately), while the expensive component receives the deferred value (updating only when React has capacity). This decouples the input's render update from the expensive computation, eliminating typing lag.

**Beginner-Friendly Explanation:** When you type fast, your search box sends a new value every few milliseconds. If the results list tries to update every single time, the browser gets overwhelmed and the input feels slow. `useDeferredValue` lets the input update instantly while the results list waits and only updates when the browser isn't busy. This makes typing feel smooth even when filtering huge lists.

### Purposes

- To keep search inputs, sliders, and filters responsive during heavy data processing.
- To decouple fast-changing input state from expensive downstream computations.
- To eliminate typing lag caused by synchronous filtering of large arrays.
- To improve Interaction to Next Paint (INP) and perceived performance.
- To combine with `React.memo` and `useMemo` for optimal re-render skipping.
- To work with values received as props or from custom Hooks.

### Syntax Rules and Structure

**General Syntax for Search Input Optimisation:**
```jsx
import { useState, useDeferredValue, useMemo, memo } from 'react';

const HeavyList = memo(function HeavyList({ query }) {
  const filtered = useMemo(() => {
    // Expensive filtering operation
    return data.filter(item => item.includes(query));
  }, [query]);

  return <ul>{filtered.map(item => <li key={item}>{item}</li>)}</ul>;
});

function SearchApp() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <HeavyList query={deferredQuery} />
    </div>
  );
}
```

**Component Breakdown:**
- `useState('')`: Manages the urgent input value.
- `useDeferredValue(query)`: Creates a deferred copy for the heavy component.
- `<input value={query}>`: Uses the urgent value for instant feedback.
- `<HeavyList query={deferredQuery}>`: Receives the deferred value, re-rendering only when it changes.
- `memo(HeavyList)`: Prevents re-renders unless the deferred query prop changes.
- `useMemo`: Memoises the expensive filtering computation.

**Syntax for Slider Optimisation:**
```jsx
function SliderApp() {
  const [sliderValue, setSliderValue] = useState(50);
  const deferredSliderValue = useDeferredValue(sliderValue);

  return (
    <div>
      <input
        type="range"
        min="0"
        max="100"
        value={sliderValue}
        onChange={(e) => setSliderValue(Number(e.target.value))}
      />
      <ExpensiveVisualisation value={deferredSliderValue} />
    </div>
  );
}
```

**Component Breakdown:**
- The slider uses the urgent value for smooth dragging.
- The visualisation uses the deferred value, updating only when React has spare time.
- The slider remains fluid even if the visualisation is computationally heavy.

**Syntax Rules:**
- Always pair `useDeferredValue` with `React.memo` on the expensive component.
- Use `useMemo` for expensive computations inside the expensive component.
- Pass the deferred value to the expensive component, not the live value.
- The input must use the live value to remain responsive.
- For multiple deferred values, call `useDeferredValue` separately for each.
- Avoid creating new objects during rendering and passing them to `useDeferredValue`.

**Constraints and Limitations:**
- `useDeferredValue` does not make rendering faster; it only delays it.
- If the expensive render takes 500ms, the UI will still jank when it eventually runs.
- Combine with virtualisation (e.g., `react-window`) for truly large lists.
- `useDeferredValue` does not prevent network requests; pair with request cancellation.
- The expensive component must be wrapped in `React.memo` for effective re-render skipping.
- Deferred values update immediately when the app is idle; the deferral only applies when React is busy.

### Annotated Code Examples

**Example 1: Optimised Search with Large Dataset**

```jsx
import React, { useState, useDeferredValue, useMemo, memo } from 'react';

// Generate a large dataset
const ALL_ITEMS = Array.from({ length: 50000 }, (_, i) => ({
  id: i,
  name: `Product ${i + 1}`,
  category: ['Electronics', 'Clothing', 'Books', 'Food'][i % 4],
}));

// Expensive component: filters and renders the list
const ProductList = memo(function ProductList({ query }) {
  // Memoise the expensive filter operation
  const filteredItems = useMemo(() => {
    if (!query) return ALL_ITEMS;
    return ALL_ITEMS.filter(item =>
      item.name.toLowerCase().includes(query.toLowerCase()) ||
      item.category.toLowerCase().includes(query.toLowerCase())
    );
  }, [query]);

  return (
    <ul>
      {filteredItems.slice(0, 100).map(item => (
        <li key={item.id}>
          {item.name} — {item.category}
        </li>
      ))}
    </ul>
  );
});

function OptimisedSearch() {
  const [query, setQuery] = useState('');
  // Defer the query for the expensive list
  const deferredQuery = useDeferredValue(query);

  // Detect stale state
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search products..."
        style={{ padding: '8px', width: '300px' }}
      />
      <p>{isStale ? 'Updating results...' : 'Results are current'}</p>
      {/* Dim the list when stale */}
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <ProductList query={deferredQuery} />
      </div>
    </div>
  );
}

export default OptimisedSearch;
```

**Expected Output:** As the user types, the input field updates instantly. The results list updates a moment later, and "Updating results..." appears briefly while the list is stale. The list is dimmed (opacity 0.5) during the stale period and returns to full opacity when the deferred value catches up.

**Why This Output Occurs:** The `setQuery` call is urgent, so the input updates immediately. `useDeferredValue` returns a deferred copy that lags behind during urgent renders. `ProductList` is wrapped in `memo`, so it only re-renders when `deferredQuery` changes. The `useMemo` inside `ProductList` ensures the expensive filter operation only runs when `deferredQuery` changes. The `isStale` flag (`query !== deferredQuery`) drives the loading message and the dimming effect, providing visual feedback without unmounting the list.

**Example 2: Slider with Deferred Chart Rendering**

```jsx
import React, { useState, useDeferredValue, useMemo, memo } from 'react';

// A component that renders a complex chart
const ComplexChart = memo(function ComplexChart({ value }) {
  // Simulate expensive chart computation
  const data = useMemo(() => {
    const result = [];
    for (let i = 0; i < 1000; i++) {
      result.push({
        x: i,
        y: Math.sin(i / 10) * value + Math.cos(i / 5) * (100 - value),
      });
    }
    return result;
  }, [value]);

  return (
    <div style={{ border: '1px solid #ccc', padding: '10px' }}>
      <p>Chart value: {value}</p>
      <p>Data points: {data.length}</p>
      {/* In a real app, this would render an SVG or canvas chart */}
    </div>
  );
});

function SliderApp() {
  const [value, setValue] = useState(50);
  const deferredValue = useDeferredValue(value);
  const isStale = value !== deferredValue;

  return (
    <div>
      <label>
        Value: {value}
        <input
          type="range"
          min="0"
          max="100"
          value={value}
          onChange={(e) => setValue(Number(e.target.value))}
          style={{ width: '100%' }}
        />
      </label>
      {isStale && <p>Updating chart...</p>}
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <ComplexChart value={deferredValue} />
      </div>
    </div>
  );
}

export default SliderApp;
```

**Expected Output:** Dragging the slider updates the displayed value instantly. The chart updates a moment later, with "Updating chart..." appearing briefly and the chart dimmed during the stale period. The slider remains fluid throughout.

**Why This Output Occurs:** The slider uses the urgent `value` state, so it updates instantly on every drag event. `useDeferredValue` creates a deferred copy for the chart. `ComplexChart` is wrapped in `memo` and uses `useMemo` to avoid recomputing the chart data unless `deferredValue` changes. The `isStale` flag drives the loading message and dimming, providing feedback while the chart catches up.

### Real-World Cases

- **E-commerce search:** Filtering thousands of products while keeping the search box responsive.
- **Dashboard sliders:** Adjusting parameters on a chart without lag.
- **Data tables:** Filtering and sorting large datasets with instant input feedback.
- **Map applications:** Updating map markers while keeping the search input responsive.
- **Audio/video editors:** Adjusting settings with real-time previews.
- **Financial dashboards:** Updating charts with new data while keeping controls fluid.

---

## Core Concept 3: Stale Value Indications

### Definitions

**Core Definition:** Stale value indication is the practice of detecting when a deferred value is lagging behind the current state and applying visual feedback (e.g., dimming, loading indicators) to inform the user that content is outdated.

**Technical Definition:** When `useDeferredValue` returns a value that differs from the current value, the UI is displaying stale content. This is detected by comparing the live value with the deferred value: `const isStale = value !== deferredValue`. The `isStale` boolean can be used to apply CSS styles (opacity, colour), show a loading spinner, or display a text message. This pattern is recommended by the React documentation and provides users with clear feedback that the content is being updated, without unmounting the existing UI or causing layout shifts.

**Beginner-Friendly Explanation:** When you type in a search box and the results list hasn't caught up yet, it's helpful to show the user that the list is "old" and new results are coming. You can do this by dimming the list slightly or showing a small loading indicator. React makes this easy: just compare the real value with the deferred value—if they're different, the content is stale.

### Purposes

- To inform users that displayed content is outdated and new content is loading.
- To provide subtle visual feedback without unmounting the existing UI.
- To prevent the "frozen UI" perception when deferred updates are in progress.
- To style stale content with reduced opacity, blur, or colour changes.
- To conditionally show loading spinners or messages.
- To improve perceived performance by acknowledging that an update is happening.

### Syntax Rules and Structure

**General Syntax:**
```jsx
function SearchApp() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  // Detect stale state
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {/* Apply visual indication when stale */}
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <SearchResults query={deferredQuery} />
      </div>
      {isStale && <p>Updating results...</p>}
    </div>
  );
}
```

**Component Breakdown:**
- `isStale = query !== deferredQuery`: Compares the live and deferred values.
- `style={{ opacity: isStale ? 0.5 : 1 }}`: Dims the stale content.
- `{isStale && <p>Updating results...</p>}`: Shows a loading message when stale.

**Syntax for Blur Effect:**
```jsx
<div style={{
  opacity: isStale ? 0.7 : 1,
  filter: isStale ? 'blur(1px)' : 'none',
  transition: 'opacity 0.2s, filter 0.2s',
}}>
  <SearchResults query={deferredQuery} />
</div>
```

**Component Breakdown:**
- `opacity`: Reduces opacity when stale.
- `filter: blur(1px)`: Applies a subtle blur to stale content.
- `transition`: Smoothly animates the changes.

**Syntax for Loading Spinner:**
```jsx
{isStale && <Spinner />}
<div style={{ opacity: isStale ? 0.5 : 1 }}>
  <SearchResults query={deferredQuery} />
</div>
```

**Component Breakdown:**
- `{isStale && <Spinner />}`: Conditionally renders a spinner.
- The stale list remains visible and interactive, dimmed in the background.

**Syntax Rules:**
- Compare the live value with the deferred value using strict inequality (`!==`).
- Apply visual feedback to the container of the deferred content, not the input.
- Use CSS transitions for smooth visual changes.
- Keep the stale content mounted and interactive; do not replace it with a skeleton.
- The `isStale` flag is derived during render and does not require an Effect.
- For multiple deferred values, compute `isStale` for each independently.

**Constraints and Limitations:**
- The `isStale` comparison uses `Object.is` semantics for objects; ensure objects are stable across renders.
- Stale indication does not prevent the deferred update from happening; it only provides feedback.
- Overly aggressive dimming or blurring may reduce readability; use subtle effects.
- The stale period may be very brief on fast devices; the indicator may flash.
- Stale indication does not apply to the initial render (when live and deferred values are equal).

### Annotated Code Examples

**Example 1: Dimming Stale Search Results**

```jsx
import React, { useState, useDeferredValue, useMemo, memo } from 'react';

const SearchResults = memo(function SearchResults({ query }) {
  const items = useMemo(() => {
    if (!query) return [];
    return Array.from({ length: 5000 }, (_, i) => `Result ${i + 1}`)
      .filter(item => item.toLowerCase().includes(query.toLowerCase()));
  }, [query]);

  return (
    <ul>
      {items.slice(0, 30).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
});

function StaleIndicatorDemo() {
  const [input, setInput] = useState('');
  const deferredInput = useDeferredValue(input);

  // Detect whether the displayed results are stale
  const isStale = input !== deferredInput;

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Search..."
        style={{ padding: '8px', width: '300px' }}
      />

      {/* Visual indication of stale content */}
      <div
        style={{
          opacity: isStale ? 0.5 : 1,
          transition: 'opacity 0.2s ease-in-out',
          marginTop: '10px',
        }}
      >
        <SearchResults query={deferredInput} />
      </div>

      {isStale && (
        <p style={{ color: '#888', fontStyle: 'italic' }}>
          Updating results...
        </p>
      )}
    </div>
  );
}

export default StaleIndicatorDemo;
```

**Expected Output:** As the user types, the input updates instantly. The search results list remains visible but is dimmed to 50% opacity, and "Updating results..." appears. When the deferred value catches up, the list returns to full opacity and the message disappears.

**Why This Output Occurs:** The `isStale` flag is `true` whenever `input` and `deferredInput` differ. During this period, the opacity is set to 0.5 and the loading message is shown. When React completes the background render and commits the new deferred value, `input` and `deferredInput` become equal, `isStale` becomes `false`, and the UI returns to normal. The CSS transition ensures the opacity change is smooth, not jarring.

**Example 2: Blur Effect for Stale Content**

```jsx
import React, { useState, useDeferredValue, useMemo, memo } from 'react';

const ArticleList = memo(function ArticleList({ filter }) {
  const articles = useMemo(() => {
    return Array.from({ length: 2000 }, (_, i) => ({
      id: i,
      title: `Article ${i + 1}`,
      category: ['Tech', 'Science', 'Art', 'History'][i % 4],
    })).filter(article =>
      filter === 'all' || article.category === filter
    );
  }, [filter]);

  return (
    <ul>
      {articles.slice(0, 20).map(article => (
        <li key={article.id}>
          <strong>{article.title}</strong> — {article.category}
        </li>
      ))}
    </ul>
  );
});

function FilterApp() {
  const [filter, setFilter] = useState('all');
  const deferredFilter = useDeferredValue(filter);
  const isStale = filter !== deferredFilter;

  return (
    <div>
      <div>
        {['all', 'Tech', 'Science', 'Art', 'History'].map(cat => (
          <button
            key={cat}
            onClick={() => setFilter(cat)}
            style={{
              margin: '4px',
              fontWeight: filter === cat ? 'bold' : 'normal',
            }}
          >
            {cat}
          </button>
        ))}
      </div>

      {/* Apply blur and dimming to stale content */}
      <div
        style={{
          opacity: isStale ? 0.6 : 1,
          filter: isStale ? 'blur(1px)' : 'none',
          transition: 'opacity 0.2s, filter 0.2s',
          marginTop: '10px',
        }}
      >
        <ArticleList filter={deferredFilter} />
      </div>

      {isStale && <p>Loading articles...</p>}
    </div>
  );
}

export default FilterApp;
```

**Expected Output:** Clicking a filter button updates the button styles instantly. The article list becomes dimmed and slightly blurred while "Loading articles..." appears. When the deferred filter catches up, the list returns to normal and the blur is removed.

**Why This Output Occurs:** The `isStale` flag drives both the opacity and blur styles. The `filter` state updates urgently (changing the button styles), while `deferredFilter` lags behind. The `ArticleList` is wrapped in `memo` and receives `deferredFilter`, so it only re-renders when the deferred value changes. The CSS transitions ensure smooth visual changes.

### Real-World Cases

- **Search interfaces:** Dimming results while the list updates.
- **Dashboard filters:** Applying a blur to charts while new data loads.
- **Tab switching:** Dimming the current tab content while the next tab renders.
- **Data tables:** Showing a subtle overlay on stale rows during sorting.
- **Image galleries:** Dimming thumbnails while a new filter is applied.
- **Code editors:** Blurring preview panes while syntax highlighting updates.

---

## Core Concept 4: Comparison with Debounce/Throttle

### Definitions

**Core Definition:** Debouncing and throttling are traditional JavaScript techniques for limiting the rate of function execution, while `useDeferredValue` is React's concurrent rendering mechanism for deferring lower-priority renders.

**Technical Definition:** **Debouncing** delays the execution of a function until a specified period of inactivity has elapsed (e.g., 300ms after the user stops typing). **Throttling** limits the execution of a function to at most once per specified interval (e.g., once every 200ms). Both use timers (`setTimeout`/`clearTimeout`) and introduce a fixed, time-based delay regardless of device performance. `useDeferredValue` is fundamentally different: it is not time-based. It defers updates only when React is busy with more important work. On fast devices, deferred values update immediately; on slow devices, they defer to prevent jank. `useDeferredValue` does not prevent network requests; it only deprioritises rendering. Debouncing and throttling are often used to reduce the frequency of API requests, while `useDeferredValue` is used to keep the UI responsive during expensive renders.

**Beginner-Friendly Explanation:** Debouncing is like waiting for someone to finish speaking before you respond. Throttling is like only responding once every few seconds. `useDeferredValue` is different: it's like saying "I'll respond immediately if I'm free, but if I'm busy, I'll get to it when I can." Debouncing always waits a fixed time; `useDeferredValue` adapts to how busy React is.

### Purposes

- To understand when to use concurrent deferrals versus traditional debounce/throttle.
- To choose the right tool for reducing API request frequency versus keeping the UI responsive.
- To avoid the fixed delay of debouncing when adaptive deferral is more appropriate.
- To combine `useDeferredValue` with request cancellation for optimal performance.
- To recognise that debouncing and `useDeferredValue` solve different problems.
- To use `useDeferredValue` for rendering performance and debouncing for request frequency.

### Syntax Rules and Structure

**Debounce Syntax (traditional):**
```javascript
import { useEffect, useState } from 'react';

function useDebouncedValue(value, delayMs) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedValue(value);
    }, delayMs);

    return () => clearTimeout(timeoutId);
  }, [value, delayMs]);

  return debouncedValue;
}

// Usage
function SearchForm() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebouncedValue(query, 300);

  // debouncedQuery updates 300ms after the user stops typing
  return (
    <input value={query} onChange={(e) => setQuery(e.target.value)} />
  );
}
```

**Component Breakdown:**
- `useDebouncedValue`: A custom Hook that delays updates by a fixed `delayMs`.
- `setTimeout`: Schedules the update after the delay.
- `clearTimeout` in cleanup: Cancels the timer if the value changes before the delay elapses.
- `debouncedQuery`: Updates only after the user stops typing for `delayMs` milliseconds.

**`useDeferredValue` Syntax:**
```jsx
import { useState, useDeferredValue } from 'react';

function SearchForm() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  // deferredQuery updates when React has spare cycles
  return (
    <input value={query} onChange={(e) => setQuery(e.target.value)} />
  );
}
```

**Component Breakdown:**
- `useDeferredValue(query)`: Returns a deferred version of `query`.
- No fixed delay; deferral is based on rendering priority.
- On fast devices, `deferredQuery` may update immediately.

**Comparison Table:**

| Aspect | Debounce | Throttle | `useDeferredValue` |
|--------|----------|----------|-------------------|
| **Delay type** | Fixed (e.g., 300ms) | Fixed (e.g., 200ms) | Adaptive (no fixed delay) |
| **Primary use** | Reduce API request frequency | Limit function execution rate | Keep UI responsive during heavy renders |
| **API requests** | Prevents excess requests | Limits request rate | Does not prevent requests |
| **Device adaptation** | No | No | Yes (faster on fast devices) |
| **Timer-based** | Yes (`setTimeout`) | Yes (`setTimeout`) | No (concurrent rendering) |
| **Interruptible** | Yes (clearTimeout) | Yes | Yes (React discards work) |
| **React integration** | Custom Hook | Custom Hook | Built-in Hook |
| **Best for** | Search API calls | Scroll/resize handlers | Expensive re-renders |

**Syntax Rules:**
- Use debouncing when the goal is to reduce the frequency of API requests (e.g., search suggestions).
- Use throttling when the goal is to limit the rate of function execution (e.g., scroll handlers).
- Use `useDeferredValue` when the goal is to keep the UI responsive during expensive renders.
- `useDeferredValue` does not prevent network requests; pair it with request cancellation (e.g., `AbortController`).
- Debouncing introduces a perceptible delay; `useDeferredValue` does not add a fixed delay.
- On fast devices, `useDeferredValue` updates immediately; debouncing always waits.

**Constraints and Limitations:**
- `useDeferredValue` does not reduce the number of renders; it only deprioritises them.
- Debouncing can be combined with `useDeferredValue` for both request reduction and render responsiveness.
- Throttling guarantees a maximum execution rate; `useDeferredValue` does not.
- `useDeferredValue` is a React-specific Hook; debouncing and throttling are framework-agnostic.
- Debouncing requires a custom Hook or library (e.g., Lodash); `useDeferredValue` is built into React.

### Annotated Code Examples

**Example 1: Debounce vs `useDeferredValue` for Search**

```jsx
import React, { useState, useEffect, useDeferredValue, useMemo, memo } from 'react';

// Custom debounce Hook
function useDebouncedValue(value, delay) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);

  return debounced;
}

// Expensive results component
const Results = memo(function Results({ query, label }) {
  const items = useMemo(() => {
    return Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`)
      .filter(item => item.toLowerCase().includes(query.toLowerCase()));
  }, [query]);

  return (
    <div>
      <h4>{label}</h4>
      <ul>
        {items.slice(0, 10).map(item => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  );
});

function ComparisonDemo() {
  const [query, setQuery] = useState('');

  // Debounce approach
  const debouncedQuery = useDebouncedValue(query, 300);

  // Deferred approach
  const deferredQuery = useDeferredValue(query);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Type to compare..."
        style={{ padding: '8px', width: '300px' }}
      />

      {/* Debounced results: update 300ms after typing stops */}
      <Results query={debouncedQuery} label="Debounced (300ms delay)" />

      {/* Deferred results: update when React has spare time */}
      <Results query={deferredQuery} label="Deferred (adaptive)" />
    </div>
  );
}

export default ComparisonDemo;
```

**Expected Output:** As the user types, the input updates instantly. The debounced results update 300ms after the user stops typing. The deferred results update when React has spare time—on a fast device, this may be almost immediate; on a slow device, it may be delayed. The deferred results may update more frequently than the debounced results.

**Why This Output Occurs:** The `useDebouncedValue` Hook uses `setTimeout` to delay the update by a fixed 300ms, clearing the timer on each keystroke. The `useDeferredValue` Hook defers the update based on React's rendering priority—there is no fixed delay. On a fast device, React may complete the background render quickly, so the deferred value updates almost immediately. On a slow device, the deferred value lags behind to prevent jank.

**Example 2: Combining `useDeferredValue` with Debounce for API Requests**

```jsx
import React, { useState, useEffect, useDeferredValue, useMemo, memo } from 'react';

function useDebouncedValue(value, delay) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);

  return debounced;
}

const SearchResults = memo(function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }

    const controller = new AbortController();
    setLoading(true);

    fetch(`https://api.example.com/search?q=${query}`, {
      signal: controller.signal,
    })
      .then(res => res.json())
      .then(data => {
        setResults(data);
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== 'AbortError') setLoading(false);
      });

    return () => controller.abort();
  }, [query]);

  if (loading) return <p>Loading...</p>;

  return (
    <ul>
      {results.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});

function HybridSearch() {
  const [query, setQuery] = useState('');

  // Debounce for API request reduction
  const debouncedQuery = useDebouncedValue(query, 300);

  // Defer for render responsiveness
  const deferredQuery = useDeferredValue(debouncedQuery);

  const isStale = debouncedQuery !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {isStale && <p>Updating...</p>}
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <SearchResults query={deferredQuery} />
      </div>
    </div>
  );
}

export default HybridSearch;
```

**Expected Output:** The input updates instantly. API requests are debounced (fired 300ms after typing stops), reducing network traffic. The deferred value ensures the results list renders responsively even if the API response triggers an expensive render.

**Why This Output Occurs:** The `useDebouncedValue` Hook reduces the frequency of API requests by delaying the query passed to `fetch`. The `useDeferredValue` Hook further defers the rendering of the results, ensuring that even if the API response triggers a heavy render, the input remains responsive. This hybrid approach combines the strengths of both techniques.

### Real-World Cases

- **Search with API:** Debounce to reduce API calls, `useDeferredValue` to keep rendering responsive.
- **Autocomplete:** Debounce for request frequency, `useDeferredValue` for rendering large suggestion lists.
- **Scroll handlers:** Throttle for event frequency, `useDeferredValue` for expensive layout recalculations.
- **Window resize:** Throttle for resize events, `useDeferredValue` for re-rendering responsive layouts.
- **Form validation:** Debounce for validation API calls, `useDeferredValue` for rendering error messages.
- **Data filtering:** `useDeferredValue` for rendering, no debounce needed if data is local.

---

## References

- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- React Official Documentation – `startTransition`: https://react.dev/reference/react/startTransition
- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Working Group – `useDeferredValue`: https://github.com/reactwg/react-18/discussions/129
- React Working Group – Concurrent React: https://github.com/reactwg/react-18/discussions/64
- React 18 Release Notes – `useDeferredValue`: https://react.dev/blog/2022/03/29/react-v18#usedeferredvalue
- React 19 Release Notes – `initialValue` for `useDeferredValue`: https://react.dev/blog/2024/12/05/react-19
- Steve Kinney – `useDeferredValue` Patterns: https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns
- Steve Kinney – `useTransition` and `useDeferredValue`: https://stevekinney.com/courses/react-performance/use-transition
- React Academy – `useDeferredValue` for Input Debouncing: https://www.coddykit.com/courses/react/usedeferredvalue-for-input-debouncing-3698190
- Stack Overflow – Debounce vs `useDeferredValue`: https://stackoverflow.com/questions/78523250/debounce-vs-usedeferredvalue
- LogRocket – A Guide to React 18's `useDeferredValue`: https://blog.logrocket.com/react-18-usedeferredvalue/
- Syncfusion – React 19 `useDeferredValue` Hook: https://www.syncfusion.com/blogs/post/react-19-usedeferredvalue-hook
- DEV Community – `useTransition` vs `useDeferredValue`: https://dev.to/use-transition-vs-usedeferredvalue
- DEV Community – React 19 `useTransition` & `useDeferredValue`: When to Use Which: https://dev.to/react-19-usetransition-usedeferredvalue-when-to-use-which
- web.dev – Interaction to Next Paint (INP): https://web.dev/articles/inp
- MDN Web Docs – `Object.is`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- MDN Web Docs – `AbortController`: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – `setTimeout`: https://developer.mozilla.org/en-US/docs/Web/API/setTimeout