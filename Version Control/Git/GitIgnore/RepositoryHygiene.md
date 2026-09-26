# Git Repository Hygiene: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Git repository hygiene is the set of practices and tools used to keep a Git repository clean, secure, and efficient by removing untracked clutter, preventing accidental commits of generated or sensitive files, and rewriting history to eliminate problematic content.

**Technical Definition**
Repository hygiene encompasses: (1) working-tree cleanup via `git clean`; (2) preventing the tracking of generated artifacts and secrets through `.gitignore` and proactive scanning; (3) removing accidentally tracked files from the index with `git rm --cached`; (4) maintaining a semantically meaningful commit history through squashing and rebasing; and (5) heavy-duty history rewriting using tools like `git-filter-repo` and BFG Repo-Cleaner to purge large blobs or exposed credentials from the entire commit history.

**Beginner-Friendly Explanation**
Think of your Git repository as a garden. If you don't weed it regularly, it fills up with junk—compiled files, temporary logs, and even secret keys that should never be there. Repository hygiene is the practice of keeping that garden clean: removing unwanted files, preventing new ones from sneaking in, and cleaning up your commit history so it tells a clear story. It's about being a good citizen in your project and protecting your team from security risks.

### Key Characteristics

- **Preventive**: The best hygiene prevents problems (secrets, large files) before they are committed.
- **Curative**: When mistakes happen, history-rewriting tools can remove them.
- **Proactive**: Secret scanning and `.gitignore` rules act as gates.
- **Semantic**: A clean history is easier to review, bisect, and understand.
- **Team-Safe**: Hygiene practices must account for collaborators' work.

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository with at least one commit.
- Understanding of `.gitignore`, the staging area, and commit history.
- For history rewriting: coordination with all collaborators.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial.
- **DevOps & CI/CD**: Clean build environments and secure pipelines.
- **Security**: Secret management and credential rotation.
- **Software Development**: Code review, debugging, and onboarding.

### Core Concepts / Features

1. **Keeping Repositories Clean** — `git clean -fd` with `--dry-run`.
2. **Avoiding Generated Artifacts** — Treating the repository as a source-code store.
3. **Avoiding Credentials in Git** — Proactive secret scanning with TruffleHog/GitGuardian.
4. **Removing Accidentally Tracked Files** — `git rm --cached`.
5. **Managing Files Tracked Before `.gitignore`** — Untracking without deletion.
6. **Maintaining a Semantic Commit History** — Squashing local iteration junk.
7. **Heavy-Duty History Scrubbing** — `git-filter-repo` and BFG Repo-Cleaner.

---

## 1. Keeping Repositories Clean (`git clean -fd` and `--dry-run`)

### Definitions

**Core Definition**
`git clean` is a command that removes untracked files from the working tree, and with `-d`, also removes untracked directories. The `--dry-run` option previews what would be deleted without actually deleting anything.

**Technical Definition**
`git clean` recursively removes files that are not under version control, starting from the current directory. By default, it does not recurse into untracked directories unless `-d` is specified. It does not remove ignored files unless `-x` is specified. Because deletion is a hard filesystem operation, `git clean` requires `-f` (force) unless `clean.requireForce` is set to `false`. The `-n` (or `--dry-run`) option shows what *would* be removed without deleting.

**Beginner-Friendly Explanation**
`git clean` is like a broom for your project folder: it sweeps away files that Git doesn't know about—build outputs, temporary files, or accidental clutter. But be careful: once deleted, these files are gone. Always use `git clean -n` first to see what will be removed.

### Purposes

- To remove untracked files from the working tree.
- To remove untracked directories with `-d`.
- To clean build artifacts and generated files with `-x`.
- To preview deletions safely with `--dry-run`.
- To reset the working tree to a clean state before a build or commit.

### Syntax Rules and Structure

**Complete General Syntax**

