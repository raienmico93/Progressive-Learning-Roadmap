# Observables — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An Observable is a lazy, push-based collection that represents a stream of values over time. It is the fundamental building block of RxJS (Reactive Extensions for JavaScript), a library for reactive programming using observables that makes it easier to compose asynchronous or callback-based code.

**Technical Definition:** Per the RxJS documentation, "The Observable object represents a push based collection. The Observer and Observable interfaces provide a generalized mechanism for push-based notification, also known as the observer design pattern. The Observable object represents the object that sends notifications (the provider); the Observer object represents the class that receives them (the observer)". Observables are the foundation of RxJS, but "RxJS is mostly useful for its operators, even though the Observable is the foundation. Operators are the essential pieces that allow complex asynchronous code to be easily composed in a declarative manner". RxJS provides an implementation of the Observable type, which is needed until the type becomes part of the language and until browsers support it.

**Beginner-Friendly Explanation:** An Observable is like a newsletter subscription. You sign up (subscribe), and then you receive issues (values) over time. You can receive many issues, and you can cancel your subscription whenever you want. Unlike a Promise, which is like a single delivery, an Observable can deliver multiple values over time, and nothing happens until you subscribe.

### Key Characteristics

- **Lazy execution:** Observable execution is deferred; computation does not start until subscription.
- **Multiple values:** Observables provide many values over time, unlike Promises which provide one.
- **Cancellable:** Subscriptions can be unsubscribed to cancel ongoing Observable execution.
- **Operator-rich:** RxJS provides a large number of operators for transforming, filtering, combining, and error-handling Observables.
- **Unicast by default:** Each subscribed Observer owns an independent execution of the Observable.
- **Multicast with Subjects:** Subjects allow values to be multicasted to many Observers.
- **Composable:** Observables can be composed using pipeable operators to build complex asynchronous workflows.

### Prerequisites

- Understanding of JavaScript functions, callbacks, and Promises.
- Familiarity with asynchronous programming and the event loop.
- Basic knowledge of the observer pattern (optional but helpful).
- Awareness of functional programming concepts (map, filter, reduce).

### Related Programming Areas

- **Reactive programming:** Observables are the core primitive of reactive programming.
- **Angular:** RxJS is extensively used in Angular for HTTP requests, form handling, and event handling.
- **Event-driven programming:** Observables can take the place of event handlers.
- **State management:** RxJS can be used for state management with any front-end framework.
- **Real-time applications:** WebSockets, Server-Sent Events, and streaming data.
- **Microservices:** Event-driven microservices and side-effect management.

### Core Concepts / Features

---

## Observables

### Definitions

**Core Definition:** An Observable is a lazy, push-based collection that emits values over time. It is the core primitive of RxJS.

**Technical Definition:** Per the RxJS documentation, "The Observable object represents a push based collection. The Observer and Observable interfaces provide a generalized mechanism for push-based notification, also known as the observer design pattern". An Observable communicates with its observers using three types of notifications: `next` (a value), `error` (an error), and `complete` (the stream is finished). Observables are unicast by default: each subscribed Observer owns an independent execution of the Observable.

**Beginner-Friendly Explanation:** An Observable is a stream of values. You subscribe to it, and it pushes values to you over time. It can push many values, or none, and it can also push an error or a completion signal.

### Purposes

- To represent a stream of values over time that can be composed and transformed.
- To provide a unified abstraction for handling asynchronous operations, events, and data streams.
- To enable declarative composition of complex asynchronous workflows.
- To support cancellation and resource cleanup through subscriptions.
- To model push-based data sources such as WebSockets, DOM events, and timers.

### Syntax Rules and Structure

```js
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});

observable.subscribe({
  next(x) { console.log('got value ' + x); },
  error(err) { console.error('something wrong occurred: ' + err); },
  complete() { console.log('done'); }
});
```

**Component Breakdown:**
- `new Observable(subscriber => { ... })`: Creates a new Observable. The function receives a `subscriber` object.
- `subscriber.next(value)`: Emits a value to the Observer.
- `subscriber.error(err)`: Emits an error, terminating the stream.
- `subscriber.complete()`: Signals that the stream is finished.
- `.subscribe(observer)`: Initiates the Observable's execution. The observer can be an object with `next`, `error`, and `complete` methods.

