# XState — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** XState is a JavaScript/TypeScript library for creating, interpreting, and executing finite state machines and statecharts, providing a formal, visual, and testable model for application logic.

**Technical Definition:** XState is a state-management library that implements the SCXML (State Chart XML) specification. A **state machine** defines a finite set of states, the events that can occur, and the transitions between states. A **statechart** extends state machines with hierarchy (nested states), parallelism (orthogonal regions), history (remembering previous states), and multi-actor communication. XState v5 introduces the **actor model**: every machine, promise, callback, observable, or transition function is an **actor** that runs independently, communicates via events, and can spawn or invoke other actors. The `createMachine` function defines the logic; `createActor(machine).start()` runs it; `actor.send(event)` dispatches events; `actor.subscribe(listener)` observes state changes. The library is framework-agnostic, with official React bindings (`@xstate/react`) and a visual inspector (Stately Studio / `@statelyai/inspect`). XState's guarantee is that **invalid states and transitions are impossible by construction** — every transition is explicit, so the system can never be in a state it was not designed to be in.

**Beginner-Friendly Explanation:** XState is a tool for modelling complex logic as a state machine. Instead of scattering boolean flags (`isLoading`, `isError`, `isSuccess`) across your code — which allows impossible combinations like "loading and success at the same time" — you define a finite set of states (`idle`, `loading`, `success`, `error`), the events that move between them (`FETCH`, `RESOLVE`, `REJECT`), and the rules for each transition. The machine is a complete, visual map of your logic: you can see every possible state and every possible path. When you run it, XState refuses any event that is not valid for the current state. This makes impossible states impossible, makes bugs reproducible, and makes the logic testable without any UI.

### Key Characteristics

- **Finite, explicit state:** The system is always in exactly one named state (or a set of parallel states), never an implicit combination of booleans.
- **Deterministic transitions:** For every state and event, the next state is defined; events with no handler are ignored (or handled by a wildcard).
- **Statecharts:** Hierarchy (nested states), parallelism (orthogonal regions), and history (shallow/deep) scale the model beyond flat FSMs.
- **Actor model (v5):** Everything is an actor — machines, promises, callbacks, observables — communicating via events with isolation and lifecycle.
- **Context (extended state):** Quantitative data (counters, user objects, form values) lives in `context`; qualitative state lives in named states.
- **Actions and guards:** `entry`/`exit`/`transition` actions perform side effects; `guard` functions decide whether a transition is allowed.
- **Invoke and spawn:** Actors can start child actors (`invoke`) or dynamically create them (`spawn`).
- **Visual and testable:** Every machine can be visualized in Stately Studio and tested as a pure function.
- **TypeScript-first:** v5 uses `setup()` with `types` for fully inferred context, events, actors, actions, and guards.

### Prerequisites

- JavaScript/TypeScript fundamentals: objects, functions, Promises, `async`/`await`.
- React function components and Hooks (`useState`, `useEffect`) for the React bindings.
- Basic understanding of state management categories (client, server, UI).
- Familiarity with Promises and callbacks for actor types.
- Conceptual understanding of finite state machines (helpful, not required).

### Related Programming Areas

- **State Management** — Redux, Zustand, MobX, Jotai.
- **Workflow Orchestration** — XState Actors, Temporal, Inngest.
- **UI State Machines** — multi-step forms, wizards, authentication flows.
- **React Integration** — `@xstate/react` (`useMachine`, `useActor`, `useSelector`).
- **Visual Editing** — Stately Studio, `@statelyai/inspect`.

---

## 1. XState

### Definitions

**Core Definition:** XState is a library for creating, interpreting, and executing finite state machines and statecharts in JavaScript and TypeScript.

**Technical Definition:** XState exports `createMachine`, `setup`, `createActor`, `assign`, `sendTo`, `raise`, `log`, `spawn`, `fromPromise`, `fromCallback`, `fromObservable`, `fromEventObservable`, `fromTransition`, `waitFor`, and utilities like `state.matches`, `state.can`, `state.hasTag`, and `state.output`. `setup({ types, actions, guards, actors, delays })` creates a typed machine factory. `createMachine({ id, initial, context, states })` defines the machine. `createActor(machine, { input, inspect })` creates a runnable actor; `.start()` launches it. The actor exposes `send`, `subscribe`, `getSnapshot`, `stop`. XState v5 replaced `interpret` with `createActor`, made all actors first-class, and removed the `services` option in favour of `actors`.

**Beginner-Friendly Explanation:** XState is the state-machine library. You write a machine with `createMachine`, run it with `createActor(...).start()`, and interact with it by sending events.

### Purposes

- To model complex logic as an explicit, finite state machine.
- To eliminate impossible states (e.g., `loading` and `error` simultaneously).
- To make every valid transition visible and testable.
- To handle async work, side effects, and actor communication declaratively.
- To integrate with React via `@xstate/react` hooks.
- To visualize and debug logic in Stately Studio and inspectors.

### Syntax Rules and Structure

**General Syntax (v5):**

```ts
import { createMachine, createActor } from 'xstate';

const machine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  context: {},
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } },
  },
});

const actor = createActor(machine);
actor.start();
actor.send({ type: 'TOGGLE' });
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `createMachine(config)` | Defines the machine's states, events, and behaviour |
| `id` | Unique machine identifier |
| `initial` | The starting state |
| `context` | Extended (quantitative) state |
| `states` | Map of state names to state configurations |
| `on` | Event handlers for transitions |
| `createActor(machine)` | Creates a runnable actor from logic |
| `actor.start()` | Starts the actor and runs initial actions |
| `actor.send(event)` | Sends an event to the actor |
| `actor.subscribe(fn)` | Subscribes to snapshot changes |
| `actor.getSnapshot()` | Returns the current snapshot |
| `actor.stop()` | Stops the actor and its children |

**Syntax Rules:**

1. Every machine has an `initial` state and at least one state.
2. Every state has an `on` map for events; events without handlers are ignored.
3. Use `setup()` to define typed actions, guards, and actors for reuse.
4. Use `createActor` (not `interpret`) in v5.
5. Start the actor before sending events; sending to a stopped actor is a no-op.
6. `actor.subscribe` emits the current snapshot immediately on subscription.

**Constraints and Limitations:**

- XState v5 is not backward compatible with v4 (`interpret`, `Machine`, `services` are removed).
- Machines should model logic, not styling; UI rendering is the consumer's job.
- Over-modelling simple toggles adds ceremony; use `useState` for trivial UI.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Toggle Machine

```ts
import { createMachine, createActor } from 'xstate';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } },
  },
});

const actor = createActor(toggleMachine);
actor.start();

actor.subscribe((snapshot) => {
  console.log('state:', snapshot.value);
});

actor.send({ type: 'TOGGLE' }); // state: active
actor.send({ type: 'TOGGLE' }); // state: inactive
```

**Expected Output:** The subscriber logs `active` and then `inactive`. Sending an unknown event is ignored.

**Why this output:** The machine has only two states and one event. `TOGGLE` transitions between them; no other event has a handler.

#### Example 2: Machine with Context and Actions

```ts
import { createMachine, createActor, assign } from 'xstate';

