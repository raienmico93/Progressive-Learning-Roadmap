# Git Resetting History: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`git reset` is a command that moves the current branch pointer (HEAD) to a specified commit and optionally updates the index (staging area) and working tree to match that commit, effectively undoing commits or unstaging changes.

**Technical Definition**
`git reset` operates on Git's three trees: HEAD (the branch reference), the index (staging area), and the working tree. In its commit-resetting form (`git reset [<mode>] [<commit>]`), it first moves the branch reference (pointed to by HEAD) to the specified commit. Depending on the mode (`--soft`, `--mixed`, or `--hard`), it then optionally resets the index and working tree to match the target commit. In its pathspec form (`git reset [<tree-ish>] [--] <pathspec>`), it copies entries from `<tree-ish>` to the index, effectively unstaging files without moving HEAD.

**Beginner-Friendly Explanation**
`git reset` is like a time machine for your branch. It lets you move your branch pointer back to an earlier commit, effectively "undoing" commits that came after it. Depending on how you use it, you can keep your changes staged, keep them unstaged, or discard them entirely. It's a powerful tool for cleaning up local history before sharing your work.

### Key Characteristics

- **Three Modes**: `--soft` (moves HEAD only), `--mixed` (default; moves HEAD and resets index), `--hard` (moves HEAD, resets index, and overwrites working tree).
- **History Rewriting**: Moves the branch pointer backward, removing commits from the branch's visible history.
- **Local Operation**: Affects only the local repository; no network access required.
- **Recoverable**: "Lost" commits remain in the reflog for a default of 90 days (reachable) or 30 days (unreachable).
- **Dual Form**: Can reset commits (moving HEAD) or unstage files (pathspec form, without moving HEAD).

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository with at least one commit.
- Understanding of Git's three-tree architecture (HEAD, index, working tree).
- Awareness that `--hard` permanently discards uncommitted changes.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Cleaning up local commits before pushing.
- **Software Development**: Undoing local mistakes and preparing clean commits.
- **Collaborative Development**: Understanding when history rewriting is safe.

### Core Concepts / Features

1. **git reset** — Moving the branch pointer and optionally resetting index/working tree.
2. **--soft** — Moves HEAD only; preserves index and working tree.
3. **--mixed** (default) — Moves HEAD and resets index; preserves working tree.
4. **--hard** — Moves HEAD, resets index, and overwrites working tree.
5. **HEAD Movement** — Rewriting local branch history pointers.
6. **Index Movement** — Staging area alignment with the new HEAD.
7. **Working-Tree Movement** — Workspace synchronization with the target commit.
8. **Safe vs. Unsafe Reset** — Local cleanup vs. rewriting shared remote branches.
9. **Emergency Recovery** — Using `git reflog` with `git reset` to resurrect lost commits.

---

## 1. git reset

### Definitions

**Core Definition**
`git reset` is a command that sets the current branch HEAD to a specified commit and optionally modifies the index and working tree to match, depending on the mode used.

**Technical Definition**
In its commit-resetting form, `git reset [--soft | --mixed [-N] | --hard | --merge | --keep] [-q] [<commit>]`, the command sets the current branch head (HEAD) to `<commit>` and optionally updates the index (resetting it to the tree of `<commit>`) and the working tree, depending on `<mode>`. Before the operation, `ORIG_HEAD` is set to the previous HEAD. In its pathspec form, `git reset [-q] [<tree-ish>] [--] <pathspec>…`, the command copies entries from `<tree-ish>` (default HEAD) to the index, unstaging the specified paths without moving HEAD.

**Beginner-Friendly Explanation**
`git reset` is your "undo" command for local history. You can move your branch back to an earlier commit, and decide how much you want to undo: just the commit pointer (`--soft`), the commit and the staging area (`--mixed`), or everything including your file changes (`--hard`). You can also use it to unstage individual files.

### Purposes

- To undo local commits that have not been pushed.
- To move the branch pointer to an earlier commit.
- To unstage files from the index (pathspec form).
- To reset the staging area to match HEAD.
- To clean up a messy local commit history before sharing.

### Syntax Rules and Structure

**Complete General Syntax**

