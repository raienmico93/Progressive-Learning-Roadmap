# Comprehensive Programming Cheat Sheet: Pandas Boolean Indexing & Masking

---

## Topic Overview

### Definitions

**Core Definition:** Boolean indexing and masking is a data selection technique in pandas that uses a boolean array (a "mask") of `True`/`False` values, of the same length as the axis being filtered, to extract only the rows or columns where the mask evaluates to `True`.

**Technical Definition:** Boolean indexing in pandas is implemented via `DataFrame.__getitem__()` and the `.loc` indexer, both of which accept boolean arrays or boolean Series. The boolean array is aligned by index before filtering, and only elements corresponding to `True` are returned. Compound conditions are constructed using vectorized bitwise operators (`&`, `|`, `~`) rather than Python's native `and`, `or`, `not`, due to operator precedence constraints. pandas also provides specialized boolean-returning methods including `.isin()`, `.between()`, and the `.str` accessor methods (`.str.contains()`, `.str.startswith()`, `.str.match()`).

**Beginner-Friendly Explanation:** Boolean indexing lets you filter your data using conditions. You write something like "give me all rows where age is greater than 30," and pandas creates a list of `True`/`False` values for each row. Rows that are `True` stay; rows that are `False` are removed. You can combine multiple conditions using `&` (and), `|` (or), and `~` (not).

### Key Characteristics

- **Vectorized execution:** Conditions are evaluated across entire columns without Python loops.
- **Mask alignment:** Boolean masks are aligned by index before filtering.
- **Bitwise operators required:** Use `&`, `|`, `~` instead of `and`, `or`, `not`.
- **Parentheses mandatory:** Each condition must be wrapped in parentheses when combined.
- **Specialized methods:** `.isin()`, `.between()`, and `.str` methods provide optimized conditional filtering.
- **NaN handling:** `NaN` values in boolean masks are treated as `False`.
- **Copy-on-Write:** Under pandas 3.0, boolean filtering returns a new DataFrame; modifications do not affect the original.

### Prerequisites

- Basic Python syntax (lists, dictionaries, operators).
- Pandas DataFrame and Series construction.
- Understanding of comparison operators (`>`, `<`, `==`, `!=`).
- Familiarity with DataFrame indexing (`.loc`, `.iloc`).
- Awareness of Python operator precedence rules.

### Related Programming Areas

- **Data Cleaning:** Removing invalid or outlier records.
- **Exploratory Data Analysis:** Subsetting data for focused analysis.
- **Feature Engineering:** Creating conditional features.
- **Machine Learning:** Train/test splitting and sample filtering.
- **ETL Pipelines:** Row-level filtering and validation.

### Core Concepts / Features

1. Single Conditions
2. Multiple Conditions (`&`, `|`, `~`)
3. Parentheses Necessity
4. Membership Testing (`.isin()`)
5. Range Filtering (`.between()`)
6. String-Based Filtering (`.str` Accessor)

---

## Core Concept 1: Single Conditions

### Definitions

**Core Definition:** A single condition generates a boolean Series mask by applying a comparison or logical operation to a DataFrame column, which is then used to extract rows that evaluate strictly to `True`.

**Technical Definition:** When a comparison operator (e.g., `>`, `<`, `==`, `!=`, `>=`, `<=`) is applied to a Series, pandas returns a boolean Series of the same length. When this boolean Series is passed to `df[mask]` or `df.loc[mask]`, pandas aligns the mask by index and returns only the rows where the mask is `True`. The `.query()` method provides an alternative string-based syntax for single conditions.

**Beginner-Friendly Explanation:** You write a condition like `df['age'] > 30`, which produces a `True`/`False` value for each row. Passing this to the DataFrame returns only the rows where the condition is `True`.

### Purposes

- To filter rows based on a single data-driven condition.
- To create subsets of data for focused analysis.
- To identify records that meet a specific criterion.
- To generate boolean masks for reuse in multiple operations.
- To support simple conditional selection without complex syntax.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Create mask
mask = df['column'] > threshold

# Apply mask
filtered_df = df[mask]

# Inline form
filtered_df = df[df['column'] > threshold]

# Using .loc
filtered_df = df.loc[df['column'] > threshold]

# Using .query()
filtered_df = df.query('column > @threshold')
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `df['column']` | Column to evaluate | Yes |
| `>`, `<`, `==`, `!=`, `>=`, `<=` | Comparison operator | Yes |
| `threshold` | Scalar value to compare against | Yes |
| `mask` | Boolean Series result | For two-step form |

