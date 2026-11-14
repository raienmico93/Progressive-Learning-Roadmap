# React Portals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A React portal is a first-class mechanism for rendering a React child into a DOM node that exists outside the DOM hierarchy of the parent component, while preserving the React component tree relationship.

**Technical Definition:** `createPortal(children, domNode, key?)` is a React DOM API that returns a React node. When React encounters this node in the render output, it places the provided `children` inside the provided `domNode` — a DOM element that must already exist in the document. The portal changes only the **physical placement** of the DOM node. In every other respect — context access, event propagation, state ownership, and lifecycle — the JSX rendered into the portal behaves as a normal child of the React component that renders it. Events propagate according to the **React tree**, not the DOM tree, and context provided by the parent React tree is fully accessible to portaled children.

**Beginner-Friendly Explanation:** A portal is React’s “teleport” for DOM nodes. Normally, when you render a component, its HTML appears inside the parent component’s HTML. A portal lets you render the component’s HTML somewhere else entirely — often at the end of `<body>` — while keeping all of React’s behavior intact. The component still belongs to its React parent: it can read the parent’s context, and events still bubble up through the React tree. This is essential for modals, tooltips, and dropdowns that need to visually escape from containers with `overflow: hidden` or `z-index` constraints.

### Key Characteristics

- **Physical DOM relocation only:** The DOM node moves; the React tree relationship does not.
- **React tree preserved:** Context, event bubbling, and state ownership follow the React component tree, not the DOM tree.
- **Escape from CSS containment:** Portals bypass `overflow: hidden`, `z-index` stacking contexts, and other CSS containment traps.
- **Container must exist:** The target `domNode` must already be present in the document when the portal renders.
- **Key support:** An optional `key` can be used when rendering arrays of portals.
- **Accessibility requires deliberate work:** Screen readers lose the DOM-level relationship between a trigger and its portaled content; ARIA attributes must restore it.

### Prerequisites

- React function components and JSX.
- Understanding of the DOM and CSS stacking contexts.
- Familiarity with React context and event handling.
- Basic knowledge of accessibility (ARIA attributes and focus management).

### Related Programming Areas

- **Rendering Performance** — portals do not change reconciliation; they only change DOM placement.
- **Concurrent Features** — portals work with Suspense and transitions like any other React node.
- **Accessibility** — focus trapping and ARIA relationships are critical for portaled modals and menus.
- **Styling Patterns** — portals are a strategy for escaping stacking contexts and overflow clipping.

---

## 1. Portals

### Definitions

**Core Definition:** A portal is a React feature that renders a component’s output into a different DOM node while keeping the component in its original position in the React tree.

**Technical Definition:** Portals provide a first-class way to render children into a DOM node that exists outside the DOM hierarchy of the parent component. The React element tree is unchanged; only the physical location of the resulting DOM nodes differs. This means the portaled content remains a child of its React parent for the purposes of context, state, lifecycle, and event propagation.

**Beginner-Friendly Explanation:** A portal lets a component “project” its HTML somewhere else on the page without leaving its place in React’s component tree.

### Purposes

- To render modals, dialogs, tooltips, and dropdowns outside containers that would clip or obscure them.
- To escape `overflow: hidden`, `transform`, `filter`, and `z-index` stacking contexts that trap positioned children.
- To preserve React context, event bubbling, and state ownership while relocating DOM nodes.
- To integrate React components into third-party DOM roots or micro-frontend containers.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { createPortal } from 'react-dom';

createPortal(children, domNode, key?);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `children` | Any renderable React content: JSX, fragment, string, number, or array. |
| `domNode` | The target DOM element. Must already exist in the document. |
| `key` | Optional unique key for arrays of portals. |

**Syntax Rules:**

1. `createPortal` is imported from `react-dom`.
2. The returned value is a React node that can be included in JSX or returned from a component.
3. The `domNode` must already exist when the portal renders; changing it during an update recreates the portal content.
4. Events propagate through the React tree, not the DOM tree.
5. Context from the parent React tree is accessible inside the portal.

