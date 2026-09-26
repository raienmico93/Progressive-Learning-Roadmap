# Git Comparing Changes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Comparing changes in Git is the practice of using `git diff` and related commands to inspect the differences between two states of a repository—whether those states are the working tree, the staging area (index), commits, or branches.

**Technical Definition**
`git diff` computes line-by-line differences between two data sources, generating output in the unified diff format (originally from the Unix `diff` utility). The command operates on Git's three-tree architecture (working tree, index, HEAD) and on arbitrary commit or tree objects. Comparison ranges use two-dot (`A..B`) notation for endpoint-to-endpoint diffs and three-dot (`A...B`) notation to compare against the merge base (common ancestor). Output includes metadata headers, file mode information, and hunks with context lines. External diff tools can be configured via `git difftool` for visual comparison.

**Beginner-Friendly Explanation**
When you work with Git, you often need to see exactly what changed. `git diff` shows you the differences between your files at different points in time. You can compare your uncommitted edits to what you last saved, compare two commits, or compare two branches. Git displays these differences in a standard format that uses `+` for added lines and `-` for removed lines. You can also use graphical diff tools if you prefer a visual side-by-side view.

### Key Characteristics

- **Read-Only**: `git diff` never modifies the repository; it only reports differences.
- **Multi-Source**: Compares working tree, index, commits, branches, and arbitrary blobs/trees.
- **Unified Format**: Uses the standard unified diff format, compatible with `patch` and other tools.
- **Composable**: Output can be piped to pagers (`less`), filtered (`grep`), or saved to patch files.
- **Configurable**: External diff tools can be configured for visual comparison.

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository with at least one commit.
- Basic familiarity with the terminal and Git concepts (commits, branches, HEAD, index).

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **Code Review**: Inspecting proposed changes before merging.
- **Patch Management**: Generating and applying `.patch` files.
- **DevOps & CI/CD**: Automated diff checks in pipelines.

### Core Concepts / Features

1. **git diff** — The primary command for comparing changes.
2. **Working-Tree Changes** — Unstaged modifications (`git diff`).
3. **Staged Changes** — Changes in the index (`git diff --staged` / `--cached`).
4. **Comparing Commits** — `git diff <commit1>..<commit2>`.
5. **Comparing Branches** — `git diff <branch1>...<branch2>` (since common ancestor).
6. **Reading Diff Output** — Metadata headers, file modes, and unified diff format.
7. **External Diff Tools** — Configuring `git difftool` with visual tools.

---

## 1. git diff

### Definitions

**Core Definition**
`git diff` is the command that shows changes between various Git data sources, such as the working tree, the index, commits, and branches.

**Technical Definition**
`git diff` computes the difference between two states and outputs it in the unified diff format. Without arguments, it compares the working tree to the index. With `--cached` (or `--staged`), it compares the index to a specified commit (default HEAD). With two commit arguments, it compares those commits. The command is a front-end to the internal diff machinery (`git diff-files`, `git diff-index`, `git diff-tree`) and supports extensive options for filtering, formatting, and whitespace handling.

**Beginner-Friendly Explanation**
`git diff` shows you exactly what has changed. If you've edited files but haven't staged them, `git diff` shows those edits. If you want to see what you've staged, use `git diff --staged`. It's like a “before and after” view for your code.

### Purposes

- To review unstaged changes before staging them.
- To review staged changes before committing.
- To compare branches or commits before merging.
- To generate patches for email or code review.
- To verify that a commit contains the expected changes.

### Syntax Rules and Structure

**Complete General Syntax**

```
git diff [<options>] [--] [<path>...]
git diff [<options>] --cached [<commit>] [--] [<path>...]
git diff [<options>] <commit> [<commit>] [--] [<path>...]
git diff [<options>] <commit>..<commit> [--] [<path>...]
git diff [<options>] <commit>...<commit> [--] [<path>...]
git diff [<options>] <blob> <blob>
git diff [<options>] --no-index [--] <path> <path>
```

**Key Forms**

| Form | Comparison |
|------|------------|
| `git diff` | Working tree vs. index (unstaged changes). |
| `git diff --cached` / `--staged` | Index vs. HEAD (staged changes). |
| `git diff HEAD` | Working tree vs. HEAD (both staged and unstaged). |
| `git diff <commit>` | Working tree vs. commit. |
| `git diff <commit1> <commit2>` | Commit1 vs. Commit2. |
| `git diff <branch1>..<branch2>` | Tip of branch1 vs. tip of branch2. |
| `git diff <branch1>...<branch2>` | Merge base vs. tip of branch2. |
| `git diff --no-index <file1> <file2>` | Two files on disk (outside Git). |

