# JLabel — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JLabel` is a Swing component that provides a display area for a short text string, an image, or both. It is a non-interactive component that does not react to input events and therefore cannot receive keyboard focus by default.

**Technical Definition.** `public class JLabel extends JComponent implements SwingConstants, Accessible`. A `JLabel` can display text, an icon, or both, and supports horizontal and vertical alignment of its contents within its display area. By default, labels are vertically centered; text-only labels are aligned to the leading edge, and image-only labels are horizontally centered. The relative position of text to icon can also be controlled, with the default being text on the trailing edge of the icon, vertically aligned. The `setIconTextGap` method specifies the pixel distance between text and icon (default 4 pixels). Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `JLabel` is a piece of static text or a small image on the screen. It is used to tell the user what something is—like “Name:”, “Password:”, or “Status:”—or to show a result. Unlike a `JTextField`, the user cannot type into a `JLabel`; it is purely for display. You use it whenever you need to put a label next to an input field, show a status message, or display an icon.

---

### Key Characteristics

1. **Non-interactive display.** `JLabel` does not react to input events and cannot receive keyboard focus. For interactive components, use `JButton`, `JTextField`, or other interactive controls.
2. **Text, icon, or both.** A label can display a short text string, an image (via `Icon`), or both simultaneously.
3. **Alignment control.** Horizontal and vertical alignment of the label’s contents within its display area can be configured using `SwingConstants` values.
4. **Mnemonic support.** A keyboard mnemonic can be displayed on the label to indicate a shortcut for an associated component, typically a text field.
5. **Accessibility integration.** The `setLabelFor` method associates a label with another component, allowing assistive technologies to inherit the label’s text as the component’s accessible name.
6. **Transparency by default.** `JLabel` is not opaque by default. To paint a background color, `setOpaque(true)` must be called explicitly.
7. **Single-thread rule and serialization caveat.** `JLabel` is not thread-safe and must be accessed on the EDT. Serialized `JLabel` objects are not guaranteed to be compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, and the `public static void main(String[] args)` entry point.
- Familiarity with Swing containers such as `JFrame`, `JPanel`, and layout managers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `Icon` and `ImageIcon` for displaying images.
- A JDK (version 8 or later; `JLabel` is available in all modern JDKs).

---

### Related Programming Areas

- **`JTextField` and `JPasswordField`:** Labels are commonly associated with these input components via `setLabelFor` to provide accessible names and mnemonics.
- **`AbstractButton`:** For interactive components that display text and icons but also respond to clicks, use `JButton` instead of `JLabel`.
- **HTML in Swing:** Labels support HTML formatting for multi-line text, multiple fonts, and colors.
- **Accessibility API:** `JLabel` implements `Accessible`, and its `AccessibleContext` is used by assistive technologies.
- **Layout managers:** `FlowLayout`, `GridBagLayout`, and others arrange labels alongside other components.

---

## Core Concepts and Features

---

## 1. JLabel — The Class

### Definitions

**Core Definition.** `JLabel` is the Swing class for creating non-interactive display areas for text, images, or both.

**Technical Definition.** `public class JLabel extends JComponent implements SwingConstants, Accessible`. The class provides constructors that accept text, an icon, and horizontal alignment. The label’s UI delegate renders its content, and the label’s preferred size is based on its text and icon content.

**Beginner-Friendly Explanation.** `JLabel` is the class you use to create a label. You create it with `new JLabel("Name:")`, optionally add an icon, and place it next to the component it describes.

---

### Purposes

