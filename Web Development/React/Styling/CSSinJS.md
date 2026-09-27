# CSS-in-JS & Next-Gen Styling Engines: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** CSS-in-JS is a styling paradigm where CSS is authored in JavaScript, either evaluated at runtime (runtime CSS-in-JS) or extracted at build time into static CSS files (zero-runtime CSS-in-JS), with the latter being the current direction of the React ecosystem.

**Technical Definition:** CSS-in-JS encompasses two architectural categories. **Runtime CSS-in-JS** (styled-components, Emotion) serialises style objects into CSS rules at render time, injecting `<style>` tags into the DOM via React context and a style registry. **Build-time / zero-runtime CSS-in-JS** (Vanilla Extract, Linaria, Panda CSS, StyleX) evaluates styles during the bundling process, emitting static `.css` files with no styling logic shipped to the client. The runtime approach offers full dynamic styling (styles depend on any JavaScript value) but incurs per-render serialisation cost, breaks under React Concurrent Rendering, and is fundamentally incompatible with React Server Components (RSC) because RSC has no DOM or React context. The build-time approach eliminates runtime overhead, enables RSC compatibility, and produces statically analysable CSS at the cost of restricting styles to compile-time-evaluable expressions.

**Beginner-Friendly Explanation:** Runtime CSS-in-JS is like writing a note for every guest at a party as they arrive—it's personalised and dynamic, but the host is busy writing instead of enjoying the party. Zero-runtime CSS-in-JS is like writing all the notes in advance and handing them out at the door—no work during the party, but you can't personalise for someone who arrives late. The React ecosystem has shifted decisively toward the second approach because React Server Components removed the "party" (the DOM and React context) where runtime CSS-in-JS did its work.

### Key Characteristics

- **Two Architectures:** Runtime (serialise at render) vs. build-time (extract at build). Runtime is declining; build-time is the growth path.
- **RSC Incompatibility:** Runtime CSS-in-JS cannot run in React Server Components because RSC has no DOM to inject styles into and no React context for theme providers. The only fix is `'use client'`, which defeats the purpose of RSC.
- **Concurrent Rendering Hazard:** Runtime style insertion interferes with React 18+ concurrent rendering: React pauses to insert a style tag, the browser parses it, React resumes, and the browser re-parses. This is "unsolvable" at the runtime level.
- **styled-components in Maintenance Mode:** As of March 2025, styled-components is in maintenance mode; its creator explicitly does not recommend it for new projects.
- **Zero-Runtime Dominance:** Vanilla Extract, Linaria, Panda CSS, and StyleX emit static CSS at build time, eliminate runtime overhead, and are RSC-compatible.
- **TypeScript as Preprocessor:** Vanilla Extract uses TypeScript as its preprocessor, providing type-safe classes, variables, and themes.

### Prerequisites

- Solid understanding of CSS selectors, specificity, and the cascade.
- Familiarity with React function components, Hooks, and the `className` prop.
- Working knowledge of a build tool (Vite, Webpack, Next.js) and how it processes CSS.
- Basic understanding of React Server Components and the `'use client'` directive.
- Awareness of React 18's concurrent rendering model.

### Related Programming Areas

- **CSS Architecture:** Cascade layers, CSS Modules, and native CSS features.
- **Build Tooling:** Babel plugins, bundler integrations, and static extraction.
- **React Server Components:** Styling in RSC environments and the client/server boundary.
- **Design Systems:** Typed theme contracts, design tokens, and variant APIs.
- **Performance:** Render overhead, bundle size, and hydration complexity.

### Core Concepts / Features

1. Runtime CSS-in-JS (Styled-components and Emotion)
2. Performance Tradeoffs: Runtime Serialisation, Concurrent Rendering, and Hydration
3. Build-Time / Zero-Runtime CSS-in-JS (Vanilla Extract and Linaria)
4. Server-Driven Architecture Integration: CSS-in-JS and React Server Components

---

## Core Concept 1: Runtime CSS-in-JS — Architecture, Usage, and Deep Nesting with Styled-components and Emotion

### Definitions

**Core Definition:** Runtime CSS-in-JS is a styling approach where CSS rules are generated during the render pass, serialised into `<style>` tags, and injected into the DOM at runtime, with styled-components and Emotion being the two dominant libraries.

**Technical Definition:** Runtime CSS-in-JS libraries compute styles from component props and theme values at render time, generate unique class names, and inject the resulting CSS into the document via a style registry. Styled-components uses tagged template literals (`` styled.button`...` ``) and a `ServerStyleSheet` for SSR. Emotion offers both a `styled` API (`@emotion/styled`) and a `css` prop (`@emotion/react`), with a smaller bundle size (~7 KB gzipped vs. ~12 KB for styled-components). Both use the Stylis preprocessor for nesting, `&` parent references, and media queries. Both rely on React context for theme propagation, which is the root cause of their RSC incompatibility. Emotion is "slightly faster" than styled-components and is the safer long-term bet among the two, but both have flat or declining downloads since 2023 as the ecosystem pivots to build-time solutions.

**Beginner-Friendly Explanation:** Runtime CSS-in-JS is like having a personal chef who cooks each dish exactly to your specifications, but only after you sit down. It is personalised and flexible, but the kitchen (the browser main thread) is busy during the meal. Styled-components and Emotion are the two most popular "personal chefs." They let you write CSS directly in your JavaScript, using props to change styles dynamically. But React Server Components removed the kitchen—there is no browser main thread on the server—so these libraries cannot work there.

