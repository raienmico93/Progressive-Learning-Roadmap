# CSS-in-JS — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** CSS-in-JS is a styling approach where CSS is written directly in JavaScript or TypeScript files rather than in separate stylesheets, allowing styles to be scoped to individual components and to depend on component props and state.

**Technical Definition:** CSS-in-JS encompasses a family of libraries and techniques in which CSS rules are authored as JavaScript values — either as tagged template literals (e.g., `` styled.div`color: red` ``) or as plain style objects (e.g., `css={{ color: 'red' }}`). These styles are then either **interpreted at runtime** and injected into the DOM as `<style>` tags (runtime CSS-in-JS), or **extracted at build time** into static CSS files (compile-time or zero-runtime CSS-in-JS). Runtime CSS-in-JS libraries like styled-components and Emotion rely on React context and DOM APIs to inject styles during render, which makes them **incompatible with React Server Components (RSC)** without a `'use client'` boundary. Compile-time libraries like Linaria and Vanilla Extract avoid this limitation by generating static CSS during the build step.

**Beginner-Friendly Explanation:** CSS-in-JS means writing your styles in the same file as your component, using JavaScript. Instead of a `.css` file, you write something like `styled.button\`background: blue\`` or `css={{ color: 'red' }}`. The library turns this into actual CSS for you. Some libraries do this while your app is running (runtime), and others do it when you build your app (compile-time). The big advantage is that styles can depend on your component's data — like making a button red when it's in an "error" state. The big catch is that runtime libraries do not work with React's newer Server Components, so the ecosystem has been shifting toward compile-time solutions.

### Key Characteristics

- **Component-scoped by default:** Styles are tied to components, preventing global collisions without naming conventions.
- **Prop-driven dynamism:** Styles can be computed from props, state, and theme values at render time (runtime libraries) or via CSS variables (compile-time libraries).
- **Two architectural camps:** **Runtime** (styled-components, Emotion) and **compile-time/zero-runtime** (Linaria, Vanilla Extract, Stitches by design).
- **RSC incompatibility (runtime):** Runtime CSS-in-JS cannot run inside React Server Components; it requires a client boundary and an SSR style registry.
- **Maintenance mode warning:** styled-components entered maintenance mode in 2025; Stitches is no longer maintained.
- **Ecosystem shift:** The React ecosystem has moved toward build-time CSS (CSS Modules, Tailwind, zero-runtime CSS-in-JS) for server-rendered and performance-sensitive applications.

### Prerequisites

- JavaScript/TypeScript fundamentals, including template literals and objects.
- React function components, props, and JSX.
- Understanding of CSS selectors and the cascade.
- Familiarity with the build tool used by your project (Vite, webpack, Next.js).

### Related Programming Areas

- **CSS Modules** — build-time scoped CSS, often the recommended default over runtime CSS-in-JS.
- **Utility-first CSS** — Tailwind and similar frameworks, which dominate the current ecosystem.
- **React Server Components** — the architectural shift that has made runtime CSS-in-JS problematic.
- **Design tokens and theming** — CSS custom properties are the foundation of modern theming strategies.

---

## 1. CSS-in-JS

### Definitions

**Core Definition:** CSS-in-JS is a styling methodology that colocates CSS with component logic in JavaScript or TypeScript files.

**Technical Definition:** CSS-in-JS libraries parse styles authored in JavaScript and produce CSS rules that target component-scoped class names. In runtime libraries, this happens during render: the library computes styles, generates a class name, and injects a `<style>` tag into the document head. In compile-time libraries, a Babel plugin or bundler integration extracts the CSS at build time, emitting static stylesheets and unique class names with zero runtime JavaScript. The key architectural distinction is **when** style computation happens — at runtime on the client, or at build time before the bundle ships.

**Beginner-Friendly Explanation:** CSS-in-JS lets you write CSS inside your JavaScript. Some libraries do the work while your app runs, and others do it before your app is deployed. Both let you keep styles close to the component they belong to.

### Purposes

- To colocate styles with the components they style, improving cohesion.
- To enable prop-driven dynamic styling without class name gymnastics.
- To provide automatic scoping without naming conventions like BEM.
- To support theming and design tokens programmatically.

### Syntax Rules and Structure

**Tagged Template Literal (styled-components, Emotion, Linaria):**

```jsx
const Button = styled.button`
  background: blue;
  color: white;
  padding: 8px 16px;
