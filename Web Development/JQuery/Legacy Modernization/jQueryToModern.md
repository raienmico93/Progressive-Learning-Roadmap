# jQuery to Modern Frameworks — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Migrating from jQuery to a modern framework is the structured process of replacing a jQuery‑based front end — imperative DOM manipulation, scattered DOM‑based state, plugin dependencies, and full‑page‑reload assumptions — with a component‑based or declarative framework (React, Vue, Angular, Svelte, Alpine, or htmx), while maintaining business continuity, preserving tested behaviour, and avoiding a big‑bang rewrite that freezes feature delivery.

**Technical Definition**
Legacy jQuery applications are characterised by **imperative DOM mutation outside any rendering abstraction**, **state implicit in the DOM**, **global event delegation without namespacing**, and **plugin side effects that are undocumented and untestable**. Modern frameworks assume **exclusive ownership of their DOM subtree** and reconcile that subtree against a virtual or compiled representation. The migration is therefore an **interoperability engineering** problem: two frameworks with different DOM ownership models must coexist long enough for an incremental migration to complete. The dominant strategy is the **Strangler Fig pattern** — building the new system around the edges of the old one and cutting over piece by piece rather than in one release. A mechanical 1:1 port routinely loses behaviour that was never written down, so migration planning must begin with an explicit **behaviour inventory** — component by component, handler by handler — before committing to a strangler boundary.

**Beginner‑Friendly Explanation**
Migrating from jQuery to a modern framework is like renovating a house while people are still living in it. You cannot knock everything down and start over, because the residents need a place to sleep. So you renovate one room at a time — replace the kitchen, then the bathroom — while the rest of the house remains functional. The **Strangler Fig pattern** is the name for this approach: the new code grows around the old code, and piece by piece, the old code is removed. A full rewrite is tempting because it promises a clean slate, but it freezes feature development, concentrates all the risk into a single release, and throws away years of tested behaviour that nobody wrote down. Most teams should migrate incrementally, using seams such as routes, pages, or modules to separate old and new code.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Strangler Fig is the default** | Incremental, seam‑based replacement distributes risk and preserves tested behaviour; full rewrites are the exception for small, isolated surfaces with no seams. |
| **Inventory before migration** | Hidden behaviours — global event delegation, plugin DOM mutation, ad‑hoc accessibility shims, unsanitised HTML string building — are routinely lost in a mechanical port and must be explicitly reproduced. |
| **Coexistence is mandatory** | Old and new code run side by side; boundaries and adapters prevent DOM ownership conflicts. |
| **Parallel running validates behaviour** | Running old and new logic simultaneously on production traffic (shadow shipping) catches mismatches before cutover. |
| **Feature flags decouple deployment from release** | A flag flip can roll back a migrated capability in seconds, without a deployment. |
| **Tests are the safety net** | End‑to‑end tests for critical journeys and visual regression baselines must be established before converting pages. |
| **Dual‑stack overhead is real** | Strangler migrations require two build pipelines, two deploy paths, and two sets of dependencies for the duration of the migration. |

---

### Prerequisites

- A working knowledge of the existing jQuery codebase, including plugin dependencies and event‑handler logic.
- Familiarity with modern JavaScript and at least one target framework (React, Vue, Angular, Svelte, Alpine, or htmx).
- Access to a test suite or the ability to create end‑to‑end and visual regression tests.
- Understanding of DOM ownership models and the target framework's rendering lifecycle.

---

### Related Programming Areas

- **Legacy modernisation** — the broader discipline of updating old systems without disrupting operations.
- **Interoperability engineering** — patterns for making two frameworks share the same runtime.
- **Software testing** — characterisation tests, end‑to‑end tests, and visual regression testing.
- **Feature flag management** — runtime control of exposure and rollback.
- **Security** — legacy jQuery versions carry known CVEs, and unsafe HTML sinks must not be carried forward.

---

### Core Concepts / Features (Index)

1. Migration Planning
2. Migration Strategy
3. Incremental Migration
4. The Strangler Pattern
5. Parallel Running
6. Testing
7. Migration Best Practices


## Core Concept 1: Migration Planning

### Definitions

