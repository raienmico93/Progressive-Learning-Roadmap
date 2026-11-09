# JFileChooser — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JFileChooser` is a Swing component that provides a simple, standard mechanism for the user to choose a file or a directory from the file system. It presents a dialog box that lets the user browse directories, filter visible files, and select one or more files for opening or saving.

**Technical Definition.** `public class JFileChooser extends JComponent implements Accessible`. A `JFileChooser` is a specialized component that allows the user to browse the file system and select a file. The easiest way to use it is with the `showOpenDialog()` and `showSaveDialog()` methods, which differ only in the text that appears on the approve button. Each of these methods returns an integer status code indicating how the user dismissed the dialog: `APPROVE_OPTION`, `CANCEL_OPTION`, or `ERROR_OPTION`. The component is not thread-safe and must be accessed on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `JFileChooser` is the familiar “Open File” or “Save File” dialog you see in almost every desktop application. When your program needs to know which file to read from or write to, you pop up a `JFileChooser`. It lets the user navigate folders, see only the file types you care about (like `.jpg` or `.txt`), and pick a file. You then ask the chooser which file the user selected and proceed with your file operation. Think of it as a standardized, OS-friendly file browser that you do not have to build from scratch.

---

### Key Characteristics

1. **Three dialog modes.** `showOpenDialog()` presents an “Open” dialog, `showSaveDialog()` presents a “Save” dialog, and `showDialog()` presents a custom dialog with a programmer-supplied approve-button text.
2. **Status-code return values.** Every show method returns an integer status: `APPROVE_OPTION` (user confirmed a selection), `CANCEL_OPTION` (user cancelled), or `ERROR_OPTION` (an error occurred or the dialog was dismissed).
3. **File filtering.** `setFileFilter()` sets a single active filter, while `addChoosableFileFilter()` adds multiple filters that the user can switch between. `FileNameExtensionFilter` is the standard convenience implementation for filtering by file extension.
4. **Selection mode.** `setFileSelectionMode()` controls whether the user can select files only (`FILES_ONLY`), directories only (`DIRECTORIES_ONLY`), or both (`FILES_AND_DIRECTORIES`).
5. **Multi-selection.** `setMultiSelectionEnabled(true)` allows the user to select multiple files in a single dialog session.
6. **Directory memory.** When a `JFileChooser` instance is reused, it automatically remembers the most recently selected directory.
7. **Customization hooks.** `FileView`, `FileSystemView`, and `setAccessory()` allow deep customization of how files are displayed and what additional UI appears in the dialog.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with the `java.io.File` class and file system concepts.
- Understanding of the EDT and `SwingUtilities.invokeLater` (all Swing code, including file choosers, must run on the EDT).
- Knowledge of `ActionListener` and the delegation event model.
- A JDK (version 8 or later; `JFileChooser` is available in all modern JDKs).

---

### Related Programming Areas

- **`java.io.File`:** The fundamental abstraction for files and directories that `JFileChooser` returns.
- **`FileNameExtensionFilter` and `FileFilter`:** The filtering mechanism for restricting which files are visible.
- **`FileView` and `FileSystemView`:** Advanced customization of file display and OS-specific file system behavior.
- **`JDialog`:** The underlying window type used to present the file chooser.
- **Swing threading policy:** The rule that all Swing component construction and access must occur on the EDT.

---

## Core Concepts and Features

---

## 1. JFileChooser — The Class

### Definitions

**Core Definition.** `JFileChooser` is the Swing class for creating a standard file-selection dialog.

**Technical Definition.** `public class JFileChooser extends JComponent implements Accessible`. It provides constructors that accept an initial directory (as a `File`, a `String` path, or a `FileSystemView`). The dialog is displayed via `showOpenDialog`, `showSaveDialog`, or `showDialog`, and the selected file is retrieved via `getSelectedFile` or `getSelectedFiles`.

**Beginner-Friendly Explanation.** `JFileChooser` is the class you use to show a file dialog. You create it, optionally configure filters and a starting directory, call a show method, and then read the result.

---

### Purposes

- To provide a standard, OS-consistent file-selection dialog for opening and saving files.
- To restrict the visible files to relevant types via filters.
- To allow single or multiple file selection based on the configured mode.
- To remember the user’s last directory when the chooser is reused.
- To serve as the standard Swing solution for file-system interaction without building a custom browser.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation and Display)