```
git reset [-q] [<tree-ish>] [--] <pathspec>…
git reset [-q] [--pathspec-from-file=<file> [--pathspec-file-nul]] [<tree-ish>]
git reset (--patch | -p) [<tree-ish>] [--] [<pathspec>…]
git reset [--soft | --mixed [-N] | --hard | --merge | --keep] [-q] [<commit>]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `--soft` | Move HEAD only; index and working tree unchanged. |
| `--mixed` (default) | Move HEAD and reset index; working tree unchanged. |
| `--hard` | Move HEAD, reset index, and overwrite working tree. |
| `--merge` | Reset index and update working tree, but abort if local changes would be lost. |
| `--keep` | Reset index and update working tree, but abort if local changes would be lost (safer variant). |
| `-q` / `--quiet` | Suppress output. |
| `-p` / `--patch` | Interactively select hunks to reset. |

**Syntax Rules**

- The `<commit>` defaults to HEAD if omitted.
- `ORIG_HEAD` is set to the previous HEAD before the operation, enabling easy recovery.
- `--soft`, `--mixed`, and `--hard` are mutually exclusive.
- In the pathspec form, HEAD is **not** moved; only index entries are updated.
- The `--` separator can be used before pathspecs to avoid ambiguity.

**Constraints and Limitations**

- **History rewriting**: `git reset` rewrites history by moving the branch pointer. Do **not** reset commits that have been pushed to a shared remote.
- **`--hard` is destructive**: Permanently discards uncommitted changes in the working tree and index.
- **Untracked files**: `git reset --hard` does not remove untracked files; use `git clean` for those.
- **Merge/keep modes**: Abort if local changes would be overwritten, preventing accidental data loss.
- **Version-specific**: `--merge` and `--keep` require Git 1.6.1+.

### Annotated Code Examples

#### Example 1: Undoing a Local Commit with `--soft`

```bash
# Create a repository and make two commits
mkdir reset-demo && cd reset-demo
git init
echo "First" > file.txt && git add file.txt && git commit -m "First commit"
echo "Second" >> file.txt && git commit -am "Second commit"

# Soft reset: undo the last commit, keep changes staged
git reset --soft HEAD~1

# Check status
git status -s
# Output: M  file.txt

git log --oneline
# Output: 1a2b3c4 First commit
```

**Expected Output:**
```
M  file.txt
1a2b3c4 First commit
```

**Why this output occurs:** `--soft` moves HEAD back one commit (to "First commit") but leaves the index and working tree unchanged. The changes from "Second commit" are now staged, ready to be re-committed with a different message or additional changes.

#### Example 2: Unstaging Files with `git reset` (Pathspec Form)

```bash
# Stage a file
echo "Staged content" > newfile.txt
git add newfile.txt

# Unstage (without moving HEAD)
git reset newfile.txt

# Check status
git status -s
# Output: ?? newfile.txt
```

**Expected Output:**
```
?? newfile.txt
```

**Why this output occurs:** The pathspec form of `git reset` copies the entry for `newfile.txt` from HEAD to the index, effectively unstaging it. HEAD is not moved. The file remains in the working tree as untracked.

### Real-World Cases

- **Undo a local commit**: A developer commits a change, realizes it's incomplete, and runs `git reset --soft HEAD~1` to uncommit while keeping changes staged for further work.
- **Clean up messy commits**: A developer makes several "work in progress" commits before pushing and uses `git reset --soft` to combine them into one clean commit.
- **Unstage a file**: A developer accidentally stages a file and runs `git reset <file>` to unstage it.

### References

- Git Documentation: git-reset Manual Page — https://git-scm.com/docs/git-reset
- Git Documentation: git-reset (Synopsis) — https://git-scm.com/docs/git-reset#_synopsis
- Git Documentation: Reset Demystified — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

---

## 2. --soft (Moves HEAD, Preserves Index and Working Tree)

### Definitions

**Core Definition**
The `--soft` mode of `git reset` moves the HEAD reference to a specified commit but leaves the index (staging area) and working tree completely unchanged.

**Technical Definition**
In `--soft` mode, `git reset` performs only the first step of the reset process: moving the branch pointer (pointed to by HEAD) to the target commit. The index and working tree are not modified. Any changes that were committed after the target commit become staged changes relative to the new HEAD. `ORIG_HEAD` is set to the previous HEAD for recovery.

**Beginner-Friendly Explanation**
`--soft` is the gentlest form of reset. It moves your branch pointer back but leaves your files and staging area exactly as they are. It's like saying "I want to pretend the last commit didn't happen, but I want to keep all my work ready to be committed again."

### Purposes

- To undo a commit while keeping changes staged.
- To combine multiple commits into one (soft reset, then re-commit).
- To amend a commit's content without using `--amend`.
- To re-commit with a different message.
- To prepare a clean single commit from several messy ones.

### Syntax Rules and Structure

**Complete General Syntax**

```
git reset --soft [<commit>]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--soft` | Move HEAD only. |
| `[<commit>]` | Target commit (default: HEAD). |

**Syntax Rules**

- `--soft` can be combined with `-q` (quiet) but not with `--mixed` or `--hard`.
- The index and working tree are untouched.
- `ORIG_HEAD` is set to the previous HEAD.
- After a soft reset, `git status` shows the changes as staged.

**Constraints and Limitations**

- **History still rewritten**: Although changes are preserved, the commit is removed from the branch history.
- **Shared branches**: Do not use on shared branches without coordination.
- **No working-tree changes**: If you want to also modify the working tree, use `--mixed` or `--hard`.

### Annotated Code Examples

#### Example 1: Combining Commits with `--soft`

```bash
# Make three commits
echo "A" > file.txt && git add . && git commit -m "Commit A"
echo "B" >> file.txt && git commit -am "Commit B"
echo "C" >> file.txt && git commit -am "Commit C"

# Soft reset to the first commit
git reset --soft HEAD~2

