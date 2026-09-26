# Git Tracking Files: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
File tracking in Git is the process of moving changes from the working tree into the staging area (index) so that they become part of the next commit snapshot.

**Technical Definition**
Tracking files involves updating the index—a binary file at `.git/index`—with blob object references and file metadata (mode, stat information, path) using the `git add` command. The index serves as the proposed next commit. Files can be unstaged using `git restore --staged` (Git 2.23+) or the legacy `git reset`. Interactive staging (`git add -p`) allows selective staging at the hunk or line level, enabling atomic, logically separated commits.

**Beginner-Friendly Explanation**
Tracking files is how you tell Git, "I want to include these changes in my next save point." You use `git add` to move changes from your working folder into a "holding zone" (the staging area). When you're ready, `git commit` saves exactly what's in that holding zone. You can add whole files, specific folders, or even just a few lines within a file.

### Key Characteristics

- **Three-Tree Architecture**: Working tree (editable files) → Index (staged snapshot) → HEAD (committed history).
- **Selective Staging**: You control precisely which changes go into each commit.
- **Content-Addressable**: `git add` stores compressed blob objects in `.git/objects/`.
- **Index as Blueprint**: The index is a binary file that records the exact file state for the next commit.
- **Atomic Commits**: Staging enables small, focused commits that are easy to review and revert.

### Prerequisites

- Git installed (version 2.23+ recommended for `git restore`; any version for `git reset`).
- A Git repository initialised or cloned.
- Files present in the working tree (tracked, modified, or untracked).
- Basic familiarity with `git status` and the terminal.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Automated builds that stage and commit artifacts.
- **Software Configuration Management (SCM)** : Auditing and change control.
- **Collaborative Development**: Preparing clean commits for code review.

### Core Concepts / Features

1. **git add** — Adding file contents to the index.
2. **Adding Individual Files** — Staging a single file by path.
3. **Adding Multiple Files** — Staging several files with globs or lists.
4. **Adding Directories** — Recursively staging all files in a directory.
5. **Understanding Staging** — The index as a commit blueprint.
6. **Unstaging Files** — `git restore --staged` (modern) and `git reset` (legacy).
7. **Interactive Staging** — `git add -p` for hunk-level precision.

---

## 1. git add

### Definitions

**Core Definition**
`git add` is the command that adds file contents from the working tree to the staging area (index), preparing them for the next commit.

**Technical Definition**
`git add` updates the index using the current content found in the working tree. It computes the blob object for each specified path, writes the object to the object database (`.git/objects/`), and records the blob's object ID along with file metadata (mode, stat information) in the index. The command can be performed multiple times before a commit; each invocation adds the *current* content of the specified files at that moment. Ignored files are not added by default; the `-f` / `--force` option overrides this.

**Beginner-Friendly Explanation**
`git add` is how you tell Git, "I want to include these changes in my next snapshot." It copies the current version of your files into a holding zone called the staging area. You can run it as many times as you like—each time, Git captures whatever the file looks like right then. When you're ready, `git commit` saves everything in the staging area as a permanent snapshot.

### Purposes

- To start tracking a new (untracked) file.
- To stage modifications made to an already-tracked file.
- To stage deletions or renames.
- To prepare a curated set of changes for the next commit.
- To build up a commit incrementally, file by file or hunk by hunk.

### Syntax Rules and Structure

**Complete General Syntax**

