# Git Branch-Based Development: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Branch-based development is a software development strategy in which work is isolated on separate Git branches—each representing a distinct line of development—and later integrated into a shared mainline through merging or pull requests.

**Technical Definition**
Branch-based development leverages Git's lightweight branching model, where a branch is a movable pointer to a commit in the Directed Acyclic Graph (DAG). Different branch types serve different purposes: feature branches isolate new functionality, bug-fix/hotfix branches address defects, release branches stabilise code before deployment, and short-lived branches enable rapid iteration. Upstream tracking (`git branch --set-upstream-to`) links local branches to remote counterparts for synchronisation, while pruning (`git remote prune`, `git fetch --prune`) removes stale remote-tracking references.

**Beginner-Friendly Explanation**
Imagine you're writing a book with a team. Instead of everyone editing the same manuscript at the same time (which would cause chaos), each person takes a copy, makes their changes, and then submits them for review. Git branches are those copies. Different types of branches serve different purposes: some are for new chapters (features), some for fixing typos (bug fixes), and some for final proofreading before printing (releases). This cheat sheet shows you how to use each type effectively.

### Key Characteristics

- **Isolation**: Branches isolate work-in-progress from stable code.
- **Lightweight**: Creating a branch costs almost nothing (a 41-byte pointer file).
- **Parallel**: Multiple developers can work simultaneously on different branches.
- **Integrable**: Branches are designed to be merged back into the mainline.
- **Trackable**: Local branches can track remote branches for synchronisation.
- **Disposable**: Short-lived branches can be deleted after merging.

### Prerequisites

- Git installed (version 2.23+ recommended for `git switch`; 2.28+ for `init.defaultBranch`).
- A Git repository with at least one commit.
- Basic familiarity with branch concepts (see the "Branch Concepts" cheat sheet).
- Understanding of `.gitignore` and the staging area.

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial.
- **DevOps & CI/CD**: Branch-based deployment pipelines.
- **Software Development**: Feature development, bug fixing, release management.
- **Collaborative Development**: Pull requests, code review, merge strategies.

### Core Concepts / Features

1. **Feature Branches** — Isolating new feature additions.
2. **Bug-Fix Branches** — Hotfixes and targeted patches.
3. **Release Branches** — Stabilisation phases before production.
4. **Local Experimentation** — Throwaway branches for testing spikes.
5. **Short-Lived Branches** — Continuous integration and minimising divergence.
6. **Upstream Tracking and Branch Synchronisation** — `git branch --set-upstream-to`.
7. **Pruning Dead Local Branches** — `git remote prune` and `git fetch --prune`.

---

## 1. Feature Branches (Isolating New Feature Additions)

### Definitions

**Core Definition**
A feature branch is a short-lived branch created from the mainline to develop a new feature or enhancement in isolation, without affecting the stable codebase.

**Technical Definition**
Feature branches (also called topic branches) branch off the main branch (e.g., `main`, `develop`) and are merged back via pull requests or merge requests after review. They isolate work-in-progress from the completed, stable code in the mainline. Git branches are cheap to create and maintain, so even small fixes should have their own feature branch. Consistent naming conventions (e.g., `feature/<description>`) make it easy to identify the work in each branch.

**Beginner-Friendly Explanation**
A feature branch is like a sandbox where you can build a new feature without disturbing the rest of the project. When it's ready, you merge it back into the main project. If it doesn't work out, you can delete the branch and no harm is done.

### Purposes

- To isolate new feature development from stable code.
- To enable code review through pull requests before merging.
- To allow parallel development of multiple features.
- To maintain a clean, high-quality main branch.
- To facilitate rollback if a feature is abandoned.

### Syntax Rules and Structure

**Complete General Syntax**

```
git switch -c feature/<description> [<start-point>]
git push -u origin feature/<description>
# ... develop, commit, push ...
# Open pull request, review, merge
git branch -d feature/<description>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `feature/<description>` | Branch name (convention). |
| `<start-point>` | Usually `main` or `develop`. |
| `-c` | Create and switch. |
| `-u` | Set upstream tracking. |

**Syntax Rules**

- Branch from the latest mainline commit.
- Use consistent naming: `feature/`, `feature/<issue>-<description>`, `users/<username>/<description>`.
- Merge via pull request to enable review.
- Delete the feature branch after merging.

**Constraints and Limitations**

- **Divergence risk**: Long-lived feature branches diverge from mainline and cause merge conflicts.
- **Review overhead**: Pull requests require reviewer availability.
- **CI cost**: Each branch may trigger CI pipelines.

### Annotated Code Examples

#### Example 1: Creating and Merging a Feature Branch

```bash
# Start from main
git switch main
git pull origin main