**Common Options**

| Option | Description |
|--------|-------------|
| `--stat` | Show diffstat summary. |
| `--name-only` | Show only file names. |
| `--name-status` | Show file names and status (A/M/D). |
| `--diff-filter=<filter>` | Select files by status (A/C/D/M/R/T). |
| `-p` / `--patch` | Show patch (default). |
| `-w` / `--ignore-all-space` | Ignore whitespace changes. |
| `-M` / `--find-renames` | Detect renames. |
| `-C` / `--find-copies` | Detect copies. |
| `--color` | Colourise output. |
| `-R` | Swap input sources (reverse diff). |

**Syntax Rules**

- `git diff` without arguments compares working tree and index.
- `git diff --cached` compares index and HEAD.
- `git diff HEAD` compares working tree and HEAD (both staged and unstaged changes).
- Path limiting restricts the diff to specified files or directories.
- `--no-index` allows comparing two arbitrary files on disk without a Git repository.

**Constraints and Limitations**

- **Binary files**: `git diff` cannot show textual diffs for binary files by default; it reports “Binary files differ.”
- **Renames**: Rename detection requires `-M`; otherwise a rename appears as a delete + add.
- **Whitespace**: Use `-w` or `--ignore-all-space` to ignore whitespace-only changes.
- **Shallow clones**: In a shallow clone, diffs against truncated history may be incomplete.
- **Version-specific**: `--no-index` requires Git 1.5.1+; `--merge-base` requires Git 2.30+.

### Annotated Code Examples

#### Example 1: `git diff` (Working Tree vs. Index)

```bash
# Create a repository and commit a file
git init diff-demo && cd diff-demo
echo "Line 1" > file.txt && git add file.txt && git commit -m "Initial"

# Modify the file without staging
echo "Line 2" >> file.txt
git diff
```

**Expected Output:**
```
diff --git a/file.txt b/file.txt
index 8a1b2c3..9d4e5f6 100644
--- a/file.txt
+++ b/file.txt
@@ -1 +1,2 @@
 Line 1
+Line 2
```

**Why this output occurs:** `git diff` compares the working tree version of `file.txt` (which has “Line 2”) against the index version (which only has “Line 1”). The `+` line indicates the addition.

#### Example 2: `git diff --stat`

```bash
git diff --stat
```

**Expected Output:**
```
 file.txt | 1 +
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** `--stat` provides a summary: one file changed, one insertion. This is useful for a quick overview before diving into the full patch.

#### Example 3: `git diff --name-status`

```bash
git diff --name-status
```

**Expected Output:**
```
M	file.txt
```

**Why this output occurs:** `--name-status` shows only the file name and its status (`M` for modified). This is useful for scripting.

### Real-World Cases

- **Pre-commit review**: A developer runs `git diff` to review unstaged changes before deciding what to stage.
- **Code review**: A reviewer uses `git diff main..feature` to see all changes introduced by a feature branch.
- **Patch generation**: `git diff > fix.patch` creates a patch file that can be applied elsewhere with `git apply` or `patch`.

### References

- Git Documentation: git-diff Manual Page — https://git-scm.com/docs/git-diff
- Git Documentation: git-diff (Synopsis) — https://git-scm.com/docs/git-diff#_synopsis
- Mintlify: git diff Documentation — https://mintlify.wiki/git/git/commands/diff

---

## 2. Working-Tree Changes (Unstaged Modifications)

### Definitions

**Core Definition**
Working-tree changes are modifications made to files in the working directory that have not yet been staged (added to the index) for the next commit.

**Technical Definition**
When a tracked file is modified in the working tree, its content differs from the version recorded in the index. `git diff` (without arguments) compares these two states, showing the unstaged modifications. The index still holds the version from the last `git add` or `git commit`. The diff is computed line-by-line using the Myers diff algorithm, with context lines surrounding each change.

**Beginner-Friendly Explanation**
When you edit a file but haven't run `git add` yet, those changes are “unstaged” or “working-tree changes.” `git diff` shows you exactly what you've edited but not yet prepared for committing.

### Purposes

- To review edits before deciding whether to stage them.
- To verify that modifications are correct before adding them.
- To check for accidental changes (e.g., debug statements, formatting).
- To generate a patch of uncommitted work.
- To inspect the extent of changes across multiple files.

### Syntax Rules and Structure

**Complete General Syntax**

```
git diff [<options>] [--] [<path>...]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `[<options>]` | Optional flags (e.g., `--stat`, `--name-only`, `-w`). |
| `[--] [<path>...]` | Optional path limiting. |

