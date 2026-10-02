# Pandas Renaming Data Elements: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Renaming data elements in pandas is the process of changing the labels of DataFrame columns and index entries, either individually through mapping dictionaries, programmatically through functions, or structurally through axis assignment methods.

**Technical Definition:** Renaming is accomplished through three primary mechanisms: `DataFrame.rename()` for label-to-label mapping via dictionaries, functions, or Series; `DataFrame.set_axis()` for wholesale replacement of axis labels with a new list-like or Index object; and direct attribute assignment (`df.index.name`, `df.columns.name`) or `rename_axis()` for changing axis names. Programmatic standardization applies string operations and regular expressions to transform column names into consistent formats such as `lower_snake_case`.

**Beginner-Friendly Explanation:** When you load data from a CSV or Excel file, the column names are often messy—they might have spaces, capital letters, special characters, or be in a different language. Renaming lets you clean them up so your code is easier to write and read. You can rename just a few columns, all of them, or change the names of the row and column axes themselves.

### Key Characteristics

- **Immutable by default:** Since pandas 3.0, `rename()` and `set_axis()` always return a new object using Copy-on-Write semantics; the `inplace` parameter is deprecated and will be removed in pandas 4.0.
- **Selective vs. wholesale:** `rename()` changes only the labels you specify; `set_axis()` replaces all labels at once.
- **Function-based transformations:** `rename()` accepts a function (e.g., `str.lower`, a lambda) that is applied to every label, enabling programmatic standardization in one call.
- **Axis names vs. axis labels:** `rename_axis()` changes the name of the index or columns (the "meta name"), not the individual labels themselves.
- **Dictionary mappings are partial:** Labels not included in the mapping dictionary are left unchanged; extra keys are ignored by default.

### Prerequisites

- Basic Python knowledge (dictionaries, lambda expressions, functions).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of the `.str` accessor for vectorized string operations.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Cleaning:** Standardizing column names is typically the first step after loading raw data.
- **Feature Engineering:** Renaming columns to descriptive names improves code readability and maintainability.
- **ETL Pipelines:** Consistent naming conventions across multiple datasets facilitate automated joins and concatenations.
- **Database Integration:** Renaming columns to match database schema conventions (e.g., `lower_snake_case`) simplifies SQL integration.

### Core Concepts / Features

1. Axis Modifications: Structural Renaming with `df.rename()`
2. Structural Maps: Dictionary Mappings vs. Lambda String Mutations
3. Index Transformations: `index.name`, `set_axis()`, and `rename_axis()`
4. Programmatic Standardization: Sanitizing Column Names


## 1. Axis Modifications: Structural Renaming with df.rename()

### Definitions

**Core Definition:** `DataFrame.rename()` changes the labels of columns or index entries by applying a mapping dictionary, function, or Series to the specified axis.

**Technical Definition:** `DataFrame.rename(mapper=None, *, index=None, columns=None, axis=None, copy=<no_default>, inplace=<no_default>, level=None, errors='ignore')` returns a new DataFrame with the specified axis labels renamed. The method supports two calling conventions: `(index=index_mapper, columns=columns_mapper, ...)` and `(mapper, axis={'index', 'columns'}, ...)`. The `mapper` can be a dict-like object or a function. When `errors='raise'`, a `KeyError` is raised if any label in the mapper is not found in the axis.

**Beginner-Friendly Explanation:** `rename()` lets you change column names by saying “change 'old_name' to 'new_name'.” You can rename just a few columns using a dictionary, or rename all of them by passing a function. The original DataFrame is not modified unless you explicitly ask for it.

### Purposes

- To rename specific columns or index labels using a dictionary mapping.
- To apply a function uniformly to all column names or index labels.
- To rename labels in a MultiIndex at a specific level using the `level` parameter.
- To rename both columns and index in a single method call.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Using columns/index keyword arguments (recommended)
df.rename(columns={'old': 'new'}, index={'old_idx': 'new_idx'})

# Using mapper + axis
df.rename({'old': 'new'}, axis='columns')
df.rename({'old_idx': 'new_idx'}, axis='index')

# Using a function for all labels
df.rename(columns=str.lower)
df.rename(columns=lambda x: x.strip().replace(' ', '_'))