**Syntax Rules:**
- Observables are not executed until `subscribe()` is called.
- The `subscribe()` call returns a `Subscription` object that can be used to cancel the execution.
- Observables can emit zero, one, or many values.
- Once an Observable completes or errors, it cannot emit further values.

**Constraints and Limitations:**
- Observables are unicast by default; each subscription creates a new execution.
- Without unsubscription, Observables can cause memory leaks if they are long-lived.
- Observables are not natively supported in JavaScript; RxJS provides the implementation.

### Annotated Code Examples

#### Example 1: Basic Observable

```js
import { Observable } from 'rxjs';

// Step 1: Create an Observable.
const observable = new Observable(subscriber => {
  subscriber.next('Hello');
  subscriber.next('World');
  subscriber.complete();
});

// Step 2: Subscribe to it.
const subscription = observable.subscribe({
  next(value) { console.log(value); },
  complete() { console.log('Done!'); }
});

// Step 3: Unsubscribe (optional, for cleanup).
// subscription.unsubscribe();
```

**Expected Output:**
```
Hello
World
Done!
```

**Why This Output:** The Observable emits `'Hello'` and `'World'`, then completes. The subscriber receives each value and the completion notification.

#### Example 2: Observable from DOM Events

```js
import { fromEvent } from 'rxjs';

// Step 1: Create an Observable from click events.
const clicks$ = fromEvent(document, 'click');

// Step 2: Subscribe to it.
const subscription = clicks$.subscribe(event => {
  console.log('Clicked at:', event.clientX, event.clientY);
});

// Step 3: Unsubscribe after 5 seconds.
setTimeout(() => subscription.unsubscribe(), 5000);
```

**Expected Output:** Each click logs the coordinates until the subscription is cancelled after 5 seconds.

**Why This Output:** `fromEvent` creates an Observable from DOM events. Each click emits an event object to the subscriber. The subscription is cancelled after 5 seconds to stop receiving events.

### Real-World Cases

- **HTTP requests:** Angular's `HttpClient` returns Observables for HTTP requests.
- **DOM events:** `fromEvent` wraps DOM events into Observables.
- **WebSockets:** Observables can represent WebSocket message streams.
- **Form input:** Observables can track form input changes with debouncing and validation.
- **State management:** RxJS can be used for state management with `BehaviorSubject` and `scan`.

---

## RxJS

### Definitions

**Core Definition:** RxJS (Reactive Extensions for JavaScript) is a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code.

**Technical Definition:** Per the Angular documentation, "RxJS (Reactive Extensions for JavaScript) is a library for reactive programming using observables that makes it easier to compose asynchronous or callback-based code". RxJS provides an implementation of the Observable type, which is needed until the type becomes part of the language and until browsers support it. RxJS has been around since 2011 and is widely used, both on its own and as the basis for a number of other libraries, such as Angular.

**Beginner-Friendly Explanation:** RxJS is the library that gives JavaScript developers the tools to work with Observables. It provides the `Observable` class, a rich set of operators for transforming data streams, and Subjects for multicasting.

### Purposes

- To provide a standardised implementation of Observables for JavaScript.
- To offer a comprehensive set of operators for composing asynchronous and event-based code.
- To enable reactive programming patterns in JavaScript applications.
- To serve as the foundation for frameworks like Angular.
- To handle complex asynchronous workflows with declarative, composable code.

### Syntax Rules and Structure

```js
import { of, from, interval, fromEvent } from 'rxjs';
import { map, filter, take, mergeMap, catchError } from 'rxjs/operators';

// Creating Observables:
const numbers$ = of(1, 2, 3);
const array$ = from([1, 2, 3]);
const timer$ = interval(1000);
const clicks$ = fromEvent(document, 'click');

// Using operators:
const doubled$ = numbers$.pipe(
  filter(x => x % 2 === 1),
  map(x => x * 2)
);

// Subscribing:
doubled$.subscribe(console.log);
```

**Component Breakdown:**
- **Creation operators:** `of`, `from`, `interval`, `fromEvent` — create Observables from various sources.
- **Pipeable operators:** `map`, `filter`, `take`, `mergeMap`, `catchError` — transform, filter, and combine Observables.
- `.pipe(operator1, operator2, ...)`: Chains operators together.

