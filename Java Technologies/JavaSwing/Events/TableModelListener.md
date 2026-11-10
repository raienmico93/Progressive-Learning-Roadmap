# TableModelListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `TableModelListener` is a functional interface in the `javax.swing.event` package for receiving **table model events**—notifications that the structure or data of a `TableModel` has changed. It declares a single method, `tableChanged(TableModelEvent e)`, which is invoked whenever a `JTable`'s underlying model reports an insertion, deletion, or update.

**Technical Definition.** `public interface TableModelListener extends EventListener`. The OpenJDK source annotates it with `@FunctionalInterface`, confirming that `tableChanged` is the only abstract method. Because the interface has only one method, **there is no corresponding adapter class**—unlike `MouseListener` or `WindowListener`, which require `MouseAdapter` and `WindowAdapter`. Listeners are registered on the `TableModel` via `addTableModelListener(TableModelListener)`, not on the `JTable` itself. The `JTable` **automatically registers itself** as a `TableModelListener` on its model; application code that also needs to react to model changes registers an additional listener. The event object, `TableModelEvent`, extends `java.util.EventObject` and provides `getFirstRow()`, `getLastRow()`, `getColumn()`, and `getType()`. All model modifications—and therefore listener notifications—should occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `TableModelListener` is the “table data just changed” listener. When a row is added to a table, a row is deleted, or a cell value is edited, the table's model fires a `TableModelEvent` to every registered `TableModelListener`. The listener's `tableChanged` method runs, and the event tells you **what kind of change** occurred (`INSERT`, `DELETE`, or `UPDATE`), **which rows** were affected (first and last row indices), and **which column** was involved (`ALL_COLUMNS` for row-level changes). The `JTable` itself uses this listener internally to repaint the affected cells—your application can register its own listener to react to data changes: logging edits, validating input, synchronising other views, or persisting changes to a database. Think of `TableModelListener` as a change-data-capture feed for a table model: it reports every structural or value change, and you decide which changes deserve a reaction.

---

### Key Characteristics

1. **Single-method functional interface.** `tableChanged` is the only method; lambdas and method references are idiomatic.
2. **No adapter class.** Because there is only one method, Swing does not provide a `TableModelAdapter`; you simply implement `tableChanged` (or use a lambda).
3. **Registered on the model, not the table.** `table.getModel().addTableModelListener(...)` is the correct registration target. The `JTable` already registers its own listener on the model.
4. **Fine-grained notification.** `TableModelEvent` identifies the exact range of rows and the column that changed, enabling efficient updates without repainting the entire table.
5. **Three event types.** `TableModelEvent.INSERT` (value 1), `TableModelEvent.UPDATE` (value 0), and `TableModelEvent.DELETE` (value -1) distinguish structural additions, value changes, and removals.
6. **Special constants.** `TableModelEvent.ALL_COLUMNS` (-1) indicates that all columns are affected; `TableModelEvent.HEADER_ROW` (-1) identifies the table header as the changed row.
7. **EDT discipline.** Model modifications must occur on the EDT. If a listener reacts to a change by modifying the model again, the modification should be deferred with `SwingUtilities.invokeLater` to avoid recursive event cascades.
8. **Automatic table synchronization.** The `JTable`'s built-in listener ensures the GUI reflects the model's current state; application listeners are additive and should not interfere with the table's own processing.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JTable`, `TableModel`, `AbstractTableModel`, and `DefaultTableModel`.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `TableModelEvent` and the `javax.swing.event` and `javax.swing.table` packages.
- Knowledge of `fireTableDataChanged`, `fireTableRowsInserted`, `fireTableRowsDeleted`, and `fireTableCellUpdated` (the notification methods of `AbstractTableModel`).
- A JDK (version 8 or later; `TableModelListener` is in `javax.swing.event`).

---

### Related Programming Areas

- **`TableModel` and `AbstractTableModel`:** The model interfaces and base class that fire `TableModelEvent`s.
- **`TableModelEvent`:** The event object carrying the type, row range, and column of the change.
- **`JTable`:** The component that consumes model events to repaint itself; it registers its own internal `TableModelListener`.
- **`ListSelectionListener`:** For selection changes in a `JTable`—distinct from data changes.
- **`SwingUtilities.invokeLater`:** Used to defer model modifications triggered by a listener, preventing recursive event cascades.
- **`SwingWorker`:** For offloading database persistence or other heavy work triggered by model changes.
- **`DocumentListener`:** The analogous listener for text components; `TableModelListener` is for table data.

