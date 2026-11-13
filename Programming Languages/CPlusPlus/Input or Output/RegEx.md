# C++ Regular Expressions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
C++ Regular Expressions is a component of the C++ Standard Library, declared in `<regex>` (since C++11), that provides facilities for pattern matching within strings using regular expressions.

**Technical Definition**
The regular expressions library provides `std::basic_regex` (with aliases `std::regex` for `char` and `std::wregex` for `wchar_t`), which encapsulates a compiled regular expression. The library offers three primary algorithm function templates — `std::regex_match` (matches the entire sequence), `std::regex_search` (searches for a match anywhere in the sequence), and `std::regex_replace` (replaces matched portions) — along with match result containers (`std::match_results`, `std::smatch`, `std::cmatch`) and iterators (`std::regex_iterator`, `std::regex_token_iterator`) for enumerating matches. Multiple grammar dialects are supported through `std::regex_constants::syntax_option_type`, including ECMAScript (the default), POSIX basic/extended, awk, grep, and egrep.

**Beginner-Friendly Explanation**
A regular expression (regex) is a mini-language for describing text patterns. Instead of writing long loops to check whether a string looks like an email address or to find all phone numbers in a document, you write a compact pattern like `\d{3}-\d{4}` and let the library do the work. `std::regex` compiles your pattern, `std::regex_match` checks whether an entire string fits the pattern, `std::regex_search` looks for the pattern anywhere inside a string, and `std::regex_replace` swaps matched text for something else.

### Key Characteristics

- **Multiple grammars** — ECMAScript (default), POSIX basic, POSIX extended, awk, grep, and egrep are available via `syntax_option_type`.
- **Compile once, match many** — constructing a `std::regex` compiles the pattern; the object can be reused for many match operations.
- **Rich match results** — `std::match_results` stores the full match and every captured sub-expression, accessible by index or by prefix/suffix.
- **Iterator-based enumeration** — `regex_iterator` walks through all matches; `regex_token_iterator` walks through sub-matches or unmatched tokens.
- **Format-string replacement** — `regex_replace` supports `$&`, `$1`, and other replacement directives via `match_flag_type`.
- **No compile-time checking** — patterns are validated at runtime; an invalid pattern throws `std::regex_error`.
- **Not header-only in all implementations** — GCC before 9.1 required linking with `-lstdc++fs` for `<filesystem>`, but `<regex>` itself is generally linked automatically with the standard library.

### Prerequisites

- C++11 (or later) compiler and standard library.
- Familiarity with `std::string` and string iteration.
- Basic understanding of exceptions (`try`/`catch`) and `std::regex_error`.
- Willingness to learn regular expression syntax (ECMAScript grammar is the default).

### Related Programming Areas

- **Text processing** — searching, validating, and transforming strings.
- **Input validation** — checking email addresses, phone numbers, dates, etc.
- **Parsing** — extracting structured data from unstructured text.
- **Search and replace** — bulk editing text files.
- **Log analysis** — filtering and extracting fields from log lines.

### Core Concepts / Features

1. `<regex>`
2. `std::regex` (and `std::wregex`)
3. `std::smatch`
4. `std::cmatch`
5. `std::regex_match`
6. `std::regex_search`
7. `std::regex_replace`
8. Regex iterators (`std::regex_iterator`, `std::regex_token_iterator`)
9. Regex performance
10. Regex best practices

---

## 1. `<regex>`

### Definitions

**Core Definition**
`<regex>` is the standard library header that declares the entire C++ regular expressions library.

**Technical Definition**
The header defines the class template `std::basic_regex` and its aliases `std::regex`/`std::wregex`, the match result class template `std::match_results` and its aliases (`std::cmatch`, `std::smatch`, `std::wcmatch`, `std::wsmatch`), the sub-match template `std::sub_match` and its aliases, the iterator templates `std::regex_iterator` and `std::regex_token_iterator` with their aliases, the algorithm function templates (`regex_match`, `regex_search`, `regex_replace`), the exception type `std::regex_error`, and the `std::regex_constants` namespace containing `syntax_option_type`, `match_flag_type`, and `error_type`.

**Beginner-Friendly Explanation**
`<regex>` is the single header you include to use regular expressions in C++. It gives you `std::regex` to compile patterns, the match functions, and the containers and iterators for working with results.

### Purposes

