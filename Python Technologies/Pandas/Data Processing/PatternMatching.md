# Pandas Pattern Matching and Filtering: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Pattern matching and filtering in pandas is the process of testing whether strings in a Series or Index contain, start with, end with, or fully match a given pattern, and using the resulting boolean Series to filter rows in a DataFrame.

**Technical Definition:** Pandas provides a suite of vectorized string methods under the `.str` accessor that return boolean Series or Index objects indicating whether each element satisfies a pattern condition. These include `str.contains()` (uses `re.search` for substring matching), `str.match()` (uses `re.match` for prefix matching), `str.fullmatch()` (requires the entire string to match), `str.startswith()` and `str.endswith()` (for positional anchors). The resulting boolean masks are used with `df[mask]` boolean indexing to filter rows. All methods support regex patterns and the `na` parameter to control the treatment of missing values.

**Beginner-Friendly Explanation:** Pattern matching is like using “find” in a document, but for an entire column of text at once. You ask pandas “which rows contain the word 'error'?” or “which rows start with '2024-'?” and it gives you a True/False answer for each row. Then you use those True/False answers to keep only the rows you want.

### Key Characteristics

- **Vectorized execution:** All pattern matching methods operate on entire columns at C-level speed, returning boolean Series.
- **Regex-powered:** Methods use Python's `re` module under the hood: `str.contains()` uses `re.search` (anywhere in the string), `str.match()` uses `re.match` (at the beginning), and `str.fullmatch()` requires the entire string to match.
- **NA-safe:** The `na` parameter controls what value is returned for missing values (default: `NaN` for object dtype, `pd.NA` for StringDtype, `False` for the `str` dtype).
- **Case control:** The `case` parameter (default `True`) controls case sensitivity; the `flags` parameter passes additional `re` module flags like `re.IGNORECASE`.
- **Boolean indexing:** The result of any pattern matching method is a boolean Series that can be used directly to filter rows: `df[df['col'].str.contains('pattern')]`.

### Prerequisites

- Basic Python knowledge (strings, booleans, regular expressions).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of boolean indexing and the `.str` accessor.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Cleaning:** Filter rows based on text patterns (e.g., find invalid emails, malformed IDs).
- **Log Analysis:** Extract rows containing error messages, warnings, or specific event types.
- **Text Mining:** Select documents that mention specific keywords or phrases.
- **Data Validation:** Verify that strings conform to expected formats using regex patterns.

### Core Concepts / Features

1. Substring Evaluation: `str.contains()`
2. Boundary Validation: `str.startswith()` and `str.endswith()`
3. Rigid Compliance Checks: `str.match()` and `str.fullmatch()`
4. Regex Integration: Flags and Fuzzy Matching
5. Boolean Indexing: Using Pattern Results as Filters


## 1. Substring Evaluation: Checking for Substring Occurrences with `str.contains()`

### Definitions

**Core Definition:** `str.contains()` tests whether a substring or regular expression pattern is contained anywhere within each string element of a Series or Index, returning a boolean Series.

**Technical Definition:** `Series.str.contains(pat, case=True, flags=0, na=<no_default>, regex=True)` returns a boolean Series indicating whether the given pattern or regex is contained within the string of each element. The method uses `re.search` internally, meaning the pattern can match at any position in the string. The `regex` parameter (default `True` in pandas 1.x and 2.x) controls whether `pat` is interpreted as a regular expression or a literal string. In pandas 3.0, the default for `regex` is planned to change to `False` for the `str` dtype.

**Beginner-Friendly Explanation:** `str.contains()` is the most commonly used pattern matching method. It asks “does this string contain the pattern anywhere inside it?” For example, `df['text'].str.contains('error')` returns `True` for every row where the text contains the word “error”.

### Purposes