```java
JFileChooser chooser = new JFileChooser();
int result = chooser.showOpenDialog(parentComponent);
if (result == JFileChooser.APPROVE_OPTION) {
    File selectedFile = chooser.getSelectedFile();
    // process the file
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `new JFileChooser()` | Creates a chooser starting in the user’s default directory. |
| `new JFileChooser(File currentDirectory)` | Creates a chooser starting in the specified directory. |
| `showOpenDialog(Component parent)` | Displays an “Open” dialog; returns the status code. |
| `showSaveDialog(Component parent)` | Displays a “Save” dialog; returns the status code. |
| `showDialog(Component parent, String approveButtonText)` | Displays a custom dialog with the given approve-button text. |
| `getSelectedFile()` | Returns the selected `File` (or `null` if none). |
| `APPROVE_OPTION` | Constant indicating the user confirmed the selection. |

#### Syntax Rules

- The `parentComponent` argument positions the dialog relative to the parent; passing `null` centers it on the screen.
- All `show*` methods are blocking (modal) and must be called on the EDT.
- `getSelectedFile()` is only meaningful when the return value is `APPROVE_OPTION`.

#### Constraints and Limitations

- The dialog blocks the EDT while open; do not perform long operations inside the same EDT task before the dialog closes.
- `JFileChooser` is not thread-safe; all construction and configuration must occur on the EDT.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Open Dialog

```java
import javax.swing.*;
import java.io.File;

public class MinimalFileChooser {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JFileChooser");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();              // 1. Create
            int result = chooser.showOpenDialog(frame);             // 2. Show
            if (result == JFileChooser.APPROVE_OPTION) {            // 3. Check
                File selected = chooser.getSelectedFile();           // 4. Get file
                System.out.println("You chose: " + selected.getAbsolutePath());
            } else {
                System.out.println("No file selected.");
            }
        });
    }
}
```

**Expected Output.** An “Open” file dialog appears. If the user selects a file and clicks “Open,” the console prints the file’s absolute path. If the user cancels, it prints “No file selected.”

**Why the code produces this result.** `showOpenDialog` returns `APPROVE_OPTION` when the user confirms a selection. `getSelectedFile` then returns the corresponding `File` object.

---

#### Example 2: Save Dialog

```java
import javax.swing.*;
import java.io.File;

public class SaveFileChooser {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Save Dialog");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();
            int result = chooser.showSaveDialog(frame);            // 1. Save dialog
            if (result == JFileChooser.APPROVE_OPTION) {
                File file = chooser.getSelectedFile();              // 2. Target file
                System.out.println("Saving to: " + file.getAbsolutePath());
            }
        });
    }
}
```

**Expected Output.** A “Save” dialog appears. The user types a file name and clicks “Save,” and the console prints the chosen save path.

**Why the code produces this result.** `showSaveDialog` presents the same dialog as `showOpenDialog` but with the approve button labeled “Save.” `getSelectedFile` returns the file the user named or selected.

---

### Real-World Cases

- **Text editors** prompting the user to choose a file to open or a location to save.
- **Image viewers** letting the user browse for image files with an image filter applied.
- **Data import tools** selecting CSV or Excel files for processing.
- **Export features** choosing a destination file for reports, PDFs, or backups.

---

## 2. File Chooser Creation

### Definitions

**Core Definition.** File chooser creation is the process of instantiating a `JFileChooser` with an optional starting directory and optional `FileSystemView`.

**Technical Definition.** `JFileChooser` provides four constructors: `JFileChooser()`, `JFileChooser(File currentDirectory)`, `JFileChooser(String currentDirectoryPath)`, and `JFileChooser(File currentDirectory, FileSystemView fsv)`. The no-argument constructor starts in the user’s default directory, which is OS-dependent (e.g., “My Documents” on Windows, the home directory on Unix).

**Beginner-Friendly Explanation.** Creating a file chooser means deciding where it should start—typically the user’s home directory or the last directory they used.

---

### Purposes

- To instantiate the chooser with a sensible starting directory.
- To prepare the chooser for filter, selection-mode, and multi-selection configuration.
- To reuse a chooser instance across multiple invocations so it remembers the last directory.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JFileChooser chooser = new JFileChooser();
JFileChooser chooser = new JFileChooser(new File(System.getProperty("user.home")));
JFileChooser chooser = new JFileChooser(lastDirectory);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JFileChooser()` | Starts in the default directory (OS-dependent). |
| `JFileChooser(File dir)` | Starts in the specified directory. |
| `JFileChooser(String path)` | Starts in the specified directory path. |
| `JFileChooser(File dir, FileSystemView fsv)` | Uses a custom file system view. |