---

## Core Concepts and Features

---

## 1. TableModelListener — The Interface

### Definitions

**Core Definition.** `TableModelListener` is a functional interface for receiving notifications when a `TableModel`'s structure or data changes.

**Technical Definition.** `public interface TableModelListener extends EventListener`. It declares `void tableChanged(TableModelEvent e)`. The interface is annotated `@FunctionalInterface`, allowing lambda implementations. Registration is via `TableModel.addTableModelListener(TableModelListener)`; there is no adapter class because the interface has only one method. The method is invoked on the EDT.

**Beginner-Friendly Explanation.** `TableModelListener` is the “table data changed” contract. Implement `tableChanged` (or write a lambda) and register it with a table's model to hear about insertions, deletions, and updates.

---

### Purposes

- To react when rows are inserted into a table model.
- To react when rows are deleted from a table model.
- To react when cell values are updated in a table model.
- To synchronise multiple views or components that depend on the same table data.
- To validate, log, or persist changes to table data.
- To trigger dependent calculations when table data changes.

---

### Syntax Rules and Structure

#### Complete General Syntax (Registration and Handling)

```java
TableModel model = table.getModel();
model.addTableModelListener(new TableModelListener() {
    @Override
    public void tableChanged(TableModelEvent e) {
        // reaction code
    }
});

// Or with a lambda (the interface is functional):
model.addTableModelListener(e -> {
    // reaction code
});
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addTableModelListener(TableModelListener)` | Registers the listener on the model. |
| `removeTableModelListener(TableModelListener)` | Unregisters the listener. |
| `tableChanged(TableModelEvent e)` | The single method to implement. |
| `e.getType()` | `INSERT`, `UPDATE`, or `DELETE`. |
| `e.getFirstRow()` | The first affected row (or `HEADER_ROW`). |
| `e.getLastRow()` | The last affected row. |
| `e.getColumn()` | The affected column, or `ALL_COLUMNS`. |

#### Syntax Rules

- Only one method is required for the listener implementation.
- Register on the **model**, not on the `JTable` itself.
- The listener must be registered before the change occurs.
- All listener code runs on the EDT; it must not block.
- If the listener modifies the model, the modification should be deferred with `SwingUtilities.invokeLater` to avoid recursive event firing.

#### Constraints and Limitations

- There is no adapter class; you must implement `tableChanged` directly.
- Modifying the model from within `tableChanged` without `invokeLater` can cause recursive event cascades or unexpected behaviour.
- The `JTable` itself uses a `TableModelListener` internally; application listeners are additional and should not interfere with the table's processing.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic TableModelListener on a DefaultTableModel

