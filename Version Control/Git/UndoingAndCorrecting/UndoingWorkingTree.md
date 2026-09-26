# Git Undoing Working-Tree Changes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Undoing working-tree changes is the practice of discarding uncommitted modifications to tracked files, or removing untracked files, to restore the repository to a clean or previous state.

**Technical Definition**
Undoing working-tree changes involves overwriting the current content of files in the working tree with content from a specified source—typically the index (staging area) or a specific commit. The primary command is `git restore`, which replaces the working tree version of specified paths with the version from the restore source. For untracked files, `git clean` recursively removes files not under version control. Both operations are destructive: discarded changes are not recoverable through Git (unlike commits, which remain in the reflog).

**Beginner-Friendly Explanation**
Sometimes you make changes you regret—a failed experiment, a broken edit, or accidental modifications. Git lets you "undo" these changes by restoring files to their last saved state. You can discard changes to one file, several files, or everything. You can even choose to discard only specific parts of a file. For files Git doesn't know about (untracked files), a separate command cleans them up. Be careful: these operations permanently delete your unsaved work.

### Key Characteristics

- **Destructive**: Discarded changes are permanently lost; they cannot be recovered via Git.
- **Local**: These operations affect only the working tree; no network access is required.
- **Selective**: You can discard changes to individual files, multiple files, or specific hunks.
- **Index-Aware**: `git restore` can target the working tree, the index, or both.
- **Safety Mechanisms**: `git clean` requires `-f` (force) and supports `-n` (dry-run) to preview deletions.

### Prerequisites

- Git installed (version 2.23+ recommended for `git restore`).
- A Git repository with tracked files and/or untracked files.
- Basic familiarity with the terminal and Git concepts (working tree, index, HEAD).
- Awareness that discarded changes are permanent.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Cleaning build environments before builds.
- **Software Development**: Reverting failed experiments and accidental edits.
- **System Administration**: Removing temporary files and build artifacts.

### Core Concepts / Features

1. **git restore** — Restoring tracked files from the index or a commit.
2. **Restoring Individual Files** — Discarding changes to one file.
3. **Restoring Multiple Files** — Discarding changes to several files or directories.
4. **Understanding Destructive Operations** — The permanence of discarded changes.
5. **Interactive Discarding (`git restore -p`)** — Discarding specific hunks within a file.
6. **Cleaning Untracked Files (`git clean -df`)** — Removing untracked files and directories safely.

---

## 1. git restore

### Definitions

**Core Definition**
`git restore` is a command that restores specified paths in the working tree with content from a restore source, effectively discarding uncommitted changes.

**Technical Definition**
`git restore` (introduced in Git 2.23) replaces the working tree version of specified paths with content from the index (by default) or from a specified commit (`--source=<tree>`). It can also restore the index with `--staged`, or both the working tree and index with `--staged --worktree`. By default, the working tree is restored from the index; if `--staged` is given, the index is restored from HEAD. The command is the modern, more explicit replacement for the file-restoration functionality of `git checkout -- <file>`.

**Beginner-Friendly Explanation**
`git restore` is your "undo" button for uncommitted changes. If you've edited a file and want to throw away those edits, run `git restore <file>`. The file goes back to how it looked the last time you staged or committed it. It's simple, direct, and replaces the older, more confusing `git checkout -- <file>` command.

### Purposes

- To discard uncommitted changes to tracked files.
- To restore a file to its state in the index (staged version).
- To restore a file to its state in a specific commit.
- To unstage changes (restore the index to match HEAD).
- To selectively discard changes at the hunk level with `--patch`.

### Syntax Rules and Structure

**Complete General Syntax**