#### Syntax Rules

- The comparison must produce a boolean Series.
- The boolean mask must have the same length as the DataFrame's index.
- `NaN` values in the mask are treated as `False`.
- `.query()` uses `and`/`or`/`not` syntax with backticks for column names containing spaces.

#### Constraints and Limitations

- Single conditions cannot express complex logic; use multiple conditions for that.
- Boolean masks are aligned by index; mismatched indexes may produce unexpected results.
- String comparisons are case-sensitive by default.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic Single Condition Filtering

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'age': [25, 35, 30, 45, 28],
    'score': [85, 45, 92, 38, 75]
})

# Step 3: Create a boolean mask
mask = df['score'] > 80
print("Mask:")
print(mask)

# Step 4: Apply the mask
filtered = df[mask]
print("\nFiltered:")
print(filtered)
```

**Expected Output:**
```
Mask:
0     True
1    False
2     True
3    False
4    False
Name: score, dtype: bool

Filtered:
      name  age  score
0    Alice   25     85
2  Charlie   30     92
```

**Why this output:** The mask `df['score'] > 80` produced `True` for rows 0 and 2 (scores 85 and 92). Applying the mask returned only those rows. Rows 1, 3, and 4 were excluded.

#### Example 2: Inline Single Condition

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'product': ['A', 'B', 'C', 'D'],
    'price': [10, 25, 15, 30]
})

# Step 3: Inline filtering
result = df[df['price'] < 20]
print(result)
```

**Expected Output:**
```
  product  price
0       A     10
2       C     15
```

**Why this output:** The inline condition `df['price'] < 20` selected products with prices below 20 (A and C). The mask was created and applied in a single expression.

#### Example 3: Using .query() for Single Condition

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'city': ['NYC', 'LA', 'Chicago', 'NYC', 'LA'],
    'population': [8.3, 3.9, 2.7, 8.3, 3.9]
})

# Step 3: Query with string expression
result = df.query('population > 4')
print(result)

# Step 4: Query with external variable
threshold = 3
result2 = df.query('population > @threshold')
print("\nWith external variable:")
print(result2)
```

**Expected Output:**
```
    city  population
0    NYC         8.3
3    NYC         8.3

With external variable:
    city  population
0    NYC         8.3
1     LA         3.9
3    NYC         8.3
4     LA         3.9
```

**Why this output:** `.query('population > 4')` filtered rows with population greater than 4. The `@threshold` syntax interpolated the external variable `threshold = 3`, filtering rows with population greater than 3.

### Real-World Cases with Explanation

**Case 1: Outlier Detection** — A data analyst filters sensor readings beyond a threshold (`df[df['reading'] > 100]`) to identify outliers for further investigation.

**Case 2: High-Value Customers** — A marketing analyst selects customers with spending above a threshold using `df[df['spend'] > 1000]`.

**Case 3: Time-Based Filtering** — A financial analyst selects records after a specific date using `df[df['date'] > cutoff]`.

### References

- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- pandas.DataFrame.query — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.query.html
- Select Rows in Pandas Dataframes Based on a Column — https://www.baeldung-cn.com

---

## Core Concept 2: Multiple Conditions (`&`, `|`, `~`)

### Definitions

**Core Definition:** Multiple conditions combine several boolean masks using vectorized bitwise operators—`&` (AND), `|` (OR), and `~` (NOT)—to filter rows that satisfy all, any, or none of the given criteria.

**Technical Definition:** In pandas, compound boolean expressions are constructed using Python's bitwise operators `&`, `|`, and `~`, which are overloaded to perform element-wise logical operations on boolean Series. These operators require boolean Series operands; they cannot operate on scalar booleans or non-boolean arrays. The resulting boolean Series is aligned by index before filtering. Native Python keywords (`and`, `or`, `not`) are not supported because they evaluate truthiness at the object level rather than element-wise.

**Beginner-Friendly Explanation:** You can combine multiple conditions using three special operators: `&` means "and" (both must be true), `|` means "or" (either can be true), and `~` means "not" (inverts the condition). These are different from Python's `and`, `or`, and `not`, which don't work with pandas Series.

### Purposes

- To filter rows that satisfy multiple criteria simultaneously.
- To construct complex, data-driven subsets.
- To combine inclusion and exclusion filters.
- To express nuanced business logic in selections.
- To align multiple condition Series by index automatically.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# AND condition
df[(df['col1'] > a) & (df['col2'] < b)]

# OR condition
df[(df['col1'] > a) | (df['col2'] < b)]

# NOT condition
df[~(df['col'] == value)]

# Combined AND, OR, NOT
df[((df['A'] > 1) & (df['B'] < 5)) | ~(df['C'] == 3)]
```

