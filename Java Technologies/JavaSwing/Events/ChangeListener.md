# ChangeListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `ChangeListener` is a functional interface in the `javax.swing.event` package for receiving **change events**—notifications that the state of a component or model has changed. It declares a single method, `stateChanged(ChangeEvent e)`, which is the callback invoked whenever the source's state changes. It is the conventional listener for `JSlider`, `JSpinner`, `JProgressBar`, `JColorChooser`, `JTabbedPane`, and the various `*Model` classes that back these components.

**Technical Definition.** `public interface ChangeListener extends EventListener`. The interface declares exactly one method: `void stateChanged(ChangeEvent e)`. Because it is a **functional interface**, it can be implemented by a lambda expression, a method reference, an anonymous inner class, or a named class. Unlike most Swing event listeners, `ChangeListener` is typically registered on the **model** of a component rather than on the component itself: `slider.getModel().addChangeListener(...)`, `spinner.getModel().addChangeListener(...)`, `progressBar.getModel().addChangeListener(...)`, `colorChooser.getSelectionModel().addChangeListener(...)`, and `tabbedPane.getModel().addChangeListener(...)`. The event object, `ChangeEvent`, extends `java.util.EventObject`—**not** `java.awt.AWTEvent`—so it does not have an event ID, and it does not support `consume()`. All change events are dispatched on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `ChangeListener` is the "the value or state just changed" listener. When a user drags a slider, clicks a spinner's up arrow, advances a progress bar, picks a different color, or selects a tab, the underlying model fires a `ChangeEvent` to every registered `ChangeListener`. The listener's `stateChanged` method runs, and you can read the new state from the source. Unlike `ActionListener`, which fires on **command activation** (like a button click), `ChangeListener` fires on **state mutation**—whenever a value changes, whether by user interaction or programmatically. Unlike `ItemListener`, which fires only on **selected/deselected** transitions for `ItemSelectable` components, `ChangeListener` works with a much wider range of models and does not restrict itself to a binary selection state. Think of `ChangeListener` as a state-change feed: it reports every mutation of a model, and you decide which mutations deserve a reaction.

---

### Key Characteristics

1. **Single-method functional interface.** `stateChanged` is the only method; lambdas and method references are idiomatic.
2. **Model-centric registration.** Most Swing components expose their state through a model (`BoundedRangeModel`, `SpinnerModel`, `ButtonModel`, `SingleSelectionModel`, `ColorSelectionModel`); the `ChangeListener` is registered on the model, not on the component.
3. **`ChangeEvent` extends `java.util.EventObject`.** Unlike AWT events, `ChangeEvent` is not an `AWTEvent`, so it has no ID and cannot be consumed. `getSource()` returns the model (or the component, depending on registration).
4. **Fires for programmatic and user-driven changes.** Calling `setValue`, `setSelectedIndex`, or `setColor` programmatically fires `ChangeListener` events just like user interaction does.
5. **No built-in "adjusting" flag.** Unlike `ListSelectionListener`, `ChangeListener` does not carry a `valueIsAdjusting` flag. For `JSlider` and `JProgressBar`, the *model* itself exposes `getValueIsAdjusting()`, which you check on the model.
6. **Fires frequently during drags.** Dragging a slider or spinner fires many `ChangeEvent`s. Heavy work in the handler should be debounced or offloaded.
7. **EDT dispatch.** All change events are delivered on the EDT; handlers must be fast and must not block.
8. **Wide applicability.** Used by sliders, spinners, progress bars, color choosers, tabbed panes, and any model implementing `ChangeEvent`-firing behaviour.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, lambda expressions, and method references.
- Familiarity with the model-based components: `JSlider`, `JSpinner`, `JProgressBar`, `JColorChooser`, `JTabbedPane`.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `EventObject` and the delegation event model.
- Knowledge of the underlying models: `BoundedRangeModel`, `SpinnerModel`, `SingleSelectionModel`, `ColorSelectionModel`, `ButtonModel`.
- A JDK (version 8 or later; `ChangeListener` and `ChangeEvent` are in `javax.swing.event`).

---

### Related Programming Areas

- **`ChangeEvent`:** The event object delivered to `ChangeListener`s; extends `java.util.EventObject`.
- **`BoundedRangeModel`:** The model behind `JSlider`, `JProgressBar`, and `JScrollBar`; fires `ChangeEvent`s on value changes and exposes `getValueIsAdjusting()`.
- **`SpinnerModel`:** The model behind `JSpinner`; fires `ChangeEvent`s on value changes.
- **`SingleSelectionModel`:** The model behind `JTabbedPane`; fires `ChangeEvent`s on tab selection.
- **`ColorSelectionModel`:** The model behind `JColorChooser`; fires `ChangeEvent`s on color changes.
- **`ButtonModel`:** The model behind `AbstractButton`; fires `ChangeEvent`s on state changes (in addition to `ActionEvent` and `ItemEvent`).
- **`DocumentListener`:** For text component content changes; `ChangeListener` is not used for text input.
- **`PropertyChangeListener`:** A more general alternative for observing property changes on beans.

---

## Core Concepts and Features

---

## 1. ChangeListener — The Interface

### Definitions

**Core Definition.** `ChangeListener` is a functional interface for receiving notifications when a model's state changes.

**Technical Definition.** `public interface ChangeListener extends EventListener`. It declares `void stateChanged(ChangeEvent e)`. The interface is implemented by classes interested in state changes on any object that fires `ChangeEvent`s. Registration is via `addChangeListener(ChangeListener)` on the model or, for some components, on the component itself (which delegates to the model). The method is invoked on the EDT.

**Beginner-Friendly Explanation.** `ChangeListener` is the "state changed" contract. Implement it (or write a lambda) and register it with a slider, spinner, progress bar, color chooser, tabbed pane, or any model that fires `ChangeEvent`s.

---

### Purposes

- To react when the value of a slider, spinner, or progress bar changes.
- To react when the selection state of a tabbed pane changes.
- To react when the selected color of a color chooser changes.
- To synchronise UI state with a model's current value.
- To implement live previews, dynamic validation, and dependent UI updates driven by state changes.

---

### Syntax Rules and Structure

#### Complete General Syntax (Registration and Handling)

```java
// Registering on a model (typical for sliders, spinners, progress bars):
slider.getModel().addChangeListener(new ChangeListener() {
    @Override
    public void stateChanged(ChangeEvent e) {
        // reaction code
    }
});

// Or with a lambda:
slider.getModel().addChangeListener(e -> {
    // reaction code
});

// Registering on the component (for JTabbedPane, JColorChooser etc.):
tabs.addChangeListener(e -> { /* tab changed */ });
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addChangeListener(ChangeListener)` | Registers the listener on the model or component. |
| `removeChangeListener(ChangeListener)` | Unregisters the listener. |
| `stateChanged(ChangeEvent e)` | The single method to implement; invoked on state change. |
| `e.getSource()` | The source of the event (typically the model). |

#### Syntax Rules

- Only one method is required for the listener implementation.
- The listener must be registered before the state change occurs.
- All listener code runs on the EDT; it must not block.
- Register on the **model** for `JSlider`, `JSpinner`, `JProgressBar`; register on the **component** for `JTabbedPane` and `JColorChooser`.
- Programmatic state changes also fire `stateChanged`; set initial state before attaching the listener.

#### Constraints and Limitations

- `ChangeEvent` extends `java.util.EventObject`, not `AWTEvent`, so it has no ID and cannot be consumed.
- There is no built-in "adjusting" flag on the event; for sliders and progress bars, check the model's `getValueIsAdjusting()`.
- Change events fire very frequently during drags; heavy work must be debounced or offloaded.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic ChangeListener on a JSlider

```java
import javax.swing.*;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;
import java.awt.*;

public class BasicSliderChangeListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Basic ChangeListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSlider slider = new JSlider(0, 100, 50);
            JLabel valueLabel = new JLabel("Value: 50", SwingConstants.CENTER);

            slider.getModel().addChangeListener(new ChangeListener() { // 1. Register
                @Override
                public void stateChanged(ChangeEvent e) {              // 2. React
                    valueLabel.setText("Value: " + slider.getValue());
                }
            });

            frame.setLayout(new BorderLayout());
            frame.add(slider, BorderLayout.CENTER);
            frame.add(valueLabel, BorderLayout.SOUTH);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Dragging the slider knob updates the value label to the slider's current value in real time.

**Why the code produces this result.** The listener is registered on the slider's `BoundedRangeModel`. Every value change—whether by dragging, clicking arrows, or programmatic `setValue`—fires a `ChangeEvent`. The listener reads the new value from the slider and updates the label.

---

#### Example 2: ChangeListener on a JSpinner

```java
import javax.swing.*;
import java.awt.*;

public class SpinnerChangeListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Spinner ChangeListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSpinner spinner = new JSpinner(
                new SpinnerNumberModel(10, 0, 100, 5));
            JLabel status = new JLabel("Value: 10", SwingConstants.CENTER);

            spinner.addChangeListener(e -> {                          // 1. Component
                status.setText("Value: " + spinner.getValue());
            });

            frame.setLayout(new BorderLayout());
            frame.add(spinner, BorderLayout.NORTH);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the spinner arrows or typing a new value updates the status label to reflect the current value.

**Why the code produces this result.** `JSpinner` delegates `addChangeListener` to its model (`SpinnerNumberModel`), so registering on the component is equivalent to registering on the model. Every value change fires `stateChanged`, which reads the new value.

---

### Real-World Cases

- **Slider-driven previews:** Font size sliders that update a preview label as the user drags.
- **Spinner-driven forms:** Quantity spinners that update a total price display.
- **Progress monitors:** Progress bars whose `ChangeListener`s update status text on completion.
- **Color pickers:** Color choosers whose selection model drives a live preview panel.
- **Tabbed interfaces:** Tabbed panes whose change listeners lazy-load the newly selected tab's content.

---

## 2. stateChanged() — The Method

### Definitions

**Core Definition.** `stateChanged(ChangeEvent e)` is the single abstract method of `ChangeListener`. It is invoked when the source's state changes.

**Technical Definition.** `void stateChanged(ChangeEvent e)`. The method is invoked on the EDT by the model (or component) whenever its state changes. The `ChangeEvent` provides the source via `getSource()`. Unlike AWT event listeners, `stateChanged` has no event-ID parameter and no consumption mechanism. If the method throws an unchecked exception, it propagates up the EDT.

**Beginner-Friendly Explanation.** `stateChanged` is the "state just changed" method. Everything you write inside it runs on the EDT when the source's state changes.

---

### Purposes

