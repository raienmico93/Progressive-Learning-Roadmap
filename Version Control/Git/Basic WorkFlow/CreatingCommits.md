# Git Creating Commits: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Creating a commit is the act of permanently recording the current state of the staging area (index) as an immutable snapshot in the Git object database, along with metadata identifying the author, committer, timestamp, and a descriptive message.

**Technical Definition**
A commit is a Git object of type `commit` stored in the object database (`.git/objects/`). It contains a reference to a tree object (the project snapshot), references to zero or more parent commits (forming a Directed Acyclic Graph), author and committer metadata, and a commit message. The commit is identified by a cryptographic hash (SHA-1: 40 hex characters; SHA-256: 64 hex characters) computed over its content plus a header. `git commit` reads the index, creates a tree object, constructs a commit object, writes it to the database, and updates the current branch reference (pointed to by `HEAD`) to the new commit.

**Beginner-Friendly Explanation**
A commit is like taking a permanent snapshot of your project at a specific moment. Before committing, you place the changes you want to save into a “staging area” (using `git add`). Then `git commit` saves that snapshot into Git's history book, along with a message explaining what you did. Each commit has a unique ID (a long hash), so you can always find your way back to any previous state of the project.

### Key Characteristics

- **Immutable**: Once created, a commit cannot be modified. Amending creates a new commit with a new hash.
- **Content-Addressable**: The commit's hash is derived from its content, ensuring integrity.
- **DAG Structure**: Commits form a Directed Acyclic Graph through parent pointers.
- **Two Identities**: Each commit records both the *author* (who wrote the code) and the *committer* (who created the commit).
- **Atomic**: A commit is all-or-nothing; it records the exact state of the index at commit time.
- **Local Operation**: Committing is a local operation; no network access is required.

### Prerequisites

- Git installed (version 2.28+ recommended for `init.defaultBranch`; version 2.23+ for `git restore`).
- A Git repository initialised or cloned (`git init` or `git clone`).
- Files staged in the index (`git add`).
- Configured user identity (`user.name` and `user.email`), set via `git config --global user.name "Your Name"` and `git config --global user.email "you@example.com"`.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Automated commit generation in pipelines.
- **Software Configuration Management (SCM)** : Auditing, traceability, and compliance.
- **Collaborative Development**: Code review, pull requests, and release management.

### Core Concepts / Features

1. **git commit** — Recording staged changes as a permanent snapshot.
2. **Commit Messages** — The 50/72 rule, imperative mood, and Conventional Commits.
3. **Commit Structure** — Author, committer, tree, parent pointers, and hashes.
4. **Good Commit Practices** — Clarity, consistency, and meaningful history.
5. **Atomic Commits** — Single, isolated logical changes.
6. **Commit Granularity** — Deciding what belongs in one commit.
7. **Modifying the Most Recent Commit** — `git commit --amend` and `--no-edit`.

---

## 1. git commit

### Definitions

**Core Definition**
`git commit` is the command that records the contents of the staging area (index) as a new commit in the repository.

**Technical Definition**
`git commit` reads the index, creates a tree object representing the staged snapshot, constructs a commit object with a reference to that tree, one or more parent commits (default: HEAD), author and committer metadata, and the commit message, writes the objects to the object database, and updates the current branch reference (or HEAD in detached state) to point to the new commit. The new commit becomes the direct child of HEAD, usually the tip of the current branch.

**Beginner-Friendly Explanation**
`git commit` is how you save your work. After staging changes with `git add`, running `git commit` takes a permanent snapshot of everything in the staging area. You can write a short message with `-m "message"`, or run `git commit` without arguments to open an editor for a longer message. The commit is saved in Git's history and can always be retrieved later.

### Purposes

- To permanently record a snapshot of the project in the repository history.
- To create a new node in the commit DAG, advancing the branch pointer.
- To provide a message that explains the “why” behind the change.
- To enable reverting, comparing, and reviewing changes over time.
- To establish a checkpoint that can be referenced by hash, tag, or branch.

### Syntax Rules and Structure

**Complete General Syntax**