### Purposes

- To co-locate styles with components, eliminating the import/class-name disconnect.
- To enable dynamic styling based on props, theme, and JavaScript values.
- To provide automatic critical CSS extraction during SSR.
- To eliminate global class name collisions through automatic scoping.
- To support deep nesting and `&` parent references via the Stylis preprocessor.

### Syntax Rules and Structure

**Styled-components — Basic Usage:**
```jsx
import styled from 'styled-components';

const Button = styled.button`
  background: ${props => props.$primary ? '#007bff' : '#6c757d'};
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-weight: 600;
  cursor: pointer;

  /* Deep nesting with & parent reference */
  &:hover {
    background: ${props => props.$primary ? '#0069d9' : '#5a6268'};
  }

  /* Nested element */
  & > span.icon {
    margin-right: 8px;
  }

  /* Media query */
  @media (min-width: 768px) {
    padding: 12px 24px;
    font-size: 18px;
  }
`;

function App() {
  return (
    <>
      <Button $primary>Primary</Button>
      <Button>Secondary</Button>
    </>
  );
}
```

**Component Breakdown:**
- `` styled.button`...` ``: Creates a styled component from a template literal.
- `${props => ...}`: Interpolates a function that receives props for dynamic values.
- `&:hover`: The `&` refers to the generated class name; nesting is handled by Stylis.
- `& > span.icon`: Deep nesting with child selectors.
- `$primary`: Transient prop (prefixed with `$`) that is not passed to the DOM.

**Emotion — `styled` and `css` APIs:**
```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';
import styled from '@emotion/styled';

// The `css` prop (requires jsxImportSource pragma)
const buttonStyle = css`
  background: #007bff;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  &:hover { background: #0069d9; }
`;

function Button({ children }) {
  return <button css={buttonStyle}>{children}</button>;
}

// The `styled` API
const Card = styled.div`
  background: ${props => props.theme.surface};
  border-radius: 8px;
  padding: 16px;

  /* Deep nesting */
  & .card-title {
    font-size: 1.25rem;
    color: ${props => props.theme.text};
  }

  & .card-body > p {
    line-height: 1.5;
  }
`;
```

**Component Breakdown:**
- `/** @jsxImportSource @emotion/react */`: Enables the `css` prop without a Babel plugin.
- `css={buttonStyle}`: Applies a serialised style object to the element.
- `styled.div`: Creates a styled component from a tag.
- `props.theme.surface`: Accesses the theme provided by `ThemeProvider`.

**Emotion — Theming:**
```jsx
import { ThemeProvider } from '@emotion/react';

const theme = {
  surface: '#ffffff',
  text: '#1a1a1a',
  primary: '#007bff',
};

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Card>Content</Card>
    </ThemeProvider>
  );
}
```

**Syntax Rules:**
- Use tagged template literals for `styled` components; use the `css` prop for one-off styles.
- Prefix transient props with `$` to prevent them from being passed to the DOM.
- Use `&` for parent references and nesting; Stylis supports Sass-like nesting.
- Wrap the app in `<ThemeProvider>` to provide theme values to all styled components.
- For SSR, use `ServerStyleSheet` (styled-components) or `@emotion/server` to extract critical CSS.
- In Next.js App Router, wrap the app in a `'use client'` registry component (see Concept 4).

**Constraints and Limitations:**
- Runtime serialisation adds 15–30 ms of render overhead per component.
- Bundle size: ~12 KB (styled-components) and ~7 KB (Emotion) gzipped.
- Both libraries rely on React context, which is unavailable in Server Components.
- Style insertion order is non-deterministic, leading to unpredictable override priority and `!important` proliferation.
- Multiple versions of the same library in the dependency tree can cause syntax incompatibilities.
- React DevTools becomes cluttered with wrapper components for style injection.
- styled-components entered maintenance mode in March 2025; its creator does not recommend it for new projects.

### Annotated Code Example: Deep Nesting with Dynamic Props

```jsx
import styled from 'styled-components';

const Card = styled.div`
  background: ${({ $dark, theme }) => $dark ? theme.darkSurface : theme.surface};
  border: 1px solid ${({ theme }) => theme.border};
  border-radius: 12px;
  padding: 20px;
  transition: box-shadow 0.2s ease;

  /* Deep nesting: hover state on the card */
  &:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }

  /* Nested header with its own hover */
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 12px;

    &:hover .card-title {
      color: ${({ theme }) => theme.primary};
    }
  }

  /* Nested title */
  .card-title {
    font-size: 1.25rem;
    font-weight: 600;
    color: ${({ $dark, theme }) => $dark ? theme.darkText : theme.text};
    transition: color 0.15s ease;
  }

  /* Nested body with a child selector */
  .card-body > p {
    line-height: 1.6;
    color: ${({ theme }) => theme.textMuted};
    margin: 0;
  }

  /* Responsive: adjust padding on larger screens */
  @media (min-width: 768px) {
    padding: 24px;
  }
`;

function App() {
  return (
    <Card $dark>
      <div className="card-header">
        <h2 className="card-title">Revenue</h2>
        <span>⋯</span>
      </div>
      <div className="card-body">
        <p>Monthly recurring revenue is up 12%.</p>
      </div>
    </Card>
  );
}
```