**Syntax Rules**

- `git diff` with no arguments shows all unstaged changes in the working tree.
- Path limiting restricts the diff to specific files or directories.
- `-w` ignores whitespace-only changes.
- `--color-words` highlights word-level changes instead of line-level.

**Constraints and Limitations**

- **Untracked files**: `git diff` does not show untracked files (use `git status` or `git diff --no-index`).
- **Staged changes**: `git diff` does not show staged changes (use `git diff --staged`).
- **Binary files**: Textual diffs are not available for binary files.

### Annotated Code Examples

#### Example 1: Reviewing Unstaged Edits

```bash
echo "New feature" >> feature.py
git diff feature.py
```

**Expected Output:**
```
diff --git a/feature.py b/feature.py
index 1234567..89abcde 100644
--- a/feature.py
+++ b/feature.py
@@ -1,3 +1,4 @@
 def feature():
     pass
+New feature
```

**Why this output occurs:** The working tree version of `feature.py` has an additional line (`New feature`), which is not yet in the index. The `+` line shows the addition.

### Real-World Cases

- **Debugging**: A developer runs `git diff` to see what they've changed while debugging.
- **Pre-staging review**: A developer reviews unstaged changes before running `git add`.
- **Accidental edit detection**: `git diff` reveals unintended changes (e.g., whitespace, debug prints) before committing.

### References

- Git Documentation: git-diff Manual Page — https://git-scm.com/docs/git-diff
- Git Documentation: gittutorial (Making Changes) — https://git-scm.com/docs/gittutorial
- Microsoft Learn: Track Changes with git diff — https://learn.microsoft.com/en-us/training/modules/work-source-control-git/

---

## 3. Staged Changes (`git diff --staged` / `--cached`)

### Definitions

**Core Definition**
`git diff --staged` (synonym: `--cached`) shows the changes that are staged for the next commit by comparing the index to HEAD.

**Technical Definition**
The `--cached` option (with `--staged` as a synonym) instructs `git diff` to compare the index against a specified commit (default: HEAD). If HEAD does not exist (e.g., an unborn branch), it shows all staged changes. This is the diff that will be included if `git commit` is run without further modification. The output is in unified diff format, with the index as the “new” side and HEAD as the “old” side.

**Beginner-Friendly Explanation**
After you run `git add`, your changes are “staged.” `git diff --staged` shows you exactly what you've staged—the changes that will go into your next commit. It's a final check before you commit.

### Purposes

- To review exactly what will be committed.
- To verify that the staging area contains the intended changes.
- To catch accidental inclusions or omissions before committing.
- To generate a patch of staged changes.
- To compare staged changes against a specific commit (not just HEAD).

### Syntax Rules and Structure

**Complete General Syntax**

```
git diff [<options>] --cached [<commit>] [--] [<path>...]
git diff [<options>] --staged [<commit>] [--] [<path>...]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--cached` / `--staged` | Compare index to commit (default HEAD). |
| `[<commit>]` | Optional commit to compare against (default HEAD). |
| `[--] [<path>...]` | Optional path limiting. |

**Syntax Rules**

- `--staged` is a synonym for `--cached`.
- If no commit is specified, HEAD is used.
- If HEAD does not exist (unborn branch), all staged changes are shown.
- `--merge-base` can be used to compare against a merge base.

**Constraints and Limitations**

- **Unborn branches**: In a new repository with no commits, `git diff --cached` shows all staged files as new.
- **Merge conflicts**: During a conflict, `git diff --cached` shows the staged (resolved) version.
- **Index vs. working tree**: This command does not show unstaged changes; use `git diff` for those.

### Annotated Code Examples

#### Example 1: Reviewing Staged Changes

```bash
echo "Staged change" >> file.txt
git add file.txt
git diff --staged
```

**Expected Output:**
```
diff --git a/file.txt b/file.txt
index 1234567..89abcde 100644
--- a/file.txt
+++ b/file.txt
@@ -1,2 +1,3 @@
 Line 1
 Line 2
+Staged change
```

**Why this output occurs:** `--staged` compares the index (which now contains `Staged change`) against HEAD (which does not). The diff shows the staged addition.

#### Example 2: `git diff --staged --stat`

```bash
git diff --staged --stat
```

