# Git Branch Commands: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Git branch commands are the set of command-line tools used to create, inspect, navigate, rename, and delete branches—the lightweight, movable pointers that enable parallel lines of development within a repository.

**Technical Definition**
Git branch commands operate on references stored in the `refs/heads/` namespace. The `git branch` command manages branch references (creation, deletion, renaming, listing, and tracking configuration). The `git switch` command (introduced in Git 2.23) navigates between branches, updating HEAD, the index, and the working tree. The legacy `git checkout` command performs both branch switching and file restoration, though its branch-switching functionality is now delegated to `git switch`. These commands collectively enable the branching workflows that are central to Git's distributed model.

**Beginner-Friendly Explanation**
Think of your Git repository as a book with multiple bookmarks. Each bookmark marks a different point in the story (a branch). Branch commands let you add new bookmarks, move between them, rename them, and remove them when you're done. The `git switch` command is like moving your reading position to a different bookmark; `git branch` is like managing the bookmarks themselves.

### Key Characteristics

- **Lightweight**: Branch operations are nearly instantaneous because branches are just pointers (41-byte files).
- **Modern vs. Legacy**: `git switch` and `git restore` (Git 2.23+) split the responsibilities of the overloaded `git checkout`.
- **Local by Default**: Branches exist locally; remote-tracking branches are separate references.
- **Non-Destructive**: Most branch operations (listing, creating, switching) do not modify commit history.
- **Tracking-Aware**: Branches can be configured to track remote branches for simplified push/pull.

### Prerequisites

- Git installed (version 2.23+ recommended for `git switch`; 2.28+ for `init.defaultBranch`).
- A Git repository with at least one commit (branches point to commits).
- Basic familiarity with the terminal and Git concepts (commits, HEAD, the working tree).

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial.
- **DevOps & CI/CD**: Branch-based deployment pipelines.
- **Software Development**: Feature branching, hotfix workflows, release management.
- **Collaborative Development**: Pull requests, code review, merge strategies.

### Core Concepts / Features

1. **git branch** — Creating, inspecting, and tracking metadata.
2. **git switch** — The modern, dedicated command for navigating branches.
3. **git checkout** — The legacy multi-purpose command.
4. **Creating Branches** — `git switch -c` and legacy `git checkout -b`.
5. **Renaming Branches** — `git branch -m` and syncing upstream targets.
6. **Deleting Branches** — `git branch -d` vs. `git branch -D`.
7. **Listing Branches** — `-a`, `-r`, and `--sort=-committerdate`.
8. **Orphan Branches** — `git switch --orphan` for decoupled histories.

---

## 1. git branch (Creating, Inspecting, and Tracking Metadata)

### Definitions

**Core Definition**
`git branch` is the command for managing branch references: creating, listing, renaming, deleting, and configuring tracking relationships.

**Technical Definition**
`git branch` reads and writes reference files in `.git/refs/heads/`. In list mode (default), it displays local branches. With `-v` / `-vv`, it shows the SHA-1 and commit subject (and tracking information for `-vv`). With `--merged` and `--no-merged`, it filters branches by merge status. It can also set or change the upstream tracking branch with `-u` / `--set-upstream-to`.

**Beginner-Friendly Explanation**
`git branch` is your branch manager. Run it without arguments to see all local branches. Use `-v` to see the last commit on each branch. Use `-u` to link a local branch to its remote counterpart. It's the command you use to create, inspect, and configure branches.

### Purposes

- To list all local branches and identify the current branch.
- To create a new branch at a specified commit.
- To rename or delete branches.
- To configure a branch to track a remote branch.
- To filter branches by merge status (`--merged`, `--no-merged`).

### Syntax Rules and Structure

**Complete General Syntax**