```
git restore [<options>] [--source=<tree>] [--staged] [--worktree] [--] <pathspec>…
git restore [<options>] [--source=<tree>] [--staged] [--worktree] --pathspec-from-file=<file> [--pathspec-file-nul]
git restore (-p|--patch) [<options>] [--source=<tree>] [--staged] [--worktree] [--] [<pathspec>…]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `--source=<tree>` / `-s <tree>` | Restore from the given tree (commit, branch, tag) instead of the default. |
| `--staged` / `-S` | Restore the index (staging area) only. |
| `--worktree` / `-W` | Restore the working tree only (default if neither `--staged` nor `--worktree` is given). |
| `--patch` / `-p` | Interactively select hunks to restore. |
| `--ours` / `--theirs` | Use stage #2 (ours) or #3 (theirs) for unmerged paths. |
| `--merge` / `-m` | Resolve conflicts in the working tree. |
| `--quiet` / `-q` | Suppress feedback messages. |

**Syntax Rules**

- If neither `--staged` nor `--worktree` is specified, the working tree is restored by default.
- If `--staged` is given without `--source`, the index is restored from HEAD.
- Use `--source` to restore from a different commit (e.g., `git restore --source=HEAD~2 file.txt`).
- `--staged --worktree` restores both the index and the working tree.
- The `<pathspec>` can be a file, directory, or glob pattern.

**Constraints and Limitations**

- **Destructive**: `git restore` overwrites working tree changes without warning. Discarded changes cannot be recovered through Git.
- **Untracked files**: `git restore` does not affect untracked files; use `git clean` for those.
- **Version-specific**: `git restore` requires Git 2.23+. Older versions must use `git checkout -- <file>`.
- **Unmerged paths**: During a merge conflict, `--ours` and `--theirs` can be used, but behaviour may differ during rebase.

### Annotated Code Examples

#### Example 1: Discarding Changes to a Single File

```bash
# Create a repository and commit a file
git init restore-demo && cd restore-demo
echo "Original content" > file.txt
git add file.txt && git commit -m "Initial commit"

# Modify the file
echo "Modified content" > file.txt

# Check status
git status -s
# Output:  M file.txt

# Discard the modification
git restore file.txt

# Verify the file is restored
cat file.txt
# Output: Original content
```

**Expected Output:**
```
 M file.txt
Original content
```

**Why this output occurs:** `git restore file.txt` replaces the working tree version of `file.txt` with the version from the index (which matches HEAD). The modification is permanently discarded.

#### Example 2: Restoring from a Specific Commit

```bash
# Restore the file to its state two commits ago
git restore --source=HEAD~2 file.txt
```

**Expected Output:**
```
(no output; file restored silently)
```

**Why this output occurs:** `--source=HEAD~2` tells `git restore` to use the tree from the commit two steps back as the restore source, rather than the index.

### Real-World Cases

- **Failed experiment**: A developer tries a new approach, decides it's wrong, and runs `git restore .` to discard all changes and start over.
- **Accidental edit**: A developer accidentally modifies a configuration file and runs `git restore config.yaml` to revert it.
- **Reverting to an older version**: A developer uses `git restore --source=v1.0 README.md` to bring back an older version of the README from a tag.

### References

- Git Documentation: git-restore Manual Page — https://git-scm.com/docs/git-restore
- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout

---

## 2. Restoring Individual Files

### Definitions

**Core Definition**
Restoring an individual file means discarding uncommitted changes to a single specified file, returning it to its state in the index or a specified commit.

**Technical Definition**
When a single path is given to `git restore`, only that file's working tree version is replaced. The index entry is not modified (unless `--staged` is used). The file's content is overwritten with the blob from the restore source, and the file's stat information is updated.

**Beginner-Friendly Explanation**
If you've messed up one file and want to undo just that file's changes, you can target it specifically: `git restore myfile.txt`. This leaves all your other changes untouched.

### Purposes

- To discard changes to one file without affecting others.
- To revert a single file to its last staged or committed state.
- To correct an accidental modification to a specific file.
- To selectively undo changes when multiple files have been modified.
- To restore a file from a specific commit.

### Syntax Rules and Structure

**Complete General Syntax**

```
git restore [<options>] <path/to/file>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<path/to/file>` | The exact relative or absolute path to the file. |
| `[<options>]` | Optional flags (e.g., `--source`, `--staged`). |

**Syntax Rules**

- The path can be relative to the current directory or absolute.
- If the file is untracked, `git restore` has no effect (use `git clean` instead).
- `--source=<commit>` can be used to restore from a specific commit.

**Constraints and Limitations**

- **Case sensitivity**: On case-insensitive filesystems, path matching may be case-insensitive.
- **Symlinks**: `git restore` follows symlinks based on the restore source.
- **Uncommitted work loss**: The file's current working tree content is permanently lost.

### Annotated Code Examples

#### Example 1: Restoring One of Several Modified Files

```bash
# Modify two files
echo "Modified A" >> a.txt
echo "Modified B" >> b.txt

# Restore only a.txt
git restore a.txt

# Check status
git status -s
# Output:  M b.txt
```

**Expected Output:**
```
 M b.txt