# All changes from B and C are now staged
git status -s
# Output: M  file.txt
git log --oneline
# Output: 1a2b3c4 Commit A

# Create a single combined commit
git commit -m "Combined commit B and C"
```

**Expected Output:**
```
M  file.txt
1a2b3c4 Commit A
[master 5d6e7f8] Combined commit B and C
```

**Why this output occurs:** `--soft HEAD~2` moves HEAD back two commits (to Commit A) but keeps the index and working tree unchanged. The changes from Commits B and C are now staged. `git commit` creates a single new commit containing all those changes. The original Commits B and C are removed from the branch history.

### Real-World Cases

- **Squashing commits**: A developer makes several small "fix" commits and uses `git reset --soft` to combine them into one clean commit before pushing.
- **Re-committing with better message**: A developer commits with a poor message, then soft-resets and re-commits with a better one.
- **Adding forgotten changes**: A developer commits, then soft-resets to add a forgotten file and re-commit.

### References

- Git Documentation: git-reset (--soft) — https://git-scm.com/docs/git-reset#Documentation/git-reset.txt---soft
- Git Documentation: Reset Demystified — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

---

## 3. --mixed (The Default Mode: Moves HEAD and Resets Index, Preserves Working Tree)

### Definitions

**Core Definition**
The `--mixed` mode (the default) moves HEAD to a specified commit and resets the index to match that commit, but leaves the working tree unchanged.

**Technical Definition**
`--mixed` performs two steps: (1) move the branch pointer to the target commit, and (2) reset the index to match the target commit's tree. The working tree is not modified. Any changes that were committed after the target commit become unstaged working-tree changes. `ORIG_HEAD` is set to the previous HEAD.

**Beginner-Friendly Explanation**
`--mixed` is the middle ground. It moves your branch pointer back and unstages everything, but your actual files stay exactly as they are. It's like saying "I want to undo the last commit and also unstage everything, but I want to keep all my work in my files."

### Purposes

- To undo a commit and unstage all changes.
- To reset the index to match HEAD.
- To prepare changes for selective re-staging.
- To undo an accidental `git add`.
- To clean up the staging area before re-committing.

### Syntax Rules and Structure

**Complete General Syntax**

```
git reset [--mixed [-N]] [<commit>]
git reset [<commit>]  # --mixed is the default
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--mixed` | Move HEAD and reset index. |
| `-N` | Treat removed paths as intent-to-add. |
| `[<commit>]` | Target commit (default: HEAD). |

**Syntax Rules**

- `--mixed` is the default when no mode is specified.
- The working tree is not modified.
- Changes from the undone commit become unstaged modifications in the working tree.
- `git status` shows them as modified but not staged.

**Constraints and Limitations**

- **History rewriting**: The commit is removed from the branch.
- **Working tree preserved**: Unlike `--hard`, no file content is lost.
- **Shared branches**: Do not use on shared branches without coordination.

### Annotated Code Examples

#### Example 1: Undoing a Commit with `--mixed`

```bash
# Make a commit
echo "Content" > file.txt && git add file.txt && git commit -m "Commit"

# Mixed reset (default)
git reset HEAD~1

# Check status
git status -s
# Output: ?? file.txt
```

**Expected Output:**
```
?? file.txt
```

**Why this output occurs:** `git reset HEAD~1` (equivalent to `--mixed`) moves HEAD back one commit and resets the index to match. The file `file.txt` was created in the undone commit, so it becomes untracked in the working tree (the index no longer tracks it).

#### Example 2: Unstaging Everything with `--mixed`

```bash
# Stage multiple files
git add file1.txt file2.txt file3.txt

# Unstage everything
git reset

# Check status
git status -s
# Output:
# M file1.txt
# M file2.txt
# M file3.txt
```

**Expected Output:**
```
 M file1.txt
 M file2.txt
 M file3.txt