```
git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] <pathspec>…
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
| `-e <pattern>` | Add an exclude pattern. |

**Syntax Rules**

- `-f` is required for actual deletion (unless `clean.requireForce` is `false`).
- `-n` (dry-run) can be used without `-f`.
- `-d` is necessary to remove untracked directories.
- `-x` extends the operation to ignored files.
- `-X` removes only ignored files.

**Constraints and Limitations**

- **Permanent deletion**: `git clean` performs a hard filesystem deletion. Deleted files cannot be recovered through Git.
- **Untracked only**: `git clean` does not affect tracked files.
- **Nested Git repositories**: `git clean -d` refuses to remove nested Git repositories unless `-f -f` is given.
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

**Why this output occurs:** `-n` performs a dry run. It lists `temp.txt` (an untracked file) but does **not** list `build/` because `-d` was not specified. No files are actually deleted.

#### Example 2: Force Delete Files and Directories (`-f -d`)

```bash
git clean -f -d
```

**Expected Output:**
```
Removing temp.txt
Removing build/
```

**Why this output occurs:** `-f` forces the deletion, and `-d` includes directories. Both `temp.txt` and `build/` are permanently deleted.

#### Example 3: Including Ignored Files (`-f -d -x`)

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

### References

- Git Documentation: git-clean Manual Page — https://git-scm.com/docs/git-clean
- Atlassian Git Tutorial: git clean — https://www.atlassian.com/git/tutorials/undoing-changes/git-clean

---

## 2. Avoiding Generated Artifacts (The Philosophy of a Pure Source-Code Store)

### Definitions

**Core Definition**
Avoiding generated artifacts means treating the Git repository as a store of source code and configuration only, excluding any files that are produced by compilers, bundlers, package managers, or other build tools.

**Technical Definition**
Generated artifacts (compiled binaries, distribution bundles, dependency directories, bytecode caches) are derivable from source code and build configuration. Committing them bloats the repository, causes merge conflicts, and obscures the actual source changes. The philosophical principle is that the repository should contain the *recipe* (source code, build scripts, dependency manifests), not the *meal* (build outputs, installed dependencies).

**Beginner-Friendly Explanation**
Your project is like a recipe book. The recipe (source code) is what you want to save and share. The cake you bake from it (compiled program, bundled app) is delicious but can always be made again from the recipe. Don't put the cake in the recipe book—it will get stale and make a mess.

### Purposes

- To keep the repository focused on source code.
- To avoid committing large binary files that bloat the repository.
- To prevent merge conflicts in generated files.
- To ensure clean, reproducible builds from source.
- To reduce clone and fetch times.

### Syntax Rules and Structure

**Common Patterns by Ecosystem**

| Ecosystem | Patterns |
|-----------|----------|
| Python | `__pycache__/`, `*.py[cod]`, `build/`, `dist/`, `*.egg-info/` |
| Java | `target/`, `*.class`, `build/` |
| JavaScript/TypeScript | `dist/`, `build/`, `out/`, `.next/`, `*.tsbuildinfo` |
| C/C++ | `*.o`, `*.so`, `*.a`, `build/` |
| Rust | `target/` |
| Go | `bin/`, `*.exe` |
| Node.js dependencies | `node_modules/`, `.pnpm-store/` |
| PHP dependencies | `vendor/` |

**Syntax Rules**

- Use directory patterns with a trailing slash.
- Use `**/` for monorepos.
- **Always commit the lock file** (`package-lock.json`, `composer.lock`, `go.sum`).
- Virtual environments (`venv/`, `.venv/`) should be ignored.

**Constraints and Limitations**

- **Installation required**: Developers must run the build toolchain after cloning.
- **Lock file must be committed**: Without the lock file, installs are not reproducible.

### Annotated Code Examples

#### Example 1: Python Build Artifacts

```gitignore
__pycache__/
*.py[cod]
build/
dist/
*.egg-info/
```

**Why these patterns work:** `__pycache__/` ignores Python bytecode directories. `*.py[cod]` matches `.pyc`, `.pyo`, and `.pyd` files. `build/` and `dist/` ignore standard packaging output directories.

#### Example 2: Node.js Dependencies and Build Output

```gitignore
node_modules/
dist/
build/
.next/
*.tsbuildinfo
```

**Why these patterns work:** `node_modules/` is the npm dependency directory. `dist/` and `build/` are common build output directories. `*.tsbuildinfo` ignores TypeScript incremental build metadata.

### Real-World Cases

- **Open-source project**: A maintainer ensures the repository contains only source; contributors run `npm install` and `npm run build` after cloning.
- **CI/CD pipeline**: The pipeline installs dependencies and builds from source, then deploys the generated artifacts.
- **Monorepo**: Multiple packages each have `dist/`; `**/dist/` ignores them all.

### References

- GitHub gitignore Repository — https://github.com/github/gitignore
- Python Packaging User Guide — https://packaging.python.org/en/latest/

---

## 3. Avoiding Credentials in Git (Proactive Secret Scanning)

### Definitions

**Core Definition**
Avoiding credentials in Git means proactively preventing secrets—API keys, passwords, tokens, private keys—from being committed to the repository by using secret scanning tools and pre-commit hooks.

**Technical Definition**
Secret scanning tools (TruffleHog, GitGuardian, Gitleaks, detect-secrets) analyse code changes for patterns that match known secret formats, high-entropy strings, or verified credential formats. They can be integrated as pre-commit hooks (blocking commits locally), CI steps (blocking merges), or history scanners (auditing existing repositories). TruffleHog verifies whether a detected secret is live by making an API call to the provider. GitGuardian offers a SaaS platform with real-time leak monitoring. Gitleaks is a fast, offline, regex-based scanner.

**Beginner-Friendly Explanation**
Imagine you're about to mail a letter, but you accidentally left your house key inside. A secret scanner is like a friend who checks the envelope before you seal it: "Hey, there's a key in there!" Pre-commit hooks do this automatically every time you commit. Tools like TruffleHog and GitGuardian check not just the envelope, but also the entire history of letters you've sent.

### Purposes

- To prevent credentials from entering the repository.
- To detect secrets already in the repository history.
- To block commits containing secrets at commit time.
- To verify whether a detected secret is still live.
- To comply with security best practices and regulations.

### Syntax Rules and Structure

**Common Tools**

| Tool | Type | Best For |
|------|------|----------|
| Gitleaks | CLI, offline | Fast pre-commit and CI blocking. |
| TruffleHog | CLI | Verifying which leaked secrets are actually live. |
| GitGuardian | SaaS, CLI (ggshield) | Real-time leak monitoring, incident management. |
| detect-secrets | CLI | Retrofitting legacy repositories with a baseline. |

**Integration Points**

| Point | Description |
|-------|-------------|
| Pre-commit hook | Blocks commits locally. |
| CI/CD step | Blocks merges in pull requests. |
| History scan | Audits existing repository history. |

**Syntax Rules**

- Install the tool (e.g., `pip install trufflehog`, `brew install gitleaks`).
- Run against the repository: `trufflehog git file://.`, `gitleaks detect`.
- Configure pre-commit hooks in `.pre-commit-config.yaml`.
- For GitGuardian: `ggshield secret scan pre-commit`.

