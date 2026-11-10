# BoundedRangeModel & DefaultBoundedRangeModel — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `BoundedRangeModel` is the **data model interface** behind `JSlider`, `JProgressBar`, and `JScrollBar`. It defines a bounded integer range with four interrelated properties—**minimum**, **maximum**, **value**, and **extent**—and guarantees that these four values always satisfy a precise mathematical invariant. `DefaultBoundedRangeModel` is the standard, concrete implementation of this interface, providing ready-made storage, validation, and `ChangeEvent` notification.

**Technical Definition.** `public interface BoundedRangeModel`. It declares four interrelated integer properties (`minimum`, `maximum`, `extent`, `value`) that define two nested ranges: the **outer range** is `[minimum, maximum]`, and the **inner range** is `[value, value + extent]`. The inner range must always lie within the outer range, yielding the invariant: `minimum ≤ value ≤ value + extent ≤ maximum`. The interface declares getters and setters for all four properties, a combined `setRangeProperties(int value, int extent, int min, int max, boolean adjusting)` method, change-listener registration (`addChangeListener` / `removeChangeListener`), and the `getValueIsAdjusting` / `setValueIsAdjusting` methods. `public class DefaultBoundedRangeModel extends Object implements BoundedRangeModel, Serializable` is "a generic implementation of `BoundedRangeModel`" with default values `value = 0`, `extent = 0`, `minimum = 0`, `maximum = 100`, and `adjusting = false`. The model uses `Swing ChangeEvents` rather than `PropertyChangeEvents` to notify clients of changes, a deliberate choice to keep the overhead of monitoring low because "changes are often reported at MouseDragged rates". All model mutations should occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** A `BoundedRangeModel` is the **"range brain"** shared by sliders, progress bars, and scroll bars. Imagine a ruler with a **minimum** mark and a **maximum** mark. Now imagine a **window** (the **extent**) that slides along the ruler. The **value** is where the left edge of the window sits. The model's job is to keep the window from sliding off the ruler: the window's left edge cannot go below the minimum, and the window's right edge (value + extent) cannot go past the maximum. When you drag a slider knob or scroll a scroll bar, you are moving this window. The **extent** tells the model how big the window is—for a scroll bar, it represents the visible portion of the scrollable content; for a progress bar, the extent is typically zero. The model fires `ChangeEvent`s every time the window moves, and the `valueIsAdjusting` flag lets listeners know whether the user is still dragging or has settled on a final value. `DefaultBoundedRangeModel` is the ready-made implementation that handles all the boundary checking for you.

---

### Key Characteristics

1. **Four interrelated properties.** `minimum`, `maximum`, `value`, and `extent` are not independent; the model enforces `minimum ≤ value ≤ value + extent ≤ maximum` at all times.
2. **Automatic "correction".** The `setMinimum` and `setMaximum` methods **correct** the other three properties to accommodate the new boundary. The `setValue` and `setExtent` methods **correct** their arguments to fit within the current constraints.
3. **`valueIsAdjusting` flag.** This boolean indicates that a series of changes is underway (e.g., a slider drag). Listeners can check `getValueIsAdjusting()` to avoid expensive updates on every intermediate event and act only on the final value.
4. **`ChangeEvent` notification.** The model fires `ChangeEvent`s, not `PropertyChangeEvent`s, to minimize overhead during high-frequency updates.
5. **Shared by three components.** `JSlider`, `JProgressBar`, and `JScrollBar` all use `BoundedRangeModel` internally; a single model can be shared across multiple components.
6. **Serialization caveat.** As with all Swing models, serialized `DefaultBoundedRangeModel` objects are not compatible with future Swing releases; serialization is appropriate only for short-term storage or RMI between applications running the same Swing version.
7. **EDT discipline.** Model mutations should occur on the EDT to avoid race conditions and display corruption.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, and lambda expressions.
- Familiarity with `JSlider`, `JProgressBar`, or `JScrollBar`.
- Understanding of `ChangeListener` and `ChangeEvent` from `javax.swing.event`.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `java.beans.PropertyChangeListener` (for contrast with `ChangeListener`).
- A JDK (version 8 or later; both `BoundedRangeModel` and `DefaultBoundedRangeModel` are in `javax.swing`).

---

### Related Programming Areas

- **`JSlider`:** Uses `BoundedRangeModel` to track the slider's minimum, maximum, and current value.
- **`JProgressBar`:** Uses `BoundedRangeModel` to track progress; the model's `value` represents the current progress.
- **`JScrollBar`:** Uses `BoundedRangeModel`; the `value` is the scroll position, and the `extent` is the visible amount.
- **`ChangeListener` and `ChangeEvent`:** The event contract between the model and its clients.
- **`DefaultBoundedRangeModel`:** The standard implementation.
- **`SwingUtilities.invokeLater`:** Ensures model mutations occur on the EDT.

---

## Core Concepts and Features

---

## 1. BoundedRangeModel — The Interface

### Definitions

**Core Definition.** `BoundedRangeModel` is the interface that defines the bounded integer range used by sliders, progress bars, and scroll bars.

**Technical Definition.** `public interface BoundedRangeModel`. It declares getters (`getMinimum`, `getMaximum`, `getValue`, `getExtent`, `getValueIsAdjusting`), setters (`setMinimum`, `setMaximum`, `setValue`, `setExtent`, `setValueIsAdjusting`), a combined setter (`setRangeProperties`), and change-listener methods (`addChangeListener`, `removeChangeListener`). The four core properties obey the invariant `minimum ≤ value ≤ value + extent ≤ maximum`.

**Beginner-Friendly Explanation.** `BoundedRangeModel` is the contract that says: "I am a range with a minimum, a maximum, a current value, and a window size (extent), and I will always keep my window on the ruler."

---

### Purposes

- To provide a uniform data model for all bounded-range components (sliders, progress bars, scroll bars).
- To enforce the invariant `minimum ≤ value ≤ value + extent ≤ maximum` automatically.
- To allow multiple components to share the same range state.
- To support high-frequency change notification through `ChangeEvent`s.
- To enable listeners to distinguish intermediate changes (drag) from final values via `valueIsAdjusting`.

---

### Syntax Rules and Structure

#### Complete General Syntax (Using the Interface)

```java
BoundedRangeModel model = slider.getModel();
model.setMinimum(0);
model.setMaximum(100);
model.setValue(50);
model.setExtent(10);
boolean adjusting = model.getValueIsAdjusting();
model.addChangeListener(e -> System.out.println("Value: " + model.getValue()));
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getMinimum()` / `setMinimum(int)` | The lower bound of the outer range. |
| `getMaximum()` / `setMaximum(int)` | The upper bound of the outer range. |
| `getValue()` / `setValue(int)` | The origin of the inner range (e.g., slider position). |
| `getExtent()` / `setExtent(int)` | The length of the inner range (e.g., visible amount). |
| `getValueIsAdjusting()` / `setValueIsAdjusting(boolean)` | Whether a series of changes is underway. |
| `setRangeProperties(int value, int extent, int min, int max, boolean adjusting)` | Sets all four properties in one call. |
| `addChangeListener(ChangeListener)` / `removeChangeListener(ChangeListener)` | Manages change listeners. |

#### Syntax Rules

- The invariant `minimum ≤ value ≤ value + extent ≤ maximum` is always enforced.
- `setMinimum` and `setMaximum` may "correct" the other three properties.
- `setValue` and `setExtent` "correct" their arguments to fit within the current constraints.
- `setValueIsAdjusting(true)` should be called at the start of a drag and `false` at the end.

#### Constraints and Limitations

- The model does not support arbitrary indexed access; it only tracks the four integer properties.
- The `extent` may be zero (e.g., for a progress bar).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading and Setting Range Properties

```java
import javax.swing.*;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;

public class BasicBoundedRangeModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Basic BoundedRangeModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSlider slider = new JSlider();                              // 1. Slider
            BoundedRangeModel model = slider.getModel();                 // 2. Model
            model.setMinimum(0);                                         // 3. Min
            model.setMaximum(100);                                       // 4. Max
            model.setValue(50);                                          // 5. Value
            model.setExtent(10);                                         // 6. Extent

            model.addChangeListener(new ChangeListener() {               // 7. Listener
                @Override
                public void stateChanged(ChangeEvent e) {
                    System.out.println("value=" + model.getValue()
                        + ", extent=" + model.getExtent()
                        + ", adjusting=" + model.getValueIsAdjusting());
                }
            });

            frame.add(slider);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A slider from 0 to 100 with an initial value of 50. Dragging the slider prints the current value, extent, and adjusting flag on each change.

**Why the code produces this result.** The `JSlider` uses a `BoundedRangeModel` internally. `setValue` updates the model and fires a `ChangeEvent`. The `valueIsAdjusting` flag is `true` during the drag and `false` when the drag ends.

---

### Real-World Cases

- **Scroll bars:** The `value` is the scroll position and the `extent` is the visible amount.
- **Progress bars:** The `value` is the progress and the `extent` is typically zero.
- **Sliders:** The `value` is the slider position; the `extent` may be zero or non-zero depending on the slider's configuration.

---

## 2. DefaultBoundedRangeModel — The Implementation

### Definitions

**Core Definition.** `DefaultBoundedRangeModel` is the standard, concrete implementation of `BoundedRangeModel`, providing storage, boundary enforcement, and `ChangeEvent` notification.

**Technical Definition.** `public class DefaultBoundedRangeModel extends Object implements BoundedRangeModel, Serializable`. Constructors include `DefaultBoundedRangeModel()` (defaults: value = 0, extent = 0, minimum = 0, maximum = 100, adjusting = false) and `DefaultBoundedRangeModel(int value, int extent, int min, int max)`. The four-argument constructor throws `IllegalArgumentException` if `min ≤ value ≤ value + extent ≤ max` is not satisfied. The class uses a single `ChangeEvent` instance per model and an `EventListenerList` to store listeners.

**Beginner-Friendly Explanation.** `DefaultBoundedRangeModel` is the "ready-made range" that every standard slider, progress bar, and scroll bar uses. You can construct it with your desired range and extent, and it handles all the boundary checking for you.

---

### Purposes

- To provide a ready-to-use implementation of `BoundedRangeModel` without writing custom model code.
- To support all standard slider, progress bar, and scroll bar configurations.
- To serve as a base class for custom range models.
- To enable shared range state across multiple components.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultBoundedRangeModel model = new DefaultBoundedRangeModel(50, 10, 0, 100);
JSlider slider = new JSlider(model);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `DefaultBoundedRangeModel()` | Defaults: value 0, extent 0, minimum 0, maximum 100, adjusting false. |
| `DefaultBoundedRangeModel(int value, int extent, int min, int max)` | Initialises all four properties; throws `IllegalArgumentException` if constraints are not met. |

#### Syntax Rules

- The four-argument constructor validates `min ≤ value ≤ value + extent ≤ max` and throws `IllegalArgumentException` on violation.
- All mutating methods fire `ChangeEvent`s.
- The model uses a single `ChangeEvent` instance for efficiency.

#### Constraints and Limitations

- The model stores integers only; for floating-point ranges, scale the values.
- Serialization is not guaranteed across Swing versions.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Custom Range Model for a Slider

```java
import javax.swing.*;
import java.awt.*;

