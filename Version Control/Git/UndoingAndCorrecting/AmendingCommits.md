# Git Amending Commits: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Amending a commit is the practice of replacing the most recent commit with a new commit that has modified content, a modified message, or both, without creating a separate new commit in the history.

**Technical Definition**
`git commit --amend` reads the current index (which may include newly staged changes), creates a new tree object representing the updated snapshot, constructs a new commit object with the same parent(s) as the original commit but potentially a different tree, message, author, or committer metadata, and updates the current branch reference to point to the new commit. The original commit object becomes unreachable from the branch but remains in the object database and is recoverable via the reflog.

**Beginner-Friendly Explanation**
Imagine you just saved a file, but then realized you forgot to include an important paragraph or made a typo in the title. `git commit --amend` is like using “undo” on your last save and immediately redoing it correctly. It lets you fix the most recent commit—change its message, add forgotten files, or both—without creating a messy “fix” commit on top. But be careful: if you've already shared that commit with others, amending it can cause confusion, because it creates a completely new commit with a different ID.

### Key Characteristics

- **History Rewriting**: Amending replaces a commit; it does not add to history.
- **SHA Change**: The amended commit has a completely new SHA-1 (or SHA-256) hash because its content (tree, message, timestamp, parent) differs.
- **Local Operation**: Amending is a local operation; it does not affect the remote until you push.
- **Recoverable**: The original commit remains in the reflog for a default of 90 days.
- **Preserves Parents**: The amended commit keeps the same parent(s) as the original; only the commit itself changes.

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository with at least one commit (HEAD must exist).
- The commit to be amended must be the most recent commit on the current branch.
- For adding files: the forgotten files must be staged with `git add` before amending.
- Awareness that amending pushed/shared history requires force-pushing and coordination.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Commit hygiene in automated pipelines.
- **Software Development**: Correcting mistakes before pushing.
- **Collaborative Development**: Managing shared history responsibly.

### Core Concepts / Features

1. **git commit --amend** — The command for replacing the most recent commit.
2. **Fixing Commit Messages** — Correcting typos or improving wording.
3. **Adding Forgotten Files** — Including files omitted from the last commit.
4. **When Amendment Is Safe** — Local commits vs. pushed/shared history.
5. **The Mechanical Reality** — Creating a new commit object and orphaning the old one.

---

## 1. git commit --amend

### Definitions

**Core Definition**
`git commit --amend` is a command that replaces the most recent commit on the current branch with a new commit, incorporating any newly staged changes and/or a new commit message.

**Technical Definition**
`git commit --amend` reads the index, creates a new tree object, constructs a new commit object with the same parent(s) as HEAD but with updated content and/or metadata, writes the new objects to the object database, and updates the current branch reference (or HEAD in detached state) to point to the new commit. The original commit becomes unreachable from the branch but remains in the reflog. The command accepts the same options as `git commit` (e.g., `-m`, `--no-edit`, `--reset-author`).

**Beginner-Friendly Explanation**
`git commit --amend` lets you fix the last commit. Forgot to add a file? Stage it and run `git commit --amend --no-edit`. Made a typo in the message? Run `git commit --amend` to edit it. It's like using “undo” on your last commit and redoing it correctly.

### Purposes

- To fix a typo or improve the wording of the last commit message.
- To add forgotten files to the last commit.
- To remove files accidentally included in the last commit.
- To change the author of the last commit (`--reset-author`).
- To GPG-sign a commit that was not signed initially.
- To correct the commit timestamp (`--date`).

### Syntax Rules and Structure

**Complete General Syntax**

