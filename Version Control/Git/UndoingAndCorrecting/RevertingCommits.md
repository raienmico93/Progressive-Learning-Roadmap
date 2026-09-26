# Git Reverting Commits: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`git revert` is a command that creates a new commit which applies the inverse of the changes introduced by one or more specified commits, effectively undoing their effects while preserving the complete project history.

**Technical Definition**
`git revert` computes the reverse patch of each specified commit and applies it to the current working tree and index, creating one new commit per reverted commit (unless `--no-commit` is used). The original commits remain in the DAG; the new revert commits are appended as children of the current HEAD. This is a forward-moving operation: the branch pointer advances, and the history is never rewritten. For merge commits, the `-m` / `--mainline` flag is required to specify which parent represents the mainline to revert relative to. Conflicts may arise if subsequent commits have modified the same lines; these must be resolved manually before continuing with `git revert --continue` or cancelled with `git revert --abort`.

**Beginner-Friendly Explanation**
Imagine you're writing a book and you realise that a chapter you wrote three chapters ago contains a factual error. You can't just tear out that page—the book is already printed and people have copies. Instead, you write a new page that says "Correction: the information in Chapter 5 is wrong; here is the correct information." That's what `git revert` does. It doesn't erase history; it adds a new entry that undoes a previous mistake. This is safe for shared projects because everyone's copy of the book remains consistent.

### Key Characteristics

- **Forward-Moving**: Creates a new commit; never rewrites or deletes existing commits.
- **Safe for Shared History**: Does not require force-pushing; collaborators' clones remain consistent.
- **Inverse Patch**: Computes the reverse of the specified commit's diff and applies it.
- **Merge-Aware**: Requires `-m` / `--mainline` for merge commits to specify the mainline parent.
- **Conflict-Prone**: May produce conflicts if later commits overlap with the reverted changes.
- **Abortable**: Can be cancelled with `git revert --abort` before completion.

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository with at least one commit.
- A clean working tree (no uncommitted modifications relative to HEAD).
- For merge commits: knowledge of the parent numbers (`-m 1` or `-m 2`).
- For conflicts: ability to resolve merge conflicts manually.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Automated rollback of faulty deployments.
- **Software Development**: Correcting bugs introduced by previous commits.
- **Collaborative Development**: Safely undoing changes on shared branches.

### Core Concepts / Features

1. **git revert** — Creating inverse commits to undo previous changes.
2. **Reverting Public History** — Why `revert` is safe for team collaboration.
3. **Reverting Merge Commits** — The `-m` / `--mainline` flag.
4. **Revert vs. Reset** — Forward-moving history creation vs. destructive pointer shifting.
5. **Handling Conflicts** — `git revert --continue` and `git revert --abort`.

---

## 1. git revert

### Definitions

**Core Definition**
`git revert` is a command that records new commits which reverse the effects of earlier commits, without altering the existing history.

**Technical Definition**
Given one or more existing commits, `git revert` reverts the changes that the related patches introduce and records new commits that record them. The command requires a clean working tree (no modifications from the HEAD commit). For each specified commit, Git computes the inverse diff, applies it to the working tree and index, and creates a new commit with an auto-generated message of the form `Revert "<original subject>"`. The original commits remain in the DAG, and the new revert commits are appended as children of the current HEAD.

**Beginner-Friendly Explanation**
`git revert` is your "undo" button that works even after you've shared your work with others. If you made a commit that introduced a bug, you can run `git revert <commit-hash>` to create a new commit that undoes exactly what that commit did. The original commit stays in the history—everyone can see that the mistake was made and then fixed.

### Purposes

- To undo the effects of a faulty commit without rewriting history.
- To safely reverse changes on branches that have been pushed to a shared remote.
- To correct a bug introduced by a specific commit while preserving the audit trail.
- To revert a range of commits (e.g., `HEAD~3..HEAD`).
- To stage revert changes without committing immediately (`--no-commit`).

### Syntax Rules and Structure

**Complete General Syntax**

```
git revert [--[no-]edit] [-n] [-m <parent-number>] [-s] [-S[<keyid>]] <commit>…
git revert (--continue | --skip | --abort | --quit)
```

**Key Options**