const counterMachine = createMachine({
  id: 'counter',
  initial: 'active',
  context: { count: 0 },
  states: {
    active: {
      on: {
        INCREMENT: {
          actions: assign({ count: ({ context }) => context.count + 1 }),
        },
        RESET: {
          actions: assign({ count: 0 }),
        },
      },
    },
  },
});

const actor = createActor(counterMachine);
actor.start();
actor.send({ type: 'INCREMENT' });
console.log(actor.getSnapshot().context.count); // 1
```

**Expected Output:** Sending `INCREMENT` updates `context.count` to 1.

**Why this output:** `assign` is the action that updates `context`; the state remains `active` while the quantitative value changes.

### Real-World Cases

- **Authentication flows:** `loggedOut` → `loggingIn` → `loggedIn` / `error`.
- **Checkout wizards:** `cart` → `shipping` → `payment` → `review` → `confirmed`.
- **Data fetching:** `idle` → `loading` → `success` / `failure`.
- **Media players:** `playing` / `paused` / `stopped` with seek and volume.

### References

- XState — Official Documentation — https://stately.ai/docs/xstate
- XState — GitHub Repository — https://github.com/statelyai/xstate
- Stately — XState v5 Migration Guide — https://stately.ai/docs/migration

---

## 2. State Machines

### Definitions

**Core Definition:** A state machine is a model of computation with a finite set of states, a set of events, and a transition function that maps state-event pairs to next states.

**Technical Definition:** A finite state machine (FSM) consists of: (1) a finite set of **states** `S`; (2) a finite set of **events** `E`; (3) a **transition function** `δ: S × E → S`; (4) an **initial state** `s₀ ∈ S`; and optionally (5) **actions** that run on entry, exit, or transition. The machine is **deterministic**: for each state and event, there is at most one next state. Events with no transition are ignored. In XState, the machine is defined with `createMachine`, and the transition function is expressed through the `on` maps of each state.

**Beginner-Friendly Explanation:** A state machine is a map of all the states your system can be in, and all the events that move it between them. It can only be in one state at a time, and every move is explicit.

### Purposes

- To model finite, deterministic behaviour explicitly.
- To make every possible state and transition visible.
- To eliminate impossible states and transitions.
- To make logic testable without rendering.
- To provide a common vocabulary for designers, developers, and QA.

### Syntax Rules and Structure

**General Syntax:**

```ts
const machine = createMachine({
  id: 'door',
  initial: 'closed',
  states: {
    closed: { on: { OPEN: 'open' } },
    open: { on: { CLOSE: 'closed' } },
  },
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `id` | Machine identifier |
| `initial` | Starting state |
| `states` | Map of state names to configurations |
| `on` | Event-to-transition map for each state |

**Syntax Rules:**

1. Every machine has exactly one `initial` state.
2. Every state has a unique name.
3. `on` maps events to target states or transition objects.
4. Events without handlers are ignored (no error).
5. A transition object can include `target`, `actions`, `guard`, and `cond`.

**Constraints and Limitations:**

- FSMs cannot express hierarchy or parallelism; use statecharts for those.
- FSMs do not model quantitative data (counters, lists); use context.
- Overly large flat FSMs become unreadable; use hierarchy.

### Annotated Code Examples

```ts
import { createMachine, createActor } from 'xstate';

const trafficLight = createMachine({
  id: 'trafficLight',
  initial: 'green',
  states: {
    green: { on: { TIMER: 'yellow' } },
    yellow: { on: { TIMER: 'red' } },
    red: { on: { TIMER: 'green' } },
  },
});

const light = createActor(trafficLight).start();
light.send({ type: 'TIMER' });
console.log(light.getSnapshot().value); // 'yellow'
light.send({ type: 'TIMER' });
console.log(light.getSnapshot().value); // 'red'
```

**Expected Output:** The light cycles `green` → `yellow` → `red` → `green`.

**Why this output:** Each state has exactly one `TIMER` transition to the next state. There is no way to jump from `green` to `red`.

### Real-World Cases

- **Traffic lights:** Classic cyclic FSM.
- **Turnstiles:** `locked` / `unlocked` with `PUSH` and `COIN`.
- **Media players:** `playing` / `paused` / `stopped`.
- **Form validation:** `invalid` / `valid` / `submitting` / `submitted`.

### References

- XState — State Machines — https://stately.ai/docs/state-machines
- Stately — State Machines and Statecharts — https://stately.ai/docs/state-machines-and-statecharts
- Wikipedia — Finite-state machine — https://en.wikipedia.org/wiki/Finite-state_machine

---

## 3. Statecharts

### Definitions

**Core Definition:** A statechart is an extension of state machines that adds hierarchy (nested states), parallelism (orthogonal regions), and history (remembering previous states).

**Technical Definition:** Statecharts are a visual formalism introduced by David Harel in 1987 to model complex reactive systems. They extend FSMs with: **hierarchical states** (a state can contain child states, enabling shared transitions and reducing duplication); **parallel states** (multiple orthogonal regions active simultaneously); **history states** (shallow `hist` and deep `deepHistory` to resume from the last active child); and **entry/exit actions** that compose across the hierarchy. XState implements statecharts and the SCXML specification. Parallel states are defined with `type: 'parallel'`, and nested states are ordinary `states` objects inside a state.

**Beginner-Friendly Explanation:** Statecharts are state machines that can contain other state machines. They let you nest states inside states, run multiple regions at the same time, and remember where you were. They scale to complex UI logic that flat FSMs cannot model cleanly.

### Purposes

- To model complex UI logic with hierarchy and parallelism.
- To share transitions across nested states (reducing duplication).
- To run independent regions concurrently (auth, theme, notifications).
- To remember the last active child (history states).
- To keep large state models readable and visualizable.

### Syntax Rules and Structure

**Hierarchical state:**

```ts
const machine = createMachine({
  id: 'app',
  initial: 'loggedOut',
  states: {
    loggedOut: { on: { LOGIN: 'loggedIn' } },
    loggedIn: {
      initial: 'dashboard',
      states: {
        dashboard: { on: { OPEN_SETTINGS: 'settings' } },
        settings: { on: { BACK: 'dashboard' } },
      },
      on: { LOGOUT: 'loggedOut' }, // Shared across all logged-in children
    },
  },
});
```

**Parallel state:**

```ts
const app = createMachine({
  type: 'parallel',
  states: {
    auth: {
      initial: 'loggedOut',
      states: {
        loggedOut: { on: { LOGIN: 'loggedIn' } },
        loggedIn: { on: { LOGOUT: 'loggedOut' } },
      },
    },
    theme: {
      initial: 'light',
      states: {
        light: { on: { TOGGLE: 'dark' } },
        dark: { on: { TOGGLE: 'light' } },
      },
    },
  },
});
```

**History state:**

```ts
const machine = createMachine({
  initial: 'visible',
  states: {
    visible: {
      initial: 'a',
      states: { a: {}, b: {} },
      on: { HIDE: 'hidden' },
    },
    hidden: { on: { SHOW: 'visible.hist' } },
  },
});
```

**Syntax Rules:**

1. Nested states are ordinary `states` objects inside a parent state.
2. Shared transitions go on the parent state's `on` map.
3. Parallel states use `type: 'parallel'`; all regions run simultaneously.
4. History states use `hist` (shallow) or `deepHistory` (deep) as the transition target.
5. A child state is active whenever its parent is active.

**Constraints and Limitations:**

- Deep hierarchy can become hard to debug without visualization.
- Parallel regions must not share state unless explicitly communicated.
- History states can surprise developers when the initial child differs from the last active child.

### Annotated Code Examples

```ts
import { createActor, createMachine } from 'xstate';

const appMachine = createMachine({
  type: 'parallel',
  states: {
    auth: {
      initial: 'loggedOut',
      states: {
        loggedOut: { on: { LOGIN: 'loggedIn' } },
        loggedIn: { on: { LOGOUT: 'loggedOut' } },
      },
    },
    theme: {
      initial: 'light',
      states: {
        light: { on: { TOGGLE: 'dark' } },
        dark: { on: { TOGGLE: 'light' } },
      },
    },
  },
});

const app = createActor(appMachine).start();
app.send({ type: 'LOGIN' });
app.send({ type: 'TOGGLE' });

console.log(app.getSnapshot().value);
// { auth: 'loggedIn', theme: 'dark' }
```

**Expected Output:** Both regions change independently; the snapshot shows both active values.

**Why this output:** Parallel regions run concurrently; `LOGIN` affects only `auth`, and `TOGGLE` affects only `theme`.

### Real-World Cases

- **Application shells:** Auth, theme, and notifications as parallel regions.
- **Wizards:** Nested steps with shared cancel and back transitions.
- **Editors:** Parallel tool, document, and selection regions.
- **Games:** Parallel player, enemy, and environment machines.

### References

- Stately — State Machines and Statecharts — https://stately.ai/docs/state-machines-and-statecharts
- XState — Parallel States — https://stately.ai/docs/parallel-states
- XState — History States — https://stately.ai/docs/history-states
- David Harel — Statecharts: A Visual Formalism for Complex Systems — https://www.sciencedirect.com/science/article/pii/0167642387900359

---

## 4. States

### Definitions

**Core Definition:** A state is a named configuration of the machine that determines which events are accepted and which actions run.

**Technical Definition:** Each state is a node in the state tree with a name, an optional `on` map of event transitions, optional `entry` and `exit` actions, optional `invoke` of child actors, optional `after` delayed transitions, optional `always` eventless transitions, optional `tags`, and optional nested `states` (making it a compound state). A state can be **atomic** (leaf), **compound** (has children), **parallel** (orthogonal regions), **final** (terminal; triggers `onDone` on the parent), or **history** (shallow/deep). The active state configuration is a set of atomic states; in non-parallel machines, exactly one atomic state is active at a time.

**Beginner-Friendly Explanation:** A state is one of the possible modes your machine can be in — `idle`, `loading`, `success`, `error`. While in a state, the machine accepts only the events defined for it.

### Purposes

- To name and isolate distinct behavioural modes.
- To define which events are accepted in each mode.
- To attach entry/exit actions and invoked actors to specific modes.
- To structure hierarchy and parallelism.
- To mark terminal states with `type: 'final'`.

### Syntax Rules and Structure

**Atomic state:**

```ts
{ idle: { on: { START: 'loading' } } }
```

**Compound state:**

```ts
{
  loading: {
    initial: 'fetching',
    states: {
      fetching: { on: { RESOLVE: 'success' } },
      success: { type: 'final' },
    },
  },
}
```

**State with actions, invoke, after, and tags:**

```ts
{
  loading: {
    entry: 'logLoading',
    exit: 'clearError',
    invoke: {
      src: 'fetchUser',
      onDone: { target: 'success', actions: 'setUser' },
      onError: { target: 'failure', actions: 'setError' },
    },
    after: {
      5000: 'timeout',
    },
    tags: ['busy'],
  },
}
```

**Final state:**

```ts
{ done: { type: 'final' } }
```

**Syntax Rules:**

1. State names are unique within their parent.
2. `type: 'final'` marks terminal states; reaching one triggers the parent's `onDone`.
3. `entry` runs when the state is entered; `exit` runs when it is left.
4. `invoke` starts a child actor; `after` schedules delayed transitions.
5. `tags` allow grouping states for `state.hasTag('busy')`.
6. `always` transitions run immediately on entry without an event.

**Constraints and Limitations:**

- A state cannot be both `final` and have children.
- `always` transitions can cause infinite loops if their guards always pass.
- Entry actions run in order; exit actions run in order on exit.

### Annotated Code Examples

```ts
import { createMachine, createActor } from 'xstate';

const fetchMachine = createMachine({
  id: 'fetch',
  initial: 'idle',
  states: {
    idle: { on: { FETCH: 'loading' } },
    loading: {
      entry: () => console.log('loading started'),
      exit: () => console.log('loading ended'),
      after: {
        5000: 'timeout',
      },
      on: {
        RESOLVE: 'success',
        REJECT: 'failure',
      },
    },
    success: { type: 'final' },
    failure: { on: { RETRY: 'loading' } },
    timeout: { on: { RETRY: 'loading' } },
  },
});

const actor = createActor(fetchMachine).start();
actor.send({ type: 'FETCH' }); // logs "loading started"
actor.send({ type: 'RESOLVE' }); // logs "loading ended"
```

**Expected Output:** Entering `loading` logs “loading started”; leaving it logs “loading ended”; `RESOLVE` moves to the final `success` state.

**Why this output:** `entry`/`exit` actions run on state transitions; `after` schedules a timeout; `type: 'final'` marks a terminal state.

### Real-World Cases

- **Auth:** `loggedOut`, `loggingIn`, `loggedIn`, `error`.
- **Checkout:** `cart`, `shipping`, `payment`, `review`, `confirmed`.
- **Data fetching:** `idle`, `loading`, `success`, `failure`, `timeout`.
- **Media:** `playing`, `paused`, `stopped`, `buffering`.

### References

- XState — States — https://stately.ai/docs/states
- XState — Final States — https://stately.ai/docs/final-states
- XState — Delayed Transitions — https://stately.ai/docs/delayed-transitions

---

## 5. Transitions

### Definitions

**Core Definition:** A transition is a rule that defines how the machine moves from one state to another in response to an event.

**Technical Definition:** Transitions are declared in a state's `on` map: `on: { EVENT: target }` for a simple transition, or `on: { EVENT: { target, actions, guard } }` for a full configuration. Transition resolution follows **SCXML semantics**: when an event occurs, XState finds the highest-priority transition from the active state configuration (the closest ancestor with a handler wins), evaluates the `guard`, and if allowed, exits the current states, runs exit actions, enters the target states, runs entry actions, and executes the transition actions. Transitions can be **eventless** (`always`), **delayed** (`after`), or **internal** (no state change but actions run). `target: undefined` runs actions without exiting the state; `target: '.child'` targets a specific child.

**Beginner-Friendly Explanation:** A transition is a rule that says “when this event happens in this state, go to that state.” Transitions are the arrows on the state diagram.

### Purposes

- To define the exact conditions under which the machine changes state.
- To attach actions and guards to state changes.
- To support delayed and eventless transitions.
- To enable internal transitions (actions without state change).
- To make every state change explicit and auditable.

### Syntax Rules and Structure

**Simple transition:**

```ts
{ idle: { on: { START: 'loading' } } }
```

**Full transition:**

```ts
{
  idle: {
    on: {
      START: {
        target: 'loading',
        actions: 'logStart',
        guard: 'canStart',
      },
    },
  },
}
```

**Self-transition and internal transition:**

```ts
{
  active: {
    on: {
      TOGGLE: 'active', // Self-transition: exits and re-enters
      UPDATE: { actions: 'updateValue' }, // Internal: no state change
    },
  },
}
```

**Delayed and eventless transitions:**

```ts
{
  loading: {
    after: { 5000: 'timeout' },
    always: { guard: 'isReady', target: 'success' },
  },
}
```

**Syntax Rules:**

1. Transitions are declared under `on` with the event type as the key.
2. `target` is the destination state; omit it for internal transitions.
3. `guard` (or `cond`) is a function or named guard that must return `true`.
4. `actions` run during the transition.
5. `after` defines timed transitions; `always` defines eventless transitions.
6. Multiple transitions for the same event can be ordered arrays with guards.

**Constraints and Limitations:**

- Without a guard, the first matching transition wins; use arrays with guards for branching.
- `target: 'active'` is a self-transition (exit and re-enter); use internal transitions for actions only.
- Delayed transitions are cancelled if the state exits before the delay elapses.

### Annotated Code Examples

```ts
import { createMachine, createActor } from 'xstate';

const machine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: {
      on: {
        TOGGLE: {
          target: 'active',
          guard: ({ context }) => context.enabled,
        },
      },
    },
    active: { on: { TOGGLE: 'inactive' } },
  },
  context: { enabled: false },
});

const actor = createActor(machine).start();
actor.send({ type: 'TOGGLE' });
console.log(actor.getSnapshot().value); // 'inactive' (guard failed)

actor.getSnapshot().context.enabled = true; // Direct context mutation
actor.send({ type: 'TOGGLE' });
console.log(actor.getSnapshot().value); // 'active'
```

**Expected Output:** The first `TOGGLE` is blocked by the guard; after `enabled` becomes `true`, the second `TOGGLE` transitions to `active`.

**Why this output:** The `guard` function decides whether the transition is allowed. If it returns `false`, the event is ignored.

### Real-World Cases

- **Form submission:** `SUBMIT` transitions from `valid` to `submitting` only if the form is valid.
- **Authentication:** `LOGIN` transitions from `loggedOut` to `loggingIn` only if credentials exist.
- **Media:** `PLAY` transitions from `paused` to `playing` only if media is loaded.
- **Wizards:** `NEXT` transitions only if the current step is valid.

### References

- XState — Transitions — https://stately.ai/docs/transitions
- XState — Guards — https://stately.ai/docs/guards
- XState — Delayed Transitions — https://stately.ai/docs/delayed-transitions
- SCXML Specification — https://www.w3.org/TR/scxml/

---

## 6. Events

### Definitions

**Core Definition:** An event is a plain object with a `type` field that signals something happened, and that the machine may respond to with a transition.

**Technical Definition:** Events are the inputs to a machine. They must have a string `type` and may carry additional payload fields. XState's TypeScript integration defines events as a union type in `setup({ types: { events: {} as ... } })`. Events are sent with `actor.send(event)`, `actorRef.send`, or `useActor`'s send function. Internally, XState adds `xstate.*` fields to the event during processing, but the user-facing event is the plain object. Built-in events include `xstate.init`, `xstate.stop`, `xstate.done.actor.<id>`, and `xstate.error.actor.<id>`.

**Beginner-Friendly Explanation:** An event is a message that something happened — a button click, an API response, a timer. The machine listens for events and decides whether to transition.

### Purposes

- To signal state changes to the machine.
- To carry payloads (form values, API results) into transitions.
- To trigger guards, actions, and actor invocations.
- To enable inter-actor communication via `sendTo` and `sendParent`.
- To provide a typed, auditable record of everything that happened.

### Syntax Rules and Structure

**Simple event:**

```ts
actor.send({ type: 'TOGGLE' });
```

**Event with payload:**

```ts
actor.send({ type: 'SUBMIT', values: { email: 'ada@example.com' } });
```

**Typed events with `setup`:**

```ts
import { setup } from 'xstate';

const machine = setup({
  types: {
    events: {} as
      | { type: 'FETCH' }
      | { type: 'RESOLVE'; data: User[] }
      | { type: 'REJECT'; error: string },
  },
}).createMachine({
  // ...
});
```

**Built-in events:**

```ts
// xstate.init, xstate.stop, xstate.done.actor.<id>, xstate.error.actor.<id>
```

**Syntax Rules:**

1. Every event must have a `type` string.
2. Payloads are additional fields on the event object.
3. Events are sent with `actor.send(event)` or the `send` function from hooks.
4. Use `setup({ types: { events } })` for typed events.
5. Built-in events are used for actor lifecycle and `invoke` completion.
6. Events are processed synchronously in order; send returns after the transition (or immediately if queued).

**Constraints and Limitations:**

- Events without a matching handler are ignored (no error).
- Event payloads must be serializable for inspection and persistence.
- Sending events to a stopped actor is a no-op.

### Annotated Code Examples

```ts
import { setup, createActor } from 'xstate';

const machine = setup({
  types: {
    events: {} as
      | { type: 'SUBMIT'; values: { email: string } }
      | { type: 'SUCCESS' }
      | { type: 'FAIL'; error: string },
  },
}).createMachine({
  id: 'form',
  initial: 'idle',
  states: {
    idle: { on: { SUBMIT: 'submitting' } },
    submitting: {
      on: {
        SUCCESS: 'done',
        FAIL: 'error',
      },
    },
    done: { type: 'final' },
    error: { on: { SUBMIT: 'submitting' } },
  },
});

const actor = createActor(machine).start();
actor.send({ type: 'SUBMIT', values: { email: 'ada@example.com' } });
actor.send({ type: 'SUCCESS' });
console.log(actor.getSnapshot().value); // 'done'
```

**Expected Output:** The machine transitions `idle` → `submitting` → `done`.

**Why this output:** `SUBMIT` triggers the first transition; `SUCCESS` triggers the second. The event payload is available to guards and actions.

### Real-World Cases

- **Form submissions:** `SUBMIT` with form values.
- **API responses:** `RESOLVE` / `REJECT` with data or errors.
- **User interactions:** `CLICK`, `TOGGLE`, `CHANGE`.
- **Actor communication:** `sendTo` events between parent and child.

### References

- XState — Events — https://stately.ai/docs/events
- XState — TypeScript — https://stately.ai/docs/typescript
- XState — Actor Communication — https://stately.ai/docs/actors

---

## 7. Actions

### Definitions

**Core Definition:** An action is a side effect that runs during a transition or on state entry/exit, such as updating context, sending events, or logging.

**Technical Definition:** Actions are functions or named actions declared in `setup({ actions: {...} })`. Built-in actions include `assign` (update context), `sendTo` (send events to actors), `sendParent` (send to parent), `raise` (send event to self), `log` (log to console), `cancel` (cancel a delayed event), `stopChild` (stop an invoked actor), and `spawnChild` (spawn an actor). Actions run in order: exit actions of the source state, transition actions, entry actions of the target state. Actions must be **pure descriptions** where possible; side effects are executed by the interpreter. `assign` is the idiomatic way to update context.

**Beginner-Friendly Explanation:** Actions are the things that happen when the machine transitions — updating data, sending events, logging. `assign` updates context; `sendTo`/`raise` send events; `log` logs.

### Purposes

- To update context with new data (`assign`).
- To send events to self, parent, or other actors.
- To log for debugging.
- To cancel or stop child actors.
- To perform side effects on entry, exit, or transition.

### Syntax Rules and Structure

**`assign`:**

```ts
import { assign } from 'xstate';

actions: assign({
  count: ({ context }) => context.count + 1,
})
```

**`sendTo` / `sendParent` / `raise`:**

```ts
import { sendTo, sendParent, raise } from 'xstate';

actions: sendParent({ type: 'CHILD_DONE' })
actions: raise({ type: 'INTERNAL_EVENT' })
actions: sendTo('childId', { type: 'PING' })
```

**`log`:**

```ts
actions: log(({ context }) => `count is ${context.count}`)
```

**Named actions:**

```ts
setup({
  actions: {
    trackClick: ({ context, event }) => { /* side effect */ },
  },
}).createMachine({
  on: { CLICK: { actions: 'trackClick' } },
})
```

**Syntax Rules:**

1. Actions run in this order: exit source, transition actions, entry target.
2. `assign` must be the only way to update context.
3. Use `raise` for internal events; `sendTo` for external actors.
4. Named actions are defined in `setup({ actions })`.
5. Actions should describe side effects; the interpreter executes them.

**Constraints and Limitations:**

- Do not mutate `context` directly in an action; use `assign`.
- Multiple `assign` actions in one transition are executed in order.
- Side effects in actions are executed by the actor, not the pure machine.

### Annotated Code Examples

```ts
import { setup, assign, createActor } from 'xstate';

const machine = setup({
  types: { context: {} as { count: number } },
  actions: {
    logCount: ({ context }) => console.log('count:', context.count),
  },
}).createMachine({
  id: 'counter',
  context: { count: 0 },
  initial: 'active',
  states: {
    active: {
      on: {
        INCREMENT: {
          actions: [
            assign({ count: ({ context }) => context.count + 1 }),
            'logCount',
          ],
        },
      },
    },
  },
});

const actor = createActor(machine).start();
actor.send({ type: 'INCREMENT' }); // logs "count: 1"
```

**Expected Output:** `INCREMENT` updates the count and logs the new value.

**Why this output:** `assign` updates context; `logCount` is a named action that runs after the assignment.

### Real-World Cases

- **Context updates:** Incrementing counters, setting user data.
- **Notifications:** Sending events to parent machines.
- **Debugging:** Logging state transitions.
- **Actor orchestration:** Spawning and stopping children.

### References

- XState — Actions — https://stately.ai/docs/actions
- XState — Built-in Actions — https://stately.ai/docs/actions#built-in-actions
- XState — Assign Action — https://stately.ai/docs/actions#assign-action

---

## 8. Guards

### Definitions

**Core Definition:** A guard is a predicate that determines whether a transition is allowed, based on context, event, or both.

**Technical Definition:** Guards are functions or named guards declared in `setup({ guards: {...} })`. In a transition, `guard` (or the legacy `cond`) receives `{ context, event }` and returns `true` or `false`. If the guard returns `false`, the transition is not taken and the machine remains in the current state. Multiple transitions for the same event can be defined as an array, each with its own guard; XState evaluates them in order and takes the first allowed transition. Guards must be pure functions. Inline guards are allowed but named guards are recommended for reuse and testability.

**Beginner-Friendly Explanation:** A guard is a condition that decides whether a transition is allowed. If the guard returns `false`, the machine ignores the event and stays where it is.

### Purposes

- To prevent transitions that should not happen.
- To branch between multiple targets based on conditions.
- To keep state logic declarative and testable.
- To validate form inputs before submission.
- To enforce permissions before navigation.

### Syntax Rules and Structure

**Inline guard:**

```ts
on: {
  SUBMIT: {
    target: 'submitting',
    guard: ({ context }) => context.form.isValid,
  },
}
```

**Named guard:**

```ts
setup({
  guards: {
    isValid: ({ context }) => context.form.isValid,
  },
}).createMachine({
  on: { SUBMIT: { target: 'submitting', guard: 'isValid' } },
})
```

**Multiple guarded transitions:**

```ts
on: {
  CHECK: [
    { guard: 'isAdmin', target: 'admin' },
    { guard: 'isUser', target: 'user' },
    { target: 'guest' },
  ],
}
```

**Syntax Rules:**

1. Guards are pure functions returning a boolean.
2. Use named guards for reuse and testability.
3. Multiple transitions for the same event are evaluated in order; the first allowed wins.
4. Guards can read `context` and `event` but must not mutate them.
5. A transition with no guard is always allowed.

**Constraints and Limitations:**

- Guard evaluation is synchronous; async validation should happen before sending the event.
- Guards cannot perform side effects.
- Guard functions must be deterministic given the same context and event.

### Annotated Code Examples

```ts
import { setup, createActor } from 'xstate';

const machine = setup({
  types: { context: {} as { user: { role: 'admin' | 'user' } | null } },
  guards: {
    isAdmin: ({ context }) => context.user?.role === 'admin',
    isLoggedIn: ({ context }) => context.user !== null,
  },
}).createMachine({
  id: 'router',
  context: { user: null },
  initial: 'idle',
  states: {
    idle: {
      on: {
        NAVIGATE: [
          { guard: 'isAdmin', target: 'adminPanel' },
          { guard: 'isLoggedIn', target: 'dashboard' },
          { target: 'login' },
        ],
      },
    },
    adminPanel: {},
    dashboard: {},
    login: {},
  },
});

const actor = createActor(machine).start();
actor.send({ type: 'NAVIGATE' });
console.log(actor.getSnapshot().value); // 'login'
```

**Expected Output:** With `user: null`, the first two guards fail and the machine transitions to `login`.

**Why this output:** Guards are evaluated in order; the first one that returns `true` wins. The unguarded `login` transition is the fallback.

### Real-World Cases

- **Authentication:** Only allow navigation when logged in.
- **Permissions:** Admin-only routes.
- **Form validation:** Submit only when valid.
- **Feature flags:** Enable flows only when flags are on.

### References

- XState — Guards — https://stately.ai/docs/guards
- XState — TypeScript Guards — https://stately.ai/docs/typescript#guards
- SCXML — Conditional Transitions — https://www.w3.org/TR/scxml/#cond

---

## 9. Context

### Definitions

**Core Definition:** Context is the extended, quantitative state of a machine — data like counters, user objects, and form values — that lives alongside the qualitative state names.

**Technical Definition:** Context is the `context` property of the machine configuration. It is a plain object that holds data relevant across states. Context is updated only through `assign` actions; direct mutation is forbidden in XState's actor model (though `snapshot.context` is mutable in some contexts, the recommended path is `assign`). Context can be typed with `setup({ types: { context: {} as {...}, input: {} as {...} } })`. Actors receive initial context via `input` when created, enabling per-instance context. `state.context` and `snapshot.context` expose the current context. Context is not reactive on its own; changes trigger transitions and re-renders through the actor's subscription.

**Beginner-Friendly Explanation:** Context is the data inside your machine — the count, the user, the form values. The state names describe “what mode” the machine is in; context describes “what data it holds.”

### Purposes

- To store quantitative data alongside qualitative state.
- To share data across states without duplicating it.
- To receive per-actor initial data via `input`.
- To update data declaratively via `assign`.
- To keep a clear separation between state names and data.

### Syntax Rules and Structure

**Initial context:**

```ts
const machine = createMachine({
  context: { count: 0, user: null as User | null },
  // ...
});
```

**Typed context and input:**

```ts
const machine = setup({
  types: {
    context: {} as { count: number },
    input: {} as { initialCount: number },
  },
}).createMachine({
  context: ({ input }) => ({ count: input.initialCount }),
  // ...
});
```

**Updating context with `assign`:**

```ts
actions: assign({
  count: ({ context, event }) => context.count + event.by,
})
```

**Syntax Rules:**

1. Define `context` as an object literal or a function of `input`.
2. Update context only with `assign` actions.
3. Use `context` in guards and actions to read current data.
4. Use `input` to pass per-actor initial data.
5. Access context via `snapshot.context` or `actor.getSnapshot().context`.

**Constraints and Limitations:**

- Direct mutation of `snapshot.context` bypasses assignment and can cause subtle bugs.
- Context is not deeply reactive; changes notify the actor's subscribers.
- Large context objects increase inspection payload size.

### Annotated Code Examples

```ts
import { setup, assign, createActor } from 'xstate';

const machine = setup({
  types: {
    context: {} as { items: string[]; total: number },
    input: {} as { initialItems: string[] },
  },
}).createMachine({
  context: ({ input }) => ({
    items: input.initialItems,
    total: input.initialItems.length,
  }),
  id: 'cart',
  initial: 'active',
  states: {
    active: {
      on: {
        ADD_ITEM: {
          actions: assign({
            items: ({ context, event }) => [...context.items, event.item],
            total: ({ context }) => context.total + 1,
          }),
        },
      },
    },
  },
});

const actor = createActor(machine, {
  input: { initialItems: ['apple'] },
}).start();

actor.send({ type: 'ADD_ITEM', item: 'banana' });
console.log(actor.getSnapshot().context);
// { items: ['apple', 'banana'], total: 2 }
```

**Expected Output:** The context starts with the input items and updates immutably when `ADD_ITEM` is sent.

**Why this output:** `assign` produces new context; `input` seeds per-actor data.

### Real-World Cases

- **Carts:** Items and totals.
- **Forms:** Field values and validation results.
- **Auth:** Current user and permissions.
- **Games:** Scores, levels, and inventory.

### References

- XState — Context — https://stately.ai/docs/context
- XState — Input — https://stately.ai/docs/input
- XState — TypeScript Context — https://stately.ai/docs/typescript#context

---

## 10. Actors

### Definitions

**Core Definition:** An actor is a running instance of a machine, promise, callback, observable, or transition function that communicates exclusively through events.

**Technical Definition:** In XState v5, every piece of logic is an actor. Actor logic types include: `createMachine` (state machine/statechart), `fromPromise` (async task), `fromCallback` (callback with event listeners), `fromObservable` (RxJS-style observable), `fromEventObservable` (observable emitting events), and `fromTransition` (a reducer-like transition function). Actors are created with `createActor(logic, { input })`, started with `.start()`, and interact via `.send(event)`. Parent machines can `invoke` child actors (declared in `invoke`) or `spawn` them dynamically. Actors can have parents (`sendParent`), children (`sendTo`), and peers (`sendTo` by actor ref). The **actor model** guarantees isolation: actors do not share memory; they communicate only by events.

**Beginner-Friendly Explanation:** An actor is a running thing — a machine, a Promise, a callback — that talks to the world by sending and receiving events. Machines can start other actors, and all communication is explicit.

### Purposes

- To encapsulate independent units of logic with isolated state.
- To model async tasks (promises, callbacks, observables) as first-class actors.
- To orchestrate parent-child communication.
- To enable dynamic creation of actors (`spawn`).
- To make system behaviour observable and testable at every level.

### Syntax Rules and Structure

**Invoke a promise actor:**

```ts
import { setup, fromPromise } from 'xstate';

const fetchUser = fromPromise(async ({ input }: { input: { id: string } }) => {
  const res = await fetch(`/api/users/${input.id}`);
  return res.json();
});

setup({
  actors: { fetchUser },
}).createMachine({
  initial: 'loading',
  states: {
    loading: {
      invoke: {
        src: 'fetchUser',
        input: ({ context }) => ({ id: context.id }),
        onDone: { target: 'success', actions: assign({ user: ({ event }) => event.output }) },
        onError: { target: 'failure' },
      },
    },
    success: {},
    failure: {},
  },
});
```

**Spawn a child actor:**

```ts
import { spawn } from 'xstate';

actions: assign({
  child: ({ spawn }) => spawn('childMachine', { id: 'child' }),
})
```

**Send to an actor:**

```ts
import { sendTo } from 'xstate';

actions: sendTo('child', { type: 'PING' })
```

**`fromCallback`:**

```ts
const listener = fromCallback(({ sendBack, receive }) => {
  const onMessage = (e: MessageEvent) => sendBack({ type: 'MESSAGE', data: e.data });
  const ws = new WebSocket('wss://example.com');
  ws.onmessage = onMessage;
  return () => ws.close();
});
```

**Syntax Rules:**

1. Every actor is created from logic with `createActor(logic, options)`.
2. Parent machines `invoke` children via `invoke: { src, input, onDone, onError }`.
3. Dynamic actors are created with `spawn` inside `assign`.
4. Communication is via `sendTo` (to child/actor), `sendParent` (to parent), `raise` (to self).
5. `fromCallback` cleans up via the returned function; `fromPromise` resolves/rejects.
6. `input` passes initial data to actors; `output` is the actor's final value.

**Constraints and Limitations:**

- Actors are isolated; they cannot directly read each other's context.
- `invoke` actors are stopped when the state exits; `spawn` actors live until stopped or the parent stops.
- Async actors must handle errors (`onError`) to avoid unhandled rejections.
- Actor refs are not serializable; persist state, not actors.

### Annotated Code Examples

```ts
import { setup, assign, fromPromise, createActor } from 'xstate';

const fetchTodos = fromPromise(async () => {
  const res = await fetch('/api/todos');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return (await res.json()) as Todo[];
});

const machine = setup({
  types: {
    context: {} as { todos: Todo[]; error: string | null },
  },
  actors: { fetchTodos },
  actions: {
    setTodos: assign({ todos: ({ event }) => event.output }),
    setError: assign({ error: ({ event }) => event.error.message }),
  },
}).createMachine({
  id: 'todos',
  context: { todos: [], error: null },
  initial: 'idle',
  states: {
    idle: { on: { FETCH: 'loading' } },
    loading: {
      invoke: {
        src: 'fetchTodos',
        onDone: { target: 'success', actions: 'setTodos' },
        onError: { target: 'failure', actions: 'setError' },
      },
    },
    success: { on: { FETCH: 'loading' } },
    failure: { on: { RETRY: 'loading' } },
  },
});

const actor = createActor(machine).start();
actor.send({ type: 'FETCH' });
```

**Expected Output:** `FETCH` invokes `fetchTodos`; on success, `todos` is set and the machine moves to `success`; on failure, `error` is set and it moves to `failure`.

**Why this output:** `invoke` starts the promise actor; `onDone`/`onError` handle resolution; `assign` updates context.

### Real-World Cases

- **Data fetching:** `fromPromise` for API calls.
- **WebSockets:** `fromCallback` for persistent connections.
- **Background workers:** `spawn` for per-item tasks.
- **Parent-child orchestration:** Sending events between machines.

### References

- XState — Actors — https://stately.ai/docs/actors
- XState — Invoke — https://stately.ai/docs/invoke
- XState — Spawn — https://stately.ai/docs/spawn
- XState — Actors (fromPromise, fromCallback, fromObservable, fromTransition) — https://stately.ai/docs/actors#actor-logic

---

## 11. XState React

### Definitions

**Core Definition:** `@xstate/react` is the official React binding for XState, providing hooks (`useMachine`, `useActor`, `useActorRef`, `useSelector`) to connect actors to React components.

**Technical Definition:** `@xstate/react` exports `useMachine(logic, options?)`, `useActor(actorLogic, options?)`, `useActorRef(logic, options?)`, `useSelector(actorRef, selector, compare?)`, and `createActorContext(logic)` for global actor context. `useMachine` creates and starts an actor, returns `[snapshot, send, actorRef]`, and cleans up on unmount. `useActor` is the same for actor logic. `useActorRef` returns the actor ref without subscribing to snapshots (useful when combined with `useSelector`). `useSelector` subscribes to a slice of the snapshot and re-renders only when the selected value changes (using `Object.is` by default, or a custom compare). `createActorContext` provides a typed context for sharing an actor across a subtree.

**Beginner-Friendly Explanation:** `@xstate/react` gives you hooks to run a machine in a React component. `useMachine` starts it and gives you the snapshot and a `send` function. `useSelector` lets you subscribe to just part of the snapshot.

### Purposes

- To run an XState actor inside a React component.
- To subscribe components to actor snapshots.
- To send events from React event handlers.
- To select slices of state for fine-grained re-renders.
- To share an actor across a subtree with `createActorContext`.
- To clean up actors on unmount.

### Syntax Rules and Structure

**`useMachine`:**

```tsx
import { useMachine } from '@xstate/react';
import { toggleMachine } from './machine';

function Toggle() {
  const [snapshot, send] = useMachine(toggleMachine);
  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {snapshot.value === 'active' ? 'ON' : 'OFF'}
    </button>
  );
}
```

**`useActorRef` + `useSelector`:**

```tsx
import { useActorRef, useSelector } from '@xstate/react';

function Count() {
  const actorRef = useActorRef(counterMachine);
  const count = useSelector(actorRef, (s) => s.context.count);
  return <button onClick={() => actorRef.send({ type: 'INCREMENT' })}>{count}</button>;
}
```

**`createActorContext`:**

```tsx
import { createActorContext } from '@xstate/react';

const MachineContext = createActorContext(toggleMachine);

function App() {
  return (
    <MachineContext.Provider>
      <Toggle />
    </MachineContext.Provider>
  );
}

function Toggle() {
  const snapshot = MachineContext.useSelector((s) => s.value);
  const actorRef = MachineContext.useActorRef();
  return <button onClick={() => actorRef.send({ type: 'TOGGLE' })}>{snapshot}</button>;
}
```

**Syntax Rules:**

1. `useMachine` starts the actor on mount and stops it on unmount.
2. `useMachine` returns `[snapshot, send, actorRef]`.
3. `useActor` works with any actor logic (not just machines).
4. `useActorRef` returns the ref without subscribing; combine with `useSelector`.
5. `useSelector` re-renders only when the selected value changes (`Object.is` by default).
6. `createActorContext` provides typed hooks (`useSelector`, `useActorRef`) scoped to a provider.

**Constraints and Limitations:**

- `useMachine` subscribes to the whole snapshot; use `useSelector` for fine-grained control.
- `useSelector` selector must be pure and return a stable value.
- Actor cleanup depends on unmount; do not recreate logic inside render.
- React 18 Strict Mode double-invokes effects; XState handles this but side effects may run twice in development.

### Annotated Code Examples

```tsx
import { useMachine, useSelector } from '@xstate/react';
import { setup, assign } from 'xstate';

const counterMachine = setup({
  types: { context: {} as { count: number } },
}).createMachine({
  id: 'counter',
  context: { count: 0 },
  initial: 'active',
  states: {
    active: {
      on: {
        INCREMENT: {
          actions: assign({ count: ({ context }) => context.count + 1 }),
        },
      },
    },
  },
});

function Counter() {
  const [snapshot, send] = useMachine(counterMachine);
  const count = useSelector(snapshot, (s) => s.context.count);
  return (
    <button onClick={() => send({ type: 'INCREMENT' })}>
      Count: {count}
    </button>
  );
}
```

**Expected Output:** The button displays the count and increments on click; the component re-renders only when `count` changes.

**Why this output:** `useMachine` starts the actor and provides `snapshot`/`send`; `useSelector` subscribes to `context.count` only.

### Real-World Cases

- **Feature components:** Machines for auth, checkout, and multi-step forms.
- **Local UI logic:** Toggle, modal, and wizard machines.
- **Shared actors:** `createActorContext` for app-wide auth.
- **Performance-sensitive components:** `useSelector` for fine-grained subscriptions.

### References

- XState — React Integration — https://stately.ai/docs/xstate-react
- @xstate/react — npm — https://www.npmjs.com/package/@xstate/react
- XState — createActorContext — https://stately.ai/docs/xstate-react#createactorcontext
- XState — useSelector — https://stately.ai/docs/xstate-react#useselector

---

## 12. XState Best Practices

### Definitions

**Core Definition:** XState best practices are the guidelines for modelling machines, structuring statecharts, handling effects, and integrating with React to build maintainable, testable, and scalable applications.

**Technical Definition:** Best practices combine **modelling from events, not states** (decide which events exist first, then derive states), **naming states by meaning** (not by UI), **keeping context minimal** (only data that cannot be derived), **using guards for conditions** (not for state), **using `assign` for context updates** (never direct mutation), **using `fromPromise`/`fromCallback` for async** (instead of side effects in actions), **using `setup()` for typed machines** (actions, guards, actors), **using `createActorContext` for shared actors**, **using `useSelector` for fine-grained subscriptions**, **testing machines as pure functions**, and **visualizing in Stately Studio**.

**Beginner-Friendly Explanation:** Good XState usage means naming states by what they mean, keeping context small, using guards for conditions, using `assign` for updates, using actors for async work, typing machines with `setup`, and testing the machine without React.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Model events first** | Decide what can happen, then derive states |
| **Name states by meaning** | `loading` not `showSpinner` |
| **Keep context minimal** | Only quantitative data that cannot be derived |
| **Use guards for conditions** | Do not encode conditions in state names |
| **Use `assign` for context** | Never mutate context directly |
| **Use actors for async** | `fromPromise`/`fromCallback`, not side effects in actions |
| **Use `setup()` for typing** | Typed context, events, actors, actions, guards |
| **Use `createActorContext`** | Share actors across a subtree with typed hooks |
| **Use `useSelector`** | Fine-grained subscriptions in React |
| **Test machines purely** | Send events and assert snapshots without React |
| **Visualise in Stately Studio** | Catch modelling errors before coding |
| **Avoid over-modelling** | Use `useState` for trivial UI toggles |

### Annotated Code Examples

**✅ Good Practices:**

```ts
import { setup, assign, fromPromise } from 'xstate';

const fetchUser = fromPromise(async ({ input }: { input: { id: string } }) => {
  const res = await fetch(`/api/users/${input.id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
});

const userMachine = setup({
  types: {
    context: {} as { user: User | null; error: string | null },
    events: {} as { type: 'FETCH'; id: string },
  },
  actors: { fetchUser },
  actions: {
    setUser: assign({ user: ({ event }) => event.output }),
    setError: assign({ error: ({ event }) => event.error.message }),
  },
}).createMachine({
  id: 'user',
  context: { user: null, error: null },
  initial: 'idle',
  states: {
    idle: { on: { FETCH: 'loading' } },
    loading: {
      invoke: {
        src: 'fetchUser',
        input: ({ event }) => ({ id: event.id }),
        onDone: { target: 'success', actions: 'setUser' },
        onError: { target: 'failure', actions: 'setError' },
      },
    },
    success: { on: { FETCH: 'loading' } },
    failure: { on: { RETRY: 'loading' } },
  },
});
```

**❌ Bad Practices:**

```ts
// ❌ Boolean flags instead of named states
context: { isLoading: false, isError: false, isSuccess: false }

// ❌ Direct context mutation
actions: ({ context }) => { context.count++; }

// ❌ Async side effect inside an action
actions: async () => { await fetch('/api'); }

// ❌ Conditions encoded in state names
states: { validForm: {}, invalidForm: {} } // use a guard instead

// ❌ Recreating the machine inside render
function Bad() {
  const machine = createMachine({ ... }); // New machine every render
}
```

**Expected Output:** The good practices produce typed, testable, and maintainable machines. The bad practices cause impossible states, mutation bugs, unhandled async, explosion of state names, and re-render issues.

**Why this output:** Boolean flags allow impossible combinations; direct mutation bypasses XState's actor model; async actions are not tracked; state names encoding conditions multiply combinatorially; recreating machines loses actor identity.

### Real-World Cases

- **All XState applications:** These practices apply universally.
- **Large apps:** Feature-scoped machines and `createActorContext`.
- **Async-heavy apps:** `fromPromise`/`fromCallback` for all side effects.
- **Team collaboration:** Visual models in Stately Studio.

### References

- XState — Best Practices — https://stately.ai/docs/best-practices
- XState — Modelling — https://stately.ai/docs/modelling
- XState — TypeScript — https://stately.ai/docs/typescript
- Stately Studio — Visual Editor — https://stately.ai/editor
- XState — Testing — https://stately.ai/docs/testing

---

## References

- XState — Official Documentation — https://stately.ai/docs/xstate
- XState — GitHub Repository — https://github.com/statelyai/xstate
- XState — State Machines — https://stately.ai/docs/state-machines
- Stately — State Machines and Statecharts — https://stately.ai/docs/state-machines-and-statecharts
- XState — States — https://stately.ai/docs/states
- XState — Final States — https://stately.ai/docs/final-states
- XState — Transitions — https://stately.ai/docs/transitions
- XState — Guards — https://stately.ai/docs/guards
- XState — Delayed Transitions — https://stately.ai/docs/delayed-transitions
- XState — Events — https://stately.ai/docs/events
- XState — Actions — https://stately.ai/docs/actions
- XState — Built-in Actions — https://stately.ai/docs/actions#built-in-actions
- XState — Context — https://stately.ai/docs/context
- XState — Input — https://stately.ai/docs/input
- XState — Actors — https://stately.ai/docs/actors
- XState — Invoke — https://stately.ai/docs/invoke
- XState — Spawn — https://stately.ai/docs/spawn
- XState — Parallel States — https://stately.ai/docs/parallel-states
- XState — History States — https://stately.ai/docs/history-states
- XState — React Integration — https://stately.ai/docs/xstate-react
- @xstate/react — npm — https://www.npmjs.com/package/@xstate/react
- XState — TypeScript — https://stately.ai/docs/typescript
- XState — Best Practices — https://stately.ai/docs/best-practices
- XState — Modelling — https://stately.ai/docs/modelling
- XState — Testing — https://stately.ai/docs/testing
- XState — Migration Guide — https://stately.ai/docs/migration
- Stately Studio — Visual Editor — https://stately.ai/editor
- W3C — SCXML Specification — https://www.w3.org/TR/scxml/
- David Harel — Statecharts: A Visual Formalism for Complex Systems — https://www.sciencedirect.com/science/article/pii/0167642387900359
- Wikipedia — Finite-state machine — https://en.wikipedia.org/wiki/Finite-state_machine