**Constraints and Limitations**

- **False positives**: Entropy detection may flag non-secrets.
- **Performance**: Deep history scans can be slow on large repositories.
- **Verification requires network**: TruffleHog's verification makes API calls.
- **Secrets already committed**: Rotation is required; scanning alone does not remove secrets.

### Annotated Code Examples

#### Example 1: TruffleHog Pre-commit Hook

```bash
# Install trufflehog
pip install trufflehog

# Scan the current repository
trufflehog git file://. --only-verified
```

**Expected Output:**
```
Found verified result 🐷🔑
Detector Type: AWS
Raw result: AKIA...
```

**Why this output occurs:** TruffleHog scans the repository and verifies detected secrets by making API calls. Only verified (live) secrets are reported, reducing false positives.

#### Example 2: Gitleaks in CI

```yaml
# .github/workflows/secret-scan.yml
- name: Gitleaks
  uses: gitleaks/gitleaks-action@v2
  with:
    args: --verbose
```

**Why this works:** Gitleaks runs in CI and blocks the pipeline if secrets are detected in the PR.

### Real-World Cases

- **Open-source project**: A maintainer adds Gitleaks to CI; a contributor accidentally commits an API key, and the pipeline fails before the PR can be merged.
- **Enterprise team**: A security team installs GitGuardian across all repositories; a developer receives an alert when a secret is pushed and rotates it immediately.
- **Legacy repository**: A team uses `detect-secrets` to create a baseline of existing secrets, then prevents new ones from being added.