**Core Definition**
Migration planning is the structured process of understanding the legacy system, inventorying its behaviours and dependencies, selecting a target framework and migration approach, and defining the phases, risks, rollback procedures, and success criteria before any code is changed.

**Technical Definition**
Migration planning for a jQuery codebase is fundamentally an **inventory problem**. A mechanical 1:1 port into a modern framework routinely loses behaviour that was never written down: event delegation bound at `document` level, DOM mutations performed by third‑party plugins outside any framework render cycle, accessibility behaviour (focus trapping, ARIA toggling) implemented ad hoc in a plugin, and unsanitised HTML string building that must not be carried forward unchanged. The planning phase must therefore produce a **component‑by‑component, handler‑by‑handler inventory** before committing to a strangler boundary. The plan also selects a migration strategy (incremental coexistence versus big‑bang rewrite), defines the seams, establishes the testing safety net, and specifies rollback procedures for each phase.

**Beginner‑Friendly Explanation**
Migration planning is the “look before you leap” phase. You do not start rewriting code on day one. You start by understanding what you have: every jQuery plugin, every global event handler, every piece of state hidden in the DOM, and every accessibility behaviour that a plugin silently provides. You also decide how you will migrate — one page at a time, one component at a time, or a full rewrite — and you write down how you will know when each phase is done and how to roll back if something goes wrong. Skipping this step is the single most common cause of migration failure: the new code looks correct but silently drops behaviours that nobody remembered were there.

---

### Purposes

- **To inventory hidden behaviours** — event delegation, plugin side effects, accessibility shims, and unsanitised HTML — before any migration plan commits to a boundary.
- **To identify global state hidden in the DOM or jQuery data**, which is the hardest thing to migrate and the source of most regressions.
- **To select the migration strategy** (incremental coexistence versus big‑bang rewrite) based on risk, size, and seams.
- **To define the phases, rollback procedures, and success criteria** so that the migration is measurable and reversible.
- **To establish the testing safety net** (end‑to‑end and visual regression baselines) before converting pages.

---

### Syntax Rules and Structure

| Planning Artifact | What It Captures |
|---|---|
| **jQuery usage inventory** | Every `$(`, `$.ajax`, `.on`, `.html`, plugin registration, and global variable. |
| **Behaviour inventory** | Event delegation targets, plugin side effects (DOM mutation, timers, global state), accessibility shims. |
| **State inventory** | Where state lives: DOM, jQuery data, global variables, cookies, hidden inputs. |
| **Plugin mapping** | Each jQuery plugin → native component or modern replacement. |
| **Seam map** | Routes, pages, modules, or iframes that separate old and new code. |
| **Rollback plan** | Per‑phase procedure for reverting to the legacy implementation. |

---

### Constraints and Limitations

- **Inventory is tedious but non‑negotiable**; a plan that skips it ships a component tree that looks equivalent and silently regresses on events, side effects, or accessibility.
- **Hidden state is the hardest migration problem**; it is routinely missed and breaks converted features.
- **Planning does not replace execution**; the plan must be paired with a testing safety net and a rollback strategy.

---

### Annotated Code Example (Inventory Script)

```bash
# Step 1: Inventory jQuery usage
grep -rE '\$\(|jQuery\(|\.ajax|\.get\(|\.post\(|\.on\(|\.html\(' \
  resources/js public/js --include='*.js' > jquery-usage.txt

# Step 2: Inventory plugin registrations
grep -rE '\.fn\.|\.plugin\(' resources/js --include='*.js' > plugins.txt

# Step 3: Inventory global state candidates
grep -rE 'window\.[a-zA-Z_]+|var [a-zA-Z_]+ =' resources/js --include='*.js' > globals.txt
```

**Expected Output**
Three files listing every jQuery call site, plugin registration, and global variable. The inventory feeds the behaviour inventory and the seam map.

**Why this output occurs**: `grep` surfaces every static occurrence, making the hidden behaviours visible before any migration decision is committed.

---

### Real‑World Cases

- **Banking platform modernisation**: The Strangler Fig approach required a detailed inventory of legacy modules before incremental replacement and rollback could be planned.
- **Internal tools application**: A five‑phase migration began with a component‑by‑component inventory of jQuery event handlers and plugin side effects.

---

## Core Concept 2: Migration Strategy

