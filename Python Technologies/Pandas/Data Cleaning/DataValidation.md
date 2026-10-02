# Pandas Data Validation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Data validation in pandas is the systematic process of verifying that a DataFrame or Series meets predefined structural, domain, and relational constraints before it is used in analysis, modeling, or downstream processing.

**Technical Definition:** Data validation encompasses a set of programmatic checks—implemented through native pandas methods (e.g., `df.dtypes`, `pd.api.types`, `Series.str.match`), assertion functions, and external validation libraries such as Pandera and Great Expectations—that verify column presence, data types, value ranges, categorical membership, uniqueness constraints, and non-nullable field requirements. Validation can be applied at function boundaries using decorators or `.pipe()` workflows, and can return boolean results, raise exceptions, or produce detailed diagnostic reports.

**Beginner-Friendly Explanation:** Before you trust your data, you need to check it. Data validation is like a quality-control inspection: you verify that all the expected columns are there, the types are correct, the numbers are within acceptable ranges, the text follows the right patterns, and there are no unexpected duplicates. If something is wrong, validation catches it early—before it causes problems downstream.

### Key Characteristics

- **Schema-first approach:** Define the expected structure (columns, dtypes, constraints) once, then validate data against it repeatedly.
- **Layered validation:** Checks can be applied at the column level (dtype, range, pattern), row level (uniqueness, non-null), and DataFrame level (column presence, column order).
- **Fail-fast vs. collect-all:** Some validation approaches stop at the first error (assertions, `.pipe()`), while others (Pandera with lazy validation, Great Expectations) collect all errors before reporting.
- **Ecosystem integration:** Pandera integrates with pandas, Polars, PySpark, and Dask; Great Expectations supports Pandas, Spark, SQL, and BigQuery.
- **Declarative vs. imperative:** Declarative approaches (Pandera schemas, Great Expectations suites) separate the “what” from the “how”; imperative approaches (manual `assert` statements, `.pipe()` functions) interleave validation with processing logic.

### Prerequisites

- Basic Python knowledge (functions, decorators, lambda expressions).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of dtypes, missing data, and boolean masking.
- Installation of pandas (`pip install pandas`); optional: Pandera (`pip install 'pandera[pandas]'`), Great Expectations (`pip install great_expectations`).

### Related Programming Areas

- **ETL/ELT Pipelines:** Validation at ingestion, transformation, and output boundaries ensures data integrity.
- **Machine Learning Pipelines:** Schema validation prevents training on malformed data and catches feature drift.
- **Data Quality Engineering:** Validation metrics feed into observability dashboards (OpenTelemetry, Prometheus) for monitoring.
- **Database Integration:** Validation mirrors SQL constraints (NOT NULL, UNIQUE, CHECK, FOREIGN KEY) before loading data.

### Core Concepts / Features

1. Structural Checks: Confirming Data Layouts, Schema Constraints, and Type Alignments
2. Domain Boundaries: Range Validation and Regular Expression Parsing
3. Category Enforcement: Validating Categorical Membership
4. Relational Constraints: Uniqueness, Composite Keys, and Non-Nullable Fields
5. Validation Ecosystems: Assertions, `.pipe()` Workflows, and External Libraries


## 1. Structural Checks: Confirming Data Layouts, Schema Constraints, and Type Alignments

### Definitions

**Core Definition:** Structural checks verify that a DataFrame has the expected columns, in the expected order, with the expected data types, and that the overall shape and index structure match expectations.

**Technical Definition:** Structural validation is performed by inspecting `DataFrame.columns`, `DataFrame.dtypes`, `DataFrame.shape`, and `DataFrame.index`. The `pd.api.types` module provides functions such as `is_numeric_dtype()`, `is_string_dtype()`, `is_datetime64_any_dtype()`, and `is_categorical_dtype()` for programmatic dtype checking. Column presence is verified with `set(df.columns) == expected_columns` or `df.columns.tolist() == expected_order`. External libraries like Pandera define schemas with `pa.Column(int)`, `pa.Column(float)`, and `pa.Column(str)` to enforce structural constraints declaratively.

**Beginner-Friendly Explanation:** Structural checks are like checking that a form has all the required fields filled in the right places. You verify that every column you expect is present, that the columns are in the right order (if order matters), and that each column contains the right kind of data (numbers, text, dates).

### Purposes

- To verify that all expected columns are present in the DataFrame.
- To detect extra or missing columns that could cause downstream errors.
- To confirm that each column has the expected dtype family (numeric, string, datetime, categorical).
- To validate column order when positional indexing or feature arrays depend on it.
- To ensure the index structure (type, uniqueness, frequency) meets expectations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Column presence
set(df.columns) == expected_columns
df.columns.tolist() == expected_order

# dtype checking
df.dtypes
pd.api.types.is_numeric_dtype(series)
pd.api.types.is_string_dtype(series)
pd.api.types.is_datetime64_any_dtype(series)
pd.api.types.is_categorical_dtype(series)