```
git branch [--list] [<pattern>...]
git branch [(-v | -vv | --verbose)] [--abbrev=<n>] [--list] [<pattern>...]
git branch [--merged [<commit>] | --no-merged [<commit>]] [--list] [<pattern>...]
git branch <branch-name> [<start-point>]
git branch (-d | -D) <branch-name>...
git branch (-m | -M) [<old-branch>] <new-branch>
git branch (-u | --set-upstream-to=<upstream>) [<branch>]
git branch --unset-upstream [<branch>]
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
| `-d` / `--delete` | Delete a merged branch. |
| `-D` | Force delete an unmerged branch. |
| `-m` / `--move` | Rename a branch. |
| `-M` | Force rename. |
| `-u` / `--set-upstream-to` | Set upstream tracking. |

**Syntax Rules**

- `git branch` without arguments lists local branches.
- `git branch <name>` creates a new branch pointing to the current HEAD (or specified start point).
- `git branch -v` shows the abbreviated commit hash and subject for each branch.
- `git branch -vv` adds upstream tracking information.
- `--merged` and `--no-merged` filter the list based on merge status.
- `-u <upstream>` sets the upstream branch for the current (or specified) branch.

**Constraints and Limitations**

- **Commit required**: Branches can only point to commits.
- **Name collision**: A branch cannot have the same name as an existing branch.
- **Version-specific**: `git branch --set-upstream-to` requires Git 1.8.0+.
- **Detached HEAD**: In detached HEAD state, `git branch` shows no current branch (no `*`).

### Annotated Code Examples

#### Example 1: Listing Branches with `-v` and `-vv`

```bash
# Create a repository with branches
git init branch-demo && cd branch-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
git branch feature
git branch bugfix

# List branches with -v
git branch -v
# Output:
#   bugfix  4d5e6f7 Commit A
#   feature 4d5e6f7 Commit A
# * master  4d5e6f7 Commit A

# List branches with -vv (shows tracking)
git branch -vv
# Output:
#   bugfix  4d5e6f7 Commit A
#   feature 4d5e6f7 Commit A
# * master  4d5e6f7 [origin/master] Commit A
```

**Expected Output:**
```
  bugfix  4d5e6f7 Commit A
  feature 4d5e6f7 Commit A
* master  4d5e6f7 Commit A
```

**Why this output occurs:** All branches point to the same commit (Commit A) because they were all created at the same point. The `*` indicates `master` is the current branch. With `-vv`, `master` shows its upstream tracking (`origin/master`).

#### Example 2: Setting Upstream Tracking

```bash
# Set upstream for a branch
git branch -u origin/feature feature

# Verify tracking
git branch -vv
# Output:
#   bugfix  4d5e6f7 Commit A
#   feature 4d5e6f7 [origin/feature] Commit A
# * master  4d5e6f7 [origin/master] Commit A
```

**Expected Output:**
```
  bugfix  4d5e6f7 Commit A
  feature 4d5e6f7 [origin/feature] Commit A
* master  4d5e6f7 [origin/master] Commit A
```

**Why this output occurs:** `git branch -u origin/feature feature` sets the upstream of `feature` to `origin/feature`. The `-vv` output now shows the tracking relationship in brackets.

### Real-World Cases

- **Feature development**: A developer creates a `feature/login` branch with `git branch feature/login` and tracks it with `-u`.
- **Release management**: A release manager uses `git branch --merged` to find branches that can be safely deleted.
- **Code review**: A reviewer uses `git branch -vv` to see which branches are ahead/behind their upstreams.

### References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: Git Branching – Branch Management — https://git-scm.com/book/en/v2/Git-Branching-Branch-Management
- Microsoft Learn: Branch Commands in Git — https://learn.microsoft.com/en-us/training/modules/work-source-control-git/

---

## 2. git switch (The Modern, Dedicated Command)

### Definitions

**Core Definition**
`git switch` is the modern command (introduced in Git 2.23) for switching branches, created to replace the branch-switching functionality of the overloaded `git checkout` command.

**Technical Definition**
`git switch` updates HEAD to point to a specified branch, and updates the index and working tree to match that branch's tip commit. It can also create new branches (`-c`), force-create/reset branches (`-C`), detach HEAD (`--detach`), and create orphan branches (`--orphan`). Unlike `git checkout`, `git switch` does not restore files; that functionality is handled by `git restore`.

**Beginner-Friendly Explanation**
`git switch` is like moving your bookmark to a different page in the book. It's simpler and safer than the old `git checkout` command because it only does one thing: switch branches. If you want to restore files, use `git restore` instead.

### Purposes

- To switch to an existing branch.
- To create and switch to a new branch in one step (`-c`).
- To detach HEAD for inspection (`--detach`).
- To create an orphan branch with no history (`--orphan`).
- To provide a clear, unambiguous alternative to `git checkout`.

### Syntax Rules and Structure

**Complete General Syntax**

```
git switch [<options>] [--no-guess] <branch>
git switch [<options>] --detach [<start-point>]
git switch [<options>] (-c|-C) <new-branch> [<start-point>]
git switch [<options>] --orphan <new-branch>
```

**Key Options**

| Option | Description |
|--------|-------------|
| `<branch>` | Branch to switch to. |
| `-c <new-branch>` / `--create` | Create and switch to a new branch. |
| `-C <new-branch>` / `--force-create` | Create/reset and switch to a branch. |
| `--detach` / `-d` | Switch to a commit (detached HEAD). |
| `--orphan <new-branch>` | Create a new orphan branch. |
| `-` | Switch to the previously checked-out branch. |
| `--discard-changes` | Discard local changes (dangerous). |
| `--merge` | Attempt a three-way merge with local changes. |

**Syntax Rules**

- `git switch` requires a clean working tree unless `--discard-changes` or `--merge` is used.
- `git switch -c <name>` combines creation and switching.
- `git switch -` returns to the previous branch (shortcut for `@{-1}`).
- `git switch --orphan <name>` creates a branch with no parent commit.
- `git switch` was introduced in Git 2.23.0 and is still marked as experimental.

**Constraints and Limitations**

- **Version-specific**: Requires Git 2.23+.
- **Uncommitted changes**: Switching is blocked if local changes would be overwritten.
- **Detached HEAD**: `--detach` enters detached HEAD state.
- **No file restoration**: Use `git restore` for restoring files.

### Annotated Code Examples

#### Example 1: Switching and Creating Branches

```bash
# Switch to an existing branch
git switch feature
# Output: Switched to branch 'feature'