**Syntax Rules:**
- Operators are imported from `rxjs/operators` (or `rxjs` in RxJS 7+).
- `.pipe()` returns a new Observable; it does not modify the original.
- Subscribing to the piped Observable subscribes to the original.

**Constraints and Limitations:**
- RxJS adds a bundle size of approximately 45.6kB (minified).
- Learning curve for the large number of operators.
- Requires careful subscription management to avoid memory leaks.

### Annotated Code Examples

#### Example 1: Basic RxJS Pipeline

```js
import { of } from 'rxjs';
import { map, filter } from 'rxjs/operators';

// Step 1: Create an Observable from values.
const numbers$ = of(1, 2, 3, 4, 5);

// Step 2: Build a pipeline.
const result$ = numbers$.pipe(
  filter(x => x % 2 === 1),  // Keep odd numbers
  map(x => x * 10)            // Multiply by 10
);

// Step 3: Subscribe.
result$.subscribe(value => console.log(value));
```

**Expected Output:**
```
10
30
50
```

**Why This Output:** The pipeline filters odd numbers (1, 3, 5), then multiplies each by 10.

#### Example 2: HTTP Request with Angular

```js
import { HttpClient } from '@angular/common/http';
import { catchError } from 'rxjs/operators';

@Injectable()
export class UserService {
  constructor(private http: HttpClient) {}

  getUsers() {
    return this.http.get('/api/users').pipe(
      catchError(error => {
        console.error('Failed to fetch users:', error);
        throw error;
      })
    );
  }
}
```

**Expected Output:** The users are returned when the request succeeds; errors are logged and re-thrown.

**Why This Output:** Angular's `HttpClient` returns an Observable. The `catchError` operator handles errors from the HTTP request.

### Real-World Cases

- **Angular applications:** RxJS is the default for HTTP, routing, and forms.
- **React state management:** RxJS can be used for reusable state management.
- **Node.js microservices:** RxJS for event-driven microservices.
- **Real-time chat applications:** Observables for WebSocket streams.
- **Data visualisation:** Streaming data to charts and dashboards.

---

## Observables vs Promises

### Definitions

**Core Definition:** Observables and Promises are both used for handling asynchronous operations, but they differ in fundamental ways: Observables can emit multiple values over time, are lazy, and are cancellable; Promises emit a single value, execute immediately, and are not cancellable.

**Technical Definition:** Per the Angular documentation, "Observable execution is deferred; computation does not start until subscription. Promises execute immediately on creation. This makes observables useful for defining recipes that can be run whenever you need the result. Observables provide many values. Promises provide one". Additionally, "Observables can be transformed with operators as well as in the subscription. The rich variety of RxJS operators observables enables complex transformations that can be passed around to other parts of the system, without causing the work to be executed prematurely. Promises have .then() clauses which can transform values but only after the work is done".

**Beginner-Friendly Explanation:** A Promise is like a single delivery: you order once, and you get one package. An Observable is like a subscription: you sign up and receive many issues over time. A Promise starts working immediately when created; an Observable starts only when you subscribe. You can cancel an Observable subscription, but you cannot cancel a Promise.

### Purposes

- To understand when to use Observables vs Promises.
- To recognise that Observables are more powerful but also more complex than Promises.
- To choose the right tool for the task: Promises for single-value async operations, Observables for streams and events.
- To leverage the operator-rich ecosystem of RxJS for complex transformations.

### Syntax Rules and Structure

```js
// Promise:
const promise = fetch('/api/data');
promise.then(data => console.log(data));

// Observable:
const observable$ = from(fetch('/api/data'));
observable$.subscribe(data => console.log(data));
```

**Component Breakdown:**
- **Promise:** Created with `new Promise()` or returned by async APIs. Starts immediately.
- **Observable:** Created with `new Observable()` or creation operators. Starts on `subscribe()`.

**Syntax Rules:**
- Promises use `.then()`, `.catch()`, `.finally()`.
- Observables use `.subscribe()`, `.pipe()`, and return a `Subscription`.
- Observables can be converted to Promises with `lastValueFrom()` or `firstValueFrom()`.