# Shape checking
df.shape == (expected_rows, expected_cols)

# Index checking
df.index.is_unique
df.index.dtype
```

**Component Breakdown:**

| Check | Method/Function |
|-------|-----------------|
| Column presence | `set(df.columns) == expected` |
| Column order | `df.columns.tolist() == expected_order` |
| dtype per column | `df.dtypes` |
| dtype family | `pd.api.types.is_*_dtype()` |
| Shape | `df.shape` |
| Index uniqueness | `df.index.is_unique` |

**Syntax Rules:**

- `set(df.columns) == expected_columns` ignores order and duplicates (duplicates in columns are rare but possible).
- `df.columns.tolist() == expected_order` checks exact order, which is important for positional indexing.
- `pd.api.types.is_numeric_dtype()` returns `True` for both integer and float dtypes.
- `is_string_dtype()` returns `True` for both `object` and `str` dtypes in pandas 2.x.

**Constraints and Limitations:**

- dtype checking may need to account for nullable extension types (e.g., `Int64` vs `int64`).
- Column order is not always relevant, but it matters for positional operations.
- `is_string_dtype()` may return `True` for columns containing mixed objects; consider checking for `object` dtype explicitly.

### Annotated Code Examples

**Example 1: Column Presence and dtype Validation**

```python
import pandas as pd
import numpy as np

# Sample DataFrame
df = pd.DataFrame({
    'user_id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [50000.0, 60000.0, 70000.0]
})

# Expected schema
expected_columns = {'user_id', 'name', 'age', 'salary'}
expected_dtypes = {
    'user_id': 'int64',
    'name': 'object',
    'age': 'int64',
    'salary': 'float64'
}

# Check column presence
missing_cols = expected_columns - set(df.columns)
extra_cols = set(df.columns) - expected_columns
print(f"Missing columns: {missing_cols}")
print(f"Extra columns: {extra_cols}")
# Expected output:
# Missing columns: set()
# Extra columns: set()

# Check dtypes
for col, expected_dtype in expected_dtypes.items():
    actual = str(df[col].dtype)
    if actual != expected_dtype:
        print(f"Mismatch in {col}: expected {expected_dtype}, got {actual}")
print("All dtypes match.")
# Expected output: All dtypes match.
```

**Why this output:** The `set` operations identify missing and extra columns. The dtype check compares each column's actual dtype against the expected string. Since the DataFrame was constructed with the correct types, no mismatches are reported.

**Example 2: Using pd.api.types for dtype Family Checking**

```python
# Check dtype families
numeric_cols = df.select_dtypes(include='number').columns.tolist()
print(f"Numeric columns: {numeric_cols}")
# Expected output: Numeric columns: ['user_id', 'age', 'salary']

# Check if a specific column is numeric
print(pd.api.types.is_numeric_dtype(df['age']))   # True
print(pd.api.types.is_string_dtype(df['name']))   # True
print(pd.api.types.is_numeric_dtype(df['name']))  # False
# Expected output:
# True
# True
# False
```

**Why this output:** `select_dtypes(include='number')` returns columns with numeric dtypes. `pd.api.types` functions provide programmatic dtype family checks that are more robust than string comparisons because they handle multiple dtypes within a family.

**Example 3: Structural Validation with Pandera**

```python
import pandera.pandas as pa
from pandera.typing import DataFrame, Series

# Define a schema
schema = pa.DataFrameSchema({
    "user_id": pa.Column(int, nullable=False, unique=True),
    "name": pa.Column(str, nullable=False),
    "age": pa.Column(int, pa.Check.in_range(0, 120)),
    "salary": pa.Column(float, pa.Check.ge(0)),
}, strict=True)  # strict=True disallows extra columns

# Validate
try:
    validated = schema.validate(df)
    print("Validation passed.")
except pa.errors.SchemaError as e:
    print(f"Validation failed: {e}")
