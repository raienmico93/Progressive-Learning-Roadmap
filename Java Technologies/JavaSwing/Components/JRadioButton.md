# JRadioButton — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JRadioButton` is a Swing component that represents a radio button—an item that can be selected or deselected, displaying its state to the user. It is typically used with a `ButtonGroup` object to create a group of buttons in which only one button at a time can be selected.

**Technical Definition.** `public class JRadioButton extends JToggleButton implements Accessible`. An implementation of a radio button—an item that can be selected or deselected, and which displays its state to the user. Used with a `ButtonGroup` object to create a group of buttons in which only one button at a time can be selected. The `ButtonGroup` object is a logical grouping—not a physical grouping; to create a button panel, you should still create a `JPanel` or similar container-object and add a `Border` to it to set it off from surrounding components. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `JRadioButton` is the circular button you click to choose one option from a set. When you select one radio button in a group, any previously selected button in that group is automatically deselected. Think of it like the old car radio preset buttons: pressing one pops out all the others. Radio buttons are used for mutually exclusive choices—you can pick only one option at a time. If you need independent on/off choices (where multiple can be selected), use `JCheckBox` instead. Radio buttons shine when the available options are few—typically two to five—and when seeing all options at once adds clarity.

---

### Key Characteristics

1. **Mutual exclusivity via ButtonGroup.** A `JRadioButton` on its own acts much like a checkbox: it can be selected or deselected. The key difference emerges when you put multiple `JRadioButton`s into a `ButtonGroup`; the group enforces the rule that only one can be selected at a time.
2. **Logical grouping, not physical.** The `ButtonGroup` object is a logical grouping—not a physical grouping. To create a visual button panel, you should still create a `JPanel` or similar container-object and add a `Border` to it.
3. **Initially unselected.** Initially, all buttons managed by a `ButtonGroup` instance are unselected. You must explicitly set one as selected if you want a default choice.
4. **ItemEvent-centric.** When a radio button is pressed and released, an `ItemEvent` is sent to all registered `ItemListener`s to inform them that the radio button has been selected or deselected. `ItemListener` fires only when the selected/deselected state changes, unlike `ActionListener`, which fires on every click even if the button is already selected.
5. **Keyboard navigation.** Swing automatically wires up keyboard behavior so that arrow keys move between group members and the space bar toggles the currently focused button.
6. **Look-and-feel aware.** `JRadioButton` respects the current Look and Feel (L&F) and supports icons, tool tips, and HTML-formatted labels.
7. **Single-thread rule and serialization caveat.** `JRadioButton` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, and lambda expressions.
- Familiarity with Swing containers such as `JFrame`, `JPanel`, and layout managers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `ActionListener`, `ItemListener`, and the `java.awt.event` package.
- A JDK (version 8 or later; `JRadioButton` is available in all modern JDKs).

---

### Related Programming Areas

- **`JToggleButton` and `AbstractButton`:** The superclasses of `JRadioButton`; provide most of the button API, including `ButtonModel`, `setSelected`, and `isSelected`.
- **`ButtonGroup`:** The class that enforces mutual exclusivity among a set of radio buttons.
- **`JCheckBox`:** For independent, multiple-selection choices (where more than one option can be selected).
- **`JComboBox` and `JList`:** Alternatives when there are many options (say, 10+ items) and a column of radio buttons would be unwieldy.
- **`ItemListener` and `ActionListener`:** The two primary event interfaces for radio button interaction.
- **`UIManager` and Look and Feel:** Control the appearance of radio buttons across platforms.

---

## Core Concepts and Features

---

## 1. JRadioButton — The Class

### Definitions

**Core Definition.** `JRadioButton` is the Swing class for creating mutually exclusive selectable options that display a circular indicator with a filled dot when selected.

**Technical Definition.** `public class JRadioButton extends JToggleButton implements Accessible`. It provides constructors that accept text, an icon, and an initial selection state. The class inherits the full `AbstractButton` API, including support for `Action`, mnemonics, tooltips, and `ButtonModel`.

**Beginner-Friendly Explanation.** `JRadioButton` is the class you use to create a radio button. You create it with a label like "Small," "Medium," or "Large," add it to a `ButtonGroup`, and the group ensures only one size can be selected at a time.

