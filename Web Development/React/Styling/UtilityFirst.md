# Utility-First Styling (Tailwind CSS): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Tailwind CSS is a utility-first CSS framework that provides low-level utility classes to build completely custom designs without ever leaving your HTML.

**Technical Definition:** Tailwind CSS is a utility-first CSS framework that generates CSS on-demand by scanning source files for class name candidates and emitting only the styles that are actually used. Tailwind CSS v4 introduces a high-performance engine built in Rust called Oxide, delivering up to 10x faster builds and better developer experience. The framework uses a CSS-first configuration model via the `@theme` directive, replacing the traditional JavaScript config file, and automatically exposes design tokens as native CSS custom properties. Tailwind v4 requires Safari 16.4+, Chrome 111+, and Firefox 128+.

**Beginner-Friendly Explanation:** Tailwind is like a box of Lego bricks for styling. Instead of writing custom CSS for every component, you snap together small, single-purpose classes directly in your HTML. Need padding? Add `p-4`. Need a blue background? Add `bg-blue-500`. The framework scans your code, finds which classes you used, and generates only those styles—nothing more. This makes your CSS file tiny in production and lets you build designs without ever leaving your markup.

### Key Characteristics

- **Utility-First:** Low-level utility classes compose to build custom designs without fighting opinionated component styles.
- **On-Demand Generation:** JIT (Just-In-Time) compilation scans source files and generates only the CSS classes actually used.
- **Rust-Powered Oxide Engine:** v4's engine delivers 10x faster full builds (105ms vs 960ms) and 100x faster incremental builds.
- **CSS-First Configuration:** Theme values are defined in CSS via the `@theme` directive instead of `tailwind.config.js`.
- **Automatic Content Detection:** No `content` array needed; Tailwind automatically detects source files and respects `.gitignore`.
- **Native CSS Variables:** All theme tokens are exposed as CSS custom properties, enabling dynamic theming and runtime switching.
- **Composable Variants:** Responsive, state, group, peer, and `has-*` variants chain together for complex interactive designs.
- **Minimal Production Bundle:** Unused CSS is automatically removed, resulting in the smallest possible file size.

### Prerequisites

- Solid understanding of HTML and CSS (selectors, specificity, the box model).
- Familiarity with a build tool (Vite, Next.js, PostCSS) and npm/yarn/pnpm.
- Working knowledge of React function components and JSX.
- Basic understanding of responsive design and CSS custom properties.
- Awareness of utility-first CSS philosophy and its trade-offs.

### Related Programming Areas

- **CSS Architecture:** Cascade layers, CSS Modules, and native CSS features.
- **Design Systems:** Design tokens, theming, and component libraries.
- **Responsive Design:** Media queries, container queries, and fluid typography.
- **Build Tooling:** Vite plugins, PostCSS, and Lightning CSS.
- **Component Libraries:** shadcn/ui, Headless UI, and Radix Primitives.

### Core Concepts / Features

1. Installation, Compilation Pipelines, and the Tailwind v4 Engine
2. Optimizing Bundle Sizes via JIT Compilation and Scanning Architectures
3. UI Scaling with Responsive Utilities and Aspect-Ratio Modifiers
4. Interaction Design with State Variants
5. Design-Token Integration: Themes, Fluid Spacing, and Color Palettes
6. Production Organization: Class Sorting and Conflict Resolution

---

## Core Concept 1: Installation, Compilation Pipelines, and the Tailwind v4 Engine

### Definitions

**Core Definition:** Installing Tailwind CSS v4 involves adding the `tailwindcss` package and a build-tool plugin (Vite, PostCSS, or CLI), importing Tailwind in a CSS file, and configuring theme values directly in CSS via the `@theme` directive.

**Technical Definition:** Tailwind CSS v4 is installed as a standalone package with a single `@import "tailwindcss";` statement replacing the three directives from v3 (`@tailwind base;`, `@tailwind components;`, `@tailwind utilities;`). The recommended installation for Vite projects uses the first-party `@tailwindcss/vite` plugin, which replaces PostCSS, autoprefixer, and manual config wiring. The v4 engine, code-named Oxide, is a ground-up Rust rewrite that unifies development and production builds, handles CSS parsing via Lightning CSS, and performs incremental compilation that is up to 100x faster than v3. There is no `init` process in v4; the `npx tailwindcss init` command has been completely removed.

**Beginner-Friendly Explanation:** In v3, setting up Tailwind was a multi-step process: install packages, create a config file, add three directives to your CSS, and configure PostCSS. In v4, it is much simpler: install the package and the Vite plugin, add one import line to your CSS, and you are done. The new Rust engine is like swapping a bicycle for a race car—same destination, dramatically faster.

### Purposes

- To install Tailwind CSS with a single command and a single CSS import.
- To leverage the Rust-powered Oxide engine for faster builds and incremental compilation.
- To use the Vite plugin as the recommended installation path for Vite projects.
- To configure theme values directly in CSS via the `@theme` directive.
- To eliminate the need for PostCSS, autoprefixer, and `postcss-import` in v4.

### Syntax Rules and Structure

**Installation (Vite — Recommended):**
```bash
npm install tailwindcss@latest @tailwindcss/vite@latest
```

**Vite Configuration:**
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [tailwindcss()],
});
```

**CSS Import:**
```css
/* src/styles/tailwind.css */
@import "tailwindcss";

@theme {
  --color-primary: #3b82f6;
  --color-secondary: #8b5cf6;
  --spacing-18: 4.5rem;
  --spacing-128: 32rem;
}
```

**Component Breakdown:**
- `@import "tailwindcss"`: Single entry point that injects all four logical layers (theme, base, components, utilities).
- `@theme { ... }`: Defines design tokens directly in CSS; values are automatically exposed as CSS custom properties and generate utility classes.
- `--color-primary`: Creates `bg-primary`, `text-primary`, `border-primary`, etc.
- `--spacing-18`: Creates `p-18`, `m-18`, `gap-18`, etc.

**Manual Installation (PostCSS):**
```bash
npm install tailwindcss@latest @tailwindcss/postcss@latest
```

```javascript
// postcss.config.js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

**Using the CLI:**
```bash
npm install tailwindcss@latest @tailwindcss/cli@latest
npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --watch
```

**Syntax Rules:**
- Use `@tailwindcss/vite` for Vite projects; it is the recommended, fastest path.
- Use `@tailwindcss/postcss` for PostCSS-based setups (Next.js, Webpack).
- Use `@tailwindcss/cli` for standalone builds without a bundler.
- Replace `@tailwind base; @tailwind components; @tailwind utilities;` with `@import "tailwindcss";`.
- Remove `tailwind.config.js` if you are migrating to CSS-first configuration; v4 does not use it by default.
- Node.js 20+ is required for Tailwind v4.

**Constraints and Limitations:**
- v4 requires Safari 16.4+, Chrome 111+, and Firefox 128+; older browsers need v3.4.
- v4 no longer supports Sass, Less, or Stylus preprocessors; Lightning CSS handles CSS parsing instead.
- The `init` command and `tailwind.config.js` are no longer required but are still supported via `@config` for backward compatibility.
- Automatic content detection skips `.gitignore` and `node_modules` for performance; use `@source` to register additional paths.