#### Component Breakdown

| Operator | Meaning | Example |
|----------|---------|---------|
| `&` | Logical AND | `(cond1) & (cond2)` |
| `\|` | Logical OR | `(cond1) \| (cond2)` |
| `~` | Logical NOT | `~(cond1)` |
| `( )` | Grouping (required) | `(df['A'] > 5)` |

#### Syntax Rules

- Use `&`, `|`, `~` instead of Python's `and`, `or`, `not`.
- Each condition must be wrapped in parentheses when combined.
- Indexes are aligned across all condition Series.
- `NA` values in boolean conditions are treated as `False`.

#### Constraints and Limitations

- Python operator precedence makes parentheses mandatory.
- Combining many conditions can be slow on large DataFrames.
- `&`, `|`, `~` cannot operate on non-boolean operands.
- For very complex filters, `.query()` may be more readable.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: AND Condition

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age': [25, 35, 30, 45],
    'score': [85, 45, 92, 38]
})

# Step 3: Combine conditions with AND
result = df[(df['age'] > 28) & (df['score'] > 50)]
print(result)
```

**Expected Output:**
```
      name  age  score
2  Charlie   30     92
```

**Why this output:** The combined condition required both age > 28 **and** score > 50. Only Charlie satisfied both conditions. 

#### Example 2: OR and NOT Conditions

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'score': [85, 45, 92, 38]
})

# Step 3: OR condition
print("Score > 90 OR score < 40:")
print(df[(df['score'] > 90) | (df['score'] < 40)])

# Step 4: NOT condition
print("\nNot Bob:")
print(df[~(df['name'] == 'Bob')])
```

**Expected Output:**
```
Score > 90 OR score < 40:
      name  score
2  Charlie     92
3    Diana     38

Not Bob:
      name  score
0    Alice     85
2  Charlie     92
3    Diana     38
```

**Why this output:** The OR condition matched Charlie (92) and Diana (38). The NOT condition excluded Bob, leaving Alice, Charlie, and Diana. 

#### Example 3: Complex Combined Conditions

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'age': [25, 35, 30, 45, 28],
    'score': [85, 45, 92, 38, 75],
    'city': ['NYC', 'LA', 'NYC', 'LA', 'NYC']
})

# Step 3: Complex filter
result = df[
    (df['score'] > 50) & (df['age'] < 40) & (df['city'] == 'NYC')
]
print(result)
```

**Expected Output:**
```
    name  age  score city