### References

- TruffleHog Documentation — https://github.com/trufflesecurity/trufflehog
- GitGuardian Documentation — https://docs.gitguardian.com/
- Gitleaks Documentation — https://github.com/gitleaks/gitleaks
- detect-secrets Documentation — https://github.com/Yelp/detect-secrets
- Safeguard.sh: Best Secrets Scanning Tools in 2026 — https://safeguard.sh/resources/blog/best-secrets-scanning-tools-2026

---

## 4. Removing Accidentally Tracked Files (`git rm --cached`)

### Definitions

**Core Definition**
`git rm --cached` removes a file from the Git index (staging area) while preserving it in the working directory, effectively untracking it without deleting it from disk.

**Technical Definition**
`git rm --cached <file>` removes the file from the index, so it is no longer tracked by Git. The file remains in the working directory as an untracked file. This is the correct command when a file was accidentally committed and should now be ignored. In contrast, `git rm <file>` removes the file from both the index and the working directory. The `--cached` option is essential for preserving local files.

**Beginner-Friendly Explanation**
Imagine you accidentally put your diary in a shared filing cabinet (the repository). You want to take it out of the cabinet but keep it on your desk (working directory). `git rm --cached` does exactly that: it removes the file from Git's tracking but leaves the actual file on your computer.

### Purposes

- To stop tracking a file without deleting it.
- To remove a file from the repository that should have been ignored.
- To prepare for adding the file to `.gitignore`.
- To correct an accidental `git add` of a sensitive or generated file.
- To keep the file locally while removing it from version control.

### Syntax Rules and Structure

**Complete General Syntax**

```
git rm --cached <file>
git rm --cached -r <directory>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--cached` | Remove from index only; keep in working directory. |
| `<file>` | The file to untrack. |
| `-r` | Recursive; required for directories. |

**Syntax Rules**

- `--cached` preserves the file in the working directory.
- `-r` is required for directories.
- After running, the file appears as untracked in `git status`.
- The change must be committed to take effect in the repository.

**Constraints and Limitations**

- **Commit required**: The untracking takes effect only after committing the change.
- **History remains**: The file's previous versions remain in Git history (use history rewriting to remove them completely).
- **`.gitignore` needed**: After untracking, add the file to `.gitignore` to prevent re-adding.

### Annotated Code Examples

#### Example 1: Untracking a File

```bash
# Accidentally commit a file
echo "secret" > config.yaml
git add config.yaml && git commit -m "Add config"

# Untrack the file (keep local copy)
git rm --cached config.yaml

# Add to .gitignore
echo "config.yaml" >> .gitignore

# Commit the untracking
git add .gitignore
git commit -m "Untrack config.yaml and add to gitignore"
```

**Expected Output:**
```
rm 'config.yaml'
[master 4d5e6f7] Untrack config.yaml and add to gitignore
 2 files changed, 1 insertion(+), 1 deletion(-)
 delete mode 100644 config.yaml
```

**Why this output occurs:** `git rm --cached` removes `config.yaml` from the index. The file remains on disk. The commit records the deletion from the repository. `git status` will now show `config.yaml` as untracked (but ignored due to `.gitignore`).

### Real-World Cases