- To provide the declarations for all C++ regular expression components.
- To expose the `std::regex` family of types and functions.
- To enable runtime pattern matching and replacement.
- To support multiple regex grammars through `syntax_option_type`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <regex>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<regex>` | Standard header for the regular expressions library. |

**Syntax Rules**
- Must be included before using any regex type or function.
- The header is self-contained; no additional headers are required for basic usage.

**Constraints and Limitations**
- Requires C++11 or later.
- Some early implementations had incomplete support or performance issues.

### Annotated Code Example

```cpp
#include <regex>    // enables std::regex and algorithms
#include <iostream>

int main() {
    std::regex re("\\d+");              // compile a pattern
    std::string s = "abc 123 def";
    std::smatch m;
    if (std::regex_search(s, m, re)) {
        std::cout << "Found: " << m.str() << '\n';
    }
}
```

**Expected Output**
```
Found: 123
```

**Why this output?**  
`<regex>` declares `std::regex`, `std::smatch`, and `std::regex_search`; the pattern `\d+` matches the digit sequence `123`.

### Real-World Cases

- Any program that needs pattern-based text processing.
- Input validation layers in applications.
- Log parsing and data extraction tools.

### References

- `<regex>` header — https://en.cppreference.com/w/cpp/header/regex

---

## 2. `std::regex`

### Definitions

**Core Definition**
`std::regex` is the type alias for `std::basic_regex<char>`, representing a compiled regular expression for narrow-character strings.

**Technical Definition**
`std::basic_regex<CharT, Traits = std::regex_traits<CharT>>` is a class template that holds a compiled regular expression. The alias `std::regex` is `basic_regex<char>` and `std::wregex` is `basic_regex<wchar_t>`. The grammar is selected by `syntax_option_type` flags passed to the constructor; if no grammar is chosen, ECMAScript is assumed. Construction throws `std::regex_error` if the pattern is invalid.

**Beginner-Friendly Explanation**
`std::regex` is the compiled form of your pattern. You create it once with a pattern string, and then reuse it for matching, searching, or replacing. Think of it as a “loaded weapon” for pattern matching.

### Purposes

- To compile a regular expression pattern for reuse.
- To select the grammar dialect (ECMAScript, POSIX, etc.).
- To apply modifiers such as case-insensitivity or optimisation.
- To serve as the pattern argument for all regex algorithms.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::regex re("pattern");
std::regex re("pattern", std::regex::flag);
std::regex re("pattern", std::regex::ECMAScript | std::regex::icase);
```

**Component Breakdown**

| Syntax Option | Effect |
|---------------|--------|
| `ECMAScript` (default) | Modified ECMAScript grammar |
| `basic` | POSIX basic regular expressions (BRE) |
| `extended` | POSIX extended regular expressions (ERE) |
| `awk` | POSIX awk grammar |
| `grep` | POSIX grep grammar (basic + newline alternation) |
| `egrep` | POSIX egrep grammar (extended + newline alternation) |
| `icase` | Case-insensitive matching |
| `nosubs` | Treat marked sub-expressions as non-marking; `mark_count()` becomes 0 |
| `optimize` | Instruct the engine to make matching faster at the cost of construction time |
| `collate` | Make character ranges locale-sensitive |
| `multiline` (C++17) | `^` and `$` match line boundaries in ECMAScript mode |

**Syntax Rules**
- At most one grammar option can be selected from `ECMAScript`, `basic`, `extended`, `awk`, `grep`, `egrep`; if none is chosen, ECMAScript is assumed.
- Grammar variations (`icase`, `nosubs`, `optimize`, `collate`, `multiline`) can be combined with `|`.
- The pattern string must be valid for the selected grammar; otherwise `std::regex_error` is thrown.

**Constraints and Limitations**
- `std::regex` construction is relatively expensive; compile once and reuse.
- The ECMAScript engine is generally the most feature-complete but can suffer from catastrophic backtracking on certain patterns.
- `std::wregex` works with `wchar_t` strings but requires wide-character handling throughout.

### Annotated Code Examples