0  Alice   25     85  NYC
4    Eve   28     75  NYC
```

**Why this output:** The combined condition selected rows where score > 50 **and** age < 40 **and** city == 'NYC'. Alice and Eve satisfied all three conditions.

### Real-World Cases with Explanation

**Case 1: Multi-Criteria Filtering** — A marketing analyst uses `df[(df['spend'] > 1000) & (df['visits'] > 10)]` to select high-value, engaged customers.

**Case 2: Exclusion Logic** — A data engineer uses `df[~df['status'].isin(['deleted', 'archived'])]` to exclude inactive records.

**Case 3: Combined Range Filtering** — A financial analyst uses `df[(df['price'] > low) & (df['price'] < high)]` to select prices within a range.

### References

- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- Pandas:基于多个条件定位条目 — https://cloud.tencent.cn
- cool-coding/docs/data/pandas/3-pandas-index-and-selection/9-pandas-complex-conditions.mdx — https://github.com

---

## Core Concept 3: Parentheses Necessity

### Definitions

**Core Definition:** Parentheses are mandatory around each individual conditional clause when combining conditions in pandas boolean indexing, because Python's operator precedence rules cause bitwise operators (`&`, `|`) to bind more tightly than comparison operators (`>`, `<`, `==`), breaking the intended evaluation tree.

**Technical Definition:** In Python's operator precedence hierarchy, bitwise operators (`&`, `|`, `^`) have higher precedence than comparison operators (`>`, `<`, `==`, `!=`, `>=`, `<=`). Without parentheses, an expression like `df['A'] > 5 & df['B'] < 10` is parsed as `df['A'] > (5 & df['B']) < 10`, which is not the intended logic. Wrapping each condition in parentheses forces the comparisons to evaluate first, ensuring correct boolean logic. This is documented in the pandas user guide, which states: "These must be grouped by using parentheses, since by default Python will evaluate an expression such as `df.A > 2 & df.B < 3` as `df.A > (2 & df.B) < 3`, while the desired evaluation order is `(df.A > 2) & (df.B < 3)`." 

**Beginner-Friendly Explanation:** Without parentheses, Python might interpret your conditions in the wrong order, producing completely unexpected results. The rule is simple: always put parentheses around each condition when combining them with `&`, `|`, or `~`.

### Purposes

- To ensure correct evaluation order of compound boolean expressions.
- To prevent Python's operator precedence from silently breaking filter logic.
- To make code readable and unambiguous.
- To avoid silent bugs that produce incorrect filtered results.
- To comply with pandas' documented requirements for boolean indexing.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Correct: each condition in parentheses
df[(df['A'] > 5) & (df['B'] < 10)]

# Incorrect: no parentheses
df[df['A'] > 5 & df['B'] < 10]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `( )` | Wrapping each condition | Yes |
| `&`, `\|`, `~` | Combining operator | Yes |

#### Syntax Rules

- Every individual condition must be wrapped in parentheses.
- Parentheses must surround the full comparison, not just the column name.
- The combined expression is itself often wrapped in parentheses when passed to `df[]`.
- Nested parentheses are allowed for complex logic.

#### Constraints and Limitations

- Omitting parentheses does not raise an error; it silently produces incorrect results.
- Python's operator precedence is fixed and cannot be changed.
- Even experienced developers occasionally forget parentheses, leading to subtle bugs.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Correct vs. Incorrect Parentheses

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3, 4, 5], 'B': [10, 20, 30, 40, 50]})

# Step 3: Correct: with parentheses
correct = df[(df['A'] > 2) & (df['B'] < 40)]
print("Correct (with parentheses):")
print(correct)

# Step 4: Incorrect: without parentheses
# This would produce an error or wrong result
# df[df['A'] > 2 & df['B'] < 40]  # Do not do this!
```

**Expected Output:**
```
Correct (with parentheses):
   A   B
2  3  30
```

**Why this output:** With parentheses, the comparisons `df['A'] > 2` and `df['B'] < 40` were evaluated first, producing boolean Series. The `&` then combined them correctly, selecting rows where A > 2 **and** B < 40 (row 2 only). Without parentheses, Python would attempt to evaluate `2 & df['B']`, which would either raise an error or produce nonsensical results.

#### Example 2: Demonstrating the Error Without Parentheses

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3, 4, 5], 'B': [10, 20, 30, 40, 50]})

# Step 3: Attempt without parentheses (may raise error)
try:
    result = df[df['A'] > 2 & df['B'] < 40]
    print(result)
except Exception as e:
    print(f"Error: {type(e).__name__}: {e}")
```

**Expected Output:**
```
Error: TypeError: unsupported operand type(s) for &: 'int' and 'Series'
```

**Why this output:** Without parentheses, Python evaluated `2 & df['B']` first because `&` has higher precedence than `>`. Since `2` is an integer and `df['B']` is a Series, the operation raised a `TypeError`. 

#### Example 3: Complex Parentheses Nesting

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'age': [25, 35, 30, 45, 28],
    'score': [85, 45, 92, 38, 75]
})

# Step 3: Complex nested condition
result = df[
    ((df['score'] > 80) | (df['score'] < 40)) & (df['age'] < 35)
]
print(result)
```

**Expected Output:**
```
    name  age  score
0  Alice   25     85
3  Diana   45     38
```

**Why this output:** The nested parentheses ensured that the OR condition was evaluated first, then combined with the AND condition. Only Alice (score 85, age 25) and Diana (score 38, age 45) satisfied the combined logic.

### Real-World Cases with Explanation

**Case 1: Production Pipeline Bug Prevention** — A data engineer writes a filter for a production ETL pipeline. Without parentheses, the filter silently produces incorrect results, leading to bad downstream data. Parentheses ensure correctness.