### Definitions

**Core Definition**
Migration strategy is the high‑level approach chosen for moving from jQuery to a modern framework — typically **incremental coexistence** (Strangler Fig) or **big‑bang rewrite** — along with the interoperability patterns that allow old and new code to share the runtime during the transition.

**Technical Definition**
Two strategies dominate. **Big‑bang rewrite** replaces the entire front end in a single release; it is appropriate only for small, frozen applications with clear boundaries, and it carries high risk, long duration, and a feature freeze. **Incremental coexistence** introduces the new framework alongside jQuery, mounts new components into existing pages, and replaces one section at a time behind a routing layer or feature flag. The interoperability patterns for incremental coexistence include: mounting framework components inside jQuery‑managed containers, wrapping jQuery plugins as framework components, a custom event bridge, a shared state synchronisation layer, route‑based strangling, islands‑style adaptation, and build tooling such as module federation. The strategy must also account for **DOM ownership conflict**: jQuery assumes it can freely read and mutate the DOM, while the framework assumes exclusive ownership of its subtree; the two work only when their assumptions hold in different regions of the document.

**Beginner‑Friendly Explanation**
Migration strategy is the big decision: do you replace everything at once, or do you replace one piece at a time? Replacing everything at once is like moving to a new house in a single day — it is fast if everything goes perfectly, but if something breaks, you have nowhere to sleep. Replacing one piece at a time is like renovating room by room — slower, but you always have a working house. Most teams choose incremental coexistence because it lets them keep shipping features and because each small change is easy to verify and easy to undo. The strategy also decides how the old and new code will talk to each other: will the new framework live inside the old page, or will a routing layer send some users to the new app and others to the old one?

---

### Purposes

- **To choose between incremental coexistence and big‑bang rewrite** based on risk, size, and seams.
- **To select the interoperability patterns** that let jQuery and the framework share the runtime safely.
- **To define the routing or boundary layer** that decides which code serves each request.
- **To avoid the failure modes of big‑bang rewrites** — feature freeze, concentrated risk, and knowledge loss.
- **To align the strategy with the team's capacity** to maintain two stacks during the transition.

---

### Syntax Rules and Structure

| Strategy | When to Choose | Key Mechanism |
|---|---|---|
| **Strangler Fig (incremental)** | Default for most applications with seams | Routing layer directs traffic to old or new code |
| **Big‑bang rewrite** | Small, frozen apps with clear boundaries | Full replacement in one release |
| **Component‑level coexistence** | Pages with isolated widgets | Mount framework components into jQuery containers |
| **Route‑level strangling** | Applications with distinct page routes | New routes → new app; old routes → legacy |

---

### Constraints and Limitations

- **Big‑bang rewrites concentrate risk** and freeze feature delivery; they are rarely the right first move.
- **Incremental coexistence requires dual‑stack capacity**; the team must maintain two build pipelines and two dependency sets.
- **DOM ownership conflicts must be resolved** with explicit boundaries, or the application will flicker and break.

---

### Annotated Code Example (Route‑Level Strangler)

```javascript
// Routing layer: new routes → new framework; old routes → jQuery
const LEGACY_ROUTES = ['/dashboard', '/reports'];
const NEW_ROUTES = ['/new-dashboard', '/new-reports'];

app.use((req, res, next) => {
  if (NEW_ROUTES.some(r => req.path.startsWith(r))) {
    return newFrameworkHandler(req, res, next);
  }
  if (LEGACY_ROUTES.some(r => req.path.startsWith(r))) {
    return legacyJQueryHandler(req, res, next);
  }
  next();
});
```

**Expected Output**
Requests to `/new-dashboard` are served by the new framework; requests to `/dashboard` are served by the legacy jQuery app. As routes are migrated, they move from `LEGACY_ROUTES` to `NEW_ROUTES`.

**Why this output occurs**: The routing layer implements the Strangler Fig decision per request, allowing both applications to coexist in production.

---

### Real‑World Cases

- **FreeMarker + Vue 2 + jQuery → React**: The team explicitly chose Strangler Fig over a Next.js full SPA rewrite to maintain delivery speed, minimise uncertainty, and avoid the cost of dual development.
- **Banking platform**: React components were introduced incrementally through a routing layer, with rollback possible at any time.
- **E‑commerce platform**: Route‑based strangling with micro‑frontends replaced jQuery interfaces incrementally.

