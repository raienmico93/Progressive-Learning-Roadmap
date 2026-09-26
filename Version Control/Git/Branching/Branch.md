# Git Branch Concepts: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A Git branch is a lightweight, movable pointer to a specific commit within the repository's Directed Acyclic Graph (DAG), enabling parallel lines of development that can be independently modified, merged, or abandoned.

**Technical Definition**
In Git, a branch is a reference—a named pointer stored as a plain-text file inside `.git/refs/heads/`—that contains the SHA-1 (or SHA-256) hash of the most recent commit on that line of development. The current branch is tracked by `HEAD`, a symbolic reference stored in `.git/HEAD` that points to one of these branch references. Branches are not copies of the repository; they are simply movable labels that advance automatically as new commits are created on them.

**Beginner-Friendly Explanation**
Think of a Git branch as a bookmark in a book. The book is your project's history, and the bookmark tells Git where you are currently reading (working). You can have multiple bookmarks, each pointing to a different place in the story. When you write a new chapter (make a commit), only the bookmark you're currently using moves forward. Creating a new branch is as simple as putting a new bookmark at the same page you're already on—it costs almost nothing.

### Key Characteristics

- **Lightweight**: A branch is just a 41-byte file (a 40-character SHA-1 hash plus a newline).
- **Movable**: The branch pointer advances automatically with each new commit made on that branch.
- **Mutable**: Branches can be renamed, deleted, or force-moved to different commits.
- **DAG-Based**: Branches reference commits within the Directed Acyclic Graph of the repository's history.
- **Local by Default**: Branches exist locally; remote-tracking branches are separate references that mirror remote state.

### Prerequisites

- Git installed (version 2.23+ recommended for `git switch`; 2.28+ for `init.defaultBranch`).
- A Git repository with at least one commit (branches point to commits, so a commit must exist).
- Basic familiarity with the terminal and Git concepts (commits, HEAD, the working tree).

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial.
- **DevOps & CI/CD**: Branch-based deployment pipelines.
- **Software Development**: Feature branching, hotfix workflows, release management.
- **Collaborative Development**: Pull requests, code review, merge strategies.

### Core Concepts / Features

1. **What a Branch Represents** — A lightweight, mutable pointer to a commit.
2. **Branch Pointers** — Plain-text files inside `.git/refs/heads/`.
3. **HEAD** — The active window pointer tracking the current branch or commit.
4. **Detached HEAD** — Causes, implications, and recovery.
5. **Branch Creation** — New branches point to the same commit as HEAD.
6. **Branch Switching** — Updating working tree, index, and HEAD simultaneously.

---

## 1. What a Branch Represents

### Definitions

**Core Definition**
A branch represents a line of development—a movable pointer to the most recent commit on that line, enabling independent work without affecting other lines.

**Technical Definition**
In Git, a branch is a reference (a named pointer) that lives in the `refs/heads/` namespace and points to a commit object. The most recent commit on a branch is called the **tip** (or **head**) of that branch. As new commits are created on the branch, the branch reference is automatically updated to point to the new commit. Because branches are simply pointers, creating a branch does not duplicate any data—it merely creates a new reference to an existing commit.

**Beginner-Friendly Explanation**
A branch is like a bookmark that moves forward as you read a book. If you want to explore a different plot direction (try a new feature), you place a new bookmark at your current page and start writing there. Your original bookmark stays where it was. You can switch between bookmarks at any time, and Git will update your working files to match the corresponding point in the story.

### Purposes

- To enable parallel development of features, bug fixes, and experiments.
- To isolate unstable work from stable code.
- To facilitate code review through feature branches and pull requests.
- To support release management with release branches and hotfix branches.
- To allow multiple developers to work simultaneously without interfering.

### Syntax Rules and Structure

**Branch Reference Storage**

```
.git/
├── refs/
│   ├── heads/           # Local branches
│   │   ├── main         # Contains SHA-1 of the latest commit on 'main'
│   │   ├── feature-x    # Contains SHA-1 of the latest commit on 'feature-x'
│   │   └── bugfix-123   # Contains SHA-1 of the latest commit on 'bugfix-123'
│   └── remotes/         # Remote-tracking branches
│       └── origin/
│           ├── main
│           └── feature-x
└── HEAD                 # Symbolic reference to the current branch
```