```
git commit [-a | --all] [-m <msg> | --message=<msg>] [--amend]
           [--no-edit] [--author=<author>] [--date=<date>]
           [-F <file> | --file=<file>] [--squash=<commit>]
           [--fixup=<commit>] [--reset-author] [--allow-empty]
           [-S[<keyid>]] [--no-verify] [-v | --verbose]
           [--] [<pathspec>...]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `-m <msg>` / `--message=<msg>` | Use the given message as the commit message. |
| `-a` / `--all` | Automatically stage modified and deleted tracked files, then commit. |
| `--amend` | Replace the most recent commit with a new commit. |
| `--no-edit` | Use the existing commit message without opening an editor (with `--amend`). |
| `-F <file>` / `--file=<file>` | Read the commit message from a file. |
| `--author=<author>` | Override the author. |
| `--allow-empty` | Allow a commit with no changes. |
| `-S[<keyid>]` | GPG-sign the commit. |
| `-v` / `--verbose` | Show the diff in the editor. |
| `--no-verify` | Bypass pre-commit and commit-msg hooks. |

**Syntax Rules**

- The commit message can be provided inline with `-m`, from a file with `-F`, or via an editor (if neither `-m` nor `-F` is given).
- Multiple `-m` options can be used; each becomes a separate paragraph separated by a blank line.
- `-a` only stages files that are already tracked; new (untracked) files must still be added with `git add`.
- `--amend` replaces the previous commit; the old commit becomes unreachable (but remains in the reflog for recovery).
- `--no-edit` is typically used with `--amend` to keep the existing message without opening an editor.

**Constraints and Limitations**

- **Identity required**: Git must know the author's name and email. Configure with `git config --global user.name` and `git config --global user.email`. If not configured, Git may use a system-derived default or fail.
- **Empty commits**: By default, Git refuses to create a commit with no changes. Use `--allow-empty` to override.
- **Shallow clones**: Committing in a shallow clone works, but history is truncated.
- **Detached HEAD**: Commits made in detached HEAD state are not associated with any branch and can be lost. Use `git switch -c <branch>` to create a branch.
- **Version-specific**: `--fixup=amend:` requires Git 2.32+; `--reset-author` requires Git 1.6.0+.

### Annotated Code Examples

#### Example 1: Inline Commit with `-m`

```bash
# Create a repository and stage a file
mkdir commit-demo && cd commit-demo && git init
echo "Hello, Git!" > hello.txt
git add hello.txt

# Commit with an inline message
git commit -m "Add hello.txt with initial greeting"
```

**Expected Output:**
```
[master (root-commit) 4d5e6f7] Add hello.txt with initial greeting
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
```

**Why this output occurs:** `git commit -m` reads the index (containing `hello.txt`), creates a blob object for the file, a tree object for the directory, and a commit object. The branch `master` now points to the new commit `4d5e6f7`. The output shows the abbreviated commit hash, the branch name, the commit message, and a summary of changes.

#### Example 2: Editor-Based Multi-Line Commit

```bash
# Modify the file and stage it
echo "Second line" >> hello.txt
git add hello.txt

# Commit without -m: opens the configured editor
git commit
```

**Editor opens (e.g., Vim):**
```
Add second line to hello.txt

This line provides more detail about the change.
It explains why the second line was added and
what impact it has on the project.
```

**Expected Output:**
```
[master 1a2b3c4] Add second line to hello.txt
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** Without `-m`, Git opens the configured editor (`$EDITOR` or `core.editor`). The first line becomes the subject; the blank line separates it from the body. Git strips comment lines starting with `#` and uses the rest as the message.

#### Example 3: Skipping Staging with `-a`

```bash
# Modify a tracked file without staging
echo "Third line" >> hello.txt

# Commit with -a (auto-stage tracked files)
git commit -a -m "Add third line using -a"
```

**Expected Output:**
```
[master 9a0b1c2] Add third line using -a
 1 file changed, 1 insertion(+)
```

**Why this output occurs:** `-a` automatically stages modifications and deletions to already-tracked files before committing. `hello.txt` is tracked, so its modification is staged and committed in one step. Untracked files are not affected.

### Real-World Cases

- **Feature development**: A developer stages and commits a new feature with a descriptive message.
- **Bug fixes**: A developer commits a fix with a message referencing the issue number (e.g., `fix: resolve null pointer in login #123`).
- **CI/CD automation**: A CI script commits generated artifacts with `git commit -m "chore: update build artifacts [skip ci]"`.
- **Documentation updates**: A technical writer commits changes to `README.md` with `git commit -m "docs: update installation instructions"`.

### References

- Git Documentation: git-commit Manual Page — https://git-scm.com/docs/git-commit
- Git Documentation: gittutorial – Committing Changes — https://git-scm.com/docs/gittutorial
- Pro Git Book: Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

---

## 2. Commit Messages

### Definitions

**Core Definition**
A commit message is a human-readable description of the changes contained in a commit, consisting of a short subject line and an optional body.

**Technical Definition**
The commit message is stored as part of the commit object in the object database. By convention, the first line (up to the first blank line) is the *subject* or *title*, and the remainder is the *body*. Git tools (e.g., `git log --oneline`, `git format-patch`) treat the subject line as the primary identifier of the commit. The message is subject to cleanup rules (comment stripping, whitespace trimming) controlled by `commit.cleanup` and `--cleanup=<mode>`.

**Beginner-Friendly Explanation**
A commit message is a note to your future self (and your teammates) explaining what you changed and why. A good message has a short summary (like a headline) and, if needed, a longer explanation. Writing clear messages makes it much easier to understand the project's history later.

### Purposes

- To explain **why** a change was made, not just **what** changed.
- To provide context for reviewers and future maintainers.
- To enable automated tools (changelog generation, semantic versioning).
- To make `git log` and `git blame` more informative.
- To facilitate debugging and bisecting by identifying relevant changes.

### Syntax Rules and Structure

**The 50/72 Rule**

