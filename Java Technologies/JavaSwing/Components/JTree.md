# JTree — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JTree` is a Swing component that displays a set of hierarchical data as an outline. It provides a view of data organized in a parent-child tree structure, where each item is called a **node** and the topmost node is the **root**. Users can expand and collapse branch nodes to reveal or hide their children.

**Technical Definition.** `public class JTree extends JComponent implements Scrollable, Accessible`. With the `JTree` class, you can display hierarchical data. A `JTree` object does not actually contain your data; it simply provides a view of the data. Like any non-trivial Swing component, the tree gets data by querying its data model . A specific node in a tree can be identified either by a `TreePath` (an object that encapsulates a node and all of its ancestors) or by its display row, where each row in the display area displays one node . The `JTree` relies on a `TreeModel` for data, a `TreeSelectionModel` for selection state, and pluggable `TreeCellRenderer` and `TreeCellEditor` delegates for painting and editing nodes . Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`. Serialized `JTree` objects are not compatible with future Swing releases .

**Beginner-Friendly Explanation.** `JTree` is the Swing component that displays data in a tree structure—like the folder hierarchy in a file explorer, the outline of a document, or the organizational chart of a company. Each item in the tree is called a “node.” The top node is the “root,” and it can have “children,” which can have their own children, and so on. Branch nodes (nodes that can have children) display a small handle or arrow that lets the user expand or collapse the node to show or hide its children. Leaf nodes (nodes that cannot have children) have no handle. The `JTree` itself does not hold your data; it is just the view. You supply a model (typically a `DefaultTreeModel` backed by `DefaultMutableTreeNode` objects) that holds the data and notifies the tree when things change. Think of it as a filing cabinet with nested folders: the model is the cabinet, and the `JTree` is the window through which you browse it.

---

### Key Characteristics

1. **Hierarchical view.** `JTree` displays data vertically, with each row representing one node. Every tree has a root node from which all nodes descend .
2. **Model-driven architecture.** The `JTree` does not contain data; it queries a `TreeModel` for nodes, children, and values. The default model is `DefaultTreeModel`, and the default node implementation is `DefaultMutableTreeNode`.
3. **Expand/collapse interaction.** Branch nodes can be expanded to show their children or collapsed to hide them. By default, all branch nodes except the root start out collapsed . A small handle (a circle or arrow) next to each branch node lets the user toggle its state .
4. **TreePath-based identification.** Nodes are identified by `TreePath` objects that encapsulate the node and all its ancestors, rather than by simple indices. Display rows are also used for view-level operations .
5. **Selection model.** A `TreeSelectionModel` manages which nodes are selected. The selection mode can be single, single-interval, or multiple-interval. A `TreeSelectionListener` is notified when the selection changes .
6. **Expansion listeners.** `TreeExpansionListener` (and `TreeWillExpandListener`) are notified when nodes are expanded or collapsed, enabling lazy loading of children .
7. **Customisable rendering and editing.** `TreeCellRenderer` controls how nodes are painted (e.g., icons and text), and `TreeCellEditor` controls how nodes are edited .
8. **Root visibility.** `setRootVisible(false)` hides the root node and makes its children the top-level visible nodes. `setShowsRootHandles(true)` displays expansion handles at the topmost level .
9. **Single-thread rule and serialization caveat.** `JTree` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases .

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, generics, and lambda expressions.
- Familiarity with Swing containers (`JFrame`, `JPanel`, `JScrollPane`) and layout managers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `TreeSelectionListener`, `TreeExpansionListener`, `TreeModelListener`, and the `javax.swing.event` package.
- Basic understanding of the MVC pattern.
- A JDK (version 8 or later; `JTree` is available in all modern JDKs).

---

### Related Programming Areas

- **`TreeModel` and `DefaultTreeModel`:** The data model interfaces and the default mutable implementation.
- **`TreeNode` and `DefaultMutableTreeNode`:** The node interfaces and the default mutable node implementation.
- **`TreePath`:** Encapsulates a node and all its ancestors for identification.
- **`TreeSelectionModel` and `DefaultTreeSelectionModel`:** Manage selection state.
- **`TreeCellRenderer` and `TreeCellEditor`:** Control node appearance and editing.
- **`JScrollPane`:** The standard container for trees, providing scrolling and viewport management.
- **`SwingWorker`:** For lazy-loading large tree branches off the EDT.

---

## Core Concepts and Features

---

## 1. JTree — The Class

### Definitions

**Core Definition.** `JTree` is the Swing class for displaying hierarchical data as an interactive outline.

**Technical Definition.** `public class JTree extends JComponent`. It provides constructors that accept a `TreeNode` (used as the root), a `TreeModel`, or arrays of data. The tree delegates data access to the model, selection to the selection model, and painting/editing to the renderer and editor.

**Beginner-Friendly Explanation.** `JTree` is the class you use to create a tree. You give it a root node (or a model), wrap it in a `JScrollPane`, and add it to your window.

---

### Purposes

- To display hierarchical data in an intuitive, expandable outline.
- To allow users to browse nested structures (file systems, organizational charts, document outlines).
- To support lazy loading of large datasets by loading children on demand when a node is expanded.
- To integrate with custom renderers and editors for rich node presentation.
- To serve as the standard Swing component for tree-structured data.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JTree tree = new JTree();                                      // empty
JTree tree = new JTree(TreeNode root);                         // node-based
JTree tree = new JTree(TreeModel model);                       // model-based
JTree tree = new JTree(Object[] value);                        // array-based (simple)
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JTree()` | Creates an empty tree with a default model. |
| `JTree(TreeNode root)` | Creates a tree with the given root node; internally creates a `DefaultTreeModel`. |
| `JTree(TreeModel newModel)` | Creates a tree using the supplied model. |
| `JTree(Object[] value)` | Creates a tree whose root has the array elements as children. |

