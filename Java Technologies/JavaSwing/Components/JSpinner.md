# JSpinner — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JSpinner` is a Swing component that lets the user select a number or an object value from an ordered sequence. It provides a single-line input field with a pair of small up/down arrow buttons that increment or decrement the current value.

**Technical Definition.** `public class JSpinner extends JComponent implements Accessible`. A `JSpinner` is a single-line input field in which a number or an object value can be selected from an ordered sequence. Spinners typically provide a pair of tiny arrow buttons for stepping through the elements of the sequence. The keyboard up/down arrow keys also cycle through the elements. The user may also be allowed to type a legal value directly into the spinner. Although combo boxes provide similar functionality, spinners are sometimes preferred because they do not require a drop-down list that can cover up important parts of the program. The spinner's model is represented by a `SpinnerModel` interface. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `JSpinner` is the small input box with up/down arrows that you use to pick a number, a date, or an item from a list. Think of it as a compact “nudge” control: click the up arrow to increase the value, click the down arrow to decrease it, or type a value directly into the field. It is perfect for settings like “Number of copies,” “Font size,” or “Meeting date” where the user needs to adjust a value within a known range. Unlike a slider, the spinner shows the exact value and lets the user type it; unlike a combo box, it does not require a drop-down that may obscure other content.

---

### Key Characteristics

1. **Three built-in models.** Swing provides three specialized models: `SpinnerNumberModel` for numbers, `SpinnerListModel` for a fixed list of objects, and `SpinnerDateModel` for dates. A custom `SpinnerModel` can be implemented for arbitrary sequences.
2. **Ordered sequence semantics.** The spinner navigates an ordered sequence defined by the model. The model provides `getNextValue()`, `getPreviousValue()`, and `getValue()`, and fires `ChangeEvent`s when the value changes.
3. **Editable text field.** The user can type a value directly into the spinner’s text field. When the value is committed (Enter or focus loss), the model validates and applies it. The `JFormattedTextField` editor enforces formatting and parsing rules.
4. **Change events.** The spinner fires `ChangeEvent`s to registered `ChangeListener`s whenever the value changes, including changes initiated by the arrow buttons, keyboard arrows, or direct text entry.
5. **Editable state.** `setEnabled(false)` disables the entire spinner (field and arrows). `setEditor` can replace the default editor with a custom one.
6. **Compact footprint.** Spinners occupy minimal screen space and do not require a drop-down list, making them preferable to combo boxes in tight layouts.
7. **Single-thread rule and serialization caveat.** `JSpinner` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, generics, and lambda expressions.
- Familiarity with Swing containers (`JFrame`, `JPanel`) and layout managers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `ChangeListener` and the `javax.swing.event` package.
- Familiarity with `Date`, `Calendar`, or `java.time` APIs for `SpinnerDateModel`.
- A JDK (version 8 or later; `JSpinner` is available in all modern JDKs).

---

### Related Programming Areas

- **`SpinnerModel` and its implementations:** `SpinnerNumberModel`, `SpinnerListModel`, `SpinnerDateModel`.
- **`JFormattedTextField`:** The default editor used by `JSpinner` for formatted input.
- **`ChangeListener`:** The event interface for observing spinner value changes.
- **`JComboBox`:** Alternative for one-of-many selection; spinners avoid the drop-down overlay.
- **`JSlider`:** Alternative for continuous numeric input; spinners provide precise, typed input.
- **`Calendar` and `Date`:** Used by `SpinnerDateModel` for date navigation.

---

## Core Concepts and Features

---

## 1. JSpinner — The Class

### Definitions

**Core Definition.** `JSpinner` is the Swing class for creating a compact value-adjustment control with up/down arrows and an editable field.

**Technical Definition.** `public class JSpinner extends JComponent implements Accessible`. The class provides constructors that accept a `SpinnerModel` or create a default model. The spinner delegates value management to its model and uses a `JFormattedTextField` (via the model’s `getEditor` or the default editor) to display and edit the current value.

**Beginner-Friendly Explanation.** `JSpinner` is the class you use to create a spinner. You choose a model (number, list, or date), create the spinner with that model, and add it to your window. The spinner handles the arrows, keyboard navigation, and text editing.

---

### Purposes

- To provide a compact control for adjusting a value within an ordered sequence.
- To support typed input alongside arrow-button navigation.
- To avoid the screen real estate and overlay issues of drop-down lists.
- To serve as the standard Swing component for numeric, list-based, and date selection.
- To integrate with custom models for specialized value sequences.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JSpinner spinner = new JSpinner();
JSpinner spinner = new JSpinner(SpinnerModel model);
JSpinner spinner = new JSpinner(new SpinnerNumberModel(0, 0, 100, 1));
JSpinner spinner = new JSpinner(new SpinnerListModel(String[] items));
JSpinner spinner = new JSpinner(new SpinnerDateModel());
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JSpinner()` | Creates a spinner with a `SpinnerNumberModel` initialized to 0. |
| `JSpinner(SpinnerModel model)` | Creates a spinner using the given model. |

#### Syntax Rules

- The model must not be `null`; passing `null` throws `IllegalArgumentException`.
- The default model is a `SpinnerNumberModel(0, null, null, 1)`.
- Construction must occur on the EDT.

#### Constraints and Limitations

- The spinner’s editor is a `JFormattedTextField`; typed values must conform to the model’s type.
- Invalid input may be rejected or cause a parse exception depending on the model.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JSpinner with Default Model

```java
import javax.swing.*;
import java.awt.*;

public class MinimalSpinner {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JSpinner");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSpinner spinner = new JSpinner();                        // 1. Default model
            spinner.setValue(5);                                       // 2. Set value

            spinner.addChangeListener(e ->                            // 3. ChangeListener
                System.out.println("Value: " + spinner.getValue()));

            frame.add(spinner, BorderLayout.CENTER);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A spinner displaying “5” with up/down arrows. Clicking the arrows changes the value, and each change prints “Value: [n]” to the console.

**Why the code produces this result.** The default `SpinnerNumberModel` allows any integer. `setValue(5)` sets the initial value, and the `ChangeListener` fires on each change.

---

### Real-World Cases

- **Settings dialogs:** Number of copies, font size, or font weight.
- **Booking forms:** Number of guests, number of nights, or seat count.
- **Date pickers:** Meeting date or appointment time.
- **Preferences:** Timeout values, refresh intervals, or thread counts.

---

## 2. Spinner Creation

### Definitions

**Core Definition.** Spinner creation is the process of instantiating a `JSpinner` with an appropriate model that defines the sequence of allowed values.

**Technical Definition.** `JSpinner` provides a no-argument constructor that creates a default `SpinnerNumberModel` and a `SpinnerModel`-accepting constructor for custom sequences. The model can also be replaced later via `setModel`.

**Beginner-Friendly Explanation.** Creating a spinner means deciding what kind of values it should accept—numbers, list items, or dates—and passing the appropriate model to the constructor.

---

### Purposes

- To instantiate the spinner with the correct model for the expected data type.
- To set initial values, ranges, and step sizes for numeric spinners.
- To prepare the spinner for listener attachment and container addition.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JSpinner numberSpinner = new JSpinner(new SpinnerNumberModel(10, 0, 100, 5));
JSpinner listSpinner = new JSpinner(new SpinnerListModel(new String[]{"S", "M", "L"}));
JSpinner dateSpinner = new JSpinner(new SpinnerDateModel());
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `SpinnerNumberModel(value, min, max, step)` | Numeric model with range and step size. |
| `SpinnerListModel(values)` | Fixed list of items. |
| `SpinnerDateModel()` | Date model with default range and step. |

#### Syntax Rules

- The model defines the type of value the spinner accepts.
- Passing `null` for the model throws `IllegalArgumentException`.

#### Constraints and Limitations

- The spinner’s editor uses the model’s value type; mismatched types cause parse errors.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating Spinners with Different Models

```java
import javax.swing.*;
import java.awt.*;

public class SpinnerConstructors {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Spinner Constructors");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new GridLayout(3, 1, 5, 5));

            JSpinner number = new JSpinner(
                new SpinnerNumberModel(5, 1, 10, 1));                  // 1. Number
            JSpinner list = new JSpinner(
                new SpinnerListModel(new String[]{"Small", "Medium", "Large"})); // 2. List
            JSpinner date = new JSpinner(new SpinnerDateModel());      // 3. Date

            frame.add(number);
            frame.add(list);
            frame.add(date);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three spinners stacked vertically: a number spinner (1–10, initial 5), a list spinner (“Small,” “Medium,” “Large”), and a date spinner showing today’s date.

**Why the code produces this result.** Each model defines the type and range of values, and the spinner renders the appropriate editor.

---

### Real-World Cases

- **Quantity selection:** A number spinner with a 1–99 range.
- **Size selection:** A list spinner with predefined sizes.
- **Appointment scheduling:** A date spinner with a restricted range.

---

## 3. Spinner Models — SpinnerNumberModel

### Definitions

**Core Definition.** `SpinnerNumberModel` is a `SpinnerModel` implementation for sequences of numbers. It supports a minimum, a maximum, a step size, and an initial value.

**Technical Definition.** `public class SpinnerNumberModel extends AbstractSpinnerModel implements Serializable`. It accepts a `Number` value, a `Comparable` minimum, a `Comparable` maximum, and a `Number` step size. The model rejects values outside the range and values that are not multiples of the step size. It fires `ChangeEvent`s when the value changes.

**Beginner-Friendly Explanation.** `SpinnerNumberModel` is the model you use for numeric spinners. You tell it the starting value, the minimum, the maximum, and how much to add or subtract each time the user clicks an arrow.

---

### Purposes

- To restrict numeric input to a defined range.
- To enforce a step size (e.g., increments of 5).
- To provide a starting value for the spinner.
- To prevent invalid numeric input by rejecting out-of-range values.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SpinnerNumberModel model = new SpinnerNumberModel(
    10,    // initial value
    0,     // minimum
    100,   // maximum
    5);    // step size
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Number value` | The initial value. |
| `Comparable minimum` | The minimum allowed value (`null` for no minimum). |
| `Comparable maximum` | The maximum allowed value (`null` for no maximum). |
| `Number stepSize` | The increment/decrement for arrows. |

#### Syntax Rules

- The minimum and maximum must be `Comparable` to the value type.
- The step size must be positive.
- Passing `null` for min or max removes that bound.

#### Constraints and Limitations

- The model does not enforce that the value is a multiple of the step size; it only uses the step for arrow navigation.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Number Spinner with Range and Step

```java
import javax.swing.*;
import java.awt.*;

public class NumberModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Number Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            SpinnerNumberModel model = new SpinnerNumberModel(
                10, 0, 100, 5);                                      // 1. Range and step
            JSpinner spinner = new JSpinner(model);

            spinner.addChangeListener(e ->
                System.out.println("Value: " + spinner.getValue())); // 2. Change

            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A spinner starting at 10, with arrows that increment/decrement by 5. The value cannot go below 0 or above 100. Each change prints the new value.

**Why the code produces this result.** The model enforces the range and step size. The `ChangeListener` observes each change.

---

### Real-World Cases

- **Quantity pickers:** 1–99 with step 1.
- **Percentage settings:** 0–100 with step 5.
- **Port numbers:** 1024–65535 with step 1.

---

## 4. Spinner Models — SpinnerListModel

### Definitions

**Core Definition.** `SpinnerListModel` is a `SpinnerModel` implementation whose value is an element of a fixed list.

**Technical Definition.** `public class SpinnerListModel extends AbstractSpinnerModel implements Serializable`. The list can be provided as an array, a `List`, or a custom object with `getList()` and `setList()` methods. The model cycles through the list in order; the first element is the “next” after the last, and vice versa.

**Beginner-Friendly Explanation.** `SpinnerListModel` is the model you use when the spinner should cycle through a fixed set of items, like sizes (“Small,” “Medium,” “Large”) or days of the week.

---

### Purposes

- To provide a spinner that cycles through a fixed list of items.
- To enforce that only values from the list can be selected.
- To support list-based settings with a compact control.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SpinnerListModel model = new SpinnerListModel(new String[]{"A", "B", "C"});
JSpinner spinner = new JSpinner(model);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `SpinnerListModel(List<?> values)` | Creates a model from a `List`. |
| `SpinnerListModel(Object[] values)` | Creates a model from an array. |

#### Syntax Rules

- The list must not be empty.
- The model cycles through the list; the first element follows the last.

#### Constraints and Limitations

- The user cannot enter values outside the list (though the editor may allow text entry that gets rejected).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: List Spinner for Sizes

```java
import javax.swing.*;

public class ListModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("List Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            SpinnerListModel model = new SpinnerListModel(
                new String[]{"Small", "Medium", "Large", "Extra Large"}); // 1. List
            JSpinner spinner = new JSpinner(model);

            spinner.addChangeListener(e ->
                System.out.println("Size: " + spinner.getValue()));   // 2. Change

            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A spinner starting with “Small.” Clicking the up arrow cycles through “Medium,” “Large,” “Extra Large,” and back to “Small.” Each change prints the new size.

**Why the code produces this result.** The `SpinnerListModel` defines the sequence, and the spinner cycles through it.

---

### Real-World Cases

- **Size selection:** Clothing sizes, paper sizes, or storage capacities.
- **Day of week:** A spinner cycling through Monday–Sunday.
- **Priority levels:** Low, Medium, High, Critical.

---

## 5. Spinner Models — SpinnerDateModel

### Definitions

**Core Definition.** `SpinnerDateModel` is a `SpinnerModel` implementation for sequences of dates.

**Technical Definition.** `public class SpinnerDateModel extends AbstractSpinnerModel implements Serializable`. It maintains a `Calendar`-based value, a start date, an end date, and a `Calendar` field specifying the unit of change (e.g., `Calendar.DAY_OF_MONTH`, `Calendar.HOUR`). The default field is `Calendar.DAY_OF_MONTH`.

**Beginner-Friendly Explanation.** `SpinnerDateModel` is the model you use for date spinners. You can set the range of allowed dates and what unit the arrows change (days, hours, etc.).

---

### Purposes

- To provide a compact date-selection control.
- To restrict dates to a valid range.
- To allow the user to navigate dates by day, week, month, or other calendar field.
- To support date-based settings without a full calendar picker.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SpinnerDateModel model = new SpinnerDateModel(
    new Date(),       // initial date
    startDate,        // earliest date (or null)
    endDate,          // latest date (or null)
    Calendar.DAY_OF_MONTH); // unit of change
JSpinner spinner = new JSpinner(model);
spinner.setEditor(new JSpinner.DateEditor(spinner, "yyyy-MM-dd"));
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Date value` | Initial date. |
| `Comparable start` | Earliest allowed date (`null` for no bound). |
| `Comparable end` | Latest allowed date (`null` for no bound). |
| `int calendarField` | The unit changed by the arrows (e.g., `Calendar.DAY_OF_MONTH`). |

#### Syntax Rules

- The `calendarField` must be a valid `Calendar` field constant.
- The start and end dates may be `null` to indicate no bounds.
- The editor format is controlled by `JSpinner.DateEditor`.

#### Constraints and Limitations

- The range check compares `Date` values; incorrect use of time zones may cause off-by-one issues.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Date Spinner with Range and Format

```java
import javax.swing.*;
import java.util.Calendar;
import java.util.Date;

public class DateModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Date Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            Calendar cal = Calendar.getInstance();
            Date initial = cal.getTime();
            cal.add(Calendar.YEAR, -1);
            Date start = cal.getTime();
            cal.add(Calendar.YEAR, 2);
            Date end = cal.getTime();

            SpinnerDateModel model = new SpinnerDateModel(
                initial, start, end, Calendar.DAY_OF_MONTH);           // 1. Date model
            JSpinner spinner = new JSpinner(model);
            spinner.setEditor(new JSpinner.DateEditor(spinner, "yyyy-MM-dd")); // 2. Format

            spinner.addChangeListener(e ->
                System.out.println("Date: " + spinner.getValue()));    // 3. Change

            frame.add(spinner);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A spinner displaying today’s date in “yyyy-MM-dd” format. The arrows change the date by one day, constrained to the range [today−1 year, today+1 year]. Each change prints the new date.

**Why the code produces this result.** The model enforces the range and day-based increments. The `DateEditor` controls the display format.

---

### Real-World Cases

- **Appointment scheduling:** Picking a date within a booking window.
- **Report filters:** Selecting a date for data retrieval.
- **Deadline selection:** Choosing a date within a project timeline.

---

## 6. Spinner Methods — getValue()

### Definitions

**Core Definition.** `getValue()` returns the current value of the spinner as an `Object`.

**Technical Definition.** `public Object getValue()`. The method delegates to the model’s `getValue()`. The returned object’s type depends on the model: `Number` for `SpinnerNumberModel`, list element for `SpinnerListModel`, `Date` for `SpinnerDateModel`.

**Beginner-Friendly Explanation.** `getValue()` tells you the current value shown in the spinner.

---

### Purposes

- To read the current value for processing or storage.
- To compare the value against expected thresholds.
- To synchronize the spinner with a domain model.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Object value = spinner.getValue();
// cast to the expected type:
int number = ((Number) spinner.getValue()).intValue();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getValue()` | Returns the model’s current value. |

#### Syntax Rules

- The returned object type depends on the model.
- Casting is required to use the value as a specific type.

#### Constraints and Limitations

- The method returns the model’s value, which may differ from the editor’s text if the user is mid-edit.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading the Spinner Value

```java
import javax.swing.*;

public class GetValueDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("getValue Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSpinner spinner = new JSpinner(
                new SpinnerNumberModel(10, 0, 100, 1));

            JButton read = new JButton("Read Value");
            read.addActionListener(e -> {
                int value = ((Number) spinner.getValue()).intValue(); // 1. Cast
                System.out.println("Value: " + value);
            });

            JPanel panel = new JPanel();
            panel.add(spinner);
            panel.add(read);