- **Accidentally committed `.env`**: A developer commits `.env`, then runs `git rm --cached .env`, adds it to `.gitignore`, and commits the change.
- **Build artifact committed**: A developer accidentally commits `dist/bundle.js`, untracks it, and adds `dist/` to `.gitignore`.
- **Personal IDE settings**: A developer commits `.vscode/settings.json`, untracks it, and adds `.vscode/` to their global ignore file.

### References

- Git Documentation: git-rm Manual Page — https://git-scm.com/docs/git-rm
- GitHub Community: How to avoid deleting ignored files — https://github.com/orgs/community/discussions/118406

---

## 5. Managing Files Already Tracked Before They Were Added to `.gitignore`

### Definitions

**Core Definition**
Managing files that were already tracked before being added to `.gitignore` requires explicitly untracking them with `git rm --cached`, because `.gitignore` rules do not apply to files already in the index.

**Technical Definition**
`.gitignore` has no effect on files that are already tracked by Git. Once a file is in the index, it remains tracked even if it matches a pattern in `.gitignore`. To stop tracking it, the file must be removed from the index with `git rm --cached <file>` and the change committed. The file then becomes untracked, and future `.gitignore` rules will apply.

**Beginner-Friendly Explanation**
You can't ignore something that's already in the system. If you add a file to `.gitignore` but it's already being tracked, Git will keep tracking it. You have to explicitly tell Git to stop tracking it—using `git rm --cached`—and then commit that change.

### Purposes

- To stop tracking a file that should be ignored.
- To apply `.gitignore` rules to previously tracked files.
- To remove generated or sensitive files from the repository while keeping them locally.
- To clean up a repository that has accumulated unwanted tracked files.
- To prepare for a fresh start with proper ignore rules.

### Syntax Rules and Structure

**Complete General Syntax**

```
git rm --cached <file>
git commit -m "Stop tracking <file>"
```

**Step-by-Step Workflow**

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `git rm --cached <file>` | Remove from index, keep in working directory. |
| 2 | `echo "<file>" >> .gitignore` | Add to ignore rules. |
| 3 | `git add .gitignore` | Stage the updated ignore file. |
| 4 | `git commit -m "Stop tracking <file>"` | Commit the change. |

**Syntax Rules**

- `.gitignore` does not affect tracked files.
- `git rm --cached` is the correct command; `git rm` would delete the file.
- The change must be committed for the untracking to take effect.
- For directories, use `-r`.

**Constraints and Limitations**

- **History remains**: The file's previous versions remain in Git history.
- **Team coordination**: Other developers' local copies will not be automatically untracked; they must pull and the file will appear as deleted in their working tree unless they also untrack it.
- **One-time operation**: After untracking, `.gitignore` will prevent future tracking.

### Annotated Code Examples

#### Example 1: Untracking a File That Was Already Committed

```bash
# File was committed before .gitignore was updated
git log --oneline -- secret.key
# Output: 1a2b3c4 Add secret key

# Add to .gitignore
echo "secret.key" >> .gitignore

# Untrack the file
git rm --cached secret.key

# Commit the untracking
git add .gitignore
git commit -m "Untrack secret.key and add to gitignore"
```

**Expected Output:**
```
rm 'secret.key'
[master 5d6e7f8] Untrack secret.key and add to gitignore
 2 files changed, 1 insertion(+), 1 deletion(-)
 delete mode 100644 secret.key
```

**Why this output occurs:** The file was already tracked. Adding it to `.gitignore` alone would not untrack it. `git rm --cached` removes it from the index, and the commit records the change. The file remains on disk but is now ignored.

### Real-World Cases

- **Legacy repository cleanup**: A team inherits a repository with committed `node_modules/`; they untrack it and add it to `.gitignore`.
- **`.env` file committed before ignore rules**: A developer untracks the `.env` file and commits the change so future `.env` files are ignored.
- **IDE configuration**: A project commits `.idea/` files; the team untracks them and adds `.idea/` to `.gitignore`.

### References