#### Syntax Rules

- The constructor must run on the EDT.
- If the specified directory does not exist, the chooser falls back to the default directory.

#### Constraints and Limitations

- Creating a new chooser for every invocation loses the “last directory” memory; reuse the instance for better UX.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Starting in a Specific Directory

```java
import javax.swing.*;
import java.io.File;

public class SpecificDirectoryChooser {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Specific Directory");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            File startDir = new File(System.getProperty("user.home"));
            JFileChooser chooser = new JFileChooser(startDir);    // 1. Start dir
            int result = chooser.showOpenDialog(frame);
            if (result == JFileChooser.APPROVE_OPTION) {
                System.out.println("Selected: " + chooser.getSelectedFile());
            }
        });
    }
}
```

**Expected Output.** The dialog opens in the user’s home directory.

**Why the code produces this result.** The `File` constructor sets the initial current directory to the user’s home.

---

### Real-World Cases

- **Project-based tools** starting in the project’s working directory.
- **Preference dialogs** starting in the last-used directory stored in preferences.

---

## 3. File Chooser Methods — showOpenDialog()

### Definitions

**Core Definition.** `showOpenDialog(Component parent)` displays an “Open File” dialog and returns a status code indicating how the user dismissed it.

**Technical Definition.** `public int showOpenDialog(Component parent) throws HeadlessException`. The method pops up an “Open File” chooser dialog and returns `JFileChooser.APPROVE_OPTION`, `JFileChooser.CANCEL_OPTION`, or `JFileChooser.ERROR_OPTION`. The `parent` argument determines the dialog’s position and modality owner.

**Beginner-Friendly Explanation.** `showOpenDialog` shows the standard “Open” dialog. You call it, check the return value, and if the user clicked “Open,” you read the selected file.

---

### Purposes