`;
```

**Object Syntax (Emotion, Stitches):**

```jsx
<button css={{ background: 'blue', color: 'white' }}>
```

**Syntax Rules:**

1. Styles are authored in JavaScript/TypeScript files.
2. Runtime libraries require a client-side execution environment.
3. Compile-time libraries extract CSS during the build step.
4. The choice between runtime and compile-time determines RSC compatibility.

### Annotated Code Examples

**Runtime CSS-in-JS (styled-components):**

```jsx
import styled from 'styled-components';

const Button = styled.button`
  background: ${(props) => (props.primary ? 'blue' : 'gray')};
  color: white;
  padding: 8px 16px;
`;

function App() {
  return <Button primary>Click</Button>;
}
```

**Expected Output:** A blue button. At runtime, styled-components computes the background from the `primary` prop, generates a class name, and injects a `<style>` tag.

**Why this output:** The tagged template literal is evaluated at render time. The prop interpolation produces `'blue'`, which is included in the generated CSS rule.

**Compile-Time CSS-in-JS (Linaria):**

```jsx
import { styled } from '@linaria/react';

const Button = styled.button`
  background: blue;
  color: white;
  padding: 8px 16px;
`;

function App() {
  return <Button>Click</Button>;
}
```

**Expected Output:** A blue button. The CSS is extracted to a static `.css` file at build time; no style computation happens at runtime.

**Why this output:** Linaria’s Babel plugin extracts the CSS rules during the build and generates unique class names. The runtime simply applies the pre-generated class name.

### Real-World Cases

- **Design systems with dynamic theming:** Components whose styles depend on runtime theme values.
- **Rapid prototyping:** Colocating styles with components speeds up iteration.
- **Legacy client-rendered SPAs:** Applications that predate React Server Components and have heavy prop-driven styling needs.

### References

- Next.js — Styling: CSS-in-JS — https://nextjs.org/docs/app/building-your-application/styling/css-in-js
- pkgpulse — Emotion vs styled-components in 2026 — https://www.pkgpulse.com/guides/emotion-vs-styled-components-2026
- GitHub — CSS-in-JS and styling status — https://github.com/saschb2b/skills/blob/main/skills/engineering/javascript-ecosystem/references/ui/css-in-js.md

---

## 2. Styled Components

### Definitions

**Core Definition:** styled-components is a runtime CSS-in-JS library that lets you style components using tagged template literals and prop-based interpolation.

**Technical Definition:** styled-components is a CSS-in-JS library that works with the component-level styling paradigm in React. It provides a `styled` function that creates React components with attached styles, authored as tagged template literals. Styles can interpolate props, enabling dynamic styling. The library generates unique class names at runtime and injects styles via the `useInsertionEffect`-based style injection mechanism, which requires a client-side runtime. **As of 2025, styled-components is in maintenance mode**: it still receives fixes and v6.4.x added `createTheme` CSS-variable theming and RSC-compatible output, but it is no longer the default choice for new applications.

**Beginner-Friendly Explanation:** styled-components lets you write CSS in a special syntax that looks like a function call. You write `` styled.button`color: blue` `` and it creates a button component with that style. You can use props to change the styles. But it needs to run in the browser, so it does not work in React Server Components.

### Purposes

- To create component-scoped styles without naming collisions.
- To enable prop-driven dynamic styling with a clean API.
- To provide automatic vendor prefixing and scoping.
- To support theming via `ThemeProvider`.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import styled from 'styled-components';

const StyledComponent = styled.element`
  property: value;
  property: ${(props) => props.dynamicValue};
`;
```

**Syntax Rules:**

1. `styled.element` creates a styled DOM element or component.
2. Tagged template literals contain CSS rules.
3. `${(props) => ...}` interpolates prop values into the CSS.
4. `ThemeProvider` supplies theme values via context.
5. Requires a client-side runtime and an SSR style registry for server rendering.

**Constraints and Limitations:**

- **Not compatible with React Server Components** without `'use client'`.
- Adds runtime bundle weight and style computation cost.
- **In maintenance mode** since 2025; not the growth path.
- Prop-based interpolation runs on every render, which can impact performance.

### Annotated Code Examples

```jsx
import styled from 'styled-components';

const Button = styled.button`
  background: ${(props) => (props.primary ? 'blue' : 'gray')};
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
`;