| Rule | Description |
|------|-------------|
| **50-character subject line** | The first line should be no more than 50 characters (max 72). This ensures it fits in terminal views and GitHub's commit list without truncation. |
| **72-character body lines** | Each line in the body should be wrapped at 72 characters to ensure readability in terminals and code editors. |
| **Blank line separator** | A single blank line must separate the subject from the body. This is critical for tools that treat the first line as the title. |

**Imperative Mood**
The subject line should be written in the imperative mood, as if giving a command: “Fix bug” not “Fixed bug” or “Fixes bug.” This matches the convention used by Git itself for merge and revert commits. The imperative mood completes the sentence: “If applied, this commit will _____________.”

**Conventional Commits**
The Conventional Commits specification provides a lightweight convention for commit messages:
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

| Element | Description |
|---------|-------------|
| `type` | Required. A noun such as `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. |
| `scope` | Optional. A noun in parentheses indicating the affected area (e.g., `feat(parser)`). |
| `description` | Required. A short, imperative description. |
| `body` | Optional. Additional explanatory text. |
| `footer` | Optional. Trailers such as `BREAKING CHANGE:`, `Fixes #123`, `Co-authored-by:`. |

**Commit Types and Their Meanings**

| Type | Meaning | Semantic Versioning Impact |
|------|---------|---------------------------|
| `feat` | A new feature | MINOR |
| `fix` | A bug fix | PATCH |
| `docs` | Documentation only | None |
| `style` | Formatting, missing semicolons, etc. | None |
| `refactor` | Code restructuring without behaviour change | None |
| `perf` | Performance improvement | None |
| `test` | Adding or fixing tests | None |
| `build` | Build system or dependency changes | None |
| `ci` | CI configuration changes | None |
| `chore` | Maintenance tasks | None |
| `revert` | Reverting a previous commit | None |

A `BREAKING CHANGE:` footer or a `!` after the type/scope indicates a MAJOR version change.

**Syntax Rules**

- The subject line should not end with a period.
- The subject line should begin with a capital letter (unless using Conventional Commits, which uses lowercase types).
- Use the body to explain *what* and *why*, not *how* (the diff shows how).
- Reference issues and pull requests in the footer (e.g., `Closes #42`).
- Use `BREAKING CHANGE:` (uppercase) as a footer to indicate incompatible API changes.

**Constraints and Limitations**

- **Character limits are guidelines**: The 50/72 rule is a convention, not enforced by Git. Some teams use different limits.
- **Case sensitivity**: Conventional Commits types are case-sensitive; `BREAKING CHANGE` must be uppercase.
- **Automation dependency**: Tools like `commitlint` and `semantic-release` depend on strict adherence to the specification.
- **Version-specific**: Conventional Commits is a community specification, not a Git feature. It requires team-wide adoption.

### Annotated Code Examples

#### Example 1: Good vs. Poor Commit Messages

```bash
# POOR: Vague, no context
git commit -m "fixed stuff"

# GOOD: Imperative, specific, explains why
git commit -m "Fix null pointer exception in user login

The login handler did not check for a null response from
the authentication service. Added a null check and an
appropriate error message.

Fixes #1234"
```

**Why the good message is better:** The subject line is concise and imperative. The body explains the *why* (null check missing) and references the issue. Future maintainers can understand the change without reading the diff.

#### Example 2: Conventional Commits

```bash
# Feature with scope
git commit -m "feat(auth): add JWT token refresh"

# Bug fix with issue reference
git commit -m "fix(ui): correct button alignment

Closes #456"

# Breaking change with footer
git commit -m "feat!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6."
```

**Expected Output:**
```
[master 3d4e5f6] feat(auth): add JWT token refresh
[master 7a8b9c0] fix(ui): correct button alignment
[master 1d2e3f4] feat!: drop support for Node 6
```

**Why these messages work:** Each follows the Conventional Commits format, making them machine-readable for automated changelog generation and semantic versioning. The `!` and `BREAKING CHANGE:` footer signal a major version bump.

### Real-World Cases

- **Open-source projects**: Kubernetes, Angular, and many others enforce Conventional Commits for automated release notes.
- **Enterprise teams**: Commit messages referencing Jira ticket numbers (e.g., `PROJ-123: implement caching layer`).
- **Automated changelogs**: Tools like `standard-version` and `semantic-release` parse Conventional Commits to generate `CHANGELOG.md`.
- **Code review**: Reviewers use commit messages to understand the intent behind a change before reading the diff.

### References

- Conventional Commits Specification — https://www.conventionalcommits.org/en/v1.0.0/
- The 50/72 Rule of Git — https://raw.githubusercontent.com/ardalis/DevIQ-gatsby/main/src/docs/practices/50-72-rule.md
- Udacity Git Commit Message Style Guide — https://udacity.github.io/git-styleguide/
- Git Documentation: git-commit (Discussion) — https://git-scm.com/docs/git-commit#_discussion

---

## 3. Commit Structure (Author, Committer, Tree, Parent Pointers, and Hashes)