- To filter rows where a text column contains a specific substring (e.g., find all emails containing “@gmail”).
- To search for multiple alternative patterns using regex alternation (e.g., `'error|warning|critical'`).
- To perform case-insensitive substring searches using `case=False` or `flags=re.IGNORECASE`.
- To treat missing values as `False` (non-matching) using `na=False`, ensuring the result is a clean boolean Series.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
Series.str.contains(pat, case=True, flags=0, na=<no_default>, regex=True)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `pat` | Character sequence or regular expression |
| `case` | `True` (default): case sensitive; `False`: case insensitive |
| `flags` | `re` module flags (e.g., `re.IGNORECASE`, `re.MULTILINE`) |
| `na` | Value for missing values; default: `NaN` for object dtype, `pd.NA` for StringDtype, `False` for `str` dtype |
| `regex` | `True` (default): `pat` is a regex; `False`: `pat` is a literal string |
| Returns | Series or Index of booleans |

**Syntax Rules:**

- `str.contains()` uses `re.search`, so the pattern can match anywhere in the string.
- The `regex` parameter defaults to `True` in pandas 2.x; use `regex=False` for literal substring matching.
- The `na` parameter accepts a scalar; `na=False` converts missing values to `False`, returning a `bool` dtype instead of `object`.
- `case=False` is equivalent to `flags=re.IGNORECASE` when `regex=True`.

**Constraints and Limitations:**

- **Version-specific:** In pandas 3.0, the default for `regex` will change to `False` for the `str` dtype. Explicitly pass `regex=True` or `regex=False` to be future-proof.
- When `regex=True`, special characters in `pat` are interpreted as regex metacharacters. Use `re.escape()` for literal patterns containing special characters.
- The `na` parameter cannot be set to a non-boolean value for dtypes that do not allow it (deprecated).

### Annotated Code Examples

**Example 1: Basic Substring Search**

```python
import pandas as pd
import numpy as np

s1 = pd.Series(['Mouse', 'dog', 'house and parrot', '23', np.nan])

# Literal substring search for 'og'
result = s1.str.contains('og', regex=False)
print(result)
```

Expected output:
```
0    False
1     True
2    False
3    False
4      NaN
dtype: object
```

**Why this output:** With `regex=False`, `'og'` is treated as a literal substring. Only “dog” contains “og”. The `NaN` value returns `NaN` by default.

**Example 2: Handling Missing Values with na=False**

```python
# Convert missing values to False
result_na = s1.str.contains('og', na=False, regex=True)
print(result_na)
```

Expected output:
```
0    False
1     True
2    False
3    False
4    False
dtype: bool
```

**Why this output:** The `na=False` parameter replaces the `NaN` result with `False`, and the Series dtype becomes `bool` instead of `object`.

**Example 3: Regex Alternation and Case Insensitivity**

```python
# Search for 'house' or 'dog' (regex alternation)
print(s1.str.contains('house|dog', regex=True))
```

Expected output:
```
0    False
1     True
2     True
3    False
4      NaN
dtype: object
```
\
**Case-insensitive search using flags**
```py
import re
print(s1.str.contains('PARROT', flags=re.IGNORECASE, regex=True))
```

Expected output:
```
0    False
1    False
2     True
3    False
4      NaN
dtype: object
```

**Why this output:** The `|` operator in regex means “or”. The `re.IGNORECASE` flag makes the search case-insensitive, so `'PARROT'` matches “parrot” in “house and parrot”.

**Example 4: Regex Metacharacter Pitfall**

```python
s2 = pd.Series(['40', '40.0', '41', '41.0', '35'])

# '.0' as regex matches any character followed by 0
print(s2.str.contains('.0', regex=True))
```

Expected output:
```
0     True
1     True
2    False
3     True
4    False
dtype: bool
```

**Why this output:** In regex, `.` matches any character. So `.0` matches “40” (4 followed by 0), “40.0” (any char followed by 0, multiple matches), and “41.0”. To match a literal period, use `r'\.0'` or `regex=False`.

### Real-World Cases

- **Log filtering:** `df[df['message'].str.contains('ERROR|CRITICAL', regex=True)]` extracts error and critical log entries.
- **Email domain filtering:** `df[df['email'].str.contains('@gmail\.com', regex=True)]` finds Gmail users.
- **Product search:** `df[df['description'].str.contains('wireless', case=False, na=False)]` finds products mentioning “wireless”.

### References

- pandas.Series.str.contains — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.contains.html
- Working with Text Data (Testing for Strings that Match or Contain a Pattern) — https://pandas.pydata.org/docs/user_guide/text.html#testing-for-strings-that-match-or-contain-a-pattern


## 2. Boundary Validation: Testing Positional Anchors with `str.startswith()` and `str.endswith()`

### Definitions

**Core Definition:** `str.startswith()` and `str.endswith()` test whether each string element starts or ends with a specified substring, returning a boolean Series.

**Technical Definition:** `Series.str.startswith(pat, na=<no_default>)` is equivalent to Python's `str.startswith()` and tests if the start of each string element matches the pattern. `Series.str.endswith(pat, na=<no_default>)` tests the end. Neither method accepts regular expressions; `pat` must be a literal string. The `na` parameter controls the treatment of missing values.

**Beginner-Friendly Explanation:** These methods check the beginning or end of strings. For example, `df['filename'].str.endswith('.csv')` finds all CSV files, and `df['code'].str.startswith('US-')` finds codes that start with “US-”.

### Purposes

- To filter rows where a string starts with a specific prefix (e.g., IDs starting with “INV-”).
- To filter rows where a string ends with a specific suffix (e.g., file extensions, country codes).
- To validate that strings conform to positional format requirements.
- To combine prefix and suffix checks with other boolean conditions for complex filtering.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
Series.str.startswith(pat, na=<no_default>)
Series.str.endswith(pat, na=<no_default>)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `pat` | Literal string (regular expressions not accepted) |
| `na` | Value for missing values; default: `NaN` for object dtype, `pd.NA` for StringDtype, `False` for `str` dtype |
| Returns | Series or Index of booleans |

**Syntax Rules:**

- `pat` must be a literal string; regex patterns are not supported.
- `na=False` converts missing values to `False`, returning a `bool` dtype.
- Both methods are case-sensitive by default; there is no `case` parameter. Use `.str.lower()` before the check for case-insensitive matching.
- To check multiple prefixes/suffixes, use `.str.startswith(('A', 'B', 'C'))` with a tuple.

**Constraints and Limitations:**

- Regular expressions are not accepted; use `str.contains(r'^pattern')` or `str.match()` for regex-based prefix matching.
- The `pat` parameter must be a string or tuple of strings; other types raise `TypeError`.
- The `na` parameter cannot be a non-boolean value for dtypes that do not allow it (deprecated).

### Annotated Code Examples

**Example 1: Basic startswith and endswith**

```python
import pandas as pd
import numpy as np

s = pd.Series(['bat', 'Bear', 'cat', np.nan])

# Check if starts with 'b'
print(s.str.startswith('b'))
# Expected output:
# 0     True
# 1    False
# 2    False
# 3      NaN
# dtype: object

# Check if ends with 't'
print(s.str.endswith('t'))
# Expected output:
# 0     True
# 1    False
# 2     True
# 3      NaN
# dtype: object
```

**Why this output:** “bat” starts with “b” and ends with “t”. “Bear” starts with “B” (case-sensitive, so “b” does not match) and ends with “r”. “cat” ends with “t”.

**Example 2: Handling Missing Values**

```python
# Convert NaN to False
print(s.str.startswith('b', na=False))
```
Expected output:
```
0     True
1    False
2    False
3    False
dtype: bool
```

**Why this output:** The `na=False` parameter replaces the `NaN` with `False`, and the dtype becomes `bool`.

**Example 3: Multiple Prefixes with a Tuple**

```python
s2 = pd.Series(['apple', 'banana', 'cherry', 'avocado'])

# Check for multiple prefixes
print(s2.str.startswith(('a', 'b')))
```
Expected output:
```
0     True
1     True
2    False
3     True
dtype: bool
```

**Why this output:** The tuple `('a', 'b')` specifies two possible prefixes. A string matches if it starts with either prefix.

### Real-World Cases

- **File type filtering:** `df[df['filename'].str.endswith('.parquet')]` selects Parquet files.
- **ID validation:** `df[df['order_id'].str.startswith('ORD-')]` finds orders with the correct ID format.
- **Country code filtering:** `df[df['phone'].str.startswith('+1')]` finds North American phone numbers.

### References

- pandas.Series.str.startswith — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.startswith.html
- pandas.Series.str.endswith — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.endswith.html
- Working with Text Data (Methods like startswith and endswith) — https://pandas.pydata.org/docs/user_guide/text.html


## 3. Rigid Compliance Checks: `str.match()` vs. `str.fullmatch()`

### Definitions

**Core Definition:** `str.match()` tests whether each string starts with a match of a regular expression (the pattern can match a prefix), while `str.fullmatch()` requires the entire string to match the regular expression.

**Technical Definition:** `Series.str.match(pat, case=True, flags=0, na=<no_default>)` uses `re.match` internally, which anchors the pattern at the beginning of the string but does not require the entire string to match. `Series.str.fullmatch(pat, case=True, flags=0, na=<no_default>)` requires the entire string to match the pattern. Both methods accept regular expressions and return boolean Series.

**Beginner-Friendly Explanation:** `str.match()` is like checking if a string starts with a pattern. `str.fullmatch()` is stricter: the string must match the pattern completely from start to finish. For example, `str.match(r'\d+')` returns `True` for “123abc” (starts with digits), while `str.fullmatch(r'\d+')` returns `False` because “abc” is not a digit.

### Purposes

- To validate that strings conform to a specific format from the beginning (`str.match()`).
- To enforce complete format compliance where the entire string must match a pattern (`str.fullmatch()`).
- To extract or validate structured data such as dates, phone numbers, or IDs.
- To distinguish between prefix matches and full-string matches in data validation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
Series.str.match(pat, case=True, flags=0, na=<no_default>)
Series.str.fullmatch(pat, case=True, flags=0, na=<no_default>)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `pat` | Character sequence or regular expression |
| `case` | `True` (default): case sensitive |
| `flags` | `re` module flags (e.g., `re.IGNORECASE`) |
| `na` | Value for missing values |
| Returns | Series or Index of booleans |

**Syntax Rules:**

- `str.match()` uses `re.match`, which anchors the pattern at the start of the string but does not require the entire string to match.
- `str.fullmatch()` requires the entire string to match the pattern.
- `str.contains()` is the least strict, using `re.search` to find the pattern anywhere in the string.
- All three methods accept regex patterns and the `na` parameter.

**Constraints and Limitations:**

- `str.match()` does not require a full-string match; a pattern like `r'\d+'` matches “123abc” because it matches the prefix “123”.
- To mimic `str.fullmatch()` with `str.match()`, append `$` to the pattern (e.g., `r'\d+$'`).
- In pandas < 1.1.0, `str.fullmatch()` is not available; use `str.match(r'pattern$')` instead.

### Annotated Code Examples

**Example 1: match() vs. fullmatch() vs. contains()**

```python
import pandas as pd

s = pd.Series(['cat', 'duck', 'dove', 'dog'])

# str.match(r'd') — starts with 'd'
print("match(r'd'):")
print(s.str.match(r'd'))
# Expected output:
# 0    False
# 1     True
# 2     True
# 3     True
# dtype: bool

# str.fullmatch(r'd') — entire string is 'd'
print("\nfullmatch(r'd'):")
print(s.str.fullmatch(r'd'))
# Expected output:
# 0    False
# 1    False
# 2    False
# 3    False
# dtype: bool

# str.contains(r'd') — contains 'd' anywhere
print("\ncontains(r'd'):")
print(s.str.contains(r'd'))
# Expected output:
# 0    False
# 1     True
# 2     True
# 3     True
# dtype: bool
```

**Why this output:** `match(r'd')` returns `True` for strings starting with “d” (duck, dove, dog). `fullmatch(r'd')` returns `False` for all because no string is exactly “d”. `contains(r'd')` returns `True` for all strings containing “d” anywhere.

**Example 2: fullmatch() for Complete Format Validation**

```python
ser = pd.Series(["cat", "duck", "dove"])

# Full match: entire string must match 'd.+'
print(ser.str.fullmatch(r'd.+'))
# Expected output:
# 0    False
# 1     True
# 2     True
# dtype: bool
```

**Why this output:** The pattern `d.+` requires the string to start with “d” followed by at least one more character. “cat” does not match, “duck” and “dove” do.

**Example 3: match() with na Parameter**

```python
import numpy as np

s2 = pd.Series(['apple', np.nan, 'banana'])

# match with na=False
print(s2.str.match(r'a', na=False))
# Expected output:
# 0     True
# 1    False
# 2    False
# dtype: bool
```

**Why this output:** The pattern `a` matches strings starting with “a”. “apple” matches, “banana” does not, and `NaN` is converted to `False` via `na=False`.

### Real-World Cases

- **Date validation:** `df[df['date'].str.fullmatch(r'\d{4}-\d{2}-\d{2}')]` finds rows with valid ISO date format.
- **Phone number validation:** `df[df['phone'].str.fullmatch(r'\+1-\d{3}-\d{3}-\d{4}')]` validates US phone numbers.
- **Product code filtering:** `df[df['sku'].str.match(r'[A-Z]{3}-')]` finds SKUs starting with three uppercase letters followed by a dash.

### References

- pandas.Series.str.match — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.match.html
- pandas.Series.str.fullmatch — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.fullmatch.html
- Working with Text Data (Testing for Strings that Match or Contain a Pattern) — https://pandas.pydata.org/docs/user_guide/text.html


## 4. Regex Integration: Deploying Regular Expression Flags for Fuzzy Matching

### Definitions

**Core Definition:** Regex integration in pandas pattern matching refers to the use of Python's `re` module flags (e.g., `re.IGNORECASE`, `re.MULTILINE`) to modify how regular expression patterns are matched, enabling fuzzy or case-insensitive matching.

**Technical Definition:** The `flags` parameter in `str.contains()`, `str.match()`, `str.fullmatch()`, and `str.extract()` accepts an integer bitmask of `re` module flags. Multiple flags can be combined using the bitwise OR operator (`|`). The most commonly used flag is `re.IGNORECASE`, which makes pattern matching case-insensitive. Other useful flags include `re.MULTILINE` (for `^` and `$` to match at line boundaries) and `re.DOTALL` (for `.` to match newlines).

**Beginner-Friendly Explanation:** Flags are like settings that change how pattern matching works. `re.IGNORECASE` makes the search ignore case, so “Hello” and “hello” both match the pattern “hello”. You pass flags to methods like `str.contains()` to customize the search behaviour.

### Purposes

- To perform case-insensitive pattern matching using `re.IGNORECASE`.
- To enable multi-line matching where `^` and `$` match at the start/end of each line using `re.MULTILINE`.
- To allow `.` to match newline characters using `re.DOTALL`.
- To combine multiple flags for complex matching requirements.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
import re

# Case-insensitive matching
series.str.contains('pattern', flags=re.IGNORECASE, regex=True)

# Combining multiple flags
series.str.contains('pattern', flags=re.IGNORECASE | re.MULTILINE, regex=True)

# Using case parameter instead of flags
series.str.contains('pattern', case=False, regex=True)
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `re.IGNORECASE` | Case-insensitive matching |
| `re.MULTILINE` | `^` and `$` match at start/end of each line |
| `re.DOTALL` | `.` matches any character including newline |
| `re.VERBOSE` | Allow verbose regex with whitespace and comments |
| `re.ASCII` | Make `\w`, `\b`, `\d` match only ASCII characters |

**Syntax Rules:**

- The `flags` parameter accepts an integer bitmask; use `|` to combine flags.
- `case=False` is equivalent to `flags=re.IGNORECASE` when `regex=True`.
- Flags are only applied when `regex=True` (the default in pandas 2.x).
- `re.escape()` should be used to escape special characters when building patterns dynamically.

**Constraints and Limitations:**

- **Version-specific:** In pandas 3.0, the default for `regex` will change to `False`; explicitly pass `regex=True` when using flags.
- Using `flags` with a compiled regex pattern raises an error.
- The `case` and `flags` parameters cannot be used together with a compiled regex pattern.

### Annotated Code Examples

**Example 1: Case-Insensitive Matching**

```python
import pandas as pd
import re

s = pd.Series(['Mouse', 'dog', 'house and parrot', '23', np.nan])

# Case-insensitive search using flags
print(s.str.contains('PARROT', flags=re.IGNORECASE, regex=True))
# Expected output:
# 0    False
# 1    False
# 2     True
# 3    False
# 4      NaN
# dtype: object

# Equivalent using case=False
print(s.str.contains('PARROT', case=False, regex=True))
# Expected output:
# 0    False
# 1    False
# 2     True
# 3    False
# 4      NaN
# dtype: object
```

**Why this output:** The `re.IGNORECASE` flag makes the search ignore case, so `'PARROT'` matches “parrot” in “house and parrot”.

**Example 2: Combining Multiple Flags**

```python
# Multi-line and case-insensitive
text = pd.Series(['Error on line 1\nWarning on line 2', 'No issues'])
print(text.str.contains('^warning', flags=re.IGNORECASE | re.MULTILINE, regex=True))
# Expected output:
# 0     True
# 1    False
# dtype: bool
```

**Why this output:** `re.MULTILINE` makes `^` match at the start of each line, so `^warning` matches “Warning” at the beginning of the second line. `re.IGNORECASE` makes it case-insensitive.

**Example 3: Using re.escape() with Dynamic Patterns**

```python
# Dynamic pattern with special characters
search_term = "c++"
# Escape special characters
pattern = re.escape(search_term)
s3 = pd.Series(['c++ is great', 'java is okay'])
print(s3.str.contains(pattern, regex=True))
# Expected output:
# 0     True
# 1    False
# dtype: bool
```

**Why this output:** `re.escape("c++")` converts “c++” to `c\+\+`, which matches the literal “c++” rather than treating “+” as a regex quantifier.

### Real-World Cases

- **Log analysis:** `df[df['message'].str.contains('error', flags=re.IGNORECASE, regex=True)]` finds all error messages regardless of case.
- **Multi-line text:** `df[df['text'].str.contains('^NOTE:', flags=re.MULTILINE, regex=True)]` finds lines starting with “NOTE:”.
- **User search:** `df[df['title'].str.contains(re.escape(user_input), case=False, regex=True)]` safely searches for user-provided terms.

### References

- pandas.Series.str.contains (flags parameter) — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.contains.html
- Python re module — https://docs.python.org/3/library/re.html
- Working with Text Data (Regular expressions) — https://pandas.pydata.org/docs/user_guide/text.html


## 5. Boolean Indexing: Using Pattern Results as Filter Criteria

### Definitions

**Core Definition:** Boolean indexing with pattern matching uses the boolean Series returned by string methods (e.g., `str.contains()`, `str.startswith()`) as a mask to filter rows in a DataFrame.

**Technical Definition:** When a boolean Series is passed to `df[mask]`, pandas returns a new DataFrame containing only the rows where the mask is `True`. The mask must have the same length as the DataFrame and be aligned by index. Boolean masks can be combined using `&` (and), `|` (or), and `~` (not) operators, with parentheses around each condition.

**Beginner-Friendly Explanation:** Once you know which rows match your pattern (True) and which don't (False), you use that True/False list to keep only the matching rows. It's like using a highlighter to mark the rows you want, then hiding the rest.

### Purposes

- To filter a DataFrame to rows where a text column contains a specific pattern.
- To combine multiple pattern conditions (e.g., rows containing “error” AND starting with “2024-”).
- To exclude rows matching a pattern using the `~` (not) operator.
- To select rows based on string length, prefix, suffix, or full-match conditions.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Single condition
df[df['col'].str.contains('pattern', na=False)]

# Multiple conditions with & (and)
df[df['col1'].str.contains('A') & df['col2'].str.startswith('B')]

# Multiple conditions with | (or)
df[df['col'].str.contains('A') | df['col'].str.contains('B')]

# Negation with ~ (not)
df[~df['col'].str.contains('pattern', na=False)]
```

**Component Breakdown:**

| Operator | Description |
|----------|-------------|
| `&` | Logical AND; both conditions must be `True` |
| `\|` | Logical OR; at least one condition must be `True` |
| `~` | Logical NOT; inverts the boolean mask |
| `na=False` | Converts `NaN` results to `False`, ensuring a clean boolean mask |

**Syntax Rules:**

- Use `na=False` to convert `NaN` results to `False`, avoiding `ValueError` during filtering.
- Use `df[mask]` for filtering; `mask` must be a boolean Series.
- Combine conditions with `&`, `|`, `~` (not `and`, `or`, `not`).
- Enclose each condition in parentheses when combining.

**Constraints and Limitations:**

- The mask must be aligned with the DataFrame's index; boolean Series from `.str` methods are aligned by default.
- Without `na=False`, `NaN` values in the mask cause a `ValueError` when used for indexing.
- Combining conditions with mismatched indexes may produce unexpected results.

### Annotated Code Examples

**Example 1: Basic Filtering with str.contains()**

```python
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'email': ['alice@gmail.com', 'bob@yahoo.com', 'charlie@gmail.com', 'diana@outlook.com']
})

# Filter to Gmail users
gmail_users = df[df['email'].str.contains('@gmail\.com', regex=True, na=False)]
print(gmail_users)
# Expected output:
#       name            email
# 0    Alice  alice@gmail.com
# 2  Charlie  charlie@gmail.com
```

**Why this output:** The pattern `@gmail\.com` matches the Gmail domain. The mask returns `True` for Alice and Charlie, so only those rows are kept.

**Example 2: Combining Conditions with & and |**

```python
# Filter to Gmail users whose name starts with 'A' or 'C'
mask = (
    df['email'].str.contains('@gmail\.com', regex=True, na=False) &
    (df['name'].str.startswith('A') | df['name'].str.startswith('C'))
)
result = df[mask]
print(result)
# Expected output:
#       name            email
# 0    Alice  alice@gmail.com
# 2  Charlie  charlie@gmail.com
```

**Why this output:** The `&` operator requires both conditions to be `True`: the email must be Gmail AND the name must start with “A” or “C”. Both Alice and Charlie satisfy both conditions.

**Example 3: Negation with ~**

```python
# Filter to non-Gmail users
non_gmail = df[~df['email'].str.contains('@gmail\.com', regex=True, na=False)]
print(non_gmail)
# Expected output:
#    name            email
# 1   Bob    bob@yahoo.com
# 3 Diana  diana@outlook.com
```

**Why this output:** The `~` operator inverts the boolean mask, selecting rows where the email is NOT Gmail. Bob and Diana are retained.

### Real-World Cases

- **Error log filtering:** `df[df['message'].str.contains('ERROR|CRITICAL', regex=True, na=False)]` extracts error and critical log lines.
- **Product search:** `df[df['description'].str.contains('wireless|bluetooth', case=False, na=False)]` finds products with wireless or Bluetooth connectivity.
- **Data quality checks:** `df[~df['email'].str.contains('@', na=False)]` finds rows with invalid email addresses (missing “@”).

### References

- Indexing and selecting data (Boolean indexing) — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- Working with Text Data (Filtering with str.contains) — https://pandas.pydata.org/docs/user_guide/text.html


## References

- pandas.Series.str.contains — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.contains.html
- pandas.Series.str.match — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.match.html
- pandas.Series.str.fullmatch — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.fullmatch.html
- pandas.Series.str.startswith — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.startswith.html
- pandas.Series.str.endswith — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.endswith.html
- Working with Text Data — https://pandas.pydata.org/docs/user_guide/text.html
- Indexing and selecting data (Boolean indexing) — https://pandas.pydata.org/docs/user_guide/indexing.html
- Python re module — https://docs.python.org/3/library/re.html
- Leveraging Regular Expressions in Pandas — https://github.com/LMK89/Machine-Learning-MD/blob/main/Data-Science/Leveraging%20Regular%20Expressions%20in%20Pandas.md
- Pandas String Operations: Vectorized Text Cleaning — https://docs.kanaries.net/topics/Pandas/pandas-string-operations
- Pandas Filter Rows: Selecting Data by Condition in Python — https://docs.kanaries.net/articles/pandas-filter-rows