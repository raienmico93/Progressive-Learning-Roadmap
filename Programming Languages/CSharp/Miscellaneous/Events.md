# C# Events — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **event** in C# is a member that enables a class or object (the **publisher**) to notify other classes or objects (the **subscribers**) when something of interest occurs. Events are built on top of **delegates**, which define the signature that event handlers must match. The publisher decides *when* to raise the event; the subscribers decide *what action* to take in response. Events are the backbone of the .NET event model, powering everything from UI button clicks to file-system watchers and domain notifications.

**Technical Definition**
An event is a member that enables an object to trigger notifications. Event users can attach executable code for events by supplying event handlers. The `event` keyword declares an event, which is of a **delegate type**. Events are **multicast delegates** that can only be invoked from within the class (or derived classes) that declares them — the publisher. Subscribers attach handlers using the `+=` operator and detach them using `-=`. When the publisher raises the event, all attached handlers are invoked **synchronously** in the order they were added. In the .NET class library, the standard event pattern uses the `EventHandler` and `EventHandler<TEventArgs>` delegate types, paired with the `EventArgs` base class. Events support access modifiers (`public`, `private`, `protected`, `internal`), and the accessors can be compiler-generated (**field-like events**) or explicitly defined using `add` and `remove` blocks.

**Beginner-Friendly Explanation**
An event is a **notification system**. Imagine a button on a screen. The button does not know — or care — what should happen when it is clicked. It simply announces: “I was clicked!” (that is the **event**). Any part of the program that cares can **subscribe** to that announcement and run its own code when it hears it. The button is the **publisher**; the code that reacts is the **subscriber**. The glue that connects them is a **delegate**, which is a type-safe reference to a method. Events keep publishers and subscribers **loosely coupled**: the button does not need to know anything about the code that reacts to it, and multiple subscribers can react to the same event independently. This loose coupling is what makes event-driven programming so powerful — you can add, remove, or change subscribers without touching the publisher.

---

### Key Characteristics

- **Publisher–Subscriber model** — The publisher raises the event; subscribers handle it.
- **Built on delegates** — Events define the method signature that handlers must match.
- **Multicast** — Multiple subscribers can attach to the same event; all handlers are invoked when the event is raised.
- **Encapsulated invocation** — Events can only be raised from within the declaring class (or derived classes), preventing external code from firing the event.
- **`+=` and `-=` subscription** — Handlers are added and removed via operators that invoke the event’s `add` and `remove` accessors.
- **Standard pattern** — `EventHandler` for events without payload; `EventHandler<TEventArgs>` for events with custom data.
- **Null-safe raising** — Use `?.Invoke` to avoid `NullReferenceException` when no subscribers exist.
- **Memory retention risk** — The publisher holds strong references to subscribers; forgetting to unsubscribe can cause memory leaks.
- **Weak event pattern** — An alternative that uses weak references to allow subscribers to be garbage-collected even if the publisher outlives them.

---

### Prerequisites

- Understanding of C# **delegates** (declaration, instantiation, invocation, multicast).
- Familiarity with `Func<T>`, `Action<T>`, and the built-in `EventHandler` delegates.
- Knowledge of **lambda expressions** and **anonymous methods** (for inline handlers).
- Awareness of **reference types**, **garbage collection**, and **memory leaks**.
- Basic understanding of the **publisher–subscriber** design pattern.

---

### Related Programming Areas

- **Delegates** — Events are built on delegates; every event has an underlying delegate type.
- **Design patterns** — Observer pattern, pub-sub, and event-driven architecture.
- **UI frameworks** — WPF, WinForms, ASP.NET Core, and MAUI rely heavily on events.
- **Asynchronous programming** — Events are typically synchronous, but async handlers are possible.
- **Memory management** — Weak events address the memory-leak problem inherent in strong event subscriptions.

---

### Core Concepts / Features

1. Events (Overarching Concept)
2. `event` Keyword
3. Event Declaration
4. Event Subscription
5. Event Unsubscription
6. Event Handlers
7. Event Accessors
8. Event Best Practices
9. Weak Event Pattern

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Events (Overarching Concept)

### Definitions

**Core Definition**
An event is a member that allows an object to broadcast notifications to interested parties, using delegates as the underlying mechanism.

**Technical Definition**
Events are, like delegates, a **late-binding mechanism**. Events are a way for an object to broadcast (to all interested components in the system) that something happened. Events are based on the language support for delegates. The `event` keyword defines an event of a delegate type; subscribers attach handlers with `+=` and detach with `-=`. The publisher raises the event, typically by invoking the underlying delegate, often through a protected virtual method named `On…`.

**Beginner-Friendly Explanation**
An event is a **shout into the room**. Anyone who has subscribed to the shout hears it and reacts.

---

### Purposes

- To **notify multiple subscribers** when something happens.
- To **decouple** the sender (publisher) from the receivers (subscribers).
- To support **event-driven architecture** and the observer pattern.
- To enable **UI programming** (button clicks, menu selections, etc.).
- To provide **type-safe notifications** via delegates.

---

### Syntax Rules and Structure

```
// Declaration
public event EventHandler? MyEvent;

// Subscription
publisher.MyEvent += OnMyEvent;

// Raising
MyEvent?.Invoke(this, EventArgs.Empty);
```

**Component breakdown**

- `event` — The keyword that declares the event.
- `EventHandler` — The delegate type (or a custom delegate).
- `+=` — Subscribes a handler.
- `-=` — Unsubscribes a handler.
- `?.Invoke` — Null-safe invocation.

#### Syntax Rules

- Events are declared with the `event` keyword followed by a delegate type.
- Events can be `public`, `private`, `protected`, `internal`, or `protected internal`.
- Events can be `static`, `virtual`, `override`, `abstract`, or `sealed`.
- Events can only be raised from within the declaring class (or derived classes).
- Events without subscribers are never raised.

#### Constraints and Limitations

- The publisher holds **strong references** to subscribers, which can cause memory leaks if subscribers do not unsubscribe.
- Events are typically invoked **synchronously**; async handlers require special handling.
- An exception in one handler prevents subsequent handlers from running.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Event

```csharp
using System;

public class Button
{
    // Declare the event using the standard EventHandler delegate
    public event EventHandler? Clicked;

    public void OnClick()
    {
        // Raise the event safely
        Clicked?.Invoke(this, EventArgs.Empty);
    }
}

class Program
{
    static void Main()
    {
        var button = new Button();
        button.Clicked += (sender, e) => Console.WriteLine("Button clicked!");
        button.OnClick();   // Button clicked!
    }
}
```

**Expected Output**

```
Button clicked!
```

**Why this output occurs**

- `Clicked` is declared as an `EventHandler?` event. The subscriber attaches a lambda via `+=`. When `OnClick` calls `Clicked?.Invoke`, the lambda runs and prints the message.

---

#### Example 2: Multiple Subscribers

```csharp
using System;

public class Publisher
{
    public event EventHandler? SomethingHappened;

    public void Trigger() => SomethingHappened?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void Main()
    {
        var pub = new Publisher();

        pub.SomethingHappened += (s, e) => Console.WriteLine("Handler 1");
        pub.SomethingHappened += (s, e) => Console.WriteLine("Handler 2");

        pub.Trigger();
        // Handler 1
        // Handler 2
    }
}
```

**Expected Output**

```
Handler 1
Handler 2
```

**Why this output occurs**

- Both subscribers are attached to the multicast event. Invoking the event runs handlers in subscription order.

---

### Real-World Cases

- **UI frameworks** — Button clicks, menu selections, text changes.
- **File system watchers** — `FileSystemWatcher` raises events when files change.
- **Domain events** — `OrderPlaced`, `UserRegistered` in DDD.
- **Timers** — `System.Timers.Timer.Elapsed`.

---

### References

- Events — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/
- Examine events in C# — Training - https://learn.microsoft.com/en-us/training/modules/create-manage-events/2-examine-events-csharp
- Introduction to events — C# - https://learn.microsoft.com/en-us/dotnet/csharp/events-overview

---

## 2. `event` Keyword

### Definitions

**Core Definition**
The `event` keyword is the syntactic marker that declares an event member inside a class or struct.

**Technical Definition**
The `event` keyword declares an event. The event is of a **delegate type**. While an object triggers an event, the event invokes all supplied event handlers. Event handlers are delegate instances added to the event and executed when the event is raised. The `event` keyword provides **encapsulation** over a delegate field: external code can only subscribe (`+=`) or unsubscribe (`-=`), but cannot invoke the event or replace the delegate.

**Beginner-Friendly Explanation**
`event` is the **“announcement system” keyword**. It tells C# to create a protected delegate that outsiders can listen to but cannot fire.

---

### Purposes

- To declare an event member with proper encapsulation.
- To prevent external code from raising the event.
- To provide a subscription surface (`+=`/`-=`) for handlers.
- To enable the compiler to generate `add`/`remove` accessors.

---

### Syntax Rules and Structure

```
[access modifier] event DelegateType EventName;
```

**Component breakdown**

- `access modifier` — `public`, `private`, etc.
- `event` — The keyword.
- `DelegateType` — The delegate type (e.g., `EventHandler`, `EventHandler<T>`).
- `EventName` — The event identifier.

#### Syntax Rules

- The delegate type determines the handler signature.
- Events can be `static`, `virtual`, `override`, `abstract`, or `sealed`.
- Events cannot be `const`, `readonly`, or `volatile`.
- The event keyword can be used with custom `add`/`remove` accessors.

#### Constraints and Limitations

- External code cannot invoke the event directly.
- External code cannot assign a new delegate to the event (`=` is forbidden).
- The underlying delegate is accessible only within the declaring class.

---

### Annotated Code Examples and Expected Outputs

#### Example: `event` Keyword Encapsulation

```csharp
using System;

public class Publisher
{
    public event EventHandler? MyEvent;

    public void Raise() => MyEvent?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void Main()
    {
        var pub = new Publisher();
        pub.MyEvent += (s, e) => Console.WriteLine("Handled");

        // pub.MyEvent = null; // Compile error: can only use += or -=
        // pub.MyEvent();       // Compile error: can only appear on left side of += or -=

        pub.Raise();   // Handled
    }
}
```

**Expected Output**

```
Handled
```

**Why this output occurs**

- The `event` keyword prevents external assignment and invocation; only the publisher can raise the event.

---

### Real-World Cases

- **All event declarations** — The `event` keyword is mandatory.

---

### References

- The `event` keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/event
- Events — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-events

---

## 3. Event Declaration

### Definitions

**Core Definition**
Event declaration is the syntax that introduces a new event, specifying its delegate type and name, and optionally custom accessors.

**Technical Definition**
Event declaration uses the `event` keyword followed by a delegate type and an identifier. There are two forms: **field-like events**, where the compiler generates the underlying delegate field and the `add`/`remove` accessors automatically, and **custom events**, where the developer provides explicit `add` and `remove` accessor blocks. Field-like events are the most common form.

**Beginner-Friendly Explanation**
Declaring an event is **creating the announcement channel**. You pick the delegate type (the shape of the message) and give the event a name.

---

### Purposes

- To introduce a new event member.
- To specify the handler signature via the delegate type.
- To choose between compiler-generated and custom accessors.
- To control accessibility and modifiers.

---

### Syntax Rules and Structure

```
// Field-like event (compiler-generated accessors)
public event EventHandler? MyEvent;

// Custom event with explicit accessors
private EventHandler? _myEvent;
public event EventHandler? MyEvent
{
    add { _myEvent += value; }
    remove { _myEvent -= value; }
}
```

**Component breakdown**

- `event` — The keyword.
- `DelegateType` — The delegate type.
- `EventName` — The event identifier.
- `{ add { ... } remove { ... } }` — Optional custom accessors.

#### Syntax Rules

- Field-like events automatically create a private delegate field and public accessors.
- Custom events require the developer to store and manage the delegate manually.
- The `add` and `remove` accessors receive the handler as the implicit `value` parameter.
- Events can be declared with any access modifier.

#### Constraints and Limitations

- Custom events must handle null delegates carefully.
- Field-like events are not thread-safe by default for compound assignment.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Field-Like Event

```csharp
using System;

public class Thermometer
{
    public event EventHandler<double>? TemperatureChanged;
}
```

**Expected Output (conceptual)**

```
// Compiler generates a private delegate field and add/remove accessors.
```

**Why this output occurs**

- The field-like event syntax tells the compiler to generate the storage and accessors automatically.

---

#### Example 2: Custom Event with Explicit Accessors

```csharp
using System;

public class Thermometer
{
    private EventHandler<double>? _temperatureChanged;

    public event EventHandler<double>? TemperatureChanged
    {
        add
        {
            _temperatureChanged += value;
            Console.WriteLine("Subscriber added");
        }
        remove
        {
            _temperatureChanged -= value;
            Console.WriteLine("Subscriber removed");
        }
    }

    public void SetTemperature(double t) =>
        _temperatureChanged?.Invoke(this, t);
}
```

**Expected Output (conceptual)**

```
// Custom accessors allow logging, filtering, or alternative storage.
```

**Why this output occurs**

- The explicit `add` and `remove` accessors intercept subscription and unsubscription, enabling custom logic.

---

### Real-World Cases

- **Large classes with many events** — Custom accessors can store handlers in a dictionary to avoid many null fields.
- **Interface implementations** — Explicit event accessors for interface members.
- **Event relay** — Forwarding subscriptions to another object.

---

### References

- Classes — C# language specification (event declaration) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-events
- Event accessors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1583-event-accessors
- The `event` keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/event

---

## 4. Event Subscription

### Definitions

**Core Definition**
Event subscription is the process of attaching a handler to an event using the `+=` operator, which invokes the event's `add` accessor.

**Technical Definition**
You subscribe to an event by using the `+=` operator. The `+=` operation compiles to a call to the event's `add_XXX` method, which adds the handler to the underlying delegate's invocation list. Multiple handlers can be attached; each is invoked in subscription order when the event is raised.

**Beginner-Friendly Explanation**
Subscribing is **putting your name on the notification list** with `+=`.

---

### Purposes

- To **receive notifications** when the event is raised.
- To **attach multiple independent handlers** to a single event.
- To **use lambdas, method groups, or anonymous methods** as handlers.

---

### Syntax Rules and Structure

```
// Lambda handler
publisher.Event += (sender, args) => { ... };

// Method group handler
publisher.Event += HandlerMethod;

// Anonymous method handler
publisher.Event += delegate { ... };
```

**Component breakdown**

- `publisher.Event` — The event to subscribe to.
- `+=` — The subscription operator.
- Handler — Lambda, method group, or anonymous method.

#### Syntax Rules

- The handler signature must match the delegate type.
- Multiple `+=` calls add multiple handlers.
- The same handler can be added multiple times (it will be invoked multiple times).

#### Constraints and Limitations

- Subscribing inside a constructor can cause issues if the object is not fully initialized.
- Subscriptions create strong references, risking memory leaks if not removed.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Lambda Subscription

```csharp
using System;

public class Button
{
    public event EventHandler? Clicked;
    public void Click() => Clicked?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void Main()
    {
        var btn = new Button();
        btn.Clicked += (s, e) => Console.WriteLine("Clicked!");
        btn.Click();   // Clicked!
    }
}
```

**Expected Output**

```
Clicked!
```

**Why this output occurs**

- The lambda is converted to an `EventHandler` and added to the invocation list.

---

#### Example 2: Method Group Subscription

```csharp
using System;

public class Button
{
    public event EventHandler? Clicked;
    public void Click() => Clicked?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void HandleClick(object? sender, EventArgs e) =>
        Console.WriteLine("Handled by method");

    static void Main()
    {
        var btn = new Button();
        btn.Clicked += HandleClick;
        btn.Click();
    }
}
```

**Expected Output**

```
Handled by method
```

**Why this output occurs**

- The method group `HandleClick` is converted to the delegate type via target typing.

---

### Real-World Cases

- **UI event handlers** — Subscribing to button clicks, text changes, etc.
- **Domain events** — Subscribing to `OrderPlaced` in a separate service.
- **Logging** — Attaching log handlers to application events.

---

### References

- How to subscribe to and unsubscribe from events — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/how-to-subscribe-to-and-unsubscribe-from-events
- Examine events in C# — Training (subscription with `+=`) - https://learn.microsoft.com/en-us/training/modules/create-manage-events/2-examine-events-csharp

---

## 5. Event Unsubscription

### Definitions

**Core Definition**
Event unsubscription is the process of detaching a handler from an event using the `-=` operator, which invokes the event's `remove` accessor.

**Technical Definition**
The `-=` operator removes the handler from the underlying delegate's invocation list. The `-=` operation compiles to a call to the event's `remove_XXX` method. Removing a handler that was never added is a no-op. Failing to unsubscribe can cause **memory leaks** because the publisher holds strong references to subscribers.

**Beginner-Friendly Explanation**
Unsubscribing is **taking your name off the notification list** with `-=`.

---

### Purposes

- To **stop receiving notifications**.
- To **prevent memory leaks** in long-lived publishers.
- To **clean up resources** when a subscriber is disposed.
- To **avoid duplicate handler invocations**.

---

### Syntax Rules and Structure

```
publisher.Event -= HandlerMethod;
```

**Component breakdown**

- `publisher.Event` — The event to unsubscribe from.
- `-=` — The unsubscription operator.
- `HandlerMethod` — The handler to remove.

#### Syntax Rules

- The handler must match a previously added instance.
- Removing a non-existent handler is a no-op.
- Unsubscribing in `Dispose` is a common pattern.

#### Constraints and Limitations

- Unsubscribing a lambda requires storing the delegate instance.
- Forgetting to unsubscribe causes memory leaks in long-lived publishers.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Unsubscribe with a Method Group

```csharp
using System;

public class Publisher
{
    public event EventHandler? MyEvent;
    public void Raise() => MyEvent?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void Handler(object? s, EventArgs e) => Console.WriteLine("Handled");

    static void Main()
    {
        var pub = new Publisher();
        pub.MyEvent += Handler;
        pub.Raise();   // Handled

        pub.MyEvent -= Handler;
        pub.Raise();   // (no output)
    }
}
```

**Expected Output**

```
Handled
```

**Why this output occurs**

- The handler is removed; the second `Raise` finds no subscribers.

---

#### Example 2: Unsubscribe in `Dispose`

```csharp
using System;

public class Subscriber : IDisposable
{
    private readonly Publisher _pub;
    private readonly EventHandler _handler;

    public Subscriber(Publisher pub)
    {
        _pub = pub;
        _handler = (s, e) => Console.WriteLine("Received");
        _pub.MyEvent += _handler;
    }

    public void Dispose() => _pub.MyEvent -= _handler;
}
```

**Expected Output (conceptual)**

```
// The handler is removed during Dispose, preventing memory leaks.
```

**Why this output occurs**

- Storing the handler delegate allows unsubscription in `Dispose`.

---

### Real-World Cases

- **IDisposable pattern** — Unsubscribe in `Dispose`.
- **UI lifecycle** — Unsubscribe when a view is closed.
- **Long-lived services** — Prevent leaks from orphaned subscribers.

---

### References

- How to subscribe to and unsubscribe from events — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/how-to-subscribe-to-and-unsubscribe-from-events
- Event unsubscription best practices — https://learncsharpmastery.com/event-best-practices (search result)

---

## 6. Event Handlers

### Definitions

**Core Definition**
An event handler is a method (or lambda) that matches the event's delegate signature and is invoked when the event is raised.

**Technical Definition**
Event handlers are delegate instances added to the event and executed when the event is raised. The standard .NET pattern requires handlers with signature `void Handler(object? sender, TEventArgs e)`. The `sender` is the object that raised the event; `e` carries event-specific data. For events without payload, `EventArgs.Empty` is used.

**Beginner-Friendly Explanation**
An event handler is the **code that runs when the announcement is heard**. It must have the right shape: `(sender, args) => { ... }`.

---

### Purposes

- To **respond to events** with custom logic.
- To **provide event-specific data** via `TEventArgs`.
- To **identify the sender** for context.
- To **support multiple independent reactions** to the same event.

---

### Syntax Rules and Structure

```
// Standard handler signature
void Handler(object? sender, EventArgs e) { ... }

// Generic handler with custom data
void Handler(object? sender, MyEventArgs e) { ... }
```

**Component breakdown**

- `sender` — The publisher (usually `this`).
- `e` — Event arguments; `EventArgs.Empty` if no data.
- Return type — `void`.

#### Syntax Rules

- The handler must match the delegate type's signature.
- Handlers are invoked synchronously in subscription order.
- Exceptions in one handler skip subsequent handlers.

#### Constraints and Limitations

- Cannot return a value (handlers return `void`).
- Should avoid long-running operations (consider async patterns).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Standard `EventHandler`

```csharp
using System;

public class Button
{
    public event EventHandler? Clicked;
    public void Click() => Clicked?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void OnClicked(object? sender, EventArgs e) =>
        Console.WriteLine($"Sender: {sender?.GetType().Name}");

    static void Main()
    {
        var btn = new Button();
        btn.Clicked += OnClicked;
        btn.Click();
    }
}
```

**Expected Output**

```
Sender: Button
```

**Why this output occurs**

- The handler receives the button as `sender` and prints its type name.

---

#### Example 2: `EventHandler<TEventArgs>` with Custom Data

```csharp
using System;

public class TemperatureChangedEventArgs : EventArgs
{
    public double OldValue { get; }
    public double NewValue { get; }
    public TemperatureChangedEventArgs(double oldValue, double newValue)
    {
        OldValue = oldValue;
        NewValue = newValue;
    }
}

public class Thermostat
{
    public event EventHandler<TemperatureChangedEventArgs>? TemperatureChanged;
    private double _temperature;

    public void SetTemperature(double newValue)
    {
        var old = _temperature;
        _temperature = newValue;
        TemperatureChanged?.Invoke(this, new TemperatureChangedEventArgs(old, newValue));
    }
}

class Program
{
    static void Main()
    {
        var thermostat = new Thermostat();
        thermostat.TemperatureChanged += (s, e) =>
            Console.WriteLine($"{e.OldValue} → {e.NewValue}");
        thermostat.SetTemperature(25);   // 0 → 25
    }
}
```

**Expected Output**

```
0 → 25
```

**Why this output occurs**

- The handler receives a `TemperatureChangedEventArgs` instance carrying the old and new values.

---

### Real-World Cases

- **UI event handlers** — `Button_Click`, `TextChanged`.
- **Domain event handlers** — `OrderPlacedHandler`.
- **Progress reporting** — `ProgressChanged`.

---

### References

- Standard .NET event patterns — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/event-pattern
- EventHandler<TEventArgs> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler-1
- EventHandler Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler

---

## 7. Event Accessors

### Definitions

**Core Definition**
Event accessors are the `add` and `remove` blocks that control how handlers are attached to and detached from an event. By default, the compiler generates them automatically for field-like events.

**Technical Definition**
An event's accessors are the implementations of its `+=` and `-=` functions. By default, accessors are implemented implicitly by the compiler: for a field-like event `public event EventHandler PriceChanged;`, the compiler generates a **private delegate field** and **public `add`/`remove` accessors** that forward to that field. You can take over this process by defining **explicit event accessors** using the `add` and `remove` keywords. With explicit accessors, you can apply more complex strategies to the storage and access of the underlying delegate.

**Beginner-Friendly Explanation**
Event accessors are the **hooks** that run when someone subscribes (`+=`) or unsubscribes (`-=`). By default, the compiler writes them for you; you can write your own for custom behavior.

---

### Purposes

- To **customize subscription behavior** (e.g., logging, filtering).
- To **relay subscriptions** to another object.
- To **store handlers in a dictionary** when a class has many events.
- To **implement interface events explicitly**.

---

### Syntax Rules and Structure

```
private EventHandler? _myEvent;

public event EventHandler? MyEvent
{
    add { _myEvent += value; }
    remove { _myEvent -= value; }
}
```

**Component breakdown**

- `add { ... }` — Invoked on `+=`.
- `remove { ... }` — Invoked on `-=`.
- `value` — The implicit parameter holding the handler.

#### Syntax Rules

- Custom accessors must be provided for both `add` and `remove`.
- The backing delegate must be managed manually.
- Accessors compile to `add_XXX` and `remove_XXX` methods.

#### Constraints and Limitations

- Custom accessors are more complex and error-prone.
- Thread safety must be handled manually.

---

### Annotated Code Examples and Expected Outputs

#### Example: Custom Accessors with Logging

```csharp
using System;

public class Publisher
{
    private EventHandler? _myEvent;

    public event EventHandler? MyEvent
    {
        add
        {
            _myEvent += value;
            Console.WriteLine("Subscribed");
        }
        remove
        {
            _myEvent -= value;
            Console.WriteLine("Unsubscribed");
        }
    }

    public void Raise() => _myEvent?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void Main()
    {
        var pub = new Publisher();
        EventHandler handler = (s, e) => Console.WriteLine("Handled");

        pub.MyEvent += handler;   // Subscribed
        pub.Raise();              // Handled
        pub.MyEvent -= handler;   // Unsubscribed
    }
}
```

**Expected Output**

```
Subscribed
Handled
Unsubscribed
```

**Why this output occurs**

- The custom `add` and `remove` accessors log the subscription lifecycle.

---

### Real-World Cases

- **Event relay** — Forwarding subscriptions to a child object.
- **Sparse events** — Storing handlers in a dictionary for classes with many events.
- **Interface events** — Explicit implementation of interface-declared events.

---

### References

- Event accessors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1583-event-accessors
- Chapter 4. Advanced C# — Event Accessors - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2008/ff652506(v=orm.10)
- C# Event Accessors — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/12-c-sharp-delegates-and-events/7-c-sharp-event-accessors.mdx

---

## 8. Event Best Practices

### Definitions

**Core Definition**
Event best practices are the design and implementation guidelines that ensure events are **safe, maintainable, and leak-free**.

**Technical Definition**
Best practices for events derive from the .NET Framework Design Guidelines and community experience. Key recommendations include: **always unsubscribe** when a subscriber is no longer needed (preferably in `Dispose`), **use the standard `EventHandler` / `EventHandler<TEventArgs>` pattern**, **raise events through a protected virtual `On…` method**, **use `?.Invoke` for null-safe raising**, **avoid subscribing in constructors**, **consider weak events for long-lived publishers**, and **do not expose public delegate fields** (use `event` instead).

**Beginner-Friendly Explanation**
Best practices for events are: **always unsubscribe**, **use the standard pattern**, **raise safely with `?.Invoke`**, and **do not subscribe in constructors**.

---

### Purposes

- To **prevent memory leaks** from forgotten subscriptions.
- To **maintain consistency** with the .NET event model.
- To **allow derived classes to raise events** via `On…` methods.
- To **avoid null-reference exceptions** when raising events.
- To **choose the right pattern** for long-lived publishers.

