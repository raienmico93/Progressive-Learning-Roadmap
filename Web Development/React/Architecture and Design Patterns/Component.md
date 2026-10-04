# Modern Component Architecture & Layout: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Modern Component Architecture & Layout is the discipline of organising a React application's components, logic, and assets into a coherent, scalable structure that reflects business domains, separates concerns across rendering environments, and optimises for performance and maintainability.

**Technical Definition:** Modern React component architecture encompasses several orthogonal concerns: **component composition** (Atomic Design, feature-based co-location), **horizontal layering** (presentation, domain, infrastructure), **vertical slicing** (bounded contexts, Feature-Sliced Design), **rendering environment boundaries** (React Server Components, `'use client'` directives, serialisation constraints), and **progressive hydration strategies** (islands architecture, selective hydration). These concerns are not mutually exclusive; production applications typically combine several patterns. The architectural decision space has expanded significantly with React 19's stabilisation of Server Components, which introduce a module-graph-level boundary that determines whether component code ships to the browser. Modern folder structures increasingly favour feature-based or domain-oriented organisation over traditional type-based separation (components/, hooks/, utils/), because type-based structures scatter related code across multiple directories and violate cohesion.

**Beginner-Friendly Explanation:** As your React app grows, how you organise your files matters enormously. A good structure makes it easy to find things, safe to change things, and fast for new developers to understand. There's no single "right" way — some teams organise by UI building blocks (Atomic Design), some by business features (feature-based), and some by layers of responsibility (domain, infrastructure, UI). With React Server Components, there's now an extra dimension: some components run only on the server and never ship JavaScript to the browser, while others run on both. Understanding these patterns helps you choose the right structure for your project's size and needs.

### Key Characteristics

- **Cohesion Over Separation:** Related code (component, hook, test, styles) lives together in feature or domain folders, rather than being scattered across type-based directories.
- **Explicit Boundaries:** Architectural boundaries — between features, between layers, and between server and client — are enforced by folder structure, import rules, and directives like `'use client'`.
- **Rendering-Environment Awareness:** Components are classified by where they run (server-only, client-only, or both), and data flows across boundaries through serialisable props.
- **Progressive Enhancement:** Islands architecture and selective hydration allow static content to render immediately while interactive regions hydrate independently, reducing JavaScript payload and improving Time to Interactive.
- **Composition Over Configuration:** Atomic Design and feature-based structures rely on composition (children, slots, render props) rather than complex prop APIs, keeping components flexible and reusable.

### Prerequisites

- Solid understanding of React components, props, state, and Hooks.
- Familiarity with JavaScript/TypeScript module systems and import/export syntax.
- Basic understanding of server-side rendering (SSR) and client-side hydration.
- Awareness of Domain-Driven Design (DDD) concepts (bounded contexts, entities, use cases) for layered and domain-oriented architectures.
- Experience with a build tool (Vite, Next.js) that supports the chosen architecture.

### Related Programming Areas

- **Domain-Driven Design (DDD):** Bounded contexts, entities, value objects, use cases.
- **Clean Architecture:** Layer separation, dependency inversion, testable business logic.
- **Feature-Sliced Design (FSD):** A specific methodology for feature-oriented frontend organisation.
- **Server-Side Rendering (SSR):** Rendering React on the server for performance and SEO.
- **Progressive Hydration:** Hydrating interactive regions of a page independently.
- **Monorepo Management:** Organising multiple packages and apps in a single repository.

### Core Concepts / Features

1. Atomic Design
2. Feature-Based Architecture
3. Layered & Domain-Oriented Organization
4. React Server Components (RSC) Architecture
5. Isomorphic & Islands Architecture

---

## Core Concept 1: Atomic Design

### Definitions

**Core Definition:** Atomic Design is a methodology for structuring UI components into five hierarchical levels — Atoms, Molecules, Organisms, Templates, and Pages — based on their composition complexity and reusability.

**Technical Definition:** Atomic Design was introduced by Brad Frost and organises components by their level of abstraction: **Atoms** are the smallest indivisible UI elements (Button, Input, Label) that do not depend on other components; **Molecules** are simple combinations of atoms that form a functional unit (SearchBar, FormField); **Organisms** are complex compositions of molecules and atoms that form distinct sections of an interface (Header, ProductCard, DiaryCard); **Templates** are page-level layout structures that arrange organisms into a reusable skeleton; and **Pages** are specific instances of templates populated with real data. The critical architectural rule is one-directional dependency: higher-level components may import lower-level components, but lower-level components must never import higher-level components. This ensures atoms remain reusable across all contexts.

**Beginner-Friendly Explanation:** Atomic Design is like building with LEGO. Atoms are the individual bricks (buttons, inputs, labels). Molecules are small assemblies (a search bar made from an input and a button). Organisms are larger structures (a header made from a logo, navigation, and search bar). Templates are the layout blueprint, and pages are the finished model with all the details filled in. The rule is simple: you can use small pieces to build bigger things, but you can't use a big thing inside a small piece.

### Purposes

- To organise UI components by composition level, making reuse and dependency direction explicit.
- To ensure atoms remain pure, reusable, and independent of higher-level components.
- To provide a shared vocabulary for designers and developers.
- To improve consistency across the application by building from a controlled set of atoms and molecules.
- To simplify testing by ensuring atoms and molecules have predictable, isolated behaviour.
- To support design system development, where atoms and molecules form the reusable component library.

### Syntax Rules and Structure

**Folder Structure:**

```
src/components/
├── atoms/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── index.ts
│   ├── Input/
│   └── Label/
├── molecules/
│   ├── SearchBar/
│   ├── FormField/
│   └── CardHeader/
├── organisms/
│   ├── Header/
│   ├── LoginForm/
│   └── ProductCard/
├── templates/
│   ├── PageLayout/
│   └── DashboardLayout/
└── pages/ (or app/ in Next.js App Router)
    ├── LoginPage/
    └── DashboardPage/
```

**Component Breakdown:**
- `atoms/`: Basic UI elements (Button, Input, Label). No imports from other component levels.
- `molecules/`: Combinations of atoms (SearchBar = Input + Button).
- `organisms/`: Complex compositions (Header = Logo + Navigation + SearchBar).
- `templates/`: Page-level layouts arranging organisms.
- `pages/`: Specific instances of templates with real data.

**General Syntax for an Atom:**