**Constraints and Limitations:**

- Portals do not affect CSS; the portaled content inherits CSS from its new DOM parent, not its React parent.
- Focus management and ARIA relationships must be handled manually for accessible overlays.
- The portal container must be reachable in the DOM; server-rendered portals require the container to exist during SSR.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Portal to `document.body`

```jsx
import { createPortal } from 'react-dom';

function MyComponent() {
  return (
    <div style={{ border: '2px solid black' }}>
      <p>This child is placed in the parent div.</p>
      {createPortal(
        <p>This child is placed in the document body.</p>,
        document.body
      )}
    </div>
  );
}
```

**Expected Output:** The first paragraph appears inside the bordered `div`. The second paragraph appears directly under `<body>`, visually outside the border.

**Why this output:** `createPortal` returns a React node that React places inside `document.body`. The portal changes only the DOM placement; the React tree still treats the second paragraph as a child of `MyComponent`.

#### Example 2: Portal with a Key

```jsx
function ModalList({ modals }) {
  return (
    <>
      {modals.map(modal =>
        createPortal(
          <ModalContent {...modal} />,
          document.getElementById('modal-root'),
          modal.id
        )
      )}
    </>
  );
}
```

**Expected Output:** Each modal is rendered into `#modal-root` with its own key, allowing React to correctly reconcile multiple portals.

**Why this output:** The optional `key` helps React identify each portal when rendering an array, preventing incorrect matching during reconciliation.

### Real-World Cases

- **Modal dialogs:** Rendering the modal overlay and content into a dedicated `#modal-root` to escape the component’s DOM ancestors.
- **Tooltips and popovers:** Rendering positioned tooltips outside `overflow: hidden` containers.
- **Dropdown menus:** Escaping stacking contexts created by transformed or filtered ancestors.
- **Third-party integration:** Mounting React content into a DOM node managed by a legacy library.

### References

- React — createPortal — https://react.dev/reference/react-dom/createPortal
- React — createPortal (React 18) — https://18.react.dev/reference/react-dom/createPortal
- React — Portals (Legacy) — https://ru.react.js.org/docs/portals.html

---

## 2. `ReactDOM.createPortal`

### Definitions

**Core Definition:** `ReactDOM.createPortal` is the function that creates a portal by binding a React child to a specific DOM container.

**Technical Definition:** `createPortal(children, container, key?)` is a top-level API exported from `react-dom`. It returns a React element of type `Symbol(react.portal)` that React’s renderer recognises. During commit, React inserts the rendered DOM nodes for `children` into `container`. The container is not a React root; it is a regular DOM node that receives the portaled content.

**Beginner-Friendly Explanation:** `createPortal` is the function you call to make a portal happen. You give it the content and the DOM node where it should go.

### Purposes

- To create a portal element that can be embedded in JSX.
- To bind React content to an existing DOM node outside the component’s hierarchy.
- To support multiple portals with distinct keys.
- To enable modal, tooltip, and dropdown patterns that escape CSS containment.

### Syntax Rules and Structure

```jsx
import { createPortal } from 'react-dom';

const portal = createPortal(children, container, key?);
```

**Syntax Rules:**

1. `createPortal` is a named export from `react-dom`.
2. The first argument is the renderable content; the second is the DOM container; the third is an optional key.
3. The returned React node can be placed anywhere JSX is valid.
4. React inserts the content into `container` during the commit phase.

**Constraints and Limitations:**

- The container must exist before the portal renders; passing `null` or `undefined` causes an error.
- The container is not a React root; you cannot call `createRoot` on it while the portal is active.
- Changing the container during an update recreates the portal’s content.

### Annotated Code Examples

```jsx
import { createPortal } from 'react-dom';

function Tooltip({ children, anchorRect }) {
  const container = document.getElementById('tooltip-root');

  return createPortal(
    <div
      style={{
        position: 'fixed',
        top: anchorRect.bottom,
        left: anchorRect.left,
      }}
    >
      {children}
    </div>,
    container
  );
}
```