### Definitions

**Core Definition**
A commit object is a data structure in Git's object database that encapsulates a project snapshot (tree), its lineage (parent pointers), authorship metadata, and a message, all identified by a cryptographic hash.

**Technical Definition**
A commit object is stored as a text file in `.git/objects/` (compressed) with the following format:
```
tree <SHA-1 of tree object>
parent <SHA-1 of parent commit>   (zero or more)
author <name> <email> <timestamp> <timezone>
committer <name> <email> <timestamp> <timezone>

<commit message>
```
The SHA-1 (or SHA-256) hash of this content (including the header `commit <size>\0`) is the commit's unique identifier. The `tree` points to the root directory snapshot; `parent` pointers link to previous commits, forming a DAG; `author` is the person who wrote the change; `committer` is the person who applied it (they differ in rebases, patch applications, etc.).

**Beginner-Friendly Explanation**
Every commit is a small file that says: “Here is a snapshot of the project (tree), here is the commit that came before me (parent), here is who wrote the code (author), who made the commit (committer), and here is my message.” Because this file's content is hashed, the commit gets a unique ID. If you change anything, the ID changes.

### Purposes

- To link a project snapshot (tree) to its history (parents) in a DAG.
- To attribute changes to authors and committers.
- To provide a unique, verifiable identifier (hash) for each commit.
- To enable Git operations like diff, merge, rebase, and bisect.
- To ensure data integrity through cryptographic hashing.

### Syntax Rules and Structure

**Commit Object Format (Line by Line)**

| Line | Description |
|------|-------------|
| `tree <hash>` | SHA-1 of the root tree object representing the snapshot. |
| `parent <hash>` | SHA-1 of a parent commit. Zero for initial commit, one for normal commit, two or more for merge commit. |
| `author <name> <email> <timestamp> <tz>` | The original author of the change. |
| `committer <name> <email> <timestamp> <tz>` | The person who created the commit. |
| (blank line) | Separates headers from the message. |
| `<message>` | The commit message text. |

**Hash Computation**
The commit hash is computed as:
```
SHA1("commit " + <content-length> + "\0" + <content>)
```
The content includes everything from the `tree` line through the commit message. Any change to any field produces a different hash.

**Parent Pointer Rules**

| Commit Type | Parent Count |
|-------------|--------------|
| Initial (root) commit | 0 |
| Normal commit | 1 |
| Merge commit | 2 or more |

**Syntax Rules**

- The `tree` line is mandatory and always comes first.
- `parent` lines are optional but must come before `author`.
- `author` and `committer` lines are mandatory.
- The blank line between headers and message is mandatory.
- The commit message is free-form text.

**Constraints and Limitations**

- **Immutability**: The commit object cannot be changed after creation. Amending creates a new object.
- **Hash algorithm**: SHA-1 is the default and is cryptographically broken. SHA-256 support is experimental and non-interoperable as of Git 2.49.
- **Author vs. committer**: They are usually the same, but differ in rebases (`git rebase` sets committer to the rebaser, author remains original), `git commit --amend --reset-author`, and patch application (`git am`).
- **Timezone**: The timestamp includes a timezone offset, preserving local time.

### Annotated Code Examples

#### Example 1: Inspecting a Commit Object

```bash
# Create a commit
git init structure-demo && cd structure-demo
echo "content" > file.txt
git add file.txt
git commit -m "Initial commit"

# Inspect the raw commit object
git cat-file -p HEAD
```

**Expected Output:**
```
tree 2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c
author User <user@example.com> 1700000000 +0000
committer User <user@example.com> 1700000000 +0000

Initial commit
```

**Why this output occurs:** `git cat-file -p HEAD` reads the commit object pointed to by HEAD and pretty-prints it. There is no `parent` line because this is the initial (root) commit. The `tree` line points to the snapshot. Author and committer are the same because no rebase or patch application was involved.

#### Example 2: Commit with Parent (Normal Commit)

```bash
# Make a second commit
echo "more" >> file.txt
git add file.txt
git commit -m "Second commit"

# Inspect the commit
git cat-file -p HEAD
```

**Expected Output:**
```
tree 3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d
parent 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
author User <user@example.com> 1700000001 +0000
committer User <user@example.com> 1700000001 +0000

Second commit
```

**Why this output occurs:** The `parent` line points to the first commit. This is what forms the DAG. The `tree` line points to a new tree object because the content changed. The hash of this commit is different from the first because it includes the parent line and a different tree and timestamp.

#### Example 3: Merge Commit with Two Parents

```bash
# Create a branch, commit, and merge
git checkout -b feature
echo "feature" > feature.txt
git add feature.txt
git commit -m "Add feature"

git checkout master
echo "master" > master.txt
git add master.txt
git commit -m "Add master file"

git merge feature --no-edit

# Inspect the merge commit
git cat-file -p HEAD
```

**Expected Output:**
```
tree 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
parent 2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c
parent 5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f
author User <user@example.com> 1700000002 +0000
committer User <user@example.com> 1700000002 +0000

Merge branch 'feature'
```