```tsx
// atoms/Button/Button.tsx
import type { FC } from 'react';

export type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
  className?: string;
};

export const Button: FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  children,
  onClick,
  className = '',
}) => {
  return (
    <button
      className={`btn btn--${variant} btn--${size} ${className}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
};
```

**Component Breakdown:**
- The atom has no imports from molecules, organisms, or templates.
- Props are fully typed with `FC<ButtonProps>`.
- The `className` prop allows customisation without breaking encapsulation.
- The atom is self-contained and reusable in any context.

**General Syntax for a Molecule (Composed of Atoms):**

```tsx
// molecules/SearchBar/SearchBar.tsx
import { Input } from '@/components/atoms/Input';
import { Button } from '@/components/atoms/Button';

export type SearchBarProps = {
  value: string;
  onChange: (value: string) => void;
  onSearch: () => void;
  placeholder?: string;
};

export function SearchBar({ value, onChange, onSearch, placeholder }: SearchBarProps) {
  return (
    <div className="search-bar">
      <Input
        value={value}
        onChange={onChange}
        placeholder={placeholder ?? 'Search...'}
      />
      <Button variant="primary" onClick={onSearch}>
        Search
      </Button>
    </div>
  );
}
```

**Component Breakdown:**
- The molecule imports atoms (`Input`, `Button`) but not organisms or templates.
- It composes atoms into a functional unit.
- Props are typed and the molecule remains reusable.

**Syntax Rules:**
- Higher-level components may import lower-level components, but never the reverse.
- Atoms must not import from molecules, organisms, templates, or pages.
- Each component level has a single responsibility: atoms are presentational, molecules add simple composition, organisms add business context.
- Components should be self-contained in their own folders with tests, stories, and an index file.
- Use TypeScript for all props; export prop types for reuse.
- Use the `@/` path alias for cross-level imports.

**Constraints and Limitations:**
- Atomic Design can be overly rigid for small applications; the overhead of five levels may not be justified.
- The line between molecules and organisms is subjective and can lead to inconsistency.
- Atomic Design focuses on UI composition; it does not address data fetching, state management, or domain logic.
- For large applications, feature-based or domain-oriented structures are often preferred because Atomic Design scatters feature-related components across multiple levels.
- Templates and pages may be redundant in frameworks like Next.js that have their own routing conventions.

### Annotated Code Examples

**Example 1: Complete Atomic Design Component Tree**

```tsx
// atoms/Input/Input.tsx
import type { FC } from 'react';

export type InputProps = {
  type?: string;
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  id?: string;
};

export const Input: FC<InputProps> = ({
  type = 'text',
  value,
  onChange,
  placeholder,
  id,
}) => (
  <input
    id={id}
    type={type}
    value={value}
    onChange={(e) => onChange(e.target.value)}
    placeholder={placeholder}
    className="input"
  />
);
```

```tsx
// atoms/Label/Label.tsx
import type { FC } from 'react';

export type LabelProps = {
  htmlFor: string;
  children: React.ReactNode;
};

export const Label: FC<LabelProps> = ({ htmlFor, children }) => (
  <label htmlFor={htmlFor} className="label">
    {children}
  </label>
);
```

```tsx
// molecules/FormField/FormField.tsx
import { Input } from '@/components/atoms/Input';
import { Label } from '@/components/atoms/Label';

export type FormFieldProps = {
  id: string;
  label: string;
  type?: string;
  value: string;
  onChange: (value: string) => void;
};

export function FormField({ id, label, type, value, onChange }: FormFieldProps) {
  return (
    <div className="form-field">
      <Label htmlFor={id}>{label}</Label>
      <Input id={id} type={type} value={value} onChange={onChange} />
    </div>
  );
}
```

```tsx
// organisms/LoginForm/LoginForm.tsx
import { FormField } from '@/components/molecules/FormField';
import { Button } from '@/components/atoms/Button';

export type LoginFormProps = {
  onSubmit: (email: string, password: string) => void;
};

