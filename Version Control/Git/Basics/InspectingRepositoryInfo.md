# Git Inspecting Repository Information: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Inspecting repository information is the practice of using Git's diagnostic commands to examine the state of the working tree, staging area, commit history, references, remote configuration, and local operation log without modifying any data.

**Technical Definition**
Git provides a suite of read-only inspection commands that query the repository's three core data structures—the working tree, the index, and the object database (DAG)—as well as configuration files and reference logs (reflogs). Commands such as `git status`, `git log`, `git show`, `git diff`, `git branch`, `git remote`, `git config --list`, and `git reflog` report differences, metadata, and historical records by reading these structures without altering them.

**Beginner-Friendly Explanation**
When you work with Git, you often need to ask questions: “What have I changed?”, “What is in this commit?”, “Which branches exist?”, “What did I do yesterday?” Git provides commands that answer these questions by looking at the repository and printing information. These commands are “read-only”—they don't change anything. They are your diagnostic toolkit for understanding what is happening in your project.

### Key Characteristics

- **Read-Only**: Inspection commands do not modify the repository, index, or working tree.
- **Locally Executable**: Most inspection commands operate entirely on local data, requiring no network access.
- **Composable**: Output can be piped to other tools (`grep`, `awk`, `less`) for filtering and formatting.
- **Format-Rich**: Many commands support `--format`, `--pretty`, and output modifiers (e.g., `--oneline`, `--stat`).
- **Safety Net**: `git reflog` records all reference movements, enabling recovery of “lost” commits.

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository (initialised or cloned).
- Basic familiarity with terminal commands and text editors.
- Understanding of the three-tree architecture (working tree, index, HEAD).

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Scripts that inspect repository state for build decisions.
- **Software Configuration Management (SCM)** : Auditing and compliance.
- **Collaborative Development**: Reviewing changes before merging.

### Core Concepts / Features

1. **git status** — Working tree and staging area status.
2. **git log** — Commit history inspection (including `--oneline`, `--graph`, `--stat`, `-p`).
3. **git show** — Inspecting specific objects (commits, blobs, trees, tags).
4. **git diff** — Comparing working directory to index.
5. **git diff --staged / --cached** — Comparing index to HEAD.
6. **git branch** — Listing, filtering (`--merged`), and verbose tracking (`-vv`).
7. **git remote** — Managing and inspecting remotes (`show`).
8. **git config --list** — Inspecting configuration at all levels.
9. **git reflog** — Tracking HEAD movements and recovering lost commits.

---

## 1. git status

### Definitions

**Core Definition**
`git status` is a command that displays the state of the working tree and the staging area, showing which files are modified, staged, untracked, or ignored.

**Technical Definition**
`git status` compares three trees: HEAD (the last commit), the index (staging area), and the working tree. It reports differences between HEAD and the index (staged changes), between the index and the working tree (unstaged changes), and files present in the working tree but not tracked (untracked). The `-s` / `--short` option produces a compact, two-column output where the first column indicates the index status and the second column indicates the working tree status for each file.

**Beginner-Friendly Explanation**
`git status` is your Git dashboard. It tells you what's going on: which files you've changed, which changes you've already staged for the next commit, and which files Git doesn't know about yet. When you're confused, just run `git status`.

### Purposes

- To determine the current state of files in the working tree and staging area.
- To identify which changes are staged and which are not.
- To discover untracked files that Git is not yet tracking.
- To verify the current branch and its tracking relationship with the remote.
- To guide the user on the next steps (e.g., `git add`, `git commit`).

### Syntax Rules and Structure

**Complete General Syntax**

```
git status [<options>] [--] [<pathspec>...]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `-s` / `--short` | Give output in short format. |
| `-b` / `--branch` | Show branch information even in short format. |
| `--porcelain` | Machine-readable output (stable across Git versions). |
| `-v` / `--verbose` | Show textual changes of uncommitted files. |
| `-u[<mode>]` / `--untracked-files[=<mode>]` | Show untracked files (no, normal, all). |
| `--ignored[=<mode>]` | Show ignored files (traditional, no, matching). |

**Short Format (`-s`) Structure**

```
XY PATH
```

| Code | Meaning |
|------|---------|
| `??` | Untracked file |
| `A` | Added (staged) |
| `M` | Modified |
| `D` | Deleted |
| `R` | Renamed |
| `C` | Copied |
| `U` | Updated but unmerged |

**Syntax Rules**

- The first column (`X`) shows the status of the file in the **index** (staging area).
- The second column (`Y`) shows the status of the file in the **working tree**.
- `??` means the file is untracked.
- A file that is staged but not modified in the working tree shows `A ` (A in first column, space in second).
- A file that is modified but not staged shows ` M` (space in first column, M in second).
- A file that is both staged and modified shows `MM`.

**Constraints and Limitations**

- **Performance**: On very large repositories, `git status` can be slow. The untracked cache and FSMonitor extensions improve performance.
- **Platform-specific**: File mode changes (executable bit) may not be detected on filesystems that don't support Unix permissions.
- **Ignored files**: By default, ignored files are not shown; use `--ignored` to include them.

### Annotated Code Examples

#### Example 1: Normal `git status`

```bash
# Create a repository and modify files
mkdir status-demo && cd status-demo && git init
echo "Hello" > tracked.txt && git add tracked.txt && git commit -m "Initial"
echo "World" >> tracked.txt          # Modify tracked file
echo "New" > untracked.txt           # Create untracked file
git add tracked.txt                 # Stage the modification