# Create feature branch
git switch -c feature/user-authentication

# Develop and commit
echo "Auth logic" > auth.js
git add auth.js
git commit -m "feat: add user authentication"

# Push and set upstream
git push -u origin feature/user-authentication

# After review, merge (via UI or CLI)
git switch main
git merge --no-ff feature/user-authentication

# Delete the feature branch
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication
```

**Expected Output:**
```
Switched to a new branch 'feature/user-authentication'
[feature/user-authentication 1a2b3c4] feat: add user authentication
 1 file changed, 1 insertion(+)
...
Merge made by the 'ort' strategy.
...
Deleted branch feature/user-authentication (was 1a2b3c4).
```

**Why this output occurs:** The feature branch is created from `main`, the commit is made on the branch, and it is pushed with upstream tracking. After merging with `--no-ff` (preserving a merge commit), the branch is deleted locally and remotely.

### Real-World Cases

- **SaaS product**: A team uses feature branches for each new feature (`feature/billing`, `feature/notifications`), merging them into `main` after review.
- **Open-source project**: Contributors fork the repository, create feature branches, and submit pull requests.
- **Enterprise**: Feature branches are linked to work items (e.g., `feature/1234-add-login`).

### References

- Microsoft Learn: Git Branching Guidance — https://learn.microsoft.com/en-us/azure/devops/repos/git/git-branching-guidance
- AWS Prescriptive Guidance: Choosing a Git Branching Strategy — https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-git-branch-approach/
- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch

---

## 2. Bug-Fix Branches (Hotfixes and Targeted Patches)

### Definitions

**Core Definition**
A bug-fix branch is a short-lived branch created to address a specific defect. It branches from the appropriate base (release branch, main, or develop) depending on the severity and target environment.

**Technical Definition**
Two primary types exist: **bugfix branches** (branched from a release branch for defects not yet in production) and **hotfix branches** (branched from `main` for urgent production defects). A bugfix branch is always branched from a release branch and merged back into it after testing. A hotfix branch is always branched from `main` and merged into `main` (and possibly the release branch).

**Beginner-Friendly Explanation**
When something is broken, you create a special branch to fix it. For urgent production bugs, you branch from the live code (hotfix). For non-urgent bugs in a release being tested, you branch from the release branch (bugfix). After fixing and testing, you merge the fix back into the appropriate branches.

### Purposes

- To fix production defects urgently without disrupting ongoing development.
- To address bugs in release branches before production deployment.
- To isolate fixes for code review and testing.
- To maintain a stable mainline while fixing defects.
- To ensure fixes are propagated to all affected branches.

### Syntax Rules and Structure

**Complete General Syntax**

```
# Hotfix (urgent production fix)
git switch main
git pull origin main
git switch -c hotfix/<ticket>_<description>
# ... fix, commit ...
git switch main
git merge --no-ff hotfix/<ticket>_<description>
git tag -a v1.0.1 -m "Hotfix version 1.0.1"
git switch develop
git merge --no-ff hotfix/<ticket>_<description>

# Bugfix (non-urgent release fix)
git switch release/v1.0
git switch -c bugfix/<ticket>_<description>
# ... fix, commit ...
git switch release/v1.0
git merge --no-ff bugfix/<ticket>_<description>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `hotfix/` | Branch name prefix for production fixes. |
| `bugfix/` | Branch name prefix for release fixes. |
| `<ticket>` | Issue/ticket number. |
| `<description>` | Short descriptor. |

**Syntax Rules**

- Hotfix branches from `main`; merge into `main` and `develop`.
- Bugfix branches from a release branch; merge back into the release branch.
- Use `--no-ff` to preserve a merge commit for traceability.
- Tag the hotfix release (e.g., `v1.0.1`).

**Constraints and Limitations**

- **Urgency**: Hotfixes bypass normal release cycles; testing may be abbreviated.
- **Merge conflicts**: Hotfixes must be merged back to `develop` to avoid regressions.
- **Coordination**: Multiple hotfixes may conflict.

### Annotated Code Examples

#### Example 1: Hotfix Workflow