**Constraints and Limitations:**
- Observables are unicast by default; Promises are multicast.
- Promises cannot be cancelled; Observables can be unsubscribed.
- RxJS adds bundle size; Promises are built-in.

### Annotated Code Examples

#### Example 1: Promise vs Observable

```js
// Promise:
const promise = new Promise(resolve => {
  console.log('Promise executor runs immediately');
  resolve('Promise value');
});
promise.then(value => console.log(value));

// Observable:
const observable = new Observable(subscriber => {
  console.log('Observable subscriber runs on subscribe');
  subscriber.next('Observable value');
  subscriber.complete();
});
observable.subscribe(value => console.log(value));
```

**Expected Output:**
```
Promise executor runs immediately
Promise value
Observable subscriber runs on subscribe
Observable value
```

**Why This Output:** The Promise executor runs immediately when the Promise is created. The Observable subscriber function runs only when `subscribe()` is called.

#### Example 2: Multiple Values vs Single Value

```js
// Observable: multiple values.
const multi$ = interval(1000).pipe(take(3));
multi$.subscribe(v => console.log('Observable:', v));

// Promise: single value.
const single = Promise.resolve('Promise: one value');
single.then(v => console.log(v));
```

**Expected Output:**
```
Promise: one value
Observable: 0
Observable: 1
Observable: 2
```

**Why This Output:** The Observable emits three values over three seconds. The Promise emits a single value immediately.

### Real-World Cases

- **Promises:** Single HTTP requests where you only need one response.
- **Observables:** WebSocket streams, DOM events, and complex data flows.
- **Angular:** Uses Observables for HTTP by default but can convert to Promises.
- **React:** Promises are common; Observables can be used with libraries like RxJS.

---

## Operators

### Definitions

**Core Definition:** Operators are functions that allow complex asynchronous code to be easily composed in a declarative manner. They are the essential building blocks of RxJS.

**Technical Definition:** Per the RxJS documentation, "RxJS is mostly useful for its operators, even though the Observable is the foundation. Operators are the essential pieces that allow complex asynchronous code to be easily composed in a declarative manner". There are two kinds of operators: **Pipeable Operators** (piped to Observables using `observableInstance.pipe(operator)`) and **Creation Operators** (standalone functions that create new Observables). A Pipeable Operator is a function that takes an Observable as its input and returns another Observable. It is a pure operation: the previous Observable stays unmodified.

**Beginner-Friendly Explanation:** Operators are like tools in a toolbox. You can use `map` to transform values, `filter` to keep only certain values, `mergeMap` to flatten nested Observables, and `catchError` to handle errors. They let you build complex data pipelines declaratively.

### Purposes

- To transform, filter, combine, and aggregate Observable values.
- To handle errors gracefully with operators like `catchError` and `retry`.
- To control timing with operators like `debounceTime`, `throttleTime`, and `delay`.
- To flatten higher-order Observables with `mergeMap`, `switchMap`, `concatMap`, and `exhaustMap`.
- To compose complex asynchronous workflows declaratively.

### Syntax Rules and Structure

```js
import { of } from 'rxjs';
import { map, filter, take } from 'rxjs/operators';

const result$ = of(1, 2, 3, 4, 5).pipe(
  filter(x => x % 2 === 1),
  map(x => x * 2),
  take(2)
);

result$.subscribe(console.log);
```

**Component Breakdown:**
- `of(1, 2, 3, 4, 5)`: Creation operator that emits the values.
- `.pipe(...)`: Method for chaining pipeable operators.
- `filter(...)`: Keeps values that pass the predicate.
- `map(...)`: Transforms each value.
- `take(2)`: Takes only the first 2 values.

**Syntax Rules:**
- Operators are imported from `rxjs/operators` (RxJS 6) or `rxjs` (RxJS 7+).
- `.pipe()` returns a new Observable; the original is unchanged.
- Operators are pure functions; they do not cause side effects until subscribed.

**Constraints and Limitations:**
- The large number of operators can be overwhelming for beginners.
- Some operators (e.g., `mergeMap`) can lead to memory leaks if inner Observables are not properly managed.
- Choosing the wrong flattening operator can cause race conditions or concurrency issues.

### Annotated Code Examples