**Branch Reference File Contents**

```
$ cat .git/refs/heads/main
750b4ead9c87ceb3ddb7a390e6c7074521797fb3
```

**Syntax Rules**

- A branch name must be a valid Git reference name: alphanumeric, hyphens, underscores, slashes, and dots are allowed; spaces, `~`, `^`, `:`, `?`, `*`, `[`, and `\` are not.
- Branch names are case-sensitive on case-sensitive filesystems.
- The convention is that the default branch is called `main` (modern) or `master` (historical).
- A branch always points to a commit, never to a tree or blob.

**Constraints and Limitations**

- **Commit required**: Branches can only point to commits; a repository with no commits has no branches.
- **No empty branches**: Git does not support branches without at least one commit (unborn branches exist only as HEAD references).
- **Namespace collision**: A branch cannot have the same name as an existing tag or remote-tracking branch in certain contexts.

### Annotated Code Examples

#### Example 1: Inspecting Branch References

```bash
# Create a repository and make a commit
mkdir branch-demo && cd branch-demo
git init
echo "Hello" > file.txt
git add file.txt
git commit -m "Initial commit"

# Inspect the branch reference file
cat .git/refs/heads/master
# Output: 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e

# Verify with git rev-parse
git rev-parse HEAD
# Output: 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
```

**Expected Output:**
```
4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
```

**Why this output occurs:** The file `.git/refs/heads/master` contains the SHA-1 hash of the commit that `master` points to. `git rev-parse HEAD` follows the chain: `HEAD` → `refs/heads/master` → commit hash. Both commands return the same hash because `HEAD` is a symbolic reference to `master`, which points to the commit.

#### Example 2: Branch Advances with New Commits

```bash
# Make a second commit
echo "World" >> file.txt
git commit -am "Second commit"

# The branch file now contains the new commit hash
cat .git/refs/heads/master
# Output: 5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e
```

**Expected Output:**
```
5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e
```

**Why this output occurs:** When a new commit is made on `master`, Git automatically updates the file `.git/refs/heads/master` to contain the new commit's hash. The branch pointer has moved forward.

### Real-World Cases

- **Feature development**: A developer creates a `feature/login` branch to work on authentication without affecting `main`.
- **Hotfix workflow**: A `hotfix/critical-bug` branch is created from a production tag, fixed, and merged back into both `main` and the release branch.
- **Release management**: A `release/v2.0` branch is created for stabilisation while `main` continues to receive new features.

### References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: Git Internals – Git References — https://git-scm.com/book/en/v2/Git-Internals-Git-References
- Git Glossary: Branch — https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefbranchabranch

---

## 2. Branch Pointers (How Git Stores Branches)

### Definitions

**Core Definition**
Branch pointers are the plain-text files inside `.git/refs/heads/` that contain the SHA-1 hash of the commit each branch currently points to.

**Technical Definition**
Git stores each local branch as a file in the `.git/refs/heads/` directory. The filename is the branch name, and the file content is the 40-character hexadecimal SHA-1 hash of the commit at the tip of that branch, followed by a newline. When a new commit is created on a branch, Git simply overwrites this file with the new commit's hash. This is why branches are described as "lightweight"—they are nothing more than references stored as plain text.

**Beginner-Friendly Explanation**
Imagine each branch is a sticky note attached to a specific page in your project's history book. The sticky note has a name (like "main" or "feature") and a number written on it (the commit hash). The sticky notes are kept in a drawer called `.git/refs/heads/`. When you write a new page (make a commit), you move the sticky note to the new page. That's all a branch is.

### Purposes

- To provide an efficient, human-readable storage mechanism for branch references.
- To enable instant branch creation (just create a small file).
- To allow direct inspection and manipulation of branch pointers.
- To support Git's distributed model where branches are local references.
- To enable atomic branch updates through file replacement.

### Syntax Rules and Structure

**Complete General Syntax (Directory Layout)**

```
.git/refs/heads/
├── main              # Points to commit A
├── feature           # Points to commit B
└── bugfix            # Points to commit C
```

**File Format**

```
<40-character-hexadecimal-SHA-1>\n
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `.git/refs/heads/` | Directory containing local branch references. |
| `<branch-name>` | Filename equal to the branch name. |
| `<40-char-hex>` | The commit hash the branch points to. |
| `\n` | Trailing newline (optional but conventional). |

