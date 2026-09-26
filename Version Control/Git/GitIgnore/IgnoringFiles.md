# Git Ignoring Files: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Ignoring files in Git is the practice of configuring Git to exclude specific files and directories from tracking, staging, and commit operations, using pattern-matching rules defined in `.gitignore` files or other exclude sources.

**Technical Definition**
Git's ignore mechanism reads pattern specifications from multiple sources—`.gitignore` files in the working tree, `$GIT_DIR/info/exclude`, and the file specified by `core.excludesFile`—and applies them to determine whether untracked paths should be excluded from Git's view. Patterns use a syntax derived from shell globs but with Git-specific extensions (`**`, `!`, trailing `/`). Ignore rules affect only untracked files; already-tracked files are not affected and must be explicitly untracked with `git rm --cached`.

**Beginner-Friendly Explanation**
Sometimes you have files in your project folder that you never want Git to save or share—like build outputs, temporary files, or your editor's configuration. Git lets you create a "list of things to ignore" (called `.gitignore`), and Git will pretend those files don't exist. You can have project-wide ignore rules, personal ignore rules, or ignore rules that apply only to your computer.

### Key Characteristics

- **Pattern-Based**: Uses glob patterns with Git-specific extensions.
- **Multi-Source**: Reads from `.gitignore`, `.git/info/exclude`, and global excludes file.
- **Precedence-Ordered**: Later patterns within a file override earlier ones; more specific sources override less specific ones.
- **Untracked-Only**: Ignore rules affect only untracked files; tracked files remain tracked.
- **Directional**: Patterns can match at any depth or be anchored to specific directories.

### Prerequisites

- Git installed (version 2.28+ recommended for full feature support).
- A Git repository (initialised or cloned).
- Basic familiarity with the terminal and text editors.
- Understanding of Git's three-tree architecture (working tree, index, HEAD).

### Related Programming Areas

- **Version Control Systems (VCS)** : Git, Mercurial, Subversion.
- **DevOps & CI/CD**: Excluding build artefacts, environment files, and secrets.
- **Software Development**: Keeping repositories clean and focused.
- **Collaborative Development**: Sharing project-wide ignore conventions.

### Core Concepts / Features

1. **.gitignore Syntax, Precedence, and Scope** — Root vs. nested subdirectories.
2. **Ignore Patterns** — Literal file matching.
3. **Directory Patterns** — Trailing slash matching.
4. **Wildcards** — Single `*` vs. recursive `**` asterisks.
5. **Negation Patterns** — The `!` prefix and its structural ordering limitations.
6. **Global Ignore Rules** — `core.excludesfile` for personal environment pollution.
7. **Local Project-Only Ignores** — The `.git/info/exclude` file.

---

## 1. .gitignore Syntax, Precedence Rules, and Scope

### Definitions

**Core Definition**
A `.gitignore` file is a text file containing patterns that tell Git which untracked files and directories to ignore. It can be placed at the repository root or in any subdirectory, and its patterns apply relative to its location.

**Technical Definition**
Each line in a `.gitignore` file specifies a pattern. When deciding whether to ignore a path, Git checks patterns from multiple sources with the following order of precedence (highest to lowest): (1) patterns read from the command line for commands that support them, (2) patterns from a `.gitignore` file in the same directory as the path or in any parent directory (up to the top-level working tree), with patterns in higher-level files overridden by those in lower-level files down to the directory containing the file, (3) patterns from `$GIT_DIR/info/exclude`, and (4) patterns from the file specified by `core.excludesFile`. Within one level of precedence, the last matching pattern decides the outcome.

**Beginner-Friendly Explanation**
A `.gitignore` file is like a "do not disturb" list for Git. You put it in your project folder, and it lists files or patterns you don't want Git to track. If you have multiple `.gitignore` files (say, one in the root and one in a subfolder), the rules in the deeper folder take precedence for files in that folder. The most specific rule always wins.

### Purposes

- To prevent accidental tracking of generated or sensitive files.
- To keep the repository clean and focused on source code.
- To share ignore conventions with all collaborators via committed `.gitignore` files.
- To allow per-directory customisation of ignore rules.
- To reduce noise in `git status` output.

### Syntax Rules and Structure