---

## Core Concept 3: Incremental Migration

### Definitions

**Core Definition**
Incremental migration is the practice of moving an application from jQuery to a modern framework in small, independent, verifiable steps — replacing one route, page, module, or component at a time — while keeping the application functional and shippable throughout the transition.

**Technical Definition**
Incremental migration distributes risk across many small releases rather than concentrating it into a single cutover. Each increment is scoped by a **seam** — a route boundary, an iframe boundary, a distinct page load, or a module boundary — where legacy and new code can coexist without deep coupling. The migration proceeds **business‑risk‑first**: low‑risk surfaces (internal admin pages, isolated widgets) are migrated before high‑risk surfaces (checkout, authentication). The literature describes incremental migration as the default because replacing a serious system takes a long time, and users cannot wait for new features while it happens; old and new systems coexist in production for the whole duration. The incremental approach also enables **progressive exposure** and **instant rollback** through feature flags, moving one capability at a time and comparing it against the legacy path.

**Beginner‑Friendly Explanation**
Incremental migration is like replacing the parts of a car while it is still driving. You do not take the engine out and hope for the best; you replace one part at a time, test it, and move on to the next part. Each part is a small, contained change that you can verify and, if necessary, roll back. This is safer than a big‑bang rewrite, which is like taking the whole car apart and hoping it goes back together correctly. Incremental migration keeps the product shippable and the team sane.

---

### Purposes

- **To distribute risk** across many small, reversible changes.
- **To keep the application shippable** throughout the migration.
- **To preserve tested behaviour** by migrating one slice at a time and diffing behaviour.
- **To maintain feature velocity** by avoiding a feature freeze.
- **To enable rollback** at each phase if something goes wrong.

---

### Syntax Rules and Structure

| Phase | Activity |
|---|---|
| **Assessment** | Inventory jQuery usage, map dependencies, identify high‑risk areas, define success metrics. |
| **Safety net** | Add end‑to‑end tests, visual regression baselines, feature flags, and rollback procedures. |
| **Incremental execution** | Replace components or routes one at a time using the Strangler Fig pattern; run old and new paths in parallel. |
| **Stabilisation** | Run regression suites, monitor adoption and error rates, deprecate legacy paths, document the new architecture. |

---

### Constraints and Limitations

- **Dual‑stack overhead** is real: two build pipelines, two deploy paths, and two sets of dependencies must be maintained.
- **Seams may not exist** in tightly coupled codebases; a bounded rewrite of a small surface may be necessary.
- **Shared auth/session models** must be solved before coexistence begins.

---

### Annotated Code Example (Component‑Level Increment)

```javascript
// Phase 1: Mount a new React widget into a jQuery-managed page
$('#react-widget').each(function() {
  ReactDOM.createRoot(this).render(<NewWidget />);
});

// Phase 2: Migrate a second widget
$('#react-widget-2').each(function() {
  ReactDOM.createRoot(this).render(<SecondWidget />);
});

// Phase 3: Once all widgets are migrated, remove jQuery
```

**Expected Output**
Each new React widget is mounted into its container. jQuery continues to manage the rest of the page until all widgets are migrated and jQuery is removed.

**Why this output occurs**: The incremental approach introduces the framework at the component level, leaving the existing jQuery code untouched. Each migration step is independent and verifiable.

---

### Real‑World Cases

- **Vue 3 logistics dashboard**: Phased migration starting with the shipment table, then state management, then routing, over multiple releases.
- **React banking platform**: Strangler Fig migration with zero downtime, using React component libraries and incremental traffic routing.
- **FreeMarker + Vue 2 + jQuery → React**: Incremental replacement of high‑impact components, with the business stopping only during brief cutover windows.

---

## Core Concept 4: The Strangler Pattern

### Definitions

**Core Definition**
The Strangler Fig pattern is a migration strategy where the new system is built around the edges of the old system and gradually replaces it — route by route, module by module — until the old system can be removed entirely.