**Syntax Rules**

- The file content is the raw commit hash, not a symbolic reference.
- The filename is the branch name, with slashes creating subdirectories (e.g., `feature/login` → `.git/refs/heads/feature/login`).
- A branch reference file is updated atomically by Git when commits are made.
- Remote-tracking branches are stored in `.git/refs/remotes/` with a different namespace.

**Constraints and Limitations**

- **Do not edit manually**: While you can edit branch files manually, it is error-prone and generally discouraged. Use `git branch`, `git update-ref`, or `git switch` instead.
- **Packed refs**: In repositories with many branches, Git may pack references into `.git/packed-refs` for efficiency. The individual files may be absent but the references still exist.
- **Case sensitivity**: On case-insensitive filesystems, branch names that differ only in case may conflict.

### Annotated Code Examples

#### Example 1: Creating a Branch Manually (Demonstration)

```bash
# Create a repository and commit
git init manual-branch && cd manual-branch
echo "content" > file.txt
git add file.txt && git commit -m "Commit"

# Get the current commit hash
HASH=$(git rev-parse HEAD)

# Manually create a branch reference file
echo "$HASH" > .git/refs/heads/my-manual-branch

# Verify the branch exists
git branch
# Output:
# * master
#   my-manual-branch
```

**Expected Output:**
```
* master
  my-manual-branch
```

**Why this output occurs:** Writing the commit hash to a file in `.git/refs/heads/` creates a new branch that points to that commit. Git recognises the file as a branch reference. This demonstrates the underlying mechanism, though in practice you should use `git branch <name>`.

#### Example 2: Inspecting Branch Pointers

```bash
# Create two branches
git branch feature
git branch bugfix

# Inspect all branch reference files
for branch in .git/refs/heads/*; do
  echo "$(basename $branch): $(cat $branch)"
done
```

**Expected Output:**
```
bugfix: 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
feature: 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
master: 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
```

**Why this output occurs:** All three branches point to the same commit because they were all created at the same point in history. Each branch is a separate file containing the same commit hash.

### Real-World Cases

- **Debugging branch state**: A developer inspects `.git/refs/heads/` to understand which commits branches point to when `git branch -v` output is unclear.
- **Repository recovery**: In a corrupted repository, manually restoring a branch reference file from a backup can recover a lost branch.
- **Understanding `packed-refs`**: In a large repository, `cat .git/packed-refs` reveals all packed branch references in a single file.

### References

- Git Documentation: Git Internals – Git References — https://git-scm.com/book/en/v2/Git-Internals-Git-References
- Git Documentation: git-update-ref Manual Page — https://git-scm.com/docs/git-update-ref
- Git Documentation: gitrepository-layout — https://git-scm.com/docs/gitrepository-layout

---

## 3. HEAD (The Active Window Pointer)

### Definitions

**Core Definition**
HEAD is a symbolic reference that points to the currently checked-out branch (or directly to a commit in detached HEAD state), acting as the "active window" that determines where new commits are recorded and which files are checked out in the working tree.

**Technical Definition**
`HEAD` is a file at `.git/HEAD` that contains either a symbolic reference to a branch (e.g., `ref: refs/heads/main`) or a raw commit hash (in detached HEAD state). When Git performs operations like `commit`, `merge`, or `rebase`, it uses HEAD to determine the current context. In normal operation, HEAD points to a branch, which in turn points to a commit. The chain is: `HEAD` → `refs/heads/<branch>` → `<commit-hash>`.

**Beginner-Friendly Explanation**
HEAD is like the "You are here" marker on a map. It tells Git where you are currently working. Normally, it points to a branch (like `main`), and that branch points to a commit. When you make a new commit, Git moves the branch pointer forward, and HEAD follows because it's still pointing to the same branch. HEAD is your current position in the project's history.

### Purposes