```

**Why this output occurs:** `a.txt` is restored to its indexed version, so it no longer shows as modified. `b.txt` remains modified.

### Real-World Cases

- **Typo fix gone wrong**: A developer tries to fix a typo in `README.md`, makes it worse, and runs `git restore README.md` to start over.
- **Config file mishap**: A developer accidentally breaks `settings.py` and restores it individually.

### References

- Git Documentation: git-restore (Examples) — https://git-scm.com/docs/git-restore#_examples

---

## 3. Restoring Multiple Files

### Definitions

**Core Definition**
Restoring multiple files means discarding uncommitted changes to several files at once, using space-separated paths, glob patterns, or `.` to target all files in the current directory.

**Technical Definition**
When multiple pathspecs are given, `git restore` processes each one, replacing the working tree version of each matching file with the corresponding blob from the restore source. Glob patterns are expanded by the shell or handled by Git's pathspec machinery. `.` restores all modified tracked files in the current directory and subdirectories.

**Beginner-Friendly Explanation**
You can undo changes to many files at once: `git restore file1.txt file2.txt` or `git restore .` to discard all changes in the current folder. It's a quick way to get back to a clean state.

### Purposes

- To discard all uncommitted changes in the current directory.
- To restore several specific files in one command.
- To restore all files matching a pattern.
- To quickly reset the working tree to a clean state.
- To prepare for a fresh start after a failed experiment.

### Syntax Rules and Structure

**Complete General Syntax**

```
git restore [<options>] <file1> <file2> ...
git restore [<options>] <glob-pattern>
git restore [<options>] .
git restore [<options>] -- <path1> <path2>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<file1> <file2> ...` | Space-separated list of files. |
| `<glob-pattern>` | Shell glob (quote to let Git handle recursion). |
| `.` | All modified tracked files in the current directory and below. |
| `--` | Separates options from paths (useful when paths start with `-`). |

**Syntax Rules**

- `git restore .` restores all modified tracked files in the current directory recursively.
- Multiple paths can be given; each is processed independently.
- Glob patterns should be quoted (`git restore '*.js'`) to prevent shell expansion.

**Constraints and Limitations**

- **`.` is current-directory-relative**: Use `git restore :/` or `git restore -- .` from the repository root for repository-wide restoration.
- **Untracked files**: `git restore` does not remove untracked files.

### Annotated Code Examples

#### Example 1: Restoring All Modified Files

```bash
# Modify several files
echo "Change 1" >> file1.txt
echo "Change 2" >> file2.txt
echo "Change 3" >> file3.txt

# Restore all
git restore .

# Check status
git status -s
# Output: (clean)
```

**Expected Output:**
```
(no output; working tree clean)
```

**Why this output occurs:** `git restore .` restores all modified tracked files in the current directory to their indexed versions, discarding all uncommitted changes.

### Real-World Cases

- **Start over**: A developer runs `git restore .` to discard all changes and begin again.
- **After a failed merge**: A developer restores all files to start the merge over.

### References

- Git Documentation: git-restore (Examples) — https://git-scm.com/docs/git-restore#_examples

---

## 4. Understanding Destructive Operations (Permanent Loss of Uncommitted Work)

### Definitions

**Core Definition**
A destructive operation in Git is one that permanently discards uncommitted work, making it unrecoverable through Git's normal mechanisms.

**Technical Definition**
Git's data model is append-only for committed objects: once a commit is created, it is immutable and recoverable via the reflog. However, **uncommitted changes**—modifications in the working tree that have not been staged or committed—are not stored in the object database. When `git restore` or `git checkout -- <file>` overwrites a file, the previous working tree content is lost. Similarly, `git clean` performs a hard filesystem deletion (`rm`) on untracked files. The reflog can recover commits, but not edits that were never committed.

**Beginner-Friendly Explanation**
Git is very good at recovering things you've committed. But if you've made changes and haven't saved them to a commit yet, those changes are not in Git's database—they're just in your files. If you discard them with `git restore`, they're gone forever. There's no "undo" for discarded working-tree changes. This is why Git prompts you to be careful.

### Purposes

- To make developers aware of the irreversibility of discarding uncommitted work.
- To encourage careful review before running destructive commands.
- To explain why `git clean` requires force and supports dry-run.
- To distinguish between recoverable (committed) and unrecoverable (uncommitted) data.
- To promote safe workflows (stashing, committing, or backing up before discarding).

### Syntax Rules and Structure

**Destructive Commands Overview**

| Command | What It Destroys | Recoverable? |
|---------|------------------|--------------|
| `git restore <file>` | Uncommitted working-tree changes to tracked files | No |
| `git checkout -- <file>` | Same as above (legacy) | No |
| `git restore --staged <file>` | Staged changes (index only; working tree preserved) | Working tree preserved |
| `git clean -f` | Untracked files | No (hard filesystem deletion) |
| `git clean -fd` | Untracked files and directories | No |
| `git reset --hard` | All uncommitted changes (working tree and index) | No |

**Safety Mechanisms**

| Mechanism | Description |
|-----------|-------------|
| `git clean -n` (dry-run) | Shows what would be removed without deleting anything. |
| `git clean -f` (force) | Required to actually delete files; prevents accidental deletion. |
| `clean.requireForce` | Configuration (default `true`) that enforces the `-f` requirement. |
| `git stash` | Temporarily saves uncommitted changes for later recovery. |
| `git diff` / `git diff --staged` | Review changes before discarding. |

**Constraints and Limitations**

- **No Git recovery**: Discarded uncommitted changes cannot be recovered via `git reflog` or any other Git mechanism.
- **Filesystem recovery**: Deleted files may be recoverable via filesystem tools (e.g., `extundelete`, `TestDisk`), but this is outside Git's scope.
- **IDE undo**: Some IDEs maintain their own local history, which may allow recovery, but this is not guaranteed.
- **Stashing**: `git stash` is the recommended way to save uncommitted changes before discarding or switching branches.

### Annotated Code Examples

#### Example 1: Demonstrating Permanent Loss

```bash
# Create a file and commit it
echo "Original" > important.txt
git add important.txt && git commit -m "Add important file"