**Technical Definition**
Named after the strangler fig vine that gradually envelops and eventually replaces its host tree, the pattern was described by Martin Fowler as an alternative to big‑bang rewrites. The new system is built alongside the old one, and a **routing layer** or **adapter layer** directs traffic to either the legacy or the new implementation depending on the route, feature, or user segment. The old system continues to operate until every surface has been migrated, at which point it is decommissioned. In frontend migrations, the routing layer can be as simple as a list of migrated paths. The pattern is the **default recommendation** for jQuery migrations because it distributes risk, preserves tested behaviour, and avoids feature freezes. A **bounded rewrite** is recommended only when no viable seam exists, the surface is small and isolated, and the team explicitly accepts the discovery risk.

**Beginner‑Friendly Explanation**
The Strangler Fig pattern is like building a new road next to an old road and gradually diverting traffic to the new road, one lane at a time. The old road stays open until every car is using the new road. Then you close the old road. In software, the new system grows around the old one, and a routing layer decides which system handles each request. This is safer than closing the old road and hoping the new road works. The Strangler Fig is the default way to migrate from jQuery because it lets you keep shipping features while you modernise.

---

### Purposes

- **To distribute migration risk** across many small cutovers.
- **To preserve business logic** embedded in the legacy system.
- **To maintain feature velocity** during the migration.
- **To enable rollback** at the route or feature level.
- **To avoid the failure modes of big‑bang rewrites** — feature freeze, concentrated risk, knowledge loss.

---

### Syntax Rules and Structure

```
┌─────────────────┐
│   Routing Layer  │
│  (nginx, CDN,   │
│   framework)    │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼───┐
│Legacy │ │ New  │
│jQuery │ │React │
└───────┘ └──────┘
```

**Rule**: New routes go to the modern app; old routes stay in the legacy app. Migrate one route at a time until the legacy app is empty.

---

### Constraints and Limitations

- **No seams at all** is the strongest signal toward a bounded rewrite of that specific surface.
- **Dual‑stack operational overhead** must be sustainable for the duration of the migration.
- **Shared auth/session models** must be solved before coexistence begins.

---

### Annotated Code Example (Strangler with Feature Flag)

```javascript
// Feature flag controls which implementation serves the route
const useNewCheckout = featureFlag.isEnabled('new-checkout', user);

if (useNewCheckout) {
  return newCheckoutHandler(req, res);
}
return legacyCheckoutHandler(req, res);
```

**Expected Output**
Users for whom the flag is enabled receive the new checkout; all others receive the legacy checkout. Rollback is a flag flip, not a deployment.

**Why this output occurs**: The feature flag decouples deployment from release, allowing progressive exposure and instant rollback.

---

### Real‑World Cases

- **Banking platform**: The Strangler Fig pattern enabled zero‑downtime migration from jQuery monoliths to React, with incremental replacement and rollback at any time.
- **FreeMarker + Vue 2 + jQuery → React**: The team chose Strangler Fig over a full Next.js rewrite to maintain delivery speed and minimise uncertainty.
- **Enterprise UI modernisation**: Route‑level strangler fig with both apps coexisting, migrating one route at a time.

---

## Core Concept 5: Parallel Running

### Definitions

**Core Definition**
Parallel running (also called shadow shipping or shadow mode) is the practice of executing both the legacy and the new implementation on the same production traffic, serving the legacy result to the user while recording every mismatch between the two, so that behavioural differences are discovered before the new path is exposed.

**Technical Definition**
In a parallel run, every request executes the legacy path and the new path; the user always receives the legacy result, and the system records any mismatch between the two. Parallel runs are safe only around code that does not change data; they are used to validate behavioural parity, not to test side effects. The pattern turns feature‑flag anxiety into data‑driven confidence by running both versions and observing how they differ. BDD with shadow mode testing verifies that the new implementation matches legacy behaviour before switching production traffic, reducing migration risk. Feature‑flagged rollout then uses the mismatch data to decide when the new path is ready for progressive exposure.

**Beginner‑Friendly Explanation**
Parallel running is like having two calculators solve the same math problem at the same time. The user sees the answer from the old calculator, but you secretly compare it to the answer from the new calculator. If the answers match, you gain confidence that the new calculator works. If they differ, you have found a bug before the user ever saw it. This is much safer than switching everyone to the new calculator and hoping it works. The only catch is that you cannot use this technique for actions that change data — you cannot “parallel run” a bank transfer, because the money would move twice.