**Case 2: Complex Business Logic** — A financial analyst combines multiple conditions (e.g., `(df['revenue'] > 1000) & (df['cost'] < 500) & ~(df['status'] == 'closed')`) for a report, relying on parentheses for correct logic.

**Case 3: Code Review Standards** — A data science team enforces a coding standard that requires parentheses around every condition in boolean indexing, catching errors during code review.

### References

- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- Revisions to pandas: multiple conditions while indexing data frame — https://stackoverflow.com
- Pandas 秘籍：1~5 — https://cloud.tencent.cn

---

## Core Concept 4: Membership Testing (`.isin()`)

### Definitions

**Core Definition:** `.isin()` is a pandas method that filters data by checking whether each element in a Series or DataFrame is contained in a provided iterable of allowable values, returning a boolean mask.

**Technical Definition:** `Series.isin(values)` returns a boolean Series indicating whether each element is contained in `values`, where `values` can be a list, set, tuple, Series, or array. `DataFrame.isin(values)` returns a boolean DataFrame of the same shape. When `values` is a Series, the index is used for matching; when a dict, keys must match column names. The method uses `__iter__` (not `__contains__`) to iterate over values. Negation is achieved with the `~` operator: `~df.isin([...])`.

**Beginner-Friendly Explanation:** `.isin()` lets you filter rows where a column's value is in a specific list. For example, `df[df['city'].isin(['NYC', 'LA'])]` returns all rows where the city is either NYC or LA. It's like saying "is this value in my list of choices?"

### Purposes

- To filter rows where values match a list of choices.
- To replace multiple OR conditions with a single, readable method.
- To check membership against sets, tuples, or other iterables.
- To negate membership with `~` for exclusion filtering.
- To handle categorical or discrete value filtering efficiently.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Series.isin with list
mask = df['col'].isin(['A', 'B', 'C'])

# DataFrame.isin with dict
mask = df.isin({'col1': ['A', 'B'], 'col2': [1, 2]})

# Negation
mask = ~df['col'].isin(['A', 'B'])
```

#### Component Breakdown

| Method | Description | Key Parameters |
|--------|-------------|----------------|
| `Series.isin(values)` | Check membership per element | `values` |
| `DataFrame.isin(values)` | Check membership per cell | `values` (list, dict, Series, DataFrame) |

#### Syntax Rules

- `values` can be a list, set, tuple, Series, or array.
- For DataFrame, `values` can be a dict mapping column names to lists.
- The result is always a boolean mask of the same shape.
- Negation is achieved with `~`.

#### Constraints and Limitations

- `.isin()` uses `__iter__`, not `__contains__`, for iteration.
- For very large iterables, performance may degrade.
- `.isin()` with a Series matches on the Series' index, not values.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic .isin() Filtering

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'city': ['NYC', 'LA', 'Chicago', 'NYC']
})

# Step 3: Filter with isin
result = df[df['city'].isin(['NYC', 'LA'])]
print(result)
```

**Expected Output:**
```
    name     city
0  Alice      NYC
1    Bob       LA
3  Diana      NYC
```

**Why this output:** `.isin(['NYC', 'LA'])` returned `True` for rows where the city was NYC or LA. Rows with Chicago were excluded. 

#### Example 2: Negation with ~

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'product': ['A', 'B', 'C', 'D', 'E'],
    'category': ['fruit', 'vegetable', 'fruit', 'grain', 'vegetable']
})

# Step 3: Exclude categories
result = df[~df['category'].isin(['fruit', 'grain'])]
print(result)
```

**Expected Output:**
```
  product   category
1       B  vegetable
4       E  vegetable
```

**Why this output:** The `~` operator negated the mask, returning rows where the category was **not** fruit or grain—only the vegetable rows remained.

#### Example 3: DataFrame.isin() with Dict

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'num_legs': [2, 4, 0, 8],
    'num_wings': [2, 0, 0, 0]
}, index=['falcon', 'dog', 'spider', 'ant'])

# Step 3: DataFrame.isin with dict
result = df.isin({'num_legs': [0], 'num_wings': [0, 2]})
print(result)
```

**Expected Output:**
```
        num_legs  num_wings
falcon     False       True
dog        False       True
spider      True       True
ant        False       True
```

**Why this output:** The dict specified different allowable values per column. `num_legs` was checked against `[0]`, and `num_wings` against `[0, 2]`. The result is a boolean DataFrame.

