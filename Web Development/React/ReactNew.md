# React Comprehensive, Structured, and Progressive Learning Roadmap

## From Component Foundations to Advanced Rendering, State Management, and Production React Engineering

React is best learned as more than "a library for building UIs." The progression should cover **JavaScript prerequisites → JSX → components → props → state → hooks → effects → context → refs → rendering → reconciliation → forms → routing → data fetching → state management → performance → testing → accessibility → styling → server components → frameworks → production engineering**.

---

# I. React Foundations

- **1. What React Is**
  - React
  - React vs other frameworks
  - React vs Vue
  - React vs Angular
  - React vs Svelte
  - React vs Solid
  - Declarative UI
  - Component-based architecture
  - Virtual DOM
  - React Fiber
  - React reconciliation
  - Unidirectional data flow
  - JSX
  - React ecosystem
  - React Native
  - React DOM
  - React Server Components
  - React version history
  - React 16, 17, 18, 19
  - React philosophy

- **2. Prerequisites**
  - HTML
  - CSS
  - JavaScript
  - ES6+ features
  - Modules
  - Arrow functions
  - Destructuring
  - Spread/rest
  - Template literals
  - Classes
  - Promises
  - Async/await
  - Array methods
    - `map`
    - `filter`
    - `reduce`
    - `find`
    - `some`
    - `every`
    - `sort`
  - Higher-order functions
  - Closures
  - `this` binding
  - Immutability
  - Functional programming concepts
  - npm
  - Node.js basics

- **3. Setting Up React**
  - Create React App
  - Vite
  - Next.js
  - Remix
  - Gatsby
  - Astro
  - Parcel
  - Custom Webpack setup
  - Custom Rollup setup
  - TypeScript setup
  - ESLint setup
  - Prettier setup
  - React DevTools
  - Browser DevTools
  - VS Code extensions
  - React Developer Tools

- [**4. JSX**](/Web%20Development/React/Basics/JSX.md)
  - JSX syntax
  - JSX expressions
  - JSX attributes
  - JSX children
  - JSX fragments
  - JSX comments
  - JSX conditional rendering
  - JSX lists
  - JSX keys
  - JSX spread attributes
  - JSX event handlers
  - JSX className
  - JSX htmlFor
  - JSX style prop
  - JSX dangerouslySetInnerHTML
  - JSX transpilation
  - Babel
  - SWC
  - esbuild
  - JSX vs createElement
  - JSX best practices
  - JSX pitfalls

---

# II. Components

- [**5. Component Fundamentals**](/Web%20Development/React/Components/Component.md)
  - Components
  - Component types
    - Function components
    - Class components
  - Component naming
  - Component composition
  - Component reuse
  - Component lifecycle
  - Component state
  - Component props
  - Component rendering
  - Component updates
  - Component unmounting
  - Pure components
  - Impure components
  - Component boundaries
  - Component patterns

- [**6. Function Components**](/Web%20Development/React/Components/Function.md)
  - Function components
  - Arrow function components
  - Component definition
  - Component invocation
  - Component return values
  - Returning `null`
  - Returning fragments
  - Returning arrays
  - Component naming conventions
  - Component file organization

- [**7. Class Components**](/Web%20Development/React/Components/Class.md)
  - Class components
  - `React.Component`
  - `React.PureComponent`
  - `render()` method
  - `constructor()`
  - Component lifecycle methods
    - `componentDidMount()`
    - `componentDidUpdate()`
    - `componentWillUnmount()`
    - `shouldComponentUpdate()`
    - `getDerivedStateFromProps()`
    - `getSnapshotBeforeUpdate()`
    - `componentDidCatch()`
  - `this.state`
  - `this.setState()`
  - `this.props`
  - Binding methods
  - Class fields
  - Class components vs function components
  - When to use class components
  - Migrating to function components

- [**8. Props**](/Web%20Development/React/Components/Props.md)
  - Props
  - Prop passing
  - Prop drilling
  - Prop types
  - Default props
  - Destructuring props
  - Rest props
  - Spread props
  - Children prop
  - Render props
  - Function as children
  - Prop validation
  - `prop-types`
  - TypeScript props
  - Props immutability
  - Props best practices

- [**9. Component Composition**](/Web%20Development/React/Components/ComponentComposition.md)
  - Composition
  - Composition vs inheritance
  - Containment
  - Specialization
  - Slots pattern
  - Children as props
  - Named slots
  - Higher-order components
  - Render props
  - Compound components
  - Headless components
  - Controlled components
  - Uncontrolled components
  - Presentational components
  - Container components
  - Smart vs dumb components
  - Composition patterns

- [**10. Component Patterns**](/Web%20Development/React/Components/ComponentPatterns.md)
  - Container/Presentational pattern
  - Higher-Order Component (HOC)
  - Render props
  - Compound components
  - Provider pattern
  - Context pattern
  - Custom hooks pattern
  - State reducer pattern
  - Controlled props pattern
  - Proxy component pattern
  - Layout components
  - Headless components
  - Polymorphic components
  - Slot pattern
  - Factory pattern
  - Component composition best practices

---

# III. State

- [**11. State Fundamentals**](/Web%20Development/React/State%20Management/State.md)
  - State
  - Component state
  - Local state
  - Global state
  - Derived state
  - State vs props
  - State initialization
  - State updates
  - State immutability
  - State batching
  - State updates are asynchronous
  - State updates are merged
  - State updates are shallow
  - State updates and closures
  - State update pitfalls

- [**12. `useState` Hook**](/Web%20Development/React/Hooks/useState.md)
  - `useState`
  - State initialization
  - Lazy initialization
  - State updater function
  - Functional updates
  - State destructuring
  - Multiple state variables
  - State with objects
  - State with arrays
  - State with primitives
  - State batching
  - State update ordering
  - State update pitfalls
  - `useState` best practices

- [**13. `useReducer` Hook**](/Web%20Development/React/Hooks/useReducer.md)
  - `useReducer`
  - Reducer function
  - Initial state
  - Action objects
  - Action types
  - Dispatching actions
  - State transitions
  - Complex state logic
  - `useReducer` vs `useState`
  - Lazy initialization
  - Reducer composition
  - Reducer testing
  - `useReducer` best practices

- [**14. State Patterns**](/Web%20Development/React/State%20Management/StatePatterns.md)
  - Lifting state up
  - Colocating state
  - Derived state
  - Computed state
  - State machines
  - State charts
  - Finite state machines
  - XState
  - State reducers
  - State normalization
  - State immutability
  - Immutable.js
  - Immer
  - Structural sharing
  - State update patterns
  - State management best practices

---

# IV. Hooks

- [**15. Hook Fundamentals**](/Web%20Development/React/Hooks/Hooks.md)
  - Hooks
  - Hook rules
  - Hook rules of hooks
  - Only call hooks at top level
  - Only call hooks from React functions
  - Custom hooks
  - Hook naming
  - Hook order
  - Hook dependencies
  - Hook linting
  - `eslint-plugin-react-hooks`
  - Hook execution order
  - Hook state
  - Hook closure
  - Hook memory