# Create and switch to a new branch
git switch -c hotfix
# Output: Switched to a new branch 'hotfix'

# Switch back to the previous branch
git switch -
# Output: Switched to branch 'feature'
```

**Expected Output:**
```
Switched to branch 'feature'
Switched to a new branch 'hotfix'
Switched to branch 'feature'
```

**Why this output occurs:** `git switch feature` updates HEAD to point to `feature`. `git switch -c hotfix` creates a new branch at the current commit and switches to it. `git switch -` returns to the previously checked-out branch.

#### Example 2: Detaching HEAD

```bash
# Switch to a specific commit (detached HEAD)
git switch --detach HEAD~2
# Output: Note: switching to 'HEAD~2'.
#         You are in 'detached HEAD' state.
#         HEAD is now at 1a2b3c4 Commit A
```

**Expected Output:**
```
Note: switching to 'HEAD~2'.
You are in 'detached HEAD' state.
HEAD is now at 1a2b3c4 Commit A
```

**Why this output occurs:** `--detach HEAD~2` checks out the commit two steps back, entering detached HEAD state. New commits made in this state are not associated with any branch.

### Real-World Cases

- **Feature branching**: A developer runs `git switch -c feature/login` to start a new feature.
- **Code review**: A reviewer runs `git switch contributor/feature` to test a PR locally.
- **Detached HEAD inspection**: A developer runs `git switch --detach v1.0.0` to inspect an old release.

### References

- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch
- Git 2.23.0 Release Notes (switch and restore) — https://github.com/git/git/blob/master/Documentation/RelNotes/2.23.0.adoc
- Git-Tower: Git Checkout & Switch — https://www.git-tower.com/learn/git/faq/git-checkout-switch-branch/

---

## 3. git checkout (The Legacy Multi-Purpose Command)

### Definitions

**Core Definition**
`git checkout` is the legacy command that performs both branch switching and file restoration, now largely replaced by the more focused `git switch` (branch operations) and `git restore` (file operations).

**Technical Definition**
`git checkout <branch>` updates HEAD to point to the specified branch and updates the working tree and index to match. `git checkout -- <file>` restores a file from the index. `git checkout -b <new-branch>` creates and switches to a new branch. Because of its dual purpose, `git checkout` is prone to user confusion; Git 2.23 introduced `git switch` and `git restore` to split these responsibilities.

**Beginner-Friendly Explanation**
`git checkout` is the old Swiss Army knife of Git. It can do many things: switch branches, create branches, restore files, and more. But because it does so many things, it's easy to make mistakes. The new commands `git switch` and `git restore` are safer because each does only one thing.

### Purposes

- To switch branches (legacy).
- To create and switch to a new branch (legacy).
- To restore files from the index or a commit.
- To detach HEAD at a specific commit.
- To maintain compatibility with older Git versions and scripts.

### Syntax Rules and Structure

**Complete General Syntax**

```
git checkout <branch>
git checkout -b <new-branch> [<start-point>]
git checkout -B <new-branch> [<start-point>]
git checkout --orphan <new-branch>
git checkout [--detach] <commit>
git checkout [--] <pathspec>...
git checkout <tree-ish> [--] <pathspec>...
```

**Key Options**

| Option | Description |
|--------|-------------|
| `<branch>` | Switch to a branch. |
| `-b <new-branch>` | Create and switch to a new branch. |
| `-B <new-branch>` | Create/reset and switch. |
| `--orphan <new-branch>` | Create an orphan branch. |
| `--detach` | Detach HEAD at a commit. |
| `-- <pathspec>` | Restore files (use `git restore` instead). |
| `-f` / `--force` | Force checkout, discarding local changes. |

**Syntax Rules**

- `git checkout <branch>` switches branches.
- `git checkout -b <name>` creates and switches.
- `git checkout -- <file>` restores a file from the index.
- `git checkout <commit> -- <file>` restores a file from a specific commit.
- `git checkout` is still widely used and supported, but `git switch` and `git restore` are recommended for new users.

**Constraints and Limitations**

- **Overloaded command**: Easy to confuse branch switching with file restoration.
- **Deprecated in favour of switch/restore**: Though still supported, new workflows should use `git switch` and `git restore`.
- **Force checkout risk**: `-f` discards local changes.

### Annotated Code Examples

#### Example 1: Legacy Branch Switching vs. Modern

```bash
# Legacy: switch branch
git checkout feature