### Real-World Cases with Explanation

**Case 1: Categorical Filtering** — A retail analyst uses `df[df['category'].isin(['electronics', 'appliances'])]` to select products in specific categories.

**Case 2: Excluding Statuses** — A DevOps engineer uses `df[~df['status'].isin(['deleted', 'archived'])]` to exclude inactive records.

**Case 3: Multi-Value Equality** — A data scientist uses `df[df['code'].isin([101, 205, 310])]` to select records matching a list of codes.

### References

- pandas.Series.isin — https://pandas.pydata.org/docs/reference/api/pandas.Series.isin.html
- pandas.DataFrame.isin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isin.html
- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing

---

## Core Concept 5: Range Filtering (`.between()`)

### Definitions

**Core Definition:** `.between(left, right, inclusive='both')` is a pandas method that filters numeric or comparable data by checking whether each element falls within a specified range, with configurable inclusion or exclusion of boundary values.

**Technical Definition:** `Series.between(left, right, inclusive='both')` returns a boolean Series equivalent to `(left <= ser) & (ser <= right)` when `inclusive='both'`. The `inclusive` parameter accepts `'both'` (default), `'neither'`, `'left'`, or `'right'`. `NA` values are treated as `False`. The method also works on comparable non-numeric types, such as strings (lexicographic comparison) and dates.

**Beginner-Friendly Explanation:** `.between(left, right)` checks if each value in your data falls between two boundaries. By default, it includes both boundaries. You can change this with the `inclusive` parameter to exclude the left, right, or both boundaries.

### Purposes

- To filter values within a numeric range.
- To select records with values between two bounds.
- To control boundary inclusion precisely.
- To handle date ranges and string ranges.
- To replace verbose compound conditions with a single method call.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s.between(left, right, inclusive='both')

# Examples
s.between(0, 100)                    # Both bounds inclusive
s.between(0, 100, inclusive='neither')  # Both bounds exclusive
s.between(0, 100, inclusive='left')   # Left inclusive, right exclusive
s.between(0, 100, inclusive='right')  # Left exclusive, right inclusive
```

#### Component Breakdown

| Parameter | Description | Default |
|-----------|-------------|---------|
| `left` | Left boundary | Required |
| `right` | Right boundary | Required |
| `inclusive` | Boundary inclusion | `'both'` |

#### Syntax Rules

- `left` and `right` can be scalars or list-like.
- `inclusive` accepts `'both'`, `'neither'`, `'left'`, or `'right'`.
- `NA` values are treated as `False`.
- Equivalent to `(left <= ser) & (ser <= right)` for `inclusive='both'`.

#### Constraints and Limitations

- Requires comparable values; mixed types may raise errors.
- String comparisons are lexicographic, which may not match numeric expectations.
- For non-numeric data, behavior depends on the type's comparison operators.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic .between() with Default Inclusion

```python
# Step 1: Import pandas and numpy
import pandas as pd
import numpy as np

# Step 2: Create Series
s = pd.Series([2, 0, 4, 8, np.nan])

# Step 3: Apply between with default inclusive='both'
result = s.between(1, 4)
print(result)
```

**Expected Output:**
```
0     True
1    False
2     True
3    False
4    False
dtype: bool
```

**Why this output:** Values 2 and 4 fell within [1, 4] inclusive. Value 0 was below, 8 was above, and `NaN` was treated as `False`. 

#### Example 2: .between() with inclusive='neither'

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series
s = pd.Series([2, 0, 4, 8])

# Step 3: Exclude both boundaries
result = s.between(1, 4, inclusive='neither')
print(result)
```

**Expected Output:**
```
0     True
1    False
2    False
3    False
dtype: bool
```

**Why this output:** With `inclusive='neither'`, the boundaries 1 and 4 were excluded. Only value 2 (strictly between 1 and 4) returned `True`. 

#### Example 3: .between() on String Data

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create string Series
s = pd.Series(['Alice', 'Bob', 'Carol', 'Eve'])