| Option | Description |
|--------|-------------|
| `<commit>…` | One or more commits to revert. |
| `-e` / `--edit` | Edit the commit message prior to committing (default in terminal). |
| `--no-edit` | Do not start the commit message editor. |
| `-n` / `--no-commit` | Apply the revert to the working tree and index but do not create commits. |
| `-m <parent-number>` / `--mainline <parent-number>` | For merge commits: specify the parent number of the mainline. |
| `-s` / `--signoff` | Add a Signed-off-by trailer. |
| `-S[<keyid>]` | GPG-sign the revert commit. |
| `--continue` | Continue the revert after resolving conflicts. |
| `--skip` | Skip the current commit and continue with remaining ones. |
| `--abort` | Cancel the revert and return to the pre-sequence state. |
| `--quit` | Forget about the current operation in progress. |

**Syntax Rules**

- The working tree must be clean before running `git revert`.
- Multiple commits can be specified: `git revert <commit1> <commit2>`.
- A commit range can be used: `git revert HEAD~3..HEAD`.
- `--no-commit` applies the inverse patches without creating commits, allowing multiple reverts to be batched into one commit.
- For merge commits, `-m` is mandatory to specify the mainline parent.

**Constraints and Limitations**

- **Clean working tree required**: `git revert` refuses to run if there are uncommitted changes.
- **Merge commits**: Cannot be reverted without `-m`; the parent number must be specified.
- **Conflicts**: May occur if later commits modified the same lines; must be resolved manually.
- **Binary files**: Reverting binary file changes may not be possible.
- **Revert-of-a-revert**: Reverting a revert commit re-applies the original changes.
- **Merge revert caveat**: Reverting a merge commit declares you never want the tree changes brought in by the merge; later merges will not bring those changes back automatically.

### Annotated Code Examples

#### Example 1: Reverting a Single Commit

```bash
# Create a repository with a faulty commit
mkdir revert-demo && cd revert-demo
git init
echo "Line 1" > file.txt
git add file.txt && git commit -m "Add file with line 1"
echo "Buggy line" >> file.txt
git commit -am "Add buggy line"

# Revert the faulty commit
git revert HEAD
```

**Expected Output:**
```
[master 4d5e6f7] Revert "Add buggy line"
 1 file changed, 1 deletion(-)
```

**Why this output occurs:** `git revert HEAD` computes the inverse of the changes introduced by `HEAD` (the addition of "Buggy line") and creates a new commit that removes it. The original commit remains in history; the new revert commit is appended.

#### Example 2: Reverting Multiple Commits with `--no-commit`

```bash
# Revert the last three commits as a single revert commit
git revert --no-commit HEAD~3..HEAD
git commit -m "Revert last three commits"
```

**Expected Output:**
```
[master 7a8b9c0] Revert last three commits
 3 files changed, 15 deletions(-)
```

**Why this output occurs:** `--no-commit` applies all inverse patches to the working tree and index without creating individual commits. The developer then creates a single commit that reverses all three changes. This is useful for batching related reverts.

#### Example 3: Reverting with `--no-edit`

```bash
git revert --no-edit HEAD
```

**Expected Output:**
```
[master 1a2b3c4] Revert "Add buggy line"
 1 file changed, 1 deletion(-)
```

**Why this output occurs:** `--no-edit` uses the auto-generated revert message without opening the editor. This is useful for scripting and automated workflows.

### Real-World Cases

- **Faulty deployment**: A CI/CD pipeline detects a bug in production and automatically runs `git revert <commit>` to roll back the faulty change.
- **Bug fix correction**: A developer commits a fix that introduces a regression; another developer reverts the fix and then applies a corrected version.
- **Feature rollback**: A feature is merged but causes performance issues; the team reverts the merge commit to restore stability.

### References

- Git Documentation: git-revert Manual Page — https://git-scm.com/docs/git-revert
- Git Documentation: git-revert (SYNOPSIS) — https://git-scm.com/docs/git-revert
- NYCU CSIT Mirror: git-revert(1) Manual Page — http://kernel.cs.nycu.edu.tw/software/scm/git/docs/git-revert.html

---

## 2. Reverting Public History (Why It Is Safe for Team Collaboration)

### Definitions

**Core Definition**
Reverting public history is the practice of using `git revert` to undo changes on commits that have already been pushed to a shared remote, without rewriting the repository's history.