```

**Why this output occurs:** `git reset` with no arguments resets the index to HEAD, unstaging all files. The working tree modifications remain, now shown as unstaged.

### Real-World Cases

- **Undo commit and re-stage selectively**: A developer commits a change, then uses `git reset` to uncommit and selectively re-stage only part of the changes.
- **Unstage all**: A developer runs `git add .`, realizes it included unwanted files, and runs `git reset` to unstage everything.
- **Clean index before commit**: A developer uses `git reset` to ensure the index matches HEAD before preparing a new commit.

### References

- Git Documentation: git-reset (--mixed) — https://git-scm.com/docs/git-reset#Documentation/git-reset.txt---mixed
- Git Documentation: Reset Demystified — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

---

## 4. --hard (Moves HEAD, Resets Index, and Overwrites Working Tree)

### Definitions

**Core Definition**
The `--hard` mode of `git reset` moves HEAD to a specified commit, resets the index, and overwrites the working tree to match the target commit exactly, discarding all uncommitted changes.

**Technical Definition**
`--hard` performs all three reset steps: (1) move the branch pointer to the target commit, (2) reset the index to match the target commit's tree, and (3) overwrite all files and directories in the working tree with the version from the target commit. Any uncommitted changes (staged or unstaged) in tracked files are permanently lost. Untracked files are not affected (use `git clean` for those).

**Beginner-Friendly Explanation**
`--hard` is the "nuclear option." It moves your branch pointer back and also throws away all your uncommitted changes—both staged and unstaged. Your files will look exactly as they did at the target commit. This is useful for completely resetting to a clean state, but dangerous because discarded changes cannot be recovered.

### Purposes

- To completely reset the working tree to match a specific commit.
- To discard all uncommitted changes (staged and unstaged).
- To undo a merge or rebase that went wrong.
- To start fresh from a known-good commit.
- To recover from a messy state by returning to a clean commit.

### Syntax Rules and Structure

**Complete General Syntax**

```
git reset --hard [<commit>]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--hard` | Move HEAD, reset index, and overwrite working tree. |
| `[<commit>]` | Target commit (default: HEAD). |

**Syntax Rules**

- `--hard` cannot be combined with `--soft`, `--mixed`, `--merge`, or `--keep`.
- Untracked files are not removed.
- `ORIG_HEAD` is set to the previous HEAD for recovery.
- Any uncommitted changes to tracked files are permanently lost.

**Constraints and Limitations**

- **Destructive**: Uncommitted changes are permanently discarded. Cannot be recovered through Git.
- **Shared branches**: Never use on shared branches; it rewrites history and requires force-pushing.
- **Untracked files**: Not affected; use `git clean -fd` to remove them.
- **Recovery**: Commits can be recovered via reflog; working-tree changes cannot.

### Annotated Code Examples

#### Example 1: Discarding All Changes with `--hard`

```bash
# Make a commit
echo "Original" > file.txt && git add file.txt && git commit -m "Commit"

# Make uncommitted changes
echo "Modified" > file.txt
echo "New file" > new.txt && git add new.txt

# Hard reset to HEAD (discards all changes)
git reset --hard HEAD

# Check status
git status -s
# Output: (clean)
cat file.txt
# Output: Original
```

**Expected Output:**
```
(clean)
Original
```

**Why this output occurs:** `git reset --hard HEAD` resets the index and working tree to match HEAD. The modification to `file.txt` is discarded, and `new.txt` is unstaged (but remains as an untracked file in the working tree).

#### Example 2: Resetting to an Earlier Commit with `--hard`

```bash
# Reset to the previous commit
git reset --hard HEAD~1

# The branch now points to the earlier commit
git log --oneline
# Output: 1a2b3c4 Previous commit
```

**Expected Output:**
```
1a2b3c4 Previous commit
```

**Why this output occurs:** `--hard HEAD~1` moves the branch back one commit and overwrites the working tree and index to match. All changes from the removed commit are gone.

### Real-World Cases

- **Abandon a failed experiment**: A developer tries a new approach, decides it's wrong, and runs `git reset --hard HEAD` to discard all changes and start over.
- **Undo a bad merge**: A merge introduces conflicts; the developer runs `git reset --hard ORIG_HEAD` to abort the merge and return to the pre-merge state.
- **CI/CD cleanup**: A build script runs `git reset --hard` to ensure a clean working tree before building.

### References

- Git Documentation: git-reset (--hard) — https://git-scm.com/docs/git-reset#Documentation/git-reset.txt---hard
- Git Documentation: Reset Demystified — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

---

## 5. HEAD Movement (Rewriting Local Branch History Pointers)

### Definitions

**Core Definition**
HEAD movement in `git reset` refers to changing the commit that the current branch points to, effectively rewriting the branch's history by removing commits that came after the target commit.

**Technical Definition**
In its commit-resetting form, `git reset` first sets the current branch head (pointed to by HEAD) to the target commit. This moves the branch reference (e.g., `refs/heads/main`) to point to a different commit. Commits that were previously reachable from the branch but are no longer reachable become orphaned (recoverable via reflog). `ORIG_HEAD` is set to the previous HEAD before the move.

**Beginner-Friendly Explanation**
When you reset, the first thing Git does is move your branch pointer back to an earlier commit. This is like tearing pages out of a book—the commits after that point are no longer part of the branch. But Git keeps a copy in its "trash can" (the reflog), so you can get them back if needed.

### Purposes

- To undo commits by moving the branch pointer backward.
- To remove commits from the branch history.
- To reset the branch to a known-good state.
- To prepare for re-committing changes differently.
- To align the branch with a different commit (e.g., a remote branch).

### Syntax Rules and Structure

**HEAD Movement by Mode**

| Mode | HEAD Moves? | Index Resets? | Working Tree Resets? |
|------|-------------|---------------|----------------------|
| `--soft` | ✅ Yes | ❌ No | ❌ No |
| `--mixed` | ✅ Yes | ✅ Yes | ❌ No |
| `--hard` | ✅ Yes | ✅ Yes | ✅ Yes |

**Syntax Rules**

- `ORIG_HEAD` is set to the previous HEAD before any reset.
- The branch reference file (e.g., `.git/refs/heads/main`) is updated to point to the target commit.
- Commits after the target become unreachable from the branch.
- In detached HEAD state, HEAD itself is moved (no branch reference).

**Constraints and Limitations**

- **History rewriting**: Moving HEAD rewrites history. Do not do this on shared branches.
- **Recovery**: Orphaned commits remain in the reflog for 30-90 days.
- **Detached HEAD**: Resetting in detached HEAD moves HEAD directly, not a branch reference.

### Annotated Code Examples

#### Example 1: Observing HEAD Movement

```bash
# Create commits
git init head-demo && cd head-demo
echo "A" > file.txt && git add . && git commit -m "A"
echo "B" >> file.txt && git commit -am "B"
echo "C" >> file.txt && git commit -am "C"