### Annotated Code Example: Complete Vite + React Setup

```bash
# 1. Create a Vite React project
npm create vite@latest my-app -- --template react-ts
cd my-app

# 2. Install Tailwind CSS v4 and the Vite plugin
npm install tailwindcss@latest @tailwindcss/vite@latest
```

```typescript
// 3. Configure Vite
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

```css
/* 4. Create the Tailwind CSS entry point */
/* src/styles/tailwind.css */
@import "tailwindcss";

@theme {
  --color-brand: #6366f1;
  --color-brand-dark: #4f46e5;
  --spacing-18: 4.5rem;
}
```

```tsx
// 5. Import the CSS in your app entry
// src/main.tsx
import './styles/tailwind.css';
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

```tsx
// 6. Use Tailwind utilities in components
// src/App.tsx
export default function App() {
  return (
    <div className="min-h-screen bg-gray-50 flex items-center justify-center">
      <button className="bg-brand hover:bg-brand-dark text-white font-bold py-2 px-4 rounded transition-colors">
        Click me
      </button>
    </div>
  );
}
```

**Expected Output:** A full-screen grey background with a centered button. The button has the brand colour (`#6366f1`), a darker hover state, white text, bold weight, padding, and rounded corners. The `bg-brand` utility is generated automatically from the `--color-brand` theme variable.

**Why This Output Occurs:** The `@import "tailwindcss"` statement injects Tailwind's layers. The `@theme` block defines `--color-brand`, which Tailwind converts into the `bg-brand` and `hover:bg-brand-dark` utilities. The Vite plugin scans the JSX for class names and generates only the CSS needed for those classes.

### Real-World Cases

- **Next.js App Router:** Use `@tailwindcss/postcss` with the App Router; styling works in both Server and Client Components.
- **Vite + React/Vue/Svelte:** Use `@tailwindcss/vite` for the fastest development experience.
- **Astro:** Use `@tailwindcss/vite` with Astro's Vite integration.
- **Monorepos:** Use the CLI or PostCSS plugin with a shared Tailwind CSS entry point across packages.
- **Enterprise migration:** Use `npx @tailwindcss/upgrade@next` to automate v3 to v4 migration.

### References

- Tailwind CSS – Installation: https://mintlify.wiki/tailwindlabs/tailwindcss/installation
- Tailwind CSS – Quickstart Guide: https://mintlify.wiki/tailwindlabs/tailwindcss/quickstart
- Tailwind CSS – Upgrade Guide: https://tailwindcss.com/docs/upgrade-guide
- Steve Kinney – Tailwind Oxide: https://stevekinney.com/courses/tailwind/tailwind-oxide
- Tailwind CSS – Introduction: https://mintlify.wiki/tailwindlabs/tailwindcss/introduction

---

## Core Concept 2: Optimizing Bundle Sizes via JIT Compilation and Scanning Architectures

### Definitions

**Core Definition:** JIT (Just-In-Time) compilation scans source files for class name candidates and generates only the CSS rules that are actually used, resulting in minimal production bundle sizes.

**Technical Definition:** Tailwind CSS v4's JIT engine operates in three steps: (1) **Scan source files** — the engine reads all source files (HTML, JS, JSX, Vue, etc.) as plain text without syntax parsing; (2) **Extract candidates** — regular expressions extract all strings that could be Tailwind utility classes; (3) **Generate CSS** — only the CSS rules for recognised, valid utility classes are generated and injected into the final stylesheet. Unlike v3, JIT is always enabled in v4—there is no separate mode to enable, and all builds use on-demand compilation. Automatic content detection respects `.gitignore` and skips `node_modules` by default for performance. The `@source` directive allows explicit registration of additional scan paths.

**Beginner-Friendly Explanation:** Imagine you are building a house and you have a catalogue of 10,000 possible materials. Instead of ordering all 10,000 materials and letting 9,900 sit unused, you read your blueprint (source files), figure out exactly which 100 materials you need, and order only those. That is JIT: it reads your code, finds the classes you used, and generates only those styles. The result is a tiny CSS file instead of a massive one.

### Purposes

- To generate only the CSS classes actually used in the project.
- To minimise production bundle size by eliminating unused styles.
- To enable automatic content detection without a manual `content` configuration.
- To support explicit path registration via `@source` for edge cases (monorepos, component libraries).
- To handle dynamic class names safely without generating invalid CSS.

### Syntax Rules and Structure

**How JIT Works (Three Steps):**

| Step | Action | Detail |
|---|---|---|
| **1. Scan** | Read source files as plain text | No AST parsing; fast regex-based extraction |
| **2. Extract** | Find candidate class strings | Matches patterns like `bg-`, `text-`, `p-`, etc. |
| **3. Generate** | Emit CSS for valid utilities | Only recognised classes produce CSS rules |

**Explicit Source Registration (`@source`):**
```css
/* main.css */
@import "tailwindcss";

/* Register additional scan paths */
@source "./src/**/*.{js,jsx,ts,tsx}";

/* Set a base path for relative sources */
@source "source('./src')";

/* Exclude specific paths */
@source not "./src/vendor/**/*.js";
```

**Component Breakdown:**
- `@source "./src/**/*.{js,jsx,ts,tsx}"`: Explicitly tells the engine to scan these files.
- `source('./src')`: Sets the base directory for resolving `@source` paths.
- `not "./src/vendor/**/*.js"`: Excludes vendor files from scanning.

**Dynamic Class Names (Handling):**
```jsx
// ❌ Dynamic class names are NOT detected by the scanner
<div className={`bg-${color}-500`}>...</div>

// ✅ Use complete class names that the scanner can detect
const colorClasses = {
  red: 'bg-red-500',
  blue: 'bg-blue-500',
  green: 'bg-green-500',
};
<div className={colorClasses[color]}>...</div>

// ✅ Or use inline styles for truly dynamic values
<div style={{ backgroundColor: `var(--color-${color})` }}>...</div>
```

**Component Breakdown:**
- The scanner only finds complete class strings; it does not evaluate JavaScript.
- Map dynamic values to full class names in a lookup object.
- For truly dynamic values, use inline styles with CSS variables.

**Syntax Rules:**
- No `content` array is needed in v4; automatic detection handles most projects.
- Use `@source` to register paths outside the default scan scope (e.g., `node_modules` for component libraries).
- Never construct class names dynamically (`bg-${color}-500`); the scanner cannot detect them.
- Use complete class names in lookup objects for conditional styling.
- Use inline styles with CSS variables for truly runtime-dynamic values.
- The scanner skips `.gitignore` and `node_modules` for performance.

**Constraints and Limitations:**
- The scanner does not evaluate JavaScript; dynamic class names are invisible to it.
- Files in `.gitignore` and `node_modules` are not scanned by default.
- Very large monorepos may need explicit `@source` registration for shared packages.
- The plain-text scanning approach can produce false positives (strings that look like classes but are not), but Tailwind validates candidates before generating CSS.
- Arbitrary value classes with brackets (e.g., `w-[137px]`) are detected and generated correctly.