- [**16. `useEffect` Hook**](/Web%20Development/React/Hooks/useEffect.md)
  - `useEffect`
  - Effect function
  - Cleanup function
  - Dependency array
  - No dependency array
  - Empty dependency array
  - Effect execution timing
  - Effect cleanup timing
  - Effect ordering
  - Effect batching
  - Effect pitfalls
  - Effect alternatives
  - `useLayoutEffect`
  - `useInsertionEffect`
  - `useEffect` best practices
  - Common `useEffect` patterns
    - Data fetching
    - Subscriptions
    - Event listeners
    - Timers
    - DOM manipulation
    - Manual DOM updates
    - Animation
    - Syncing with external systems

- [**17. `useContext` Hook**](/Web%20Development/React/Hooks/useContext.md)
  - `useContext`
  - Context
  - `React.createContext`
  - Context provider
  - Context consumer
  - Context value
  - Context default value
  - Context nesting
  - Context performance
  - Context vs props
  - Context vs state management
  - Context best practices
  - Context pitfalls

- [**18. `useRef` Hook**](/Web%20Development/React/Hooks/useRef.md)
  - `useRef`
  - Ref objects
  - `current` property
  - DOM refs
  - Mutable values
  - Persisting values
  - Ref forwarding
  - `forwardRef`
  - `useImperativeHandle`
  - Callback refs
  - Ref cleanup
  - Refs vs state
  - Ref best practices

- [**19. `useMemo` Hook**](/Web%20Development/React/Hooks/)
  - `useMemo`
  - Memoization
  - Dependency array
  - Expensive computations
  - Referential equality
  - When to use `useMemo`
  - When not to use `useMemo`
  - `useMemo` pitfalls
  - `useMemo` best practices

- [**20. `useCallback` Hook**](/Web%20Development/React/Hooks/useCallback.md)
  - `useCallback`
  - Function memoization
  - Dependency array
  - Referential equality
  - When to use `useCallback`
  - When not to use `useCallback`
  - `useCallback` pitfalls
  - `useCallback` best practices

- [**21. `useTransition` Hook**](/Web%20Development/React/Hooks/useTransition.md)
  - `useTransition`
  - Transitions
  - Pending state
  - `startTransition`
  - Concurrent rendering
  - Non-urgent updates
  - Urgent vs non-urgent updates
  - `useTransition` best practices

- [**22. `useDeferredValue` Hook**](/Web%20Development/React/Hooks/useDefferedValue.md)
  - `useDeferredValue`
  - Deferred values
  - Concurrent rendering
  - Stale values
  - `useDeferredValue` use cases
  - `useDeferredValue` best practices

- [**23. `useId` Hook**](/Web%20Development/React/Hooks/useId.md)
  - `useId`
  - Unique IDs
  - Server-side rendering
  - Hydration
  - Accessibility
  - `useId` best practices

- [**24. `useSyncExternalStore` Hook**](/Web%20Development/React/Hooks/useSyncExternalStore.md)
  - `useSyncExternalStore`
  - External stores
  - Subscribe function
  - Get snapshot function
  - Server snapshot
  - Concurrent rendering
  - Tearing prevention
  - `useSyncExternalStore` use cases

- [**25. `use` Hook (React 19)**](/Web%20Development/React/Hooks/use.md)
  - `use` hook
  - Reading promises
  - Reading context
  - Conditional use
  - Async components
  - Suspense integration
  - `use` best practices

- [**26. Custom Hooks**](/Web%20Development/React/Hooks/CustomHooks.md)
  - Custom hooks
  - Hook naming
  - Hook composition
  - Hook reuse
  - Hook testing
  - [Common custom hooks](/Web%20Development/React/Hooks/CommonCustomHooks.md)
    - `useFetch`
    - `useLocalStorage`
    - `useSessionStorage`
    - `useDebounce`
    - `useThrottle`
    - `useMediaQuery`
    - `useIntersectionObserver`
    - `useResizeObserver`
    - `useMutationObserver`
    - `useClickOutside`
    - `useKeyPress`
    - `useHover`
    - `useFocus`
    - `useForm`
    - `useToggle`
    - `useCounter`
    - `useInterval`
    - `useTimeout`
    - `usePrevious`
    - `useIsMounted`
    - `useAsync`
    - `usePagination`
    - `useInfiniteScroll`
    - `useCopyToClipboard`
    - `useDarkMode`
    - `useScrollPosition`
    - `useWindowSize`
    - `useOnlineStatus`
    - `useGeolocation`
    - `usePermission`
    - `useBroadcastChannel`
    - `useWebSocket`
    - `useEventSource`
  - Custom hook best practices

- [**27. Hook Pitfalls**](/Web%20Development/React/Hooks/Pitfalls.md)
  - Hook rules violations
  - Conditional hooks
  - Hooks in loops
  - Hooks in nested functions
  - Stale closures
  - Missing dependencies
  - Overuse of `useEffect`
  - Overuse of `useMemo`
  - Overuse of `useCallback`
  - Effect races
  - Effect cleanup issues
  - Infinite loops
  - Memory leaks
  - Hook debugging
  - Hook best practices

---

# V. Rendering and Reconciliation

- [**28. Rendering Fundamentals**](/Web%20Development/React/Rendering/Rendering.md)
  - Rendering
  - Initial render
  - Re-rendering
  - Render phase
  - Commit phase
  - Reconciliation
  - Diffing algorithm
  - Virtual DOM
  - React elements
  - React fiber
  - Fiber tree
  - Work loop
  - Rendering priority
  - Rendering scheduling
  - Rendering batching
  - Automatic batching
  - Rendering triggers
  - Rendering optimization

- [**29. Reconciliation**](/Web%20Development/React/Rendering/Reconciliation.md)
  - Reconciliation
  - Diffing algorithm
  - Key prop
  - Keys and lists
  - Keys and identity
  - Keys and state
  - Keys and performance
  - Keys pitfalls
  - Element identity
  - Component identity
  - Type identity
  - Position identity
  - Reconciliation heuristics
  - Reconciliation performance
  - Reconciliation best practices

- [**30. React Fiber**](/Web%20Development/React/Rendering/ReactFiber.md)
  - React Fiber
  - Fiber nodes
  - Fiber tree
  - Work loop
  - Rendering phases
  - Render phase
  - Commit phase
  - Fiber reconciliation
  - Fiber scheduling
  - Time slicing
  - Priority levels
  - Interruptible rendering
  - Concurrent rendering
  - Fiber vs stack reconciler
  - Fiber best practices

- [**31. Concurrent Rendering**](/Web%20Development/React/Rendering/Concurrent.md)
  - Concurrent rendering
  - Concurrent mode
  - Concurrent features
  - Transitions
  - Suspense
  - `useTransition`
  - `useDeferredValue`
  - Streaming SSR
  - Selective hydration
  - Time slicing
  - Priority scheduling
  - Interruptible rendering
  - Concurrent rendering best practices