```java
import javax.swing.*;
import javax.swing.event.TableModelEvent;
import javax.swing.event.TableModelListener;
import javax.swing.table.DefaultTableModel;
import java.awt.*;

public class BasicTableModelListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Basic TableModelListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(
                new Object[][]{{"Alice", 30}, {"Bob", 25}},
                new Object[]{"Name", "Age"});
            JTable table = new JTable(model);
            JLabel status = new JLabel("Ready", SwingConstants.CENTER);

            model.addTableModelListener(new TableModelListener() {     // 1. Register
                @Override
                public void tableChanged(TableModelEvent e) {          // 2. React
                    String type;
                    switch (e.getType()) {
                        case TableModelEvent.INSERT: type = "INSERT"; break;
                        case TableModelEvent.UPDATE: type = "UPDATE"; break;
                        case TableModelEvent.DELETE: type = "DELETE"; break;
                        default: type = "OTHER";
                    }
                    status.setText(type + " rows " + e.getFirstRow()
                        + "-" + e.getLastRow() + ", col " + e.getColumn());
                }
            });

            JPanel controls = new JPanel();
            JButton add = new JButton("Add Row");
            add.addActionListener(ev -> model.addRow(new Object[]{"New", 0}));
            JButton remove = new JButton("Remove Last");
            remove.addActionListener(ev -> {
                if (model.getRowCount() > 0) {
                    model.removeRow(model.getRowCount() - 1);
                }
            });
            controls.add(add);
            controls.add(remove);

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(table), BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.add(controls, BorderLayout.NORTH);
            frame.setSize(450, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Add Row” fires an `INSERT` event and updates the status label to “INSERT rows 2-2, col -1”. Clicking “Remove Last” fires a `DELETE` event. Editing a cell fires an `UPDATE` event with the specific row and column.

**Why the code produces this result.** The listener is registered on the `DefaultTableModel`. The model's `addRow` and `removeRow` methods fire `TableModelEvent`s with `INSERT` and `DELETE` types, respectively. Cell edits fire `UPDATE` events with the affected row and column. The listener reads the event's type, row range, and column to update the status label.

---

### Real-World Cases

- **Audit logging:** Recording every table data change with its type, row range, and column.
- **Database persistence:** Saving changes to a database when the model is modified.
- **Multi-view synchronisation:** Updating a chart or summary panel when table data changes.
- **Data validation:** Checking new values as they are inserted or updated.
- **Undo/redo systems:** Capturing model changes for reversible operations.

---

## 2. tableChanged() — The Method

### Definitions

**Core Definition.** `tableChanged(TableModelEvent e)` is the single abstract method of `TableModelListener`. It is invoked when the table model's structure or data changes.

**Technical Definition.** `void tableChanged(TableModelEvent e)`. The method is invoked on the EDT by the model's `fireTableChanged` mechanism. The `TableModelEvent` provides the type of change (`INSERT`, `UPDATE`, or `DELETE`), the first and last affected row, and the affected column (or `ALL_COLUMNS`). If the method throws an unchecked exception, it propagates up the EDT.

**Beginner-Friendly Explanation.** `tableChanged` is the “table data just changed” method. Everything you write inside it runs on the EDT when the model changes.

---

### Purposes

- To contain the reaction logic for table data changes.
- To receive the `TableModelEvent` context for deciding what to do based on the type and range of the change.
- To trigger immediate UI updates (status labels, dependent panels, charts).
- To synchronise state with a domain model or external system.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void tableChanged(TableModelEvent e) {
    int type = e.getType();
    int firstRow = e.getFirstRow();
    int lastRow = e.getLastRow();
    int column = e.getColumn();
    // react to the change
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `@Override` | Confirms the method overrides the interface declaration. |
| `TableModelEvent e` | The event object with type, row range, and column. |
| `e.getType()` | `INSERT`, `UPDATE`, or `DELETE`. |

#### Syntax Rules

- The method must be `public` and accept a single `TableModelEvent` parameter.
- The method should perform its work quickly; long tasks must be offloaded.
- If the method modifies the model, use `SwingUtilities.invokeLater` to defer the modification.

#### Constraints and Limitations

- The EDT is blocked until the method returns.
- Uncaught exceptions can disrupt the event pipeline.
- Modifying the model directly from within `tableChanged` can cause recursive events.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Distinguishing Event Types

```java
import javax.swing.*;
import javax.swing.event.TableModelEvent;
import javax.swing.table.DefaultTableModel;

public class DistinguishEventTypes {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Distinguish Event Types");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(
                new Object[]{"Item"}, 0);
            JTable table = new JTable(model);

            model.addTableModelListener(e -> {
                switch (e.getType()) {
                    case TableModelEvent.INSERT:
                        System.out.println("Row inserted: " + e.getFirstRow());
                        break;
                    case TableModelEvent.UPDATE:
                        System.out.println("Cell updated: row " + e.getFirstRow()
                            + ", col " + e.getColumn());
                        break;
                    case TableModelEvent.DELETE:
                        System.out.println("Row deleted: " + e.getFirstRow());
                        break;
                }
            });

            JPanel controls = new JPanel();
            JButton add = new JButton("Add");
            add.addActionListener(ev -> model.addRow(new Object[]{"X"}));
            JButton remove = new JButton("Remove");
            remove.addActionListener(ev -> {
                if (model.getRowCount() > 0) {
                    model.removeRow(model.getRowCount() - 1);
                }
            });
            controls.add(add);
            controls.add(remove);

            frame.setLayout(new java.awt.BorderLayout());
            frame.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            frame.add(controls, java.awt.BorderLayout.SOUTH);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Add” prints “Row inserted: 0”; clicking “Remove” prints “Row deleted: 0”. Editing a cell prints “Cell updated: row [n], col [c]”.

**Why the code produces this result.** The listener inspects `getType()` to dispatch to the appropriate branch. `getFirstRow()` returns the affected row, and `getColumn()` returns the specific column for `UPDATE` events (or `ALL_COLUMNS` for row-level changes).

---

### Real-World Cases