### Annotated Code Example: JIT in Action

```jsx
// App.tsx
export default function App() {
  return (
    <div className="p-4 bg-blue-500 text-white rounded-lg shadow-md">
      <h1 className="text-xl font-bold mb-2">Dashboard</h1>
      <p className="text-sm opacity-90">Welcome back!</p>
      <button className="mt-4 px-4 py-2 bg-white text-blue-500 rounded hover:bg-gray-100 transition-colors">
        View Details
      </button>
    </div>
  );
}
```

**Generated CSS (only what is used):**
```css
.p-4 { padding: 1rem; }
.bg-blue-500 { background-color: oklch(0.623 0.214 259.815); }
.text-white { color: #fff; }
.rounded-lg { border-radius: 0.5rem; }
.shadow-md { box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1); }
.text-xl { font-size: 1.25rem; line-height: 1.75rem; }
.font-bold { font-weight: 700; }
.mb-2 { margin-bottom: 0.5rem; }
.text-sm { font-size: 0.875rem; line-height: 1.25rem; }
.opacity-90 { opacity: 0.9; }
.mt-4 { margin-top: 1rem; }
.px-4 { padding-left: 1rem; padding-right: 1rem; }
.py-2 { padding-top: 0.5rem; padding-bottom: 0.5rem; }
.bg-white { background-color: #fff; }
.text-blue-500 { color: oklch(0.623 0.214 259.815); }
.hover\:bg-gray-100:hover { background-color: oklch(0.967 0.003 264.542); }
.transition-colors { transition-property: color, background-color, border-color; transition-duration: 150ms; }
```

**Expected Output:** A styled card with padding, blue background, white text, rounded corners, and a shadow. The heading is bold and larger; the paragraph is smaller and slightly transparent. The button has a white background, blue text, and a hover effect.

**Why This Output Occurs:** The JIT engine scans `App.tsx`, extracts every class name string it finds, validates them against Tailwind's utility definitions, and generates CSS only for those classes. Classes that are not used anywhere in the project are never generated, keeping the CSS file minimal.

### Real-World Cases

- **Production SPAs:** A typical React app uses 50–200 utility classes; the generated CSS is often under 10 KB.
- **Design systems:** A component library ships Tailwind classes; consumers use `@source` to scan the library's compiled output.
- **Monorepos:** Shared UI packages register their source paths with `@source` so Tailwind generates the classes they use.
- **Content-heavy sites:** JIT ensures that even with hundreds of components, only the classes in the shipped markup are generated.

### References

- Tailwind CSS v4 Architecture (ZlatanCN): https://raw.githubusercontent.com/ZlatanCN/personal-website/refs/heads/master/data/blog/css/tailwind-css-v4-architecture.mdx
- Tailwind CSS v4 – Architecture, Features, and Performance Upgrades: https://dev.to
- Tailwind CSS – Source Detection (`@source`): https://tailwindcss.com/docs/detecting-classes-in-source-files
- Tailwind CSS – Introduction: https://mintlify.wiki/tailwindlabs/tailwindcss/introduction

---

## Core Concept 3: UI Scaling with Responsive Utilities and Aspect-Ratio Modifiers

### Definitions

**Core Definition:** Responsive utilities are variant-prefixed classes (e.g., `md:`, `lg:`) that apply styles conditionally at specific breakpoints, while aspect-ratio modifiers control the proportional relationship between an element's width and height.

**Technical Definition:** Tailwind's responsive system uses breakpoint variants prefixed to any utility. The default breakpoints are `sm` (40rem), `md` (48rem), `lg` (64rem), `xl` (80rem), and `2xl` (96rem), defined as `--breakpoint-*` theme variables. Aspect-ratio utilities (`aspect-<ratio>`, `aspect-square`, `aspect-video`, `aspect-auto`) control the CSS `aspect-ratio` property, with support for custom values via `aspect-[<value>]` syntax and theme variables via `--aspect-*`. Responsive prefixes combine with aspect-ratio utilities: `md:aspect-square` applies a square ratio at medium breakpoints and above.

**Beginner-Friendly Explanation:** Responsive utilities let you say "on small screens, stack these cards vertically; on medium screens and up, show them side by side." Aspect-ratio modifiers let you say "this image should always be a square" or "this video should always be 16:9." Together, they make components that look great on every device—from a phone in one hand to a widescreen monitor.

### Purposes

- To apply different styles at different viewport widths using breakpoint prefixes.
- To maintain consistent proportions for images, videos, and containers across screen sizes.
- To combine responsive and aspect-ratio utilities for adaptive media layouts.
- To define custom breakpoints and aspect ratios via theme variables.
- To use container queries (in v4) for component-level responsiveness.

### Syntax Rules and Structure

**Responsive Breakpoints:**
```html
<!-- Stack on mobile, side-by-side on medium and up -->
<div class="flex flex-col md:flex-row gap-4">
  <div class="w-full md:w-1/2">Column 1</div>
  <div class="w-full md:w-1/2">Column 2</div>
</div>
```

**Component Breakdown:**
- `flex flex-col`: Default (mobile) layout is vertical.
- `md:flex-row`: At `md` (48rem) and above, switch to horizontal.
- `w-full md:w-1/2`: Full width on mobile; half width on `md` and above.

**Aspect-Ratio Utilities:**
```html
<!-- Square avatar -->
<img class="aspect-square w-12 rounded-full" src="/avatar.jpg" alt="Avatar" />

<!-- 16:9 video embed -->
<iframe class="aspect-video w-full" src="https://www.youtube.com/embed/..."></iframe>

<!-- Custom 4:3 ratio -->
<div class="aspect-[4/3] bg-gray-200">4:3 container</div>

<!-- Theme variable ratio -->
<iframe class="aspect-retro w-full" src="..."></iframe>
```

**Component Breakdown:**
- `aspect-square`: `aspect-ratio: 1 / 1` — square.
- `aspect-video`: `aspect-ratio: 16 / 9` — widescreen video.
- `aspect-[4/3]`: Arbitrary value — 4:3 ratio.
- `aspect-retro`: Custom ratio defined in `@theme`.

**Combining Responsive + Aspect-Ratio:**
```html
<!-- Square on mobile, 16:9 on medium and up -->
<div class="aspect-square md:aspect-video bg-gray-200">
  Responsive aspect ratio
</div>
```

**Component Breakdown:**
- `aspect-square`: Default (mobile) is square.
- `md:aspect-video`: At `md` and above, switch to 16:9.

**Customizing Aspect Ratios:**
```css
@theme {
  --aspect-retro: 4 / 3;
  --aspect-portrait: 3 / 4;
}
```

```html
<div class="aspect-retro">4:3</div>
<div class="aspect-portrait">3:4</div>
```

