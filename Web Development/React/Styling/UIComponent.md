# UI Component Systems & Design Engineering: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** UI component systems and design engineering is the discipline of building, organising, and maintaining reusable interface components—spanning pre-styled libraries, headless primitives, atomic design hierarchies, design token systems, responsive layout engines, and accessibility standards—to create scalable, consistent, and inclusive user interfaces.

**Technical Definition:** UI component systems and design engineering encompasses the architectural decisions, tooling, and standards that govern how user interface components are built, composed, themed, and made accessible in React applications. It spans the **component library layer** (pre-styled libraries like MUI, Ant Design, and Chakra UI vs. headless primitives like Radix UI, Ark UI, and React Aria), the **design system architecture layer** (Atomic Design methodology with atoms, molecules, organisms, templates, and pages), the **token layer** (typography, spacing, semantic colour, and elevation scales synchronised across platforms), the **layout layer** (fluid, responsive, and container-query-driven layout components), and the **accessibility layer** (semantic HTML, WAI-ARIA patterns, focus trap management, and keyboard navigation testing).

**Beginner-Friendly Explanation:** Building a UI without a system is like building a house without a blueprint—every room looks different, the doors don't match, and nobody can find anything. UI component systems and design engineering is the blueprint. It decides what building blocks you use (component libraries or headless primitives), how you stack them (Atomic Design), what colours and spacing you use (design tokens), how they respond to different screen sizes (responsive layouts), and how everyone—including people using screen readers and keyboards—can use them (accessibility).

### Key Characteristics

- **Library Spectrum:** Pre-styled libraries (MUI, Ant Design) offer speed and completeness at the cost of design control; headless primitives (Radix, Ark UI, React Aria) offer full design control at the cost of more engineering effort.
- **shadcn/ui Dominance:** The copy-paste model (components copied into your repo, not installed as a dependency) has become the default stack for most new applications, built on Radix or Base UI primitives and Tailwind CSS.
- **Base UI Successor:** Base UI 1.0, released by the MUI team in early 2025, is positioned as the maintained successor to Radix Primitives, with active maintenance and a single-package install.
- **Atomic Design Hierarchy:** Components are organised into five levels—Atoms, Molecules, Organisms, Templates, Pages—creating a clear hierarchy from indivisible elements to complete screens.
- **Three-Layer Token Model:** Design tokens follow a primitives → semantic → component hierarchy, enabling consistent theming and dark mode without hardcoded values.
- **Container Queries Over Media Queries:** Modern responsive components use container queries (`@container`) to adapt to their own container's size, not the viewport, making them truly portable.
- **Accessibility as a Foundation:** WAI-ARIA patterns, focus trapping, keyboard navigation, and screen reader testing are non-negotiable requirements, not afterthoughts.

### Prerequisites

- Solid understanding of React function components, JSX, and props.
- Familiarity with CSS selectors, the cascade, and responsive design principles.
- Working knowledge of TypeScript (for typed component APIs and design tokens).
- Basic understanding of accessibility principles (WCAG 2.2, ARIA roles and attributes).
- Awareness of build tooling (Vite, Webpack) and CSS preprocessing.

### Related Programming Areas

- **Design Systems:** Design tokens, theming infrastructure, and component libraries.
- **CSS Architecture:** Tailwind CSS, CSS Modules, and cascade layers.
- **Accessibility (a11y):** WAI-ARIA, focus management, and assistive technology testing.
- **Responsive Design:** Container queries, fluid typography, and intrinsic layouts.
- **Component Architecture:** Compound components, polymorphic components, and slot-based composition.

### Core Concepts / Features

1. Component Libraries: MUI, Ant Design, and Chakra UI
2. Headless UI & Primitives: Radix UI, Ark UI, and React Aria
3. Atomic Design Systems: Architecture, Deployment, and Composition
4. System Tokens and Synchronisation: Typography, Spacing, Color, and Elevation
5. Building Highly Flexible, Fluid, and Responsive Layout Components
6. Accessibility Standards (a11y): Semantic HTML, WAI-ARIA, Focus Traps, and Keyboard Testing

---

## Core Concept 1: Component Libraries — MUI, Ant Design, and Chakra UI

### Definitions

**Core Definition:** Pre-styled component libraries are complete, ready-to-use UI kits that ship with styled components, theming systems, and comprehensive documentation, enabling rapid development at the cost of design control.

**Technical Definition:** Pre-styled component libraries provide a full component system with default styling, theming APIs, and accessibility primitives baked in. **MUI (Material UI)** is the most popular React UI library, implementing Google's Material Design with CSS-in-JS (Emotion), a comprehensive component library, MUI X components (Data Grid, Date Pickers, Charts), and strong TypeScript support, with approximately 1.4 million weekly downloads. **Ant Design** is a pre-styled, dense, enterprise-flavoured library with Less/CSS-in-JS styling, approximately 1.1 million weekly downloads, and a strong presence in admin and back-office UIs. **Chakra UI v3** is a pre-styled, themeable library built on Ark UI, approximately 70 KB gzipped, with a strong accessibility posture and a focus on developer experience, at approximately 587,000 weekly downloads.

**Beginner-Friendly Explanation:** Pre-styled libraries are like buying furniture from IKEA—you get everything you need in one box, it looks good out of the box, and you can assemble it quickly. But everyone's living room looks a little similar. If you want something completely custom, you will need to modify it heavily or build from scratch. MUI is the biggest furniture store; Ant Design specialises in office furniture; Chakra UI is the modern, accessible option.

### Purposes

- **MUI:** To build enterprise applications quickly with Material Design aesthetics and comprehensive data-heavy components.
- **Ant Design:** To build dense, data-intensive admin panels and back-office UIs with an enterprise-flavoured visual language.
- **Chakra UI:** To build accessible, themeable applications with a developer-friendly style props API.
- **All three:** To reduce development time by providing pre-built, tested, and accessible components.
- **All three:** To provide consistent theming infrastructure for light/dark mode and brand customisation.

### Syntax Rules and Structure

**MUI — Basic Usage:**
```jsx
import { Button, TextField, Box } from '@mui/material';
import { createTheme, ThemeProvider } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: { main: '#007bff' },
    secondary: { main: '#6c757d' },
  },
});

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Box sx={{ p: 2 }}>
        <TextField label="Email" variant="outlined" fullWidth />
        <Button variant="contained" color="primary" sx={{ mt: 2 }}>
          Submit
        </Button>
      </Box>
    </ThemeProvider>
  );
}
```

**Component Breakdown:**
- `createTheme({ palette })`: Creates a custom theme with brand colours.
- `<ThemeProvider>`: Provides the theme to all MUI components.
- `sx={{ p: 2 }}`: The `sx` prop applies theme-aware styles (spacing, colours, breakpoints).
- `variant="contained"`: MUI's variant system for button styles.

**Ant Design — Basic Usage:**
```jsx
import { Button, Input, Form, ConfigProvider } from 'antd';

function App() {
  return (
    <ConfigProvider theme={{ token: { colorPrimary: '#007bff' } }}>
      <Form layout="vertical">
        <Form.Item label="Email" name="email">
          <Input placeholder="you@example.com" />
        </Form.Item>
        <Form.Item>
          <Button type="primary" htmlType="submit">Submit</Button>
        </Form.Item>
      </Form>
    </ConfigProvider>
  );
}
```

**Component Breakdown:**
- `<ConfigProvider theme={{ token }}>`: Configures Ant Design's design tokens.
- `<Form.Item label name>`: Combines label, input, and validation in one component.
- `type="primary"`: Ant Design's button type system.

**Chakra UI v3 — Basic Usage:**
```jsx
import { Button, Input, Box, ChakraProvider, defaultSystem } from '@chakra-ui/react';

function App() {
  return (
    <ChakraProvider value={defaultSystem}>
      <Box p={4}>
        <Input placeholder="you@example.com" mb={2} />
        <Button colorPalette="blue" variant="solid">Submit</Button>
      </Box>
    </ChakraProvider>
  );
}
```