```
git commit --amend [<options>] [--] [<pathspec>...]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `--amend` | Replace the most recent commit. |
| `--no-edit` | Use the existing commit message without opening an editor. |
| `-m <msg>` | Replace the commit message with the given message. |
| `--reset-author` | Reset the author to the current user (and update the author timestamp). |
| `--author=<author>` | Override the author. |
| `--date=<date>` | Override the author date. |
| `--allow-empty` | Allow amending with no changes. |
| `-S[<keyid>]` | GPG-sign the amended commit. |
| `--no-verify` | Bypass pre-commit and commit-msg hooks. |

**Syntax Rules**

- `--amend` alone opens the editor with the existing message for editing.
- `--amend --no-edit` keeps the existing message and does not open an editor.
- `--amend -m "new message"` replaces the message without opening an editor.
- Staged changes are included in the amended commit. Unstaged changes are not.
- The original commit is replaced; its hash changes, and it becomes unreachable from the branch.
- The amended commit has the same parent(s) as the original. To change parents, use `git rebase`.

**Constraints and Limitations**

- **History rewriting**: `--amend` rewrites history. Do **not** amend commits that have been pushed to a shared remote, as this forces other developers to rebase.
- **Recovery**: The original commit can be recovered via `git reflog` and `git reset --hard <old-hash>`.
- **Parent unchanged**: The amended commit has the same parent(s) as the original.
- **Detached HEAD**: Amending in detached HEAD works but the amended commit has no branch reference.
- **Version-specific**: `--amend` has been available since Git 1.0; `--no-edit` since Git 1.6.0.
- **Uncommitted changes**: Unstaged changes are not included; only staged changes are.

### Annotated Code Examples

#### Example 1: Amending a Commit Message

```bash
# Create a repository and commit with a typo
git init amend-demo && cd amend-demo
echo "content" > file.txt
git add file.txt
git commit -m "Add file with typo"

# Amend the message (opens editor)
git commit --amend
```

**Editor opens:**
```
Add file without typo

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
```

**Expected Output:**
```
[master 1a2b3c4] Add file without typo
 Date: ...
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** `--amend` without `-m` opens the editor with the existing message. The developer edits the message and saves. Git creates a new commit object with the same tree and parent but a new message and hash.

#### Example 2: Adding a Forgotten File with `--no-edit`

```bash
# Commit a file but forget to add another
echo "file1" > file1.txt
echo "file2" > file2.txt
git add file1.txt
git commit -m "Add file1"

# Oops, forgot file2.txt. Stage it and amend without editing the message.
git add file2.txt
git commit --amend --no-edit
```

**Expected Output:**
```
[master 5d6e7f8] Add file1
 Date: ...
 2 files changed, 2 insertions(+)
 create mode 100644 file1.txt
 create mode 100644 file2.txt
```

**Why this output occurs:** `--no-edit` keeps the existing message (“Add file1”). The staged `file2.txt` is included in the amended commit. The commit now contains both files.

#### Example 3: Changing the Author with `--reset-author`

```bash
# Amend the last commit and reset the author to the current user
git commit --amend --reset-author --no-edit
```

**Expected Output:**
```
[master 7a8b9c0] Add file1
 Author: Current User <current@example.com>
 Date: ...
 2 files changed, 2 insertions(+)
```

**Why this output occurs:** `--reset-author` sets the author to the current `user.name` and `user.email` configuration and updates the author timestamp. This is useful when a commit was made with the wrong identity.

### Real-World Cases

- **Fixing a typo in the commit message**: A developer runs `git commit --amend` to correct a misspelling before pushing.
- **Adding a forgotten file**: A developer stages a forgotten file and runs `git commit --amend --no-edit` to include it.
- **Changing the author**: A developer who committed with the wrong email runs `git commit --amend --reset-author` after updating `user.email`.
- **Signing a commit**: A developer who forgot to GPG-sign runs `git commit --amend -S --no-edit` to add a signature.

### References

- Git Documentation: git-commit Manual Page — https://git-scm.com/docs/git-commit
- Git Documentation: git-commit (--amend) — https://www.man.he.net/man1/git-commit
- Git Documentation: git-commit (--no-edit) — https://manpages.org/git-commit#1

---

## 2. Fixing Commit Messages

### Definitions

**Core Definition**
Fixing a commit message is the act of replacing the message of the most recent commit with a corrected or improved version using `git commit --amend`.

**Technical Definition**
The commit message is stored as part of the commit object. `git commit --amend` reads the existing commit message (from `.git/COMMIT_EDITMSG` or the original commit object), opens it in the configured editor for modification, and constructs a new commit object with the updated message. The original commit becomes unreachable.

**Beginner-Friendly Explanation**
If you made a typo in your commit message or want to add more detail, `git commit --amend` opens your editor with the old message so you can fix it. It's like editing a saved document—you're correcting the record before sharing it.

### Purposes

- To correct spelling or grammatical errors in a commit message.
- To improve the clarity or specificity of a commit message.
- To add a body or footer to a previously terse message.
- To change the commit message to follow a convention (e.g., Conventional Commits).
- To fix an incorrect issue reference.