# Make a valuable uncommitted change
echo "Valuable work" >> important.txt

# Discard the change
git restore important.txt

# The valuable work is gone
cat important.txt
# Output: Original
```

**Expected Output:**
```
Original
```

**Why this output occurs:** `git restore` overwrote the working tree version of `important.txt` with the indexed version. The "Valuable work" line was never staged or committed, so it is not in Git's object database. It cannot be recovered.

### Real-World Cases

- **Lost work**: A developer spends hours on changes, accidentally runs `git restore .`, and loses everything. This is why frequent commits are recommended.
- **Build cleanup**: A CI system uses `git clean -fdx` to remove build artifacts, accepting that these files are not part of the repository and are regenerable.
- **Safety training**: Teams teach new developers to use `git stash` instead of `git restore .` to avoid accidental loss.

### References

- Stack Overflow: How to Undo Changes in Git Safely — https://stackoverflow.com/questions/43541167
- GitHub: Git for Beginners – Undoing Changes — https://github.com/nisalgunawardhana/Github-for-beginners

---

## 5. Interactive Discarding (`git restore -p` / `--patch`)

### Definitions

**Core Definition**
Interactive discarding allows the developer to select specific hunks within a file to discard, using `git restore -p` (or `--patch`).

**Technical Definition**
`git restore -p` (or `--patch`) enters an interactive mode where Git presents each change hunk between the restore source and the working tree, and prompts the user to decide whether to restore (discard) it. The interface is the same as `git add -p`: the user can accept (`y`), reject (`n`), split (`s`), edit (`e`), or quit (`q`). Only the selected hunks are discarded; unselected hunks remain in the working tree.

**Beginner-Friendly Explanation**
Sometimes you've made several changes in a file, but you only want to undo some of them. `git restore -p` shows you each "chunk" of change and asks: "Do you want to discard this one?" You answer yes or no, and Git discards only the chunks you choose.

### Purposes

- To discard only specific changes within a file.
- To keep some edits while discarding others.
- To selectively revert parts of a file to a previous state.
- To clean up a file without losing all changes.
- To combine with `--source` to selectively restore from an older commit.

### Syntax Rules and Structure

**Complete General Syntax**

```
git restore -p [<options>] [--source=<tree>] [--] [<pathspec>…]
git restore --patch [<options>] [--source=<tree>] [--] [<pathspec>…]
```

**Interactive Commands**

The interactive commands are identical to those of `git add -p`:

| Command | Description |
|---------|-------------|
| `y` | Discard (restore) this hunk. |
| `n` | Do not discard this hunk. |
| `q` | Quit; do not discard this hunk or any remaining hunks. |
| `a` | Discard this hunk and all later hunks in the file. |
| `d` | Do not discard this hunk or any later hunks in the file. |
| `s` | Split the current hunk into smaller hunks. |
| `e` | Manually edit the current hunk. |
| `?` | Print help. |

**Syntax Rules**

- `-p` is equivalent to `--patch`.
- If no pathspec is given, all modified tracked files are processed.
- The `s` command splits a hunk into smaller hunks if possible.
- The `e` command opens the hunk in the editor for manual editing.

**Constraints and Limitations**

- **New files**: `git restore -p` does not work on untracked files.
- **Binary files**: Patch mode does not work on binary files.
- **Large hunks**: Splitting may not be possible if the hunk lacks sufficient context lines.
- **Editor dependency**: The `e` command requires a configured editor.

### Annotated Code Examples

#### Example 1: Selectively Discarding Hunks

```bash
# Create a file with two separate changes
git init patch-demo && cd patch-demo
echo "Line 1" > file.txt
git add file.txt && git commit -m "Initial"