function App() {
  return (
    <>
      <Button primary>Primary</Button>
      <Button>Secondary</Button>
    </>
  );
}
```

**Expected Output:** A blue “Primary” button and a gray “Secondary” button.

**Why this output:** The `primary` prop is `true` for the first button, so the background interpolation returns `'blue'`. For the second button, it returns `'gray'`. The library generates unique class names and injects the corresponding styles at runtime.

### Real-World Cases

- **Legacy client-rendered React applications:** SPAs with existing styled-components codebases.
- **Design systems with heavy prop-driven styling:** Where runtime dynamism is a genuine requirement.
- **Rapid prototyping:** The concise API speeds up initial development.

### References

- styled-components — FAQs — https://styled-components.com/docs/faqs
- GitHub — styled-components maintenance mode discussion — https://github.com/styled-components/styled-components/discussions/5568
- pkgpulse — Emotion vs styled-components in 2026 — https://www.pkgpulse.com/guides/emotion-vs-styled-components-2026

---

## 3. Emotion

### Definitions

**Core Definition:** Emotion is a performant and flexible CSS-in-JS library that supports both string and object styles, with strong style composition capabilities.

**Technical Definition:** Emotion is a CSS-in-JS library designed for high-performance style composition. It provides two primary APIs: the `css` prop (or `css` function) for object or template-literal styles, and `styled` for creating styled components. Emotion uses intelligent style caching to minimise redundant style injection and offers excellent server-side rendering support by streaming style tags just before the HTML chunks that need them. Like styled-components, Emotion is a **runtime** library and is **not supported in React Server Components**; it requires a `'use client'` boundary and an SSR style registry. Emotion is the styling engine behind Material UI and has a slightly smaller bundle size (~7KB gzip) and competitive performance compared to styled-components.

**Beginner-Friendly Explanation:** Emotion is another CSS-in-JS library, similar to styled-components but often faster and more flexible. It lets you write styles as objects or strings, and it caches styles so it does not repeat work. It still runs in the browser, so it does not work in Server Components.

### Purposes

- To provide a high-performance runtime CSS-in-JS solution with style caching.
- To support both object and template-literal style syntaxes.
- To enable strong style composition and theming.
- To serve as the styling engine for Material UI.

### Syntax Rules and Structure

**Object Syntax (`css` prop):**

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';

<div css={css({ color: 'red', fontWeight: 'bold' })} />
```

**Styled API:**

```jsx
import styled from '@emotion/styled';

const Button = styled.button`
  background: blue;
  color: white;
`;
```

**Syntax Rules:**

1. The `css` prop requires the `@jsxImportSource @emotion/react` pragma.
2. `styled` provides a tagged-template API similar to styled-components.
3. Styles can be composed using the `css` function.
4. Requires a client-side runtime; not RSC-compatible.

**Constraints and Limitations:**

- **Not compatible with React Server Components** without `'use client'`.
- Runtime bundle weight (~7KB gzip).
- The Emotion team’s exploration of zero-runtime Emotion saw minimal adoption.

### Annotated Code Examples

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';

function Alert({ type, children }) {
  return (
    <div
      css={css({
        padding: 12,
        borderRadius: 4,
        background: type === 'error' ? '#fee2e2' : '#e0f2fe',
        color: type === 'error' ? '#991b1b' : '#075985',
      })}
    >
      {children}
    </div>
  );
}
```

**Expected Output:** An alert box with a red background for `type="error"` and a blue background for other types.

**Why this output:** The `css` function receives an object whose values are computed from the `type` prop. Emotion generates a class name, caches the style, and injects it at runtime.

### Real-World Cases

- **Material UI applications:** Emotion is the default styling engine for MUI.
- **Design systems requiring fine-grained style composition:** The `css` function supports composition and merging.
- **Client-rendered applications with complex theming needs.**

### References

- Emotion — Introduction — https://emotion.sh/docs/introduction
- Emotion — Server-Side Rendering — https://emotion.sh/docs/ssr
- pkgpulse — Emotion vs styled-components in 2026 — https://www.pkgpulse.com/guides/emotion-vs-styled-components-2026

---

## 4. Linaria

### Definitions

**Core Definition:** Linaria is a zero-runtime CSS-in-JS library that extracts CSS to static files at build time, eliminating runtime style computation.

**Technical Definition:** Linaria lets you write CSS in JavaScript using tagged template literals, but a Babel plugin extracts the CSS rules to real CSS files during the build and generates unique class names. **No runtime is included** in the final bundle. Dynamic prop-based styles are supported through CSS custom properties: instead of recomputing styles at runtime, Linaria generates a static rule that reads a CSS variable, and the component sets that variable inline based on props. Linaria supports familiar CSS syntax with Sass-like nesting, source maps, stylelint, and optional CSS preprocessors. Because it is a compile-time library, it is **compatible with React Server Components**.

**Beginner-Friendly Explanation:** Linaria lets you write CSS in JavaScript, but it does not run in the browser. When you build your app, Linaria pulls out all the CSS into normal `.css` files. This means your app stays fast and works with Server Components. For dynamic styles, it uses CSS variables instead of recalculating styles on every render.

### Purposes

- To provide a zero-runtime CSS-in-JS experience with static CSS extraction.
- To enable prop-driven dynamic styles without runtime cost, using CSS variables.
- To support React Server Components.
- To offer a familiar CSS syntax with nesting and preprocessor support.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { styled } from '@linaria/react';

const Button = styled.button`
  background: blue;
  color: white;
  padding: 8px 16px;