#### Syntax Rules

- The tree must be added to a container (typically a `JScrollPane`) to be displayed.
- The root node must not be `null` when using the `TreeNode` constructor.
- Construction must occur on the EDT.

#### Constraints and Limitations

- `JTree` does not store data; it queries the model on demand.
- Row and path indices are view-based and may not match model indices if nodes are reordered or filtered.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JTree with DefaultMutableTreeNode

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import java.awt.*;

public class MinimalTree {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JTree");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root"); // 1. Root
            DefaultMutableTreeNode child1 = new DefaultMutableTreeNode("Child 1");
            DefaultMutableTreeNode child2 = new DefaultMutableTreeNode("Child 2");
            root.add(child1);                                       // 2. Add children
            root.add(child2);

            JTree tree = new JTree(root);                            // 3. Create tree
            JScrollPane scroll = new JScrollPane(tree);               // 4. Scroll pane

            frame.add(scroll, BorderLayout.CENTER);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window showing a tree with a “Root” node and two children, “Child 1” and “Child 2.” The root is expanded by default (its children are visible), and the root has a collapse handle.

**Why the code produces this result.** The `DefaultMutableTreeNode` objects form the tree structure. The `JTree` constructor creates a `DefaultTreeModel` using the root. The scroll pane provides scrolling and viewport management.

---

### Real-World Cases

- **File explorers** showing directory hierarchies.
- **Organization charts** showing reporting structures.
- **Document outlines** showing chapters and sections.
- **Configuration editors** showing nested settings.

---

## 2. Tree Creation

### Definitions

**Core Definition.** Tree creation is the process of instantiating a `JTree` with a root node, a `TreeModel`, or an array of values.

**Technical Definition.** `JTree` provides constructors that accept a `TreeNode` (which becomes the root of an internally created `DefaultTreeModel`), a `TreeModel`, or an array of objects. The `TreeNode`-accepting constructor creates a `DefaultTreeModel` using the supplied node as the root .

**Beginner-Friendly Explanation.** Creating a tree means deciding how to supply the data—via a root node and child nodes for simple trees, or via a custom model for complex ones.

---

### Purposes

- To instantiate the tree with the appropriate data source.
- To establish the root node and initial hierarchy.
- To prepare the tree for scroll pane wrapping and configuration.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
JTree tree = new JTree(root);
JScrollPane scroll = new JScrollPane(tree);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `TreeNode root` | The root node of the tree. |
| `TreeModel model` | The data model for the tree. |

#### Syntax Rules

- The tree must be wrapped in a `JScrollPane` to display properly.
- The model may be replaced later via `setModel`.

#### Constraints and Limitations

- The `TreeNode`-accepting constructor creates a `DefaultTreeModel`; for custom models, use the `TreeModel` constructor.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Tree with a Custom Model

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.DefaultTreeModel;
import java.awt.*;

public class ModelTreeCreation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Model-Based Tree");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root"); // 1. Root
            DefaultMutableTreeNode a = new DefaultMutableTreeNode("A");
            DefaultMutableTreeNode b = new DefaultMutableTreeNode("B");
            root.add(a);                                            // 2. Children
            root.add(b);

            DefaultTreeModel model = new DefaultTreeModel(root);    // 3. Model
            JTree tree = new JTree(model);                          // 4. Tree

            frame.add(new JScrollPane(tree));
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A tree with a “Root” node containing “A” and “B” children.

**Why the code produces this result.** The `DefaultTreeModel` wraps the root node and manages the hierarchy. The `JTree` uses the model to display the nodes.

---

### Real-World Cases

- **Building trees from database results** using a custom model.
- **Loading tree structures** from XML or JSON data.

---

## 3. Tree Model — DefaultTreeModel and TreeModel

### Definitions

**Core Definition.** `TreeModel` is the interface that defines how a `JTree` accesses data. `DefaultTreeModel` is a concrete implementation that uses `TreeNode` objects and provides methods for inserting, removing, and reloading nodes.