---

### Purposes

- To provide a mutually exclusive selection control for choosing one option from a set.
- To enforce single-selection semantics within a group of related options.
- To serve as the standard Swing component for exclusive choices in preferences, settings, and forms.
- To integrate with `ButtonGroup` for centralized selection management.
- To support keyboard and mouse interaction for accessible option selection.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JRadioButton rb = new JRadioButton();
JRadioButton rb = new JRadioButton(String text);
JRadioButton rb = new JRadioButton(String text, boolean selected);
JRadioButton rb = new JRadioButton(Icon icon);
JRadioButton rb = new JRadioButton(String text, Icon icon, boolean selected);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JRadioButton()` | Creates an unselected radio button with no text or icon. |
| `JRadioButton(String text)` | Creates an unselected radio button with the specified text. |
| `JRadioButton(String text, boolean selected)` | Creates a radio button with text and the specified initial state. |
| `JRadioButton(Icon icon)` | Creates an unselected radio button with an icon. |
| `JRadioButton(String text, Icon icon, boolean selected)` | Creates a radio button with text, icon, and initial state. |

#### Syntax Rules

- The default selection state is `false` (unselected).
- The radio button must be added to a `ButtonGroup` for mutual exclusivity and to a container for display.
- Construction must occur on the EDT.

#### Constraints and Limitations

- A `JRadioButton` without a `ButtonGroup` behaves like a checkbox (independent toggle).
- Radio buttons are for exclusive choices; for independent options, use `JCheckBox`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JRadioButton with ButtonGroup

```java
import javax.swing.*;
import java.awt.*;

public class MinimalRadioButton {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JRadioButton");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton small = new JRadioButton("Small");          // 1. Create
            JRadioButton medium = new JRadioButton("Medium");
            JRadioButton large = new JRadioButton("Large");

            ButtonGroup group = new ButtonGroup();                    // 2. Group
            group.add(small);
            group.add(medium);
            group.add(large);

            medium.setSelected(true);                                 // 3. Default

            small.addItemListener(e -> {                              // 4. Listen
                if (e.getStateChange() == ItemEvent.SELECTED) {
                    System.out.println("Selected: Small");
                }
            });

            JPanel panel = new JPanel(new GridLayout(3, 1));          // 5. Layout
            panel.setBorder(BorderFactory.createTitledBorder("Size"));
            panel.add(small);
            panel.add(medium);
            panel.add(large);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a titled panel "Size" containing three radio buttons: "Small," "Medium," and "Large." "Medium" is selected by default. Clicking "Small" deselects "Medium" and prints "Selected: Small."

**Why the code produces this result.** The `ButtonGroup` enforces mutual exclusivity. `setSelected(true)` sets the default. The `ItemListener` captures the state change when "Small" becomes selected.

---

### Real-World Cases

- **Sort order dialogs:** Ascending vs. Descending.
- **Unit selection:** Imperial vs. Metric.
- **Theme selection:** Light vs. Dark mode.
- **Survey questions:** Yes/No/Maybe options where only one answer is valid.

---

## 2. Radio Button Creation

### Definitions

**Core Definition.** Radio button creation is the process of instantiating a `JRadioButton` with the desired text, icon, and initial selection state.

**Technical Definition.** `JRadioButton` provides constructors that accept combinations of text, icon, and a boolean initial-selection flag. The initial state matters because the `ButtonGroup` starts with all buttons unselected, and you must explicitly set one as the default if a default choice is desired.

**Beginner-Friendly Explanation.** Creating a radio button means deciding what label it should show and whether it should start selected. You then add it to a `ButtonGroup` to make it exclusive with its peers.

---

### Purposes

- To instantiate the radio button with the appropriate visual label.
- To set the initial selection state as a default choice.
- To prepare the component for addition to a `ButtonGroup` and a container.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JRadioButton rb = new JRadioButton("Option A");
rb.setSelected(true); // default selection
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `String text` | The label displayed next to the radio button. |
| `boolean selected` | The initial state (`true` = selected). |

#### Syntax Rules

- If a default selection is desired, call `setSelected(true)` on one button in the group.
- Without a default, all buttons start unselected.

#### Constraints and Limitations

- Only one button per `ButtonGroup` should be selected at a time; the group enforces this.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating Radio Buttons with Different Constructors

```java
import javax.swing.*;
import java.awt.*;

public class RadioButtonConstructors {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Radio Constructors");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton basic = new JRadioButton("Basic");          // 1. Text
            JRadioButton preSelected = new JRadioButton("Pre-selected", true); // 2. Initial true
            JRadioButton withIcon = new JRadioButton("With Icon",    // 3. Text + icon
                new ImageIcon(new java.awt.image.BufferedImage(
                    16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB)));

            ButtonGroup group = new ButtonGroup();
            group.add(basic);
            group.add(preSelected);
            group.add(withIcon);

            JPanel panel = new JPanel(new GridLayout(3, 1));
            panel.add(basic);
            panel.add(preSelected);
            panel.add(withIcon);

            frame.add(panel);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three radio buttons: "Basic" (unselected), "Pre-selected" (selected), and "With Icon" (unselected, with a small icon).

**Why the code produces this result.** Each constructor initializes the radio button differently, and the `ButtonGroup` ensures only one is selected.

---

### Real-World Cases

- **Default settings:** A "Standard shipping" radio button is pre-selected in a checkout form.
- **Opt-in defaults:** A "No" radio button is selected by default for sensitive choices.
- **Iconic options:** Radio buttons with icons for visual emphasis.

---

## 3. Button Group — ButtonGroup

### Definitions

**Core Definition.** `ButtonGroup` is a class that creates a multiple-exclusion scope for a set of buttons, ensuring that only one button in the group can be selected at a time.

**Technical Definition.** `public class ButtonGroup`. This class is used to create a multiple-exclusion scope for a set of buttons. Creating a set of buttons with the same `ButtonGroup` object means that turning "on" one of those buttons turns off all other buttons in the group. Initially, all buttons in the group are unselected. You can use `ButtonGroup` with any set of objects that inherit from `AbstractButton`—typically `JRadioButton`, `JRadioButtonMenuItem`, or `JToggleButton`.

**Beginner-Friendly Explanation.** `ButtonGroup` is the “referee” that enforces the rule that only one radio button can be selected. You create a group, add your radio buttons to it, and the group automatically deselects the previous button when a new one is selected.

---

### Purposes

- To enforce mutual exclusivity among a set of radio buttons.
- To manage the selected/unselected state centrally.
- To allow programmatic access to the selected button via `getSelection()`.
- To provide a `clearSelection()` method to deselect all buttons.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
ButtonGroup group = new ButtonGroup();
group.add(radioButton1);
group.add(radioButton2);
group.add(radioButton3);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `ButtonGroup()` | Creates a new group. |
| `add(AbstractButton)` | Adds a button to the group. |
| `remove(AbstractButton)` | Removes a button from the group. |
| `getSelection()` | Returns the selected `ButtonModel`, or `null` if none. |
| `clearSelection()` | Clears the selection of all buttons in the group. |

#### Syntax Rules

- The group is a logical grouping, not a physical one; you still need a `JPanel` for visual grouping.
- Adding a button to a group removes it from any previous group.
- Initially, all buttons in the group are unselected.

#### Constraints and Limitations

- `ButtonGroup` does not affect layout or appearance; it only manages selection.
- To create the appearance of “none selected,” add an invisible radio button to the group and programmatically select it to turn off all displayed radio buttons.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: ButtonGroup with Programmatic Selection

```java
import javax.swing.*;
import java.awt.*;

public class ButtonGroupDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ButtonGroup Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton red = new JRadioButton("Red");
            JRadioButton green = new JRadioButton("Green");
            JRadioButton blue = new JRadioButton("Blue");

            ButtonGroup group = new ButtonGroup();                    // 1. Create group
            group.add(red);
            group.add(green);
            group.add(blue);

            red.setSelected(true);                                    // 2. Default

            JButton clear = new JButton("Clear Selection");
            clear.addActionListener(e -> group.clearSelection());    // 3. Clear

            JPanel panel = new JPanel();
            panel.add(red);
            panel.add(green);
            panel.add(blue);
            panel.add(clear);

            frame.add(panel);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three color radio buttons with "Red" selected by default. Clicking "Clear Selection" deselects all buttons.

**Why the code produces this result.** The `ButtonGroup` manages the selection, and `clearSelection()` deselects all buttons in the group.

---

### Real-World Cases

- **Preference dialogs** with mutually exclusive options.
- **Payment method selection** (Credit Card, PayPal, Bank Transfer).
- **Questionnaires** with single-choice answers.

---

## 4. Radio Button Methods — isSelected()

### Definitions

**Core Definition.** `isSelected()` returns the current selection state of the radio button as a boolean.

**Technical Definition.** `public boolean isSelected()` is inherited from `AbstractButton`. It queries the `ButtonModel` and returns `true` if the radio button is selected, `false` otherwise.

**Beginner-Friendly Explanation.** `isSelected()` tells you whether the radio button is currently chosen. You call it to read the user's choice.

---

### Purposes

- To read the user's selection when a form is submitted.
- To branch logic based on which option is selected.
- To synchronize the UI state with a domain model.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
if (rb.isSelected()) {
    // option is selected
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `isSelected()` | Returns `true` if selected. |

#### Syntax Rules

- Safe to call at any time on the EDT.
- Returns the model's current state.

#### Constraints and Limitations

- Does not indicate whether the user has interacted; only the current state.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading Selection State

```java
import javax.swing.*;

public class IsSelectedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("isSelected Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton yes = new JRadioButton("Yes", true);
            JRadioButton no = new JRadioButton("No");
            ButtonGroup group = new ButtonGroup();
            group.add(yes);
            group.add(no);

            JButton submit = new JButton("Submit");
            submit.addActionListener(e -> {
                if (yes.isSelected()) {                               // 1. Read state
                    System.out.println("User selected Yes");
                } else if (no.isSelected()) {
                    System.out.println("User selected No");
                }
            });

            JPanel panel = new JPanel();
            panel.add(yes);
            panel.add(no);
            panel.add(submit);

            frame.add(panel);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking "Submit" prints "User selected Yes" or "User selected No" depending on the selected option.

**Why the code produces this result.** `isSelected()` reads the current state of each radio button.

---

### Real-World Cases

- **Form submission:** Checking which option the user chose.
- **Conditional enabling:** Showing additional fields based on the selected option.
- **Analytics:** Tracking which options are most frequently selected.

---

## 5. Radio Button Methods — setSelected()

### Definitions

**Core Definition.** `setSelected(boolean b)` programmatically selects or deselects the radio button.

**Technical Definition.** `public void setSelected(boolean b)` is inherited from `AbstractButton`. It updates the `ButtonModel`'s selected state. If the button belongs to a `ButtonGroup`, selecting it automatically deselects the previously selected button in the group.

**Beginner-Friendly Explanation.** `setSelected(true)` selects the radio button; `setSelected(false)` deselects it. When you select one button in a group, the group automatically deselects the others.

---

### Purposes

- To initialize the default selection.
- To respond to changes in a domain model.
- To reset or clear selections programmatically.
- To synchronize the UI with stored preferences.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
rb.setSelected(true);   // select
rb.setSelected(false);  // deselect
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setSelected(boolean)` | Sets the selected state. |

#### Syntax Rules

- Selecting a button in a `ButtonGroup` deselects the previously selected button.
- The initial state should be set before attaching listeners to avoid premature logic execution.

#### Constraints and Limitations

- Programmatic selection is still subject to the group's rules.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting Default Selection

```java
import javax.swing.*;

public class SetSelectedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setSelected Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton optionA = new JRadioButton("Option A");
            JRadioButton optionB = new JRadioButton("Option B");
            ButtonGroup group = new ButtonGroup();
            group.add(optionA);
            group.add(optionB);

            optionA.setSelected(true);                               // 1. Default

            JButton selectB = new JButton("Select B");
            selectB.addActionListener(e -> optionB.setSelected(true)); // 2. Programmatic

            JPanel panel = new JPanel();
            panel.add(optionA);
            panel.add(optionB);
            panel.add(selectB);

            frame.add(panel);
            frame.setSize(350, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Option A is selected by default. Clicking "Select B" selects Option B and deselects Option A.

**Why the code produces this result.** `setSelected` changes the state, and the `ButtonGroup` enforces exclusivity.

---

### Real-World Cases

- **Loading preferences:** Setting the saved option as selected when the dialog opens.
- **Resetting forms:** Restoring the default selection.
- **Model synchronization:** Updating the UI when the underlying data changes.

---

## 6. Radio Button Methods — setText()

### Definitions

**Core Definition.** `setText(String text)` changes the label displayed next to the radio button.

**Technical Definition.** `public void setText(String text)` is inherited from `AbstractButton`. It updates the label and triggers a revalidation of the component.

**Beginner-Friendly Explanation.** `setText` changes the radio button's label. Use it when the label needs to reflect changing context.

---

### Purposes

- To update the label dynamically based on application state.
- To support localization and internationalization.
- To correct or clarify the option's description.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
rb.setText("Updated label");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setText(String)` | Sets the label text. |

#### Syntax Rules

- The text can be changed at any time on the EDT.

#### Constraints and Limitations

- The label should remain meaningful and unambiguous.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Label Update

```java
import javax.swing.*;

public class SetTextDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setText Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton rb = new JRadioButton("Old label");
            JButton rename = new JButton("Rename");

            rename.addActionListener(e ->
                rb.setText("New label"));                             // 1. Update

            JPanel panel = new JPanel();
            panel.add(rb);
            panel.add(rename);

            frame.add(panel);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking "Rename" changes the radio button label to "New label."

**Why the code produces this result.** `setText` replaces the label with the new string.

---

### Real-World Cases

- **Localization:** Swapping labels when the user changes the language.
- **Contextual labels:** Changing the label to reflect the current mode.

---

## 7. Radio Button Events — ActionListener

### Definitions

**Core Definition.** `ActionListener` is an interface for receiving `ActionEvent`s that fire when the user activates the radio button (by clicking it), even if it is already selected.

**Technical Definition.** `public interface ActionListener extends EventListener`. The `actionPerformed(ActionEvent e)` method is invoked when the radio button fires an action event. An `ActionListener` on a `JRadioButton` will fire every time a user clicks on it, even if it is already selected.

**Beginner-Friendly Explanation.** An `ActionListener` runs every time the user clicks the radio button. If the button is already selected, clicking it again still fires the action event.

---

### Purposes

- To react to every user click on the radio button.
- To perform actions that should happen on every activation (e.g., re-triggering a preview).
- To distinguish user-initiated clicks from programmatic changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
rb.addActionListener(e -> {
    // user clicked the radio button
});
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addActionListener(ActionListener)` | Registers the listener. |
| `actionPerformed(ActionEvent e)` | Called on user click. |

#### Syntax Rules

- The listener is invoked on the EDT.
- Fires on every click, even if the button is already selected.

#### Constraints and Limitations

- Does not indicate whether the state actually changed; use `ItemListener` for state-change semantics.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: ActionListener on Radio Button

```java
import javax.swing.*;
import java.awt.event.*;

public class ActionListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ActionListener Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton rb = new JRadioButton("Click me");
            rb.addActionListener(e ->
                System.out.println("Clicked (selected=" + rb.isSelected() + ")")); // 1. On click

            frame.add(rb);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the radio button prints "Clicked (selected=true)" every time, even if it is already selected.

**Why the code produces this result.** `ActionListener` fires on every user click, regardless of the selection state.

---

### Real-World Cases

- **Audit logging:** Logging that the user interacted with a specific option.
- **Immediate preview:** Re-running a preview action when the user clicks an option.

---

## 8. Radio Button Events — ItemListener

### Definitions

**Core Definition.** `ItemListener` is an interface for receiving `ItemEvent`s that fire when the radio button's selection state changes—whether the change is user-initiated or programmatic.

**Technical Definition.** `public interface ItemListener extends EventListener`. The `itemStateChanged(ItemEvent e)` method is invoked when the item's state changes. `ItemEvent.getStateChange()` returns `ItemEvent.SELECTED` or `ItemEvent.DESELECTED`. An `ItemListener` fires **only** if the selected or deselected state changes. This is the preferred listener for detecting state changes on radio buttons.

**Beginner-Friendly Explanation.** An `ItemListener` runs when the radio button becomes selected or deselected. It does not run if the button is already selected and you click it again. It tells you the new state.

---

### Purposes

- To react to selection state changes accurately.
- To enable or disable dependent components based on the selected option.
- To synchronize the UI with a domain model.
- To implement logic that depends on which option is selected.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
rb.addItemListener(e -> {
    if (e.getStateChange() == ItemEvent.SELECTED) {
        // option was selected
    } else {
        // option was deselected
    }
});
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addItemListener(ItemListener)` | Registers the listener. |
| `itemStateChanged(ItemEvent e)` | Called on state change. |
| `e.getStateChange()` | Returns `SELECTED` or `DESELECTED`. |

#### Syntax Rules

- Fires only when the state changes (unlike `ActionListener`).
- The listener is invoked on the EDT.

#### Constraints and Limitations

- The event fires for both user-initiated and programmatic changes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: ItemListener for Selection Change

```java
import javax.swing.*;
import java.awt.event.*;

public class ItemListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ItemListener Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton yes = new JRadioButton("Yes");
            JRadioButton no = new JRadioButton("No", true);
            ButtonGroup group = new ButtonGroup();
            group.add(yes);
            group.add(no);

            JLabel status = new JLabel("Selected: No");

            ItemListener listener = e -> {                            // 1. Shared listener
                if (e.getStateChange() == ItemEvent.SELECTED) {
                    JRadioButton source = (JRadioButton) e.getSource();
                    status.setText("Selected: " + source.getText());
                }
            };
            yes.addItemListener(listener);                             // 2. Register
            no.addItemListener(listener);

            JPanel panel = new JPanel();
            panel.add(yes);
            panel.add(no);
            panel.add(status);

            frame.add(panel);
            frame.setSize(400, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The status label shows "Selected: No" initially. Clicking "Yes" updates it to "Selected: Yes."

**Why the code produces this result.** The shared `ItemListener` reacts only when a button becomes selected, using `getStateChange() == ItemEvent.SELECTED`.

---

### Real-World Cases

- **Dependent UI:** Enabling a text field when a specific radio button is selected.
- **Settings synchronization:** Updating a configuration object when the selection changes.
- **Form validation:** Checking that a required option has been selected.

---

## 9. JRadioButton Best Practices

### Definitions

**Core Definition.** `JRadioButton` best practices are the conventions and patterns that ensure radio buttons are used effectively, accessibly, and maintainably in Swing applications.

**Technical Definition.** Best practices include: always using `ButtonGroup` for mutually exclusive choices; using `ItemListener` (not `ActionListener`) to detect state changes; setting an initial selection; using meaningful labels; grouping visually with `JPanel` and `Border`; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for radio buttons are the “rules of thumb” that prevent common mistakes: forgetting the `ButtonGroup`, using the wrong listener, or leaving the user without a default choice. They include things like “always group your radio buttons,” “use `ItemListener` to detect changes,” and “set a default selection.”

---

### Purposes

- To enforce single-selection semantics via `ButtonGroup`.
- To use the correct listener (`ItemListener`) for state-change detection.
- To provide a clear default selection.
- To create accessible, well-labeled options.
- To maintain thread safety by respecting the EDT.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JRadioButton rb1 = new JRadioButton("Option A");
    JRadioButton rb2 = new JRadioButton("Option B");
    rb1.setSelected(true);                                        // 1. Default

    ButtonGroup group = new ButtonGroup();                        // 2. Group
    group.add(rb1);
    group.add(rb2);

    rb1.addItemListener(e -> {                                    // 3. ItemListener
        if (e.getStateChange() == ItemEvent.SELECTED) {
            // react to selection
        }
    });

    JPanel panel = new JPanel(new GridLayout(2, 1));              // 4. Visual group
    panel.setBorder(BorderFactory.createTitledBorder("Options"));
    panel.add(rb1);
    panel.add(rb2);
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Always use `ButtonGroup` | `group.add(rb)` for each radio button. |
| Set default selection | `rb.setSelected(true)` before attaching listeners. |
| Use `ItemListener` | Not `ActionListener`, for state-change detection. |
| Avoid both listeners | Do not register `ActionListener` and `ItemListener` on the same button. |
| Meaningful labels | Descriptive text; avoid ambiguity. |
| Visual grouping | `JPanel` with `TitledBorder` for related options. |
| Consider alternatives | Use `JCheckBox` for multiple selection; `JComboBox`/`JList` for many options. |
| EDT | `SwingUtilities.invokeLater(...)`. |

#### Syntax Rules

- **Always use a `ButtonGroup`** for radio buttons that are meant to be mutually exclusive.
- **Always set an initial selection** unless the workflow requires a null choice.
- **Use `ItemListener`**, not `ActionListener`, to detect state changes.
- **Never register both listeners** on the same radio button.

#### Constraints and Limitations

- A `ButtonGroup` is logical, not physical; use a `JPanel` for visual grouping.
- Radio buttons are not suitable for many options (10+); use a combo box or list instead.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Radio Button Group

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ItemEvent;

public class BestPracticeRadioButtons {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeRadioButtons::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Radio Buttons");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JRadioButton light = new JRadioButton("Light theme");     // 1. Create
        JRadioButton dark = new JRadioButton("Dark theme");
        dark.setSelected(true);                                    // 2. Default

        ButtonGroup group = new ButtonGroup();                     // 3. Group
        group.add(light);
        group.add(dark);

        JLabel status = new JLabel("Theme: Dark");

        light.addItemListener(e -> {                               // 4. ItemListener
            if (e.getStateChange() == ItemEvent.SELECTED) {
                status.setText("Theme: Light");
            }
        });
        dark.addItemListener(e -> {
            if (e.getStateChange() == ItemEvent.SELECTED) {
                status.setText("Theme: Dark");
            }
        });

        JPanel panel = new JPanel(new GridLayout(3, 1, 5, 5));     // 5. Visual group
        panel.setBorder(BorderFactory.createTitledBorder("Appearance"));
        panel.add(light);
        panel.add(dark);
        panel.add(status);

        frame.add(panel);
        frame.setSize(300, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A titled panel "Appearance" with two radio buttons: "Light theme" (unselected) and "Dark theme" (selected by default). The status label shows "Theme: Dark." Clicking "Light theme" updates the label to "Theme: Light."

**Why the code produces this result.** The `ButtonGroup` enforces exclusivity, the default is set, and `ItemListener`s react only to the selected state change.

---

#### Example 2: Common Mistake — Forgetting the ButtonGroup

```java
import javax.swing.*;

public class BadRadioButtons {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Radio Buttons");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JRadioButton a = new JRadioButton("A");
            JRadioButton b = new JRadioButton("B");
            JRadioButton c = new JRadioButton("C");

            // WRONG: no ButtonGroup — all three can be selected independently

            JPanel panel = new JPanel();
            panel.add(a);
            panel.add(b);
            panel.add(c);

            frame.add(panel);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** All three radio buttons can be selected simultaneously, violating the expected single-selection semantics.

**Why this is wrong.** Without a `ButtonGroup`, each `JRadioButton` acts as an independent toggle (like a checkbox). The group is what enforces mutual exclusivity.

---

### Real-World Cases

- **Production Swing forms** use `ButtonGroup` + `ItemListener` for every mutually exclusive option set.
- **Settings dialogs** set a default selection and react to changes with `ItemListener`.
- **Educational examples** use the best-practice skeleton to demonstrate correct `JRadioButton` usage from the start.

---

## References

Oracle. *Class JRadioButton*. Java Platform, Standard Edition 27 API Specification. https://docs.oracle.com/en/java/javase/27/docs/api/java.desktop/javax/swing/JRadioButton.html

Oracle. *How to Use the ButtonGroup Component*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/buttongroup.html

Oracle. *How to Use Buttons, Check Boxes, and Radio Buttons*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/button.html

Cleverence. *JRadioButton in Java SE 8: Complete Guide with Examples and Best Practices*. https://www.cleverence.com/articles/oracle-documentation/jradiobutton-java-platform-se-8-5381/

Stack Overflow. *JRadioButton ActionListener vs ItemListener*. https://stackoverflow.com/questions/9882845/jcheckbox-actionlistener-and-itemlistener

Swing JRadioButton API Specification. https://www.cs.vu.nl/~eliens/afstuderen/media/@archive/online/online/archive/one/swing-0.3/doc/radio.html