**Why this output occurs:** A merge commit has **two** `parent` lines: one for each branch tip being merged. This is how the DAG records the joining of two histories. The commit message is auto-generated by Git but can be edited.

### Real-World Cases

- **Rebase**: `git rebase` creates new commits with new hashes, same author, but different committer and parent pointers.
- **Cherry-pick**: `git cherry-pick` applies a commit's changes onto another branch, creating a new commit with the original author but a new committer and parent.
- **Signed commits**: `git commit -S` adds a GPG signature to the commit object, which is stored in the `gpgsig` header.
- **Auditing**: Tools like `git log --format="%H %an %ae %cn %ce"` extract author and committer information for compliance reports.

### References

- Git Documentation: Git Internals – Git Objects — https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
- Git Documentation: git-commit-tree Manual Page — https://git-scm.com/docs/git-commit-tree
- MIT Understanding Git (PDF) — http://web.mit.edu/cluedumps/slides/understanding-git-2008.pdf
- Git Documentation: hash-function-transition — https://git-scm.com/docs/hash-function-transition

---

## 4. Good Commit Practices

### Definitions

**Core Definition**
Good commit practices are guidelines and conventions that produce a clean, navigable, and useful project history.

**Technical Definition**
Good commit practices encompass message formatting (50/72 rule, imperative mood, Conventional Commits), atomicity (one logical change per commit), granularity (balancing size and cohesion), and consistency (team-wide conventions). They are not enforced by Git but are essential for collaborative development, code review, and long-term maintenance.

**Beginner-Friendly Explanation**
Good commit practices are habits that make your project's history easy to understand. They include writing clear messages, making small focused commits, and following your team's conventions. These habits save time when debugging, reviewing code, or onboarding new developers.

### Purposes

- To make the project history easy to understand and navigate.
- To enable effective code review and collaboration.
- To support automated tooling (changelogs, semantic versioning).
- To facilitate debugging through `git bisect` and `git blame`.
- To provide a reliable audit trail for compliance.

### Syntax Rules and Structure

**Core Principles**

| Principle | Description |
|-----------|-------------|
| **Atomicity** | Each commit should contain one logical change. |
| **Clarity** | The commit message should explain *what* and *why*, not *how*. |
| **Consistency** | Follow the same format and conventions across the project. |
| **Completeness** | Each commit should leave the codebase in a working state. |
| **Reviewability** | Commits should be small enough to review in one sitting. |

**Checklist Before Committing**

1. Run `git status` and `git diff --staged` to review what will be committed.
2. Ensure all tests pass.
3. Write a clear, imperative commit message.
4. Reference relevant issues or tickets.
5. Split unrelated changes into separate commits (use `git add -p`).
6. Avoid committing generated files, secrets, or large binaries.

**Constraints and Limitations**

- **Team conventions vary**: Some teams prefer Conventional Commits; others use ticket IDs; others prefer free-form. Consistency within a project is more important than universal rules.
- **Tooling dependency**: Automated tools (commitlint, semantic-release) require strict adherence.
- **Cultural differences**: The imperative mood convention is English-centric; teams working in other languages may adapt.

### Annotated Code Examples

#### Example 1: A Well-Structured Commit

```bash
# Stage only the relevant changes
git add src/auth.js
git diff --staged   # Review

# Commit with a Conventional Commit message
git commit -m "feat(auth): add JWT token refresh

Implement automatic token refresh when the access token
expires. The refresh token is stored in an HTTP-only cookie
for security.

Closes #789"
```

**Expected Output:**
```
[main 4d5e6f7] feat(auth): add JWT token refresh
 1 file changed, 45 insertions(+), 2 deletions(-)
```

**Why this is good:** The subject follows Conventional Commits (`feat(auth):`). The body explains the *why* and *what*. The footer links the issue. The commit is focused on one logical change.

#### Example 2: Reviewing Before Committing

```bash
# Check what's staged
git diff --staged --stat

# Check for whitespace errors
git diff --staged --check

# Commit
git commit -m "fix: resolve memory leak in connection pool

The connection pool was not releasing connections on timeout.
Added a finally block to ensure release.

Fixes #321"
```

**Expected Output:**
```
 src/pool.js | 8 +++++---
 1 file changed, 5 insertions(+), 3 deletions(-)
[main 7a8b9c0] fix: resolve memory leak in connection pool
 1 file changed, 5 insertions(+), 3 deletions(-)
```

**Why this is good:** The developer reviews staged changes, checks for whitespace errors, and writes a message that explains the root cause and the fix.

### Real-World Cases

- **Code review**: Reviewers can understand a change from the commit message alone, reducing review time.
- **Debugging**: `git bisect` relies on each commit being a working state so that the breaking commit can be identified.
- **Release notes**: Conventional Commits enable automatic generation of changelogs.
- **Onboarding**: New team members read `git log` to understand project evolution.

### References