**Complete General Syntax (Pattern Format)**

```
# Comment line
pattern
!negated-pattern
/root-anchored-pattern
**/any-depth-pattern
```

**Precedence Order (Highest to Lowest)**

| Priority | Source | Description |
|----------|--------|-------------|
| 1 | Command-line patterns | Patterns passed to commands that support them. |
| 2 | `.gitignore` in same directory | Patterns in the deepest `.gitignore` win. |
| 3 | `.gitignore` in parent directories | Overridden by deeper `.gitignore` files. |
| 4 | `$GIT_DIR/info/exclude` | Repository-local, not shared. |
| 5 | `core.excludesFile` | Global personal ignore file. |

**Syntax Rules**

- Blank lines match no files and serve as separators.
- Lines starting with `#` are comments; use `\#` to match a literal `#`.
- Trailing spaces are ignored unless quoted with backslash.
- Patterns are relative to the location of the `.gitignore` file.
- A leading slash anchors the pattern to the `.gitignore` directory.
- A trailing slash restricts the match to directories only.

**Constraints and Limitations**

- **Untracked only**: Ignore rules do not affect files already tracked by Git. Use `git rm --cached` to untrack.
- **Parent directory exclusion**: It is not possible to re-include a file if a parent directory of that file is excluded.
- **Performance**: Git does not list excluded directories for performance reasons, so patterns on contained files have no effect.

### Annotated Code Examples

#### Example 1: Root vs. Nested .gitignore Precedence

```bash
# Root .gitignore
echo "*.log" > .gitignore

# Create a subdirectory with its own .gitignore
mkdir app && echo "!important.log" > app/.gitignore

# Create log files
echo "root log" > debug.log
echo "app log" > app/debug.log
echo "important app log" > app/important.log

# Check status
git status -s
```

**Expected Output:**
```
?? app/important.log
```

**Why this output occurs:** The root `.gitignore` ignores all `*.log` files. The `app/.gitignore` contains `!important.log`, which negates the ignore for `app/important.log` only. The nested `.gitignore` takes precedence for files in `app/`. `app/debug.log` is ignored by the root rule; `root/debug.log` is also ignored. Only `app/important.log` appears as untracked.

### Real-World Cases

- **Monorepo with mixed stacks**: A root `.gitignore` ignores `node_modules/`; a `backend/` subdirectory adds `*.pyc`; a `frontend/` subdirectory adds `.next/`.
- **Open-source project**: The maintainer commits a `.gitignore` with common patterns for the project's language, ensuring all contributors ignore build outputs.
- **Personal workflow**: A developer adds a `.gitignore` to a subdirectory to ignore their local configuration files without affecting the rest of the team.

### References

- Git Documentation: gitignore Manual Page — https://git-scm.com/docs/gitignore
- Git Documentation: git-check-ignore Manual Page — https://git-scm.com/docs/git-check-ignore
- GitHub Docs: Ignoring Files — https://docs.github.com/en/get-started/git-basics/ignoring-files

---

## 2. Ignore Patterns (Literal File Matching)

### Definitions

**Core Definition**
A literal ignore pattern matches files or directories by their exact name, without wildcards, matching at any depth unless anchored with a slash.

**Technical Definition**
A pattern without a slash (or with only a trailing slash) is matched against all files and folders recursively. If the pattern contains a slash at the beginning or middle, it is relative to the directory level of the `.gitignore` file. Otherwise, the pattern may match at any level below the `.gitignore` level.

**Beginner-Friendly Explanation**
If you write `debug.log` in your `.gitignore`, Git will ignore any file or folder named `debug.log` anywhere in your project. It's a simple, exact name match.

### Purposes

- To ignore specific files by exact name.
- To ignore files with common names at any depth.
- To avoid the complexity of wildcards when a name is sufficient.
- To target specific files without affecting similarly named files.
- To provide clear, readable ignore rules.

### Syntax Rules and Structure

**Complete General Syntax**

```
filename
filename.ext
```

**Syntax Rules**

- A pattern with no slash matches at any depth below the `.gitignore` file.
- A pattern with a leading slash (`/filename`) matches only at the `.gitignore` directory level.
- A pattern with a slash in the middle (`dir/filename`) matches relative to the `.gitignore` directory.
- Literal matching is case-sensitive on case-sensitive filesystems.