- GitHub Community: How to avoid deleting ignored files — https://github.com/orgs/community/discussions/118406
- Stack Overflow: Difference between git ignore and untrack — https://stackoverflow.com/questions/17888695

---

## 6. Maintaining a Useful, Semantic Commit History (Squashing Local Iteration Junk)

### Definitions

**Core Definition**
Maintaining a semantic commit history means cleaning up local "work in progress" commits before sharing them, so the public history contains only meaningful, atomic commits that describe logical changes.

**Technical Definition**
During development, developers often make many small, iterative commits ("WIP", "fix typo", "address review"). Before pushing to a shared branch, these are consolidated into one or more semantically meaningful commits using interactive rebase (`git rebase -i`), `--fixup`/`--autosquash`, or squash merges. The goal is a history where each commit represents a single logical change, making `git bisect`, code review, and `git blame` more effective.

**Beginner-Friendly Explanation**
Think of your commit history as a story. While writing, you might scribble notes and make corrections. But when you present the story to others, you want it to read cleanly—no scribbles, no "oops" corrections. Squashing lets you clean up your scribbles into a polished final version before sharing.

### Purposes

- To create a clean, readable commit history.
- To make `git bisect` and `git blame` effective.
- To facilitate code review by isolating logical changes.
- To avoid cluttering the shared history with WIP commits.
- To produce a professional, maintainable project history.

### Syntax Rules and Structure

**Common Techniques**

| Technique | Command | Description |
|-----------|---------|-------------|
| Interactive rebase | `git rebase -i HEAD~n` | Squash, reword, or reorder commits. |
| Autosquash | `git rebase -i --autosquash HEAD~n` | Automatically squash `fixup!` commits. |
| Fixup commits | `git commit --fixup HEAD` | Create a fixup commit to be autosquashed later. |
| Squash merge | GitHub/GitLab UI | Squash all PR commits into one on merge. |
| Soft reset | `git reset --soft HEAD~n` | Combine commits and re-commit as one. |

**What to Squash vs. Keep**

| Squash | Keep Separate |
|--------|---------------|
| WIP commits | Each logical feature |
| "Fix typo" after feature | Bug fixes (different concern) |
| "Address review feedback" | Refactoring (different intent) |
| Multiple attempts at same thing | Test additions (reviewable unit) |

**Syntax Rules**

- Interactive rebase opens an editor with the commit list.
- Change `pick` to `squash` (or `s`) to combine with the previous commit.
- Change `pick` to `fixup` (or `f`) to combine and discard the commit message.
- `--autosquash` automatically reorders and squashes `fixup!` commits.
- **Never rewrite published history** that others have pulled.

**Constraints and Limitations**

- **Published history**: Squashing rewritten commits that have been pushed requires force-pushing and coordination.
- **Conflict risk**: Rebasing may produce conflicts.
- **Team workflow**: Squash-merge workflows handle this automatically on the remote.

### Annotated Code Examples

#### Example 1: Interactive Rebase to Squash WIP Commits

```bash
# Three WIP commits
git log --oneline -3
# Output:
# 4d5e6f7 WIP: more fixes
# 1a2b3c4 WIP: validation
# 9a0b1c2 feat: add user validation

# Start interactive rebase
git rebase -i HEAD~3
```

**Editor opens:**
```
pick 9a0b1c2 feat: add user validation
squash 1a2b3c4 WIP: validation
squash 4d5e6f7 WIP: more fixes
```

**After saving, combine into one commit:**
```
feat: add user validation with edge case handling
```

**Expected Output:**
```
[detached HEAD 5d6e7f8] feat: add user validation with edge case handling
 1 file changed, 10 insertions(+), 2 deletions(-)
```

**Why this output occurs:** The `squash` command combines all three commits into one. The editor opens again to allow editing the combined commit message. The final history has one clean commit instead of three WIP commits.

#### Example 2: Using `--fixup` and `--autosquash`