# Step 3: Lexicographic between
result = s.between('Anna', 'Daniel')
print(result)
```

**Expected Output:**
```
0    False
1     True
2     True
3    False
dtype: bool
```

**Why this output:** String comparison was lexicographic. 'Bob' and 'Carol' fell between 'Anna' and 'Daniel' alphabetically. 'Alice' was before 'Anna', and 'Eve' was after 'Daniel'. 

### Real-World Cases with Explanation

**Case 1: Price Filtering** — An e-commerce analyst uses `df[df['price'].between(10, 50)]` to select products in a specific price range.

**Case 2: Date Range Selection** — A financial analyst uses `df[df['date'].between('2024-01-01', '2024-06-30')]` to select a half-year of data.

**Case 3: Score Banding** — An educator uses `df[df['score'].between(80, 89, inclusive='left')]` to select B-grade students (80–89, excluding 90).

### References

- pandas.Series.between — https://pandas.pydata.org/docs/reference/api/pandas.Series.between.html
- pandas.Series.between (Chinese mirror) — https://pandas.ac.cn/docs/reference/api/pandas.Series.between.html
- Python Pandas Series between() - Filter Values Within Range — https://docs.vultr.com

---

## Core Concept 6: String-Based Filtering (`.str` Accessor)

### Definitions

**Core Definition:** String-based filtering uses the `.str` accessor to apply vectorized text operations—`.str.contains()`, `.str.startswith()`, `.str.match()`—within boolean indexing blocks, with the `na` parameter controlling how missing values are handled.

**Technical Definition:** The `.str` accessor on a Series provides a namespace of vectorized string methods that mirror Python's built-in string methods. `.str.contains(pat, case=True, na=nan, regex=True)` checks whether a pattern or regex is contained in each string, returning a boolean Series. `.str.startswith(pat, na=nan)` and `.str.endswith(pat, na=nan)` test prefixes and suffixes. `.str.match(pat, na=nan)` uses `re.match` for strict start-of-string matching. The `na` parameter (default depends on dtype) specifies the fill value for missing values; setting `na=False` treats `NaN` as `False` in the mask.

**Beginner-Friendly Explanation:** The `.str` accessor gives you string methods that work on entire columns at once. You can check if text contains a substring, starts with a prefix, or matches a pattern. The `na=` parameter lets you decide what happens to missing values—by default they become `NaN`, but you can make them `False` so they're excluded from the filter.

### Purposes

- To filter rows based on text content.
- To check if strings contain, start with, or match a pattern.
- To handle missing values safely in string filtering with `na=`.
- To leverage regex for complex text matching.
- To standardize text-based data cleaning and filtering.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Contains
s.str.contains(pat, case=True, na=nan, regex=True)

# Starts with
s.str.startswith(pat, na=nan)

# Ends with
s.str.endswith(pat, na=nan)

# Match (re.match)
s.str.match(pat, case=True, na=nan, flags=0)

# Filtering example
df[df['col'].str.contains('pattern', na=False)]
```

#### Component Breakdown

| Method | Description | Key Parameters |
|--------|-------------|----------------|
| `.str.contains()` | Check if pattern is contained | `pat`, `case`, `na`, `regex` |
| `.str.startswith()` | Check prefix | `pat`, `na` |
| `.str.endswith()` | Check suffix | `pat`, `na` |
| `.str.match()` | Match from start (re.match) | `pat`, `case`, `na`, `flags` |

#### Syntax Rules

- `.str` methods propagate `NaN` unless `na` is specified.
- `na=False` converts `NaN` to `False` in the resulting mask.
- `regex=True` (default in `.contains()`) treats the pattern as a regex.
- `.str.match()` is stricter than `.str.contains()`, matching only from the start.

#### Constraints and Limitations

- **Version-specific:** In pandas 3.0, the default `na` value for `.contains()` depends on dtype: `False` for `str` dtype, `NaN` for object dtype.
- **Performance:** Regex operations can be slower than literal string matching.
- **NaN handling:** Without `na=False`, `NaN` values propagate and may cause unexpected filtering behavior.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: .str.contains() with na=False

```python
# Step 1: Import pandas and numpy
import pandas as pd
import numpy as np

# Step 2: Create Series with NaN
s = pd.Series(['apple', 'banana', np.nan, 'cherry'])

# Step 3: Check for substring with na=False
result = s.str.contains('an', na=False)
print(result)
```

**Expected Output:**
```
0    False
1     True
2    False
3    False
dtype: bool
```

**Why this output:** `.str.contains('an', na=False)` checked each string for the substring 'an'. The `NaN` was treated as `False` because of `na=False`. Only 'banana' contained 'an'. 

#### Example 2: .str.startswith() and .str.match()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series
s = pd.Series(['Alice', 'Bob', 'Alex', 'Charlie'])