**Constraints and Limitations**

- **Partial matches**: `debug.log` does not match `debug.log.old` unless a wildcard is used.
- **Case sensitivity**: On case-insensitive filesystems (macOS, Windows), matching may be case-insensitive.
- **Depth**: Without anchoring, the pattern matches at any depth, which may be broader than intended.

### Annotated Code Examples

#### Example 1: Ignoring a Specific File at Any Depth

```bash
# .gitignore
echo "debug.log" > .gitignore

# Create files at different depths
echo "root" > debug.log
mkdir src && echo "nested" > src/debug.log
echo "different" > debug.log.old

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
?? debug.log.old
```

**Why this output occurs:** `debug.log` matches both the root and nested files, ignoring them. `debug.log.old` does not match the literal pattern `debug.log`, so it remains untracked.

### Real-World Cases

- **Ignoring `.DS_Store`**: macOS creates `.DS_Store` files in every directory; a single pattern `*.DS_Store` or `.DS_Store` ignores them all.
- **Ignoring `Thumbs.db`**: Windows creates `Thumbs.db` files; a pattern `Thumbs.db` ignores them at any depth.
- **Ignoring a specific configuration file**: `local.properties` (Android) is ignored by name.

### References

- Git Documentation: gitignore (Pattern Format) — https://git-scm.com/docs/gitignore#_pattern_format
- Stack Overflow: Difference Between /dir and /dir/ — https://stackoverflow.com/questions/17888695

---

## 3. Directory Patterns (Trailing Slash Matching)

### Definitions

**Core Definition**
A directory pattern is an ignore pattern ending with a slash (`/`), which matches only directories and paths underneath them, not regular files or symbolic links with the same name.

**Technical Definition**
If the pattern ends with a slash, it is removed for the purpose of matching, but the pattern will only find a match with a directory. For example, `foo/` will match a directory `foo` and paths underneath it, but will not match a regular file or a symbolic link named `foo`. A pattern without a trailing slash matches files, directories, and symlinks alike.

**Beginner-Friendly Explanation**
If you write `build/` in your `.gitignore`, Git will ignore the `build` directory and everything inside it—but only if `build` is a directory. If there happens to be a file named `build`, it won't be ignored. This is useful when you want to be specific about directories.

### Purposes

- To ignore entire directories without affecting similarly named files.
- To ensure that only directory structures (not files) are excluded.
- To target build output directories, cache directories, and dependency folders.
- To distinguish between a file and a directory with the same name.
- To prevent accidental exclusion of files that happen to share a directory name.

### Syntax Rules and Structure

**Complete General Syntax**

```
dirname/
/path/to/dirname/
```

**Syntax Rules**

- A trailing slash restricts the match to directories only.
- Without a leading slash, the pattern matches at any depth.
- With a leading slash, the pattern is anchored to the `.gitignore` directory.
- The trailing slash is removed for the purpose of matching the directory name.
- A directory pattern does not match files or symbolic links with the same name.

**Constraints and Limitations**

- **Directory-only**: Cannot be used to ignore files with the same name.
- **Recursive by default**: Ignoring a directory also ignores all its contents.
- **Parent exclusion**: Once a directory is excluded, its contents cannot be re-included.

### Annotated Code Examples

#### Example 1: Directory vs. File with Same Name

```bash
# .gitignore
echo "build/" > .gitignore

# Create a directory named build and a file named build
mkdir build && echo "binary" > build/output.bin
echo "not a directory" > build

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
?? build
```

**Why this output occurs:** The pattern `build/` only matches the directory `build`, ignoring it and its contents. The file `build` does not match the directory pattern, so it remains untracked and appears in the status output.

#### Example 2: Anchored Directory Pattern

```bash
# .gitignore
echo "/logs/" > .gitignore

# Create log directories at different levels
mkdir logs && echo "log" > logs/app.log
mkdir src/logs && echo "log" > src/logs/app.log

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
?? src/logs/app.log
```

**Why this output occurs:** `/logs/` is anchored to the root `.gitignore` directory, so it matches only the root `logs` directory. The `src/logs/` directory is not matched because the pattern is anchored.

