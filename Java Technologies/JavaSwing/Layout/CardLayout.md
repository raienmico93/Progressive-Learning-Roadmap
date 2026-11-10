# CardLayout — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `CardLayout` is a layout manager that treats each component in a container as a card in a stack. Only one card is visible at any time, and the container acts as a stack of cards. The first component added is the visible component when the container is first displayed. The ordering of cards is determined by the container's own internal ordering of its component objects, and `CardLayout` provides methods that allow an application to flip through these cards sequentially or to show a specified card.

**Technical Definition.** `public class CardLayout extends Object implements LayoutManager2, Serializable`. `CardLayout` manages two or more components (usually `JPanel` instances) that share the same display space. Each component added to the layout is associated with a string identifier (the "constraints" object), which can be used with the `show(Container parent, String name)` method for fast random access. The layout manager stores a `Vector` of `Card` objects, each pairing a component with its name, and maintains an index of the currently visible card. `CardLayout` does not lay components out side by side; instead, it manages visibility, ensuring that exactly one component is visible at a time, sized to fill the container's display area (minus insets and gaps). Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `CardLayout` is the "stack of cards" layout manager. Imagine a deck of playing cards where only the top card is visible. Each card is a full panel of your GUI—one for login, one for the main application, one for settings, and so on. You do not arrange these panels side by side; instead, you stack them in the same space and flip between them. `CardLayout` gives you the methods to flip: `first()` shows the first card, `next()` flips to the next card (wrapping around), `previous()` flips backward, `last()` shows the last card, and `show()` jumps directly to a card by name. It is the classic way to implement a "single-page, multiple-views" interface, such as a wizard, a login flow, or a tabbed interface without the tabs.

---

### Key Characteristics

1. **Single visible component.** Only one component is visible at a time; the container acts as a stack of cards. All other components are hidden.
2. **Named cards.** Every component added to a `CardLayout` container must be associated with a unique string identifier (the constraints object). This name is used by `show()` for direct navigation.
3. **Sequential and random access.** `first()`, `next()`, `previous()`, and `last()` navigate sequentially; `show()` jumps directly to a named card.
4. **Preferred size from all cards.** The preferred layout size is determined by taking the widest and tallest preferred sizes of all components (not necessarily from the same component), then adding insets and gaps. This means the container is sized to accommodate the largest card, even if that card is not currently visible.
5. **Gaps act as insets.** The `hgap` and `vgap` constructor parameters add space around the displayed component; with `CardLayout`, there is no space between components because only one is visible at a time—think of the gaps as insets.
6. **No visual navigation.** `CardLayout` provides no GUI for switching cards; the programmer must provide buttons, combo boxes, menus, or other controls to trigger the navigation methods.
7. **Ordering follows insertion.** The order of cards is determined by the order in which components were added to the container; `next()` and `previous()` cycle through this order.
8. **Single-thread rule and serialization caveat.** `CardLayout` is not thread-safe; construction and manipulation must occur on the EDT. Serialized `CardLayout` objects are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, and lambda expressions.
- Familiarity with `Container`, `Component`, `JPanel`, and `JFrame`.
- Understanding of `getPreferredSize` and component size hints.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `BorderLayout` and `FlowLayout` for nesting controls alongside the card container.
- A JDK (version 8 or later; `CardLayout` is in `java.awt`).

---

### Related Programming Areas

- **`LayoutManager2`:** The interface `CardLayout` implements; adds constraint-aware methods such as `addLayoutComponent(Component, Object)`.
- **`JTabbedPane`:** An alternative that provides built-in tab-based navigation for the same "shared space" use case.
- **`BorderLayout` and `FlowLayout`:** Commonly used to place navigation controls (buttons, combo boxes) around the card container.
- **`JPanel`:** The usual component type for individual cards.
- **`SwingWorker`:** For background tasks that may trigger card switches upon completion.
- **`revalidate()` and `repaint()`:** Essential after adding or removing cards dynamically.

---

## Core Concepts and Features

---

## 1. CardLayout — The Layout Manager

### Definitions