- To identify the current branch or commit for all Git operations.
- To determine where new commits are recorded.
- To track the current position in the repository's history.
- To enable switching between branches (by changing where HEAD points).
- To serve as the reference for relative commit notation (e.g., `HEAD~1`, `HEAD^`).

### Syntax Rules and Structure

**Complete General Syntax (File Contents)**

```
# Normal (attached) HEAD:
ref: refs/heads/<branch-name>

# Detached HEAD:
<40-character-hexadecimal-SHA-1>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ref:` | Indicates a symbolic reference. |
| `refs/heads/<branch>` | The branch that HEAD points to. |
| `<commit-hash>` | Direct commit reference (detached HEAD only). |

**Syntax Rules**

- `HEAD` is always uppercase.
- In attached state, HEAD points to a branch reference, not directly to a commit.
- In detached state, HEAD contains a raw commit hash.
- `HEAD` is updated by `git switch`, `git checkout`, `git commit`, `git reset`, and other commands.
- `ORIG_HEAD` is set by some operations (merge, reset, rebase) to allow recovery.

**Constraints and Limitations**

- **Single HEAD**: A repository has only one HEAD (per worktree).
- **Detached HEAD risks**: Commits made in detached HEAD are not associated with any branch and can be lost.
- **Worktrees**: Linked worktrees have their own HEAD files in their respective `.git` directories.

### Annotated Code Examples

#### Example 1: Inspecting HEAD in Normal State

```bash
# Create a repository and commit
git init head-demo && cd head-demo
echo "content" > file.txt
git add file.txt && git commit -m "Commit"

# Inspect HEAD
cat .git/HEAD
# Output: ref: refs/heads/master

# Resolve HEAD to a commit
git rev-parse HEAD
# Output: 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
```

**Expected Output:**
```
ref: refs/heads/master
4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
```

**Why this output occurs:** `HEAD` contains `ref: refs/heads/master`, indicating it points to the `master` branch. `git rev-parse HEAD` follows the chain and resolves to the commit hash.

#### Example 2: HEAD Updates When Switching Branches

```bash
# Create and switch to a new branch
git switch -c feature

# Inspect HEAD
cat .git/HEAD
# Output: ref: refs/heads/feature
```

**Expected Output:**
```
ref: refs/heads/feature
```

**Why this output occurs:** `git switch -c feature` creates a new branch and updates HEAD to point to it. The HEAD file now contains `ref: refs/heads/feature`.

### Real-World Cases

- **Understanding repository state**: Running `git status` shows the branch HEAD points to (e.g., "On branch main").
- **Relative commit references**: `HEAD~3` refers to the commit three generations before the current HEAD.
- **Recovery with ORIG_HEAD**: After a reset, `ORIG_HEAD` points to the previous HEAD, enabling `git reset --hard ORIG_HEAD` to undo.

### References

- Git Documentation: Git Internals – Git References (The HEAD) — https://git-scm.com/book/en/v2/Git-Internals-Git-References
- Git Documentation: gitglossary (HEAD) — https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefHEADaHEAD
- Git Documentation: gitrevisions (HEAD) — https://git-scm.com/docs/gitrevisions

---

## 4. The "Detached HEAD" State

### Definitions

**Core Definition**
Detached HEAD is a state where HEAD points directly to a specific commit instead of a branch, meaning new commits made in this state are not associated with any branch and can be lost if you switch away.

**Technical Definition**
When HEAD points to a commit hash rather than a branch reference, the repository is in "detached HEAD" state. This occurs when you check out a commit by its hash, check out a tag, check out a remote branch without creating a local tracking branch, or during operations like rebase. Commits made in this state are "orphaned"—no branch reference points to them. They remain in the object database and are recoverable via `git reflog` for a default of 30 days before garbage collection.

**Beginner-Friendly Explanation**
Normally, HEAD points to a branch (like a bookmark in a book). Detached HEAD is like taking the bookmark out and pointing directly at a word on a page. You can still read and even write on that page, but if you move to another page, your writing might be lost because no bookmark is holding your place. Detached HEAD is useful for looking around at old code, but you need to be careful if you make changes you want to keep.

### Purposes