#### Example 1: Transformation with `map` and `filter`

```js
import { of } from 'rxjs';
import { map, filter } from 'rxjs/operators';

const numbers$ = of(1, 2, 3, 4, 5);

numbers$.pipe(
  filter(x => x % 2 === 0),  // Keep even numbers
  map(x => x * x)            // Square each
).subscribe(console.log);
```

**Expected Output:**
```
4
16
```

**Why This Output:** `filter` keeps 2 and 4. `map` squares them, producing 4 and 16.

#### Example 2: Error Handling with `catchError`

```js
import { of } from 'rxjs';
import { map, catchError } from 'rxjs/operators';

of(1, 2, 3).pipe(
  map(x => {
    if (x === 2) throw new Error('Something went wrong');
    return x;
  }),
  catchError(err => {
    console.error('Caught:', err.message);
    return of('fallback');
  })
).subscribe(console.log);
```

**Expected Output:**
```
Caught: Something went wrong
fallback
```

**Why This Output:** The `map` operator throws an error when `x === 2`. `catchError` catches it, logs it, and returns a fallback Observable.

#### Example 3: Flattening with `switchMap`

```js
import { fromEvent, of } from 'rxjs';
import { switchMap, debounceTime } from 'rxjs/operators';

const input = document.getElementById('search');
fromEvent(input, 'input').pipe(
  debounceTime(300),
  switchMap(event => of(event.target.value.toUpperCase()))
).subscribe(console.log);
```

**Expected Output:** After 300ms of no typing, the uppercased input value is logged.

**Why This Output:** `debounceTime` waits for a pause in typing. `switchMap` cancels the previous inner Observable and switches to the new one.

### Real-World Cases

- **Search autocomplete:** `debounceTime` + `switchMap` for efficient API calls.
- **HTTP retries:** `retry` and `catchError` for resilient API calls.
- **Combining streams:** `combineLatest`, `merge`, `concat`, `zip` for combining data sources.
- **Rate limiting:** `throttleTime` for limiting event frequency.
- **State management:** `scan` for accumulating state over time.

---

## Subjects

### Definitions

**Core Definition:** A Subject is a special type of Observable that allows values to be multicasted to many Observers. It acts as both an Observable and an Observer.

**Technical Definition:** Per the RxJS documentation, "An RxJS Subject is a special type of Observable that allows values to be multicasted to many Observers. While plain Observables are unicast (each subscribed Observer owns an independent execution of the Observable), Subjects are multicast. A Subject is like an Observable, but can multicast to many Observers. Subjects are like EventEmitters: they maintain a registry of many listeners". Every Subject is an Observable and an Observer: it has the methods `next(v)`, `error(e)`, and `complete()`.

**Beginner-Friendly Explanation:** A Subject is like a radio station. Multiple listeners (Observers) can tune in, and the station broadcasts (multicasts) the same values to everyone. Unlike a regular Observable, which creates a separate execution for each subscriber, a Subject shares a single execution among all subscribers.

### Purposes

- To multicast values to multiple Observers.
- To act as a bridge between the observer and observable worlds.
- To provide a central event bus for sharing data across components.
- To enable state management with `BehaviorSubject` and `ReplaySubject`.
- To allow values to be pushed into an Observable from external code.

### Syntax Rules and Structure

```js
import { Subject } from 'rxjs';

const subject = new Subject<number>();

// Subscribe observers:
subject.subscribe({ next: (v) => console.log(`observerA: ${v}`) });
subject.subscribe({ next: (v) => console.log(`observerB: ${v}`) });

// Push values:
subject.next(1);
subject.next(2);
```

**Component Breakdown:**
- `new Subject<T>()`: Creates a new Subject.
- `subject.subscribe(observer)`: Registers an Observer.
- `subject.next(value)`: Emits a value to all registered Observers.
- `subject.error(err)`: Emits an error to all Observers.
- `subject.complete()`: Completes the Subject.

**Syntax Rules:**
- Subjects are multicast: all Observers receive the same values.
- Subjects are both Observable and Observer.
- Once completed or errored, a Subject cannot emit further values.

**Constraints and Limitations:**
- Subjects are hot: they emit values regardless of whether there are subscribers.
- Late subscribers miss previously emitted values (unless using `ReplaySubject` or `BehaviorSubject`).
- Subjects must be manually completed to release resources.