```bash
# 1. Branch from main
git switch main
git pull origin main
git switch -c hotfix/1234_critical_login_fix

# 2. Fix the bug
echo "Fixed login" > login.js
git add login.js
git commit -m "fix: resolve null pointer in login handler"

# 3. Merge into main
git switch main
git merge --no-ff hotfix/1234_critical_login_fix
git tag -a v1.0.1 -m "Hotfix version 1.0.1"

# 4. Merge into develop (if applicable)
git switch develop
git merge --no-ff hotfix/1234_critical_login_fix
```

**Expected Output:**
```
Switched to a new branch 'hotfix/1234_critical_login_fix'
[hotfix/1234_critical_login_fix 5d6e7f8] fix: resolve null pointer in login handler
 1 file changed, 2 insertions(+), 1 deletion(-)
...
Merge made by the 'ort' strategy.
...
```

**Why this output occurs:** The hotfix branch is created from `main`, the fix is committed, and the branch is merged back into `main` with a merge commit and tag. It is then merged into `develop` to prevent regression.

### Real-World Cases

- **Production outage**: A critical authentication bug is fixed via a hotfix branch and deployed immediately.
- **Release candidate bug**: A bug found during release testing is fixed on a bugfix branch and merged into the release branch.
- **Security patch**: A vulnerability is patched via a hotfix and tagged for release.

### References

- AWS Prescriptive Guidance: Choosing a Git Branching Strategy (Hotfix/Bugfix) — https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-git-branch-approach/
- GitFlow Workflow Guide — https://github.com/aiskillstore/marketplace
- Git Documentation: git-merge Manual Page — https://git-scm.com/docs/git-merge

---

## 3. Release Branches (Stabilisation Before Production)

### Definitions

**Core Definition**
A release branch is a temporary branch created from the develop branch when development for a release is complete, used for final bug fixes, version bumps, and stabilisation before merging into main and tagging a production release.

**Technical Definition**
Release branches are created from `develop` when the code is feature-complete for a target release. They allow final stabilisation without blocking ongoing development on `develop`. Once stabilised, the release branch is merged into `main` (with a tag) and back into `develop` to incorporate any release-specific fixes. Naming conventions typically follow `release/v<major>.<minor>`.

**Beginner-Friendly Explanation**
Before you ship a new version of your software, you want to make sure everything works perfectly. A release branch is like a "final preparations" room where you fix last-minute bugs and test the release without stopping work on the next version. Once it's ready, you ship it (merge to main and tag it).

### Purposes

- To stabilise a release without blocking ongoing feature development.
- To allow final bug fixes and version bumps in isolation.
- To prepare release candidates (RCs) for testing.
- To provide a clean, tagged release on main.
- To merge release-specific fixes back into develop.

### Syntax Rules and Structure

**Complete General Syntax**

```
git switch develop
git switch -c release/v1.2.0
# ... stabilise, fix bugs, bump version ...
git switch main
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git switch develop
git merge --no-ff release/v1.2.0
git branch -d release/v1.2.0
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `release/v<major>.<minor>` | Branch name convention. |
| `<start-point>` | `develop`. |
| `--no-ff` | Preserve merge commit. |
| `-a v1.2.0` | Annotated tag. |

**Syntax Rules**

- Branch from `develop` when feature-complete.
- Only bug fixes and release-specific changes are allowed on the release branch.
- Merge into `main` and tag the release.
- Merge back into `develop` to incorporate fixes.
- Delete the release branch after merging.

**Constraints and Limitations**

- **Complexity**: Adds additional branches and merges to manage.
- **Cherry-picking**: Fixes must be merged, not cherry-picked, to avoid divergence.
- **Tagging discipline**: Tags must be created on `main` after merge.

### Annotated Code Examples

#### Example 1: Release Branch Workflow

```bash
# 1. Create release branch from develop
git switch develop
git switch -c release/v1.2.0

# 2. Stabilise (fix bugs, bump version)
echo "1.2.0" > VERSION
git add VERSION
git commit -m "chore: bump version to 1.2.0"

# 3. Merge into main
git switch main
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"

# 4. Merge back into develop
git switch develop
git merge --no-ff release/v1.2.0

# 5. Delete release branch
git branch -d release/v1.2.0
```

**Expected Output:**
```
Switched to a new branch 'release/v1.2.0'
[release/v1.2.0 3d4e5f6] chore: bump version to 1.2.0
 1 file changed, 1 insertion(+)
