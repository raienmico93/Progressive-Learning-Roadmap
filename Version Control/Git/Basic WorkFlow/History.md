# Git Viewing History: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Viewing history in Git is the practice of inspecting the repository's commit log, individual commit contents, and the evolution of files and lines over time using read-only commands such as `git log`, `git show`, and their many filtering and formatting options.

**Technical Definition**
Git stores history as a Directed Acyclic Graph (DAG) of commit objects in the object database. Viewing history involves traversing this DAG from one or more starting points (default: HEAD), optionally filtering by author, date, message, path, or content changes, and formatting the output for human or machine consumption. Commands like `git log` combine revision traversal (via `git rev-list`) with diff generation and pretty-printing.

**Beginner-Friendly Explanation**
When you work on a project with Git, every change you save is recorded as a "commit." Viewing history means asking Git to show you those commits—who made them, when, what they changed, and why. You can look at the entire history, search for a specific person's commits, find when a particular line of code was added, or see a visual map of how branches diverged and merged.

### Key Characteristics

- **Read-Only**: History viewing commands do not modify the repository.
- **Local**: Most history operations work entirely on local data, requiring no network access.
- **Extensible**: `git log` supports hundreds of options for filtering, formatting, and diffing.
- **DAG Traversal**: History is traversed by following parent pointers from one or more starting commits.
- **Composable**: Output can be piped to tools like `grep`, `awk`, `less`, or `wc` for further processing.

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository with at least one commit.
- Basic familiarity with the terminal and Git concepts (commits, branches, HEAD).

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **Software Archaeology**: Understanding how and why code evolved.
- **DevOps & CI/CD**: Generating changelogs and auditing changes.
- **Collaborative Development**: Code review and blame attribution.

### Core Concepts / Features

1. **git log** — The primary command for viewing commit history.
2. **One-Line History (`--oneline`)** — Compact commit summaries.
3. **Graph Visualization (`--graph`, `--all`, `--decorate`)** — Visual branch topology.
4. **Author Information** — Viewing who made each commit.
5. **Commit Timestamps** — Viewing and formatting dates.
6. **Filtering History** — By author, date, message, file path, pickaxe (`-S`), and line log (`-L`).
7. **git show** — Inspecting individual commits, metadata, and relative references.

---

## 1. git log

### Definitions

**Core Definition**
`git log` is the command that displays the commit history of a repository, showing commits in reverse-chronological order by default.

**Technical Definition**
`git log` traverses the commit DAG starting from the current HEAD (or specified revision range) and prints commit objects. By default, it shows each commit's full SHA-1 hash, author name and email, date, and commit message. It supports extensive options for limiting the set of commits shown (`--author`, `--since`, `--grep`, `-S`, `-L`) and for formatting the output (`--oneline`, `--graph`, `--stat`, `-p`). The command is a user-friendly front-end to `git rev-list` combined with `git diff`.

**Beginner-Friendly Explanation**
`git log` is your project's history book. Run it without arguments and it shows every commit made in the current branch, most recent first. Each entry includes a long ID (hash), who made the commit, when, and the commit message. You can add options to see a shorter version, a visual graph, or to filter by person, date, or message.

### Purposes

- To view the chronological history of commits in a repository.
- To understand the branch and merge structure of the project.
- To inspect what changed in each commit with `-p` or `--stat`.
- To search for specific commits by author, date, message, or content changes.
- To provide data for scripts, reports, and changelog generation.

### Syntax Rules and Structure

**Complete General Syntax**

```
git log [<options>] [<revision-range>] [[--] <path>...]
```

**Key Options**

| Option | Description |
|--------|-------------|
| `--oneline` | One line per commit: abbreviated hash + subject. |
| `--graph` | ASCII graph showing branch/merge structure. |
| `--decorate` | Show ref names (branches, tags) next to commits. |
| `--all` | Show commits from all branches and tags. |
| `--stat` | Show file change statistics per commit. |
| `-p` / `--patch` | Show full diff for each commit. |
| `--author=<pattern>` | Filter by author name/email. |
| `--since=<date>`, `--after=<date>` | Filter commits after a date. |
| `--until=<date>`, `--before=<date>` | Filter commits before a date. |
| `--grep=<pattern>` | Filter by commit message content. |
| `-S<string>` | Pickaxe: find commits that change the number of occurrences of a string. |
| `-L<start>,<end>:<file>` | Trace the evolution of a line range in a file. |
| `-n <number>` | Limit to the last `<number>` commits. |