# Make two changes
echo "Line 2 (keep)" >> file.txt
echo "Line 3 (discard)" >> file.txt

# Interactively discard only the second change
git restore -p file.txt
```

**Interactive Session:**
```
diff --git a/file.txt b/file.txt
index ...
--- a/file.txt
+++ b/file.txt
@@ -1 +1,3 @@
 Line 1
+Line 2 (keep)
+Line 3 (discard)

Discard this hunk from worktree [y,n,q,a,d,s,e,?]? s
```

**After splitting:**
```
Discard this hunk from worktree [y,n,q,a,d,s,e,?]? n
Discard this hunk from worktree [y,n,q,a,d,s,e,?]? y
```

**Result:**
```
# Line 2 is kept; Line 3 is discarded
cat file.txt
# Output:
# Line 1
# Line 2 (keep)
```

**Expected Output:**
```
Line 1
Line 2 (keep)
```

**Why this output occurs:** The `s` command split the combined hunk into two. The first hunk (Line 2) was rejected (`n`), so it was kept. The second hunk (Line 3) was accepted (`y`), so it was discarded. Only the selected change was reverted.

### Real-World Cases

- **Partial undo**: A developer adds a debug statement and a real fix in the same file; `git restore -p` discards only the debug statement.
- **Refactoring cleanup**: A developer starts a refactoring, decides part of it is wrong, and uses `-p` to discard only the bad parts.
- **Combining with `--source`**: A developer uses `git restore -p --source=HEAD~1 file.txt` to selectively restore parts of a file from an older commit.

### References

- Git Documentation: git-restore (--patch) — https://git-scm.com/docs/git-restore#Documentation/git-restore.txt---patch
- Git Documentation: git-add (Interactive Mode) — https://git-scm.com/docs/git-add#_interactive_mode

---

## 6. Cleaning Untracked Files Safely (`git clean -df` and `--dry-run`)

### Definitions

**Core Definition**
`git clean` is a command that removes untracked files from the working tree. With `-d`, it also removes untracked directories. With `-f`, it actually performs the deletion.

**Technical Definition**
`git clean` recursively removes files that are not under version control, starting from the current directory. It operates on untracked files only—files that have never been added to the index. By default, it does not recurse into untracked directories unless `-d` is specified. Because deletion is a hard filesystem operation (`rm`), `git clean` requires `-f` (force) unless `clean.requireForce` is set to `false`. The `-n` (or `--dry-run`) option shows what *would* be removed without actually deleting anything.

**Beginner-Friendly Explanation**
`git clean` is like a broom for your project folder: it sweeps away files that Git doesn't know about. This is useful for removing build artifacts, temporary files, or accidental clutter. But be careful—once deleted, these files are gone (Git can't bring them back). Always use `git clean -n` first to see what will be removed.

### Purposes

- To remove untracked files from the working tree.
- To remove untracked directories (with `-d`).
- To clean build artifacts and generated files (with `-x`).
- To preview deletions safely with `--dry-run`.
- To reset the working tree to a clean state before a build or commit.

### Syntax Rules and Structure

**Complete General Syntax**

```
git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] [<pathspec>…]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `-d` | Remove untracked directories in addition to untracked files. |
| `-f` / `--force` | Required to actually delete files (unless `clean.requireForce` is false). |
| `-n` / `--dry-run` | Show what would be removed without deleting anything. |
| `-i` / `--interactive` | Interactive mode: show what would be removed and allow selective deletion. |
| `-x` | Remove ignored files as well as untracked files. |
| `-X` | Remove only ignored files. |
| `-e <pattern>` | Add an exclude pattern (in addition to `.gitignore`). |
| `-q` / `--quiet` | Suppress output. |