# Modern equivalent
git switch feature

# Legacy: create and switch
git checkout -b hotfix

# Modern equivalent
git switch -c hotfix
```

**Expected Output:**
```
Switched to branch 'feature'
Switched to a new branch 'hotfix'
```

**Why this output occurs:** Both command pairs are functionally equivalent. The modern commands are preferred for clarity.

### Real-World Cases

- **Legacy scripts**: Existing CI/CD scripts use `git checkout` and continue to work.
- **Compatibility**: Projects supporting older Git versions use `git checkout`.
- **File restoration**: `git checkout -- file.txt` is still used, though `git restore file.txt` is preferred.

### References

- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout
- Git 2.23.0 Release Notes — https://github.com/git/git/blob/master/Documentation/RelNotes/2.23.0.adoc
- Stack Overflow: git switch vs git checkout — https://stackoverflow.com/questions/57265785

---

## 4. Creating Branches (`git switch -c` and `git checkout -b`)

### Definitions

**Core Definition**
Creating a branch means creating a new branch pointer that points to a specified commit, enabling a new line of development from that point.

**Technical Definition**
When a new branch is created, Git creates a reference file in `.git/refs/heads/<new-branch>` containing the commit hash of the start point (default: HEAD). The branch is not checked out unless `-c` (switch) or `-b` (checkout) is used, which combine creation and switching. The new branch points to the exact same commit as HEAD at the moment of creation.

**Beginner-Friendly Explanation**
Creating a branch is like placing a new bookmark at your current page. You can do it without switching to the new branch (`git branch new-branch`) or switch immediately (`git switch -c new-branch`). The new branch starts at the same commit you're currently on.

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
git switch -C <new-branch> [<start-point>]
git checkout -b <new-branch> [<start-point>]   # Legacy
git checkout -B <new-branch> [<start-point>]   # Legacy
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<new-branch>` | Name of the new branch. |
| `<start-point>` | Optional commit, branch, or tag (default: HEAD). |
| `-c` / `--create` | Create and switch (git switch). |
| `-C` / `--force-create` | Create or reset and switch (git switch). |
| `-b` | Create and switch (legacy git checkout). |

**Syntax Rules**

- Without a `<start-point>`, the new branch points to the same commit as HEAD.
- With a `<start-point>`, the new branch points to that commit.
- `git branch <name>` creates the branch but does not switch to it.
- `git switch -c <name>` creates and switches in one step.
- `-C` (or `-B`) resets an existing branch to the start point.

**Constraints and Limitations**

- **Commit required**: The start point must resolve to a commit.
- **Name collision**: A branch cannot be created with an existing name (unless forced).
- **Unborn branch**: In a repository with no commits, `git branch` fails.

### Annotated Code Examples

#### Example 1: Creating a Branch Without Switching

```bash
# Create a branch (stays on current branch)
git branch feature

# Verify
git branch
# Output:
# * master
#   feature
```

**Expected Output:**
```
* master
  feature
```