**Syntax Rules**

- The `<revision-range>` can be a single commit (`HEAD`), a range (`A..B`), or multiple refs (`--all`).
- Path limiting restricts output to commits that touch the specified files.
- `--oneline` is a shorthand for `--pretty=oneline --abbrev-commit`.
- Options can be combined freely; later options override earlier ones when conflicting.

**Constraints and Limitations**

- **Performance**: On very large repositories, `git log -p` can be slow. Use `-n` to limit output.
- **Shallow clones**: In a shallow clone, history is truncated; `git log` shows only available commits.
- **Merge commits**: By default, `git log` simplifies merge history; use `--full-history` or `-m` to see all commits.
- **Binary files**: `-p` cannot show textual diffs for binary files.

### Annotated Code Examples

#### Example 1: Basic `git log`

```bash
# View the full commit history
git log
```

**Expected Output:**
```
commit 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
Author: John Doe <john@example.com>
Date:   Mon Oct 27 15:25:32 2014 -0400

    Add user authentication

commit 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
Author: Jane Smith <jane@example.com>
Date:   Sun Oct 26 10:15:00 2014 -0400

    Initial commit
```

**Why this output occurs:** `git log` lists commits in reverse-chronological order. Each entry includes the full SHA-1 hash, author name and email, date, and commit message. The most recent commit (`4d5e6f7...`) appears first.

### Real-World Cases

- **Code review**: A reviewer runs `git log --oneline` to see the sequence of commits in a pull request.
- **Release notes**: A maintainer runs `git log --since="1 month ago" --oneline` to generate release notes.
- **Debugging**: A developer uses `git log -p -- src/auth.js` to see all changes to a specific file.

### References

- Git Documentation: git-log Manual Page — https://git-scm.com/docs/git-log
- Pro Git Book: Viewing the Commit History — https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History

---

## 2. One-Line History (`--oneline`)

### Definitions

**Core Definition**
`git log --oneline` displays each commit on a single line, showing an abbreviated commit hash and the commit subject.

**Technical Definition**
`--oneline` is a shorthand for `--pretty=oneline --abbrev-commit`. It prints the abbreviated commit hash (usually 7 characters) followed by the first line of the commit message (the subject). This compact format is ideal for quickly scanning history and for scripting.

**Beginner-Friendly Explanation**
Instead of seeing a full page of details for each commit, `git log --oneline` shows just one line per commit: a short hash and the commit message. It's a quick way to see the "big picture" of your project's history.

### Purposes

- To quickly scan the sequence of commits.
- To provide a compact history for scripts and tools.
- To reduce terminal clutter when many commits are present.
- To combine with `--graph` for a visual overview.
- To generate abbreviated commit lists for reports.

### Syntax Rules and Structure

**Complete General Syntax**

```
git log --oneline [<options>] [<revision-range>] [[--] <path>...]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--oneline` | Shorthand for `--pretty=oneline --abbrev-commit`. |
| `--abbrev-commit` | Use abbreviated (short) commit hashes. |
| `--pretty=oneline` | Print hash and subject on one line. |

**Syntax Rules**

- The output format is: `<abbreviated-hash> <commit-subject>`.
- The hash length is configurable via `core.abbrev` (default: auto, usually 7 characters).
- `--oneline` can be combined with `--graph`, `--decorate`, and filtering options.
- The commit subject is the first line of the commit message, truncated at the first newline.

**Constraints and Limitations**

- **No body**: Only the subject line is shown; the body is omitted.
- **No metadata**: Author, date, and full hash are not shown by default.
- **Ambiguity**: Abbreviated hashes may become ambiguous in very large repositories; Git automatically extends them when necessary.

### Annotated Code Examples

#### Example 1: Compact History

```bash
git log --oneline
```