```
git add [--verbose | -v] [--dry-run | -n] [--force | -f] [--interactive | -i]
        [--patch | -p] [--edit | -e] [--[no-]all | -A] [--[no-]ignore-removal | --ignore-removal]
        [--intent-to-add | -N] [--refresh] [--ignore-errors] [--ignore-missing]
        [--sparse] [--chmod=(+|-)x] [--pathspec-from-file=<file>]
        [--pathspec-file-nul] [--] [<pathspec>...]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `<pathspec>` | Files, directories, or patterns to stage. |
| `-A` / `--all` | Stage all changes (new, modified, deleted) across the entire working tree. |
| `-u` / `--update` | Stage modifications and deletions, but not new files. |
| `-p` / `--patch` | Interactively select hunks to stage. |
| `-i` / `--interactive` | Enter interactive staging shell. |
| `-n` / `--dry-run` | Show what *would* be added without actually adding. |
| `-f` / `--force` | Add ignored files. |
| `-v` / `--verbose` | Be verbose. |
| `-N` / `--intent-to-add` | Record only the fact that the path will be added later. |

**Syntax Rules**

- The `<pathspec>` can be a file, a directory, a glob pattern, or a `.` (current directory).
- When a directory is given, the operation is applied recursively to all files and subdirectories within it. Git handles this recursion internally; there is no need for a separate `-r` flag.
- Git's directory recursion and filename globbing **silently ignore** files matched by `.gitignore` unless `-f` is used.
- The command can be run multiple times; each run adds the content as it exists at that moment.
- If no `<pathspec>` is given with `-A` or `-u`, the entire working tree is affected.

**Constraints and Limitations**

- **Ignored files**: Files matching `.gitignore` or `.git/info/exclude` are not added by default. Use `-f` to force-add them.
- **Stat information**: `git add` records the file's stat information (mtime, size) to quickly detect future modifications. If a file is touched without content change, `git add` may still update the index entry.
- **Large files**: Adding very large files can bloat the repository. Git LFS is recommended for large binaries.
- **Version-specific**: The `--sparse` option requires Git 2.35+. The `--pathspec-from-file` option requires Git 2.25+.
- **Index corruption**: If the index is corrupted, `git add` may fail. Run `git reset` to rebuild it.

### Annotated Code Examples

#### Example 1: Adding a New File

```bash
# Create a new repository and a new file
mkdir add-demo && cd add-demo && git init
echo "Hello, Git!" > hello.txt

# Check status (file is untracked)
git status -s
# Output: ?? hello.txt

# Stage the file
git add hello.txt

# Check status (file is staged)
git status -s
# Output: A  hello.txt

# Inspect the index
git ls-files --stage
# Output: 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	hello.txt
```

**Expected Output:**
```
?? hello.txt
A  hello.txt
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	hello.txt
```

**Why this output occurs:** Initially, `hello.txt` is untracked (`??`). After `git add`, the file is staged (`A ` in the first column). `git ls-files --stage` confirms the index now contains one entry for `hello.txt` with mode `100644` and the blob's SHA-1 hash.

#### Example 2: Staging a Modification

```bash
# Commit the file first
git commit -m "Add hello.txt"

# Modify the file
echo "Second line" >> hello.txt

# Check status (modified, not staged)
git status -s
# Output:  M hello.txt

# Stage the modification
git add hello.txt
git status -s
# Output: M  hello.txt
```

**Expected Output:**
```
 M hello.txt
M  hello.txt
```

**Why this output occurs:** After editing, the working tree version differs from the index, so the second column shows `M` (` M`). After `git add`, the index is updated with the new content, so the first column shows `M` (`M `).

### Real-World Cases

- **First commit of a new project**: A developer runs `git add .` to stage all initial project files for the first commit.
- **Staging a bug fix**: A developer edits a source file, runs `git add src/fix.c`, and commits only that file.
- **Adding a generated file**: A developer force-adds a build artifact with `git add -f dist/app.js` when it's normally ignored.

### References

- Git Documentation: git-add Manual Page — https://git-scm.com/docs/git-add
- Pro Git Book: Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

---

## 2. Adding Individual Files

### Definitions

**Core Definition**
Adding an individual file means staging the changes to a single specified file by providing its exact path to `git add`.

**Technical Definition**
When a single path is given, `git add <file>` updates the index entry for that specific path. If the file is untracked, a new index entry is created; if the file is tracked and modified, the existing entry is updated with the new blob object ID and refreshed stat information.

**Beginner-Friendly Explanation**
If you've only changed one file and you want that change in your next commit, you can tell Git exactly which file to stage: `git add myfile.txt`. This is the most precise way to stage changes.

### Purposes

- To stage changes to exactly one file without affecting others.
- To start tracking a specific new file.
- To maintain precise control over commit contents.
- To stage a file when multiple files have been modified.
- To build a commit incrementally, file by file.

### Syntax Rules and Structure

**Complete General Syntax**

```
git add [<options>] <path/to/file>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<path/to/file>` | The exact relative or absolute path to the file. |
| `<options>` | Optional flags (e.g., `-v`, `-n`, `-f`). |

**Syntax Rules**

- The path can be relative to the current directory or absolute.
- If the file does not exist, `git add` reports an error unless `--ignore-missing` is used.
- If the file is ignored, `git add` silently skips it unless `-f` is used.
- Running `git add` on an already-staged file is a no-op (no error, no change).

**Constraints and Limitations**

- **Case sensitivity**: On case-insensitive filesystems (macOS, Windows), `git add File.txt` and `git add file.txt` may refer to the same file.
- **Path separators**: On Windows, both `/` and `\` may work, but `/` is recommended for portability.
- **Symlinks**: `git add` stages the symlink target's path, not the content, on systems that support symlinks.

### Annotated Code Examples

#### Example 1: Staging One of Several Modified Files

```bash
# Create and commit two files
echo "File A" > a.txt && echo "File B" > b.txt
git add a.txt b.txt && git commit -m "Add both files"

# Modify both files
echo "Modified A" >> a.txt
echo "Modified B" >> b.txt

# Stage only a.txt
git add a.txt

# Check status
git status -s
# Output:
# M  a.txt
#  M b.txt
```

**Expected Output:**
```
M  a.txt
 M b.txt
```

**Why this output occurs:** `a.txt` is staged (first column `M`), while `b.txt` remains modified but unstaged (second column `M`). Only `a.txt` will be included in the next commit.

### Real-World Cases

- **Fixing a typo**: A developer fixes a typo in `README.md` and stages only that file.
- **Updating a config**: A developer changes `config.yaml` and stages it alone for a focused commit.
- **Staging a new module**: A developer creates `src/new_module.py` and stages it individually before committing.

### References

- Git Documentation: git-add (Examples) — https://git-scm.com/docs/git-add#_examples
- Pro Git Book: Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

---

## 3. Adding Multiple Files

### Definitions

**Core Definition**
Adding multiple files means staging changes to several files in a single `git add` invocation, using space-separated paths, glob patterns, or `.` to include all files in the current directory.

**Technical Definition**
When multiple pathspecs are given, `git add` processes each one, updating the index entry for each matching file. If a path matches multiple files (e.g., a glob), all matching non-ignored files are staged. The `.` pathspec stages all files in the current directory and its subdirectories, but does not recursively add files unless `-A` is also specified in certain Git versions.

**Beginner-Friendly Explanation**
Instead of adding files one by one, you can add several at once: `git add file1.txt file2.txt` or use a pattern like `git add *.py`. You can also stage everything in the current folder with `git add .`.

### Purposes

- To stage several related files in one command.
- To stage all files of a certain type (e.g., all `.js` files).
- To stage all changes in the current directory.
- To save time when many files have been modified.
- To prepare a multi-file commit efficiently.

### Syntax Rules and Structure

**Complete General Syntax**

```
git add [<options>] <file1> <file2> ...
git add [<options>] <glob-pattern>
git add [<options>] .
git add [<options>] -A
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<file1> <file2> ...` | Space-separated list of files. |
| `<glob-pattern>` | Shell glob (quoted to prevent shell expansion if needed). |
| `.` | Stage all files in the current directory and below. |
| `-A` / `--all` | Stage all changes across the entire working tree (new, modified, deleted). |
| `-u` / `--update` | Stage modifications and deletions, but not new files. |

**Syntax Rules**

- Multiple files must be separated by spaces.
- Glob patterns are expanded by the shell; quote them (`git add '*.txt'`) to let Git handle recursion into subdirectories.
- `git add .` stages all changes in the current directory and subdirectories, but does not stage deletions of files outside the current directory unless `-A` is used.
- `git add -A` stages all changes across the *entire* repository, regardless of current directory.
- `git add -u` stages only modifications and deletions to tracked files, not new files.

**Constraints and Limitations**

- **Shell globbing**: Unquoted globs are expanded by the shell before Git sees them; this may miss files in subdirectories.
- **`.gitignore`**: Ignored files are silently skipped unless `-f` is used.
- **`git add .` vs. `git add -A`**: `.` is current-directory-relative; `-A` is repository-wide.
- **Performance**: Staging a large number of files with `git add .` can be slow on very large repositories.

### Annotated Code Examples

#### Example 1: Staging Multiple Files by Name

```bash
# Create three files
echo "One" > one.txt && echo "Two" > two.txt && echo "Three" > three.txt

# Stage two of them
git add one.txt two.txt

# Check status
git status -s
# Output:
# A  one.txt
# A  two.txt
# ?? three.txt
```

**Expected Output:**
```
A  one.txt
A  two.txt
?? three.txt
```

**Why this output occurs:** `one.txt` and `two.txt` are staged (`A`), while `three.txt` remains untracked (`??`).

#### Example 2: Staging All Files with `git add .`

```bash
# Stage everything in the current directory
git add .

# Check status
git status -s
# Output:
# A  one.txt
# A  three.txt
# A  two.txt
```

**Expected Output:**
```
A  one.txt
A  three.txt
A  two.txt
```

**Why this output occurs:** `git add .` stages all untracked and modified files in the current directory and below. All three files now show as staged.

### Real-World Cases

- **Staging all Python files**: `git add '*.py'` stages all `.py` files in the current tree.
- **Staging a full directory**: `git add src/` stages all changes within `src/` recursively.
- **Staging everything for a release commit**: `git add -A` stages all pending changes for a comprehensive commit.

### References

- Git Documentation: git-add (Examples) — https://git-scm.com/docs/git-add#_examples
- Git Documentation: git-add (Pathspec) — https://git-scm.com/docs/git-add#_pathspec
- Pro Git Book: Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

---

## 4. Adding Directories

### Definitions

**Core Definition**
Adding a directory means staging all files and subdirectories within that directory recursively, using the directory path as the pathspec.

**Technical Definition**
When a directory path is given to `git add`, Git recursively processes every file and subdirectory within it. For each file, the blob object is computed and the index entry is created or updated. Directories themselves are not stored as objects; only the files they contain are staged. Git's recursion respects `.gitignore` rules and does not add ignored files unless `-f` is used.

**Beginner-Friendly Explanation**
You can stage an entire folder at once: `git add myfolder/`. Git will walk through every file inside that folder (and any subfolders) and stage them all. It's a quick way to add a whole new module or project directory.

### Purposes

- To stage an entire directory tree in one command.
- To add a new module or package containing many files.
- To stage all changes within a specific project subdirectory.
- To recursively add files without listing each one.
- To prepare a directory-level commit.

### Syntax Rules and Structure

**Complete General Syntax**

```
git add [<options>] <directory>/
git add [<options>] <directory>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<directory>/` | The directory path (trailing slash optional). |
| `<options>` | Optional flags (e.g., `-v`, `-n`, `-f`, `-A`). |

**Syntax Rules**

- The trailing slash is optional; `git add dir` and `git add dir/` are equivalent.
- Git processes the directory recursively, adding all non-ignored files.
- Empty directories are not tracked by Git; only files are staged.
- If the directory contains ignored files, they are silently skipped.
- `git add .` is equivalent to `git add ./` (the current directory).

**Constraints and Limitations**

- **Empty directories**: Git does not track empty directories. Place a `.gitkeep` file inside to preserve them.
- **Ignored files**: Files matching `.gitignore` are skipped. Use `-f` to force-add them.
- **Symlinked directories**: Git does not recurse into symlinked directories by default.
- **Performance**: Staging a very large directory with many files can be slow.

### Annotated Code Examples

#### Example 1: Staging a Directory Recursively

```bash
# Create a directory structure
mkdir -p project/src project/docs
echo "code" > project/src/main.c
echo "readme" > project/docs/README.md

# Stage the entire project directory
git add project/

# Check status
git status -s
# Output:
# A  project/docs/README.md
# A  project/src/main.c
```

**Expected Output:**
```
A  project/docs/README.md
A  project/src/main.c
```

**Why this output occurs:** `git add project/` recursively stages all files within `project/` and its subdirectories. Both `README.md` and `main.c` are staged, even though they are in different subdirectories.

### Real-World Cases

- **Adding a new module**: `git add src/new_module/` stages all files in a newly created Python package.
- **Staging documentation**: `git add docs/` stages all documentation files.
- **Adding assets**: `git add assets/images/` stages all images in a directory.

### References

- Git Documentation: git-add (Examples) — https://git-scm.com/docs/git-add#_examples
- Git Documentation: git-add (Description) — https://git-scm.com/docs/git-add#_description

---

## 5. Understanding Staging (The Index as a Commit Blueprint)

### Definitions

**Core Definition**
Staging is the act of placing changes into the index, a binary file that serves as the blueprint for the next commit.

**Technical Definition**
The index (`.git/index`) is a binary file containing a sorted list of tracked file entries. Each entry records the file's mode, object ID (blob hash), stat information, and path. When `git commit` is run, Git reads the index and creates a tree object representing the staged snapshot. The index is the *only* source of truth for what goes into a commit—changes not in the index are not committed, even if they exist in the working tree.

**Beginner-Friendly Explanation**
Think of the staging area as a "draft" of your next commit. When you run `git add`, you're copying changes from your working folder into this draft. When you run `git commit`, Git saves exactly what's in the draft—no more, no less. This lets you carefully choose what goes into each commit, keeping your project history clean and logical.

### Purposes

- To provide a precise blueprint for the next commit.
- To enable atomic, logically separated commits.
- To allow review of changes before they become permanent.
- To bridge the working tree and the object database.
- To support partial staging (hunks, lines) for fine-grained control.

### Syntax Rules and Structure

**Index Structure**

```
Header (12 bytes):
  - Signature: "DIRC"
  - Version: 2, 3, or 4
  - Entry count

Entries (sorted by path):
  - ctime, mtime, dev, ino, mode, uid, gid, size
  - Object ID (SHA-1: 20 bytes, SHA-256: 32 bytes)
  - Flags (stage, assume-valid, extended)
  - Path name
```

**The Three Trees**

| Tree | Role | Location |
|------|------|----------|
| HEAD | Last commit snapshot | `.git/HEAD` → `refs/heads/<branch>` |
| Index | Proposed next commit | `.git/index` (binary file) |
| Working Tree | Editable files | Filesystem |

**Syntax Rules**

- The index is not edited manually; it is updated via `git add`, `git rm`, `git reset`, and `git restore`.
- The index is sorted by path name for efficient lookup.
- During a merge conflict, the index contains multiple entries per path (stages 1–3).
- The index uses network byte order for all numeric fields.

**Constraints and Limitations**

- **Binary format**: The index is not human-readable. Corrupting it can break the repository.
- **Version 4**: More compact but not supported by older Git versions.
- **Hash algorithm**: Object IDs are 20 bytes (SHA-1) or 32 bytes (SHA-256). Mixed repositories are not supported as of Git 2.49.
- **Index extensions**: Some extensions (untracked cache, FSMonitor) improve performance but are version-specific.

### Annotated Code Examples

#### Example 1: The Index as a Blueprint

```bash
# Create a file and stage it
echo "Version 1" > blueprint.txt
git add blueprint.txt

# Check the index
git ls-files --stage
# Output: 100644 <hash> 0	blueprint.txt

# Commit the staged content
git commit -m "Add blueprint.txt"

# Now modify the working tree without staging
echo "Version 2" >> blueprint.txt

# The index still holds Version 1
git show :blueprint.txt
# Output: Version 1

# The working tree holds Version 2
cat blueprint.txt
# Output: Version 1
# Version 2

# The index is the blueprint for the next commit
git diff --staged
# (no output; index matches HEAD)
```

**Expected Output:**
```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	blueprint.txt
Version 1
Version 1
Version 2
```

**Why this output occurs:** `git ls-files --stage` reads the index directly. `git show :blueprint.txt` reads the index version (Version 1). `cat blueprint.txt` reads the working tree version (Version 1 + Version 2). `git diff --staged` is empty because the index matches HEAD; the working tree changes have not been staged.

### Real-World Cases

- **Atomic commits**: A developer stages only the files relevant to a single bug fix, leaving unrelated changes unstaged.
- **Partial staging**: A developer uses `git add -p` to stage only specific hunks within a file, creating a focused commit.
- **Merge conflict resolution**: During a conflict, the index holds three entries per conflicted file (stages 1, 2, 3). Resolving and running `git add` collapses them to stage 0.
- **Pre-commit review**: A developer runs `git diff --staged` to review exactly what will be committed.

### References

- Git Documentation: git-add (Description) — https://git-scm.com/docs/git-add#_description
- MIT OpenCourseWare: Reading 5 – Version Control (Staging Area) — https://www.ocw.mit.edu
- Pro Git Book: Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

---

## 6. Unstaging Files (`git restore --staged` and `git reset`)

### Definitions

**Core Definition**
Unstaging is the act of removing changes from the staging area (index) without modifying the working tree, so they are no longer included in the next commit.

**Technical Definition**
`git restore --staged <path>` (Git 2.23+) restores the index entry for the specified path to match HEAD (or a specified `--source` commit), effectively unstaging changes while preserving the working tree. The legacy equivalent is `git reset [<tree-ish>] -- <path>`, which resets the index entry to match the specified commit (default HEAD). Both operations update only the index, not the working tree.

**Beginner-Friendly Explanation**
If you accidentally staged a file that you didn't want to commit yet, "unstaging" removes it from the staging area but leaves your edits in the file. Think of it as taking an item out of your shopping basket and putting it back on the shelf—it's no longer in your purchase (commit), but the item still exists.

### Purposes

- To remove a file from the staging area without discarding changes.
- To correct an accidental `git add`.
- To selectively unstage files before committing.
- To reset the index to match HEAD.
- To prepare a different set of changes for the next commit.

### Syntax Rules and Structure

**Complete General Syntax**

```
git restore --staged [<options>] <pathspec>
git restore --staged [<options>] --source=<tree> <pathspec>

git reset [<tree-ish>] [--] <pathspec>
git reset [<tree-ish>] <pathspec>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--staged` | (restore) Restore the index only, not the working tree. |
| `--source=<tree>` | (restore) Restore from a specific commit/tree instead of HEAD. |
| `[<tree-ish>]` | (reset) The commit to reset the index to (default HEAD). |
| `<pathspec>` | The file(s) or directory(ies) to unstage. |

**Syntax Rules**

- `git restore --staged <file>` is the modern (Git 2.23+) way to unstage a file.
- `git reset <file>` is the legacy way; it is equivalent to `git restore --staged <file>`.
- `git reset` without arguments unstages *all* files (resets the entire index to HEAD).
- `git reset --hard` is **not** for unstaging—it discards working tree changes as well. Use `git restore --staged` or `git reset` (without `--hard`).
- `git restore --staged --worktree <file>` restores both the index and working tree.

**Constraints and Limitations**

- **Working tree preserved**: Both commands preserve working tree changes. Only the index is affected.
- **Unborn branch**: On a new repository with no commits, `git restore --staged` may fail because HEAD does not exist. Use `git rm --cached <file>` instead.
- **Version-specific**: `git restore` requires Git 2.23+. `git reset` is available in all versions.
- **`--hard` danger**: `git reset --hard` permanently discards working tree changes. It is not a substitute for unstaging.

### Annotated Code Examples

#### Example 1: Unstaging with `git restore --staged` (Modern)

```bash
# Stage a file
echo "Content" > file.txt
git add file.txt

# Confirm staged
git status -s
# Output: A  file.txt

# Unstage
git restore --staged file.txt

# Confirm unstaged (working tree unchanged)
git status -s
# Output: ?? file.txt
cat file.txt
# Output: Content
```

**Expected Output:**
```
A  file.txt
?? file.txt
Content
```

**Why this output occurs:** `git restore --staged file.txt` resets the index entry for `file.txt` to match HEAD (which does not contain the file, so the entry is removed). The working tree file remains unchanged (`Content` is still printed by `cat`).

#### Example 2: Unstaging with `git reset` (Legacy)

```bash
# Stage a file
git add file.txt

# Unstage with reset
git reset file.txt

# Confirm unstaged
git status -s
# Output: ?? file.txt
```

**Expected Output:**
```
?? file.txt
```

**Why this output occurs:** `git reset file.txt` (equivalent to `git reset HEAD file.txt`) resets the index entry for `file.txt` to match HEAD. The working tree is not affected.

#### Example 3: Unstaging All Files

```bash
# Stage multiple files
git add file1.txt file2.txt file3.txt

# Unstage all (restore)
git restore --staged .

# Or (legacy)
git reset
```

**Expected Output:**
```
(no output; all files unstaged)
```

**Why this output occurs:** `git restore --staged .` resets the index to match HEAD for all files in the current directory. `git reset` does the same for the entire index.

### Real-World Cases

- **Accidental `git add`**: A developer stages a file by mistake and runs `git restore --staged <file>` to unstage it without losing edits.
- **Selective commit**: A developer stages several files, then decides to unstage one and commit the rest.
- **Pre-commit cleanup**: A developer runs `git reset` to unstage everything and start the staging process over.
- **CI/CD script**: A CI script uses `git restore --staged .` to ensure a clean index before running tests.

### References

- Git Documentation: git-restore Manual Page — https://git-scm.com/docs/git-restore
- Git Documentation: git-reset Manual Page — https://git-scm.com/docs/git-reset
- CoreUI: How to Undo git add — https://coreui.io

---

## 7. Interactive Staging (`git add -p` / `--patch`)

### Definitions

**Core Definition**
Interactive staging (also called patch mode) allows the developer to stage specific hunks or lines within a file, rather than the entire file, using the `-p` / `--patch` option to `git add`.

**Technical Definition**
`git add -p` (or `git add --patch`) enters an interactive mode where Git presents each change hunk (a contiguous block of modified lines) and prompts the user to decide whether to stage it. The user can accept (`y`), reject (`n`), split the hunk into smaller hunks (`s`), edit the hunk manually (`e`), or use other commands. This enables atomic commits that separate logically distinct changes within the same file.

**Beginner-Friendly Explanation**
Sometimes you've made several unrelated changes in the same file—maybe a bug fix and a formatting cleanup. `git add -p` lets you stage just the bug fix and leave the formatting for a separate commit. Git shows you each "chunk" of changes and asks: "Do you want this one?" You answer yes or no, and Git builds a custom commit from your choices.

### Purposes

- To stage only specific changes within a file.
- To create atomic, logically separated commits.
- To separate bug fixes from refactoring or formatting changes.
- To review changes carefully before committing.
- To avoid committing unintended modifications (e.g., debug statements).

### Syntax Rules and Structure

**Complete General Syntax**

```
git add -p [<pathspec>...]
git add --patch [<pathspec>...]
```

**Interactive Commands (at the `Stage this hunk?` prompt)**

| Command | Description |
|---------|-------------|
| `y` | Stage this hunk. |
| `n` | Do not stage this hunk. |
| `q` | Quit; do not stage this hunk or any remaining hunks. |
| `a` | Stage this hunk and all later hunks in the file. |
| `d` | Do not stage this hunk or any later hunks in the file. |
| `g` | Select a hunk to go to. |
| `/` | Search for a hunk matching a regex. |
| `j` | Leave this hunk undecided; go to next undecided hunk. |
| `J` | Leave this hunk undecided; go to next hunk. |
| `k` | Leave this hunk undecided; go to previous undecided hunk. |
| `K` | Leave this hunk undecided; go to previous hunk. |
| `s` | Split the current hunk into smaller hunks. |
| `e` | Manually edit the current hunk. |
| `?` | Print help. |

**Syntax Rules**

- `-p` is equivalent to `--patch`.
- If no pathspec is given, all modified tracked files are processed.
- The `s` (split) command is only available if the hunk contains multiple change blocks separated by context lines.
- The `e` (edit) command opens the hunk in your editor; you manually delete lines you don't want to stage.
- Hunks are presented in the order they appear in the file.

**Constraints and Limitations**

- **New files**: `git add -p` cannot stage new (untracked) files; it only works on modified tracked files. Use `git add -N` to make an untracked file "intent-to-add" first.
- **Binary files**: Patch mode does not work on binary files.
- **Large hunks**: Very large hunks may be difficult to split if they lack sufficient context lines.
- **Editor dependency**: The `e` command requires a configured editor.
- **Version-specific**: The `s` (split) command requires Git 1.5.0+. The `e` (edit) command requires a configured editor.

### Annotated Code Examples

#### Example 1: Staging a Specific Hunk

```bash
# Create a file with two separate changes
mkdir interactive-demo && cd interactive-demo && git init
cat > app.py << 'EOF'
def main():
    print("Hello")

def helper():
    pass
EOF
git add app.py && git commit -m "Initial"

# Make two separate changes
cat > app.py << 'EOF'
def main():
    print("Hello, World!")  # Change 1: greeting

def helper():
    return 42  # Change 2: return value
EOF

# Interactively stage
git add -p app.py
```

**Interactive Session:**
```
diff --git a/app.py b/app.py
index ...
--- a/app.py
+++ b/app.py
@@ -1,6 +1,6 @@
 def main():
-    print("Hello")
+    print("Hello, World!")  # Change 1: greeting
 
 def helper():
-    pass
+    return 42  # Change 2: return value

Stage this hunk [y,n,q,a,d,s,e,?]? s
```

**After splitting:**
```
Stage this hunk [y,n,q,a,d,s,e,?]? y
Stage this hunk [y,n,q,a,d,s,e,?]? n
```

**Result:**
```
# Only Change 1 is staged
git diff --staged
# Shows only the greeting change

git diff
# Shows only the return value change
```

**Expected Output:**
```
diff --git a/app.py b/app.py
--- a/app.py
+++ b/app.py
@@ -1,4 +1,4 @@
 def main():
-    print("Hello")
+    print("Hello, World!")  # Change 1: greeting
```

**Why this output occurs:** After splitting (`s`) and staging only the first hunk (`y`), only Change 1 is in the index. Change 2 remains unstaged in the working tree. This allows a focused commit for the greeting change.

### Real-World Cases

- **Separating bug fix from refactor**: A developer fixes a bug and renames a variable in the same file. `git add -p` stages only the bug fix for one commit, and the refactor for another.
- **Removing debug statements**: A developer adds `console.log` for debugging, then uses `git add -p` to stage everything *except* the debug lines.
- **Atomic commits for code review**: A developer splits a large change into multiple logical commits using `git add -p`, making review easier.
- **CI/CD partial staging**: A build script uses `git add -p` (in non-interactive mode with `printf` piping) to stage only specific hunks for automated commits.

### References

- Git Documentation: git-add (Interactive Mode) — https://git-scm.com/docs/git-add#_interactive_mode
- Git Documentation: git-add (Patch Mode) — https://git-scm.com/docs/git-add#_patch_mode
- Pro Git Book: Interactive Staging — https://git-scm.com/book/en/v2/Git-Tools-Interactive-Staging

---

## References

- Git Documentation: git-add Manual Page — https://git-scm.com/docs/git-add
- Git Documentation: git-restore Manual Page — https://git-scm.com/docs/git-restore
- Git Documentation: git-reset Manual Page — https://git-scm.com/docs/git-reset
- Git Documentation: git-add (Interactive Mode) — https://git-scm.com/docs/git-add#_interactive_mode
- Git Documentation: git-add (Patch Mode) — https://git-scm.com/docs/git-add#_patch_mode
- Pro Git Book: Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository
- Pro Git Book: Interactive Staging — https://git-scm.com/book/en/v2/Git-Tools-Interactive-Staging
- MIT OpenCourseWare: Reading 5 – Version Control (Staging Area) — https://www.ocw.mit.edu
- CoreUI: How to Undo git add — https://coreui.io
- Cornell University: Git Three Trees (Index) — https://www.cs.cornell.edu