### Syntax Rules and Structure

**Complete General Syntax**

```
git commit --amend
git commit --amend -m "<new message>"
git commit --amend --no-edit
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--amend` | Replace the most recent commit. |
| `-m "<msg>"` | Provide a new message inline, bypassing the editor. |
| `--no-edit` | Keep the existing message (useful when only adding files). |

**Syntax Rules**

- Without `-m` or `--no-edit`, the editor opens with the existing message.
- `-m` can be used multiple times; each becomes a separate paragraph.
- The message can also be read from a file with `-F <file>`.
- The `--cleanup` option controls comment stripping and whitespace trimming.

**Constraints and Limitations**

- **Pushed history**: If the commit has been pushed, amending the message requires a force-push, which can disrupt collaborators.
- **Editor dependency**: The editor-based approach requires a configured editor (`core.editor` or `$EDITOR`).
- **No undo**: The original message is replaced; it can be recovered only via the reflog (if the original commit is still reachable).

### Annotated Code Examples

#### Example 1: Correcting a Typo with the Editor

```bash
# Commit with a typo
git commit -m "Fix teh login bug"

# Amend the message
git commit --amend
```

**Editor opens:**
```
Fix the login bug

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
```

**Expected Output:**
```
[master 1a2b3c4] Fix the login bug
 Date: ...
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** The editor opens with the original message. The developer corrects “teh” to “the,” saves, and closes the editor. Git creates a new commit with the corrected message and a new hash.

#### Example 2: Inline Replacement with `-m`

```bash
git commit --amend -m "fix: correct null pointer in login handler"
```

**Expected Output:**
```
[master 5d6e7f8] fix: correct null pointer in login handler
 Date: ...
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** `-m` provides the new message inline, bypassing the editor. The commit object is replaced with one using the new message.

### Real-World Cases

- **Pre-push cleanup**: A developer reviews their commit messages before pushing and corrects any that are unclear.
- **Convention compliance**: A developer rewrites a commit message to follow Conventional Commits (e.g., changing “fixed bug” to “fix: resolve null pointer”).
- **Issue reference correction**: A developer updates a commit message to reference the correct issue number.

### References

- Git Documentation: git-commit (--amend) — https://git-scm.com/docs/git-commit
- Git Documentation: git-commit (-m) — https://www.man.he.net/man1/git-commit

---

## 3. Adding Forgotten Files

### Definitions

**Core Definition**
Adding forgotten files to a commit means staging files that were omitted from the most recent commit and then amending the commit to include them.

**Technical Definition**
After staging the forgotten files with `git add`, `git commit --amend --no-edit` reads the updated index, creates a new tree object that includes the additional blobs, and constructs a new commit object with the same message but the updated tree. The original commit becomes unreachable.

**Beginner-Friendly Explanation**
You committed your work but forgot to include a file. Just run `git add forgotten-file.txt` and then `git commit --amend --no-edit`. The file is added to the last commit, and the message stays the same. No extra commit needed.

### Purposes

- To include files accidentally omitted from the last commit.
- To keep the commit history clean by avoiding “fix” commits.
- To ensure a commit is complete and self-contained.
- To correct the snapshot of a commit before sharing it.
- To add generated or configuration files that were missing.

### Syntax Rules and Structure

**Complete General Syntax**

```
git add <forgotten-file>
git commit --amend --no-edit
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `git add <file>` | Stage the forgotten file(s). |
| `--amend` | Replace the last commit. |
| `--no-edit` | Keep the existing commit message unchanged. |

**Syntax Rules**

- Only staged changes are included in the amended commit.
- `--no-edit` is recommended when the commit message is still accurate.
- If the message needs updating (e.g., to mention the new file), omit `--no-edit`.
- Multiple files can be staged before amending.

**Constraints and Limitations**

- **Pushed history**: Amending a pushed commit requires force-pushing.
- **Unstaged changes**: Only staged changes are included; unstaged changes remain in the working tree.
- **File tracking**: The forgotten file must be a tracked file (or newly added with `git add`).

### Annotated Code Examples

#### Example 1: Adding a Forgotten File

```bash
# Commit one file but forget another
echo "important" > important.txt
echo "also important" > also-important.txt
git add important.txt
git commit -m "Add important file"

