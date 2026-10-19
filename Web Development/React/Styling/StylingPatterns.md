# React Styling Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React styling patterns are reusable architectural solutions for applying, organising, and maintaining visual design in React applications — covering theming, responsive design, design tokens, composition, dynamic styling, and motion.

**Technical Definition:** Styling patterns in React encompass the structural approaches for connecting CSS to components, managing design decisions as code, and adapting presentation to context (theme, viewport, interaction state). The central architectural tension is **where styles live** (colocated vs. external), **when they are computed** (build time vs. runtime), and **how design decisions are shared** (hardcoded values vs. tokens). Modern React styling patterns favour zero-runtime solutions (CSS Modules, Tailwind, compile-time CSS-in-JS) for server-rendered surfaces, CSS custom properties for theming and dynamic values, and `cn`/`clsx` utilities for conditional class composition.

**Beginner-Friendly Explanation:** Styling patterns are the "recipes" for making your app look good in a consistent, maintainable way. They answer questions like: Where should I put my styles? How do I make a dark mode? How do I make my layout work on phones and desktops? How do I change a style based on data? Instead of solving these problems ad hoc every time, you use patterns that other developers have already figured out.

### Key Characteristics

- **Token-driven:** Design decisions (colors, spacing, typography) are defined once as tokens and referenced everywhere.
- **Zero-runtime bias:** Build-time styling (CSS Modules, Tailwind, compile-time CSS-in-JS) is preferred for RSC and performance-sensitive surfaces.
- **CSS variables for dynamism:** Truly dynamic values (themes, user-controlled properties) use CSS custom properties rather than inline style recomputation.
- **Composition over configuration:** Conditional and variant styles are composed via `cn`/`clsx`, not string concatenation.
- **Mobile-first responsiveness:** Base styles target mobile; breakpoint prefixes progressively enhance for larger screens.
- **Accessibility-aware motion:** Animations respect `prefers-reduced-motion` and use GPU-friendly properties.

### Prerequisites

- React function components, props, and state.
- CSS fundamentals (selectors, the cascade, custom properties).
- Understanding of the project's styling approach (CSS Modules, Tailwind, or CSS-in-JS).
- Familiarity with the browser's rendering and paint pipeline.

### Related Programming Areas

- **Design systems** — the organisational layer above styling patterns.
- **CSS architecture** — cascade layers, specificity, naming conventions.
- **Component libraries** — pre-built implementations of these patterns.
- **Accessibility** — reduced motion, contrast, focus states.

---

## 1. Theming

### Definitions

**Core Definition:** Theming is the practice of defining a set of visual design decisions (colors, typography, spacing, shadows) that can be swapped at runtime to change the entire application's appearance.

**Technical Definition:** Theming in React is implemented by distributing theme state through React Context and applying design tokens as CSS custom properties on a root element (typically `<html>` or `<body>`). A `ThemeProvider` component reads an initial theme (often from `localStorage`), sets a `data-theme` attribute or class on the document root, and updates CSS variables via the cascade. Because CSS variables respond to attribute or class changes without React re-rendering, theme switching is instantaneous and does not propagate re-renders through the component tree.

**Beginner-Friendly Explanation:** Theming means having a "light theme" and a "dark theme" (or a "brand A" and "brand B") and being able to switch between them. You define all your colors and styles in one place, and changing a single attribute on the page updates everything at once.

### Purposes

- To centralise all visual design decisions in one place.
- To enable runtime theme switching (light/dark, brand variants).
- To avoid hardcoding color and spacing values across components.
- To synchronise theme state across the component tree without prop drilling.
- To persist user theme preferences across sessions.

### Syntax Rules and Structure

**General Syntax (ThemeProvider + CSS Variables):**