**Component Breakdown:**
- `<ChakraProvider value={defaultSystem}>`: Provides the theme system.
- `p={4}`, `mb={2}`: Style props for spacing (theme-aware).
- `colorPalette="blue"`: Chakra's semantic colour palette system.

**Syntax Rules:**
- Wrap the app in the library's theme provider (`ThemeProvider`, `ConfigProvider`, `ChakraProvider`).
- Use the library's theming API to customise colours, spacing, and typography.
- MUI: Use `sx` prop for one-off styles; use `styled()` for reusable styled components.
- Ant Design: Use `ConfigProvider` with design tokens; use `Form` for form layouts.
- Chakra UI: Use style props for rapid styling; use the `colorPalette` system for semantic colours.
- Import components from path imports (e.g., `@mui/material/Button`) to avoid slow dev startup.

**Constraints and Limitations:**
- **MUI:** Material Design aesthetic may not fit all brands; larger baseline weight; theme customisation can be complex; breaking changes between major versions.
- **Ant Design:** Opinionated visual language; denser and less customisable than headless options; primarily English/Chinese documentation focus.
- **Chakra UI v3:** v2 to v3 is a full rewrite (not a simple migration); fewer enterprise data widgets (Data Grid, Charts) than MUI.
- **All three:** Trade control for speed; customising deeply requires overriding library styles.
- **All three:** Add bundle size; MUI and Ant Design are heavier than headless alternatives.

### Annotated Code Example: MUI Data Grid with Custom Theme

```jsx
import { DataGrid } from '@mui/x-data-grid';
import { createTheme, ThemeProvider } from '@mui/material/styles';
import { Box, Button } from '@mui/material';

const theme = createTheme({
  palette: {
    primary: { main: '#6366f1' },
    background: { default: '#f9fafb' },
  },
  shape: { borderRadius: 8 },
});

const columns = [
  { field: 'id', headerName: 'ID', width: 70 },
  { field: 'name', headerName: 'Name', flex: 1 },
  { field: 'email', headerName: 'Email', flex: 1 },
  { field: 'role', headerName: 'Role', width: 130 },
];

const rows = [
  { id: 1, name: 'Alice', email: 'alice@example.com', role: 'Admin' },
  { id: 2, name: 'Bob', email: 'bob@example.com', role: 'User' },
];

export default function UserTable() {
  return (
    <ThemeProvider theme={theme}>
      <Box sx={{ height: 400, width: '100%', p: 2 }}>
        <DataGrid
          rows={rows}
          columns={columns}
          pageSizeOptions={[5, 10, 25]}
          initialState={{
            pagination: { paginationModel: { pageSize: 5 } },
          }}
          sx={{ borderRadius: 2, bgcolor: 'background.paper' }}
        />
        <Button variant="contained" sx={{ mt: 2 }}>
          Add User
        </Button>
      </Box>
    </ThemeProvider>
  );
}
```

**Expected Output:** A data grid with a custom indigo primary colour, rounded corners, and a light grey background. The grid supports pagination (5, 10, or 25 rows per page). An "Add User" button is displayed below the grid.

**Why This Output Occurs:** `createTheme` overrides MUI's default palette with a custom primary colour and border radius. `DataGrid` inherits the theme automatically. The `sx` prop applies theme-aware spacing and background colours. The `initialState` configures the grid's pagination model.

### Real-World Cases

- **MUI:** Enterprise dashboards, financial applications, and data-heavy admin panels.
- **Ant Design:** Back-office systems, data management tools, and applications with a China-market presence.
- **Chakra UI:** Accessibility-forward SaaS products, marketing sites, and mid-size applications.
- **MUI X:** Applications needing advanced Data Grid, Date Pickers, and Charts.

### References

- AI-Agents-public – Component Library Comparison Guide (January 2026): https://github.com/vasilyu1983/AI-Agents-public/blob/3424d6f5e94010409da012eeb1cb84aaecec88b3/frameworks/shared-skills/skills/software-ui-ux-design/references/component-library-comparison.md
- react-kb – UI Libraries Overview (May 2026): https://github.com/Nguyen-Mau-Anh/react-kb
- Mantine vs Chakra UI vs MUI (2026): https://adminlte.io
- PkgPulse – Best React UI Libraries: MUI vs Ant Design (2026): https://www.pkgpulse.com
- MUI vs Ant Design vs Mantine vs Chakra: The React UI Library Showdown: https://dev.to

---

## Core Concept 2: Headless UI & Primitives — Radix UI, Ark UI, and React Aria

### Definitions

**Core Definition:** Headless UI libraries provide unstyled, accessible components that handle behaviour (focus management, keyboard navigation, ARIA attributes) without any visual styling, giving developers complete control over appearance.

**Technical Definition:** Headless UI primitives separate behaviour and accessibility from presentation. **Radix UI** is the original primitive library that popularised headless components in React, with approximately 28.6k GitHub stars and ~10 component sets, providing automatic focus trapping, ARIA attributes, and keyboard navigation. **React Aria** (Adobe) offers the deepest WAI-ARIA conformance with 40+ components, 4.47 million weekly downloads, 30+ locales, RTL support, and accessible drag-and-drop—no other library matches it for accessibility-critical applications. **Ark UI** is the only major cross-framework headless option, supporting React, Vue, and Solid from one codebase, using XState machines internally. **Base UI 1.0** (released early 2025 by the MUI team) is positioned as the maintained successor to Radix, with a render-prop pattern instead of `asChild` and a single-package install.

**Beginner-Friendly Explanation:** Headless UI is like buying a car chassis and engine without the body. You get all the working parts—steering, brakes, suspension—but you design the exterior yourself. Radix UI is the most popular chassis; React Aria is the most rigorously engineered; Ark UI works with multiple engines (React, Vue, Solid). The key rule: never mix headless libraries in one component tree, or you will have focus-trap and layering conflicts.

### Purposes

- **Radix UI:** To build custom design systems with full styling control and strong accessibility defaults.
- **React Aria:** To build accessibility-critical applications (government, healthcare, i18n-heavy) with the deepest WAI-ARIA conformance.
- **Ark UI:** To build cross-framework component libraries that work in React, Vue, and Solid from one codebase.
- **Base UI:** To build custom design systems with a maintained Radix successor from the MUI team.
- **All:** To own the visual layer completely while delegating behaviour and accessibility to a tested primitive.

### Syntax Rules and Structure

**Radix UI — Dialog:**
```jsx
import * as Dialog from '@radix-ui/react-dialog';

function MyDialog() {
  return (
    <Dialog.Root>
      <Dialog.Trigger asChild>
        <button className="btn">Open Dialog</button>
      </Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Overlay className="overlay" />
        <Dialog.Content className="dialog-content" aria-describedby="dialog-desc">
          <Dialog.Title>Confirm Action</Dialog.Title>
          <p id="dialog-desc">Are you sure you want to proceed?</p>
          <Dialog.Close asChild>
            <button className="btn-secondary">Cancel</button>
          </Dialog.Close>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

**Component Breakdown:**
- `<Dialog.Root>`: Manages open/close state.
- `<Dialog.Trigger asChild>`: Renders as its child (button) with the correct onClick.
- `<Dialog.Portal>`: Renders outside the parent DOM hierarchy.
- `<Dialog.Overlay>`: The backdrop; styles are entirely yours.
- `aria-describedby`: Links the dialog to its description for screen readers.

**React Aria — Select:**
```jsx
import { Select, Label, Button, Popover, ListBox, ListBoxItem } from 'react-aria-components';

function MySelect() {
  return (
    <Select>
      <Label>Choose a fruit</Label>
      <Button>
        <span>Select</span>
      </Button>
      <Popover>
        <ListBox>
          <ListBoxItem>Apple</ListBoxItem>
          <ListBoxItem>Banana</ListBoxItem>
          <ListBoxItem>Cherry</ListBoxItem>
        </ListBox>
      </Popover>
    </Select>
  );
}
```

**Component Breakdown:**
- `<Select>`: Manages selection state and accessibility.
- `<ListBox>` and `<ListBoxItem>`: Full keyboard navigation (arrow keys, Home/End).
- All components are unstyled; you provide CSS.

**Ark UI — Accordion (Cross-Framework):**
```jsx
import { Accordion } from '@ark-ui/react/accordion';