`;
```

**Dynamic Styles with CSS Variables:**

```jsx
import { styled } from '@linaria/react';

const Button = styled.button`
  background: var(--button-bg);
`;

<Button style={{ '--button-bg': 'blue' }} />
```

**Syntax Rules:**

1. Styles are authored in tagged template literals.
2. A Babel plugin extracts CSS at build time.
3. Dynamic styles use CSS custom properties set via inline `style`.
4. No runtime JavaScript is included for styling.

**Constraints and Limitations:**

- Requires Babel plugin integration in the build pipeline.
- Dynamic styles are limited to what CSS variables can express (they cannot generate entirely new rules at runtime).
- Less suitable for applications that need to compute arbitrary styles from complex JavaScript logic at runtime.

### Annotated Code Examples

```jsx
import { styled } from '@linaria/react';

const Card = styled.div`
  padding: 16px;
  border-radius: 8px;
  border: 1px solid #ddd;
  background: ${(props) => props.highlighted ? '#fef9c3' : 'white'};
`;

function App() {
  return (
    <>
      <Card>Normal card</Card>
      <Card highlighted>Highlighted card</Card>
    </>
  );
}
```

**Expected Output:** A normal card with a white background and a highlighted card with a yellow background. The CSS is extracted to a static file at build time.

**Why this output:** Linaria’s Babel plugin extracts the CSS rules, including the prop-based interpolation, during the build. The generated CSS uses a class for each variant, and the component applies the correct class at runtime. No style computation happens in the browser.

### Real-World Cases

- **RSC-based applications:** Linaria’s compile-time extraction makes it compatible with React Server Components.
- **Performance-sensitive client applications:** Zero runtime overhead means smaller bundles and faster interactions.
- **Teams that prefer CSS syntax over utility classes but want zero runtime.**

### References

- Linaria — Zero-runtime CSS in JS — https://github.com/callstack/linaria
- npm — @linaria/server — https://www.npmjs.com/package/@linaria/server

---

## 5. Stitches

### Definitions

**Core Definition:** Stitches is a CSS-in-JS library with near-zero runtime, a fully-typed API, and first-class variant support — but it is no longer maintained.

**Technical Definition:** Stitches provides a `styled` function with a near-zero runtime by avoiding unnecessary prop interpolations at runtime. It offers a fully-typed API, built-in theming with CSS variables, and first-class variants (including compound variants and default variants). However, **Stitches is no longer actively maintained** as of June 2023, due to changes in the React ecosystem and maintainer availability. Its architecture was influential — it helped popularise CSS-variables-in-JS and inspired Vanilla Extract — but it should not be chosen for new projects.

**Beginner-Friendly Explanation:** Stitches was a fast CSS-in-JS library with really good TypeScript support and a clever way of handling variants. But it is not maintained anymore, so you should not start a new project with it.

### Purposes

- (Historical) To provide a near-zero-runtime CSS-in-JS library with fully-typed variants and theming.
- (Historical) To offer a best-in-class developer experience with TypeScript.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { styled } from '@stitches/react';