### Annotated Code Examples

#### Example 1: Basic Subject

```js
import { Subject } from 'rxjs';

const subject = new Subject<number>();

// Step 1: Subscribe two observers.
subject.subscribe({ next: (v) => console.log(`observerA: ${v}`) });
subject.subscribe({ next: (v) => console.log(`observerB: ${v}`) });

// Step 2: Push values.
subject.next(1);
subject.next(2);

// Step 3: Complete.
subject.complete();
```

**Expected Output:**
```
observerA: 1
observerB: 1
observerA: 2
observerB: 2
```

**Why This Output:** Both observers receive the same values because Subjects multicast.

#### Example 2: BehaviorSubject

```js
import { BehaviorSubject } from 'rxjs';

// BehaviorSubject requires an initial value.
const subject = new BehaviorSubject<number>(0);

// Observer A subscribes immediately.
subject.subscribe({ next: (v) => console.log(`observerA: ${v}`) });

// Push values.
subject.next(1);
subject.next(2);

// Observer B subscribes later — receives the current value (2).
subject.subscribe({ next: (v) => console.log(`observerB: ${v}`) });
```

**Expected Output:**
```
observerA: 0
observerA: 1
observerA: 2
observerB: 2
```

**Why This Output:** `BehaviorSubject` emits its current value to new subscribers. Observer B receives the latest value (2) upon subscribing.

### Subject Variants

| Variant | Initial Value | Replay Behavior | Use Case |
|---------|--------------|-----------------|----------|
| `Subject` | No | No | Standard multicasting |
| `BehaviorSubject` | Yes (required) | Emits current value to new subscribers | State management, current value |
| `ReplaySubject` | No | Emits specified number of last values to new subscribers | History replay |
| `AsyncSubject` | No | Emits latest value upon completion | One-time async result |

**ReplaySubject Example:**

```js
import { ReplaySubject } from 'rxjs';

const subject = new ReplaySubject<number>(2); // Replay last 2 values

subject.next(1);
subject.next(2);
subject.next(3);

subject.subscribe({ next: (v) => console.log(`Late observer: ${v}`) });
// Output: Late observer: 2, Late observer: 3
```

### Real-World Cases

- **State management:** `BehaviorSubject` for current application state.
- **Event bus:** `Subject` for cross-component communication.
- **Caching:** `ReplaySubject` for replaying cached data to new subscribers.
- **One-time results:** `AsyncSubject` for HTTP responses that only need the final value.

---

## Subscription Management

### Definitions

**Core Definition:** A Subscription is an object that represents a disposable resource, usually the execution of an Observable. It has an `unsubscribe()` method that disposes the resource and cancels the Observable execution.

**Technical Definition:** Per the RxJS documentation, "A Subscription is an object that represents a disposable resource, usually the execution of an Observable. A Subscription has one important method, `unsubscribe`, that takes no argument and just disposes the resource held by the subscription". Subscriptions can be grouped together using the `add()` method, so that a single `unsubscribe()` call cancels multiple subscriptions.

**Beginner-Friendly Explanation:** A Subscription is like a receipt for your Observable subscription. When you're done, you call `unsubscribe()` to cancel the subscription and free up resources. If you forget to unsubscribe, you can cause memory leaks.

### Purposes

- To cancel ongoing Observable execution and release resources.
- To prevent memory leaks from long-lived subscriptions.
- To group multiple subscriptions for easier cleanup.
- To manage the lifecycle of Observables in components and services.

### Syntax Rules and Structure

```js
import { interval } from 'rxjs';

const observable$ = interval(1000);
const subscription = observable$.subscribe(x => console.log(x));

// Later:
subscription.unsubscribe();
```

**Component Breakdown:**
- `subscription`: The object returned by `subscribe()`.
- `subscription.unsubscribe()`: Cancels the execution and releases resources.
- `subscription.add(childSubscription)`: Groups subscriptions.
- `subscription.remove(childSubscription)`: Removes a child subscription.

**Syntax Rules:**
- Always unsubscribe when the subscription is no longer needed.
- Use `takeUntil` or `take` operators to automatically complete subscriptions.
- Use `AsyncPipe` in Angular templates for automatic unsubscription.