function MyAccordion() {
  return (
    <Accordion.Root collapsible>
      <Accordion.Item value="item-1">
        <Accordion.ItemTrigger>Section 1</Accordion.ItemTrigger>
        <Accordion.ItemContent>Content 1</Accordion.ItemContent>
      </Accordion.Item>
      <Accordion.Item value="item-2">
        <Accordion.ItemTrigger>Section 2</Accordion.ItemTrigger>
        <Accordion.ItemContent>Content 2</Accordion.ItemContent>
      </Accordion.Item>
    </Accordion.Root>
  );
}
```

**Component Breakdown:**
- Same API works in React, Vue, and Solid.
- `collapsible`: Allows all items to be closed.
- Each item manages its own expanded state and ARIA attributes.

**Syntax Rules:**
- Never mix headless libraries in one component tree (React Aria dropdown inside a Radix modal causes focus-trap conflicts).
- Use `asChild` (Radix) or the render-prop pattern (Base UI) to compose with your own elements.
- Provide accessible names for all interactive components (`aria-label`, `<Label>`).
- Style with your preferred CSS solution (Tailwind, CSS Modules, etc.).
- Test keyboard navigation (Tab, Enter, Space, Escape, Arrow keys) for every primitive.

**Constraints and Limitations:**
- **Radix UI:** Acquired by WorkOS; updates have slowed for some components; the team moved to Base UI.
- **React Aria:** More engineering time required; steeper learning curve; fewer pre-built visual examples.
- **Ark UI:** Smaller community than Radix; fewer component examples.
- **Base UI:** Newer; fewer community resources; API may evolve.
- **All:** No default styling means more CSS work; accessibility still requires you to provide labels and contrast.

### Annotated Code Example: Radix Dialog with Custom Styles

```jsx
import * as Dialog from '@radix-ui/react-dialog';
import './dialog.css';