- To inspect historical code at a specific commit.
- To test a specific version or tag without modifying a branch.
- To perform `git bisect` for bug hunting.
- To review pull request commits without merging.
- To support internal operations like `git rebase` (which temporarily detaches HEAD).

### Syntax Rules and Structure

**Entering Detached HEAD**

| Command | Description |
|---------|-------------|
| `git checkout <commit-hash>` | Check out a specific commit. |
| `git checkout <tag>` | Check out a tag (tags point to commits). |
| `git checkout origin/main` | Check out a remote-tracking branch directly. |
| `git switch --detach <commit>` | Modern command to detach HEAD. |
| `git rebase -i HEAD~3` | Internal rebase operation. |

**Exiting Detached HEAD**

| Command | Description |
|---------|-------------|
| `git switch <branch>` | Return to a branch (discard detached commits). |
| `git switch -c <new-branch>` | Create a new branch from the detached commit and switch to it. |
| `git switch -` | Return to the previously checked-out branch. |

**Syntax Rules**

- Detached HEAD is indicated in `git status` by "HEAD detached at <commit>".
- Commits made in detached HEAD are not reachable from any branch.
- `git reflog` records detached HEAD movements for recovery.
- `git switch -c <name>` captures the current detached commit by creating a branch at that point.

**Constraints and Limitations**

- **Orphaned commits**: Commits made in detached HEAD become unreachable when you switch away.
- **Recovery window**: Orphaned commits are kept in the reflog for 30 days (default) before garbage collection.
- **No branch association**: Pull requests and other branch-based workflows cannot reference detached commits.
- **Accidental detachment**: Checking out a remote branch directly (e.g., `git checkout origin/main`) detaches HEAD instead of creating a local branch.

### Annotated Code Examples

#### Example 1: Entering Detached HEAD

```bash
# Create commits
git init detached-demo && cd detached-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
echo "B" >> file.txt && git commit -am "Commit B"
echo "C" >> file.txt && git commit -am "Commit C"

# Check out the first commit (detached HEAD)
git checkout HEAD~2
```

**Expected Output:**
```
Note: switching to 'HEAD~2'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

HEAD is now at 1a2b3c4 Commit A
```

**Why this output occurs:** `HEAD~2` refers to Commit A (two steps back from HEAD). Because a specific commit hash is checked out instead of a branch name, Git enters detached HEAD state. The HEAD file now contains the commit hash directly.

#### Example 2: Making a Commit in Detached HEAD

```bash
# Make a commit in detached HEAD
echo "Experimental" >> file.txt
git commit -am "Experimental commit"
# Output: [detached HEAD 5d6e7f8] Experimental commit

# Switch back to master
git switch master
# Output: Warning: you are leaving 1 commit behind, not connected to
# any of your branches:
#   5d6e7f8 Experimental commit
```

**Expected Output:**
```
[detached HEAD 5d6e7f8] Experimental commit
Warning: you are leaving 1 commit behind, not connected to
any of your branches:
  5d6e7f8 Experimental commit
```

**Why this output occurs:** The commit `5d6e7f8` is created, but no branch points to it. When switching back to `master`, Git warns that the commit is now orphaned. It remains recoverable via `git reflog`.

#### Example 3: Safely Capturing Detached HEAD Changes

```bash
# While in detached HEAD, create a new branch
git switch -c experimental
# Output: Switched to a new branch 'experimental'

# The commit is now on a branch and safe
git log --oneline
# Output: 5d6e7f8 (HEAD -> experimental) Experimental commit
```

**Expected Output:**
```
Switched to a new branch 'experimental'
5d6e7f8 (HEAD -> experimental) Experimental commit
```

**Why this output occurs:** `git switch -c experimental` creates a new branch at the current (detached) commit and attaches HEAD to it. The commit is now associated with a branch and cannot be orphaned.

#### Example 4: Recovering Orphaned Commits with Reflog

```bash
# After losing a commit by switching away
git reflog
# Output:
# 5d6e7f8 HEAD@{0}: checkout: moving from 5d6e7f8 to master
# 5d6e7f8 HEAD@{1}: commit: Experimental commit

# Create a branch at the orphaned commit
git branch recovery 5d6e7f8
```