**Expected Output:** The tooltip content is rendered inside `#tooltip-root`, positioned relative to the viewport, escaping any `overflow: hidden` ancestor.

**Why this output:** `createPortal` binds the tooltip JSX to `#tooltip-root`. React places the tooltip’s DOM node there during commit, while the `Tooltip` component remains a child of its React parent.

### Real-World Cases

- **Fixed-position overlays:** Rendering tooltips and popovers that must not be clipped.
- **Modals:** Binding modal content to a dedicated root outside the app container.
- **Micro-frontends:** Mounting React widgets into host page containers.

### References

- React — createPortal — https://react.dev/reference/react-dom/createPortal
- React — createPortal (React 18) — https://18.react.dev/reference/react-dom/createPortal

---

## 3. Portal Containers

### Definitions

**Core Definition:** A portal container is the DOM element into which a portal’s content is rendered.

**Technical Definition:** The container is a regular DOM node — often a `<div>` with a specific `id` placed at the end of `<body>` (e.g., `<div id="modal-root"></div>`). React inserts the portaled DOM nodes into this container during commit. The container must already exist in the document when the portal renders. Using a dedicated container per overlay type (modals, tooltips, dropdowns) is a common organisational pattern.

**Beginner-Friendly Explanation:** The container is the “destination” of the portal. It is a placeholder DOM element that you create in your HTML, and the portal’s content appears inside it.

### Purposes

- To provide a stable DOM destination for portaled content.
- To isolate overlay content from the application’s main DOM hierarchy.
- To organise multiple overlays into separate roots (e.g., `#modal-root`, `#tooltip-root`).
- To control stacking order by placing containers at the end of `<body>`.

### Syntax Rules and Structure

**HTML:**

```html
<body>
  <div id="root"></div>
  <div id="modal-root"></div>
  <div id="tooltip-root"></div>
</body>
```

**React:**

```jsx
const modalRoot = document.getElementById('modal-root');
createPortal(<ModalContent />, modalRoot);
```

**Syntax Rules:**

1. The container must exist in the DOM before the portal renders.
2. A container can receive content from multiple portals; use keys to manage arrays.
3. Do not create the container inside the same React tree that uses the portal; create it in the HTML shell.
4. Changing the container during an update recreates the portal’s content.

**Constraints and Limitations:**

- Server rendering requires the container to be present in the initial HTML.
- If the container is removed while the portal is active, React’s DOM operations may throw.
- Multiple portals sharing a container can conflict if their content is not keyed.

### Annotated Code Examples

```jsx
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  const modalRoot = document.getElementById('modal-root');
  if (!modalRoot) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    modalRoot
  );
}
```

**Expected Output:** When `isOpen` is true, the modal overlay and content render inside `#modal-root`, outside the main `#root` container.

**Why this output:** `createPortal` targets `#modal-root`. React commits the modal’s DOM nodes into that container, separating the modal from the application’s DOM hierarchy while preserving the React tree relationship.

### Real-World Cases

- **Modal roots:** A dedicated `#modal-root` for all dialogs.
- **Tooltip roots:** A `#tooltip-root` for transient overlays.
- **Notification roots:** A `#toast-root` for stacked notifications.

### References

- React — createPortal: Parameters — https://18.react.dev/reference/react-dom/createPortal
- React — Portals (Legacy): Container — https://ru.react.js.org/docs/portals.html

---

## 4. Portal Events

### Definitions

**Core Definition:** Portal events are DOM events that originate inside portaled content and propagate through the React component tree, not the DOM tree.

**Technical Definition:** React attaches event listeners at the React root and dispatches synthetic events along the React fiber tree. When an event fires inside a portal, React propagates it to the portal’s React ancestors — even if those ancestors are not DOM ancestors. This means a click inside a portaled modal can trigger an `onClick` handler on the React parent that rendered the portal, even though the modal’s DOM node is in a different part of the document. If this behaviour is undesirable, `e.stopPropagation()` inside the portal or moving the portal higher in the React tree can prevent it.