const Button = styled('button', {
  background: 'blue',
  color: 'white',
  variants: {
    color: {
      gray: { background: 'gainsboro' },
      blue: { background: 'dodgerblue' },
    },
  },
});
```

**Syntax Rules:**

1. Styles are written as configuration objects.
2. `variants` is a first-class concept with compound variants.
3. Theming uses CSS variables under the hood.
4. The API is fully typed in TypeScript.

**Constraints and Limitations:**

- **No longer maintained** since June 2023.
- Not recommended for new projects.
- Near-zero runtime, but not zero-runtime in the sense of Linaria or Vanilla Extract.

### Annotated Code Examples

```jsx
import { styled } from '@stitches/react';

const Button = styled('button', {
  padding: '8px 16px',
  borderRadius: 4,
  border: 'none',
  color: 'white',
  variants: {
    color: {
      primary: { background: 'blue' },
      secondary: { background: 'gray' },
    },
  },
});

function App() {
  return <Button color="primary">Click</Button>;
}
```

**Expected Output:** A blue button with the “primary” variant.

**Why this output:** Stitches resolves the variant at runtime with minimal overhead and applies the corresponding styles. The API is fully typed, so `color="primary"` is checked against the allowed variant values.

### Real-World Cases

- **Existing codebases:** Applications already built with Stitches.
- **Historical reference:** Understanding the evolution of near-zero-runtime CSS-in-JS.
- **Not recommended for new projects** due to lack of maintenance.

### References

- Stitches — Official site — https://stitches.dev
- GitHub — Stitches maintenance notice — https://github.com/stitchesjs/stitches

---

## 6. Vanilla Extract

### Definitions

**Core Definition:** Vanilla Extract is a zero-runtime CSS-in-TypeScript library that generates static CSS files at build time, using TypeScript as a preprocessor.

**Technical Definition:** Vanilla Extract lets you write styles in `.css.ts` files using TypeScript, with locally scoped class names, CSS variables, and themes. All code in these files is executed **during the build process**, not in the user’s browser, and the output is static CSS files. It provides type-safe styles via CSSType, a high-level theme system, and optional runtime APIs for development and dynamic theming. Because it is compile-time, it is **compatible with React Server Components**. Vanilla Extract is often described as “CSS Modules in TypeScript” with scoped CSS variables and a richer API.

**Beginner-Friendly Explanation:** Vanilla Extract lets you write your styles in TypeScript files. When you build your app, it turns those TypeScript files into normal CSS. Nothing runs in the browser, so it is fast and works with Server Components. You get TypeScript autocomplete and type checking for your styles.

### Purposes

- To provide zero-runtime, type-safe styling with TypeScript as the authoring language.
- To enable locally scoped class names and CSS variables without runtime cost.
- To support React Server Components.
- To offer a high-level theme system with multiple simultaneous themes.

### Syntax Rules and Structure

**General Syntax:**

```ts
// styles.css.ts
import { createTheme, style } from '@vanilla-extract/css';

export const [themeClass, vars] = createTheme({
  color: { brand: 'blue' },
  font: { body: 'arial' },
});

export const exampleStyle = style({
  backgroundColor: vars.color.brand,
  fontFamily: vars.font.body,
  color: 'white',
  padding: 10,
});
```

```jsx
import { themeClass, exampleStyle } from './styles.css.ts';

<section className={themeClass}>
  <h1 className={exampleStyle}>Hello world!</h1>
</section>
```

**Syntax Rules:**

1. Styles are written in `.css.ts` (or `.css.js`) files.
2. The code in these files runs at build time, not in the browser.
3. Class names and CSS variables are locally scoped.
4. `createTheme` generates a theme class and a variables object.
5. `style` creates a scoped class name.

**Constraints and Limitations:**

- Requires build-tool integration (Vite, webpack, Next.js plugin).
- All style logic must be expressible in TypeScript executed at build time.
- Dynamic runtime theming is optional and limited compared to runtime CSS-in-JS.

### Annotated Code Examples

```ts
// button.css.ts
import { style, createTheme } from '@vanilla-extract/css';

export const [themeClass, vars] = createTheme({
  color: { primary: 'blue', secondary: 'gray' },
});

export const button = style({
  padding: '8px 16px',
  border: 'none',
  borderRadius: 4,
  color: 'white',
  background: vars.color.primary,
  selectors: {
    '&:hover': { opacity: 0.9 },
  },
});
```

```jsx
import { themeClass, button } from './button.css.ts';