**Core Definition.** `CardLayout` is a layout manager that stacks components like cards, showing only one at a time and providing methods to flip between them.

**Technical Definition.** `public class CardLayout extends Object implements LayoutManager2, Serializable`. It maintains an internal `Vector` of `Card` objects (each pairing a `String` name with a `Component`) and an index of the currently visible card. The layout manager implements `addLayoutComponent(Component, Object)` (where the `Object` is the string name), `removeLayoutComponent`, `preferredLayoutSize`, `minimumLayoutSize`, and `layoutContainer`. It also implements the `LayoutManager2` methods `addLayoutComponent(Component, Object)`, `maximumLayoutSize`, `getLayoutAlignmentX`, `getLayoutAlignmentY`, and `invalidateLayout`.

**Beginner-Friendly Explanation.** `CardLayout` is the class you use to create a stacked-panel interface. You create it, install it on a container, add each panel with a unique name, and then call navigation methods to switch between them.

---

### Purposes

- To manage multiple components (usually panels) that share the same display space, showing only one at a time.
- To provide a simple mechanism for wizard-style flows, login screens, and multi-view interfaces.
- To avoid the overhead of adding and removing components when switching views; all cards remain in the container, only their visibility changes.
- To enable direct navigation by name via `show()`, as well as sequential navigation via `first()`, `next()`, `previous()`, and `last()`.
- To serve as the underlying layout for custom tab-like implementations when `JTabbedPane` is not desired.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation and Setup)

```java
CardLayout cardLayout = new CardLayout();
JPanel cardPanel = new JPanel(cardLayout);
cardPanel.add(panel1, "card1");
cardPanel.add(panel2, "card2");
cardLayout.show(cardPanel, "card2");
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `new CardLayout()` | Creates a card layout with zero gaps. |
| `new CardLayout(int hgap, int vgap)` | Creates a card layout with the specified gaps. |
| `container.add(component, "name")` | Adds a component as a card with the given name. |
| `cardLayout.show(container, "name")` | Shows the named card. |

#### Syntax Rules

- Every component added to a `CardLayout` container must be associated with a unique string name.
- The name is passed as the constraints object in the `add` method.
- The layout manager must be installed on the container before cards are added, or `revalidate()` must be called after setting it.
- Construction and modification should occur on the EDT.

#### Constraints and Limitations

- `CardLayout` provides no built-in navigation controls; the programmer must supply buttons, combo boxes, or other UI elements.
- The preferred size of the container is the maximum preferred size of all cards, not just the visible one; this can make the container larger than necessary for some cards.
- `CardLayout` does not support vertical writing modes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal CardLayout with Two Cards

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class MinimalCardLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal CardLayout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout cardLayout = new CardLayout();               // 1. Card layout
            JPanel cardPanel = new JPanel(cardLayout);               // 2. Container

            JPanel card1 = new JPanel();
            card1.add(new JLabel("This is Card 1"));
            card1.setBackground(Color.LIGHT_GRAY);

            JPanel card2 = new JPanel();
            card2.add(new JLabel("This is Card 2"));
            card2.setBackground(Color.CYAN);

            cardPanel.add(card1, "card1");                           // 3. Add cards
            cardPanel.add(card2, "card2");

            JButton nextButton = new JButton("Next");
            nextButton.addActionListener(e ->
                cardLayout.next(cardPanel));                          // 4. Navigate

            JPanel main = new JPanel(new BorderLayout());
            main.add(cardPanel, BorderLayout.CENTER);
            main.add(nextButton, BorderLayout.SOUTH);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window showing the first card (light gray) with a "Next" button at the bottom. Clicking "Next" switches to the second card (cyan); clicking again wraps back to the first card.

**Why the code produces this result.** The `CardLayout` is installed on `cardPanel`, and the two cards are added with unique names. `cardLayout.next(cardPanel)` advances to the next card in the insertion order, wrapping around when the end is reached.

---

### Real-World Cases

- **Login/registration flows** that switch between a login panel and a registration panel.
- **Wizard dialogs** that walk the user through a sequence of steps.
- **Dashboard views** that switch between different data visualisations without recreating panels.
- **Settings dialogs** that use a combo box or sidebar to switch between preference categories.

---

## 2. Card Layout Creation

### Definitions

**Core Definition.** Card layout creation is the process of instantiating a `CardLayout` with optional gaps and installing it on a container.

**Technical Definition.** `CardLayout` provides two constructors: `CardLayout()` with zero gaps, and `CardLayout(int hgap, int vgap)` with the specified horizontal and vertical gaps. The gaps are applied around the displayed component; with `CardLayout`, there is no space between components because only one is visible at a time—the gaps act as insets around the visible card.

**Beginner-Friendly Explanation.** Creating a `CardLayout` means deciding whether there should be padding around the visible card. You then install the layout on a panel and add cards to it.

---

### Purposes

- To instantiate the layout manager with appropriate gap settings.
- To install the manager on a container that will hold multiple cards.
- To prepare the container for card addition and navigation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
CardLayout layout = new CardLayout();
CardLayout layout = new CardLayout(int hgap, int vgap);
JPanel cardPanel = new JPanel(layout);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `CardLayout()` | Creates a card layout with zero gaps. |
| `CardLayout(int hgap, int vgap)` | Creates a card layout with the specified horizontal and vertical gaps. |

#### Syntax Rules

- Gaps may be zero or positive.
- The layout manager is installed on the container via the constructor or `setLayout`.

#### Constraints and Limitations

- Negative gaps are allowed but chop off components at the edges of the container; they are rarely useful.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating with Gaps

```java
import javax.swing.*;
import java.awt.*;