**Why this output occurs:** `git branch feature` creates a new branch at the current HEAD but does not switch to it. The `*` remains on `master`.

#### Example 2: Creating and Switching in One Step

```bash
# Create and switch
git switch -c feature-2

# Verify
git branch
# Output:
# * feature-2
#   feature
#   master
```

**Expected Output:**
```
* feature-2
  feature
  master
```

**Why this output occurs:** `git switch -c feature-2` creates the branch and updates HEAD to point to it. The `*` now indicates `feature-2`.

### Real-World Cases

- **Feature branching**: A developer creates a `feature/user-auth` branch from `main`.
- **Hotfix**: A `hotfix/security-patch` branch is created from a release tag.
- **Release preparation**: A `release/v2.0` branch is created for stabilisation.

### References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch
- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout

---

## 5. Renaming Branches Locally and Remotely

### Definitions

**Core Definition**
Renaming a branch locally changes the branch's name in the local repository; renaming it remotely requires pushing the new name and deleting the old one from the remote, as Git does not support direct remote renaming.

**Technical Definition**
`git branch -m [<old-name>] <new-name>` renames a local branch by moving its reference file in `.git/refs/heads/`. Remote branches cannot be renamed directly; the workflow is: rename locally, push the new branch to the remote, delete the old branch from the remote, and reset the upstream tracking.

**Beginner-Friendly Explanation**
Renaming a branch locally is easy: `git branch -m new-name`. But renaming it on GitHub or GitLab requires a few extra steps: rename it locally, push the new name, then delete the old name from the remote. It's like changing your name: you update your local records, tell the remote server, and remove the old entry.

### Purposes

- To correct unclear or inconsistent branch names.
- To adopt a new naming convention (e.g., `feature/user-login`).
- To replace outdated terminology (e.g., `master` → `main`).
- To sync local branch names with remote conventions.
- To improve repository organisation.

### Syntax Rules and Structure

**Complete General Syntax**

```
# Rename local branch
git branch -m [<old-name>] <new-name>
git branch -M [<old-name>] <new-name>   # Force rename

# Push renamed branch to remote
git push origin <new-name>

# Delete old branch from remote
git push origin --delete <old-name>

# Set upstream tracking
git push --set-upstream origin <new-name>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `-m` / `--move` | Rename a branch. |
| `-M` | Force rename (overwrite existing). |
| `<old-name>` | Optional: the branch to rename (default: current). |
| `<new-name>` | The new branch name. |

**Syntax Rules**

- `git branch -m <new-name>` renames the current branch.
- `git branch -m <old-name> <new-name>` renames a specific branch.
- After renaming locally, push the new branch and delete the old remote branch.
- Update the upstream tracking with `git push --set-upstream`.
- If the renamed branch is the default branch, update the default branch setting on the remote hosting platform.

**Constraints and Limitations**

- **Remote renaming not supported**: Git does not support direct remote branch renaming.
- **Upstream tracking lost**: After renaming, the new branch does not automatically track the old remote branch.
- **Default branch**: Renaming the default branch requires updating the remote platform's settings.
- **Collaborators**: Other developers must update their local references.

### Annotated Code Examples

#### Example 1: Renaming a Local Branch

```bash
# Rename current branch
git branch -m new-name

# Rename a specific branch
git branch -m old-name new-name
```

**Expected Output:**
```
(no output; branch renamed)
```

**Why this output occurs:** `git branch -m` renames the branch by moving the reference file. No output is produced on success.

#### Example 2: Renaming a Remote Branch (Full Workflow)

```bash
# 1. Rename locally
git branch -m dev-branch main-dev

# 2. Push the renamed branch
git push origin main-dev

# 3. Delete the old branch from remote
git push origin --delete dev-branch

# 4. Reset upstream tracking
git push --set-upstream origin main-dev
```

**Expected Output:**
```
To https://github.com/user/repo.git
 * [new branch]      main-dev -> main-dev
To https://github.com/user/repo.git
 - [deleted]         dev-branch