export function LoginForm({ onSubmit }: LoginFormProps) {
  const [email, setEmail] = React.useState('');
  const [password, setPassword] = React.useState('');

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        onSubmit(email, password);
      }}
    >
      <FormField id="email" label="Email" value={email} onChange={setEmail} />
      <FormField
        id="password"
        label="Password"
        type="password"
        value={password}
        onChange={setPassword}
      />
      <Button type="submit">Log In</Button>
    </form>
  );
}
```

```tsx
// templates/PageLayout/PageLayout.tsx
export function PageLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="page-layout">
      <header className="page-layout__header">My App</header>
      <main className="page-layout__main">{children}</main>
      <footer className="page-layout__footer">© 2026</footer>
    </div>
  );
}
```

**Expected Output:** The `LoginForm` organism renders two `FormField` molecules, each composed of a `Label` atom and an `Input` atom, plus a `Button` atom. The `PageLayout` template wraps the form in a header-main-footer structure.

**Why This Output Occurs:** Each level imports only from the level below it. Atoms (`Input`, `Label`, `Button`) have no component imports. Molecules (`FormField`) import atoms. Organisms (`LoginForm`) import molecules and atoms. Templates (`PageLayout`) import organisms (or accept them as children). This one-directional dependency graph ensures that atoms remain reusable and that changes to higher-level components do not ripple down to lower levels.

### Real-World Cases

- **Design systems:** Building a component library where atoms (Button, Input, Icon) and molecules (Card, Modal, Tooltip) are shared across multiple applications.
- **E-commerce:** Organisms like ProductCard, CartSummary, and CheckoutForm composed from atoms and molecules.
- **SaaS dashboards:** Templates like DashboardLayout arranging organisms like StatsCard, ChartPanel, and RecentActivityTable.
- **Marketing sites:** Pages built from templates populated with content organisms (Hero, FeatureGrid, TestimonialCarousel).

---

## Core Concept 2: Feature-Based Architecture

### Definitions

**Core Definition:** Feature-based architecture organises a React application by business features, co-locating all related components, hooks, services, tests, and assets inside isolated feature folders.

**Technical Definition:** Feature-based architecture (also called feature-first or feature-sliced) replaces type-based organisation (components/, hooks/, services/) with feature-based organisation, where each feature is a self-contained module containing everything it needs. A typical feature folder includes `components/`, `hooks/`, `services/`, `utils/`, `types/`, and `__tests__/`. Features communicate through well-defined public APIs (usually an `index.ts` barrel file) and may depend on a shared layer (`shared/` or `core/`) for cross-cutting concerns like UI primitives, HTTP clients, and global types. Feature-Sliced Design (FSD) formalises this with six layers: `app` → `pages` → `widgets` → `features` → `entities` → `shared`, with strict rules about which layers may import from which. The key principle is **high cohesion within features, low coupling between features**.

**Beginner-Friendly Explanation:** Instead of putting all your buttons in one folder and all your hooks in another, feature-based architecture groups everything related to a specific feature together. If you're building a "checkout" feature, all the checkout components, hooks, API calls, and tests live in a `features/checkout/` folder. This means you can find everything you need in one place, and you can change or remove the feature without hunting through the entire codebase.

### Purposes

- To co-locate all code related to a business feature, improving discoverability and cohesion.
- To isolate features so changes to one feature do not affect others.
- To enable independent development and testing of features.
- To scale team collaboration by allowing teams to own separate feature folders.
- To reduce change amplification: a feature requirement touches files within a single folder.
- To provide clear boundaries and public APIs between features.

### Syntax Rules and Structure

**Folder Structure:**

```
src/
├── app/                    # Application bootstrap, routing, providers
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── RegisterForm.tsx
│   │   ├── hooks/
│   │   │   └── use-auth.ts
│   │   ├── services/
│   │   │   └── auth.service.ts
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   ├── utils/
│   │   │   └── auth.utils.ts
│   │   ├── __tests__/
│   │   │   └── LoginForm.test.tsx
│   │   └── index.ts        # Public API (barrel)
│   └── checkout/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       └── index.ts
├── shared/                 # Cross-feature utilities and components
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   └── types/
└── App.tsx
```

**Component Breakdown:**
- `app/`: Application-level bootstrap, routing configuration, and global providers.
- `features/`: One folder per business feature, each self-contained.
- `features/<name>/index.ts`: The public API — only what is exported here can be imported by other features.
- `shared/`: Cross-cutting concerns used by multiple features (HTTP client, design system primitives, global types).

**General Syntax for a Feature's Public API:**

```typescript
// features/auth/index.ts
// Only export what other features need
export { LoginForm } from './components/LoginForm';
export { RegisterForm } from './components/RegisterForm';
export { useAuth } from './hooks/use-auth';
export type { User, AuthState } from './types/auth.types';
// Do NOT export internal utilities, services, or private hooks
```

**Component Breakdown:**
- The barrel file exposes only the feature's public interface.
- Internal implementation details (services, utils) are not exported.
- Other features import from `@/features/auth`, never from deep paths like `@/features/auth/services/auth.service`.

**General Syntax for Feature-Sliced Design Layers:**

```
src/
├── app/          # Root configuration, providers, routing
├── pages/        # Route-level compositions (one per route)
├── widgets/      # Independent UI blocks (Header, Sidebar, Feed)
├── features/     # User interactions (AddToCart, ToggleTheme)
├── entities/     # Business entities (User, Product, Order)
└── shared/       # UI kit, libs, API clients, config
```

**Component Breakdown:**
- **app**: Initialisation, providers, global styles, routing.
- **pages**: Compositions of widgets and features for a specific route.
- **widgets**: Self-contained UI blocks that combine features and entities.
- **features**: User actions and interactions (e.g., "add to cart", "like post").
- **entities**: Business domain models (User, Product, Order) with their UI representations.
- **shared**: Reusable infrastructure (UI kit, API client, utilities).

**Syntax Rules:**
- Each feature must have a public API (`index.ts`) that defines what other parts of the app can import.
- Features must not import from other features' internal files — only from their public APIs.
- The `shared/` layer must not import from any feature.
- Feature-Sliced Design enforces a strict layering rule: a layer may only import from layers below it (e.g., `features` may import from `entities` and `shared`, but not from `pages` or `widgets`).
- Keep feature folders flat; avoid deep nesting inside features.
- Co-locate tests, stories, and styles with the components they test.

**Constraints and Limitations:**
- Feature-based architecture requires discipline: without enforced import rules, features can become coupled.
- The public API (barrel file) adds a small maintenance overhead.
- For very small applications, feature-based structure may be over-engineered.
- FSD's six layers can be confusing for teams new to the methodology; start with a simpler feature-based structure and evolve.
- Cross-feature communication (e.g., auth affecting checkout) requires careful design — typically through the `shared` layer or a global store.

### Annotated Code Examples

**Example 1: Feature-Based Auth Module**

```typescript
// features/auth/types/auth.types.ts
export type User = {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
};

export type AuthState = {
  user: User | null;
  isAuthenticated: boolean;
};
```

```typescript
// features/auth/services/auth.service.ts
import type { User } from '../types/auth.types';

export async function login(email: string, password: string): Promise<User> {
  const res = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });
  if (!res.ok) throw new Error('Invalid credentials');
  return res.json();
}
```

```typescript
// features/auth/hooks/use-auth.ts
import { useState, useCallback } from 'react';
import { login as loginService } from '../services/auth.service';
import type { User } from '../types/auth.types';

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (email: string, password: string) => {
    const userData = await loginService(email, password);
    setUser(userData);
  }, []);

  const logout = useCallback(() => setUser(null), []);

  return { user, login, logout, isAuthenticated: !!user };
}
```

```tsx
// features/auth/components/LoginForm.tsx
import { useState } from 'react';
import { useAuth } from '../hooks/use-auth';
import { Button } from '@/shared/components/Button';
import { Input } from '@/shared/components/Input';

