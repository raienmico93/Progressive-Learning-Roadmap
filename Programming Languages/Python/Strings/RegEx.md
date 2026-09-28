# Python Regular Expressions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Regular expressions (regex or regexes) are a tiny, highly specialized programming language embedded inside Python and made available through the `re` module. They specify a set of strings that match a given pattern, enabling powerful text searching, extraction, and transformation.

### Technical Definition

The `re` module provides regular expression matching operations similar to those found in Perl. Both patterns and strings to be searched can be Unicode strings (`str`) as well as 8-bit strings (`bytes`). Regular expressions are compiled into a series of bytecodes, which are then executed by a matching engine written in C. The module supports pattern compilation, matching, searching, substitution, and splitting, along with a rich syntax of metacharacters, character classes, quantifiers, groups, and assertions.

### Beginner-Friendly Explanation

Regular expressions are like a super-powered "find and replace" for text. Instead of searching for exact words, you describe a pattern: "find all phone numbers," "find words that start with 'cat'," or "replace every date with its ISO format." Python's `re` module gives you the tools to write these patterns and apply them to text.

### Key Characteristics

- **Pattern-based matching**: Uses a specialized syntax to describe text patterns.
- **Compiled execution**: Patterns are compiled to bytecode for efficient execution.
- **Raw string convention**: Patterns are typically written as raw strings (`r"..."`) to avoid backslash collisions.
- **Rich syntax**: Supports character classes, quantifiers, groups, anchors, lookarounds, and conditional patterns.
- **Performance considerations**: Backtracking engines can suffer from catastrophic backtracking; precompilation and careful pattern design mitigate this.

### Prerequisites

- Python 3.x installed.
- Basic understanding of strings and string methods.
- Familiarity with the `re` module import (`import re`).
- A text editor or IDE for writing and testing patterns.

### Related Programming Areas

- **Text processing**: Parsing, cleaning, and transforming text data.
- **Data validation**: Email addresses, phone numbers, URLs, dates.
- **Web scraping**: Extracting data from HTML/XML.
- **Log analysis**: Searching and filtering log entries.
- **Lexical analysis**: Tokenizing programming languages.

### Core Concepts / Features

The following sections cover each core concept using a uniform structure.

---

## 1. `re` Module Functions

### Definitions

**Core Definition**: The `re` module provides a set of functions for compiling patterns and performing match, search, findall, finditer, and other operations on strings.

**Technical Definition**: The `re` module offers both module-level functions (which compile patterns on the fly) and methods on compiled pattern objects (created via `re.compile()`). The principal functions are `re.search()`, `re.match()`, `re.fullmatch()`, `re.findall()`, `re.finditer()`, `re.sub()`, `re.subn()`, and `re.split()`.

**Beginner-Friendly Explanation**: These are the tools you use to apply a regex pattern to a string. `re.search()` finds the first match anywhere; `re.match()` only checks the beginning; `re.findall()` returns all matches as a list; `re.finditer()` returns them one at a time. `re.compile()` lets you build a reusable pattern object for efficiency.

### Purposes

- To compile a regex pattern for reuse (avoiding repeated compilation).
- To search for the first occurrence of a pattern anywhere in a string.
- To check if a string starts with a pattern.
- To verify that an entire string matches a pattern.
- To find all non-overlapping matches in a string.
- To iterate over matches lazily for memory efficiency.
- To replace matches with a replacement string or function.
- To split a string at each match of a pattern.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import re

re.search(pattern, string, flags=0)
re.match(pattern, string, flags=0)
re.fullmatch(pattern, string, flags=0)
re.findall(pattern, string, flags=0)
re.finditer(pattern, string, flags=0)
re.sub(pattern, repl, string, count=0, flags=0)
re.subn(pattern, repl, string, count=0, flags=0)
re.split(pattern, string, maxsplit=0, flags=0)
re.compile(pattern, flags=0)
```

**Component Breakdown**:
- `pattern` — the regular expression pattern (string or compiled pattern).
- `string` — the target string to search.
- `flags` — optional flags (e.g., `re.IGNORECASE`).
- `repl` — replacement string or callable.
- `count` — maximum number of replacements; `0` means all.
- `maxsplit` — maximum number of splits; `0` means all.

#### Function Comparison Table

| Function | Match Location | Return Value |
|----------|---------------|--------------|
| `re.search()` | Anywhere in string | Match object or `None` |
| `re.match()` | Start of string only | Match object or `None` |
| `re.fullmatch()` | Entire string | Match object or `None` |
| `re.findall()` | All non-overlapping | List of strings or tuples |
| `re.finditer()` | All non-overlapping | Iterator of Match objects |
| `re.sub()` | All matches | New string |
| `re.subn()` | All matches | Tuple (new string, count) |
| `re.split()` | All matches | List of strings |

#### Syntax Rules

1. **Raw string recommended**: Use `r"..."` for patterns to avoid backslash escaping issues.
2. **Flags are optional**: Combine flags with `|` (bitwise OR).
3. **Compiled patterns**: `re.compile()` returns a `Pattern` object with the same methods.
4. **Module-level caching**: Python caches recently compiled patterns (up to ~512), so one-shot calls are cheap.
5. **`re.match()` vs `re.search()`**: `match` only checks the beginning; `search` scans the entire string.
6. **`re.fullmatch()`**: Requires the entire string to match; equivalent to anchoring with `^...$`.

#### Constraints and Limitations

- Unicode and 8-bit strings cannot be mixed: a bytes pattern must match a bytes string.
- Patterns may not contain null bytes; use `\x00` instead.
- Invalid escape sequences in string literals now generate `SyntaxWarning` (and will become `SyntaxError` in the future).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Core Functions

```python
import re

text = "The price is $49.99 and the discount is $10.00"

# step1: search() — find first match anywhere
match = re.search(r'\$\d+\.\d{2}', text)
print(match.group())               # $49.99

# step2: match() — only at start (fails here)
print(re.match(r'\$\d+\.\d{2}', text))  # None

# step3: findall() — all matches
print(re.findall(r'\$\d+\.\d{2}', text))  # ['$49.99', '$10.00']

# step4: finditer() — iterator of match objects
for m in re.finditer(r'\$\d+\.\d{2}', text):
    print(m.group(), m.span())
# $49.99 (12, 18)
# $10.00 (39, 45)

# step5: fullmatch() — entire string must match
print(re.fullmatch(r'\d+', '12345'))  # <re.Match object; span=(0, 5), match='12345'>
print(re.fullmatch(r'\d+', '123abc')) # None
```

**Expected Output**:
```
$49.99
None
['$49.99', '$10.00']
$49.99 (12, 18)
$10.00 (39, 45)
<re.Match object; span=(0, 5), match='12345'>
None
```

**Why**: Each function has a different matching scope and return type. `search` scans anywhere, `match` only at the start, `findall` returns all matches as a list, `finditer` yields match objects lazily, and `fullmatch` requires the whole string.

#### Example 2: Compiled Patterns

```python
import re

# step1: Compile a pattern for reuse
EMAIL_RE = re.compile(r'[\w.+-]+@[\w-]+\.[\w.-]+')

# step2: Use the compiled pattern
text = "Contact alice@example.com or bob@test.org"
print(EMAIL_RE.findall(text))
# ['alice@example.com', 'bob@test.org']

# step3: Method on compiled pattern
match = EMAIL_RE.search(text)
print(match.group())  # alice@example.com
print(match.span())   # (8, 25)
```

**Expected Output**:
```
['alice@example.com', 'bob@test.org']
alice@example.com
(8, 25)
```

**Why**: Compiling once and reusing the pattern object avoids repeated compilation overhead and improves readability through descriptive naming.

#### Example 3: Substitution

```python
import re

# step1: Basic substitution
text = "blue socks and red shoes"
result = re.sub(r'(blue|white|red)', 'colour', text)
print(result)  # colour socks and colour shoes