Branch 'main-dev' set up to track remote branch 'main-dev' from 'origin'.
```

**Why this output occurs:** The local rename is followed by pushing the new branch, deleting the old remote branch, and setting up tracking. The remote now has `main-dev` instead of `dev-branch`.

### Real-World Cases

- **Default branch rename**: An organisation renames `master` to `main` locally, pushes, deletes the old remote branch, and updates the default branch setting on GitHub.
- **Convention change**: A team moves from `loginFeature` to `feature/user-login` and renames all branches accordingly.
- **Typo correction**: A developer corrects a misspelled branch name.

### References

- GitHub Community: How to Rename a Git Branch Both Locally and Remotely? — https://github.com/orgs/community/discussions/163286
- Codecademy: How to Rename a Branch in Git Locally and Remotely — https://www.codecademy.com/article/rename-git-branch
- Git-Tower: How to rename local and remote branches in Git — https://www.git-tower.com/learn/git/faq/rename-branch

---

## 6. Deleting Branches Safely

### Definitions

**Core Definition**
Deleting a branch removes the branch pointer, making its commits unreachable if they are not referenced by another branch or tag. `git branch -d` deletes only merged branches, while `git branch -D` forces deletion of unmerged branches.

**Technical Definition**
`git branch -d <branch>` deletes the branch only if it has been fully merged into its upstream branch (or HEAD). If the branch contains unmerged commits, the deletion is refused. `git branch -D <branch>` (equivalent to `--delete --force`) deletes the branch regardless of merge status, potentially losing unmerged commits.

**Beginner-Friendly Explanation**
Deleting a branch is like removing a bookmark. If you've already merged the branch's changes into another branch, it's safe to delete (`-d`). If you haven't merged them, Git will warn you and refuse—unless you force it with `-D`, which can lose work. Always use `-d` first; only use `-D` if you're certain.

### Purposes

- To clean up branches that have been merged.
- To remove experimental branches that are no longer needed.
- To reduce branch clutter in the repository.
- To free up branch names for reuse.
- To maintain a tidy branch listing.

### Syntax Rules and Structure

**Complete General Syntax**

```
git branch -d <branch>...
git branch -D <branch>...
git branch --delete <branch>...
git push origin --delete <branch>...   # Delete remote branch
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `-d` / `--delete` | Delete only if fully merged. |
| `-D` | Force delete (even if unmerged). |
| `<branch>` | The branch to delete. |
| `--delete` (with push) | Delete a remote-tracking branch. |

**Syntax Rules**

- `-d` checks merge status against the upstream or HEAD.
- `-D` skips the merge check.
- Multiple branches can be deleted in one command.
- Deleting a remote branch requires `git push origin --delete <branch>`.
- The current branch cannot be deleted; switch away first.

**Constraints and Limitations**

- **Unmerged commits**: `-d` refuses to delete unmerged branches.
- **Force deletion risk**: `-D` can permanently lose unmerged commits.
- **Recovery**: Deleted branch tips are recoverable via `git reflog` for 30 days.
- **Remote deletion**: Remote branch deletion is not automatically mirrored locally; use `git fetch --prune`.

### Annotated Code Examples

#### Example 1: Safe Deletion with `-d`

```bash
# Create and merge a branch
git switch -c feature
echo "Feature" > feature.txt && git add . && git commit -m "Feature"
git switch master
git merge feature

# Delete the merged branch
git branch -d feature
# Output: Deleted branch feature (was 4d5e6f7).
```

**Expected Output:**
```
Deleted branch feature (was 4d5e6f7).
```

**Why this output occurs:** `feature` has been merged into `master`, so `-d` deletes it safely. The commit remains reachable from `master`.

#### Example 2: Refused Deletion with `-d`

```bash
# Create an unmerged branch
git switch -c experiment
echo "Experiment" > exp.txt && git add . && git commit -m "Experiment"
git switch master

# Try to delete (fails)
git branch -d experiment
# Output: error: The branch 'experiment' is not fully merged.
#         If you are sure you want to delete it, run 'git branch -D experiment'.
```

**Expected Output:**
```
error: The branch 'experiment' is not fully merged.
If you are sure you want to delete it, run 'git branch -D experiment'.
```

**Why this output occurs:** `experiment` contains commits not merged into `master`. Git refuses to delete it to prevent data loss. The developer must use `-D` to force deletion or merge the branch first.

#### Example 3: Force Deletion with `-D`

```bash
git branch -D experiment
# Output: Deleted branch experiment (was 5d6e7f8).
```

**Expected Output:**
```
Deleted branch experiment (was 5d6e7f8).
```

**Why this output occurs:** `-D` forces deletion without checking merge status. The commit `5d6e7f8` becomes unreachable (recoverable via reflog).

### Real-World Cases