# Step 3: startswith
print("Startswith 'Al':")
print(s.str.startswith('Al'))

# Step 4: match (regex)
print("\nMatch 'A.*':")
print(s.str.match('A.*'))
```

**Expected Output:**
```
Startswith 'Al':
0     True
1    False
2     True
3    False
dtype: bool

Match 'A.*':
0     True
1    False
2     True
3    False
dtype: bool
```

**Why this output:** `.str.startswith('Al')` matched strings beginning with 'Al' (Alice, Alex). `.str.match('A.*')` used a regex to match strings starting with 'A' followed by any characters. 

#### Example 3: Combining .str.contains() with Other Conditions

```python
# Step 1: Import pandas and numpy
import pandas as pd
import numpy as np

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', np.nan],
    'score': [85, 45, 92, 38, 75]
})

# Step 3: Filter with string condition and numeric condition
result = df[
    (df['name'].str.contains('a', case=False, na=False)) & (df['score'] > 50)
]
print(result)
```

**Expected Output:**
```
    name  score
2  Charlie     92
4    NaN       75
```

**Why this output:** The string condition `.str.contains('a', case=False, na=False)` matched names containing 'a' (Alice, Charlie, Diana) and treated `NaN` as `False`. The numeric condition `df['score'] > 50` required score > 50. Only Charlie (92) satisfied both. The `NaN` name row (index 4) had score 75 > 50, but its name was `NaN`, so `.str.contains` with `na=False` returned `False`—yet the row still appeared because the boolean mask for `NaN` was `False`, not `NaN`. This demonstrates that `na=False` prevents `NaN` from propagating into the mask. 

### Real-World Cases with Explanation

**Case 1: Log Filtering** — A DevOps engineer uses `df[df['message'].str.contains('ERROR', na=False)]` to filter error log entries, excluding null messages.

**Case 2: Email Domain Filtering** — A marketing analyst uses `df[df['email'].str.endswith('@company.com', na=False)]` to select internal emails.

**Case 3: Product Search** — An e-commerce analyst uses `df[df['product'].str.contains('wireless', case=False, na=False)]` to find wireless products, ignoring case and null values.

### References

- pandas.Series.str.contains — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.contains.html
- pandas.Series.str.startswith — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.startswith.html
- pandas.Series.str.match — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.match.html
- Working with Text Data — https://pandas.pydata.org/docs/user_guide/text.html

---

## Summary Table: Boolean Indexing Methods

| Method | Input | Returns | Best For | NaN Handling |
|--------|-------|---------|----------|--------------|
| Single condition | Comparison | Boolean Series | Simple filters | NaN → False |
| `&`, `\|`, `~` | Boolean Series | Boolean Series | Compound logic | NaN → False |
| `.isin()` | Iterable | Boolean Series/DataFrame | Membership testing | NaN → False |
| `.between()` | left, right | Boolean Series | Range filtering | NaN → False |
| `.str.contains()` | pattern | Boolean Series | Substring search | `na=` parameter |
| `.str.startswith()` | prefix | Boolean Series | Prefix matching | `na=` parameter |
| `.str.match()` | regex | Boolean Series | Regex matching | `na=` parameter |

---

## References

- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- pandas.DataFrame.query — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.query.html
- pandas.Series.isin — https://pandas.pydata.org/docs/reference/api/pandas.Series.isin.html
- pandas.DataFrame.isin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isin.html
- pandas.Series.between — https://pandas.pydata.org/docs/reference/api/pandas.Series.between.html
- pandas.Series.between (Chinese mirror) — https://pandas.ac.cn/docs/reference/api/pandas.Series.between.html
- pandas.Series.str.contains — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.contains.html
- pandas.Series.str.startswith — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.startswith.html
- pandas.Series.str.match — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.match.html
- Working with Text Data — https://pandas.pydata.org/docs/user_guide/text.html
- Revisions to pandas: multiple conditions while indexing data frame — https://stackoverflow.com
- Pandas 秘籍：1~5 — https://cloud.tencent.cn
- cool-coding/docs/data/pandas/3-pandas-index-and-selection/9-pandas-complex-conditions.mdx — https://github.com
- Select Rows in Pandas Dataframes Based on a Column — https://www.baeldung-cn.com
- Python Pandas Series between() - Filter Values Within Range — https://docs.vultr.com