# step2: subn() returns count
result, count = re.subn(r'(blue|white|red)', 'colour', text)
print(f"Result: {result}, Replacements: {count}")
# Result: colour socks and colour shoes, Replacements: 2

# step3: Using a function as replacement
def hex_repl(match):
    return hex(int(match.group()))

print(re.sub(r'\d+', hex_repl, 'Call 65490 for printing'))
# Call 0xffd2 for printing
```

**Expected Output**:
```
colour socks and colour shoes
Result: colour socks and colour shoes, Replacements: 2
Call 0xffd2 for printing
```

**Why**: `sub()` replaces all matches; `subn()` also returns the count; a function replacement allows dynamic replacement values.

### Real-World Cases

- **Log parsing**: Extracting IP addresses, timestamps, and error codes.
- **Data validation**: Checking email, phone, and URL formats.
- **Web scraping**: Extracting data from HTML.
- **Configuration files**: Parsing key-value pairs.

### References

- `re` — Regular expression operations - https://docs.python.org/3/library/re.html
- Regular expression HOWTO - https://docs.python.org/3/howto/regex.html
- `re.compile()` - https://docs.python.org/3/library/re.html#re.compile

---

## 2. Patterns

### Definitions

**Core Definition**: A pattern is a string that describes a set of strings to be matched, using a combination of literal characters, metacharacters, anchors, and flags.

**Technical Definition**: Regular expression patterns are composed of ordinary characters (which match themselves) and metacharacters (which have special meanings). The metacharacters are `. ^ $ * + ? { } [ ] \ | ( )`. Patterns can be concatenated to form new patterns. Anchors (`^`, `$`, `\b`, `\B`, `\A`, `\Z`) assert positions in the string without consuming characters. Flags modify the behaviour of the matching engine.

**Beginner-Friendly Explanation**: A pattern is what you're searching for. Literal characters like `abc` match themselves. Special characters like `.` (any character), `^` (start of string), and `\d` (digit) let you describe patterns flexibly. Flags like `re.IGNORECASE` make matching case-insensitive.

### Purposes

- To describe a set of strings using a concise syntax.
- To assert positions (start, end, word boundaries) in the string.
- To modify matching behaviour with flags.
- To combine literal and special characters for flexible matching.
- To construct complex patterns from simpler primitives.

### Syntax Rules and Structure

#### Complete General Syntax

```python
r"pattern"                     # Raw string pattern
r"^pattern$"                   # Anchored pattern
r"\bword\b"                    # Word boundary
r"pattern" with flags          # Flags modify behaviour
```

**Component Breakdown**:
- `^` — start of string (or line with `re.MULTILINE`).
- `$` — end of string (or line with `re.MULTILINE`).
- `\b` — word boundary.
- `\B` — non-word boundary.
- `\A` — absolute start of string.
- `\Z` — absolute end of string.

#### Common Flags

| Flag | Short Form | Effect |
|------|-----------|--------|
| `re.IGNORECASE` | `re.I` | Case-insensitive matching |
| `re.MULTILINE` | `re.M` | `^` and `$` match at line boundaries |
| `re.DOTALL` | `re.S` | `.` matches newline characters |
| `re.VERBOSE` | `re.X` | Allow whitespace and comments in patterns |
| `re.ASCII` | `re.A` | `\w`, `\d`, `\s` match ASCII only |

#### Syntax Rules

1. **Raw strings**: Use `r"..."` to avoid Python's escape sequence interpretation.
2. **Anchors**: `^` and `$` match positions, not characters.
3. **Word boundaries**: `\b` matches between a word character and a non-word character.
4. **Flags combine with `|`**: `re.I | re.M`.
5. **Inline flags**: `(?i)` for case-insensitive, `(?m)` for multiline, `(?s)` for dotall.
6. **Absolute anchors**: `\A` and `\Z` match only at the very start/end, regardless of multiline mode.

#### Constraints and Limitations

- Patterns may not contain null bytes.
- Invalid escape sequences generate warnings.
- Flags are not available in all functions; use compiled patterns or inline flags.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Anchors and Boundaries

```python
import re

# step1: Start anchor
print(re.findall(r'^\w+', 'hello world\npython rocks', re.MULTILINE))
# ['hello', 'python']

# step2: End anchor
print(re.findall(r'\w+$', 'hello world\npython rocks', re.MULTILINE))
# ['world', 'rocks']

# step3: Word boundary
print(re.findall(r'\bcat\b', 'cat concatenate cat.'))
# ['cat', 'cat']

# step4: Non-word boundary
print(re.findall(r'\Bcat\B', 'concatenate'))
# ['cat']
```

**Expected Output**:
```
['hello', 'python']
['world', 'rocks']
['cat', 'cat']
['cat']
```

**Why**: `^` and `$` anchor to start/end; with `re.MULTILINE` they match at line boundaries. `\b` matches word boundaries; `\B` matches non-boundaries.

#### Example 2: Flags

```python
import re

text = "Hello World\nPython Rocks"

# step1: Case-insensitive
print(re.findall(r'hello', text, re.IGNORECASE))
# ['Hello']

# step2: Multiline
print(re.findall(r'^\w+', text, re.MULTILINE))
# ['Hello', 'Python']

# step3: Dotall
print(re.findall(r'Hello.World', text, re.DOTALL))
# ['Hello World']

# step4: Combined flags
print(re.findall(r'^hello', text, re.IGNORECASE | re.MULTILINE))
# ['Hello']
```

**Expected Output**:
```
['Hello']
['Hello', 'Python']
['Hello World']
['Hello']
```

**Why**: Flags modify matching behaviour: `IGNORECASE` ignores case, `MULTILINE` makes anchors line-aware, `DOTALL` makes `.` match newlines.

#### Example 3: Verbose Mode

```python
import re

# step1: Verbose pattern with comments
pattern = re.compile(r"""
    \b          # word boundary
    \d{3}       # area code
    -           # separator
    \d{4}       # local number
    \b          # word boundary
""", re.VERBOSE)

print(pattern.findall("Call 555-1234 or 555-5678"))
# ['555-1234', '555-5678']
```

**Expected Output**:
```
['555-1234', '555-5678']
```

**Why**: `re.VERBOSE` allows whitespace and comments in patterns, making complex patterns more readable.

### Real-World Cases

- **Line-based parsing**: Using `re.MULTILINE` for log files.
- **Case-insensitive search**: User input matching.
- **Multi-line patterns**: Extracting blocks of text.
- **Readable patterns**: Using `re.VERBOSE` for complex regexes.

### References

- Regular Expression Syntax - https://docs.python.org/3/library/re.html#regular-expression-syntax
- Compilation Flags - https://docs.python.org/3/library/re.html#flags

---

## 3. Character Classes

### Definitions

**Core Definition**: A character class specifies a set of characters, any one of which can match at a given position. Character classes are defined with square brackets `[...]` or with built-in shorthand types like `\d`, `\w`, `\s`.

**Technical Definition**: Character classes are sets of characters that a pattern can match. They can be defined by listing characters individually (`[abc]`), by ranges (`[a-z]`), or by negation (`[^abc]`). Built-in shorthand classes include `\d` (digit), `\D` (non-digit), `\w` (word character), `\W` (non-word), `\s` (whitespace), and `\S` (non-whitespace).

**Beginner-Friendly Explanation**: Character classes let you say "match any one of these characters." `[abc]` matches a, b, or c. `[^abc]` matches anything except a, b, or c. `\d` is shorthand for any digit, `\w` for any letter, digit, or underscore, and `\s` for any whitespace.

### Purposes

- To match any one of a set of characters.
- To match characters in a range (e.g., all lowercase letters).
- To exclude specific characters from matching.
- To use shorthand classes for common character categories.
- To combine multiple character classes in a single pattern.

### Syntax Rules and Structure

#### Complete General Syntax

```python
[abc]          # Any of a, b, c
[a-z]          # Range: lowercase letters
[^abc]         # Any character except a, b, c
[a-zA-Z0-9]    # Alphanumeric
\d             # Digit [0-9]
\D             # Non-digit [^0-9]
\w             # Word character [a-zA-Z0-9_]
\W             # Non-word character
\s             # Whitespace [ \t\n\r\f\v]
\S             # Non-whitespace
```

**Component Breakdown**:
- `[...]` — character class delimiters.
- `^` at start — negation.
- `a-z` — range.
- `\d`, `\w`, `\s` — built-in shorthand classes.

#### Syntax Rules

1. **Metacharacters inside classes**: Most metacharacters lose their special meaning inside `[...]` and match themselves (except `\`, `]`, `^` at start, and `-` in ranges).
2. **Ranges**: `[a-z]` matches any lowercase letter; `[0-9]` matches any digit.
3. **Negation**: `[^...]` matches any character not in the set.
4. **Shorthand classes**: `\d`, `\w`, `\s` are Unicode-aware by default.
5. **ASCII-only**: Use `re.ASCII` to restrict `\w`, `\d`, `\s` to ASCII.
6. **Matching a literal `]`**: Place it first in the class: `[]]`.
7. **Matching a literal `-`**: Place it first or last: `[-a]` or `[a-]`.

#### Constraints and Limitations

- Character classes match exactly one character (unless combined with a quantifier).
- Negated classes match any character not in the set, including newlines (unless `re.DOTALL` is involved).
- Unicode-aware by default; use `re.ASCII` for ASCII-only.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Character Classes

```python
import re