# MultiIndex level-specific renaming
df.rename(columns={'old': 'new'}, level=0)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `mapper` | Dict-like or function to apply to the axis values |
| `index` | Alternative to `mapper` + `axis=0` |
| `columns` | Alternative to `mapper` + `axis=1` |
| `axis` | `0` or `'index'` (default), `1` or `'columns'` |
| `level` | For MultiIndex, specify which level to rename |
| `errors` | `'ignore'` (default) or `'raise'` |
| `inplace` | Deprecated since pandas 3.1.0 |

**Syntax Rules:**

- Labels not present in a dictionary mapper are left unchanged.
- Extra keys in the mapper are ignored when `errors='ignore'`.
- The `mapper` argument is positional and can be combined with `axis`.
- Function mappers are applied to every label on the target axis.
- `inplace=True` is deprecated; use reassignment instead.

**Constraints and Limitations:**

- `errors='ignore'` silently ignores missing labels; use `errors='raise'` to catch typos in mapper keys.
- Function mappers may produce duplicate labels if the transformation is not injective.
- The `level` parameter is only meaningful for MultiIndex axes.

### Annotated Code Examples

**Example 1: Renaming Columns with a Dictionary**

```python
import pandas as pd

df = pd.DataFrame({
    "A": [1, 2, 3],
    "B": [4, 5, 6]
})

# Rename column A to 'alpha' and B to 'beta'
df_renamed = df.rename(columns={"A": "alpha", "B": "beta"})
print(df_renamed)
# Expected output:
#    alpha  beta
# 0      1     4
# 1      2     5
# 2      3     6

# Original is unchanged
print(df)
# Expected output:
#    A  B
# 0  1  4
# 1  2  5
# 2  3  6
```

**Why this output:** The dictionary `{"A": "alpha", "B": "beta"}` maps old column names to new ones. Columns not in the dictionary would remain unchanged. Since `inplace` is not used, the original `df` is preserved.

**Example 2: Renaming Index Labels and Using axis**

```python
# Rename index labels
df2 = df.rename(index={0: "row_a", 1: "row_b", 2: "row_c"})
print(df2)
# Expected output:
#        A  B
# row_a  1  4
# row_b  2  5
# row_c  3  6

# Using mapper + axis convention
df3 = df.rename({"A": "X"}, axis="columns")
print(df3)
# Expected output:
#    X  B
# 0  1  4
# 1  2  5
# 2  3  6
```

**Why this output:** The `index` parameter renames row labels. The `mapper` + `axis='columns'` convention is equivalent to using `columns=`. Both calling conventions are supported.

**Example 3: Using a Function to Rename All Columns**

```python
df4 = pd.DataFrame({
    "  Name ": ["Alice", "Bob"],
    " AGE ": [25, 30],
    "Salary ($)": [50000, 60000]
})

# Apply a function to all column names
df4_clean = df4.rename(columns=lambda x: x.strip().lower().replace(" ", "_").replace("($)", ""))
print(df4_clean.columns.tolist())
# Expected output: ['name', 'age', 'salary_']
```

**Why this output:** The lambda function strips whitespace, converts to lowercase, replaces spaces with underscores, and removes the dollar sign. All column names are transformed in one call.

### Real-World Cases

- **CSV imports:** After `pd.read_csv()`, rename cryptic column headers (e.g., `Unnamed: 0`, `col_1`) to meaningful names.
- **Database exports:** Rename columns to match a downstream schema before loading into a database.
- **Multi-language datasets:** Translate column names from one language to another using a dictionary mapper.

### References

- pandas.DataFrame.rename — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename.html
- Renaming / mapping labels (user guide) — https://pandas.pydata.org/docs/user_guide/basics.html#basics-rename


## 2. Structural Maps: Dictionary Mappings vs. Lambda String Mutations

### Definitions

**Core Definition:** The `columns` parameter of `rename()` accepts either a dictionary (for selective, explicit renaming) or a function/lambda (for programmatic, rule-based renaming of all columns).

**Technical Definition:** When a dictionary is passed, pandas performs a one-to-one lookup: each key in the dictionary is matched against the existing column labels, and the corresponding value becomes the new label. When a function is passed, pandas applies the function to every column label in sequence, using the function's return value as the new label. The function can be a built-in string method (e.g., `str.lower`), a lambda, or a named function. Dictionary values must be unique (1-to-1 mapping); extra keys are ignored by default.

**Beginner-Friendly Explanation:** A dictionary is like a translation table: “replace this name with that name.” A function is like a rule: “apply this transformation to every name.” Use a dictionary when you need to rename a few specific columns; use a function when you want to clean up all columns in a consistent way.