...
Merge made by the 'ort' strategy.
...
```

**Why this output occurs:** The release branch is created from `develop`, the version is bumped, and the branch is merged into both `main` (with a tag) and `develop`. The release branch is then deleted.

### Real-World Cases

- **Enterprise software**: A release branch is cut for each major version (`release/v2.0`), stabilised over several weeks, then shipped.
- **Mobile app**: A release branch is created for App Store submission, with final bug fixes and metadata updates.
- **Open-source**: A release branch is maintained for each minor version to backport security fixes.

### References

- AWS Prescriptive Guidance: Choosing a Git Branching Strategy (Release Branches) — https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-git-branch-approach/
- GitFlow Workflow Guide — https://github.com/aiskillstore/marketplace
- Git Documentation: git-tag Manual Page — https://git-scm.com/docs/git-tag

---

## 4. Local Experimentation (Throwaway Branches for Testing Spikes)

### Definitions

**Core Definition**
A local experimentation branch (throwaway branch, spike branch) is a temporary branch created purely for testing an idea, with no intention of merging it into the mainline.

**Technical Definition**
Git's branching model encourages disposable experimentation: create a branch to try an idea, commit several times, and if it doesn't work out, simply delete the branch—abandoning the work with nobody else ever seeing it. These branches are local-only (never pushed) and serve as a safe sandbox for risky changes, performance experiments, or proof-of-concept work.

**Beginner-Friendly Explanation**
Sometimes you want to try something crazy without any risk. Create a throwaway branch, experiment freely, and if it doesn't work, just delete the branch. No one ever has to know. It's like a scratchpad for code.

### Purposes

- To test risky ideas without affecting the mainline.
- To explore alternative implementations before committing.
- To conduct time-boxed "spikes" for research.
- To safely abandon failed experiments.
- To learn new libraries or techniques in isolation.

### Syntax Rules and Structure

**Complete General Syntax**

```
git switch -c experiment/<description>
# ... experiment freely ...
# If successful: merge or cherry-pick
# If failed: delete
git switch main
git branch -D experiment/<description>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `experiment/` | Branch name prefix (convention). |
| `-c` | Create and switch. |
| `-D` | Force delete (branch may be unmerged). |

**Syntax Rules**

- Branch from the current mainline.
- Never push experiment branches to shared remotes (unless collaborating).
- Use `-D` to delete unmerged experiment branches.
- Cherry-pick successful commits if merging the whole branch is undesirable.

**Constraints and Limitations**

- **Not shared**: Experiment branches are local-only.
- **No review**: No code review for experiment branches.
- **Time-boxed**: Spikes should have a time limit.

### Annotated Code Examples

#### Example 1: Experiment and Abandon

```bash
# Create experiment branch
git switch -c experiment/new-algorithm

# Experiment
echo "version 1" > algo.js
git add algo.js && git commit -m "Try algorithm v1"
echo "version 2" > algo.js
git commit -am "Try algorithm v2"

# Abandon: switch back and delete
git switch main
git branch -D experiment/new-algorithm
```

**Expected Output:**
```
Switched to a new branch 'experiment/new-algorithm'
[experiment/new-algorithm 1a2b3c4] Try algorithm v1
[experiment/new-algorithm 5d6e7f8] Try algorithm v2
Switched to branch 'main'
Deleted branch experiment/new-algorithm (was 5d6e7f8).
```

**Why this output occurs:** The experiment branch is created, commits are made, and then the branch is abandoned by switching back to `main` and force-deleting it. The commits become unreachable (recoverable via reflog).

### Real-World Cases

- **Performance spike**: A developer tests a new caching strategy in an experiment branch and discards it if performance doesn't improve.
- **Library evaluation**: A team experiments with a new framework in a throwaway branch before deciding to adopt it.
- **Proof of concept**: A developer builds a quick prototype in an experiment branch to demonstrate feasibility.

### References

- Git Documentation: Git Branching – Branches in a Nutshell — https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell
- University of Maryland: Git Branching — https://www.cs.umd.edu
- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch

---

## 5. Short-Lived Branches (Continuous Integration and Minimising Divergence)

### Definitions

**Core Definition**
Short-lived branches are feature or bug-fix branches that are merged back into the mainline within a very short time frame (typically hours to a couple of days), minimising divergence and merge conflicts.