**Constraints and Limitations:**
- Forgetting to unsubscribe causes memory leaks.
- Some Observables complete naturally (e.g., HTTP requests), so unsubscription may not be necessary.
- Unsubscribing from a completed Observable has no effect.

### Annotated Code Examples

#### Example 1: Basic Unsubscription

```js
import { interval } from 'rxjs';

const observable$ = interval(1000);
const subscription = observable$.subscribe(x => console.log(x));

// Step 1: Log values for 3 seconds.
setTimeout(() => {
  subscription.unsubscribe();
  console.log('Unsubscribed');
}, 3000);
```

**Expected Output:**
```
0
1
2
Unsubscribed
```

**Why This Output:** The interval emits values every second. After 3 seconds, `unsubscribe()` stops the Observable from emitting further values.

#### Example 2: Grouping Subscriptions

```js
import { interval } from 'rxjs';

const observable1$ = interval(400);
const observable2$ = interval(300);

const subscription = observable1$.subscribe(x => console.log('first: ' + x));
const childSubscription = observable2$.subscribe(x => console.log('second: ' + x));

subscription.add(childSubscription);

setTimeout(() => {
  subscription.unsubscribe(); // Unsubscribes BOTH
  console.log('Both unsubscribed');
}, 1000);
```

**Expected Output:**
```
second: 0
first: 0
second: 1
first: 1
second: 2
Both unsubscribed
```

**Why This Output:** The `add()` method groups the child subscription with the parent. Calling `unsubscribe()` on the parent unsubscribes both.

#### Example 3: `takeUntil` for Automatic Unsubscription

```js
import { interval, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

const destroy$ = new Subject<void>();

interval(1000).pipe(
  takeUntil(destroy$)
).subscribe(x => console.log(x));

// Later, when the component is destroyed:
destroy$.next();
destroy$.complete();
```

**Expected Output:** Values are logged until `destroy$.next()` is called, at which point the subscription is automatically unsubscribed.

**Why This Output:** `takeUntil` completes the Observable when `destroy$` emits, providing a declarative way to manage subscriptions.

### Real-World Cases

- **Angular components:** Unsubscribe in `ngOnDestroy` or use `takeUntil` with a destroy Subject.
- **React components:** Unsubscribe in `useEffect` cleanup functions.
- **Services:** Manage subscriptions carefully to avoid leaks.
- **RxJS best practice:** Use `AsyncPipe` in templates for automatic unsubscription.

---

## Use Cases

### Definitions

**Core Definition:** Observables and RxJS are used in a wide variety of scenarios, from simple DOM event handling to complex state management and real-time data streaming.

**Technical Definition:** Per the RxJS Cookbook, RxJS provides "40+ real-world solutions for building async, event-driven web apps". RxJS excels in handling asynchronous streams of data and is specifically designed for event-driven programming. It can be used for state management with any front-end framework and is widely used both on its own and as the basis for libraries like Angular.

**Beginner-Friendly Explanation:** Observables and RxJS are useful whenever you need to handle streams of data over time — like user input, WebSocket messages, or HTTP responses. They're especially powerful for complex scenarios like search autocomplete, real-time dashboards, and state management.

### Purposes

- To handle complex event streams (keyboard, mouse, WebSocket).
- To implement search autocomplete with debouncing and cancellation.
- To manage application state with `BehaviorSubject` and `scan`.
- To handle HTTP requests with retry, error handling, and cancellation.
- To build real-time applications with WebSocket streams.
- To compose multiple data sources with `combineLatest`, `merge`, and `zip`.

### Syntax Rules and Structure

Common RxJS use case patterns:

```js
// 1. Search autocomplete:
fromEvent(input, 'input').pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => http.get(`/api/search?q=${query}`))
).subscribe(results => render(results));

// 2. State management:
const state$ = new BehaviorSubject({ count: 0 });
const increment$ = new Subject<void>();
increment$.pipe(
  scan(state => ({ count: state.count + 1 }), { count: 0 })
).subscribe(state$);

// 3. WebSocket stream:
const messages$ = webSocket('wss://api.example.com');
messages$.pipe(
  filter(msg => msg.type === 'notification'),
  map(msg => msg.data)
).subscribe(notification => showNotification(notification));
```