### Real-World Cases

- **Ignoring `node_modules/`**: A pattern `node_modules/` ignores all `node_modules` directories at any depth.
- **Ignoring `target/`**: Java Maven projects ignore the `target/` build directory.
- **Ignoring `dist/`**: JavaScript projects ignore the `dist/` distribution directory.
- **Ignoring `__pycache__/`**: Python projects ignore `__pycache__` directories.

### References

- Git Documentation: gitignore (Trailing Slash) — https://git-scm.com/docs/gitignore#_pattern_format
- Stack Overflow: Difference Between /dir and /dir/ — https://stackoverflow.com/questions/17888695

---

## 4. Wildcards (Globbing Patterns, Single `*` vs. Recursive `**` Asterisks)

### Definitions

**Core Definition**
Git ignore patterns support shell-style glob wildcards: `*` matches any sequence of characters except `/`, `?` matches exactly one character except `/`, and `**` matches directories recursively across any depth.

**Technical Definition**
Git treats patterns as shell globs suitable for `fnmatch(3)` with the `FNM_PATHNAME` flag: wildcards in the pattern will not match a `/` in the pathname. Two consecutive asterisks (`**`) in patterns matched against full pathnames have special meaning: a leading `**/` matches in all directories; a trailing `/**` matches everything inside; `/**/` matches zero or more directories.

**Beginner-Friendly Explanation**
Wildcards let you match many files at once without listing each one. `*.log` matches any file ending in `.log`. `file?.txt` matches `file1.txt` and `fileA.txt` but not `file10.txt`. `**/logs/` matches a `logs` directory anywhere in the project. The double asterisk is special: it crosses directory boundaries.

### Purposes

- To match multiple files with a common pattern.
- To avoid listing every file individually.
- To target files by extension (e.g., `*.pyc`, `*.log`).
- To match files with variable characters in their names.
- To recursively match directories at any depth.

### Syntax Rules and Structure

**Wildcard Reference Table**

| Pattern | Meaning | Example |
|---------|---------|---------|
| `*` | Matches zero or more characters within a single directory level | `*.log` matches `app.log`, `debug.log` |
| `?` | Matches exactly one character (not `/`) | `file?.txt` matches `file1.txt`, not `file10.txt` |
| `[abc]` | Matches one character from the set | `temp[0-9].txt` matches `temp0.txt`–`temp9.txt` |
| `[a-z]` | Matches one character from the range | `[a-z].log` matches `a.log`, `b.log` |
| `**/` (leading) | Matches in all directories | `**/logs/` matches `logs/`, `src/logs/` |
| `/**` (trailing) | Matches everything inside | `docs/**` matches all files under `docs/` |
| `/**/` (middle) | Matches zero or more directories | `a/**/b` matches `a/b`, `a/x/b`, `a/x/y/b` |

**Syntax Rules**

- `*` does not match `/`; use `**` to cross directory boundaries.
- `?` matches exactly one character except `/`.
- Square brackets `[ ]` match one character from a set or range.
- Escape special characters with `\` to match them literally.
- A leading `**/` means match in all directories.
- A trailing `/**` matches everything inside a directory.

**Constraints and Limitations**

- **`*` vs. `**`**: A single `*` does not cross directory boundaries; `**` does.
- **`?` vs. `*`**: `?` matches exactly one character; `*` matches zero or more.
- **Case sensitivity**: Matching is case-sensitive on case-sensitive filesystems.
- **No regex**: Git uses glob patterns, not regular expressions.

### Annotated Code Examples

#### Example 1: Single Asterisk vs. Double Asterisk

```bash
# .gitignore
cat > .gitignore << 'EOF'
*.log
docs/**/*.pdf
**/temp/
EOF

# Create files
mkdir -p docs/sub && echo "pdf" > docs/sub/manual.pdf
echo "pdf" > docs/readme.pdf
mkdir temp && echo "temp" > temp/data.txt
mkdir src/temp && echo "temp" > src/temp/data.txt
echo "log" > app.log

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
?? docs/readme.pdf
```

**Why this output occurs:** `*.log` ignores `app.log`. `docs/**/*.pdf` ignores PDFs in `docs/` and any subdirectory (so `docs/sub/manual.pdf` is ignored, but `docs/readme.pdf` is not because the pattern requires a subdirectory). `**/temp/` ignores all `temp` directories at any depth (both root `temp/` and `src/temp/`).

#### Example 2: Question Mark Wildcard

```bash
# .gitignore
echo "file?.txt" > .gitignore