**Beginner-Friendly Explanation:** Events inside a portal bubble up through React’s component tree, not through the HTML tree. So a click inside a modal can still trigger a click handler on the component that created the modal, even though the modal is not inside that component in the DOM.

### Purposes

- To allow parent components to handle events from portaled content without DOM ancestry.
- To enable flexible abstractions that do not depend on physical DOM placement.
- To let developers intentionally stop propagation when portal events should not reach React ancestors.

### Syntax Rules and Structure

**Event Propagation:**

| Aspect | Behaviour |
|--------|-----------|
| Propagation path | React tree (fiber tree), not DOM tree |
| React ancestors | Events bubble to React ancestors regardless of DOM location |
| Stopping propagation | `e.stopPropagation()` inside the portal |
| Alternative | Move the portal higher in the React tree |

**Syntax Rules:**

1. Events from portals propagate according to the React tree, not the DOM tree.
2. `e.stopPropagation()` inside the portal prevents the event from reaching React ancestors.
3. Moving the portal to a higher React ancestor changes which handlers receive the event.

### Annotated Code Examples

```jsx
function Parent() {
  function handleClick() {
    console.log('Parent click handler fired');
  }

  return (
    <div onClick={handleClick}>
      <p>Parent content</p>
      {createPortal(
        <button onClick={() => console.log('Portal button clicked')}>
          Click me
        </button>,
        document.body
      )}
    </div>
  );
}
```

**Expected Output:** Clicking the portaled button logs “Portal button clicked” and then “Parent click handler fired”.

**Why this output:** The button is a DOM child of `<body>`, but a React child of `Parent`. React propagates the click event through the React tree, reaching `Parent`’s `onClick` handler.

### Real-World Cases

- **Modal close handlers:** A parent’s `onClick` can close a modal when the overlay is clicked, even though the modal is portaled.
- **Global event delegation:** Parent components can handle events from portaled dropdowns and tooltips.
- **Stopping propagation:** Preventing a portaled menu’s click from closing an outer dropdown.

### References

- React — Portals: Event Bubbling Through Portals — https://ru.react.js.org/docs/portals.html
- React — createPortal: Caveats — https://18.react.dev/reference/react-dom/createPortal
- CoreUI — How to use portals in React — https://coreui.io

---

## 5. Portal Context

### Definitions

**Core Definition:** Portal context is the React Context available to portaled content from its React parent tree, regardless of the portal’s DOM location.

**Technical Definition:** Context in React is propagated through the React component tree, not the DOM tree. Because a portal’s content remains a child of its React parent, it has full access to any context provider above that parent. This means a portaled modal can read a theme context, authentication context, or locale context provided by the application root, even though its DOM node is elsewhere.

**Beginner-Friendly Explanation:** Context works normally inside a portal. The portaled content can read any context provided by its React ancestors, because React still treats it as part of the same component tree.

### Purposes

- To allow portaled content to consume application-wide context (theme, auth, locale).
- To avoid duplicating context providers around portal containers.
- To keep portaled overlays consistent with the application’s state and styling.

### Syntax Rules and Structure

```jsx
<ThemeContext.Provider value="dark">
  <Parent>
    {createPortal(<ThemedModal />, document.body)}
  </Parent>
</ThemeContext.Provider>
```

**Syntax Rules:**

1. Context providers above the portal’s React parent are accessible to the portaled content.
2. Context providers around the DOM container do not affect the portal; only the React tree matters.
3. No special configuration is required; context works automatically.

### Annotated Code Examples

```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Modal />
    </ThemeContext.Provider>
  );
}

function Modal() {
  return createPortal(
    <ThemedContent />,
    document.getElementById('modal-root')
  );
}

function ThemedContent() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>Themed modal content</div>;
}
```