# Oops, forgot also-important.txt
git add also-important.txt
git commit --amend --no-edit
```

**Expected Output:**
```
[master 1a2b3c4] Add important file
 Date: ...
 2 files changed, 2 insertions(+)
 create mode 100644 important.txt
 create mode 100644 also-important.txt
```

**Why this output occurs:** `git add also-important.txt` stages the forgotten file. `git commit --amend --no-edit` creates a new commit with the same message but a tree that includes both files. The original commit (with only one file) is replaced.

#### Example 2: Adding a Forgotten File and Updating the Message

```bash
git add also-important.txt
git commit --amend -m "Add important files (important and also-important)"
```

**Expected Output:**
```
[master 5d6e7f8] Add important files (important and also-important)
 Date: ...
 2 files changed, 2 insertions(+)
```

**Why this output occurs:** `-m` provides a new message that accurately reflects the amended commit. The message now mentions both files.

### Real-World Cases

- **Configuration files**: A developer forgets to commit a `.env.example` file and amends the commit to include it.
- **Test files**: A developer forgets to add a test file corresponding to a feature and amends the commit.
- **Documentation**: A developer forgets to update `README.md` and amends the commit to include the documentation.

### References

- Git Documentation: git-commit (--amend) — https://git-scm.com/docs/git-commit
- Git Documentation: git-add Manual Page — https://git-scm.com/docs/git-add

---

## 4. Understanding When Amendment Is Safe

### Definitions

**Core Definition**
Amending is safe when the commit being amended exists only in the local repository and has never been pushed to a shared remote; it is unsafe when the commit has been pushed and other developers may have based work on it.

**Technical Definition**
Amending creates a new commit object with a different SHA-1 hash. If the original commit has been pushed to a remote, the local and remote histories diverge. Pushing the amended commit requires a force-push (`git push --force` or `git push --force-with-lease`), which overwrites the remote history. Other developers who have pulled the original commit will have a diverged history and must rebase or reset, potentially losing work.

**Beginner-Friendly Explanation**
Think of commits like messages in a group chat. If you haven't sent the message yet, you can edit it freely. But if you've already sent it, editing it changes the message for everyone—and people who already read the old version get confused. The same is true for Git: amend commits that are only on your computer; don't amend commits you've already pushed to a shared repository.

### Purposes

- To determine whether an amend operation is safe.
- To understand the risks of rewriting shared history.
- To follow best practices for collaborative development.
- To know when to use `--force-with-lease` instead of `--force`.
- To avoid disrupting teammates' work.

### Syntax Rules and Structure

**Safety Decision Matrix**

| Scenario | Safe to Amend? | Action Required |
|----------|----------------|-----------------|
| Commit is local (not pushed) | ✅ Yes | Amend freely. |
| Commit is on a private feature branch (not shared) | ✅ Yes | Amend freely. |
| Commit is pushed to a shared branch (main, develop) | ❌ No | Create a new commit instead. |
| Commit is pushed to your own fork (personal branch) | ⚠️ Caution | Amend only if you're the sole user. |
| Commit is in a pull request under review | ⚠️ Caution | Amend only with reviewer coordination. |

**Force-Push Options**

| Option | Description | Safety |
|--------|-------------|--------|
| `git push --force` | Overwrites remote history unconditionally. | Dangerous; can lose others' work. |
| `git push --force-with-lease` | Overwrites only if the remote is at the expected state. | Safer; prevents overwriting others' changes. |
| `git push --force-if-includes` | Like `--force-with-lease`, but also checks reflog. | Safest (Git 2.30+). |

**Syntax Rules**

- **Golden Rule**: Never amend commits that have been pushed to a shared branch.
- **Feature branches**: Amending is generally safe on feature branches that you alone are working on.
- **Pull requests**: If a pull request is under review, amending forces reviewers to re-review; coordinate first.
- **Force-push with lease**: Always prefer `--force-with-lease` over `--force` to avoid overwriting others' work.

**Constraints and Limitations**

- **Team disruption**: Force-pushing a shared branch can cause teammates to lose work or face complex rebases.
- **CI/CD impact**: Amending pushed commits can trigger confusing CI runs or break build references.
- **GitHub/GitLab protections**: Many repositories have branch protection rules that block force-pushes to protected branches.
- **Recovery complexity**: Recovering from a bad force-push requires coordination and reflog surgery.

### Annotated Code Examples

#### Example 1: Safe Amendment (Local Commit)

```bash
# Create a local commit
git commit -m "Add feature"