export function LoginForm() {
  const { login } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  return (
    <form onSubmit={(e) => { e.preventDefault(); login(email, password); }}>
      <Input value={email} onChange={setEmail} placeholder="Email" />
      <Input value={password} onChange={setPassword} type="password" placeholder="Password" />
      <Button type="submit">Log In</Button>
    </form>
  );
}
```

```typescript
// features/auth/index.ts
export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/use-auth';
export type { User, AuthState } from './types/auth.types';
```

**Expected Output:** Other parts of the application can import `{ LoginForm, useAuth, User }` from `@/features/auth`. The internal `auth.service.ts` and `auth.types.ts` are not directly accessible from outside the feature.

**Why This Output Occurs:** The `index.ts` barrel file defines the public API. The `shared/` layer provides `Button` and `Input` components that are used by the auth feature but are not owned by it. The auth feature is self-contained: all its components, hooks, services, and types live within its folder. This structure makes it easy to find auth-related code and to test or remove the feature independently.

### Real-World Cases

- **SaaS applications:** Features like `billing`, `team-management`, `analytics`, and `settings` each with their own components, hooks, and services.
- **E-commerce:** Features like `cart`, `checkout`, `product-search`, and `user-reviews`.
- **Social platforms:** Features like `feed`, `messaging`, `notifications`, and `profile`.
- **Enterprise applications:** Features owned by different teams, each with its own folder and public API.
- **Monorepos:** Features as separate packages or workspace modules, shared across multiple applications.

---

## Core Concept 3: Layered & Domain-Oriented Organization

### Definitions

**Core Definition:** Layered and domain-oriented organisation combines horizontal layering (separating UI, application, domain, and infrastructure concerns) with vertical slicing (organising by business domains or bounded contexts) to create a scalable, testable architecture.

**Technical Definition:** This hybrid architecture draws from Clean Architecture and Domain-Driven Design (DDD). **Horizontal layering** separates code into layers with strict dependency rules: the **UI Layer** (React components) depends on the **Application Layer** (use cases, services), which depends on the **Domain Layer** (pure business logic, entities, value objects), while the **Infrastructure Layer** (API clients, database adapters, third-party integrations) implements interfaces defined by the domain and application layers. Dependencies always point inward toward the domain, which has zero external dependencies. **Vertical slicing** partitions the application into bounded contexts (e.g., Collection, Wishlist, Maintenance), each with its own domain, application, infrastructure, and UI layers. Bounded contexts are isolated — no imports between contexts — and communicate through shared infrastructure or events. This combination ensures that business logic is testable in isolation, features are decoupled, and the architecture scales as new domains are added.

**Beginner-Friendly Explanation:** Imagine your app is a building. Layered architecture says: keep the plumbing (infrastructure) separate from the electrical (domain logic) separate from the interior design (UI). Domain-oriented says: also keep the different departments (billing, shipping, inventory) in separate wings. Combining both means each department has its own plumbing, electrical, and design, and departments don't interfere with each other. This makes it much easier to renovate one department without shutting down the whole building.

### Purposes

- To separate business logic from UI and infrastructure, making it testable in isolation.
- To enforce dependency inversion: the domain layer defines interfaces that the infrastructure layer implements.
- To partition the application into bounded contexts that can be developed and deployed independently.
- To prevent coupling between unrelated business domains.
- To provide a clear, scalable structure that accommodates new domains without modifying existing code.
- To align the codebase with the business's language and domain model.

### Syntax Rules and Structure

**Folder Structure (Hybrid Clean Architecture + DDD):**

```
src/
├── app/                          # Application foundation
│   ├── main.tsx                  # Entry point
│   ├── App.tsx                   # Root component + routing
│   ├── providers/                # Global providers (DI, theme)
│   └── config/                   # DI container composition
├── shared/                       # Shared kernel (cross-context)
│   ├── domain/                   # Shared value objects, Result pattern
│   ├── infrastructure/           # HTTP client, IndexedDB base adapter
│   └── ui/                       # Shared UI components
├── collection/                   # BOUNDED CONTEXT: Game Collection
│   ├── domain/
│   │   ├── entities/             # Game.ts
│   │   ├── value-objects/        # GameTitle.ts, Platform.ts
│   │   └── repositories/         # IGameRepository.ts (interface)
│   ├── application/
│   │   └── use-cases/            # AddGame.ts, SearchGames.ts
│   ├── infrastructure/
│   │   ├── persistence/          # IndexedDBGameRepository.ts
│   │   ├── api/                  # IGDBAdapter.ts
│   │   └── di/                   # collection.container.ts
│   └── ui/
│       ├── pages/                # CollectionPage.tsx
│       └── components/           # GameCard.tsx
├── wishlist/                     # BOUNDED CONTEXT: Wishlist
│   └── [same layer structure]
└── maintenance/                  # BOUNDED CONTEXT: Console Maintenance
    └── [same layer structure]
```

**Component Breakdown:**
- `domain/`: Pure business logic — entities, value objects, and repository interfaces. No external dependencies.
- `application/`: Use cases that orchestrate domain logic. Depends only on the domain layer.
- `infrastructure/`: Adapters that implement domain/application interfaces (API clients, persistence).
- `ui/`: React components that depend on the application layer.
- Each bounded context (collection, wishlist, maintenance) has its own four-layer stack.
- `shared/`: Cross-context utilities (HTTP client, shared value objects, UI primitives).

**General Syntax for the Dependency Rule:**

```typescript
// domain/entities/Game.ts — Pure business logic, no imports
export class Game {
  constructor(
    public readonly id: string,
    public readonly title: string,
    public readonly platform: string,
    public readonly addedAt: Date,
  ) {}

  isRecentlyAdded(): boolean {
    const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
    return this.addedAt > thirtyDaysAgo;
  }
}

// domain/repositories/IGameRepository.ts — Interface defined by domain
import type { Game } from '../entities/Game';

export interface IGameRepository {
  findById(id: string): Promise<Game | null>;
  save(game: Game): Promise<void>;
  search(query: string): Promise<Game[]>;
}

// application/use-cases/AddGame.ts — Use case depends on domain interface
import type { IGameRepository } from '../../domain/repositories/IGameRepository';
import { Game } from '../../domain/entities/Game';

export class AddGame {
  constructor(private repository: IGameRepository) {}

  async execute(title: string, platform: string): Promise<Game> {
    const game = new Game(crypto.randomUUID(), title, platform, new Date());
    await this.repository.save(game);
    return game;
  }
}

// infrastructure/persistence/IndexedDBGameRepository.ts — Implements domain interface
import type { IGameRepository } from '../../domain/repositories/IGameRepository';
import type { Game } from '../../domain/entities/Game';