**Expected Output:**
```
4d5e6f7 Add user authentication
1a2b3c4 Fix login bug
9a0b1c2 Update README
3d4e5f6 Initial commit
```

**Why this output occurs:** Each commit is shown on one line with its abbreviated hash and subject. This makes it easy to scan the history quickly.

#### Example 2: `--oneline` with `--decorate`

```bash
git log --oneline --decorate
```

**Expected Output:**
```
4d5e6f7 (HEAD -> main) Add user authentication
1a2b3c4 Fix login bug
9a0b1c2 (tag: v1.0) Update README
3d4e5f6 Initial commit
```

**Why this output occurs:** `--decorate` adds ref names (branch, tag) in parentheses after the hash, showing which commits are pointed to by branches or tags.

### Real-World Cases

- **Quick overview**: A developer runs `git log --oneline` to see the last few commits before starting work.
- **Scripting**: A CI script uses `git log --oneline -1` to get the latest commit hash for tagging a build.
- **Changelog generation**: `git log --oneline v1.0..v2.0` lists all commits between two tags.

### References

- Git Documentation: git-log (--oneline) — https://git-scm.com/docs/git-log#Documentation/git-log.txt---oneline
- Git Documentation: Pretty Formats — https://git-scm.com/docs/git-log#_pretty_formats

---

## 3. Graph Visualization (`--graph`, `--all`, `--decorate`)

### Definitions

**Core Definition**
`git log --graph` draws a text-based graphical representation of the commit history, showing the branching and merging structure of the DAG.

**Technical Definition**
The `--graph` option prepends an ASCII art graph to each commit line, using characters like `*`, `|`, `/`, and `\` to represent commits, branch lines, and merges. When combined with `--oneline` and `--all`, it provides a compact yet complete visual map of all branches and their relationships. `--decorate` adds ref names (branches, tags) to the graph.

**Beginner-Friendly Explanation**
`git log --graph` turns your text-based history into a picture. It shows how branches split apart and come back together (merge). It's like a family tree for your commits. Adding `--all` shows all branches, not just the current one.

### Purposes

- To visualise the branch and merge topology of the repository.
- To understand how feature branches diverge from and merge into the main branch.
- To identify merge commits and their parents.
- To provide a quick overview of the entire project's history structure.
- To help debug branching issues and merge conflicts.

### Syntax Rules and Structure

**Complete General Syntax**

```
git log --graph [--oneline] [--all] [--decorate] [<options>]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--graph` | Draw a text-based graph of the commit history. |
| `--oneline` | Compact format (recommended with `--graph`). |
| `--all` | Include all branches, tags, and remotes. |
| `--decorate` | Show ref names (branches, tags) next to commits. |
| `--pretty=format:"%h %s"` | Custom format for more control. |

**Syntax Rules**

- `--graph` can be combined with any other `git log` options.
- `--all` includes all refs under `refs/`, making the graph show the full repository topology.
- `--decorate` is often used with `--graph` to label branch tips.
- The graph is drawn using ASCII characters: `*` for commits, `|` for vertical lines, `/` and `\` for diagonal branch lines.

**Constraints and Limitations**

- **Terminal width**: Very wide graphs may wrap or become unreadable in narrow terminals.
- **Complexity**: Repositories with many concurrent branches produce complex graphs that are hard to read.
- **Performance**: `--graph --all` on large repositories can be slow.
- **Merge simplification**: By default, `--graph` may simplify merge history; use `--full-history` to see all commits.

### Annotated Code Examples

#### Example 1: Basic Graph

```bash
# Create a branch and merge
git init graph-demo && cd graph-demo
echo "A" > file.txt && git add . && git commit -m "Commit A"
echo "B" >> file.txt && git commit -am "Commit B"
git checkout -b feature
echo "C" >> file.txt && git commit -am "Commit C"
git checkout main
git merge feature --no-edit

git log --graph --oneline --all
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