export function ConfirmDialog({ onConfirm, children }) {
  return (
    <Dialog.Root>
      <Dialog.Trigger asChild>{children}</Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Overlay className="dialog-overlay" />
        <Dialog.Content className="dialog-content" aria-describedby="confirm-desc">
          <Dialog.Title className="dialog-title">Confirm Deletion</Dialog.Title>
          <p id="confirm-desc" className="dialog-desc">
            This action cannot be undone. Are you sure?
          </p>
          <div className="dialog-actions">
            <Dialog.Close asChild>
              <button className="btn btn-secondary">Cancel</button>
            </Dialog.Close>
            <Dialog.Close asChild>
              <button className="btn btn-danger" onClick={onConfirm}>
                Delete
              </button>
            </Dialog.Close>
          </div>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

```css
/* dialog.css */
.dialog-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  animation: fadeIn 150ms ease;
}

.dialog-content {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  border-radius: 12px;
  padding: 24px;
  max-width: 400px;
  width: 90vw;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
  animation: slideUp 200ms ease;
}

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes slideUp { from { transform: translate(-50%, -40%); opacity: 0; } to { transform: translate(-50%, -50%); opacity: 1; } }
```

**Expected Output:** A "Delete" button that opens a modal dialog with a backdrop, title, description, and Cancel/Delete buttons. Focus is trapped inside the dialog, Escape closes it, and focus returns to the trigger when closed. The dialog has custom animations (fade-in backdrop, slide-up content).

**Why This Output Occurs:** Radix handles all behaviour—focus trapping, Escape key, ARIA attributes, portal rendering. The CSS files provide all visual styling. The `asChild` prop composes the trigger and close buttons with custom `<button>` elements while preserving Radix's behaviour.

### Real-World Cases

- **shadcn/ui:** Built on Radix UI + Tailwind CSS; the most popular copy-paste component collection.
- **Government applications:** React Aria for WCAG 2.2 AA/AAA compliance and i18n-heavy requirements.
- **Cross-framework design systems:** Ark UI for teams maintaining React, Vue, and Solid applications.
- **Enterprise design systems:** Base UI as a maintained Radix successor for long-term projects.
- **Accessibility-critical apps:** React Aria for screen-reader edge cases and keyboard navigation.

### References

- ai-web-design-codex – Component Libraries & Headless UI (2025–2026): https://github.com/Eneryleen/ai-web-design-codex
- LogRocket – Headless UI alternatives: Radix Primitives vs. React Aria vs. Ark UI vs. Base UI (March 2026): https://blog.logrocket.com
- GreatFrontend – Top Headless UI libraries for React in 2026: https://www.greatfrontend.com
- DesignRevision – Base UI Alternatives: 8 Headless Component Libraries (2026): https://designrevision.com
- Radix UI – Accessibility: https://mintlify.wiki/radix-ui/primitives/accessibility

---

## Core Concept 3: Atomic Design Systems — Architecture, Deployment, and Composition

### Definitions

**Core Definition:** Atomic Design is a methodology for organising component libraries into five hierarchical levels—Atoms, Molecules, Organisms, Templates, and Pages—creating a clear, scalable hierarchy from indivisible UI elements to complete application screens.

**Technical Definition:** Atomic Design methodology organises components based on complexity and reusability: **Atoms** are basic building blocks that cannot be broken down further (Button, Input, Icon, Label, Badge, Avatar, Skeleton, Spinner); **Molecules** are groups of atoms functioning together as a unit (Card, FormField, Alert, Dropdown, Modal, SearchInput, Tabs); **Organisms** are complex components combining molecules and atoms into distinct sections (FilterPanel, ProductGrid, Navigation, ShoppingCart, CheckoutForm, DataTable); **Templates** are page-level layouts without real content (MainLayout, ShopLayout, CheckoutLayout); and **Pages** are complete application screens with real content (14 pages in a typical e-commerce app). This hierarchy reduces confusion, enforces single responsibility, and makes components easy to locate and update.

**Beginner-Friendly Explanation:** Atomic Design is like building with Lego. **Atoms** are the individual bricks (a single button, an input). **Molecules** are small assemblies (a search bar with a button). **Organisms** are larger structures (a navigation bar with logo, links, and search). **Templates** are the blueprints for a page (where the header, sidebar, and content go). **Pages** are the finished model with all the details filled in. You start with the smallest pieces and build up, so everything is consistent and reusable.

### Purposes

- To create a clear component hierarchy that reduces confusion and enforces single responsibility.
- To maximise reusability by designing atoms for multiple contexts.
- To enable rapid development by composing larger components from smaller, tested ones.
- To maintain consistency through shared prop patterns and design language.
- To support tree-shaking, lazy loading, and minimal bundle size.
- To make onboarding easier with intuitive component discovery and clear import paths.

### Syntax Rules and Structure

**Directory Structure:**
```
src/components/
├── atoms/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.stories.tsx
│   │   └── index.ts
│   ├── Input/
│   ├── Icon/
│   └── Badge/
├── molecules/
│   ├── Card/
│   ├── FormField/
│   ├── Alert/
│   └── Modal/
├── organisms/
│   ├── FilterPanel/
│   ├── ProductGrid/
│   ├── Navigation/
│   └── DataTable/
├── templates/
│   ├── MainLayout/
│   └── ShopLayout/
└── pages/
    ├── HomePage/
    └── ProductPage/
```

**Component Breakdown:**
- **Atoms:** `Button.tsx`, `Input.tsx`, `Icon.tsx` — no margins, no positioning; pure presentation.
- **Molecules:** `Card.tsx` composes `Button`, `Text`, and `Badge` into a simple unit.
- **Organisms:** `ProductGrid.tsx` composes `Card` molecules into a grid with filtering logic.
- **Templates:** `MainLayout.tsx` defines the page shell (header, sidebar, main, footer).
- **Pages:** `HomePage.tsx` fills `MainLayout` with real content.

**Atom Example (Button):**
```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

export function Button({ variant = 'primary', size = 'md', disabled, children, onClick }: ButtonProps) {
  return (
    <button
      className={`btn btn--${variant} btn--${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

**Molecule Example (SearchField):**
```tsx
import { Input } from '../atoms/Input';
import { Button } from '../atoms/Button';

export function SearchField({ onSearch }: { onSearch: (query: string) => void }) {
  const [query, setQuery] = React.useState('');

  return (
    <div className="search-field">
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <Button onClick={() => onSearch(query)}>Search</Button>
    </div>
  );
}
```

**Organism Example (Navigation):**
```tsx
import { SearchField } from '../molecules/SearchField';
import { Icon } from '../atoms/Icon';

export function Navigation({ onSearch }: { onSearch: (q: string) => void }) {
  return (
    <nav className="navigation">
      <Icon name="logo" />
      <ul className="nav-links">
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
      </ul>
      <SearchField onSearch={onSearch} />
    </nav>
  );
}
```

**Syntax Rules:**
- Atoms should be written without margins; spacing is the responsibility of the parent.
- Each component has its own folder with component, tests, stories, and index.
- Use PascalCase for component files and folders; use camelCase for utility files.
- Compound components use dot notation (e.g., `Menu.Item`, `Tabs.Panel`).
- Boolean props start with `is`, `has`, or `should`; event handlers start with `on`.
- Follow the "three uses rule" before abstracting: do not abstract until a pattern appears three times.

**Constraints and Limitations:**
- Initial setup time: required upfront planning and component categorisation decisions.
- Learning curve: teams must understand atomic design principles.
- Over-abstraction risk: temptation to over-engineer simple components.
- Strict categorisation can be confusing when components fit multiple levels (e.g., a Card can be a molecule or an organism).
- Not all projects need all five levels; small projects may only use atoms and molecules.

### Annotated Code Example: Building a Product Card

```tsx
// atoms/Badge.tsx
export function Badge({ children, variant = 'default' }: { children: React.ReactNode; variant?: string }) {
  return <span className={`badge badge--${variant}`}>{children}</span>;
}

// atoms/Button.tsx
export function Button({ children, variant = 'primary', onClick }: { children: React.ReactNode; variant?: string; onClick?: () => void }) {
  return <button className={`btn btn--${variant}`} onClick={onClick}>{children}</button>;
}

// molecules/ProductCard.tsx
import { Badge } from '../atoms/Badge';
import { Button } from '../atoms/Button';

interface ProductCardProps {
  name: string;
  price: number;
  image: string;
  isNew?: boolean;
  onAddToCart: () => void;
}

export function ProductCard({ name, price, image, isNew, onAddToCart }: ProductCardProps) {
  return (
    <div className="product-card">
      <img src={image} alt={name} className="product-card__image" />
      {isNew && <Badge variant="success">New</Badge>}
      <h3 className="product-card__title">{name}</h3>
      <p className="product-card__price">${price}</p>
      <Button onClick={onAddToCart}>Add to Cart</Button>
    </div>
  );
}

// organisms/ProductGrid.tsx
import { ProductCard } from '../molecules/ProductCard';

export function ProductGrid({ products, onAddToCart }: { products: any[]; onAddToCart: (id: number) => void }) {
  return (
    <div className="product-grid">
      {products.map((product) => (
        <ProductCard
          key={product.id}
          name={product.name}
          price={product.price}
          image={product.image}
          isNew={product.isNew}
          onAddToCart={() => onAddToCart(product.id)}
        />
      ))}
    </div>
  );
}
```

**Expected Output:** A grid of product cards, each with an image, title, price, "New" badge (if applicable), and "Add to Cart" button. The grid is responsive (1 column on mobile, 2–3 on larger screens).

**Why This Output Occurs:** `Badge` and `Button` are atoms. `ProductCard` is a molecule that composes them. `ProductGrid` is an organism that composes multiple `ProductCard` molecules. Each level adds complexity and business logic while reusing the level below.

### Real-World Cases

- **E-commerce platforms:** Atoms (buttons, inputs), molecules (product cards), organisms (product grids), templates (shop layout), pages (product detail).
- **Design systems:** Component libraries organised by atomic levels for easy discovery.
- **Storybook documentation:** Each atomic level has its own Storybook section, making the hierarchy visible.
- **Team onboarding:** New developers can find components by understanding the atomic hierarchy.

### References

- amanahfy-ui-components – React Project Architecture with Atomic Design: https://www.npmjs.com/package/amanahfy-ui-components
- ADR-012: Atomic Design System for Component Library (January 2026): https://raw.githubusercontent.com/Azure-Samples/holiday-peak-hub/refs/heads/main/docs/architecture/adrs/adr-012-atomic-design-system.md
- Front-Commerce – React components structure (June 2026): https://developers.front-commerce.com
- Andela – React app structure: Atomic design: https://www.andela.com
- Super Hexagon – Component Architecture (March 2026): https://mintlify.wiki

---

## Core Concept 4: System Tokens and Synchronisation — Typography, Spacing, Color, and Elevation

### Definitions

**Core Definition:** Design tokens are the platform-agnostic, named values that define a design system's visual language—typography, spacing, colour, elevation, and motion—stored in a three-layer hierarchy (primitive → semantic → component) and synchronised across platforms via CSS custom properties.

**Technical Definition:** Design tokens follow a three-layer hierarchy: **Primitive tokens** are raw values (e.g., `--color-blue-500: #3b82f6`, `--space-4: 16px`); **Semantic tokens** reference primitives and carry contextual meaning (e.g., `--color-text-primary: var(--color-gray-900)`, `--color-surface-elevated: var(--color-gray-50)`); **Component tokens** are specific to a component (e.g., `--button-bg: var(--color-primary)`, `--card-border: var(--color-gray-200)`). Token pipelines (Style Dictionary, Figma Tokens) automate the transformation of design tokens into CSS custom properties, iOS, and Android formats. In React, tokens are exposed via CSS variables and consumed by components through `var(--token-name)`. Light/dark themes are implemented by overriding semantic tokens under `[data-theme="dark"]`.

**Beginner-Friendly Explanation:** Design tokens are like the master recipe for your brand's look. Instead of saying "use blue #3b82f6 everywhere," you say "use the brand colour." When you change the brand colour, everything updates. The three layers are like a cooking hierarchy: raw ingredients (primitives), prepared components (semantic), and finished dishes (component tokens). This makes it easy to maintain consistency across web, mobile, and even print.

### Purposes

- To establish a single source of truth for all visual design decisions.
- To enable consistent theming across light, dark, and multi-brand modes.
- To synchronise design tokens between design tools (Figma) and code.
- To provide semantic naming that describes purpose, not appearance.
- To support multi-platform token generation (CSS, iOS, Android).
- To make token changes API changes with semantic versioning.

### Syntax Rules and Structure

**Token Hierarchy (CSS Custom Properties):**
```css
/* app/globals.css */

/* --- Layer 1: Primitive Tokens (raw values) --- */
:root {
  /* Colors */
  --color-gray-50: #fafafa;
  --color-gray-100: #f5f5f5;
  --color-gray-900: #171717;
  --color-blue-500: #3b82f6;
  --color-blue-600: #2563eb;
  --color-white: #ffffff;

  /* Spacing (8px grid) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;

  /* Typography */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;

  /* Elevation */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

/* --- Layer 2: Semantic Tokens (contextual meaning) --- */
:root {
  --color-text-primary: var(--color-gray-900);
  --color-text-secondary: var(--color-gray-600);
  --color-surface-default: var(--color-white);
  --color-surface-elevated: var(--color-gray-50);
  --color-border-default: var(--color-gray-200);
  --color-interactive-primary: var(--color-blue-500);
}

[data-theme="dark"] {
  --color-text-primary: var(--color-gray-50);
  --color-text-secondary: var(--color-gray-400);
  --color-surface-default: var(--color-gray-900);
  --color-surface-elevated: var(--color-gray-800);
  --color-border-default: var(--color-gray-700);
  --color-interactive-primary: var(--color-blue-400);
}

/* --- Layer 3: Component Tokens (component-specific) --- */
:root {
  --button-bg-primary: var(--color-interactive-primary);
  --button-text-primary: var(--color-white);
  --button-padding: var(--space-2) var(--space-4);
  --card-bg: var(--color-surface-elevated);
  --card-border: var(--color-border-default);
  --card-shadow: var(--shadow-md);
}
```

**Component Breakdown:**
- **Primitive layer:** Raw values with no semantic meaning.
- **Semantic layer:** Contextual names that reference primitives; override under `[data-theme="dark"]`.
- **Component layer:** Specific values for individual components.
- Changing `--color-blue-500` in the primitive layer propagates through semantic and component layers.

**React Theme Provider:**
```tsx
import { createContext, useContext, useEffect, useState } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    return localStorage.getItem('theme') || 'light';
  });

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  return useContext(ThemeContext);
}
```

**Component Breakdown:**
- `theme`: 'light' or 'dark', initialised from `localStorage`.
- `useEffect`: Sets `data-theme` on `<html>` and persists to `localStorage`.
- Components reference semantic tokens (`var(--color-surface-default)`), which change automatically.

**Style Dictionary Configuration (Token Pipeline):**
```javascript
// style-dictionary.config.js
module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'src/styles/',
      files: [
        { destination: 'tokens.css', format: 'css/variables' },
      ],
    },
    ios: {
      transformGroup: 'ios',
      buildPath: 'ios/',
      files: [{ destination: 'Tokens.swift', format: 'ios-swift/class.swift' }],
    },
    android: {
      transformGroup: 'android',
      buildPath: 'android/',
      files: [{ destination: 'tokens.xml', format: 'android/resources' }],
    },
  },
};
```

**Component Breakdown:**
- `source`: The token JSON files (exported from Figma).
- `platforms.css`: Generates CSS custom properties.
- `platforms.ios` / `platforms.android`: Generate platform-specific formats.

**Syntax Rules:**
- **Name by purpose, not appearance:** Use `--color-text-primary`, not `--color-dark-gray`.
- **Maintain the three-layer hierarchy:** Primitives → Semantic → Component.
- **Never hardcode values in components:** Always reference a token.
- **Version tokens:** Treat changes as API changes with semantic versioning.
- **Test all theme combinations:** Verify every component in light, dark, and any brand themes.
- **Automate the pipeline:** CI/CD for Figma-to-code synchronisation.

**Constraints and Limitations:**
- Token sprawl: too many tokens without clear hierarchy.
- Inconsistent naming: mixed conventions (camelCase vs. kebab-case).
- Missing dark mode: tokens that don't adapt to theme changes.
- Hardcoded values: using raw values instead of tokens.
- Circular references: tokens referencing each other in a loop.
- Token pipeline complexity: automation requires tooling and maintenance.

### Annotated Code Example: Complete Token System with Dark Mode

```css
/* tokens.css */

/* ===== Primitive Tokens ===== */
:root {
  /* Color Primitives */
  --gray-50: #fafafa;
  --gray-100: #f5f5f5;
  --gray-200: #e5e5e5;
  --gray-400: #a3a3a3;
  --gray-600: #525252;
  --gray-800: #262626;
  --gray-900: #171717;
  --blue-400: #60a5fa;
  --blue-500: #3b82f6;
  --blue-600: #2563eb;
  --white: #ffffff;

  /* Spacing Primitives (8px grid) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;

  /* Typography Primitives */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;

  /* Elevation Primitives */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

/* ===== Semantic Tokens — Light ===== */
:root {
  --color-text-primary: var(--gray-900);
  --color-text-secondary: var(--gray-600);
  --color-surface-default: var(--white);
  --color-surface-elevated: var(--gray-50);
  --color-border-default: var(--gray-200);
  --color-interactive-primary: var(--blue-500);
  --color-interactive-hover: var(--blue-600);
}

/* ===== Semantic Tokens — Dark ===== */
[data-theme="dark"] {
  --color-text-primary: var(--gray-50);
  --color-text-secondary: var(--gray-400);
  --color-surface-default: var(--gray-900);
  --color-surface-elevated: var(--gray-800);
  --color-border-default: var(--gray-700);
  --color-interactive-primary: var(--blue-400);
  --color-interactive-hover: var(--blue-500);
}

/* ===== Component Tokens ===== */
:root {
  --button-bg: var(--color-interactive-primary);
  --button-bg-hover: var(--color-interactive-hover);
  --button-text: var(--white);
  --button-padding: var(--space-2) var(--space-4);

  --card-bg: var(--color-surface-elevated);
  --card-border: 1px solid var(--color-border-default);
  --card-shadow: var(--shadow-md);
  --card-padding: var(--space-6);
  --card-radius: 12px;
}
```

```tsx
// App.tsx
import { ThemeProvider, useTheme } from './ThemeProvider';

function Card({ title, children }) {
  return (
    <div style={{
      background: 'var(--card-bg)',
      border: 'var(--card-border)',
      boxShadow: 'var(--card-shadow)',
      padding: 'var(--card-padding)',
      borderRadius: 'var(--card-radius)',
    }}>
      <h2 style={{ fontSize: 'var(--font-size-xl)', color: 'var(--color-text-primary)' }}>
        {title}
      </h2>
      <p style={{ color: 'var(--color-text-secondary)' }}>{children}</p>
    </div>
  );
}

function ThemeToggle() {
  const { theme, setTheme } = useTheme();
  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      style={{
        background: 'var(--button-bg)',
        color: 'var(--button-text)',
        padding: 'var(--button-padding)',
        border: 'none',
        borderRadius: 8,
        cursor: 'pointer',
      }}
    >
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}

export default function App() {
  return (
    <ThemeProvider>
      <div style={{ padding: 'var(--space-8)' }}>
        <ThemeToggle />
        <div style={{ marginTop: 'var(--space-4)' }}>
          <Card title="Revenue">Monthly recurring revenue is up 12%.</Card>
        </div>
      </div>
    </ThemeProvider>
  );
}
```

**Expected Output:** A card with a background, border, shadow, padding, and rounded corners, all driven by tokens. A "Switch to dark mode" button that toggles between light and dark themes. When toggled, the card's background, text, border, and shadow all update automatically.

**Why This Output Occurs:** The `ThemeProvider` sets `data-theme="dark"` on `<html>`. The `[data-theme="dark"]` rule overrides the semantic tokens. The card's styles reference `var(--card-bg)`, `var(--card-border)`, etc., which resolve through the component → semantic → primitive chain. Changing the theme attribute re-points all variables in the same frame without re-rendering React components.

### Real-World Cases

- **Multi-brand theming:** Different brands define different primitive and semantic token values.
- **Light/dark mode:** `[data-theme="dark"]` overrides semantic tokens; components reference `var()` and update automatically.
- **Figma-to-code pipeline:** Style Dictionary transforms Figma tokens into CSS, iOS, and Android formats.
- **Design system documentation:** Storybook reads the token hierarchy to document the design system.
- **White-label apps:** Tenants customise tokens without code changes.

### References

- wshobson/agents – Design System Patterns: https://raw.githubusercontent.com/wshobson/agents/main/plugins/ui-design/skills/design-system-patterns/SKILL.md
- @paul-portfolio/tokens – npm: https://www.npmjs.com/package/@paul-portfolio/tokens
- @dynamic-mockups/design-system – npm: https://www.npmjs.com/package/@dynamic-mockups/design-system
- @hellocode/mobile-ui-design-tokens – JSR: https://jsr.io
- uiloft – npm: https://www.npmjs.com/package/uiloft

---

## Core Concept 5: Building Highly Flexible, Fluid, and Responsive Layout Components

### Definitions

**Core Definition:** Fluid responsive layout components are React components that adapt to any screen or container size using intrinsic CSS techniques (auto-fit, minmax, clamp, container queries) rather than fixed media-query breakpoints.

**Technical Definition:** Modern responsive layout components move beyond traditional breakpoint-based media queries toward **intrinsic layouts** that use CSS Grid's `auto-fit` and `minmax` to determine column counts without explicit breakpoints, **fluid typography** using `clamp()` and container query units (`cqi`, `cqw`), and **container queries** (`@container`) that let a component respond to its own container's size rather than the viewport. Libraries like **FluviUI** use smart CSS `auto-fit` logic: `FluviDiv` with `minWidth="320px"` automatically calculates how many items fit in a row and smoothly wraps items when the screen gets too small. **Masonix** provides variable-height responsive grids, and **fluidity-ts** offers a framework-agnostic responsive toolkit with typed breakpoints and container queries.

**Beginner-Friendly Explanation:** Traditional responsive design is like a train timetable—changes happen only at specific stations (breakpoints). Fluid responsive design is like a river—it flows smoothly and adapts to any width continuously. Instead of saying "at 768px, show 2 columns," you say "show as many columns as fit, with each at least 320px wide." The browser figures out the rest. Container queries go further: a card adapts to the width of its *container*, not the browser window, so it works in a sidebar and a main column without modification.

### Purposes

- To create layouts that adapt smoothly to any screen size without explicit breakpoints.
- To make components responsive to their own container, not the viewport.
- To reduce the number of CSS rules needed for responsive behaviour.
- To provide fluid typography and spacing that scales with the container.
- To support masonry layouts with variable-height items.
- To improve developer experience by eliminating media-query boilerplate.

### Syntax Rules and Structure

**Fluid Grid with `auto-fit` and `minmax`:**
```css
/* No media queries needed */
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
}
```

**Component Breakdown:**
- `auto-fit`: Fits as many columns as possible.
- `minmax(280px, 1fr)`: Each column is at least 280px and grows to fill space.
- The browser automatically determines the column count based on available width.

**React Component with Fluid Grid:**
```tsx
function ResponsiveGrid({ children, minWidth = '280px', gap = '1.5rem' }) {
  return (
    <div style={{
      display: 'grid',
      gridTemplateColumns: `repeat(auto-fit, minmax(${minWidth}, 1fr))`,
      gap,
    }}>
      {children}
    </div>
  );
}

// Usage
<ResponsiveGrid minWidth="320px" gap="2rem">
  <Card>Plan 1</Card>
  <Card>Plan 2</Card>
  <Card>Plan 3</Card>
</ResponsiveGrid>
```

**Component Breakdown:**
- `minWidth`: The minimum width each item should have before wrapping.
- `gap`: The spacing between grid items.
- The grid automatically calculates how many items fit.

**Fluid Typography with `clamp()`:**
```css
.fluid-heading {
  font-size: clamp(1.5rem, 4vw + 1rem, 3rem);
}

.fluid-body {
  font-size: clamp(1rem, 1.5vw + 0.5rem, 1.25rem);
}
```

**Component Breakdown:**
- `clamp(min, preferred, max)`: The font size scales between `min` and `max` based on the preferred value.
- `4vw + 1rem`: Uses viewport width for fluid scaling.

**Container Queries for Component Responsiveness:**
```css
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card { display: flex; flex-direction: row; gap: 16px; }
  .card__image { width: 160px; height: 160px; }
}

@container card (max-width: 399px) {
  .card { display: flex; flex-direction: column; }
  .card__image { width: 100%; aspect-ratio: 16/9; }
}
```

**Component Breakdown:**
- `container-type: inline-size`: Declares the wrapper as a query container.
- `@container card (min-width: 400px)`: Applies when the container is at least 400px wide.
- The component adapts to its own container, not the viewport.

**FluviUI — Zero Media Query Layout:**
```tsx
import { FluviDiv } from 'fluvi-ui';

function PricingPage() {
  return (
    <div className="p-10">
      <h1 className="text-3xl font-extrabold mb-8 text-center">Our Pricing Plans</h1>
      <FluviDiv minWidth="320px" gap="2rem" className="max-w-7xl mx-auto">
        <div className="card bg-base-200 p-8 rounded-2xl shadow-xl">
          <h2 className="text-2xl font-bold">Starter</h2>
          <p className="text-4xl font-black my-4">$29</p>
          <button className="btn btn-primary w-full">Choose Plan</button>
        </div>
        {/* More pricing cards */}
      </FluviDiv>
    </div>
  );
}
```

**Component Breakdown:**
- `<FluviDiv>`: A fluid layout engine that wraps children in a smart grid.
- `minWidth="320px"`: The minimum width per item.
- `gap="2rem"`: Spacing between items.
- No media queries required.

**Syntax Rules:**
- Use `auto-fit` + `minmax()` for grid layouts that adapt automatically.
- Use `clamp()` for fluid typography and spacing.
- Use `container-type: inline-size` on wrappers where container queries are used.
- Use `@container` instead of `@media` for component-level responsiveness.
- Use `cqi`/`cqw` units for fluid values relative to the container.
- Always declare `container-type` on the wrapper; without it, queries never match.
- Test with `@supports` for progressive enhancement.

**Constraints and Limitations:**
- `auto-fit` + `minmax()` cannot control the exact number of columns at specific breakpoints.
- Container queries require a defined container (`container-type`); forgetting it means the query never matches.
- Size containment (`container-type: size`) can affect percentage and overflow behaviour; `inline-size` is usually safer.
- Media queries remain better for page-level layout and `prefers-*` features.
- Fluid typography can harm readability if the min and max bounds are too far apart.

### Annotated Code Example: Responsive Pricing Grid with Container Queries

```tsx
// Card.tsx
import './Card.css';

export function Card({ title, price, features }: { title: string; price: number; features: string[] }) {
  return (
    <div className="card-wrapper">
      <div className="card">
        <h3 className="card__title">{title}</h3>
        <p className="card__price">${price}/mo</p>
        <ul className="card__features">
          {features.map((f) => <li key={f}>{f}</li>)}
        </ul>
        <button className="card__cta">Choose Plan</button>
      </div>
    </div>
  );
}
```

```css
/* Card.css */
.card-wrapper {
  container-type: inline-size;
  container-name: pricing;
}

.card {
  background: var(--color-surface-elevated);
  border: 1px solid var(--color-border-default);
  border-radius: 12px;
  padding: 24px;
}

@container pricing (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 24px;
  }
  .card__features {
    flex: 1;
  }
  .card__cta {
    align-self: center;
  }
}

@container pricing (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
    text-align: center;
  }
}
```

**Expected Output:** Pricing cards that display vertically (stacked) in a narrow container and horizontally (side-by-side with features and CTA) in a wide container. The same component works in both contexts without modifier classes.

**Why This Output Occurs:** The `.card-wrapper` declares `container-type: inline-size`, making it a query container. The `@container pricing (min-width: 400px)` and `@container pricing (max-width: 399px)` rules apply based on the container's width. The card adapts to its container, not the viewport.

### Real-World Cases

- **Pricing pages:** Grids that automatically adjust column count based on available width.
- **Product listings:** Product cards that switch between horizontal and vertical layouts based on their container.
- **Dashboard widgets:** Widgets that reflow based on their grid cell width.
- **CMS components:** Modular components that can be placed in different container widths.
- **Marketing sites:** Fluid typography and spacing that scale smoothly across devices.

### References

- FluviUI – npm: https://www.npmjs.com/package/fluvi-ui
- OrchestKit – Responsive Patterns: https://raw.githubusercontent.com/yonatangross/orchestkit/main/src/skills/responsive-patterns/SKILL.md
- lemnyscate – npm: https://socket.dev
- Masonix – npm: https://www.npmjs.com/package/masonix
- fluidity-ts – npm: https://www.npmjs.com/package/fluidity-ts
- CoreUI – Containers: https://coreui.io

---

## Core Concept 6: Accessibility Standards (a11y) — Semantic HTML, WAI-ARIA, Focus Traps, and Keyboard Testing

### Definitions

**Core Definition:** Accessibility (a11y) in UI component systems is the practice of ensuring that every component is perceivable, operable, and understandable by all users, including those using screen readers, keyboards, and other assistive technologies, through semantic HTML, WAI-ARIA patterns, focus management, and rigorous testing.

**Technical Definition:** Web Content Accessibility Guidelines (WCAG) 2.2 AA compliance requires semantic HTML as the foundation, supplemented by WAI-ARIA when native elements are insufficient. **Semantic HTML** uses `<button>`, `<nav>`, `<main>`, `<dialog>`, and other native elements that carry built-in accessibility semantics. **WAI-ARIA** provides roles (`role="dialog"`, `role="switch"`), states (`aria-expanded`, `aria-checked`, `aria-invalid`), and properties (`aria-labelledby`, `aria-describedby`, `aria-modal`). **Focus trap management** ensures keyboard focus stays within modal dialogs, dropdowns, and other overlay components, restoring focus to the trigger when closed. **Keyboard navigation testing** verifies that all interactive elements are reachable and operable via Tab, Enter, Space, Escape, Arrow keys, Home, and End. Tools include `@axe-core/react` for development-time checks, `jest-axe` for unit tests, and manual testing with screen readers (NVDA, JAWS, VoiceOver).

**Beginner-Friendly Explanation:** Accessibility is about making sure everyone can use your app, not just people with a mouse and perfect vision. Semantic HTML is like using the right tool for the job—a `<button>` is a button, not a `<div>` pretending to be one. WAI-ARIA is extra labelling that tells screen readers "this is a dialog" or "this is expanded." Focus traps keep keyboard focus inside a modal so users don't get lost. And testing means actually trying to use your app with a keyboard and a screen reader, not just assuming it works.

### Purposes

- To ensure all interactive elements are keyboard accessible.
- To provide screen-reader users with meaningful roles, states, and labels.
- To manage focus correctly when modals, dropdowns, and other overlays open and close.
- To comply with WCAG 2.2 AA (and beyond) for legal and ethical requirements.
- To test with real assistive technologies, not just automated tools.
- To catch accessibility bugs early via `@axe-core/react` and `jest-axe`.

### Syntax Rules and Structure

**Semantic HTML First:**
```jsx
// ❌ Bad: div pretending to be a button
<div className="btn" onClick={handleClick}>Click me</div>

// ✅ Good: native button
<button className="btn" onClick={handleClick}>Click me</button>

// ❌ Bad: div with role="button" without keyboard handling
<div role="button" tabIndex={0} onClick={handleClick}>Click me</div>

// ✅ Good: native button with proper semantics
<button type="button" className="btn" onClick={handleClick}>Click me</button>
```

**WAI-ARIA for Dialogs:**
```jsx
import { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom';

const FOCUSABLE_SELECTOR = [
  'a[href]',
  'button:not([disabled])',
  'textarea:not([disabled])',
  'input:not([disabled])',
  'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(',');

function Modal({ isOpen, onClose, labelledBy, describedBy, children }) {
  const dialogRef = useRef(null);
  const previousFocusRef = useRef(null);

  useEffect(() => {
    if (!isOpen) return;

    previousFocusRef.current = document.activeElement;
    dialogRef.current?.focus();

    function handleKeyDown(e) {
      if (e.key === 'Escape') { onClose(); return; }
      if (e.key !== 'Tab') return;

      const focusable = dialogRef.current?.querySelectorAll(FOCUSABLE_SELECTOR);
      if (!focusable?.length) return;

      const first = focusable[0];
      const last = focusable[focusable.length - 1];

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault(); last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault(); first.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      previousFocusRef.current?.focus();
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="dialog-overlay" onClick={onClose}>
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby={labelledBy}
        aria-describedby={describedBy}
        tabIndex={-1}
        className="dialog-content"
        onClick={(e) => e.stopPropagation()}
      >
        {children}
      </div>
    </div>,
    document.body
  );
}
```

**Component Breakdown:**
- `role="dialog"`: Identifies the element as a dialog.
- `aria-modal="true"`: Tells assistive tech that content outside is inert.
- `aria-labelledby` / `aria-describedby`: Links the dialog to its title and description.
- `tabIndex={-1}`: Makes the dialog programmatically focusable.
- Focus trap: Tab and Shift+Tab cycle within the dialog.
- Escape closes; focus returns to the trigger on close.
- `createPortal`: Renders outside the parent DOM for z-index and stacking reasons.

**Keyboard Navigation Patterns (Radix UI):**
```jsx
// Radix handles keyboard navigation automatically
import * as Tabs from '@radix-ui/react-tabs';

<Tabs.Root defaultValue="tab1">
  <Tabs.List>
    <Tabs.Trigger value="tab1">Tab 1</Tabs.Trigger>
    <Tabs.Trigger value="tab2">Tab 2</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="tab1">Content 1</Tabs.Content>
  <Tabs.Content value="tab2">Content 2</Tabs.Content>
</Tabs.Root>
// Keyboard: Arrow Left/Right, Home, End, Tab into panel
```

**Component Breakdown:**
- Arrow Left/Right: Moves focus between tabs.
- Home/End: Focuses first/last tab.
- Tab: Moves focus into the tab panel.
- All ARIA attributes (`aria-selected`, `aria-controls`, `role="tabpanel"`) are managed automatically.

**Accessibility Testing with `jest-axe`:**
```jsx
import { render, screen } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { Button } from './Button';

expect.extend(toHaveNoViolations);

test('Button has no accessibility violations', async () => {
  const { container } = render(<Button>Click me</Button>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

test('Button is keyboard accessible', () => {
  render(<Button>Click me</Button>);
  const button = screen.getByRole('button', { name: /click me/i });
  button.focus();
  expect(button).toHaveFocus();
});
```

**Component Breakdown:**
- `axe(container)`: Runs axe-core accessibility checks on the rendered output.
- `toHaveNoViolations()`: Asserts that no violations were found.
- `getByRole('button', { name })`: Finds the button by its accessible role and name.
- `toHaveFocus()`: Verifies that the element receives keyboard focus.

**Syntax Rules:**
- Use semantic HTML first (`<button>`, `<nav>`, `<main>`, `<dialog>`).
- Use WAI-ARIA only when native semantics are insufficient.
- Always provide accessible names (`aria-label`, `aria-labelledby`, visible text).
- Use `aria-invalid` and `aria-describedby` for form validation errors.
- Trap focus in modals and restore focus to the trigger when closed.
- Test keyboard navigation: Tab, Shift+Tab, Enter, Space, Escape, Arrow keys, Home, End.
- Test with real screen readers (NVDA, JAWS, VoiceOver), not just automated tools.
- Never use `outline: none` without a visible replacement.
- Use `@axe-core/react` in development and `jest-axe` in tests.

**Constraints and Limitations:**
- Automated tools like axe-core catch only 30–50% of accessibility issues; manual testing is required.
- Focus trapping requires careful implementation; libraries like Radix and React Aria handle it correctly.
- `aria-modal="true"` is not supported by all screen readers; `aria-hidden` on background siblings is a fallback.
- Keyboard navigation patterns vary by component type; follow WAI-ARIA Authoring Practices.
- Colour contrast cannot be fully verified by automated tools; manual checking is required.
- Testing with real screen readers requires access to assistive technology and time.

### Annotated Code Example: Accessible Modal with Focus Trap and axe Testing

```tsx
import { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom';

const FOCUSABLE_SELECTOR = [
  'a[href]', 'button:not([disabled])', 'textarea:not([disabled])',
  'input:not([disabled])', 'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(',');

export function AccessibleModal({ isOpen, onClose, title, description, children }) {
  const dialogRef = useRef(null);
  const previousFocusRef = useRef(null);

  useEffect(() => {
    if (!isOpen) return;

    previousFocusRef.current = document.activeElement;
    dialogRef.current?.focus();

    function handleKeyDown(e) {
      if (e.key === 'Escape') { onClose(); return; }
      if (e.key !== 'Tab') return;

      const focusable = dialogRef.current?.querySelectorAll(FOCUSABLE_SELECTOR);
      if (!focusable?.length) return;

      const first = focusable[0];
      const last = focusable[focusable.length - 1];

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault(); last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault(); first.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      previousFocusRef.current?.focus();
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        aria-describedby="modal-desc"
        tabIndex={-1}
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
      >
        <h2 id="modal-title">{title}</h2>
        <p id="modal-desc">{description}</p>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body
  );
}
```

**Expected Output:** A modal dialog that traps focus (Tab cycles within), closes on Escape, restores focus to the trigger when closed, and has proper ARIA attributes. Screen readers announce the dialog and its title/description.

**Why This Output Occurs:** The `useEffect` captures the previously focused element, focuses the dialog, and adds a keydown listener for Escape and Tab. The Tab handler cycles focus between the first and last focusable elements. On cleanup (close), focus returns to the previously focused element. The `role="dialog"`, `aria-modal="true"`, `aria-labelledby`, and `aria-describedby` attributes provide the correct semantics for assistive technology.

### Real-World Cases

- **Modal dialogs:** Focus trapping, Escape key, focus restoration, ARIA attributes.
- **Tabs:** Arrow key navigation, Home/End, `aria-selected`, `aria-controls`, `role="tabpanel"`.
- **Accordions:** Space/Enter to toggle, Arrow Up/Down to navigate, `aria-expanded`.
- **Dropdowns and menus:** Arrow key navigation, Escape to close, `aria-haspopup`, `aria-expanded`.
- **Form validation:** `aria-invalid`, `aria-describedby`, `role="alert"` for error messages.
- **Screen reader testing:** NVDA, JAWS, VoiceOver for real-world validation.

### References

- Radix UI – Accessibility: https://mintlify.wiki/radix-ui/primitives/accessibility
- ASOasis – Building an Accessible React Modal Dialog with WAI-ARIA, Focus Management, and Portals (July 2026): https://asoasis.tech
- dev48v/portal-focus-trap – GitHub: https://github.com/dev48v/portal-focus-trap
- golid – Accessibility Documentation (February 2026): https://github.com/golid-ai/golid
- Capabilities – Accessibility: axe-core, Focus Management, Keyboard Nav, Screen-Reader Testing: https://raw.githubusercontent.com
- React Accessibility Guide 2026: WCAG Compliance for React & Next.js Apps: https://ratedwithai.com

---

## Comparison and Decision Guidance

| Concern | Recommended Approach | When to Use | Key Risk |
|---|---|---|---|
| **Pre-styled library** | MUI, Ant Design, Chakra UI | Rapid development, enterprise apps, data-heavy UIs | Design control, bundle size |
| **Headless primitives** | Radix UI, Base UI, React Aria, Ark UI | Custom design systems, full styling control | More engineering time |
| **Copy-paste components** | shadcn/ui | Most new applications (default stack) | Manual updates |
| **Atomic Design** | 5-level hierarchy | Component libraries, design systems | Over-abstraction risk |
| **Design tokens** | 3-layer hierarchy (primitive → semantic → component) | All design systems | Token sprawl |
| **Fluid layouts** | `auto-fit` + `minmax()`, `clamp()`, container queries | Responsive components | `container-type` required |
| **Accessibility** | Semantic HTML + WAI-ARIA + focus trap + testing | All applications | Automated tools catch only 30–50% |
| **Focus management** | Focus trap in modals, restore on close | Modals, dropdowns, overlays | Complex to implement manually |
| **Screen reader testing** | NVDA, JAWS, VoiceOver | Accessibility-critical apps | Requires access to AT |

**Decision Guidance:**
- **Start with shadcn/ui + Tailwind + Radix** for most new applications; it is the default stack and the most flexible.
- **Use MUI** for enterprise applications with data-heavy components (Data Grid, Charts) and Material Design requirements.
- **Use Ant Design** for dense, data-intensive admin panels and back-office UIs.
- **Use Chakra UI** for accessibility-forward SaaS products (greenfield only; v2 to v3 is a full rewrite).
- **Use React Aria** for government, healthcare, or i18n-heavy applications requiring WCAG 2.2 AA/AAA.
- **Use Ark UI** for cross-framework design systems (React, Vue, Solid).
- **Use Base UI** as a maintained Radix successor for long-term projects.
- **Never mix headless libraries** in one component tree; focus-trap conflicts will occur.
- **Follow the Atomic Design hierarchy** for component libraries with 50+ components.
- **Define design tokens in a 3-layer hierarchy** and expose them as CSS custom properties.
- **Use container queries** for reusable components; use media queries for page layout and `prefers-*`.
- **Test with real screen readers** and keyboard-only navigation; automated tools are not sufficient.

---

## References

- AI-Agents-public – Component Library Comparison Guide (January 2026): https://github.com/vasilyu1983/AI-Agents-public/blob/3424d6f5e94010409da012eeb1cb84aaecec88b3/frameworks/shared-skills/skills/software-ui-ux-design/references/component-library-comparison.md
- react-kb – UI Libraries Overview (May 2026): https://github.com/Nguyen-Mau-Anh/react-kb
- Mantine vs Chakra UI vs MUI (2026): https://adminlte.io
- PkgPulse – Best React UI Libraries: MUI vs Ant Design (2026): https://www.pkgpulse.com
- MUI vs Ant Design vs Mantine vs Chakra: The React UI Library Showdown: https://dev.to
- ai-web-design-codex – Component Libraries & Headless UI (2025–2026): https://github.com/Eneryleen/ai-web-design-codex
- LogRocket – Headless UI alternatives: Radix Primitives vs. React Aria vs. Ark UI vs. Base UI (March 2026): https://blog.logrocket.com
- GreatFrontend – Top Headless UI libraries for React in 2026: https://www.greatfrontend.com
- DesignRevision – Base UI Alternatives: 8 Headless Component Libraries (2026): https://designrevision.com
- Radix UI – Accessibility: https://mintlify.wiki/radix-ui/primitives/accessibility
- amanahfy-ui-components – React Project Architecture with Atomic Design: https://www.npmjs.com/package/amanahfy-ui-components
- ADR-012: Atomic Design System for Component Library (January 2026): https://raw.githubusercontent.com/Azure-Samples/holiday-peak-hub/refs/heads/main/docs/architecture/adrs/adr-012-atomic-design-system.md
- Front-Commerce – React components structure (June 2026): https://developers.front-commerce.com
- Andela – React app structure: Atomic design: https://www.andela.com
- Super Hexagon – Component Architecture (March 2026): https://mintlify.wiki
- wshobson/agents – Design System Patterns: https://raw.githubusercontent.com/wshobson/agents/main/plugins/ui-design/skills/design-system-patterns/SKILL.md
- @paul-portfolio/tokens – npm: https://www.npmjs.com/package/@paul-portfolio/tokens
- @dynamic-mockups/design-system – npm: https://www.npmjs.com/package/@dynamic-mockups/design-system
- @hellocode/mobile-ui-design-tokens – JSR: https://jsr.io
- uiloft – npm: https://www.npmjs.com/package/uiloft
- FluviUI – npm: https://www.npmjs.com/package/fluvi-ui
- OrchestKit – Responsive Patterns: https://raw.githubusercontent.com/yonatangross/orchestkit/main/src/skills/responsive-patterns/SKILL.md
- lemnyscate – npm: https://socket.dev
- Masonix – npm: https://www.npmjs.com/package/masonix
- fluidity-ts – npm: https://www.npmjs.com/package/fluidity-ts
- CoreUI – Containers: https://coreui.io
- ASOasis – Building an Accessible React Modal Dialog with WAI-ARIA, Focus Management, and Portals (July 2026): https://asoasis.tech
- dev48v/portal-focus-trap – GitHub: https://github.com/dev48v/portal-focus-trap
- golid – Accessibility Documentation (February 2026): https://github.com/golid-ai/golid
- Capabilities – Accessibility: axe-core, Focus Management, Keyboard Nav, Screen-Reader Testing: https://raw.githubusercontent.com
- React Accessibility Guide 2026: WCAG Compliance for React & Next.js Apps: https://ratedwithai.com