- To display descriptive text near an input component, such as a field label.
- To show an image or icon that conveys information or decoration.
- To present status messages, instructions, or results that the user should read but not edit.
- To provide an accessible name for another component via `setLabelFor`.
- To combine text and icons in a single aligned display area.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JLabel label = new JLabel("Text");
JLabel label = new JLabel("Text", SwingConstants.CENTER);
JLabel label = new JLabel(icon);
JLabel label = new JLabel("Text", icon, SwingConstants.CENTER);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JLabel()` | Creates an empty label with no text or icon. |
| `JLabel(String text)` | Creates a label with the specified text. |
| `JLabel(String text, int horizontalAlignment)` | Creates a label with text and the specified horizontal alignment. |
| `JLabel(Icon image)` | Creates a label with the specified image. |
| `JLabel(String text, Icon icon, int horizontalAlignment)` | Creates a label with text, icon, and horizontal alignment. |

#### Syntax Rules

- The label must be added to a container to be displayed.
- The default horizontal alignment is `LEADING` for text-only labels and `CENTER` for image-only labels.
- Construction must occur on the EDT.

#### Constraints and Limitations

- `JLabel` does not support user input; for editable text, use `JTextField` or `JTextArea`.
- Labels are not designed to act as containers for other components; their preferred size is based on text and icon only, not on child components.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JLabel

```java
import javax.swing.*;
import java.awt.*;

public class MinimalLabel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JLabel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel label = new JLabel("Hello, JLabel!");              // 1. Create label
            label.setHorizontalAlignment(SwingConstants.CENTER);      // 2. Center text

            frame.add(label);                                          // 3. Add to frame
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with the text “Hello, JLabel!” centered horizontally.

**Why the code produces this result.** The label is created with the specified text. `setHorizontalAlignment(SwingConstants.CENTER)` centers the text within the label’s display area. The label is added to the frame’s content pane.

---

#### Example 2: Label with Icon and Text

```java
import javax.swing.*;
import java.awt.*;

public class IconLabel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Icon Label");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            ImageIcon icon = new ImageIcon(new java.awt.image.BufferedImage(
                32, 32, java.awt.image.BufferedImage.TYPE_INT_RGB));

            JLabel label = new JLabel("Status: OK", icon, SwingConstants.CENTER); // 1. Text + icon
            label.setVerticalTextPosition(SwingConstants.BOTTOM);      // 2. Text below icon
            label.setHorizontalTextPosition(SwingConstants.CENTER);    // 3. Text centered

            frame.add(label);
            frame.setSize(300, 180);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window showing a 32×32 icon with the text “Status: OK” centered below it.

**Why the code produces this result.** The constructor accepts text, an icon, and horizontal alignment. `setVerticalTextPosition(BOTTOM)` places the text below the icon, and `setHorizontalTextPosition(CENTER)` centers the text horizontally relative to the icon.

---

### Real-World Cases

- **Form field labels.** Every text field in a data-entry form is typically preceded by a `JLabel` describing its purpose (“Username:”, “Email:”, “Password:”).
- **Status bars.** A `JLabel` at the bottom of a window displays messages such as “Ready,” “Saving…,” or “Connection lost.”
- **Image display.** A `JLabel` with an `ImageIcon` is the simplest way to show a logo, photograph, or diagram.
- **Result output.** After a calculation, a `JLabel` can display the computed value or a message.

---

## 2. Label Creation

### Definitions

**Core Definition.** Label creation is the process of instantiating a `JLabel` with the desired text, icon, and horizontal alignment, and preparing it for addition to a container.

**Technical Definition.** `JLabel` provides multiple constructors that accept combinations of text, icon, and horizontal alignment. After construction, the label’s properties can be further customized with alignment, mnemonic, tooltip, and label-for methods.

**Beginner-Friendly Explanation.** Creating a label means deciding what it should say or show (text, icon, or both) and how its content should be aligned. You then configure it—give it a mnemonic, associate it with a text field—and add it to your window.

---

### Purposes

- To instantiate a label with the appropriate visual content.
- To prepare the label for association with an input component.
- To establish the label’s initial alignment and orientation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JLabel label = new JLabel("Name:");
JLabel label = new JLabel("Name:", SwingConstants.RIGHT);
JLabel label = new JLabel(icon);
JLabel label = new JLabel("Name:", icon, SwingConstants.LEFT);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `String text` | The text to display. |
| `Icon image` | The icon to display. |
| `int horizontalAlignment` | One of `SwingConstants.LEFT`, `CENTER`, `RIGHT`, `LEADING`, or `TRAILING`. |

#### Syntax Rules

- If no horizontal alignment is specified, text-only labels default to `LEADING` and image-only labels default to `CENTER`.
- The label’s text and icon can be changed later with `setText` and `setIcon`.

#### Constraints and Limitations

- A label with neither text nor icon will display nothing.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating Labels with Different Alignments

```java
import javax.swing.*;
import java.awt.*;

public class LabelAlignments {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Label Alignments");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new GridLayout(3, 1, 5, 5));

            JLabel left = new JLabel("Left", SwingConstants.LEFT);     // 1. LEFT
            JLabel center = new JLabel("Center", SwingConstants.CENTER); // 2. CENTER
            JLabel right = new JLabel("Right", SwingConstants.RIGHT);  // 3. RIGHT

            frame.add(left);
            frame.add(center);
            frame.add(right);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three labels in a 3×1 grid, with text aligned left, center, and right, respectively.

**Why the code produces this result.** Each constructor specifies a different horizontal alignment constant, positioning the text within the label’s display area.

---

### Real-World Cases

- **Form labels** aligned to the right so they sit close to their text fields.
- **Status labels** centered at the bottom of a window.
- **Icon labels** with centered images.

---

## 3. Label Properties — Text

### Definitions

**Core Definition.** The text property is the string displayed by the label. It can be changed at runtime with `setText`.

**Technical Definition.** The text is managed by `setText(String)` and `getText()`. The label’s preferred size and layout are based on the text’s font metrics. HTML can be used within the text to provide limited formatting, such as multiple lines, fonts, and colors.

**Beginner-Friendly Explanation.** The text is what the label says. You can change it whenever you need to update a status message or show a different result.

---

### Purposes

- To display descriptive or status text.
- To update the label’s message dynamically.
- To provide accessible text for an associated component.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setText("New text");
String current = label.getText();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setText(String)` | Sets the label’s text. `null` clears the text. |
| `getText()` | Returns the current text. |

#### Syntax Rules

- The text can be changed at any time.
- HTML formatting is supported if the text begins with `<html>`.

#### Constraints and Limitations

- Very long text may be truncated or cause layout issues unless the layout manager handles it.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Text Update

```java
import javax.swing.*;
import java.awt.*;

public class TextUpdate {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Text Update");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel status = new JLabel("Status: Idle");                // 1. Initial text
            JButton update = new JButton("Update");

            update.addActionListener(e ->
                status.setText("Status: Running"));                   // 2. Update text

            frame.setLayout(new FlowLayout());
            frame.add(status);
            frame.add(update);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label initially shows “Status: Idle.” Clicking “Update” changes it to “Status: Running.”

**Why the code produces this result.** `setText` replaces the label’s text, and the label repaints with the new string.

---

### Real-World Cases

- **Status indicators** that change from “Loading…” to “Done.”
- **Result displays** that show computed values after a calculation.
- **Dynamic form validation messages** (“Username is available”).

---

## 4. Label Properties — Icon

### Definitions

**Core Definition.** The icon property is the image displayed by the label. It can be set at construction or changed later with `setIcon`.

**Technical Definition.** The icon is managed by `setIcon(Icon)` and `getIcon()`. The label’s preferred size accounts for the icon’s dimensions, and the relative position of text and icon can be controlled with `setHorizontalTextPosition` and `setVerticalTextPosition`.

**Beginner-Friendly Explanation.** The icon is the small picture on the label—a logo, a status symbol, or an illustration.

---

### Purposes

- To provide a visual indicator or decoration.
- To display an image without needing a separate component.
- To combine icon and text for a richer label.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setIcon(icon);
label.setIcon(null); // remove icon
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setIcon(Icon)` | Sets the label’s icon. `null` removes it. |

#### Syntax Rules

- The icon is displayed according to the label’s text-position properties.

#### Constraints and Limitations

- The icon’s size affects the label’s preferred size.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting and Removing an Icon

```java
import javax.swing.*;
import java.awt.*;

public class IconUpdate {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Icon Update");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            ImageIcon icon = new ImageIcon(new java.awt.image.BufferedImage(
                24, 24, java.awt.image.BufferedImage.TYPE_INT_RGB));

            JLabel label = new JLabel("No icon");                     // 1. Text only
            JButton toggle = new JButton("Toggle Icon");