**Technical Definition**
In trunk-based development, developers create short-lived feature branches from `main`, work in isolation, and merge back via pull request within 1–2 days. The short lifespan reduces the chance of diverging far from `main`, making merges simpler and conflicts smaller. Branches should ideally live less than 24 hours, with commits to `main` multiple times per day.

**Beginner-Friendly Explanation**
The longer you work on a separate branch, the harder it becomes to merge back. Short-lived branches solve this by keeping the branch lifetime short—hours or a day or two—so conflicts are small and merging is easy. This is the core philosophy of continuous integration.

### Purposes

- To minimise branch divergence from the mainline.
- To reduce merge conflicts and integration pain.
- To enable rapid feedback through frequent integration.
- To support continuous integration and delivery (CI/CD).
- To keep the mainline always deployable.

### Syntax Rules and Structure

**Complete General Syntax**

```
# Create short-lived branch
git switch -c feature/<small-change> main

# Work quickly (< 1 day)
git add .
git commit -m "feat: small change"

# Push and open PR immediately
git push -u origin feature/<small-change>

# Merge within hours
git switch main
git merge --no-ff feature/<small-change>
git branch -d feature/<small-change>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `feature/<small-change>` | Branch name (small scope). |
| `< 1 day` | Target lifetime. |
| `--no-ff` | Optional merge commit. |

**Syntax Rules**

- Branch from the latest `main`.
- Keep changes small and focused.
- Merge within 1–2 days.
- Rebase or merge `main` into the branch if it lives longer than a day.
- Delete the branch immediately after merging.

**Constraints and Limitations**

- **Discipline required**: Developers must resist scope creep.
- **Feature flags**: Large features may require feature flags to merge incomplete work.
- **Review speed**: Pull requests must be reviewed quickly.

### Annotated Code Examples

#### Example 1: One-Day Feature Branch

```bash
# Morning: create branch
git switch main
git pull
git switch -c feature/add-logging

# Work: add logging
echo "log" > logger.js
git add logger.js && git commit -m "feat: add request logging"

# Afternoon: push and open PR
git push -u origin feature/add-logging

# Same day: merge after review
git switch main
git merge --no-ff feature/add-logging
git branch -d feature/add-logging
```

**Expected Output:**
```
Switched to a new branch 'feature/add-logging'
[feature/add-logging 1a2b3c4] feat: add request logging
 1 file changed, 1 insertion(+)
...
Merge made by the 'ort' strategy.
...
Deleted branch feature/add-logging (was 1a2b3c4).
```

**Why this output occurs:** The branch is created in the morning, the change is made and pushed, and the merge happens the same day. The branch lifetime is only a few hours.

### Real-World Cases

- **Trunk-based development**: Teams at Google, Facebook, and Netflix use short-lived branches merged within hours.
- **CI/CD pipelines**: Every push triggers a pipeline; branches are merged after passing checks.
- **High-velocity teams**: Multiple merges to `main` per day.

### References

- Microsoft Learn: Apply Trunk-Based Development (Short-Lived Branches) — https://learn.microsoft.com/en-us/training/modules/trigger-azure-machine-learn-jobs-github-actions/
- Git-Tower: Understanding the Trunk-Based Development Workflow — https://www.git-tower.com/learn/git/faq/trunk-based-development
- Trunk Based Development — https://trunkbaseddevelopment.com/

---

## 6. Upstream Tracking and Branch Synchronisation (`git branch --set-upstream-to`)

### Definitions

**Core Definition**
Upstream tracking (also called branch tracking) is the configuration that links a local branch to a remote-tracking branch, enabling simplified `git pull` and `git push` operations and providing ahead/behind information.

**Technical Definition**
When a local branch is configured to track an upstream branch, Git knows which remote branch to push to and pull from by default. The upstream branch is sometimes referred to as "tracking information". Setting upstream is done with `git branch --set-upstream-to=<upstream>` or automatically with `git push -u origin <branch>`.

**Beginner-Friendly Explanation**
When you create a local branch, Git doesn't automatically know which remote branch it corresponds to. Upstream tracking creates that link, so you can just type `git pull` and `git push` without specifying the remote and branch every time. It's like pairing your phone with your car's Bluetooth—once paired, they connect automatically.

### Purposes

- To simplify `git pull` and `git push` commands.
- To track ahead/behind status relative to the remote.
- To enable `git status` to show branch tracking information.
- To synchronise local branches with remote counterparts.
- To support collaboration on shared branches.

### Syntax Rules and Structure

**Complete General Syntax**

```
git branch --set-upstream-to=<upstream> [<branch>]
git branch -u <upstream> [<branch>]
git push --set-upstream origin <branch>
git push -u origin <branch>
git branch --unset-upstream [<branch>]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--set-upstream-to=<upstream>` | Set upstream tracking. |
| `-u <upstream>` | Short form. |
| `<branch>` | Local branch (default: current). |
| `--unset-upstream` | Remove tracking. |

**Syntax Rules**

- `git branch -u origin/main` sets the current branch to track `origin/main`.
- `git push -u origin <branch>` pushes and sets upstream in one step.
- `git branch -vv` shows tracking information.
- `git status` shows ahead/behind counts for tracked branches.

**Constraints and Limitations**

- **Remote branch must exist**: The upstream branch must exist on the remote.
- **One upstream**: A local branch can only track one upstream branch.
- **Not automatic**: New local branches do not automatically track remote branches unless created from a remote-tracking branch.

### Annotated Code Examples

#### Example 1: Setting Upstream Tracking

```bash
# Create a local branch
git switch -c feature