**Expected Output:**
```
 file.txt | 1 +
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** `--stat` summarises the staged changes. This is useful for a quick check before committing.

### Real-World Cases

- **Pre-commit hook**: A pre-commit hook runs `git diff --staged --check` to detect whitespace errors before allowing a commit.
- **Selective commit**: A developer stages several files and runs `git diff --staged` to confirm only the intended changes are included.
- **Patch extraction**: `git diff --staged > staged.patch` creates a patch of the proposed commit.

### References

- Git Documentation: git-diff (--cached) — https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---cached
- Git Documentation: git-diff (--staged) — https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---staged
- Git Documentation: gittutorial (Reviewing Changes) — https://git-scm.com/docs/gittutorial

---

## 4. Comparing Commits (`git diff commit1..commit2`)

### Definitions

**Core Definition**
`git diff <commit1>..<commit2>` shows the differences between the snapshots of two commits.

**Technical Definition**
When two commit references are given, `git diff` compares the tree objects of those commits. The two-dot notation (`A..B`) is a shorthand for `A B`, meaning the diff is computed from the tip of A to the tip of B. The output shows what changed between the two snapshots. This is equivalent to `git diff <commit1> <commit2>`. The comparison is endpoint-to-endpoint, not range-based.

**Beginner-Friendly Explanation**
You can compare any two points in your project's history. For example, `git diff HEAD~3 HEAD` shows what changed in the last three commits. `git diff abc123 def456` shows the difference between those two specific commits.

### Purposes

- To see what changed between two specific points in history.
- To review the cumulative effect of a series of commits.
- To verify the contents of a release compared to a previous release.
- To generate a patch between two versions.
- To understand the evolution of a file or project.

### Syntax Rules and Structure

**Complete General Syntax**

```
git diff <commit1>..<commit2> [--] [<path>...]
git diff <commit1> <commit2> [--] [<path>...]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<commit1>` | The “old” commit (base). |
| `<commit2>` | The “new” commit (target). |
| `..` | Range notation (equivalent to a space). |
| `[--] [<path>...]` | Optional path limiting. |

**Syntax Rules**

- `A..B` and `A B` are equivalent for `git diff`.
- The order matters: `A..B` shows changes from A to B; `B..A` shows the reverse.
- Abbreviated hashes, branch names, tags, and relative references (`HEAD~1`) are all valid.
- `git diff <commit1>..<commit2>` does not include changes from commits that are ancestors of both (unlike three-dot notation).

**Constraints and Limitations**

- **Endpoint comparison**: Two-dot diff compares only the two endpoints, not the entire range of commits.
- **Merge commits**: Comparing a merge commit to a parent may show a large diff; use `-m` for individual parent diffs.
- **Performance**: Comparing very old commits on large repositories can be slow.

### Annotated Code Examples

#### Example 1: Comparing Two Commits

```bash
# Create three commits
git init commit-diff-demo && cd commit-diff-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
echo "B" >> file.txt && git commit -am "Commit B"
echo "C" >> file.txt && git commit -am "Commit C"

# Compare the first and third commits
git diff HEAD~2..HEAD
```

**Expected Output:**
```
diff --git a/file.txt b/file.txt
index 1234567..89abcde 100644
--- a/file.txt
+++ b/file.txt
@@ -1 +1,3 @@
 A
+B
+C
```

**Why this output occurs:** `HEAD~2` is Commit A, and `HEAD` is Commit C. The diff shows the cumulative changes from A to C: the addition of “B” and “C”. This is an endpoint-to-endpoint comparison.

#### Example 2: Comparing a File Between Two Commits

```bash
git diff HEAD~1..HEAD -- file.txt
```

**Expected Output:**
```
diff --git a/file.txt b/file.txt
index 1234567..89abcde 100644
--- a/file.txt
+++ b/file.txt
@@ -1,2 +1,3 @@
 A
 B