**Expected Output:**
```
5d6e7f8 HEAD@{0}: checkout: moving from 5d6e7f8 to master
5d6e7f8 HEAD@{1}: commit: Experimental commit
```

**Why this output occurs:** The reflog records every HEAD movement, including the commit made in detached HEAD. The commit hash `5d6e7f8` can be used to create a new branch (`recovery`), permanently saving the orphaned commit.

### Real-World Cases

- **Inspecting old code**: A developer checks out a tag to reproduce a bug reported in an old release, entering detached HEAD temporarily.
- **CI/CD builds**: CI systems check out specific commits (detached HEAD) to build and test exact revisions.
- **Bisecting**: `git bisect` uses detached HEAD to test commits between known-good and known-bad states.
- **Rebase**: Git internally detaches HEAD during rebase to replay commits onto a new base.
- **Accidental detachment**: A developer runs `git checkout origin/main` instead of `git switch main`, inadvertently entering detached HEAD.

### References

- Git Documentation: git-checkout (DETACHED HEAD) — https://git-scm.com/docs/git-checkout
- Git Documentation: git-switch (--detach) — https://git-scm.com/docs/git-switch
- CircleCI: Recovering from the Git detached HEAD state — https://circleci.com/blog/git-detached-head-state/
- OneUptime: How to Fix 'Detached HEAD' State in Git — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-24-git-detached-head-state/README.md
- GeeksforGeeks: How to Fix Detached Head in GIT? — https://origin.geeksforgeeks.org

---

## 5. Branch Creation

### Definitions

**Core Definition**
Branch creation is the act of creating a new branch pointer that points to a specified commit, enabling a new line of development to begin from that point.

**Technical Definition**
When a new branch is created without specifying a starting point, it points to the exact same commit as HEAD at the moment of creation. Git achieves this by creating a new reference file in `.git/refs/heads/<new-branch>` containing the commit hash of the current HEAD. Optionally, a different start point can be specified, allowing the branch to begin at any commit in history. The branch is not switched to unless `-c` or `--create` is used with `git switch`.

**Beginner-Friendly Explanation**
Creating a branch is like placing a new bookmark at your current page before you start writing a new chapter. Both bookmarks (the old branch and the new one) point to the same page. When you start writing in the new chapter, only the new bookmark moves forward. Creating a branch costs almost nothing because it's just a new label pointing to an existing commit.

### Purposes

- To start a new line of development from the current state.
- To isolate experimental work from stable code.
- To create a branch from a historical point for hotfixes or releases.
- To enable parallel development by multiple developers.
- To prepare for a pull request or code review.

### Syntax Rules and Structure

**Complete General Syntax**

```
git branch <new-branch> [<start-point>]
git switch -c <new-branch> [<start-point>]
git switch -C <new-branch> [<start-point>]   # Force create/reset
git checkout -b <new-branch> [<start-point>]  # Legacy
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<new-branch>` | Name of the new branch. |
| `<start-point>` | Optional commit, branch, or tag to start from (default: HEAD). |
| `-c` / `--create` | Create and switch to the new branch (git switch). |
| `-C` / `--force-create` | Create or reset the branch and switch (git switch). |
| `-b` | Create and switch (legacy git checkout). |

**Syntax Rules**

- Without a `<start-point>`, the new branch points to the same commit as HEAD.
- With a `<start-point>`, the new branch points to that commit (or the commit that the ref resolves to).
- `git branch <name>` creates the branch but does not switch to it.
- `git switch -c <name>` creates the branch and switches to it in one step.
- `git switch -C <name>` resets an existing branch to the start point and switches.

**Constraints and Limitations**

- **Commit required**: The start point must resolve to a commit.
- **Name collision**: A branch cannot be created with a name that already exists (unless forced with `-C` or `-f`).
- **Unborn branch**: In a repository with no commits, `git branch` fails because there is no commit to point to.
- **Worktree restrictions**: A branch cannot be checked out in multiple worktrees simultaneously.

### Annotated Code Examples

#### Example 1: Creating a Branch from HEAD