# Amend it (safe, not pushed)
git commit --amend -m "Add feature with correct message"
```

**Why this is safe:** The commit has never been pushed. The amend only affects the local repository. No one else is impacted.

#### Example 2: Unsafe Amendment (Pushed Commit)

```bash
# Push a commit
git push origin main

# Amend it (UNSAFE)
git commit --amend -m "Corrected message"

# Force-push (dangerous)
git push --force origin main
```

**Why this is unsafe:** The original commit is already on the remote. Force-pushing overwrites the remote history. Anyone who has pulled the original commit now has a diverged history and must reset or rebase, potentially losing work.

#### Example 3: Safer Force-Push with Lease

```bash
# Amend and force-push with lease
git commit --amend -m "Corrected message"
git push --force-with-lease origin feature-branch
```

**Why this is safer:** `--force-with-lease` checks that the remote branch is still at the expected commit before overwriting. If someone else has pushed in the meantime, the push is rejected, preventing accidental overwrites.

### Real-World Cases

- **Solo feature branch**: A developer works on a feature branch alone and amends freely before creating a pull request.
- **Shared branch**: A developer accidentally commits to `main` and realizes the mistake; instead of amending (which would require force-push), they create a new commit that corrects the issue.
- **Pull request review**: A reviewer asks for a commit message change; the developer amends and force-pushes with `--force-with-lease`, then notifies the reviewer.
- **CI/CD**: A pipeline commits build artifacts; amending those commits would break the pipeline's assumptions about commit hashes.

### References

- GitHub Docs: Changing a Commit Message — https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/changing-a-commit-message
- Stack Overflow: Amending a Commit That's Already Been Pushed — https://stackoverflow.com/questions/18549040
- Git Documentation: git-push (--force-with-lease) — https://git-scm.com/docs/git-push

---

## 5. The Mechanical Reality of --amend

### Definitions

**Core Definition**
The mechanical reality of `git commit --amend` is that it creates a brand-new commit object in the object database, updates the branch pointer to reference the new commit, and leaves the original commit orphaned (unreachable from any branch or tag).

**Technical Definition**
When `git commit --amend` is executed, Git: (1) reads the index, (2) creates a new tree object (if the index differs from the original tree), (3) constructs a new commit object with the same parent(s) as the original commit, the new tree, and the (possibly updated) message and metadata, (4) writes the new commit object to `.git/objects/`, (5) updates the current branch reference (e.g., `refs/heads/main`) to point to the new commit. The original commit object remains in the object database but is no longer referenced by any branch or tag; it becomes an "orphaned" or "unreachable" object. It remains accessible via the reflog until it is garbage-collected.

**Beginner-Friendly Explanation**
When you amend a commit, Git doesn't actually change the old commit. Instead, it creates a new commit that looks like the old one but with your corrections, and moves the branch pointer to the new commit. The old commit is left behind—like a draft you threw away but that's still in the trash can (the reflog) for a while. Eventually, Git's garbage collector cleans it up.

### Purposes

- To understand what actually happens when you amend.
- To know why the commit hash changes.
- To understand how to recover the original commit (via reflog).
- To reason about repository state and object reachability.
- To explain why amending is a history-rewriting operation.

### Syntax Rules and Structure

**Step-by-Step Mechanical Process**

| Step | Action | Result |
|------|--------|--------|
| 1 | Read the index | Current staged content identified. |
| 2 | Create a new tree object | Snapshot of the staged content (if changed). |
| 3 | Construct a new commit object | Same parent(s), new tree/message/metadata. |
| 4 | Write the new commit object | Stored in `.git/objects/`. |
| 5 | Update the branch reference | Branch now points to the new commit. |
| 6 | Original commit becomes unreachable | No ref points to it; recoverable via reflog. |

**Object Reachability Diagram**

```
Before Amend:
  refs/heads/main ──→ Commit A ──→ Commit B (HEAD)

After Amend:
  refs/heads/main ──→ Commit A' ──→ Commit B
                                    ↑
  (orphaned) Commit A ──────────────┘
  (recoverable via reflog)