**Syntax Rules:**
- Prefix any utility with a breakpoint (`sm:`, `md:`, `lg:`, `xl:`, `2xl:`) to apply it at that breakpoint and above.
- Use `aspect-square` for 1:1, `aspect-video` for 16:9, and `aspect-auto` for the browser's default.
- Use `aspect-[<value>]` for arbitrary ratios (e.g., `aspect-[4/3]`).
- Use `aspect-(<custom-property>)` for CSS variable ratios.
- Define custom ratios in `@theme` with `--aspect-*` variables.
- Combine breakpoint prefixes with aspect-ratio utilities for responsive proportions.

**Constraints and Limitations:**
- Aspect-ratio utilities require the element to have a defined width (or be in a flex/grid container that gives it one).
- Tailwind v4 Preflight ships `img { height: auto }`, which can override the `height` attribute on images; use explicit aspect-ratio classes instead.
- Dynamically generated aspect-ratio classes (`aspect-[${ratio}]`) are not detected by the scanner; use a lookup object or inline styles.
- Breakpoints are viewport-based; use container queries for component-level responsiveness.

### Annotated Code Example: Responsive Product Grid with Aspect Ratios

```jsx
export default function ProductGrid({ products }) {
  return (
    <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 p-4">
      {products.map((product) => (
        <div key={product.id} className="group rounded-lg overflow-hidden shadow-md hover:shadow-xl transition-shadow">
          {/* Responsive aspect ratio: square on mobile, 4:3 on large screens */}
          <div className="aspect-square lg:aspect-[4/3] bg-gray-100">
            <img
              src={product.image}
              alt={product.name}
              className="w-full h-full object-cover"
            />
          </div>
          <div className="p-4">
            <h3 className="font-semibold text-lg group-hover:text-blue-600 transition-colors">
              {product.name}
            </h3>
            <p className="text-gray-600 text-sm mt-1">${product.price}</p>
          </div>
        </div>
      ))}
    </div>
  );
}
```

**Expected Output:** A product grid with 1 column on mobile, 2 on small screens, and 3 on large screens. Each product card has a square image on mobile and a 4:3 image on large screens. The card lifts on hover with a stronger shadow, and the title turns blue.

**Why This Output Occurs:** `grid-cols-1 sm:grid-cols-2 lg:grid-cols-3` changes the number of columns at each breakpoint. `aspect-square lg:aspect-[4/3]` changes the image proportion at the `lg` breakpoint. The `group` class on the card and `group-hover:` on the title enable the hover effect.

### Real-World Cases

- **E-commerce product grids:** Responsive column counts with consistent image proportions.
- **Blog post cards:** Thumbnail images that maintain aspect ratio across devices.
- **Video platforms:** 16:9 embeds that scale fluidly with the container.
- **Profile pages:** Square avatars that crop consistently regardless of the source image.
- **Dashboard widgets:** Charts and graphs with fixed aspect ratios in responsive grid cells.

### References

- Tailwind CSS – aspect-ratio: https://tailwindcss.com/docs/aspect-ratio
- Tailwind CSS – Responsive Design: https://tailwindcss.com/docs/responsive-design
- Tailwind CSS – Theme Customization: https://mintlify.wiki/tailwindlabs/tailwindcss/customization/theme
- voku/agent-skills – Responsive Aspect Ratio: https://github.com/voku/agent-skills

---

## Core Concept 4: Interaction Design with State Variants

### Definitions

**Core Definition:** State variants are prefixes (`hover:`, `focus:`, `active:`, `group-hover:`, `peer-checked:`) that apply utility classes conditionally based on an element's state, its parent's state (`group`), or a sibling's state (`peer`).

**Technical Definition:** Tailwind provides first-class support for styling elements on `hover`, `focus`, `active`, `disabled`, `visited`, `focus-within`, `focus-visible`, and more. The `group` variant styles descendants based on the parent's state: add `group` to the parent, then use `group-hover:`, `group-focus:`, etc. on children. The `peer` variant styles an element based on the state of a *previous sibling*: add `peer` to the sibling, then use `peer-hover:`, `peer-checked:`, `peer-invalid:`, etc. on subsequent siblings. Named groups (`group/name`) and named peers (`peer/name`) allow nesting and targeting specific ancestors or siblings. The `has-*` variant styles an element based on its descendants: `has-[:checked]` targets an element when a descendant checkbox is checked.

**Beginner-Friendly Explanation:** State variants let you write styles like "when the user hovers over this card, make the title blue" or "when this checkbox is checked, show this message." `group` is for parent-to-child effects (hover the card → highlight the title). `peer` is for sibling-to-sibling effects (check the checkbox → style the label after it). `has-*` is for child-to-parent effects (any child focused → highlight the parent).

### Purposes

- **`hover:`** — To apply styles when the pointer is over an element.
- **`focus:` / `focus-visible:` / `focus-within:`** — To apply styles when an element or its descendant receives focus.
- **`active:`** — To apply styles while an element is being pressed.
- **`disabled:`** — To style disabled form controls.
- **`group-hover:` / `group-focus:`** — To style children when a parent is hovered or focused.
- **`peer-checked:` / `peer-invalid:`** — To style siblings based on a preceding sibling's state.
- **`has-[:checked]`** — To style a parent when a descendant matches a selector.

### Syntax Rules and Structure

**Basic State Variants:**
```html
<button class="bg-blue-500 hover:bg-blue-700 focus:ring-2 focus:ring-blue-300 active:bg-blue-800 disabled:opacity-50 disabled:cursor-not-allowed text-white px-4 py-2 rounded">
  Click me
</button>
```

**Component Breakdown:**
- `hover:bg-blue-700`: Darker background on hover.
- `focus:ring-2 focus:ring-blue-300`: Focus ring for keyboard users.
- `active:bg-blue-800`: Even darker while pressed.
- `disabled:opacity-50 disabled:cursor-not-allowed`: Dimmed and non-interactive when disabled.

**Group Modifiers (Parent → Child):**
```html
<div class="group rounded-lg bg-white p-6 shadow-md hover:shadow-xl transition-shadow">
  <h3 class="mb-2 text-lg font-semibold transition-colors group-hover:text-blue-600">
    Product Card
  </h3>
  <p class="mb-4 text-gray-600 transition-colors group-hover:text-gray-800">
    A fantastic product description.
  </p>
  <button class="rounded bg-blue-500 px-4 py-2 text-white transition-all group-hover:scale-105 group-hover:bg-blue-600">
    Buy Now
  </button>
</div>
```

**Component Breakdown:**
- `group` on the parent enables `group-*` variants on descendants.
- `group-hover:text-blue-600`: Title turns blue when the card is hovered.
- `group-hover:scale-105`: Button scales up slightly when the card is hovered.

**Peer Modifiers (Sibling → Sibling):**
```html
<div class="space-y-1">
  <input type="email" id="email" class="peer block rounded-sm outline-1" required />
  <p class="peer-invalid:text-red-500 peer-invalid:block hidden">
    Please provide a valid email
  </p>
</div>
```