git status
```

**Expected Output:**
```
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   tracked.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	untracked.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

**Why this output occurs:** `tracked.txt` has been modified and staged, so it appears under “Changes to be committed.” `untracked.txt` is new and has never been tracked, so it appears under “Untracked files.” Git provides hints on how to unstage or add files.

#### Example 2: Compact `git status -s`

```bash
git status -s
```

**Expected Output:**
```
M  tracked.txt
?? untracked.txt
```

**Why this output occurs:** The short format uses two columns. `M ` (M in first column, space in second) means the file is modified in the index (staged) but not modified in the working tree. `??` means untracked. This compact view is useful for scripting and quick checks.

### Real-World Cases

- **Pre-commit check**: A developer runs `git status` before committing to verify which files will be included.
- **CI/CD scripting**: A CI script uses `git status --porcelain` to check if there are uncommitted changes that would cause a build failure.
- **Conflict resolution**: During a merge conflict, `git status` shows “Unmerged paths” listing conflicted files.

### References

- Git Documentation: git-status Manual Page — https://git-scm.com/docs/git-status
- Git Documentation: Git Basics – Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

---

## 2. git log

### Definitions

**Core Definition**
`git log` is a command that displays the commit history of a repository, showing commits in reverse-chronological order by default.

**Technical Definition**
`git log` traverses the commit DAG starting from the current HEAD (or specified revision) and prints commit objects. It supports extensive formatting options including `--oneline` (one line per commit), `--graph` (ASCII graph of branch structure), `--stat` (file change statistics per commit), and `-p` / `--patch` (full diff for each commit).

**Beginner-Friendly Explanation**
`git log` is your project's history book. It shows you every commit that has been made, who made it, when, and what changed. You can customise how much detail you see—from a simple one-line summary to a full diff of every change.

### Purposes

- To view the chronological history of commits.
- To understand the branch and merge structure of the project.
- To inspect what changed in each commit.
- To search for specific commits by author, date, or message.
- To provide data for scripts and reports.

### Syntax Rules and Structure

**Complete General Syntax**

```
git log [<options>] [<revision-range>] [[--] <path>...]
```

**Key Formatting Options**

| Option | Description |
|--------|-------------|
| `--oneline` | One line per commit: abbreviated hash + subject. |
| `--graph` | ASCII graph showing branch/merge structure. |
| `--stat` | Show file change statistics (insertions/deletions) per commit. |
| `-p` / `--patch` | Show full diff (patch) for each commit. |
| `--pretty=<format>` | Custom format (e.g., `format:"%h %s"`, `fuller`). |
| `--abbrev-commit` | Use abbreviated commit hashes. |
| `--decorate` | Show ref names (branches, tags). |
| `--all` | Show all branches, not just current. |
| `-n <number>` | Limit to the last `<number>` commits. |

**Syntax Rules**

- `--oneline` is a shorthand for `--pretty=oneline --abbrev-commit`.
- `--graph` can be combined with `--oneline`, `--stat`, or `-p`.
- `--stat` summarises changes; `-p` shows full textual diffs.
- Revision ranges like `A..B` show commits reachable from B but not A.
- Path limiting restricts output to commits that touch specified files.

**Constraints and Limitations**

- **Performance**: On very large repositories, `git log -p` can be slow.
- **Shallow clones**: In a shallow clone, history is truncated; `git log` shows only the available commits.
- **Merge commits**: By default, `git log` may simplify history; use `--full-history` or `-m` to see all commits.

### Annotated Code Examples

#### Example 1: `git log --oneline --graph`