# Expected output: Validation passed.
```

**Why this output:** The `pa.DataFrameSchema` declares the expected columns, their dtypes, and additional constraints. `strict=True` ensures that no extra columns are present. `schema.validate(df)` returns the validated DataFrame or raises `SchemaError` if any constraint is violated.

### Real-World Cases

- **ETL pipelines:** Validate that incoming CSV files have the expected columns and dtypes before loading into a database.
- **API responses:** Verify that a JSON response converted to a DataFrame has the expected structure before processing.
- **ML feature stores:** Ensure that feature DataFrames have the correct columns and dtypes before training.

### References

- pandas arrays, scalars, and data types — https://pandas.pydata.org/pandas-docs/version/2.2/reference/arrays.html
- pandas.api.types — https://pandas.pydata.org/docs/reference/api/pandas.api.types.is_numeric_dtype.html
- Pandera DataFrame Schemas — https://pandera.readthedocs.io/en/stable/dataframe_schemas.html
- daffy — Function decorators for Pandas and Polars dataframe column name and data type validation — https://pypi.org/project/daffy/


## 2. Domain Boundaries: Range Validation and Regular Expression Parsing

### Definitions

**Core Definition:** Domain boundary validation verifies that values fall within acceptable numerical ranges (min/max) or match expected string patterns using regular expressions.

**Technical Definition:** Range validation uses boolean masking (`series.between(min_val, max_val)`, `series >= min_val`, `series <= max_val`) to identify values outside acceptable bounds. Regular expression validation uses `Series.str.match(pattern, na=False)` to check whether each string conforms to a specified pattern. The `na=False` parameter treats missing values as non-matching, preventing errors. External libraries such as Pandera provide built-in checks like `pa.Check.in_range(min, max)`, `pa.Check.str_length(min, max)`, and `pa.Check.str_matches(pattern)`.

**Beginner-Friendly Explanation:** Domain boundaries are about what values are allowed. For numbers, you check that they fall between a minimum and a maximum (e.g., age between 0 and 120). For text, you check that strings follow a pattern (e.g., email addresses match `user@domain.com`).

### Purposes

- To verify that numerical values fall within acceptable minimum and maximum bounds.
- To validate that string values conform to expected patterns (emails, phone numbers, IDs).
- To check string length constraints (minimum and maximum length).
- To detect outliers or data entry errors that produce impossible values.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Range validation
series.between(left, right, inclusive='both')
series >= min_val
series <= max_val
(series >= min_val) & (series <= max_val)

# Regex validation
series.str.match(pattern, na=False)
series.str.contains(pattern, na=False)
series.str.fullmatch(pattern, na=False)

# Length validation
series.str.len() >= min_len
series.str.len() <= max_len
```

**Component Breakdown:**

| Check | Method |
|-------|--------|
| Numeric range | `Series.between()` or boolean comparison |
| Regex match (start) | `Series.str.match()` |
| Regex match (anywhere) | `Series.str.contains()` |
| Regex full match | `Series.str.fullmatch()` |
| String length | `Series.str.len()` |

**Syntax Rules:**

- `between(left, right, inclusive='both')` includes both endpoints by default; use `inclusive='neither'`, `'left'`, or `'right'` for exclusive bounds.
- `str.match()` matches from the beginning of the string; use `str.fullmatch()` for the entire string.
- `na=False` treats `NaN` values as `False` (non-matching), preventing `NaN` from propagating.
- Regex patterns should be raw strings (`r"pattern"`) to avoid escape sequence issues.

**Constraints and Limitations:**

- `str.match()` does not require the pattern to match the entire string unless anchored with `^` and `$`.
- Regex validation on large datasets can be slow; consider compiled patterns for repeated use.
- Range validation on nullable types (`Int64`, `Float64`) returns `pd.NA` for missing values, which may need separate handling.

### Annotated Code Examples

**Example 1: Numeric Range Validation**

```python
import pandas as pd

df = pd.DataFrame({
    'age': [25, 30, -5, 150, 35],
    'score': [0.5, 0.8, 1.2, 0.9, 0.7]
})

# Check age is between 0 and 120
age_valid = df['age'].between(0, 120, inclusive='both')
print(age_valid.tolist())
# Expected output: [True, True, False, False, True]

# Check score is between 0 and 1
score_valid = df['score'].between(0, 1, inclusive='both')
print(score_valid.tolist())
# Expected output: [True, True, False, True, True]

# Filter invalid rows
invalid_rows = df[~age_valid | ~score_valid]
print(invalid_rows)
# Expected output:
#    age  score
# 2   -5    1.2
# 3  150    0.9
```

**Why this output:** `between(0, 120)` returns `False` for ages -5 and 150, which are outside the acceptable range. `between(0, 1)` returns `False` for score 1.2. The combined mask identifies rows with at least one invalid value.

**Example 2: Regex Validation for Email Addresses**

```python
# Sample data with email addresses
emails = pd.Series([
    'alice@example.com',
    'bob.smith@company.org',
    'invalid-email',
    'charlie@',
    None
])

# Email regex pattern
pattern = r'^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'

# Validate
valid = emails.str.match(pattern, na=False)
print(valid.tolist())
# Expected output: [True, True, False, False, False]

# Filter valid emails
print(emails[valid].tolist())
# Expected output: ['alice@example.com', 'bob.smith@company.org']
```

**Why this output:** The pattern requires at least one alphanumeric character before `@`, a domain name, and a top-level domain of at least two letters. `invalid-email` lacks `@`; `charlie@` lacks a domain; `None` is treated as `False` due to `na=False`.

**Example 3: Pandera Range and Regex Checks**