**Component Breakdown:**
- **Search autocomplete:** `debounceTime` for delay, `switchMap` for cancellation.
- **State management:** `BehaviorSubject` for current state, `scan` for accumulation.
- **WebSocket:** `webSocket` for connection, `filter` and `map` for processing.

**Syntax Rules:**
- Use `debounceTime` to reduce API calls.
- Use `switchMap` to cancel previous requests.
- Use `BehaviorSubject` for state that needs a current value.
- Use `scan` to accumulate state over time.

**Constraints and Limitations:**
- RxJS adds bundle size and complexity.
- Overuse of RxJS can make code harder to understand.
- Not all problems require Observables; sometimes Promises or async/await are simpler.

### Annotated Code Examples

#### Example 1: Search Autocomplete

```js
import { fromEvent } from 'rxjs';
import { debounceTime, map, distinctUntilChanged, switchMap } from 'rxjs/operators';

const input = document.getElementById('search');

fromEvent(input, 'input').pipe(
  map(event => event.target.value),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => fetch(`/api/search?q=${query}`).then(r => r.json()))
).subscribe(results => {
  console.log('Results:', results);
});
```

**Expected Output:** After 300ms of no typing, the search results are logged. Previous requests are cancelled.

**Why This Output:** `debounceTime` waits for a pause in typing. `distinctUntilChanged` skips duplicate queries. `switchMap` cancels the previous request and starts a new one.

#### Example 2: State Management with BehaviorSubject

```js
import { BehaviorSubject, Subject } from 'rxjs';
import { scan, startWith } from 'rxjs/operators';

// Actions:
const increment$ = new Subject<void>();
const decrement$ = new Subject<void>();

// Reducer:
const state$ = new BehaviorSubject<number>(0);

increment$.pipe(
  scan(count => count + 1, 0)
).subscribe(state$);

decrement$.pipe(
  scan(count => count - 1, 0)
).subscribe(state$);

// Subscribe to state:
state$.subscribe(count => console.log('Count:', count));

increment$.next();
increment$.next();
decrement$.next();
```

**Expected Output:**
```
Count: 0
Count: 1
Count: 2
Count: 1
```

**Why This Output:** The `BehaviorSubject` holds the current state. Increment and decrement actions update the state via `scan`.

### Real-World Cases

- **Search autocomplete:** Debouncing and cancellation with `debounceTime` and `switchMap`.
- **Real-time dashboards:** WebSocket streams with `webSocket` and `filter`.
- **Form validation:** Observables for tracking input changes and validation.
- **HTTP with retry:** `retry` and `catchError` for resilient API calls.
- **State management:** `BehaviorSubject` and `scan` for application state.
- **Event bus:** `Subject` for cross-component communication.
- **Animations:** `animationFrames` for frame-based animations.

---

## References

- RxJS Documentation – Observable — https://rxjs.dev/guide/observable
- RxJS Documentation – Subject — https://rxjs.dev/guide/subject
- RxJS Documentation – Subscription — https://rxjs.dev/guide/subscription
- RxJS Documentation – Operators — https://rxjs.dev/guide/operators
- RxJS Documentation – API List — https://rxjs.dev/api
- Angular Documentation – Observables compared to other techniques — https://v17.angular.io/guide/comparing-observables
- Learn RxJS – Subject Variants — https://www.learnrxjs.io/learn-rxjs/subjects
- Learn RxJS – Operators — https://www.learnrxjs.io/learn-rxjs/operators
- ReactiveX – Observable Contract — http://reactivex.io/documentation/contract.html
- ReactiveX – Subject — http://reactivex.io/documentation/subject.html
- Auth0 – JavaScript Promises vs. RxJS Observables — https://auth0.com/blog/javascript-promises-vs-rxjs-observables/
- InfoQ – RxJS Best Practices in Angular 16 — https://www.infoq.com/articles/rxjs-best-practices-angular-16/
- Toptal – Using RxJS and React for Reusable State Management — https://www.toptal.com/react/rxjs-react-state-management
- Halodoc – Automating RxJS Cleanup — https://blogs.halodoc.io/automating-rxjs-cleanup/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf