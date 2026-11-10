# SpinnerModel — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `SpinnerModel` is the **data model interface** behind every `JSpinner`. It defines a potentially unbounded, ordered sequence of object values and tracks the **current element** of that sequence. The visible `JSpinner` is only a view; the model holds the sequence, the current value, and the logic for computing the next and previous values.

**Technical Definition.** `public interface SpinnerModel`. It declares six methods: `getValue()`, `setValue(Object)`, `getNextValue()`, `getPreviousValue()`, `addChangeListener(ChangeListener)`, and `removeChangeListener(ChangeListener)`. The interface describes three properties: **value** (the current element, read/write), **nextValue** (the following element, or `null` if the sequence ends), and **previousValue** (the preceding element, or `null` if the sequence begins). A `SpinnerModel` does **not** support indexed random access to its elements; only the current, next, and previous values are accessible at any time. When the `value` property changes, registered `ChangeListener`s must be notified. The standard abstract base class is `AbstractSpinnerModel`, and Swing provides three concrete implementations: `SpinnerNumberModel`, `SpinnerListModel`, and `SpinnerDateModel`. All model mutations should occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** A `SpinnerModel` is the **“sequence brain”** of a `JSpinner`. The spinner itself is just a small text field with up and down arrows. The model is the object that knows **what values are allowed**, **which value is currently selected**, and **what the next and previous values should be** when the user clicks an arrow or presses a key. Think of the model as a **deck of cards** arranged in a fixed order: the spinner shows the top card, the up arrow flips to the next card, and the down arrow flips back to the previous card. The model decides whether the deck is a sequence of numbers (`SpinnerNumberModel`), a fixed list of labels (`SpinnerListModel`), or a sequence of dates (`SpinnerDateModel`). Because the sequence logic lives entirely in the model, you can swap a numeric spinner for a date spinner without changing the spinner component itself — you just give it a different model.

---

### Key Characteristics

1. **Unbounded sequence.** Unlike `ListModel`, a `SpinnerModel`'s sequence may be infinite (e.g., all integers) or bounded (e.g., 0–100).
2. **No random access.** Only three values are accessible at any time: the current value, the next value, and the previous value.
3. **Change-event driven.** The model fires `ChangeEvent`s when its value changes; the spinner and any application listeners observe these events.
4. **Three built-in models.** `SpinnerNumberModel` handles numeric sequences with optional minimum, maximum, and step size. `SpinnerListModel` handles fixed arrays or `List` sequences. `SpinnerDateModel` handles `Date` sequences with optional start/end bounds and a `Calendar` field for stepping.
5. **Editable and non-editable.** The spinner's editor (`JFormattedTextField`) allows the user to type a value; the model validates it via `setValue` and may throw `IllegalArgumentException` if the value is invalid.
6. **EDT discipline.** Model mutations and listener interactions should occur on the EDT to avoid race conditions and display corruption.
7. **Custom sequences.** Applications can implement `SpinnerModel` directly or extend `AbstractSpinnerModel` for domain-specific sequences (e.g., a list of domain objects, a custom range).

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, and lambda expressions.
- Familiarity with `JSpinner` and basic Swing containers.
- Understanding of `ChangeListener` and `ChangeEvent` from `javax.swing.event`.
- Knowledge of `Calendar`, `Date`, and numeric wrapper types (`Integer`, `Double`, etc.).
- Understanding of the Event Dispatch Thread (EDT) and `SwingUtilities.invokeLater`.
- A JDK (version 8 or later; all spinner models are in `javax.swing`).

---

### Related Programming Areas

- **`JSpinner`:** The component that consumes `SpinnerModel`.
- **`AbstractSpinnerModel`:** The abstract base class that implements the `ChangeListener` plumbing.
- **`ChangeListener` and `ChangeEvent`:** The event contract between the model and the spinner.
- **`JFormattedTextField` and `JSpinner.DefaultEditor`:** The editor that displays and validates the model's current value.
- **`SpinnerNumberModel`:** Numeric sequences with bounds and step size.
- **`SpinnerListModel`:** Fixed sequences defined by an array or `List`.
- **`SpinnerDateModel`:** Date sequences with start/end bounds and a `Calendar` field.
- **`SwingUtilities.invokeLater`:** Ensures model mutations occur on the EDT.

---

## Core Concepts and Features

---

## 1. SpinnerModel — The Interface

### Definitions

**Core Definition.** `SpinnerModel` is the contract that any data source must fulfill to back a `JSpinner`. It defines the current value, the next value, the previous value, and the change-listener registration methods.

**Technical Definition.** `public interface SpinnerModel`. The interface declares:

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `getValue()` | `Object` | Returns the current element of the sequence. |
| `setValue(Object value)` | `void` | Sets the current element; throws `IllegalArgumentException` if the value is not supported. |
| `getNextValue()` | `Object` | Returns the element after the current one, or `null` if at the end. |
| `getPreviousValue()` | `Object` | Returns the element before the current one, or `null` if at the beginning. |
| `addChangeListener(ChangeListener l)` | `void` | Registers a listener for value changes. |
| `removeChangeListener(ChangeListener l)` | `void` | Unregisters the listener. |

The model has three properties: **value** (read/write), **nextValue** (read-only, `null` at the end), and **previousValue** (read-only, `null` at the beginning).

**Beginner-Friendly Explanation.** `SpinnerModel` is the “rulebook” for a spinner: it knows the current value, what comes next, what came before, and how to notify listeners when the value changes.

---

### Purposes

- To separate the spinner's data and sequence logic from its visual presentation.
- To enable arbitrary sequences — numeric ranges, fixed lists, dates, or domain-specific values — to drive a single spinner component.
- To provide a uniform change-notification contract that the spinner and application listeners can observe.
- To support bounded and unbounded sequences through `null` next/previous values.
- To enable custom models for specialised domains.

---

### Syntax Rules and Structure

#### Complete General Syntax (Implementing a Custom SpinnerModel)

```java
public class MySpinnerModel implements SpinnerModel {
    private Object value;
    private final List<ChangeListener> listeners = new ArrayList<>();

    public Object getValue() { return value; }

    public void setValue(Object v) {
        if (!isValid(v)) throw new IllegalArgumentException("Invalid value");
        Object old = value;
        value = v;
        fireStateChanged();
    }

    public Object getNextValue() { /* compute next or return null */ }
    public Object getPreviousValue() { /* compute previous or return null */ }

    public void addChangeListener(ChangeListener l) { listeners.add(l); }
    public void removeChangeListener(ChangeListener l) { listeners.remove(l); }

    protected void fireStateChanged() {
        ChangeEvent e = new ChangeEvent(this);
        for (ChangeListener l : listeners) l.stateChanged(e);
    }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getValue` | The current element displayed by the spinner. |
| `setValue` | Validates and sets the current element; fires a `ChangeEvent`. |
| `getNextValue` | The next element in the sequence, or `null` at the end. |
| `getPreviousValue` | The previous element, or `null` at the beginning. |
| `addChangeListener` / `removeChangeListener` | Manage the listener list. |

#### Syntax Rules

- `setValue` must validate the value and throw `IllegalArgumentException` for unsupported values.
- `getNextValue` and `getPreviousValue` must **not** change the current value; they only compute and return the neighbouring values.
- The model must fire a `ChangeEvent` whenever its `value` property changes.
- All model access should occur on the EDT.

#### Constraints and Limitations

- `SpinnerModel` does not support indexed access to arbitrary sequence elements; only current, next, and previous are accessible.
- Custom models must handle `null` returns from `getNextValue` / `getPreviousValue` gracefully.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom SpinnerModel for a Domain-Specific Sequence

```java
import javax.swing.*;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;
import java.util.ArrayList;
import java.util.List;

public class PriorityModel implements SpinnerModel {
    private static final String[] VALUES = {"Low", "Medium", "High", "Critical"}; // 1. Sequence
    private int index = 0;                                                         // 2. Current
    private final List<ChangeListener> listeners = new ArrayList<>();

    @Override
    public Object getValue() { return VALUES[index]; }                             // 3. Get

    @Override
    public void setValue(Object value) {                                           // 4. Set
        for (int i = 0; i < VALUES.length; i++) {
            if (VALUES[i].equals(value)) {
                index = i;
                fireStateChanged();
                return;
            }
        }
        throw new IllegalArgumentException("Invalid priority: " + value);
    }

    @Override
    public Object getNextValue() {                                                 // 5. Next
        return index < VALUES.length - 1 ? VALUES[index + 1] : null;
    }

    @Override
    public Object getPreviousValue() {                                             // 6. Previous
        return index > 0 ? VALUES[index - 1] : null;
    }

    @Override
    public void addChangeListener(ChangeListener l) { listeners.add(l); }
    @Override
    public void removeChangeListener(ChangeListener l) { listeners.remove(l); }

    private void fireStateChanged() {
        ChangeEvent e = new ChangeEvent(this);
        for (ChangeListener l : listeners) l.stateChanged(e);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Priority Spinner");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JSpinner spinner = new JSpinner(new PriorityModel());
            spinner.addChangeListener(e ->
                System.out.println("Priority: " + spinner.getValue()));
            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner shows “Low” initially. Clicking the up arrow cycles through Medium, High, and Critical; clicking down cycles back. Each change prints the current priority.

**Why the code produces this result.** The custom model implements the six `SpinnerModel` methods, validates values in `setValue`, and fires `ChangeEvent`s. The `JSpinner` uses the model's `getValue`, `getNextValue`, and `getPreviousValue` to drive its arrows.

---

### Real-World Cases

- **Domain-object spinners:** A spinner that cycles through `enum` values or domain objects.
- **Custom ranges:** A spinner with a non-linear sequence (e.g., logarithmic steps).
- **Virtual sequences:** A spinner whose values are computed on demand from a database or service.

---

## 2. SpinnerNumberModel

### Definitions

**Core Definition.** `SpinnerNumberModel` is the built-in `SpinnerModel` implementation for sequences of numbers, with optional minimum and maximum bounds and a configurable step size.

**Technical Definition.** `public class SpinnerNumberModel extends AbstractSpinnerModel implements Serializable`. The sequence is bounded by `minimum` and `maximum` (both may be `null` for unbounded ranges), and the step between elements is defined by `stepSize`. All properties are defined in terms of `Number` and `Comparable`, but the implementation internally supports `Double`, `Float`, `Long`, `Integer`, `Short`, and `Byte`. The model fires `ChangeEvent`s when `value`, `stepSize`, `minimum`, or `maximum` changes.

**Beginner-Friendly Explanation.** `SpinnerNumberModel` is the “number spinner” brain. You tell it the starting value, the minimum, the maximum, and the step size, and it produces a numeric sequence. For example, `new SpinnerNumberModel(50, 0, 100, 1)` creates a spinner from 0 to 100 that starts at 50 and increments by 1.

---

### Purposes

- To provide a ready-made numeric spinner without writing custom model code.
- To enforce numeric bounds (minimum and maximum) and step sizes.
- To support integer, floating-point, and arbitrary numeric sequences.
- To serve as the default model for most quantity, percentage, and numeric input fields.
- To allow unbounded numeric sequences by passing `null` for minimum and maximum.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SpinnerNumberModel model = new SpinnerNumberModel(
    50,    // initial value
    0,     // minimum (null for none)
    100,   // maximum (null for none)
    1      // step size
);
JSpinner spinner = new JSpinner(model);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `SpinnerNumberModel(Number value, Comparable<?> minimum, Comparable<?> maximum, Number stepSize)` | Full control over initial value, bounds, and step. |
| `SpinnerNumberModel(int value, int min, int max, int step)` | Convenience overload for integers. |
| `SpinnerNumberModel(double value, double min, double max, double step)` | Convenience overload for doubles. |
| `SpinnerNumberModel()` | Default: value 0, no bounds, step 1. |

#### Key Methods

| Method | Purpose |
|--------|---------|
| `getNumber()` | Returns the current value as a `Number`. |
| `setValue(Object)` | Sets the current value; throws `IllegalArgumentException` if out of bounds. |
| `getNextValue()` | Returns the next value by adding `stepSize`. |
| `getPreviousValue()` | Returns the previous value by subtracting `stepSize`. |
| `getMinimum()` / `setMinimum(Comparable)` | Get/set the lower bound. |
| `getMaximum()` / `setMaximum(Comparable)` | Get/set the upper bound. |
| `getStepSize()` / `setStepSize(Number)` | Get/set the step size. |

#### Syntax Rules

- The initial value must be within `[minimum, maximum]` if bounds are specified; otherwise `IllegalArgumentException` is thrown.
- `minimum`, `maximum`, and `value` must be mutually `Comparable`.
- If `minimum > maximum`, `IllegalArgumentException` is thrown.
- `stepSize` must be positive.

#### Constraints and Limitations

- The model internally supports only the primitive wrapper types (`Integer`, `Long`, `Float`, `Double`, `Short`, `Byte`); arbitrary `Number` subclasses may not work correctly.
- The step size is not automatically aligned to the bounds; the user can select values that are not multiples of the step from the lower bound.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Integer Spinner with Bounds

```java
import javax.swing.*;
import java.awt.*;

public class NumberModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SpinnerNumberModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            SpinnerNumberModel model = new SpinnerNumberModel(               // 1. Model
                50, 0, 100, 5);                                              //    50, 0–100, step 5
            JSpinner spinner = new JSpinner(model);                          // 2. Spinner

            spinner.addChangeListener(e -> {                                 // 3. Listen
                Number n = (Number) spinner.getValue();
                System.out.println("Value: " + n.intValue());
            });

            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner shows 50. Clicking up adds 5 (55, 60, …); clicking down subtracts 5. Values are constrained between 0 and 100.

**Why the code produces this result.** `SpinnerNumberModel` defines the range and step. The `JSpinner` uses `getNextValue` and `getPreviousValue` to compute the steps, and the model validates bounds in `setValue`.

---

#### Example 2: Double Spinner with Unbounded Range

```java
SpinnerNumberModel model = new SpinnerNumberModel(0.0, null, null, 0.1);
JSpinner spinner = new JSpinner(model);
```

**Expected Output.** An unbounded double spinner starting at 0.0 and incrementing by 0.1.

**Why the code produces this result.** Passing `null` for minimum and maximum removes the bounds; the model uses the `double` overload to support floating-point steps.

---

#### Example 3: Dynamic Bound Updates

```java
SpinnerNumberModel model = new SpinnerNumberModel(10, 0, 100, 1);
// Later, in response to user input:
model.setMaximum(500);   // Expands the range
model.setStepSize(5);    // Changes the step
```

**Expected Output.** The spinner's range expands to 500 and steps by 5; the `ChangeListener` fires for the `maximum` and `stepSize` changes.

---

### Real-World Cases

- **Quantity selectors:** 1–99 items, step 1.
- **Percentage inputs:** 0–100%, step 5.
- **Price fields:** 0.00–9999.99, step 0.01.
- **Port numbers:** 1024–65535, step 1.

---

## 3. SpinnerListModel

### Definitions

**Core Definition.** `SpinnerListModel` is a `SpinnerModel` implementation whose sequence of values is defined by a fixed array or `List`.

**Technical Definition.** `public class SpinnerListModel extends AbstractSpinnerModel implements Serializable`. It stores a reference to the supplied array or `List`; the initial value is `values.get(0)`, and `getNextValue` / `getPreviousValue` return the neighbouring elements or `null` at the ends. The class fires `ChangeEvent`s when the `value` or `list` property changes. If the underlying list is modified directly (outside the model), the application is responsible for calling `fireStateChanged()`.

**Beginner-Friendly Explanation.** `SpinnerListModel` is the “list spinner” brain. You give it a fixed sequence of items — strings, objects, whatever — and the spinner cycles through them. For example, `new SpinnerListModel(new String[]{"Small", "Medium", "Large"})` creates a size spinner.

---

### Purposes

- To provide a spinner that cycles through a fixed, finite list of values.
- To support arbitrary object sequences, not just numbers or dates.
- To enforce that only values from the list are selectable.
- To serve as the model for enumerated option spinners (sizes, priorities, days, etc.).
- To allow the list to be replaced at runtime via `setList`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SpinnerListModel model = new SpinnerListModel(
    new String[]{"Small", "Medium", "Large"}
);
JSpinner spinner = new JSpinner(model);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `SpinnerListModel(Object[] values)` | Creates a model from an array. |
| `SpinnerListModel(List<?> values)` | Creates a model from a `List`. |
| `SpinnerListModel()` | Creates an effectively empty model (value `null`). |

#### Key Methods

| Method | Purpose |
|--------|---------|
| `getList()` / `setList(List<?>)` | Get or replace the underlying list. |
| `getValue()` | Returns the current list element. |
| `setValue(Object)` | Sets the current element; throws `IllegalArgumentException` if not in the list. |
| `getNextValue()` | Returns the next list element, or `null` at the end. |
| `getPreviousValue()` | Returns the previous list element, or `null` at the beginning. |

#### Syntax Rules

- The list must not be `null` or empty; otherwise `IllegalArgumentException` is thrown.
- The initial value is the first element of the list.
- `setValue` only accepts values that are present in the list.
- `setList` resets the model's index to zero.

#### Constraints and Limitations

- The model stores a reference to the list; external modifications require a manual `fireStateChanged()`.
- The sequence is finite; `getNextValue` returns `null` at the last element.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Size Spinner

```java
import javax.swing.*;
import java.awt.*;

public class ListModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SpinnerListModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            SpinnerListModel model = new SpinnerListModel(                   // 1. List
                new String[]{"Small", "Medium", "Large", "Extra Large"});
            JSpinner spinner = new JSpinner(model);                           // 2. Spinner

            spinner.addChangeListener(e ->
                System.out.println("Size: " + spinner.getValue()));          // 3. Listen

            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner shows “Small.” Clicking up cycles through Medium, Large, Extra Large; clicking down cycles back.

**Why the code produces this result.** `SpinnerListModel` uses the array as the sequence. `getNextValue` and `getPreviousValue` move through the array, returning `null` at the ends.

---

#### Example 2: Replacing the List at Runtime

```java
SpinnerListModel model = new SpinnerListModel(
    new String[]{"Red", "Green", "Blue"});
// Later:
model.setList(Arrays.asList("Cyan", "Magenta", "Yellow"));
```

**Expected Output.** The spinner's sequence changes to the new list, and the current value resets to “Cyan.”

---

### Real-World Cases

- **Size selectors:** Small, Medium, Large.
- **Priority selectors:** Low, Medium, High, Critical.
- **Day-of-week selectors:** Monday through Sunday.
- **Category selectors:** Predefined categories in a form.

---

## 4. SpinnerDateModel

### Definitions

**Core Definition.** `SpinnerDateModel` is a `SpinnerModel` implementation for sequences of `Date` values, with optional start and end bounds and a `Calendar` field that determines the step size.

**Technical Definition.** `public class SpinnerDateModel extends AbstractSpinnerModel implements Serializable`. The sequence is bounded by `start` and `end` (both may be `null` for unbounded ranges). The `calendarField` property (a `Calendar` constant such as `Calendar.DAY_OF_MONTH`, `Calendar.HOUR`, etc.) determines how much the date changes with each step. For example, if `calendarField` is `Calendar.DAY_OF_WEEK`, `getNextValue` returns a `Date` 24 hours after the current value. The model fires `ChangeEvent`s when `value`, `calendarField`, `start`, or `end` changes.

**Beginner-Friendly Explanation.** `SpinnerDateModel` is the “date spinner” brain. You can give it a start date, an end date, and a unit (days, hours, minutes, etc.), and the spinner steps through dates by that unit.

---

### Purposes

- To provide a spinner for selecting dates and times.
- To enforce date bounds (start and end).
- To support stepping by day, hour, minute, month, year, or any other `Calendar` field.
- To serve as the model for appointment, deadline, and scheduling spinners.
- To allow unbounded date sequences by passing `null` for start and end.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SpinnerDateModel model = new SpinnerDateModel(
    new Date(),          // initial value
    startDate,           // earliest date (null for none)
    endDate,             // latest date (null for none)
    Calendar.DAY_OF_MONTH // step unit
);
JSpinner spinner = new JSpinner(model);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `SpinnerDateModel(Date value, Comparable start, Comparable end, int calendarField)` | Full control over initial date, bounds, and step unit. |
| `SpinnerDateModel()` | Current date, no bounds, step by `Calendar.DAY_OF_MONTH`. |

#### Key Methods

| Method | Purpose |
|--------|---------|
| `getDate()` | Returns the current `Date`. |
| `setValue(Object)` | Sets the current `Date`. |
| `getNextValue()` | Returns the next date, or `null` if after `end`. |
| `getPreviousValue()` | Returns the previous date, or `null` if before `start`. |
| `getStart()` / `setStart(Comparable)` | Get or set the earliest date. |
| `getEnd()` / `setEnd(Comparable)` | Get or set the latest date. |
| `getCalendarField()` / `setCalendarField(int)` | Get or set the step unit. |

#### Syntax Rules

- The `calendarField` must be one of the `Calendar` constants that specify a field within a `Calendar`.
- `start` and `end` may be `null` for unbounded sequences.
- The initial value must be within `[start, end]` if bounds are specified.

#### Constraints and Limitations

- The model uses `Date` and `Calendar`, which are legacy APIs; applications needing modern date handling may need to convert.
- The step size is determined by `calendarField`; there is no separate numeric step.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Date Spinner with Bounds

```java
import javax.swing.*;
import java.awt.*;
import java.util.Calendar;
import java.util.Date;

public class DateModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SpinnerDateModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            Calendar cal = Calendar.getInstance();                           // 1. Bounds
            Date start = cal.getTime();
            cal.add(Calendar.YEAR, 1);
            Date end = cal.getTime();

            SpinnerDateModel model = new SpinnerDateModel(                   // 2. Model
                new Date(), start, end, Calendar.DAY_OF_MONTH);
            JSpinner spinner = new JSpinner(model);                           // 3. Spinner

            spinner.addChangeListener(e ->
                System.out.println("Date: " + spinner.getValue()));          // 4. Listen

            frame.add(spinner);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A date spinner starting at today's date, constrained to one year from today, stepping by day.

**Why the code produces this result.** `SpinnerDateModel` uses `calendarField = Calendar.DAY_OF_MONTH` to step by one day. The start and end bounds limit the range.

---

#### Example 2: Hour/Minute Spinner

```java
SpinnerDateModel model = new SpinnerDateModel(
    new Date(), null, null, Calendar.MINUTE);
JSpinner spinner = new JSpinner(model);
spinner.setEditor(new JSpinner.DateEditor(spinner, "HH:mm"));
```

**Expected Output.** A spinner showing the current time in `HH:mm` format, stepping by one minute.

---

### Real-World Cases

- **Appointment schedulers:** Date and time selection with bounded ranges.
- **Deadline pickers:** Date selection within a project timeline.
- **Report date filters:** Selecting a date range for data retrieval.

---

## 5. Spinner Model Best Practices

### Definitions

**Core Definition.** Spinner model best practices are the conventions and patterns that ensure spinner data handling is correct, thread-safe, and maintainable.

**Technical Definition.** Best practices include: using the built-in models (`SpinnerNumberModel`, `SpinnerListModel`, `SpinnerDateModel`) for common cases; extending `AbstractSpinnerModel` for custom sequences; mutating models on the EDT; using `ChangeListener` for value changes; formatting date spinners with `JSpinner.DateEditor`; and validating values in `setValue`.

**Beginner-Friendly Explanation.** Best practices for spinner models are the “rules of thumb” that prevent frozen GUIs, invalid values, and unmaintainable code. They include things like “use the right built-in model,” “mutate on the EDT,” and “format dates properly.”

---

### Purposes

- To choose the correct built-in model for the data type.
- To keep the GUI responsive by respecting the EDT.
- To enforce valid values through model validation.
- To format date and number output appropriately.
- To enable custom sequences when the built-in models are insufficient.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    SpinnerNumberModel model = new SpinnerNumberModel(                // 1. Built-in
        10, 0, 100, 5);
    JSpinner spinner = new JSpinner(model);                           // 2. Spinner
    spinner.addChangeListener(e -> {                                  // 3. Listener
        Number n = (Number) spinner.getValue();
        // react to n
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use built-in models | `SpinnerNumberModel`, `SpinnerListModel`, `SpinnerDateModel` |
| Extend `AbstractSpinnerModel` for custom | `class MyModel extends AbstractSpinnerModel` |
| EDT | `SwingUtilities.invokeLater(...)` for construction and mutation |
| Validate in `setValue` | Throw `IllegalArgumentException` for invalid values |
| Format dates | `new JSpinner.DateEditor(spinner, "yyyy-MM-dd")` |
| Use `ChangeListener` | Observe value changes via the model |

#### Syntax Rules

- **Use `SpinnerNumberModel` for numeric sequences**, `SpinnerListModel` for fixed lists, and `SpinnerDateModel` for dates.
- **Extend `AbstractSpinnerModel` for custom sequences**; it handles `ChangeListener` plumbing.
- **Mutate models on the EDT** to avoid race conditions.
- **Always validate values in `setValue`** and throw `IllegalArgumentException` for invalid input.
- **Format date spinners** with a `JSpinner.DateEditor` to produce readable output.

#### Constraints and Limitations

- `SpinnerNumberModel` internally supports only standard numeric wrapper types.
- `SpinnerListModel` stores a reference to the list; external changes require `fireStateChanged()`.
- `SpinnerDateModel` uses legacy `Date`/`Calendar` APIs.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Number Spinner with Validation

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeSpinnerModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeSpinnerModel::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice SpinnerModel");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        SpinnerNumberModel model = new SpinnerNumberModel(                 // 1. Model
            10, 0, 100, 5);
        JSpinner spinner = new JSpinner(model);                            // 2. Spinner
        spinner.setToolTipText("Quantity (0–100, step 5)");

        JLabel status = new JLabel("Quantity: 10");

        spinner.addChangeListener(e -> {                                   // 3. Listener
            Number n = (Number) spinner.getValue();
            status.setText("Quantity: " + n.intValue());
        });

        JPanel panel = new JPanel(new BorderLayout(5, 5));
        panel.add(spinner, BorderLayout.CENTER);
        panel.add(status, BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(300, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A spinner with a tooltip, a label showing the current quantity, and a constrained range 0–100 with step 5.

**Why the code produces this result.** The built-in `SpinnerNumberModel` handles the numeric sequence and bounds. The `ChangeListener` reads the current value and updates the status label. The tooltip describes the valid range.

---

#### Example 2: Common Mistake — Using `SpinnerNumberModel` for Non-Numeric Data

```java
import javax.swing.*;

public class BadModelChoice {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Model Choice");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            // WRONG: using a number model for string values
            SpinnerNumberModel model = new SpinnerNumberModel(0, 0, 10, 1); // 1. Wrong
            model.setValue("High");                                          // 2. Exception
            JSpinner spinner = new JSpinner(model);

            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** `setValue("High")` throws `IllegalArgumentException` because `SpinnerNumberModel` only accepts numeric values.

**Why this is wrong.** `SpinnerNumberModel` is designed for numbers. For string or object sequences, use `SpinnerListModel` or a custom model. The correct approach is to match the model type to the data type.

---

#### Example 3: Common Mistake — Mutating the Model Off the EDT

```java
import javax.swing.*;

public class BadEdtModelMutation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad EDT Mutation");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            SpinnerNumberModel model = new SpinnerNumberModel(0, 0, 100, 1);
            JSpinner spinner = new JSpinner(model);

            new Thread(() -> {                                            // WRONG
                model.setValue(50);                                       // Off-EDT
            }).start();

            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner may update unpredictably, or the model may throw a `ConcurrentModificationException` or produce a corrupted display.

**Why this is wrong.** Swing models are not thread-safe. Mutations must occur on the EDT. The correct approach is to wrap the mutation in `SwingUtilities.invokeLater`.

---

### Real-World Cases

- **Production Swing applications** use built-in spinner models for quantities, dates, and enumerated options, always mutating on the EDT.
- **Custom domain models** extend `AbstractSpinnerModel` for specialised sequences (e.g., priority levels, custom date ranges).
- **Date spinners** use `JSpinner.DateEditor` with a specific format string for locale-appropriate display.

---

## References

Oracle. *Interface SpinnerModel*. Java Platform, Standard Edition 6 API Specification. https://people.csail.mit.edu/dfhuynh/research/javadoc/jdk1.5.0/javax/swing/SpinnerModel.html

Oracle. *How to Use Spinners*. The Java Tutorials. http://discipline.elcom.pub.ro/swrtc/tutorial/uiswing/components/spinner.html

Cleverence. *How to Use Spinners in Java Swing: Complete JSpinner Guide with Examples*. https://www.cleverence.com/articles/oracle-documentation/how-to-use-spinners-in-java-swing-jspinner-tutorial-5829/

Oracle. *Class SpinnerNumberModel*. Java Platform, Standard Edition 21 API Specification (OpenJDK diff). https://cr.openjdk.org/~serb/8175266/specdiff/javax/swing/SpinnerNumberModel-report.html

Oracle. *Class SpinnerListModel*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/SpinnerListModel.html

Oracle. *Class SpinnerDateModel*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/SpinnerDateModel.html

Oracle. *Class AbstractSpinnerModel*. Java Platform, Standard Edition 15 API Specification. https://docs.oracle.com/javase/jp/15/docs/api/java.desktop/javax/swing/class-use/AbstractSpinnerModel.html

Oracle. *Interface ChangeListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/ChangeListener.html

Oracle. *Class JSpinner*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JSpinner.html

Oracle. *Class Calendar*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Stack Overflow. *JSpinner with a SpinnerNumberModel always returns value 0*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/