+C
```

**Why this output occurs:** Path limiting restricts the diff to `file.txt`. Only the change from Commit B to Commit C is shown.

### Real-World Cases

- **Release comparison**: `git diff v1.0..v2.0` shows all changes between two releases.
- **Code review**: A reviewer uses `git diff abc123..def456` to see the cumulative changes in a pull request.
- **Patch generation**: `git diff v1.0..v2.0 > update.patch` creates a patch file for applying changes.

### References

- Git Documentation: git-diff (Comparing Commits) — https://git-scm.com/docs/git-diff#Documentation/git-diff.txt-ltcommitgtltcommitgt
- Git Documentation: gitrevisions (Specifying Ranges) — https://git-scm.com/docs/gitrevisions#_specifying_ranges
- Graphite: Using git diff Between Branches — https://www.graphite.com

---

## 5. Comparing Branches (`git diff branch1...branch2`)

### Definitions

**Core Definition**
`git diff <branch1>...<branch2>` shows the changes on `branch2` since it diverged from `branch1` (i.e., since the common ancestor).

**Technical Definition**
The three-dot notation (`A...B`) in `git diff` is equivalent to `git diff $(git merge-base A B) B`. It finds the merge base (common ancestor) of the two branches and compares it to the tip of `branch2`. This shows only the changes that were made on `branch2` since the branches diverged, ignoring changes that were made only on `branch1`. The two-dot notation (`A..B`) compares the two branch tips directly.

**Beginner-Friendly Explanation**
When you want to see what a feature branch has changed relative to the main branch, you use three dots: `git diff main...feature`. This shows only the changes made on the feature branch since it was created, not the changes that happened on main in the meantime. Two dots (`main..feature`) compares the two branch tips directly, which can include changes from both branches.

### Purposes

- To see what a feature branch adds relative to the main branch.
- To review changes before merging a feature branch.
- To understand the scope of a branch's divergence.
- To generate a patch of a feature branch.
- To compare a local branch with its remote-tracking counterpart.

### Syntax Rules and Structure

**Complete General Syntax**

```
git diff <branch1>...<branch2> [--] [<path>...]
git diff <branch1>..<branch2> [--] [<path>...]
git diff <branch1> <branch2> [--] [<path>...]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<branch1>...<branch2>` | Compare merge base to `<branch2>` tip. |
| `<branch1>..<branch2>` | Compare `<branch1>` tip to `<branch2>` tip. |
| `<branch1> <branch2>` | Same as two-dot (direct comparison). |

**Syntax Rules**

- Three-dot notation (`...`) uses the merge base as the “old” side.
- Two-dot notation (`..`) uses the tip of the first branch as the “old” side.
- The order matters: `main...feature` shows changes on `feature` since it diverged from `main`.
- `git diff main...feature` is equivalent to `git diff $(git merge-base main feature) feature`.

**Constraints and Limitations**

- **Merge base ambiguity**: If multiple merge bases exist (in complex histories), Git uses one arbitrarily; use `git merge-base --all` to see all.
- **No common ancestor**: Unrelated histories (e.g., after `git checkout --orphan`) have no merge base; three-dot diff fails.
- **Remote branches**: You can compare against remote-tracking branches (e.g., `git diff main...origin/feature`).

### Annotated Code Examples

#### Example 1: Three-Dot vs. Two-Dot Branch Comparison

```bash
# Create a branch and diverge
git init branch-diff-demo && cd branch-diff-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
git checkout -b feature
echo "B" >> file.txt && git commit -am "Commit B"
git checkout main
echo "C" >> file.txt && git commit -am "Commit C"

# Three-dot: changes on feature since it branched from main
git diff main...feature
```

**Expected Output:**
```
diff --git a/file.txt b/file.txt
index 1234567..89abcde 100644
--- a/file.txt
+++ b/file.txt
@@ -1 +1,2 @@
 A
+B
```

**Why this output occurs:** The merge base of `main` and `feature` is Commit A. The three-dot diff compares Commit A to the tip of `feature` (Commit B), showing only the addition of “B”. It does not show the addition of “C” on `main`.

#### Example 2: Two-Dot Branch Comparison

```bash
git diff main..feature
```

**Expected Output:**
```
diff --git a/file.txt b/file.txt
index 89abcde..1234567 100644
--- a/file.txt
+++ b/file.txt
@@ -1,2 +1,2 @@
 A