- [**32. Suspense**](/Web%20Development/React/Rendering/Suspense.md)
  - Suspense
  - `Suspense` component
  - Fallback UI
  - Suspense boundaries
  - Nested suspense
  - Suspense with lazy
  - Suspense with data fetching
  - Suspense with server components
  - Suspense with streaming
  - Suspense best practices
  - Suspense pitfalls

- [**33. Portals**](/Web%20Development/React/Rendering/Portals.md)
  - Portals
  - `ReactDOM.createPortal`
  - Portal containers
  - Portal events
  - Portal context
  - Portal styling
  - Portal accessibility
  - Portal use cases
  - Portal best practices

- [**34. Error Boundaries**](/Web%20Development/React/Rendering/ErrorBoundaries.md)
  - Error boundaries
  - `componentDidCatch`
  - `getDerivedStateFromError`
  - Error boundary placement
  - Error boundary granularity
  - Error boundary limitations
  - Error boundary alternatives
  - Error boundary best practices

- [**35. Strict Mode**](/Web%20Development/React/Rendering/StrictMode.md)
  - StrictMode
  - Double rendering
  - Double effect invocation
  - Deprecation warnings
  - Identifying unsafe lifecycles
  - Identifying side effects
  - Strict mode in development
  - Strict mode in production
  - Strict mode best practices

---

# VI. Forms

- [**36. Form Fundamentals**](/Web%20Development/React/Forms%20and%20Validation/Form.md)
  - Forms
  - Form elements
  - Form submission
  - Form validation
  - Form state
  - Form data
  - Form events
    - `onSubmit`
    - `onChange`
    - `onInput`
    - `onBlur`
    - `onFocus`
    - `onReset`
  - Form accessibility
  - Form best practices

- [**37. Controlled Components**](/Web%20Development/React/Forms%20and%20Validation/ControlledComponents.md)
  - Controlled components
  - Value prop
  - onChange handler
  - Controlled inputs
  - Controlled text inputs
  - Controlled checkboxes
  - Controlled radio buttons
  - Controlled selects
  - Controlled textareas
  - Controlled file inputs
  - Controlled components benefits
  - Controlled components pitfalls
  - Controlled components performance

- [**38. Uncontrolled Components**](/Web%20Development/React/Forms%20and%20Validation/UncontrolledComponents.md)
  - Uncontrolled components
  - DefaultValue prop
  - DefaultChecked prop
  - Refs
  - Uncontrolled inputs
  - Uncontrolled components benefits
  - Uncontrolled components pitfalls
  - Controlled vs uncontrolled
  - When to use each

- [**39. Form Libraries**](/Web%20Development/React/Forms%20and%20Validation/FormLibraries.md)
  - Formik
  - React Hook Form
  - Redux Form
  - Final Form
  - TanStack Form
  - Form library comparison
  - Form library selection
  - Form library performance
  - Form library best practices

- [**40. Form Validation**](/Web%20Development/React/Forms%20and%20Validation/Validation.md)
  - Client-side validation
  - Server-side validation
  - Synchronous validation
  - Asynchronous validation
  - Field-level validation
  - Form-level validation
  - Schema validation
  - Yup
  - Zod
  - Joi
  - Validation libraries
  - Error messages
  - Error display
  - Validation best practices

- [**41. Form Patterns**](/Web%20Development/React/Forms%20and%20Validation/Patterns.md)
  - Form state management
  - Form submission
  - Form reset
  - Form persistence
  - Multi-step forms
  - Wizard forms
  - Dynamic forms
  - Nested forms
  - Array fields
  - Conditional fields
  - Dependent fields
  - Form accessibility
  - Form best practices

- [**42. Advanced Form Features**](/Web%20Development/React/Forms%20and%20Validation/AdvancedFormFeatures.md)
  - Dynamic fields
  - Multi-step forms
  - File uploads
  - Async validation
  - Draft persistence
  - Error summaries

---

# VII. Routing

- [**43. Routing Fundamentals**](/Web%20Development/React/Routing/Routing.md)
  - [Client-side routing](/Web%20Development/React/Routing/ClientSideRouting.md)
  - Server-side routing
  - Single-page applications
  - Multi-page applications
  - History API
  - Hash routing
  - Route matching
  - Route parameters
  - Query parameters
  - Nested routes
  - Route guards
  - Route transitions
  - Route rendering
  - Route performance
  - Route best practices

- [**44. React Router**](/Web%20Development/React/Routing/ReactRouter.md)
  - React Router
  - `BrowserRouter`
  - `HashRouter`
  - `MemoryRouter`
  - `Routes`
  - `Route`
  - `Link`
  - `NavLink`
  - `Navigate`
  - `Outlet`
  - `useNavigate`
  - `useParams`
  - `useSearchParams`
  - `useLocation`
  - `useMatch`
  - `useResolvedPath`
  - `useHref`
  - `useNavigationType`
  - Route configuration
  - Route nesting
  - Route layouts
  - Route loaders
  - Route actions
  - Route errors
  - Route lazy loading
  - Route data APIs
  - React Router versions
  - React Router best practices

- [**45. TanStack Router**](/Web%20Development/React/Routing/TanStack.md)
  - TanStack Router
  - Type-safe routing
  - Route trees
  - Route loaders
  - Route actions
  - Route search params
  - Route validation
  - Route caching
  - Route prefetching
  - TanStack Router vs React Router
  - TanStack Router best practices

- [**46. Next.js Routing**](/Web%20Development/React/Routing/NextJSRouting.md)
  - Next.js App Router
  - Next.js Pages Router
  - File-based routing
  - Dynamic routes
  - Catch-all routes
  - Optional catch-all routes
  - Route groups
  - Parallel routes
  - Intercepting routes
  - Route handlers
  - Route middleware
  - Route loading UI
  - Route error UI
  - Route layout
  - Route templates
  - Next.js routing best practices

- [**47. Routing Patterns**](/Web%20Development/React/Routing/RoutingPatterns.md)
  - Protected routes
  - Public routes
  - Private routes
  - Role-based routes
  - Authentication routes
  - Authorization routes
  - Redirects
  - Route guards
  - Route transitions
  - Route animations
  - Route prefetching
  - Route lazy loading
  - Route code splitting
  - Route error handling
  - Route best practices

- [**48. Advanced Routing**](/Web%20Development/React/Routing/AdvancedRouting.md)
  - Protected routes
  - Authentication-aware navigation
  - Error routes
  - Lazy-loaded routes
  - Route-level data loading
  - Nested layouts
  - URL state management

---

# XII. Data Fetching and APIs

* [**49. HTTP Fundamentals for React**](/Web%20Development/React/Data%20Fetching%20and%20APIs/HTTP.md)
  - GET
  - POST
  - PUT
  - PATCH
  - DELETE
  - HTTP status codes
  - Headers
  - JSON
  - CORS
  - Credentials