---

### Guidelines and Structure

#### General Guidelines

```
// 1. Unsubscribe in Dispose
public void Dispose() => _publisher.MyEvent -= _handler;

// 2. Use EventHandler<T> for custom data
public event EventHandler<MyEventArgs>? MyEvent;

// 3. Raise via protected virtual On… method
protected virtual void OnMyEvent(MyEventArgs e) =>
    MyEvent?.Invoke(this, e);

// 4. Null-safe invocation
MyEvent?.Invoke(this, EventArgs.Empty);

// 5. Avoid subscribing in constructors
// 6. Consider WeakEventManager for long-lived publishers
```

**Component breakdown**

- **Unsubscribe in `Dispose`** — Prevents leaks.
- **Standard delegates** — `EventHandler` / `EventHandler<T>`.
- **`On…` method** — Enables derived classes to raise events.
- **`?.Invoke`** — Null-safe.
- **Weak events** — For independent lifetimes.

#### Rules and Constraints

- **Do** unsubscribe when the subscriber is disposed.
- **Do** use `EventHandler<TEventArgs>` for custom data.
- **Do** raise events through a protected `On…` method.
- **Do** use `?.Invoke` to avoid null-reference exceptions.
- **Do not** subscribe in constructors.
- **Do not** expose public delegate fields.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Event Pattern

```csharp
using System;

public class FileFoundEventArgs : EventArgs
{
    public string FileName { get; }
    public FileFoundEventArgs(string fileName) => FileName = fileName;
}

public class FileSearcher
{
    public event EventHandler<FileFoundEventArgs>? FileFound;

    protected virtual void OnFileFound(FileFoundEventArgs e) =>
        FileFound?.Invoke(this, e);

    public void Search()
    {
        // ... find file ...
        OnFileFound(new FileFoundEventArgs("report.pdf"));
    }
}

class Program
{
    static void Main()
    {
        var searcher = new FileSearcher();
        searcher.FileFound += (s, e) =>
            Console.WriteLine($"Found: {e.FileName}");
        searcher.Search();   // Found: report.pdf
    }
}
```

**Expected Output**

```
Found: report.pdf
```

**Why this output occurs**

- The event uses `EventHandler<TEventArgs>`, is raised via a protected `OnFileFound` method, and is invoked with `?.Invoke`.

---

### Real-World Cases

- **Library design** — Public events follow the standard pattern.
- **UI components** — `OnClick` methods enable derived controls to raise events.
- **Long-running services** — Unsubscribe and weak events prevent leaks.

---

### References

- Standard .NET event patterns — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/event-pattern
- Events — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/
- Event Best Practices — https://learncsharpmastery.com/event-best-practices (search result)
- Framework Design Guidelines — Events - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/event

---

## 9. Weak Event Pattern

### Definitions

**Core Definition**
The **weak event pattern** is a design pattern that allows event subscribers to be garbage-collected even when the publisher outlives them, by using **weak references** instead of strong references.

**Technical Definition**
In the standard .NET event implementation, the publisher holds a **strong reference** to each subscriber's delegate, preventing the subscriber from being garbage-collected as long as the publisher lives. The weak event pattern addresses this by using a **`WeakEventManager`** (or custom weak references) to mediate between publisher and subscriber. The manager adds and removes listeners using weak references, allowing subscribers to be collected when they are no longer needed. `WeakEventManager` is available in WPF (`System.Windows.WeakEventManager`) and as a generic `WeakEventManager<TEventSource, TEventArgs>`. The pattern is recommended when the event source has an **object lifetime independent of the event listeners**.

**Beginner-Friendly Explanation**
The weak event pattern is **a way to listen to events without preventing the listener from being garbage-collected**. Normally, subscribing to an event makes the publisher keep your object alive forever. With weak events, the publisher only *weakly* references you, so you can still be collected when you are done.

---

### Purposes

- To **prevent memory leaks** when publishers outlive subscribers.
- To allow **subscribers to be garbage-collected** independently.
- To support **long-lived publishers** (UI controls, services, data sources).
- To maintain loose coupling without lifetime coupling.

---

### Syntax Rules and Structure

```
// Using WeakEventManager<TEventSource, TEventArgs>
WeakEventManager<EventSource, EventArgs>.AddHandler(
    source, "MyEvent", OnMyEvent);
```

**Component breakdown**

- `WeakEventManager<TEventSource, TEventArgs>` — Type-safe weak event manager.
- `AddHandler` — Subscribes with a weak reference.
- `RemoveHandler` — Unsubscribes.

#### Syntax Rules