- **Conditional persistence:** Only save to the database for `UPDATE` events, not for `INSERT` during initial loading.
- **UI synchronisation:** Enable a “Delete” button only when a row is selected (though this is better handled by `ListSelectionListener`).
- **Change-specific logging:** Log insertions and deletions differently from updates.

---

## 3. TableModelEvent — The Event Object

### Definitions

**Core Definition.** `TableModelEvent` is the event object delivered to `TableModelListener`s. It describes a change to a `TableModel`'s structure or data, including the type of change, the affected row range, and the affected column.

**Technical Definition.** `public class TableModelEvent extends EventObject`. It provides the following constants: `INSERT = 1`, `UPDATE = 0`, `DELETE = -1`, `ALL_COLUMNS = -1`, and `HEADER_ROW = -1`. Methods include `getFirstRow()`, `getLastRow()`, `getColumn()`, and `getType()`. The event's `getType()` returns one of `INSERT`, `UPDATE`, or `DELETE`.

**Beginner-Friendly Explanation.** `TableModelEvent` is the “what changed” note delivered to your listener. It tells you the type of change, the rows involved, and the column (or all columns).

---

### Purposes

- To identify the type of change (`INSERT`, `UPDATE`, or `DELETE`).
- To identify the range of rows affected.
- To identify the column affected, or that all columns are involved.
- To support efficient, range-based handling without querying the entire model.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
if (e.getType() == TableModelEvent.INSERT) {
    int first = e.getFirstRow();
    int last = e.getLastRow();
    // react to insertion
}
```

#### Component Breakdown

| Method / Constant | Meaning |
|-------------------|---------|
| `getType()` | `INSERT`, `UPDATE`, or `DELETE`. |
| `getFirstRow()` | The first affected row (or `HEADER_ROW`). |
| `getLastRow()` | The last affected row. |
| `getColumn()` | The affected column, or `ALL_COLUMNS`. |
| `INSERT` | Constant for row insertion. |
| `UPDATE` | Constant for cell/row update. |
| `DELETE` | Constant for row deletion. |
| `ALL_COLUMNS` | Indicates all columns are affected. |
| `HEADER_ROW` | Indicates the header row is affected. |

#### Syntax Rules

- `getFirstRow()` and `getLastRow()` define an inclusive range of rows.
- For `INSERT` and `DELETE` events, `getColumn()` is typically `ALL_COLUMNS`.
- For `UPDATE` events, `getColumn()` may be a specific column index or `ALL_COLUMNS`.

#### Constraints and Limitations

- The event does not carry the new data values; query the model via `getValueAt` if needed.
- The meaning of "first" and "last" row for `DELETE` events can be subtle; in some implementations, the range refers to the rows that were removed, but the indices may no longer exist in the model after the deletion.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting the Event Range

```java
import javax.swing.*;
import javax.swing.event.TableModelEvent;
import javax.swing.table.DefaultTableModel;

public class InspectTableModelEvent {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Inspect TableModelEvent");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(
                new Object[]{"A", "B"}, 0);
            JTable table = new JTable(model);

            model.addTableModelListener(e -> {
                System.out.println("Type: " + e.getType()
                    + ", Rows: " + e.getFirstRow() + "-" + e.getLastRow()
                    + ", Col: " + e.getColumn());
            });

            JButton add = new JButton("Add Row");
            add.addActionListener(ev ->
                model.addRow(new Object[]{"x", "y"}));
            JButton update = new JButton("Update Cell");
            update.addActionListener(ev -> {
                if (model.getRowCount() > 0) {
                    model.setValueAt("z", 0, 0);
                }
            });

            JPanel controls = new JPanel();
            controls.add(add);
            controls.add(update);
            frame.setLayout(new java.awt.BorderLayout());
            frame.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            frame.add(controls, java.awt.BorderLayout.SOUTH);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Adding a row prints “Type: 1, Rows: 0-0, Col: -1” (`INSERT`, row 0, `ALL_COLUMNS`). Updating a cell prints “Type: 0, Rows: 0-0, Col: 0” (`UPDATE`, row 0, column 0).

**Why the code produces this result.** `addRow` fires an `INSERT` event covering the new row with `ALL_COLUMNS`. `setValueAt` fires an `UPDATE` event for the specific row and column. The listener prints the type, row range, and column to demonstrate the event's fine-grained information.

---

### Real-World Cases

- **Selective persistence:** Only persist `UPDATE` events after a debounce, while ignoring `INSERT` events during initial data loading.
- **Range-based repainting:** Custom views can repaint only the affected row range instead of the entire table.