```bash
# Create a branch and merge
git init log-demo && cd log-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
echo "B" >> file.txt && git commit -am "Commit B"
git checkout -b feature
echo "C" >> file.txt && git commit -am "Commit C"
git checkout main
git merge feature --no-edit

# View the graph
git log --oneline --graph --all
```

**Expected Output:**
```
*   4d5e6f7 (HEAD -> main) Merge branch 'feature'
|\
| * 1a2b3c4 (feature) Commit C
* | 9a0b1c2 Commit B
|/
* 3d4e5f6 Commit A
```

**Why this output occurs:** `--graph` draws the DAG with ASCII characters, showing the merge and branch structure. `--oneline` compresses each commit to one line. `--all` includes all branches.

#### Example 2: `git log --stat`

```bash
git log --stat -2
```

**Expected Output:**
```
commit 4d5e6f7...
Author: User <user@example.com>
Date:   ...

    Merge branch 'feature'

 file.txt | 2 ++
 1 file changed, 2 insertions(+)
```

**Why this output occurs:** `--stat` shows a summary of file changes per commit. It lists each changed file with the number of insertions and deletions, followed by a total line.

#### Example 3: `git log -p`

```bash
git log -p -1
```

**Expected Output:**
```
commit 4d5e6f7...
Author: User <user@example.com>
Date:   ...

    Merge branch 'feature'

diff --git a/file.txt b/file.txt
index 1234567..89abcde 100644
--- a/file.txt
+++ b/file.txt
@@ -1,2 +1,2 @@
 A
 B
+C
```

**Why this output occurs:** `-p` shows the full patch (diff) for the commit, including the commit message, author, date, and the textual changes.

### Real-World Cases

- **Release notes**: A maintainer runs `git log --oneline --since="1 month ago"` to generate release notes.
- **Code review**: A reviewer uses `git log -p -- <file>` to see the history of changes to a specific file.
- **Debugging**: A developer uses `git log --graph --all` to understand how branches diverged and merged.

### References

- Git Documentation: git-log Manual Page — https://git-scm.com/docs/git-log
- Git Documentation: Pretty Formats — https://git-scm.com/docs/git-log#_pretty_formats

---

## 3. git show

### Definitions

**Core Definition**
`git show` is a command that displays various Git objects (commits, tags, trees, blobs) in a human-readable format.

**Technical Definition**
`git show` resolves the given object (default: HEAD) and prints its content. For commits, it shows the commit message and textual diff. For tags, it shows the tag message and referenced objects. For trees, it lists entries (equivalent to `git ls-tree`). For blobs, it prints the raw content.

**Beginner-Friendly Explanation**
`git show` lets you examine a single Git object in detail. You can look at a commit, a tag, a file at a specific version, or even a whole directory tree. It's like a magnifying glass for Git's internal objects.

### Purposes

- To inspect a specific commit's metadata and changes.
- To view the content of a file (blob) at a given revision.
- To examine annotated tag information.
- To list the contents of a tree object.
- To verify the existence and content of Git objects.

### Syntax Rules and Structure

**Complete General Syntax**

```
git show [<options>] [<object>...]
```

**Common Object Specifiers**

| Specifier | Description |
|-----------|-------------|
| `HEAD` | The latest commit on the current branch. |
| `<commit-hash>` | A specific commit by SHA-1. |
| `<tag>` | An annotated tag. |
| `<commit>:<path>` | A blob at a specific path in a commit. |
| `<commit>^{tree}` | The tree object of a commit. |

**Key Options**

| Option | Description |
|--------|-------------|
| `--stat` | Show diffstat instead of full patch. |
| `--oneline` | One line per object. |
| `-p` / `--patch` | Show patch (default for commits). |
| `--pretty=<format>` | Custom commit format. |

**Syntax Rules**

- If no object is specified, `HEAD` is used.
- For commits, the output includes the commit metadata, message, and diff.
- For annotated tags, the output includes the tag object and the referenced commit.
- For trees, the output lists entries with mode, type, hash, and name.

**Constraints and Limitations**

- **Binary blobs**: `git show` may output binary data to the terminal; use `--textconv` or redirect to a file.
- **Merge commits**: By default, `git show` may show a combined diff; use `-m` to show individual parents.

### Annotated Code Examples

#### Example 1: Inspecting a Commit

```bash
# Show the latest commit
git show HEAD
```

**Expected Output:**
```
commit 4d5e6f7...
Author: User <user@example.com>
Date:   ...

    Commit message

diff --git a/file.txt b/file.txt
...
```

**Why this output occurs:** `git show HEAD` prints the commit metadata, message, and the diff between the commit and its parent.

#### Example 2: Inspecting a Blob