---

### Purposes

- **To validate behavioural parity** between legacy and new implementations before cutover.
- **To discover mismatches** in production traffic without exposing users to the new path.
- **To build data‑driven confidence** for progressive rollout and feature‑flag decisions.
- **To reduce migration risk** by catching silent regressions that tests may miss.
- **To complement feature flags** by providing the evidence needed to expand exposure safely.

---

### Syntax Rules and Structure

```javascript
// Parallel run: execute both paths, serve legacy, record mismatch
async function handleRequest(req, res) {
  const legacyResult = await legacyPath(req);
  const newResult = await newPath(req);

  if (!isEqual(legacyResult, newResult)) {
    recordMismatch({ req, legacyResult, newResult });
  }

  return res.send(legacyResult);   // user always gets legacy result
}
```

**Constraint**: Only safe around **read‑only** code paths; never parallel‑run mutations.

---

### Constraints and Limitations

- **Parallel runs are only safe around code that does not change data**.
- **Both paths must be fully executable**; partial implementations cannot be parallel‑run.
- **Mismatch noise** must be triaged; not every difference is a bug.

---

### Annotated Code Example (Shadow Shipping)

```javascript
// ✅ Shadow shipping: both versions run, legacy result served
function processOrder(order) {
  const legacyTotal = legacyCalculateTotal(order);
  const newTotal = newCalculateTotal(order);

  if (legacyTotal !== newTotal) {
    logMismatch({ orderId: order.id, legacyTotal, newTotal });
  }

  return legacyTotal;              // legacy result is served
}
```

**Expected Output**
The user receives the legacy total, and any difference between the two calculations is logged for investigation. Once mismatches reach zero, the new path can be enabled via feature flag.

**Why this output occurs**: The parallel run provides empirical evidence of behavioural equivalence before cutover, turning feature‑flag anxiety into data‑driven confidence.

---

### Real‑World Cases

- **Mergify shadow shipping**: Ran old and new logic in parallel to validate behaviour in production before rollout, turning feature‑flag anxiety into data‑driven confidence.
- **BDD with shadow mode**: Verified that the new implementation matched legacy behaviour before switching production traffic.
- **Feature‑flagged canary rollout**: Used parallel‑run mismatch data to decide when to expand exposure.

---

## Core Concept 6: Testing

### Definitions

**Core Definition**
Migration testing is the safety net that proves the new implementation behaves like the legacy system — using end‑to‑end tests for critical user journeys, visual regression baselines for layout and styling, characterisation tests for legacy behaviour, and parallel‑run mismatch records for production parity.

**Technical Definition**
Migration testing operates at four levels. **End‑to‑end tests** exercise critical user journeys through the browser, confirming that the migrated flow works from the user's perspective. **Visual regression testing** captures screenshots of legacy pages and compares them against the migrated pages, catching unintended CSS, layout, and rendering changes that assertion‑based tests miss. **Characterisation tests** (also called golden master tests) capture the current behaviour of the legacy code so that any unintended change during refactoring is detected. **Parallel‑run mismatch records** provide production evidence of behavioural differences between the old and new paths. The migration checklist explicitly requires end‑to‑end tests and a visual baseline before converting pages, and warns that skipping tests removes the only proof that the conversion is safe.

**Beginner‑Friendly Explanation**
Testing is how you know the migration is working. End‑to‑end tests walk through the important user journeys — logging in, searching, checking out — and confirm that everything still works. Visual regression tests take screenshots of the old pages and compare them to the new pages, catching subtle layout changes that a functional test would miss. Characterisation tests lock in the current behaviour of the legacy code so that refactoring does not accidentally change it. And parallel‑run mismatch records show you, from real production traffic, where the old and new code disagree. Without these tests, you are guessing; with them, you have proof.

---

### Purposes

- **To prove that the migrated implementation behaves like the legacy system**.
- **To catch visual and layout regressions** that functional tests miss.
- **To lock in legacy behaviour** before refactoring with characterisation tests.
- **To provide production evidence of parity** through parallel‑run mismatch records.
- **To enable confident, reversible cutover** by knowing that the new path is safe.