-B
+C
```

**Why this output occurs:** The two-dot diff compares the tip of `main` (Commit C) to the tip of `feature` (Commit B). It shows that “B” was removed and “C” was added—a direct endpoint-to-endpoint comparison. This is different from the three-dot result.

### Real-World Cases

- **Pull request review**: A reviewer runs `git diff main...feature` to see only the changes introduced by the feature branch.
- **Release branch comparison**: `git diff release...main` shows what's new on main since the release branch was created.
- **Fork comparison**: A maintainer runs `git diff upstream/main...contributor/feature` to see a contributor's changes since they forked.

### References

- Git Documentation: git-diff (Comparing Branches) — https://git-scm.com/docs/git-diff#Documentation/git-diff.txt-ltcommitgtltcommitgt
- Stack Overflow: Difference Between Two-Dot and Three-Dot Diff — https://stackoverflow.com/questions/39927277
- Graphite: Using git diff Between Branches — https://www.graphite.com

---

## 6. Reading Diff Output (Unified Diff Format)

### Definitions

**Core Definition**
The unified diff format is a standard textual representation of differences between two files, showing changed lines with `+` and `-` prefixes and surrounding context lines.

**Technical Definition**
The unified diff format (originally from the Unix `diff -u` command) begins with a two-line header identifying the old and new files. It then presents one or more “hunks,” each preceded by a hunk header (`@@ -old_start,old_count +new_start,new_count @@`). Within each hunk, lines common to both files begin with a space, lines removed from the old file begin with `-`, and lines added to the new file begin with `+`. Git extends this format with additional metadata headers (e.g., `diff --git`, `index`, `new file mode`, `deleted file mode`, `similarity index`).

**Beginner-Friendly Explanation**
When you run `git diff`, the output has a specific format. The first few lines tell you which files are being compared. Then come the “hunks”—blocks of changes. Lines that were removed start with `-`, lines that were added start with `+`, and unchanged lines start with a space. This format is used by many tools and can be applied as a patch.

### Purposes

- To understand exactly which lines changed between two versions.
- To manually review changes before committing or merging.
- To generate patches that can be applied elsewhere.
- To debug merge conflicts by inspecting the conflicting changes.
- To feed diff output to external tools or scripts.

### Syntax Rules and Structure

**Complete General Syntax (Unified Diff Format)**

```
diff --git a/<old-file> b/<new-file>
[metadata headers]
--- a/<old-file>
+++ b/<new-file>
@@ -<old-start>,<old-count> +<new-start>,<new-count> @@
 context line
-removed line
+added line
 context line
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `diff --git a/file b/file` | Git-specific header identifying the file pair. |
| `index <old-hash>..<new-hash> <mode>` | Abbreviated blob hashes and file mode. |
| `--- a/<file>` | The “old” file (pre-image). |
| `+++ b/<file>` | The “new” file (post-image). |
| `@@ -old_start,old_count +new_start,new_count @@` | Hunk header: line ranges in old and new files. |
| ` ` (space) | A context line (unchanged). |
| `-` | A line removed from the old file. |
| `+` | A line added to the new file. |

**Metadata Headers in Git Diff Output**

| Header | Meaning |
|--------|---------|
| `diff --git a/<old> b/<new>` | Identifies the file pair. |
| `index <hash>..<hash> <mode>` | Blob hashes and mode. |
| `new file mode <mode>` | The file was created. |
| `deleted file mode <mode>` | The file was deleted. |
| `similarity index <n>%` | Rename/copy similarity percentage. |
| `rename from <old>` / `rename to <new>` | Rename detection. |
| `copy from <old>` / `copy to <new>` | Copy detection. |
| `Binary files differ` | Binary file change (no textual diff). |

**Hunk Header Line Number Format**

| Format | Meaning |
|--------|---------|
| `-1,3` | Starting at line 1, 3 lines in the old file. |
| `+1,4` | Starting at line 1, 4 lines in the new file. |
| `-1` | Single line at line 1 (count omitted). |
| `+0,0` | Empty hunk (e.g., file creation/deletion). |

**Syntax Rules**

- The `---` and `+++` headers use `a/` and `b/` prefixes by default (configurable via `--no-prefix`).
- The hunk header uses `@@` delimiters; the line numbers are 1-based.
- Context lines begin with a space character; removed lines with `-`; added lines with `+`.
- The `\ No newline at end of file` marker indicates a file without a trailing newline.

**Constraints and Limitations**

- **Binary files**: Unified diff is not meaningful for binary files; Git reports “Binary files differ.”
- **Large diffs**: Very large hunks can be difficult to read; use `-U<lines>` to reduce context.
- **Line endings**: CRLF vs. LF differences may appear as entire-file rewrites unless `core.autocrlf` is configured.
- **Mode changes**: File mode changes (e.g., executable bit) appear as `old mode` / `new mode` headers.

### Annotated Code Examples

#### Example 1: Annotated Unified Diff

```bash
git diff HEAD~1..HEAD
```

**Expected Output:**
```
diff --git a/example.txt b/example.txt
index 8a1b2c3..9d4e5f6 100644
--- a/example.txt
+++ b/example.txt
@@ -1,4 +1,5 @@
 First line
 Second line
-Third line (old)
+Third line (new)
+Fourth line
```

**Breakdown:**
- `diff --git a/example.txt b/example.txt`: Git header identifying the file pair.
- `index 8a1b2c3..9d4e5f6 100644`: Abbreviated blob hashes and file mode (`100644` = regular non-executable file).
- `--- a/example.txt`: The old file (pre-image).
- `+++ b/example.txt`: The new file (post-image).
- `@@ -1,4 +1,5 @@`: Hunk header. Old file: starting at line 1, 4 lines. New file: starting at line 1, 5 lines.
- ` First line`: Context line (unchanged).
- ` Second line`: Context line (unchanged).
- `-Third line (old)`: Line removed from the old file.
- `+Third line (new)`: Line added to the new file.
- `+Fourth line`: Another line added.

#### Example 2: File Mode Change

```bash
chmod +x script.sh
git diff
```

**Expected Output:**
```
diff --git a/script.sh b/script.sh
old mode 100644
new mode 100755
```

**Why this output occurs:** The file's mode changed from `100644` (regular file) to `100755` (executable). Git reports this as a mode change without a content diff.

### Real-World Cases

- **Patch generation**: `git diff > fix.patch` creates a unified diff that can be applied with `git apply` or `patch -p1`.
- **Code review**: Reviewers read unified diff output to understand proposed changes.
- **Conflict resolution**: During a merge conflict, the conflicted file contains unified diff markers (`<<<<<<<`, `=======`, `>>>>>>>`).
- **CI checks**: Scripts parse `git diff --check` output to detect whitespace errors.

### References

- GNU Diffutils: Detailed Description of Unified Format — https://www.gnu.org/software/diffutils/manual/html_node/Detailed-Unified.html
- GNU Diffutils: Unified Format — https://www.gnu.org/software/diffutils/manual/html_node/Unified-Format.html
- Git Documentation: git-diff (Generating Patch Text) — https://git-scm.com/docs/git-diff#_generating_patch_text_with_p
- Git Documentation: git-format-patch — https://git-scm.com/docs/git-format-patch

---

## 7. External Diff Tools (`git difftool`)

### Definitions

**Core Definition**
`git difftool` is a Git command that launches an external diff tool to display changes in a visual, side-by-side format instead of the default terminal-based unified diff.

**Technical Definition**
`git difftool` is a front-end to `git diff` that accepts the same options and arguments. It invokes a configured external diff program (e.g., `meld`, `kdiff3`, `vimdiff`, `emerge`, `kompare`) for each file pair. The tool is configured via the `diff.tool` configuration variable, and per-tool settings (`difftool.<tool>.path`, `difftool.<tool>.cmd`) allow customisation. Temporary files containing the pre-image (`$LOCAL`) and post-image (`$REMOTE`) are passed to the tool.

**Beginner-Friendly Explanation**
If you prefer a visual side-by-side comparison over the text-based `git diff`, you can use `git difftool`. It opens your favourite diff program (like Meld or KDiff3) and shows the changes in a graphical window. You can configure which tool to use and customise how it's launched.

### Purposes

- To view differences in a visual, side-by-side format.
- To use familiar GUI diff tools for complex changes.
- To compare files in a directory tree (`--dir-diff`).
- To integrate Git with existing diff workflows.
- To make code review easier for visual learners.

### Syntax Rules and Structure

**Complete General Syntax**

```
git difftool [<options>] [<commit> [<commit>]] [--] [<path>...]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `-t <tool>` / `--tool=<tool>` | Use the specified diff tool. |
| `--tool-help` | List available diff tools. |
| `-d` / `--dir-diff` | Copy files to a temporary location and perform a directory diff. |
| `-y` / `--no-prompt` | Do not prompt before launching the diff tool. |
| `--prompt` | Prompt before each invocation (default). |
| `-g` / `--gui` | Use `diff.guitool` instead of `diff.tool`. |
| `-x <command>` / `--extcmd=<command>` | Use a custom command instead of a configured tool. |
| `--symlinks` / `--no-symlinks` | Control symlink creation in `--dir-diff` mode. |

**Configuration Variables**

| Variable | Description |
|----------|-------------|
| `diff.tool` | Default diff tool for `git difftool`. |
| `diff.guitool` | Default GUI diff tool (used with `-g`/`--gui`). |
| `difftool.<tool>.path` | Explicit path to the tool's executable. |
| `difftool.<tool>.cmd` | Custom command line for the tool. |
| `difftool.prompt` | Whether to prompt before each invocation. |

**Syntax Rules**

- `git difftool` accepts the same arguments as `git diff`.
- If no tool is specified, `diff.tool` is used.
- `$LOCAL` is set to the pre-image file; `$REMOTE` is set to the post-image file.
- `$MERGED` is the name of the file being compared; `$BASE` is provided for compatibility.
- `--dir-diff` creates a temporary directory structure for directory-level comparison.

**Constraints and Limitations**

- **GUI dependency**: Requires a graphical environment for GUI tools.
- **Tool availability**: The tool must be installed and on `PATH` (or configured with `path`).
- **Performance**: Launching an external tool for each file can be slow for large changesets.
- **Version-specific**: `--dir-diff` requires Git 1.7.11+; `difftool.guiDefault` requires Git 2.28+.

### Annotated Code Examples

#### Example 1: Configuring and Using Meld

```bash
# Configure Meld as the default difftool
git config --global diff.tool meld