```bash
# Show the content of a file at a specific commit
git show HEAD:file.txt
```

**Expected Output:**
```
A
B
C
```

**Why this output occurs:** The syntax `HEAD:file.txt` tells Git to show the blob object for `file.txt` in the commit pointed to by HEAD. Only the file content is printed.

#### Example 3: Inspecting a Tree

```bash
git show HEAD^{tree}
```

**Expected Output:**
```
100644 blob a1b2c3d4e5f6...	file.txt
100644 blob f6e5d4c3b2a1...	README.md
```

**Why this output occurs:** `HEAD^{tree}` refers to the tree object of the HEAD commit. `git show` lists the tree entries: mode, type, object hash, and filename.

### Real-World Cases

- **Verifying a tag**: A release manager runs `git show v1.0.0` to confirm the tag points to the correct commit and contains the right message.
- **Examining a file at an old revision**: A developer uses `git show abc123:src/main.c` to view the file as it was in an older commit.
- **Debugging a merge**: `git show -m <merge-commit>` shows the diffs against each parent, helping understand merge resolution.

### References

- Git Documentation: git-show Manual Page — https://git-scm.com/docs/git-show
- Git Documentation: gitrevisions (Object Specifiers) — https://git-scm.com/docs/gitrevisions

---

## 4. git diff

### Definitions

**Core Definition**
`git diff` is a command that shows changes between various Git data sources, such as the working tree, the index, commits, and branches.

**Technical Definition**
`git diff` computes the difference between two states. Without arguments, it shows changes between the working tree and the index. With `--cached` (or `--staged`), it shows changes between the index and a specified commit (default: HEAD). It can also compare two commits, two branches, or two files.

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
```

**Key Forms**

| Form | Comparison |
|------|------------|
| `git diff` | Working tree vs. index. |
| `git diff --cached` | Index vs. HEAD (staged changes). |
| `git diff <commit>` | Working tree vs. commit. |
| `git diff <commit1> <commit2>` | Commit1 vs. Commit2. |
| `git diff <branch1>..<branch2>` | Tip of branch1 vs. tip of branch2. |

**Common Options**

| Option | Description |
|--------|-------------|
| `--stat` | Show diffstat summary. |
| `--name-only` | Show only file names. |
| `--name-status` | Show file names and status (A/M/D). |
| `-p` / `--patch` | Show patch (default). |
| `--color` | Colourise output. |
| `-w` | Ignore whitespace changes. |

**Syntax Rules**

- `git diff` without arguments compares working tree and index.
- `git diff --cached` compares index and HEAD.
- `git diff HEAD` compares working tree and HEAD (both staged and unstaged changes).
- Path limiting restricts the diff to specified files or directories.

**Constraints and Limitations**

- **Binary files**: `git diff` cannot show textual diffs for binary files by default; it reports “Binary files differ.”
- **Renames**: Rename detection requires `-M`; otherwise a rename appears as a delete + add.
- **Whitespace**: Use `-w` or `--ignore-all-space` to ignore whitespace-only changes.

### Annotated Code Examples

#### Example 1: `git diff` (Working Tree vs. Index)

```bash
# Modify a file without staging
echo "New line" >> file.txt
git diff
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
+New line
```

**Why this output occurs:** `git diff` compares the working tree version of `file.txt` with the index version. The unstaged change (`New line`) appears as a `+` line.

#### Example 2: `git diff --stat`

```bash
git diff --stat
```

**Expected Output:**
```
 file.txt | 1 +
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** `--stat` provides a summary: one file changed, one insertion.

### Real-World Cases

- **Pre-commit review**: A developer runs `git diff` to review unstaged changes before deciding what to stage.
- **Code review**: A reviewer uses `git diff main..feature` to see all changes introduced by a feature branch.
- **Patch generation**: `git diff > fix.patch` creates a patch file that can be applied elsewhere.

### References

- Git Documentation: git-diff Manual Page — https://git-scm.com/docs/git-diff
- Git Documentation: Git Diff – Inspecting Changes — https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History

---

## 5. git diff --staged (--cached)

### Definitions

**Core Definition**
`git diff --staged` (synonym: `--cached`) shows the changes that are staged for the next commit by comparing the index to HEAD.

**Technical Definition**
The `--cached` option (with `--staged` as a synonym) instructs `git diff` to compare the index against a specified commit (default: HEAD). If HEAD does not exist (e.g., an unborn branch), it shows all staged changes. This is the diff that will be included if `git commit` is run without further modification.

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
 A
 B
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

**Why this output occurs:** `--stat` summarises the staged changes.

### Real-World Cases