**Expected Output:** The portaled modal content renders with the `"dark"` theme class.

**Why this output:** `ThemedContent` is a React child of `Modal`, which is inside `ThemeContext.Provider`. The portal changes only the DOM location; context flows through the React tree.

### Real-World Cases

- **Themed modals:** Modals that respect the app’s light/dark theme.
- **Auth-aware tooltips:** Tooltips that display user-specific information.
- **Localised overlays:** Dropdowns that use the app’s locale context.

### References

- React — createPortal: Context — https://es.react.dev/reference/react-dom/createPortal
- React — Portals (Legacy): Context — https://ru.react.js.org/docs/portals.html

---

## 6. Portal Styling

### Definitions

**Core Definition:** Portal styling refers to the CSS behaviour of portaled content, which inherits styles from its new DOM ancestors rather than its React ancestors.

**Technical Definition:** Because a portal changes the physical DOM location of content, the content becomes subject to the CSS cascade of its new DOM parent. It no longer inherits styles from the React parent’s DOM ancestors. This is often the primary reason for using a portal — to escape a stacking context created by `transform`, `filter`, `opacity`, `will-change`, or `isolation` on an ancestor. However, it also means the developer must ensure the portal container has appropriate styles and that the portaled content’s positioning is correct relative to the viewport or another reference.

**Beginner-Friendly Explanation:** Styling inside a portal follows the DOM, not React. If you move a tooltip to `<body>`, it no longer inherits styles from its React parent’s `<div>`. This is usually what you want — it escapes clipping and z-index traps — but you need to make sure the tooltip looks right in its new home.

### Purposes

- To escape stacking contexts that trap `z-index` and clipping ancestors.
- To render overlays that are not clipped by `overflow: hidden`.
- To position fixed or absolute elements relative to the viewport without interference.
- To apply independent styling to portaled content.

### Syntax Rules and Structure

**Stacking Context Escape:**

| Trigger | Effect |
|---------|--------|
| `transform` | Creates a stacking context and containing block |
| `filter` | Creates a stacking context |
| `opacity < 1` | Creates a stacking context |
| `will-change: transform` | Creates a stacking context |
| `isolation: isolate` | Creates a stacking context |

**Syntax Rules:**

1. Portaled content inherits styles from its new DOM parent, not its React parent.
2. Render portaled overlays at `document.body` or a dedicated root to escape stacking contexts.
3. Use CSS custom properties for z-index tokens to keep overlay layering consistent.
4. Ensure the container itself does not create a new stacking context that reintroduces the problem.

**Constraints and Limitations:**

- Portaled content loses inherited styles (font, color, CSS variables) from the React parent’s DOM ancestors.
- If the container has `transform` or `filter`, the portal may still be trapped.
- Fixed positioning inside a transformed container behaves relative to the container, not the viewport.

### Annotated Code Examples

```jsx
// ❌ Trapped: parent has transform, clipping the tooltip
function BadTooltip() {
  return (
    <div style={{ transform: 'translateY(10px)' }}>
      <div className="tooltip" style={{ position: 'absolute', top: 0 }}>
        Tooltip
      </div>
    </div>
  );
}

// ✅ Escaped: portal renders tooltip at body, outside the transform context
function GoodTooltip() {
  return (
    <>
      <div style={{ transform: 'translateY(10px)' }}>
        Trigger
      </div>
      {createPortal(
        <div className="tooltip" style={{ position: 'fixed', top: 100 }}>
          Tooltip
        </div>,
        document.body
      )}
    </>
  );
}
```

**Expected Output:** The `GoodTooltip`’s tooltip is positioned relative to the viewport and is not clipped or offset by the parent’s `transform`. The `BadTooltip`’s tooltip is trapped inside the transformed parent.

**Why this output:** The `transform` on the parent creates a containing block and stacking context. The portal moves the tooltip outside that context, allowing `position: fixed` to work relative to the viewport.

### Real-World Cases

