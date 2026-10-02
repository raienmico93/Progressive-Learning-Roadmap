# Pandas Replacing Values: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Replacing values in pandas is the process of substituting existing data elements in a Series or DataFrame with new values, either through exact matching, dictionary-based translation, conditional masking, or regular expression pattern matching.

**Technical Definition:** Pandas provides four primary mechanisms for value replacement: `DataFrame.replace()` / `Series.replace()` for exact scalar or collection matching, dictionary-based `to_replace` mappings for structural translation, `DataFrame.where()` and `DataFrame.mask()` for conditional replacement governed by boolean masks, and regular expression substitution via `regex=True` in `replace()` or `Series.str.replace()` for pattern-based text cleanup. These methods operate on the block manager and return new objects under pandas 3.0's Copy-on-Write semantics, with `inplace` deprecated and scheduled for removal.

**Beginner-Friendly Explanation:** Sometimes your data contains values you want to change—like fixing typos, translating codes to labels, or replacing negative numbers with zero. Pandas gives you several ways to do this: you can replace exact values, use a translation table, replace values based on a condition, or use pattern matching for text cleanup.

### Key Characteristics

- **Four complementary approaches:** Exact matching (`replace()`), dictionary translation (`replace()` with dict), conditional masking (`where()`/`mask()`), and regex patterns (`str.replace()` / `replace(regex=True)`).
- **Immutable by default:** All replacement methods return new objects; `inplace=True` is deprecated in pandas 3.0.
- **Nested dictionaries for column-specific replacement:** For DataFrames, `replace()` accepts nested dictionaries where outer keys are column names and inner dictionaries specify value mappings.
- **`where()` vs. `mask()`:** `where()` keeps values where the condition is `True` and replaces where `False`; `mask()` does the inverse.
- **Regex requires strings:** Regular expression substitution only works on string values; numeric columns must be cast to string first.

### Prerequisites

- Basic Python knowledge (dictionaries, functions, regular expressions).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of boolean masking and `.loc` indexing.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Cleaning:** Fixing typos, standardizing categories, and handling placeholder values.
- **Data Transformation:** Encoding categorical variables, scaling numerical values, and normalizing text.
- **Feature Engineering:** Creating binary indicator columns and conditional labels.
- **ETL Pipelines:** Applying business rules and translation tables during data processing.

### Core Concepts / Features

1. Exact Matching: Scalar and Collection Replacement
2. Structural Translation: Dictionary-Based Lookups
3. Selective Criteria Masking: `where()` and `mask()`
4. Pattern Matching: Regular Expression Replacement


## 1. Exact Matching: Replacing Scalar Coordinates or Collections

### Definitions

**Core Definition:** Exact matching replacement substitutes specific scalar values or collections of values with new values, using `DataFrame.replace()` or `Series.replace()` with `to_replace` and `value` parameters.

**Technical Definition:** `DataFrame.replace(to_replace=None, value=<no_default>, *, inplace=False, limit=None, regex=False, method=<no_default>)` replaces values in `to_replace` with `value`. When `to_replace` is a scalar (numeric, string, or regex), all values equal to `to_replace` are replaced with `value`. When `to_replace` is a list, all values in the list are replaced with the single scalar `value`. When both `to_replace` and `value` are lists, they must be the same length, and each element of `to_replace` is replaced with the corresponding element of `value`.

**Beginner-Friendly Explanation:** `replace()` is the “find and replace” tool. You can replace a single value (like all `-999` with `NaN`), or replace a list of values (like `['a', 'b', 'c']` all with `'unknown'`).

### Purposes

- To replace a single scalar value throughout a Series or DataFrame with a new value.
- To replace a list of values with a single replacement value.
- To replace multiple values with multiple corresponding values using paired lists.
- To handle sentinel values (e.g., `-999`, `'N/A'`, `'missing'`) by replacing them with `NaN`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Scalar replacement
df.replace(to_replace, value)

# List replacement (single value)
df.replace([val1, val2, val3], new_value)

# Paired list replacement
df.replace([old1, old2], [new1, new2])