---

## 4. Table Model Listener Best Practices

### Definitions

**Core Definition.** Table model listener best practices are the conventions and patterns that ensure table data change handling is correct, thread-safe, and free of recursive event cascades.

**Technical Definition.** Best practices include: registering the listener on the `TableModel` (not the `JTable`); deferring model modifications triggered by `tableChanged` with `SwingUtilities.invokeLater` to avoid recursive events; using the event type to distinguish insertions, deletions, and updates; offloading heavy work (database persistence, network calls) to `SwingWorker`; removing listeners when no longer needed; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for table model listeners are the “rules of thumb” that prevent infinite loops, threading bugs, and performance problems. They include things like “don't modify the model directly from `tableChanged`—use `invokeLater`,” “register on the model,” and “offload database writes to a background thread.”

---

### Purposes

- To prevent recursive event cascades by deferring model modifications.
- To keep the GUI responsive by offloading heavy work.
- To correctly identify and handle the different event types.
- To prevent memory leaks by removing listeners when no longer needed.
- To ensure thread safety by respecting the EDT.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
model.addTableModelListener(e -> {                                // 1. Model
    if (e.getType() == TableModelEvent.UPDATE) {                  // 2. Type
        SwingUtilities.invokeLater(() -> {                        // 3. Defer
            // modify model or update dependent UI
        });
    }
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Register on the model | `table.getModel().addTableModelListener(...)` |
| Defer model modifications | `SwingUtilities.invokeLater(...)` inside `tableChanged` |
| Filter by event type | `if (e.getType() == TableModelEvent.UPDATE)` |
| Offload heavy work | `SwingWorker` for database writes |
| Remove listeners | `removeTableModelListener(...)` on disposal |
| EDT | `SwingUtilities.invokeLater(...)` for construction |

#### Syntax Rules

- **Always register on the `TableModel`**, not on the `JTable` (the table already registers its own listener).
- **Never modify the model directly from `tableChanged`** without `SwingUtilities.invokeLater`; doing so can trigger recursive `tableChanged` calls and unexpected behaviour.
- **Always filter by `getType()`** when the reaction differs between insertions, deletions, and updates.
- **Never perform heavy work directly in `tableChanged`**; use `SwingWorker` for database persistence or I/O.
- **Always remove the listener** from long-lived models when the listening component is disposed.

#### Constraints and Limitations

- The `JTable`'s internal listener must complete its processing before your listener modifies the model; `invokeLater` ensures this ordering.
- `TableModelEvent` does not carry the new values; query the model if the reaction needs the changed data.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Deferred Model Modification

```java
import javax.swing.*;
import javax.swing.event.TableModelEvent;
import javax.swing.table.DefaultTableModel;
import java.awt.*;

public class BestPracticeTableModelListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeTableModelListener::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice TableModelListener");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        DefaultTableModel model = new DefaultTableModel(
            new Object[]{"Name"}, 0);
        JTable table = new JTable(model);
        JLabel status = new JLabel("Ready", SwingConstants.CENTER);

        model.addTableModelListener(e -> {                            // 1. Model
            if (e.getType() == TableModelEvent.INSERT) {             // 2. Type
                SwingUtilities.invokeLater(() -> {                   // 3. Defer
                    // safe to modify the model or update dependent UI
                    status.setText("Inserted row " + e.getFirstRow());
                });
            }
        });

        JButton add = new JButton("Add Row");
        add.addActionListener(ev -> model.addRow(new Object[]{"Item"}));

        frame.setLayout(new BorderLayout());
        frame.add(new JScrollPane(table), BorderLayout.CENTER);
        frame.add(add, BorderLayout.NORTH);
        frame.add(status, BorderLayout.SOUTH);
        frame.setSize(350, 250);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Clicking “Add Row” inserts a row and updates the status label to “Inserted row [n]” after the model change has fully settled.

**Why the code produces this result.** The listener filters on `INSERT` and defers the status update with `invokeLater`. This ensures that the `JTable`'s internal listener has finished processing the insertion before the application's reaction runs, avoiding any interference with the table's own update cycle.

---

#### Example 2: Common Mistake — Modifying the Model Directly in `tableChanged`

```java
import javax.swing.*;
import javax.swing.event.TableModelEvent;
import javax.swing.table.DefaultTableModel;

public class BadDirectModification {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Direct Modification");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(
                new Object[]{"Count"}, 0);
            JTable table = new JTable(model);

            model.addTableModelListener(e -> {
                if (e.getType() == TableModelEvent.INSERT) {
                    // WRONG: modifying model inside listener
                    model.addRow(new Object[]{model.getRowCount()}); // Recursive!
                }
            });

            JButton add = new JButton("Add");
            add.addActionListener(ev -> model.addRow(new Object[]{"x"}));

            frame.setLayout(new java.awt.BorderLayout());
            frame.add(new JScrollPane(table), java.awt.BorderLayout.CENTER);
            frame.add(add, java.awt.BorderLayout.SOUTH);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Add” triggers an infinite loop: the listener adds a row, which fires another `INSERT` event, which adds another row, recursively, until a `StackOverflowError` or an unresponsive application occurs.

**Why this is wrong.** Modifying the model directly from within `tableChanged` causes a recursive event cascade. The correct approach is to defer the modification with `SwingUtilities.invokeLater`, which places the modification at the end of the EDT queue so the current event processing can complete first.

---

#### Example 3: Best-Practice with Event Type Filtering and Offloaded Work

```java
import javax.swing.*;
import javax.swing.event.TableModelEvent;
import javax.swing.table.DefaultTableModel;
import java.awt.*;

public class OffloadedTableWork {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Offloaded Table Work");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(
                new Object[]{"Name", "Score"}, 0);
            JTable table = new JTable(model);
            JLabel status = new JLabel("Ready", SwingConstants.CENTER);

            model.addTableModelListener(e -> {
                if (e.getType() != TableModelEvent.UPDATE) return;   // 1. Update only
                int row = e.getFirstRow();
                int col = e.getColumn();
                Object value = model.getValueAt(row, col);

                new SwingWorker<Void, Void>() {                       // 2. Offload
                    @Override
                    protected Void doInBackground() {
                        persistChange(row, col, value);               // 3. Background
                        return null;
                    }
                    @Override
                    protected void done() {
                        status.setText("Saved row " + row);           // 4. EDT
                    }
                }.execute();
            });

            model.addRow(new Object[]{"Alice", 10});
            model.setValueAt(20, 0, 1);                               // Trigger update

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(table), BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static void persistChange(int row, int col, Object value) {
        try { Thread.sleep(500); } catch (InterruptedException e) {}
        System.out.println("Persisted: row=" + row + ", col=" + col + ", value=" + value);
    }
}
```

**Expected Output.** The `setValueAt` call triggers an `UPDATE` event. The listener filters on `UPDATE`, reads the changed value, and offloads the persistence work to a `SwingWorker`. The status label updates to “Saved row 0” after the background work completes.

**Why the code produces this result.** The listener filters out non-`UPDATE` events, reads the new value from the model, and starts a `SwingWorker`. The heavy persistence work runs off the EDT, and the UI update occurs in `done()`, which runs on the EDT. This keeps the GUI responsive during database writes.

---

### Real-World Cases

- **Database persistence:** Saving cell edits to a database via `SwingWorker` when `UPDATE` events fire.
- **Audit trails:** Logging `INSERT`, `UPDATE`, and `DELETE` events with their row ranges and columns.
- **Dependent UI updates:** Updating a summary chart or total label when table data changes, using `invokeLater` to defer the update until after the model change settles.
- **Multi-table synchronisation:** Propagating changes from one table's model to another via a shared listener pattern.

---

## References

Oracle. *Interface TableModelListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/TableModelListener.html

Oracle. *Class TableModelEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/TableModelEvent.html

Oracle. *How to Write a Table Model Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/tablemodellistener.html

Oracle. *How to Use Tables*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/table.html

OpenJDK. *TableModelListener.java source*. https://cr.openjdk.org/~alexsch/8007716/webrev.00/src/share/classes/javax/swing/event/TableModelListener.java.rhs.html

OpenJDK. *TableModelEvent.java source*. https://cr.openjdk.org/

OpenJDK. *javax.swing package-info — threading policy*. https://cr.openjdk.org/

Oracle. *Class AbstractTableModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/table/AbstractTableModel.html

Stack Overflow. *Is modifying model of JTable allowed while processing TableModelEvent?*. https://stackoverflow.com/questions/ (confirmation that modifications should be wrapped in `SwingUtilities.invokeLater`)

Stack Overflow. *TableModelListener best practices*. https://stackoverflow.com/questions/ (community consensus on registering on the model and offloading heavy work)

Oracle. *Class SwingWorker*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html