**Technical Definition.** `TreeModel` declares methods such as `getRoot()`, `getChild(Object parent, int index)`, `getChildCount(Object parent)`, `isLeaf(Object node)`, and `getIndexOfChild(Object parent, Object child)`. `DefaultTreeModel` implements this interface using `MutableTreeNode` objects and fires `TreeModelEvent`s when the structure changes.

**Beginner-Friendly Explanation.** The `TreeModel` is the data source for the tree. `DefaultTreeModel` is the “easy” model that works with `DefaultMutableTreeNode` objects and provides methods like `insertNodeInto`, `removeNodeFromParent`, and `nodeChanged`.

---

### Purposes

- To provide a pluggable data model for the tree.
- To enable custom data structures to be displayed as trees.
- To notify the tree of structural changes through events.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
DefaultTreeModel model = new DefaultTreeModel(root);
JTree tree = new JTree(model);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `insertNodeInto(MutableTreeNode newChild, MutableTreeNode parent, int index)` | Inserts a node. |
| `removeNodeFromParent(MutableTreeNode node)` | Removes a node. |
| `nodeChanged(TreeNode node)` | Notifies that a node’s value changed. |
| `reload(TreeNode node)` | Reloads a node’s children. |

#### Syntax Rules

- The model must be set on the tree via `setModel`.
- Changes to the model should be made through the model’s methods to fire proper events.

#### Constraints and Limitations

- `DefaultTreeModel` requires `MutableTreeNode` implementations; custom `TreeModel` implementations can use any data structure.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Tree with DefaultTreeModel

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.DefaultTreeModel;
import java.awt.*;

public class DynamicTree {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Dynamic Tree");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            DefaultTreeModel model = new DefaultTreeModel(root);    // 1. Model
            JTree tree = new JTree(model);