# Create files
echo "one" > file1.txt
echo "two" > file2.txt
echo "ten" > file10.txt

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
?? file10.txt
```

**Why this output occurs:** `file?.txt` matches `file1.txt` and `file2.txt` (single character), but not `file10.txt` (two characters). The `?` wildcard matches exactly one character.

### Real-World Cases

- **Python projects**: `*.py[cod]` ignores `.pyc`, `.pyo`, and `.pyd` files.
- **Node.js projects**: `node_modules/` and `*.log` are common patterns.
- **Documentation**: `docs/**/*.pdf` ignores generated PDFs in all documentation subdirectories.
- **Build systems**: `**/target/` ignores `target` directories in all subprojects of a monorepo.

### References

- Git Documentation: gitignore (Pattern Format) — https://git-scm.com/docs/gitignore#_pattern_format
- LabEx: What Other Wildcard Patterns Exist? — https://labex.io/questions/what-other-wildcard-patterns-exist-268996
- Stack Overflow: Double Asterisk in .gitignore — https://stackoverflow.com/questions/17888695

---

## 5. Negation Patterns (The `!` Prefix and Its Structural Ordering Limitations)

### Definitions

**Core Definition**
A negation pattern begins with `!` and re-includes files that were excluded by a previous pattern, overriding lower-precedence ignore rules.

**Technical Definition**
An optional prefix `!` negates the pattern; any matching file excluded by a previous pattern will become included again. It is not possible to re-include a file if a parent directory of that file is excluded. Git does not list excluded directories for performance reasons, so any patterns on contained files have no effect, no matter where they are defined. To use a literal `!` at the start of a pattern, escape it as `\!`.

**Beginner-Friendly Explanation**
The `!` prefix means "don't ignore this." If you have a rule that ignores all `.log` files but you want to keep `important.log`, you can write `!important.log` to make an exception. However, there's a catch: if the parent directory is ignored, you can't re-include a file inside it—you have to re-include the directory first.

### Purposes

- To create exceptions to broad ignore rules.
- To keep specific files while ignoring similar ones.
- To re-include files in otherwise ignored directories (with caveats).
- To maintain a whitelist of files within ignored folders.
- To override ignore patterns from lower-precedence sources.

### Syntax Rules and Structure

**Complete General Syntax**

```
!pattern
!/anchored-pattern
!dir/file
```

**Syntax Rules**

- The `!` must be the first character of the line.
- Negation overrides patterns **above it** in the same file (last matching pattern wins).
- Negation overrides lower-precedence sources but not higher-precedence ones.
- A negated pattern can re-include a file only if no parent directory is excluded.
- To match a literal `!`, use `\!`.

**Ordering and Structural Limitations**

| Limitation | Description |
|------------|-------------|
| **Parent directory exclusion** | You cannot re-include a file if its parent directory is excluded. |
| **Order matters** | The negation must come **after** the pattern it overrides. |
| **Depth limitation** | Git does not descend into excluded directories, so patterns inside them have no effect. |
| **Precedence ceiling** | Negation cannot override command-line patterns or higher-precedence sources. |

**Workaround for Parent Directory Exclusion**

```
# This does NOT work:
node_modules/
!node_modules/keep/