function App() {
  return (
    <div className={themeClass}>
      <button className={button}>Click</button>
    </div>
  );
}
```

**Expected Output:** A blue button with hover opacity. The styles are extracted to a static CSS file at build time.

**Why this output:** The `.css.ts` file is evaluated during the build. `style` generates a unique class name, and `createTheme` generates a theme class and CSS variables. The component applies the pre-generated class names. No styling JavaScript is shipped to the browser.

### Real-World Cases

- **RSC-based design systems:** Vanilla Extract’s zero-runtime nature makes it suitable for React Server Components.
- **TypeScript-heavy projects:** Type safety for styles is a core feature.
- **Performance-sensitive applications:** Static CSS extraction eliminates runtime style computation.

### References

- Vanilla Extract — Official site — https://vanilla-extract.style
- npm — @vanilla-extract/css — https://www.npmjs.com/package/@vanilla-extract/css
- Feature-Sliced Design — Vanilla Extract: The Perfect CSS Architecture? — https://feature-sliced.design

---

## 7. CSS-in-JS Performance

### Definitions

**Core Definition:** CSS-in-JS performance refers to the runtime and build-time costs associated with generating, injecting, and applying styles through JavaScript.

**Technical Definition:** Runtime CSS-in-JS libraries (styled-components, Emotion) execute style computation during render. This means that on every render where props or theme values change, the library must compute styles, generate class names, and inject or update `<style>` tags. This work runs on the main thread, contributing to render cost and potentially affecting Interaction to Next Paint (INP). Additionally, the styling engine itself adds to the JavaScript bundle (e.g., ~12KB for styled-components, ~7KB for Emotion). Compile-time libraries (Linaria, Vanilla Extract) eliminate these runtime costs entirely: styles are extracted at build time, and the browser only downloads and applies static CSS.

**Beginner-Friendly Explanation:** Runtime CSS-in-JS makes your app do extra work while it is running, because it has to calculate and insert styles on the fly. That can slow things down, especially on slower devices. Compile-time CSS-in-JS does all that work before your app is even deployed, so the browser just uses regular CSS.

### Purposes

- To understand the trade-off between dynamic styling and runtime performance.
- To choose the right CSS-in-JS approach for performance-sensitive applications.
- To recognise when runtime overhead is justified by genuine dynamism.

### Syntax Rules and Structure

**Performance Characteristics:**

| Approach | Runtime Cost | Bundle Cost | RSC Compatible |
|----------|-------------|-------------|----------------|
| Runtime CSS-in-JS (styled-components, Emotion) | Style computation + DOM injection on render | ~7–12KB | No (requires `'use client'`) |
| Compile-time CSS-in-JS (Linaria, Vanilla Extract) | None | Minimal | Yes |
| CSS Modules | None | None | Yes |
| Tailwind CSS | None | ~0KB runtime | Yes |

**Syntax Rules:**

1. Runtime CSS-in-JS adds main-thread work during render.
2. Compile-time CSS-in-JS moves all work to the build step.
3. The performance impact of runtime libraries is most significant in frequently re-rendering components and large component trees.

### Annotated Code Examples

**Runtime Overhead (styled-components):**

```jsx
const Button = styled.button`
  background: ${(props) => props.bg};
`;