```

**Syntax Rules**

- The new commit has the same parent(s) as the original.
- The new commit has a different SHA-1 because its content (tree, message, timestamp) differs.
- The original commit is not deleted immediately; it remains in the object database.
- The reflog records the amend operation: `HEAD@{0}: commit (amend): ...`.
- Garbage collection (`git gc`) eventually removes unreachable objects (default: 90 days for reachable, 30 days for unreachable).

**Constraints and Limitations**

- **No in-place modification**: Git objects are immutable; amending always creates a new object.
- **Reflog dependency**: Recovery depends on the reflog; if the reflog is expired or deleted, the original commit may be unrecoverable.
- **Garbage collection**: Once `git gc --prune=now` is run, orphaned commits are permanently deleted.
- **Branch pointer only**: Only the branch reference is updated; other references (tags, remote-tracking branches) still point to the original.

### Annotated Code Examples

#### Example 1: Observing the New Commit Object

```bash
# Create a commit
git init amend-mechanics && cd amend-mechanics
echo "content" > file.txt
git add file.txt
git commit -m "Original message"

# Record the original hash
git rev-parse HEAD
# Output: 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b

# Amend the message
git commit --amend -m "Amended message"

# Record the new hash
git rev-parse HEAD
# Output: 5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e
```

**Expected Output:**
```
1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e
```

**Why this output occurs:** The original commit has hash `1a2b3c4...`. After amending, the new commit has a completely different hash `5d6e7f8...` because its content (the message) changed. The branch now points to the new commit.

#### Example 2: Recovering the Orphaned Commit via Reflog

```bash
# View the reflog
git reflog
# Output:
# 5d6e7f8 (HEAD -> master) HEAD@{0}: commit (amend): Amended message
# 1a2b3c4 HEAD@{1}: commit: Original message

# Recover the original commit
git reset --hard HEAD@{1}
```

**Expected Output:**
```
HEAD is now at 1a2b3c4 Original message
```

**Why this output occurs:** The reflog records both the amended commit (`HEAD@{0}`) and the original commit (`HEAD@{1}`). `git reset --hard HEAD@{1}` restores the branch to the original commit. The amended commit remains in the reflog until it expires.

#### Example 3: Inspecting the Orphaned Object

```bash
# Show the orphaned commit (still accessible by hash)
git show 1a2b3c4
```

**Expected Output:**
```
commit 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
Author: User <user@example.com>
Date: ...

    Original message
```

**Why this output occurs:** The orphaned commit object still exists in the object database. Git can display it by its hash. It becomes permanently unreachable only after garbage collection.

### Real-World Cases

- **Accidental amend recovery**: A developer amends a commit, realizes the original message was needed, and recovers it via `git reflog`.
- **Understanding SHA changes**: A developer notices that CI references the old commit hash after an amend and needs to update the pipeline.
- **Garbage collection**: A repository administrator runs `git gc` to clean up orphaned commits, reducing repository size.
- **Forensic analysis**: An investigator uses `git fsck --lost-found` to find orphaned commits and reconstruct history.

### References

- Git Documentation: Git Internals – Git Objects — https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
- Git Documentation: git-reflog Manual Page — https://git-scm.com/docs/git-reflog
- CoreUI: How to Undo git commit --amend — https://coreui.io/answers/how-to-undo-git-commit-amend/#author
- GitLab: Unreachable Objects — https://gitlab.sib.swiss

---

## References

- Git Documentation: git-commit Manual Page — https://git-scm.com/docs/git-commit
- Git Documentation: git-reflog Manual Page — https://git-scm.com/docs/git-reflog
- Git Documentation: Git Internals – Git Objects — https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
- Git Documentation: git-push Manual Page — https://git-scm.com/docs/git-push
- Git Documentation: git-reset Manual Page — https://git-scm.com/docs/git-reset
- GitHub Docs: Changing a Commit Message — https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/changing-a-commit-message
- CoreUI: How to Undo git commit --amend — https://coreui.io/answers/how-to-undo-git-commit-amend/
- Stack Overflow: Amending a Commit That's Already Been Pushed — https://stackoverflow.com/questions/18549040
- GeeksforGeeks: How to Amend a Commit Without Changing Commit Message — https://origin.geeksforgeeks.org
- man.he.net: git-commit Manual Page — https://www.man.he.net/man1/git-commit
- manpages.org: git-commit Manual Page — https://manpages.org/git-commit
- GitLab: Unreachable Objects — https://gitlab.sib.swiss
- Atlassian: Git Prune Tutorial — https://www.atlassian.com/git/tutorials/git-prune