* [**50. Data Fetching Fundamentals**](/Web%20Development/React/Data%20Fetching%20and%20APIs/FetchingData.md)
  - Data fetching lifecycle
  - Async functions
  - Request state
  - Loading states
  - Error states
  - Empty states
  - Success states
  - Client-side fetching
  - Server-side fetching
  - Static generation
  - Incremental static regeneration
  - Server-side rendering
  - Streaming
  - Data caching
  - Data invalidation
  - Data refetching
  - Data fetching best practices

* [**51. Fetch API**](/Web%20Development/React/Data%20Fetching%20and%20APIs/FetchAPI.md)
  - `fetch`
  - Request options
  - Response handling
  - JSON parsing
  - Error handling
  - AbortController
  - Timeouts
  - Fetch in effects
  - Fetch pitfalls
  - Fetch best practices

* [**52. Axios**](/Web%20Development/React/Data%20Fetching%20and%20APIs/Axios.md)
  - Axios
  - Axios instance
  - Axios configuration
  - Axios interceptors
  - Axios error handling
  - Axios cancellation
  - Axios vs `fetch`
  - Axios best practices

* [**53. API Integration & Patterns**](/Web%20Development/React/Data%20Fetching%20and%20APIs/APIIntegration.md)
  - REST APIs
  - GraphQL
  - Authentication headers
  - Request interceptors
  - Response transformation
  - Error normalization
  - Fetch on mount
  - Fetch on event
  - Fetch on route change
  - Fetch on dependency change
  - Parallel fetching
  - Sequential fetching
  - Waterfall fetching

* [**54. Advanced Data Management & Real-Time**](/Web%20Development/React/Data%20Fetching%20and%20APIs/AdvancedDataManagement.md)
  - Caching
  - Deduplication
  - Pagination
  - Infinite scrolling
  - Prefetching
  - Optimistic updates
  - Background synchronization / fetching
  - Polling & Long polling
  - Server-Sent Events (SSE)
  - WebSockets
  - Streaming

* [**55. TanStack Query (React Query)**](/Web%20Development/React/Data%20Fetching%20and%20APIs/TanStackQuery.md)
  - TanStack Query / React Query
  - `useQuery`
  - `useMutation`
  - `useInfiniteQuery`
  - `useQueries`
  - Query keys & Query functions
  - Query caching, invalidation, & refetching
  - Query prefetching
  - Query pagination & infinite scrolling
  - Query optimistic updates
  - Query devtools
  - Query best practices

* [**56. SWR**](/Web%20Development/React/Data%20Fetching%20and%20APIs/SWR.md)
  - SWR & `useSWR`
  - SWR configuration
  - SWR caching & revalidation
  - SWR mutations
  - SWR pagination & infinite loading
  - SWR optimistic updates
  - SWR vs TanStack Query
  - SWR best practices

* [**57. Apollo Client**](/Web%20Development/React/Data%20Fetching%20and%20APIs/ApolloClient.md)
  - Apollo Client
  - GraphQL queries, mutations, & subscriptions
  - Apollo cache & local state
  - Apollo links
  - Apollo optimistic UI
  - Apollo pagination
  - Apollo best practices

* [**58. Relay**](/Web%20Development/React/Data%20Fetching%20and%20APIs/Relay.md)
  - Relay & Relay compiler
  - Relay fragments, queries, & mutations
  - Relay pagination
  - Relay store
  - Relay best practices

---

# IX. State Management

- [**59. State Management Fundamentals**](/Web%20Development/React/State%20Management/State.md)
  - State management fundamentals & categories
  - Local state
  - UI state
  - Shared state
  - Global state
  - Server state
  - URL state
  - Form state
  - Derived state
  - Persistent state
  - State categories
  - State management approaches
  - State management selection
  - State management best practices

- [**60. Context API**](/Web%20Development/React/State%20Management/ContextBasedState.md)
  - Context API
  - `React.createContext`
  - Context provider
  - Context consumer
  - `useContext`
  - Context value
  - Context default value
  - Context nesting
  - Context composition
  - Context performance
  - Context splitting
  - Context memoization
  - Context best practices
  - Context pitfalls

- [**61. Redux**](/Web%20Development/React/State%20Management/Redux.md)
  - Redux
  - Redux principles
    - Single source of truth
    - State is read-only
    - Changes are made with pure functions
  - Actions
  - Action creators
  - Reducers
  - Store
  - `createStore`
  - `configureStore`
  - Dispatch
  - Subscribe
  - Selectors
  - Middleware
  - Redux Thunk
  - Redux Saga
  - Redux Observable
  - Redux Toolkit
    - `createSlice`
    - `createAsyncThunk`
    - `createEntityAdapter`
    - `createSelector`
    - RTK Query
  - Redux DevTools
  - Redux best practices
  - Redux pitfalls

- [**62. Redux Toolkit**](/Web%20Development/React/State%20Management/ReduxToolkit.md)
  - Redux Toolkit
  - `configureStore`
  - `createSlice`
  - `createReducer`
  - `createAction`
  - `createAsyncThunk`
  - `createEntityAdapter`
  - `createSelector`
  - `createListenerMiddleware`
  - RTK Query
  - Redux Toolkit best practices

- [**63. Zustand**](/Web%20Development/React/State%20Management/Zustand.md)
  - Zustand
  - Store creation
  - State selectors
  - Actions
  - Middleware
  - Persistence
  - Immer integration
  - Devtools integration
  - Zustand vs Redux
  - Zustand best practices

- [**64. Jotai**](/Web%20Development/React/State%20Management/Jotai.md)
  - Jotai
  - Atoms
  - Derived atoms
  - Async atoms
  - Atom families
  - Jotai vs Recoil
  - Jotai best practices

- [**65. Recoil**](/Web%20Development/React/State%20Management/Recoil.md)
  - Recoil
  - Atoms
  - Selectors
  - Atom families
  - Async selectors
  - Recoil best practices

- [**66. MobX**](/Web%20Development/React/State%20Management/MobX.md)
  - MobX
  - Observables
  - Actions
  - Computed values
  - Reactions
  - MobX React
  - MobX best practices

- [**67. Valtio**](/Web%20Development/React/State%20Management/Valtio.md)
  - Valtio
  - Proxy state
  - `useSnapshot`
  - Valtio best practices

- [**68. XState**](/Web%20Development/React/State%20Management/XState.md)
  - XState
  - State machines
  - State charts
  - States
  - Transitions
  - Events
  - Actions
  - Guards
  - Context
  - Actors
  - XState React
  - XState best practices

- [**69. Server State**](/Web%20Development/React/State%20Management/ServerState.md)
  - Query state
  - Cache state
  - Synchronization with APIs
  - React Query / TanStack Query concepts
  - Mutations
  - Cache invalidation
  - Optimistic updates
  - Background refetching

