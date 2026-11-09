# JProgressBar — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JProgressBar` is a Swing component that visually displays the progress of a task. As the task advances toward completion, the progress bar shows the task's percentage of completion—typically as a rectangle that starts empty and gradually fills in. The component can also display a textual representation of this percentage.

**Technical Definition.** `public class JProgressBar extends JComponent implements SwingConstants, Accessible`. `JProgressBar` uses a `BoundedRangeModel` as its data model, with the `value` property representing the current state of the task, and the `minimum` and `maximum` properties representing the beginning and end points, respectively. To indicate that a task of unknown length is executing, the progress bar can be placed into **indeterminate mode**, in which it animates continuously to show that work is occurring. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `JProgressBar` is the familiar horizontal (or vertical) bar that fills up as a task progresses—like the loading bar you see during software installation, file downloads, or long calculations. It tells the user “something is happening, and here is how much is left.” For tasks where the total length is unknown, the bar switches to an “indeterminate” mode, where it animates back and forth to signal that work is ongoing without pretending to know a percentage. Think of it as a visual thermometer for the completion of work: when the task is measurable, the mercury rises predictably; when the task is unknown, the thermometer simply shows that it is active.

---

### Key Characteristics

1. **Determinate and indeterminate modes.** In determinate mode, the bar fills according to the current `value` relative to the `minimum` and `maximum`. In indeterminate mode, the bar animates continuously to indicate ongoing work without a known completion percentage.
2. **BoundedRangeModel backing.** The component delegates value management to a `BoundedRangeModel`, which tracks `minimum`, `maximum`, `value`, and `extent`. This model can be shared with other components such as `JSlider` or `JScrollBar`.
3. **String painting.** A textual representation of the progress (or a custom string) can be displayed on the bar by calling `setStringPainted(true)`. The default string shows the percentage of completion.
4. **Orientation.** Progress bars can be horizontal (default) or vertical, set via `setOrientation`.
5. **Border painting.** The bar’s border can be toggled with `setBorderPainted(boolean)`.
6. **Change events.** `ChangeListener` is notified whenever the model’s value, minimum, maximum, or other properties change, enabling coordinated UI updates.
7. **Single-thread rule and serialization caveat.** `JProgressBar` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, and lambda expressions.
- Familiarity with Swing containers (`JFrame`, `JPanel`, `JDialog`) and layout managers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `ChangeListener` and the `javax.swing.event` package.
- Familiarity with `SwingWorker` or background threading for long-running tasks.
- A JDK (version 8 or later; `JProgressBar` is available in all modern JDKs).

---

### Related Programming Areas

- **`BoundedRangeModel` and `DefaultBoundedRangeModel`:** The underlying data model for progress values.
- **`SwingWorker`:** The standard mechanism for running long tasks off the EDT and posting progress updates safely.
- **`JSlider` and `JScrollBar`:** Other components that share the `BoundedRangeModel` architecture.
- **`ProgressMonitor` and `ProgressMonitorInputStream`:** Higher-level progress-tracking utilities built on `JProgressBar`.
- **`ChangeListener`:** The event interface for observing model changes.
- **EDT and Swing threading policy:** The rules governing safe UI updates from background threads.

---

## Core Concepts and Features

---

## 1. JProgressBar — The Class

### Definitions

**Core Definition.** `JProgressBar` is the Swing class for creating a visual progress indicator for a task.

**Technical Definition.** `public class JProgressBar extends JComponent implements SwingConstants, Accessible`. It provides constructors that accept orientation, minimum, maximum, and an initial value, or a `BoundedRangeModel`. The component’s UI delegate renders the filled portion, the border, and the optional progress string.

**Beginner-Friendly Explanation.** `JProgressBar` is the class you use to create a progress bar. You tell it the range of the task (from `minimum` to `maximum`) and update its `value` as work proceeds. The bar fills proportionally.

---

### Purposes

- To visually communicate the completion status of a measurable task.
- To reassure users that a long-running operation is progressing and has not frozen.
- To provide a determinate percentage display (optionally as text) for tasks with a known length.
- To indicate ongoing activity for tasks of unknown length via indeterminate mode.
- To integrate with `SwingWorker` and background threads for safe progress reporting.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JProgressBar bar = new JProgressBar();
JProgressBar bar = new JProgressBar(int min, int max);
JProgressBar bar = new JProgressBar(int orientation);
JProgressBar bar = new JProgressBar(int orientation, int min, int max);
JProgressBar bar = new JProgressBar(BoundedRangeModel model);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JProgressBar()` | Horizontal, range 0–100, initial value 0. |
| `JProgressBar(int min, int max)` | Horizontal, specified range. |
| `JProgressBar(int orientation)` | Specified orientation (`HORIZONTAL` or `VERTICAL`), default range. |
| `JProgressBar(int orientation, int min, int max)` | Full control over orientation and range. |
| `JProgressBar(BoundedRangeModel model)` | Uses the supplied model. |

#### Syntax Rules

- The `minimum` must be less than or equal to the `maximum`.
- The `value` must be within `[minimum, maximum]`.
- Construction must occur on the EDT.

#### Constraints and Limitations

- `JProgressBar` is for display only; it does not execute or manage the task itself.
- The component does not provide built-in task scheduling or threading; the programmer must update the value from the appropriate thread.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Determinate Progress Bar

```java
import javax.swing.*;
import java.awt.*;

public class MinimalProgressBar {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JProgressBar");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 100);          // 1. Range 0–100
            bar.setValue(0);                                       // 2. Initial value
            bar.setStringPainted(true);                            // 3. Show percentage

            JButton advance = new JButton("Advance 10%");
            advance.addActionListener(e -> {
                int next = Math.min(bar.getValue() + 10, bar.getMaximum());
                bar.setValue(next);                                // 4. Update value
            });

            JPanel panel = new JPanel(new BorderLayout(5, 5));
            panel.add(bar, BorderLayout.CENTER);
            panel.add(advance, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A progress bar starting at 0% with the percentage text “0%” painted on it. Each click on “Advance 10%” increases the value by 10, filling the bar and updating the text, until it reaches 100%.

**Why the code produces this result.** The constructor sets the range 0–100. `setStringPainted(true)` enables the percentage text. `setValue` updates the model, and the UI delegate repaints the filled portion proportionally.

---

### Real-World Cases

- **File downloads and installations** that show the percentage of data transferred or files copied.
- **Data import/export operations** that report how many records have been processed.
- **Long-running calculations** (e.g., simulations, rendering) that display progress toward completion.
- **Batch processing** in enterprise applications where users need reassurance that the system is working.

---

## 2. Progress Bar Creation

### Definitions

**Core Definition.** Progress bar creation is the process of instantiating a `JProgressBar` with the appropriate orientation, range, and initial value.

**Technical Definition.** `JProgressBar` provides constructors that accept orientation, minimum, maximum, and a `BoundedRangeModel`. The no-argument constructor creates a horizontal bar with a 0–100 range and an initial value of 0.

**Beginner-Friendly Explanation.** Creating a progress bar means deciding whether it should be horizontal or vertical and what range of values it should represent—for example, 0 to 100 for percentages, or 0 to the total number of files to process.

---

### Purposes

- To instantiate the progress indicator with the correct orientation and value range.
- To set an initial value that reflects the starting state of the task.
- To prepare the bar for value updates and listener attachment.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JProgressBar bar = new JProgressBar(0, totalTasks);
bar.setValue(0);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `int min` | The starting value of the task. |
| `int max` | The total length or maximum value of the task. |
| `int orientation` | `JProgressBar.HORIZONTAL` or `JProgressBar.VERTICAL`. |

#### Syntax Rules

- The `maximum` should reflect the total amount of work (e.g., number of files, bytes, or 100 for percentages).
- The initial `value` is typically `minimum` (usually 0).

#### Constraints and Limitations

- The range is integer-based; for fractional progress, scale the values (e.g., 0–1000 for tenths of a percent).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Progress Bar for a Known Task Length

```java
import javax.swing.*;
import java.awt.*;

public class CreationDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Creation Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            int totalFiles = 50;                                   // 1. Task length
            JProgressBar bar = new JProgressBar(0, totalFiles);   // 2. Range
            bar.setValue(0);                                       // 3. Start at 0
            bar.setStringPainted(true);

            frame.add(bar, BorderLayout.CENTER);
            frame.setSize(400, 100);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A horizontal progress bar representing 0 out of 50 files, with “0%” displayed.

**Why the code produces this result.** The constructor sets the range 0–50, and the string painting displays the percentage (value ÷ maximum × 100).

---

### Real-World Cases

- **Task monitors** that show the number of items processed out of a total.
- **Percentage-based progress** using a 0–100 range.

---

## 3. Progress Bar Properties — Minimum

### Definitions

**Core Definition.** The `minimum` property is the starting point of the progress bar’s range, representing zero progress.

**Technical Definition.** Managed by the `BoundedRangeModel`. Set at construction or via `setMinimum(int)`. The progress bar’s filled portion is computed relative to `(value − minimum) / (maximum − minimum)`.

**Beginner-Friendly Explanation.** The minimum is the value at which the bar is empty—the starting point of the task.

---

### Purposes

- To define the starting point of the task’s measurable range.
- To allow non-zero-based progress (e.g., starting at 1 instead of 0).
- To establish the scale for percentage calculation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.setMinimum(0);
int min = bar.getMinimum();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setMinimum(int)` | Sets the minimum value. |
| `getMinimum()` | Returns the minimum. |

#### Syntax Rules

- `minimum` must be ≤ `maximum`.
- Changing `minimum` may adjust `value` if it falls below the new minimum.

#### Constraints and Limitations

- For percentage-style bars, `minimum` is almost always 0.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Non-Zero Minimum

```java
import javax.swing.*;

public class MinimumDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimum Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(1, 100);          // 1. Start at 1
            bar.setValue(1);

            frame.add(bar);
            frame.setSize(300, 80);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A progress bar whose range begins at 1 (e.g., for “1-based” counting).

**Why the code produces this result.** The minimum is set to 1, so the bar is empty at value 1.

---

### Real-World Cases

- **File numbering** where items are counted from 1 to N.
- **Custom scales** where zero is not meaningful.

---

## 4. Progress Bar Properties — Maximum

### Definitions

**Core Definition.** The `maximum` property is the end point of the progress bar’s range, representing 100% completion.

**Technical Definition.** Managed by the `BoundedRangeModel`. Set at construction or via `setMaximum(int)`. The filled portion reaches full when `value == maximum`.

**Beginner-Friendly Explanation.** The maximum is the total amount of work; when the value reaches it, the bar is full.

---

### Purposes

- To define the total length of the task.
- To compute the percentage of completion.
- To allow dynamic adjustment when the task length becomes known (especially after indeterminate mode).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.setMaximum(total);
int max = bar.getMaximum();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setMaximum(int)` | Sets the maximum value. |
| `getMaximum()` | Returns the maximum. |

#### Syntax Rules

- `maximum` must be ≥ `minimum`.
- Changing `maximum` may affect the displayed percentage.

#### Constraints and Limitations

- If the task length changes, update `maximum` and `value` together to avoid inconsistent display.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Maximum Update

```java
import javax.swing.*;

public class MaximumDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Maximum Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar();
            bar.setStringPainted(true);

            JButton update = new JButton("Set Total to 200");
            update.addActionListener(e -> {
                bar.setMaximum(200);                              // 1. New total
                bar.setValue(50);                                 // 2. 25% progress
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(bar, BorderLayout.CENTER);
            panel.add(update, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(400, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Initially, the bar shows 0% (default range 0–100). Clicking the button sets the maximum to 200 and the value to 50, displaying “25%.”

**Why the code produces this result.** The percentage is computed as `(value − min) / (max − min) × 100`; 50 out of 200 is 25%.

---

### Real-World Cases

- **Batch jobs** where the total number of records is determined after the job starts.
- **File transfers** where the total size is known only after the connection is established.

---

## 5. Progress Bar Properties — Value

### Definitions

**Core Definition.** The `value` property is the current amount of progress achieved, measured in the same units as `minimum` and `maximum`.

**Technical Definition.** Managed by the `BoundedRangeModel`. Set at construction or via `setValue(int)`. The UI delegate uses this value to determine how much of the bar to fill.

**Beginner-Friendly Explanation.** The value is “how far along” the task is. You update it as work completes.

---

### Purposes

- To reflect the current state of the task.
- To drive the visual filling of the bar.
- To trigger `ChangeEvent`s for coordinated updates.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int current = bar.getValue();
bar.setValue(current + 1);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getValue()` | Returns the current value. |
| `setValue(int)` | Sets the current value. |

#### Syntax Rules

- `value` is clamped to `[minimum, maximum]`.
- Setting the value fires a `ChangeEvent`.

#### Constraints and Limitations

- Updating the value too frequently from a background thread without `SwingUtilities.invokeLater` causes thread-safety violations.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Incrementing Progress

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class ValueDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Value Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 10);
            bar.setStringPainted(true);

            Timer timer = new Timer(500, new ActionListener() {
                @Override
                public void actionPerformed(ActionEvent e) {
                    int next = bar.getValue() + 1;
                    if (next > bar.getMaximum()) {
                        ((Timer) e.getSource()).stop();
                        next = bar.getMaximum();
                    }
                    bar.setValue(next);                           // 1. Increment
                }
            });
            timer.start();

            frame.add(bar);
            frame.setSize(300, 100);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The bar advances by one unit every 500 ms until it reaches 10 (100%).

**Why the code produces this result.** `setValue` updates the model; the UI delegate repaints the filled portion. The `Timer` runs on the EDT, ensuring thread safety.

---

### Real-World Cases

- **Progress tracking** for downloads, installations, or data processing.
- **Simulation steps** where each iteration advances the bar.

---

## 6. Progress Bar Properties — String

### Definitions

**Core Definition.** The `string` property is the text that may be painted on the progress bar. By default, it is the percentage of completion, but it can be replaced with a custom string.

**Technical Definition.** The string is displayed when `stringPainted` is `true`. `setString(String)` sets a custom string; `setStringPainted(boolean)` controls whether the string is rendered. If no custom string is set, the UI delegate generates a default percentage string.

**Beginner-Friendly Explanation.** The string is the text on the bar—usually the percentage, but you can put anything you like, such as “Copying file 3 of 10” or “Processing…”.

---

### Purposes

- To provide additional textual context beyond the visual fill.
- To display custom messages (e.g., current step, file name, or time remaining).
- To show percentages when `stringPainted` is enabled.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.setStringPainted(true);
bar.setString("Custom message");
String s = bar.getString();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setStringPainted(boolean)` | Enables or disables string rendering. |
| `setString(String)` | Sets a custom string. |
| `getString()` | Returns the current string. |

#### Syntax Rules

- The string is visible only if `stringPainted` is `true`.
- Setting `null` restores the default percentage string.

#### Constraints and Limitations

- Custom strings may not fit within the bar if they are too long.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom String Painting

```java
import javax.swing.*;

public class StringDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("String Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 100);
            bar.setStringPainted(true);                            // 1. Enable
            bar.setString("Downloading...");                       // 2. Custom text

            JButton update = new JButton("Update to 50%");
            update.addActionListener(e -> {
                bar.setValue(50);
                bar.setString("50% — Downloading file.zip");      // 3. Dynamic text
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(bar, BorderLayout.CENTER);
            panel.add(update, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(400, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The bar initially shows the custom string “Downloading...”. After clicking the button, it shows “50% — Downloading file.zip.”

**Why the code produces this result.** `setStringPainted(true)` enables text rendering, and `setString` overrides the default percentage text.

---

### Real-World Cases

- **File copy dialogs** showing “Copying file 3 of 10: report.pdf.”
- **Installation wizards** showing the current component being installed.
- **Data processing** showing “Processed 1,250 of 10,000 records.”

---

## 7. Progress Bar Properties — Indeterminate

### Definitions

**Core Definition.** The `indeterminate` property toggles the progress bar between determinate mode (where it fills according to `value`) and indeterminate mode (where it animates continuously to show ongoing activity without a known completion percentage).

**Technical Definition.** Set via `setIndeterminate(boolean)`. When `true`, the progress bar ignores `value` for filling purposes and instead animates a moving block. When the task length becomes known, set `maximum` and `value`, then call `setIndeterminate(false)` to switch back to determinate mode.

**Beginner-Friendly Explanation.** Indeterminate mode is for tasks where you cannot measure progress—like waiting for a network response or an unknown-length computation. The bar animates to say “I'm working on it,” rather than showing a fake percentage.

---

### Purposes

- To indicate ongoing activity when the total task length is unknown.
- To prevent users from assuming the application has frozen.
- To transition smoothly to determinate mode once the task length becomes known.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.setIndeterminate(true);   // unknown-length task
// ... later, when length is known:
bar.setMaximum(newLength);
bar.setValue(current);
bar.setIndeterminate(false);  // switch to determinate
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setIndeterminate(boolean)` | `true` enables indeterminate animation; `false` restores determinate mode. |
| `isIndeterminate()` | Returns the current mode. |

#### Syntax Rules

- In indeterminate mode, `value`, `minimum`, and `maximum` are irrelevant to the animation, though they are preserved.
- The bar animates automatically on the EDT.

#### Constraints and Limitations

- The animation speed is look-and-feel-dependent.
- Indeterminate mode does not provide any information about how much work remains.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching from Indeterminate to Determinate

```java
import javax.swing.*;

public class IndeterminateDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Indeterminate Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar();
            bar.setStringPainted(true);
            bar.setIndeterminate(true);                            // 1. Unknown length
            bar.setString("Connecting...");

            JButton found = new JButton("Length Known: 500");
            found.addActionListener(e -> {
                bar.setMaximum(500);                               // 2. Set total
                bar.setValue(100);                                 // 3. Set current
                bar.setString(null);                               // 4. Default string
                bar.setIndeterminate(false);                       // 5. Determinate
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(bar, BorderLayout.CENTER);
            panel.add(found, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(400, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Initially, the bar animates continuously with the text “Connecting...”. Clicking the button stops the animation and shows a determinate bar at 20% (100 out of 500).

**Why the code produces this result.** `setIndeterminate(true)` triggers the animation. After setting `maximum` and `value` and calling `setIndeterminate(false)`, the bar renders the proportional fill.

---

### Real-World Cases

- **Network connections** where the server response time is unknown.
- **Database queries** whose execution time varies.
- **File system scans** where the number of files is not known in advance.

---

## 8. Progress Bar Methods — setValue()

### Definitions

**Core Definition.** `setValue(int n)` sets the current progress value, driving the filled portion of the bar.

**Technical Definition.** `public void setValue(int n)`. The value is clamped to `[minimum, maximum]`. The method fires a `ChangeEvent` on the model.

**Beginner-Friendly Explanation.** `setValue` is how you tell the bar “we are now this far along.”

---

### Purposes

- To update the visual progress as work completes.
- To reflect the current state of the task.
- To trigger coordinated updates via `ChangeListener`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.setValue(currentProgress);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setValue(int)` | Sets the current progress value. |

#### Syntax Rules

- The value is clamped to the valid range.
- Should be called on the EDT.

#### Constraints and Limitations

- Very frequent calls from a background thread without `invokeLater` cause thread-safety issues.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Updating Progress in a Loop

```java
import javax.swing.*;

public class SetValueDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setValue Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 20);
            bar.setStringPainted(true);

            new Timer(200, e -> {
                int next = bar.getValue() + 1;
                if (next > 20) {
                    ((Timer) e.getSource()).stop();
                    next = 20;
                }
                bar.setValue(next);                                // 1. Update
            }).start();

            frame.add(bar);
            frame.setSize(300, 100);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The bar advances by one unit every 200 ms until it reaches 20.

**Why the code produces this result.** `setValue` increments the model, and the bar repaints proportionally.

---

### Real-World Cases

- **Step-by-step processing** where each iteration updates the bar.
- **Download progress** updated as chunks arrive.

---

## 9. Progress Bar Methods — setStringPainted()

### Definitions

**Core Definition.** `setStringPainted(boolean b)` controls whether the progress bar renders a textual representation of its progress.

**Technical Definition.** `public void setStringPainted(boolean b)`. When `true`, the bar paints the current `string` (or the default percentage). When `false`, no text is shown.

**Beginner-Friendly Explanation.** `setStringPainted(true)` turns on the text on the bar; `false` turns it off.

---

### Purposes

- To display the percentage or a custom message on the bar.
- To provide additional context to the user.
- To allow the bar to be purely visual when text is unnecessary.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.setStringPainted(true);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setStringPainted(boolean)` | `true` enables text painting. |

#### Syntax Rules

- The text is visible only if `stringPainted` is `true`.
- The default text is the percentage.

#### Constraints and Limitations

- Text may be truncated if the bar is too narrow.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Toggling String Painting

```java
import javax.swing.*;

public class StringPaintedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("String Painted");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 100);
            bar.setValue(50);
            bar.setStringPainted(true);                            // 1. Enable

            JCheckBox toggle = new JCheckBox("Show Text", true);
            toggle.addItemListener(e ->
                bar.setStringPainted(toggle.isSelected()));        // 2. Toggle

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(bar, BorderLayout.CENTER);
            panel.add(toggle, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The bar shows “50%” when the checkbox is checked; the text disappears when unchecked.

**Why the code produces this result.** `setStringPainted` toggles the rendering of the progress string.

---

### Real-World Cases

- **Progress dialogs** where percentage text is helpful.
- **Minimalist UIs** where only the visual fill is desired.

---

## 10. Progress Bar Methods — setIndeterminate()

### Definitions

**Core Definition.** `setIndeterminate(boolean newValue)` switches the progress bar between determinate and indeterminate modes.

**Technical Definition.** `public void setIndeterminate(boolean newValue)`. In indeterminate mode, the bar animates continuously. In determinate mode, it fills according to `value`.

**Beginner-Friendly Explanation.** `setIndeterminate(true)` makes the bar animate for unknown-length tasks; `setIndeterminate(false)` returns it to normal progress display.

---

### Purposes

- To signal ongoing work without a known completion percentage.
- To transition to determinate mode once the task length is known.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.setIndeterminate(true);
bar.setIndeterminate(false);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setIndeterminate(boolean)` | `true` = indeterminate animation; `false` = determinate fill. |

#### Syntax Rules

- In indeterminate mode, `value` is ignored for filling.
- The animation runs on the EDT.

#### Constraints and Limitations

- The animation style depends on the look and feel.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Indeterminate Animation

```java
import javax.swing.*;

public class IndeterminateToggleDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Indeterminate Toggle");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar();
            bar.setStringPainted(true);
            bar.setIndeterminate(true);                            // 1. Start

            JButton toggle = new JButton("Toggle Mode");
            toggle.addActionListener(e ->
                bar.setIndeterminate(!bar.isIndeterminate()));      // 2. Toggle

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(bar, BorderLayout.CENTER);
            panel.add(toggle, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The bar animates continuously in indeterminate mode. Clicking the button switches to determinate mode (showing 0%), and clicking again returns to animation.

**Why the code produces this result.** `setIndeterminate` toggles the mode, and the UI delegate updates the rendering accordingly.

---

### Real-World Cases

- **Startup splash screens** that animate while the application loads.
- **Network operations** where the response time is unknown.

---

## 11. Progress Bar Events — ChangeListener

### Definitions

**Core Definition.** `ChangeListener` is an interface for receiving `ChangeEvent`s that fire when the progress bar’s model changes—including changes to `value`, `minimum`, `maximum`, and other properties.

**Technical Definition.** `public interface ChangeListener extends EventListener`. The `stateChanged(ChangeEvent e)` method is invoked whenever the `BoundedRangeModel` changes. The event source is the model or the progress bar, depending on how the listener is registered.

**Beginner-Friendly Explanation.** A `ChangeListener` runs every time the progress value or range changes. You can use it to update other UI elements or to detect when the task completes.

---

### Purposes

- To react to progress changes and update related components.
- To detect task completion when `value` reaches `maximum`.
- To coordinate multiple progress indicators or status labels.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
bar.addChangeListener(e -> {
    if (bar.getValue() >= bar.getMaximum()) {
        // task complete
    }
});
```

#### Component Breakdown

| Element | Meaning |
|--------|---------|
| `addChangeListener(ChangeListener)` | Registers the listener. |
| `stateChanged(ChangeEvent e)` | Called on model change. |

#### Syntax Rules

- The listener is invoked on the EDT.
- The event fires for all property changes, not only `value`.

#### Constraints and Limitations

- The listener may fire frequently; keep the handler lightweight.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting Completion

```java
import javax.swing.*;
import java.awt.*;

public class ChangeListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ChangeListener Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 10);
            bar.setStringPainted(true);
            JLabel status = new JLabel("In progress...");

            bar.addChangeListener(e -> {                            // 1. Register
                if (bar.getValue() >= bar.getMaximum()) {
                    status.setText("Done!");                       // 2. Complete
                }
            });

            new Timer(300, ev -> {
                if (bar.getValue() < 10) {
                    bar.setValue(bar.getValue() + 1);
                }
            }).start();

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(bar, BorderLayout.CENTER);
            panel.add(status, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The bar advances until it reaches 10; the status label changes from “In progress...” to “Done!” when the maximum is reached.

**Why the code produces this result.** The `ChangeListener` checks whether the value equals the maximum and updates the label accordingly.

---

### Real-World Cases

- **Task completion detection** to enable a “Next” button.
- **Multi-bar coordination** where one bar’s completion triggers another.
- **Status updates** based on progress thresholds.

---

## 12. JProgressBar Best Practices

### Definitions

**Core Definition.** `JProgressBar` best practices are the conventions and patterns that ensure progress bars are used effectively, thread-safely, and maintainably in Swing applications.

**Technical Definition.** Best practices include: never blocking the EDT with the task itself; using `SwingWorker` or a background thread to execute long tasks; updating the progress bar on the EDT via `SwingUtilities.invokeLater` or `SwingWorker`'s progress mechanism; using indeterminate mode when the task length is unknown; and switching to determinate mode once the length becomes known.

**Beginner-Friendly Explanation.** Best practices for progress bars are the “rules of thumb” that prevent frozen UIs and race conditions. They include things like “do the long work in a background thread,” “update the bar on the GUI thread,” and “use indeterminate mode only when you truly cannot measure progress.”

---

### Purposes

- To keep the GUI responsive by moving long-running work off the EDT.
- To ensure thread-safe updates to the progress bar.
- To provide accurate and meaningful progress feedback to the user.
- To transition smoothly between indeterminate and determinate modes.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JProgressBar bar = new JProgressBar(0, total);
    bar.setStringPainted(true);

    new SwingWorker<Void, Integer>() {
        @Override
        protected Void doInBackground() {
            for (int i = 0; i < total; i++) {
                // perform work item i
                publish(i);                                  // 1. Post progress
            }
            return null;
        }
        @Override
        protected void process(java.util.List<Integer> chunks) {
            int latest = chunks.get(chunks.size() - 1);
            bar.setValue(latest);                            // 2. EDT update
        }
        @Override
        protected void done() {
            bar.setValue(total);                             // 3. Final state
        }
    }.execute();
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Offload long work | `SwingWorker.doInBackground` |
| Update bar on EDT | `publish`/`process` or `SwingUtilities.invokeLater` |
| Indeterminate for unknown tasks | `setIndeterminate(true)` |
| Switch to determinate when length known | `setMaximum`, `setValue`, `setIndeterminate(false)` |
| Keep listeners light | Avoid heavy work in `ChangeListener` |
| Do not block EDT | Never call `Thread.sleep` or I/O on the EDT |

#### Syntax Rules

- **Never perform the task on the EDT.** Doing so freezes the GUI and prevents the progress bar from repainting.
- **Always update the progress bar on the EDT.** Use `SwingWorker.publish/process` or `SwingUtilities.invokeLater`.
- **Use indeterminate mode only when the task length is genuinely unknown.** Switch to determinate mode as soon as the length is known.
- **Keep `ChangeListener` handlers lightweight.** Offload heavy reactions to background threads if needed.

#### Constraints and Limitations

- Indeterminate mode does not convey how much work remains; use it only when necessary.
- Serialized `JProgressBar` objects are not compatible with future Swing releases.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Progress Bar with SwingWorker

```java
import javax.swing.*;
import java.awt.*;
import java.util.List;

public class BestPracticeProgressBar {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeProgressBar::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Progress");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        int total = 100;
        JProgressBar bar = new JProgressBar(0, total);
        bar.setStringPainted(true);                               // 1. Text

        JLabel status = new JLabel("Starting...");

        new SwingWorker<Void, Integer>() {
            @Override
            protected Void doInBackground() throws Exception {
                for (int i = 1; i <= total; i++) {
                    Thread.sleep(50);                             // 2. Simulate work
                    publish(i);                                   // 3. Publish progress
                }
                return null;
            }

            @Override
            protected void process(List<Integer> chunks) {
                int latest = chunks.get(chunks.size() - 1);
                bar.setValue(latest);                             // 4. EDT update
                status.setText("Processing " + latest + "%");
            }

            @Override
            protected void done() {
                bar.setValue(total);                              // 5. Ensure full
                status.setText("Complete!");
            }
        }.execute();

        JPanel panel = new JPanel(new BorderLayout(5, 5));
        panel.add(bar, BorderLayout.CENTER);
        panel.add(status, BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(400, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** The bar fills smoothly from 0% to 100% over approximately 5 seconds, with the status label updating to the current percentage. When complete, the label shows “Complete!”.

**Why the code produces this result.** `SwingWorker` runs the loop in a background thread, calls `publish` to queue progress updates, and `process` (on the EDT) updates the bar and label. The `done` method ensures the bar reaches 100%.

---

#### Example 2: Common Mistake — Blocking the EDT

```java
import javax.swing.*;

public class BadProgressBar {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Progress");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JProgressBar bar = new JProgressBar(0, 100);
            bar.setStringPainted(true);

            JButton start = new JButton("Start (Freezes)");
            start.addActionListener(e -> {
                for (int i = 0; i <= 100; i++) {
                    try { Thread.sleep(50); } catch (InterruptedException ex) {}
                    bar.setValue(i);                          // WRONG: EDT blocked
                }
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(bar, BorderLayout.CENTER);
            panel.add(start, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The UI freezes during the loop; the progress bar does not repaint and the window becomes unresponsive until the loop completes.

**Why this is wrong.** The `ActionListener` runs on the EDT. `Thread.sleep` and the loop block the EDT, preventing repaints and event processing. As the StackOverflow discussion emphasizes: “Swing is single threaded and not thread safe — this means you must not execute long running or blocking operations within the Event Dispatching Thread”. The correct approach is to use `SwingWorker` or a background thread and update the bar via `invokeLater`.

---

### Real-World Cases

- **Production Swing applications** use `SwingWorker` to keep the EDT free while updating progress bars safely.
- **Enterprise batch processors** show determinate progress for measurable workloads and switch to indeterminate during unpredictable phases.
- **Educational examples** use the best-practice skeleton to demonstrate correct threading and progress updating from the start.

---

## References

Oracle. *Class JProgressBar*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JProgressBar.html

Oracle. *Class JProgressBar*. Java Development Kit Version 24 API Specification. https://docs.oracle.com/en/java/javase/24/docs/api/java.desktop/javax/swing/JProgressBar.html

Oracle. *How to Monitor Progress*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/progress.html

Stack Overflow. *JProgressBar indeterminate mode, why thread required?* https://stackoverflow.com/questions/76984865/jprogressbar-indeterminate-mode-why-thread-required

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *Class SwingWorker*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html