### Purposes

- To rename a small number of columns with specific new names using a dictionary.
- To apply a consistent transformation (lowercase, strip, replace) to all columns using a function.
- To combine dictionary and function approaches in a single `rename()` call.
- To rename columns based on pattern matching using regular expressions within a lambda.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Dictionary mapping (selective)
df.rename(columns={"old_name": "new_name"})

# Function mapping (applied to all)
df.rename(columns=str.lower)
df.rename(columns=lambda x: x.strip().replace(" ", "_"))

# Combining both (not directly supported; use separate calls or a function)
df.rename(columns=lambda x: mapping.get(x, x))
```

**Component Breakdown:**

| Approach | Behaviour |
|----------|-----------|
| Dictionary | Only listed labels are renamed; others unchanged |
| Function | Every label is passed through the function |
| `str.lower` | Built-in string method; equivalent to `lambda x: x.lower()` |
| Lambda | Custom transformation with arbitrary Python logic |

**Syntax Rules:**

- Dictionary keys must match existing labels; otherwise, they are ignored (with `errors='ignore'`).
- Function mappers must return a hashable value for each label.
- Built-in methods like `str.lower`, `str.upper`, `str.title` can be passed directly.
- Lambdas can incorporate regex via `re.sub()` or pandas string methods.

**Constraints and Limitations:**

- Function mappers cannot rename only specific columns unless the lambda includes conditional logic.
- Duplicate resulting labels cause a `ValueError` when assigning back to the DataFrame.
- Dictionary mappers are faster than function mappers for large column sets.

### Annotated Code Examples

**Example 1: Dictionary vs. Function Comparison**

```python
import pandas as pd

df = pd.DataFrame({
    "First Name": ["Alice", "Bob"],
    "Last Name": ["Smith", "Jones"],
    "Age (Years)": [25, 30]
})

# Dictionary: rename specific columns
df_dict = df.rename(columns={
    "First Name": "first_name",
    "Last Name": "last_name"
})
print(df_dict.columns.tolist())
# Expected output: ['first_name', 'last_name', 'Age (Years)']

# Function: apply transformation to all columns
df_func = df.rename(columns=lambda x: x.lower().replace(" ", "_").replace("(", "").replace(")", ""))
print(df_func.columns.tolist())
# Expected output: ['first_name', 'last_name', 'age_years']
```

**Why this output:** The dictionary approach renames only `First Name` and `Last Name`, leaving `Age (Years)` unchanged. The function approach applies the transformation to all three columns, producing `first_name`, `last_name`, and `age_years`.

**Example 2: Using str.lower Directly**

```python
df2 = pd.DataFrame({
    "NAME": [1],
    "AGE": [2],
    "SALARY": [3]
})

# Pass str.lower as the mapper
df2_lower = df2.rename(columns=str.lower)
print(df2_lower.columns.tolist())
# Expected output: ['name', 'age', 'salary']
```

**Why this output:** `str.lower` is a built-in method that takes a string and returns its lowercase version. pandas applies it to every column label, producing lowercase names.

**Example 3: Regex-Based Renaming with Lambda**

```python
import re

df3 = pd.DataFrame({
    "Col_1 ($)": [1],
    "Col_2 ($)": [2],
    "Col_3 (%)": [3]
})