**Component Breakdown:**
- `peer` on the input enables `peer-*` variants on subsequent siblings.
- `peer-invalid:text-red-500`: The message turns red when the input is invalid.
- `peer-invalid:block hidden`: The message is hidden by default and shown when invalid.

**Named Groups (Nested):**
```html
<div class="group/outer">
  <div class="group/inner">
    <p class="group-hover/inner:text-blue-500 group-hover/outer:text-red-500">
      Responds to specific parent
    </p>
  </div>
</div>
```

**Component Breakdown:**
- `group/outer` and `group/inner`: Named groups for nesting.
- `group-hover/inner:text-blue-500`: Blue when the inner group is hovered.
- `group-hover/outer:text-red-500`: Red when the outer group is hovered.

**`has-*` Variant:**
```html
<div class="group has-[:focus]:ring-2 has-[:focus]:ring-blue-500 p-4 rounded">
  <input type="text" placeholder="Focus me" />
  <input type="text" placeholder="Or me" />
</div>
```

**Component Breakdown:**
- `has-[:focus]:ring-2`: Applies a ring when any descendant has focus.
- The parent is styled based on its children's state.

**Syntax Rules:**
- Use `hover:`, `focus:`, `active:`, `disabled:` directly on the element.
- Use `group` + `group-*:` for parent-to-child state effects.
- Use `peer` + `peer-*:` for previous-sibling-to-subsequent-sibling effects.
- Use named groups (`group/name`, `peer/name`) for nesting and targeting.
- Use `has-*` for child-to-parent effects.
- Chain variants: `group-has-focus:not-disabled:opacity-100`.
- `peer` only works on subsequent siblings in the DOM.

**Constraints and Limitations:**
- `group` and `peer` require the marker class on the correct element (parent or previous sibling).
- `peer` cannot target elements before it in the DOM.
- Overusing `group` can make components harder to reason about; use named groups for clarity.
- `has-*` has broad but not universal browser support (Baseline 2023+).
- `hover:` only applies on devices that support hover; `hoverOnlyWhenSupported` is now the default in v4.

### Annotated Code Example: Interactive Form with Peer Variants

```jsx
export default function LoginForm() {
  return (
    <div className="max-w-sm mx-auto p-6 bg-white rounded-lg shadow-md">
      <h2 className="text-xl font-bold mb-4">Sign In</h2>

      {/* Email field with peer-invalid */}
      <div className="mb-4">
        <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">
          Email
        </label>
        <input
          id="email"
          type="email"
          required
          className="peer block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 invalid:border-red-500 invalid:text-red-500"
          placeholder="you@example.com"
        />
        <p className="peer-invalid:block hidden mt-1 text-sm text-red-500">
          Please enter a valid email address.
        </p>
      </div>

      {/* Password field with group-hover on the button */}
      <div className="mb-4">
        <label htmlFor="password" className="block text-sm font-medium text-gray-700 mb-1">
          Password
        </label>
        <input
          id="password"
          type="password"
          required
          className="block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
        />
      </div>

      {/* Submit button with group-hover from the form container */}
      <div className="group">
        <button
          type="submit"
          className="w-full bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded transition-all group-hover:shadow-lg"
        >
          Sign In
        </button>
        <p className="text-center text-xs text-gray-400 mt-2 group-hover:text-gray-600 transition-colors">
          Secured by SSL
        </p>
      </div>
    </div>
  );
}
```

**Expected Output:** A login form with email and password fields. When the email field is invalid (e.g., the user types "test" without `@`), a red error message appears below it. When the user hovers over the submit button's container, the button gets a shadow and the "Secured by SSL" text darkens.

**Why This Output Occurs:** `peer` on the email input enables `peer-invalid:` on the error message, which is hidden by default and shown when the input is invalid. `group` on the button container enables `group-hover:shadow-lg` on the button and `group-hover:text-gray-600` on the text.

### Real-World Cases

- **Interactive cards:** Hovering a card highlights its title and lifts its shadow (`group-hover`).
- **Form validation:** Showing error messages when an input is invalid (`peer-invalid`).
- **Checkbox toggles:** Styling a label when a checkbox is checked (`peer-checked`).
- **Focus rings:** Applying focus styles to a parent when a child input is focused (`has-[:focus]`).
- **Navigation menus:** Highlighting an entire dropdown item when hovering over its icon (`group-hover`).

### References

- Steve Kinney – Group and Peer Modifiers: https://stevekinney.com/courses/tailwind/group-and-peer-modifiers
- Steve Kinney – State Variants: https://stevekinney.com/courses/tailwind/state-variants
- Tailwind CSS – Hover, Focus, and Other States: https://tailwindcss.com/docs/hover-focus-and-other-states
- Tailwind CSS v4 – Composable Variants: https://raw.githubusercontent.com/stevekinney/stevekinney.net/refs/heads/main/courses/tailwind/tailwind-4.md

---

## Core Concept 5: Design-Token Integration — Themes, Fluid Spacing, and Color Palettes

### Definitions

**Core Definition:** Design-token integration in Tailwind v4 is the practice of defining design tokens (colors, spacing, typography, breakpoints) directly in CSS via the `@theme` directive, where they are automatically exposed as CSS custom properties and generate corresponding utility classes.

**Technical Definition:** The `@theme` directive allows you to define theme values directly in CSS files. Theme values are organized into namespaces using CSS custom properties: `--color-*` for colors, `--spacing-*` for spacing, `--font-size-*` for typography, `--breakpoint-*` for responsive breakpoints, and `--container-*` for container queries. Values can have sub-properties using the `--` separator (e.g., `--font-size-xs--line-height`). The `@theme reference` mode defines values available to configuration but not emitted as CSS variables, useful for plugin development and keeping generated CSS smaller. Setting a namespace to `initial` clears all values in that namespace (e.g., `--color-*: initial` removes all default colors). Tailwind v4 supports any integer spacing value dynamically from the `--spacing` variable (e.g., `w-17`, `p-29`).

**Beginner-Friendly Explanation:** Design tokens are the named values that define your brand's look: "brand blue," "large padding," "small text." In v3, you defined these in a JavaScript config file. In v4, you define them directly in CSS using `@theme`. This means your design tokens are native CSS variables—they work with media queries, CSS functions, and even JavaScript. And because Tailwind generates utilities from them, every token automatically gets classes like `bg-brand-blue`, `p-large`, and `text-small`.

### Purposes

- To define design tokens in CSS instead of JavaScript configuration.
- To automatically generate utility classes from theme values.
- To expose design tokens as native CSS custom properties for runtime theming.
- To customize colors, spacing, typography, breakpoints, and aspect ratios.
- To use fluid spacing and typography scales via dynamic spacing values.
- To clear default theme values for complete control over the design system.

### Syntax Rules and Structure

**Basic Theme Definition:**
```css
@import "tailwindcss";

@theme {
  /* Colors */
  --color-primary: #3b82f6;
  --color-secondary: #8b5cf6;
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;

  /* Spacing */
  --spacing-18: 4.5rem;
  --spacing-128: 32rem;

  /* Typography */
  --font-size-2xs: 0.625rem;
  --font-size-2xs--line-height: 0.75rem;

  /* Breakpoints */
  --breakpoint-3xl: 120rem;

  /* Aspect ratios */
  --aspect-retro: 4 / 3;
}
```