**Technical Definition**
`git revert` creates new commits that are appended to the branch, leaving all existing commits intact. Because the branch pointer moves forward (fast-forwardable), collaborators can pull the new revert commits without any history divergence. No force-push is required, and no one's local repository becomes inconsistent. This contrasts with `git reset` or `git commit --amend`, which rewrite history and require force-pushing, causing collaborators' clones to diverge.

**Beginner-Friendly Explanation**
When you've already shared your work with others, you can't just "erase" a commit—everyone else already has it. `git revert` solves this by adding a new commit that undoes the old one. It's like making a correction in a meeting minutes document that has already been distributed: you don't shred everyone's copy; you send out a new version with a correction note.

### Purposes

- To safely undo changes on shared branches without disrupting collaborators.
- To maintain a complete, auditable history of all changes (including mistakes and corrections).
- To avoid the risks and coordination overhead of force-pushing.
- To comply with branch protection rules that block force-pushes.
- To enable automated rollbacks in CI/CD pipelines.

### Syntax Rules and Structure

**Comparison: Revert vs. Reset for Public History**

| Aspect | `git revert` | `git reset` |
|--------|-------------|-------------|
| History | Preserved; new commit added | Rewritten; commit removed |
| Branch pointer | Moves forward | Moves backward |
| Force-push required? | No | Yes |
| Safe for shared branches? | ✅ Yes | ❌ No |
| Collaborator impact | None | Requires reset/rebase |
| Use case | Public commits | Local commits only |

**Syntax Rules**

- Always prefer `git revert` for commits that have been pushed to a shared remote.
- Use `git reset` only for local, unpushed commits.
- If you must rewrite public history, use `git push --force-with-lease` instead of `--force` to minimise risk.

**Constraints and Limitations**

- **Revert-of-a-revert complexity**: Reverting a revert re-applies the original changes, which may be confusing.
- **Merge revert caveat**: Reverting a merge commit permanently removes the merged changes; re-merging later will not bring them back automatically.
- **Conflict potential**: Reverting old commits may conflict with subsequent changes.

### Annotated Code Examples

#### Example 1: Safe Revert on a Shared Branch

```bash
# A faulty commit has been pushed to the shared main branch
git log --oneline
# Output:
# 4d5e6f7 (HEAD -> main, origin/main) Add buggy feature
# 1a2b3c4 Initial commit

# Revert the faulty commit (safe for shared history)
git revert --no-edit HEAD
git push origin main
```

**Expected Output:**
```
[main 9a0b1c2] Revert "Add buggy feature"
 1 file changed, 5 deletions(-)
To https://github.com/user/repo.git
   4d5e6f7..9a0b1c2  main -> main
```

**Why this output occurs:** `git revert` creates a new commit (`9a0b1c2`) that undoes the faulty feature. The push is a fast-forward update from `4d5e6f7` to `9a0b1c2`—no force-push needed. Collaborators can pull the change without any history divergence.

#### Example 2: Unsafe Reset on a Shared Branch

```bash
# DANGEROUS: Reset a shared branch (requires force-push)
git reset --hard HEAD~1
git push --force origin main
```

**Why this is unsafe:** `git reset --hard` removes the faulty commit from the branch, and `--force` overwrites the remote history. Collaborators who have pulled the original commit now have a diverged history and must reset or rebase, potentially losing work. This is why `git revert` is the preferred method for public history.

### Real-World Cases

- **Production incident**: A team pushes a faulty configuration change; they use `git revert` to safely undo it without disrupting the CI/CD pipeline.
- **Open-source contribution**: A contributor's PR introduces a regression after merging; the maintainer reverts the merge commit, and the contributor can re-submit a corrected version.
- **Branch protection**: A repository with branch protection rules blocks force-pushes; `git revert` is the only way to undo a change on the protected branch.

### References

- TortoiseGit Documentation: Fixing a Mistake with a New Commit — https://tortoisegit.org/docs/TortoiseGit.pdf#228#25
- Karchunt: Reset and Revert — https://mintlify.wiki/KarChunT/karchunt.com/docs/git/reset-and-revert
- Stack Overflow: Safe Rollback with git revert — https://stackoverflow.com/questions/18549040

