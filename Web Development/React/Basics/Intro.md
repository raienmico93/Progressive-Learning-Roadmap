# Introduction to React

## React Definition

React is a **JavaScript library for building user interfaces**, originally created by Facebook (Meta) and released in 2013. It has since become one of the most widely used tools for front-end development.

- **JavaScript library for building user interfaces** — React focuses specifically on the "view" layer of an application. It provides the tools to describe what the UI should look like at any given moment, without prescribing how the rest of your application should be structured.

- **Component-oriented development model** — React encourages you to break your UI into small, self-contained pieces called *components*. Each component encapsulates its own structure, logic, and (optionally) styles, and components can be composed together to form complex interfaces.

- **Declarative approach to UI construction** — Instead of manually issuing step-by-step instructions to mutate the DOM, you *describe* what the UI should look like for a given state. React then figures out how to update the DOM efficiently to match that description.

## Core Goals

React was designed with several guiding objectives in mind:

- **To build interactive user interfaces** — React is optimized for UIs that respond to user input, network events, and other dynamic data sources.

- **To organize interfaces into reusable components** — By treating UI elements as composable units, React promotes code reuse, separation of concerns, and easier reasoning about complex interfaces.

- **To manage changing application state** — React provides mechanisms (such as `useState`, `useReducer`, and context) to track data that changes over time and drive re-renders when it does.

- **To synchronize UI with application data** — The core idea is that the UI is a *function of state*. When state changes, React re-renders the affected parts of the UI so it stays consistent with the underlying data.

## React Ecosystem

React itself is deliberately minimal, and a rich ecosystem has grown around it:

- **React** — The core library: components, hooks, and the rendering model.
- **React DOM** — The package that renders React components to the browser DOM. (Separate renderers exist for other targets.)
- **React Native** — A framework for building native mobile apps for iOS and Android using React concepts.
- **React Router** — The de facto standard library for client-side routing in React web apps.
- **State-management libraries** — Tools like Redux, Zustand, Jotai, MobX, and Recoil for handling complex or shared application state.
- **Build tools** — Vite, webpack, Parcel, and others for bundling, transpiling (e.g., JSX), and optimizing React code.
- **Testing tools** — Jest, React Testing Library, Vitest, and Playwright/Cypress for unit, integration, and end-to-end testing.
- **Frameworks such as Next.js** — Meta-frameworks (Next.js, Remix, Gatsby, Astro) that build on React to add routing, server rendering, data fetching, and more.

## React versus Traditional DOM Manipulation

With **traditional DOM manipulation** (e.g., using vanilla JavaScript or jQuery), you write imperative code that directly finds and mutates DOM nodes:

```js
const el = document.getElementById("count");
el.textContent = count;
```

You must track which parts of the DOM need updating and update them yourself. As applications grow, this becomes error-prone and hard to reason about.

React takes a **declarative** approach:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

You describe the output for a given state; React handles the DOM updates. Internally, React uses a **virtual DOM** and a diffing algorithm to compute the minimal set of changes needed.

## React versus Template-Based Approaches

Template-based frameworks (e.g., Angular templates, Vue templates, or server-side templating like Handlebars) typically use an **HTML-like syntax with special directives**:

```html
<!-- Vue example -->
<button @click="count++">{{ count }}</button>
```

React instead uses **JSX**, which is a syntax extension of JavaScript:

```jsx
<button onClick={() => setCount(count + 1)}>{count}</button>
```

Key differences:

- **Templates** separate markup from logic and rely on framework-specific directives and syntax.
- **JSX** keeps markup and logic together in JavaScript, giving you the full power of the language (expressions, functions, conditionals, loops) inside your markup.
- React components are just JavaScript functions, making them easier to test, compose, and refactor with standard tooling.

## React versus Other UI Libraries and Frameworks

| Aspect | React | Vue | Angular | Svelte |
|---|---|---|---|---|
| Type | Library (view layer) | Progressive framework | Full framework | Compiler |
| Language | JSX | Templates / JSX | TypeScript + templates | Svelte syntax |
| DOM strategy | Virtual DOM | Virtual DOM | Incremental DOM | Compile-time |
| Learning curve | Moderate | Gentle | Steep | Gentle |
| Opinionatedness | Low | Medium | High | Medium |
| Ecosystem | Very large | Large | Large (built-in) | Growing |

- **React vs. Vue** — Vue is more opinionated and offers official solutions for routing and state; React leaves those choices to the ecosystem.
- **React vs. Angular** — Angular is a complete, opinionated framework with dependency injection, RxJS, and TypeScript-first design; React is a focused library.
- **React vs. Svelte** — Svelte shifts work to compile time, producing highly optimized vanilla JS with no virtual DOM; React relies on runtime diffing.
- **React vs. Web Components** — Web Components are a browser standard for encapsulated custom elements; React is a library that can interoperate with them.

In short, React's defining characteristics are its **component model**, **declarative style**, and **minimal core with a rich ecosystem** — making it a flexible foundation for building modern interactive UIs on web and beyond.