- To let the user select an existing file for reading.
- To provide a standard, accessible file-browsing experience.
- To return a clear status code for branching logic.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int result = chooser.showOpenDialog(parentComponent);
if (result == JFileChooser.APPROVE_OPTION) {
    File file = chooser.getSelectedFile();
    // open the file
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `showOpenDialog(Component)` | Displays the “Open” dialog. |
| `APPROVE_OPTION` | User confirmed a selection. |
| `CANCEL_OPTION` | User cancelled the dialog. |
| `ERROR_OPTION` | An error occurred, or the dialog was dismissed without a clear action. |

#### Syntax Rules

- Must be called on the EDT.
- The `parent` may be `null` (centers on screen) or a `Component` (centers over the component).
- Always check the return value before calling `getSelectedFile()`.

#### Constraints and Limitations

- The dialog is modal; it blocks the EDT until the user responds.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Open Dialog with Status Handling

```java
import javax.swing.*;
import java.io.File;

public class OpenDialogDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Open Dialog");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();
            int result = chooser.showOpenDialog(frame);             // 1. Show open
            switch (result) {
                case JFileChooser.APPROVE_OPTION:
                    File file = chooser.getSelectedFile();           // 2. Get file
                    System.out.println("Opening: " + file.getName());
                    break;
                case JFileChooser.CANCEL_OPTION:
                    System.out.println("Open cancelled.");
                    break;
                default:
                    System.out.println("Error or dialog dismissed.");
            }
        });
    }
}
```

**Expected Output.** Selecting a file prints “Opening: [filename].” Cancelling prints “Open cancelled.” Any other outcome prints an error message.

**Why the code produces this result.** The switch handles all three possible return values from `showOpenDialog`.

---

### Real-World Cases

- **Document editors** selecting a file to open.
- **Media players** selecting a media file to play.
- **Import wizards** selecting a data file to read.

---

## 4. File Chooser Methods — showSaveDialog()

### Definitions

**Core Definition.** `showSaveDialog(Component parent)` displays a “Save File” dialog and returns a status code indicating how the user dismissed it.

**Technical Definition.** `public int showSaveDialog(Component parent)`. The method behaves identically to `showOpenDialog` except that the approve button is labeled “Save” and the dialog’s title reflects a save operation.

**Beginner-Friendly Explanation.** `showSaveDialog` shows the “Save As” dialog. The user picks a location and a file name, and you receive the `File` object representing the target.

---

### Purposes

- To let the user choose a destination file for saving data.
- To allow the user to create a new file by typing a name.
- To provide a consistent save-experience across the application.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
int result = chooser.showSaveDialog(parentComponent);
if (result == JFileChooser.APPROVE_OPTION) {
    File target = chooser.getSelectedFile();
    // write to target
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `showSaveDialog(Component)` | Displays the “Save” dialog. |
| `getSelectedFile()` | Returns the file the user named or selected. |

#### Syntax Rules

- Must be called on the EDT.
- The returned `File` may not yet exist; the user may have typed a new name.
- Check for overwrite confirmation separately if the file already exists.

#### Constraints and Limitations

- The dialog does not automatically append a file extension; the programmer must handle that.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Save Dialog with Overwrite Check

```java
import javax.swing.*;
import java.io.File;

public class SaveDialogDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Save Dialog");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();
            int result = chooser.showSaveDialog(frame);             // 1. Show save
            if (result == JFileChooser.APPROVE_OPTION) {
                File file = chooser.getSelectedFile();               // 2. Target
                if (file.exists()) {                                 // 3. Overwrite?
                    int confirm = JOptionPane.showConfirmDialog(
                        frame,
                        "File exists. Overwrite?",
                        "Confirm",
                        JOptionPane.YES_NO_OPTION);
                    if (confirm != JOptionPane.YES_OPTION) {
                        return;
                    }
                }
                System.out.println("Saving to: " + file.getAbsolutePath());
            }
        });
    }
}
```

**Expected Output.** If the user selects an existing file, a confirmation dialog asks whether to overwrite. Choosing “No” aborts the save.

**Why the code produces this result.** `showSaveDialog` returns the target `File`; the programmer checks `file.exists()` and prompts accordingly.

---

### Real-World Cases

- **Text editors** saving a new document or “Save As.”
- **Export features** choosing a destination for PDFs or reports.
- **Backup tools** selecting a backup destination.

---

## 5. File Chooser Methods — getSelectedFile() / getSelectedFiles()

### Definitions

**Core Definition.** `getSelectedFile()` returns the currently selected `File`. `getSelectedFiles()` returns an array of `File` objects when multi-selection is enabled.

**Technical Definition.** `public File getSelectedFile()` and `public File[] getSelectedFiles()`. `getSelectedFile` returns `null` if nothing is selected. `getSelectedFiles` returns an empty array if nothing is selected; it is only meaningful when `setMultiSelectionEnabled(true)` has been called.

**Beginner-Friendly Explanation.** After the user confirms a selection, `getSelectedFile` tells you which file they picked. If you allowed multiple selection, `getSelectedFiles` gives you all of them.

---

### Purposes

- To retrieve the single file the user selected for opening or saving.
- To retrieve multiple files when the chooser is configured for multi-selection.
- To feed the selected file(s) into subsequent file I/O operations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
File single = chooser.getSelectedFile();
File[] multiple = chooser.getSelectedFiles();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSelectedFile()` | Returns the selected `File`, or `null`. |
| `getSelectedFiles()` | Returns an array of selected `File`s (multi-selection mode). |

#### Syntax Rules

- Only call after `APPROVE_OPTION` is returned.
- `getSelectedFiles` returns an empty array (not `null`) when nothing is selected.

#### Constraints and Limitations

- `getSelectedFiles` is only populated when multi-selection is enabled.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading the Selected File

```java
import javax.swing.*;
import java.io.File;

public class GetSelectedFileDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Get Selected File");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();
            int result = chooser.showOpenDialog(frame);
            if (result == JFileChooser.APPROVE_OPTION) {
                File file = chooser.getSelectedFile();               // 1. Get file
                System.out.println("Path: " + file.getAbsolutePath());
                System.out.println("Exists: " + file.exists());
            }
        });
    }
}
```

**Expected Output.** After the user selects a file, the console prints its absolute path and whether it exists on disk.

**Why the code produces this result.** `getSelectedFile` returns the `File` object corresponding to the user’s choice.

---

### Real-World Cases

- **File readers** opening the selected file for input.
- **Batch processors** iterating over multiple selected files.

---

## 6. File Chooser Methods — setMultiSelectionEnabled()

### Definitions

**Core Definition.** `setMultiSelectionEnabled(boolean b)` controls whether the user can select more than one file at a time in the file chooser.

**Technical Definition.** `public void setMultiSelectionEnabled(boolean b)`. By default, only one file can be selected. When enabled, the user may select multiple files (using Ctrl-click or Shift-click, depending on the OS), and `getSelectedFiles()` returns all selected files.

**Beginner-Friendly Explanation.** `setMultiSelectionEnabled(true)` lets the user pick several files at once instead of just one.

---

### Purposes

- To allow batch operations on multiple files.
- To support features like “Open multiple images” or “Import multiple CSVs.”
- To reduce the number of dialog interactions for the user.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
chooser.setMultiSelectionEnabled(true);
File[] files = chooser.getSelectedFiles();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setMultiSelectionEnabled(boolean)` | Enables or disables multi-file selection. |
| `isMultiSelectionEnabled()` | Returns the current setting. |