- Conventional Commits Specification — https://www.conventionalcommits.org/en/v1.0.0/
- Pro Git Book: Contributing to a Project – Commit Guidelines — https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project
- Git Documentation: gittutorial (A note on commit messages) — https://git-scm.com/docs/gittutorial

---

## 5. Atomic Commits (Single, Isolated Logical Changes)

### Definitions

**Core Definition**
An atomic commit is a commit that contains exactly one logical change, making it self-contained, revertible, and reviewable in isolation.

**Technical Definition**
Atomicity in commits means that each commit represents a single, complete, and coherent unit of work. It should be possible to revert the commit without affecting unrelated functionality. An atomic commit may span multiple files if they are all part of the same logical change, but it should not mix unrelated concerns (e.g., a feature addition and a formatting cleanup).

**Beginner-Friendly Explanation**
An atomic commit does one thing and does it completely. If you fix a bug and also rename a variable, that's two commits, not one. Atomic commits make it easy to undo a specific change without accidentally undoing something else.

### Purposes

- To enable clean reverts of individual changes.
- To support `git bisect` for identifying bug-introducing commits.
- To make code review easier by isolating concerns.
- To facilitate cherry-picking changes across branches.
- To produce a meaningful, navigable project history.

### Syntax Rules and Structure

**Characteristics of an Atomic Commit**

| Characteristic | Description |
|----------------|-------------|
| **Single logical change** | One feature, one bug fix, one refactor, one doc update. |
| **Self-contained** | The change makes sense on its own. |
| **Revertible** | Reverting the commit does not break unrelated functionality. |
| **Testable** | Tests should pass after applying the commit. |
| **Coherent** | The commit message describes one thing, not a list of unrelated changes. |

**Strategies for Creating Atomic Commits**

| Strategy | Description |
|----------|-------------|
| **Interactive staging** | `git add -p` to stage specific hunks. |
| **File-based splitting** | Stage entire files that belong to one logical change. |
| **Commit before context switch** | Commit work before starting a new task. |
| **Separate tests** | Add tests in their own commit before functionality changes. |
| **Separate formatting** | Keep formatting changes in their own commit. |

**Constraints and Limitations**

- **Practicality**: Sometimes changes are interdependent and cannot be split without breaking the build. In such cases, a larger commit may be necessary.
- **Overhead**: Very fine-grained commits can create noise. Balance is needed.
- **Merge conflicts**: Splitting changes may increase merge conflict complexity.
- **Team culture**: Some teams prefer squash-merging, which collapses atomic commits into one.

### Annotated Code Examples

#### Example 1: Splitting a Mixed Change into Atomic Commits

```bash
# A developer has made two unrelated changes:
# 1. Fixed a bug in calculateTotal()
# 2. Renamed variable 'x' to 'count' throughout the file

# Stage only the bug fix
git add -p app.js
# Select only the hunks related to the bug fix

git commit -m "fix: correct rounding error in calculateTotal"

# Stage the variable rename
git add app.js
git commit -m "refactor: rename variable x to count for clarity"
```

**Expected Output:**
```
[main 1a2b3c4] fix: correct rounding error in calculateTotal
[main 5d6e7f8] refactor: rename variable x to count for clarity
```

**Why this is good:** The bug fix and the refactor are separate commits. If the bug fix needs to be reverted, the rename remains intact. Each commit is atomic.

#### Example 2: Adding Tests as a Separate Commit

```bash
# First, add a failing test for the new behaviour
git add tests/test_auth.js
git commit -m "test: add failing test for token refresh"

# Then, implement the feature
git add src/auth.js
git commit -m "feat(auth): implement JWT token refresh"
```

**Expected Output:**
```
[main 3d4e5f6] test: add failing test for token refresh
[main 7a8b9c0] feat(auth): implement JWT token refresh
```

**Why this is good:** The test commit demonstrates the expected behaviour before the implementation. Reviewers can see that the test fails before the feature and passes after. This is a recommended practice in projects like Cargo (Rust).

### Real-World Cases

- **Bug fix + refactor**: A developer fixes a bug and also cleans up code. Atomic commits separate these so the bug fix can be backported without the refactor.
- **Feature + tests**: Tests are committed first (failing), then the implementation. This proves the tests are valid.
- **Dependency upgrade**: Upgrading a dependency is its own commit, separate from any code changes that use the new version.
- **Formatting changes**: Running a formatter (e.g., Prettier) is committed separately from functional changes to keep diffs clean.

### References

- Cargo Contribution Guide: Atomic Commits — https://rust.googlesource.com/cargo/
- Atomic Commits Explained — https://www.phparch.com/2025/06/atomic-commits-explained/
- Pro Git Book: Rewriting History — https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History

---

## 6. Commit Granularity

### Definitions

**Core Definition**
Commit granularity refers to the level of detail at which changes are divided into commits—how much is included in a single commit.

**Technical Definition**
Commit granularity is the trade-off between smaller, more focused commits and larger, more comprehensive commits. Fine-grained commits isolate individual changes, while coarse-grained commits group related changes together. The optimal granularity depends on the project, team workflow, and review process. Git's own commit discipline follows the principle of “each commit is a minimal coherent idea.”