// Every time bg changes, the interpolation runs and styles are regenerated
<Button bg={isActive ? 'blue' : 'gray'} />
```

**Compile-Time Overhead (Vanilla Extract):**

```ts
// button.css.ts — runs at BUILD time only
export const button = style({ background: vars.color.primary });
```

**Expected Output:** The runtime version computes styles on each render; the compile-time version ships static CSS.

**Why this output:** styled-components evaluates the interpolation at render time. Vanilla Extract’s `style` call runs during the build, producing a static class name that the browser applies without any JavaScript style computation.

### Real-World Cases

- **High-frequency interaction UIs:** Dashboards and real-time applications where INP matters.
- **Server-rendered applications:** Where runtime CSS-in-JS is incompatible with RSC.
- **Bundle-size-sensitive applications:** Where the styling engine’s bundle weight matters.

### References

- RavenClaude — Styling approach decision trees — https://github.com/mcorbett51090/RavenClaude/blob/main/plugins/frontend-engineering/knowledge/styling-and-bundle-decision-trees.md
- Telerik — Static Extraction in CSS-in-JS for Efficiency in React Apps — https://www.telerik.com/blogs/static-extraction-css-in-js
- pkgpulse — Emotion vs styled-components in 2026 — https://www.pkgpulse.com/guides/emotion-vs-styled-components-2026

---

## 8. CSS-in-JS Trade-offs

### Definitions

**Core Definition:** CSS-in-JS trade-offs are the benefits and costs that must be weighed when choosing a CSS-in-JS approach over alternatives.

**Technical Definition:** CSS-in-JS offers **prop-driven dynamism**, **component-level colocation**, and **automatic scoping** at the cost of **runtime render overhead**, **increased bundle weight**, and **RSC incompatibility** (for runtime libraries). Compile-time CSS-in-JS eliminates the runtime costs but imposes **build-time complexity** and **limited runtime dynamism** (dynamic styles are typically restricted to CSS variables). The ecosystem has shifted toward zero-runtime and build-time solutions, making runtime CSS-in-JS a deliberate choice rather than a default.

**Beginner-Friendly Explanation:** CSS-in-JS gives you convenient, dynamic styling, but it can make your app slower and does not work with the newest React features. Compile-time versions are faster but less flexible for dynamic styles.

### Trade-offs Table

| Dimension | Runtime CSS-in-JS | Compile-Time CSS-in-JS | CSS Modules / Tailwind |
|-----------|-------------------|------------------------|------------------------|
| **Dynamic prop-based styles** | Yes, full JavaScript dynamism | Limited to CSS variables | Via inline styles or classes |
| **Runtime performance cost** | Moderate (style computation + injection) | None | None |
| **Bundle size** | ~7–12KB for the engine | Minimal | None |
| **RSC compatibility** | No (requires `'use client'`) | Yes | Yes |
| **Build complexity** | Low (drop-in library) | Moderate (Babel/bundler plugin) | Low–moderate |
| **Maintenance status** | styled-components: maintenance mode; Emotion: active but declining | Linaria, Vanilla Extract: active | Active |

### Annotated Code Examples

**Runtime Trade-off:**

```jsx
// ✅ Convenient: full JavaScript dynamism
const Box = styled.div`
  width: ${(props) => props.percent}%;
`;

// ❌ Cost: runs on every render where percent changes
```

**Compile-Time Trade-off:**

```ts
// ✅ Zero runtime: static CSS
export const box = style({ width: 'var(--box-width)' });

// ❌ Less convenient: dynamic value must be set via CSS variable
<div className={box} style={{ '--box-width': `${percent}%` }} />
```

**Expected Output:** The runtime version offers simpler dynamic styling; the compile-time version offers zero runtime cost with slightly more indirection.

**Why this output:** The runtime version’s interpolation is evaluated during render. The compile-time version generates a static rule that reads a CSS variable, shifting the dynamic part to an inline style.

### Real-World Cases

- **Choosing a styling approach for a new project:** The decision tree depends on RSC usage, design-system needs, and runtime-cost tolerance.
- **Migrating an existing codebase:** Understanding the cost of remaining on runtime CSS-in-JS versus migrating to a zero-runtime solution.
- **Performance budgeting:** Estimating the impact of the styling engine on bundle size and render performance.

### References

- RavenClaude — Styling approach decision trees — https://github.com/mcorbett51090/RavenClaude/blob/main/plugins/frontend-engineering/knowledge/styling-and-bundle-decision-trees.md
- GitHub — CSS-in-JS and styling status — https://github.com/saschb2b/skills/blob/main/skills/engineering/javascript-ecosystem/references/ui/css-in-js.md
- pkgpulse — Emotion vs styled-components in 2026 — https://www.pkgpulse.com/guides/emotion-vs-styled-components-2026

---

## 9. CSS-in-JS Best Practices

### Definitions

**Core Definition:** CSS-in-JS best practices are guidelines for using CSS-in-JS libraries safely, performantly, and maintainably in modern React applications.

**Technical Definition:** Best practices for CSS-in-JS in 2025/2026 are shaped by the React ecosystem’s shift toward Server Components and zero-runtime styling. The core guidance is: **never use runtime CSS-in-JS inside React Server Components**; move it behind a `'use client'` boundary or replace it with a build-time solution. For new applications, prefer build-time CSS (CSS Modules, Tailwind, or zero-runtime CSS-in-JS like Vanilla Extract) unless there is a genuine need for full runtime dynamism. When runtime CSS-in-JS is used, it should be confined to client components, themed via CSS variables where possible, and monitored for bundle and render performance.

**Beginner-Friendly Explanation:** If you are using CSS-in-JS, do not use the runtime versions in Server Components. For new projects, consider build-time alternatives first. If you do use runtime CSS-in-JS, keep it in client components and watch your bundle size.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Never use runtime CSS-in-JS in RSC** | Move behind `'use client'` or replace with build-time CSS. |
| **Prefer build-time for new projects** | CSS Modules, Tailwind, or zero-runtime CSS-in-JS are safer defaults. |
| **Choose runtime only for genuine dynamism** | If prop-driven styles are a real requirement, budget the cost. |
| **Use CSS variables for theming** | Reduces the need for runtime prop interpolation. |
| **Monitor bundle size** | Runtime engines add 7–12KB; track the impact. |
| **Set up an SSR style registry** | If using runtime CSS-in-JS with SSR, use `useServerInsertedHTML` or equivalent. |
| **Be aware of maintenance mode** | styled-components is in maintenance mode; Stitches is unmaintained. |

### Annotated Code Examples

**✅ Good Practice (RSC-Safe Build-Time):**

```tsx
// RSC-safe: zero-runtime CSS-in-JS (Vanilla Extract)
import { button } from './button.css';