```python
import pandera.pandas as pa

schema = pa.DataFrameSchema({
    "email": pa.Column(str, pa.Check.str_matches(
        r'^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'
    )),
    "age": pa.Column(int, pa.Check.in_range(0, 120)),
    "score": pa.Column(float, pa.Check.in_range(0.0, 1.0)),
})

try:
    validated = schema.validate(df)
    print("Validation passed.")
except pa.errors.SchemaError as e:
    print(f"Validation failed:\n{e}")
```

**Why this output:** Pandera's `Check.str_matches()` applies the regex to the entire string. `Check.in_range(min, max)` verifies that all values fall within the specified interval. If any check fails, a `SchemaError` is raised with details.

### Real-World Cases

- **Healthcare:** Validate that patient ages fall between 0 and 120 and that blood pressure readings are within physiological ranges.
- **E-commerce:** Validate that product prices are positive and that SKU codes match a specific alphanumeric pattern.
- **Financial services:** Validate that transaction amounts are within acceptable limits and that account numbers match a regex pattern.

### References

- pandas.Series.str.match — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.match.html
- pandas.Series.between — https://pandas.pydata.org/docs/reference/api/pandas.Series.between.html
- Pandera Checks — https://pandera.readthedocs.io/en/stable/checks.html
- Pandera Check Reference — https://huggingface.co/datasets/DavydenkoGr/AFTER/raw/main/skills/validation/references/pandera_checks_reference.md


## 3. Category Enforcement: Validating Categorical Membership

### Definitions

**Core Definition:** Category enforcement verifies that all values in a column belong to a predefined set of allowed categories, rejecting or flagging values that are not in the master list.

**Technical Definition:** Categorical membership validation can be performed by casting a column to `CategoricalDtype` with explicit `categories` — values not in the categories become `NaN`. Alternatively, `Series.isin(allowed_values)` returns a boolean mask indicating membership. The `CategoricalDtype` is fully described by two attributes: `categories` (a sequence of unique values with no missing values) and `ordered` (a boolean). A `CategoricalDtype` can be used in any place pandas expects a dtype, including `read_csv()`, `astype()`, or the Series constructor.

**Beginner-Friendly Explanation:** Category enforcement is like checking that every answer on a multiple-choice test is one of the allowed options. If a value is not in the list of allowed categories, it is flagged as invalid.

### Purposes

- To verify that all values in a column belong to a predefined set of allowed categories.
- To enforce a master definition of valid categories across multiple datasets.
- To detect data entry errors where values are misspelled or belong to an outdated category set.
- To enable ordered comparisons (e.g., “Low” < “Medium” < “High”) after validation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Membership check
series.isin(allowed_values)

# Categorical casting (values not in categories become NaN)
series.astype(pd.CategoricalDtype(categories=allowed, ordered=False))

# Detect invalid values
invalid_mask = ~series.isin(allowed_values)
invalid_values = series[invalid_mask].unique()
```

**Component Breakdown:**

| Check | Method |
|-------|--------|
| Membership | `Series.isin(values)` |
| Categorical casting | `Series.astype(CategoricalDtype(categories, ordered))` |
| Invalid detection | `~series.isin(allowed)` |
| Category accessor | `series.cat.categories` |

**Syntax Rules:**

- `isin()` accepts a list, set, Series, or dict of allowed values.
- When casting to `CategoricalDtype`, values not in `categories` become `NaN`.
- The `categories` sequence must contain unique values and no missing values.
- Unordered categoricals compare equal to each other regardless of the order of categories; ordered categoricals require matching order.

**Constraints and Limitations:**

- `isin()` does not distinguish between `NaN` and values not in the allowed set; both return `False`.
- Casting to `CategoricalDtype` silently converts invalid values to `NaN`, which may hide data quality issues if not checked separately.
- Category enforcement is only meaningful when the set of allowed values is known and finite.

### Annotated Code Examples

**Example 1: Membership Validation with isin()**

```python
import pandas as pd

# Sample data with valid and invalid categories
df = pd.DataFrame({
    'size': ['Small', 'Medium', 'Large', 'Extra-Large', 'small', None],
    'color': ['Red', 'Blue', 'Green', 'Yellow', 'Red', 'Purple']
})

allowed_sizes = ['Small', 'Medium', 'Large']
allowed_colors = ['Red', 'Green', 'Blue']

# Check size membership
size_valid = df['size'].isin(allowed_sizes)
print("Size valid:", size_valid.tolist())
# Expected output: Size valid: [True, True, True, False, False, False]

# Check color membership
color_valid = df['color'].isin(allowed_colors)
print("Color valid:", color_valid.tolist())
# Expected output: Color valid: [True, True, True, False, True, False]

# Identify invalid values
print("Invalid sizes:", df.loc[~size_valid, 'size'].unique().tolist())
# Expected output: Invalid sizes: ['Extra-Large', 'small', None]
```

**Why this output:** `isin()` returns `True` for values in the allowed list. `'Extra-Large'` is not in `['Small', 'Medium', 'Large']`; `'small'` is not in the list (case-sensitive); `None` is not in the list. Similarly, `'Yellow'` and `'Purple'` are not valid colors.

**Example 2: Categorical Casting for Validation**

```python
from pandas.api.types import CategoricalDtype