            frame.add(panel);
            frame.setSize(350, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Read Value” prints the current numeric value of the spinner.

**Why the code produces this result.** `getValue()` returns a `Number`, which is cast to `int`.

---

### Real-World Cases

- **Form submission:** Reading the numeric value for processing.
- **Settings save:** Storing the spinner value in preferences.

---

## 7. Spinner Methods — setValue()

### Definitions

**Core Definition.** `setValue(Object value)` sets the spinner’s current value to the specified object.

**Technical Definition.** `public void setValue(Object value)`. The method delegates to the model’s `setValue`. The value must be valid for the model; otherwise, an `IllegalArgumentException` may be thrown.

**Beginner-Friendly Explanation.** `setValue` changes the value shown in the spinner.

---

### Purposes

- To set an initial or default value.
- To programmatically restore a saved value.
- To synchronize the spinner with a domain model.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
spinner.setValue(42);
spinner.setValue(new Date());
spinner.setValue("Medium");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setValue(Object)` | Sets the current value. |

#### Syntax Rules

- The value must be of the type expected by the model.
- Setting an invalid value may throw an exception.

#### Constraints and Limitations

- For `SpinnerNumberModel`, the value must be within the min/max range.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting the Spinner Value

```java
import javax.swing.*;

public class SetValueDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setValue Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSpinner spinner = new JSpinner(
                new SpinnerNumberModel(5, 0, 100, 1));
            spinner.setValue(25);                                     // 1. Set value

            JButton reset = new JButton("Reset to 5");
            reset.addActionListener(e -> spinner.setValue(5));       // 2. Reset

            JPanel panel = new JPanel();
            panel.add(spinner);
            panel.add(reset);

            frame.add(panel);
            frame.setSize(350, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner initially shows 25. Clicking “Reset to 5” sets it back to 5.

**Why the code produces this result.** `setValue` updates the model’s value, and the spinner repaints.

---

### Real-World Cases

- **Restoring saved preferences:** Setting the spinner to the saved value on startup.
- **Reset buttons:** Restoring the default value.

---

## 8. Spinner Methods — setModel()

### Definitions

**Core Definition.** `setModel(SpinnerModel model)` replaces the spinner’s model with the specified model.

**Technical Definition.** `public void setModel(SpinnerModel model)`. The method sets the spinner’s model, notifies property change listeners, and reconfigures the editor to match the new model’s value type.

**Beginner-Friendly Explanation.** `setModel` swaps the entire sequence of values the spinner can cycle through.

---

### Purposes

- To change the type or range of values the spinner accepts at runtime.
- To switch between numeric, list, and date models dynamically.
- To reconfigure the spinner based on application state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
spinner.setModel(new SpinnerNumberModel(0, 0, 10, 1));
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setModel(SpinnerModel)` | Replaces the model. |

#### Syntax Rules

- The model must not be `null`.
- The editor is reconfigured based on the new model.

#### Constraints and Limitations

- Changing the model type (e.g., from number to date) may require reconfiguring the editor format.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching Spinner Models

```java
import javax.swing.*;

public class SetModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setModel Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSpinner spinner = new JSpinner(
                new SpinnerNumberModel(1, 1, 10, 1));

            JButton switchButton = new JButton("Switch to List");
            switchButton.addActionListener(e ->
                spinner.setModel(new SpinnerListModel(
                    new String[]{"Red", "Green", "Blue"})));          // 1. Switch

            JPanel panel = new JPanel();
            panel.add(spinner);
            panel.add(switchButton);

            frame.add(panel);
            frame.setSize(400, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner initially shows a number (1). Clicking “Switch to List” changes it to a list spinner with “Red,” “Green,” and “Blue.”

**Why the code produces this result.** `setModel` replaces the model, and the spinner reconfigures its editor.

---

### Real-World Cases

- **Mode switching:** Changing the spinner’s model based on a selected category.
- **Dynamic configuration:** Adapting the spinner to different data types.

---

## 9. Spinner Events — ChangeListener

### Definitions

**Core Definition.** `ChangeListener` is an interface for receiving `ChangeEvent`s that fire whenever the spinner’s value changes.

**Technical Definition.** `public interface ChangeListener extends EventListener`. The `stateChanged(ChangeEvent e)` method is invoked when the spinner’s value changes—whether by arrow buttons, keyboard arrows, or direct text entry. The event source is the `JSpinner`.

**Beginner-Friendly Explanation.** A `ChangeListener` runs whenever the spinner’s value changes. You use it to react to the new value—for example, updating a preview or saving a setting.

---

### Purposes

- To react to value changes in the spinner.
- To update other UI components based on the new value.
- To persist the value when it changes.
- To validate or constrain the value as it changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
spinner.addChangeListener(e -> {
    Object value = spinner.getValue();
    // react to change
});
```

#### Component Breakdown

| Element | Meaning |
|--------|---------|
| `addChangeListener(ChangeListener)` | Registers the listener. |
| `stateChanged(ChangeEvent e)` | Called on value change. |

#### Syntax Rules

- The listener is invoked on the EDT.
- The event source is the `JSpinner`.
- The listener fires for all changes, including programmatic `setValue`.

#### Constraints and Limitations

- The event does not carry the new value; query `getValue()`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Updating a Label on Change

```java
import javax.swing.*;
import java.awt.*;

public class ChangeListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ChangeListener Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JSpinner spinner = new JSpinner(
                new SpinnerNumberModel(5, 1, 10, 1));
            JLabel status = new JLabel("Value: 5");

            spinner.addChangeListener(e -> {                          // 1. Register
                status.setText("Value: " + spinner.getValue());       // 2. Update
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(spinner, BorderLayout.CENTER);
            panel.add(status, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label shows “Value: 5” initially. Clicking the spinner arrows updates it to reflect the new value.

**Why the code produces this result.** The `ChangeListener` fires on each value change and reads the new value via `getValue()`.

---

### Real-World Cases

- **Live preview:** Updating a font preview as the user adjusts a size spinner.
- **Settings save:** Persisting the value as soon as it changes.
- **Validation:** Enforcing business rules when the value changes.

---

## 10. JSpinner Best Practices

### Definitions

**Core Definition.** `JSpinner` best practices are the conventions and patterns that ensure spinners are used effectively, accessibly, and maintainably in Swing applications.

**Technical Definition.** Best practices include: choosing the correct model for the data type; setting sensible ranges and step sizes; always checking `getValue()` on the EDT; using `ChangeListener` for value changes; configuring the editor format for date spinners; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for spinners are the “rules of thumb” that prevent common problems: using the wrong model, leaving ranges unconstrained, or forgetting to format date output. They include things like “use `SpinnerNumberModel` for numbers,” “set min and max,” and “use `DateEditor` to format dates.”

---

### Purposes

- To ensure the spinner accepts only valid values.
- To provide a good user experience with sensible defaults and constraints.
- To maintain thread safety by respecting the EDT.
- To format date and number output appropriately.
- To avoid runtime exceptions from invalid model configurations.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    SpinnerNumberModel model = new SpinnerNumberModel(
        10, 0, 100, 5);                                          // 1. Range and step
    JSpinner spinner = new JSpinner(model);                       // 2. Spinner
    spinner.setToolTipText("Adjust the value (0–100)");           // 3. Tooltip

    spinner.addChangeListener(e -> {                              // 4. Listener
        int value = ((Number) spinner.getValue()).intValue();
        // react
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Correct model type | `SpinnerNumberModel`, `SpinnerListModel`, `SpinnerDateModel` |
| Sensible range and step | `new SpinnerNumberModel(value, min, max, step)` |
| Format date output | `new JSpinner.DateEditor(spinner, "yyyy-MM-dd")` |
| Provide tooltips | `spinner.setToolTipText("...")` |
| Use `ChangeListener` | `spinner.addChangeListener(e -> ...)` |
| Cast types correctly | `((Number) spinner.getValue()).intValue()` |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Always use the correct model** for the data type; do not force a number into a list model.
- **Always set sensible minimum and maximum bounds** for numeric spinners to prevent invalid input.
- **Always format date spinners** with a `JSpinner.DateEditor` to produce readable output.
- **Always cast the value** returned by `getValue()` to the expected type.
- **Use `ChangeListener`**, not `ActionListener`, to observe value changes.

#### Constraints and Limitations

- The spinner's text editor may reject invalid input without warning; consider adding validation logic in a `ChangeListener`.
- `SpinnerDateModel` uses `Date` and `Calendar`, which are legacy APIs; consider `java.time` conversions if needed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Number Spinner

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class BestPracticeSpinner {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeSpinner::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Spinner");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        SpinnerNumberModel model = new SpinnerNumberModel(
            10, 0, 100, 5);                                          // 1. Model
        JSpinner spinner = new JSpinner(model);
        spinner.setToolTipText("Adjust the quantity (0–100, step 5)"); // 2. Tooltip

        JLabel status = new JLabel("Quantity: 10");

        spinner.addChangeListener(e -> {                             // 3. Listener
            int value = ((Number) spinner.getValue()).intValue();
            status.setText("Quantity: " + value);
        });

        JPanel panel = new JPanel(new BorderLayout(5, 5));            // 4. Layout
        panel.add(spinner, BorderLayout.CENTER);
        panel.add(status, BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(300, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A spinner with a quantity value, a tooltip, and a status label. Adjusting the spinner updates the status label.

**Why the code produces this result.** The model enforces the range and step, the tooltip explains the valid values, and the listener reacts to changes.

---

#### Example 2: Common Mistake — Missing Range Constraints

```java
import javax.swing.*;

public class BadSpinner {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Spinner");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            // WRONG: no min/max constraints
            JSpinner spinner = new JSpinner(new SpinnerNumberModel()); // 1. No bounds
            spinner.setValue(-999999);                                 // 2. Invalid logic

            frame.add(spinner);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The spinner accepts any integer, including negative values that may violate the application's expectations.

**Why this is wrong.** Without minimum and maximum bounds, the spinner allows values that may be invalid for the application. Always use `SpinnerNumberModel(value, min, max, step)` with sensible bounds.

---

### Real-World Cases

- **Production settings dialogs** use `SpinnerNumberModel` with ranges and `ChangeListener` for persistence.
- **Date selection** uses `SpinnerDateModel` with a `DateEditor` and a range.
- **List-based options** use `SpinnerListModel` for fixed choices.

---

## References

Oracle. *Class JSpinner*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JSpinner.html

Oracle. *How to Use Spinners*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/spinner.html

Oracle. *Class SpinnerNumberModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SpinnerNumberModel.html

Oracle. *Class SpinnerListModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SpinnerListModel.html

Oracle. *Class SpinnerDateModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SpinnerDateModel.html

Oracle. *Class JSpinner.DateEditor*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JSpinner.DateEditor.html

Stack Overflow. *JSpinner getValue and setValue casting*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/19822802/jspinner-value-change-listener

Stack Overflow. *JSpinner date format*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/8430354/jspinner-date-format

Cleverence. *JSpinner in Java Swing: Complete Guide with Examples*. https://www.cleverence.com/articles/oracle-documentation/jspinner-java-platform-se-8-5381/