```bash
# Commit a feature
git commit -m "feat: add validation"

# Make a fixup commit
git commit --fixup HEAD
# Output: [master 1a2b3c4] fixup! feat: add validation

# Later, autosquash all fixups
git rebase -i --autosquash HEAD~2
```

**Expected Output:**
```
[detached HEAD 5d6e7f8] feat: add validation
 1 file changed, 5 insertions(+)
```

**Why this output occurs:** `--fixup HEAD` creates a commit with the message `fixup! <original>`. `--autosquash` automatically reorders and marks it for squashing. The result is a single clean commit.

### Real-World Cases

- **Feature branch cleanup**: A developer makes 15 WIP commits on a feature branch, then squashes them into 3 logical commits before creating a pull request.
- **Code review**: A reviewer asks for changes; the developer uses `--fixup` and `--autosquash` to integrate the feedback cleanly.
- **Open-source contribution**: A contributor squashes their commits to match the project's convention of one commit per logical change.

### References

- GitHub Community: Do I really need to squash my commits? — https://github.com/orgs/community/discussions/182075
- History Hygiene Rules (orchestkit) — https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/commit/rules/history-hygiene.md
- Commit Discipline — https://github.com/yonatangross/orchestkit

---

## 7. Heavy-Duty History Scrubbing (`git-filter-repo` and BFG Repo-Cleaner)

### Definitions

**Core Definition**
Heavy-duty history scrubbing is the use of specialized tools (`git-filter-repo` and BFG Repo-Cleaner) to rewrite a repository's entire commit history, removing large blobs, exposed credentials, or other unwanted content from every commit.

**Technical Definition**
`git-filter-repo` is a Python-based tool that rewrites Git history by applying filters to every commit. It is the recommended replacement for the deprecated `git filter-branch` command. BFG Repo-Cleaner is a Scala-based tool that provides a simpler, faster (10–720x) alternative for common cleanup tasks: removing large files (`--strip-blobs-bigger-than`), deleting specific files (`--delete-files`), and replacing text (`--replace-text`). Both tools modify commit hashes, requiring a force-push and coordination with collaborators.

**Beginner-Friendly Explanation**
Imagine your project's history is a book, and someone accidentally printed a page with your password on it. You can't just tear out that page—every copy of the book has it. These tools are like a magic eraser that goes through every copy of the book and removes the password from every page, everywhere. It's powerful but requires everyone to get the new version.

### Purposes

- To remove large files from the entire repository history.
- To purge exposed credentials from every commit.
- To reduce repository size for faster clones.
- To comply with security and privacy requirements.
- To clean up a repository that has accumulated unwanted content over time.

### Syntax Rules and Structure

**Tool Comparison**

| Aspect | `git-filter-repo` | BFG Repo-Cleaner |
|--------|-------------------|-------------------|
| Language | Python | Scala |
| Speed | Fast | Very fast (10–720x faster than filter-branch) |
| Configuration | CLI arguments, callbacks | Simple CLI flags |
| Use case | Official, highly configurable, fine-grained | Fast, simple syntax for common tasks |
| Installation | `pip install git-filter-repo` | Java JAR download |

**Common Commands**

| Task | `git-filter-repo` | BFG |
|------|-------------------|-----|
| Remove file | `git filter-repo --path file --invert-paths` | `bfg --delete-files file` |
| Remove large blobs | `git filter-repo --strip-blobs-bigger-than 1M` | `bfg --strip-blobs-bigger-than 1M` |
| Replace text | `git filter-repo --replace-text passwords.txt` | `bfg --replace-text passwords.txt` |
| Remove directory | `git filter-repo --path dir/ --invert-paths` | — |

**Syntax Rules**

- **Back up the repository** before running either tool.
- Both tools rewrite history, changing all commit hashes.
- After rewriting, force-push to the remote.
- All collaborators must reclone or reset their local copies.
- **Rotate any exposed credentials immediately**—history rewriting does not invalidate them.

**Constraints and Limitations**