**Why this output occurs:** `--graph` draws the DAG: the merge commit (`4d5e6f7`) has two parents (Commit B and Commit C), shown by the `|\` and `|/` lines. `--oneline` compresses each commit to one line. `--all` includes the `feature` branch.

#### Example 2: Graph with Decorations

```bash
git log --graph --oneline --decorate --all
```

**Expected Output:**
```
*   4d5e6f7 (HEAD -> main, origin/main) Merge branch 'feature'
|\
| * 1a2b3c4 (feature) Commit C
* | 9a0b1c2 Commit B
|/
* 3d4e5f6 (tag: v1.0) Commit A
```

**Why this output occurs:** `--decorate` adds branch and tag names in parentheses. `HEAD -> main` shows the current branch, `origin/main` shows the remote-tracking branch, and `tag: v1.0` shows a tag pointing to Commit A.

### Real-World Cases

- **Understanding branch structure**: A developer joins a project and runs `git log --graph --oneline --all` to see how the project has evolved.
- **Debugging merges**: A merge conflict occurs; the developer uses `--graph` to see which branches diverged and where the merge base is.
- **Release management**: A release manager uses `--graph --decorate` to see which commits are on the release branch versus the main branch.

### References

- Git Documentation: git-log (--graph) — https://git-scm.com/docs/git-log#Documentation/git-log.txt---graph
- Git Documentation: git-log (--decorate) — https://git-scm.com/docs/git-log#Documentation/git-log.txt---decorate
- CoreUI: How to view Git log graph — https://coreui.io

---

## 4. Author Information

### Definitions

**Core Definition**
Author information in Git history identifies who wrote each change (the author) and who created the commit (the committer).

**Technical Definition**
Each commit object stores two identity fields: `author` (the person who originally wrote the change) and `committer` (the person who applied the commit). By default, `git log` shows the author. The `--author` and `--committer` options filter commits by these fields using regular expressions. The `--format` option can display author and committer names, emails, and timestamps.

**Beginner-Friendly Explanation**
Every commit records who made it. Usually the author and committer are the same person, but they differ when you rebase, cherry-pick, or apply someone else's patch. `git log` shows the author by default, and you can filter by author with `--author="Name"`.

### Purposes

- To see who made each commit.
- To filter history by a specific person.
- To attribute changes for code review or auditing.
- To understand collaboration patterns.
- To generate contributor statistics.

### Syntax Rules and Structure

**Complete General Syntax**

```
git log --author=<pattern> [<options>]
git log --committer=<pattern> [<options>]
git log --format="%an %ae %cn %ce" [<options>]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `--author=<pattern>` | Filter by author name or email (regex). |
| `--committer=<pattern>` | Filter by committer name or email (regex). |
| `%an` | Author name. |
| `%ae` | Author email. |
| `%cn` | Committer name. |
| `%ce` | Committer email. |
| `%ad` | Author date. |
| `%cd` | Committer date. |

**Syntax Rules**

- The pattern is a regular expression matched against the author/committer name and email.
- Use `--author="John"` to match any author containing "John".
- Use `--author="@example.com"` to match any author with that email domain.
- `--format` can be combined with `--author` to customise output.

**Constraints and Limitations**

- **Regex matching**: The pattern is a regex, so special characters must be escaped.
- **Case sensitivity**: Matching is case-sensitive by default; use `-i` for case-insensitive.
- **Author vs. committer**: Filtering by `--author` does not filter by committer; use `--committer` separately.

### Annotated Code Examples

#### Example 1: Filter by Author

```bash
git log --author="Jane" --oneline
```

**Expected Output:**
```
1a2b3c4 Fix login bug
9a0b1c2 Update README
```

**Why this output occurs:** Only commits where the author name or email contains "Jane" are shown. Other commits are filtered out.

#### Example 2: Custom Author Format

```bash
git log --format="%h %an <%ae> %s" -5
```

**Expected Output:**
```
4d5e6f7 John Doe <john@example.com> Add user authentication
1a2b3c4 Jane Smith <jane@example.com> Fix login bug
9a0b1c2 Jane Smith <jane@example.com> Update README
3d4e5f6 John Doe <john@example.com> Initial commit
```

**Why this output occurs:** The `--format` string specifies: abbreviated hash (`%h`), author name (`%an`), author email in angle brackets (`<%ae>`), and subject (`%s`). This provides a custom, compact author view.