- To contain the reaction logic for state changes.
- To receive the `ChangeEvent` and read the new state from the source.
- To trigger immediate UI updates (labels, previews, dependent controls).
- To synchronise state with a domain model (persisting preferences, updating configuration).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void stateChanged(ChangeEvent e) {
    Object source = e.getSource();
    // read new state from source and react
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `@Override` | Confirms the method overrides the interface declaration. |
| `ChangeEvent e` | The event object with the source. |
| `e.getSource()` | The object that fired the event (typically the model). |

#### Syntax Rules

- The method must be `public` and accept a single `ChangeEvent` parameter.
- The method should perform its work quickly; long tasks must be offloaded.
- Read the new state from the source, not from the event—`ChangeEvent` does not carry the new value.

#### Constraints and Limitations

- The EDT is blocked until the method returns.
- Uncaught exceptions can disrupt the event pipeline.
- The event does not identify what changed; if the source has multiple properties, you must inspect the source or track state externally.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading State from the Source

```java
import javax.swing.*;
import javax.swing.event.ChangeEvent;
import java.awt.*;

public class ReadStateFromSource {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Read State from Source");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 100);
            bar.setValue(0);
            bar.setStringPainted(true);

            JLabel status = new JLabel("Starting...", SwingConstants.CENTER);

            bar.getModel().addChangeListener((ChangeEvent e) -> {     // 1. Register
                int value = bar.getValue();                          // 2. Read source
                if (value >= bar.getMaximum()) {
                    status.setText("Complete!");
                } else {
                    status.setText("Progress: " + value + "%");
                }
            });

            frame.setLayout(new BorderLayout());
            frame.add(bar, BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            new Timer(200, ev -> {
                if (bar.getValue() < bar.getMaximum()) {
                    bar.setValue(bar.getValue() + 10);
                }
            }).start();
        });
    }
}
```

**Expected Output.** The progress bar advances by 10 every 200 ms; the status label updates to the current percentage and changes to "Complete!" when the maximum is reached.

**Why the code produces this result.** The listener is registered on the progress bar's model. Each `setValue` fires `stateChanged`, which reads the new value from the progress bar and updates the status label.

---

### Real-World Cases

- **Task completion detection:** Detecting when a progress bar reaches its maximum.
- **Synchronised displays:** Updating a label as a slider moves.
- **Preference persistence:** Saving the current value when a spinner changes.

---

## 3. ChangeEvent — The Event Object

### Definitions

**Core Definition.** `ChangeEvent` is the event object delivered to `ChangeListener`s. It describes a state change on a source object (typically a model).

**Technical Definition.** `public class ChangeEvent extends EventObject`. Unlike most Swing events, `ChangeEvent` is **not** an `AWTEvent`, so it has no event ID, no timestamp, and no consumption mechanism. The only information it carries is the source object (`getSource()`). All state-specific data must be retrieved from the source itself.

**Beginner-Friendly Explanation.** `ChangeEvent` is the "something changed" notification. It tells you *which object* changed, but not *what value* it changed to—you must ask the source for that.

---

### Purposes

- To notify listeners that a state change has occurred on a model or component.
- To provide the source object for state retrieval.
- To support a lightweight, uniform notification mechanism across many model types.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public void stateChanged(ChangeEvent e) {
    Object source = e.getSource();
    if (source instanceof BoundedRangeModel) {
        BoundedRangeModel model = (BoundedRangeModel) source;
        int value = model.getValue();
        // react
    }
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `e.getSource()` | Returns the object that fired the event. |

#### Syntax Rules

- The only data accessor on `ChangeEvent` is `getSource()`.
- The source is typically a model (`BoundedRangeModel`, `SpinnerModel`, `SingleSelectionModel`, `ColorSelectionModel`), not the component itself.
- To read the new state, call methods on the source.

#### Constraints and Limitations

- No event ID: the listener cannot distinguish different types of state changes on the same source from the event alone.
- No consumption: `ChangeEvent` cannot be consumed to prevent downstream handling.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Type-Checking the Source

```java
import javax.swing.*;
import javax.swing.event.ChangeEvent;
import java.awt.*;

public class TypeCheckSource {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Type-Check Source");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSlider slider = new JSlider(0, 100, 25);
            JLabel status = new JLabel("Slider value: 25", SwingConstants.CENTER);

            slider.getModel().addChangeListener((ChangeEvent e) -> {   // 1. Register
                Object src = e.getSource();                            // 2. Source
                if (src instanceof BoundedRangeModel) {
                    int value = ((BoundedRangeModel) src).getValue();
                    status.setText("Slider value: " + value);
                }
            });

            frame.setLayout(new BorderLayout());
            frame.add(slider, BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Dragging the slider updates the status label with the new value. The listener obtains the value from the model (the event source), not from the slider directly.

**Why the code produces this result.** The event source is the `BoundedRangeModel` (the slider's model). The listener checks the source type and casts it to `BoundedRangeModel` to read the current value.

---

### Real-World Cases

- **Shared models:** A listener registered on a shared `BoundedRangeModel` receives events when any component using that model changes, and the `getSource()` distinguishes the model from the component.
- **Generic listeners:** Listeners that react to any model changes and read the state from the source polymorphically.

---

## 4. Change Listener Best Practices

### Definitions

**Core Definition.** Change listener best practices are the conventions and patterns that ensure state-change handling is correct, thread-safe, and performant.

**Technical Definition.** Best practices include: registering on the model for sliders, spinners, and progress bars; checking `getValueIsAdjusting()` on the model (not on the event) for sliders and progress bars; offloading heavy work to `SwingWorker`; setting initial state before attaching the listener; removing listeners when components are disposed; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for change listeners are the "rules of thumb" that prevent duplicate handling, frozen GUIs, and memory leaks. They include things like "use the model, not the event, for adjusting state," "don't do heavy work in `stateChanged`," and "set initial state before attaching the listener."

---

### Purposes

- To avoid duplicate or intermediate handling during drag operations.
- To keep the GUI responsive by offloading long work.
- To prevent spurious reactions during initialization.
- To prevent memory leaks by removing listeners.
- To ensure thread safety by respecting the EDT.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JSlider slider = new JSlider(0, 100, 50);                     // 1. Initial state
    slider.getModel().addChangeListener(e -> {                     // 2. Model
        BoundedRangeModel model = slider.getModel();
        if (model.getValueIsAdjusting()) return;                  // 3. Adjusting
        updatePreview(slider.getValue());                          // 4. Lightweight
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Register on the model | `slider.getModel().addChangeListener(...)` |
| Check adjusting state | `model.getValueIsAdjusting()` on the **model**, not the event |
| Initial state first | Set `setValue` / `setSelectedIndex` before attaching |
| Lightweight handlers | No heavy computation or I/O in `stateChanged` |
| Offload long work | `SwingWorker` or background thread for expensive reactions |
| Remove listeners | `removeChangeListener(...)` on disposal |
| EDT | `SwingUtilities.invokeLater(...)` for construction |

#### Syntax Rules

- **Always register on the model** for `JSlider`, `JSpinner`, and `JProgressBar`. For `JTabbedPane` and `JColorChooser`, register on the component's change-listener API.
- **Always check `getValueIsAdjusting()` on the model** for sliders and progress bars to avoid acting on every intermediate drag event. There is no `getValueIsAdjusting()` on `ChangeEvent`.
- **Always set initial state before attaching the listener** to avoid spurious events during setup.
- **Never perform heavy work directly in `stateChanged`**; use `SwingWorker`.
- **Always remove the listener** from long-lived models when the listening component is disposed.

#### Constraints and Limitations

- `ChangeEvent` carries no state; the listener must read the new value from the source.
- Change events fire very frequently during drags; without `getValueIsAdjusting()` filtering, expensive handlers can degrade performance.
- The adjusting check is on the model, not the event—this is a common source of confusion for developers coming from `ListSelectionListener`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Slider with Adjusting Guard

```java
import javax.swing.*;
import javax.swing.BoundedRangeModel;
import java.awt.*;

public class BestPracticeChangeListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeChangeListener::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice ChangeListener");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JSlider slider = new JSlider(0, 100, 50);                     // 1. Initial
        JLabel preview = new JLabel("Font size: 50pt", SwingConstants.CENTER);

        slider.getModel().addChangeListener(e -> {                     // 2. Model
            BoundedRangeModel model = slider.getModel();
            if (model.getValueIsAdjusting()) return;                   // 3. Adjusting
            preview.setText("Font size: " + slider.getValue() + "pt"); // 4. React
        });

        frame.setLayout(new BorderLayout());
        frame.add(slider, BorderLayout.CENTER);
        frame.add(preview, BorderLayout.SOUTH);
        frame.setSize(450, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Dragging the slider updates the font-size label only when the user releases the knob, avoiding intermediate updates during the drag.

**Why the code produces this result.** The listener is registered on the slider's model and checks `model.getValueIsAdjusting()` to filter out intermediate events. The preview label is updated only for the final settled value.

---

#### Example 2: Common Mistake — Looking for `getValueIsAdjusting()` on the Event

```java
import javax.swing.*;
import javax.swing.event.ChangeEvent;

public class BadAdjustingCheck {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Adjusting Check");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSlider slider = new JSlider(0, 100, 50);

            slider.getModel().addChangeListener((ChangeEvent e) -> {
                // WRONG: ChangeEvent has no getValueIsAdjusting()
                // if (e.getValueIsAdjusting()) return;   // does not compile!

                // Also wrong: assuming the event tells you what changed
                System.out.println("State changed");
            });

            frame.add(slider);
            frame.setSize(300, 100);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The code compiles only after removing the `e.getValueIsAdjusting()` call, which does not exist on `ChangeEvent`. Without the model-based adjusting check, the handler runs many times per drag gesture, flooding the console with "State changed" messages.

**Why this is wrong.** `ChangeEvent` extends `java.util.EventObject` and does **not** carry a `valueIsAdjusting` flag. The adjusting state belongs to the `BoundedRangeModel`, which you query via `model.getValueIsAdjusting()`. This is a critical difference from `ListSelectionListener`, whose `ListSelectionEvent` **does** carry `getValueIsAdjusting()`.

---

#### Example 3: Best-Practice Tabbed Pane with Lazy Loading

```java
import javax.swing.*;
import java.awt.*;

public class TabbedPaneLazyLoad {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Tabbed Pane Lazy Load");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTabbedPane tabs = new JTabbedPane();
            tabs.addTab("Summary", new JLabel("Summary content", SwingConstants.CENTER));
            JPanel details = new JPanel();                             // 1. Placeholder
            tabs.addTab("Details", details);

            tabs.addChangeListener(e -> {                              // 2. Component
                int index = tabs.getSelectedIndex();
                if (index == 1 && details.getComponentCount() == 0) {
                    details.add(new JLabel("Loaded details on first view",
                        SwingConstants.CENTER));                       // 3. Lazy load
                    details.revalidate();
                    details.repaint();
                }
            });

            frame.add(tabs);
            frame.setSize(450, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The "Details" tab is empty when the application starts. When the user clicks "Details" for the first time, its content is loaded and displayed. Subsequent clicks reuse the loaded content.

**Why the code produces this result.** The `JTabbedPane`'s `SingleSelectionModel` fires `ChangeEvent`s when the selected index changes. The `ChangeListener` detects the first selection of the "Details" tab and populates it lazily, avoiding costly initialization until needed.

---

#### Example 4: Best-Practice Spinner with Offloaded Work

```java
import javax.swing.*;
import java.awt.*;

public class OffloadedSpinnerWork {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Offloaded Spinner Work");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSpinner spinner = new JSpinner(
                new SpinnerNumberModel(5, 1, 100, 1));
            JLabel status = new JLabel("Ready", SwingConstants.CENTER);

            spinner.addChangeListener(e -> {                           // 1. Register
                int count = ((Number) spinner.getValue()).intValue();
                new SwingWorker<String, Void>() {                      // 2. Offload
                    @Override
                    protected String doInBackground() {
                        return loadReport(count);                      // 3. Background
                    }
                    @Override
                    protected void done() {
                        try {
                            status.setText(get());                     // 4. EDT update
                        } catch (Exception ex) {
                            status.setText("Error");
                        }
                    }
                }.execute();
            });

            frame.setLayout(new BorderLayout());
            frame.add(spinner, BorderLayout.NORTH);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static String loadReport(int count) {
        try { Thread.sleep(800); } catch (InterruptedException e) {}
        return "Loaded report with " + count + " items";
    }
}
```

**Expected Output.** Changing the spinner value triggers a background load; the status label updates when the load completes. The UI remains responsive during the load.

**Why the code produces this result.** The `stateChanged` handler is lightweight—it reads the new value and starts a `SwingWorker`. The heavy loading work runs off the EDT, and the UI update occurs in `done()`, which runs on the EDT.

---

### Real-World Cases

- **Production Swing applications** register on models for sliders, spinners, and progress bars, and check `getValueIsAdjusting()` on the model for drag-sensitive reactions.
- **Live previews** (e.g., font size, brightness, zoom) use `ChangeListener` with adjusting guards to update previews only when the drag settles.
- **Lazy-loaded tabs** use `JTabbedPane`'s `ChangeListener` to populate content only when first viewed.
- **Long-running operations** triggered by spinner changes use `SwingWorker` to avoid freezing the EDT.

---

## References

Oracle. *Interface ChangeListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/ChangeListener.html

Oracle. *Class ChangeEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/ChangeEvent.html

Oracle. *How to Write a Change Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/changelistener.html

Oracle. *Interface BoundedRangeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/BoundedRangeModel.html

Oracle. *Interface SpinnerModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SpinnerModel.html

Oracle. *Interface SingleSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SingleSelectionModel.html

Oracle. *Class JSlider*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JSlider.html

Oracle. *Class JSpinner*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JSpinner.html

Oracle. *Class JProgressBar*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JProgressBar.html

Oracle. *Class JTabbedPane*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JTabbedPane.html

Oracle. *Class JColorChooser*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JColorChooser.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *Class SwingWorker*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html