- **Irreversible**: History rewriting cannot be undone without a backup.
- **Team coordination**: All collaborators must be notified and must reclone.
- **Force-push required**: The rewritten history must be force-pushed to the remote.
- **Not a substitute for rotation**: Exposed secrets must be rotated regardless.

### Annotated Code Examples

#### Example 1: Removing a Large File with `git-filter-repo`

```bash
# Install git-filter-repo
pip install git-filter-repo

# Remove a large file from all history
git filter-repo --path large-video.mp4 --invert-paths

# Force-push the cleaned history
git push --force --all
git push --force --tags
```

**Expected Output:**
```
Parsed 150 commits
New history written in 2.34 seconds
```

**Why this output occurs:** `--path large-video.mp4` selects the file, and `--invert-paths` removes it from all commits. The tool rewrites every commit, removing the file from the entire history. Force-pushing updates the remote.

#### Example 2: Removing Credentials with BFG

```bash
# Create a file with passwords to replace
echo "PASSWORD" > passwords.txt
echo "API_KEY" >> passwords.txt

# Run BFG
java -jar bfg.jar --replace-text passwords.txt my-repo.git

# Clean up and force-push
cd my-repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

**Expected Output:**
```
Found 5 commits with sensitive data
Replaced 3 occurrences of PASSWORD
Replaced 2 occurrences of API_KEY
```

**Why this output occurs:** BFG scans all commits and replaces the specified text patterns. The `reflog expire` and `gc` commands clean up the old objects. Force-pushing updates the remote with the scrubbed history.

### Real-World Cases

- **Accidental credential commit**: A developer commits an AWS key; the team uses BFG to remove it from all history and rotates the key.
- **Large binary in history**: A repository is bloated with a 500MB video file; `git-filter-repo` removes it from all commits, dramatically reducing clone time.
- **Open-source cleanup**: A project discovers that a contributor committed proprietary code; the maintainer uses `git-filter-repo` to remove it from history.

### References

- git-filter-repo Documentation — https://github.com/newren/git-filter-repo
- BFG Repo-Cleaner Documentation — https://rtyley.github.io/bfg-repo-cleaner/
- Microsoft Learn: Purge Data from Source Control — https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/7-purge-repository-data
- KodeKloud: Purge Data from Source Control — https://notes.kodekloud.com/docs/AZ-400-Designing-and-Implementing-Microsoft-DevOps-Solutions/Configuring-and-Managing-Repositories/Purge-Data-from-Source-Control
- GitHub Docs: Removing Sensitive Data from a Repository — https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository

---

## References

- Git Documentation: git-clean Manual Page — https://git-scm.com/docs/git-clean
- Git Documentation: git-rm Manual Page — https://git-scm.com/docs/git-rm
- Git Documentation: gitignore Manual Page — https://git-scm.com/docs/gitignore
- git-filter-repo Documentation — https://github.com/newren/git-filter-repo
- BFG Repo-Cleaner Documentation — https://rtyley.github.io/bfg-repo-cleaner/
- TruffleHog Documentation — https://github.com/trufflesecurity/trufflehog
- GitGuardian Documentation — https://docs.gitguardian.com/
- Gitleaks Documentation — https://github.com/gitleaks/gitleaks
- detect-secrets Documentation — https://github.com/Yelp/detect-secrets
- Microsoft Learn: Purge Data from Source Control — https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/7-purge-repository-data
- KodeKloud: Purge Data from Source Control — https://notes.kodekloud.com/docs/AZ-400-Designing-and-Implementing-Microsoft-DevOps-Solutions/Configuring-and-Managing-Repositories/Purge-Data-from-Source-Control
- GitHub Community: How to avoid deleting ignored files — https://github.com/orgs/community/discussions/118406
- GitHub Community: Do I really need to squash my commits? — https://github.com/orgs/community/discussions/182075
- History Hygiene Rules (orchestkit) — https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/commit/rules/history-hygiene.md
- Safeguard.sh: Best Secrets Scanning Tools in 2026 — https://safeguard.sh/resources/blog/best-secrets-scanning-tools-2026