- **Pre-commit hook**: A pre-commit hook runs `git diff --staged --check` to detect whitespace errors before allowing a commit.
- **Selective commit**: A developer stages several files and runs `git diff --staged` to confirm only the intended changes are included.
- **Patch extraction**: `git diff --staged > staged.patch` creates a patch of the proposed commit.

### References

- Git Documentation: git-diff (--cached) — https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---cached
- Git Documentation: git-diff (--staged) — https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---staged

---

## 6. git branch

### Definitions

**Core Definition**
`git branch` is a command for listing, creating, deleting, and managing branches.

**Technical Definition**
`git branch` reads and writes reference files under `refs/heads/`. In list mode (the default), it displays local branches. With `--merged`, it filters branches whose tips are reachable from a specified commit (default HEAD). With `-vv` (verbose twice), it shows the SHA-1, commit subject, upstream branch, and worktree path for each branch.

**Beginner-Friendly Explanation**
`git branch` is your branch manager. Run it without arguments to see all local branches. Use `--merged` to find branches that have already been merged into your current branch (safe to delete). Use `-vv` to see extra details: which commit each branch points to, and what remote branch it tracks.

### Purposes

- To list all local branches.
- To identify branches that have been merged (`--merged`).
- To identify branches that have not been merged (`--no-merged`).
- To show tracking relationships with remote branches (`-vv`).
- To create, rename, or delete branches.

### Syntax Rules and Structure

**Complete General Syntax**

```
git branch [--list] [<pattern>...]
git branch (-v | -vv | --verbose) [--list] [<pattern>...]
git branch (--merged [<commit>] | --no-merged [<commit>]) [--list] [<pattern>...]
git branch <branch-name> [<start-point>]
git branch (-d | -D) <branch-name>...
git branch (-m | -M) [<old-branch>] <new-branch>
```

**Key Options**

| Option | Description |
|--------|-------------|
| `--list` | List branches (default). |
| `-v` / `--verbose` | Show SHA-1 and commit subject. |
| `-vv` | Show additional tracking info (upstream, worktree). |
| `--merged [<commit>]` | List branches merged into `<commit>` (default HEAD). |
| `--no-merged [<commit>]` | List branches not merged into `<commit>`. |
| `-a` / `--all` | List both local and remote-tracking branches. |
| `-r` / `--remotes` | List remote-tracking branches. |

**Syntax Rules**

- `--merged` implies `--list`.
- `-vv` prints the upstream branch name and the path of the linked worktree (if any).
- `--merged` is useful to find branches that can be safely deleted.
- `--no-merged` finds branches that still need to be merged.

**Constraints and Limitations**

- **Remote-tracking branches**: `git branch -r` lists remote-tracking branches, but these are read-only references.
- **Detached HEAD**: In detached HEAD state, `git branch` shows no current branch (nothing is printed with `*`).
- **Performance**: On repositories with thousands of branches, `git branch -a` can be slow.

### Annotated Code Examples

#### Example 1: Listing Branches with `-vv`

```bash
git branch -vv
```

**Expected Output:**
```
* main    4d5e6f7 [origin/main] Commit message
  feature 1a2b3c4 Commit on feature
```

**Why this output occurs:** `-vv` shows the current branch (`main`) with an asterisk, its abbreviated SHA-1, its upstream branch (`origin/main`), and the commit subject. `feature` has no upstream, so no tracking info is shown.

#### Example 2: Filtering with `--merged`

```bash
git branch --merged
```

**Expected Output:**
```
* main
  feature
```

**Why this output occurs:** `--merged` lists only branches whose tips are reachable from HEAD. Both `main` and `feature` are merged into HEAD (meaning their commits are ancestors of HEAD). Branches not fully merged are omitted.

#### Example 3: Finding Unmerged Branches

```bash
git branch --no-merged
```

**Expected Output:**
```
  experiment
```

**Why this output occurs:** `--no-merged` lists branches whose tips are **not** reachable from HEAD. `experiment` has commits that are not in HEAD, so it appears.

### Real-World Cases

- **Cleanup**: A developer runs `git branch --merged` to find branches that can be safely deleted after merging.
- **Tracking check**: `git branch -vv` shows which local branches are tracking remote branches and whether they are ahead/behind.
- **Release management**: A release manager uses `git branch --no-merged release` to see which feature branches are not yet in the release branch.

### References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: Git Branching – Branch Management — https://git-scm.com/book/en/v2/Git-Branching-Branch-Management

---

## 7. git remote

### Definitions

**Core Definition**
`git remote` is a command for managing the set of remote repositories whose branches are tracked locally.