public class GappedCardLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Gapped CardLayout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout layout = new CardLayout(10, 10);              // 1. Gaps
            JPanel cardPanel = new JPanel(layout);                   // 2. Container

            JPanel card1 = new JPanel();
            card1.add(new JLabel("Card 1"));
            card1.setBackground(Color.PINK);

            JPanel card2 = new JPanel();
            card2.add(new JLabel("Card 2"));
            card2.setBackground(Color.ORANGE);

            cardPanel.add(card1, "one");
            cardPanel.add(card2, "two");

            JButton next = new JButton("Next");
            next.addActionListener(e -> layout.next(cardPanel));

            JPanel main = new JPanel(new BorderLayout());
            main.add(cardPanel, BorderLayout.CENTER);
            main.add(next, BorderLayout.SOUTH);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The visible card is inset by 10 pixels on all sides from the container's edges.

**Why the code produces this result.** The `CardLayout(10, 10)` constructor sets horizontal and vertical gaps of 10 pixels, which act as padding around the visible card.

---

### Real-World Cases

- **Padded card displays** where the card should not touch the container edges.
- **Zero-gap layouts** for a seamless card-to-container fit.

---

## 3. Card Layout Methods — first()

### Definitions

**Core Definition.** `first(Container parent)` flips to the first card in the container.

**Technical Definition.** `public void first(Container parent)`. The method sets the visible component to the first card in the container's internal ordering and hides all others.

**Beginner-Friendly Explanation.** `first()` jumps to the first card in the stack, regardless of which card is currently visible.

---

### Purposes

- To return to the first card (e.g., the start of a wizard or login flow).
- To reset the card sequence to the beginning.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
cardLayout.first(container);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `first(Container)` | Shows the first card. |

#### Syntax Rules

- The container must be the one managed by this `CardLayout`.

#### Constraints and Limitations

- If the container has no cards, the method has no effect.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Jump to First Card