**Syntax Rules**

- `-f` is required for actual deletion (unless `clean.requireForce` is set to `false`).
- `-n` (dry-run) can be used without `-f`; it simply lists what would be removed.
- `-d` is necessary to remove untracked directories; without it, only files are removed.
- `-x` extends the operation to ignored files (e.g., build outputs).
- `-X` removes only ignored files, leaving other untracked files intact.

**Constraints and Limitations**

- **Permanent deletion**: `git clean` performs a hard filesystem deletion. Deleted files cannot be recovered through Git.
- **Untracked only**: `git clean` does not affect tracked files; use `git restore` for those.
- **Nested Git repositories**: By default, `git clean -d` refuses to remove nested Git repositories unless `-f -f` is given.
- **`.gitignore` respected**: By default, ignored files are not removed unless `-x` is specified.

### Annotated Code Examples

#### Example 1: Dry Run with `-n` (Safe Preview)

```bash
# Create untracked files
echo "temp" > temp.txt
mkdir build && echo "binary" > build/output.bin

# Preview what would be removed
git clean -n
```

**Expected Output:**
```
Would remove temp.txt
```

**Why this output occurs:** `-n` performs a dry run. It lists `temp.txt` (an untracked file) but does **not** list `build/` because `-d` was not specified. No files are actually deleted. This is the safest way to use `git clean`.

#### Example 2: Dry Run Including Directories (`-n -d`)

```bash
git clean -n -d
```

**Expected Output:**
```
Would remove temp.txt
Would remove build/
```

**Why this output occurs:** Adding `-d` includes untracked directories in the preview. Both `temp.txt` and the `build/` directory would be removed.

#### Example 3: Force Delete Files and Directories (`-f -d`)

```bash
git clean -f -d
```

**Expected Output:**
```
Removing temp.txt
Removing build/
```

**Why this output occurs:** `-f` forces the deletion, and `-d` includes directories. Both `temp.txt` and `build/` are permanently deleted from the filesystem.

#### Example 4: Including Ignored Files (`-f -d -x`)

```bash
# Create a .gitignore and an ignored file
echo "*.log" > .gitignore
echo "log data" > app.log

# Clean including ignored files
git clean -f -d -x
```

**Expected Output:**
```
Removing app.log
```

**Why this output occurs:** `-x` extends the cleaning to ignored files. `app.log` matches `*.log` in `.gitignore` and is removed. This is useful for removing build artifacts.

### Real-World Cases

- **Build cleanup**: A CI pipeline runs `git clean -fdx` to remove all untracked and ignored files before a fresh build.
- **Accidental clutter**: A developer creates temporary files and runs `git clean -n -d` to preview, then `git clean -f -d` to remove them.
- **Switching branches**: Before switching branches, a developer runs `git clean -fd` to remove untracked files that might conflict.
- **Removing build artifacts**: `git clean -fdx` removes `node_modules/`, `dist/`, and other ignored outputs.

### References

- Git Documentation: git-clean Manual Page — https://git-scm.com/docs/git-clean
- Atlassian Git Tutorial: git clean — https://www.atlassian.com/git/tutorials/undoing-changes/git-clean
- Stack Overflow: Git Clean Dry Run Safety — https://stackoverflow.com/questions/22096434

---

## References

- Git Documentation: git-restore Manual Page — https://git-scm.com/docs/git-restore
- Git Documentation: git-clean Manual Page — https://git-scm.com/docs/git-clean
- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout
- Git Documentation: git-add (Interactive Mode) — https://git-scm.com/docs/git-add#_interactive_mode
- Atlassian Git Tutorial: git clean — https://www.atlassian.com/git/tutorials/undoing-changes/git-clean
- Stack Overflow: Git Clean Dry Run Safety — https://stackoverflow.com/questions/22096434
- Stack Overflow: How to Undo Changes in Git Safely — https://stackoverflow.com/questions/43541167
- GitHub: Git for Beginners – Undoing Changes — https://github.com/nisalgunawardhana/Github-for-beginners
- GitLab Docs: Undo Changes — https://docs.gitlab.com/ee/topics/git/undo_changes.html
- GitHub Training Kit: Advanced Git Cheat Sheet — https://mintlify.wiki/git/git/commands