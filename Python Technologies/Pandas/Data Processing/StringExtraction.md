# Pandas String Extraction and Parsing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** String extraction and parsing in pandas is the process of isolating structured information from unstructured or semi-structured text columns using regular expression capture groups, and reshaping the results into tabular features.

**Technical Definition:** Pandas provides three primary vectorized methods for string extraction and parsing: `Series.str.extract(pat, flags=0, expand=True)`, which extracts capture groups from the **first** match of a regular expression in each subject string; `Series.str.extractall(pat, flags=0)`, which extracts capture groups from **all** matches, returning a DataFrame with a MultiIndex; and `Series.str.split(pat, expand=True)` combined with `DataFrame.explode(column)`, which divides strings into list-like elements and expands them into separate rows. Named capture groups using the `(?P<name>...)` syntax dynamically name the resulting columns, eliminating manual column assignment.

**Beginner-Friendly Explanation:** Real-world text data often packs multiple pieces of information into a single column—like “ERROR 2024-01-15: Connection timeout” or “john@example.com | +1-555-1234”. String extraction lets you pull out those pieces into separate, clean columns. You write a pattern that describes what you're looking for, and pandas extracts the matching parts for every row at once.

### Key Characteristics

- **First-match vs. all-matches:** `str.extract()` returns the first match per subject string; `str.extractall()` returns every match, producing multiple rows per original string.
- **Named capture groups become column names:** Using `(?P<name>...)` syntax, the extracted columns are automatically named, producing readable, self-documenting DataFrames.
- **MultiIndex from extractall:** The result of `extractall()` has a MultiIndex with the original index plus a `'match'` level that numbers each match.
- **Non-matches become NaN:** Rows that do not match the pattern are filled with `NaN` in the extracted columns.
- **Vectorized and regex-powered:** All methods use Python's `re` module under the hood and operate on entire columns at C-level speed.

### Prerequisites

- Basic Python knowledge (strings, functions, dictionaries).
- Familiarity with regular expressions (capture groups, quantifiers, character classes).
- Understanding of pandas Series and DataFrame objects.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Log Analysis:** Parsing log lines into structured fields (timestamp, level, message).
- **Data Cleaning:** Extracting emails, phone numbers, or IDs from free-text columns.
- **ETL Pipelines:** Unpacking semi-structured data (JSON-like strings, delimited lists) into tabular features.
- **NLP Preprocessing:** Extracting specific entities before tokenization and vectorization.

### Core Concepts / Features

1. Single-Row Extraction: `str.extract()`
2. Multi-Row Extraction: `str.extractall()`
3. Regex Capture Groups: Named Groups for Dynamic Column Naming
4. Unpacking Structures: Parsing Semi-Structured Variables
5. Row Expansion: `str.split()` + `explode()`


## 1. Single-Row Extraction: Isolating the First Pattern Match with `.str.extract()`

### Definitions

**Core Definition:** `Series.str.extract()` extracts capture groups from the first match of a regular expression in each subject string, returning a DataFrame with one row per original string and one column per capture group.

**Technical Definition:** `Series.str.extract(pat, flags=0, expand=True)` calls `re.search` on each element of the Series. It returns a DataFrame with one row for each subject string and one column for each capture group. If `expand=False` and the pattern has only one capture group, it returns a Series (or Index). Any capture group names in the regular expression will be used for column names; otherwise, capture group numbers (0, 1, 2, …) are used. The dtype of each result column is always `object`, even when no match is found.

**Beginner-Friendly Explanation:** `str.extract()` is like using a magnifying glass to find the first occurrence of a pattern in each string and pull out the interesting parts. You write a pattern with parentheses around the parts you want to keep, and pandas gives you a table with those parts as columns.

### Purposes

- To extract structured fields from a single column of unstructured text.
- To create multiple new columns from one text column in a single operation.
- To parse log lines, URLs, or IDs into their component parts.
- To isolate the first match when multiple matches may exist.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
Series.str.extract(pat, flags=0, expand=True)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `pat` | Regular expression pattern with at least one capture group |
| `flags` | `re` module flags (e.g., `re.IGNORECASE`); default 0 |
| `expand` | `True` (default): return DataFrame; `False`: return Series if one group |
| Returns | DataFrame with one row per subject string, one column per group |

**Syntax Rules:**

- The pattern must contain at least one capture group (parentheses).
- Named groups `(?P<name>...)` become column names; unnamed groups become integer column labels (0, 1, 2, …).
- Non-matching strings produce a row filled with `NaN`.
- When `expand=False` and the pattern has exactly one group, the result is a Series (or Index).
- When `expand=False` and the pattern has multiple groups, the result is a DataFrame.

**Constraints and Limitations:**

- `str.extract()` returns only the **first** match per string. Use `str.extractall()` to get all matches.
- The result dtype is always `object`, even for numeric extractions. Cast with `.astype()` if needed.
- Calling `.str.extract()` on a non-string Series raises `AttributeError`.

### Annotated Code Examples

**Example 1: Basic Extraction with Multiple Groups**

```python
import pandas as pd

s = pd.Series(['a1', 'b2', 'c3'])

# Extract letter and digit as separate columns
result = s.str.extract(r'([ab])(\d)')
print(result)
```

Expected output:
```
     0    1
0    a    1
1    b    2
2  NaN  NaN
```

**Why this output:** The pattern `([ab])(\d)` has two capture groups: the first matches “a” or “b”, the second matches a digit. “c3” does not match because “c” is not in `[ab]`, so its row is filled with `NaN`.

**Example 2: Named Groups for Column Names**

```python
# Named groups become column names
result_named = s.str.extract(r'(?P<letter>[ab])(?P<digit>\d)')
print(result_named)
```

Expected output:
```
  letter digit
0      a     1
1      b     2
2    NaN   NaN
```

**Why this output:** The `(?P<letter>...)` and `(?P<digit>...)` syntax names the capture groups. Pandas uses these names as column headers instead of integer indices.

**Example 3: expand=False for a Single Group**

```python
# Single group with expand=False returns a Series
single = s.str.extract(r'[ab](\d)', expand=False)
print(single)
print(type(single))
```

Expected output:
```
0      1
1      2
2    NaN
dtype: object
<class 'pandas.core.series.Series'>
```

**Why this output:** With one capture group and `expand=False`, pandas returns a Series rather than a DataFrame. The `dtype` is `object` because non-matching rows introduce `NaN`.

### Real-World Cases

- **Log parsing:** Extract timestamp, log level, and message from log lines using a single regex.
- **URL decomposition:** Extract protocol, domain, and path from URLs into separate columns.
- **ID parsing:** Extract year, month, and sequence number from structured IDs (e.g., “INV-2024-03-0042”).

### References

- pandas.Series.str.extract — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.extract.html
- pandas.Series.str.extract (stable) — https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.str.extract.html


## 2. Multi-Row Extraction: Capturing Repeated Occurrences with `.str.extractall()`

### Definitions

**Core Definition:** `Series.str.extractall()` extracts capture groups from **all** matches of a regular expression in each subject string, returning a DataFrame with one row per match and one column per capture group.

**Technical Definition:** `Series.str.extractall(pat, flags=0)` calls `re.finditer` on each element of the Series, returning a DataFrame with one row for each match. The resulting DataFrame has a MultiIndex where the first levels come from the subject Series and the last level is named `'match'`, indexing the matches within each original string. When each subject string has exactly one match, `extractall(pat).xs(0, level='match')` is equivalent to `extract(pat)`.

**Beginner-Friendly Explanation:** Sometimes a single string contains multiple pieces of the same kind of information—like multiple email addresses in one cell, or multiple error codes in one log line. `extractall()` finds every occurrence and creates a separate row for each one, while keeping track of which original row it came from.

### Purposes

- To extract all occurrences of a pattern when a string may contain multiple matches.
- To parse strings with repeated structures (e.g., multiple timestamps, multiple IDs).
- To convert a single string with embedded lists into a tidy, one-item-per-row format.
- To count or analyze repeated patterns within strings.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
Series.str.extractall(pat, flags=0)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `pat` | Regular expression pattern with at least one capture group |
| `flags` | `re` module flags (e.g., `re.IGNORECASE`); default 0 |
| Returns | DataFrame with one row per match, MultiIndex with `'match'` level |

**Syntax Rules:**

- The result has a MultiIndex: original index levels + a `'match'` level.
- Rows with no matches do **not** appear in the result (unlike `extract()`, which includes them with `NaN`).
- Named groups become column names; unnamed groups become integer labels.
- `extractall()` always returns a DataFrame (no `expand` parameter).

**Constraints and Limitations:**

- The MultiIndex may need to be reset with `.reset_index()` for further analysis.
- Empty matches produce rows with `NaN` in the optional group columns.
- Performance may be slower than `extract()` for large datasets because it finds all matches.

### Annotated Code Examples

**Example 1: Extracting All Matches**

```python
import pandas as pd

s = pd.Series(['a1a2', 'b1', 'c1'], index=['A', 'B', 'C'])

# Extract all letter-digit pairs
result = s.str.extractall(r'([ab])(\d)')
print(result)
```

Expected output:
```
        0    1
  match
A 0      a    1
  1      a    2
B 0      b    1
```

**Why this output:** “a1a2” contains two matches (`a1` and `a2`), so it produces two rows. “b1” produces one row. “c1” produces no rows because “c” is not in `[ab]`, so it does not appear in the result at all.

**Example 2: Named Groups in extractall()**

```python
# Named groups become column names
result_named = s.str.extractall(r'(?P<letter>[ab])(?P<digit>\d)')
print(result_named)
```

Expected output:
```
        letter digit
  match
A 0          a     1
  1          a     2
B 0          b     1
```

**Why this output:** The named groups `letter` and `digit` are used as column names. The MultiIndex has `A` and `B` from the original index, plus the `match` level numbering each occurrence within each string.

**Example 3: Resetting the MultiIndex**

```python
# Reset index to get a flat DataFrame
flat = result_named.reset_index()
print(flat)
```

Expected output:
```
  level_0  match letter digit
0       A      0      a     1
1       A      1      a     2
2       B      0      b     1
```

**Why this output:** `.reset_index()` converts the MultiIndex into regular columns, making the DataFrame easier to work with for downstream operations like grouping or merging.

### Real-World Cases

- **Email extraction:** Extract all email addresses from a column of text where each row may contain multiple addresses.
- **Hashtag extraction:** Extract all hashtags from social media posts.
- **Error code parsing:** Extract all error codes from a log line containing multiple errors.

### References

- pandas.Series.str.extractall — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.extractall.html
- pandas.Series.str.extractall (stable) — https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.str.extractall.html


## 3. Regex Capture Groups: Named Groups for Dynamic Column Naming

### Definitions

**Core Definition:** Named capture groups use the syntax `(?P<name>...)` to assign a name to a capture group, which pandas automatically uses as the column name in the result of `extract()` or `extractall()`.

**Technical Definition:** In Python's `re` module, a named group is defined as `(?P<name>pattern)`, where `name` is a valid Python identifier. When passed to pandas' `extract()` or `extractall()`, any capture group names in the regular expression are used for column names in the resulting DataFrame; otherwise, capture group numbers (0, 1, 2, …) are used. Named groups can be referenced in the pattern itself with `(?P=name)` for backreferences.

**Beginner-Friendly Explanation:** Instead of getting columns named “0”, “1”, “2”, you can name them “year”, “month”, “day” right in the pattern. This makes your extracted data self-documenting and easier to use.

### Purposes

- To produce self-documenting extracted columns with meaningful names.
- To avoid manual column renaming after extraction.
- To reference captured values later in the same pattern using backreferences.
- To make complex extraction pipelines more readable and maintainable.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Named group syntax in regex
r'(?P<group_name>pattern)'

# Backreference to a named group
r'(?P<name>pattern)...(?P=name)'
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `(?P<name>` | Opens a named capture group |
| `pattern` | The regex pattern to capture |
| `)` | Closes the group |
| `(?P=name)` | Backreference to the named group |

**Syntax Rules:**

- Group names must be valid Python identifiers (letters, digits, underscores; cannot start with a digit).
- Group names must be unique within a pattern.
- Named groups work identically in `extract()` and `extractall()`.
- Unnamed groups can be mixed with named groups; unnamed groups receive integer labels.

**Constraints and Limitations:**

- Group names cannot contain spaces or special characters.
- Duplicate group names raise `re.error`.
- In some pandas versions with Arrow-backed strings, unnamed capture groups may raise errors when mixed with named ones.

### Annotated Code Examples

**Example 1: Basic Named Group Extraction**

```python
import pandas as pd

s = pd.Series(['a1', 'b2', 'c3'])

# Named groups become column names
result = s.str.extract(r'(?P<letter>[ab])(?P<digit>\d)')
print(result)
# Expected output:
#   letter digit
# 0      a     1
# 1      b     2
# 2    NaN   NaN
```

**Why this output:** The `(?P<letter>...)` and `(?P<digit>...)` syntax names the capture groups. Pandas uses these names as column headers, producing a readable DataFrame.

**Example 2: Named Groups in extractall()**

```python
s2 = pd.Series(['a1a2', 'b1'], index=['X', 'Y'])
result2 = s2.str.extractall(r'(?P<letter>[ab])(?P<digit>\d)')
print(result2)
# Expected output:
#         letter digit
#   match
# X 0          a     1
#   1          a     2
# Y 0          b     1
```

**Why this output:** The named groups produce columns `letter` and `digit`. The MultiIndex includes the original index (`X`, `Y`) and the `match` level numbering each occurrence.

**Example 3: Backreferences with Named Groups**

```python
s3 = pd.Series(['abcabc', 'xyzxyz', 'hello'])
result3 = s3.str.extract(r'(?P<chunk>\w+)(?P=chunk)')
print(result3)
# Expected output:
#   chunk
# 0   abc
# 1   xyz
# 2   NaN
```

**Why this output:** The backreference `(?P=chunk)` requires the same text to appear twice in a row. “abcabc” and “xyzxyz” match; “hello” does not.

### Real-World Cases

- **Date parsing:** `(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})` extracts date components with named columns.
- **Log parsing:** `(?P<timestamp>\S+)\s+(?P<level>\w+)\s+(?P<message>.+)` extracts log fields with descriptive names.
- **Email parsing:** `(?P<user>[^@]+)@(?P<domain>.+)` separates username and domain.

### References

- pandas.Series.str.extract (Named groups) — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.extract.html
- Python re module (Named groups) — https://docs.python.org/3/library/re.html#regular-expression-syntax


## 4. Unpacking Structures: Parsing Semi-Structured Variables

### Definitions

**Core Definition:** Unpacking semi-structured variables is the process of parsing strings that contain embedded structured information—such as log lines, email strings, or phone numbers—into separate tabular columns using regex capture groups.

**Technical Definition:** Semi-structured text (e.g., `"ERROR 2024-01-15T10:30:00 Connection timeout"` or `"john@example.com | +1-555-123-4567"`) can be parsed into tabular features using a single regex pattern with multiple capture groups passed to `Series.str.extract()`. The pattern describes the expected format, and each capture group becomes a column. This technique is particularly powerful for log analysis, where each line follows a consistent but not fully structured format.

**Beginner-Friendly Explanation:** Many real-world text fields pack multiple pieces of information into one string. For example, a log line might contain a timestamp, a severity level, and a message all in one string. By writing a pattern that describes the format, you can split that one string into three separate columns.

### Purposes

- To transform log lines into structured DataFrames with columns for timestamp, level, and message.
- To extract contact information (email, phone) from free-text fields.
- To parse structured IDs (order numbers, product codes) into their component parts.
- To convert semi-structured data into a format suitable for analysis and visualization.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Log parsing pattern
log_pattern = r'(?P<timestamp>\S+)\s+(?P<level>\w+)\s+(?P<message>.+)'
df_log = log_series.str.extract(log_pattern)

# Email and phone extraction
contact_pattern = r'(?P<email>[\w.+-]+@[\w-]+\.[\w.]+).*?(?P<phone>\d{3}[-.\s]\d{3}[-.\s]\d{4})'
df_contact = text_series.str.extract(contact_pattern)
```

**Component Breakdown:**

| Pattern Element | Description |
|-----------------|-------------|
| `\S+` | One or more non-whitespace characters (timestamp) |
| `\w+` | One or more word characters (log level) |
| `.+` | One or more of any character (message) |
| `[\w.+-]+@[\w-]+\.[\w.]+` | Email address pattern |
| `\d{3}[-.\s]\d{3}[-.\s]\d{4}` | US phone number pattern |

**Syntax Rules:**

- Use named groups for clarity and automatic column naming.
- Use `.*?` (non-greedy) to match the shortest possible text between patterns.
- Use `re.IGNORECASE` to handle case variations in log levels.
- Escape special characters (e.g., `.`, `+`, `(`, `)`) with `\` when matching literal characters.

**Constraints and Limitations:**

- The pattern must match the entire relevant portion of the string; partial matches may extract incorrectly.
- Very complex patterns can be slow on large datasets.
- Different log formats require different patterns.

### Annotated Code Examples

**Example 1: Parsing Log Lines**

```python
import pandas as pd

logs = pd.Series([
    '2024-01-15 10:30:00 ERROR Connection timeout',
    '2024-01-15 10:31:00 INFO Service started',
    '2024-01-15 10:32:00 WARNING Disk usage high'
])

# Parse log lines into timestamp, level, and message
pattern = r'(?P<timestamp>\S+ \S+)\s+(?P<level>\w+)\s+(?P<message>.+)'
log_df = logs.str.extract(pattern)
print(log_df)
# Expected output:
#               timestamp    level            message
# 0  2024-01-15 10:30:00    ERROR  Connection timeout
# 1  2024-01-15 10:31:00     INFO    Service started
# 2  2024-01-15 10:32:00  WARNING   Disk usage high
```

**Why this output:** The pattern captures the timestamp (date and time separated by a space), the log level (word characters), and the message (everything remaining). Each capture group becomes a column.

**Example 2: Extracting Email and Phone from Text**

```python
texts = pd.Series([
    'Contact me at john@example.com or call 555-123-4567',
    'Reach us at support@company.org, phone: 555.987.6543'
])

# Extract email and phone number
pattern = r'(?P<email>[\w.+-]+@[\w-]+\.[\w.]+).*?(?P<phone>\d{3}[-.\s]\d{3}[-.\s]\d{4})'
contact_df = texts.str.extract(pattern)
print(contact_df)
# Expected output:
#                  email         phone
# 0    john@example.com  555-123-4567
# 1  support@company.org  555.987.6543
```

**Why this output:** The pattern uses named groups for email and phone. The `.*?` between them matches any characters non-greedily, allowing the phone number to be found after the email.

**Example 3: Parsing Structured IDs**

```python
ids = pd.Series(['INV-2024-03-0042', 'INV-2024-04-0157', 'ORD-2023-12-0001'])

# Extract components of structured IDs
pattern = r'(?P<type>[A-Z]+)-(?P<year>\d{4})-(?P<month>\d{2})-(?P<seq>\d{4})'
id_df = ids.str.extract(pattern)
print(id_df)
# Expected output:
#   type  year month   seq
# 0  INV  2024    03  0042
# 1  INV  2024    04  0157
# 2  ORD  2023    12  0001
```

**Why this output:** The pattern decomposes the ID into four named groups: type (letters), year (4 digits), month (2 digits), and sequence (4 digits). Each becomes a column.

### Real-World Cases

- **Server log analysis:** Parse Nginx/Apache access logs into IP, timestamp, method, path, and status code.
- **CRM data cleaning:** Extract email and phone from free-text contact notes.
- **Financial transaction parsing:** Decompose transaction IDs into date, branch, and sequence components.

### References

- Working with Text Data (Extracting substrings) — https://pandas.pydata.org/docs/user_guide/text.html
- Parsing log lines with str.extract — https://stackoverflow.com/questions/56228334
- Leveraging Regular Expressions in Pandas — https://github.com/LMK89/Machine-Learning-MD


## 5. Row Expansion: Transforming Delimited Lists into Rows with `.str.split()` + `.explode()`

### Definitions

**Core Definition:** Row expansion is the process of splitting a delimited string into a list of elements using `.str.split()`, then converting each element into a separate row using `.explode()`.

**Technical Definition:** `Series.str.split(pat=None, *, n=-1, expand=False)` splits each string around a delimiter, returning a Series of lists when `expand=False`. `DataFrame.explode(column, ignore_index=False)` then transforms each element of a list-like column into a row, replicating the index values and other column values for each element. This is the canonical pattern for converting a column of comma-separated (or otherwise delimited) strings into a tidy, one-item-per-row format.

**Beginner-Friendly Explanation:** Imagine a column where each cell contains a list of tags, like “python,pandas,data”. `str.split()` turns that into a list `['python', 'pandas', 'data']`, and `explode()` gives each tag its own row, duplicating the other columns as needed.

### Purposes

- To convert a column of delimited strings (e.g., comma-separated tags) into a tidy, one-item-per-row format.
- To normalize data for groupby, aggregation, and analysis.
- To expand multiple list-like columns simultaneously (multi-column explode).
- To handle semi-structured data where multiple values are packed into a single cell.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Step 1: Split the string column into a list
df['col'] = df['col'].str.split(delimiter)

# Step 2: Explode the list into rows
df_exploded = df.explode('col')

# Single-line pattern
df_exploded = df.assign(col=df['col'].str.split(delimiter)).explode('col')

# Multi-column explode (pandas 1.3+)
df_exploded = df.explode(['col1', 'col2'])
```

**Component Breakdown:**

| Method | Parameter | Description |
|--------|-----------|-------------|
| `str.split()` | `pat` | Delimiter (string or regex); default whitespace |
| | `n` | Maximum number of splits; `-1` (default) = all |
| | `expand` | `False` (default): Series of lists; `True`: DataFrame |
| `explode()` | `column` | Column label(s) to explode |
| | `ignore_index` | `True`: reset index to 0, 1, …, n−1 |

**Syntax Rules:**

- `str.split()` with `expand=False` (default) returns a Series of lists, which is required for `.explode()`.
- `.explode()` replicates the index values for each element in the list.
- Empty lists result in `NaN` for that row.
- Multi-column explode requires all specified columns to have matching list lengths per row.
- Scalars in the column are returned unchanged by `explode()`.

**Constraints and Limitations:**

- The delimiter must be consistent; mixed delimiters require a regex pattern (e.g., `r',| and '`).
- `explode()` on a set produces non-deterministic ordering.
- Multi-column explode requires matching lengths across columns.
- Empty lists become `NaN`, which may require additional cleaning.

### Annotated Code Examples

**Example 1: Single-Column Explode**

```python
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob'],
    'tags': ['python,pandas,data', 'java,spring']
})

# Step 1: Split the tags column
df['tags_list'] = df['tags'].str.split(',')
print(df)
# Expected output:
#     name                tags                tags_list
# 0  Alice  python,pandas,data  [python, pandas, data]
# 1    Bob        java,spring        [java, spring]

# Step 2: Explode into rows
df_exploded = df.explode('tags_list')
print(df_exploded)
# Expected output:
#     name                tags tags_list
# 0  Alice  python,pandas,data    python
# 0  Alice  python,pandas,data    pandas
# 0  Alice  python,pandas,data      data
# 1    Bob        java,spring      java
# 1    Bob        java,spring    spring
```

**Why this output:** `str.split(',')` converts each comma-separated string into a list. `.explode('tags_list')` creates one row per list element, replicating the `name` and original `tags` values for each new row.

**Example 2: One-Liner with assign() and explode()**

```python
# One-liner: assign split column and explode
df_exploded = df.assign(tag=df['tags'].str.split(',')).explode('tag')
print(df_exploded[['name', 'tag']])
# Expected output:
#     name     tag
# 0  Alice  python
# 0  Alice  pandas
# 0  Alice    data
# 1    Bob    java
# 1    Bob  spring
```

**Why this output:** `assign()` creates a new column `tag` with the split list, and `.explode('tag')` expands it into rows. The original `tags` column is preserved.

**Example 3: Multi-Column Explode**

```python
df2 = pd.DataFrame({
    'id': [1, 2],
    'tags': [['a', 'b'], ['c', 'd']],
    'values': [[10, 20], [30, 40]]
})

# Explode both tags and values together
df2_exploded = df2.explode(['tags', 'values'])
print(df2_exploded)
# Expected output:
#    id tags  values
# 0   1    a      10
# 0   1    b      20
# 1   2    c      30
# 1   2    d      40
```

**Why this output:** Multi-column explode expands both `tags` and `values` simultaneously, keeping the pairing intact. Each row in the original DataFrame produces two rows, one for each element in the parallel lists.

### Real-World Cases

- **Movie genres:** A movie with genres “Action, Adventure, Sci-Fi” becomes three rows, one per genre.
- **Product tags:** E-commerce products with multiple tags are expanded for tag-based analysis.
- **Survey responses:** Multiple-choice responses stored as comma-separated strings are expanded for frequency counting.

### References

- pandas.DataFrame.explode — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.explode.html
- pandas.Series.str.split — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.split.html
- Reshaping and pivot tables (explode) — https://pandas.pydata.org/docs/user_guide/reshaping.html
- Pandas String Operations: Vectorized Text Cleaning — https://docs.kanaries.net/topics/Pandas/pandas-string-operations


## References

- pandas.Series.str.extract — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.extract.html
- pandas.Series.str.extract (stable) — https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.str.extract.html
- pandas.Series.str.extractall — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.extractall.html
- pandas.Series.str.extractall (stable) — https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.str.extractall.html
- pandas.DataFrame.explode — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.explode.html
- pandas.Series.str.split — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.split.html
- Working with Text Data — https://pandas.pydata.org/docs/user_guide/text.html
- Reshaping and pivot tables (explode) — https://pandas.pydata.org/docs/user_guide/reshaping.html
- Python re module — https://docs.python.org/3/library/re.html
- Leveraging Regular Expressions in Pandas — https://github.com/LMK89/Machine-Learning-MD
- Pandas String Operations: Vectorized Text Cleaning — https://docs.kanaries.net/topics/Pandas/pandas-string-operations
- Parsing log lines with str.extract — https://stackoverflow.com/questions/56228334
- Extracting data from a log file into a Pandas DataFrame — https://stackoverflow.com/questions/76696097