- **Tooltips and popovers:** Escaping transformed or filtered ancestors.
- **Modals:** Rendering above all app content with predictable z-index.
- **Product tours:** Overlays that must not be clipped by cards or sidebars.

### References

- Z-index wars: how product tour overlays actually work — https://github.com/domidex/tour-kit
- CoreUI — How to use portals in React — https://coreui.io

---

## 7. Portal Accessibility

### Definitions

**Core Definition:** Portal accessibility is the practice of preserving focus management, ARIA relationships, and screen-reader semantics when content is moved outside its natural DOM hierarchy.

**Technical Definition:** Portals create a disconnect between the DOM tree and the semantic relationships that assistive technologies rely on. A screen reader may not perceive that a portaled dropdown belongs to its trigger button, because the dropdown is no longer a DOM descendant of the trigger. The solution is to use ARIA attributes — `aria-owns`, `aria-expanded`, `aria-haspopup`, `aria-labelledby`, and appropriate roles — to explicitly declare the relationship. For modal dialogs, focus must be trapped inside the modal and returned to the trigger on close, following the WAI-ARIA Modal Dialog pattern.

**Beginner-Friendly Explanation:** When you portal content to another part of the page, screen readers lose track of how it relates to the trigger. You need to add ARIA attributes to reconnect them, and you need to manage focus so keyboard users do not get lost.

### Purposes

- To restore semantic relationships between triggers and portaled content.
- To ensure keyboard focus remains within modal dialogs and returns to the trigger on close.
- To communicate expanded/collapsed state and menu roles to screen readers.
- To comply with WAI-ARIA authoring practices for dialogs and menus.

### Syntax Rules and Structure

**Accessible Dropdown Pattern:**

| Element | ARIA attributes |
|---------|-----------------|
| Trigger button | `aria-expanded`, `aria-haspopup="menu"`, `aria-owns` (when open) |
| Portaled menu | `role="menu"`, `aria-labelledby` (trigger ID), `id` (target of `aria-owns`) |

**Syntax Rules:**

1. Use `aria-owns` on the trigger to declare ownership of the portaled content when it is open.
2. Use `aria-labelledby` on the portaled content to reference the trigger’s `id`.
3. Use `aria-expanded` and `aria-haspopup` on the trigger to communicate state.
4. Trap focus inside modal dialogs and return focus to the trigger on close.
5. Use `useId` to generate unique IDs for the trigger and portaled content.

**Constraints and Limitations:**

- `aria-owns` is not a silver bullet; test with actual screen readers.
- Focus management must be implemented manually or with a focus-trap library.
- Portaled content can still be reached by screen readers unless `aria-hidden` is managed on background content.

### Annotated Code Examples

```jsx
import { useId, useState } from 'react';
import { createPortal } from 'react-dom';

function AccessibleDropdown({ options }) {
  const [isOpen, setIsOpen] = useState(false);
  const buttonId = useId();
  const menuId = useId();

  return (
    <div>
      <button
        id={buttonId}
        onClick={() => setIsOpen(!isOpen)}
        aria-expanded={isOpen}
        aria-haspopup="menu"
        aria-owns={isOpen ? menuId : undefined}
      >
        Options
      </button>
      {isOpen &&
        createPortal(
          <ul
            id={menuId}
            role="menu"
            aria-labelledby={buttonId}
          >
            {options.map(option => (
              <li key={option.value} role="menuitem">
                {option.label}
              </li>
            ))}
          </ul>,
          document.body
        )}
    </div>
  );
}
```

**Expected Output:** Screen readers announce the button as a menu trigger with expanded state, and associate the portaled menu with the button.

**Why this output:** `aria-owns` creates the semantic parent-child relationship across the DOM gap. `aria-labelledby` names the menu after the trigger. `aria-expanded` and `aria-haspopup` communicate the interaction model.

### Real-World Cases