---

## 3. Reverting Merge Commits (The -m / --mainline Flag)

### Definitions

**Core Definition**
Reverting a merge commit requires the `-m` / `--mainline` flag to specify which parent of the merge represents the mainline, allowing Git to compute the inverse of the merged changes relative to that parent.

**Technical Definition**
A merge commit has two or more parents. To revert it, Git needs to know which parent's tree should be considered the "mainline" (the branch you want to return to). The `-m <parent-number>` option specifies the parent number (starting from 1) of the mainline. The revert then reverses the changes introduced by the other parent(s) relative to the specified mainline. Reverting a merge commit declares that you will never want the tree changes brought in by the merge; later merges will only bring in changes that are not ancestors of the reverted merge.

**Beginner-Friendly Explanation**
A merge commit is like a junction where two roads (branches) come together. If you want to undo the merge, you need to tell Git which road is the "main" one you want to stay on. The `-m 1` flag says "the first parent (usually the branch you merged into) is the mainline." Git then removes the changes that came from the other road. This is essential because Git can't guess which side you want to keep.

### Purposes

- To undo a merge that introduced bugs or unwanted changes.
- To roll back a feature branch that was merged but later deemed problematic.
- To restore the mainline branch to its state before the merge.
- To reverse the effects of a merge without rewriting history.
- To safely undo a merge on a shared branch.

### Syntax Rules and Structure

**Complete General Syntax**

```
git revert -m <parent-number> <merge-commit>
git revert --mainline <parent-number> <merge-commit>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `-m <parent-number>` | Specify the parent number of the mainline (1-based). |
| `<merge-commit>` | The SHA-1 or reference of the merge commit to revert. |

**Parent Numbering**

| Parent | Meaning |
|--------|---------|
| `1` | The first parent — typically the branch you merged **into** (e.g., `main`). |
| `2` | The second parent — typically the branch that was merged **from** (e.g., `feature`). |

**Syntax Rules**

- `-m` is mandatory for merge commits; without it, Git reports an error.
- `-m 1` is the most common choice, reverting the merge relative to the branch that was merged into.
- `-m 2` reverts relative to the branch that was merged from.
- After reverting a merge, re-merging the same branch will not bring back the reverted changes automatically.

**Constraints and Limitations**

- **Permanent merge undo**: Reverting a merge permanently removes the merged changes; re-merging requires reverting the revert first.
- **Complex histories**: In octopus merges (more than two parents), all parent numbers must be considered.
- **Conflict potential**: High, especially if the merged branch has continued to evolve.

### Annotated Code Examples

#### Example 1: Reverting a Merge Commit with `-m 1`

```bash
# Create a feature branch and merge it
git init merge-revert-demo && cd merge-revert-demo
echo "Main content" > main.txt
git add main.txt && git commit -m "Main commit"
git checkout -b feature
echo "Feature content" > feature.txt
git add feature.txt && git commit -m "Feature commit"
git checkout main
git merge feature --no-edit

# The merge commit is now HEAD. Revert it.
git revert -m 1 HEAD
```

**Expected Output:**
```
[main 4d5e6f7] Revert "Merge branch 'feature'"
 1 file changed, 1 deletion(-)
 delete mode 100644 feature.txt
```

**Why this output occurs:** `-m 1` tells Git that parent 1 (the `main` branch tip before the merge) is the mainline. The revert removes the changes introduced by parent 2 (the `feature` branch), deleting `feature.txt`. The mainline branch is restored to its pre-merge state.

#### Example 2: Checking Parent Numbers Before Reverting

```bash
# Inspect the merge commit to see its parents
git log --oneline --graph -3
# Output:
# *   4d5e6f7 (HEAD -> main) Merge branch 'feature'
# |\
# | * 1a2b3c4 (feature) Feature commit
# * | 9a0b1c2 Main commit
# |/