            toggle.addActionListener(e -> {
                if (label.getIcon() == null) {
                    label.setIcon(icon);                               // 2. Set icon
                } else {
                    label.setIcon(null);                               // 3. Remove icon
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(label);
            frame.add(toggle);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label initially has no icon. Clicking “Toggle Icon” adds the icon; clicking again removes it.

**Why the code produces this result.** `getIcon()` returns `null` when no icon is set. The listener conditionally sets or clears the icon.

---

### Real-World Cases

- **Logo labels** in splash screens or about dialogs.
- **Status icons** (green check, red exclamation) next to status text.
- **Thumbnail previews** in file browsers.

---

## 5. Label Properties — Alignment

### Definitions

**Core Definition.** Alignment properties control where the label’s content (text and/or icon) is positioned within the label’s display area.

**Technical Definition.** Horizontal alignment is set via `setHorizontalAlignment(int)` using `SwingConstants.LEFT`, `CENTER`, `RIGHT`, `LEADING`, or `TRAILING`. Vertical alignment is set via `setVerticalAlignment(int)` using `SwingConstants.TOP`, `CENTER`, or `BOTTOM`. The default vertical alignment is `CENTER`. The default horizontal alignment is `LEADING` for text-only labels and `CENTER` for image-only labels.

**Beginner-Friendly Explanation.** Alignment controls whether the label’s text sits on the left, in the middle, or on the right (horizontal), and whether it sits at the top, middle, or bottom (vertical) within the space the label occupies.

---

### Purposes

- To position text and icons precisely within the label’s bounds.
- To align labels consistently with adjacent components in a form.
- To support right-to-left languages through `LEADING` and `TRAILING`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setHorizontalAlignment(SwingConstants.CENTER);
label.setVerticalAlignment(SwingConstants.TOP);
```

#### Component Breakdown

| Method | Valid Constants |
|--------|-----------------|
| `setHorizontalAlignment(int)` | `LEFT`, `CENTER`, `RIGHT`, `LEADING`, `TRAILING` |
| `setVerticalAlignment(int)` | `TOP`, `CENTER`, `BOTTOM` |

#### Syntax Rules

- The constants are defined in `SwingConstants`.
- Default vertical alignment is `CENTER`.

#### Constraints and Limitations

- The label must have extra space within its bounds for alignment to be visible.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Vertical Alignment Comparison

```java
import javax.swing.*;
import java.awt.*;

public class VerticalAlignmentDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Vertical Alignment");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new GridLayout(1, 3, 5, 5));

            JLabel top = new JLabel("Top", SwingConstants.CENTER);
            top.setVerticalAlignment(SwingConstants.TOP);              // 1. TOP

            JLabel center = new JLabel("Center", SwingConstants.CENTER);
            center.setVerticalAlignment(SwingConstants.CENTER);        // 2. CENTER

            JLabel bottom = new JLabel("Bottom", SwingConstants.CENTER);
            bottom.setVerticalAlignment(SwingConstants.BOTTOM);        // 3. BOTTOM

            frame.add(top);
            frame.add(center);
            frame.add(bottom);
            frame.setSize(450, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Three labels in a horizontal grid. “Top” is vertically aligned to the top of its area, “Center” is centered, and “Bottom” is aligned to the bottom.

**Why the code produces this result.** Each label’s vertical alignment is set explicitly, positioning the text within the label’s bounds.

---

### Real-World Cases

- **Form layouts** where labels should align with the top of multi-line text areas.
- **Icon-and-text labels** where the text should be centered relative to the icon.

---

## 6. Label Properties — Mnemonic

### Definitions

**Core Definition.** A mnemonic is a keyboard shortcut character that, when activated (typically with the Alt key), transfers focus to the component associated with the label.

**Technical Definition.** The mnemonic is set via `setDisplayedMnemonic(int)` using a `KeyEvent.VK_*` constant. The mnemonic character is underlined in the label’s text. For the mnemonic to function, the label must be associated with a component via `setLabelFor`.

**Beginner-Friendly Explanation.** A mnemonic is the underlined letter in a label like “**U**sername:”. Pressing Alt+U moves the cursor to the associated text field. It is a keyboard accessibility feature.

---

### Purposes

- To provide keyboard shortcuts for focusing input components.
- To improve accessibility for users who cannot use a mouse.
- To indicate the shortcut visually with an underlined character.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setDisplayedMnemonic(KeyEvent.VK_U);
label.setLabelFor(textField);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setDisplayedMnemonic(int)` | Sets the mnemonic key code. |
| `setLabelFor(Component)` | Associates the label with a component. |

#### Syntax Rules

- Both `setDisplayedMnemonic` and `setLabelFor` must be set for the mnemonic to transfer focus.
- The mnemonic character should appear in the label’s text (the first occurrence is underlined).

#### Constraints and Limitations

- The mnemonic does not work if `setLabelFor` is not set.
- The mnemonic character may conflict with other mnemonics in the same window.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Mnemonic with `setLabelFor`

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;

public class MnemonicDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Mnemonic Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JLabel nameLabel = new JLabel("Username: ");              // 1. Label
            JTextField nameField = new JTextField(15);                 // 2. Field

            nameLabel.setDisplayedMnemonic(KeyEvent.VK_U);             // 3. Mnemonic
            nameLabel.setLabelFor(nameField);                          // 4. Associate

            frame.add(nameLabel);
            frame.add(nameField);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label “Username:” displays the “U” underlined. Pressing Alt+U moves keyboard focus to the text field.

**Why the code produces this result.** `setDisplayedMnemonic(KeyEvent.VK_U)` underlines the “U,” and `setLabelFor(nameField)` tells the label which component should receive focus when the mnemonic is activated.

---

### Real-World Cases

- **Login forms** with Alt+U for username and Alt+P for password.
- **Data-entry forms** where every field has a mnemonic for keyboard navigation.

---

## 7. Label Methods — setText()

### Definitions

**Core Definition.** `setText(String text)` changes the text displayed by the label.

**Technical Definition.** `public void setText(String text)`. Passing `null` results in an empty label. The label’s preferred size is recalculated based on the new text’s font metrics.

**Beginner-Friendly Explanation.** `setText` updates what the label says. Use it to show dynamic status messages or results.

---

### Purposes

- To update the label’s text at runtime.
- To display changing status information.
- To clear the label’s text.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setText("New text");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setText(String)` | Sets the text. `null` clears it. |

#### Syntax Rules

- Must be called on the EDT.
- HTML formatting is supported if the string begins with `<html>`.

#### Constraints and Limitations

- Very long text may be truncated.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Status Update

```java
import javax.swing.*;
import java.awt.*;

public class SetTextDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetText Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel label = new JLabel("Waiting...");
            JButton done = new JButton("Done");

            done.addActionListener(e ->
                label.setText("Operation complete."));                 // 1. Update

            frame.setLayout(new FlowLayout());
            frame.add(label);
            frame.add(done);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label shows “Waiting...” initially. Clicking “Done” changes it to “Operation complete.”

**Why the code produces this result.** `setText` replaces the label’s text, and the label repaints with the new string.

---

### Real-World Cases

- **Progress indicators** that update from “Loading…” to “Ready.”
- **Validation messages** that appear after form submission.

---

## 8. Label Methods — setIcon()

### Definitions

**Core Definition.** `setIcon(Icon icon)` changes the icon displayed by the label.

**Technical Definition.** `public void setIcon(Icon icon)`. Passing `null` removes the icon. The label’s preferred size is recalculated based on the icon’s dimensions.

**Beginner-Friendly Explanation.** `setIcon` changes the picture on the label or removes it.

---

### Purposes

- To change the label’s visual indicator.
- To remove an icon when it is no longer relevant.
- To swap icons based on application state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setIcon(icon);
label.setIcon(null); // remove
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setIcon(Icon)` | Sets or clears the icon. |

#### Syntax Rules

- The icon’s size affects the label’s preferred size.

#### Constraints and Limitations

- Large icons can make the label very large.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Swapping Icons

```java
import javax.swing.*;
import java.awt.*;

public class SwapIconDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Swap Icon");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            ImageIcon onIcon = new ImageIcon(new java.awt.image.BufferedImage(
                16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB));
            ImageIcon offIcon = new ImageIcon(new java.awt.image.BufferedImage(
                16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB));

            JLabel label = new JLabel("Status", offIcon, SwingConstants.LEFT);
            JButton toggle = new JButton("Toggle");

            toggle.addActionListener(e -> {
                if (label.getIcon() == offIcon) {
                    label.setIcon(onIcon);                             // 1. On
                } else {
                    label.setIcon(offIcon);                            // 2. Off
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(label);
            frame.add(toggle);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label initially shows the “off” icon. Clicking “Toggle” switches to the “on” icon, and clicking again switches back.

**Why the code produces this result.** `getIcon()` identifies the current icon, and `setIcon` replaces it.

---

### Real-World Cases

- **Connection status** icons (connected/disconnected).
- **Message severity** icons (info, warning, error).

---

## 9. Label Methods — setHorizontalAlignment()

### Definitions

**Core Definition.** `setHorizontalAlignment(int alignment)` sets the horizontal alignment of the label’s content within its display area.

**Technical Definition.** `public void setHorizontalAlignment(int alignment)`. Valid constants are `SwingConstants.LEFT`, `CENTER`, `RIGHT`, `LEADING`, and `TRAILING`. The default is `LEADING` for text-only labels and `CENTER` for image-only labels.

**Beginner-Friendly Explanation.** `setHorizontalAlignment` decides whether the text sits on the left, in the middle, or on the right of the label.

---

### Purposes

- To align label text with adjacent components in a form.
- To center text or icons within the label.
- To support leading/trailing alignment for internationalization.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setHorizontalAlignment(SwingConstants.RIGHT);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `LEFT` | Align left |
| `CENTER` | Center horizontally |
| `RIGHT` | Align right |
| `LEADING` | Align to leading edge (default for text-only) |
| `TRAILING` | Align to trailing edge |

#### Syntax Rules

- The alignment is relative to the label’s display area.

#### Constraints and Limitations

- Requires extra space to be visible.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Right-Aligned Label

```java
import javax.swing.*;
import java.awt.*;

public class RightAlignDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Right Align");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel label = new JLabel("Name:");
            label.setHorizontalAlignment(SwingConstants.RIGHT);         // 1. RIGHT

            frame.add(label);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text “Name:” is aligned to the right side of the label’s area.

**Why the code produces this result.** `setHorizontalAlignment(SwingConstants.RIGHT)` positions the text at the right edge.

---

### Real-World Cases

- **Form labels** right-aligned so they sit close to the input fields.
- **Header labels** centered above content.

---

## 10. Label Methods — setVerticalAlignment()

### Definitions

**Core Definition.** `setVerticalAlignment(int alignment)` sets the vertical alignment of the label’s content within its display area.

**Technical Definition.** `public void setVerticalAlignment(int alignment)`. Valid constants are `SwingConstants.TOP`, `CENTER`, and `BOTTOM`. The default is `CENTER`.

**Beginner-Friendly Explanation.** `setVerticalAlignment` decides whether the text sits at the top, middle, or bottom of the label’s space.

---

### Purposes

- To align labels with the top of tall components.
- To center text vertically.
- To position text at the bottom of the label.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setVerticalAlignment(SwingConstants.TOP);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `TOP` | Align top |
| `CENTER` | Center vertically (default) |
| `BOTTOM` | Align bottom |

#### Syntax Rules

- The alignment is relative to the label’s display area.

#### Constraints and Limitations

- Requires extra vertical space to be visible.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Top-Aligned Label

```java
import javax.swing.*;
import java.awt.*;

public class TopAlignDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Top Align");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel label = new JLabel("Top");
            label.setVerticalAlignment(SwingConstants.TOP);             // 1. TOP
            label.setPreferredSize(new Dimension(150, 100));

            frame.add(label);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text “Top” is aligned to the top of the label’s 100-pixel-tall area.

**Why the code produces this result.** `setVerticalAlignment(SwingConstants.TOP)` positions the text at the top edge.

---

### Real-World Cases

- **Labels next to tall text areas** that should align with the top of the area.
- **Image labels** where text should appear at the bottom.

---

## 11. Label Methods — setDisplayedMnemonic()

### Definitions

**Core Definition.** `setDisplayedMnemonic(int key)` sets the mnemonic character that is displayed (underlined) on the label and used for keyboard activation.

**Technical Definition.** `public void setDisplayedMnemonic(int key)`. The key is typically a `KeyEvent.VK_*` constant. For the mnemonic to transfer focus, the label must be associated with a component via `setLabelFor`.

**Beginner-Friendly Explanation.** `setDisplayedMnemonic` chooses which letter in the label’s text becomes the keyboard shortcut.

---

### Purposes

- To define the keyboard shortcut character.
- To underline the shortcut character in the label’s text.
- To provide keyboard access to the associated component.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setDisplayedMnemonic(KeyEvent.VK_N);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setDisplayedMnemonic(int)` | Sets the mnemonic key code. |

#### Syntax Rules

- The mnemonic character should appear in the label’s text.
- Must be combined with `setLabelFor` for focus transfer.

#### Constraints and Limitations

- If the character does not appear in the text, no underline is shown.
- Mnemonic conflicts can occur within the same window.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Mnemonic Activation

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;

public class MnemonicActivation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Mnemonic Activation");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JLabel label = new JLabel("Name:");
            JTextField field = new JTextField(15);

            label.setDisplayedMnemonic(KeyEvent.VK_N);                 // 1. Mnemonic
            label.setLabelFor(field);                                   // 2. Associate

            frame.add(label);
            frame.add(field);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “N” in “Name:” is underlined. Pressing Alt+N moves focus to the text field.

**Why the code produces this result.** `setDisplayedMnemonic` underlines the character, and `setLabelFor` directs focus to the text field.

---

### Real-World Cases

- **Forms with keyboard shortcuts** for every field.
- **Dialogs** where Enter and mnemonics provide full keyboard navigation.

---

## 12. Label Methods — setLabelFor()

### Definitions

**Core Definition.** `setLabelFor(Component c)` associates the label with another component, enabling the mnemonic to transfer focus and allowing assistive technologies to inherit the label’s text as the component’s accessible name.

**Technical Definition.** `public void setLabelFor(Component c)`. When set, the associated component’s `AccessibleContext` inherits the label’s `AccessibleName` and `AccessibleDescription`. If both `displayedMnemonic` and `labelFor` are set, the label requests focus for the associated component when the mnemonic is activated.

**Beginner-Friendly Explanation.** `setLabelFor` tells the label which component it describes. This makes the mnemonic work and helps screen readers announce the label’s text when the component receives focus.

---

### Purposes

- To link a label to its corresponding input component.
- To enable mnemonic-based focus transfer.
- To provide an accessible name for the associated component.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
label.setLabelFor(component);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setLabelFor(Component)` | Associates the label with the given component. |

#### Syntax Rules

- The component can be any `Component`.
- The association is used by the accessibility API and by mnemonic handling.

#### Constraints and Limitations

- The label’s text is inherited as the accessible name only if the component does not already have one.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible Label Association

```java
import javax.swing.*;
import java.awt.*;

public class LabelForDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("LabelFor Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JLabel label = new JLabel("Email:");
            JTextField field = new JTextField(20);
            label.setLabelFor(field);                                   // 1. Associate

            frame.add(label);
            frame.add(field);
            frame.setSize(300, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label “Email:” is associated with the text field. Screen readers will announce “Email” when the field receives focus.

**Why the code produces this result.** `setLabelFor` establishes the association, and the accessibility API uses the label’s text as the field’s accessible name.

---

### Real-World Cases

- **Accessible forms** where every input has a properly associated label.
- **Keyboard-navigable interfaces** using mnemonics.

---

## 13. JLabel Best Practices

### Definitions

**Core Definition.** `JLabel` best practices are the conventions and patterns that ensure labels are used effectively, accessibly, and maintainably in Swing applications.

**Technical Definition.** Best practices include: using `setLabelFor` to associate labels with input components; providing mnemonics for keyboard accessibility; calling `setOpaque(true)` when setting a background color; using layout managers instead of absolute positioning; and keeping the EDT free of long-running tasks.

**Beginner-Friendly Explanation.** Best practices are the “rules of thumb” for labels. They include things like “always associate labels with their text fields,” “make the label opaque if you give it a background color,” and “don’t try to use a label as a container.”

---

### Purposes

- To ensure accessibility for screen readers and keyboard users.
- To avoid common painting issues with background colors.
- To maintain portable, layout-manager-based GUIs.
- To use the correct component for the task (label vs. button vs. panel).

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
JLabel label = new JLabel("Username:");
label.setDisplayedMnemonic(KeyEvent.VK_U);                         // 1. Mnemonic
label.setLabelFor(usernameField);                                  // 2. LabelFor
label.setOpaque(true);                                             // 3. Opaque if bg
label.setBackground(Color.WHITE);                                  // 4. Background
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Associate label with component | `label.setLabelFor(component)` |
| Provide mnemonic | `label.setDisplayedMnemonic(KeyEvent.VK_X)` |
| Make opaque for background | `label.setOpaque(true)` |
| Use layout managers | Avoid `null` layout and `setBounds` |
| Do not use as a container | Use `JPanel` for grouping |
| Do not do heavy work on EDT | Use `SwingWorker` for long tasks |

#### Syntax Rules

- **Always associate labels with their input components** using `setLabelFor` for accessibility and mnemonic support.
- **Always call `setOpaque(true)`** before setting a background color on a `JLabel`; otherwise, the background will not be painted.
- **Never use a `JLabel` as a container** for other components; its preferred size does not account for children.
- **Use layout managers** instead of absolute positioning.

#### Constraints and Limitations

- Labels are not interactive; do not use them where a button is needed.
- HTML formatting adds processing overhead; use `setFont`/`setForeground` for simple styling.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Form Label

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;

public class BestPracticeLabel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Label");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JLabel label = new JLabel("Username: ");
            JTextField field = new JTextField(15);

            label.setDisplayedMnemonic(KeyEvent.VK_U);                 // 1. Mnemonic
            label.setLabelFor(field);                                   // 2. Associate
            label.setOpaque(true);                                      // 3. Opaque
            label.setBackground(Color.WHITE);                           // 4. Background

            frame.add(label);
            frame.add(field);
            frame.setSize(350, 120);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A label “Username:” with the “U” underlined and a white background, associated with the text field. Pressing Alt+U focuses the field.

**Why the code produces this result.** `setDisplayedMnemonic` and `setLabelFor` enable the keyboard shortcut and accessibility. `setOpaque(true)` allows the white background to be painted.

---

#### Example 2: Common Mistake — Using JLabel as a Container

```java
import javax.swing.*;

public class BadLabelContainer {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Label Container");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel label = new JLabel("Container Label");
            label.add(new JButton("Child Button"));                    // WRONG: label as container

            frame.add(label);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The child button may not appear or may not be laid out correctly because `JLabel` is not designed to hold child components.

**Why this is wrong.** As the Java documentation and community guidelines state, `JLabel` is not built to act as a container for other components. Its preferred size is based on its text and icon, not on its children. Use a `JPanel` instead.

---

### Real-World Cases

- **Production Swing applications** use `setLabelFor` and mnemonics for every form label, ensuring accessibility and keyboard navigation.
- **Themed applications** set `setOpaque(true)` before applying background colors to labels.
- **Educational examples** use the best-practice skeleton to demonstrate correct `JLabel` usage from the start.

---

## References

Oracle. *Class JLabel*. Java Platform, Standard Edition 11 API Specification. https://docs.oracle.com/javase/jp/11/docs/api/java.desktop/javax/swing/JLabel.html 

Oracle. *How to Use Labels*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/label.html 

IBM. *IBM Human Ability and Accessibility Center: Developer guidelines for Java accessibility*. https://www.ibm.com/able/guidelines/java/accessjava.html 

Oracle. *Solving Common Painting Problems*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/painting/problems.html 

Stack Overflow. *JLabel is not a container*. https://stackoverflow.com/revisions/514473c6-0cfd-411a-be2f-346d8904bd21/view-source 

OpenJDK. *TextFieldMnemonic.java (JDK-8036819)*. https://bugs.openjdk.org/secure/attachment/19680/TextFieldMnemonic.java 