**Technical Definition**
`git remote` reads and writes the `[remote "<name>"]` sections in `.git/config`. Each remote has a name, one or more fetch URLs, and a fetch refspec. The `git remote show <name>` subcommand queries the remote (or uses cached information with `-n`) to display its URL, tracking branches, and local branch configuration.

**Beginner-Friendly Explanation**
A “remote” is a repository hosted somewhere else—like GitHub, GitLab, or a colleague's machine. `git remote` lets you list your remotes, add new ones, change their URLs, and inspect their details. `git remote show origin` gives you a detailed report about the remote named `origin`.

### Purposes

- To list all configured remotes.
- To add, rename, or remove remotes.
- To change remote URLs.
- To inspect a remote's URL, tracking branches, and configuration (`show`).
- To prune stale remote-tracking branches.

### Syntax Rules and Structure

**Complete General Syntax**

```
git remote [-v | --verbose]
git remote add <name> <url>
git remote rename <old> <new>
git remote remove <name>
git remote set-url <name> <new-url>
git remote show [-n] <name>
git remote prune [-n | --dry-run] <name>
```

**Key Subcommands**

| Subcommand | Description |
|------------|-------------|
| `add` | Add a new remote. |
| `rename` | Rename a remote. |
| `remove` | Remove a remote. |
| `set-url` | Change a remote's URL. |
| `show` | Show detailed information about a remote. |
| `prune` | Delete stale remote-tracking branches. |

**`git remote show` Output Sections**

| Section | Description |
|---------|-------------|
| Fetch URL | The URL used for fetching. |
| Push URL | The URL used for pushing (if different). |
| HEAD branch | The remote's default branch. |
| Remote branches | Branches on the remote and their tracking status. |
| Local branches | Local branches configured for `git pull`. |
| Local refs | Local references configured for `git push`. |

**Syntax Rules**

- `git remote` with no arguments lists remote names.
- `git remote -v` shows remote names and URLs.
- `git remote show <name>` queries the remote (unless `-n` is used).
- `git remote prune <name>` removes remote-tracking branches that no longer exist on the remote.

**Constraints and Limitations**

- **Network dependency**: `git remote show` without `-n` requires network access to query the remote.
- **Cached information**: `-n` uses cached remote-tracking information, which may be stale.
- **Multiple URLs**: A remote can have multiple fetch URLs, but only one push URL is used by default.

### Annotated Code Examples

#### Example 1: Listing Remotes

```bash
git remote -v
```

**Expected Output:**
```
origin  https://github.com/user/repo.git (fetch)
origin  https://github.com/user/repo.git (push)
```

**Why this output occurs:** `-v` shows each remote's name followed by its fetch and push URLs. The default remote created by `git clone` is named `origin`.

#### Example 2: Inspecting a Remote with `show`

```bash
git remote show origin
```

**Expected Output:**
```
* remote origin
  Fetch URL: https://github.com/user/repo.git
  Push  URL: https://github.com/user/repo.git
  HEAD branch: main
  Remote branches:
    main    tracked
    feature tracked
  Local branches configured for 'git pull':
    main    merges with remote main
  Local refs configured for 'git push':
    main    pushes to main (up to date)
```

**Why this output occurs:** `git remote show origin` queries the remote and displays its URL, default branch, remote-tracking branches, and the local branches configured for pull and push.

### Real-World Cases

- **Fork workflow**: A developer adds an `upstream` remote pointing to the original repository: `git remote add upstream https://github.com/original/repo.git`.
- **URL change**: A repository moves to a new host; the developer runs `git remote set-url origin <new-url>`.
- **Pruning**: After branches are deleted on the remote, `git remote prune origin` removes the stale remote-tracking references.

### References

- Git Documentation: git-remote Manual Page — https://git-scm.com/docs/git-remote
- Git Documentation: Git Basics – Working with Remotes — https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes

---

## 8. git config --list

### Definitions

**Core Definition**
`git config --list` (or `git config list`) displays all configuration variables from all configuration sources (system, global, local, worktree).

**Technical Definition**
`git config list` reads configuration from the system file (`/etc/gitconfig`), global file (`~/.gitconfig` or `$XDG_CONFIG_HOME/git/config`), local file (`.git/config`), and worktree-specific file (`.git/config.worktree`). It prints each key-value pair in `key=value` format. The `--show-origin` option adds the source file for each entry.

**Beginner-Friendly Explanation**
`git config --list` shows all of Git's settings. It's like opening the settings panel of an application and seeing every option that has been configured. You can see settings from the system level (all users), global level (your user), and local level (this repository).

### Purposes