- **Post-merge cleanup**: A developer deletes a feature branch after merging it into `main`.
- **Abandoned experiment**: A developer force-deletes an experimental branch that is no longer needed.
- **Remote cleanup**: A CI script deletes remote branches after they are merged.

### References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: Git Branching – Branch Management — https://git-scm.com/book/en/v2/Git-Branching-Branch-Management

---

## 7. Listing Branches

### Definitions

**Core Definition**
Listing branches displays the branches in the repository, optionally including remote-tracking branches and sorted by various criteria such as recent commit date.

**Technical Definition**
`git branch` lists local branches by default. `-a` / `--all` lists both local and remote-tracking branches. `-r` / `--remotes` lists only remote-tracking branches. `--sort=<key>` sorts the output by the specified key (e.g., `committerdate`, `authordate`). The `branch.sort` configuration variable sets a default sort order.

**Beginner-Friendly Explanation**
Listing branches shows you all the bookmarks in your book. Use `git branch` for local branches, `git branch -a` for both local and remote, and `git branch --sort=-committerdate` to see the most recently used branches first.

### Purposes

- To see all available branches.
- To identify the current branch.
- To find branches that have been merged or not merged.
- To view remote-tracking branches.
- To sort branches by recent activity for quick navigation.

### Syntax Rules and Structure

**Complete General Syntax**

```
git branch [--list] [<pattern>...]
git branch -a [--list] [<pattern>...]
git branch -r [--list] [<pattern>...]
git branch --sort=<key> [--list] [<pattern>...]
git branch --merged [<commit>] [--list]
git branch --no-merged [<commit>] [--list]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `-a` / `--all` | List both local and remote-tracking branches. |
| `-r` / `--remotes` | List remote-tracking branches only. |
| `--sort=<key>` | Sort by the given key (e.g., `committerdate`). |
| `--merged [<commit>]` | List branches merged into `<commit>`. |
| `--no-merged [<commit>]` | List branches not merged into `<commit>`. |
| `-v` / `-vv` | Show commit hash and tracking info. |

**Syntax Rules**

- `git branch` lists local branches only.
- `-a` includes remote-tracking branches (prefixed with `remotes/origin/`).
- `-r` lists only remote-tracking branches.
- `--sort=-committerdate` sorts by most recent commit (newest first).
- `--sort=committerdate` sorts oldest first.
- `branch.sort` configuration sets a persistent default.

**Constraints and Limitations**

- **Remote branches are read-only**: They cannot be switched to directly.
- **Performance**: Listing thousands of branches can be slow.
- **Sorting requires Git 2.19+**: Earlier versions do not support `--sort`.

### Annotated Code Examples

#### Example 1: Listing All Branches

```bash
git branch -a
# Output:
#   feature
# * master
#   remotes/origin/HEAD -> origin/master
#   remotes/origin/master
#   remotes/origin/feature
```

**Expected Output:**
```
  feature
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/feature
```

**Why this output occurs:** `-a` lists local branches (`feature`, `master`) and remote-tracking branches (`remotes/origin/...`). The `*` indicates the current branch. `HEAD` shows the remote's default branch.

#### Example 2: Sorting by Recent Commit

```bash
git branch --sort=-committerdate
# Output:
#   feature   4d5e6f7 Commit A
#   hotfix    5d6e7f8 Commit B
# * master    1a2b3c4 Commit C
```

**Expected Output:**
```
  feature   4d5e6f7 Commit A
  hotfix    5d6e7f8 Commit B