```java
import javax.swing.*;
import java.awt.*;

public class FirstDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("first() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout layout = new CardLayout();
            JPanel cards = new JPanel(layout);

            for (int i = 1; i <= 3; i++) {
                JPanel p = new JPanel();
                p.add(new JLabel("Card " + i));
                p.setBackground(i == 1 ? Color.LIGHT_GRAY :
                    i == 2 ? Color.CYAN : Color.PINK);
                cards.add(p, "card" + i);
            }

            JButton first = new JButton("First");
            first.addActionListener(e -> layout.first(cards));       // 1. First
            JButton next = new JButton("Next");
            next.addActionListener(e -> layout.next(cards));

            JPanel controls = new JPanel();
            controls.add(first);
            controls.add(next);

            JPanel main = new JPanel(new BorderLayout());
            main.add(cards, BorderLayout.CENTER);
            main.add(controls, BorderLayout.SOUTH);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking "First" always returns to the first card, regardless of the current card.

**Why the code produces this result.** `layout.first(cards)` sets the visible card to the first in insertion order.

---

### Real-World Cases

- **Wizard "Start Over"** functionality.
- **Login flow "Back to Login"** button.

---

## 4. Card Layout Methods — next()

### Definitions

**Core Definition.** `next(Container parent)` flips to the next card in the container, wrapping around to the first card after the last.

**Technical Definition.** `public void next(Container parent)`. The method advances the visible card to the next card in the container's internal ordering; after the last card, it wraps to the first.

**Beginner-Friendly Explanation.** `next()` moves forward one card in the stack, cycling back to the beginning when it reaches the end.

---

### Purposes

- To advance through a sequence of cards (e.g., "Next" in a wizard).
- To cycle through cards in a loop.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
cardLayout.next(container);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `next(Container)` | Advances to the next card, wrapping. |

#### Syntax Rules

- The container must be the one managed by this `CardLayout`.

#### Constraints and Limitations

- With a single card, `next()` has no visible effect.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Next with Wrap-Around

```java
import javax.swing.*;
import java.awt.*;