# Push and set upstream
git push -u origin feature
# Output:
# To https://github.com/user/repo.git
#  * [new branch]      feature -> feature
# Branch 'feature' set up to track remote branch 'feature' from 'origin'.

# Verify tracking
git branch -vv
# Output:
# * feature 4d5e6f7 [origin/feature] Commit message
```

**Expected Output:**
```
To https://github.com/user/repo.git
 * [new branch]      feature -> feature
Branch 'feature' set up to track remote branch 'feature' from 'origin'.
* feature 4d5e6f7 [origin/feature] Commit message
```

**Why this output occurs:** `git push -u origin feature` pushes the branch and sets `origin/feature` as the upstream. The `-vv` output shows the tracking relationship in brackets.

#### Example 2: Changing Upstream

```bash
# Change upstream to a different remote branch
git branch -u origin/main
# Output: Branch 'feature' set up to track remote branch 'main' from 'origin'.
```

**Expected Output:**
```
Branch 'feature' set up to track remote branch 'main' from 'origin'.
```

**Why this output occurs:** `git branch -u origin/main` changes the upstream of the current branch (`feature`) to track `origin/main` instead of its previous upstream.

### Real-World Cases

- **Feature branch collaboration**: A developer pushes a feature branch and sets upstream so collaborators can pull it easily.
- **Fork workflow**: A developer sets `upstream/main` as the upstream to keep their fork in sync.
- **Remote branch rename**: After renaming a remote branch, upstream tracking is re-established.

### References

- Git Documentation: git-branch Manual Page (--set-upstream-to) — https://git-scm.com/docs/git-branch
- Git Documentation: Git Branching – Remote Branches — https://git-scm.com/book/en/v2/Git-Branching-Remote-Branches
- Raw GitHub: Upstream Branch Fields — https://raw.githubusercontent.com/posit-dev/btw/refs/tags/v1.2.1/R/utils-gitignore.R

---

## 7. Pruning Dead Local Branches (`git remote prune` / `git fetch --prune`)

### Definitions

**Core Definition**
Pruning is the process of removing local remote-tracking references to branches that have been deleted on the remote repository, keeping the local branch listing in sync with the remote.

**Technical Definition**
When a teammate deletes a branch on a shared remote, the branch remains in your local remote-tracking references until you explicitly prune it. `git fetch --prune` and `git remote prune <name>` both delete stale remote-tracking branches that no longer exist on the remote. Pruning only affects remote-tracking branches; local branches are never affected.

**Beginner-Friendly Explanation**
When someone deletes a branch on GitHub, your local Git still remembers it as a "remote branch." Pruning cleans up those stale references so your branch list is accurate. It's like cleaning out old contacts from your phone that no longer exist.

### Purposes

- To remove stale remote-tracking references.
- To keep the branch listing accurate and uncluttered.
- To avoid confusion from branches that no longer exist.
- To synchronise local state with remote deletions.
- To maintain repository hygiene.

### Syntax Rules and Structure

**Complete General Syntax**

```
git fetch --prune [<remote>]
git fetch -p [<remote>]
git remote prune [--dry-run] <name>
git config --global fetch.prune true
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--prune` / `-p` | Remove stale remote-tracking branches. |
| `--dry-run` | Show what would be pruned. |
| `<remote>` | Remote name (e.g., `origin`). |
| `fetch.prune` | Config to always prune on fetch. |

**Syntax Rules**

- `git fetch --prune` fetches new data and prunes in one operation.
- `git remote prune origin` prunes without fetching.
- `--dry-run` previews without deleting.
- `fetch.prune=true` enables automatic pruning on every fetch.
- Pruning does not affect local branches.

**Constraints and Limitations**

- **Local only**: Pruning removes remote-tracking references, not local branches.
- **Accidental deletion**: If a remote branch is accidentally deleted, pruning removes the local reference (recoverable via reflog on the server if enabled).
- **Not automatic**: Pruning is not enabled by default unless configured.

### Annotated Code Examples

#### Example 1: Pruning Stale Remote Branches

```bash
# A teammate deletes feature-x on the remote
# Your local remote-tracking reference still exists
git branch -r
# Output:
#   origin/main
#   origin/feature-x