#### Syntax Rules

- Must be set before showing the dialog.
- When enabled, `getSelectedFiles()` should be used instead of `getSelectedFile()`.

#### Constraints and Limitations

- Not all look-and-feel implementations fully support multi-selection rendering, though the selection logic works.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Selection Open Dialog

```java
import javax.swing.*;
import java.io.File;

public class MultiSelectionDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Multi-Selection");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();
            chooser.setMultiSelectionEnabled(true);                 // 1. Enable
            int result = chooser.showOpenDialog(frame);
            if (result == JFileChooser.APPROVE_OPTION) {
                File[] files = chooser.getSelectedFiles();           // 2. Get all
                for (File f : files) {
                    System.out.println("Selected: " + f.getName());
                }
            }
        });
    }
}
```

**Expected Output.** The user can select multiple files (Ctrl-click). After confirming, the console prints the name of each selected file.

**Why the code produces this result.** `setMultiSelectionEnabled(true)` allows multiple selection, and `getSelectedFiles()` returns the array.

---

### Real-World Cases

- **Image galleries** importing multiple photos at once.
- **Data pipelines** selecting multiple input files for batch processing.
- **Music players** adding multiple tracks to a playlist.

---

## 7. File Chooser Methods — setFileFilter()

### Definitions

**Core Definition.** `setFileFilter(FileFilter filter)` sets the active file filter, restricting the visible files to those accepted by the filter.

**Technical Definition.** `public void setFileFilter(FileFilter filter)`. The `FileFilter` is an abstract class in `javax.swing.filechooser` with `accept(File f)` and `getDescription()` methods. `FileNameExtensionFilter` is a convenient concrete implementation that filters by file extension.

**Beginner-Friendly Explanation.** `setFileFilter` tells the file chooser to show only certain types of files—for example, only `.jpg` and `.png` images.

---

### Purposes

- To reduce visual clutter by hiding irrelevant files.
- To guide the user toward the correct file type.
- To prevent the user from accidentally selecting incompatible files.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
FileNameExtensionFilter filter =
    new FileNameExtensionFilter("Images", "jpg", "jpeg", "png", "gif");
chooser.setFileFilter(filter);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `FileNameExtensionFilter(String description, String... extensions)` | Creates an extension-based filter. |
| `setFileFilter(FileFilter)` | Sets the active filter. |
| `addChoosableFileFilter(FileFilter)` | Adds a filter to the user-selectable list. |

#### Syntax Rules

- `setFileFilter` replaces the current filter; `addChoosableFileFilter` adds an additional option.
- The “Accept All” filter is present by default and can be removed with `setAcceptAllFileFilterUsed(false)`.

#### Constraints and Limitations

- `FileNameExtensionFilter` matches by extension only; content-based filtering requires a custom `FileFilter`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Image Filter

```java
import javax.swing.*;
import javax.swing.filechooser.FileNameExtensionFilter;
import java.io.File;

public class FilterDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Filter Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();
            FileNameExtensionFilter filter = new FileNameExtensionFilter(
                "Image Files", "jpg", "jpeg", "png", "gif");         // 1. Filter
            chooser.setFileFilter(filter);                            // 2. Apply

            int result = chooser.showOpenDialog(frame);
            if (result == JFileChooser.APPROVE_OPTION) {
                File file = chooser.getSelectedFile();
                System.out.println("Image: " + file.getName());
            }
        });
    }
}
```

**Expected Output.** The dialog shows only files with `.jpg`, `.jpeg`, `.png`, or `.gif` extensions.

**Why the code produces this result.** The `FileNameExtensionFilter` accepts files matching the listed extensions, and `setFileFilter` makes it the active filter.

---

### Real-World Cases

- **Image editors** showing only image formats.
- **Text editors** showing `.txt`, `.md`, `.java`, etc.
- **Audio applications** showing `.mp3`, `.wav`, `.ogg`.

---

## 8. File Chooser Methods — setCurrentDirectory()

### Definitions

**Core Definition.** `setCurrentDirectory(File dir)` sets the directory that the file chooser displays when it opens.

**Technical Definition.** `public void setCurrentDirectory(File dir)`. The method changes the current directory to the specified `File`. If the directory is invalid, the chooser may fall back to the previous or default directory.

**Beginner-Friendly Explanation.** `setCurrentDirectory` tells the file chooser which folder to open in.

---

### Purposes

- To start the dialog in a contextually relevant directory.
- To restore the last-used directory from preferences.
- To guide the user to a specific project or data folder.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
chooser.setCurrentDirectory(new File("/path/to/dir"));
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setCurrentDirectory(File)` | Sets the directory shown when the dialog opens. |
| `getCurrentDirectory()` | Returns the current directory. |

#### Syntax Rules

- Must be called before `show*Dialog`.
- The directory should exist for reliable behavior.

#### Constraints and Limitations

- If the directory does not exist or is not accessible, the chooser may use a default.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Starting in a Project Directory

```java
import javax.swing.*;
import java.io.File;

public class CurrentDirectoryDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Current Directory");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JFileChooser chooser = new JFileChooser();
            chooser.setCurrentDirectory(new File("src"));           // 1. Set dir
            int result = chooser.showOpenDialog(frame);
            if (result == JFileChooser.APPROVE_OPTION) {
                System.out.println("Selected: " + chooser.getSelectedFile());
            }
        });
    }
}
```

**Expected Output.** The dialog opens in the `src` directory relative to the working directory.

**Why the code produces this result.** `setCurrentDirectory` changes the initial directory displayed by the chooser.

---

### Real-World Cases

- **IDE plugins** opening the chooser in the project root.
- **Configuration tools** starting in the application’s config directory.

---

## 9. JFileChooser Best Practices

### Definitions

**Core Definition.** `JFileChooser` best practices are the conventions and patterns that ensure file dialogs are used safely, thread-correctly, and with good user experience.

**Technical Definition.** Best practices include: always creating and showing the chooser on the EDT; reusing a single chooser instance to leverage directory memory; applying file filters to guide the user; checking the return status before accessing the selected file; handling overwrite confirmation in save dialogs; and never performing long-running file I/O on the EDT.

**Beginner-Friendly Explanation.** Best practices for file choosers are the “rules of thumb” that prevent frozen UIs, wrong-thread errors, and confusing user experiences. They include things like “always use the EDT,” “reuse the chooser,” “check the result before using the file,” and “do the actual file reading/writing in a background thread.”

---

### Purposes

- To maintain thread safety by respecting Swing’s EDT rule.
- To provide a consistent directory experience by reusing the chooser instance.
- To improve usability through appropriate filters and selection modes.
- To prevent accidental data loss via overwrite confirmation.
- To keep the UI responsive by offloading file I/O to background threads.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JFileChooser chooser = new JFileChooser();                      // 1. Reusable
    chooser.setFileFilter(new FileNameExtensionFilter(
        "Text Files", "txt", "md"));                                 // 2. Filter

    int result = chooser.showOpenDialog(parent);                    // 3. Show on EDT
    if (result == JFileChooser.APPROVE_OPTION) {                    // 4. Check status
        File file = chooser.getSelectedFile();
        new SwingWorker<Void, Void>() {                             // 5. Off-EDT I/O
            @Override
            protected Void doInBackground() throws Exception {
                // read the file here (not on EDT)
                return null;
            }
        }.execute();
    }
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT construction and display | `SwingUtilities.invokeLater(...)` around the chooser creation and `show*Dialog`. |
| Reuse the chooser | Keep a field reference; call `show*Dialog` multiple times. |
| Apply a filter | `setFileFilter(new FileNameExtensionFilter(...))`. |
| Check the status code | `if (result == JFileChooser.APPROVE_OPTION)`. |
| Handle overwrite | `if (file.exists())` and confirm with the user. |
| Offload file I/O | Use `SwingWorker` or a background thread for reading/writing. |
| Set a sensible directory | `setCurrentDirectory(...)` or rely on the chooser’s memory. |

#### Syntax Rules

- **Always construct and show `JFileChooser` on the EDT.** The component is not thread-safe.
- **Always check the return value** of `show*Dialog` before calling `getSelectedFile()`.
- **Reuse a single `JFileChooser` instance** across invocations to preserve the last-visited directory.
- **Never perform file I/O on the EDT** after the dialog closes; use a background thread.
- **Always configure a filter** when the application expects a specific file type.

#### Constraints and Limitations

- `JFileChooser` is modal; it blocks the EDT while open, but this is by design and does not freeze the application.
- Multi-selection is not fully rendered by all look-and-feel implementations, though the selection data is correct.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice File Open with Background Reading

```java
import javax.swing.*;
import javax.swing.filechooser.FileNameExtensionFilter;
import java.io.File;
import java.nio.file.Files;