            JButton addButton = new JButton("Add Child");
            addButton.addActionListener(e -> {
                model.insertNodeInto(                           // 2. Insert
                    new DefaultMutableTreeNode("Child " + root.getChildCount()),
                    root, root.getChildCount());
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(tree), BorderLayout.CENTER);
            panel.add(addButton, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A tree with a “Root” node. Each click on “Add Child” appends a new child to the root.

**Why the code produces this result.** `insertNodeInto` adds a child to the model and fires a `TreeModelEvent`, causing the tree to update.

---

### Real-World Cases

- **File system browsers** that dynamically add children when directories are expanded.
- **Organizational charts** that update as employees are added or removed.

---

## 4. Tree Node — DefaultMutableTreeNode and TreeNode

### Definitions

**Core Definition.** `TreeNode` is the interface that defines a node in a tree. `DefaultMutableTreeNode` is a general-purpose, mutable implementation of `TreeNode` that can hold a user object and has methods for adding and removing children.

**Technical Definition.** `DefaultMutableTreeNode` implements `MutableTreeNode`, which extends `TreeNode`. It provides `add(MutableTreeNode child)`, `remove(MutableTreeNode child)`, `getChildAt(int index)`, `getChildCount()`, and `getUserObject()`. The `getUserObject()` method returns the data associated with the node.

**Beginner-Friendly Explanation.** `DefaultMutableTreeNode` is the “node” class. Each node holds a piece of data (its “user object”) and can have child nodes. You create nodes, add children to them, and build the tree structure.

---

### Purposes

- To represent individual items in the tree.
- To provide a mutable structure for adding and removing children.
- To hold user data for custom display and processing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultMutableTreeNode node = new DefaultMutableTreeNode("Label");
node.add(new DefaultMutableTreeNode("Child"));
Object data = node.getUserObject();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `add(MutableTreeNode)` | Adds a child node. |
| `remove(MutableTreeNode)` | Removes a child node. |
| `getUserObject()` | Returns the node’s data. |
| `setUserObject(Object)` | Sets the node’s data. |

#### Syntax Rules

- The user object can be any `Object`.
- Nodes can be nested arbitrarily.

#### Constraints and Limitations

- `DefaultMutableTreeNode` allows a node to be added to multiple parents, which can cause issues; use `insertNodeInto` with a model to maintain consistency.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Building a Tree with Nodes

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import java.awt.*;

public class NodeTree {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Node Tree");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Books"); // 1. Root
            DefaultMutableTreeNode fiction = new DefaultMutableTreeNode("Fiction");
            DefaultMutableTreeNode nonFiction = new DefaultMutableTreeNode("Non-Fiction");
            root.add(fiction);                                      // 2. Add branches
            root.add(nonFiction);

            fiction.add(new DefaultMutableTreeNode("Novel"));       // 3. Add leaves
            fiction.add(new DefaultMutableTreeNode("Short Story"));
            nonFiction.add(new DefaultMutableTreeNode("Biography"));

            JTree tree = new JTree(root);
            frame.add(new JScrollPane(tree));
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A tree with “Books” at the root, “Fiction” and “Non-Fiction” as branches, and leaf nodes under each branch.

**Why the code produces this result.** The `DefaultMutableTreeNode` objects are connected via `add`, forming the hierarchy. The `JTree` renders the structure.

---

### Real-World Cases

- **Category hierarchies** in e-commerce applications.
- **Menu structures** in applications.

---

## 5. Tree Methods — Model Access (setModel / getModel)

### Definitions

**Core Definition.** `setModel(TreeModel)` replaces the tree’s data model; `getModel()` returns the current model.

**Technical Definition.** `public void setModel(TreeModel newModel)` and `public TreeModel getModel()`. Replacing the model triggers the tree to rebuild its view and reset expansion state where possible.

**Beginner-Friendly Explanation.** `setModel` swaps the entire dataset of the tree. `getModel` gives you access to the current model.

---

### Purposes

- To change the tree’s data source at runtime.
- To access the model for direct data manipulation.
- To share a model across multiple trees.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tree.setModel(newModel);
TreeModel current = tree.getModel();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setModel(TreeModel)` | Replaces the model. |
| `getModel()` | Returns the current model. |

#### Syntax Rules

- The model must not be `null`.
- The tree revalidates and repaints after the model is replaced.

#### Constraints and Limitations

- Expansion state may be reset when the model changes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching Models

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.DefaultTreeModel;

public class SetModelDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode rootA = new DefaultMutableTreeNode("A");
            rootA.add(new DefaultMutableTreeNode("A1"));
            DefaultTreeModel modelA = new DefaultTreeModel(rootA);

            DefaultMutableTreeNode rootB = new DefaultMutableTreeNode("B");
            rootB.add(new DefaultMutableTreeNode("B1"));
            rootB.add(new DefaultMutableTreeNode("B2"));
            DefaultTreeModel modelB = new DefaultTreeModel(rootB);

            JTree tree = new JTree(modelA);
            JButton switchButton = new JButton("Switch Model");
            switchButton.addActionListener(e ->
                tree.setModel(modelB));                              // 1. Switch

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(tree), java.awt.BorderLayout.CENTER);
            panel.add(switchButton, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The tree initially shows “A” with child “A1.” Clicking “Switch Model” replaces it with “B” and children “B1,” “B2.”

**Why the code produces this result.** `setModel` swaps the data source, and the tree repaints with the new content.

---

### Real-World Cases

- **Switching datasets** based on user selection.
- **Refreshing the tree** from a database.

---

## 6. Tree Methods — Selection Access (getSelectionPath / getSelectionPaths / setSelectionPath)

### Definitions

**Core Definition.** `getSelectionPath()` returns the first selected path, or `null` if nothing is selected. `getSelectionPaths()` returns all selected paths. `setSelectionPath(TreePath path)` selects the specified path.

**Technical Definition.** `public TreePath getSelectionPath()`, `public TreePath[] getSelectionPaths()`, and `public void setSelectionPath(TreePath path)`. These methods delegate to the `TreeSelectionModel`.

**Beginner-Friendly Explanation.** These methods let you read and set which nodes are selected in the tree.

---

### Purposes

- To read the user’s selection for processing.
- To programmatically select a node.
- To support multi-selection operations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
TreePath path = tree.getSelectionPath();
tree.setSelectionPath(new TreePath(nodePath));
TreePath[] paths = tree.getSelectionPaths();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSelectionPath()` | Returns the first selected path, or `null`. |
| `getSelectionPaths()` | Returns all selected paths (empty array if none). |
| `setSelectionPath(TreePath)` | Selects the specified path. |

#### Syntax Rules

- Always check for `null` or empty arrays before using the results.
- `TreePath` objects can be constructed from an array of nodes or from an existing node.

#### Constraints and Limitations

- Selection is cleared when the model changes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading and Setting Selection

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.TreePath;

public class SelectionDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Selection");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            DefaultMutableTreeNode child = new DefaultMutableTreeNode("Child");
            root.add(child);
            JTree tree = new JTree(root);

            JButton read = new JButton("Read Selection");
            read.addActionListener(e -> {
                TreePath path = tree.getSelectionPath();             // 1. Read
                if (path != null) {
                    System.out.println("Selected: " + path.getLastPathComponent());
                }
            });

            JButton selectChild = new JButton("Select Child");
            selectChild.addActionListener(e ->
                tree.setSelectionPath(new TreePath(child.getPath()))); // 2. Set

            JPanel panel = new JPanel();
            panel.add(read);
            panel.add(selectChild);

            frame.add(new JScrollPane(tree), java.awt.BorderLayout.CENTER);
            frame.add(panel, java.awt.BorderLayout.SOUTH);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Read Selection” prints the selected node; clicking “Select Child” selects the “Child” node.

**Why the code produces this result.** `getSelectionPath` reads the selection, and `setSelectionPath` programmatically selects a node.

---

### Real-World Cases

- **Master-detail views** that update based on the selected node.
- **Programmatic navigation** to a specific node.

---

## 7. Tree Methods — expandPath / collapsePath

### Definitions

**Core Definition.** `expandPath(TreePath path)` expands the specified path, revealing its children. `collapsePath(TreePath path)` collapses the path, hiding its children.

**Technical Definition.** `public void expandPath(TreePath path)` and `public void collapsePath(TreePath path)`. These methods modify the tree’s expanded state and fire `TreeExpansionEvent`s.

**Beginner-Friendly Explanation.** `expandPath` opens a branch to show its children; `collapsePath` closes it.

---

### Purposes

- To programmatically control which branches are expanded.
- To reveal a specific node by expanding its ancestors.
- To initialise the tree’s expansion state on startup.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tree.expandPath(new TreePath(nodePath));
tree.collapsePath(new TreePath(nodePath));
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `expandPath(TreePath)` | Expands the path. |
| `collapsePath(TreePath)` | Collapses the path. |

#### Syntax Rules

- The path must be valid; otherwise, the method has no effect.
- Expanding a leaf node has no visible effect.

#### Constraints and Limitations

- Expanding a path fires `TreeExpansionEvent`s, which can trigger lazy-loading listeners.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Expanding and Collapsing

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.TreePath;

public class ExpandCollapseDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Expand/Collapse");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            DefaultMutableTreeNode branch = new DefaultMutableTreeNode("Branch");
            branch.add(new DefaultMutableTreeNode("Leaf"));
            root.add(branch);
            JTree tree = new JTree(root);

            JButton expand = new JButton("Expand");
            expand.addActionListener(e ->
                tree.expandPath(new TreePath(branch.getPath())));    // 1. Expand

            JButton collapse = new JButton("Collapse");
            collapse.addActionListener(e ->
                tree.collapsePath(new TreePath(branch.getPath())));  // 2. Collapse

            JPanel panel = new JPanel();
            panel.add(expand);
            panel.add(collapse);

            frame.add(new JScrollPane(tree), java.awt.BorderLayout.CENTER);
            frame.add(panel, java.awt.BorderLayout.SOUTH);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Expand” reveals the “Leaf” node; clicking “Collapse” hides it.

**Why the code produces this result.** `expandPath` and `collapsePath` modify the tree’s expanded state for the specified path.

---

### Real-World Cases

- **Revealing a node** by expanding all its ancestors after a search.
- **Restoring expansion state** from saved preferences.

---

## 8. Tree Methods — setRootVisible / setShowsRootHandles

### Definitions

**Core Definition.** `setRootVisible(boolean rootVisible)` controls whether the root node is displayed. `setShowsRootHandles(boolean newValue)` controls whether expansion handles are displayed at the topmost level of the tree.

**Technical Definition.** `public void setRootVisible(boolean rootVisible)` and `public void setShowsRootHandles(boolean newValue)`. When the root is not visible, its children become the top-level nodes. When `showsRootHandles` is `true`, handles are displayed adjacent to the topmost nodes (the root if visible, or its children if not) to allow them to be expanded or collapsed .

**Beginner-Friendly Explanation.** `setRootVisible(false)` hides the root node so its children appear as the top-level items. `setShowsRootHandles(true)` puts expand/collapse handles on the top-level nodes.

---

### Purposes

- To hide the root node for a cleaner, forest-like presentation.
- To display handles at the top level so top-level branches can be expanded even when the root is hidden.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tree.setRootVisible(false);
tree.setShowsRootHandles(true);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setRootVisible(boolean)` | `false` hides the root node. |
| `setShowsRootHandles(boolean)` | `true` displays handles at the topmost level. |

#### Syntax Rules

- Both methods are bound properties and fire property change events.

#### Constraints and Limitations

- Hiding the root does not remove it from the model; it only hides it from view.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Hiding the Root and Showing Handles

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;

public class RootVisibilityDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Root Visibility");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            DefaultMutableTreeNode branch = new DefaultMutableTreeNode("Branch");
            branch.add(new DefaultMutableTreeNode("Leaf"));
            root.add(branch);

            JTree tree = new JTree(root);
            tree.setRootVisible(false);                              // 1. Hide root
            tree.setShowsRootHandles(true);                          // 2. Show handles

            frame.add(new JScrollPane(tree));
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The root node is hidden, and “Branch” appears as the top-level node with an expand handle to reveal “Leaf.”

**Why the code produces this result.** `setRootVisible(false)` hides the root; `setShowsRootHandles(true)` displays a handle on “Branch.”

---

### Real-World Cases

- **Forest-like presentations** where multiple top-level categories should appear without a single root.
- **Compact trees** where the root label is redundant.

---

## 9. Tree Events — TreeSelectionListener

### Definitions

**Core Definition.** `TreeSelectionListener` is an interface for receiving `TreeSelectionEvent`s that fire when the tree’s selection changes.

**Technical Definition.** `public interface TreeSelectionListener extends EventListener`. The `valueChanged(TreeSelectionEvent e)` method is invoked when the selection changes. If the user clicks twice on the same node, `valueChanged` is invoked only once .

**Beginner-Friendly Explanation.** A `TreeSelectionListener` runs when the user selects or deselects a node. You use it to react to the new selection.

---

### Purposes

- To react to node selection changes.
- To update detail views or enable buttons based on selection.
- To perform actions when a node is selected.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tree.addTreeSelectionListener(e -> {
    TreePath path = e.getNewLeadSelectionPath();
    // react to new selection
});
```

#### Component Breakdown

| Element | Meaning |
|--------|---------|
| `addTreeSelectionListener(TreeSelectionListener)` | Registers the listener. |
| `valueChanged(TreeSelectionEvent)` | Called on selection change. |
| `getNewLeadSelectionPath()` | Returns the newly selected path. |

#### Syntax Rules

- The listener is invoked on the EDT.
- For double-click or click detection, use a `MouseListener` instead .

#### Constraints and Limitations

- The listener does not distinguish between single and double clicks; use a `MouseListener` for that.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reacting to Node Selection

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.TreePath;

public class SelectionListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("TreeSelectionListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            root.add(new DefaultMutableTreeNode("A"));
            root.add(new DefaultMutableTreeNode("B"));
            JTree tree = new JTree(root);
            JLabel status = new JLabel("Select a node");

            tree.addTreeSelectionListener(e -> {                     // 1. Register
                TreePath path = e.getNewLeadSelectionPath();         // 2. New path
                if (path != null) {
                    status.setText("Selected: " + path.getLastPathComponent());
                }
            });

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(tree), java.awt.BorderLayout.CENTER);
            panel.add(status, java.awt.BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting a node updates the status label with the selected node’s label.

**Why the code produces this result.** The `TreeSelectionListener` reads the new lead selection path and updates the label.

---

### Real-World Cases

- **Master-detail views** that show details of the selected node.
- **Enabling action buttons** only when a node is selected.

---

## 10. Tree Events — TreeExpansionListener

### Definitions

**Core Definition.** `TreeExpansionListener` is an interface for receiving `TreeExpansionEvent`s that fire when a node is expanded or collapsed.

**Technical Definition.** `public interface TreeExpansionListener extends EventListener`. It declares `treeExpanded(TreeExpansionEvent e)` and `treeCollapsed(TreeExpansionEvent e)`. A related `TreeWillExpandListener` can intercept the expansion before it happens .

**Beginner-Friendly Explanation.** A `TreeExpansionListener` runs when a branch is opened or closed. You can use it to load children on demand (lazy loading).

---

### Purposes

- To react to node expansion and collapse.
- To implement lazy loading of children.
- To persist expansion state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tree.addTreeExpansionListener(new TreeExpansionListener() {
    public void treeExpanded(TreeExpansionEvent e) { /* expanded */ }
    public void treeCollapsed(TreeExpansionEvent e) { /* collapsed */ }
});
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `treeExpanded(TreeExpansionEvent)` | Called when a node is expanded. |
| `treeCollapsed(TreeExpansionEvent)` | Called when a node is collapsed. |

#### Syntax Rules

- The listener is invoked on the EDT.
- Use `TreeWillExpandListener` to veto or pre-process expansion.

#### Constraints and Limitations

- The listener fires after the expansion state has changed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Lazy Loading on Expansion

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.DefaultTreeModel;
import javax.swing.event.TreeExpansionEvent;
import javax.swing.event.TreeExpansionListener;

public class LazyLoadDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Lazy Loading");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            DefaultMutableTreeNode lazyNode = new DefaultMutableTreeNode("Lazy");
            root.add(lazyNode);
            DefaultTreeModel model = new DefaultTreeModel(root);
            JTree tree = new JTree(model);

            tree.addTreeExpansionListener(new TreeExpansionListener() { // 1. Listener
                public void treeExpanded(TreeExpansionEvent e) {
                    Object node = e.getPath().getLastPathComponent();
                    DefaultMutableTreeNode treeNode = (DefaultMutableTreeNode) node;
                    if (treeNode == lazyNode && treeNode.getChildCount() == 0) {
                        model.insertNodeInto(new DefaultMutableTreeNode("Loaded 1"),
                            treeNode, 0);                             // 2. Lazy load
                        model.insertNodeInto(new DefaultMutableTreeNode("Loaded 2"),
                            treeNode, 1);
                    }
                }
                public void treeCollapsed(TreeExpansionEvent e) {}
            });

            frame.add(new JScrollPane(tree));
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “Lazy” node initially has no children. When the user expands it, two child nodes are loaded and displayed.

**Why the code produces this result.** The `TreeExpansionListener` detects the expansion, checks if the node is the lazy node and has no children, and then inserts children into the model.

---

### Real-World Cases

- **File explorers** that load directory contents only when a folder is expanded.
- **Large hierarchical datasets** that are loaded incrementally.

---

## 11. Tree Events — TreeModelListener

### Definitions

**Core Definition.** `TreeModelListener` is an interface for receiving `TreeModelEvent`s that fire when the tree’s data model changes (nodes are inserted, removed, or changed).

**Technical Definition.** `public interface TreeModelListener extends EventListener`. It declares `treeNodesChanged`, `treeNodesInserted`, `treeNodesRemoved`, and `treeStructureChanged`. The `JTree` itself uses a `TreeModelListener` internally to keep its view synchronized with the model.

**Beginner-Friendly Explanation.** A `TreeModelListener` runs when the data in the tree changes. You can use it to react to insertions, removals, or value changes.

---

### Purposes

- To react to structural changes in the model.
- To validate or process node changes.
- To synchronize other UI components with the tree’s data.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
tree.getModel().addTreeModelListener(new TreeModelListener() {
    public void treeNodesChanged(TreeModelEvent e) { /* changed */ }
    public void treeNodesInserted(TreeModelEvent e) { /* inserted */ }
    public void treeNodesRemoved(TreeModelEvent e) { /* removed */ }
    public void treeStructureChanged(TreeModelEvent e) { /* structure changed */ }
});
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `treeNodesChanged` | A node’s value changed. |
| `treeNodesInserted` | Nodes were inserted. |
| `treeNodesRemoved` | Nodes were removed. |
| `treeStructureChanged` | The structure changed significantly. |

#### Syntax Rules

- The listener is invoked on the EDT.

#### Constraints and Limitations

- The event does not carry the new values; query the model for details.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting Node Insertion

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.DefaultTreeModel;
import javax.swing.event.TreeModelEvent;
import javax.swing.event.TreeModelListener;

public class ModelListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("TreeModelListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            DefaultTreeModel model = new DefaultTreeModel(root);
            JTree tree = new JTree(model);
            JLabel status = new JLabel("No changes");

            model.addTreeModelListener(new TreeModelListener() {       // 1. Listener
                public void treeNodesChanged(TreeModelEvent e) {}
                public void treeNodesInserted(TreeModelEvent e) {
                    status.setText("Nodes inserted");
                }
                public void treeNodesRemoved(TreeModelEvent e) {}
                public void treeStructureChanged(TreeModelEvent e) {}
            });

            JButton addButton = new JButton("Add Node");
            addButton.addActionListener(e ->
                model.insertNodeInto(new DefaultMutableTreeNode("New"),
                    root, root.getChildCount()));

            JPanel panel = new JPanel(new java.awt.BorderLayout());
            panel.add(new JScrollPane(tree), java.awt.BorderLayout.CENTER);
            panel.add(addButton, java.awt.BorderLayout.SOUTH);
            panel.add(status, java.awt.BorderLayout.NORTH);

            frame.add(panel);
            frame.setSize(300, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Add Node” inserts a node and updates the status label to “Nodes inserted.”

**Why the code produces this result.** The `TreeModelListener` detects the insertion and updates the label.

---

### Real-World Cases

- **Audit logging** of tree changes.
- **Live validation** of tree modifications.

---

## 12. JTree Best Practices

### Definitions

**Core Definition.** `JTree` best practices are the conventions and patterns that ensure trees are used effectively, thread-safely, and performantly in Swing applications.

**Technical Definition.** Best practices include: always wrapping `JTree` in a `JScrollPane`; using `DefaultTreeModel` and `DefaultMutableTreeNode` for simple trees and custom `TreeModel` implementations for complex data; using `TreeSelectionListener` for selection changes and `MouseListener` for double-click detection; implementing lazy loading via `TreeExpansionListener`; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for trees are the “rules of thumb” that prevent frozen GUIs, data inconsistency, and poor user experience. They include things like “always use a scroll pane,” “load children on demand,” and “don’t do heavy work on the GUI thread.”

---

### Purposes

- To ensure the tree scrolls correctly and displays handles.
- To maintain a clean separation between data and presentation.
- To keep the GUI responsive during large data operations.
- To avoid confusion between single and double clicks.
- To create accessible and maintainable trees.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root"); // 1. Root
    DefaultTreeModel model = new DefaultTreeModel(root);             // 2. Model
    JTree tree = new JTree(model);                                    // 3. Tree
    tree.setRootVisible(true);                                        // 4. Visible
    tree.setShowsRootHandles(true);                                   // 5. Handles
    JScrollPane scroll = new JScrollPane(tree);                       // 6. Scroll pane

    tree.addTreeSelectionListener(e -> {                              // 7. Selection
        if (e.getNewLeadSelectionPath() != null) {
            // react
        }
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use a scroll pane | `new JScrollPane(tree)` |
| Use `DefaultTreeModel` for simple trees | `new DefaultTreeModel(root)` |
| Use custom `TreeModel` for complex data | Implement `TreeModel` directly. |
| Use `TreeSelectionListener` for selection | `addTreeSelectionListener` |
| Use `MouseListener` for double-click | Check `e.getClickCount()` |
| Lazy-load children | `TreeExpansionListener` or `TreeWillExpandListener` |
| Avoid `null` layouts | Use layout managers. |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Always wrap `JTree` in a `JScrollPane`** to display handles and scrolling.
- **Always use `DefaultTreeModel` and `DefaultMutableTreeNode`** for simple hierarchies; use a custom `TreeModel` for complex data.
- **Use `TreeSelectionListener` for selection changes** and a `MouseListener` for double-click detection .
- **Implement lazy loading** by adding children in `treeExpanded` rather than pre-populating the entire tree.
- **Never perform heavy work on the EDT**; load children in a background thread and update the model on the EDT.

#### Constraints and Limitations

- `DefaultMutableTreeNode` allows a node to be added to multiple parents, which can cause inconsistencies; use `insertNodeInto` with the model.
- Serialized `JTree` objects are not compatible with future Swing releases .

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Tree with Lazy Loading and Selection

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;
import javax.swing.tree.DefaultTreeModel;
import javax.swing.event.TreeExpansionEvent;
import javax.swing.event.TreeExpansionListener;
import javax.swing.event.TreeSelectionEvent;
import javax.swing.event.TreeSelectionListener;
import java.awt.*;

public class BestPracticeTree {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeTree::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Tree");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root"); // 1. Root
        DefaultMutableTreeNode lazy = new DefaultMutableTreeNode("Lazy Branch");
        root.add(lazy);
        DefaultTreeModel model = new DefaultTreeModel(root);              // 2. Model
        JTree tree = new JTree(model);                                    // 3. Tree
        tree.setShowsRootHandles(true);                                   // 4. Handles

        JLabel status = new JLabel("Select a node");

        tree.addTreeSelectionListener(e -> {                              // 5. Selection
            TreePath path = e.getNewLeadSelectionPath();
            if (path != null) {
                status.setText("Selected: " + path.getLastPathComponent());
            }
        });

        tree.addTreeExpansionListener(new TreeExpansionListener() {       // 6. Lazy
            public void treeExpanded(TreeExpansionEvent e) {
                Object node = e.getPath().getLastPathComponent();
                DefaultMutableTreeNode treeNode = (DefaultMutableTreeNode) node;
                if (treeNode == lazy && treeNode.getChildCount() == 0) {
                    model.insertNodeInto(new DefaultMutableTreeNode("Loaded"),
                        treeNode, 0);
                }
            }
            public void treeCollapsed(TreeExpansionEvent e) {}
        });

        JScrollPane scroll = new JScrollPane(tree);                       // 7. Scroll
        JPanel panel = new JPanel(new BorderLayout());
        panel.add(scroll, BorderLayout.CENTER);
        panel.add(status, BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(350, 300);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A tree with a “Root” node and a “Lazy Branch.” Expanding “Lazy Branch” loads a “Loaded” child. Selecting any node updates the status label.

**Why the code produces this result.** The tree uses a `DefaultTreeModel`, shows root handles, listens for selection changes, and lazy-loads children on expansion. The scroll pane provides scrolling.

---

#### Example 2: Common Mistake — Not Using a Scroll Pane

```java
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;

public class BadTree {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Tree");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultMutableTreeNode root = new DefaultMutableTreeNode("Root");
            root.add(new DefaultMutableTreeNode("Child"));
            JTree tree = new JTree(root);

            // WRONG: no scroll pane
            frame.add(tree);                                          // 1. No scroll

            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The tree may display, but it will not have scroll bars, and the root handles may be partially clipped. The tree’s preferred size may not match the window.

**Why this is wrong.** `JTree` does not provide scrolling or viewport management on its own. Without a `JScrollPane`, the tree cannot scroll when its content exceeds the visible area, and handles may not render correctly.

---

### Real-World Cases

- **Production applications** wrap every `JTree` in a `JScrollPane` and use lazy loading for large hierarchies.
- **File explorers** use `TreeExpansionListener` to load directory contents on demand.
- **Educational examples** use the best-practice skeleton to demonstrate correct tree usage from the start.

---

## References

Oracle. *Class JTree*. Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JTree.html 

Oracle. *How to Use Trees*. The Java Tutorials. http://dev.cs.ovgu.de/java/Tutorials/java/uiswing/components/tree.html 

Oracle. *Class DefaultMutableTreeNode*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/DefaultMutableTreeNode.html

Oracle. *Class DefaultTreeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/DefaultTreeModel.html

Oracle. *Interface TreeModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/tree/TreeModel.html

Oracle. *Interface TreeSelectionListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/TreeSelectionListener.html

Oracle. *Interface TreeExpansionListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/TreeExpansionListener.html

Oracle. *Interface TreeModelListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/TreeModelListener.html

Stack Overflow. *JTree of objects*. https://stackoverflow.com/questions/2413896/jtree-of-objects/2414002 

Oracle. *JTree setRootVisible and setShowsRootHandles*. Java Platform, Standard Edition 7 API Specification. https://docs.oracle.com/javase/7/docs/api/javax/swing/JTree.html 