# step1: List of characters
print(re.findall(r'[abc]', 'abcdef'))  # ['a', 'b', 'c']

# step2: Range
print(re.findall(r'[a-z]', 'Hello World'))  # ['e', 'l', 'l', 'o', 'o', 'r', 'l', 'd']

# step3: Negation
print(re.findall(r'[^abc]', 'abcdef'))  # ['d', 'e', 'f']

# step4: Alphanumeric
print(re.findall(r'[a-zA-Z0-9]', 'Hi 123!'))  # ['H', 'i', '1', '2', '3']
```

**Expected Output**:
```
['a', 'b', 'c']
['e', 'l', 'l', 'o', 'o', 'r', 'l', 'd']
['d', 'e', 'f']
['H', 'i', '1', '2', '3']
```

**Why**: Character classes match any one character from the set; ranges and negation extend the flexibility.

#### Example 2: Shorthand Classes

```python
import re

text = "abc 123 !@#"

# step1: \d — digits
print(re.findall(r'\d', text))    # ['1', '2', '3']

# step2: \D — non-digits
print(re.findall(r'\D', text))    # ['a', 'b', 'c', ' ', ' ', '!', '@', '#']

# step3: \w — word characters
print(re.findall(r'\w', text))    # ['a', 'b', 'c', '1', '2', '3']

# step4: \W — non-word characters
print(re.findall(r'\W', text))    # [' ', ' ', '!', '@', '#']

# step5: \s — whitespace
print(re.findall(r'\s', text))    # [' ', ' ']

# step6: \S — non-whitespace
print(re.findall(r'\S', text))    # ['a', 'b', 'c', '1', '2', '3', '!', '@', '#']
```

**Expected Output**:
```
['1', '2', '3']
['a', 'b', 'c', ' ', ' ', '!', '@', '#']
['a', 'b', 'c', '1', '2', '3']
[' ', ' ', '!', '@', '#']
[' ', ' ']
['a', 'b', 'c', '1', '2', '3', '!', '@', '#']
```

**Why**: Shorthand classes provide concise matching for common character categories.

#### Example 3: Combining Classes

```python
import re

# step1: Multiple ranges
print(re.findall(r'[a-zA-Z]', 'Hi 123!'))  # ['H', 'i']

# step2: Escaping inside classes
print(re.findall(r'[\d+]', '1 + 2 = 3'))   # ['1', '+', '2', '3']

# step3: Literal hyphen
print(re.findall(r'[a-]', 'a-b'))          # ['a', '-']
```

**Expected Output**:
```
['H', 'i']
['1', '+', '2', '3']
['a', '-']
```

**Why**: Multiple ranges, escaped metacharacters, and literal hyphens are supported inside character classes.

### Real-World Cases

- **Password validation**: Checking for uppercase, lowercase, digits, and special characters.
- **Data cleaning**: Removing non-alphanumeric characters.
- **Tokenization**: Splitting text at non-word characters.
- **Search**: Finding words, numbers, or punctuation.

### References

- Character Classes - https://docs.python.org/3/library/re.html#character-classes
- Special Sequences - https://docs.python.org/3/library/re.html#special-sequences

---

## 4. Quantifiers

### Definitions

**Core Definition**: Quantifiers specify how many times the preceding element (character, class, or group) must match. Quantifiers include `*`, `+`, `?`, and `{m,n}`.

**Technical Definition**: Quantifiers control repetition of the preceding regular expression element. By default, quantifiers are greedy — they match as many repetitions as possible. Lazy (non-greedy) quantifiers match as few as possible and are specified by appending `?` to the quantifier.

**Beginner-Friendly Explanation**: Quantifiers say "match this zero or more times" (`*`), "one or more times" (`+`), "zero or one time" (`?`), or "between m and n times" (`{m,n}`). By default, they grab as much as they can; adding `?` makes them grab as little as possible.

### Purposes

- To match repeated occurrences of a pattern.
- To make an element optional (`?`).
- To specify exact or ranged repetition counts (`{m,n}`).
- To control greedy versus lazy matching behaviour.
- To implement flexible pattern matching for variable-length text.

### Syntax Rules and Structure

#### Complete General Syntax

```python
a*        # Zero or more (greedy)
a+        # One or more (greedy)
a?        # Zero or one (greedy)
a{m}      # Exactly m
a{m,n}    # Between m and n
a{m,}     # At least m
a*?       # Zero or more (lazy)
a+?       # One or more (lazy)
a??       # Zero or one (lazy)
a{m,n}?   # Between m and n (lazy)
```

**Component Breakdown**:
- `*` — zero or more.
- `+` — one or more.
- `?` — zero or one.
- `{m,n}` — range.
- `?` appended — lazy variant.

#### Greedy vs. Lazy Comparison

| Pattern | Input | Greedy Match | Lazy Match |
|---------|-------|-------------|------------|
| `a+` | `"aaa"` | `"aaa"` | `"a"` (with `+?`) |
| `\d+` | `"123"` | `"123"` | `"1"` (with `+?`) |
| `<.*>` | `"<a><b>"` | `"<a><b>"` | `"<a>"` (with `*?`) |

#### Syntax Rules

1. **Greedy by default**: `*`, `+`, `?`, `{m,n}` match as many repetitions as possible.
2. **Lazy variants**: Append `?` to make the quantifier lazy: `*?`, `+?`, `??`, `{m,n}?`.
3. **Quantifier scope**: Applies to the immediately preceding character, class, or group.
4. **Group quantifiers**: `(ab)+` repeats the group `ab`.
5. **Zero-width assertions cannot be quantified**: You cannot apply a quantifier to `^`, `$`, `\b`, or lookarounds.

#### Constraints and Limitations

- Greedy quantifiers can cause catastrophic backtracking if not carefully designed.
- Lazy quantifiers still require backtracking but typically less.
- Quantifiers cannot be applied to anchors or assertions.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Greedy vs. Lazy

```python
import re

html = "<a><b>"

# step1: Greedy — matches as much as possible
print(re.findall(r'<.*>', html))     # ['<a><b>']

# step2: Lazy — matches as little as possible
print(re.findall(r'<.*?>', html))    # ['<a>', '<b>']

# step3: Greedy with digits
print(re.findall(r'\d+', '123 456'))  # ['123', '456']

# step4: Lazy with digits
print(re.findall(r'\d+?', '123 456')) # ['1', '2', '3', '4', '5', '6']
```

**Expected Output**:
```
['<a><b>']
['<a>', '<b>']
['123', '456']
['1', '2', '3', '4', '5', '6']
```

**Why**: Greedy quantifiers match the longest possible sequence; lazy quantifiers match the shortest.

#### Example 2: Exact and Ranged Repetition

```python
import re