# Define master categories
cat_type = CategoricalDtype(categories=['Small', 'Medium', 'Large'], ordered=True)

# Cast — invalid values become NaN
df['size_validated'] = df['size'].astype(cat_type)
print(df[['size', 'size_validated']])
# Expected output:
#           size size_validated
# 0        Small          Small
# 1       Medium         Medium
# 2        Large          Large
# 3  Extra-Large            NaN
# 4        small            NaN
# 5         None            NaN
```

**Why this output:** Casting to `CategoricalDtype` converts values not in the categories to `NaN`. This makes invalid values visible as missing, but they are not deleted or flagged separately. The `ordered=True` flag enables logical comparisons.

**Example 3: Pandera Category Enforcement**

```python
import pandera.pandas as pa

schema = pa.DataFrameSchema({
    "size": pa.Column(str, pa.Check.isin(['Small', 'Medium', 'Large'])),
    "color": pa.Column(str, pa.Check.isin(['Red', 'Green', 'Blue'])),
})

try:
    validated = schema.validate(df)
    print("Validation passed.")
except pa.errors.SchemaError as e:
    print(f"Validation failed:\n{e}")
# Expected output: Validation failed: column 'size' contains values not in ['Small', 'Medium', 'Large']
```

**Why this output:** Pandera's `Check.isin(allowed_values)` verifies that all non-null values in the column are members of the allowed set. The schema raises `SchemaError` if any value is invalid.

### Real-World Cases

- **Product catalogs:** Validate that product categories (Electronics, Clothing, Food) belong to the master category list.
- **Survey data:** Validate that Likert-scale responses are one of the five allowed options.
- **Geographic data:** Validate that country codes belong to the ISO 3166 standard list.

### References

- pandas.Series.isin — https://pandas.pydata.org/docs/reference/api/pandas.Series.isin.html
- Categorical data — https://pandas.pydata.org/docs/user_guide/categorical.html
- pandas.CategoricalDtype — https://pandas.pydata.org/docs/reference/api/pandas.CategoricalDtype.html
- Pandera Checks — https://pandera.readthedocs.io/en/stable/checks.html


## 4. Relational Constraints: Uniqueness, Composite Keys, and Non-Nullable Fields

### Definitions

**Core Definition:** Relational constraints verify that values satisfy uniqueness requirements (single or composite keys), that required fields are not null, and that referential integrity holds between related DataFrames.

**Technical Definition:** Uniqueness validation uses `Series.is_unique` for single columns and `df.set_index(['col_A', 'col_B']).index.is_unique` for composite keys. The `df.duplicated(subset=['col_A', 'col_B']).any()` method provides an alternative check. Non-nullable field validation uses `Series.notna().all()` or `Series.isna().sum() == 0`. Referential integrity checks compare foreign keys in a child DataFrame against primary keys in a parent DataFrame. Pandera provides `nullable=False` and `unique=True` parameters in `pa.Column()` for declarative constraint definition.

**Beginner-Friendly Explanation:** Relational constraints are about the rules that connect rows and tables. Uniqueness means no two rows should have the same ID. Composite keys mean a combination of columns should be unique (e.g., product + warehouse). Non-nullable fields mean certain columns must always have a value. Referential integrity means that a foreign key in one table must exist as a primary key in another.

### Purposes

- To verify that a column contains no duplicate values (single-column uniqueness).
- To verify that a combination of columns forms a unique identifier (composite key).
- To ensure that critical fields contain no missing values (non-nullable constraints).
- To validate referential integrity between related DataFrames (foreign key constraints).
- To prevent downstream errors caused by duplicate keys or null values in required fields.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Single-column uniqueness
series.is_unique
series.duplicated().sum() == 0

# Composite key uniqueness
df.set_index(['col_A', 'col_B']).index.is_unique
df.duplicated(subset=['col_A', 'col_B']).any()  # True if duplicates exist

# Non-nullable field
series.notna().all()
series.isna().sum() == 0

# Referential integrity
child_df['foreign_key'].isin(parent_df['primary_key']).all()

# Pandera constraints
pa.Column(int, nullable=False, unique=True)
```

**Component Breakdown:**

| Constraint | Method |
|------------|--------|
| Single uniqueness | `Series.is_unique` |
| Composite uniqueness | `df.set_index([cols]).index.is_unique` |
| Non-nullable | `Series.notna().all()` |
| Referential integrity | `Series.isin(parent_keys).all()` |
| Pandera | `pa.Column(..., unique=True, nullable=False)` |

**Syntax Rules:**