export class IndexedDBGameRepository implements IGameRepository {
  async findById(id: string): Promise<Game | null> {
    // IndexedDB implementation
  }
  async save(game: Game): Promise<void> {
    // IndexedDB implementation
  }
  async search(query: string): Promise<Game[]> {
    // IndexedDB implementation
  }
}
```

**Component Breakdown:**
- `Game`: A domain entity with pure business logic (`isRecentlyAdded`). No React, no fetch, no database.
- `IGameRepository`: An interface defined by the domain layer. The domain does not know how data is stored.
- `AddGame`: A use case that orchestrates domain logic. It depends on the `IGameRepository` interface, not on a concrete implementation.
- `IndexedDBGameRepository`: An infrastructure adapter that implements `IGameRepository` using IndexedDB.
- The dependency direction is inward: UI → Application → Domain ← Infrastructure.

**Syntax Rules:**
- Dependencies always point inward toward the domain. The domain layer has zero external dependencies.
- The infrastructure layer implements interfaces defined by the domain and application layers.
- Bounded contexts must not import from each other. Cross-context communication happens through shared infrastructure or events.
- Use path aliases (`@Collection/*`, `@Shared/*`) to enforce import boundaries.
- The domain layer must not contain React imports, `useState`, `fetch`, or any framework-specific code.
- Use dependency injection (manual DI in the infrastructure layer) to wire use cases to repository implementations.
- Value objects (e.g., `GameTitle`, `Platform`) encapsulate validation and formatting logic.

**Constraints and Limitations:**
- This architecture adds significant boilerplate; it is best suited for complex, long-lived applications with multiple domains.
- The learning curve is steep for teams unfamiliar with Clean Architecture or DDD.
- Over-engineering is a risk for small applications with a single domain.
- Dependency injection in React requires careful setup (context providers or a DI container).
- Bounded context isolation means cross-context features (e.g., "add to wishlist from the collection page") require explicit integration design.

### Annotated Code Examples

**Example 1: Complete Use Case with Dependency Inversion**

```typescript
// domain/value-objects/GameTitle.ts
export class GameTitle {
  private constructor(public readonly value: string) {}

  static create(value: string): GameTitle {
    if (!value || value.trim().length === 0) {
      throw new Error('Game title cannot be empty');
    }
    if (value.length > 200) {
      throw new Error('Game title cannot exceed 200 characters');
    }
    return new GameTitle(value.trim());
  }
}
```

```typescript
// domain/entities/Game.ts
import { GameTitle } from '../value-objects/GameTitle';

export class Game {
  constructor(
    public readonly id: string,
    public readonly title: GameTitle,
    public readonly platform: string,
    public readonly addedAt: Date,
  ) {}

  isRecentlyAdded(): boolean {
    const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
    return this.addedAt > thirtyDaysAgo;
  }
}
```

```typescript
// application/use-cases/AddGame.ts
import type { IGameRepository } from '../../domain/repositories/IGameRepository';
import { Game } from '../../domain/entities/Game';
import { GameTitle } from '../../domain/value-objects/GameTitle';

export class AddGame {
  constructor(private repository: IGameRepository) {}

  async execute(titleInput: string, platform: string): Promise<Game> {
    const title = GameTitle.create(titleInput); // Validation in value object
    const game = new Game(crypto.randomUUID(), title, platform, new Date());
    await this.repository.save(game);
    return game;
  }
}
```

```typescript
// infrastructure/di/collection.container.ts
import { IndexedDBGameRepository } from '../persistence/IndexedDBGameRepository';
import { AddGame } from '../../application/use-cases/AddGame';

export function createCollectionContainer() {
  const gameRepository = new IndexedDBGameRepository();
  const addGame = new AddGame(gameRepository);

  return { addGame, gameRepository };
}
```

```tsx
// ui/components/AddGameForm.tsx
import { useState } from 'react';
import { useCollection } from '../hooks/useCollection';

export function AddGameForm() {
  const { addGame } = useCollection();
  const [title, setTitle] = useState('');
  const [platform, setPlatform] = useState('PC');

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    try {
      await addGame.execute(title, platform);
      setTitle('');
    } catch (error) {
      alert(error.message); // Value object validation error
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={(e) => setTitle(e.target.value)} placeholder="Game title" />
      <select value={platform} onChange={(e) => setPlatform(e.target.value)}>
        <option>PC</option>
        <option>PlayStation</option>
        <option>Xbox</option>
      </select>
      <button type="submit">Add Game</button>
    </form>
  );
}
```

**Expected Output:** The form validates the game title via the `GameTitle` value object. If the title is empty or too long, an error is thrown and displayed. If valid, the game is saved via the `AddGame` use case, which delegates to the `IndexedDBGameRepository`.

**Why This Output Occurs:** The domain layer (`Game`, `GameTitle`) contains pure business logic with no external dependencies. The application layer (`AddGame`) orchestrates the use case. The infrastructure layer (`IndexedDBGameRepository`) implements the repository interface. The UI layer (`AddGameForm`) depends on the application layer. Dependencies point inward, and the domain layer is fully testable without React or a database.

### Real-World Cases

- **Enterprise SaaS:** Bounded contexts for billing, user management, analytics, and reporting, each with its own domain model.
- **E-commerce:** Bounded contexts for catalog, cart, orders, and payments, with shared infrastructure for HTTP and persistence.
- **Healthcare:** Bounded contexts for patients, appointments, prescriptions, and billing, with strict domain isolation for compliance.
- **Financial services:** Bounded contexts for accounts, transactions, and reporting, with pure domain logic for regulatory calculations.
- **Multi-tenant platforms:** Bounded contexts per tenant or per business capability, with shared infrastructure for authentication and data access.

---

## Core Concept 4: React Server Components (RSC) Architecture

### Definitions

**Core Definition:** React Server Components (RSC) is a rendering architecture that splits the component tree between a server module graph and a client module graph, keeping server-only components on the server and shipping only client components to the browser.

**Technical Definition:** React Server Components introduce a module-graph-level boundary between server and client code. Server Components run exclusively on the server, ship no JavaScript to the browser, and can access server-only resources (databases, file systems, secrets). Client Components, marked with the `'use client'` directive, run on both the server (for initial HTML) and the browser (for interactivity and hydration). The boundary is defined by the `'use client'` directive at the top of a file: when a Server Component imports a file marked with `'use client'`, that import becomes the boundary between server and client module graphs. Everything imported by a `'use client'` file (directly or indirectly) becomes part of the client bundle. Data crosses the boundary through serialisable props — only JSON-compatible values, Server Actions (functions defined with `'use server'`), and certain built-in types (Date, Map, Set) can be passed from Server to Client Components. Non-serialisable values (regular functions, class instances, Symbols) cause build or runtime errors. The server renders the component tree into an **RSC Payload**, a serialised description of the UI that references Client Components, which the client uses to render the final HTML.

**Beginner-Friendly Explanation:** Normally, all your React components run in the browser. With Server Components, some components run only on the server. They can access the database directly, use secret API keys, and don't send any JavaScript to the browser — making your app faster and more secure. But server components can't use `useState` or `onClick`. For anything interactive, you mark a file with `'use client'`, and that becomes the boundary: everything below it runs in the browser. The tricky part is that server components can only pass plain, serialisable data to client components — no functions, no class instances.

### Purposes

- To keep server-only code (database queries, secrets, large dependencies) on the server, reducing JavaScript bundle size.
- To enable direct data access from components without API layers, simplifying data fetching.
- To improve initial page load performance by shipping less JavaScript.
- To provide a clear, enforced boundary between server and client code.
- To enable streaming server rendering, where parts of the page are sent to the browser as they become ready.
- To support Server Actions for mutations without building separate API endpoints.

### Syntax Rules and Structure

**The `'use client'` Directive:**

```tsx
// app/hello.tsx — Server Component (default)
export default function Hello() {
  return <p>Hello from the server</p>;
}
```

```tsx
// app/counter.tsx — Client Component
'use client';
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}
```

**Component Breakdown:**
- Server Components are the default in frameworks that support RSC. They run only on the server and ship no JavaScript.
- `'use client'` marks a file (and everything it imports) as a Client Component. The directive must be at the very top of the file, above any imports.
- Client Components run on both the server (for HTML) and the browser (for hydration and interactivity).
- Server Components cannot use `useState`, `useEffect`, `useContext`, or browser APIs.
- Client Components cannot directly access server-only resources like databases or secrets.

**The Server/Client Boundary:**

```
Server Component
├─ Server Component
└─ Client Component (boundary — 'use client')
    └─ Client Component
```

**Component Breakdown:**
- The boundary is at the module level, not the render tree level. When a Server Component imports a `'use client'` file, that import becomes the boundary.
- Everything below the boundary is part of the client bundle.
- A Client Component can render a Server Component only if the Server Component is passed as a `children` prop from a Server Component parent.

**Serialisable Props:**

```tsx
// ✅ SAFE: Serializable props
<ClientCard
  item={{ id: 1, name: 'Game' }}    // Plain object
  tags={['featured', 'sale']}        // Array of primitives
  isActive={true}                    // Boolean
  onAction={handleClick}             // Server Action ('use server')
/>

// ❌ ERROR: Non-serializable props
<ClientCard
  onClick={() => {}}                 // Regular function — cannot serialize
  formatter={new Intl.NumberFormat()} // Class instance — cannot serialize
  icon={Symbol('star')}              // Symbol — cannot serialize
/>
```

**Component Breakdown:**
- Serializable types: primitives (string, number, boolean, null, undefined), plain objects/arrays, Server Actions, Date, Map, Set, TypedArray, ArrayBuffer.
- Non-serializable types: regular functions, class instances, Symbols, WeakMap, WeakSet.
- Use Server Actions when you need to pass callable behaviour from Server to Client Components.
- Convert class instances to plain objects before passing: `{ url: myUrl.toString() }`.

**Syntax Rules:**
- `'use client'` must be at the very top of the file, above any imports (comments are allowed above).
- `'use server'` marks a function as a Server Action, which is serialisable and callable from Client Components.
- Server Components can import Client Components; Client Components cannot import Server Components directly (but can receive them as `children` props).
- Only serialisable data can cross the Server → Client boundary.
- Define event handlers (`onClick`, `onChange`) inside Client Components, not in Server Components.
- Move `'use client'` as low in the component tree as possible to minimise the client bundle.
- Passing Server Components as `children` to Client Components allows Server Components to render inside Client Components without becoming client components themselves.

**Constraints and Limitations:**
- Server Components cannot use React Hooks that depend on client state (`useState`, `useReducer`, `useEffect`, `useContext`).
- Server Components cannot use browser APIs (`window`, `document`, `localStorage`).
- Client Components cannot directly access server-only resources (databases, secrets).
- Non-serialisable props cause build-time or runtime errors at the boundary.
- The RSC model is framework-dependent; it works in Next.js App Router and other compatible frameworks, but not in plain React SPAs without a framework.
- Server Actions are not recommended for data fetching; use Server Components for reading data and Server Actions for mutations.

### Annotated Code Examples

**Example 1: Server Component Fetching Data and Passing to Client Component**

```tsx
// app/products/page.tsx — Server Component
import { db } from '@/lib/db';
import { ProductCard } from './ProductCard';

export default async function ProductsPage() {
  // Direct database access — no API layer needed
  const products = await db.product.findMany();

  return (
    <div>
      <h1>Products</h1>
      {products.map((product) => (
        <ProductCard
          key={product.id}
          id={product.id}
          name={product.name}
          price={product.price}
        />
      ))}
    </div>
  );
}
```

```tsx
// app/products/ProductCard.tsx — Client Component
'use client';
import { useState } from 'react';
import { addToCart } from '@/app/actions';

export function ProductCard({ id, name, price }: {
  id: string;
  name: string;
  price: number;
}) {
  const [adding, setAdding] = useState(false);

  async function handleAddToCart() {
    setAdding(true);
    await addToCart(id); // Server Action
    setAdding(false);
  }

  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p>${price}</p>
      <button onClick={handleAddToCart} disabled={adding}>
        {adding ? 'Adding...' : 'Add to Cart'}
      </button>
    </div>
  );
}
```

```typescript
// app/actions.ts — Server Action
'use server';
import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export async function addToCart(productId: string) {
  await db.cartItem.create({ data: { productId } });
  revalidatePath('/cart');
}
```

**Expected Output:** The `ProductsPage` Server Component fetches products directly from the database and renders them. Each `ProductCard` is a Client Component that can be interacted with (the "Add to Cart" button). The `addToCart` Server Action is called from the client, executes on the server, and revalidates the cart path.

**Why This Output Occurs:** The `ProductsPage` runs only on the server and ships no JavaScript. The `ProductCard` is marked with `'use client'`, so it runs in the browser and can use `useState` and `onClick`. The boundary is crossed with serialisable props (`id`, `name`, `price` — all primitives). The `addToCart` Server Action is a serialisable function that the client can call over the network. This architecture keeps the database query server-side, ships minimal JavaScript, and enables interactivity where needed.

### Real-World Cases

- **E-commerce:** Server Components fetch product data directly from the database; Client Components handle "Add to Cart" interactions.
- **SaaS dashboards:** Server Components fetch analytics data; Client Components handle filters, charts, and interactive controls.
- **Content platforms:** Server Components render article content; Client Components handle comment forms and like buttons.
- **Enterprise applications:** Server Components access internal APIs and databases; Client Components handle user interactions and client-side state.

---

## Core Concept 5: Isomorphic & Islands Architecture

### Definitions

**Core Definition:** Isomorphic architecture runs the same React code on both the server and the client, while islands architecture renders most of the page as static HTML and hydrates only small, independent interactive regions ("islands").

**Technical Definition:** **Isomorphic rendering** (also called universal rendering) means that React components are rendered to HTML on the server and then hydrated on the client, combining SSR and CSR. The same component code runs in both environments, producing matching output. The server sends fully rendered HTML for fast initial paint and SEO, and the client takes over for subsequent interactions. **Islands architecture** (also called partial hydration) takes a different approach: the page is rendered as static or server-rendered HTML, and only the interactive regions — the "islands" — receive JavaScript and hydrate independently. Unlike full-page hydration, where the entire component tree hydrates as one unit, islands hydrate in isolation and execute asynchronously. This means a slow-hydrating island (e.g., a complex chart) does not block a fast one (e.g., a search box). Islands architecture is the default in frameworks like Astro and is supported in React via libraries and patterns. **Progressive hydration** is related: it hydrates the page incrementally, prioritising the parts the user is interacting with first.

**Beginner-Friendly Explanation:** Isomorphic rendering means your React components run on the server to produce HTML, then "wake up" in the browser to handle clicks and typing. Islands architecture takes this further: most of your page is just HTML with no JavaScript at all. Only the small interactive parts — like a search box or a "like" button — get JavaScript and become interactive. This is much faster because the browser isn't downloading and running JavaScript for the static parts of the page that never change.

### Purposes

- To combine the fast initial paint of server-rendered HTML with the interactivity of client-side React (isomorphic).
- To ship less JavaScript by hydrating only interactive regions (islands).
- To improve Time to Interactive (TTI) by allowing islands to hydrate independently.
- To support progressive enhancement: the page is usable (as HTML) even before JavaScript loads.
- To prioritise hydration of the parts the user is interacting with (progressive hydration).
- To reduce the "uncanny valley" between the initial HTML and the hydrated app.

### Syntax Rules and Structure

**Isomorphic Rendering (Server + Client):**

```tsx
// server.js — renders React to HTML on the server
import { renderToString } from 'react-dom/server';
import App from './App';

app.get('/', (req, res) => {
  const html = renderToString(<App />);
  res.send(`
    <!DOCTYPE html>
    <html>
      <head><title>My App</title></head>
      <body>
        <div id="root">${html}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `);
});

// client.js — hydrates the server-rendered HTML
import { hydrateRoot } from 'react-dom/client';
import App from './App';

hydrateRoot(document.getElementById('root'), <App />);
```

**Component Breakdown:**
- `renderToString(<App />)`: Renders the React tree to an HTML string on the server.
- `hydrateRoot(...)`: Attaches React to the server-rendered HTML on the client, making it interactive.
- The same `App` component runs in both environments.
- The server must produce HTML that matches what the client would render during the first render.

**Islands Architecture (Conceptual):**

```html
<!-- Static HTML shell -->
<h1>Product Page</h1>
<p>This content is static HTML. No JavaScript needed.</p>

<!-- Island: interactive counter -->
<div id="counter-island"></div>

<!-- Island: interactive search -->
<div id="search-island"></div>

<script type="module" src="/islands.js"></script>
```

```tsx
// islands.js — hydrate each island independently
import { createRoot } from 'react-dom/client';
import { Counter } from './Counter';
import { Search } from './Search';

// Each island hydrates independently
const counterNode = document.getElementById('counter-island');
if (counterNode) {
  createRoot(counterNode).render(<Counter />);
}

const searchNode = document.getElementById('search-island');
if (searchNode) {
  createRoot(searchNode).render(<Search />);
}
```

**Component Breakdown:**
- The static HTML is rendered without any JavaScript for the non-interactive parts.
- Each island is a `div` with an ID that is targeted by the client-side JavaScript.
- Each island is rendered with `createRoot` independently — one island's hydration does not block another's.
- Islands execute asynchronously and independently, unlike full-page hydration which is top-down and sequential.

**Progressive Hydration (React `Suspense` + Islands):**

```tsx
import { Suspense } from 'react';
import { Counter } from './Counter';
import { Search } from './Search';

function App() {
  return (
    <div>
      <h1>Static content — no hydration needed</h1>
      <Suspense fallback={<div>Loading counter...</div>}>
        <Counter />
      </Suspense>
      <Suspense fallback={<div>Loading search...</div>}>
        <Search />
      </Suspense>
    </div>
  );
}
```

**Component Breakdown:**
- Each interactive region is wrapped in a `<Suspense>` boundary.
- React can hydrate each Suspense boundary independently.
- If a user interacts with one boundary before it hydrates, React prioritises hydrating that boundary first.
- This is "selective hydration" — a React feature that enables islands-like behaviour within a full React tree.

**Syntax Rules:**
- Isomorphic rendering requires the server and client to produce matching HTML for the first render.
- Islands architecture requires identifying which regions are truly interactive and marking them as islands.
- Each island should be self-contained: it manages its own state and does not depend on other islands.
- Use `Suspense` boundaries to enable selective hydration in React.
- Islands should be small and focused; a large island that hydrates slowly defeats the purpose.
- Progressive hydration prioritises islands based on user interaction (e.g., clicking a region triggers its hydration first).

**Constraints and Limitations:**
- Isomorphic rendering requires a server runtime (Node.js, Deno, edge functions) and a bundler that supports SSR.
- Islands architecture is framework-dependent; React does not natively support islands without a framework like Astro or a custom setup.
- Islands cannot easily share state with each other; each island is an independent React root.
- Isomorphic rendering can cause hydration mismatches if the server and client produce different HTML (e.g., due to `Date.now()` or `Math.random()`).
- Progressive hydration via `Suspense` requires the entire React runtime to be loaded before any island can hydrate, unlike true islands which can hydrate independently.

### Annotated Code Examples

**Example 1: Isomorphic Rendering with Data Fetching**

```tsx
// server.tsx — Server-side rendering with data
import express from 'express';
import { renderToString } from 'react-dom/server';
import App from './App';

const app = express();

app.get('/', async (req, res) => {
  const userData = await fetchUser(req.userId);
  const html = renderToString(<App initialUser={userData} />);

  res.send(`
    <!DOCTYPE html>
    <html>
      <head><title>My App</title></head>
      <body>
        <div id="root">${html}</div>
        <script>window.__INITIAL_DATA__ = ${JSON.stringify({ user: userData })}</script>
        <script src="/client.js"></script>
      </body>
    </html>
  `);
});
```

```tsx
// App.tsx — Shared component
import { useState } from 'react';

export default function App({ initialUser }: { initialUser: User }) {
  const [user, setUser] = useState(initialUser);

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <button onClick={() => setUser({ ...user, name: 'Updated' })}>
        Change Name
      </button>
    </div>
  );
}
```

```tsx
// client.tsx — Hydration
import { hydrateRoot } from 'react-dom/client';
import App from './App';

const initialData = window.__INITIAL_DATA__;
hydrateRoot(document.getElementById('root'), <App initialUser={initialData.user} />);
```

**Expected Output:** The server fetches user data, renders the `App` to HTML with the user's name, and sends the HTML along with the initial data. The client hydrates the HTML, and the "Change Name" button becomes interactive.

**Why This Output Occurs:** The same `App` component runs on the server and client. The server fetches data and passes it as a prop. The client uses the same data to hydrate. The `window.__INITIAL_DATA__` script provides the client with the data it needs to match the server-rendered HTML, preventing hydration mismatches.

**Example 2: Islands Architecture with Independent Hydration**

```html
<!-- product.html — Static HTML with islands -->
<!DOCTYPE html>
<html>
  <head><title>Product</title></head>
  <body>
    <h1>Product Name</h1>
    <p>This description is static HTML. No JavaScript.</p>

    <!-- Island 1: Add to Cart -->
    <div id="add-to-cart"></div>

    <!-- Island 2: Reviews -->
    <div id="reviews"></div>

    <script type="module" src="/islands.js"></script>
  </body>
</html>
```

```tsx
// islands.js — Each island hydrates independently
import { createRoot } from 'react-dom/client';
import { AddToCart } from './AddToCart';
import { Reviews } from './Reviews';

// Island 1: Add to Cart (hydrates immediately)
const cartNode = document.getElementById('add-to-cart');
if (cartNode) {
  createRoot(cartNode).render(<AddToCart productId="123" />);
}

// Island 2: Reviews (hydrates when visible)
const reviewsNode = document.getElementById('reviews');
if (reviewsNode) {
  const observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting) {
      createRoot(reviewsNode).render(<Reviews productId="123" />);
      observer.disconnect();
    }
  });
  observer.observe(reviewsNode);
}
```

**Expected Output:** The page loads with static HTML for the product name and description. The "Add to Cart" island hydrates immediately, and the "Reviews" island hydrates only when it scrolls into view.

**Why This Output Occurs:** Each island is an independent React root. The "Add to Cart" island is critical for conversion and hydrates immediately. The "Reviews" island is below the fold and hydrates lazily when the user scrolls to it. This reduces the initial JavaScript execution and improves Time to Interactive.

### Real-World Cases

- **E-commerce product pages:** Static product description with islands for "Add to Cart", image gallery, and reviews.
- **Documentation sites:** Static content with islands for search, theme toggle, and code copy buttons.
- **Marketing pages:** Static hero sections with islands for sign-up forms and interactive demos.
- **News articles:** Static article content with islands for comments, share buttons, and related-article carousels.
- **SaaS dashboards:** Isomorphic rendering for user-specific dashboards with progressive hydration of interactive widgets.

---

## References

- React Server Components – React (az.react.dev): https://az.react.dev/reference/rsc/server-components
- `'use client'` – React (zh-hans.react.dev): https://zh-hans.react.dev/reference/react/use-client
- Server and Client Boundary – Next.js: https://nextjs.org/docs/app/guides/server-and-client-boundary
- Server Components and Streaming SSR – Steve Kinney: https://stevekinney.com/courses/enterprise-ui/server-components-and-streaming-ssr
- Island Architecture – Steve Kinney: https://stevekinney.com/courses/enterprise-ui/island-architecture
- RSC Serialization Constraints – OrchestKit: https://github.com/yonatangross/orchestkit/blob/main/docs/site/content/docs/reference/skills/react-server-components-framework.mdx
- ADR-002: Clean Architecture + DDD Bounded Contexts – GitHub: https://raw.githubusercontent.com/pplancq/lab-clean-architecture-react/refs/heads/main/docs/architecture/adr/ADR-002-clean-architecture-ddd.md
- Architecture Overview – GitHub: https://raw.githubusercontent.com/pplancq/lab-clean-architecture-react/dbce608e722f2d5b8ab0ec02570e27834e22a79a/docs/architecture/README.md
- The Perfect Folder Structure for Scalable Frontend – Feature-Sliced Design: https://feature-sliced.design/blog/frontend-folder-structure
- React Project Skeleton: Monorepo Structure and Boilerplate – RT Camp: https://rtcamp.com/handbook/react-best-practices/project-skeleton/
- Feature-Based React Structure – GitHub: https://github.com/naserrasoulii/feature-based-react
- create-react-feature – npm: https://www.npmjs.com/package/create-react-feature
- Atomic Design Skill – GitHub: https://github.com/toshi-hm/dialy/blob/main/.codex/skills/react-component/SKILL.md
- ADR-012: Atomic Design System for Component Library – GitHub: https://github.com/Azure-Samples/holiday-peak-hub/blob/main/docs/architecture/adrs/adr-012-atomic-design-system.md
- Frontend Patterns – AI Agent Skill: https://askill.sh/skills/gh/besync-labs/antigravity-ai-kit/@frontend-patterns
- Monorepo with Turborepo and Yarn Workspaces – GitHub: https://github.com/dhruv-m-patel/monorepo
- Isomorphic React vs Gatsby – Stack Overflow: https://stackoverflow.com/questions/52002265/isomorphic-react-vs-gatsby-static-site-react