public class BestPracticeFileChooser {
    private static final JFileChooser chooser = new JFileChooser(); // 1. Reusable

    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeFileChooser::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice FileChooser");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);

        chooser.setFileFilter(new FileNameExtensionFilter(
            "Text Files", "txt", "md", "log"));                       // 2. Filter

        JButton openButton = new JButton("Open File");
        openButton.addActionListener(e -> {
            int result = chooser.showOpenDialog(frame);               // 3. Show
            if (result == JFileChooser.APPROVE_OPTION) {              // 4. Check
                File file = chooser.getSelectedFile();
                new SwingWorker<String, Void>() {                     // 5. Background
                    @Override
                    protected String doInBackground() throws Exception {
                        return Files.readString(file.toPath());
                    }
                    @Override
                    protected void done() {
                        try {
                            System.out.println("File length: " + get().length());
                        } catch (Exception ex) {
                            ex.printStackTrace();
                        }
                    }
                }.execute();
            }
        });

        frame.add(openButton);
    }
}
```

**Expected Output.** The user selects a text file, and the file is read in a background thread. The console prints the file’s length in characters without freezing the UI.

**Why the code produces this result.** The chooser is reused, a filter restricts file types, the status code is checked, and the actual file reading is offloaded to a `SwingWorker` so the EDT remains responsive.

---

#### Example 2: Common Mistake — Showing the Chooser Off the EDT

```java
import javax.swing.*;
import java.io.File;

public class BadFileChooser {
    public static void main(String[] args) {
        // WRONG: not on the EDT
        JFileChooser chooser = new JFileChooser();                    // 1. Off-EDT
        int result = chooser.showOpenDialog(null);                    // 2. Off-EDT
        if (result == JFileChooser.APPROVE_OPTION) {
            File file = chooser.getSelectedFile();
            System.out.println(file);
        }
    }
}
```

**Expected Output.** The dialog may appear, but it can behave unpredictably, throw concurrency-related exceptions, or fail to render correctly because Swing components are not thread-safe.

**Why this is wrong.** All Swing components, including `JFileChooser`, must be constructed and accessed on the EDT. Calling the chooser from the main thread violates Swing’s threading policy and can cause intermittent, hard-to-reproduce bugs.

---

### Real-World Cases

- **Production Swing applications** reuse a single `JFileChooser` instance and wrap its construction in `SwingUtilities.invokeLater` for thread safety.
- **Enterprise tools** apply filters and overwrite confirmation to prevent user errors.
- **Educational examples** use the best-practice skeleton to demonstrate correct threading and file-handling patterns from the start.

---

## References

Oracle. *Class JFileChooser*. Java Development Kit Version 17 API Specification. https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/javax/swing/JFileChooser.html

Oracle. *How to Use File Choosers*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/filechooser.html

O'Reilly. *JFileChooser — Java Swing ( chapter 3.16 )*. http://www.cs.ait.ac.th/~on/O/oreilly/java-ent/jfc/ch03_16.htm

Stack Overflow. *Using JFileChooser — showOpenDialog, setFileFilter, getSelectedFile*. https://stackoverflow.com/posts/26317840/revisions

Stack Overflow. *JFileChooser best practices — Swing threading and EDT*. https://stackoverflow.com/revisions/...

Oracle. *FileNameExtensionFilter (Java Platform SE 8)*. https://docs.oracle.com/javase/8/docs/api/javax/swing/filechooser/FileNameExtensionFilter.html

Oracle. *JFileChooser.showOpenDialog() return values*. https://docs.oracle.com/javase/8/docs/api/javax/swing/JFileChooser.html#showOpenDialog-java.awt.Component-