# Parent 1 is 9a0b1c2 (main); Parent 2 is 1a2b3c4 (feature)
git revert -m 1 4d5e6f7
```

**Why this is useful:** Before reverting, inspecting the merge commit with `git log --graph` helps identify which parent is which. Parent 1 is usually the branch you were on when you ran `git merge`.

### Real-World Cases

- **Faulty feature merge**: A feature branch is merged into `main`, but it introduces a critical bug. The team reverts the merge commit with `-m 1` to restore `main` to a stable state.
- **Accidental merge**: A developer accidentally merges a branch; they revert the merge with `-m 1` and then re-merge correctly.
- **Release rollback**: A release branch is merged into `main`, but the release is delayed; the merge is reverted with `-m 1` until the release is ready.

### References

- Git Documentation: git-revert (-m) — https://git-scm.com/docs/git-revert
- NYCU CSIT Mirror: git-revert (-m) — http://kernel.cs.nycu.edu.tw/software/scm/git/docs/git-revert.html
- GitHub Community: Manual Revert with -m 1 — https://github.com/orgs/community/discussions/168111
- Metasploit Docs: Rolling Back Merges — https://docs.metasploit.com

---

## 4. Revert Versus Reset (Forward-Moving History Creation vs. Destructive Backward Pointer Shifting)

### Definitions

**Core Definition**
`git revert` and `git reset` are two fundamentally different ways to undo commits: `revert` creates new commits that undo previous changes (forward-moving), while `reset` moves the branch pointer backward, removing commits from the branch (destructive).

**Technical Definition**
`git revert` computes the inverse diff of a specified commit and appends a new commit to the branch, preserving the original commit in the DAG. `git reset` moves the current branch reference (and HEAD) to a specified commit, effectively removing all commits after that point from the branch. `reset` has three modes: `--soft` (moves HEAD, keeps changes staged), `--mixed` (default; moves HEAD, keeps changes unstaged), and `--hard` (moves HEAD, discards all changes). `revert` is safe for shared history; `reset` rewrites history and requires force-pushing if the commits were already pushed.

**Beginner-Friendly Explanation**
Think of your commit history as a book. `git revert` is like adding a new page that says "Correction: Chapter 5 had an error." The original page stays in the book. `git reset` is like tearing pages out of the book and gluing the binding back together—the pages are gone, and if someone else already has a copy with those pages, their book no longer matches yours.

### Purposes

- To understand when to use `revert` (shared history) vs. `reset` (local history).
- To avoid disrupting collaborators by rewriting public history.
- To choose the correct reset mode for different local undo scenarios.
- To maintain a complete audit trail with `revert`.
- To clean up local commits before pushing with `reset`.

### Syntax Rules and Structure

**Comparison Table**

| Aspect | `git revert` | `git reset` |
|--------|-------------|-------------|
| **Direction** | Forward (new commit) | Backward (pointer moves) |
| **History** | Preserved | Rewritten |
| **Original commit** | Remains in DAG | Removed from branch |
| **Branch pointer** | Advances | Moves backward |
| **Safe for shared branches?** | ✅ Yes | ❌ No |
| **Force-push required?** | No | Yes (if pushed) |
| **Undo the undo** | Revert the revert | Reflog + reset |
| **Use case** | Public commits | Local commits |

**Reset Modes**

| Mode | HEAD | Index | Working Tree | Use Case |
|------|------|-------|--------------|----------|
| `--soft` | Moves | Unchanged | Unchanged | Undo commit, keep changes staged |
| `--mixed` (default) | Moves | Resets | Unchanged | Undo commit, keep changes unstaged |
| `--hard` | Moves | Resets | Resets | Undo commit, discard all changes |

**Syntax Rules**

- Use `git revert` for commits that have been pushed to a shared remote.
- Use `git reset` for local commits that have not been shared.
- Use `git reset --soft` to undo a commit but keep changes staged for re-committing.
- Use `git reset --hard` only when you are certain you want to permanently discard changes.
- After `git reset`, if the commits were already pushed, a force-push (`--force-with-lease`) is required.

**Constraints and Limitations**

- **`reset` is destructive**: `--hard` permanently discards uncommitted changes.
- **`reset` rewrites history**: Collaborators who have pulled the original commits will have diverged histories.
- **`revert` creates noise**: Many reverts can clutter the history with "Revert" commits.
- **`revert` cannot undo the original commit**: It only reverses its effects; the original remains.

### Annotated Code Examples

#### Example 1: `git revert` (Forward-Moving)

```bash
# Create a commit
echo "Feature" > feature.txt
git add feature.txt && git commit -m "Add feature"