- **Modal dialogs:** Focus trapping and returning focus to the trigger.
- **Dropdown menus:** ARIA relationships across the portal boundary.
- **Comboboxes:** Portaled listboxes with proper `aria-controls` and `aria-activedescendant`.

### References

- CoreUI — Maintaining Accessibility with React createPortal and aria-owns — https://coreui.io/blog/maintaining-accessibility-with-react-createportal-and-aria-owns/
- React — Portals: Accessibility — https://ru.react.js.org/docs/portals.html
- accessibility-agents — Focus management for portals — https://github.com/Community-Access/accessibility-agents

---

## 8. Portal Use Cases

### Definitions

**Core Definition:** Portal use cases are the UI patterns that benefit from rendering content outside the normal DOM hierarchy while preserving React tree behaviour.

**Technical Definition:** Portals are appropriate when content must visually escape its container’s CSS constraints (overflow clipping, stacking contexts, transformed ancestors) while remaining logically part of the React component that controls it. The canonical use cases are modal dialogs, tooltips, popovers, dropdown menus, toast notifications, and product tour overlays. Portals are also used to integrate React into non-React DOM roots.

**Beginner-Friendly Explanation:** Portals are for things that need to “float” above the rest of the page — modals, tooltips, dropdowns, and notifications — without being trapped by the layout of their parent components.

### Purposes

- To render overlays that visually escape clipping and stacking contexts.
- To keep overlay logic connected to its React trigger while moving the DOM node.
- To integrate React content into external DOM containers.
- To implement accessible modal dialogs and menus with proper ARIA relationships.

### Use Cases Table

| Use case | Why portals |
|----------|-------------|
| Modal dialogs | Escape `overflow: hidden` and stacking contexts; cover the viewport. |
| Tooltips and popovers | Avoid clipping by transformed or filtered ancestors. |
| Dropdown menus | Render above other content without z-index wars. |
| Toast notifications | Stack notifications at a fixed viewport position. |
| Product tours | Overlay highlights and blur layers above all app content. |
| Micro-frontends | Mount React widgets into host page containers. |

### Annotated Code Examples

```jsx
function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>Open Modal</button>
      {isModalOpen &&
        createPortal(
          <div className="modal-overlay">
            <div className="modal">
              <h2>Modal Title</h2>
              <button onClick={() => setIsModalOpen(false)}>Close</button>
            </div>
          </div>,
          document.getElementById('modal-root')
        )}
    </div>
  );
}
```

**Expected Output:** The modal renders inside `#modal-root` at the end of `<body>`, covering the viewport and escaping any clipping from the app’s DOM ancestors.

**Why this output:** The portal moves the modal’s DOM nodes outside the app container, allowing `position: fixed` and high `z-index` to work without being trapped by ancestor CSS.

### Real-World Cases

- **E-commerce:** Quick-view modals and cart drawers that overlay the page.
- **SaaS dashboards:** Dropdown filters and tooltips that escape scrollable containers.
- **Content sites:** Image lightboxes and share popovers.
- **Product tours:** Spotlight overlays and step tooltips.

### References

- React — Portals (Legacy): Usage — https://ru.react.js.org/docs/portals.html
- CoreUI — How to use portals in React — https://coreui.io
- Z-index wars: how product tour overlays actually work — https://github.com/domidex/tour-kit

---

## 9. Portal Best Practices

### Definitions

**Core Definition:** Portal best practices are the guidelines for using portals safely, accessibly, and performantly in React applications.

**Technical Definition:** Best practices encompass dedicated container management (one root per overlay type), event propagation awareness (React tree, not DOM tree), context preservation (portals inherit React context), accessibility implementation (ARIA relationships and focus management), styling discipline (avoiding new stacking contexts in the container), and cleanup (removing containers on unmount). Portals should be used deliberately for overlay patterns, not as a general rendering strategy.