# This DOES work (re-include parent first):
node_modules/*
!node_modules/keep/
```

**Syntax Rules**

- The second pattern re-includes the `keep` directory before re-including its contents.
- Using `*` instead of the directory name allows re-inclusion of the directory itself.

**Constraints and Limitations**

- **Cannot re-include inside ignored parent**: The most common pitfall.
- **Performance**: Git avoids listing excluded directories, so deep re-inclusion may not work as expected.
- **Multiple levels**: Re-including nested structures requires re-including each ancestor level.

### Annotated Code Examples

#### Example 1: Basic Negation

```bash
# .gitignore
cat > .gitignore << 'EOF'
*.log
!important.log
EOF

# Create log files
echo "debug" > debug.log
echo "important" > important.log

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
?? important.log
```

**Why this output occurs:** `*.log` ignores all log files. `!important.log` re-includes `important.log` because it appears after the ignore pattern. `debug.log` remains ignored.

#### Example 2: Negation with Parent Directory Exclusion (Failure Case)

```bash
# .gitignore
cat > .gitignore << 'EOF'
logs/
!logs/keep.txt
EOF

# Create files
mkdir logs && echo "keep" > logs/keep.txt
echo "debug" > logs/debug.txt

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
```

**Why this output occurs:** The pattern `logs/` excludes the entire `logs` directory. Git does not descend into excluded directories, so `!logs/keep.txt` has no effect. Both `logs/keep.txt` and `logs/debug.txt` remain ignored. This demonstrates the parent directory exclusion limitation.

#### Example 3: Correct Negation with Parent Re-inclusion

```bash
# .gitignore
cat > .gitignore << 'EOF'
logs/*
!logs/keep.txt
EOF

# Create files
mkdir logs && echo "keep" > logs/keep.txt
echo "debug" > logs/debug.txt

# Check status
git status -s
```

**Expected Output:**
```
?? .gitignore
?? logs/keep.txt
```

**Why this output occurs:** `logs/*` ignores all files inside `logs/` but does not exclude the `logs` directory itself. Git descends into the directory, and `!logs/keep.txt` re-includes the specific file. This is the correct way to whitelist files inside a directory.

### Real-World Cases

- **Keeping `.gitkeep`**: A common pattern is to ignore all files in a directory but keep a `.gitkeep` placeholder:
  ```
  uploads/*
  !uploads/.gitkeep
  ```
- **Environment files**: Ignore all `.env` files but keep `.env.example`:
  ```
  .env*
  !.env.example
  ```
- **Build outputs**: Ignore `dist/` but keep a specific static file:
  ```
  dist/*
  !dist/favicon.ico
  ```

### References

- Git Documentation: gitignore (Negation) — https://git-scm.com/docs/gitignore#_pattern_format
- Stack Overflow: Gitignore Negation Patterns — https://stackoverflow.com/questions/17888695
- Raw GitHub: Consolidated .gitignore Syntax Summary — https://raw.githubusercontent.com/posit-dev/btw/refs/tags/v1.2.1/R/utils-gitignore.R

---

## 6. Global Ignore Rules (`core.excludesfile`)

### Definitions

**Core Definition**
The global ignore file is a personal exclude file, configured via `core.excludesFile`, that applies to all Git repositories on a user's machine, regardless of the project.

**Technical Definition**
Patterns which a user wants Git to ignore in all situations (e.g., backup or temporary files generated by the user's editor of choice) generally go into a file specified by `core.excludesFile` in the user's `~/.gitconfig`. Its default value is `$XDG_CONFIG_HOME/git/ignore`. If `$XDG_CONFIG_HOME` is either not set or empty, `$HOME/.config/git/ignore` is used instead. This file is not committed to any repository and does not affect other users.

**Beginner-Friendly Explanation**
Your global ignore file is your personal "do not disturb" list that applies to every project on your computer. It's perfect for ignoring files your editor creates, your operating system generates, or your personal tools produce—things that other people on your team probably don't want to ignore. You set it up once, and it works everywhere.

### Purposes

- To ignore files generated by the user's editor or IDE (e.g., `.vscode/`, `.idea/`).
- To ignore operating system artefacts (e.g., `.DS_Store`, `Thumbs.db`).
- To ignore personal tool configuration files.
- To avoid cluttering project `.gitignore` files with user-specific entries.
- To apply consistent ignore rules across all repositories.

### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Set the global ignore file location
git config --global core.excludesFile ~/.gitignore_global

# Or use the default location
# ~/.config/git/ignore
```

**Configuration Options**

| Setting | Default | Description |
|---------|---------|-------------|
| `core.excludesFile` | `$XDG_CONFIG_HOME/git/ignore` | Path to the global ignore file. |
| `~/.gitconfig` | `[core] excludesFile = ...` | Alternative configuration location. |

**Syntax Rules**

- The file uses the same pattern syntax as `.gitignore`.
- Patterns apply to **all** Git repositories on the machine.
- The file is not version-controlled and is not shared with others.
- The default location is `~/.config/git/ignore` if `$XDG_CONFIG_HOME` is set; otherwise `$HOME/.config/git/ignore`.
- Can be set per-machine or per-user.

**Constraints and Limitations**

- **Personal only**: Not shared with collaborators; each user must set their own.
- **Overridden by local patterns**: Project `.gitignore` and `.git/info/exclude` take precedence.
- **No version control**: Changes are not tracked; backup is the user's responsibility.
- **Platform-specific paths**: The default path differs across operating systems.

### Annotated Code Examples

#### Example 1: Setting Up a Global Ignore File

```bash
# Create the global ignore file
cat > ~/.gitignore_global << 'EOF'
# Editor directories
.vscode/
.idea/
*.swp
*~

# OS artefacts
.DS_Store
Thumbs.db

# Personal tool files
.cspell.json
project-words.txt
EOF

# Configure Git to use it
git config --global core.excludesFile ~/.gitignore_global

# Verify
git config --global core.excludesFile
```

**Expected Output:**
```
/home/user/.gitignore_global
```

**Why this output occurs:** `core.excludesFile` is set to the path of the global ignore file. Git reads this file for every repository, ignoring `.vscode/`, `.DS_Store`, and other personal files everywhere.

#### Example 2: Using the Default Location

```bash
# Create the default global ignore file
mkdir -p ~/.config/git
cat > ~/.config/git/ignore << 'EOF'
.DS_Store
Thumbs.db
EOF

# No configuration needed; Git uses this by default
```

**Why this works:** Git automatically checks `~/.config/git/ignore` as the default global exclude file when `core.excludesFile` is not set.

### Real-World Cases

- **Cross-platform development**: A developer works on macOS and Windows; their global ignore includes `.DS_Store` and `Thumbs.db`.
- **Multiple editors**: A developer uses VS Code and Vim; their global ignore includes `.vscode/`, `*.swp`, and `*~`.
- **Personal linting tools**: A developer uses `cspell` and ignores `project-words.txt` globally so it doesn't clutter each repository's `.gitignore`.

### References

- Git Documentation: gitignore (Global Excludes) — https://git-scm.com/docs/gitignore
- GitHub Docs: Configuring Ignored Files for All Repositories — https://docs.github.com/en/get-started/git-basics/ignoring-files
- LabEx: How to Use Global Gitignore — https://labex.io

---

## 7. Local Project-Only Ignores Without Committing Code (The `.git/info/exclude` File)

### Definitions

**Core Definition**
The `.git/info/exclude` file is a repository-local exclude file that applies only to the current clone, is not committed, and is not shared with other developers.

**Technical Definition**
Patterns which are specific to a particular repository but which do not need to be shared with other related repositories (e.g., auxiliary files that live inside the repository but are specific to one user's workflow) should go into the `$GIT_DIR/info/exclude` file. The file uses the same pattern syntax as `.gitignore`. It is located at `.git/info/exclude` within the repository and is never committed or pushed.

**Beginner-Friendly Explanation**
Sometimes you have files that are specific to your local setup—maybe a personal configuration file or a tool that only you use—and you don't want to add them to the project's shared `.gitignore` (because other people don't need to ignore them), but you also don't want them showing up in `git status`. The `.git/info/exclude` file is the perfect place for these: it ignores files only for you, in this repository, without affecting anyone else.

### Purposes

- To ignore repository-specific files without modifying the shared `.gitignore`.
- To handle personal workflow files that shouldn't be committed.
- To avoid polluting the project's ignore rules with user-specific entries.
- To provide a local-only ignore mechanism that is not shared.
- To test ignore patterns without committing them.

### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Edit the file directly
vim .git/info/exclude

# Or append patterns
echo ".claude/" >> .git/info/exclude
echo "local-config.yaml" >> .git/info/exclude
```

**File Location**

```
.git/
├── info/
│   └── exclude    ← Local ignore file
├── HEAD
├── config
└── ...
```

**Syntax Rules**

- Uses the same pattern syntax as `.gitignore`.
- Located at `.git/info/exclude` within the repository.
- Not committed to the repository; not shared with collaborators.
- Patterns apply only to the current clone of the repository.
- Overridden only by command-line patterns and higher-precedence `.gitignore` files.

**Constraints and Limitations**

- **Local only**: Other clones of the repository do not inherit these rules.
- **Not version-controlled**: Changes are not tracked and not backed up with the repository.
- **No sharing**: Cannot be used to enforce team-wide ignore rules.
- **Precedence**: Lower than `.gitignore` patterns in the same directory.

### Annotated Code Examples

#### Example 1: Adding a Local-Only Ignore

```bash
# Navigate to the repository
cd my-project

# Add a personal configuration file to the local exclude
echo "my-local-config.yaml" >> .git/info/exclude
echo ".claude/" >> .git/info/exclude

# Create the ignored file
echo "local settings" > my-local-config.yaml

# Check status
git status -s
```

**Expected Output:**
```
(no output for my-local-config.yaml)
```

**Why this output occurs:** `my-local-config.yaml` is listed in `.git/info/exclude`, so Git ignores it in this repository only. It does not appear in `git status`, and the exclusion is not committed or shared with other developers.

#### Example 2: Comparing `.gitignore` vs. `.git/info/exclude`

```bash
# .gitignore (shared, committed)
echo "*.log" > .gitignore
git add .gitignore && git commit -m "Add shared gitignore"

# .git/info/exclude (local, not committed)
echo "personal-notes.txt" >> .git/info/exclude

# Create files
echo "log" > app.log
echo "notes" > personal-notes.txt

# Check status
git status -s
```

**Expected Output:**
```
(clean)
```

**Why this output occurs:** `app.log` is ignored by the shared `.gitignore`. `personal-notes.txt` is ignored by the local `.git/info/exclude`. Neither appears in `git status`. Other developers cloning the repository will see `app.log` ignored (from `.gitignore`) but will not have `personal-notes.txt` ignored (since it's in `.git/info/exclude`).

### Real-World Cases

- **Personal editor configs**: A developer uses a specific `.vscode/settings.json` that they don't want to commit; they add `.vscode/` to `.git/info/exclude`.
- **Local tool configurations**: A developer uses `Volta` for Node version management; they add `.volta/` to `.git/info/exclude`.
- **Temporary files**: A developer creates a `scratch/` directory for temporary work; they add `scratch/` to `.git/info/exclude` so it doesn't appear in `git status`.
- **Experimental linting**: A developer runs ESLint locally before the team adopts it; they ignore `.eslintcache` in `.git/info/exclude`.

### References

- Git Documentation: gitignore ($GIT_DIR/info/exclude) — https://git-scm.com/docs/gitignore
- GitHub Docs: Excluding Local Files — https://docs.github.com/en/get-started/git-basics/ignoring-files
- Stack Overflow: Difference Between .git/info/exclude and .gitignore — https://stackoverflow.com/questions/17888695
- DevelopersIO: Using .git/info/exclude — https://dev.classmethod.jp

---

## References

- Git Documentation: gitignore Manual Page — https://git-scm.com/docs/gitignore
- Git Documentation: git-check-ignore Manual Page — https://git-scm.com/docs/git-check-ignore
- GitHub Docs: Ignoring Files — https://docs.github.com/en/get-started/git-basics/ignoring-files
- Git Documentation: Git Basics – Recording Changes (Ignoring Files) — https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository
- NYCU CSIT Mirror: gitignore(5) Manual Page — http://kernel.cs.nycu.edu.tw/software/scm/git/docs/gitignore.html
- Raw GitHub: Consolidated .gitignore Syntax Summary — https://raw.githubusercontent.com/posit-dev/btw/refs/tags/v1.2.1/R/utils-gitignore.R
- Stack Overflow: Difference Between /dir and /dir/ — https://stackoverflow.com/questions/17888695
- LabEx: What Other Wildcard Patterns Exist? — https://labex.io/questions/what-other-wildcard-patterns-exist-268996
- DevelopersIO: Using .git/info/exclude — https://dev.classmethod.jp
- Brandon Pugh: Personal Git Ignore per Repository — https://raw.githubusercontent.com/bpugh/brandonpugh.com/refs/heads/main/content/til/git/personal-git-ignore.md