public class NextDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("next() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout layout = new CardLayout();
            JPanel cards = new JPanel(layout);

            String[] names = {"one", "two", "three"};
            Color[] colors = {Color.LIGHT_GRAY, Color.CYAN, Color.PINK};
            for (int i = 0; i < names.length; i++) {
                JPanel p = new JPanel();
                p.add(new JLabel("Card: " + names[i]));
                p.setBackground(colors[i]);
                cards.add(p, names[i]);
            }

            JButton next = new JButton("Next");
            next.addActionListener(e -> layout.next(cards));          // 1. Next

            JPanel main = new JPanel(new BorderLayout());
            main.add(cards, BorderLayout.CENTER);
            main.add(next, BorderLayout.SOUTH);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Each click on "Next" advances to the next card; after the third card, it wraps back to the first.

**Why the code produces this result.** `layout.next(cards)` cycles forward through the insertion order, wrapping at the end.

---

### Real-World Cases

- **Wizard "Next"** buttons.
- **Carousel-style card viewers**.

---

## 5. Card Layout Methods — previous()

### Definitions

**Core Definition.** `previous(Container parent)` flips to the previous card in the container, wrapping around to the last card before the first.

**Technical Definition.** `public void previous(Container parent)`. The method moves the visible card backward one position in the container's internal ordering; before the first card, it wraps to the last.

**Beginner-Friendly Explanation.** `previous()` moves backward one card in the stack, cycling to the end when it reaches the beginning.

---

### Purposes

- To go back through a sequence of cards (e.g., "Back" in a wizard).
- To cycle backward through cards.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
cardLayout.previous(container);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `previous(Container)` | Moves to the previous card, wrapping. |

#### Syntax Rules

- The container must be the one managed by this `CardLayout`.

#### Constraints and Limitations

- With a single card, `previous()` has no visible effect.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Previous with Wrap-Around

```java
import javax.swing.*;
import java.awt.*;

public class PreviousDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("previous() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout layout = new CardLayout();
            JPanel cards = new JPanel(layout);

            for (int i = 1; i <= 3; i++) {
                JPanel p = new JPanel();
                p.add(new JLabel("Card " + i));
                p.setBackground(i == 1 ? Color.LIGHT_GRAY :
                    i == 2 ? Color.CYAN : Color.PINK);
                cards.add(p, "card" + i);
            }

            JButton prev = new JButton("Previous");
            prev.addActionListener(e -> layout.previous(cards));     // 1. Previous
            JButton next = new JButton("Next");
            next.addActionListener(e -> layout.next(cards));

            JPanel controls = new JPanel();
            controls.add(prev);
            controls.add(next);

            JPanel main = new JPanel(new BorderLayout());
            main.add(cards, BorderLayout.CENTER);
            main.add(controls, BorderLayout.SOUTH);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking "Previous" moves backward one card; from the first card, it wraps to the last card.

**Why the code produces this result.** `layout.previous(cards)` moves backward through the insertion order, wrapping at the beginning.

---

### Real-World Cases

- **Wizard "Back"** buttons.
- **Reverse card navigation**.

---

## 6. Card Layout Methods — last()

### Definitions

**Core Definition.** `last(Container parent)` flips to the last card in the container.

**Technical Definition.** `public void last(Container parent)`. The method sets the visible component to the last card in the container's internal ordering and hides all others.

**Beginner-Friendly Explanation.** `last()` jumps to the last card in the stack.

---

### Purposes

- To jump to the end of a card sequence.
- To return to a "final" or "summary" card.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
cardLayout.last(container);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `last(Container)` | Shows the last card. |

#### Syntax Rules

- The container must be the one managed by this `CardLayout`.

#### Constraints and Limitations

- If the container has no cards, the method has no effect.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Jump to Last Card

```java
import javax.swing.*;
import java.awt.*;

public class LastDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("last() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout layout = new CardLayout();
            JPanel cards = new JPanel(layout);

            for (int i = 1; i <= 3; i++) {
                JPanel p = new JPanel();
                p.add(new JLabel("Card " + i));
                p.setBackground(i == 1 ? Color.LIGHT_GRAY :
                    i == 2 ? Color.CYAN : Color.PINK);
                cards.add(p, "card" + i);
            }

            JButton last = new JButton("Last");
            last.addActionListener(e -> layout.last(cards));         // 1. Last
            JButton first = new JButton("First");
            first.addActionListener(e -> layout.first(cards));

            JPanel controls = new JPanel();
            controls.add(first);
            controls.add(last);

            JPanel main = new JPanel(new BorderLayout());
            main.add(cards, BorderLayout.CENTER);
            main.add(controls, BorderLayout.SOUTH);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking "Last" always jumps to the third card, regardless of the current card.

**Why the code produces this result.** `layout.last(cards)` sets the visible card to the last in insertion order.

---

### Real-World Cases

- **"Go to Summary"** buttons in wizards.
- **"View Last"** in card-based galleries.

---

## 7. Card Layout Methods — show()

### Definitions

**Core Definition.** `show(Container parent, String name)` shows the card with the specified name.

**Technical Definition.** `public void show(Container parent, String name)`. The method looks up the component associated with the given name in the container's internal table and makes it the visible card. If no component with that name exists, the method has no effect.

**Beginner-Friendly Explanation.** `show()` jumps directly to a card by its name, without stepping through the cards in sequence.

---

### Purposes

- To jump directly to a specific card by name.
- To implement navigation controls (combo boxes, tabs, menus) that map to card names.
- To enable arbitrary card switching without sequential navigation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
cardLayout.show(container, "cardName");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `show(Container, String)` | Shows the named card. |

#### Syntax Rules

- The name must match the string used when the component was added.
- If the name is not found, no change occurs.

#### Constraints and Limitations

- Names must be unique within the container.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Show by Name with a Combo Box

```java
import javax.swing.*;
import java.awt.*;

public class ShowDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("show() Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout layout = new CardLayout();
            JPanel cards = new JPanel(layout);

            JPanel red = new JPanel();
            red.add(new JLabel("Red Card"));
            red.setBackground(Color.RED);
            cards.add(red, "red");

            JPanel green = new JPanel();
            green.add(new JLabel("Green Card"));
            green.setBackground(Color.GREEN);
            cards.add(green, "green");

            JPanel blue = new JPanel();
            blue.add(new JLabel("Blue Card"));
            blue.setBackground(Color.BLUE);
            cards.add(blue, "blue");

            JComboBox<String> combo = new JComboBox<>(
                new String[]{"red", "green", "blue"});
            combo.addActionListener(e -> {
                String selected = (String) combo.getSelectedItem();
                layout.show(cards, selected);                         // 1. Show
            });

            JPanel main = new JPanel(new BorderLayout());
            main.add(combo, BorderLayout.NORTH);
            main.add(cards, BorderLayout.CENTER);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting a color from the combo box shows the corresponding card.

**Why the code produces this result.** `layout.show(cards, selected)` displays the card whose name matches the combo box selection.

---

### Real-World Cases

- **Settings dialogs** with a sidebar or combo box for category navigation.
- **Tab-like interfaces** where the navigation is provided by custom controls.

---

## 8. CardLayout Best Practices

### Definitions

**Core Definition.** `CardLayout` best practices are the conventions and patterns that ensure card-based layouts are used effectively, thread-safely, and with good user experience.

**Technical Definition.** Best practices include: always naming every card with a unique string identifier; retaining a reference to the `CardLayout` instance for navigation; using `show()` for direct access and `next()`/`previous()` for sequential flows; keeping navigation controls outside the card panel (in a separate `BorderLayout` region); calling `revalidate()` and `repaint()` after dynamic card changes; and considering `JTabbedPane` when built-in tab navigation is desired.

**Beginner-Friendly Explanation.** Best practices for `CardLayout` are the “rules of thumb” that prevent lost cards, missing navigation, and layout confusion. They include things like “always name your cards,” “keep a reference to the layout manager,” “put your navigation buttons outside the card area,” and “call `revalidate()` after adding cards dynamically.”

---

### Purposes

- To ensure cards can be accessed reliably by name via `show()`.
- To maintain a reference to the layout manager for navigation.
- To provide a clean separation between the card container and its navigation controls.
- To keep the GUI responsive and correctly laid out after dynamic changes.
- To choose the right tool (CardLayout vs. JTabbedPane) for the use case.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    CardLayout layout = new CardLayout();                         // 1. Card layout
    JPanel cards = new JPanel(layout);                            // 2. Container

    cards.add(panel1, "card1");                                   // 3. Named cards
    cards.add(panel2, "card2");

    JPanel nav = new JPanel(new FlowLayout());                     // 4. Navigation
    JButton next = new JButton("Next");
    next.addActionListener(e -> layout.next(cards));
    nav.add(next);

    JPanel main = new JPanel(new BorderLayout());                  // 5. Nest
    main.add(cards, BorderLayout.CENTER);
    main.add(nav, BorderLayout.SOUTH);
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Unique names | `cards.add(component, "uniqueName")` |
| Retain layout reference | `CardLayout layout = new CardLayout()` |
| Navigation controls outside cards | `main.add(nav, BorderLayout.SOUTH)` |
| Direct access via `show()` | `layout.show(cards, "name")` |
| Sequential access via `next()`/`previous()` | `layout.next(cards)` / `layout.previous(cards)` |
| Revalidate after changes | `cards.revalidate(); cards.repaint()` |
| Consider `JTabbedPane` | When built-in tab navigation is desired |

#### Syntax Rules

- **Always name every card with a unique string**; unnamed cards cannot be accessed via `show()`.
- **Always retain a reference to the `CardLayout`** because navigation methods are instance methods that require the layout manager.
- **Always place navigation controls outside the card container** (e.g., in a `BorderLayout.SOUTH` region) to avoid the controls being hidden when cards switch.
- **Always call `revalidate()` and `repaint()`** after adding or removing cards dynamically.
- **Prefer `JTabbedPane`** when you want built-in tab navigation; `CardLayout` requires you to build the navigation yourself.

#### Constraints and Limitations

- `CardLayout` does not provide any visual indication of which card is visible; the programmer must supply that feedback.
- The container's preferred size is the maximum of all cards, which may be larger than necessary for some cards.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice CardLayout with Combo Box Navigation

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeCardLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeCardLayout::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice CardLayout");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        CardLayout layout = new CardLayout();                        // 1. Layout
        JPanel cards = new JPanel(layout);                           // 2. Container

        JPanel login = new JPanel();
        login.add(new JLabel("Login Panel"));
        login.setBackground(Color.LIGHT_GRAY);
        cards.add(login, "login");                                    // 3. Named

        JPanel mainPanel = new JPanel();
        mainPanel.add(new JLabel("Main Application"));
        mainPanel.setBackground(Color.CYAN);
        cards.add(mainPanel, "main");

        JPanel settings = new JPanel();
        settings.add(new JLabel("Settings Panel"));
        settings.setBackground(Color.PINK);
        cards.add(settings, "settings");

        JComboBox<String> nav = new JComboBox<>(
            new String[]{"login", "main", "settings"});               // 4. Navigation
        nav.addActionListener(e -> {
            String selected = (String) nav.getSelectedItem();
            layout.show(cards, selected);                             // 5. show()
        });

        JPanel main = new JPanel(new BorderLayout());                 // 6. Nest
        main.add(nav, BorderLayout.NORTH);
        main.add(cards, BorderLayout.CENTER);

        frame.add(main);
        frame.setSize(500, 350);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window with a combo box at the top for selecting cards and the corresponding card displayed below. The layout uses `show()` for direct navigation by name.

**Why the code produces this result.** Every card has a unique name, the `CardLayout` reference is retained, and the navigation combo box is placed outside the card container. `show()` jumps directly to the selected card.

---

#### Example 2: Common Mistake — Unnamed Cards

```java
import javax.swing.*;
import java.awt.*;

public class BadUnnamedCards {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Unnamed Cards");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            CardLayout layout = new CardLayout();
            JPanel cards = new JPanel(layout);

            JPanel card1 = new JPanel();
            card1.add(new JLabel("Card 1"));
            cards.add(card1);                                         // WRONG: no name
            JPanel card2 = new JPanel();
            card2.add(new JLabel("Card 2"));
            cards.add(card2);                                         // WRONG: no name

            JButton showCard2 = new JButton("Show Card 2");
            showCard2.addActionListener(e -> {
                // layout.show(cards, "card2"); // Cannot: no name!
                layout.next(cards);                                   // Only sequential
            });

            JPanel main = new JPanel(new BorderLayout());
            main.add(cards, BorderLayout.CENTER);
            main.add(showCard2, BorderLayout.SOUTH);

            frame.add(main);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The cards can only be navigated sequentially with `next()`; there is no way to jump directly to "Card 2" because it was added without a name.

**Why this is wrong.** In Java 1.1 and later, all components added to a `CardLayout` must be named. Unnamed cards cannot be accessed via `show()`, and the layout manager requires names for reliable navigation. The correct approach is to always supply a unique string name when adding each card.

---

### Real-World Cases

- **Production Swing applications** use `CardLayout` for multi-step flows (login, wizards, dashboards) with named cards and external navigation controls.
- **Settings dialogs** use a combo box or sidebar to drive `show()` for direct category access.
- **Educational examples** use the best-practice skeleton to demonstrate correct `CardLayout` usage from the start.

---

## References

Oracle. *How to Use CardLayout*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/layout/card.html

Oracle. *Class CardLayout*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/CardLayout.html

Oracle. *Class CardLayout*. Java Development Kit Version 22 API Specification. https://docs.oracle.com/en/java/javase/22/docs/api/java.desktop/java/awt/CardLayout.html

O'Reilly. *Java AWT — Chapter 7: Layouts — CardLayout*. https://www.oreilly.com/openbook/javawt/book/ch07.pdf

OpenJDK. *CardLayout.java source*. https://cr.openjdk.org/~serb/8055326/webrev.00/src/java.desktop/share/classes/java/awt/CardLayout.java-.html

Stack Overflow. *CardLayout best practices — naming and navigation*. https://stackoverflow.com/questions/ (community consensus on named cards and layout references)

Stack Overflow. *Using CardLayout for dynamic JPanel switching*. https://devgex.com/ (discussion of CardLayout as the recommended solution for shared-space views)