**Example 1: Basic compilation and matching**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::regex re("\\d{3}-\\d{4}");     // ECMAScript grammar (default)
    std::string s = "Call 555-1234 now";
    std::smatch m;
    if (std::regex_search(s, m, re)) {
        std::cout << "Phone: " << m.str() << '\n';
    }
}
```

**Expected Output**
```
Phone: 555-1234
```

**Why this output?**  
The pattern `\d{3}-\d{4}` matches three digits, a hyphen, and four digits.

**Example 2: Case-insensitive matching**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::regex re("hello", std::regex::icase);
    std::string s = "HELLO world";
    std::smatch m;
    if (std::regex_search(s, m, re)) {
        std::cout << "Matched: " << m.str() << '\n';
    }
}
```

**Expected Output**
```
Matched: HELLO
```

**Why this output?**  
`icase` makes the pattern match regardless of case.

### Real-World Cases

- **Input validation** — compiling a regex for email or date formats.
- **Search utilities** — reusing a compiled pattern across many strings.
- **Grammar-sensitive parsing** — selecting POSIX extended for Unix-style tools.

### References

- `std::basic_regex` — https://en.cppreference.com/w/cpp/regex/basic_regex
- `std::regex_constants::syntax_option_type` — https://en.cppreference.com/w/cpp/regex/syntax_option_type

---

## 3. `std::smatch`

### Definitions

**Core Definition**
`std::smatch` is the type alias for `std::match_results<std::string::const_iterator>`, holding the results of a regex match against a `std::string`.

**Technical Definition**
`std::match_results<BidirIt>` is a container-like class that stores a collection of character sequences representing the result of a regular expression match. The first sub-match (index 0) represents the full match; subsequent sub-matches correspond to the parenthesised capture groups in the pattern, in left-to-right order. `std::smatch` is the specialisation for `std::string::const_iterator`.

**Beginner-Friendly Explanation**
After you run a match or search, an `smatch` object holds the result. You can ask it what the whole match was (`m.str()` or `m[0]`), what each capture group matched (`m[1]`, `m[2]`, …), and what text came before and after the match (`m.prefix()`, `m.suffix()`).

### Purposes

- To capture and inspect the results of a regex match.
- To access individual capture groups.
- To obtain the prefix and suffix around a match.
- To serve as the output parameter for `regex_match` and `regex_search` when working with `std::string`.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::smatch m;
std::regex_search(s, m, re);
m[0].str();      // full match
m[1].str();      // first capture group
m.prefix().str(); // text before the match
m.suffix().str(); // text after the match
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| `m[0]` | Full match (a `sub_match`). |
| `m[n]` | n-th capture group. |
| `m.str(n)` | String of the n-th match. |
| `m.prefix()` | Text before the match. |
| `m.suffix()` | Text after the match. |
| `m.size()` | Number of sub-matches (1 + capture groups). |
| `m.ready()` | Whether the match results are valid. |

**Syntax Rules**
- `smatch` is only valid for `std::string` (use `cmatch` for C-strings).
- Capture groups are numbered by the order of their opening parentheses.
- `m[0]` always represents the entire match.

**Constraints and Limitations**
- If `regex_search` fails, the `smatch` object is left empty (`m.ready()` returns `false`).
- Accessing `m[n]` for an out-of-range `n` is undefined behaviour.

### Annotated Code Examples

**Example 1: Accessing capture groups**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string s = "John Smith";
    std::regex re("(\\w+) (\\w+)");
    std::smatch m;

    if (std::regex_match(s, m, re)) {
        std::cout << "Full: " << m[0].str() << '\n';
        std::cout << "First: " << m[1].str() << '\n';
        std::cout << "Last: " << m[2].str() << '\n';
    }
}
```

**Expected Output**
```
Full: John Smith
First: John
Last: Smith
```

**Why this output?**  
`m[1]` corresponds to the first `(\w+)` group, `m[2]` to the second.

**Example 2: Prefix and suffix**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string s = "abc 123 def";
    std::regex re("\\d+");
    std::smatch m;

    if (std::regex_search(s, m, re)) {
        std::cout << "Prefix: '" << m.prefix().str() << "'\n";
        std::cout << "Match:  '" << m[0].str() << "'\n";
        std::cout << "Suffix: '" << m.suffix().str() << "'\n";
    }
}
```

**Expected Output**
```
Prefix: 'abc '
Match:  '123'
Suffix: ' def'
```

**Why this output?**  
`prefix()` and `suffix()` return the text before and after the matched portion.

### Real-World Cases

- **Data extraction** — pulling fields out of structured text.
- **Validation feedback** — reporting which part of a pattern failed.
- **Parsing** — capturing key-value pairs from configuration lines.

### References

- `std::match_results` — https://en.cppreference.com/w/cpp/regex/match_results

---

## 4. `std::cmatch`

### Definitions

**Core Definition**
`std::cmatch` is the type alias for `std::match_results<const char*>`, holding the results of a regex match against a C-style string.

**Technical Definition**
`std::cmatch` is the specialisation of `std::match_results` for iterators of type `const char*`. It behaves like `smatch` but is used when the subject is a null-terminated character array or a `const char*` range.

**Beginner-Friendly Explanation**
`std::cmatch` is the C-string version of `std::smatch`. Use it when you are matching against `const char*` data rather than `std::string`.

### Purposes

- To capture match results when working with C-style strings.
- To interface regex matching with legacy C APIs.
- To avoid constructing a `std::string` when only a `const char*` is available.

### Syntax Rules and Structure

**General Syntax**
```cpp
const char* s = "text";
std::cmatch m;
std::regex_search(s, m, re);
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| Same interface as `smatch` | `m[0]`, `m[1]`, `prefix()`, `suffix()`, etc. |
| Underlying iterator type | `const char*` |

**Syntax Rules**
- The subject must be a `const char*` or a range of `const char*` iterators.
- Cannot be used with `std::string` subjects directly (use `smatch` for that).

**Constraints and Limitations**
- The subject must remain valid for the lifetime of the `cmatch` results (the sub-matches refer into the original buffer).

### Annotated Code Example

```cpp
#include <regex>
#include <iostream>

int main() {
    const char* s = "ID: 42";
    std::regex re("(\\d+)");
    std::cmatch m;

    if (std::regex_search(s, m, re)) {
        std::cout << "Number: " << m[1].str() << '\n';
    }
}
```

**Expected Output**
```
Number: 42
```

**Why this output?**  
`cmatch` captures the digits from the C-string subject.

### Real-World Cases

- **C interop** — processing `const char*` buffers received from C libraries.
- **Embedded systems** — avoiding `std::string` allocation.
- **Legacy code** — matching against null-terminated arrays.

### References

- `std::match_results` — https://en.cppreference.com/w/cpp/regex/match_results

---

## 5. `std::regex_match`

### Definitions

**Core Definition**
`std::regex_match` is a function template that determines whether a regular expression matches an **entire** character sequence.

**Technical Definition**
`std::regex_match` attempts to match the regex against the whole input range `[first, last)` (or the whole string). It returns `true` only if the entire sequence is consumed by the match. Overloads accept `std::match_results` to store the result; if the match fails, the match results are undefined (typically empty).

**Beginner-Friendly Explanation**
`std::regex_match` asks: “Does this **entire** string fit the pattern?” It returns `true` only if the pattern matches from the first character to the last. For finding a pattern anywhere inside a string, use `regex_search` instead.

### Purposes

- To validate that a string conforms exactly to a pattern.
- To extract all capture groups from a full match.
- To enforce strict format requirements (e.g., “the whole line must be a number”).
- To distinguish full matches from partial matches.

### Syntax Rules and Structure

**General Syntax**
```cpp
bool ok = std::regex_match(s, re);
bool ok = std::regex_match(s, m, re);
bool ok = std::regex_match(first, last, m, re);
```

**Component Breakdown**

| Overload | Description |
|----------|-------------|
| `(s, re)` | Returns whether the whole `s` matches `re`. |
| `(s, m, re)` | Also stores the match in `m`. |
| `(first, last, m, re)` | Matches an iterator range. |

**Syntax Rules**
- The entire sequence must be matched; a partial match returns `false`.
- The `match_results` object is only valid if the function returns `true`.

**Constraints and Limitations**
- Does not support unanchored searches (use `regex_search` for that).
- Performance can degrade with complex patterns on long strings due to backtracking.

### Annotated Code Examples

**Example 1: Full-string validation**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::regex re("\\d{3}-\\d{4}");
    std::string s1 = "555-1234";
    std::string s2 = "Call 555-1234";

    std::cout << std::boolalpha;
    std::cout << std::regex_match(s1, re) << '\n';   // true
    std::cout << std::regex_match(s2, re) << '\n';   // false
}
```

**Expected Output**
```
true
false
```

**Why this output?**  
`s1` is entirely a phone number; `s2` contains extra text, so the full match fails.