- To view all Git configuration settings in effect.
- To diagnose configuration issues (e.g., wrong user email).
- To verify which configuration file a setting comes from (`--show-origin`).
- To inspect settings at a specific level (`--system`, `--global`, `--local`).
- To export configuration for auditing or replication.

### Syntax Rules and Structure

**Complete General Syntax**

```
git config list [<options>]
git config --list [<options>]
git config --list --show-origin
git config --list --global
git config --list --system
git config --list --local
```

**Key Options**

| Option | Description |
|--------|-------------|
| `--list` | List all variables (default for `list` subcommand). |
| `--show-origin` | Show the origin file for each config entry. |
| `--show-scope` | Show the scope (system, global, local, worktree). |
| `--global` | List only global configuration. |
| `--system` | List only system configuration. |
| `--local` | List only local (repository) configuration. |
| `--worktree` | List only worktree configuration. |

**Syntax Rules**

- `git config list` and `git config --list` are equivalent.
- Without scope options, all scopes are read (system, global, local, worktree) in order of increasing precedence.
- The last value for a key wins.
- `--show-origin` helps identify which file is responsible for a setting.

**Constraints and Limitations**

- **Precedence**: Local overrides global, which overrides system. The listed output may include duplicate keys, with the most specific scope appearing later.
- **Includes**: Configuration files can include other files (`[include]` sections); `--list` shows the effective configuration after includes are processed.
- **Version-specific**: The `list` subcommand was introduced in Git 2.46.0; older versions use `--list`.

### Annotated Code Examples

#### Example 1: Basic `git config --list`

```bash
git config --list
```

**Expected Output:**
```
user.name=John Doe
user.email=john@example.com
core.editor=vim
init.defaultBranch=main
remote.origin.url=https://github.com/user/repo.git
```

**Why this output occurs:** Git reads all configuration files and prints each key-value pair. The output includes settings from system, global, and local scopes.

#### Example 2: `git config --list --show-origin`

```bash
git config --list --show-origin
```

**Expected Output:**
```
file:/etc/gitconfig    core.autocrlf=input
file:/home/user/.gitconfig    user.name=John Doe
file:/home/user/.gitconfig    user.email=john@example.com
file:.git/config    remote.origin.url=https://github.com/user/repo.git
```

**Why this output occurs:** `--show-origin` prefixes each entry with the file path where the setting is defined. This is invaluable for debugging configuration conflicts.

#### Example 3: `git config --list --local`

```bash
git config --list --local
```

**Expected Output:**
```
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
remote.origin.url=https://github.com/user/repo.git
```

**Why this output occurs:** `--local` limits the output to the repository's `.git/config` file. This shows only settings specific to this repository.

### Real-World Cases

- **Troubleshooting identity**: A developer commits with the wrong email and runs `git config --list --show-origin | grep user.email` to find which file defines the incorrect email.
- **CI configuration**: A CI script runs `git config --list` to document the environment for reproducibility.
- **Onboarding**: A new team member compares their `git config --list --global` with a colleague's to match settings.

### References

- Git Documentation: git-config Manual Page — https://git-scm.com/docs/git-config
- Git Documentation: git-config (list subcommand) — https://git-scm.com/docs/git-config#Documentation/git-config.txt-list

---

## 9. git reflog

### Definitions

**Core Definition**
`git reflog` is a command that displays the reference log, which records when the tips of branches and other references (including HEAD) were updated in the local repository.

**Technical Definition**
The reflog is stored in `.git/logs/` and records every movement of HEAD and branch references. Each entry includes the old and new commit hashes, the author, timestamp, and a message describing the action. `git reflog show` is an alias for `git log -g --abbrev-commit --pretty=oneline`. The reflog is local-only and is not pushed to remotes.

**Beginner-Friendly Explanation**
Git keeps a diary of every time HEAD moves—every commit, checkout, reset, rebase, and merge. This diary is called the reflog. If you accidentally delete a branch or reset to an older commit, the reflog can help you find the commit you lost and recover it. It's Git's safety net.

### Purposes

- To track the movement of HEAD and branch references over time.
- To recover commits that appear to be “lost” after a reset or rebase.
- To find the hash of a previous state of a branch.
- To diagnose unexpected repository states.
- To provide a chronological audit of local reference updates.

### Syntax Rules and Structure

**Complete General Syntax**

```
git reflog [show] [<ref>] [<options>]
git reflog list
git reflog exists <ref>
git reflog delete <ref>@{<index>}
git reflog expire [--expire=<time>] [--all]
```

**Key Subcommands**