# step1: Exactly 3 digits
print(re.findall(r'\d{3}', '12 345 6789'))  # ['345', '678']

# step2: Between 2 and 4 digits
print(re.findall(r'\d{2,4}', '1 12 123 1234 12345'))
# ['12', '123', '1234', '1234']

# step3: At least 2 digits
print(re.findall(r'\d{2,}', '1 12 123 1234'))
# ['12', '123', '1234']
```

**Expected Output**:
```
['345', '678']
['12', '123', '1234', '1234']
['12', '123', '1234']
```

**Why**: `{m}`, `{m,n}`, and `{m,}` provide precise control over repetition counts.

#### Example 3: Quantifying Groups

```python
import re

# step1: Repeat a group
print(re.findall(r'(ab)+', 'ababab ab'))  # ['ab', 'ab']

# step2: Optional group
print(re.findall(r'colou?r', 'color colour'))  # ['color', 'colour']

# step3: Nested quantifiers
print(re.findall(r'(a+)b', 'aab aaab'))  # ['aa', 'aaa']
```

**Expected Output**:
```
['ab', 'ab']
['color', 'colour']
['aa', 'aaa']
```

**Why**: Quantifiers can be applied to groups, enabling repetition of multi-character sequences.

### Real-World Cases

- **HTML parsing**: Lazy quantifiers for tags.
- **Number validation**: Exact digit counts for phone numbers.
- **Optional text**: `?` for optional prefixes/suffixes.
- **Password rules**: `{8,}` for minimum length.

### References

- Quantifiers - https://docs.python.org/3/library/re.html#quantifiers
- Greedy vs. Lazy - https://docs.python.org/3/howto/regex.html#greedy-versus-non-greedy

---

## 5. Groups

### Definitions

**Core Definition**: Groups are subpatterns enclosed in parentheses `(...)` that capture the matched text for later retrieval or backreferencing.

**Technical Definition**: Groups serve two purposes: they group subpatterns together so that quantifiers can apply to the entire group, and they capture the matched substring, which can be retrieved via `match.group(n)` or referenced later in the pattern with backreferences like `\1`. Named groups use the syntax `(?P<name>...)`.

**Beginner-Friendly Explanation**: Groups are like parentheses in math — they let you treat part of a pattern as a unit. They also save whatever they match so you can use it later. `(\d+)-(\d+)` captures two numbers separately.

### Purposes

- To capture substrings for extraction.
- To apply quantifiers to multiple characters.
- To create backreferences within the pattern.
- To name groups for readability.
- To structure complex patterns hierarchically.

### Syntax Rules and Structure

#### Complete General Syntax

```python
(...)             # Capturing group
(?P<name>...)     # Named capturing group
\1, \2, ...       # Backreference to group N
(?P=name)         # Backreference to named group
```

**Component Breakdown**:
- `(...)` — capturing group.
- `(?P<name>...)` — named capturing group.
- `\N` — backreference to group number N.
- `(?P=name)` — backreference to named group.

#### Syntax Rules

1. **Group numbering**: Groups are numbered from 1 in the order their opening parentheses appear.
2. **Capturing**: Groups capture the text they match for later retrieval.
3. **Backreferences**: `\1` matches the same text as the first group.
4. **Named groups**: `(?P<name>...)` and `(?P=name)` for named backreferences.
5. **Nested groups**: Groups can be nested; numbering follows opening parentheses.
6. **Quantified groups**: `(ab)+` repeats the group.

#### Constraints and Limitations

- Backreferences only match the exact text previously captured, not the pattern.
- Backreferences cannot be used inside character classes.
- Group numbers refer to the opening parenthesis order, which can be confusing with nested groups.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Capturing Groups

```python
import re

# step1: Basic capturing
match = re.search(r'(\d+)-(\d+)', 'Call 555-1234')
print(match.group())     # 555-1234
print(match.group(1))    # 555
print(match.group(2))    # 1234
print(match.groups())    # ('555', '1234')

# step2: Named groups
match = re.search(r'(?P<area>\d+)-(?P<local>\d+)', 'Call 555-1234')
print(match.group('area'))   # 555
print(match.group('local'))  # 1234
print(match.groupdict())     # {'area': '555', 'local': '1234'}
```

**Expected Output**:
```
555-1234
555
1234
('555', '1234')
555
1234
{'area': '555', 'local': '1234'}
```

**Why**: Groups capture substrings; named groups provide dictionary-style access.

#### Example 2: Backreferences

```python
import re

# step1: Match repeated words
text = "Paris in the the spring"
match = re.search(r'\b(\w+)\s+\1\b', text)
print(match.group())  # the the

# step2: Named backreference
match = re.search(r'(?P<word>\w+)\s+(?P=word)', text)
print(match.group())  # the the
```

**Expected Output**:
```
the the
the the
```

**Why**: Backreferences match the same text as a previously captured group, useful for detecting repeated words.

#### Example 3: Group Extraction in Practice

```python
import re

log = "ERROR 2026-09-28 14:30:45 Connection timeout"

# step1: Extract structured fields
pattern = r'(?P<level>\w+)\s+(?P<date>\d{4}-\d{2}-\d{2})\s+(?P<time>\d{2}:\d{2}:\d{2})\s+(?P<msg>.+)'
match = re.search(pattern, log)
print(match.groupdict())
# {'level': 'ERROR', 'date': '2026-09-28', 'time': '14:30:45', 'msg': 'Connection timeout'}

# step2: Access individual fields
print(match.group('level'))  # ERROR
print(match.group('msg'))    # Connection timeout
```

**Expected Output**:
```
{'level': 'ERROR', 'date': '2026-09-28', 'time': '14:30:45', 'msg': 'Connection timeout'}
ERROR
Connection timeout
```

**Why**: Named groups make it easy to extract structured data from log lines.

### Real-World Cases

- **Log parsing**: Extracting timestamp, level, and message.
- **Date parsing**: Extracting year, month, and day.
- **URL parsing**: Extracting protocol, domain, and path.
- **Repeated word detection**: Using backreferences.

### References

- Grouping - https://docs.python.org/3/library/re.html#grouping
- Backreferences - https://docs.python.org/3/library/re.html#backreferences

---

## 6. Capturing

### Definitions

**Core Definition**: Capturing is the process by which groups in a pattern store the matched substrings, which can then be retrieved from the match object.

**Technical Definition**: When a group matches, the substring it matched is stored in the match object. The `group()` method returns the entire match or a specific group; `groups()` returns a tuple of all groups; `groupdict()` returns a dictionary of named groups. The `start()`, `end()`, and `span()` methods return position information.

**Beginner-Friendly Explanation**: Capturing means "saving" what a group matches so you can look at it later. After a match, you can ask for `match.group(1)` to get the first captured piece.

### Purposes

- To extract specific substrings from a larger match.
- To access parts of a match for further processing.
- To retrieve position information for matches.
- To work with named fields for readability.
- To build structured data from unstructured text.

### Syntax Rules and Structure

#### Complete General Syntax

```python
match.group()        # Entire match
match.group(n)       # Group n
match.group(name)    # Named group
match.groups()       # Tuple of all groups
match.groupdict()    # Dict of named groups
match.start()        # Start index
match.end()          # End index
match.span()         # (start, end) tuple
```

**Component Breakdown**:
- `.group()` — returns the full match or specified group.
- `.groups()` — returns all groups as a tuple.
- `.groupdict()` — returns named groups as a dictionary.
- `.start()`, `.end()`, `.span()` — position methods.

#### Syntax Rules

1. **Group numbering**: Group 0 is the entire match.
2. **Groups 1+**: Correspond to capturing groups in order.
3. **Unmatched groups**: Return `None` if a group did not participate in the match.
4. **Named groups**: Accessed by name or by number.
5. **Span**: `span()` returns a tuple `(start, end)`; `start()` and `end()` return individual indices.

#### Constraints and Limitations

- Groups that don't participate in a match return `None`.
- `groupdict()` only includes named groups.
- Position indices are zero-based.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Match Object Methods

```python
import re