```bash
# Create a repository and commit
git init create-demo && cd create-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
echo "B" >> file.txt && git commit -am "Commit B"

# Create a new branch (does not switch)
git branch feature

# Verify both branches point to the same commit
git branch -v
# Output:
#   feature 4d5e6f7 Commit B
# * master  4d5e6f7 Commit B
```

**Expected Output:**
```
  feature 4d5e6f7 Commit B
* master  4d5e6f7 Commit B
```

**Why this output occurs:** `git branch feature` creates a new branch at the current HEAD (Commit B). Both `master` and `feature` point to the same commit. The `*` indicates the current branch is still `master`.

#### Example 2: Creating and Switching in One Step

```bash
# Create and switch to a new branch
git switch -c feature-2

# Verify HEAD points to the new branch
git branch -v
# Output:
# * feature-2 4d5e6f7 Commit B
#   feature   4d5e6f7 Commit B
#   master    4d5e6f7 Commit B
```

**Expected Output:**
```
* feature-2 4d5e6f7 Commit B
  feature   4d5e6f7 Commit B
  master    4d5e6f7 Commit B
```

**Why this output occurs:** `git switch -c feature-2` creates the branch and updates HEAD to point to it. The `*` now indicates `feature-2` is the current branch.

#### Example 3: Creating a Branch from a Specific Commit

```bash
# Create a branch starting at an earlier commit
git switch -c hotfix HEAD~1

# Verify the branch points to Commit A
git log --oneline -1 hotfix
# Output: 1a2b3c4 Commit A
```

**Expected Output:**
```
1a2b3c4 Commit A
```

**Why this output occurs:** `HEAD~1` refers to Commit A. The new branch `hotfix` starts at that commit, not at the current HEAD. This is useful for creating hotfix branches from a known-good release point.

### Real-World Cases

- **Feature branching**: A developer creates a `feature/user-auth` branch from `main` to implement authentication.
- **Hotfix**: A `hotfix/security-patch` branch is created from a release tag to fix a critical vulnerability.
- **Release preparation**: A `release/v2.0` branch is created from `develop` for stabilisation.
- **Experiment**: A developer creates a `experiment/new-algorithm` branch to try a risky approach without affecting `main`.

### References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch
- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout
- Microsoft Learn: Branch Commands in Git — https://learn.microsoft.com/en-us/training/modules/work-source-control-git/

---

## 6. Branch Switching

### Definitions

**Core Definition**
Branch switching is the act of changing the current branch by updating HEAD to point to a different branch and synchronizing the working tree and index to match that branch's tip commit.

**Technical Definition**
When switching branches, Git performs three simultaneous operations: (1) updates HEAD to point to the target branch reference, (2) updates the index (staging area) to match the target branch's tree, and (3) updates the working tree files to match the target branch's tree. Git refuses to switch if doing so would overwrite uncommitted changes, unless `--discard-changes` or `--merge` is specified. The modern command is `git switch` (Git 2.23+); the legacy command is `git checkout`.

**Beginner-Friendly Explanation**
Switching branches is like moving your bookmark to a different page in the book. Git replaces all your project files with the versions from the branch you're switching to. If you have unsaved changes that would be overwritten, Git warns you and asks you to commit or discard them first. It's like Git saying: "I can't move your bookmark because you have loose papers on this page—please tidy up first."

### Purposes

- To move between different lines of development.
- To work on a feature branch and then return to the main branch.
- To test changes in one branch without affecting another.
- To prepare for merging or rebasing.
- To enable parallel workflows (e.g., hotfix while feature development continues).

### Syntax Rules and Structure

**Complete General Syntax**

```
git switch [<options>] <branch>
git switch [<options>] --detach [<start-point>]
git switch [<options>] (-c|-C) <new-branch> [<start-point>]
git checkout <branch>   # Legacy
```

**Key Options**

| Option | Description |
|--------|-------------|
| `<branch>` | The branch to switch to. |
| `-c <new-branch>` | Create and switch to a new branch. |
| `-C <new-branch>` | Create/reset and switch. |
| `--detach` | Switch to a commit (detached HEAD). |
| `--discard-changes` | Discard local changes (dangerous). |
| `--merge` | Attempt a three-way merge with local changes. |
| `-` | Switch to the previously checked-out branch. |
| `@{-N}` | Switch to the N-th last branch. |