| Subcommand | Description |
|------------|-------------|
| `show` (default) | Show the reflog for a ref (default HEAD). |
| `list` | List all refs that have a reflog. |
| `exists` | Check if a ref has a reflog. |
| `expire` | Prune old reflog entries. |

**Reflog Entry Format**

```
<old-hash> <new-hash> <author> <timestamp> <message>
```

**Syntax Rules**

- `git reflog` is equivalent to `git reflog show HEAD`.
- `HEAD@{<n>}` refers to the position of HEAD `n` moves ago.
- `master@{one.week.ago}` refers to where `master` pointed one week ago.
- Reflog entries are local and not shared with remotes.
- Reflog entries expire after 90 days (default) for reachable entries and 30 days for unreachable entries.

**Constraints and Limitations**

- **Local only**: Reflogs are not cloned or pushed.
- **Expiration**: Reflog entries are eventually pruned by `git gc`.
- **Not a substitute for commits**: Reflog entries are not commits; they point to commits.
- **Detached HEAD**: The reflog records detached HEAD movements, which is crucial for recovery.

### Annotated Code Examples

#### Example 1: Viewing the Reflog

```bash
git reflog
```

**Expected Output:**
```
4d5e6f7 (HEAD -> main) HEAD@{0}: commit: Add feature
9a0b1c2 HEAD@{1}: commit: Fix bug
3d4e5f6 HEAD@{2}: checkout: moving from feature to main
1a2b3c4 HEAD@{3}: commit: Start feature
```

**Why this output occurs:** `git reflog` shows the last few movements of HEAD. `HEAD@{0}` is the most recent action (a commit), `HEAD@{1}` is the previous action, and so on.

#### Example 2: Recovering a Lost Commit After a Reset

```bash
# Make a commit
echo "Important" > important.txt
git add important.txt
git commit -m "Important commit"
# Output: [main abc1234] Important commit

# Reset to the previous commit (appears to lose the commit)
git reset --hard HEAD~1

# View the reflog
git reflog
```

**Expected Output:**
```
abc1234 (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
def5678 HEAD@{1}: commit: Important commit
...
```

**Why this output occurs:** The reflog records the reset (`HEAD@{0}`) and the commit (`HEAD@{1}`). The commit `def5678` is still in the object database. To recover it:

```bash
git cherry-pick def5678
# or
git reset --hard def5678
```

#### Example 3: Finding a Lost Branch

```bash
# Delete a branch
git branch -D feature

# Check reflog for the branch
git reflog show feature
# If it doesn't exist, check HEAD reflog for the last commit on that branch
git reflog
# Find the commit hash and recreate the branch
git branch feature <hash>
```

**Why this output occurs:** The reflog for a deleted branch may still exist temporarily. If not, the HEAD reflog records the last commit made on that branch. The hash can be used to recreate the branch.

### Real-World Cases

- **Accidental reset**: A developer runs `git reset --hard HEAD~3`, losing three commits. They use `git reflog` to find the lost commit hash and `git reset --hard <hash>` to recover.
- **Rebase recovery**: A rebase goes wrong; the original commits can be recovered from the reflog.
- **Detached HEAD recovery**: A developer makes commits in detached HEAD, switches away, and later recovers the commits from the reflog.

### References

- Git Documentation: git-reflog Manual Page — https://git-scm.com/docs/git-reflog
- Git Documentation: gitrevisions (Reflog Syntax) — https://git-scm.com/docs/gitrevisions#_the_reflog
- Git Documentation: Git Tools – Rewriting History (Recovering Lost Commits) — https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History

---

## References

- Git Documentation: git-status Manual Page — https://git-scm.com/docs/git-status
- Git Documentation: git-log Manual Page — https://git-scm.com/docs/git-log
- Git Documentation: git-show Manual Page — https://git-scm.com/docs/git-show
- Git Documentation: git-diff Manual Page — https://git-scm.com/docs/git-diff
- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: git-remote Manual Page — https://git-scm.com/docs/git-remote
- Git Documentation: git-config Manual Page — https://git-scm.com/docs/git-config
- Git Documentation: git-reflog Manual Page — https://git-scm.com/docs/git-reflog
- Git Documentation: gitrevisions Manual Page — https://git-scm.com/docs/gitrevisions
- Pro Git Book: Git Basics – Viewing the Commit History — https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History
- Pro Git Book: Git Basics – Working with Remotes — https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes
- Pro Git Book: Git Branching – Branch Management — https://git-scm.com/book/en/v2/Git-Branching-Branch-Management
- Pro Git Book: Git Tools – Reset Demystified — https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified
- Pro Git Book: Git Tools – Rewriting History — https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History