# Remove all non-alphanumeric characters using regex
df3_clean = df3.rename(columns=lambda x: re.sub(r'[^a-z0-9]+', '_', x.lower()).strip('_'))
print(df3_clean.columns.tolist())
# Expected output: ['col_1', 'col_2', 'col_3']
```

**Why this output:** The lambda lowercases the string, replaces all non-alphanumeric sequences with underscores, and strips leading/trailing underscores. This is a common sanitization pattern for messy column names.

### Real-World Cases

- **Excel exports:** Columns often contain spaces, parentheses, and special characters; use a lambda with regex to standardize them.
- **API responses:** JSON keys may be in camelCase; use a function mapper to convert to snake_case.
- **Multi-source integration:** Use a dictionary to align column names from different sources to a common schema.

### References

- pandas.DataFrame.rename — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename.html
- pandas string methods — https://pandas.pydata.org/docs/reference/series.html#string-handling


## 3. Index Transformations: Directly Modifying Axis Attributes

### Definitions

**Core Definition:** Index transformations modify the structural attributes of a DataFrame's axes—the row index labels, column labels, and the names assigned to each axis—using `index.name`, `columns.name`, `set_axis()`, and `rename_axis()`.

**Technical Definition:** `DataFrame.index` and `DataFrame.columns` are `Index` objects with a `.name` attribute that can be set directly (`df.index.name = 'ID'`) or via `rename_axis()`. `DataFrame.set_axis(labels, *, axis=0, copy=<no_default>)` replaces all labels on the specified axis with a new list-like or Index object. `DataFrame.rename_axis(mapper=<no_default>, *, index=<no_default>, columns=<no_default>, axis=0, copy=<no_default>, inplace=<no_default>)` sets the name of the axis for the index or columns, which is useful for labeling axes in a MultiIndex or providing descriptive names.

**Beginner-Friendly Explanation:** `set_axis()` is for replacing all row or column labels at once. `rename_axis()` and `index.name`/`columns.name` are for giving names to the axes themselves—like labeling the rows “Customer ID” and the columns “Product Features.”

### Purposes

- To replace all row or column labels with a new list using `set_axis()`.
- To assign descriptive names to the index and columns axes using `rename_axis()` or `.name`.
- To remove axis names by setting them to `None`.
- To set axis names for MultiIndex DataFrames.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Set all row labels
df.set_axis(new_labels, axis=0)

# Set all column labels
df.set_axis(new_labels, axis=1)

# Set index name directly
df.index.name = 'row_id'
df.columns.name = 'features'

# Using rename_axis
df.rename_axis('row_id', axis=0)
df.rename_axis('features', axis=1)
df.rename_axis(index='row_id', columns='features')

# Remove axis names
df.rename_axis(None, axis=0)
df.rename_axis(None, axis=1)
```

**Component Breakdown:**

| Method/Attribute | Description |
|------------------|-------------|
| `set_axis(labels, axis=0)` | Replace all row labels |
| `set_axis(labels, axis=1)` | Replace all column labels |
| `index.name` | Direct attribute for row axis name |
| `columns.name` | Direct attribute for column axis name |
| `rename_axis()` | Functional approach for setting axis names |
| `rename_axis(None)` | Remove axis name |

**Syntax Rules:**

- `set_axis()` requires the new labels to be the same length as the current axis.
- `set_axis()` returns a new object since pandas 3.0; `inplace` is deprecated.
- `index.name` and `columns.name` can be assigned directly or set to `None` to remove the name.
- `rename_axis()` supports both `(mapper, axis=...)` and `(index=..., columns=...)` calling conventions.
- In pandas 3.0, `copy` parameters are ignored due to Copy-on-Write.

**Constraints and Limitations:**

- `set_axis()` replaces all labels; it cannot selectively rename individual labels (use `rename()` for that).
- Direct attribute assignment (`df.index.name = ...`) modifies the object in place and may not be CoW-compatible in all contexts.
- `rename_axis()` with a dict-like mapper changes axis labels (deprecated behaviour); use `rename()` for label changes.

### Annotated Code Examples

**Example 1: Using set_axis() to Replace All Labels**

```python
import pandas as pd

df = pd.DataFrame({
    "A": [1, 2, 3],
    "B": [4, 5, 6]
})

# Replace row labels
df_rows = df.set_axis(["a", "b", "c"], axis="index")
print(df_rows)
# Expected output:
#    A  B
# a  1  4
# b  2  5
# c  3  6

# Replace column labels
df_cols = df.set_axis(["I", "II"], axis="columns")
print(df_cols)
# Expected output:
#    I  II
# 0  1   4
# 1  2   5
# 2  3   6
```

**Why this output:** `set_axis()` replaces the specified axis labels entirely. The `axis` parameter determines whether rows or columns are updated. The new labels must match the length of the existing axis.

**Example 2: Setting Axis Names with index.name and columns.name**

```python
# Set axis names directly
df_named = df.copy()
df_named.index.name = "row_id"
df_named.columns.name = "features"
print(df_named)
# Expected output:
# features  A  B
# row_id
# 0         1  4
# 1         2  5
# 2         3  6

print(df_named.index.name)    # row_id
print(df_named.columns.name)  # features
```

**Why this output:** Setting `.index.name` and `.columns.name` adds descriptive labels to the axes. These names appear in the printed output, with `features` above the column headers and `row_id` to the left of the row labels.

**Example 3: Using rename_axis() for Functional Axis Naming**

```python
# Using rename_axis for both axes
df_named2 = df.rename_axis(index="row_id", columns="features")
print(df_named2)
# Expected output:
# features  A  B
# row_id
# 0         1  4
# 1         2  5
# 2         3  6

# Remove axis names
df_unnamed = df_named2.rename_axis(index=None, columns=None)
print(df_unnamed.index.name)    # None
print(df_unnamed.columns.name)  # None
```

**Why this output:** `rename_axis()` provides a functional alternative to direct attribute assignment. It supports both `index` and `columns` keyword arguments. Setting to `None` removes the axis names.

### Real-World Cases

- **Financial reports:** Label the row index as “Date” and the column axis as “Metrics” for clarity.
- **MultiIndex DataFrames:** Use `rename_axis()` to name each level of a MultiIndex (e.g., `['Year', 'Quarter']`).
- **Database exports:** Remove default axis names (often `None` or `'index'`) before exporting to CSV or SQL.

### References

- pandas.DataFrame.set_axis — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.set_axis.html
- pandas.DataFrame.rename_axis — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename_axis.html
- pandas.Index.name — https://pandas.pydata.org/docs/reference/api/pandas.Index.name.html


## 4. Programmatic Standardization: Sanitizing Column Names

### Definitions

**Core Definition:** Programmatic standardization is the automated process of cleaning and normalizing column names into a consistent format, typically `lower_snake_case`, by removing whitespace, converting to lowercase, replacing special characters with underscores, and eliminating leading/trailing underscores.

**Technical Definition:** Column name sanitization leverages pandas' `.str` accessor on the `Index` object (`df.columns.str`) to chain vectorized string operations: `.str.strip()` removes leading/trailing whitespace, `.str.lower()` converts to lowercase, `.str.replace(r'[^a-z0-9]+', '_', regex=True)` replaces non-alphanumeric sequences with underscores, and `.str.strip('_')` removes leading/trailing underscores. Alternatively, `rename()` with a lambda function can apply the same transformations. The result is a set of column names that are predictable, lowercase, underscore-separated, and free of special characters.

**Beginner-Friendly Explanation:** Standardizing column names means making them all follow the same rules: all lowercase, words separated by underscores, no spaces or special characters. This makes your code easier to write because you don't have to remember whether a column is called `"Total Amount"`, `"total amount"`, or `"Total_Amount"`.

### Purposes

- To remove spaces, parentheses, dollar signs, and other special characters from column names.
- To convert all column names to `lower_snake_case` for consistency and Python-friendliness.
- To ensure that column names are valid Python identifiers (no spaces or special characters).
- To create a reusable utility function for cleaning incoming DataFrames.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Using .str accessor on columns
df.columns = (
    df.columns
    .str.strip()
    .str.lower()
    .str.replace(r'[^a-z0-9]+', '_', regex=True)
    .str.strip('_')
)

# Using rename with a lambda
df = df.rename(columns=lambda x: (
    x.strip().lower()
    .replace(' ', '_')
    .replace('(', '').replace(')', '')
    .replace('$', '')
))
```

**Component Breakdown:**

| Operation | Purpose |
|-----------|---------|
| `.str.strip()` | Remove leading/trailing whitespace |
| `.str.lower()` | Convert to lowercase |
| `.str.replace(r'[^a-z0-9]+', '_', regex=True)` | Replace non-alphanumeric sequences with underscores |
| `.str.strip('_')` | Remove leading/trailing underscores |
| `re.sub()` in lambda | Custom regex replacement |

**Syntax Rules:**

- The `.str` accessor requires the Index to be of string type; use `.astype(str)` first if needed.
- `regex=True` is required when using regular expression patterns with `.str.replace()`.
- The pattern `[^a-z0-9]+` matches any sequence of characters that are not lowercase letters or digits.
- Chaining is applied left-to-right; `.str.strip('_')` should come after the replace step.

**Constraints and Limitations:**

- Aggressive regex replacement may collapse distinct column names into the same name (e.g., `"Col 1"` and `"Col_1"` both become `"col_1"`), causing duplicates.
- Column names that start with a digit (e.g., `"2024_sales"`) are not valid Python identifiers but are still valid pandas column labels.
- The `.str` accessor is slower than a direct list comprehension for small DataFrames.

### Annotated Code Examples

**Example 1: Basic Column Sanitization**

```python
import pandas as pd

# Messy column names from an Excel export
df = pd.DataFrame({
    " Order ID ": [1001, 1002],
    "Customer Name (Full)": ["Alice", "Bob"],
    "Total Amount ($)": [49.99, 120.00],
    "Order-Date": ["2024-03-12", "2024-03-15"]
})

# Sanitize using .str accessor chain
df.columns = (
    df.columns
    .str.strip()                          # remove leading/trailing spaces
    .str.lower()                          # lowercase
    .str.replace(r"[^a-z0-9]+", "_", regex=True)  # replace non-alphanum with _
    .str.strip("_")                       # remove leading/trailing underscores
)

print(df.columns.tolist())
# Expected output: ['order_id', 'customer_name_full', 'total_amount', 'order_date']
```

**Why this output:** The chain strips whitespace, lowercases, replaces `" "`, `"("`, `")"`, `"$"`, and `"-"` with underscores (via the regex), and removes leading/trailing underscores. The result is a clean set of `lower_snake_case` names.

**Example 2: Using rename with a Lambda**

```python
# Same DataFrame with messy columns
df2 = pd.DataFrame({
    "  Name ": ["Alice"],
    " AGE ": [25],
    "Salary ($)": [50000]
})

# Sanitize with rename + lambda
df2_clean = df2.rename(columns=lambda x: (
    x.strip().lower()
    .replace(' ', '_')
    .replace('(', '').replace(')', '')
    .replace('$', '')
))

print(df2_clean.columns.tolist())
# Expected output: ['name', 'age', 'salary']
```

**Why this output:** The lambda applies a sequence of string replacements. Unlike the `.str` accessor approach, this method returns a new DataFrame (since `rename()` returns a copy) rather than modifying `df.columns` in place.

**Example 3: Handling Duplicate Names After Sanitization**

```python
df3 = pd.DataFrame({
    "Col 1": [1],
    "Col_1": [2],
    "Col-1": [3]
})

# Sanitize — all become 'col_1'
df3_clean = df3.rename(columns=lambda x: x.strip().lower().replace(' ', '_').replace('-', '_'))
print(df3_clean.columns.tolist())
# Expected output: ['col_1', 'col_1', 'col_1'] — duplicates!

# Resolve by appending a suffix
def make_unique(cols):
    seen = {}
    result = []
    for c in cols:
        if c in seen:
            seen[c] += 1
            result.append(f"{c}_{seen[c]}")
        else:
            seen[c] = 0
            result.append(c)
    return result

df3.columns = make_unique(df3.columns)
print(df3.columns.tolist())
# Expected output: ['col_1', 'col_1_1', 'col_1_2']
```

**Why this output:** Aggressive sanitization can collapse distinct names into the same string. The `make_unique` helper appends numeric suffixes to resolve collisions, ensuring all column names remain unique.

### Real-World Cases

- **Excel/CSV imports:** Column names often contain spaces, parentheses, and currency symbols; sanitization makes them Python-friendly.
- **Multi-source data integration:** Standardizing column names across datasets enables automated `pd.concat()` and `pd.merge()` operations.
- **Reusable ETL utilities:** Define a `clean_columns(df)` function that applies the sanitization chain and returns the cleaned DataFrame for use across all pipeline stages.

### References

- pandas.Index.str — https://pandas.pydata.org/docs/reference/api/pandas.Index.str.html
- pandas.Series.str.replace — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.replace.html
- pandas.DataFrame.rename — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename.html


## References

- pandas.DataFrame.rename — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename.html
- pandas.DataFrame.set_axis — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.set_axis.html
- pandas.DataFrame.rename_axis — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rename_axis.html
- pandas.Index.name — https://pandas.pydata.org/docs/reference/api/pandas.Index.name.html
- pandas.Index.str — https://pandas.pydata.org/docs/reference/api/pandas.Index.str.html
- pandas.Series.str.replace — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.replace.html
- Renaming / mapping labels (user guide) — https://pandas.pydata.org/docs/user_guide/basics.html#basics-rename
- Pandas rename columns tutorial (DataCamp) — https://www.datacamp.com/tutorial/pandas-rename-columns
- pandas 3.0 Copy-on-Write — https://pandas.pydata.org/docs/user_guide/copy_on_write.html
- PDEP-8 In-place methods in pandas — https://pandas.pdeps/0008-inplace-methods-in-pandas.html