**Beginner-Friendly Explanation**
Commit granularity is about deciding how big each commit should be. Should you commit every tiny change separately, or group related changes together? The sweet spot is usually “one logical change per commit”—not too small, not too big.

### Purposes

- To balance reviewability with practicality.
- To optimise the usefulness of `git bisect`, `git revert`, and `git cherry-pick`.
- To align with team workflow (e.g., squash-merge vs. merge-commit).
- To avoid both commit noise and monolithic commits.
- To produce a history that is both informative and manageable.

### Syntax Rules and Structure

**Granularity Spectrum**

| Granularity | Description | Pros | Cons |
|-------------|-------------|------|------|
| **Very fine** | One line per commit | Precise revert, easy bisect | Noisy history, many commits |
| **Fine** | One logical change per commit | Clean revert, good bisect | Requires discipline |
| **Medium** | A few related changes | Fewer commits | Harder to revert individual parts |
| **Coarse** | Many unrelated changes | Fewer commits | Hard to review, hard to revert |
| **Monolithic** | Entire feature in one commit | Simple | Unreviewable, un-revertible |

**Guidelines for Choosing Granularity**

| Guideline | Description |
|-----------|-------------|
| **One logical change** | If you need “and” to describe it, it's probably two commits. |
| **Revertability** | Can you revert this commit without breaking other things? |
| **Reviewability** | Can a reviewer understand this in one sitting? |
| **Testability** | Do tests pass after this commit? |
| **Cohesion** | Do all the changes serve one purpose? |
| **File count** | If a commit touches more than 3 files in different domains, it may be too large. |

**Constraints and Limitations**

- **Interdependencies**: Some changes cannot be separated without breaking the build (e.g., an API change and its consumers).
- **Team workflow**: Squash-merge workflows collapse atomic commits into one, reducing the value of fine granularity on feature branches.
- **Review overhead**: Too many tiny commits can overwhelm reviewers.
- **Tooling**: Some CI systems trigger a build per commit, making fine granularity expensive.

### Annotated Code Examples

#### Example 1: Too Coarse vs. Just Right

```bash
# TOO COARSE: Mixing bug fix, refactor, and docs
git commit -m "Various fixes and cleanup"
# (Includes: bug fix in auth.js, rename in utils.js, README update)

# JUST RIGHT: Three atomic commits
git add src/auth.js
git commit -m "fix: correct null check in login handler"

git add src/utils.js
git commit -m "refactor: extract validateEmail into helper"

git add README.md
git commit -m "docs: update authentication section"
```

**Expected Output:**
```
[main 1a2b3c4] fix: correct null check in login handler
[main 5d6e7f8] refactor: extract validateEmail into helper
[main 9a0b1c2] docs: update authentication section
```

**Why the second approach is better:** Each commit is reviewable, revertible, and has a clear purpose. The “Various fixes” commit is a black box that cannot be reverted without affecting unrelated changes.

#### Example 2: Using Interactive Staging for Fine Granularity

```bash
# A single file contains a bug fix and a formatting change
git add -p app.js
# Stage only the bug fix hunks, commit
git commit -m "fix: handle edge case in parseInput"

# Stage the formatting hunks, commit
git add app.js
git commit -m "style: apply consistent indentation"
```

**Expected Output:**
```
[main 3d4e5f6] fix: handle edge case in parseInput
[main 7a8b9c0] style: apply consistent indentation
```

**Why this works:** `git add -p` allows fine-grained staging within a single file. The bug fix and formatting are separate commits, each with a clear purpose.

### Real-World Cases

- **Large teams**: Prefer fine granularity for easier code review and parallel development.
- **Solo projects**: May prefer coarse granularity for speed and simplicity.
- **CI/CD**: Fine granularity allows precise rollback of individual changes.
- **Open source**: Contributors are often asked to split large PRs into atomic commits.

### References

- Cargo Contribution Guide: Atomic Commits — https://rust.googlesource.com/cargo/
- Pro Git Book: Rewriting History — https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
- Git Documentation: git-add (Interactive Mode) — https://git-scm.com/docs/git-add#_interactive_mode

---

## 7. Modifying the Most Recent Commit (`git commit --amend` and `--no-edit`)

### Definitions

**Core Definition**
`git commit --amend` is a command that replaces the most recent commit with a new commit, allowing the developer to modify the commit message, add staged changes, or both.

**Technical Definition**
`git commit --amend` creates a new commit object that replaces the current HEAD commit. The new commit has the same parent(s) as the original, but may have a different tree (if additional changes are staged), a different message, or both. The original commit becomes unreachable from the branch but remains in the reflog for recovery. The `--no-edit` option suppresses the editor and keeps the existing commit message unchanged.

**Beginner-Friendly Explanation**
`git commit --amend` lets you fix the last commit. Forgot to add a file? Stage it and run `git commit --amend --no-edit`. Made a typo in the message? Run `git commit --amend` to edit it. It's like using “undo” on your last commit and redoing it correctly.