```jsx
const ThemeContext = createContext(undefined);

function ThemeProvider({ children }) {
  const [mode, setMode] = useState(() => localStorage.getItem('theme') || 'light');

  useEffect(() => {
    localStorage.setItem('theme', mode);
    document.documentElement.setAttribute('data-theme', mode);
  }, [mode]);

  return (
    <ThemeContext.Provider value={{ mode, toggleTheme: () => setMode(m => m === 'light' ? 'dark' : 'light') }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

```css
:root { --bg: #fff; --text: #333; }
[data-theme="dark"] { --bg: #181924; --text: #fff; }
body { background: var(--bg); color: var(--text); }
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ThemeContext` | React Context for theme state. |
| `ThemeProvider` | Manages state, persists to `localStorage`, sets `data-theme` on `<html>`. |
| `data-theme` attribute | Gates CSS variable overrides. |
| CSS variables | Design tokens consumed by component styles. |

**Syntax Rules:**

1. Store theme tokens as CSS custom properties on `:root` and theme-specific selectors.
2. Set a `data-theme` attribute or `dark` class on `document.documentElement`.
3. Use a `useTheme` custom hook with a runtime check to consume the context.
4. Persist the user's choice to `localStorage` and read it synchronously on initial mount to avoid flash.
5. Prefer CSS variables over passing theme objects as props to every component.

**Constraints and Limitations:**

- Runtime CSS-in-JS theming (e.g., styled-components `ThemeProvider`) requires a client boundary and is incompatible with RSC.
- Theme flash (FOWT) occurs if the initial theme is applied after React hydrates; use an inline script in `<head>` to set the class before paint.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Context-Based Theme Provider

```jsx
import { createContext, useContext, useState, useEffect } from 'react';

const ThemeContext = createContext(undefined);

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}

export function ThemeProvider({ children }) {
  const [mode, setMode] = useState(
    () => localStorage.getItem('app-theme') || 'light'
  );

  useEffect(() => {
    localStorage.setItem('app-theme', mode);
    document.documentElement.setAttribute('data-theme', mode);
  }, [mode]);

  return (
    <ThemeContext.Provider
      value={{ mode, toggleTheme: () => setMode(m => m === 'light' ? 'dark' : 'light') }}
    >
      {children}
    </ThemeContext.Provider>
  );
}
```

**Expected Output:** A theme context that persists the user's choice and toggles the `data-theme` attribute on `<html>`.

**Why this output:** The provider reads the initial theme from `localStorage`, writes changes back, and sets the `data-theme` attribute. Components consuming `useTheme()` receive the current mode and a toggle function without prop drilling.

#### Example 2: CSS Variable Theme Tokens

```css
/* themes.css */
:root {
  --color-bg: #ffffff;
  --color-text: #333333;
  --color-primary: #321fdb;
}

[data-theme="dark"] {
  --color-bg: #181924;
  --color-text: #ffffff;
  --color-primary: #4638c2;
}
```

```jsx
function Card({ children }) {
  return (
    <div style={{ background: 'var(--color-bg)', color: 'var(--color-text)' }}>
      {children}
    </div>
  );
}
```

**Expected Output:** A card whose background and text colors change instantly when the `data-theme` attribute changes.

**Why this output:** The CSS variables are resolved by the browser based on the `data-theme` attribute. No React re-render is required for the color change.

### Real-World Cases

- **SaaS applications:** Light/dark mode with user preference persistence.
- **Multi-brand platforms:** Swapping brand colors by changing theme tokens.
- **Enterprise dashboards:** Centralised theming across dozens of components.

### References

- CoreUI — How to create a theme provider in React — https://coreui.io/answers/how-to-create-a-theme-provider-in-react/
- CoreUI — How to build a design system in React — https://coreui.io/answers/how-to-build-a-design-system-in-react/
- GitHub — MUI theming patterns — https://github.com/agents-inc/skills/blob/main/src/skills/web-ui-mui/SKILL.md

---

## 2. Dark Mode

### Definitions

**Core Definition:** Dark mode is a theme variant that uses dark backgrounds and light text to reduce eye strain and respect user preferences.

**Technical Definition:** Dark mode can be implemented through two primary strategies: **media-query-based** (`@media (prefers-color-scheme: dark)`) which follows the operating system setting, and **class-based** (`.dark` on `<html>`) which allows explicit user control. The class-based strategy is preferred for applications where users work for extended periods because it lets the user's saved preference override the system setting. To avoid flash-of-wrong-theme (FOWT), an inline script in `<head>` must set the class synchronously before the browser paints, reading from `localStorage` and resolving against `prefers-color-scheme`.

**Beginner-Friendly Explanation:** Dark mode is a version of your app with a dark background and light text. You can make it follow the system setting, or you can let the user toggle it with a button. The tricky part is making sure the correct mode appears immediately, without a flash of the wrong theme.

### Purposes

- To respect the user's operating system dark mode preference.
- To provide explicit user control over the visual theme.
- To reduce eye strain in low-light environments.
- To avoid flash-of-wrong-theme (FOWT) on initial page load.

### Syntax Rules and Structure

**Media-Query Strategy:**

```css
@media (prefers-color-scheme: dark) {
  :root { --bg: #181924; --text: #fff; }
}
```

**Class-Based Strategy (Recommended for User Control):**

```html
<!-- Inline script in <head> BEFORE React bundle -->
<script>
  (function() {
    const saved = localStorage.getItem('theme');
    const systemDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    const theme = saved || (systemDark ? 'dark' : 'light');
    document.documentElement.classList.toggle('dark', theme === 'dark');
  })();
</script>
```

```css
:root { --bg: #fff; --text: #333; }
.dark { --bg: #181924; --text: #fff; }
```

**Syntax Rules:**

1. Use the class-based strategy when users need explicit control over the theme.
2. Inject an inline script in `<head>` before React loads to set the class synchronously and prevent FOWT.
3. Store the user's explicit choice in `localStorage`; resolve to the system preference only when no explicit choice exists.
4. Use the `dark` class on `<html>` so that Tailwind's `dark:` variants and CSS variable overrides both work.
5. For Tailwind v4, configure `@custom-variant dark (&:where(.dark, .dark *));`.

**Constraints and Limitations:**

- A media-query-only approach cannot honour a user's explicit choice to use light mode on a dark system (or vice versa).
- Without the inline script, the first paint will use the wrong theme, causing FOWT.

### Annotated Code Examples

```jsx
// ThemeProvider component (class-based)
function ThemeProvider({ children }) {
  const [mode, setMode] = useState(() => {
    if (typeof window === 'undefined') return 'light';
    return localStorage.getItem('theme') ||
      (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
  });

  useEffect(() => {
    const root = document.documentElement;
    root.classList.toggle('dark', mode === 'dark');
    localStorage.setItem('theme', mode);
  }, [mode]);

  return (
    <ThemeContext.Provider value={{ mode, setMode }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

**Expected Output:** The `dark` class is toggled on `<html>`, and the user's preference is persisted. The initial render (after hydration) matches the synchronously applied class from the inline script.

**Why this output:** The inline script sets the class before paint, so the first paint is correct. The provider then syncs React state with the persisted preference and applies future changes.

### Real-World Cases

- **Productivity tools:** Users working all day need explicit control over light/dark mode.
- **Content platforms:** Respecting the reader's system preference by default.
- **Design systems:** Providing a `dark` variant for every component.

### References

- Architecture Decision Records — Class-based dark mode toggle — https://raw.githubusercontent.com/justinchuby/flightdeck/refs/heads/main/packages/docs/reference/architecture-decisions.md
- CoreUI — How to implement dark mode in React — https://coreui.io/answers/how-to-implement-dark-mode-in-react/
- ASOasis — React Dark Mode Theme Toggle: A Modern, Accessible Tutorial — https://asoasis.tech

---

## 3. Responsive Design

### Definitions

**Core Definition:** Responsive design is the practice of building layouts that adapt to different screen sizes and device capabilities using flexible grids, fluid images, and media queries (or container queries).

**Technical Definition:** In React, responsive design is typically implemented through CSS media queries (via CSS Modules, Tailwind breakpoint variants, or CSS-in-JS media query helpers) and, increasingly, container queries that respond to the size of a parent container rather than the viewport. Tailwind's mobile-first breakpoint system applies unprefixed utilities at all sizes and prefixed utilities (`sm:`, `md:`, `lg:`, etc.) from that breakpoint upward. For components that need fundamentally different markup on mobile versus desktop, a `useMediaQuery` hook or a `useIsMobile` hook can conditionally render different structures.

**Beginner-Friendly Explanation:** Responsive design means your app looks good on phones, tablets, and desktops. You start with a layout that works on small screens, then use breakpoints to adjust it for bigger screens. Sometimes you need completely different components for mobile and desktop, and that is where hooks come in.

### Purposes

- To provide a usable experience across all device sizes.
- To avoid horizontal scrolling and broken layouts on small screens.
- To optimise touch interactions for mobile users.
- To progressively enhance the layout as screen space increases.

### Syntax Rules and Structure

**Tailwind Mobile-First Breakpoints:**

| Breakpoint | Min-width | Target |
|------------|-----------|--------|
| (default) | 0px | Mobile phones |
| `sm:` | 640px | Large phones (landscape) |
| `md:` | 768px | Tablets |
| `lg:` | 1024px | Small laptops |
| `xl:` | 1280px | Desktops |
| `2xl:` | 1536px | Large monitors |

**Syntax Rules:**

1. Start with mobile base styles; add complexity with breakpoint prefixes.
2. Use `flex flex-col md:flex-row` rather than hardcoding `window.innerWidth` checks.
3. Prefer CSS-only solutions (hidden/shown with breakpoints) over JavaScript conditional rendering when possible.
4. Use container queries for components that appear in different layout contexts (sidebar vs. main content).
5. Ensure touch targets are at least 44×44px and hover states have touch equivalents (`active:` states).

### Annotated Code Examples

```jsx
function ResponsiveCard() {
  return (
    <div className="flex flex-col gap-4 p-4 md:flex-row md:items-center md:justify-between md:p-6 lg:p-8">
      <h2 className="text-lg font-bold md:text-xl lg:text-2xl">Title</h2>
      <button className="rounded bg-blue-600 px-4 py-2 text-white active:bg-blue-800">
        Action
      </button>
    </div>
  );
}
```

**Expected Output:** A card that stacks vertically on mobile and switches to a horizontal layout from `md` upward. Padding and font size increase on larger screens. The button has an `active` state for touch feedback.

**Why this output:** The base classes define the mobile layout. The `md:` variants switch to a row layout and increase padding and font size. The `active:` variant provides touch feedback.

### Real-World Cases

- **Dashboards:** Sidebars that collapse into drawers on mobile.
- **Data tables:** Horizontal scrolling on mobile, full tables on desktop.
- **Forms:** Single-column on mobile, multi-column on desktop.

### References

- ADR-013: Responsive Mobile-First Design — https://raw.githubusercontent.com/Sourdough-start/sourdough/refs/heads/master/docs/adr/013-responsive-mobile-first-design.md
- CoreUI — Breakpoints — https://coreui.io
- GitHub — React Responsive Patterns — https://github.com/yonatangross/orchestkit

---

## 4. Mobile-First Design

### Definitions

**Core Definition:** Mobile-first design is a strategy that starts with the smallest screen as the base and progressively enhances the layout for larger screens.

**Technical Definition:** Mobile-first design in CSS means writing base styles for mobile viewports without any media query, then adding `min-width` media queries (or Tailwind breakpoint prefixes) to adjust for larger screens. This is the inverse of desktop-first (where base styles target desktop and `max-width` queries override for mobile). Mobile-first produces cleaner, more maintainable CSS because additions are progressive rather than subtractive. It also aligns with the reality that mobile usage dominates and constrained devices benefit most from a focused baseline.

**Beginner-Friendly Explanation:** Mobile-first means you design for the phone first, then add improvements for bigger screens. It is easier to add things than to take them away.

### Purposes

- To ensure core functionality works on constrained devices.
- To produce cleaner, more maintainable CSS through progressive enhancement.
- To align with the dominant mobile usage pattern.
- To avoid desktop-centric layouts that break on small screens.

### Syntax Rules and Structure

```css
/* Mobile-first: base styles target mobile */
.card { padding: 16px; }

/* Progressive enhancement for larger screens */
@media (min-width: 768px) {
  .card { padding: 24px; display: flex; }
}
```

**Syntax Rules:**

1. Write base styles without media queries for mobile.
2. Use `min-width` media queries (not `max-width`) to add enhancements.
3. In Tailwind, unprefixed utilities are the mobile base; prefixed utilities apply from that breakpoint upward.
4. Test across key mobile widths (320px, 375px, 414px, 768px) to catch layout breaks.

### Annotated Code Examples

```jsx
function Navigation() {
  return (
    <nav className="flex flex-col gap-2 bg-gray-100 p-4 md:flex-row md:items-center md:gap-6">
      <a href="/" className="py-2 text-gray-900 active:bg-gray-200">Home</a>
      <a href="/about" className="py-2 text-gray-900 active:bg-gray-200">About</a>
    </nav>
  );
}
```

**Expected Output:** A vertically stacked navigation on mobile; a horizontal navigation from `md` upward. Each link has an `active` state for touch feedback.

**Why this output:** The base classes (`flex-col`, `gap-2`) define the mobile layout. The `md:` variants switch to a row layout and increase spacing. The `active:` variants provide touch feedback.

### Real-World Cases

- **Progressive web apps:** Core functionality must work on low-end mobile devices.
- **E-commerce:** Mobile checkout flows must be flawless.
- **Content sites:** Readable typography and touch-friendly navigation on phones.

### References

- ADR-013: Responsive Mobile-First Design — https://raw.githubusercontent.com/Sourdough-start/sourdough/refs/heads/master/docs/adr/013-responsive-mobile-first-design.md
- Compile-N-Run — React Responsive Design — https://github.com/Compile-N-Run/Compile-N-Run
- Volcengine — Mobile responsive testing — https://www.volcengine.com

---

## 5. Design Tokens

### Definitions

**Core Definition:** Design tokens are named, platform-agnostic values for design decisions (colors, spacing, typography, shadows, radii) that are stored as variables and referenced throughout an application.

**Technical Definition:** Design tokens are the single source of truth for design decisions. In CSS, they are implemented as CSS custom properties (e.g., `--color-primary`, `--spacing-4`, `--radius-md`) defined on `:root` and theme selectors. In JavaScript, they can be typed objects that generate CSS variables or inline style objects. A token-first architecture separates **primitive tokens** (raw values like `--blue-500: #3b82f6`) from **semantic tokens** (contextual aliases like `--color-primary: var(--blue-500)`). Semantic tokens are what components should reference, enabling theme changes without touching component code.

**Beginner-Friendly Explanation:** Design tokens are names for your design decisions. Instead of writing `#3b82f6` everywhere, you write `--color-primary`. If you want to change the primary color, you change it in one place and everything updates.

### Purposes

- To centralise design decisions and eliminate hardcoded values.
- To enable theme switching without changing component code.
- To ensure consistency across components and teams.
- To bridge design tools (Figma) and code through a shared token vocabulary.
- To support multiple platforms (web, iOS, Android) from the same token source.

### Syntax Rules and Structure

**CSS Custom Properties (Primitive + Semantic):**

```css
:root {
  /* Primitive tokens */
  --blue-500: #3b82f6;
  --gray-50: #f9fafb;
  --spacing-unit: 4px;

  /* Semantic tokens */
  --color-primary: var(--blue-500);
  --color-bg: var(--gray-50);
  --spacing-4: calc(var(--spacing-unit) * 4);
}
```

**Usage:**

```css
.button {
  background: var(--color-primary);
  padding: var(--spacing-4);
}
```

**Syntax Rules:**

1. Define tokens as CSS custom properties on `:root`.
2. Separate primitive tokens (raw values) from semantic tokens (contextual aliases).
3. Components should reference semantic tokens, not primitive tokens or raw values.
4. Override semantic tokens in theme selectors (`.dark`, `[data-theme="brand"]`).
5. In TypeScript, declare a `CSSProperties` extension for custom properties or use `React.CSSProperties` with a cast.

### Annotated Code Examples

```tsx
// tokens.css.ts (Vanilla Extract example)
import { createTheme } from '@vanilla-extract/css';

export const [themeClass, vars] = createTheme({
  color: {
    primary: '#3b82f6',
    background: '#ffffff',
    text: '#111827',
  },
  spacing: {
    sm: '4px',
    md: '8px',
    lg: '16px',
  },
});
```

```jsx
import { themeClass, vars } from './tokens.css';

function Button({ children }) {
  return (
    <button
      className={themeClass}
      style={{
        background: vars.color.primary,
        color: 'white',
        padding: `${vars.spacing.md} ${vars.spacing.lg}`,
      }}
    >
      {children}
    </button>
  );
}
```

**Expected Output:** A button that uses the theme's primary color and spacing tokens. Changing the token values updates every button that references them.

**Why this output:** The tokens are defined once and exported as CSS variables and a theme class. The button references the semantic tokens (`vars.color.primary`, `vars.spacing.md`), not raw values.

### Real-World Cases

- **Design systems:** Tokens are the foundation of the entire system.
- **Multi-brand products:** Swapping token values changes the entire brand appearance.
- **Design-to-code handoff:** Tokens exported from Figma (via Tokens Studio) map directly to CSS variables.

### References

- GitHub — Design System Patterns — https://github.com/aiskillstore/marketplace
- CoreUI — How to build a design system in React — https://coreui.io/answers/how-to-build-a-design-system-in-react/
- GitHub — Design tokens with CSS custom properties — https://github.com/wshobson/agents
- npm — react-design-tokens — https://www.npmjs.com/package/react-design-tokens

---

## 6. Design Systems

### Definitions

**Core Definition:** A design system is a collection of reusable components, design tokens, guidelines, and documentation that teams use to build consistent user interfaces at scale.

**Technical Definition:** A React design system is a structured, documented, and reusable collection of UI components, patterns, and tokens. Architecturally, it combines **design tokens** (the design decisions), a **theme provider** (distributing tokens via Context and CSS variables), and **atomic component architecture** (atoms → molecules → organisms). Modern design systems require RSC compatibility, headless accessibility primitives (Radix, React Aria), and tree-shakeable bundles as table stakes. Tools like Storybook provide documentation and visual testing.

**Beginner-Friendly Explanation:** A design system is like a company's official toolbox for building interfaces. It has all the approved buttons, inputs, colors, and spacing rules, so every team builds things that look and behave consistently.

### Purposes

- To ensure visual and behavioural consistency across applications and teams.
- To accelerate development by providing ready-made, tested components.
- To centralise accessibility and quality standards.
- To enable efficient theming and brand updates.
- To provide a shared vocabulary between designers and developers.

### Syntax Rules and Structure

**Design System Architecture Layers:**

| Layer | Description | Example |
|-------|-------------|---------|
| Tokens | Design decisions as variables | `--color-primary`, `--spacing-4` |
| Theme | Token distribution and switching | `ThemeProvider`, `.dark` class |
| Primitives | Unstyled accessible components | Radix UI, React Aria |
| Components | Styled, composed components | Button, Input, Card |
| Patterns | Composed layouts and flows | Forms, navigation, data tables |
| Documentation | Usage guidelines and examples | Storybook, MDX docs |

**Syntax Rules:**

1. Start with tokens; never hardcode colors or spacing in components.
2. Use a theme provider to distribute tokens via Context and CSS variables.
3. Build on accessible primitives (Radix, React Aria) rather than reinventing behaviour.
4. Organise components by complexity (Atomic Design: atoms → molecules → organisms).
5. Document every component with usage guidelines, props, and accessibility notes.
6. Ensure RSC compatibility by using zero-runtime styling (CSS Modules, Tailwind, compile-time CSS-in-JS).

### Annotated Code Examples

```tsx
// design-system/tokens.css.ts
export const tokens = {
  color: { primary: '#3b82f6', surface: '#ffffff', text: '#111827' },
  spacing: { sm: '4px', md: '8px', lg: '16px' },
};

// design-system/Button.tsx
import { tokens } from './tokens';

export function Button({ variant = 'primary', children, ...props }) {
  return (
    <button
      style={{
        background: variant === 'primary' ? tokens.color.primary : 'transparent',
        color: variant === 'primary' ? 'white' : tokens.color.text,
        padding: `${tokens.spacing.md} ${tokens.spacing.lg}`,
        border: variant === 'outline' ? `1px solid ${tokens.color.primary}` : 'none',
      }}
      {...props}
    >
      {children}
    </button>
  );
}
```

**Expected Output:** A button component that uses design tokens for its colors, spacing, and border. The token values can be changed centrally to update all buttons.

**Why this output:** The component references tokens instead of hardcoded values. The tokens are the single source of truth for design decisions.

### Real-World Cases

- **Enterprise organisations:** Multiple teams building consistent internal tools.
- **SaaS platforms:** A single component library shared across products.
- **Open-source projects:** Documented, reusable component systems (e.g., MUI, Ant Design).

### References

- CoreUI — How to build a design system in React — https://coreui.io/answers/how-to-build-a-design-system-in-react/
- GitHub — React Design System Patterns — https://github.com/Compile-N-Run/Compile-N-Run
- GitHub — Design System Implementation at Scale — https://github.com/sujeet-pro/sujeet.pro
- NASSCOM — Building a React design system in Storybook — https://community.nasscom.in

---

## 7. Style Composition

### Definitions

**Core Definition:** Style composition is the practice of combining multiple style sources (base classes, variant classes, conditional classes, consumer overrides) into a single, conflict-free `className` string.

**Technical Definition:** Style composition in React is achieved through the `cn` utility, which combines `clsx` (for conditional class logic) and `tailwind-merge` (for resolving conflicting Tailwind utility classes). `clsx` accepts strings, arrays, and objects with boolean values, producing a merged class string. `tailwind-merge` then resolves conflicts by keeping the last conflicting class (e.g., `cn('px-4', 'px-6')` → `'px-6'`). The `cn` utility is the industry-standard pattern for building flexible React components with Tailwind. For CSS Modules, composition is handled by the `composes` property.

**Beginner-Friendly Explanation:** Style composition is how you combine different sets of styles without creating conflicts. If you have base styles, variant styles, and custom styles, you use a helper to merge them so the last one wins where they overlap.

### Purposes

- To combine base, variant, conditional, and consumer styles cleanly.
- To resolve conflicting utility classes automatically.
- To allow component consumers to override styles without fighting specificity.
- To avoid whitespace bugs and string concatenation errors.

### Syntax Rules and Structure

**The `cn` Utility:**

```ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**Usage:**

```tsx
<button
  className={cn(
    'rounded px-4 py-2 font-medium',      // Base
    variant === 'primary' && 'bg-blue-600 text-white',
    variant === 'secondary' && 'bg-gray-200 text-gray-900',
    disabled && 'opacity-50 pointer-events-none',
    className                               // Consumer override (last)
  )}
/>
```

**Syntax Rules:**

1. Always use `cn` (or `clsx` + `tailwind-merge`) for conditional class composition.
2. Place base classes first, variant classes next, and the consumer's `className` last.
3. Never concatenate class strings manually with `+` or template literals for conditional logic.
4. For CSS Modules, use `composes` in the CSS file instead of runtime composition.
5. Accept `className` as a prop and merge it last so consumers can override styles.

**Constraints and Limitations:**

- `tailwind-merge` only understands Tailwind utility classes; custom classes are passed through.
- Overusing conditional logic in `className` can make components hard to read; extract variants into a `cva` or similar helper.

### Annotated Code Examples

```tsx
import { cn } from '@/lib/utils';

function Badge({ variant = 'default', size = 'md', className, ...props }) {
  return (
    <span
      className={cn(
        'inline-flex items-center rounded-full font-medium',
        {
          'px-2 py-1 text-xs': size === 'sm',
          'px-3 py-1.5 text-sm': size === 'md',
          'px-4 py-2 text-base': size === 'lg',
        },
        {
          'bg-gray-100 text-gray-800': variant === 'default',
          'bg-blue-100 text-blue-800': variant === 'primary',
          'bg-red-100 text-red-800': variant === 'danger',
        },
        className
      )}
      {...props}
    />
  );
}
```

**Expected Output:** A badge component that renders with the correct size and variant classes. If the consumer passes `className="bg-green-100"`, `tailwind-merge` resolves the conflict and keeps the consumer's background.

**Why this output:** `clsx` evaluates the conditional objects and combines the matching classes. `tailwind-merge` resolves conflicts by keeping the last conflicting utility. The consumer's `className` is placed last, giving it override priority.

### Real-World Cases

- **Design system components:** Every component with variants and consumer overrides.
- **Layout components:** Components that accept additional spacing or background classes.
- **Conditional UI:** Toggling classes based on loading, error, or disabled states.

### References

- GitHub — React Component Styling Instructions (clsx + tailwind-merge) — https://github.com/trae-op/electron-modular-boilerplate
- GitHub — cn Utility Patterns — https://github.com/yonatangross/orchestkit
- Steve Kinney — Merging and Deduplicating Class Names — https://stevekinney.com
- npm — @ripperdoc-chrome77/utils — https://www.npmjs.com/package/@ripperdoc-chrome77/utils

---

## 8. Conditional Styles

### Definitions

**Core Definition:** Conditional styles are styles that are applied or removed based on component state, props, or user interaction.

**Technical Definition:** Conditional styles in React are implemented through conditional class application (using `clsx`/`cn`), conditional inline style objects, or data attributes. The preferred approach is conditional class application because it keeps styles in CSS and avoids inline style performance costs. Data attributes (e.g., `data-state="open"`) can be combined with CSS attribute selectors to style based on state without JavaScript class manipulation. For state that changes frequently (hover, focus, active), CSS pseudo-classes should be used instead of JavaScript state.

**Beginner-Friendly Explanation:** Conditional styles mean changing how something looks based on what is happening. For example, a button might be blue normally, gray when disabled, and red when there is an error. You use conditional logic to switch the classes.

### Purposes

- To reflect component state visually (loading, error, disabled, active).
- To respond to user interaction (hover, focus, selected).
- To adapt styles based on props (variant, size, theme).
- To avoid manual DOM manipulation for state-based styling.

### Syntax Rules and Structure

**Conditional Classes with `cn`:**

```tsx
<button
  className={cn(
    'rounded px-4 py-2',
    isLoading && 'opacity-50 cursor-wait',
    isError && 'bg-red-600 text-white',
    isDisabled && 'pointer-events-none opacity-50'
  )}
/>
```

**Data Attributes:**

```tsx
<div data-state={isOpen ? 'open' : 'closed'} />
```

```css
[data-state="open"] { display: block; }
[data-state="closed"] { display: none; }
```

**Syntax Rules:**

1. Use `cn`/`clsx` for conditional class application; never concatenate strings manually.
2. Use CSS pseudo-classes (`:hover`, `:focus`, `:active`) for interaction states rather than JavaScript state.
3. Use data attributes for state that is shared between React and CSS (e.g., `data-state="open"`).
4. Avoid conditional inline styles for properties that can be expressed as classes; reserve inline styles for truly dynamic values.

### Annotated Code Examples

```tsx
function Alert({ variant, dismissible, onDismiss, children }) {
  return (
    <div
      role="alert"
      className={cn(
        'rounded-lg border p-4',
        variant === 'info' && 'border-blue-200 bg-blue-50 text-blue-800',
        variant === 'error' && 'border-red-200 bg-red-50 text-red-800',
        variant === 'success' && 'border-green-200 bg-green-50 text-green-800'
      )}
    >
      {children}
      {dismissible && (
        <button onClick={onDismiss} className="ml-4 text-sm underline">
          Dismiss
        </button>
      )}
    </div>
  );
}
```

**Expected Output:** An alert box with blue, red, or green styling depending on the `variant` prop, and a dismiss button only when `dismissible` is true.

**Why this output:** `cn` applies the base classes and the matching variant classes conditionally. The `dismissible` prop conditionally renders the dismiss button.

### Real-World Cases

- **Form validation:** Red borders and error messages when validation fails.
- **Loading states:** Disabled buttons and spinners while data is being submitted.
- **Selection states:** Highlighting the active tab or selected list item.

### References

- Chakra UI — Styling Performance: Data attributes for state-based styling — https://next.chakra-ui.com/guides/styling-performance
- GitHub — React Component Styling Instructions — https://github.com/trae-op/electron-modular-boilerplate

---

## 9. Dynamic Styles

### Definitions

**Core Definition:** Dynamic styles are styles whose values are computed at runtime from JavaScript variables, props, or state.

**Technical Definition:** Dynamic styles in React are implemented through inline `style` objects (for values that cannot be expressed as static classes) or CSS custom properties (for values that can be consumed by static CSS rules). Inline styles are appropriate **only** for values computed at runtime (e.g., a progress bar width from a percentage). For values that change frequently or are used across many components, CSS custom properties are preferred because they are resolved by the browser and do not require React re-renders. Inline style objects should always be typed with `React.CSSProperties` in TypeScript.

**Beginner-Friendly Explanation:** Dynamic styles are styles that change based on data. For example, a progress bar's width changes based on a percentage. You compute the value in JavaScript and pass it to the style.

### Purposes

- To set style values that cannot be known ahead of time (positions, percentages, user-selected colors).
- To respond to real-time data (animations, drag positions, progress).
- To avoid creating a class for every possible value.

### Syntax Rules and Structure

**Inline Style (Dynamic Values Only):**

```tsx
<div style={{ width: `${percent}%`, backgroundColor: color }} />
```

**CSS Custom Property (Preferred for Frequent Changes):**

```tsx
<div style={{ '--progress': `${percent}%` } as React.CSSProperties} />
```

```css
.progress-bar { width: var(--progress); }
```

**Syntax Rules:**

1. Use inline styles **only** for values computed at runtime from props or state.
2. Type inline style objects with `React.CSSProperties` in TypeScript.
3. For values that change frequently (animations, user-controlled properties), use CSS custom properties instead of inline styles.
4. Never use inline styles for pseudo-classes, media queries, or animations.
5. Memoize dynamic style objects with `useMemo` when passing them to memoized components.

### Annotated Code Examples

```tsx
import type { CSSProperties } from 'react';

function ProgressBar({ percent }: { percent: number }) {
  return (
    <div className="h-2 w-full rounded bg-gray-200">
      <div
        className="h-2 rounded bg-blue-600 transition-[width] duration-300"
        style={{ width: `${percent}%` } as CSSProperties}
      />
    </div>
  );
}
```

**Expected Output:** A progress bar whose filled width matches the `percent` prop and smoothly transitions when the value changes.

**Why this output:** The `width` is a truly dynamic value that cannot be expressed as a static class. The inline style is typed with `CSSProperties`. The `transition-[width]` class handles the animation without JavaScript.

### Real-World Cases

- **Progress indicators:** Width based on completion percentage.
- **Drag-and-drop:** Updating `transform` during dragging.
- **User-controlled colors:** Applying a selected accent color.

### References

- GitHub — React Styling Implementation (inline styles for dynamic values only) — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Chakra UI — Leverage CSS Variables for Dynamic Values — https://next.chakra-ui.com/guides/styling-performance
- Refine — A Quick Introduction to Meta's Stylex — https://refine.dev

---

## 10. Animation

### Definitions

**Core Definition:** Animation is the controlled change of visual properties over time to provide feedback, guide attention, or create a sense of continuity.

**Technical Definition:** Web animation in React can be implemented at three levels: **CSS transitions** (for simple state changes on hover, focus, or enter/exit), **CSS keyframe animations** (for looping or multi-step sequences), and **JavaScript animation libraries** (Framer Motion, React Spring) for gesture-driven, layout, and orchestrated animations. The performance rule is to animate **compositor-friendly properties** (`transform`, `opacity`, `filter`, `clip-path`) which run on the GPU compositor thread, avoiding properties that trigger layout or paint (`width`, `height`, `top`, `left`). Animations must respect `prefers-reduced-motion` and should generally stay under 300ms for UI feedback.

**Beginner-Friendly Explanation:** Animation makes your app feel alive. Simple things like a button fading in or a menu sliding open can be done with CSS. More complex things, like dragging a card or animating a layout change, need a JavaScript library. The key is to animate the right properties (transform and opacity) so it stays smooth.

### Purposes

- To provide visual feedback for interactions (hover, click, focus).
- To guide the user's attention to changes (notifications, errors).
- To create a sense of continuity between UI states (page transitions, layout changes).
- To make the interface feel polished and responsive.
- To respect users who prefer reduced motion.

### Syntax Rules and Structure

**CSS Transition:**

```css
.button {
  transition: transform 150ms ease-out, opacity 150ms ease-out;
}
.button:hover {
  transform: scale(1.02);
}
```

**CSS Keyframes:**

```css
@keyframes fade-in {
  from { opacity: 0; transform: translateY(4px); }
  to { opacity: 1; transform: translateY(0); }
}
```

**Reduced Motion:**

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

**Syntax Rules:**

1. Use CSS transitions for simple hover, focus, and enter/exit animations.
2. Use CSS keyframes for looping or multi-step sequences.
3. Animate `transform` and `opacity` for GPU-accelerated, 60fps performance; avoid animating layout-affecting properties.
4. Keep UI feedback animations under 300ms.
5. Always respect `prefers-reduced-motion` using the `useReducedMotion` hook (Framer Motion) or a CSS media query.
6. Do not install two animation libraries that animate the same property; they will conflict.

**Constraints and Limitations:**

- CSS transitions cannot animate `display: none` to `display: block`; use `@starting-style` or a library for enter/exit animations.
- JavaScript animation libraries add bundle weight; use them only when CSS cannot achieve the required effect.
- Animating SVG elements directly may not be GPU-accelerated; wrap them in a `<div>` and animate the wrapper.

### Annotated Code Examples

```tsx
import { motion, useReducedMotion } from 'framer-motion';

function FadeIn({ children }) {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      initial={shouldReduceMotion ? false : { opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.2, ease: 'easeOut' }}
    >
      {children}
    </motion.div>
  );
}
```

**Expected Output:** Content fades in and slides up slightly on mount. If the user prefers reduced motion, the content appears immediately without animation.

**Why this output:** `useReducedMotion` detects the user's preference. The `initial` prop is disabled when reduced motion is preferred. The animation uses `opacity` and `transform` (`y`), which are GPU-friendly.

### Real-World Cases

- **Toast notifications:** Slide-in and fade-out animations.
- **Modal dialogs:** Scale and fade transitions.
- **Drag-and-drop:** Spring-based movement following the pointer.
- **Page transitions:** Shared element animations between routes.

### References

- Emil Kowalski Animation Best Practices — https://github.com/pproenca/dot-skills
- UI Craft — Animation stack selection — https://github.com/educlopez/ui-craft
- CoreUI — How to animate components in React — https://coreui.io

---

## 11. Transitions

### Definitions

**Core Definition:** A transition is an animation that smoothly interpolates between two states of a property when that property changes.

**Technical Definition:** CSS transitions are defined with the `transition` shorthand, specifying the property (or properties) to animate, the duration, the easing function, and an optional delay. Transitions are triggered by a change in the computed value of a property, typically caused by a class change, pseudo-class activation (`:hover`, `:focus`), or a style change. Unlike keyframe animations, transitions are implicitly reversible: when the trigger condition is removed, the property transitions back to its previous value. For performance, transitions should target compositor-friendly properties (`transform`, `opacity`).

**Beginner-Friendly Explanation:** A transition is a smooth change from one style to another. When you hover over a button and it smoothly changes color, that is a transition.

### Purposes

- To provide smooth visual feedback for state changes.
- To make interactions feel responsive and polished.
- To avoid jarring instant changes.
- To create a sense of continuity between UI states.

### Syntax Rules and Structure

```css
.element {
  transition-property: transform, opacity;
  transition-duration: 200ms;
  transition-timing-function: ease-out;
  transition-delay: 0ms;
}

/* Shorthand */
.element {
  transition: transform 200ms ease-out, opacity 200ms ease-out;
}
```

**Syntax Rules:**

1. Specify which properties transition; avoid `transition: all` because it animates unintended properties and harms performance.
2. Use `ease-out` as the default easing for UI feedback.
3. Keep durations short (150–250ms for micro-interactions; under 300ms for UI feedback).
4. Transition only `transform` and `opacity` for GPU-accelerated performance.
5. Ensure the transition is disabled under `prefers-reduced-motion`.

### Annotated Code Examples

```jsx
function Button({ children }) {
  return (
    <button className="rounded bg-blue-600 px-4 py-2 text-white transition-[transform,background-color] duration-150 ease-out hover:scale-105 hover:bg-blue-700 active:scale-95 motion-reduce:transition-none motion-reduce:hover:scale-100">
      {children}
    </button>
  );
}
```

**Expected Output:** A button that scales up slightly and darkens on hover, scales down on active press, and does not animate when reduced motion is preferred.

**Why this output:** `transition-[transform,background-color]` explicitly limits the transition to transform and background color. `duration-150` keeps it fast. `motion-reduce:transition-none` disables the transition for users who prefer reduced motion.

### Real-World Cases

- **Buttons and links:** Hover, focus, and active state feedback.
- **Cards:** Shadow and transform transitions on hover.
- **Form inputs:** Border color transitions on focus.
- **Navigation menus:** Smooth open/close transitions.

### References

- @vitus-labs/kinetic — CSS transitions performance — https://www.npmjs.com/package/@vitus-labs/kinetic
- CoreUI — How to animate components in React — https://coreui.io
- Emil Kowalski Animation Best Practices (Timing & Duration) — https://github.com/pproenca/dot-skills

---

## 12. Styling Best Practices

### Definitions

**Core Definition:** Styling best practices are guidelines for writing maintainable, performant, and consistent styles in React applications.

**Technical Definition:** React styling best practices in 2026 centre on **zero-runtime styling for server-rendered surfaces** (CSS Modules, Tailwind, compile-time CSS-in-JS), **token-driven design decisions** (semantic CSS variables, not raw values), **composition over string concatenation** (`cn`/`clsx`/`tailwind-merge`), **minimal global CSS** (resets and variables only), and **accessibility-aware motion** (reduced motion, focus states, contrast). Runtime CSS-in-JS (styled-components, Emotion) is restricted to client components and is no longer the default choice for new projects.

**Beginner-Friendly Explanation:** Best practices are the habits that keep your styling clean. Use tokens instead of hardcoded values. Use zero-runtime tools for server-rendered apps. Use a helper to combine classes. Keep global styles minimal. Make sure animations respect user preferences.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Prefer zero-runtime styling** | CSS Modules, Tailwind, or compile-time CSS-in-JS for RSC and performance. |
| **Use semantic design tokens** | Reference `--color-primary` (semantic) rather than `--blue-500` (primitive) or raw hex values. |
| **Use `cn` for class composition** | Never concatenate class strings manually. |
| **Minimal global CSS** | Global styles should contain only resets, base typography, and CSS variables. |
| **Avoid inline styles for static styles** | Use inline styles only for values computed at runtime. |
| **Use CSS variables for dynamic values** | Better performance than inline style recomputation for frequent changes. |
| **Respect `prefers-reduced-motion`** | Disable animations for users who prefer reduced motion. |
| **Use the `dark` class strategy** | Allows explicit user control over dark mode. |
| **Test across mobile breakpoints** | 320px, 375px, 414px, 768px. |
| **Never use runtime CSS-in-JS in RSC** | It requires `useContext` for style injection, which RSC forbids. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Semantic tokens
:root {
  --color-bg: var(--gray-50);
  --color-text: var(--gray-900);
  --color-primary: var(--blue-600);
}
.dark {
  --color-bg: var(--gray-950);
  --color-text: var(--gray-50);
}
```

```tsx
// cn utility for composition
import { cn } from '@/lib/utils';

function Card({ className, children }) {
  return (
    <div className={cn('rounded-lg bg-[var(--color-bg)] p-6 shadow-sm', className)}>
      {children}
    </div>
  );
}
```

**❌ Bad Practices:**

```tsx
// ❌ Hardcoded values
<div className="bg-[#f9fafb] text-[#111827]" />

// ❌ String concatenation for conditional classes
<div className={'px-4 ' + (isActive ? 'bg-blue-500' : '')} />

// ❌ Runtime CSS-in-JS in RSC
import styled from 'styled-components';
```

**Expected Output:** The good practices produce consistent, themeable, and maintainable styles. The bad practices produce hardcoded, fragile, and RSC-incompatible code.

**Why this output:** Semantic tokens enable theme switching. `cn` resolves conflicts and handles conditional logic cleanly. Hardcoded values cannot be themed. String concatenation causes whitespace bugs. Runtime CSS-in-JS breaks in RSC.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Design systems:** Token discipline and zero-runtime styling are foundational.
- **Performance-sensitive surfaces:** Zero-runtime styling and GPU-friendly animations are critical.

### References

- RT Camp — Styling Best Practice for React-based projects — https://rtcamp.com
- GitHub — AI Web Design Codex: CSS styling and Tailwind — https://github.com/Eneryleen/ai-web-design-codex
- Impertio Studio — React Styling Implementation — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- GitHub — Styling Approach Decision Trees — https://github.com/mcorbett51090/RavenClaude

---

## References

- CoreUI — How to create a theme provider in React — https://coreui.io/answers/how-to-create-a-theme-provider-in-react/
- CoreUI — How to build a design system in React — https://coreui.io/answers/how-to-build-a-design-system-in-react/
- CoreUI — How to implement dark mode in React — https://coreui.io/answers/how-to-implement-dark-mode-in-react/
- CoreUI — How to animate components in React — https://coreui.io
- CoreUI — Breakpoints — https://coreui.io
- Architecture Decision Records — Class-based dark mode toggle — https://raw.githubusercontent.com/justinchuby/flightdeck/refs/heads/main/packages/docs/reference/architecture-decisions.md
- ADR-013: Responsive Mobile-First Design — https://raw.githubusercontent.com/Sourdough-start/sourdough/refs/heads/master/docs/adr/013-responsive-mobile-first-design.md
- GitHub — MUI theming patterns — https://github.com/agents-inc/skills/blob/main/src/skills/web-ui-mui/SKILL.md
- GitHub — React Component Styling Instructions (clsx + tailwind-merge) — https://github.com/trae-op/electron-modular-boilerplate
- GitHub — cn Utility Patterns — https://github.com/yonatangross/orchestkit
- Steve Kinney — Merging and Deduplicating Class Names — https://stevekinney.com
- GitHub — React Styling Implementation (inline styles for dynamic values only) — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Chakra UI — Styling Performance: Data attributes and CSS variables — https://next.chakra-ui.com/guides/styling-performance
- Emil Kowalski Animation Best Practices — https://github.com/pproenca/dot-skills
- UI Craft — Animation stack selection — https://github.com/educlopez/ui-craft
- @vitus-labs/kinetic — CSS transitions performance — https://www.npmjs.com/package/@vitus-labs/kinetic
- RT Camp — Styling Best Practice for React-based projects — https://rtcamp.com
- GitHub — AI Web Design Codex: CSS styling and Tailwind — https://github.com/Eneryleen/ai-web-design-codex
- GitHub — Styling Approach Decision Trees — https://github.com/mcorbett51090/RavenClaude
- GitHub — Design System Patterns — https://github.com/aiskillstore/marketplace
- GitHub — Design tokens with CSS custom properties — https://github.com/wshobson/agents
- npm — react-design-tokens — https://www.npmjs.com/package/react-design-tokens
- GitHub — React Design System Patterns — https://github.com/Compile-N-Run/Compile-N-Run
- GitHub — Design System Implementation at Scale — https://github.com/sujeet-pro/sujeet.pro
- NASSCOM — Building a React design system in Storybook — https://community.nasscom.in
- ASOasis — React Dark Mode Theme Toggle: A Modern, Accessible Tutorial — https://asoasis.tech
- Refine — A Quick Introduction to Meta's Stylex — https://refine.dev
- npm — @ripperdoc-chrome77/utils — https://www.npmjs.com/package/@ripperdoc-chrome77/utils
- GitHub — React Responsive Patterns — https://github.com/yonatangross/orchestkit
- Compile-N-Run — React Responsive Design — https://github.com/Compile-N-Run/Compile-N-Run
- Volcengine — Mobile responsive testing — https://www.volcengine.com