**Expected Output:** A card with a dark background (because `$dark` is true), a header with a title that changes colour on hover, and a body with muted text. Padding increases at the `768px` breakpoint.

**Why This Output Occurs:** The `$dark` transient prop controls the background and title colour via interpolation functions. The `&:hover` rule applies to the generated card class. The `.card-header:hover .card-title` rule demonstrates deep nesting with a descendant selector. Stylis compiles all of this into flat CSS rules with the generated class name replacing `&`. The media query adjusts padding at the viewport level (not container level).

### Real-World Cases

- **Legacy React applications:** styled-components and Emotion remain in millions of existing components that predate RSC.
- **MUI:** Material UI uses Emotion as its default styling engine.
- **SPAs without RSC:** Applications that are purely client-rendered can still use runtime CSS-in-JS without RSC concerns.
- **Rapid prototyping:** The dynamic styling capabilities of runtime CSS-in-JS make it convenient for prototypes and experiments.

### References

- Emotion – Introduction: https://emotion.sh/docs/introduction
- styled-components – Documentation: https://styled-components.com/docs
- PkgPulse – Emotion vs styled-components in 2026: https://www.pkgpulse.com/blog/emotion-vs-styled-components-2026
- Feature-Sliced Design – Why I Chose Emotion for My CSS Architecture: https://feature-sliced.design/ja/blog/emotion-css-architecture

---

## Core Concept 2: Performance Tradeoffs — Runtime Serialisation, Concurrent Rendering, and Hydration

### Definitions

**Core Definition:** The performance tradeoffs of runtime CSS-in-JS arise from serialising style rules at render time, injecting them into the DOM, and re-parsing them during hydration—costs that interfere with React 18+ concurrent rendering and degrade SSR performance.

**Technical Definition:** Runtime CSS-in-JS imposes three distinct performance costs. **Serialisation:** on every render, the library evaluates template literals or style objects, hashes the result, checks the cache, and injects a new `<style>` tag if the hash is new. This adds 15–30 ms per component render. **Concurrent rendering interference:** React 18's scheduler pauses rendering to insert a style tag, the browser parses the style, React resumes rendering, and the browser re-parses the same style—an "unsolvable" performance problem at the runtime level. **Hydration complexity:** during SSR, the server must collect all styles into a registry, inject them before hydration, and the client must rehydrate the cache without duplicating rules; miscoordination causes FOUC (flash of unstyled content). GitHub's migration from CSS-in-JS to CSS Modules reduced SSR time by 55% and component initialisation time by 25%. Linear reported performance regression after upgrading to React 18 with styled-components.

**Beginner-Friendly Explanation:** Runtime CSS-in-JS is like a chef who cooks each dish after you order. The food is fresh and personalised, but you wait for every dish. When many people are ordering at once (concurrent rendering), the kitchen gets confused—the chef starts cooking, gets interrupted to plate another dish, comes back, and re-cooks the same thing. Zero-runtime CSS-in-JS is like a buffet: everything is prepared in advance, so you just pick what you want. No waiting, no confusion.

### Purposes

- To understand the cost structure of runtime CSS-in-JS before adopting it.
- To quantify the performance impact of style serialisation on render time.
- To recognise why runtime CSS-in-JS is incompatible with concurrent rendering.
- To understand the hydration complexity that runtime CSS-in-JS adds to SSR.
- To evaluate migration options (CSS Modules, zero-runtime CSS-in-JS, Tailwind) based on measured performance.

### Syntax Rules and Structure

**Performance Cost Breakdown:**

| Cost | Runtime CSS-in-JS | Zero-Runtime CSS-in-JS |
|---|---|---|
| **Per-render overhead** | 15–30 ms per component | 0 ms (styles are in static CSS files) |
| **Bundle size (styling runtime)** | 7–12 KB gzipped | 0 KB (only class name strings) |
| **Concurrent rendering** | Interferes with scheduler (pauses, re-parses) | No interference |
| **Hydration** | Requires style registry + `useServerInsertedHTML` | No hydration complexity |
| **SSR time** | Higher (style collection + injection) | Lower (CSS files served separately) |
| **Caching** | Cache keys depend on runtime hashing | Static CSS files cached by browser |

**GitHub's Migration Results:**
GitHub migrated Primer from CSS-in-JS to CSS Modules:
- SSR time reduced by 55%
- Component initialisation time reduced by 25%

**Linear's Migration Results:**
Linear migrated from styled-components to StyleX after:
- Performance regression after upgrading to React 18
- styled-components entered maintenance mode
- The "final straw" was the performance regression from concurrent rendering

**Syntax Rules (Diagnosing Runtime CSS-in-JS Overhead):**
- Use React DevTools Profiler to measure render time with and without styled-components.
- Look for `<style>` tags being injected during render in the Elements panel.
- Measure SSR time before and after removing runtime CSS-in-JS.
- Use Lighthouse to measure Total Blocking Time (TBT) and Largest Contentful Paint (LCP).
- Check bundle size with `source-map-explorer` or `webpack-bundle-analyzer`.

**Constraints and Limitations:**
- The serialisation cost is paid on every render, not just the first.
- Concurrent rendering interference is an architectural limitation, not a bug that can be fixed in a library update.
- Hydration complexity increases with the number of components using runtime CSS-in-JS.
- The runtime overhead is more pronounced on mid-range mobile devices with slower connections.
- Even with caching, the cache lookup and hash computation still add overhead per render.