### Purposes

- To fix a typo or improve the wording of the last commit message.
- To add forgotten files to the last commit.
- To remove files accidentally included in the last commit.
- To change the author of the last commit (`--reset-author`).
- To GPG-sign a commit that was not signed initially.

### Syntax Rules and Structure

**Complete General Syntax**

```
git commit --amend [--no-edit] [-m <msg>] [--author=<author>]
                  [--reset-author] [--allow-empty] [-S[<keyid>]]
                  [--] [<pathspec>...]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `--amend` | Replace the most recent commit. |
| `--no-edit` | Use the existing commit message without opening an editor. |
| `-m <msg>` | Replace the commit message with the given message. |
| `--reset-author` | Reset the author to the current user (and update the author timestamp). |
| `--allow-empty` | Allow amending with no changes. |
| `-S[<keyid>]` | GPG-sign the amended commit. |

**Syntax Rules**

- `--amend` alone opens the editor with the existing message for editing.
- `--amend --no-edit` keeps the existing message and does not open an editor.
- `--amend -m "new message"` replaces the message without opening an editor.
- Staged changes are included in the amended commit. Unstaged changes are not.
- The original commit is replaced; its hash changes, and it becomes unreachable from the branch.

**Constraints and Limitations**

- **History rewriting**: `--amend` rewrites history. Do **not** amend commits that have been pushed to a shared remote, as this forces other developers to rebase.
- **Recovery**: The original commit can be recovered via `git reflog` and `git reset --hard <old-hash>`.
- **Parent unchanged**: The amended commit has the same parent(s) as the original. To change parents, use `git rebase`.
- **Detached HEAD**: Amending in detached HEAD works but the amended commit has no branch reference.
- **Version-specific**: `--amend` has been available since Git 1.0; `--no-edit` since Git 1.6.0.

### Annotated Code Examples

#### Example 1: Amending a Commit Message

```bash
# Create a commit with a typo
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

**Why this output occurs:** `--no-edit` keeps the existing message (“Add file1”). The staged `file2.txt` is included in the amended commit. The commit now contains both files. Note that the message now says “Add file1” but the commit contains two files—the developer might want to amend the message to be more accurate.

#### Example 3: Recovering an Amended Commit from the Reflog

```bash
# Amend a commit
git commit --amend -m "New message"

# Realise the original message is needed
git reflog
# Output:
# 1a2b3c4 (HEAD -> master) HEAD@{0}: commit (amend): New message
# 5d6e7f8 HEAD@{1}: commit: Original message

# Recover the original commit
git reset --hard HEAD@{1}
```

**Expected Output:**
```
HEAD is now at 5d6e7f8 Original message
```

**Why this output occurs:** The reflog records both the original commit (`HEAD@{1}`) and the amended commit (`HEAD@{0}`). `git reset --hard HEAD@{1}` restores the branch to the original commit. The amended commit remains in the reflog until it expires.

### Real-World Cases

- **Fixing a typo in the commit message**: A developer runs `git commit --amend` to correct a misspelling before pushing.
- **Adding a forgotten file**: A developer stages a forgotten file and runs `git commit --amend --no-edit` to include it.
- **Changing the author**: A developer who committed with the wrong email runs `git commit --amend --reset-author` after updating `user.email`.
- **Signing a commit**: A developer who forgot to GPG-sign runs `git commit --amend -S --no-edit` to add a signature.
- **Interactive rebase**: `git rebase -i` uses `--amend` internally for the `edit` and `reword` commands.

### References

- Git Documentation: git-commit (--amend) — https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---amend
- Git Documentation: git-commit (--no-edit) — https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---no-edit
- Pro Git Book: Undoing Things — https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things
- Pro Git Book: Git Tools – Rewriting History — https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History

---

## References

- Git Documentation: git-commit Manual Page — https://git-scm.com/docs/git-commit
- Git Documentation: gittutorial — https://git-scm.com/docs/gittutorial
- Git Documentation: git-commit-tree Manual Page — https://git-scm.com/docs/git-commit-tree
- Git Documentation: Git Internals – Git Objects — https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
- Git Documentation: hash-function-transition — https://git-scm.com/docs/hash-function-transition
- Conventional Commits Specification — https://www.conventionalcommits.org/en/v1.0.0/
- The 50/72 Rule of Git — https://raw.githubusercontent.com/ardalis/DevIQ-gatsby/main/src/docs/practices/50-72-rule.md
- Udacity Git Commit Message Style Guide — https://udacity.github.io/git-styleguide/
- Pro Git Book: Recording Changes to the Repository — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository
- Pro Git Book: Undoing Things — https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things
- Pro Git Book: Rewriting History — https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
- Cargo Contribution Guide: Atomic Commits — https://rust.googlesource.com/cargo/
- MIT Understanding Git (PDF) — http://web.mit.edu/cluedumps/slides/understanding-git-2008.pdf
- Atomic Commits Explained — https://www.phparch.com/2025/06/atomic-commits-explained/