# Compare the working tree to the index
git difftool
```

**Expected Output:**
```
Viewing (1/3): 'file1.txt'
Launch 'meld' [Y/n]? y
...
```

**Why this output occurs:** `git difftool` reads the `diff.tool` configuration, finds Meld on `PATH`, and launches it for each changed file. The prompt asks for confirmation unless `--no-prompt` is used.

#### Example 2: Using a Custom Tool via `--extcmd`

```bash
git difftool --extcmd="diff -u" HEAD~1..HEAD
```

**Expected Output:**
```
--- a/file.txt
+++ b/file.txt
@@ -1 +1,2 @@
 Line 1
+Line 2
```

**Why this output occurs:** `--extcmd="diff -u"` tells `git difftool` to run the Unix `diff -u` command instead of a configured tool. `$LOCAL` and `$REMOTE` are passed as arguments to `diff -u`.

#### Example 3: Directory Diff with `--dir-diff`

```bash
git difftool --dir-diff HEAD~1..HEAD
```

**Expected Output:**
```
Viewing (1/1): 'src/'
Launch 'meld' [Y/n]? y
```

**Why this output occurs:** `--dir-diff` copies the entire directory tree to a temporary location and launches the diff tool in directory-comparison mode. This is useful for comparing multiple files at once.

### Real-World Cases

- **Code review**: A developer uses `git difftool` with Meld to review changes visually before committing.
- **Merge conflict resolution**: `git difftool --tool=meld` helps resolve conflicts by showing both versions side-by-side.
- **Large refactoring**: `git difftool --dir-diff` compares entire directory trees after a refactoring.
- **CI integration**: A CI script uses `git difftool --extcmd="diff -u"` to generate unified diffs for automated checks.

### References

- Git Documentation: git-difftool Manual Page — https://git-scm.com/docs/git-difftool
- Git Documentation: git-config (diff.tool) — https://git-scm.com/docs/git-config#Documentation/git-config.txt-difftool
- Git Documentation: git-difftool (Configuration Variables) — https://git-scm.com/docs/git-difftool#_configuration_variables
- NYCU CSIT Mirror: git-difftool Manual Page — http://kernel.cs.nycu.edu.tw/software/scm/git/docs/git-difftool.html

---

## References

- Git Documentation: git-diff Manual Page — https://git-scm.com/docs/git-diff
- Git Documentation: git-difftool Manual Page — https://git-scm.com/docs/git-difftool
- Git Documentation: git-config Manual Page — https://git-scm.com/docs/git-config
- Git Documentation: gitrevisions Manual Page — https://git-scm.com/docs/gitrevisions
- Git Documentation: gittutorial — https://git-scm.com/docs/gittutorial
- GNU Diffutils: Detailed Description of Unified Format — https://www.gnu.org/software/diffutils/manual/html_node/Detailed-Unified.html
- GNU Diffutils: Unified Format — https://www.gnu.org/software/diffutils/manual/html_node/Unified-Format.html
- Mintlify: git diff Documentation — https://mintlify.wiki/git/git/commands/diff
- Stack Overflow: Difference Between Two-Dot and Three-Dot Diff — https://stackoverflow.com/questions/39927277
- Graphite: Using git diff Between Branches — https://www.graphite.com
- Microsoft Learn: Track Changes with git diff — https://learn.microsoft.com/en-us/training/modules/work-source-control-git/
- Pro Git Book: Viewing the Commit History — https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History
- Pro Git Book: Git Diff – Inspecting Changes — https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History