### Real-World Cases

- **Code review**: A reviewer runs `git log --author="Alice"` to see all changes made by a specific contributor.
- **Auditing**: A compliance officer runs `git log --format="%h %an %ae %ad %s"` to generate an audit trail.
- **Contributor statistics**: `git shortlog -sn` provides a summary of commits per author.

### References

- Git Documentation: git-log (--author) — https://git-scm.com/docs/git-log#Documentation/git-log.txt---authorltpatterngt
- Git Documentation: Pretty Formats (%an, %ae) — https://git-scm.com/docs/git-log#_pretty_formats

---

## 5. Commit Timestamps

### Definitions

**Core Definition**
Commit timestamps record when a commit was authored and when it was committed, stored as Unix timestamps with timezone offsets.

**Technical Definition**
Each commit object stores two timestamps: the **author date** (when the change was originally written) and the **committer date** (when the commit was created). Both are Unix epoch timestamps with a timezone offset (e.g., `1700000000 +0000`). `git log` displays dates in a human-readable format by default, and supports various date formats via `--date=`.

**Beginner-Friendly Explanation**
Every commit records two times: when the code was written (author date) and when the commit was actually made (committer date). These are usually the same, but differ during rebases or patch applications. You can see and format these dates with `git log --date=`.

### Purposes

- To see when each commit was made.
- To filter commits by date range.
- To understand the timeline of project development.
- To format dates for reports or scripts.
- To distinguish author date from committer date.

### Syntax Rules and Structure

**Complete General Syntax**

```
git log --date=<format> [<options>]
git log --since=<date> --until=<date> [<options>]
```

**Date Format Options**

| Format | Description |
|--------|-------------|
| `relative` | Relative to now (e.g., “2 weeks ago”). |
| `iso` | ISO 8601 format (e.g., `2024-01-15 10:30:00 +0000`). |
| `rfc` | RFC 2822 format. |
| `short` | Short format (e.g., `2024-01-15`). |
| `raw` | Raw Unix timestamp with timezone. |
| `human` | Human-readable (e.g., `Mon Jan 15 10:30:00 2024 +0000`). |
| `unix` | Unix epoch seconds. |
| `format:<string>` | Custom strftime format. |

**Filtering Options**

| Option | Description |
|--------|-------------|
| `--since=<date>` / `--after=<date>` | Show commits more recent than `<date>`. |
| `--until=<date>` / `--before=<date>` | Show commits older than `<date>`. |
| `--since-as-filter=<date>` | Show all commits more recent than `<date>` (visits all commits). |

**Syntax Rules**

- `<date>` can be an absolute date (e.g., `2024-01-15`), a relative date (e.g., `2 weeks ago`), or a Unix timestamp.
- `--since` and `--until` can be combined to specify a range.
- `--date=<format>` controls how dates are displayed, not how they are filtered.
- The author date is shown by default; use `--format="%cd"` for committer date.

**Constraints and Limitations**

- **Timezone**: Dates are stored with timezone offsets; filtering uses the specified timezone or local time.
- **Relative dates**: “2 weeks ago” is interpreted relative to the current system time.
- **Date parsing**: Git accepts many date formats, but ambiguous formats may be misinterpreted.

### Annotated Code Examples

#### Example 1: Viewing Dates

```bash
git log --date=short --format="%h %ad %s" -3
```

**Expected Output:**
```
4d5e6f7 2024-01-15 Add user authentication
1a2b3c4 2024-01-14 Fix login bug
9a0b1c2 2024-01-13 Update README
```

**Why this output occurs:** `--date=short` formats the author date as `YYYY-MM-DD`. The `--format` string shows abbreviated hash (`%h`), author date (`%ad`), and subject (`%s`).

#### Example 2: Filtering by Date Range

```bash
git log --since="2024-01-01" --until="2024-01-31" --oneline
```

**Expected Output:**
```
4d5e6f7 Add user authentication
1a2b3c4 Fix login bug
```

**Why this output occurs:** Only commits with author dates between January 1 and January 31, 2024 are shown.

### Real-World Cases