- `Series.is_unique` returns a single boolean value (`True` if all values are unique).
- `df.set_index([cols]).index.is_unique` creates a MultiIndex and checks uniqueness across the combination.
- `df.duplicated(subset=[cols]).any()` returns `True` if any duplicate combination exists.
- For referential integrity, `isin()` returns `False` for values not found in the parent keys; `.all()` ensures every child key exists in the parent.

**Constraints and Limitations:**

- `is_unique` ignores `NaN` values; two `NaN` values are considered equal for uniqueness purposes in some contexts.
- `set_index()` creates a new DataFrame, which may be memory-intensive for large datasets; use `df.duplicated()` for a more memory-efficient check.
- Referential integrity with large parent tables can be slow; consider using `pd.merge(validate='many_to_one')` for built-in validation.

### Annotated Code Examples

**Example 1: Single-Column and Composite Key Uniqueness**

```python
import pandas as pd

# Sample DataFrame with a potential composite key
df = pd.DataFrame({
    'order_id': [1, 2, 3, 4, 5],
    'product_id': ['A', 'A', 'B', 'B', 'C'],
    'warehouse': ['W1', 'W2', 'W1', 'W1', 'W2'],
    'quantity': [10, 20, 30, 40, 50]
})

# Check single-column uniqueness
print(f"order_id is unique: {df['order_id'].is_unique}")
# Expected output: order_id is unique: True

# Check composite key uniqueness (product_id + warehouse)
print(f"product_id+warehouse is unique: {df.set_index(['product_id', 'warehouse']).index.is_unique}")
# Expected output: product_id+warehouse is unique: False

# Alternative: duplicated check
print(f"Duplicate product+warehouse: {df.duplicated(subset=['product_id', 'warehouse']).any()}")
# Expected output: Duplicate product+warehouse: True
```

**Why this output:** `order_id` has no duplicates, so `is_unique` is `True`. The combination of `product_id` and `warehouse` has a duplicate (`B` + `W1` appears twice), so the composite key check returns `False` and `duplicated().any()` returns `True`.

**Example 2: Non-Nullable Field Validation**

```python
# DataFrame with missing values in a required field
df2 = pd.DataFrame({
    'user_id': [1, 2, None, 4],
    'email': ['a@x.com', None, 'c@x.com', 'd@x.com']
})

# Check non-nullable constraint on user_id
user_id_not_null = df2['user_id'].notna().all()
print(f"user_id is fully non-null: {user_id_not_null}")
# Expected output: user_id is fully non-null: False

# Check non-nullable constraint on email
email_not_null = df2['email'].notna().all()
print(f"email is fully non-null: {email_not_null}")
# Expected output: email is fully non-null: False

# Count missing values
print(df2.isna().sum())
# Expected output:
# user_id    1
# email      1
# dtype: int64
```

**Why this output:** `notna().all()` returns `False` because each column has at least one missing value. The `isna().sum()` provides the exact count of missing values per column.

**Example 3: Pandera Relational Constraints**

```python
import pandera.pandas as pa

schema = pa.DataFrameSchema({
    "order_id": pa.Column(int, nullable=False, unique=True),
    "product_id": pa.Column(str, nullable=False),
    "warehouse": pa.Column(str, nullable=False),
})

# Validate
try:
    validated = schema.validate(df)
    print("Validation passed.")
except pa.errors.SchemaError as e:
    print(f"Validation failed:\n{e}")
# Expected output: Validation passed.
```

**Why this output:** Pandera's `nullable=False` ensures that `order_id`, `product_id`, and `warehouse` have no missing values. `unique=True` on `order_id` ensures no duplicate order IDs. If any constraint is violated, a `SchemaError` is raised.

### Real-World Cases

- **Order management:** Validate that `order_id` is unique and non-null before processing orders.
- **Inventory management:** Validate that the combination of `product_id` and `warehouse` is unique to prevent duplicate inventory records.
- **Database loading:** Validate foreign key constraints before loading data into a relational database.

### References

- pandas.Series.is_unique — https://pandas.pydata.org/docs/reference/api/pandas.Series.is_unique.html
- pandas.DataFrame.duplicated — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.duplicated.html
- Pandera DataFrame Schemas — https://pandera.readthedocs.io/en/stable/dataframe_schemas.html
- pandas.DataFrame.merge (validate parameter) — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.merge.html


## 5. Validation Ecosystems: Assertions, `.pipe()` Workflows, and External Libraries

### Definitions

**Core Definition:** Validation ecosystems combine native pandas assertions, the `.pipe()` method for chaining validation functions, and external libraries (Pandera, Great Expectations) to create maintainable, reusable data validation workflows.

**Technical Definition:** Native assertions use Python's `assert` statement to raise `AssertionError` when a validation condition fails. The `.pipe()` method applies a chain of functions to a DataFrame, enabling readable validation pipelines; note that `.pipe()` stops at the first failure unless error collection is implemented. Pandera provides a lightweight, Pythonic schema validation framework with object-based (`DataFrameSchema`) and class-based (`DataFrameModel`) APIs that integrate with pandas, Polars, PySpark, and Dask. Great Expectations (GX) provides declarative “expectations” with rich reporting and is suited for enterprise data platforms.