# Record current HEAD
git rev-parse HEAD
# Output: 4d5e6f7...

# Soft reset to HEAD~1
git reset --soft HEAD~1

# New HEAD
git rev-parse HEAD
# Output: 1a2b3c4...

# ORIG_HEAD points to the original
git rev-parse ORIG_HEAD
# Output: 4d5e6f7...
```

**Expected Output:**
```
4d5e6f7...
1a2b3c4...
4d5e6f7...
```

**Why this output occurs:** `git reset --soft HEAD~1` moves the branch pointer from Commit C (`4d5e6f7`) to Commit B (`1a2b3c4`). `ORIG_HEAD` records the previous HEAD (`4d5e6f7`) for recovery.

### Real-World Cases

- **Undo last commit**: A developer runs `git reset --soft HEAD~1` to move the branch back one commit while keeping changes staged.
- **Reset to remote**: A developer runs `git reset --hard origin/main` to align their local branch with the remote.
- **Recover from bad state**: A developer runs `git reset --hard ORIG_HEAD` to undo an unwanted reset.

### References

- Git Documentation: git-reset (HEAD movement) — https://git-scm.com/docs/git-reset
- Git Documentation: Reset Demystified (Step 1: Moving HEAD) — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

---

## 6. Index Movement (Staging Area Alignment)

### Definitions

**Core Definition**
Index movement in `git reset` refers to updating the staging area (index) to match the target commit, either fully (`--mixed`, `--hard`) or not at all (`--soft`).

**Technical Definition**
In `--mixed` and `--hard` modes, `git reset` copies entries from the target commit's tree to the index, replacing the current index entries. This means the index now matches the target commit exactly. Any changes that were staged are either unstaged (`--mixed`) or discarded (`--hard`). In `--soft` mode, the index is not modified; it still contains the entries from the previous HEAD.

**Beginner-Friendly Explanation**
The index is your "staging area"—the place where changes wait to be committed. When you reset with `--mixed` or `--hard`, Git also updates this staging area to match the commit you're resetting to. With `--soft`, the staging area is left alone, so your staged changes remain staged.

### Purposes

- To unstage changes (`--mixed`).
- To reset the staging area to match a commit.
- To prepare for re-committing changes.
- To clean up the index before a new commit.
- To align the index with a different commit (e.g., after a reset).

### Syntax Rules and Structure

**Index Behavior by Mode**

| Mode | Index Updated? | Staged Changes |
|------|----------------|----------------|
| `--soft` | ❌ No | Remain staged |
| `--mixed` | ✅ Yes | Become unstaged |
| `--hard` | ✅ Yes | Discarded |

**Syntax Rules**

- In the pathspec form (`git reset <pathspec>`), the index is updated but HEAD is not moved.
- The index is reset by copying entries from the target commit's tree.
- After `--mixed`, `git status` shows changes as unstaged.
- After `--soft`, `git status` shows changes as staged.

**Constraints and Limitations**

- **Index-only updates**: The pathspec form updates only the index; HEAD and working tree are untouched.
- **Lost staging**: `--mixed` unstages all changes; they must be re-staged.
- **`--hard` discards**: Staged changes are permanently lost with `--hard`.

### Annotated Code Examples

#### Example 1: Index Reset with `--mixed`

```bash
# Stage a file
echo "Content" > file.txt && git add file.txt

# Mixed reset (unstages)
git reset

# The file is now unstaged
git status -s
# Output: ?? file.txt
```

**Expected Output:**
```
?? file.txt
```

**Why this output occurs:** `git reset` (with `--mixed` as default) resets the index to HEAD. The file `file.txt` was never committed, so it becomes untracked in the index. The working tree file remains.

#### Example 2: Index Preserved with `--soft`

```bash
# Stage a file
echo "Content" > file.txt && git add file.txt

# Soft reset (index preserved)
git reset --soft HEAD