export default async function Page() {
  return <button className={button}>Click</button>;
}
```

**❌ Bad Practice (Runtime CSS-in-JS in RSC):**

```tsx
// ❌ Breaks in RSC: runtime CSS-in-JS requires client context
import styled from 'styled-components';

const Button = styled.button`background: blue;`;

export default async function Page() {
  return <Button>Click</Button>; // Styles never applied
}
```

**Expected Output:** The RSC-safe version renders a fully styled button. The runtime version renders an unstyled button because the style injection mechanism does not exist in the server environment.

**Why this output:** Runtime CSS-in-JS relies on React context and DOM APIs, neither of which is available in React Server Components. The zero-runtime version produces static CSS that the server can reference directly.

### Real-World Cases

- **New application architecture:** Deciding between CSS Modules, Tailwind, and zero-runtime CSS-in-JS.
- **Migrating from runtime CSS-in-JS:** Understanding the boundaries and replacement strategies.
- **Design-system maintenance:** Choosing a theming strategy that works across RSC and client components.

### References

- GitHub — CSS-in-JS and styling status — https://github.com/saschb2b/skills/blob/main/skills/engineering/javascript-ecosystem/references/ui/css-in-js.md
- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Next.js — Styling: CSS-in-JS — https://nextjs.org/docs/app/building-your-application/styling/css-in-js
- RavenClaude — Styling approach decision trees — https://github.com/mcorbett51090/RavenClaude/blob/main/plugins/frontend-engineering/knowledge/styling-and-bundle-decision-trees.md

---

## References

- Next.js — Styling: CSS-in-JS — https://nextjs.org/docs/app/building-your-application/styling/css-in-js
- styled-components — FAQs — https://styled-components.com/docs/faqs
- GitHub — styled-components maintenance mode discussion — https://github.com/styled-components/styled-components/discussions/5568
- Emotion — Introduction — https://emotion.sh/docs/introduction
- Emotion — Server-Side Rendering — https://emotion.sh/docs/ssr
- Linaria — Zero-runtime CSS in JS — https://github.com/callstack/linaria
- npm — @linaria/server — https://www.npmjs.com/package/@linaria/server
- Stitches — Official site — https://stitches.dev
- GitHub — Stitches maintenance notice — https://github.com/stitchesjs/stitches
- Vanilla Extract — Official site — https://vanilla-extract.style
- npm — @vanilla-extract/css — https://www.npmjs.com/package/@vanilla-extract/css
- pkgpulse — Emotion vs styled-components in 2026 — https://www.pkgpulse.com/guides/emotion-vs-styled-components-2026
- GitHub — CSS-in-JS and styling status — https://github.com/saschb2b/skills/blob/main/skills/engineering/javascript-ecosystem/references/ui/css-in-js.md
- RavenClaude — Styling approach decision trees — https://github.com/mcorbett51090/RavenClaude/blob/main/plugins/frontend-engineering/knowledge/styling-and-bundle-decision-trees.md
- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Telerik — Static Extraction in CSS-in-JS for Efficiency in React Apps — https://www.telerik.com/blogs/static-extraction-css-in-js
- Feature-Sliced Design — Vanilla Extract: The Perfect CSS Architecture? — https://feature-sliced.design
- Tencent Cloud — 为什么和 CSS-in-JS 说拜拜 — https://cloud.tencent.com.cn/developer/article/2180504