- **Release notes**: A maintainer runs `git log --since="1 month ago" --oneline` to generate release notes for the last month.
- **Auditing**: `git log --date=iso --format="%h %ad %an %s"` produces an ISO-formatted audit trail.
- **Debugging**: A developer runs `git log --since="2 days ago" --oneline` to see recent changes.

### References

- Git Documentation: git-log (--date) — https://git-scm.com/docs/git-log#Documentation/git-log.txt---dateltformatgt
- Git Documentation: git-log (--since, --until) — https://git-scm.com/docs/git-log#Documentation/git-log.txt---sinceltdategt
- Git Documentation: Date Formats — https://git-scm.com/docs/git-log#_date_formats

---

## 6. Filtering History

### Definitions

**Core Definition**
Filtering history is the practice of narrowing the set of commits shown by `git log` using criteria such as author, date, commit message, file path, or content changes.

**Technical Definition**
`git log` supports a wide range of filtering options that act as predicates on the commit DAG traversal. Filters can be combined; each additional filter further restricts the output. Filters include author/committer patterns, date ranges, message grep, path limiting, pickaxe (`-S`/`-G`), and line log (`-L`).

**Beginner-Friendly Explanation**
When a project has thousands of commits, you don't want to see them all. You can tell Git to show only commits by a certain person, in a certain date range, with a certain word in the message, or that changed a specific file. This is called filtering history, and it's how you find the commits you care about.

### 6.1 Filtering by Author (`--author`)

**Syntax:**
```bash
git log --author="<pattern>" [<options>]
```

**Example:**
```bash
git log --author="Jane" --since="2024-01-01" --oneline
```

**Output:**
```
1a2b3c4 Fix login bug
9a0b1c2 Update README
```

**Explanation:** Filters commits to those authored by “Jane” since January 1, 2024. The pattern is a regular expression.

### 6.2 Filtering by Date (`--since`, `--until`, `--before`, `--after`)

**Syntax:**
```bash
git log --since="<date>" --until="<date>" [<options>]
```

**Example:**
```bash
git log --after="2024-01-01" --before="2024-02-01" --oneline
```

**Output:**
```
4d5e6f7 Add user authentication
1a2b3c4 Fix login bug
```

**Explanation:** Shows commits with author dates between January 1 and February 1, 2024. `--after` is a synonym for `--since`; `--before` is a synonym for `--until`.

### 6.3 Filtering by Message (`--grep`)

**Syntax:**
```bash
git log --grep="<pattern>" [<options>]
```

**Example:**
```bash
git log --grep="fix" --oneline -i
```

**Output:**
```
1a2b3c4 Fix login bug
5d6e7f8 Fixed typo in README
```

**Explanation:** Filters commits whose message contains “fix” (case-insensitive with `-i`). The pattern is a regular expression.

### 6.4 Filtering by File Path

**Syntax:**
```bash
git log [<options>] -- <path>
```

**Example:**
```bash
git log --oneline -- src/auth.js
```

**Output:**
```
4d5e6f7 Add user authentication
1a2b3c4 Fix login bug
```

**Explanation:** Shows only commits that touch the file `src/auth.js`. The `--` separates options from paths.

### 6.5 Pickaxe (`-S` and `-G`)

**Syntax:**
```bash
git log -S"<string>" [<options>]
git log -G"<regex>" [<options>]
```

**Technical Definition:** The `-S` option (the “pickaxe”) finds commits that change the **number of occurrences** of the specified string in the project. The `-G` option finds commits whose diffs contain the specified regular expression. These are powerful tools for finding when a particular piece of code was added, removed, or modified.

**Example:**
```bash
git log -S"calculateTotal" --oneline
```

**Output:**
```
4d5e6f7 Add total calculation
1a2b3c4 Refactor pricing logic
```

**Explanation:** `-S"calculateTotal"` finds commits where the string `calculateTotal` was added or removed. The commit that introduced the function and the commit that removed it would both appear.

### 6.6 Line Log (`-L`)

**Syntax:**
```bash
git log -L<start>,<end>:<file> [<options>]
git log -L:<funcname>:<file> [<options>]
```