**Beginner-Friendly Explanation:** Instead of scattering validation checks throughout your code, you can organize them into a validation pipeline. You can use simple `assert` statements for quick checks, `.pipe()` to chain validations in a readable sequence, or external libraries like Pandera and Great Expectations for comprehensive, reusable schemas.

### Purposes

- To organise validation logic into reusable, testable functions.
- To chain multiple validation steps in a readable, maintainable workflow using `.pipe()`.
- To leverage declarative schemas (Pandera, Great Expectations) for comprehensive data quality assurance.
- To collect and report multiple validation errors instead of stopping at the first failure.
- To integrate validation into ETL/ML pipelines with monitoring and alerting.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Assertion-based validation
assert condition, "Error message"

# .pipe() chaining
(df
 .pipe(validate_columns)
 .pipe(validate_dtypes)
 .pipe(validate_ranges)
)

# Pandera object-based API
schema = pa.DataFrameSchema({...})
schema.validate(df)

# Pandera class-based API
class Schema(pa.DataFrameModel):
    col1: int = pa.Field(ge=0)
    col2: float = pa.Field(lt=10)

# Great Expectations
validator.expect_column_values_to_not_be_null("id")
result = validator.validate()
```

**Component Breakdown:**

| Approach | Key Method/Class |
|----------|------------------|
| Assertions | `assert condition, message` |
| `.pipe()` | `df.pipe(func, *args, **kwargs)` |
| Pandera object-based | `pa.DataFrameSchema({...})` |
| Pandera class-based | `pa.DataFrameModel` subclass |
| Great Expectations | `validator.expect_*(...)` |

**Syntax Rules:**

- `assert` statements raise `AssertionError` and stop execution immediately; they do not collect all errors.
- `.pipe()` passes the DataFrame as the first argument to the function; the function must return a DataFrame (or Series) to continue the chain.
- Pandera's `check_types()` decorator validates function inputs and outputs at runtime.
- Great Expectations' `validate()` returns a result object with `success` and detailed `results` attributes.

**Constraints and Limitations:**

- `assert` statements can be disabled with Python's `-O` flag; avoid using them for critical production validation.
- `.pipe()` stops at the first exception; for collect-all behaviour, build an error list inside the piped function.
- Pandera's `validate()` is synchronous and single-threaded; for large DataFrames (>1GB) in tight loops, validate at batch boundaries.
- Great Expectations has a steeper learning curve and is better suited for enterprise data platforms with dedicated data quality teams.

### Annotated Code Examples

**Example 1: Assertion-Based Validation**

```python
import pandas as pd

df = pd.DataFrame({
    'user_id': [1, 2, 3],
    'age': [25, 30, 35]
})

# Assert that required columns exist
assert set(['user_id', 'age']).issubset(df.columns), "Missing required columns"
print("Column check passed.")

# Assert that age is within range
assert df['age'].between(0, 120).all(), "Age out of range"
print("Age range check passed.")

# Assert that user_id is unique
assert df['user_id'].is_unique, "user_id is not unique"
print("Uniqueness check passed.")
# Expected output:
# Column check passed.
# Age range check passed.
# Uniqueness check passed.
```

**Why this output:** Each `assert` statement checks a condition and raises an `AssertionError` with the specified message if the condition is `False`. If all conditions are `True`, the print statements execute.

**Example 2: .pipe() Validation Workflow**

```python
import pandas as pd

def validate_columns(df):
    required = {'user_id', 'age'}
    missing = required - set(df.columns)
    if missing:
        raise ValueError(f"Missing columns: {missing}")
    return df

def validate_dtypes(df):
    expected = {'user_id': 'int64', 'age': 'int64'}
    for col, dtype in expected.items():
        if str(df[col].dtype) != dtype:
            raise ValueError(f"{col}: expected {dtype}, got {df[col].dtype}")
    return df

def validate_age_range(df):
    if not df['age'].between(0, 120).all():
        raise ValueError("Age out of range")
    return df

# Chain validations with .pipe()
result = (df
    .pipe(validate_columns)
    .pipe(validate_dtypes)
    .pipe(validate_age_range)
)
print("All validations passed.")
# Expected output: All validations passed.
```

**Why this output:** Each function receives the DataFrame, performs a specific validation, and returns the DataFrame (if valid) or raises an exception. `.pipe()` chains the functions in order, stopping at the first failure.

**Example 3: Pandera Class-Based Schema**

```python
import pandas as pd
import pandera.pandas as pa
from pandera.typing import DataFrame, Series

class UserSchema(pa.DataFrameModel):
    user_id: Series[int] = pa.Field(unique=True, nullable=False)
    age: Series[int] = pa.Field(ge=0, le=120)
    email: Series[str] = pa.Field(nullable=False)

    @pa.check("email")
    def email_format(cls, series: Series[str]) -> Series[bool]:
        return series.str.match(r'^[^@\s]+@[^@\s]+\.[^@\s]+$')