m = re.search(r'(\d+)-(\d+)', 'Call 555-1234')

# step1: group()
print(m.group())       # 555-1234
print(m.group(0))      # 555-1234
print(m.group(1))      # 555
print(m.group(2))      # 1234

# step2: groups()
print(m.groups())      # ('555', '1234')

# step3: span()
print(m.span())        # (5, 13)
print(m.start())       # 5
print(m.end())         # 13
```

**Expected Output**:
```
555-1234
555-1234
555
1234
('555', '1234')
(5, 13)
5
13
```

**Why**: The match object provides comprehensive access to matched text and positions.

#### Example 2: Named Groups

```python
import re

pattern = r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})'
m = re.search(pattern, 'Date: 2026-09-28')

# step1: Named access
print(m.group('year'))    # 2026
print(m.group('month'))   # 09
print(m.group('day'))     # 28

# step2: groupdict()
print(m.groupdict())      # {'year': '2026', 'month': '09', 'day': '28'}

# step3: Position of a named group
print(m.span('year'))     # (6, 10)
```

**Expected Output**:
```
2026
09
28
{'year': '2026', 'month': '09', 'day': '28'}
(6, 10)
```

**Why**: Named groups provide readable, self-documenting access to captured fields.

#### Example 3: Optional Groups

```python
import re

# step1: Optional group
m = re.search(r'(\d+)(?:\.(\d+))?', '3.14')
print(m.group(1))  # 3
print(m.group(2))  # 14

m2 = re.search(r'(\d+)(?:\.(\d+))?', '42')
print(m2.group(1))  # 42
print(m2.group(2))  # None
```

**Expected Output**:
```
3
14
42
None
```

**Why**: Optional groups return `None` when they don't participate in the match.

### Real-World Cases

- **Form parsing**: Extracting name, email, and phone fields.
- **Log analysis**: Extracting structured data from log lines.
- **URL parsing**: Extracting components of a URL.
- **Data extraction**: Pulling specific values from text.

### References

- Match Objects - https://docs.python.org/3/library/re.html#match-objects

---

## 7. Non-Capturing Groups

### Definitions

**Core Definition**: A non-capturing group, written `(?:...)`, groups a subpattern without capturing the matched text.

**Technical Definition**: Non-capturing groups behave exactly like capturing groups in terms of grouping and quantification, but they do not store the matched substring and do not count toward group numbering. This is useful for applying quantifiers to a group without creating an unnecessary capture.

**Beginner-Friendly Explanation**: Non-capturing groups are like parentheses that don't save anything. They're useful when you want to group something (e.g., for a quantifier) but don't need to extract it later.

### Purposes

- To group subpatterns for quantification without capturing.
- To avoid unnecessary captures that complicate group numbering.
- To improve pattern performance by reducing capture overhead.
- To use alternation within a quantified element.
- To keep group numbering clean and predictable.

### Syntax Rules and Structure

#### Complete General Syntax

```python
(?:...)        # Non-capturing group
```

**Component Breakdown**:
- `(?:` — opening of a non-capturing group.
- `...` — the subpattern.
- `)` — closing parenthesis.

#### Syntax Rules

1. **No capture**: The group does not store matched text.
2. **No group number**: Non-capturing groups are not numbered.
3. **Quantifiable**: Can be quantified like any group.
4. **Nestable**: Can be nested inside other groups.
5. **Alternation support**: `(?:cat|dog)` groups alternatives.

#### Constraints and Limitations

- Cannot be backreferenced (no `\1` for a non-capturing group).
- Cannot be accessed via `match.group()`.
- Syntax is slightly more verbose than plain parentheses.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Non-Capturing vs. Capturing

```python
import re

# step1: Capturing group
m = re.search(r'(\d+)-(\d+)', '555-1234')
print(m.groups())  # ('555', '1234')

# step2: Non-capturing group
m2 = re.search(r'(?:\d+)-(?:\d+)', '555-1234')
print(m2.groups())  # ()
print(m2.group())   # 555-1234
```

**Expected Output**:
```
('555', '1234')
()
555-1234
```

**Why**: Non-capturing groups group the pattern but don't capture substrings; `groups()` returns an empty tuple.

#### Example 2: Quantifying Non-Capturing Groups

```python
import re

# step1: Repeat a non-capturing group
print(re.findall(r'(?:ab)+', 'ababab ab'))
# ['ababab', 'ab']

# step2: Alternation in non-capturing group
print(re.findall(r'(?:cat|dog)', 'cat dog cat'))
# ['cat', 'dog', 'cat']

# step3: Mixed capturing and non-capturing
m = re.search(r'(?:\d{3})-(\d{4})', '555-1234')
print(m.group(1))  # 1234
print(m.groups())  # ('1234',)
```

**Expected Output**:
```
['ababab', 'ab']
['cat', 'dog', 'cat']
1234
('1234',)
```

**Why**: Non-capturing groups can be quantified and used with alternation while keeping group numbering clean.

#### Example 3: Practical Use Case

```python
import re

# step1: Extract only the domain from URLs
urls = "Visit https://www.example.com/path and http://test.org/page"
pattern = r'https?://(?:www\.)?([\w.-]+)'
domains = re.findall(pattern, urls)
print(domains)  # ['example.com', 'test.org']

# step2: Without non-capturing group
pattern2 = r'https?://(www\.)?([\w.-]+)'
domains2 = re.findall(pattern2, urls)
print(domains2)  # [('www.', 'example.com'), (None, 'test.org')]
```

**Expected Output**:
```
['example.com', 'test.org']
[('www.', 'example.com'), (None, 'test.org')]
```

**Why**: The non-capturing group `(?:www\.)?` avoids capturing the optional `www.` prefix, keeping group numbering and results clean.

### Real-World Cases

- **URL parsing**: Ignoring optional prefixes.
- **Pattern grouping**: Quantifying alternatives without capture.
- **Performance**: Reducing capture overhead in high-volume matching.
- **Clean group numbering**: Keeping capturing groups focused on meaningful data.

### References

- Non-Capturing Groups - https://docs.python.org/3/library/re.html#non-capturing-groups

---

## 8. Lookahead

### Definitions

**Core Definition**: Lookahead assertions test whether a pattern matches ahead in the string without consuming any characters. They include positive lookahead `(?=...)` and negative lookahead `(?!...)`.

**Technical Definition**: Lookahead assertions are zero-width assertions: they match a position in the string based on what follows, but they do not consume any characters. Positive lookahead `(?=...)` succeeds if the pattern matches at the current position; negative lookahead `(?!...)` succeeds if the pattern does not match.

**Beginner-Friendly Explanation**: Lookahead lets you say "match this, but only if it's followed by that." For example, `\d+(?=px)` matches digits only if they are followed by "px" — but "px" itself is not part of the match.

### Purposes

- To assert that a pattern is followed by another pattern without consuming it.
- To exclude certain suffixes from matching.
- To implement conditional matching based on upcoming text.
- To validate passwords with multiple requirements.
- To extract values based on context.

### Syntax Rules and Structure

#### Complete General Syntax

```python
(?=...)        # Positive lookahead
(?!...)        # Negative lookahead
```

**Component Breakdown**:
- `(?=` — positive lookahead assertion.
- `(?!` — negative lookahead assertion.
- `...` — the pattern to assert.
- `)` — closing parenthesis.

#### Syntax Rules

1. **Zero-width**: Lookahead does not consume characters.
2. **No quantifiers**: Cannot be quantified directly.
3. **Can contain any pattern**: Groups, alternations, and classes are allowed.
4. **Position-based**: Tests the position, not the consumed text.
5. **Combined with other patterns**: Typically placed after the main pattern.

#### Constraints and Limitations

- Lookahead does not capture; use a separate group if capture is needed.
- Cannot be used inside character classes.
- Performance impact: may increase backtracking if not carefully designed.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Positive Lookahead

```python
import re

# step1: Match digits followed by "px"
print(re.findall(r'\d+(?=px)', '16px 8px 0.5'))
# ['16', '8']

# step2: Match "Isaac" followed by "Asimov"
print(re.findall(r'Isaac (?=Asimov)', 'Isaac Asimov Isaac Newton'))
# ['Isaac ']

# step3: Password with uppercase requirement
passwords = ['password', 'Password', 'PASSWORD', 'passWord']
valid = [p for p in passwords if re.search(r'(?=.*[A-Z])(?=.*[a-z])\d{8,}', p)]
print(valid)  # [] (none have 8+ digits)
```

**Expected Output**:
```
['16', '8']
['Isaac ']
[]
```

**Why**: Positive lookahead asserts that a pattern follows, without consuming it.

#### Example 2: Negative Lookahead

```python
import re

# step1: Match "python" not followed by "2"
print(re.findall(r'python(?!2)', 'python3 python2'))
# ['python']

# step2: Match file names not ending in .txt
files = ['report.pdf', 'data.txt', 'notes.md', 'log.txt']
valid = [f for f in files if re.search(r'.*\.(?!txt$)[a-z]+$', f)]
print(valid)  # ['report.pdf', 'notes.md']

# step3: Negative lookahead with alternation
print(re.findall(r'\b\w+(?!\w)', 'hello world test'))
# ['hello', 'world', 'test']
```

**Expected Output**:
```
['python']
['report.pdf', 'notes.md']
['hello', 'world', 'test']
```

**Why**: Negative lookahead asserts that a pattern does not follow, excluding unwanted matches.

#### Example 3: Lookahead for Validation

```python
import re

def validate_password(password):
    """Validate password: at least 8 chars, 1 uppercase, 1 lowercase, 1 digit."""
    pattern = r'^(?=.*[A-Z])(?=.*[a-z])(?=.*\d).{8,}$'
    return bool(re.match(pattern, password))

print(validate_password("Passw0rd"))   # True
print(validate_password("password"))   # False (no uppercase/digit)
print(validate_password("PASSWORD"))   # False (no lowercase/digit)
print(validate_password("Pass123"))    # True
print(validate_password("Pass1"))      # False (too short)
```

**Expected Output**:
```
True
False
False
True
False
```

**Why**: Multiple lookaheads check different requirements at the same position, enabling complex validation in a single pattern.

### Real-World Cases

- **Password validation**: Ensuring multiple character requirements.
- **Unit extraction**: Matching numbers followed by units (px, em, %).
- **File filtering**: Excluding certain extensions.
- **Context-sensitive matching**: Matching only when followed by specific text.

### References

- Lookahead Assertions - https://docs.python.org/3/library/re.html#lookahead-assertions

---

## 9. Lookbehind

### Definitions

**Core Definition**: Lookbehind assertions test whether a pattern matches behind the current position in the string without consuming any characters. They include positive lookbehind `(?<=...)` and negative lookbehind `(?<!...)`.

**Technical Definition**: Lookbehind assertions are zero-width assertions that check what precedes the current position. Positive lookbehind `(?<=...)` succeeds if the pattern matches the text immediately before the current position; negative lookbehind `(?<!...)` succeeds if the pattern does not match. The pattern inside lookbehind must be of fixed length.

**Beginner-Friendly Explanation**: Lookbehind lets you say "match this, but only if it's preceded by that." For example, `(?<=\$)\d+` matches digits only if they are preceded by a dollar sign — but the dollar sign is not part of the match.

### Purposes

- To assert that a pattern is preceded by another pattern without consuming it.
- To exclude certain prefixes from matching.
- To extract values based on preceding context.
- To validate formats with required prefixes.
- To implement conditional matching based on preceding text.

### Syntax Rules and Structure

#### Complete General Syntax

```python
(?<=...)       # Positive lookbehind
(?<!...)       # Negative lookbehind
```

**Component Breakdown**:
- `(?<=` — positive lookbehind assertion.
- `(?<!` — negative lookbehind assertion.
- `...` — the pattern to assert (must be fixed length).
- `)` — closing parenthesis.

#### Syntax Rules

1. **Fixed length required**: The pattern inside lookbehind must match a fixed number of characters (e.g., `abc` or `a|b` are allowed; `a*` and `a{3,4}` are not).
2. **Zero-width**: Does not consume characters.
3. **No quantifiers**: Cannot be quantified directly.
4. **Position-based**: Tests the position, not the consumed text.
5. **Patterns starting with lookbehind**: Will not match at the beginning of the string.

#### Constraints and Limitations

- Fixed-length requirement limits flexibility.
- Cannot use quantifiers like `*` or `+` inside lookbehind.
- Some regex engines (including Python's) have limited lookbehind support.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Positive Lookbehind

```python
import re

# step1: Match digits preceded by "$"
print(re.findall(r'(?<=\$)\d+\.\d{2}', '$9.99 and $19.99'))
# ['9.99', '19.99']

# step2: Match word after hyphen
print(re.findall(r'(?<=-)\w+', 'spam-egg'))
# ['egg']

# step3: Match text after "def "
print(re.findall(r'(?<=def )\w+', 'def foo(): pass'))
# ['foo']
```

**Expected Output**:
```
['9.99', '19.99']
['egg']
['foo']
```

**Why**: Positive lookbehind asserts that a pattern precedes the current position.

#### Example 2: Negative Lookbehind

```python
import re

# step1: Match digits not preceded by "$"
print(re.findall(r'(?<!\$)\b\d+\.\d{2}\b', 'Price: $9.99, tax: 1.50'))
# ['1.50']

# step2: Match words not preceded by "un"
print(re.findall(r'(?<!un)\w+', 'unhappy happy'))
# ['happy']

# step3: Negative lookbehind at start of string
print(re.findall(r'(?<!x)\d', '123 x456'))
# ['1', '2', '3']
```

**Expected Output**:
```
['1.50']
['happy']
['1', '2', '3']
```

**Why**: Negative lookbehind asserts that a pattern does not precede the current position.

#### Example 3: Combining Lookbehind with Lookahead

```python
import re

# step1: Extract text between two markers
text = "START hello END START world END"
print(re.findall(r'(?<=START ).+?(?= END)', text))
# ['hello', 'world']

# step2: Match digits preceded by "$" and followed by "."
print(re.findall(r'(?<=\$)\d+(?=\.)', '$9.99 $19.99'))
# ['9', '19']
```

**Expected Output**:
```
['hello', 'world']
['9', '19']
```

**Why**: Combining lookbehind and lookahead allows precise extraction without including the boundary markers.

### Real-World Cases

- **Price extraction**: Matching numbers preceded by currency symbols.
- **Function name extraction**: Matching identifiers after `def`.
- **Log parsing**: Extracting values after specific prefixes.
- **Data cleaning**: Removing prefixes without consuming them.

### References

- Lookbehind Assertions - https://docs.python.org/3/library/re.html#lookbehind-assertions

---

## 10. Substitution

### Definitions

**Core Definition**: Substitution is the process of replacing occurrences of a pattern in a string with a replacement string or the result of a replacement function.

**Technical Definition**: The `re.sub()` and `re.subn()` functions replace all non-overlapping occurrences of a pattern with a replacement. The replacement can be a string (with backreferences like `\1` or `\g<name>`) or a callable that receives a match object and returns the replacement string. `re.subn()` additionally returns the number of substitutions made.

**Beginner-Friendly Explanation**: Substitution is find-and-replace for patterns. `re.sub(r'\d+', 'N', 'abc123def456')` replaces all numbers with "N". You can also use captured groups in the replacement, like `\1` to reuse matched text.

### Purposes

- To replace all occurrences of a pattern in a string.
- To transform text using captured groups.
- To perform conditional replacements via functions.
- To count replacements with `subn()`.
- To clean or reformat text data.

### Syntax Rules and Structure

#### Complete General Syntax

```python
re.sub(pattern, repl, string, count=0, flags=0)
re.subn(pattern, repl, string, count=0, flags=0)
```

**Component Breakdown**:
- `pattern` — the regex pattern.
- `repl` — replacement string or callable.
- `string` — the target string.
- `count` — maximum number of replacements (0 = all).
- `flags` — optional flags.

#### Replacement String Syntax

| Syntax | Meaning |
|--------|---------|
| `\1`, `\2` | Backreference to group N |
| `\g<name>` | Named backreference |
| `\g<1>` | Unambiguous backreference |
| `\n` | Newline (if escaped) |

#### Syntax Rules

1. **Backreferences in replacement**: Use `\1`, `\2`, or `\g<name>`.
2. **Function replacement**: Callable receives a match object and returns a string.
3. **Count parameter**: Limits the number of replacements.
4. **`subn()` returns tuple**: `(new_string, count)`.
5. **Unknown escapes**: Left alone in the replacement string.

#### Constraints and Limitations

- Backreferences in replacement refer to groups in the pattern.
- Function replacement must return a string.
- Replacement string and pattern must be the same type (str or bytes).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Substitution

```python
import re

# step1: Replace all digits
print(re.sub(r'\d+', 'N', 'abc123def456'))
# abcNdefN

# step2: Replace with captured group
print(re.sub(r'(\w+)@(\w+)', r'\2 at \1', 'user@example'))
# example at user

# step3: Limit replacements
print(re.sub(r'\d+', 'N', '1 2 3 4 5', count=2))
# N N 3 4 5
```

**Expected Output**:
```
abcNdefN
example at user
N N 3 4 5
```

**Why**: Substitution replaces all matches by default; `count` limits the number.

#### Example 2: Named Groups in Replacement

```python
import re

# step1: Named backreference
text = "Date: 2026-09-28"
result = re.sub(
    r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})',
    r'\g<month>/\g<day>/\g<year>',
    text
)
print(result)  # Date: 09/28/2026

# step2: Using numbered backreferences
result2 = re.sub(r'(\d{4})-(\d{2})-(\d{2})', r'\2/\3/\1', text)
print(result2)  # Date: 09/28/2026
```

**Expected Output**:
```
Date: 09/28/2026
Date: 09/28/2026
```

**Why**: Named and numbered backreferences allow reordering captured groups in the replacement.

#### Example 3: Function Replacement

```python
import re

# step1: Function that returns a replacement
def hex_repl(match):
    return hex(int(match.group()))

print(re.sub(r'\d+', hex_repl, 'Call 65490 for printing'))
# Call 0xffd2 for printing

# step2: Function with conditional logic
def title_case(match):
    return match.group().capitalize()

print(re.sub(r'\b\w+\b', title_case, 'hello world python'))
# Hello World Python
```

**Expected Output**:
```
Call 0xffd2 for printing
Hello World Python
```

**Why**: A function replacement allows dynamic, context-aware substitutions.

### Real-World Cases

- **Date reformatting**: Converting between date formats.
- **Data anonymization**: Replacing sensitive data with placeholders.
- **Text normalization**: Standardizing whitespace and punctuation.
- **Template rendering**: Replacing placeholders with values.

### References

- `re.sub()` - https://docs.python.org/3/library/re.html#re.sub
- `re.subn()` - https://docs.python.org/3/library/re.html#re.subn

---

## 11. Validation

### Definitions

**Core Definition**: Validation is the use of regular expressions to verify that a string conforms to a required format or structure.

**Technical Definition**: Validation typically involves using `re.fullmatch()` or anchoring a pattern with `^...$` to ensure the entire string matches a schema. Common validation tasks include email addresses, phone numbers, URLs, dates, and postal codes.

**Beginner-Friendly Explanation**: Validation means checking that input "looks right." For example, you can use a regex to check that an email address has an `@` and a domain, or that a phone number has the right number of digits.

### Purposes

- To verify that user input matches an expected format.
- To ensure data integrity before processing.
- To provide immediate feedback on invalid input.
- To filter out malformed data.
- To enforce business rules on data formats.

### Syntax Rules and Structure

#### Complete General Syntax

```python
re.fullmatch(pattern, string)      # Entire string must match
re.match(r'^pattern$', string)     # Anchored equivalent
```

**Component Breakdown**:
- `pattern` — the validation pattern.
- `string` — the input to validate.
- Returns a match object or `None`.

#### Common Validation Patterns

| Use Case | Pattern | Notes |
|----------|---------|-------|
| Email (basic) | `[\w.+-]+@[\w-]+\.[\w.-]+` | Covers most real-world emails |
| Phone (US) | `(?:\+1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}` | Handles multiple formats |
| URL | `https?://[\w.-]+(?:\.[\w]{2,})(?:/[\w./?#&=-]*)?` | HTTP and HTTPS |
| IPv4 | `\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}` | Does not validate range |
| Date (ISO) | `\d{4}-\d{2}-\d{2}` | ISO 8601 format |

#### Syntax Rules

1. **Full match for validation**: Use `re.fullmatch()` or anchor with `^` and `$`.
2. **Escaping**: Escape special characters like `.`, `@`, and `/`.
3. **Character classes**: Use `\d` for digits, `\w` for word characters.
4. **Quantifiers**: Use `{m,n}` for precise length requirements.
5. **Lookaheads for complex rules**: Use lookaheads for multiple requirements.

#### Constraints and Limitations

- Regex validation is syntactic, not semantic (e.g., an IP pattern doesn't validate the range 0–255).
- Complex validation (e.g., email per RFC 5322) is often impractical with regex alone.
- Performance: overly broad patterns can cause backtracking.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Email Validation

```python
import re

EMAIL_RE = re.compile(r'^[\w.+-]+@[\w-]+\.[\w.-]+$')

def validate_email(email):
    return bool(EMAIL_RE.match(email))

print(validate_email("alice@example.com"))   # True
print(validate_email("bob@test.org"))        # True
print(validate_email("invalid"))             # False
print(validate_email("@missing.com"))        # False
print(validate_email("no@domain"))           # False
```

**Expected Output**:
```
True
True
False
False
False
```

**Why**: The pattern requires a local part, an `@`, a domain, and a TLD.

#### Example 2: Phone Number Validation

```python
import re

PHONE_RE = re.compile(r'^\+?1?[-.\s]?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$')

def validate_phone(phone):
    return bool(PHONE_RE.match(phone))

print(validate_phone("555-123-4567"))        # True
print(validate_phone("(555) 123-4567"))      # True
print(validate_phone("+1 555 123 4567"))     # True
print(validate_phone("5551234567"))          # True
print(validate_phone("555-1234"))            # False
```

**Expected Output**:
```
True
True
True
True
False
```

**Why**: The pattern handles multiple phone number formats with optional separators and country codes.

#### Example 3: Password Validation

```python
import re

def validate_password(password):
    """
    Validates a password:
    - At least 8 characters
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one digit
    - At least one special character
    """
    pattern = r'^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[!@#$%^&*]).{8,}$'
    return bool(re.match(pattern, password))

print(validate_password("Passw0rd!"))    # True
print(validate_password("password"))     # False
print(validate_password("PASSWORD"))     # False
print(validate_password("Pass1"))        # False (too short)
print(validate_password("Passw0rd"))     # False (no special char)
```

**Expected Output**:
```
True
False
False
False
False
```

**Why**: Multiple lookaheads check different requirements at the same position.

### Real-World Cases

- **Form validation**: Checking user input in web forms.
- **Data cleaning**: Filtering invalid records from datasets.
- **API input validation**: Ensuring request payloads are well-formed.
- **Configuration validation**: Checking config file values.

### References

- `re.fullmatch()` - https://docs.python.org/3/library/re.html#re.fullmatch
- Regular Expression HOWTO - https://docs.python.org/3/howto/regex.html

---

## 12. Regex Performance Considerations

### Definitions

**Core Definition**: Regex performance considerations are practices and techniques for ensuring that regular expressions execute efficiently and do not cause excessive backtracking or excessive memory usage.

**Technical Definition**: Python's `re` module uses a backtracking engine, which can exhibit exponential time complexity in the worst case (catastrophic backtracking) with certain pattern and input combinations. Performance can be improved by precompiling patterns (`re.compile()`), using non-greedy quantifiers, avoiding nested quantifiers, and employing atomic groups or possessive quantifiers where available.

**Beginner-Friendly Explanation**: Some regex patterns can become extremely slow on certain inputs — this is called "catastrophic backtracking." To avoid it, you should compile your patterns once, keep them simple, and avoid nested quantifiers. Python caches compiled patterns, but explicit compilation is still better for reused patterns.

### Purposes

- To avoid catastrophic backtracking that can freeze the program.
- To improve matching speed for repeated operations.
- To reduce memory usage by using iterators (`finditer()`) instead of lists.
- To build efficient patterns that minimize backtracking.
- To precompile patterns for reuse across multiple function calls.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import re

# Precompile at module level
PATTERN = re.compile(r'...')

# Use non-greedy quantifiers
r'<.*?>'

# Avoid nested quantifiers
r'(a+)+'          # BAD: catastrophic backtracking risk
r'a+'             # GOOD: simple quantifier

# Use finditer() for memory efficiency
for match in PATTERN.finditer(text):
    ...

# Use atomic groups (Python 3.11+)
r'(?>...)'
```

**Component Breakdown**:
- `re.compile()` — precompiles a pattern.
- `*?`, `+?` — lazy quantifiers reduce backtracking.
- `finditer()` — lazy iteration over matches.
- `(?>...)` — atomic group (prevents backtracking).

#### Performance Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| `(a+)+b` | Catastrophic backtracking | Use `a+b` or atomic groups |
| `.*.*` | Excessive backtracking | Use a single `.*` |
| `(a|b|ab)*` | Ambiguous alternation | Factor out common prefixes |
| Recompiling in a loop | Repeated compilation overhead | Precompile at module level |

#### Syntax Rules

1. **Precompile static patterns**: Compile at module scope when reused.
2. **Use non-greedy quantifiers**: `*?` and `+?` reduce backtracking.
3. **Avoid nested quantifiers**: `(a+)+` can cause exponential backtracking.
4. **Use atomic groups**: `(?>...)` prevents backtracking into the group (Python 3.11+).
5. **Use `finditer()` for memory**: Iterators are more memory-efficient than lists.
6. **Anchor patterns**: `^...$` can help the engine fail fast.
7. **Limit alternation**: Factor common prefixes out of alternatives.

#### Constraints and Limitations

- Atomic groups require Python 3.11+.
- Possessive quantifiers (`*+`, `++`) are not supported by Python's `re` module (but are available in the third-party `regex` module).
- The `re` module caches up to ~512 patterns; explicit compilation avoids cache lookups.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Precompilation

```python
import re
import time

# step1: Inline pattern (recompiled each call)
def extract_version_inline(text):
    match = re.search(r'v(\d+\.\d+\.\d+)', text)
    return match.group(1) if match else None

# step2: Precompiled pattern (compiled once)
VERSION_RE = re.compile(r'v(\d+\.\d+\.\d+)')

def extract_version_compiled(text):
    match = VERSION_RE.search(text)
    return match.group(1) if match else None

# step3: Benchmark
text = "v1.2.3"
N = 100000

start = time.time()
for _ in range(N):
    extract_version_inline(text)
inline_time = time.time() - start

start = time.time()
for _ in range(N):
    extract_version_compiled(text)
compiled_time = time.time() - start

print(f"Inline: {inline_time:.4f}s, Compiled: {compiled_time:.4f}s")
# Inline: 0.1234s, Compiled: 0.0876s
```

**Expected Output**:
```
Inline: 0.1234s, Compiled: 0.0876s
```

**Why**: Precompilation avoids the overhead of cache lookups and is faster for repeated use.

#### Example 2: Catastrophic Backtracking

```python
import re
import time

# step1: Dangerous pattern (nested quantifiers)
dangerous = re.compile(r'(a+)+b')

# step2: Safe pattern (no nesting)
safe = re.compile(r'a+b')

# step3: Test with safe input
text = "aaaaab"
start = time.time()
dangerous.search(text)
print(f"Dangerous (safe input): {time.time() - start:.6f}s")
# Dangerous (safe input): 0.000012s

# step4: Test with problematic input
text = "a" * 20 + "c"  # No 'b'
start = time.time()
safe.search(text)
print(f"Safe (problematic input): {time.time() - start:.6f}s")
# Safe (problematic input): 0.000015s

# Dangerous pattern on same input would take exponentially longer
```

**Expected Output**:
```
Dangerous (safe input): 0.000012s
Safe (problematic input): 0.000015s
```

**Why**: The nested quantifier `(a+)+` causes exponential backtracking on non-matching input; the simple `a+` is linear.

#### Example 3: Memory-Efficient Iteration

```python
import re

# step1: findall() returns a list (memory-heavy)
text = " ".join(str(i) for i in range(100000))
matches_list = re.findall(r'\d+', text)
print(f"findall: {len(matches_list)} matches")
# findall: 100000 matches

# step2: finditer() returns an iterator (memory-efficient)
matches_iter = re.finditer(r'\d+', text)
count = sum(1 for _ in matches_iter)
print(f"finditer: {count} matches")
# finditer: 100000 matches
```

**Expected Output**:
```
findall: 100000 matches
finditer: 100000 matches
```

**Why**: `finditer()` yields matches lazily, avoiding the memory cost of building a full list.

### Real-World Cases

- **High-volume log processing**: Precompiled patterns for performance.
- **Web scraping**: Efficient pattern matching to avoid timeouts.
- **Real-time systems**: Avoiding catastrophic backtracking.
- **Large datasets**: Using `finditer()` to process matches incrementally.

### References

- `re.compile()` - https://docs.python.org/3/library/re.html#re.compile
- Performance Considerations - https://docs.python.org/3/library/re.html#performance
- Atomic Groups - https://docs.python.org/3/library/re.html#atomic-grouping

---

## References

- `re` — Regular expression operations - https://docs.python.org/3/library/re.html
- Regular expression HOWTO - https://docs.python.org/3/howto/regex.html
- `re.compile()` - https://docs.python.org/3/library/re.html#re.compile
- `re.search()` - https://docs.python.org/3/library/re.html#re.search
- `re.match()` - https://docs.python.org/3/library/re.html#re.match
- `re.fullmatch()` - https://docs.python.org/3/library/re.html#re.fullmatch
- `re.findall()` - https://docs.python.org/3/library/re.html#re.findall
- `re.finditer()` - https://docs.python.org/3/library/re.html#re.finditer
- `re.sub()` - https://docs.python.org/3/library/re.html#re.sub
- `re.subn()` - https://docs.python.org/3/library/re.html#re.subn
- `re.split()` - https://docs.python.org/3/library/re.html#re.split
- Match Objects - https://docs.python.org/3/library/re.html#match-objects
- Regular Expression Syntax - https://docs.python.org/3/library/re.html#regular-expression-syntax
- Compilation Flags - https://docs.python.org/3/library/re.html#flags
- Character Classes - https://docs.python.org/3/library/re.html#character-classes
- Quantifiers - https://docs.python.org/3/library/re.html#quantifiers
- Grouping - https://docs.python.org/3/library/re.html#grouping
- Lookahead Assertions - https://docs.python.org/3/library/re.html#lookahead-assertions
- Lookbehind Assertions - https://docs.python.org/3/library/re.html#lookbehind-assertions
- Non-Capturing Groups - https://docs.python.org/3/library/re.html#non-capturing-groups
- Atomic Grouping - https://docs.python.org/3/library/re.html#atomic-grouping
- Performance Considerations - https://docs.python.org/3/library/re.html#performance
- Python Regex Cheat Sheet - https://www.dataquest.io/cheat-sheet/regular-expressions-cheat-sheet/
- Python Regex: The Complete Guide - https://docs.kanaries.net/topics/Python/python-regex