# Revert it (forward-moving)
git revert --no-edit HEAD

# History now contains both commits
git log --oneline
# Output:
# 4d5e6f7 (HEAD -> main) Revert "Add feature"
# 1a2b3c4 Add feature
```

**Expected Output:**
```
4d5e6f7 (HEAD -> main) Revert "Add feature"
1a2b3c4 Add feature
```

**Why this output occurs:** `git revert` adds a new commit that undoes the feature. The original commit remains in history. This is safe for shared branches.

#### Example 2: `git reset --hard` (Destructive Backward)

```bash
# Reset to the previous commit (destructive)
git reset --hard HEAD~1

# History no longer contains the feature commit
git log --oneline
# Output: 1a2b3c4 Initial commit
```

**Expected Output:**
```
1a2b3c4 Initial commit
```

**Why this output occurs:** `git reset --hard HEAD~1` moves the branch pointer back one commit and discards all changes from the removed commit. The feature commit is no longer in the branch history. If this branch had been pushed, a force-push would be required, disrupting collaborators.

#### Example 3: `git reset --soft` (Undo Commit, Keep Changes Staged)

```bash
# Undo the last commit but keep changes staged
git reset --soft HEAD~1

# Check status
git status -s
# Output: A  feature.txt
```

**Expected Output:**
```
A  feature.txt
```

**Why this output occurs:** `--soft` moves HEAD back but leaves the index and working tree unchanged. The changes from the undone commit are now staged, ready to be re-committed with a different message or additional changes.

### Real-World Cases

- **Local cleanup**: A developer makes several messy commits before pushing; they use `git reset --soft` to combine them into one clean commit.
- **Public rollback**: A team pushes a faulty commit; they use `git revert` to safely undo it without disrupting the shared branch.
- **Accidental commit**: A developer commits a file they didn't mean to include; they use `git reset --soft HEAD~1` to uncommit it, then re-commit without the file.

### References

- Karchunt: Reset and Revert — https://mintlify.wiki/KarChunT/karchunt.com/docs/git/reset-and-revert
- KodeKloud: About git revert and reset — https://kodekloud.com
- LabEx: How does 'git revert' differ from 'git reset'? — https://labex.io
- Mergify: A Developer's Guide to Git Undo Push Safely — https://articles.mergify.com

---

## 5. Handling Conflicts During a Revert

### Definitions

**Core Definition**
A conflict during a revert occurs when the inverse patch cannot be applied cleanly because subsequent commits have modified the same lines, requiring manual resolution before the revert can be completed.

**Technical Definition**
When `git revert` applies the inverse diff of a commit, Git uses a three-way merge between the current HEAD, the commit to be reverted, and its parent. If the changes in the current HEAD overlap with the changes being reverted, Git cannot automatically resolve the conflict and pauses the revert, marking the conflicted files in the working tree. The user must resolve the conflicts manually, stage the resolved files, and run `git revert --continue` to complete the operation, or `git revert --abort` to cancel.

**Beginner-Friendly Explanation**
Sometimes when you try to undo a commit, Git can't do it automatically because other changes have been made to the same lines since then. Git marks the conflicting parts of the files and asks you to decide what to keep. You edit the files to resolve the conflict, then tell Git to continue. If it gets too messy, you can abort and start over.

### Purposes

- To understand why revert conflicts occur.
- To resolve revert conflicts systematically.
- To continue a paused revert after resolution (`--continue`).
- To cancel a revert and restore the original state (`--abort`).
- To skip a problematic commit and continue with the rest (`--skip`).

### Syntax Rules and Structure

**Conflict Resolution Workflow**

| Step | Command | Description |
|------|---------|-------------|
| 1 | `git revert <commit>` | Start the revert; conflict may occur. |
| 2 | `git status` | Identify conflicted files. |
| 3 | Edit files | Resolve conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`). |
| 4 | `git add <file>` | Stage resolved files. |
| 5 | `git revert --continue` | Continue the revert and create the commit. |
| — | `git revert --abort` | Cancel the revert and return to pre-revert state. |
| — | `git revert --skip` | Skip the current commit and continue with remaining ones. |

**Conflict Markers**