- [**70. State Management Patterns**](/Web%20Development/React/State%20Management/StateLift.md)
  - Lifting state
  - Colocating state
  - State machines
  - Event-driven state
  - Reducer pattern
  - Observer pattern
  - Pub/sub
  - Event sourcing
  - CQRS
  - State normalization
  - State persistence
  - State hydration
  - State migration
  - State management best practices

---

# X. Styling

- [**71. Styling Fundamentals**](/Web%20Development/React/Styling/Styling.md)
  - Styling
  - Inline styles
  - CSS stylesheets
  - CSS modules
  - CSS-in-JS
  - Utility-first CSS
  - CSS preprocessors
  - CSS postprocessors
  - CSS variables
  - CSS nesting
  - CSS layers
  - CSS container queries
  - CSS best practices

- [**72. Inline Styles**](/Web%20Development/React/Styling/InlineStyles.md)
  - Inline styles
  - `style` prop
  - Style objects
  - Style values
  - Style camelCase
  - Style vendor prefixes
  - Inline style performance
  - Inline style limitations
  - Inline style best practices

- [**73. CSS Stylesheets**](/Web%20Development/React/Styling/CSS.md)
  - CSS stylesheets
  - Importing CSS
  - Global CSS
  - CSS specificity
  - CSS cascade
  - CSS inheritance
  - CSS naming conventions
  - BEM
  - SMACSS
  - OOCSS
  - CSS best practices

- [**74. CSS Modules**](/Web%20Development/React/Styling/CSSModules.md)
  - CSS Modules
  - Module scoping
  - Local class names
  - Global class names
  - Composing classes
  - CSS Modules with TypeScript
  - CSS Modules best practices

- [**75. CSS-in-JS**](/Web%20Development/React/Styling/CSSinJS.md)
  - CSS-in-JS
  - Styled Components
  - Emotion
  - Linaria
  - Stitches
  - Vanilla Extract
  - CSS-in-JS performance
  - CSS-in-JS trade-offs
  - CSS-in-JS best practices

- [**76. Tailwind CSS**](/Web%20Development/React/Styling/TailwindCSS.md)
  - Tailwind CSS
  - Utility classes
  - Responsive design
  - Dark mode
  - Custom configuration
  - Plugins
  - Tailwind with React
  - Tailwind best practices
  - Tailwind pitfalls

- [**77. Component Libraries**](/Web%20Development/React/Styling/ComponentLibraries.md)
  - Material UI
  - Ant Design
  - Chakra UI
  - Mantine
  - Radix UI
  - Headless UI
  - shadcn/ui
  - React Aria
  - React Spectrum
  - Bootstrap
  - Semantic UI
  - Component library selection
  - Component library best practices

- [**78. Styling Patterns**](/Web%20Development/React/Styling/StylingPatterns.md)
  - Theming
  - Dark mode
  - Responsive design
  - Mobile-first design
  - Design tokens
  - Design systems
  - Style composition
  - Conditional styles
  - Dynamic styles
  - Animation
  - Transitions
  - Styling best practices

---

# XI. Performance

- [**79. Performance Fundamentals**](/Web%20Development/React/PerformanceOpt/Performance.md)
  - Performance
  - Rendering performance
  - Runtime performance
  - Load performance
  - Perceived performance
  - Performance metrics
  - Core Web Vitals
    - Largest Contentful Paint
    - First Input Delay
    - Cumulative Layout Shift
    - Interaction to Next Paint
  - Performance budgets
  - Performance profiling
  - Performance monitoring

- [**80. Rendering Performance**](/Web%20Development/React/PerformanceOpt/RenderingPerformance.md)
  - Unnecessary re-renders
  - Re-render causes
  - Re-render optimization
  - `React.memo`
  - `useMemo`
  - `useCallback`
  - Component composition
  - Component splitting
  - State colocation
  - Children as props
  - Render optimization
  - Reconciliation optimization
  - Key optimization
  - List virtualization
  - Windowing
  - React Window
  - React Virtualized
  - TanStack Virtual
  - Rendering best practices

- [**81. Code Splitting**](/Web%20Development/React/PerformanceOpt/CodeSpitting.md)
  - Code splitting
  - Dynamic imports
  - `React.lazy`
  - `Suspense`
  - Route-based code splitting
  - Component-based code splitting
  - Vendor splitting
  - Chunk optimization
  - Preloading
  - Prefetching
  - Code splitting best practices

- [**82. Bundle Optimization**](/Web%20Development/React/PerformanceOpt/Bundle.md)
  - Bundle size
  - Tree shaking
  - Dead code elimination
  - Minification
  - Compression
  - Bundle analysis
  - Webpack Bundle Analyzer
  - Source Map Explorer
  - Vite Bundle Analyzer
  - Bundle optimization best practices

- [**83. Loading Performance**](/Web%20Development/React/PerformanceOpt/Loading.md)
  - Initial load
  - Above-the-fold content
  - Critical CSS
  - Lazy loading
  - Image optimization
  - Font optimization
  - Preloading
  - Prefetching
  - Preconnecting
  - Service workers
  - Loading performance best practices

- [**84. Runtime Performance**](/Web%20Development/React/PerformanceOpt/Runtime.md)
  - JavaScript execution
  - Memory usage
  - Memory leaks
  - Event listener leaks
  - Timer leaks
  - Subscription leaks
  - Long tasks
  - Task splitting
  - Web workers
  - Offloading work
  - Runtime performance best practices

- [**85. Concurrent Features**](/Web%20Development/React/PerformanceOpt/ConcurrentFeatures.md)
  - Concurrent rendering
  - Transitions
  - `useTransition`
  - `useDeferredValue`
  - Suspense
  - Streaming SSR
  - Selective hydration
  - Concurrent features best practices

- [**86. Performance Profiling**](/Web%20Development/React/PerformanceOpt/Profiling.md)
  - React DevTools Profiler
  - Chrome DevTools Performance
  - Flame charts
  - Component render times
  - Why did you render
  - React Scan
  - Profiling best practices

- [**87. Performance Patterns**](/Web%20Development/React/PerformanceOpt/Patterns.md)
  - [Memoization](/Web%20Development/React/PerformanceOpt/Memoization.md)
  - Virtualization
  - Windowing
  - Debouncing
  - Throttling
  - Batching
  - Lazy loading
  - Code splitting
  - Prefetching
  - Caching
  - Performance best practices

---

# XII. Testing

- [**88. Testing Fundamentals**](/Web%20Development/React/Testing%20App/Testing.md)
  - Testing
  - Test types
    - Unit tests
    - Integration tests
    - End-to-end tests
    - Snapshot tests
    - Visual regression tests
  - Test pyramid
  - Test-driven development
  - Behavior-driven development
  - Test coverage
  - Test isolation
  - Test doubles
    - Mocks
    - Stubs
    - Spies
    - Fakes
  - Testing best practices

- [**89. Unit Testing**](/Web%20Development/React/Testing%20App/UnitTesting.md)
  - Unit testing components
  - Unit testing hooks
  - Unit testing utilities
  - Unit testing reducers
  - Unit testing selectors
  - Testing pure functions
  - Testing side effects
  - Testing async code
  - Testing best practices