# The file is still staged
git status -s
# Output: A  file.txt
```

**Expected Output:**
```
A  file.txt
```

**Why this output occurs:** `--soft` does not modify the index. The file remains staged.

### Real-World Cases

- **Unstage everything**: A developer runs `git reset` to clear the staging area before re-staging selectively.
- **Keep staging after undo**: A developer uses `--soft` to undo a commit while keeping changes staged for immediate re-commit.
- **Reset index to remote**: A developer runs `git reset origin/main` to align the index with the remote branch.

### References

- Git Documentation: git-reset (Index) — https://git-scm.com/docs/git-reset
- Git Documentation: Reset Demystified (Step 2: Updating the Index) — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

---

## 7. Working-Tree Movement (Workspace Synchronization)

### Definitions

**Core Definition**
Working-tree movement in `git reset` refers to overwriting the files in the working directory to match the target commit, which occurs only in `--hard` mode.

**Technical Definition**
In `--hard` mode, after moving HEAD and resetting the index, `git reset` overwrites all tracked files in the working tree with the content from the target commit. This means any uncommitted changes (staged or unstaged) to tracked files are permanently discarded. Untracked files are not affected. The working tree is synchronized with the target commit's tree.

**Beginner-Friendly Explanation**
`--hard` doesn't just move your branch pointer and unstage your changes—it also reaches into your actual files and replaces them with the versions from the target commit. It's like restoring your entire project folder to an earlier state, discarding anything you've changed since then.

### Purposes

- To completely discard all uncommitted changes.
- To synchronize the working tree with a specific commit.
- To start fresh from a clean state.
- To undo a failed experiment or bad merge.
- To reset a working tree that has become corrupted or inconsistent.

### Syntax Rules and Structure

**Working-Tree Behavior by Mode**

| Mode | Working Tree Updated? | Uncommitted Changes |
|------|----------------------|---------------------|
| `--soft` | ❌ No | Preserved |
| `--mixed` | ❌ No | Preserved (unstaged) |
| `--hard` | ✅ Yes | Permanently discarded |

**Syntax Rules**

- Only `--hard` modifies the working tree.
- All tracked files are overwritten; untracked files are not removed.
- File mode changes (executable bit) are also applied from the target commit.
- Any local modifications are lost; they cannot be recovered.

**Constraints and Limitations**

- **Destructive**: Uncommitted changes to tracked files are permanently lost.
- **Untracked files**: Not affected; use `git clean -fd` to remove them.
- **No recovery**: Unlike commits (recoverable via reflog), working-tree changes have no recovery mechanism.
- **Shared branches**: Never use `--hard` on shared branches without understanding the consequences.

### Annotated Code Examples

#### Example 1: Working Tree Synchronization with `--hard`

```bash
# Create a commit
echo "Original" > file.txt && git add file.txt && git commit -m "Commit"

# Modify the working tree
echo "Modified" > file.txt

# Hard reset (overwrites working tree)
git reset --hard HEAD

# The modification is gone
cat file.txt
# Output: Original
```

**Expected Output:**
```
Original
```

**Why this output occurs:** `git reset --hard HEAD` overwrites the working tree with the content from HEAD. The "Modified" content is permanently lost.

### Real-World Cases

- **Abandon changes**: A developer runs `git reset --hard HEAD` to discard all local changes and return to the last commit.
- **Reset to clean state**: A build server runs `git reset --hard` to ensure a pristine working tree before building.
- **Undo a bad merge**: A developer runs `git reset --hard ORIG_HEAD` to abort a merge and restore the pre-merge working tree.

### References

- Git Documentation: git-reset (--hard) — https://git-scm.com/docs/git-reset#Documentation/git-reset.txt---hard
- Git Documentation: Reset Demystified (Step 3: Updating the Working Directory) — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

---

## 8. Safe and Unsafe Reset Scenarios

### Definitions

**Core Definition**
Safe reset scenarios involve local, unpushed commits that only affect the developer's own repository; unsafe scenarios involve commits that have been pushed to a shared remote, where resetting rewrites history and disrupts collaborators.

**Technical Definition**
When `git reset` moves a branch pointer, it rewrites the branch's history. If the branch has been pushed to a remote and other developers have pulled it, the local history diverges from the remote. Pushing the reset branch requires a force-push (`git push --force` or `--force-with-lease`), which overwrites the remote history. Other developers' local branches will no longer match the remote, requiring them to reset or rebase, potentially losing work.

**Beginner-Friendly Explanation**
Think of commits as messages in a group chat. If you haven't sent the message yet, you can edit it freely. But if you've already sent it, editing it changes the message for everyone—and people who already read the old version get confused. The same is true for Git: reset commits that are only on your computer; don't reset commits you've already pushed to a shared repository.

### Purposes

- To understand when reset is safe and when it is dangerous.
- To choose the right undo method (`reset` vs. `revert`) for the situation.
- To avoid disrupting collaborators' work.
- To follow best practices for collaborative development.
- To know when force-pushing is acceptable.

### Syntax Rules and Structure

**Safety Decision Matrix**

| Scenario | Safe to Reset? | Recommended Action |
|----------|----------------|-------------------|
| Local commit, not pushed | ✅ Yes | Reset freely. |
| Local branch, no collaborators | ✅ Yes | Reset freely. |
| Pushed to shared branch (main, develop) | ❌ No | Use `git revert` instead. |
| Pushed to your own feature branch | ⚠️ Caution | Reset only if you're the sole user. |
| Pushed and others have pulled | ❌ No | Use `git revert`; never force-push shared history. |
| In a pull request under review | ⚠️ Caution | Coordinate with reviewers before resetting. |

**Force-Push Options**

| Option | Description | Safety |
|--------|-------------|--------|
| `git push --force` | Overwrites remote history unconditionally. | Dangerous; can lose others' work. |
| `git push --force-with-lease` | Overwrites only if remote is at expected state. | Safer; prevents overwriting others' changes. |
| `git push --force-if-includes` | Like `--force-with-lease`, but checks reflog. | Safest (Git 2.30+). |

**Syntax Rules**

- **Golden Rule**: Never reset commits that have been pushed to a shared branch.
- **Feature branches**: Resetting is generally safe on feature branches that you alone are working on.
- **Use `revert` for shared history**: `git revert` creates a new commit that undoes changes without rewriting history.
- **Force-push with lease**: Always prefer `--force-with-lease` over `--force`.

**Constraints and Limitations**

- **Team disruption**: Force-pushing a shared branch can cause teammates to lose work or face complex rebases.
- **CI/CD impact**: Resetting pushed commits can trigger confusing CI runs or break build references.
- **Branch protections**: Many repositories block force-pushes to protected branches.
- **Recovery complexity**: Recovering from a bad force-push requires coordination and reflog surgery.

### Annotated Code Examples

#### Example 1: Safe Reset (Local Commit)

```bash
# Create a local commit
git commit -m "Local change"