```
<<<<<<< HEAD
Current changes (what you have now)
=======
Changes from the inverse patch (what revert wants to apply)
>>>>>>> parent of <commit> (Revert "<subject>")
```

**Syntax Rules**

- `git revert --continue` requires all conflicts to be resolved and staged.
- `git revert --abort` restores the working tree and index to the state before the revert began.
- `git revert --skip` is useful when reverting multiple commits and one cannot be applied.
- The `--quit` option forgets about the operation without restoring the pre-revert state.

**Constraints and Limitations**

- **Manual resolution required**: Git cannot auto-resolve overlapping changes.
- **Staging required**: Resolved files must be staged with `git add` before `--continue`.
- **Abort safety**: `--abort` restores the pre-revert state but may fail if there are unrelated uncommitted changes.
- **Complex conflicts**: Large overlapping changes may be difficult to resolve manually.

### Annotated Code Examples

#### Example 1: Resolving a Revert Conflict

```bash
# Create a commit, then modify the same line
echo "Original" > file.txt
git add file.txt && git commit -m "Add file"
echo "Modified" > file.txt
git commit -am "Modify file"

# Try to revert the first commit (conflict!)
git revert HEAD~1
```

**Expected Output:**
```
error: could not revert 1a2b3c4... Add file
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

**Conflict Resolution:**
```bash
# Check status
git status -s
# Output: UU file.txt

# Edit file.txt to resolve the conflict
cat file.txt
# Output:
# <<<<<<< HEAD
# Modified
# =======
# >>>>>>> parent of 1a2b3c4... Add file

# Resolve by keeping the modification
echo "Modified" > file.txt
git add file.txt

# Continue the revert
git revert --continue
```

**Expected Output:**
```
[master 4d5e6f7] Revert "Add file"
 1 file changed, 0 insertions(+), 0 deletions(-)
```

**Why this output occurs:** The revert of "Add file" conflicted with the later "Modify file" commit. The developer resolved the conflict by keeping the modified content. `git revert --continue` then created the revert commit.

#### Example 2: Aborting a Revert

```bash
# Start a revert that conflicts
git revert HEAD~1

# Decide to cancel
git revert --abort
```

**Expected Output:**
```
(no output; revert cancelled)
```

**Why this output occurs:** `git revert --abort` restores the working tree and index to the state before the revert began. The repository is returned to its pre-revert state.

### Real-World Cases

- **Complex revert**: A developer tries to revert an old commit but conflicts arise because many changes have been made since; they abort and use a different approach.
- **CI/CD conflict handling**: A pipeline script detects a revert conflict, logs the conflicted files, and exits gracefully for manual intervention.
- **Partial revert**: A developer uses `--skip` to skip a problematic commit and continue reverting the rest.

### References

- Git Documentation: git-revert (--continue, --abort) — https://git-scm.com/docs/git-revert
- Stack Overflow: Resolving Git Revert Conflicts — https://stackoverflow.com/questions/18549040
- Linux Command Library: git-revert — https://linuxcommandlibrary.com

---

## References

- Git Documentation: git-revert Manual Page — https://git-scm.com/docs/git-revert
- Git Documentation: git-reset Manual Page — https://git-scm.com/docs/git-reset
- Git Documentation: git-commit Manual Page — https://git-scm.com/docs/git-commit
- NYCU CSIT Mirror: git-revert(1) Manual Page — http://kernel.cs.nycu.edu.tw/software/scm/git/docs/git-revert.html
- TortoiseGit Documentation: Fixing a Mistake with a New Commit — https://tortoisegit.org/docs/TortoiseGit.pdf#228#25
- Karchunt: Reset and Revert — https://mintlify.wiki/KarChunT/karchunt.com/docs/git/reset-and-revert
- KodeKloud: About git revert and reset — https://kodekloud.com
- LabEx: How does 'git revert' differ from 'git reset'? — https://labex.io
- Mergify: A Developer's Guide to Git Undo Push Safely — https://articles.mergify.com
- GitHub Community: Manual Revert with -m 1 — https://github.com/orgs/community/discussions/168111
- Metasploit Docs: Rolling Back Merges — https://docs.metasploit.com
- Stack Overflow: Safe Rollback with git revert — https://stackoverflow.com/questions/18549040
- Linux Command Library: git-revert — https://linuxcommandlibrary.com