**Component Breakdown:**
- `--color-primary`: Generates `bg-primary`, `text-primary`, `border-primary`, etc.
- `--color-primary-50`: Generates `bg-primary-50`, `text-primary-50`, etc.
- `--spacing-18`: Generates `p-18`, `m-18`, `gap-18`, `w-18`, etc.
- `--font-size-2xs` + `--font-size-2xs--line-height`: Generates `text-2xs` with both font-size and line-height.
- `--breakpoint-3xl`: Creates a `3xl:` responsive variant at 120rem.
- `--aspect-retro`: Generates `aspect-retro`.

**Dynamic Spacing (Any Integer):**
```html
<!-- v4 supports any integer spacing value dynamically -->
<div class="w-17 p-29 gap-13">...</div>
<!-- Generates: width: calc(var(--spacing) * 17); etc. -->
```

**Component Breakdown:**
- `w-17`: `width: calc(var(--spacing) * 17)` — no need to define `--spacing-17`.
- The default `--spacing` is `0.25rem`, so `w-17` = `4.25rem`.
- Change `--spacing` in `@theme` to adjust the base unit for the entire scale.

**Reference Mode (No CSS Variables Emitted):**
```css
@theme reference {
  --color-brand: #3b82f6;
  --spacing-custom: 2.5rem;
}
```

**Component Breakdown:**
- Values are available to plugins and JavaScript but are not emitted as CSS variables.
- Useful for keeping the generated CSS smaller.

**Clearing Theme Values:**
```css
@theme {
  /* Remove all default colors */
  --color-*: initial;

  /* Remove a specific color */
  --color-red: initial;

  /* Remove all theme values */
  --*: initial;
}
```

**Component Breakdown:**
- `--color-*: initial`: Removes all color tokens, leaving only your custom ones.
- `--*: initial`: Removes everything; you must define all tokens from scratch.

**Fluid Spacing with `clamp()`:**
```css
@theme {
  --spacing-fluid-sm: clamp(0.5rem, 1vw + 0.25rem, 1rem);
  --spacing-fluid-lg: clamp(1rem, 2vw + 0.5rem, 2rem);
}
```

**Component Breakdown:**
- `clamp()` creates a fluid value that scales between a minimum and maximum.
- `1vw + 0.25rem` uses viewport width for fluid scaling.
- Generates `p-fluid-sm`, `m-fluid-lg`, etc.

**Syntax Rules:**
- Define theme values inside `@theme { ... }` in your CSS file.
- Use namespaced custom properties: `--color-*`, `--spacing-*`, `--font-size-*`, `--breakpoint-*`, `--container-*`.
- Use `--` for sub-properties (e.g., `--font-size-xs--line-height`).
- Use `@theme reference` for values that should not be emitted as CSS variables.
- Use `--*: initial` to clear all default theme values.
- Use `calc()` and `clamp()` for fluid scales.
- Any integer spacing value works dynamically without explicit definition.

**Constraints and Limitations:**
- Clearing all theme values (`--*: initial`) removes default utilities like `text-sm`, `bg-blue-500`; you must redefine everything.
- `@theme` values are processed at build time; runtime CSS variables (e.g., `var(--accent-9)` from a theme switcher) require `@theme inline`.
- Sub-properties use a double-dash separator, which is unusual and must be exact.
- `@theme reference` values are not available as CSS variables at runtime.

### Annotated Code Example: Complete Design System Theme

```css
/* src/styles/tailwind.css */
@import "tailwindcss";

@theme {
  /* --- Brand Colors --- */
  --color-brand-50: #eef2ff;
  --color-brand-100: #e0e7ff;
  --color-brand-500: #6366f1;
  --color-brand-600: #4f46e5;
  --color-brand-700: #4338ca;

  /* --- Neutral Colors --- */
  --color-surface: #ffffff;
  --color-surface-muted: #f9fafb;
  --color-border: #e5e7eb;
  --color-text: #111827;
  --color-text-muted: #6b7280;

  /* --- Semantic Colors --- */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-danger: #ef4444;

  /* --- Spacing --- */
  --spacing-18: 4.5rem;
  --spacing-128: 32rem;

  /* --- Typography --- */
  --font-size-2xs: 0.625rem;
  --font-size-2xs--line-height: 0.75rem;
  --font-size-display: 3rem;
  --font-size-display--line-height: 1.1;
  --font-size-display--letter-spacing: -0.02em;

  /* --- Breakpoints --- */
  --breakpoint-3xl: 120rem;

  /* --- Aspect Ratios --- */
  --aspect-retro: 4 / 3;
  --aspect-portrait: 3 / 4;
}
```

```jsx
// App.tsx
export default function App() {
  return (
    <div className="min-h-screen bg-surface-muted p-4">
      <div className="max-w-4xl mx-auto">
        <h1 className="text-display text-text mb-4">Dashboard</h1>

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <div className="bg-surface rounded-lg border border-border p-6">
            <h2 className="text-lg font-semibold text-text mb-2">Revenue</h2>
            <p className="text-text-muted text-sm">$48,290</p>
            <span className="text-success text-2xs font-medium">+12.5%</span>
          </div>

          <div className="bg-surface rounded-lg border border-border p-6">
            <h2 className="text-lg font-semibold text-text mb-2">Users</h2>
            <p className="text-text-muted text-sm">2,847</p>
            <span className="text-warning text-2xs font-medium">+3.2%</span>
          </div>

          <div className="bg-surface rounded-lg border border-border p-6">
            <h2 className="text-lg font-semibold text-text mb-2">Churn</h2>
            <p className="text-text-muted text-sm">1.8%</p>
            <span className="text-danger text-2xs font-medium">-0.4%</span>
          </div>
        </div>

        {/* Aspect ratio from theme */}
        <div className="mt-6 aspect-retro bg-surface border border-border rounded-lg">
          4:3 preview area
        </div>
      </div>
    </div>
  );
}
```

**Expected Output:** A dashboard with a large display heading, a responsive 3-column grid of metric cards, and a 4:3 aspect-ratio preview area. All colors, spacing, typography, and the aspect ratio come from the `@theme` tokens.

**Why This Output Occurs:** Each `--color-*` token generates utilities like `bg-surface`, `text-text`, `border-border`, `text-success`, etc. The `--font-size-display` token generates `text-display` with its line-height and letter-spacing sub-properties. The `--aspect-retro` token generates `aspect-retro`. The `--breakpoint-3xl` token creates a `3xl:` variant.

### Real-World Cases

- **Multi-brand theming:** Different brands define different `--color-*` values; the same components work for all.
- **Dark mode:** `[data-theme="dark"]` overrides `--color-*` tokens; components reference `var(--color-surface)` and update automatically.
- **Fluid typography:** `clamp()`-based font sizes that scale smoothly across viewports.
- **Design system documentation:** Storybook reads the `@theme` tokens to document the design system.
- **White-label apps:** Tenants customise colours and spacing by overriding theme tokens.