---

### Syntax Rules and Structure

| Test Type | What It Catches | Tool Examples |
|---|---|---|
| **End‑to‑end** | Broken user journeys across pages | Playwright, Cypress |
| **Visual regression** | Layout shifts, clipped elements, colour changes | Percy, Playwright screenshots |
| **Characterisation** | Unintended behaviour changes during refactoring | Jest, Vitest |
| **Parallel‑run mismatch** | Production behavioural differences | Custom mismatch logging |

---

### Constraints and Limitations

- **Tests must be written before conversion**, not after; retrofitting tests to migrated code does not prove parity.
- **Visual regression baselines must be captured on the legacy pages** before migration begins.
- **Characterisation tests describe current behaviour, not desired behaviour**; they are a safety net, not a specification.

---

### Annotated Code Example (Visual Regression Baseline)

```javascript
// Step 1: Capture a visual baseline of the legacy page
await page.goto('/legacy/checkout');
const baseline = await page.screenshot();

// Step 2: After migration, compare the new page to the baseline
await page.goto('/new/checkout');
const current = await page.screenshot();
expect(current).toMatchSnapshot(baseline);
```

**Expected Output**
The new page's screenshot is compared against the legacy baseline. Any unintended layout or styling change is flagged before the page reaches production.

**Why this output occurs**: Visual regression testing catches changes that assertion‑based tests miss, such as a shifted layout or a clipped element.

---

### Real‑World Cases

- **jQuery to Modern SPA Migration Checklist**: Explicitly requires end‑to‑end tests for critical journeys and a visual regression baseline of legacy pages before converting them.
- **Applitools / VisualCeption**: Visual testing tools that use screenshots and image comparison to detect unintended UI changes.
- **BDD with shadow mode**: Characterisation and parallel‑run testing verified behavioural parity before cutover.

---

## Core Concept 7: Migration Best Practices

### Definitions

**Core Definition**
jQuery migration best practices are the consolidated conventions — inventory before migrating, establish tests first, use the Strangler Fig pattern, migrate in risk order, maintain dual‑stack discipline, use feature flags for progressive exposure and instant rollback, and document the transition — that make the migration safe, measurable, and maintainable.

**Technical Definition**
Best practices synthesise the individual strategies: **(1) inventory hidden behaviours** before committing to a boundary; **(2) establish end‑to‑end and visual regression tests** before converting pages; **(3) default to the Strangler Fig pattern**, scoped by seams (routes, modules, iframes); **(4) sequence migrations business‑risk‑first** — low‑risk surfaces before high‑risk flows; **(5) use feature flags to decouple deployment from release**, enabling progressive exposure and instant rollback; **(6) use parallel runs for read‑only paths** to validate behavioural parity before cutover; **(7) avoid the DOM ownership conflict** by giving the framework exclusive control of its subtree and jQuery exclusive control of its own; and **(8) document the migration decisions and the new architecture**.

**Beginner‑Friendly Explanation**
Best practices are the rules that keep a migration from turning into a disaster. Start by understanding what you have — count the jQuery calls, list the plugins, find the global state, and write down the accessibility behaviours that plugins silently provide. Write tests before you change anything, so you know when you break something. Use the Strangler Fig pattern: replace one small piece at a time, test it, and move on. Migrate the easy, low‑risk stuff first. Use feature flags so you can turn the new code on for a few users at a time and turn it off instantly if something goes wrong. Use parallel runs on read‑only paths to compare old and new results. Keep the old and new code in separate DOM territories so they do not fight. And write down what you did, so the next person can understand it.

---

### Purposes

- **To provide a single actionable checklist** for jQuery‑to‑framework migration.
- **To prevent the most common migration failures** — silent behaviour loss, big‑bang risk concentration, and missing rollback.
- **To balance migration speed with operational stability**.
- **To establish a baseline for planning, execution, and code reviews**.

---

### Syntax Rules and Structure