# Validate
try:
    validated = UserSchema.validate(df)
    print("Pandera validation passed.")
except pa.errors.SchemaError as e:
    print(f"Pandera validation failed:\n{e}")
```

**Why this output:** The `UserSchema` class defines column types and constraints. The `@pa.check` decorator adds a custom validation function for the email format. `UserSchema.validate(df)` checks all constraints and returns the validated DataFrame or raises `SchemaError`.

**Example 4: Great Expectations Validation**

```python
import great_expectations as gx
import pandas as pd

# Initialize context
context = gx.get_context()

# Create a validator from a pandas DataFrame
df = pd.DataFrame({
    'order_id': [1, 2, 3, 4, 5],
    'total': [10.0, 20.0, 30.0, 40.0, 50.0]
})

validator = context.sources.pandas_default.read_dataframe(df)

# Define expectations
validator.expect_column_values_to_not_be_null("order_id")
validator.expect_column_values_to_be_between("total", min_value=0, max_value=1000)

# Validate
result = validator.validate()
print(f"Validation success: {result.success}")
# Expected output: Validation success: True
```

**Why this output:** Great Expectations uses a validator object to apply expectations. `expect_column_values_to_not_be_null()` checks for missing values. `expect_column_values_to_be_between()` checks the range. `validate()` returns a result object with `success` indicating whether all expectations passed.

### Real-World Cases

- **ETL pipelines:** Use Pandera schemas to validate data at ingestion, transformation, and output boundaries.
- **ML pipelines:** Use `@pa.check_types` decorators to validate function inputs and outputs, preventing silent data corruption.
- **Enterprise data platforms:** Use Great Expectations for comprehensive data profiling, documentation, and validation with rich reporting.
- **Quick validation:** Use `.pipe()` with lightweight functions for rapid, readable validation in notebooks.

### References

- pandas.DataFrame.pipe — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pipe.html
- Pandera — https://pypi.org/project/pandera/
- Pandera DataFrame Schemas — https://pandera.readthedocs.io/en/stable/dataframe_schemas.html
- Great Expectations — https://docs.greatexpectations.io/
- Great Expectations Try GX Core — https://docs.greatexpectations.io/docs/core/connect_to_data/connect_to_data_overview
- daffy — Function decorators for Pandas and Polars dataframe column name and data type validation — https://pypi.org/project/daffy/
- data-agent-skills/skills/assuring-data-pipelines — https://github.com/legout/data-agent-skills/blob/main/skills/assuring-data-pipelines/SKILL.md
- Custom validation checks — Pandas For Ml Intermediate Course — https://theneuralbase.com/pandas-for-ml-intermediate/custom-validation-checks/


## References

- pandas arrays, scalars, and data types — https://pandas.pydata.org/pandas-docs/version/2.2/reference/arrays.html
- pandas.api.types — https://pandas.pydata.org/docs/reference/api/pandas.api.types.is_numeric_dtype.html
- pandas.Series.str.match — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.match.html
- pandas.Series.between — https://pandas.pydata.org/docs/reference/api/pandas.Series.between.html
- pandas.Series.isin — https://pandas.pydata.org/docs/reference/api/pandas.Series.isin.html
- pandas.Series.is_unique — https://pandas.pydata.org/docs/reference/api/pandas.Series.is_unique.html
- pandas.DataFrame.duplicated — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.duplicated.html
- pandas.DataFrame.pipe — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pipe.html
- pandas.CategoricalDtype — https://pandas.pydata.org/docs/reference/api/pandas.CategoricalDtype.html
- pandas.DataFrame.merge (validate parameter) — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.merge.html
- Pandera — https://pypi.org/project/pandera/
- Pandera DataFrame Schemas — https://pandera.readthedocs.io/en/stable/dataframe_schemas.html
- Pandera Checks — https://pandera.readthedocs.io/en/stable/checks.html
- Great Expectations — https://docs.greatexpectations.io/
- Great Expectations Try GX Core — https://docs.greatexpectations.io/docs/core/connect_to_data/connect_to_data_overview
- daffy — Function decorators for Pandas and Polars dataframe column name and data type validation — https://pypi.org/project/daffy/
- data-agent-skills/skills/assuring-data-pipelines — https://github.com/legout/data-agent-skills/blob/main/skills/assuring-data-pipelines/SKILL.md
- pandas-eda-check — https://pypi.org/project/pandas-eda-check/
- Custom validation checks — Pandas For Ml Intermediate Course — https://theneuralbase.com/pandas-for-ml-intermediate/custom-validation-checks/
- Pandera Check Reference — https://huggingface.co/datasets/DavydenkoGr/AFTER/raw/main/skills/validation/references/pandera_checks_reference.md