**Beginner-Friendly Explanation:** Use portals for overlays. Create dedicated container elements. Remember that events bubble through React, not the DOM. Add ARIA attributes and manage focus. Keep the container free of CSS that would trap the portal again.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use dedicated containers** | One root per overlay type (e.g., `#modal-root`, `#tooltip-root`). |
| **Ensure the container exists** | Create it in the HTML shell, not inside the React tree. |
| **Handle event propagation** | Events bubble through the React tree; use `stopPropagation` if needed. |
| **Leverage context** | Portals inherit React context from their React parent. |
| **Add ARIA relationships** | Use `aria-owns`, `aria-labelledby`, and roles for portaled content. |
| **Trap focus for modals** | Follow WAI-ARIA modal practices; return focus to the trigger. |
| **Avoid stacking context traps** | Do not place `transform` or `filter` on the container. |
| **Clean up containers** | Remove dynamically created containers on unmount. |
| **Use keys for portal arrays** | Provide a unique `key` when rendering multiple portals. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
function AccessibleModal({ isOpen, onClose, title, children }) {
  const titleId = useId();
  const modalRoot = document.getElementById('modal-root');

  useEffect(() => {
    if (!isOpen) return;
    const previouslyFocused = document.activeElement;
    // Focus the modal
    const modal = document.getElementById('modal-content');
    modal?.focus();
    return () => {
      // Return focus to trigger
      previouslyFocused?.focus();
    };
  }, [isOpen]);

  if (!isOpen || !modalRoot) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div
        id="modal-content"
        role="dialog"
        aria-modal="true"
        aria-labelledby={titleId}
        tabIndex={-1}
        onClick={e => e.stopPropagation()}
      >
        <h2 id={titleId}>{title}</h2>
        {children}
      </div>
    </div>,
    modalRoot
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ No ARIA relationship: screen reader cannot connect trigger and menu
{createPortal(<ul>{items}</ul>, document.body)}

// ❌ Container inside the React tree: portal may be trapped by ancestor CSS
function BadContainer() {
  return (
    <div style={{ transform: 'scale(1)' }}>
      <div id="bad-root" />
      {createPortal(<Tooltip />, document.getElementById('bad-root'))}
    </div>
  );
}

// ❌ No focus management: keyboard users lose context
createPortal(<ModalContent />, document.body);
```

**Expected Output:** The good practices produce an accessible, focus-managed modal that escapes CSS constraints. The bad practices cause screen-reader disconnect, trapped portals, and lost keyboard focus.

**Why this output:** ARIA attributes restore semantic relationships. Dedicated containers at `document.body` escape stacking contexts. Focus management keeps keyboard users oriented.

### Real-World Cases

- **Design systems:** Accessible modal and dropdown primitives built on portals.
- **Enterprise applications:** Focus-managed dialogs that comply with WCAG.
- **Product tours:** Portaled overlays with correct stacking and accessibility.

### References

- React — createPortal: Caveats and Usage — https://18.react.dev/reference/react-dom/createPortal
- CoreUI — Maintaining Accessibility with React createPortal and aria-owns — https://coreui.io/blog/maintaining-accessibility-with-react-createportal-and-aria-owns/
- React — Portals (Legacy): Accessibility — https://ru.react.js.org/docs/portals.html
- Z-index wars: how product tour overlays actually work — https://github.com/domidex/tour-kit

---

## References

- React — createPortal — https://react.dev/reference/react-dom/createPortal
- React — createPortal (React 18) — https://18.react.dev/reference/react-dom/createPortal
- React — createPortal (Spanish) — https://es.react.dev/reference/react-dom/createPortal
- React — Portals (Legacy) — https://ru.react.js.org/docs/portals.html
- CoreUI — Maintaining Accessibility with React createPortal and aria-owns — https://coreui.io/blog/maintaining-accessibility-with-react-createportal-and-aria-owns/
- CoreUI — How to use portals in React — https://coreui.io
- Z-index wars: how product tour overlays actually work — https://github.com/domidex/tour-kit
- accessibility-agents — Focus management for portals — https://github.com/Community-Access/accessibility-agents