# Reset it (safe, not pushed)
git reset --soft HEAD~1
```

**Why this is safe:** The commit has never been pushed. The reset only affects the local repository. No one else is impacted.

#### Example 2: Unsafe Reset (Pushed Commit)

```bash
# Push a commit
git push origin main

# Reset it (UNSAFE)
git reset --hard HEAD~1

# Force-push (dangerous)
git push --force origin main
```

**Why this is unsafe:** The original commit is already on the remote. Force-pushing overwrites the remote history. Anyone who has pulled the original commit now has a diverged history and must reset or rebase, potentially losing work.

#### Example 3: Safe Alternative with `git revert`

```bash
# Pushed a faulty commit; use revert instead
git revert HEAD
git push origin main
```

**Why this is safe:** `git revert` creates a new commit that undoes the faulty one. The original commit remains in history. No force-push is needed, and collaborators can pull the change normally.

### Real-World Cases

- **Local cleanup**: A developer makes several messy commits before pushing; they use `git reset --soft` to combine them into one clean commit.
- **Shared branch mistake**: A developer accidentally pushes a faulty commit to `main`; instead of resetting (which would require force-push), they use `git revert` to safely undo it.
- **Feature branch reset**: A developer works on a feature branch alone and resets freely before creating a pull request.
- **Pull request review**: A reviewer asks for changes; the developer resets the feature branch, makes changes, and force-pushes with `--force-with-lease`, notifying the reviewer.

### References

- Mergify: A Developer's Guide to Git Undo Push Safely — https://articles.mergify.com/git-undo-push/
- Stack Overflow: Consequences of git reset --hard on a shared branch — https://stackoverflow.com/questions/59895916
- Karchunt: Reset and Revert — https://mintlify.wiki/KarChunT/karchunt.com/docs/git/reset-and-revert

---

## 9. Emergency Recovery: Using git reflog with git reset

### Definitions

**Core Definition**
Emergency recovery involves using `git reflog` to find the commit hash of a "lost" state and then using `git reset` to restore the branch pointer to that commit, recovering from an accidental `--hard` reset or other destructive operation.

**Technical Definition**
The reflog (`git reflog`) records every movement of HEAD and branch references in the local repository, stored in `.git/logs/`. Each entry includes the old and new commit hashes, the action, and a timestamp. Even after a `--hard` reset, the orphaned commits remain in the object database and are referenced by the reflog. `HEAD@{n}` syntax refers to the position of HEAD `n` moves ago. Running `git reset --hard HEAD@{n}` restores the branch to that previous state.

**Beginner-Friendly Explanation**
If you accidentally run `git reset --hard` and lose your commits, don't panic! Git keeps a detailed diary called the reflog that records every time your branch pointer moved. You can look at this diary, find the commit you lost, and use `git reset` to go back to it. It's like an "undo" button for your "undo" button.

### Purposes

- To recover commits lost after an accidental `--hard` reset.
- To restore a branch to a previous state after a bad rebase or merge.
- To undo a reset that went too far back.
- To find and recover commits that appear to be "gone."
- To provide a safety net for destructive operations.

### Syntax Rules and Structure

**Recovery Workflow**

| Step | Command | Description |
|------|---------|-------------|
| 1 | `git reflog` | View the reflog to find the lost commit. |
| 2 | Identify target | Find the `HEAD@{n}` entry before the destructive operation. |
| 3 | `git reset --hard HEAD@{n}` | Restore the branch to that state. |
| — | `git reset --hard <hash>` | Alternative: use the commit hash directly. |
| — | `git branch recovery <hash>` | Create a new branch at the lost commit (safer). |

**Reflog Syntax**

| Syntax | Meaning |
|--------|---------|
| `HEAD@{0}` | Most recent position of HEAD. |
| `HEAD@{1}` | Previous position of HEAD. |
| `HEAD@{2}` | Two moves ago. |
| `HEAD@{one.week.ago}` | Position one week ago. |
| `master@{1}` | Previous position of the `master` branch. |

**Syntax Rules**

- `git reflog` shows the reflog for HEAD by default.
- `git reflog show <branch>` shows the reflog for a specific branch.
- Reflog entries expire after 90 days (reachable) or 30 days (unreachable) by default.
- `git reset --hard HEAD@{1}` restores the state before the last HEAD movement.
- Creating a new branch at the lost commit (`git branch recovery <hash>`) is safer than resetting directly.

**Constraints and Limitations**

- **Local only**: Reflogs are not pushed to remotes; they exist only in the local repository.
- **Expiration**: Reflog entries are eventually pruned by `git gc`.
- **Not permanent**: If the reflog expires, the commits may be unrecoverable (unless found via `git fsck --lost-found`).
- **Detached HEAD**: The reflog records detached HEAD movements, which is crucial for recovery.

### Annotated Code Examples

#### Example 1: Recovering from an Accidental `--hard` Reset

```bash
# Create several commits
git init recovery-demo && cd recovery-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
echo "B" >> file.txt && git commit -am "Commit B"
echo "C" >> file.txt && git commit -am "Commit C"