**Technical Definition:** The `-L` option traces the evolution of a specific line range or function within a file, showing every commit that modified those lines. It implies `--patch` (showing diffs for each commit).

**Example:**
```bash
git log -L:calculateTotal:src/pricing.js
```

**Output:**
```
commit 4d5e6f7...
Author: John Doe <john@example.com>
Date:   ...

    Add total calculation

diff --git a/src/pricing.js b/src/pricing.js
@@ -10,6 +10,10 @@
+function calculateTotal(items) {
+  return items.reduce((sum, item) => sum + item.price, 0);
+}
```

**Explanation:** `-L:calculateTotal:src/pricing.js` shows the history of the `calculateTotal` function: when it was added, modified, or removed. The function name is matched using the same heuristics as diff hunk headers.

### Real-World Cases

- **Finding when a bug was introduced**: A developer uses `-S"buggyFunction"` to find the commit that added the function.
- **Tracing a feature's history**: `-L:featureFunction:src/feature.js` shows every change to a specific function.
- **Auditing a file**: `git log --oneline -- config.yaml` shows all commits that changed the configuration file.
- **Searching commit messages**: `git log --grep="JIRA-123"` finds all commits referencing a ticket.

### References

- Git Documentation: git-log (Commit Limiting) — https://git-scm.com/docs/git-log#_commit_limiting
- Git Documentation: git-log (History Simplification) — https://git-scm.com/docs/git-log#_history_simplification
- Git Documentation: git-log (-L) — https://git-scm.com/docs/git-log#Documentation/git-log.txt--Lltstartgtltendgtltfilegt
- Git Documentation: git-log (-S, -G) — https://git-scm.com/docs/git-log#Documentation/git-log.txt--Sltstringgt

---

## 7. git show (Inspecting Specific Commits, Metadata, and Absolute vs. Relative References)

### Definitions

**Core Definition**
`git show` is a command that displays various Git objects (commits, tags, trees, blobs) in a human-readable format, defaulting to the most recent commit.

**Technical Definition**
`git show <object>` resolves the given object and prints its content. For commits, it shows the commit metadata (hash, author, committer, date), the commit message, and the diff against the first parent (or a combined diff for merges). For tags, it shows the tag message and the referenced object. For trees, it lists entries. For blobs, it prints the raw content. Objects can be specified using absolute references (full or abbreviated SHA-1), relative references (`HEAD~1`, `HEAD^`), or message-based references (`:/pattern`).

**Beginner-Friendly Explanation**
`git show` lets you examine a single commit in detail. Run it without arguments to see the latest commit: its message, who made it, when, and exactly what changed. You can also point it at any commit using a hash or a relative reference like `HEAD~1` (the commit before the latest one). It's like a magnifying glass for individual commits.

### Purposes

- To inspect a specific commit's metadata and changes.
- To view the content of a file at a specific revision.
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
| `<commit-hash>` | A specific commit by full or abbreviated SHA-1. |
| `<tag>` | An annotated tag. |
| `<commit>:<path>` | A blob at a specific path in a commit. |
| `<commit>^{tree}` | The tree object of a commit. |
| `HEAD~<n>` | The commit `n` generations before HEAD (follows first parent). |
| `HEAD^<n>` | The `n`-th parent of HEAD (for merges). |
| `:/<pattern>` | The most recent commit whose message matches `<pattern>`. |

**Relative Reference Syntax**

| Reference | Meaning |
|-----------|---------|
| `HEAD~1` | The parent of HEAD. |
| `HEAD~2` | The grandparent of HEAD. |
| `HEAD^` | The first parent of HEAD (same as `HEAD~1`). |
| `HEAD^2` | The second parent of a merge commit. |
| `HEAD~2^2` | The second parent of the grandparent. |

**Key Options**

| Option | Description |
|--------|-------------|
| `--stat` | Show diffstat instead of full patch. |
| `--oneline` | One line per object. |
| `-p` / `--patch` | Show patch (default for commits). |
| `--pretty=<format>` | Custom commit format. |

**Syntax Rules**