- `WeakEventManager` is available in WPF and some other frameworks.
- The generic variant `WeakEventManager<TEventSource, TEventArgs>` provides type safety.
- The manager uses weak references internally to track listeners.

#### Constraints and Limitations

- `WeakEventManager` is framework-specific (WPF).
- Custom weak event implementations are possible but complex.
- Weak events may have slightly higher overhead than strong events.

---

### Annotated Code Examples and Expected Outputs

#### Example: Weak Event Pattern with `WeakEventManager`

```csharp
using System;
using System.Windows;

public class EventSource
{
    public event EventHandler? MyEvent;
    public void Raise() => MyEvent?.Invoke(this, EventArgs.Empty);
}

public class Subscriber
{
    public Subscriber(EventSource source)
    {
        WeakEventManager<EventSource, EventArgs>.AddHandler(
            source, nameof(EventSource.MyEvent), OnMyEvent);
    }

    private void OnMyEvent(object? sender, EventArgs e) =>
        Console.WriteLine("Received");

    ~Subscriber() => Console.WriteLine("Subscriber finalized");
}

class Program
{
    static void Main()
    {
        var source = new EventSource();
        var sub = new Subscriber(source);

        source.Raise();   // Received

        sub = null;
        GC.Collect();
        GC.WaitForPendingFinalizers();

        source.Raise();   // (no output; subscriber collected)
    }
}
```

**Expected Output**

```
Received
Subscriber finalized
```

**Why this output occurs**

- The `WeakEventManager` holds a weak reference to the subscriber. After `sub` is set to `null` and garbage collection runs, the subscriber is collected and no longer receives the event.

---

### Real-World Cases

- **WPF data binding** — `PropertyChangedEventManager` for `INotifyPropertyChanged`.
- **Long-lived services** — Subscribers come and go.
- **Plugin systems** — Plugins subscribe to host events and can be unloaded.

---

### References

- WeakEventManager Class (System.Windows) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.windows.weakeventmanager
- WeakEventManager<TEventSource,TEventArgs> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.windows.weakeventmanager-2
- Weak Event Patterns — WPF .NET Framework - https://learn.microsoft.com/en-us/dotnet/framework/wpf/advanced/weak-event-patterns
- C# 弱事件模式 — GitHub (cool-coding) - https://github.com/echo-cool-coding/cool-coding/blob/main/docs/language/csharp/7-c-attributes-and-events/13-c-weak-event-pattern.mdx

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `event` keyword | Core since **C# 1.0**; stable. |
| Field-like events | Core since **C# 1.0**; compiler-generated accessors. |
| Custom `add`/`remove` accessors | Core since **C# 1.0**; explicit control. |
| `EventHandler<TEventArgs>` | Introduced in **.NET 2.0**; recommended for custom data. |
| Weak event pattern | WPF-specific (`WeakEventManager`); version/framework-specific. |
| `WeakEventManager<TEventSource,TEventArgs>` | Introduced in **.NET 4.5**; type-safe weak events. |
| `?.Invoke` for events | Introduced in **C# 6.0**; null-safe raising. |
| Subscribing in constructors | **Discouraged**; can cause premature GC. |
| Public delegate fields | **Discouraged**; use `event` for encapsulation. |

---

## References (Consolidated)

- Events — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/
- Examine events in C# — Training - https://learn.microsoft.com/en-us/training/modules/create-manage-events/2-examine-events-csharp
- Introduction to events — C# - https://learn.microsoft.com/en-us/dotnet/csharp/events-overview
- The `event` keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/event
- Events — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-events
- Event accessors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1583-event-accessors
- Chapter 4. Advanced C# — Event Accessors - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2008/ff652506(v=orm.10)
- How to subscribe to and unsubscribe from events — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/how-to-subscribe-to-and-unsubscribe-from-events
- Standard .NET event patterns — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/event-pattern
- EventHandler Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler
- EventHandler<TEventArgs> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler-1
- WeakEventManager Class (System.Windows) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.windows.weakeventmanager
- WeakEventManager<TEventSource,TEventArgs> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.windows.weakeventmanager-2
- Weak Event Patterns — WPF .NET Framework - https://learn.microsoft.com/en-us/dotnet/framework/wpf/advanced/weak-event-patterns
- C# 弱事件模式 — GitHub (cool-coding) - https://github.com/echo-cool-coding/cool-coding/blob/main/docs/language/csharp/7-c-attributes-and-events/13-c-weak-event-pattern.mdx
- Event Best Practices — https://learncsharpmastery.com/event-best-practices (search result)
- Framework Design Guidelines — Events - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/event

---