| Practice | Recommended | Avoid |
|---|---|---|
| **Inventory** | Behaviour inventory before any boundary commitment | Starting without knowing the scope |
| **Tests** | End‑to‑end + visual regression before conversion | Converting without a safety net |
| **Pattern** | Strangler Fig, scoped by seams | Big‑bang rewrite |
| **Sequencing** | Low‑risk surfaces first | Starting with the highest‑risk flow |
| **Exposure control** | Feature flags for progressive rollout and instant rollback | Cutting over for everyone at once |
| **Parity validation** | Parallel runs on read‑only paths | Assuming the new code matches |
| **DOM ownership** | Framework owns its subtree; jQuery owns its own | Both mutating the same nodes |
| **Documentation** | Document decisions and new architecture | Leaving the next team guessing |

---

### Constraints and Limitations

- **Best practices must be adapted to context**; a small internal tool has different risk tolerances than a customer‑facing checkout.
- **Strangler migrations require sustained dual‑stack capacity**; if the team cannot maintain two stacks, the sequencing must change.
- **Some legacy plugins are fundamentally incompatible** with modern frameworks; a bounded rewrite of that specific surface may be the only viable option.

---

### Annotated Code Example (Consolidated Migration Checklist)

```text
✅ 1. Inventory: hidden behaviours, plugins, global state, accessibility shims
✅ 2. Tests: end-to-end + visual regression baselines before conversion
✅ 3. Seams: identify route, module, and iframe boundaries
✅ 4. Strangler: routing layer; new routes → new app, old routes → legacy
✅ 5. Sequence: migrate low-risk routes/widgets first
✅ 6. Flags: progressive exposure and instant rollback per capability
✅ 7. Parallel run: compare old and new on read-only paths
✅ 8. DOM ownership: framework owns its subtree; jQuery owns its own
✅ 9. Decommission: remove legacy code paths; document new architecture
```

**Expected Output**
A structured, risk‑managed migration that proceeds in verifiable increments and ends with the legacy system fully decommissioned.

**Why this output occurs**: Each step addresses a specific migration risk: scope uncertainty (inventory), behaviour regression (tests), concentrated risk (Strangler), uncontrolled exposure (flags), unverified parity (parallel run), DOM conflicts (ownership boundary), and incomplete migration (decommission and documentation).

---

### Real‑World Cases

- **Banking platform**: Strangler Fig with feature‑flagged canary rollout, parallel‑run validation, and per‑route rollback.
- **FreeMarker + Vue 2 + jQuery → React**: Strangler Fig chosen specifically to maintain delivery speed, minimise uncertainty, and avoid the cost of dual development.
- **Internal tools application**: Five‑phase migration with a behaviour inventory, event bridge, and progressive route exposure.

---

## References

- Modernizing a Legacy Banking Platform: Strangler Fig Approach for Zero‑Downtime Migration | JISEM — https://www.jisem-journal.com/index.php/journal/article/view/15166
- 『全部作り直し』の誘惑を捨てた。Strangler Fig Patternで挑むフロントエンドモダン化作戦 | Zenn — https://zenn.dev/gakken_leap/articles/bd8167a771d2d5
- Hybrid jQuery React Interoperability Strategies During Incremental Frontend Framework Migration | IJCTECE — https://ijctece.com/index.php/IJCTEC/article/download/716/651/1381
- Feature Flags for Cloud Migrations and Modernization | Unleash — https://docs.getunleash.io/guides/feature-flags-for-migrations
- jQuery to Modern SPA Migration Checklist | Vibgrate — https://vibgrate.com/checklists/jquery-to-spa-migration-checklist/
- Legacy jQuery to Modern Framework Review | Vanguard Frontier Agentic — https://github.com/VincentChuWaiChow/vanguard-frontier-agentic/blob/master/skills/frontend/legacy-jquery-to-modern-framework-review/SKILL.md
- Strangler Fig, Branch by Abstraction, Vertical Slices: Choosing a Frontend Migration Strategy | DEV Community — https://dev.to
- Shadow Shipping: How We Double‑Executed Code to Ship Safely | Mergify — https://blog.mergify.com
- Visual Regression Testing — https://qa.codes
- Characterisation Testing — https://en.wikipedia.org/wiki/Characterization_test
- jQuery 4.0.0 Support | Telerik — https://www.telerik.com/blogs/jquery-4-0-0-support-whats-new-kendo-ui-has-you-covered
- Legacy Modernizer | LobeHub — https://lobehub.com