- If no object is specified, `HEAD` is used.
- For commits, the output includes commit metadata, message, and diff.
- For annotated tags, the output includes the tag object and the referenced commit.
- For trees, the output lists entries with mode, type, hash, and name.
- Abbreviated hashes must be unique; Git reports an error if ambiguous.

**Constraints and Limitations**

- **Binary blobs**: `git show` may output binary data; use `--textconv` or redirect to a file.
- **Merge commits**: By default, `git show` may show a combined diff; use `-m` to show individual parents.
- **Ambiguous abbreviations**: Short hashes may be ambiguous in large repositories; Git extends them automatically or reports an error.
- **Relative references**: `~` follows the first parent; for merge commits, use `^2` to access the second parent.

### Annotated Code Examples

#### Example 1: Inspecting the Latest Commit

```bash
git show
```

**Expected Output:**
```
commit 4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e
Author: John Doe <john@example.com>
Date:   Mon Oct 27 15:25:32 2014 -0400

    Add user authentication

diff --git a/src/auth.js b/src/auth.js
...
```

**Why this output occurs:** `git show` with no arguments displays the HEAD commit: its full hash, author, date, message, and the diff against its parent.

#### Example 2: Using Relative References

```bash
# Show the parent of HEAD
git show HEAD~1

# Show the grandparent of HEAD
git show HEAD~2

# Show the second parent of a merge commit
git show HEAD^2
```

**Expected Output (HEAD~1):**
```
commit 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
Author: Jane Smith <jane@example.com>
Date:   Sun Oct 26 10:15:00 2014 -0400

    Fix login bug
...
```

**Why this output occurs:** `HEAD~1` refers to the first parent of HEAD. `HEAD~2` goes back two generations. `HEAD^2` refers to the second parent of a merge commit (used when inspecting merges).

#### Example 3: Inspecting a Blob at a Specific Commit

```bash
git show HEAD~2:src/auth.js
```

**Expected Output:**
```
// Content of auth.js as it was two commits ago
function login() {
  // ...
}
```

**Why this output occurs:** The syntax `<commit>:<path>` tells Git to show the blob object for `src/auth.js` at the commit `HEAD~2`. Only the file content is printed.

#### Example 4: Message-Based Reference

```bash
git show :/Fix login bug
```

**Expected Output:**
```
commit 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
Author: Jane Smith <jane@example.com>
Date:   Sun Oct 26 10:15:00 2014 -0400

    Fix login bug
...
```

**Why this output occurs:** `:/Fix login bug` finds the most recent commit whose message matches the pattern. This is useful when you remember the message but not the hash.

### Real-World Cases

- **Verifying a tag**: A release manager runs `git show v1.0.0` to confirm the tag points to the correct commit.
- **Examining old code**: `git show abc123:src/main.c` views the file as it was in an older commit.
- **Debugging a merge**: `git show -m <merge-commit>` shows diffs against each parent, helping understand merge resolution.
- **Recovering from a detached HEAD**: `git show HEAD` confirms the current commit before creating a branch.

### References

- Git Documentation: git-show Manual Page — https://git-scm.com/docs/git-show
- Git Documentation: gitrevisions (Object Specifiers) — https://git-scm.com/docs/gitrevisions
- Thoughtbot: Goal-Oriented Git – Refer to Commits — https://books.thoughtbot.com/assets/goal-oriented-git.pdf

---

## References

- Git Documentation: git-log Manual Page — https://git-scm.com/docs/git-log
- Git Documentation: git-show Manual Page — https://git-scm.com/docs/git-show
- Git Documentation: git-diff Manual Page — https://git-scm.com/docs/git-diff
- Git Documentation: git-rev-list Manual Page — https://git-scm.com/docs/git-rev-list
- Git Documentation: gitrevisions Manual Page — https://git-scm.com/docs/gitrevisions
- Pro Git Book: Viewing the Commit History — https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History
- Pro Git Book: Git Basics – Viewing the Commit History — https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History
- CoreUI: How to view Git log graph — https://coreui.io
- Thoughtbot: Goal-Oriented Git (Refer to Commits) — https://books.thoughtbot.com/assets/goal-oriented-git.pdf
- Mintlify: git log Documentation — https://mintlify.wiki/git/git/commands/log