# Accidentally reset --hard two commits back
git reset --hard HEAD~2

# The commits appear lost
git log --oneline
# Output: 1a2b3c4 Commit A

# View the reflog to find the lost commits
git reflog
# Output:
# 1a2b3c4 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~2
# 9a0b1c2 HEAD@{1}: commit: Commit C
# 5d6e7f8 HEAD@{2}: commit: Commit B
# 3d4e5f6 HEAD@{3}: commit: Commit A

# Recover by resetting to HEAD@{1}
git reset --hard HEAD@{1}
```

**Expected Output:**
```
HEAD is now at 9a0b1c2 Commit C
```

**Why this output occurs:** The reflog records the reset operation (`HEAD@{0}`) and the commits before it. `HEAD@{1}` is the state before the reset (Commit C). `git reset --hard HEAD@{1}` restores the branch to Commit C, recovering all lost commits.

#### Example 2: Safer Recovery with a New Branch

```bash
# Instead of resetting, create a new branch at the lost commit
git branch recovery HEAD@{1}

# The branch now points to the lost commit
git log --oneline recovery
# Output:
# 9a0b1c2 Commit C
# 5d6e7f8 Commit B
# 3d4e5f6 Commit A
```

**Why this is safer:** Creating a new branch preserves the original branch's state and provides a recovery point. You can then merge or cherry-pick from the recovery branch.

### Real-World Cases

- **Accidental hard reset**: A developer runs `git reset --hard HEAD~5`, losing five commits; they use `git reflog` to find and recover them.
- **Bad rebase**: A rebase goes wrong; the original commits are recovered from the reflog.
- **Detached HEAD recovery**: A developer makes commits in detached HEAD, switches away, and later recovers the commits from the reflog.
- **Force-push recovery**: A developer force-pushes a branch and loses commits; the reflog on the local machine still has them.

### References

- Git Documentation: git-reflog Manual Page — https://git-scm.com/docs/git-reflog
- Git Documentation: gitrevisions (Reflog Syntax) — https://git-scm.com/docs/gitrevisions#_the_reflog
- CoreUI: How to undo git reset — https://coreui.io
- KodeKloud: Recovering Data by Using Git Commands — https://notes.kodekloud.com
- Hoop.dev: Git Reset Recall – How to Recover Lost Commits with Reflog — https://hoop.dev

---

## References

- Git Documentation: git-reset Manual Page — https://git-scm.com/docs/git-reset
- Git Documentation: git-reflog Manual Page — https://git-scm.com/docs/git-reflog
- Git Documentation: gitrevisions Manual Page — https://git-scm.com/docs/gitrevisions
- Git Documentation: Reset Demystified — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified
- Git Cheat Sheet (GitHub) — https://github.com/sgoggins/Student-Materials
- Git/GitHub Tutorial Part 2 – Advanced (CERN) — https://twikiai07.cern.ch/twiki/pub/CMSPublic/CMSGitTutorialPublic/git_talk_pt2.pdf
- Mergify: A Developer's Guide to Git Undo Push Safely — https://articles.mergify.com/git-undo-push/
- Karchunt: Reset and Revert — https://mintlify.wiki/KarChunT/karchunt.com/docs/git/reset-and-revert
- CoreUI: How to undo git reset — https://coreui.io
- KodeKloud: Recovering Data by Using Git Commands — https://notes.kodekloud.com
- Hoop.dev: Git Reset Recall – How to Recover Lost Commits with Reflog — https://hoop.dev
- Stack Overflow: Consequences of git reset --hard on a shared branch — https://stackoverflow.com/questions/59895916
- GeeksforGeeks: How to Undo Git Reset? — https://origin.geeksforgeeks.org