# Prune stale references
git fetch --prune origin
# Output:
#  - [deleted]         (none)     -> origin/feature-x

# Verify
git branch -r
# Output:
#   origin/main
```

**Expected Output:**
```
  origin/main
  origin/feature-x
 - [deleted]         (none)     -> origin/feature-x
  origin/main
```

**Why this output occurs:** `git fetch --prune origin` fetches updates and removes `origin/feature-x` because it no longer exists on the remote. The `- [deleted]` line confirms the pruning.

#### Example 2: Dry-Run Preview

```bash
git remote prune origin --dry-run
# Output:
# * [would prune] origin/feature-x
```

**Expected Output:**
```
* [would prune] origin/feature-x
```

**Why this output occurs:** `--dry-run` shows what would be pruned without actually deleting anything. This is useful for previewing before running the real command.

#### Example 3: Global Auto-Prune Configuration

```bash
git config --global fetch.prune true
```

**Why this is useful:** Setting `fetch.prune` to `true` configures Git to automatically prune stale remote-tracking branches on every `git fetch`. This keeps the branch list clean without manual intervention.

### Real-World Cases

- **Team collaboration**: After a feature branch is merged and deleted on GitHub, `git fetch --prune` removes the stale reference locally.
- **CI/CD**: A CI script runs `git fetch --prune` before building to ensure accurate branch references.
- **Fork synchronisation**: A developer keeping a fork in sync uses `git fetch upstream --prune` to clean up stale upstream references.

### References

- Git-Tower: How to Use "prune" in Git to Clean Up Remote Branches — https://www.git-tower.com/learn/git/faq/cleanup-remote-branches-with-git-prune/
- Git Documentation: git-fetch Manual Page (--prune) — https://git-scm.com/docs/git-fetch
- Git Documentation: git-remote Manual Page (prune) — https://git-scm.com/docs/git-remote
- Stack Overflow: git remote prune vs git fetch --prune — https://stackoverflow.com/questions/17888695

---

## References

- Microsoft Learn: Git Branching Guidance — https://learn.microsoft.com/en-us/azure/devops/repos/git/git-branching-guidance
- AWS Prescriptive Guidance: Choosing a Git Branching Strategy — https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-git-branch-approach/
- Git-Tower: How to Use "prune" in Git to Clean Up Remote Branches — https://www.git-tower.com/learn/git/faq/cleanup-remote-branches-with-git-prune/
- Git-Tower: Understanding the Trunk-Based Development Workflow — https://www.git-tower.com/learn/git/faq/trunk-based-development
- Trunk Based Development — https://trunkbaseddevelopment.com/
- Git Documentation: git-branch Manual Page — https://git-scm.com/docs/git-branch
- Git Documentation: git-switch Manual Page — https://git-scm.com/docs/git-switch
- Git Documentation: git-fetch Manual Page — https://git-scm.com/docs/git-fetch
- Git Documentation: git-remote Manual Page — https://git-scm.com/docs/git-remote
- Git Documentation: Git Branching – Remote Branches — https://git-scm.com/book/en/v2/Git-Branching-Remote-Branches
- Microsoft Learn: Apply Trunk-Based Development (Short-Lived Branches) — https://learn.microsoft.com/en-us/training/modules/trigger-azure-machine-learn-jobs-github-actions/
- GitFlow Workflow Guide — https://github.com/aiskillstore/marketplace
- Stack Overflow: git remote prune vs git fetch --prune — https://stackoverflow.com/questions/17888695