### Annotated Code Example: Measuring the Cost

```jsx
// A component with runtime CSS-in-JS
import styled from 'styled-components';

const HeavyButton = styled.button`
  background: ${props => props.$variant === 'primary' ? '#007bff' : '#6c757d'};
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.2s;

  &:hover {
    background: ${props => props.$variant === 'primary' ? '#0069d9' : '#5a6268'};
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

// Measuring render time with React Profiler
import { Profiler } from 'react';

function App() {
  return (
    <Profiler id="HeavyButton" onRender={(id, phase, actualDuration) => {
      console.log(`${id} (${phase}): ${actualDuration.toFixed(2)}ms`);
    }}>
      <HeavyButton $variant="primary">Click</HeavyButton>
    </Profiler>
  );
}
```

**Expected Output:** The Profiler logs the render duration for each render. With runtime CSS-in-JS, the duration includes the serialisation and injection cost. The same component with a static CSS class would log a lower duration.

**Why This Output Occurs:** The `styled.button` template literal is evaluated on every render. If `$variant` changes, a new style hash is computed, and a new `<style>` tag is injected. The Profiler captures the total time, including this overhead. In concurrent rendering, the injection may cause React to pause and resume, further increasing the duration.

### Real-World Cases

- **GitHub Primer:** Migrated to CSS Modules, reducing SSR time by 55%.
- **Linear:** Migrated to StyleX after React 18 performance regression.
- **Atlassian:** Migrating Atlaskit from Emotion to Compiled (build-time CSS-in-JS) for performance and RSC compatibility.
- **Sanity:** Forked styled-components to add `useInsertionEffect`, achieving ~40% faster renders.
- **MUI:** Still uses Emotion but is exploring zero-runtime alternatives.

### References

- Lumevalley – GitHub SSR Migration: https://www.lumevalley.com
- Linear – Styling Linear for the future with StyleX: https://linear.app/blog/styling-linear-for-the-future-with-stylex
- Atlassian – RFC-73 Migrating our components to Compiled CSS-in-JS: https://community.developer.atlassian.com/t/rfc-73-migrating-our-components-to-compiled-css-in-js/85953
- Sanity – styled-components maintenance mode: A 40% faster fork: https://www.sanity.io/blog/styled-components-maintenance-mode
- PkgPulse – Emotion vs styled-components in 2026: https://www.pkgpulse.com/blog/emotion-vs-styled-components-2026

---

## Core Concept 3: Build-Time / Zero-Runtime CSS-in-JS — Static Extraction Models (Vanilla Extract, Linaria)

### Definitions

**Core Definition:** Build-time / zero-runtime CSS-in-JS is a styling approach where CSS is authored in JavaScript or TypeScript but extracted into static `.css` files during the build process, eliminating runtime style serialisation and injection entirely.

**Technical Definition:** Zero-runtime CSS-in-JS libraries evaluate style definitions at build time and emit static CSS files. **Linaria** uses a Babel plugin to statically analyse `css` and `styled` tagged templates, evaluate JavaScript expressions that are compile-time constants, extract CSS rules to real files, and generate unique class names. It relies on WyW (`@wyw-in-js/*`) for module evaluation. **Vanilla Extract** uses TypeScript as its preprocessor: styles are written in `.css.ts` files using functions like `style()` and `createTheme()`, and the build process generates static CSS files while the JavaScript output contains only the exported class name strings. It provides full type safety, locally scoped class names, CSS variables, and typed theme contracts. Both libraries eliminate runtime overhead, reduce bundle size (no styling runtime), and are compatible with React Server Components because they ship no styling logic to the client.

**Beginner-Friendly Explanation:** Zero-runtime CSS-in-JS is like writing a recipe in JavaScript but having the actual cooking done at the factory before the food reaches the restaurant. The chef (runtime) never touches the recipe—they just serve the pre-made dish. Linaria lets you write styled-components-style code, but the CSS is extracted at build time. Vanilla Extract makes you write styles in separate `.css.ts` files with TypeScript, giving you autocomplete and type checking for every CSS property. Both produce static CSS files that the browser caches, and neither ships any styling JavaScript to the client.

### Purposes

- To eliminate runtime style serialisation and injection overhead entirely.
- To reduce JavaScript bundle size by removing the styling runtime.
- To achieve compatibility with React Server Components by shipping no client-side styling logic.
- To provide type-safe styles (Vanilla Extract) with autocomplete and compile-time errors.
- To generate static CSS files that benefit from browser caching.
- To support design tokens, themes, and variants as typed contracts.

### Syntax Rules and Structure

**Vanilla Extract — Basic Usage:**
```typescript
// styles.css.ts
import { style, createTheme } from '@vanilla-extract/css';

// Typed theme contract
export const [themeClass, vars] = createTheme({
  color: {
    bg: '#ffffff',
    text: '#1a1a1a',
    primary: '#007bff',
  },
  space: {
    sm: '4px',
    md: '8px',
    lg: '16px',
  },
});

// Locally scoped class with type-safe properties
export const card = style({
  background: vars.color.bg,
  color: vars.color.text,
  padding: vars.space.lg,
  borderRadius: '8px',
  boxShadow: '0 2px 4px rgba(0, 0, 0, 0.1)',
  selectors: {
    '&:hover': {
      boxShadow: '0 4px 12px rgba(0, 0, 0, 0.15)',
    },
  },
  '@media': {
    '(min-width: 768px)': {
      padding: vars.space.lg,
    },
  },
});

// Variant API
export const button = style({
  padding: `${vars.space.sm} ${vars.space.lg}`,
  borderRadius: '4px',
  border: 'none',
  cursor: 'pointer',
  variants: {
    color: {
      primary: { background: vars.color.primary, color: 'white' },
      secondary: { background: '#6c757d', color: 'white' },
    },
  },
  defaultVariants: {
    color: 'primary',
  },
});
```

```tsx
// App.tsx
import { themeClass, card, button } from './styles.css';

function App() {
  return (
    <div className={themeClass}>
      <div className={card}>
        <h2>Revenue</h2>
        <p>Monthly recurring revenue is up 12%.</p>
        <button className={button({ color: 'primary' })}>View Details</button>
      </div>
    </div>
  );
}
```

**Component Breakdown:**
- `createTheme({...})`: Creates a typed theme contract and a class name that applies the theme's CSS variables.
- `style({...})`: Creates a locally scoped class; all properties are type-checked.
- `vars.color.primary`: Typed reference to a theme variable.
- `selectors: { '&:hover': {...} }`: Nesting and pseudo-classes.
- `variants: { color: {...} }`: A typed variant API that generates class name combinations.
- The `.css.ts` file is compiled to static CSS; the JS output contains only class name strings.

**Linaria — Styled-components Syntax with Zero Runtime:**
```jsx
import { styled } from '@linaria/react';
import { css } from '@linaria/core';

// styled API — same syntax as styled-components
const Button = styled.button`
  background: ${props => props.$primary ? '#007bff' : '#6c757d'};
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-weight: 600;
  cursor: pointer;

  &:hover {
    background: ${props => props.$primary ? '#0069d9' : '#5a6268'};
  }

  @media (min-width: 768px) {
    padding: 12px 24px;
    font-size: 18px;
  }
`;

// css tag — returns a className string
const cardStyle = css`
  background: white;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
`;

function App() {
  return (
    <div className={cardStyle}>
      <Button $primary>Click</Button>
      <Button>Secondary</Button>
    </div>
  );
}
```

**Component Breakdown:**
- `styled.button`: Same API as styled-components; Linaria's Babel plugin extracts the CSS.
- `props.$primary`: Interpolation functions are evaluated at build time; only compile-time constants are allowed.
- `css` tag: Returns a class name string; the CSS is extracted to a static file.
- The bundle contains no styling runtime; the JS output is just class name strings.

**Linaria — Configuration (Vite):**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { linaria } from '@linaria/vite';

export default defineConfig({
  plugins: [react(), linaria()],
});
```

**Component Breakdown:**
- `linaria()`: The Vite plugin extracts CSS at build time.
- No runtime imports are needed for the extracted styles.

**Syntax Rules (Vanilla Extract):**
- Write styles in `.css.ts` files.
- Use `style()` for class names; use `createTheme()` for typed themes.
- Access theme variables via the `vars` object.
- Use `selectors` for pseudo-classes and nesting; use `@media` for media queries.
- Use `variants` for typed variant APIs; call the style function with variant values.
- Apply the theme class at the root of the app.
- Do not use closures or runtime-dependent values in `style()` calls.

**Syntax Rules (Linaria):**
- Use `styled` and `css` tags from `@linaria/react` and `@linaria/core`.
- Interpolation functions must be compile-time evaluable; runtime values are not supported.
- Configure the bundler plugin (webpack, Rollup, Vite) to extract CSS.
- Critical CSS extraction is automatic with code splitting.
- Use `@linaria/babel-preset` for Babel-based setups.

**Constraints and Limitations:**
- **Vanilla Extract:** Styles must be in separate `.css.ts` files; no colocation with components. No `styled.div` API. Nested selectors are supported but via the `selectors` object, not template literals.
- **Linaria:** Interpolation functions cannot depend on runtime props; only build-time constants. Runtime dynamic styling requires a different approach (e.g., CSS variables).
- Both libraries require a build-time step; they do not work in environments without a bundler.
- Both restrict dynamic styling to what can be determined at compile time.
- Vanilla Extract's TypeScript preprocessing requires the TypeScript compiler as part of the build.

### Annotated Code Example: Vanilla Extract Theme + Variants

```typescript
// theme.css.ts
import { createTheme } from '@vanilla-extract/css';

export const [lightTheme, vars] = createTheme({
  color: {
    bg: '#ffffff',
    surface: '#f8f9fa',
    text: '#212529',
    textMuted: '#6c757d',
    primary: '#007bff',
    border: '#dee2e6',
  },
  radius: {
    sm: '4px',
    md: '8px',
    lg: '12px',
  },
  space: {
    sm: '4px',
    md: '8px',
    lg: '16px',
    xl: '24px',
  },
});

export const darkTheme = createTheme(vars, {
  color: {
    bg: '#121212',
    surface: '#1e1e1e',
    text: '#e9ecef',
    textMuted: '#adb5bd',
    primary: '#4dabf7',
    border: '#343a40',
  },
  radius: { sm: '4px', md: '8px', lg: '12px' },
  space: { sm: '4px', md: '8px', lg: '16px', xl: '24px' },
});
```

```typescript
// card.css.ts
import { style } from '@vanilla-extract/css';
import { vars } from './theme.css';

export const card = style({
  background: vars.color.surface,
  color: vars.color.text,
  border: `1px solid ${vars.color.border}`,
  borderRadius: vars.radius.md,
  padding: vars.space.lg,
  selectors: {
    '&:hover': {
      boxShadow: '0 4px 12px rgba(0, 0, 0, 0.1)',
    },
  },
});

export const title = style({
  color: vars.color.primary,
  margin: `0 0 ${vars.space.md}`,
  fontSize: '1.25rem',
});
```

```tsx
// App.tsx
import { lightTheme, darkTheme } from './theme.css';
import { card, title } from './card.css';

function App({ isDark }: { isDark: boolean }) {
  return (
    <div className={isDark ? darkTheme : lightTheme}>
      <div className={card}>
        <h2 className={title}>Revenue</h2>
        <p>Monthly recurring revenue is up 12%.</p>
      </div>
    </div>
  );
}
```

**Expected Output:** A card that adapts to light or dark theme based on the `isDark` prop. The theme is a typed contract: `vars.color.primary` is type-checked, and TypeScript provides autocomplete for all theme variables. The CSS is extracted to a static file at build time; the JS bundle contains only class name strings.

**Why This Output Occurs:** `createTheme()` generates CSS variables scoped to the theme class. `style()` references those variables via the `vars` object. When `isDark` is true, the `darkTheme` class is applied, which re-points the CSS variables. The card's styles reference `var(--color-surface)` etc., so they update automatically. No runtime style computation is involved.

### Real-World Cases

- **Panda CSS:** Used by MUI and Chakra UI for zero-runtime styling.
- **StyleX:** Built by Meta, used by Linear and Facebook; explicitly RSC-compatible.
- **Compiled CSS-in-JS:** Atlassian's build-time CSS-in-JS, used across Atlaskit components.
- **next-yak:** A Rust-based zero-runtime library that provides styled-components syntax with full RSC compatibility.
- **Vanilla Extract:** Popular in TypeScript-first design systems and component libraries.

### References

- Vanilla Extract – Zero-runtime Stylesheets in TypeScript: https://vanilla-extract.style/
- Vanilla Extract – GitHub: https://github.com/vanilla-extract-css/vanilla-extract
- CSS-Tricks – CSS in TypeScript with vanilla-extract: https://css-tricks.com/css-in-typescript-with-vanilla-extract/
- Linaria – Zero-runtime CSS in JS: https://linaria.dev/
- Linaria – DeepWiki – Zero-Runtime Architecture: https://deepwiki.com/callstack/linaria/4-zero-runtime-architecture
- Atlassian – RFC-73 Migrating to Compiled CSS-in-JS: https://community.developer.atlassian.com/t/rfc-73-migrating-our-components-to-compiled-css-in-js/85953
- next-yak – GitHub: https://github.com/DigitecGalaxus/next-yak
- DEV Community – The state of zero-runtime CSS-in-JS, mid-2026: https://dev.to

---

## Core Concept 4: Server-Driven Architecture Integration — CSS-in-JS and React Server Components

### Definitions

**Core Definition:** React Server Components (RSC) execute on the server and produce HTML directly, with no DOM and no React context; this architecture is fundamentally incompatible with runtime CSS-in-JS libraries that depend on both, forcing a choice between `'use client'` boundaries and zero-runtime alternatives.

**Technical Definition:** React Server Components run on the server (or at the edge) and stream HTML to the client. They cannot use hooks, context, or browser APIs. Runtime CSS-in-JS libraries (styled-components, Emotion) inject styles via `useContext` and DOM manipulation—neither of which exists in a Server Component environment. Importing a styled component into an RSC without `'use client'` silently renders an unstyled element: the component renders, but the styles are never applied. The only fix is marking the component as a Client Component with `'use client'`, which moves it to the client bundle and forfeits all RSC benefits for that component and its children. Next.js documentation explicitly states: "CSS-in-JS libraries which require runtime JavaScript are not currently supported in Server Components". Zero-runtime libraries (Vanilla Extract, Panda CSS, StyleX, next-yak) are RSC-compatible because they ship no styling logic to the client—only class name strings and static CSS files.

**Beginner-Friendly Explanation:** React Server Components are like a restaurant kitchen that prepares dishes on a different floor. Runtime CSS-in-JS is a chef who insists on cooking at the table—but there is no table on the kitchen floor. The chef cannot work there. You can send the chef to the table (mark the component `'use client'`), but then the chef is on the customer's floor, not the kitchen's—you lose the benefit of the remote kitchen. Zero-runtime CSS-in-JS is a chef who prepares everything in the kitchen and sends it up already plated. No chef needed at the table.

### Purposes

- To understand why runtime CSS-in-JS cannot work in React Server Components.
- To evaluate the `'use client'` boundary as a workaround and its costs.
- To identify zero-runtime libraries that are RSC-compatible.
- To configure runtime CSS-in-JS libraries in Next.js App Router when `'use client'` is acceptable.
- To choose the right styling approach for an RSC-first architecture.

### Syntax Rules and Structure

**The RSC Incompatibility (Runtime CSS-in-JS):**
```jsx
// ❌ This BREAKS silently in a React Server Component
import styled from 'styled-components';

const Button = styled.button`
  background: blue;
  color: white;
`;

export default async function Page() {
  return <Button>Click me</Button>;
  // ← Button renders, but styles are NEVER applied (unstyled button)
}
```

**The `'use client'` Workaround:**
```jsx
// ✅ Works, but sacrifices RSC
'use client';
import styled from 'styled-components';

const Button = styled.button`
  background: blue;
  color: white;
`;

export function Button({ children }) {
  return <Button>{children}</Button>;
}
```

**Component Breakdown:**
- `'use client'`: Marks the entire module (and its imports) as client-side.
- The component is now part of the client bundle; it cannot be rendered on the server.
- The `'use client'` boundary propagates upward: a Client Component forces its parent to be a Client Component too.
- You end up with most of your app as Client Components—exactly what RSC was designed to avoid.

**Next.js App Router Configuration (styled-components):**
```javascript
// next.config.js
module.exports = {
  compiler: {
    styledComponents: true,
  },
};
```

```tsx
// lib/registry.tsx
'use client';
import React, { useState } from 'react';
import { useServerInsertedHTML } from 'next/navigation';
import { ServerStyleSheet, StyleSheetManager } from 'styled-components';

export default function StyledComponentsRegistry({ children }) {
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet());

  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement();
    styledComponentsStyleSheet.instance.clearTag();
    return <>{styles}</>;
  });

  if (typeof window !== 'undefined') return <>{children}</>;

  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  );
}
```

```tsx
// app/layout.tsx
import StyledComponentsRegistry from './lib/registry';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <StyledComponentsRegistry>{children}</StyledComponentsRegistry>
      </body>
    </html>
  );
}
```

**Component Breakdown:**
- `useServerInsertedHTML`: Injects the collected styles into the server-rendered HTML.
- `ServerStyleSheet`: Collects styles during SSR.
- `StyleSheetManager`: Provides the sheet to styled-components during rendering.
- This setup allows styled-components to work in Client Components, not Server Components.

**Zero-Runtime RSC-Compatible Libraries:**

| Library | Syntax | RSC Compatible | Type Safety |
|---|---|---|---|
| **Vanilla Extract** | `.css.ts` files | ✅ | ✅ Full |
| **Panda CSS** | Style objects / JSX | ✅ | ✅ Full |
| **StyleX** | `stylex.create()` | ✅ | ✅ Full |
| **next-yak** | styled-components syntax | ✅ | ✅ Full |
| **Compiled** | styled-components-like | ✅ | ✅ Full |
| **Linaria** | styled / css tags | ✅ (static extraction) | Partial |

**Syntax Rules:**
- Runtime CSS-in-JS libraries must be used only in Client Components (`'use client'`).
- Wrap the app in a registry component (`StyledComponentsRegistry`, `EmotionRegistry`) in the root layout.
- Enable the library's compiler integration in `next.config.js`.
- For RSC compatibility, choose a zero-runtime library (Vanilla Extract, Panda CSS, StyleX, next-yak).
- Understand that `'use client'` propagates upward: a Client Component forces its parent to be a Client Component.
- Do not import runtime CSS-in-JS into Server Components; it will silently fail.

**Constraints and Limitations:**
- Runtime CSS-in-JS cannot run in Server Components—this is an architectural limitation, not a bug.
- The `'use client'` workaround defeats the purpose of RSC by moving components to the client bundle.
- Hydration complexity increases: the style registry must coordinate between server and client without duplicating rules.
- FOUC (flash of unstyled content) can occur if styles are not injected before hydration.
- Zero-runtime libraries are the supported path for RSC-first architectures.
- styled-components' creator stated in March 2025 that he no longer runs styled-components in production and does not recommend it for new projects.

### Annotated Code Example: RSC-Compatible Styling with Vanilla Extract

```typescript
// styles.css.ts
import { style, createTheme } from '@vanilla-extract/css';

export const [themeClass, vars] = createTheme({
  color: {
    bg: '#ffffff',
    text: '#1a1a1a',
    primary: '#007bff',
  },
  space: { md: '16px', lg: '24px' },
});

export const card = style({
  background: vars.color.bg,
  color: vars.color.text,
  padding: vars.space.lg,
  borderRadius: '8px',
  border: `1px solid ${vars.color.border}`,
});

export const button = style({
  background: vars.color.primary,
  color: 'white',
  padding: `${vars.space.md} ${vars.space.lg}`,
  borderRadius: '4px',
  border: 'none',
  cursor: 'pointer',
});
```

```tsx
// app/page.tsx (Server Component — no 'use client')
import { themeClass, card, button } from './styles.css';

export default async function Page() {
  const data = await fetchDashboardData(); // Server-side fetch

  return (
    <div className={themeClass}>
      <div className={card}>
        <h2>Revenue</h2>
        <p>{data.revenue}</p>
      </div>
      <button className={button}>View Details</button>
    </div>
  );
}
```

**Expected Output:** The page renders on the server with all styles applied. The HTML contains the static CSS classes from Vanilla Extract, and the CSS file is served separately. No styling JavaScript is shipped to the client. The Server Component works because Vanilla Extract's `style()` and `createTheme()` are evaluated at build time—the server only imports class name strings.

**Why This Output Occurs:** Vanilla Extract compiles `.css.ts` files at build time, emitting static CSS and a JavaScript module that exports class name strings. The Server Component imports these strings and applies them as `className`. There is no runtime style computation, no DOM injection, and no React context. The Server Component can render entirely on the server, and the client receives fully styled HTML.

### Real-World Cases

- **Next.js App Router:** The default styling path for RSC-first applications.
- **Panda CSS:** Used by MUI and Chakra UI for RSC-compatible styling.
- **StyleX:** Meta's zero-runtime library, used in Facebook and Linear.
- **next-yak:** Provides styled-components syntax with full RSC compatibility.
- **Atlassian Compiled:** Migrated all Atlaskit components from Emotion to Compiled for RSC and performance.

### References

- Next.js – CSS-in-JS: https://nextjs.org/docs/app/guides/css-in-js
- React – `'use client'` Directive: https://react.dev/reference/react/use-client
- saschb2b/skills – CSS-in-JS and styling status: https://github.com/saschb2b/skills
- PkgPulse – Emotion vs styled-components in 2026: https://www.pkgpulse.com/blog/emotion-vs-styled-components-2026
- Sanity – styled-components maintenance mode: https://www.sanity.io/blog/styled-components-maintenance-mode
- next-yak – GitHub: https://github.com/DigitecGalaxus/next-yak
- Panda CSS: https://panda-css.com/

---

## Comparison and Decision Guidance

| Factor | Runtime CSS-in-JS | Zero-Runtime CSS-in-JS | CSS Modules | Tailwind |
|---|---|---|---|---|
| **RSC compatible** | ❌ (requires `'use client'`) | ✅ | ✅ | ✅ |
| **Runtime overhead** | 15–30 ms per component | 0 ms | 0 ms | 0 ms |
| **Bundle size (styling)** | 7–12 KB gzipped | 0 KB | 0 KB | 0 KB runtime |
| **Dynamic styling** | Full (props, theme, JS) | Compile-time only | Limited (CSS variables) | Limited (variants) |
| **Type safety** | Partial (props) | ✅ Full (Vanilla Extract) | Partial (with tooling) | Partial (with plugins) |
| **Deep nesting** | ✅ (`&` with Stylis) | ✅ (Vanilla Extract `selectors`) | ✅ (native CSS) | Limited |
| **Colocation** | ✅ (styles in component) | ❌ (separate `.css.ts` files) | Partial (separate files) | ✅ (classes in JSX) |
| **Concurrent rendering** | ❌ Interferes | ✅ | ✅ | ✅ |
| **Hydration complexity** | High (style registry) | None | None | None |
| **Maintenance status** | styled-components: maintenance mode | Active (Vanilla Extract, Panda, StyleX) | Active | Active |

**Decision Guidance:**
- **New projects in 2026+:** Choose zero-runtime CSS-in-JS (Vanilla Extract, Panda CSS, StyleX), CSS Modules, or Tailwind. Do not start with runtime CSS-in-JS.
- **RSC-first applications:** Runtime CSS-in-JS is not an option. Use zero-runtime libraries or CSS Modules.
- **Legacy runtime CSS-in-JS:** Migrate incrementally to CSS Modules or zero-runtime. Start with static components.
- **TypeScript-heavy design systems:** Vanilla Extract provides the best type safety and typed theme contracts.
- **styled-components familiarity:** Use next-yak or Linaria for the same syntax with zero runtime.
- **Maximum dynamic styling:** If you must have runtime dynamic styles and can accept `'use client'`, Emotion is the safer long-term runtime choice.
- **Performance-critical applications:** Zero-runtime is the only option that eliminates style-related render overhead.

---

## References

- Emotion – Introduction: https://emotion.sh/docs/introduction
- styled-components – Documentation: https://styled-components.com/docs
- PkgPulse – Emotion vs styled-components in 2026: https://www.pkgpulse.com/blog/emotion-vs-styled-components-2026
- Feature-Sliced Design – Why I Chose Emotion for My CSS Architecture: https://feature-sliced.design/ja/blog/emotion-css-architecture
- Vanilla Extract – Zero-runtime Stylesheets in TypeScript: https://vanilla-extract.style/
- Vanilla Extract – GitHub: https://github.com/vanilla-extract-css/vanilla-extract
- CSS-Tricks – CSS in TypeScript with vanilla-extract: https://css-tricks.com/css-in-typescript-with-vanilla-extract/
- Linaria – Zero-runtime CSS in JS: https://linaria.dev/
- Linaria – DeepWiki – Zero-Runtime Architecture: https://deepwiki.com/callstack/linaria/4-zero-runtime-architecture
- Next.js – CSS-in-JS: https://nextjs.org/docs/app/guides/css-in-js
- React – `'use client'` Directive: https://react.dev/reference/react/use-client
- saschb2b/skills – CSS-in-JS and styling status: https://github.com/saschb2b/skills
- Sanity – styled-components maintenance mode: A 40% faster fork: https://www.sanity.io/blog/styled-components-maintenance-mode
- Atlassian – RFC-73 Migrating our components to Compiled CSS-in-JS: https://community.developer.atlassian.com/t/rfc-73-migrating-our-components-to-compiled-css-in-js/85953
- Linear – Styling Linear for the future with StyleX: https://linear.app/blog/styling-linear-for-the-future-with-stylex
- next-yak – GitHub: https://github.com/DigitecGalaxus/next-yak
- Panda CSS: https://panda-css.com/
- Lumevalley – GitHub SSR Migration: https://www.lumevalley.com
- DEV Community – The state of zero-runtime CSS-in-JS, mid-2026: https://dev.to