**Example 2: Capturing from a full match**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string s = "2025-01-15";
    std::regex re("(\\d{4})-(\\d{2})-(\\d{2})");
    std::smatch m;

    if (std::regex_match(s, m, re)) {
        std::cout << "Year: " << m[1].str() << '\n';
        std::cout << "Month: " << m[2].str() << '\n';
        std::cout << "Day: " << m[3].str() << '\n';
    }
}
```

**Expected Output**
```
Year: 2025
Month: 01
Day: 15
```

**Why this output?**  
`regex_match` succeeds because the whole string is a date, and each capture group is accessible.

### Real-World Cases

- **Form validation** — ensuring an input is exactly a valid email, phone number, or date.
- **Protocol parsing** — verifying that a message conforms to a strict format.
- **Unit testing** — asserting that strings match expected patterns exactly.

### References

- `std::regex_match` — https://en.cppreference.com/w/cpp/regex/regex_match

---

## 6. `std::regex_search`

### Definitions

**Core Definition**
`std::regex_search` is a function template that searches for the first occurrence of a regular expression anywhere within a character sequence.

**Technical Definition**
`std::regex_search` attempts to match the regex against any subsequence of the input. It returns `true` if a match is found and populates the `match_results` with the matched portion, its capture groups, and the prefix/suffix. To find all matches, call it in a loop, restarting from `m[0].second` of the previous result.

**Beginner-Friendly Explanation**
`std::regex_search` asks: “Is this pattern found **anywhere** in the string?” It returns `true` and tells you where the first match is. To find every match, you call it repeatedly on the remaining part of the string (or use `regex_iterator`).

### Purposes

- To locate a pattern anywhere inside a larger string.
- To extract the first occurrence and its context (prefix/suffix).
- To drive iterative matching by advancing past each match.
- To perform “find” operations without requiring a full-string match.

### Syntax Rules and Structure

**General Syntax**
```cpp
bool found = std::regex_search(s, re);
bool found = std::regex_search(s, m, re);
bool found = std::regex_search(first, last, m, re);
```

**Component Breakdown**

| Overload | Description |
|----------|-------------|
| `(s, re)` | Returns whether any match exists. |
| `(s, m, re)` | Stores the first match in `m`. |
| `(first, last, m, re)` | Searches an iterator range. |

**Syntax Rules**
- Returns `true` if at least one match exists; `false` otherwise.
- The `match_results` object contains the first match only.
- To find subsequent matches, restart from `m[0].second` or use `regex_iterator`.

**Constraints and Limitations**
- Only the first match is returned per call.
- Complex patterns can cause catastrophic backtracking on large inputs.

### Annotated Code Examples

**Example 1: Finding a substring**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string s = "The price is 42 dollars";
    std::regex re("\\d+");
    std::smatch m;

    if (std::regex_search(s, m, re)) {
        std::cout << "Found: " << m.str() << '\n';
        std::cout << "Before: " << m.prefix().str() << '\n';
        std::cout << "After: " << m.suffix().str() << '\n';
    }
}
```

**Expected Output**
```
Found: 42
Before: The price is 
After:  dollars
```

**Why this output?**  
`regex_search` finds `42` anywhere in the string and reports the surrounding text.

**Example 2: Iterative search**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string s = "a1 b2 c3";
    std::regex re("\\w\\d");
    std::smatch m;
    auto begin = s.cbegin();
    auto end = s.cend();

    while (std::regex_search(begin, end, m, re)) {
        std::cout << m.str() << ' ';
        begin = m[0].second;           // advance past the match
    }
}
```

**Expected Output**
```
a1 b2 c3
```

**Why this output?**  
Each iteration searches from the end of the previous match, finding all three tokens.

### Real-World Cases

- **Log scanning** — extracting timestamps or IDs from log lines.
- **Search-and-highlight** — finding occurrences for display.
- **Tokenisation** — pulling out words or numbers from mixed text.

### References

- `std::regex_search` — https://en.cppreference.com/w/cpp/regex/regex_search

---

## 7. `std::regex_replace`

### Definitions

**Core Definition**
`std::regex_replace` is a function template that replaces occurrences of a regular expression with formatted replacement text.

**Technical Definition**
`std::regex_replace` copies non-matched portions of the input as-is and replaces each matched subsequence with a string produced by expanding the replacement format according to `match_flag_type` rules. The replacement format can contain `$&` (the full match), `$1`–`$9` (capture groups), `$`` (prefix), `$'` (suffix), and `$$` (literal `$`). The function can operate in-place or write to an output iterator.

**Beginner-Friendly Explanation**
`regex_replace` finds every match of a pattern and swaps it for new text. You can refer to what was matched using `$&` and to captured groups using `$1`, `$2`, etc. It is the regex equivalent of “find and replace all” in a text editor.

### Purposes

- To transform matched text into a different representation.
- To redact, reformat, or reorder captured groups.
- To sanitise input by removing or replacing unwanted patterns.
- To perform bulk text substitutions in one call.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::string result = std::regex_replace(s, re, replacement);
std::regex_replace(out_it, first, last, re, replacement, flags);
```

**Component Breakdown**

| Replacement Token | Meaning |
|-------------------|---------|
| `$&` | The entire match. |
| `$1` – `$9` | The corresponding capture group. |
| `$`` | The prefix (text before the match). |
| `$'` | The suffix (text after the match). |
| `$$` | A literal `$`. |

**Syntax Rules**
- The replacement format is interpreted according to `match_flag_type` (default `format_default`; `format_sed` changes the syntax).
- By default, all non-overlapping matches are replaced.
- The `format_no_copy` flag prevents copying unmatched text.

**Constraints and Limitations**
- The replacement string syntax depends on the flags; mixing `$` and `\` conventions can be confusing.
- Does not support conditional replacement or evaluation; it is purely textual.

### Annotated Code Examples

**Example 1: Simple replacement**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string s = "I like cats and cats like me";
    std::regex re("cats");
    std::string result = std::regex_replace(s, re, "dogs");
    std::cout << result << '\n';
}
```

**Expected Output**
```
I like dogs and dogs like me
```

**Why this output?**  
Every occurrence of `cats` is replaced by `dogs`.

**Example 2: Using capture groups**

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string s = "2025-01-15";
    std::regex re("(\\d{4})-(\\d{2})-(\\d{2})");
    std::string result = std::regex_replace(s, re, "$3/$2/$1");
    std::cout << result << '\n';
}
```

**Expected Output**
```
15/01/2025
```

**Why this output?**  
`$3`, `$2`, `$1` refer to the day, month, and year capture groups, reordering them.

### Real-World Cases

- **Data transformation** — converting date formats.
- **Sanitisation** — replacing sensitive information with placeholders.
- **Templating** — substituting variables in text templates.

### References

- `std::regex_replace` — https://en.cppreference.com/w/cpp/regex/regex_replace

---

## 8. Regex Iterators

### Definitions

**Core Definition**
Regex iterators are iterator adaptors that enumerate matches (`std::regex_iterator`) or sub-matches/tokens (`std::regex_token_iterator`) within a character sequence.

**Technical Definition**
`std::regex_iterator` is a read-only forward iterator that accesses the individual matches of a regular expression within the underlying character sequence. `std::regex_token_iterator` is a read-only forward iterator that accesses individual sub-matches of every match (or the unmatched tokens between matches). Common aliases include `std::sregex_iterator` (`std::string::const_iterator`) and `std::cregex_iterator` (`const char*`), along with token-iterator counterparts.

**Beginner-Friendly Explanation**
`regex_iterator` walks through every match of a pattern in a string, like a cursor that jumps from match to match. `regex_token_iterator` is more flexible: it can walk through capture groups, or through the pieces of text **between** matches (useful for splitting strings).

### Purposes

- To enumerate all matches without writing a manual search loop.
- To split a string by a pattern using `regex_token_iterator` with `-1`.
- To iterate over specific capture groups across all matches.
- To integrate regex matching with standard algorithms and range-based loops.

### Syntax Rules and Structure

**General Syntax**
```cpp
for (std::sregex_iterator it(s.begin(), s.end(), re), end; it != end; ++it) {
    // it->str() is the full match
}

std::sregex_token_iterator it(s.begin(), s.end(), re, -1);  // unmatched tokens
```

**Component Breakdown**

| Iterator | Description |
|----------|-------------|
| `regex_iterator` | Enumerates full matches. |
| `regex_token_iterator` | Enumerates sub-matches or unmatched tokens. |
| `sregex_iterator` | For `std::string`. |
| `cregex_iterator` | For `const char*`. |

**Syntax Rules**
- The end iterator is default-constructed.
- `regex_token_iterator` with submatch index `-1` yields the text **between** matches.
- With submatch index `0`, it behaves like `regex_iterator` for full matches.

**Constraints and Limitations**
- The iterators reference the original string; the string must remain alive.
- Iteration order is the order of matches in the input.

### Annotated Code Examples

**Example 1: Enumerating all matches**

```cpp
#include <regex>
#include <iostream>
#include <string>

int main() {
    std::string s = "one 1 two 22 three 333";
    std::regex re("\\d+");

    for (std::sregex_iterator it(s.begin(), s.end(), re), end; it != end; ++it) {
        std::cout << it->str() << ' ';
    }
}
```

**Expected Output**
```
1 22 333
```

**Why this output?**  
`sregex_iterator` visits each digit sequence in turn.

**Example 2: Splitting with `regex_token_iterator`**

```cpp
#include <regex>
#include <iostream>
#include <string>

int main() {
    std::string s = "apple,banana,,cherry";
    std::regex re(",");

    std::sregex_token_iterator it(s.begin(), s.end(), re, -1), end;
    for (; it != end; ++it) {
        std::cout << "'" << it->str() << "'\n";
    }
}
```

**Expected Output**
```
'apple'
'banana'
''
'cherry'
```

**Why this output?**  
Submatch index `-1` yields the text **between** matches, effectively splitting on commas (including the empty token).

### Real-World Cases

- **CSV parsing** — splitting fields with `regex_token_iterator`.
- **Log tokenisation** — extracting every match in a line.
- **Search result highlighting** — iterating matches for display.

### References

- `std::regex_iterator` — https://en.cppreference.com/w/cpp/regex/regex_iterator
- `std::regex_token_iterator` — https://en.cppreference.com/w/cpp/regex/regex_token_iterator

---

## 9. Regex Performance

### Definitions

**Core Definition**
Regex performance refers to the runtime cost of compiling patterns and executing match, search, and replace operations, and the factors that influence it.

**Technical Definition**
`std::regex` typically uses a backtracking engine for ECMAScript grammar. Backtracking can exhibit exponential time complexity on certain patterns with nested quantifiers (catastrophic backtracking / ReDoS). Construction of `std::regex` is expensive relative to matching, so reuse is essential. The `optimize` flag can instruct the engine to trade construction time for faster matching. In practice, `std::regex` is often significantly slower than alternatives such as Boost.Regex or RE2.

**Beginner-Friendly Explanation**
Regex matching can be fast for simple patterns but very slow for certain “pathological” patterns. Compiling a regex is expensive, so you should compile once and reuse the object. If performance matters and you are matching untrusted input, consider safer or faster engines (Boost.Regex, RE2, or hand-written parsers).

### Purposes (Performance Considerations)

- To understand why certain patterns become exponentially slow.
- To choose strategies that avoid unnecessary compilation.
- To select flags (`optimize`) that improve matching speed at the cost of construction time.
- To decide when to use `std::regex` versus an alternative library.

### Key Performance Factors

| Factor | Impact |
|--------|--------|
| **Compilation cost** | `std::regex` construction is expensive; reuse the object. |
| **Backtracking** | Nested quantifiers like `(a+)+` can cause exponential blowup. |
| **Grammar choice** | ECMAScript is the most feature-rich but may be slower than POSIX on simple patterns. |
| **`optimize` flag** | Faster matching at the cost of slower construction. |
| **String length** | Longer inputs amplify backtracking costs. |
| **Implementation quality** | libstdc++ and MSVC implementations differ; some are notably slower than Boost. |

### Annotated Code Example: Reusing a compiled regex

```cpp
#include <regex>
#include <iostream>
#include <vector>

int main() {
    // Compile once — expensive
    std::regex re("\\d{3}-\\d{4}", std::regex::optimize);

    std::vector<std::string> inputs = {
        "555-1234", "abc", "111-2222", "no match"
    };

    for (const auto& s : inputs) {
        std::smatch m;
        if (std::regex_search(s, m, re)) {
            std::cout << s << " -> " << m.str() << '\n';
        }
    }
}
```

**Expected Output**
```
555-1234 -> 555-1234
111-2222 -> 111-2222
```

**Why this output?**  
The regex is compiled once and reused across all inputs; `optimize` trades construction time for faster matching.

### Real-World Cases

- **High-throughput servers** — avoiding per-request regex compilation.
- **Security-sensitive input** — recognising catastrophic backtracking risks.
- **Performance-critical tools** — benchmarking `std::regex` against Boost or RE2.

### References

- `std::regex_constants::syntax_option_type` (optimize flag) — https://en.cppreference.com/w/cpp/regex/syntax_option_type
- C++ regex performance discussion — https://stackoverflow.com/questions/2172715/why-is-stdregex-so-slow

---

## 10. Regex Best Practices

### Definitions

**Core Definition**
Regex best practices are guidelines for using `std::regex` correctly, safely, and efficiently.

**Technical Definition**
These practices include compiling patterns once and reusing them, preferring `regex_search` over `regex_match` when unanchored matching is intended, handling `std::regex_error` exceptions, avoiding catastrophic backtracking patterns, choosing the simplest adequate grammar, and considering alternative engines when performance is critical.

**Beginner-Friendly Explanation**
Best practices are good habits: don’t recompile the same pattern in a loop, be careful with patterns that can cause exponential slowdown, always handle errors, and use the right tool for the job.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Compile once, reuse** | Construct `std::regex` outside loops. |
| **Prefer `regex_search` for “contains”** | Use `regex_match` only for full-string validation. |
| **Handle `regex_error`** | Wrap regex construction in `try`/`catch`. |
| **Avoid catastrophic backtracking** | Do not use nested quantifiers like `(a+)+` on untrusted input. |
| **Choose the right grammar** | Use `basic` or `extended` when ECMAScript features are unnecessary. |
| **Use `regex_token_iterator` for splitting** | Submatch index `-1` is the idiomatic split. |
| **Benchmark before optimising** | `std::regex` may be slower than alternatives; measure first. |
| **Use raw string literals for patterns** | `R"(\d+)"` avoids escaping headaches. |

### Annotated Code Example

```cpp
#include <regex>
#include <iostream>
#include <string>

int main() {
    // Compile once (raw string literal for readability)
    std::regex re(R"(\b\w+@\w+\.\w+\b)", std::regex::icase);

    std::string input = "Contact: alice@example.com";
    std::smatch m;

    // Use regex_search for “contains” semantics
    if (std::regex_search(input, m, re)) {
        std::cout << "Email: " << m.str() << '\n';
    }
}
```

**Expected Output**
```
Email: alice@example.com
```

**Why this output?**  
The pattern is compiled once with a raw string literal; `regex_search` finds the email substring.

### Real-World Cases

- **Production input validation** — safe, reusable regex patterns.
- **CLI tools** — splitting and parsing arguments.
- **Library APIs** — documenting grammar choices and performance characteristics.

### References

- `std::regex` — https://en.cppreference.com/w/cpp/regex/basic_regex
- `std::regex_search` — https://en.cppreference.com/w/cpp/regex/regex_search
- Catastrophic backtracking (ReDoS) — https://en.wikipedia.org/wiki/ReDoS

---

## References

- Regular expressions library (C++11) — https://en.cppreference.com/w/cpp/regex
- `<regex>` header — https://en.cppreference.com/w/cpp/header/regex
- `std::basic_regex` — https://en.cppreference.com/w/cpp/regex/basic_regex
- `std::regex_constants::syntax_option_type` — https://en.cppreference.com/w/cpp/regex/syntax_option_type
- `std::match_results` — https://en.cppreference.com/w/cpp/regex/match_results
- `std::regex_match` — https://en.cppreference.com/w/cpp/regex/regex_match
- `std::regex_search` — https://en.cppreference.com/w/cpp/regex/regex_search
- `std::regex_replace` — https://en.cppreference.com/w/cpp/regex/regex_replace
- `std::regex_iterator` — https://en.cppreference.com/w/cpp/regex/regex_iterator
- `std::regex_token_iterator` — https://en.cppreference.com/w/cpp/regex/regex_token_iterator
- ECMAScript grammar (modified) — https://en.cppreference.com/w/cpp/regex/ecmascript
- Regular expressions (C++) — Microsoft Learn — https://learn.microsoft.com/en-us/cpp/standard-library/regular-expressions-cpp

---

This cheat sheet provides a comprehensive, structured reference for the C++ regular expressions library, covering every requested component individually, with definitions, purposes, syntax rules, annotated code examples, performance considerations, real-world cases, and authoritative references.