**Syntax Rules**

- `git switch` requires a clean working tree (no changes that would be overwritten).
- `git switch -` returns to the previous branch (shortcut for `@{-1}`).
- `git switch --detach <commit>` enters detached HEAD.
- `git switch -c <name>` combines creation and switching.
- Uncommitted changes are preserved if they do not conflict with the target branch.

**Constraints and Limitations**

- **Uncommitted changes**: Switching is blocked if local changes would be overwritten.
- **Untracked files**: Untracked files are not affected by switching and remain in the working tree.
- **Merge conflicts**: Cannot switch branches during a merge conflict.
- **Worktrees**: A branch cannot be switched to in a worktree if it is already checked out in another worktree.
- **Version-specific**: `git switch` requires Git 2.23+; use `git checkout` for older versions.

### Annotated Code Examples

#### Example 1: Basic Branch Switching

```bash
# Create a repository with two branches
git init switch-demo && cd switch-demo
echo "Main content" > file.txt && git add . && git commit -m "Main commit"
git switch -c feature
echo "Feature content" > feature.txt && git add . && git commit -m "Feature commit"

# Switch back to main
git switch master

# The feature file is gone from the working tree
ls
# Output: file.txt
```

**Expected Output:**
```
file.txt
```

**Why this output occurs:** `git switch master` updates HEAD to point to `master`, resets the index to match `master`'s tree, and overwrites the working tree. `feature.txt` is removed from the working tree because it doesn't exist in `master`. `file.txt` remains because it exists in both branches.

#### Example 2: Switching with Uncommitted Changes (Blocked)

```bash
# Modify a file on master without committing
echo "Uncommitted" >> file.txt

# Try to switch to feature
git switch feature
# Output: error: Your local changes to the following files would be overwritten by checkout:
#         file.txt
#       Please commit your changes or stash them before you switch branches.
```

**Expected Output:**
```
error: Your local changes to the following files would be overwritten by checkout:
	file.txt
Please commit your changes or stash them before you switch branches.
```

**Why this output occurs:** The uncommitted change to `file.txt` would be overwritten by switching to `feature` (where `file.txt` has different content). Git refuses the switch to prevent data loss. The developer must commit, stash, or discard the change first.

#### Example 3: Switching Back to the Previous Branch

```bash
# Switch to feature, then back to previous
git switch feature
git switch -
# Output: Switched to branch 'master'
```

**Expected Output:**
```
Switched to branch 'master'
```

**Why this output occurs:** `git switch -` is a shortcut for `@{-1}`, which refers to the previously checked-out branch. This is useful for quickly toggling between two branches.

### Real-World Cases

- **Feature development**: A developer switches from `main` to `feature/login` to work on authentication.
- **Hotfix**: A developer switches from `feature` to `hotfix/critical` to address an urgent bug, then switches back.
- **Code review**: A reviewer switches to a contributor's branch to test the code locally.
- **Release management**: A release manager switches between `release/v2.0` and `main` to cherry-pick commits.

### References

- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch
- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout
- Git Documentation: Git Branching – Switching Branches — https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell

---

## References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch
- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout
- Git Documentation: Git Internals – Git References — https://git-scm.com/book/en/v2/Git-Internals-Git-References
- Git Documentation: gitglossary — https://git-scm.com/docs/gitglossary
- Git Documentation: gitrevisions Manual Page — https://git-scm.com/docs/gitrevisions
- Git Documentation: gitrepository-layout — https://git-scm.com/docs/gitrepository-layout
- CircleCI: Recovering from the Git detached HEAD state — https://circleci.com/blog/git-detached-head-state/
- OneUptime: How to Fix 'Detached HEAD' State in Git — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-24-git-detached-head-state/README.md
- GeeksforGeeks: How to Fix Detached Head in GIT? — https://origin.geeksforgeeks.org
- Microsoft Learn: Branch Commands in Git — https://learn.microsoft.com/en-us/training/modules/work-source-control-git/
- Git SCM: Git Branching – Branches in a Nutshell — https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell