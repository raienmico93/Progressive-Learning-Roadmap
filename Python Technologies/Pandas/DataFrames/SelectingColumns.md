# Comprehensive Programming Cheat Sheet: Selecting & Manipulating Columns in Pandas DataFrames

---

## Topic Overview

### Definitions

**Core Definition:** Selecting and manipulating columns in a pandas DataFrame refers to the set of operations for extracting, filtering, renaming, adding, updating, and dropping columns, using bracket notation, attribute-style access, dynamic filtering functions, and explicit assignment methods.

**Technical Definition:** Column selection in pandas is performed through `DataFrame.__getitem__()` (bracket notation), attribute access (`DataFrame.__getattr__()`), and dynamic filtering methods like `select_dtypes()`. Column manipulation includes assignment via `df['new'] = values`, renaming via `df.rename(columns={...})`, dropping via `df.drop(columns=[...])`, and updating via `.loc` or direct assignment. Under pandas' Copy-on-Write (CoW) mechanism (default in pandas 3.0), column selection returns a Series that shares data until modified, at which point a copy is triggered—eliminating `SettingWithCopyWarning`.

**Beginner-Friendly Explanation:** Columns are the vertical features of your DataFrame. You can grab a single column, several columns at once, or filter columns by their data type. You can also add new columns, rename them, drop them, or update their values. Using the right syntax prevents bugs and keeps your code reliable.

### Key Characteristics

- **Bracket notation is the primary, safest access method:** `df['col']` returns a Series; `df[['col']]` returns a DataFrame.
- **Attribute access is a shortcut with caveats:** `df.col` works only if the name doesn't conflict with DataFrame methods or contain spaces.
- **Dynamic filtering is programmatic:** `select_dtypes()` filters columns by type; list comprehensions filter by name patterns.
- **Column order is controllable:** Passing a list of column names selects and reorders them.
- **Copy-on-Write eliminates SettingWithCopyWarning:** Modifications to selections never silently affect the original.
- **Assignment aligns by index:** Assigning a Series to a column aligns by index labels.

### Prerequisites

- Basic Python syntax (lists, dictionaries, functions).
- Pandas Series and DataFrame construction.
- Understanding of DataFrame indexing (`.loc`, `.iloc`).
- Familiarity with data types (`.dtypes`).

### Related Programming Areas

- **Feature Engineering:** Creating and transforming columns.
- **Data Cleaning:** Renaming, dropping, and updating columns.
- **Exploratory Data Analysis:** Subsetting columns for focused analysis.
- **Machine Learning:** Selecting feature matrices.
- **ETL Pipelines:** Schema transformation and column management.

### Core Concepts / Features

1. Single-Column Selection
2. Multiple-Column Selection & Column Lists
3. Dynamic Column Selection
4. Attribute-Style Access
5. Why Bracket Notation Is Safer
6. Column Alteration & Assignment

---

## Core Concept 1: Single-Column Selection

### Definitions

**Core Definition:** Single-column selection extracts one column from a DataFrame either as a Series using `df['col']` or as a single-column DataFrame using `df[['col']]`.

**Technical Definition:** `DataFrame.__getitem__('col')` returns a Series when the key is a scalar string matching a column label. When the key is a list containing one string, `df[['col']]`, it returns a DataFrame with that single column. The distinction matters for downstream operations: Series have a single dtype and support Series methods, while single-column DataFrames retain DataFrame methods like `.merge()` and `.assign()`.

**Beginner-Friendly Explanation:** You can pull out one column in two ways. `df['col']` gives you a Series (a single column of data). `df[['col']]` gives you a DataFrame with just that one column. The double brackets make it a table instead of a list.

### Purposes

- To extract a single feature for independent analysis.
- To obtain a Series for Series-specific operations.
- To obtain a single-column DataFrame for DataFrame-specific operations.
- To pass a column to a function expecting a Series or DataFrame.
- To inspect column contents quickly.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Series output
series = df['column_name']

# Single-column DataFrame output
df_subset = df[['column_name']]
```

#### Component Breakdown

| Syntax | Output Type | Use Case |
|--------|-------------|----------|
| `df['col']` | Series | Series methods, single-column math |
| `df[['col']]` | DataFrame | DataFrame methods, multi-column pipelines |

#### Syntax Rules

- `df['col']` requires `'col'` to be a column label.
- `df[['col']]` requires a list, even with one element.
- Missing column labels raise `KeyError`.
- Under pandas 3.0, `df['str']` returns a Series; `df[['str']]` returns a DataFrame.

#### Constraints and Limitations

- `df['col']` returns a Series and loses DataFrame-level methods (`.merge()`, `.assign()`).
- The returned Series shares memory with the DataFrame under CoW until modified.
- Non-unique column labels may return a DataFrame instead of a Series.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Series vs. Single-Column DataFrame

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['NYC', 'LA', 'Chicago']
})

# Step 3: Extract as Series
name_series = df['name']
print(f"Type: {type(name_series).__name__}")
print(name_series)

# Step 4: Extract as DataFrame
name_df = df[['name']]
print(f"\nType: {type(name_df).__name__}")
print(name_df)
```

**Expected Output:**
```
Type: Series
0      Alice
1        Bob
2    Charlie
Name: name, dtype: object

Type: DataFrame
      name
0    Alice
1      Bob
2  Charlie
```

**Why this output:** `df['name']` returned a Series (single column of data). `df[['name']` returned a DataFrame with one column. The Series has a `Name` attribute; the DataFrame has column headers.

#### Example 2: Applying Series-Specific Methods

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'score': [85, 92, 78, 95]})

# Step 3: Extract as Series and use Series method
scores = df['score']
print(f"Mean: {scores.mean()}")
print(f"Max: {scores.max()}")

# Step 4: Extract as DataFrame
scores_df = df[['score']]
print(f"\nDataFrame shape: {scores_df.shape}")
```

**Expected Output:**
```
Mean: 87.5
Max: 95

DataFrame shape: (4, 1)
```

**Why this output:** The Series supported `.mean()` and `.max()` directly. The single-column DataFrame retained 2-D shape `(4, 1)`.

#### Example 3: KeyError for Missing Column

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})

# Step 3: Attempt to select a missing column
try:
    df['C']
except KeyError as e:
    print(f"KeyError: {e}")
```

**Expected Output:**
```
KeyError: 'C'
```

**Why this output:** `'C'` is not a column label in the DataFrame, so `df['C']` raised `KeyError`.

### Real-World Cases with Explanation

**Case 1: Target Vector Extraction** — A machine learning engineer extracts the target column as a Series (`y = df['target']`) and the feature matrix as a DataFrame (`X = df[['f1', 'f2', 'f3']]`).

**Case 2: Time Series Analysis** — A financial analyst extracts a single price column as a Series to apply `.pct_change()` and `.rolling()`.

**Case 3: Subset Reporting** — A business analyst extracts a single-column DataFrame for Excel export, preserving DataFrame structure.

### References

- pandas.DataFrame.__getitem__ — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.__getitem__.html
- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html

---

## Core Concept 2: Multiple-Column Selection & Column Lists

### Definitions

**Core Definition:** Multiple-column selection extracts a subset of columns from a DataFrame by passing a list of column labels to `df[['A', 'B']]`, returning a DataFrame with the selected columns in the specified order.

**Technical Definition:** When `DataFrame.__getitem__()` receives a list of column labels, it returns a new DataFrame containing only those columns, in the order specified in the list. Column order in the output follows the order in the input list, not the original DataFrame order, enabling reordering. Missing labels raise `KeyError`. Under CoW, the returned DataFrame shares data with the original until modified.

**Beginner-Friendly Explanation:** You can grab several columns at once by putting their names in a list inside double brackets. The result is a new table with just those columns, in the order you listed them. This lets you both select and reorder columns in one step.

### Purposes

- To isolate a subset of features for focused analysis.
- To reorder columns by specifying a custom order.
- To prepare feature matrices for machine learning.
- To exclude irrelevant columns from a pipeline.
- To create a new DataFrame view with a controlled layout.

### Syntax Rules and Structure

#### Complete General Syntax

```python
df[['col1', 'col2', 'col3']]

# Reordering columns
df[['col3', 'col1', 'col2']]

# Selecting columns with a list variable
cols = ['col1', 'col2']
df[cols]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| List of strings | Column labels to select | Yes |
| Order in list | Output column order | — |

#### Syntax Rules

- Input must be a list (or list-like) of column labels.
- Output column order matches the input list order.
- Missing labels raise `KeyError`.
- Duplicate labels in the list return duplicate columns.

#### Constraints and Limitations

- Selecting many columns may consume memory proportional to the subset size.
- Non-unique column labels in the original DataFrame can produce unexpected results.
- Column order changes do not affect the original DataFrame unless reassigned.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic Multiple-Column Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob'],
    'age': [25, 30],
    'city': ['NYC', 'LA'],
    'salary': [50000, 60000]
})

# Step 3: Select two columns
subset = df[['name', 'salary']]
print(subset)
```

**Expected Output:**
```
    name  salary
0  Alice   50000
1    Bob   60000
```

**Why this output:** The list `['name', 'salary']` selected only those two columns, in that order.

#### Example 2: Reordering Columns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2], 'B': [3, 4], 'C': [5, 6]})

# Step 3: Reorder columns
reordered = df[['C', 'A', 'B']]
print(reordered)
```

**Expected Output:**
```
   C  A  B
0  5  1  3
1  6  2  4
```

**Why this output:** The list `['C', 'A', 'B']` specified the output order. The original DataFrame was unchanged.

#### Example 3: Dynamic Column List

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'f1': [1, 2], 'f2': [3, 4], 'f3': [5, 6], 'target': [0, 1]})

# Step 3: Build feature list dynamically
feature_cols = [c for c in df.columns if c.startswith('f')]
print(f"Feature columns: {feature_cols}")

# Step 4: Select features
X = df[feature_cols]
print(f"\nFeature matrix:\n{X}")
```

**Expected Output:**
```
Feature columns: ['f1', 'f2', 'f3']

Feature matrix:
   f1  f2  f3
0   1   3   5
1   2   4   6
```

**Why this output:** The list comprehension dynamically identified columns starting with `'f'`, excluding `'target'`. The resulting list was used to select the feature subset.

### Real-World Cases with Explanation

**Case 1: Feature Matrix Construction** — A data scientist selects a subset of feature columns for model training while excluding the target and ID columns.

**Case 2: Report Layout** — A business analyst reorders columns for a report so that key metrics appear first.

**Case 3: Pipeline Subsetting** — A data engineer selects only the columns needed for a downstream transformation, reducing memory usage.

### References

- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html
- pandas.DataFrame.__getitem__ — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.__getitem__.html

---

## Core Concept 3: Dynamic Column Selection

### Definitions

**Core Definition:** Dynamic column selection filters DataFrame columns programmatically based on properties such as data type, name patterns, or other metadata, using methods like `df.select_dtypes()` or list comprehensions.

**Technical Definition:** `DataFrame.select_dtypes(include=None, exclude=None)` returns a subset of columns matching the specified dtype criteria. `include` and `exclude` accept dtype strings (e.g., `'number'`, `'object'`, `'int64'`) or lists thereof. List comprehensions over `df.columns` enable name-pattern filtering (e.g., `[c for c in df.columns if c.startswith('f')]`). These techniques support programmatic schema manipulation in pipelines.

**Beginner-Friendly Explanation:** Instead of manually listing column names, you can filter columns by their properties. For example, `select_dtypes(include=['number'])` grabs all numeric columns. List comprehensions let you pick columns whose names match a pattern. This is useful when you don't know the exact column names or when the schema changes.

### Purposes

- To select columns by data type (e.g., only numeric for ML).
- To filter columns by name patterns (prefixes, suffixes, regex).
- To build dynamic pipelines that adapt to schema changes.
- To exclude columns of a specific type (e.g., object) from analysis.
- To identify columns needing conversion.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# By dtype
df.select_dtypes(include=['number'])
df.select_dtypes(include=['int64', 'float64'])
df.select_dtypes(exclude=['object'])

# By name pattern
[c for c in df.columns if c.startswith('feature_')]
[c for c in df.columns if 'date' in c.lower()]

# Combining both
numeric_cols = df.select_dtypes(include=['number']).columns.tolist()
```

#### Component Breakdown

| Method/Technique | Description | Key Parameters |
|------------------|-------------|----------------|
| `select_dtypes()` | Filter by dtype | `include`, `exclude` |
| List comprehension | Filter by name pattern | Any Python condition |

#### Syntax Rules

- `include` and `exclude` accept strings or lists of dtype strings.
- `'number'` matches all numeric dtypes (`int`, `float`, `complex`).
- `'object'` matches object dtype columns (typically strings pre-pandas 3.0).
- Under pandas 3.0, use `'str'` to match the new string dtype.

#### Constraints and Limitations

- `select_dtypes()` only filters by dtype, not by column name.
- List comprehensions over `df.columns` are Python-level and may be slower for very wide DataFrames.
- Regex-based filtering requires `re` module or pandas' `.filter(regex=...)`.
- Mixed-dtype columns appear as `object`, potentially including unintended columns.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: select_dtypes for Numeric Columns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create mixed DataFrame
df = pd.DataFrame({
    'id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [50000.0, 60000.0, 70000.0]
})

# Step 3: Select numeric columns
numeric = df.select_dtypes(include=['number'])
print(numeric)
```

**Expected Output:**
```
   id  age   salary
0   1   25  50000.0
1   2   30  60000.0
2   3   35  70000.0
```

**Why this output:** `select_dtypes(include=['number'])` selected columns with numeric dtypes (`id`, `age`, `salary`), excluding `name` (object).

#### Example 2: List Comprehension for Name Patterns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with prefixed columns
df = pd.DataFrame({
    'feature_a': [1, 2],
    'feature_b': [3, 4],
    'target': [0, 1],
    'id': [10, 20]
})

# Step 3: Select columns starting with 'feature_'
feature_cols = [c for c in df.columns if c.startswith('feature_')]
print(f"Selected: {feature_cols}\n")
print(df[feature_cols])
```

**Expected Output:**
```
Selected: ['feature_a', 'feature_b']

   feature_a  feature_b
0          1          3
1          2          4
```

**Why this output:** The list comprehension identified columns whose names start with `'feature_'`, excluding `'target'` and `'id'`.

#### Example 3: Combining Dtype and Name Filtering

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'num_feature_1': [1, 2],
    'num_feature_2': [3, 4],
    'str_feature': ['a', 'b'],
    'target': [0, 1]
})

# Step 3: Get numeric feature columns only
numeric_features = [
    c for c in df.select_dtypes(include=['number']).columns
    if c.startswith('num_')
]
print(f"Numeric feature columns: {numeric_features}")
print(f"\nSubset:\n{df[numeric_features]}")
```

**Expected Output:**
```
Numeric feature columns: ['num_feature_1', 'num_feature_2']

Subset:
   num_feature_1  num_feature_2
0              1              3
1              2              4
```

**Why this output:** The code combined dtype filtering (`select_dtypes`) with name-pattern filtering (list comprehension) to isolate numeric feature columns.

### Real-World Cases with Explanation

**Case 1: ML Feature Selection** — A data scientist automatically selects all numeric columns (`select_dtypes(include=['number'])`) for a regression model, excluding text columns.

**Case 2: Time-Series Columns** — A financial analyst selects all columns whose names contain `'_return'` using a list comprehension.

**Case 3: Schema-Agnostic Pipelines** — A data engineer builds a pipeline that dynamically selects all `float64` columns for normalization without hardcoding column names.

### References

- pandas.DataFrame.select_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.select_dtypes.html
- pandas.DataFrame.filter — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.filter.html
- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html

---

## Core Concept 4: Attribute-Style Access

### Definitions

**Core Definition:** Attribute-style access retrieves a column from a DataFrame using dot notation (`df.col_name`), providing a convenient shortcut but failing when column names conflict with DataFrame methods/attributes or contain whitespace or special characters.

**Technical Definition:** `DataFrame.__getattr__(name)` is invoked when attribute access is used. If `name` matches a column label and does not conflict with an existing DataFrame attribute or method, it returns the column as a Series. If `name` conflicts with a DataFrame method (e.g., `'mean'`, `'sum'`, `'index'`) or contains invalid Python identifier characters (e.g., spaces), it raises `AttributeError` or returns the method instead of the column.

**Beginner-Friendly Explanation:** You can sometimes grab a column by typing `df.col_name` instead of `df['col_name']`. It's shorter but risky: if your column is named `sum`, `mean`, or `index`, pandas will return the built-in method instead of your data. If the name has spaces, it won't work at all. For this reason, bracket notation is safer.

### Purposes

- To provide a concise shortcut for interactive data exploration.
- To quickly access columns in Jupyter notebooks.
- To chain Series methods with dot notation.
- To leverage IDE autocompletion for column names (when safe).

### Syntax Rules and Structure

#### Complete General Syntax

```python
df.column_name
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `df` | DataFrame object |
| `.column_name` | Valid Python identifier matching a column label |

#### Syntax Rules

- Column name must be a valid Python identifier (letters, digits, underscores; cannot start with digit).
- Column name must not conflict with DataFrame methods/attributes (e.g., `index`, `columns`, `mean`, `sum`).
- If both a column and a method share a name, the method takes precedence.
- Returns a Series.

#### Constraints and Limitations

- **Name conflicts:** Columns named `index`, `columns`, `sum`, `mean`, `min`, `max`, `count`, `size`, `shape`, `T`, `values`, etc., will not be accessible via dot notation.
- **Whitespace:** Column names with spaces (`'first name'`) cannot be accessed via dot notation.
- **Special characters:** Hyphens, parentheses, etc., break attribute access.
- **Assignment:** `df.new_col = values` works for adding columns but is discouraged in favor of bracket notation.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Successful Attribute Access

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with valid column names
df = pd.DataFrame({'name': ['Alice', 'Bob'], 'age': [25, 30]})

# Step 3: Attribute access
print(df.name)
print(f"\nType: {type(df.name).__name__}")
```

**Expected Output:**
```
0    Alice
1      Bob
Name: name, dtype: object

Type: Series
```

**Why this output:** `df.name` successfully returned the `'name'` column as a Series because `'name'` does not conflict with any DataFrame attribute or method.

#### Example 2: Failure Due to Method Conflict

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with a column named 'mean'
df = pd.DataFrame({'mean': [1, 2, 3], 'other': [4, 5, 6]})

# Step 3: Attempt attribute access
result = df.mean
print(f"Type: {type(result).__name__}")
print(f"Is it the column? {isinstance(result, pd.Series)}")
```

**Expected Output:**
```
Type: method
Is it the column? False
```

**Why this output:** `df.mean` returned the DataFrame's `.mean()` method, not the `'mean'` column, because method names take precedence over column names in attribute access. Bracket notation (`df['mean']`) would correctly return the column.

#### Example 3: Failure Due to Whitespace

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with a column containing a space
df = pd.DataFrame({'first name': ['Alice', 'Bob']})

# Step 3: Attempt attribute access
try:
    df.first name
except SyntaxError as e:
    print(f"SyntaxError: {e}")

# Step 4: Correct access with brackets
print(f"\nBracket access:\n{df['first name']}")
```

**Expected Output:**
```
SyntaxError: invalid syntax

Bracket access:
0    Alice
1      Bob
Name: first name, dtype: object
```

**Why this output:** `df.first name` is invalid Python syntax because of the space. Bracket notation `df['first name']` correctly accessed the column.

### Real-World Cases with Explanation

**Case 1: Interactive Exploration** — A data scientist uses `df.name` in a Jupyter notebook for brevity, but switches to `df['name']` in production code for safety.

**Case 2: Avoiding Conflicts** — A data engineer receives a dataset with a column named `'sum'` and must use `df['sum']` because `df.sum` returns the aggregation method.

**Case 3: Chaining Series Methods** — A data analyst uses `df.score.mean()` to chain attribute access with a Series method, relying on the column name being a valid identifier.

### References

- pandas.DataFrame.__getattr__ — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.__getattr__.html
- Attribute Access — https://pandas.pydata.org/docs/user_guide/indexing.html#attribute-access
- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html

---

## Core Concept 5: Why Bracket Notation Is Safer

### Definitions

**Core Definition:** Bracket notation (`df['col']`) is safer than attribute access (`df.col`) because it bypasses method namespace collisions, supports any string as a column name (including those with spaces or special characters), and enables dynamic variable injection via string parameters.

**Technical Definition:** `DataFrame.__getitem__()` treats the key purely as a column label, without checking for method or attribute conflicts. This makes it reliable regardless of column naming. Attribute access, by contrast, must resolve the name through Python's attribute lookup mechanism, which prioritizes DataFrame methods and attributes. Bracket notation also accepts variables (`col = 'name'; df[col]`) and expressions, enabling dynamic column selection in pipelines.

**Beginner-Friendly Explanation:** Using `df['col']` is always safe because pandas treats whatever is inside the brackets as a column name. Using `df.col` can break if your column name happens to be the same as a pandas method or has spaces. Bracket notation also lets you use variables to pick columns dynamically.

### Purposes

- To ensure code reliability regardless of column naming.
- To avoid conflicts with DataFrame methods and attributes.
- To support dynamic column selection via variables.
- To handle column names with spaces, hyphens, or special characters.
- To write maintainable, production-safe code.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Static column name
df['column_name']

# Dynamic via variable
col = 'column_name'
df[col]

# Dynamic via expression
df[prefix + '_value']

# Column names with spaces
df['first name']

# Column names conflicting with methods
df['mean']
```

#### Component Breakdown

| Technique | Safety | Use Case |
|-----------|--------|----------|
| `df['col']` | High | All scenarios |
| `df[var]` | High | Dynamic pipelines |
| `df.col` | Low | Interactive exploration only |

#### Syntax Rules

- Bracket notation accepts any string as a key.
- Variables and expressions are allowed inside brackets.
- Column names with spaces, hyphens, or leading digits work in brackets.
- Bracket notation always returns the column, never a method.

#### Constraints and Limitations

- Bracket notation requires quotes or variables; less concise than dot access.
- For `df[['col']]`, the key must be a list, not a string.
- Bracket notation with a non-existent key raises `KeyError`.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Bracket Notation with Conflicting Name

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with conflicting column name
df = pd.DataFrame({'mean': [1, 2, 3], 'sum': [4, 5, 6]})

# Step 3: Bracket access returns column
print(df['mean'])
print(f"\nType: {type(df['mean']).__name__}")

# Step 4: Attribute access returns method
print(f"\nAttribute access type: {type(df.mean).__name__}")
```

**Expected Output:**
```
0    1
1    2
2    3
Name: mean, dtype: int64

Type: Series

Attribute access type: method
```

**Why this output:** `df['mean']` correctly returned the `'mean'` column as a Series. `df.mean` returned the DataFrame's `.mean()` method. Bracket notation bypassed the namespace collision.

#### Example 2: Dynamic Column Selection via Variable

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'revenue_2023': [100, 200], 'revenue_2024': [150, 250]})

# Step 3: Dynamic column selection
year = 2024
col_name = f'revenue_{year}'
print(f"Selected column: {col_name}")
print(df[col_name])
```

**Expected Output:**
```
Selected column: revenue_2024
0    150
1    250
Name: revenue_2024, dtype: int64
```

**Why this output:** The variable `col_name` was built dynamically and passed to `df[col_name]`. This pattern is impossible with attribute access.

#### Example 3: Column Names with Spaces

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with spaces in column names
df = pd.DataFrame({
    'first name': ['Alice', 'Bob'],
    'last name': ['Smith', 'Jones'],
    'total score': [95, 87]
})

# Step 3: Access with bracket notation
print(df['first name'])
print(f"\n{df['total score']}")
```

**Expected Output:**
```
0    Alice
1      Bob
Name: first name, dtype: object

0    95
1    87
Name: total score, dtype: int64
```

**Why this output:** Bracket notation handled column names with spaces, which are impossible to access via dot notation.

### Real-World Cases with Explanation

**Case 1: Production Code** — A data engineer writes production ETL code exclusively using `df['col']` to avoid attribute conflicts and ensure reliability.

**Case 2: Dynamic Reporting** — A business analyst builds a report that selects columns based on user input (e.g., `df[user_selected_column]`).

**Case 3: Third-Party Data** — A data scientist works with a dataset from an external source where column names contain spaces and special characters, requiring bracket notation.

### References

- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html
- Attribute Access — https://pandas.pydata.org/docs/user_guide/indexing.html#attribute-access
- pandas.DataFrame.__getitem__ — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.__getitem__.html

---

## Core Concept 6: Column Alteration & Assignment

### Definitions

**Core Definition:** Column alteration and assignment refers to creating, renaming, dropping, or updating DataFrame columns, using direct assignment (`df['new'] = values`), `.rename()`, `.drop()`, and `.loc`, while avoiding `SettingWithCopyWarning` by using explicit assignments or the `.loc` accessor.

**Technical Definition:** Column assignment via `df['new'] = values` adds a new column or updates an existing one, aligning Series by index. `DataFrame.rename(columns={old: new})` returns a new DataFrame with renamed columns. `DataFrame.drop(columns=[...])` returns a new DataFrame without the specified columns. Under CoW (pandas 3.0), modifications to a DataFrame never affect a DataFrame from which it was derived, and `SettingWithCopyWarning` is removed. The recommended pattern for conditional updates is `df.loc[condition, 'col'] = value`.

**Beginner-Friendly Explanation:** You can add new columns, rename existing ones, drop unwanted ones, or update values. The safest way to modify a subset is to use `.loc` with a condition. Modern pandas (3.0+) uses Copy-on-Write, which means modifications to a derived DataFrame never affect the original, and the old `SettingWithCopyWarning` is gone.

### Purposes

- To create new columns from existing data (feature engineering).
- To rename columns for clarity or consistency.
- To drop irrelevant or redundant columns.
- To update values conditionally using `.loc`.
- To avoid `SettingWithCopyWarning` and ensure predictable behavior.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Adding a new column
df['new_col'] = values

# Renaming columns
df = df.rename(columns={'old': 'new'})
df.rename(columns={'old': 'new'}, inplace=True)

# Dropping columns
df = df.drop(columns=['col1', 'col2'])
df.drop(columns=['col1'], inplace=True)

# Conditional update
df.loc[df['A'] > 10, 'B'] = 0
```

#### Component Breakdown

| Operation | Method/Syntax | Returns |
|-----------|---------------|---------|
| Add column | `df['new'] = values` | None (modifies in place) |
| Rename | `df.rename(columns={...})` | New DataFrame |
| Drop | `df.drop(columns=[...])` | New DataFrame |
| Conditional update | `df.loc[cond, 'col'] = value` | None (modifies in place) |

#### Syntax Rules

- Assigning a scalar broadcasts it across all rows.
- Assigning a Series aligns by index; missing labels receive `NaN`.
- `rename()` and `drop()` return new DataFrames unless `inplace=True`.
- `.loc` with a boolean mask is the safe way to update subsets.
- Under CoW, `inplace=True` is discouraged; prefer reassignment.

#### Constraints and Limitations

- **SettingWithCopyWarning:** Occurs in legacy pandas when modifying a slice; removed under CoW.
- **Chained assignment:** `df[df['A'] > 0]['B'] = 0` never works reliably; use `.loc`.
- **inplace parameter:** Deprecated in many methods under CoW; prefer reassignment.
- **Index alignment:** Assigning a misaligned Series introduces `NaN`.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Adding and Updating Columns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

# Step 3: Add a new column
df['C'] = df['A'] + df['B']
print("After adding C:")
print(df)

# Step 4: Update an existing column
df['A'] = df['A'] * 10
print("\nAfter updating A:")
print(df)
```

**Expected Output:**
```
After adding C:
   A  B  C
0  1  4  5
1  2  5  7
2  3  6  9

After updating A:
    A  B  C
0  10  4  5
1  20  5  7
2  30  6  9
```

**Why this output:** `df['C'] = df['A'] + df['B']` created a new column via vectorized addition. `df['A'] = df['A'] * 10` updated the existing column in place.

#### Example 2: Renaming and Dropping Columns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'old_name': [1, 2], 'keep': [3, 4], 'drop_me': [5, 6]})

# Step 3: Rename a column
df = df.rename(columns={'old_name': 'new_name'})
print("After rename:")
print(df)

# Step 4: Drop a column
df = df.drop(columns=['drop_me'])
print("\nAfter drop:")
print(df)
```

**Expected Output:**
```
After rename:
   new_name  keep  drop_me
0         1     3        5
1         2     4        6

After drop:
   new_name  keep
0         1     3
1         2     4
```

**Why this output:** `.rename(columns={'old_name': 'new_name'})` renamed the column. `.drop(columns=['drop_me'])` removed the `drop_me` column.

#### Example 3: Conditional Update with .loc (Avoiding SettingWithCopyWarning)

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'score': [85, 45, 92, 38, 78]})

# Step 3: Update failing scores using .loc
df.loc[df['score'] < 50, 'score'] = 0
print(df)
```

**Expected Output:**
```
   score
0     85
1      0
2     92
3      0
4     78
```

**Why this output:** `.loc[df['score'] < 50, 'score'] = 0` selected rows where the score was below 50 and set those scores to 0. This pattern is safe and does not trigger `SettingWithCopyWarning` because it operates directly on the original DataFrame.

### Real-World Cases with Explanation

**Case 1: Feature Engineering** — A data scientist creates new columns (e.g., `df['ratio'] = df['A'] / df['B']`) and renames them for clarity before model training.

**Case 2: Data Cleaning** — A data engineer drops columns with excessive missing values (`df.drop(columns=cols_to_drop)`) and renames ambiguous column names.

**Case 3: Conditional Imputation** — An analyst updates missing or invalid values in a subset of rows using `.loc` with a boolean mask, ensuring the original DataFrame is modified correctly.

### References

- pandas.DataFrame.rename — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename.html
- pandas.DataFrame.drop — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.drop.html
- Copy-on-Write — https://pandas.pydata.org/docs/user_guide/copy_on_write.html
- Returning a View versus Copy — https://pandas.pydata.org/docs/user_guide/indexing.html#returning-a-view-versus-a-copy
- SettingWithCopyWarning — https://pandas.pydata.org/docs/user_guide/indexing.html#returning-a-view-versus-a-copy

---

## Summary Table: Column Selection & Manipulation Methods

| Operation | Syntax | Returns | Notes |
|-----------|--------|---------|-------|
| Single column (Series) | `df['col']` | Series | Safe, always works |
| Single column (DataFrame) | `df[['col']]` | DataFrame | Preserves 2-D structure |
| Multiple columns | `df[['A', 'B']]` | DataFrame | Order follows list |
| By dtype | `df.select_dtypes(include=['number'])` | DataFrame | Dynamic filtering |
| By name pattern | `[c for c in df.columns if ...]` | List | Python-level filtering |
| Attribute access | `df.col` | Series | Risky (conflicts, spaces) |
| Add/update column | `df['new'] = values` | None | Modifies in place |
| Rename | `df.rename(columns={...})` | DataFrame | Returns new unless `inplace=True` |
| Drop | `df.drop(columns=[...])` | DataFrame | Returns new unless `inplace=True` |
| Conditional update | `df.loc[cond, 'col'] = value` | None | Safe, avoids warnings |

---

## References

- pandas.DataFrame.__getitem__ — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.__getitem__.html
- pandas.DataFrame.__getattr__ — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.__getattr__.html
- pandas.DataFrame.select_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.select_dtypes.html
- pandas.DataFrame.rename — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename.html
- pandas.DataFrame.drop — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.drop.html
- pandas.DataFrame.filter — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.filter.html
- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html
- Attribute Access — https://pandas.pydata.org/docs/user_guide/indexing.html#attribute-access
- Returning a View versus Copy — https://pandas.pydata.org/docs/user_guide/indexing.html#returning-a-view-versus-a-copy
- Copy-on-Write — https://pandas.pydata.org/docs/user_guide/copy_on_write.html
- PDEP-7: Consistent copy/view semantics — http://pandas.pydata.org/pdeps/0007-copy-on-write.html
- What's New in pandas 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html