public class CustomBoundedRangeModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom BoundedRangeModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultBoundedRangeModel model = new DefaultBoundedRangeModel(   // 1. Model
                50, 10, 0, 200);                                            //    value, extent, min, max
            JSlider slider = new JSlider(model);                             // 2. Slider

            slider.addChangeListener(e ->
                System.out.println("Value: " + model.getValue()));           // 3. Listen

            frame.add(slider);
            frame.setSize(500, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A slider from 0 to 200 with an initial value of 50 and an extent of 10. Dragging the slider prints the current value.

**Why the code produces this result.** The `DefaultBoundedRangeModel` stores the range and value. The `JSlider` uses it as its data source, and the listener observes changes.

---

#### Example 2: Sharing a Model Between Two Components

```java
DefaultBoundedRangeModel shared = new DefaultBoundedRangeModel(0, 10, 0, 100);
JSlider slider = new JSlider(shared);
JProgressBar bar = new JProgressBar(shared);
```

**Expected Output.** Moving the slider updates the progress bar simultaneously, because both components share the same model.

---

### Real-World Cases

- **Custom sliders** with non-default ranges and extents.
- **Shared range state** between a slider and a progress bar (e.g., a media player's seek bar and progress indicator).
- **Base class for domain-specific models** that extend `DefaultBoundedRangeModel` to add constraints or side effects.

---

## 3. BoundedRangeModel Methods — Complete Reference

### Definitions

**Core Definition.** The `BoundedRangeModel` methods provide access to the four range properties, the adjusting flag, combined property setting, and change-listener registration.

**Technical Definition.** The table below summarises the key methods of the `BoundedRangeModel` interface and its `DefaultBoundedRangeModel` implementation.

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `getMinimum()` | `int` | Returns the lower bound of the outer range. |
| `setMinimum(int newMinimum)` | `void` | Sets the lower bound; may correct the other three properties. |
| `getMaximum()` | `int` | Returns the upper bound of the outer range. Note the upper limit on `value` is `maximum - extent`. |
| `setMaximum(int newMaximum)` | `void` | Sets the upper bound; may correct the other three properties. |
| `getValue()` | `int` | Returns the current value (the origin of the inner range). |
| `setValue(int newValue)` | `void` | Sets the current value, correcting it to fit the constraints if necessary. |
| `getExtent()` | `int` | Returns the length of the inner range. |
| `setExtent(int newExtent)` | `void` | Sets the extent, correcting it to fit the constraints if necessary. |
| `getValueIsAdjusting()` | `boolean` | Returns whether a series of changes is underway. |
| `setValueIsAdjusting(boolean b)` | `void` | Marks the start/end of a series of changes. |
| `setRangeProperties(int value, int extent, int min, int max, boolean adjusting)` | `void` | Sets all four properties in one call. |
| `addChangeListener(ChangeListener)` | `void` | Registers a change listener. |
| `removeChangeListener(ChangeListener)` | `void` | Unregisters a change listener. |

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setValue` | Corrects the value if it is outside `[minimum, maximum - extent]`. |
| `setExtent` | Corrects the extent if `value + extent > maximum`. |
| `setMinimum` / `setMaximum` | May correct `value` and `extent` to maintain the invariant. |
| `setValueIsAdjusting` | Tells listeners that a drag is in progress; they can skip expensive updates until `false`. |

#### Syntax Rules

- The invariant `minimum ≤ value ≤ value + extent ≤ maximum` is enforced at all times.
- The model fires `ChangeEvent`s, not `PropertyChangeEvent`s.
- `setRangeProperties` sets all four properties atomically, firing a single event if any value changes.

#### Constraints and Limitations

- The model does not support floating-point values; scale to integers.
- The `extent` may be zero (progress bar) or non-zero (scroll bar).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using `setRangeProperties` for Atomic Updates

```java
import javax.swing.*;
import javax.swing.event.ChangeEvent;

public class SetRangePropertiesDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setRangeProperties");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultBoundedRangeModel model = new DefaultBoundedRangeModel();
            JProgressBar bar = new JProgressBar(model);

            model.addChangeListener(e -> {
                System.out.println("Value: " + model.getValue()
                    + ", Extent: " + model.getExtent()
                    + ", Adjusting: " + model.getValueIsAdjusting());
            });

            model.setRangeProperties(25, 5, 0, 100, false);             // 1. Atomic

            frame.add(bar);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The progress bar shows 25 out of 100. The listener prints the new value, extent, and adjusting flag in a single event.

**Why the code produces this result.** `setRangeProperties` sets all four properties in one call, firing a single `ChangeEvent` if any value changes.

---

### Real-World Cases

- **Bulk updates:** Changing the range of a slider when the data model changes.
- **Initialization:** Setting up a model with a specific range and extent before attaching it to a component.

---

## 4. Bounded Range Model Best Practices

### Definitions

**Core Definition.** `BoundedRangeModel` best practices are the conventions and patterns that ensure range handling is correct, thread-safe, and performant.

**Technical Definition.** Best practices include: using the model's setters rather than manipulating the component directly; checking `getValueIsAdjusting()` to avoid unnecessary work during drags; mutating models on the EDT; sharing models across components when synchronization is desired; using `setRangeProperties` for atomic updates; and understanding the automatic correction behaviour of the setters.

**Beginner-Friendly Explanation.** Best practices for range models are the "rules of thumb" that prevent frozen GUIs, incorrect boundary behaviour, and unmaintainable code. They include things like "use the model, not the component," "check `valueIsAdjusting`," and "mutate on the EDT."

---

### Purposes

- To avoid expensive updates during high-frequency drags.
- To keep the GUI responsive by respecting the EDT.
- To ensure correct boundary behaviour by using the model's setters.
- To enable shared range state across components.
- To avoid race conditions and display corruption.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    DefaultBoundedRangeModel model = new DefaultBoundedRangeModel(       // 1. Model
        50, 10, 0, 100);
    JSlider slider = new JSlider(model);                                  // 2. Component

    model.addChangeListener(e -> {                                        // 3. Listener
        if (!model.getValueIsAdjusting()) {                              // 4. Adjusting
            // perform final update
        }
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use the model | `slider.getModel().setValue(...)` rather than `slider.setValue(...)` |
| Check `valueIsAdjusting` | `if (!model.getValueIsAdjusting()) { ... }` |
| Mutate on EDT | `SwingUtilities.invokeLater(...)` |
| Share models intentionally | `new JSlider(sharedModel)`, `new JProgressBar(sharedModel)` |
| Atomic updates | `model.setRangeProperties(...)` |
| Respect correction | Do not assume `setValue` sets the exact value; check `getValue` afterwards |

#### Syntax Rules

- **Always check `getValueIsAdjusting()`** before performing expensive work in a `ChangeListener`.
- **Always mutate models on the EDT** to avoid race conditions.
- **Use `setRangeProperties`** for atomic updates of multiple properties.
- **Understand that `setValue` and `setExtent` correct their arguments** to fit the constraints; do not assume the set value equals the requested value.
- **Share models intentionally**; sharing a model synchronizes all components that use it.

#### Constraints and Limitations

- The model uses `ChangeEvent`, not `PropertyChangeEvent`; do not attempt to use `PropertyChangeListener`.
- `setMinimum` and `setMaximum` may change `value` and `extent`; re-read them after setting bounds.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Progress Bar with Adjusting Guard

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeBoundedRangeModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeBoundedRangeModel::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice BoundedRangeModel");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        DefaultBoundedRangeModel model = new DefaultBoundedRangeModel(0, 0, 0, 100); // 1. Model
        JProgressBar bar = new JProgressBar(model);                               // 2. Bar
        JLabel status = new JLabel("Progress: 0%");

        model.addChangeListener(e -> {                                            // 3. Listener
            if (model.getValueIsAdjusting()) return;                             // 4. Guard
            status.setText("Progress: " + model.getValue() + "%");
        });

        // Simulate progress updates
        new Timer(100, ev -> {
            int v = model.getValue() + 5;
            if (v > 100) v = 100;
            model.setValue(v);
        }).start();

        JPanel panel = new JPanel(new BorderLayout());
        panel.add(bar, BorderLayout.CENTER);
        panel.add(status, BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(400, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** The progress bar fills from 0 to 100, and the status label updates only when the value is not adjusting (in this case, on each update since the timer does not set adjusting to true).

**Why the code produces this result.** The `DefaultBoundedRangeModel` enforces the range. The `getValueIsAdjusting()` guard ensures the status label is not updated during intermediate drag states.

---

#### Example 2: Common Mistake — Using `setValue` Without Checking Correction

```java
DefaultBoundedRangeModel model = new DefaultBoundedRangeModel(0, 10, 0, 100);
model.setValue(200);                                                       // WRONG expectation
System.out.println(model.getValue());                                      // prints 90, not 200
```

**Expected Output.** The value is set to 90, not 200, because `value + extent` must not exceed `maximum` (100). The model corrects the argument.

**Why this is wrong.** As the API documentation states, "if newValue is greater than value+extent then it's set to value+extent". The correct approach is to check `getValue()` after calling `setValue` if the exact value matters, or to adjust the range so the desired value is achievable.

---

#### Example 3: Common Mistake — Mutating the Model Off the EDT

```java
DefaultBoundedRangeModel model = new DefaultBoundedRangeModel();
new Thread(() -> model.setValue(50)).start();                              // WRONG
```

**Expected Output.** The model may update unpredictably, or the component using it may throw a `ConcurrentModificationException` or display corrupted.

**Why this is wrong.** Swing models are not thread-safe. Mutations must occur on the EDT. The correct approach is `SwingUtilities.invokeLater(() -> model.setValue(50))`.

---

### Real-World Cases

- **Production Swing applications** always check `getValueIsAdjusting()` in `ChangeListener`s for sliders and scroll bars to avoid expensive updates during drags.
- **Shared models** are used intentionally between a slider and a progress bar (e.g., a media player's seek bar and progress indicator).
- **Custom range models** extend `DefaultBoundedRangeModel` to add domain-specific constraints or side effects.

---

## References

Oracle. *Interface BoundedRangeModel*. Java Development Kit Version 23 API Specification. https://docs.oracle.com/en/java/javase/23/docs/api/java.desktop/javax/swing/BoundedRangeModel.html

Oracle. *Class DefaultBoundedRangeModel*. Java Platform, Standard Edition & Java Development Kit Version 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/DefaultBoundedRangeModel.html

Oracle. *Interface BoundedRangeModel*. Java Platform, Standard Edition 20 API Specification (OpenJDK diff). https://cr.openjdk.org/~iris/se/20/spec/latest/apidiffs/java.desktop/javax/swing/BoundedRangeModel-report.html

Oracle. *Interface BoundedRangeModel*. Java Platform, Standard Edition 12 API Specification (OpenJDK). https://cr.openjdk.org/

Oracle. *Interface BoundedRangeModel*. GNU Classpath API Documentation. https://developer.classpath.org/

Cleverence. *JSlider in Java Swing (Java SE 8) — Complete Guide with Examples*. https://www.cleverence.com/articles/oracle-documentation/jslider-java-se-8-complete-guide-4581/

Oracle. *A Swing Architecture Overview — Separable Model Architecture*. https://www.oracle.com/technetwork/java/architecture-142923.html#separable

Stack Overflow. *BoundedRangeModel vs PropertyChangeListener*. https://stackoverflow.com/

Oracle. *Class ChangeListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/ChangeListener.html

Oracle. *Class ChangeEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/ChangeEvent.html