- [**90. Component Testing**](/Web%20Development/React/Testing%20App/ComponentTesting.md)
  - Component testing
  - Rendering components
  - Querying elements
  - Firing events
  - Asserting output
  - Testing props
  - Testing state
  - Testing context
  - Testing portals
  - Testing error boundaries
  - Component testing best practices

- [**91. Interaction Testing**](/Web%20Development/React/Testing%20App/InteractionTesting.md)
  - Forms
  - Buttons
  - Navigation
  - Async behavior
  - Loading states
  - Error states

- [**92. Integration Testing**](/Web%20Development/React/Testing%20App/IntegrationTesting.md)
  - Integration testing
  - Testing component interactions
  - Testing form submissions
  - Testing routing
  - Testing data fetching
  - Testing state management
  - Testing with real dependencies
  - Integration testing best practices

- [**93. End-to-End Testing**](/Web%20Development/React/Testing%20App/EndToEndTesting.md)
  - E2E testing
  - Cypress
  - Playwright
  - Puppeteer
  - Selenium
  - WebDriver
  - Test scenarios
  - Page objects
  - Test data
  - Test environments
  - E2E testing best practices

- [**94. Testing Libraries**](/Web%20Development/React/Testing%20App/TestingLibraries.md)
  - Jest
  - Vitest
  - React Testing Library
  - Enzyme
  - Testing Library
  - `@testing-library/react`
  - `@testing-library/user-event`
  - `@testing-library/jest-dom`
  - `@testing-library/react-hooks`
  - MSW (Mock Service Worker)
  - Mirage JS
  - Nock
  - Sinon
  - Chai
  - Expect
  - Testing library comparison
  - Testing library selection

- [**95. Testing Patterns**](/Web%20Development/React/Testing%20App/TestingPatterns.md)
  - Arrange-Act-Assert
  - Given-When-Then
  - Testing hooks
  - Testing custom hooks
  - Testing async hooks
  - Testing effects
  - Testing context
  - Testing reducers
  - Testing selectors
  - Testing forms
  - Testing routing
  - Testing data fetching
  - Testing error handling
  - Snapshot testing
  - Visual regression testing
  - Accessibility testing
  - Testing best practices

- [**96. Test Automation**](/Web%20Development/React/Testing%20App/TestAutomation.md)
  - CI integration
  - Test pipelines
  - Parallel testing
  - Test reporting
  - Code coverage
  - Mutation testing
  - Property-based testing
  - Fuzz testing
  - Visual regression testing
  - Testing best practices

-  [**97. Advanced Testing**](/Web%20Development/React/Testing%20App/AdvancedTesting.md)
  - Mock service workers
  - API mocking
  - Accessibility testing
  - Snapshot testing
  - Visual regression testing
  - Test coverage analysis

---

# XIII. Accessibility

- [**98. Accessibility Fundamentals**](/Web%20Development/React/Accessibility/Accessibility.md)
  - Accessibility
  - A11y
  - WCAG
  - ARIA
  - Semantic HTML
  - Screen readers
  - Keyboard navigation
  - Focus management
  - Color contrast
  - Text alternatives
  - Accessible React ([`AccessibilityReact.md`](/Web%20Development/React/Accessibility/AccessibilityReact.md))
  - Accessibility best practices

- [**99. Semantic HTML**](/Web%20Development/React/Accessibility/SemanticHTML.md)
  - Semantic HTML
  - Semantic elements
    - `header`
    - `nav`
    - `main`
    - `aside`
    - `footer`
    - `section`
    - `article`
    - `figure`
    - `figcaption`
    - `button`
    - `form`
    - `label`
    - `input`
    - `select`
    - `textarea`
    - `table`
    - `thead`
    - `tbody`
    - `tfoot`
    - `th`
    - `td`
  - Semantic HTML benefits
  - Semantic HTML pitfalls
  - Semantic HTML best practices

- [**100. ARIA**](/Web%20Development/React/Accessibility/ARIA.md)
  - ARIA
  - ARIA roles
  - ARIA states
  - ARIA properties
  - ARIA labels
  - ARIA descriptions
  - ARIA live regions
  - ARIA relationships
  - ARIA best practices
  - ARIA pitfalls
  - When to use ARIA
  - When not to use ARIA

- [**101. Keyboard Navigation**](/Web%20Development/React/Accessibility/KeyboardNav.md)
  - Keyboard navigation
  - Focus management
  - Focus trap
  - Focus restoration
  - Tab order
  - Tab index
  - Keyboard shortcuts
  - Keyboard events
  - Keyboard navigation best practices

- [**102. Screen Readers**](/Web%20Development/React/Accessibility/ScreenReaders.md)
  - Screen readers
  - NVDA
  - JAWS
  - VoiceOver
  - TalkBack
  - Screen reader testing
  - Screen reader best practices

- [**103. Accessibility Testing**](/Web%20Development/React/Accessibility/Testing.md)
  - Accessibility testing
  - Automated testing
  - Manual testing
  - Keyboard testing
  - Screen-reader testing
  - Focus-order validation
  - Color-contrast verification
  - Lighthouse
  - axe
  - jest-axe
  - eslint-plugin-jsx-a11y
  - Accessibility testing best practices

- [**104. Accessibility Patterns**](/Web%20Development/React/Accessibility/Patterns.md)
  - Accessible forms
  - Accessible buttons
  - Accessible links
  - Accessible modals
  - Accessible dropdowns
  - Accessible tabs
  - Accessible accordions
  - Accessible carousels
  - Accessible tables
  - Accessible images
  - Accessible icons
  - Accessible notifications
  - Accessibility best practices

---

# XIV. TypeScript with React

- [**105. TypeScript Fundamentals**](/Web%20Development/React/TypeScript/TypeScript.md)
  - TypeScript
  - Types
  - Interfaces
  - Type aliases
  - Unions
  - Intersections
  - Generics
  - Type inference
  - Type guards
  - Type narrowing
  - Utility types
  - TypeScript configuration
  - TypeScript best practices

- [**106. Typing Components**](/Web%20Development/React/TypeScript/TypingComponents.md)
  - Typing function components
  - Typing props
  - Typing children
  - Typing default props
  - Typing event handlers
  - Typing refs
  - Typing context
  - Typing hooks
  - Typing reducers
  - Typing actions
  - Typing selectors
  - Typing generics
  - Typing polymorphic components
  - Typing best practices

- [**107. Advanced TypeScript**](/Web%20Development/React/TypeScript/Advanced.md)
  - Generic components
  - Conditional types
  - Mapped types
  - Template literal types
  - Type inference
  - Type composition
  - Type-level programming
  - Branded types
  - Opaque types
  - Discriminated unions
  - Exhaustive checks
  - Type testing
  - Advanced TypeScript best practices

- [**108. Type-Safe Application Architecture**](/Web%20Development/React/TypeScript/TypeSafe.md)
  - Shared types
  - Domain models
  - Form types
  - API contracts
  - Validation schemas

---

# XV. React Server Components

- **108. Server Components Fundamentals**
  - React Server Components
  - Server components
  - Client components
  - Shared components
  - Server component benefits
  - Server component limitations
  - Server component rendering
  - Server component streaming
  - Server component serialization
  - Server component best practices

- **109. Server Components vs Client Components**
  - Server components
  - Client components
  - `'use client'` directive
  - `'use server'` directive
  - Component boundaries
  - Data fetching
  - Interactivity
  - State
  - Effects
  - Browser APIs
  - When to use each
  - Component composition

- **110. Server Actions**
  - Server actions
  - `'use server'`
  - Form actions
  - Data mutations
  - Server action security
  - Server action validation
  - Server action error handling
  - Server action best practices

- **111. Streaming and Suspense**
  - Streaming SSR
  - Suspense boundaries
  - Progressive rendering
  - Selective hydration
  - Streaming best practices

---

# XVI. React Frameworks

- **112. Next.js**
  - Next.js
  - App Router
  - Pages Router
  - File-based routing
  - Server components
  - Client components
  - Server actions
  - Data fetching
  - Rendering modes
    - Static rendering
    - Dynamic rendering
    - Streaming
    - Incremental static regeneration
  - Image optimization
  - Font optimization
  - Script optimization
  - Metadata
  - Middleware
  - API routes
  - Route handlers
  - Caching
  - Revalidation
  - Deployment
  - Next.js best practices

- **113. Remix**
  - Remix
  - Remix routing
  - Loaders
  - Actions
  - Form handling
  - Error boundaries
  - Nested routes
  - Data fetching
  - Mutations
  - Progressive enhancement
  - Remix best practices

- **114. Gatsby**
  - Gatsby
  - GraphQL data layer
  - Static site generation
  - Image optimization
  - Plugins
  - Themes
  - Gatsby best practices

- **115. Astro**
  - Astro
  - Islands architecture
  - Astro components
  - Framework components
  - Server-side rendering
  - Static site generation
  - Astro best practices

- **116. TanStack Start**
  - TanStack Start
  - Type-safe routing
  - Server functions
  - Data loading
  - TanStack Start best practices

- **117. Framework Selection**
  - Framework comparison
  - Use case selection
  - Performance comparison
  - Ecosystem comparison
  - Team considerations
  - Framework best practices

---

# XVII. React Native

- **118. React Native Fundamentals**
  - React Native
  - React Native vs React DOM
  - Native components
  - Core components
    - `View`
    - `Text`
    - `Image`
    - `ScrollView`
    - `FlatList`
    - `SectionList`
    - `TextInput`
    - `Button`
    - `TouchableOpacity`
    - `Pressable`
    - `Switch`
    - `Modal`
    - `SafeAreaView`
  - Styling in React Native
  - Flexbox in React Native
  - Platform-specific code
  - React Native best practices

- **119. React Native Navigation**
  - React Navigation
  - Stack navigator
  - Tab navigator
  - Drawer navigator
  - Navigation parameters
  - Navigation options
  - Navigation best practices

- **120. React Native APIs**
  - AsyncStorage
  - Alert
  - Animated
  - Dimensions
  - Keyboard
  - Linking
  - NetInfo
  - Permissions
  - Push notifications
  - Camera
  - Geolocation
  - React Native APIs best practices

- **121. React Native Ecosystem**
  - Expo
  - Expo Router
  - Expo modules
  - Native modules
  - Turbo modules
  - Fabric
  - Hermes
  - React Native best practices

---

# XVIII. React Patterns and Architecture

- **122. Design Patterns**
  - Container/Presentational
  - Higher-Order Components
  - Render Props
  - Compound Components
  - Provider Pattern
  - Custom Hooks
  - State Reducer
  - Controlled Props
  - Proxy Component
  - Layout Components
  - Headless Components
  - Polymorphic Components
  - Slot Pattern
  - Factory Pattern
  - Observer Pattern
  - Pub/Sub
  - Event Emitter
  - Command Pattern
  - Strategy Pattern
  - Adapter Pattern
  - Facade Pattern
  - Decorator Pattern
  - Singleton Pattern
  - Factory Pattern
  - Builder Pattern

- **123. Architecture Patterns**
  - Component architecture
  - Feature-based architecture
  - Layer-based architecture
  - Clean architecture
  - Hexagonal architecture
  - Atomic design
  - Design systems
  - Micro-frontends
  - Monorepos
  - Module federation
  - Shared component libraries
  - Architecture best practices

- **124. State Architecture**
  - State colocation
  - State lifting
  - State normalization
  - State machines
  - State synchronization
  - Server state
  - Client state
  - URL state
  - Form state
  - Cache state
  - State architecture best practices

- **125. Data Architecture**
  - Data fetching architecture
  - Data caching architecture
  - Data invalidation
  - Data normalization
  - Data denormalization
  - Optimistic updates
  - Pessimistic updates
  - Data architecture best practices

- **126. Code Organization**
  - File structure
  - Folder structure
  - Feature folders
  - Component folders
  - Hook folders
  - Utility folders
  - Test folders
  - Naming conventions
  - Barrel files
  - Index files
  - Code organization best practices

- **127. React Anti-Patterns**
  - Prop drilling
  - Overuse of context
  - Overuse of `useEffect`
  - Overuse of `useMemo`
  - Overuse of `useCallback`
  - Derived state in state
  - State duplication
  - Premature optimization
  - Monolithic components
  - God components
  - Prop explosion
  - Context explosion
  - Hook explosion
  - Render prop hell
  - HOC hell
  - Anti-pattern detection
  - Anti-pattern refactoring

---

# XIX. React Projects by Difficulty

## Beginner Projects

- **1. Counter App**
  - `useState`
  - Event handling
  - Conditional rendering
  - Basic styling

- **2. Todo List**
  - `useState`
  - List rendering
  - Keys
  - Event handling
  - Local storage

- **3. Weather App**
  - Data fetching
  - `useEffect`
  - Loading states
  - Error states
  - Conditional rendering

- **4. Calculator**
  - `useState`
  - Event handling
  - Component composition
  - Conditional rendering

- **5. Quiz App**
  - `useState`
  - Component composition
  - Conditional rendering
  - Score tracking

---

## Intermediate Projects

- **6. Blog Frontend**
  - Routing
  - Data fetching
  - CRUD operations
  - Forms
  - Validation
  - Error handling

- **7. E-Commerce Frontend**
  - Product listing
  - Product details
  - Cart
  - Checkout
  - Authentication
  - State management

- **8. Chat Application**
  - WebSockets
  - Real-time updates
  - Authentication
  - Message history
  - Notifications

- **9. Dashboard**
  - Data fetching
  - Charts
  - Filtering
  - Sorting
  - Pagination
  - Responsive design

- **10. Social Media App**
  - Authentication
  - Feed
  - Posts
  - Comments
  - Likes
  - Notifications
  - Routing

---

## Advanced Projects

- **11. Real-Time Collaboration Tool**
  - WebSockets
  - Operational transforms
  - Conflict resolution
  - Presence
  - Persistence

- **12. Video Streaming Platform**
  - Media APIs
  - Streaming
  - Player controls
  - Analytics
  - Recommendations

- **13. Project Management Tool**
  - Kanban board
  - Drag and drop
  - Real-time updates
  - Collaboration
  - Permissions
  - Notifications

- **14. Design System**
  - Component library
  - Theming
  - Design tokens
  - Accessibility
  - Documentation
  - Storybook

- **15. Progressive Web App**
  - Service workers
  - Offline support
  - Push notifications
  - App manifest
  - Caching strategies

---

## Expert Projects

- **16. React Framework**
  - Custom renderer
  - Reconciler
  - Component system
  - Hooks
  - Scheduling

- **17. Component Library**
  - Reusable components
  - Theming
  - Accessibility
  - Documentation
  - Testing
  - Publishing

- **18. Micro-Frontend Platform**
  - Module federation
  - Independent deployments
  - Shared dependencies
  - Routing
  - Communication
  - Observability

- **19. Full-Stack SaaS Application**
  - Next.js
  - Server components
  - Server actions
  - Database
  - Authentication
  - Authorization
  - Billing
  - Multi-tenancy

- **20. Production E-Commerce Platform**
  - Next.js
  - Server components
  - Streaming
  - Image optimization
  - SEO
  - Performance
  - Accessibility
  - Testing
  - Deployment
  - Monitoring

---

# XX. Progressive React Learning Sequence

## Level 1 — React Fundamentals

- Master:
  - JSX
  - Components
  - Props
  - State
  - Event handling
  - Conditional rendering
  - List rendering
  - Basic styling

## Level 2 — Hooks

- Master:
  - `useState`
  - `useEffect`
  - `useContext`
  - `useRef`
  - `useMemo`
  - `useCallback`
  - `useReducer`
  - Custom hooks

## Level 3 — Rendering and Reconciliation

- Master:
  - Rendering
  - Reconciliation
  - Keys
  - React Fiber
  - Concurrent rendering
  - Suspense
  - Portals
  - Error boundaries
  - Strict mode

## Level 4 — Forms and Routing

- Master:
  - Controlled components
  - Uncontrolled components
  - Form libraries
  - Form validation
  - React Router
  - TanStack Router
  - Next.js routing
  - Protected routes

## Level 5 — Data Fetching

- Master:
  - Fetch API
  - Axios
  - TanStack Query
  - SWR
  - Apollo Client
  - Data fetching patterns
  - Caching
  - Optimistic updates

## Level 6 — State Management

- Master:
  - Context API
  - Redux
  - Redux Toolkit
  - Zustand
  - Jotai
  - Recoil
  - MobX
  - XState
  - State management patterns

## Level 7 — Styling and Accessibility

- Master:
  - CSS Modules
  - CSS-in-JS
  - Tailwind CSS
  - Component libraries
  - Theming
  - Dark mode
  - Responsive design
  - Semantic HTML
  - ARIA
  - Keyboard navigation
  - Accessibility testing

## Level 8 — Performance and Testing

- Master:
  - Rendering performance
  - Code splitting
  - Bundle optimization
  - Loading performance
  - Runtime performance
  - Concurrent features
  - Profiling
  - Unit testing
  - Component testing
  - Integration testing
  - E2E testing
  - Testing libraries

## Level 9 — TypeScript and Frameworks

- Master:
  - TypeScript with React
  - Typing components
  - Typing hooks
  - Advanced TypeScript
  - Next.js
  - Remix
  - Gatsby
  - Astro
  - React Server Components
  - Server actions

## Level 10 — Production Engineering

- Master:
  - React Native
  - Design patterns
  - Architecture patterns
  - State architecture
  - Data architecture
  - Code organization
  - Anti-patterns
  - Deployment
  - Monitoring
  - Scaling
  - Micro-frontends
  - Design systems
  - Production best practices

---

# XXI. Final React Competency Map

- **Fundamentals**

  - JSX
  - Components
  - Props
  - State
  - Event handling
  - Conditional rendering
  - List rendering

- **Hooks**

  - `useState`
  - `useEffect`
  - `useContext`
  - `useRef`
  - `useMemo`
  - `useCallback`
  - `useReducer`
  - `useTransition`
  - `useDeferredValue`
  - `useId`
  - `useSyncExternalStore`
  - `use`
  - Custom hooks

- **Rendering**

  - Rendering
  - Reconciliation
  - Keys
  - React Fiber
  - Concurrent rendering
  - Suspense
  - Portals
  - Error boundaries

- **Forms**

  - Controlled components
  - Uncontrolled components
  - Form libraries
  - Form validation
  - Form patterns

- **Routing**

  - React Router
  - TanStack Router
  - Next.js routing
  - Protected routes
  - Nested routes

- **Data Fetching**

  - Fetch API
  - Axios
  - TanStack Query
  - SWR
  - Apollo Client
  - Data fetching patterns

- **State Management**

  - Context API
  - Redux
  - Redux Toolkit
  - Zustand
  - Jotai
  - Recoil
  - MobX
  - XState

- **Styling**

  - Inline styles
  - CSS Modules
  - CSS-in-JS
  - Tailwind CSS
  - Component libraries
  - Theming
  - Responsive design

- **Performance**

  - Rendering performance
  - Code splitting
  - Bundle optimization
  - Loading performance
  - Runtime performance
  - Concurrent features
  - Profiling

- **Testing**

  - Unit testing
  - Component testing
  - Integration testing
  - E2E testing
  - Testing libraries
  - Test automation

- **Accessibility**

  - Semantic HTML
  - ARIA
  - Keyboard navigation
  - Screen readers
  - Accessibility testing

- **TypeScript**

  - Typing components
  - Typing hooks
  - Advanced TypeScript
  - TypeScript with frameworks

- **Server Components**

  - Server components
  - Client components
  - Server actions
  - Streaming
  - Suspense

- **Frameworks**

  - Next.js
  - Remix
  - Gatsby
  - Astro
  - TanStack Start

- **React Native**

  - Core components
  - Navigation
  - APIs
  - Expo
  - Native modules

- **Architecture**

  - Design patterns
  - Architecture patterns
  - State architecture
  - Data architecture
  - Code organization
  - Anti-patterns

- **Production**

  - Deployment
  - Monitoring
  - Scaling
  - Micro-frontends
  - Design systems
  - Production best practices

---

## Recommended Overall Progression

**JavaScript Fundamentals → JSX → Components → Props → State → Hooks → Rendering and Reconciliation → Forms → Routing → Data Fetching → State Management → Styling → Performance → Testing → Accessibility → TypeScript → React Server Components → Next.js → React Native → Architecture → Production Engineering**