### References

- Tailwind CSS – Theme Customization: https://mintlify.wiki/tailwindlabs/tailwindcss/customization/theme
- Tailwind CSS – Advanced Theming: https://mintlify.wiki/tailwindlabs/tailwindcss/customization/advanced-theming
- Steve Kinney – Theme Customization: https://stevekinney.com/courses/tailwind/theme-customization
- Tailwind CSS – Custom Spacing Values (Stack Overflow): https://stackoverflow.com/questions/79580716/custom-spacing-values-in-tailwind-css-v4

---

## Core Concept 6: Production Organization — Class Sorting and Conflict Resolution

### Definitions

**Core Definition:** Class sorting is the automated ordering of Tailwind utility classes for consistency and readability, while conflict resolution is the programmatic merging of class lists so that later classes override earlier ones without duplicates.

**Technical Definition:** The official `prettier-plugin-tailwindcss` automatically sorts classes following Tailwind's recommended order. It works with custom configurations and integrates with any editor that supports Prettier. The plugin sorts classes by property groups (layout, spacing, typography, colors, effects, variants), ensuring a consistent order across the codebase. `tailwind-merge` is a separate library that resolves conflicts between Tailwind classes: given `twMerge('p-5 p-2 p-4')`, it returns `'p-4'` (last conflicting class wins). It handles shorthand vs. specific conflicts (`p-3 px-5` keeps both), non-trivial conflicts (`inset-x-px -inset-1` → `-inset-1`), modifiers (`hover:p-2 hover:p-4` → `hover:p-4`), arbitrary values, and arbitrary properties. `tailwind-merge` does not resolve conflicts between arbitrary properties and their matching Tailwind classes to keep the bundle size small.

**Beginner-Friendly Explanation:** Class sorting is like arranging your bookshelf by genre and author—it doesn't change what's on the shelf, but it makes everything easier to find. `tailwind-merge` is like a smart editor that removes contradictory instructions: if you say "make this red" and then "make this blue," it keeps only "blue." This is essential for component libraries where consumers pass `className` props that should override the component's default styles.

### Purposes

- **Class sorting:** To enforce a consistent class order across the codebase.
- **Class sorting:** To improve readability and reduce merge conflicts in version control.
- **`tailwind-merge`:** To resolve conflicts when multiple classes target the same CSS property.
- **`tailwind-merge`:** To allow consumers of components to override default styles via `className` props.
- **`tailwind-merge`:** To handle modifiers, arbitrary values, and stacked variants correctly.

### Syntax Rules and Structure

**Installing the Prettier Plugin:**
```bash
npm install -D prettier prettier-plugin-tailwindcss
```

```json
// .prettierrc
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

**Before and After Sorting:**
```html
<!-- Before -->
<button class="text-white px-4 sm:px-8 py-2 sm:py-3 bg-sky-700 hover:bg-sky-800">...</button>

<!-- After -->
<button class="bg-sky-700 px-4 py-2 text-white hover:bg-sky-800 sm:px-8 sm:py-3">...</button>
```

**Component Breakdown:**
- Classes are sorted by property group: layout → spacing → typography → colors → effects → variants.
- Responsive variants (`sm:`) are grouped together and placed after the base classes.
- Hover variants (`hover:`) are placed after the base classes but before responsive variants.

**Installing `tailwind-merge`:**
```bash
npm install tailwind-merge
```

```typescript
import { twMerge } from 'tailwind-merge';

// Last conflicting class wins
twMerge('p-5 p-2 p-4'); // → 'p-4'

// Allows refinements
twMerge('p-3 px-5'); // → 'p-3 px-5'
twMerge('inset-x-4 right-4'); // → 'inset-x-4 right-4'

// Resolves non-trivial conflicts
twMerge('inset-x-px -inset-1'); // → '-inset-1'
twMerge('bottom-auto inset-y-6'); // → 'inset-y-6'
twMerge('inline block'); // → 'block'

// Supports modifiers
twMerge('p-2 hover:p-4'); // → 'p-2 hover:p-4'
twMerge('hover:p-2 hover:p-4'); // → 'hover:p-4'
twMerge('hover:focus:p-2 focus:hover:p-4'); // → 'focus:hover:p-4'

// Supports arbitrary values
twMerge('bg-black bg-(--my-color) bg-[color:var(--mystery-var)]');
// → 'bg-[color:var(--mystery-var)]'

// Supports arbitrary properties
twMerge('[mask-type:luminance] [mask-type:alpha]');
// → '[mask-type:alpha]'
```

**Using `twMerge` in a React Component:**
```tsx
import { twMerge } from 'tailwind-merge';

function Button({ className, variant = 'primary', size = 'md', ...props }) {
  const baseClasses = 'inline-flex items-center justify-center font-semibold rounded transition-colors';

  const variantClasses = {
    primary: 'bg-blue-500 hover:bg-blue-700 text-white',
    secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-800',
    danger: 'bg-red-500 hover:bg-red-700 text-white',
  };

  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  };

  const merged = twMerge(
    baseClasses,
    variantClasses[variant],
    sizeClasses[size],
    className // Consumer's className comes last — overrides defaults
  );

  return <button className={merged} {...props} />;
}

// Usage: consumer overrides the default background
<Button variant="primary" className="bg-purple-500">
  Custom Purple
</Button>
// → 'bg-purple-500' wins over 'bg-blue-500'
```

**Component Breakdown:**
- `twMerge` receives the base classes, variant classes, size classes, and the consumer's `className`.
- The consumer's `className` is last, so it wins any conflicts.
- Without `twMerge`, the element would have both `bg-blue-500` and `bg-purple-500`, and the winner would depend on CSS source order (unpredictable).

**Syntax Rules:**
- Install `prettier-plugin-tailwindcss` as a dev dependency and add it to `.prettierrc`.
- Run Prettier on save or via a `format` script to sort classes automatically.
- Install `tailwind-merge` for component libraries where `className` overrides are expected.
- Use `twMerge(base, conditional, className)` — put the consumer's `className` last.
- `tailwind-merge` follows "last conflicting class wins".
- It handles modifiers, arbitrary values, and arbitrary properties.
- For ambiguous arbitrary values, use CSS data type labels (e.g., `text-[length:...]`).

**Constraints and Limitations:**
- `tailwind-merge` does not resolve conflicts between arbitrary properties and their matching Tailwind classes.
- It adds a runtime cost (small but non-zero) to every component that uses it.
- The Prettier plugin requires Prettier to be configured; it does not work standalone.
- Class sorting does not fix actual style conflicts—it only orders them consistently.
- `tailwind-merge` needs to be updated when Tailwind adds new utility groups.

### Annotated Code Example: Component with `twMerge` and Prettier Sorting

```tsx
// Card.tsx
import { twMerge } from 'tailwind-merge';

interface CardProps {
  className?: string;
  variant?: 'elevated' | 'outlined';
  children: React.ReactNode;
}

export function Card({ className, variant = 'elevated', children }: CardProps) {
  const base = 'rounded-lg p-6 transition-shadow';

  const variants = {
    elevated: 'bg-white shadow-md hover:shadow-xl',
    outlined: 'bg-white border border-gray-200',
  };

  return (
    <div className={twMerge(base, variants[variant], className)}>
      {children}
    </div>
  );
}

// Usage
<Card variant="elevated" className="bg-blue-50 border-2 border-blue-200">
  <h2 className="text-lg font-semibold">Revenue</h2>
  <p className="text-gray-600 text-sm">$48,290</p>
</Card>
```

**Expected Output:** A card with `bg-blue-50` (the consumer's override) instead of `bg-white`, plus a blue border. The `twMerge` call resolves the conflict: `bg-blue-50` comes after `bg-white`, so it wins. The `shadow-md` from the `elevated` variant is preserved because there is no conflict.

**Why This Output Occurs:** `twMerge` recognises that `bg-white` and `bg-blue-50` both set `background-color` and keeps only the last one. It recognises that `border-2 border-blue-200` does not conflict with `shadow-md`, so both are kept. Without `twMerge`, the element would have both `bg-white` and `bg-blue-50`, and the winner would depend on the order of classes in the generated CSS.

### Real-World Cases

- **Design systems (shadcn/ui):** Every component uses `twMerge` to allow consumer overrides.
- **Monorepos:** The Prettier plugin ensures consistent class order across all packages.
- **Team collaboration:** Sorted classes reduce merge conflicts and make code reviews easier.
- **Component libraries:** `twMerge` is essential for `className` prop APIs.
- **Conditional styling:** `twMerge` resolves conflicts when multiple conditional classes are applied.

### References

- Tailwind CSS – Editor Setup (Automatic Class Sorting): https://v3.tailwindcss.com/docs/editor-setup
- tailwind-merge – Features: https://github.com/dcastil/tailwind-merge/blob/main/docs/features.md
- tailwind-merge – GitHub: https://github.com/dcastil/tailwind-merge
- Prettier Plugin Tailwind CSS – npm: https://www.npmjs.com/package/prettier-plugin-tailwindcss
- Steve Kinney – Tailwind CSS 4: https://raw.githubusercontent.com/stevekinney/stevekinney.net/refs/heads/main/courses/tailwind/tailwind-4.md

---

## Comparison and Decision Guidance

| Concern | Recommended Approach | When to Use | Key Risk |
|---|---|---|---|
| **Installation** | `@tailwindcss/vite` | Vite projects (React, Vue, Svelte) | Requires Node 20+ |
| **Installation** | `@tailwindcss/postcss` | Next.js, Webpack, PostCSS setups | Slower than Vite plugin |
| **Compilation** | Oxide engine (built-in) | All v4 projects | Requires modern browser support |
| **Bundle optimization** | JIT (always on in v4) | All projects | Dynamic class names are invisible |
| **Responsive** | Breakpoint prefixes (`md:`, `lg:`) | Page-level layout | Viewport-based, not component-based |
| **Aspect ratio** | `aspect-square`, `aspect-video`, `aspect-[...]` | Images, videos, containers | Requires defined width |
| **State variants** | `hover:`, `focus:`, `group-*`, `peer-*`, `has-*` | Interactive components | `peer` only works on subsequent siblings |
| **Design tokens** | `@theme` directive | All projects | `--*: initial` removes all defaults |
| **Fluid spacing** | `clamp()` in `@theme` | Responsive typography and spacing | Browser support for `clamp()` |
| **Class sorting** | `prettier-plugin-tailwindcss` | All projects | Requires Prettier |
| **Conflict resolution** | `tailwind-merge` | Component libraries | Runtime cost per component |

**Decision Guidance:**
- **Start with the Vite plugin** for the fastest installation and build experience.
- **Use `@theme`** for all design tokens; it is the v4-native way to configure the framework.
- **Use `aspect-square` / `aspect-video`** for common ratios; use `aspect-[...]` for custom ones.
- **Use `group` and `peer`** for interactive components; use `has-*` for child-to-parent effects.
- **Use `clamp()`** for fluid typography and spacing that scales smoothly across viewports.
- **Install `prettier-plugin-tailwindcss`** on day one for consistent class ordering.
- **Use `tailwind-merge`** in any component that accepts a `className` prop.
- **Use named groups** (`group/name`) when nesting interactive elements to avoid ambiguous targeting.
- **Use `@source`** to register component library paths in monorepos.
- **Use the upgrade tool** (`npx @tailwindcss/upgrade@next`) when migrating from v3.

---

## References

- Tailwind CSS – Introduction: https://mintlify.wiki/tailwindlabs/tailwindcss/introduction
- Tailwind CSS – Installation: https://mintlify.wiki/tailwindlabs/tailwindcss/installation
- Tailwind CSS – Quickstart Guide: https://mintlify.wiki/tailwindlabs/tailwindcss/quickstart
- Tailwind CSS – Upgrade Guide: https://tailwindcss.com/docs/upgrade-guide
- Tailwind CSS – Theme Customization: https://mintlify.wiki/tailwindlabs/tailwindcss/customization/theme
- Tailwind CSS – Advanced Theming: https://mintlify.wiki/tailwindlabs/tailwindcss/customization/advanced-theming
- Tailwind CSS – aspect-ratio: https://tailwindcss.com/docs/aspect-ratio
- Tailwind CSS – Hover, Focus, and Other States: https://tailwindcss.com/docs/hover-focus-and-other-states
- Tailwind CSS – Editor Setup (Automatic Class Sorting): https://v3.tailwindcss.com/docs/editor-setup
- Steve Kinney – Tailwind Oxide: https://stevekinney.com/courses/tailwind/tailwind-oxide
- Steve Kinney – Tailwind CSS 4: https://raw.githubusercontent.com/stevekinney/stevekinney.net/refs/heads/main/courses/tailwind/tailwind-4.md
- Steve Kinney – Group and Peer Modifiers: https://stevekinney.com/courses/tailwind/group-and-peer-modifiers
- Steve Kinney – State Variants: https://stevekinney.com/courses/tailwind/state-variants
- Steve Kinney – Theme Customization: https://stevekinney.com/courses/tailwind/theme-customization
- Tailwind CSS v4 Architecture (ZlatanCN): https://raw.githubusercontent.com/ZlatanCN/personal-website/refs/heads/master/data/blog/css/tailwind-css-v4-architecture.mdx
- tailwind-merge – Features: https://github.com/dcastil/tailwind-merge/blob/main/docs/features.md
- tailwind-merge – GitHub: https://github.com/dcastil/tailwind-merge
- voku/agent-skills – Responsive Aspect Ratio: https://github.com/voku/agent-skills
- Custom Spacing Values in Tailwind v4 (Stack Overflow): https://stackoverflow.com/questions/79580716/custom-spacing-values-in-tailwind-css-v4
- Tailwind CSS v4 – Architecture, Features, and Performance Upgrades: https://dev.to