* master    1a2b3c4 Commit C
```

**Why this output occurs:** `--sort=-committerdate` sorts branches by the date of their last commit, newest first. `master` has the oldest commit, so it appears last.

### Real-World Cases

- **Quick navigation**: A developer uses `git branch --sort=-committerdate` to see recently used branches first.
- **Cleanup**: A developer uses `git branch --merged` to find branches safe to delete.
- **Remote overview**: `git branch -r` shows all remote-tracking branches.

### References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- 30 Seconds of Code: Sort Branches by Date — https://raw.githubusercontent.com/mikeyhodl/30-seconds-of-code/0adb63a5c15d576d11bea58510c7d71017cf71a6/content/snippets/git/s/sort-branches-by-date.md
- Stack Overflow: Sort git branches by committerdate — https://stackoverflow.com/questions/5188320

---

## 8. Orphan Branches (`git switch --orphan`)

### Definitions

**Core Definition**
An orphan branch is a branch with no parent commit—its first commit has no ancestors, creating a completely independent history decoupled from all other branches in the repository.

**Technical Definition**
`git switch --orphan <new-branch>` (or legacy `git checkout --orphan <new-branch>`) creates a new branch whose first commit will have no parents. All tracked files are removed from the working tree and index. The new branch shares no history with the repository's other branches. This is useful for creating a `gh-pages` branch for GitHub Pages, a `docs` branch for documentation, or a repository with multiple independent projects.

**Beginner-Friendly Explanation**
An orphan branch is like starting a brand-new book that has nothing to do with the first book. It's a completely separate story. This is useful when you want to keep two unrelated sets of files in the same repository—like code and a website—without their histories being connected.

### Purposes

- To create a `gh-pages` branch for GitHub Pages.
- To create a documentation branch independent of source code.
- To start a new project in the same repository with no shared history.
- To create a branch with a clean slate for a fresh start.
- To host multiple unrelated projects in one repository.

### Syntax Rules and Structure

**Complete General Syntax**

```
git switch --orphan <new-branch>
git checkout --orphan <new-branch>   # Legacy
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--orphan` | Create a branch with no parent. |
| `<new-branch>` | Name of the new orphan branch. |

**Syntax Rules**

- The orphan branch starts empty (no tracked files).
- The first commit on the orphan branch has no parent.
- The orphan branch shares no history with other branches.
- Files from the previous branch are removed from the working tree.
- `git switch --orphan` requires Git 2.23+.

**Constraints and Limitations**

- **No shared history**: Merging an orphan branch with other branches is not possible without `--allow-unrelated-histories`.
- **Working tree cleared**: All tracked files are removed; untracked files remain.
- **Remote hosting**: Some platforms have specific requirements for `gh-pages` branches.

### Annotated Code Examples

#### Example 1: Creating a `gh-pages` Orphan Branch

```bash
# Create an orphan branch
git switch --orphan gh-pages
# Output: Switched to a new branch 'gh-pages'

# Create website files
echo "<h1>My Project</h1>" > index.html
git add index.html
git commit -m "Initial GitHub Pages commit"

# Push to remote
git push origin gh-pages
```

**Expected Output:**
```
Switched to a new branch 'gh-pages'
[gh-pages 5d6e7f8] Initial GitHub Pages commit
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```

**Why this output occurs:** `git switch --orphan gh-pages` creates a new branch with no parent commit. The first commit on this branch is the root of a completely independent history. GitHub Pages automatically serves content from this branch.

### Real-World Cases

- **GitHub Pages**: A project hosts its documentation website on a `gh-pages` orphan branch.
- **Documentation site**: A team maintains a `docs` orphan branch for a separate documentation website.
- **Multiple projects**: A monorepo hosts two independent projects in separate orphan branches.

### References

- Git Documentation: git-switch Manual Page (--orphan) — https://git-scm.com/docs/git-switch
- Git Documentation: git-checkout Manual Page (--orphan) — https://git-scm.com/docs/git-checkout
- GitHub Docs: Configuring a Publishing Source for GitHub Pages — https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site

---

## References

- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch
- Git Documentation: git-checkout Manual Page — https://git-scm.com/docs/git-checkout
- Git Documentation: Git Branching – Branch Management — https://git-scm.com/book/en/v2/Git-Branching-Branch-Management
- Git 2.23.0 Release Notes (switch and restore) — https://github.com/git/git/blob/master/Documentation/RelNotes/2.23.0.adoc
- Git-Tower: Git Checkout & Switch — https://www.git-tower.com/learn/git/faq/git-checkout-switch-branch/
- GitHub Community: How to Rename a Git Branch Both Locally and Remotely? — https://github.com/orgs/community/discussions/163286
- Codecademy: How to Rename a Branch in Git Locally and Remotely — https://www.codecademy.com/article/rename-git-branch
- Microsoft Learn: Branch Commands in Git — https://learn.microsoft.com/en-us/training/modules/work-source-control-git/
- 30 Seconds of Code: Sort Branches by Date — https://raw.githubusercontent.com/mikeyhodl/30-seconds-of-code/0adb63a5c15d576d11bea58510c7d71017cf71a6/content/snippets/git/s/sort-branches-by-date.md
- Stack Overflow: git switch vs git checkout — https://stackoverflow.com/questions/57265785
- Stack Overflow: Sort git branches by committerdate — https://stackoverflow.com/questions/5188320
- GitHub Docs: Configuring a Publishing Source for GitHub Pages — https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site