# Regex flag
df.replace(to_replace=pattern, value=replacement, regex=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `to_replace` | Scalar, list, dict, Series, or regex pattern to find |
| `value` | Scalar, dict, list, or regex to use as replacement |
| `regex` | If `True`, interpret `to_replace` and/or `value` as regex |
| `method` | Deprecated; use `ffill()` or `bfill()` instead |
| `limit` | Deprecated; maximum size gap for forward/backward fill |

**Syntax Rules:**

- When `to_replace` is a scalar and `value` is a scalar, all matching values are replaced.
- When `to_replace` is a list and `value` is a scalar, all values in the list are replaced with the scalar.
- When both are lists, they must be the same length.
- When `to_replace` is a dict, the `value` parameter must not be given.
- `regex=True` requires `to_replace` to be a string or compiled regex.

**Constraints and Limitations:**

- **Deprecated:** The `method` and `limit` parameters are deprecated since pandas 2.1.0; use `ffill()` or `bfill()` instead.
- **Deprecated:** The `inplace` parameter is deprecated in pandas 3.0; use reassignment.
- Regex substitution only works on strings; numeric columns must be cast to string first.
- Replacing with a different dtype may upcast the column (e.g., `int` → `float` if replacing with `NaN`).

### Annotated Code Examples

**Example 1: Scalar and List Replacement**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'A': [0, 1, 2, 3, 4],
    'B': [5, 6, 7, 8, 9],
    'C': ['a', 'b', 'c', 'd', 'e']
})

# Replace scalar 0 with 100
print(df.replace(0, 100))
# Expected output:
#      A  B  C
# 0  100  5  a
# 1    1  6  b
# 2    2  7  c
# 3    3  8  d
# 4    4  9  e

# Replace list [0, 1, 2] with 100
print(df.replace([0, 1, 2], 100))
# Expected output:
#      A  B  C
# 0  100  5  a
# 1  100  6  b
# 2  100  7  c
# 3    3  8  d
# 4    4  9  e
```

**Why this output:** `replace(0, 100)` finds all occurrences of 0 and replaces them with 100. `replace([0, 1, 2], 100)` replaces any value in the list with 100.

**Example 2: Paired List Replacement**

```python
# Replace multiple values with multiple values
print(df.replace([0, 1, 2], [100, 200, 300]))
# Expected output:
#      A  B  C
# 0  100  5  a
# 1  200  6  b
# 2  300  7  c
# 3    3  8  d
# 4    4  9  e
```

**Why this output:** The paired lists map 0→100, 1→200, and 2→300. Values not in the `to_replace` list are left unchanged.

**Example 3: Replacing Sentinel Values with NaN**

```python
df_sentinel = pd.DataFrame({
    'value': [1, 2, -999, 4, -999],
    'code': ['A', 'B', 'N/A', 'C', 'missing']
})

# Replace sentinel values with NaN
df_clean = df_sentinel.replace([-999, 'N/A', 'missing'], np.nan)
print(df_clean)
# Expected output:
#    value code
# 0    1.0    A
# 1    2.0    B
# 2    NaN  NaN
# 3    4.0    C
# 4    NaN  NaN
```

**Why this output:** All three sentinel values are replaced with `NaN`. The `value` column is upcast to `float64` to accommodate the `NaN`.

### Real-World Cases

- **Survey data:** Replace `-999` (a common “no response” code) with `NaN` for proper missing value handling.
- **Database exports:** Replace `'N/A'`, `'null'`, and `'-'` placeholder strings with `NaN`.
- **Financial data:** Replace outlier sentinel values (e.g., `-1` for missing prices) with `NaN` before analysis.

### References

- pandas.DataFrame.replace — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.replace.html
- pandas.Series.replace — https://pandas.pydata.org/pandas-docs/version/2.1.4/reference/api/pandas.Series.replace.html


## 2. Structural Translation: Dictionary-Based Lookups

### Definitions

**Core Definition:** Dictionary-based replacement uses a dictionary to map existing values to new values, enabling structural translation where different original values are replaced with different new values.

**Technical Definition:** When `to_replace` is a dictionary, keys are the values to be replaced and values are the replacements. For a DataFrame, a dictionary can specify different replacements for different columns. Nested dictionaries, e.g., `{'column_A': {'old_val': 'new_val'}}`, look in column `'column_A'` for `'old_val'` and replace it. Column names (top-level dictionary keys) cannot be regular expressions.

**Beginner-Friendly Explanation:** A dictionary is like a translation table. You say “replace 'cat' with 'kitten' and 'dog' with 'puppy'.” For DataFrames, you can even specify different translation tables for different columns.

### Purposes

- To translate categorical codes to human-readable labels (e.g., `1` → `'Active'`, `2` → `'Inactive'`).
- To apply different replacement rules to different columns using nested dictionaries.
- To perform systematic value mapping without multiple `replace()` calls.
- To standardise values across multiple columns using a shared mapping.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Simple dictionary (same mapping for all columns)
df.replace({'old1': 'new1', 'old2': 'new2'})

# Nested dictionary (column-specific mappings)
df.replace({'col_A': {'old1': 'new1'}, 'col_B': {'old2': 'new2'}})

# With regex in nested dict
df.replace({'col_A': {r'^pattern': 'new'}}, regex=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{'old': 'new'}` | Simple mapping applied to all columns |
| `{'col': {'old': 'new'}}` | Column-specific mapping |
| `{'col': {r'regex': 'new'}}` | Regex pattern within column-specific mapping |
| `regex=True` | Required when using regex patterns in dictionaries |

**Syntax Rules:**

- When `to_replace` is a dict, the `value` parameter must not be given.
- For simple dictionaries, the mapping applies to all columns.
- For nested dictionaries, outer keys are column names and inner dictionaries contain value mappings.
- Column names in nested dictionaries cannot be regex patterns.
- Values in the dictionary must match the dtype of the column for exact matching.

**Constraints and Limitations:**

- Values not in the dictionary are left unchanged (unlike `Series.map()`, which converts unmapped values to `NaN`).
- Dictionary keys must be hashable; lists and dicts cannot be used as keys.
- Nested dictionaries require the outer key to match an existing column name.

### Annotated Code Examples

**Example 1: Simple Dictionary Mapping**

```python
import pandas as pd

df = pd.DataFrame({
    'status': ['active', 'inactive', 'pending', 'active'],
    'priority': ['low', 'high', 'medium', 'low']
})

# Simple dictionary mapping
df_mapped = df.replace({'active': 'A', 'inactive': 'I', 'pending': 'P'})
print(df_mapped)
# Expected output:
#   status priority
# 0      A      low
# 1      I     high
# 2      P   medium
# 3      A      low
```

**Why this output:** The dictionary maps `'active'` → `'A'`, `'inactive'` → `'I'`, and `'pending'` → `'P'`. The `priority` column is unaffected because its values are not in the dictionary.

**Example 2: Nested Dictionary for Column-Specific Mapping**

```python
df2 = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6]
})

# Nested dictionary: different mappings per column
df2_mapped = df2.replace({'A': {1: 100, 2: 200}, 'B': {4: 400}})
print(df2_mapped)
# Expected output:
#      A    B
# 0  100  400
# 1  200    5
# 2    3    6
```

**Why this output:** In column `'A'`, 1 becomes 100 and 2 becomes 200. In column `'B'`, 4 becomes 400. The value 3 in column A and 5, 6 in column B are not in their respective inner dictionaries, so they remain unchanged.

**Example 3: Nested Dictionary with Regex**

```python
df3 = pd.DataFrame({
    'name': ['Alice Smith', 'Bob Jones', 'Charlie Brown'],
    'city': ['New York', 'Los Angeles', 'New Orleans']
})

# Replace 'New' with 'Old' only in the 'city' column
df3_mapped = df3.replace({'city': {r'^New': 'Old'}}, regex=True)
print(df3_mapped)
# Expected output:
#             name         city
# 0    Alice Smith     Old York
# 1      Bob Jones  Los Angeles
# 2  Charlie Brown    Old Orleans
```

**Why this output:** The regex pattern `r'^New'` matches strings starting with `'New'` in the `city` column. Both `'New York'` and `'New Orleans'` are affected, while `'Los Angeles'` is unchanged. The `name` column is unaffected because the mapping is column-specific.

### Real-World Cases

- **Data encoding:** Map numeric status codes to descriptive labels (`1` → `'Active'`, `2` → `'Inactive'`).
- **Multi-column standardization:** Apply different mapping rules to different columns in a single `replace()` call.
- **Category consolidation:** Merge similar categories (e.g., `'NYC'` and `'New York'` → `'New York'`).

### References

- pandas.DataFrame.replace — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.replace.html
- pandas.Series.replace — https://pandas.pydata.org/pandas-docs/version/2.1.4/reference/api/pandas.Series.replace.html


## 3. Selective Criteria Masking: Conditional Replacement with where() and mask()

### Definitions

**Core Definition:** `DataFrame.where()` and `DataFrame.mask()` perform conditional replacement by keeping values where a boolean condition is met (or not met) and replacing the rest with a specified value.

**Technical Definition:** `DataFrame.where(cond, other=nan, *, inplace=False, axis=None, level=None)` returns an object of the same shape where entries from the original are kept where `cond` is `True` and replaced with `other` where `cond` is `False`. `DataFrame.mask(cond, other=nan, *, inplace=False, axis=None, level=None)` is the inverse: it replaces entries where `cond` is `True` and keeps the original where `cond` is `False`. Both methods accept callables for `cond` and `other`, and can operate on DataFrames, Series, or arrays.

**Beginner-Friendly Explanation:** `where()` is like saying “keep the values that meet this condition, and replace the rest.” `mask()` is the opposite: “replace the values that meet this condition.” For example, `df.where(df > 0, 0)` replaces all negative values with 0.

### Purposes

- To replace values that fail a condition while keeping values that satisfy it (`where()`).
- To replace values that satisfy a condition while keeping the rest (`mask()`).
- To clip values to a range (e.g., replace values above a threshold with the threshold value).
- To apply conditional logic without using `apply()` or `np.where()`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# where(): keep where True, replace where False
df.where(cond, other=replacement_value)

# mask(): replace where True, keep where False
df.mask(cond, other=replacement_value)

# Using callables
df.where(lambda x: x > 0, lambda x: x * 2)

# In-place
df.where(cond, other, inplace=True)  # deprecated in pandas 3.0
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `cond` | Boolean Series/DataFrame, array, or callable returning boolean |
| `other` | Scalar, Series/DataFrame, or callable for replacement values |
| `axis` | Alignment axis if needed |
| `level` | Alignment level if needed |

**Syntax Rules:**

- `where()` keeps values where `cond` is `True` and replaces where `cond` is `False`.
- `mask()` keeps values where `cond` is `False` and replaces where `cond` is `True`.
- Both methods can accept callables for `cond` and `other`.
- `other` can be a scalar, Series, or DataFrame; alignment follows index/column labels.
- `inplace=True` is deprecated in pandas 3.0.

**Constraints and Limitations:**

- `where()` and `mask()` return a new object; they do not modify in place unless `inplace=True`.
- The `other` parameter's alignment with the original object follows label-based alignment.
- For simple binary conditions, `np.where()` is often faster and more concise.

### Annotated Code Examples

**Example 1: Basic where() and mask()**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame(np.arange(10).reshape(-1, 2), columns=['A', 'B'])
print(df)
# Expected output:
#    A  B
# 0  0  1
# 1  2  3
# 2  4  5
# 3  6  7
# 4  8  9

# where(): keep values > 4, replace others with -1
print(df.where(df > 4, -1))
# Expected output:
#    A  B
# 0 -1 -1
# 1 -1 -1
# 2 -1  5
# 3  6  7
# 4  8  9

# mask(): replace values > 4 with -1, keep others
print(df.mask(df > 4, -1))
# Expected output:
#    A  B
# 0  0  1
# 1  2  3
# 2  4 -1
# 3 -1 -1
# 4 -1 -1
```

**Why this output:** `where(df > 4, -1)` keeps values greater than 4 and replaces all others with -1. `mask(df > 4, -1)` does the inverse: replaces values greater than 4 with -1 and keeps the rest.

**Example 2: Clipping Values with where()**

```python
# Clip values: replace anything above 5 with 5
df_clipped = df.where(df <= 5, 5)
print(df_clipped)
# Expected output:
#    A  B
# 0  0  1
# 1  2  3
# 2  4  5
# 3  5  5
# 4  5  5
```

**Why this output:** `df.where(df <= 5, 5)` keeps values that are 5 or less, and replaces values greater than 5 with 5. This is an upper-bound clipping operation.

**Example 3: Using Callables with where()**

```python
# Using callables for condition and replacement
df_doubled = df.where(lambda x: x > 4, lambda x: x * 2)
print(df_doubled)
# Expected output:
#     A   B
# 0   0   2
# 1   4   6
# 2   8  10
# 3   6   7
# 4   8   9
```

**Why this output:** The `cond` callable keeps values greater than 4. The `other` callable multiplies the replacement values by 2. For row 0: A=0 (not > 4) becomes 0*2=0; B=1 (not > 4) becomes 1*2=2. For row 2: A=4 (not > 4) becomes 4*2=8; B=5 (> 4) is kept as 5.

### Real-World Cases

- **Outlier handling:** Replace values above the 99th percentile with the percentile value using `where()`.
- **Missing value imputation:** Replace `NaN` values with the column mean using `mask(df.isna(), df.mean())`.
- **Conditional scaling:** Apply different scaling factors to values above and below a threshold.

### References

- pandas.DataFrame.where — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.where.html
- pandas.DataFrame.mask — https://pandas.pydata.org/pandas-docs/version/1.0.2/reference/api/pandas.DataFrame.mask.html
- Indexing and selecting data (where and mask) — https://pandas.pydata.org/docs/user_guide/indexing.html#the-where-method-and-masking


## 4. Pattern Matching: Regular Expression Replacement

### Definitions

**Core Definition:** Regular expression (regex) replacement uses pattern matching to find and replace text that matches a specified pattern, enabling complex text cleanup operations that go beyond exact matching.

**Technical Definition:** Regex replacement in pandas is available through two mechanisms: `DataFrame.replace()` / `Series.replace()` with `regex=True`, which compiles the `to_replace` pattern as a regular expression and substitutes matches using `re.sub()`; and `Series.str.replace(pat, repl, regex=True)`, which provides a dedicated string method for pattern-based replacement. In `replace()`, regular expressions will only substitute on strings, meaning you cannot provide a regex matching floating-point numbers and expect it to match numeric columns. In `Series.str.replace()`, the `regex` parameter defaults to `True` in pandas 2.0+.

**Beginner-Friendly Explanation:** Regex replacement is for text cleanup. You can use patterns like `\s+` (one or more spaces) or `[^a-z0-9]` (any character that is not a lowercase letter or digit) to find and replace text systematically.

### Purposes

- To remove or replace whitespace, punctuation, and special characters in text columns.
- To extract and reformat patterns (e.g., phone numbers, dates, IDs).
- To standardise text across a column using pattern-based rules.
- To clean messy text data that cannot be handled by exact matching.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Using replace() with regex
df.replace(to_replace=r'pattern', value='replacement', regex=True)

# Using str.replace() on a Series
series.str.replace(pat=r'pattern', repl='replacement', regex=True)

# Dictionary of regex patterns with replace()
df.replace({r'pattern': 'replacement'}, regex=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `to_replace` / `pat` | Regex pattern string or compiled regex |
| `value` / `repl` | Replacement string (can use backreferences like `\1`) |
| `regex` | If `True`, interpret pattern as regex |
| `n` | Maximum number of replacements (in `str.replace`) |
| `case` | Case sensitivity (in `str.replace`) |
| `flags` | Regex flags (in `str.replace`) |

**Syntax Rules:**

- `regex=True` is required in `replace()` when using regex patterns; `str.replace()` defaults to `regex=True` in pandas 2.0+.
- Backreferences like `\1`, `\2` can reference captured groups in the pattern.
- `NaN` values are left unchanged by `str.replace()`.
- Regex patterns should be raw strings (`r'pattern'`) to avoid escape sequence issues.
- Use `re.sub()` syntax for replacement patterns.

**Constraints and Limitations:**

- **Version-specific:** In pandas 1.x, `str.replace()` defaulted to `regex=True`; in pandas 2.0+, the default is still `True` but will change to `False` in a future version.
- Regex substitution only works on strings; numeric columns must be cast to string first.
- Complex regex patterns can be slow on large datasets.
- The `case` and `flags` parameters are available in `str.replace()` but not in `replace()`.

### Annotated Code Examples

**Example 1: Removing Whitespace with str.replace()**

```python
import pandas as pd

s = pd.Series(['  hello  ', ' world ', '  pandas  '])

# Strip whitespace using regex
s_clean = s.str.replace(r'^\s+|\s+$', '', regex=True)
print(s_clean)
# Expected output:
# 0     hello
# 1     world
# 2    pandas
# dtype: object
```

**Why this output:** The pattern `r'^\s+|\s+$'` matches one or more whitespace characters at the start (`^`) or end (`$`) of the string. All leading and trailing whitespace is removed.

**Example 2: Replacing Patterns with Backreferences**

```python
s2 = pd.Series(['John Smith', 'Jane Doe', 'Bob Johnson'])

# Swap first and last names using backreferences
s_swapped = s2.str.replace(r'^(\w+)\s+(\w+)$', r'\2, \1', regex=True)
print(s_swapped)
# Expected output:
# 0    Smith, John
# 1      Doe, Jane
# 2  Johnson, Bob
# dtype: object
```

**Why this output:** The pattern captures the first name (`\w+`) and last name (`\w+`) as groups. The replacement `r'\2, \1'` reverses the order, producing “Last, First”.

**Example 3: Using replace() with regex on a DataFrame**

```python
df = pd.DataFrame({
    'text': ['abc123', 'def456', 'ghi789'],
    'code': ['A-1', 'B-2', 'C-3']
})

# Remove digits from the 'text' column
df_clean = df.replace({'text': {r'\d+': ''}}, regex=True)
print(df_clean)
# Expected output:
#   text code
# 0  abc  A-1
# 1  def  B-2
# 2  ghi  C-3
```

**Why this output:** The nested dictionary `{'text': {r'\d+': ''}}` targets only the `text` column. The regex `\d+` matches one or more digits, replacing them with an empty string. The `code` column is unaffected.

### Real-World Cases

- **Text cleaning:** Remove punctuation, HTML tags, or special characters from scraped text data.
- **Phone number formatting:** Extract digits from formatted phone numbers using `r'\D'` (non-digit characters).
- **Data standardisation:** Convert dates from one format to another using regex capture groups.
- **ID extraction:** Extract numeric IDs from alphanumeric strings using `r'[A-Z]+(\d+)'`.

### References

- pandas.Series.str.replace — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.replace.html
- pandas.DataFrame.replace (regex parameter) — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.replace.html
- Replacing via String Expression — https://pandas.pydata.org/docs/user_guide/text.html#replacing-via-string-expression


## References

- pandas.DataFrame.replace — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.replace.html
- pandas.Series.replace — https://pandas.pydata.org/pandas-docs/version/2.1.4/reference/api/pandas.Series.replace.html
- pandas.DataFrame.where — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.where.html
- pandas.DataFrame.mask — https://pandas.pydata.org/pandas-docs/version/1.0.2/reference/api/pandas.DataFrame.mask.html
- pandas.Series.str.replace — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.replace.html
- Indexing and selecting data (where and mask) — https://pandas.pydata.org/docs/user_guide/indexing.html#the-where-method-and-masking
- Replacing via String Expression — https://pandas.pydata.org/docs/user_guide/text.html#replacing-via-string-expression
- Compile-N-Run: Pandas Value Replacement — https://github.com/Compile-N-Run/Compile-N-Run