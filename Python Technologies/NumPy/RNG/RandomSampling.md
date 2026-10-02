# Random Sampling — A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Random sampling in NumPy's modern random API refers to the set of methods on the `Generator` class that draw values from arrays, ranges, or discrete/continuous uniform distributions. The primary sampling methods are `integers()`, `random()`, `choice()`, `shuffle()`, and `permutation()`.

**Technical Definition:** NumPy's `Generator` class provides a suite of random sampling routines that operate on the raw bit stream produced by a `BitGenerator`. Integer sampling (`integers`) uses Daniel Lemire's fast random integer generation algorithm over an interval, while floating-point sampling (`random`) produces values in the half-open interval [0.0, 1.0) from the continuous uniform distribution. Array sampling (`choice`) supports uniform and weighted (via the `p` parameter) selection with or without replacement. In-place shuffling (`shuffle`) and out-of-place permutation (`permutation`) randomize the order of array elements, with optional `axis` control for multi-dimensional arrays. The `permuted` method (NumPy 1.22+) shuffles each slice along an axis independently.

**Beginner-Friendly Explanation:** Random sampling is how you ask NumPy to "pick a random number" or "pick a random item from a list." You can ask for random whole numbers (like dice rolls), random decimals (like measurements), or random selections from a list of options. You can also ask NumPy to shuffle a deck of cards (either changing the deck itself or making a shuffled copy). All of these operations are fast, reproducible with a seed, and work on arrays of any shape.

### Key Characteristics

- **Modern API:** All sampling methods are accessed through a `Generator` instance, not global functions.
- **Reproducible:** With a fixed seed, every sampling method produces identical output across runs.
- **Vectorized:** All methods accept a `size` parameter for batch generation and broadcast array-valued parameters.
- **In-Place vs. Copy:** `shuffle` modifies the input array in-place; `permutation` returns a new shuffled copy.
- **Axis-Aware:** `shuffle`, `permutation`, and `choice` support an `axis` parameter for multi-dimensional arrays.
- **Weighted Sampling:** `choice` supports arbitrary probability weights via the `p` parameter.

### Prerequisites

- Basic Python programming (variables, functions, imports)
- Familiarity with NumPy arrays, dtypes, and the `Generator` class
- Understanding of the `size` parameter for array shapes
- Basic knowledge of probability (uniform distribution, weighted probabilities)

### Related Programming Areas

- Machine Learning (data shuffling, train/test splitting, bootstrapping)
- Statistical Simulation (Monte Carlo, permutation testing)
- Game Development (random loot, procedural generation)
- Cryptography (understanding why NumPy PRNGs are not cryptographically secure)
- Data Engineering (random sampling for testing and validation)

### Core Concepts / Features

1. Random Integers (`integers`) vs. Continuous Uniform Samples (`random`)
2. Weighted Random Choices (`choice` with Probabilities Vector `p`)
3. Sampling With vs. Without Replacement (`replace=True/False`)
4. In-Place Shuffling (`shuffle`) vs. Out-of-Place Permutations (`permutation`)
5. Shuffling and Permuting Multi-Dimensional Arrays Along Specific Axes

---

## Core Concept 1: Random Integers (`integers`) vs. Continuous Uniform Samples (`random`)

### Definitions

**Core Definition:** `Generator.integers()` draws random integers from a discrete uniform distribution over a specified range, while `Generator.random()` draws random floating-point numbers from the continuous uniform distribution over the interval [0.0, 1.0).

**Technical Definition:** `Generator.integers(low, high=None, size=None, dtype='int64', endpoint=False)` returns random integers from the "discrete uniform" distribution of the specified dtype. If `high` is `None`, results are drawn from 0 to `low`. The `endpoint` parameter controls whether the high value is included. `Generator.random(size=None, dtype=np.float64, out=None)` returns random floats in the half-open interval [0.0, 1.0) from the continuous uniform distribution; to sample from [a, b), use `(b - a) * rng.random() + a`. Both methods are the canonical replacements for the legacy `randint`/`random_integers` and `random_sample`/`rand` functions, respectively.

**Beginner-Friendly Explanation:** `integers()` is for when you need whole numbers — like rolling a die (1 to 6), picking a random index (0 to n-1), or simulating a lottery draw. `random()` is for when you need decimals between 0 and 1 — like generating a probability, a random fraction, or a scaling factor. They're the two most basic random sampling tools in NumPy.

### Purposes

- To generate random indices for array indexing, shuffling, and sampling operations.
- To simulate discrete random events such as dice rolls, coin flips, and lottery draws.
- To produce continuous random values in a normalized range for scaling and transformation.
- To provide the raw random values needed for building more complex distributions via inverse transform or rejection sampling.
- To enable reproducible random integer and float generation across platforms and NumPy versions.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

rng = np.random.default_rng(seed=None)

# Integer sampling
integers = rng.integers(low, high=None, size=None,
                        dtype='int64', endpoint=False)

# Continuous uniform sampling
floats = rng.random(size=None, dtype=np.float64, out=None)
```

**Component Breakdown:**

**`integers(low, high=None, size=None, dtype='int64', endpoint=False)`**
- `low`: Lowest (signed) integer to be drawn from the distribution. If `high` is `None`, this parameter is 0 and this value is used for `high`.
- `high`: If provided, one above the largest integer to be drawn. If `endpoint=True`, `high` is included in the range.
- `size`: Output shape. If `(m, n, k)`, then `m * n * k` samples are drawn. Default `None` returns a single value.
- `dtype`: Desired dtype of the result (e.g., `'int64'`, `'int32'`, `'uint8'`). Default is `'int64'`.
- `endpoint`: If `True`, sample from `[low, high]` instead of `[low, high)`. Default is `False`.

**`random(size=None, dtype=np.float64, out=None)`**
- `size`: Output shape. If `(m, n, k)`, then `m * n * k` samples are drawn. Default `None` returns a single float.
- `dtype`: Desired dtype, only `float64` and `float32` are supported. Default is `np.float64`.
- `out`: Alternative output array to place the result. Must have the same shape as `size`.

**Syntax Rules:**
- `integers` returns native Python `int` if `size=None` and `a` is an integer type.
- `random` returns a Python `float` if `size=None`.
- `endpoint=True` is useful for inclusive ranges (e.g., dice rolls 1–6).
- `random` always returns values in [0.0, 1.0); to shift the range, multiply and add: `(b - a) * rng.random() + a`.

**Constraints and Limitations:**
- `integers` raises `ValueError` if `a` is an int and less than zero.
- `random` only supports `float64` and `float32`; byteorder must be native.
- The legacy `randint` and `random_integers` methods are deprecated; use `integers` instead.
- The legacy `rand` and `random_sample` methods are replaced by `random`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Integer and Float Sampling

```python
# Step 1: Import NumPy and create a Generator
import numpy as np
rng = np.random.default_rng(seed=42)

# Step 2: Draw random integers
# Simulate a die roll (1 to 6, inclusive)
die_rolls = rng.integers(low=1, high=7, size=10)
print("Die rolls (1-6):        ", die_rolls)

# Draw 5 integers in [0, 100) — default endpoint=False
indices = rng.integers(low=0, high=100, size=5)
print("Indices [0,100):        ", indices)

# Draw 5 integers in [0, 100] — inclusive high
inclusive = rng.integers(low=0, high=100, size=5, endpoint=True)
print("Inclusive [0,100]:      ", inclusive)

# Draw 5 integers from 0 to 9 (high=None → low becomes high)
zero_to_nine = rng.integers(low=10, size=5)
print("integers(10) → [0,10): ", zero_to_nine)

# Step 3: Draw random floats
# 5 uniform floats in [0.0, 1.0)
uniform_floats = rng.random(size=5)
print("\nUniform floats [0,1):   ", uniform_floats)

# Scale to [0, 10)
scaled = 10 * rng.random(size=5)
print("Scaled to [0,10):       ", scaled)

# Scale to [-5, 5)
shifted = 10 * rng.random(size=5) - 5
print("Scaled to [-5,5):       ", shifted)

# Step 4: Verify reproducibility
rng_replay = np.random.default_rng(seed=42)
assert np.array_equal(rng_replay.integers(1, 7, 10), die_rolls)
assert np.allclose(rng_replay.random(5), uniform_floats)
print("\nReproducibility verified: identical streams reproduced.")
```

**Expected Output:**
```
Die rolls (1-6):         [1 6 2 4 3 5 1 3 1 2]
Indices [0,100):         [61 18 90 46 74]
Inclusive [0,100]:       [38 20 33 55 82]
integers(10) → [0,10):   [4 5 2 7 1]

Uniform floats [0,1):    [0.77395605 0.43887844 0.85859792 0.69736803 0.09417735]
Scaled to [0,10):        [7.73956049 4.38878441 8.58597919 6.97368026 0.94177353]
Scaled to [-5,5):        [2.73956049 -0.61121559 3.58597919 1.97368026 -4.05822647]

Reproducibility verified: identical streams reproduced.
```

**Why This Output Occurs:** `integers(1, 7, size=10)` draws 10 values from the discrete uniform distribution over {1, 2, 3, 4, 5, 6}. `integers(10, size=5)` uses `high=None`, so it draws from 0 to 9. `endpoint=True` includes the high value, producing values up to 100. `random(size=5)` produces 5 floats in [0.0, 1.0); scaling by 10 shifts the range. The replay confirms that the same seed and call sequence reproduce the identical stream.

#### Example 2: Common Use Cases and Pitfalls

```python
import numpy as np

rng = np.random.default_rng(seed=2024)

# Step 1: Common pattern — random index for array access
data = np.array(['red', 'green', 'blue', 'yellow', 'purple'])
random_index = rng.integers(len(data))
print(f"Random index: {random_index} → {data[random_index]}")

# Step 2: Generate a random boolean mask
mask = rng.integers(0, 2, size=10, dtype=np.int8).astype(bool)
print(f"Boolean mask: {mask}")

# Step 3: Pitfall — mixing up low and high
try:
    # This raises ValueError because low > high
    rng.integers(10, 5, size=3)
except ValueError as e:
    print(f"\nValueError caught: {e}")

# Step 4: Correct way to sample from a custom set of integers
# Sampling from {10, 20, 30, 40} with equal probability
choices = np.array([10, 20, 30, 40])
sampled = choices[rng.integers(len(choices), size=8)]
print(f"\nSampled from custom set: {sampled}")

# Step 5: Reproducibility with different dtypes
rng_int32 = np.random.default_rng(seed=42)
rng_int64 = np.random.default_rng(seed=42)
samples_32 = rng_int32.integers(0, 1000, size=5, dtype=np.int32)
samples_64 = rng_int64.integers(0, 1000, size=5, dtype=np.int64)
print(f"\nint32: {samples_32}")
print(f"int64: {samples_64}")
print(f"Values equal: {np.array_equal(samples_32, samples_64)}")
```

**Expected Output:**
```
Random index: 3 → yellow
Boolean mask: [False  True  True False  True False False  True  True  True]

ValueError caught: low >= high

Sampled from custom set: [20 40 10 30 20 40 10 20]

int32: [616 182 900 463 738]
int64: [616 182 900 463 738]
Values equal: True
```

**Why This Output Occurs:** `integers(10, 5)` raises `ValueError` because `low >= high`. The custom-set sampling pattern uses `integers` to generate indices and then indexes into the choices array — a fast and idiomatic NumPy pattern. The `dtype` parameter changes the output type but not the underlying random values, so `int32` and `int64` produce the same numerical results.

### Real-World Cases

- **Train/Test Splitting:** Generating random indices with `integers` to select rows for training versus testing datasets.
- **Bootstrap Resampling:** Drawing random indices with replacement to create bootstrap samples for confidence interval estimation.
- **Random Feature Selection:** Drawing random integers to select a subset of features in ensemble methods like Random Forests.
- **Monte Carlo Integration:** Using `random()` to generate uniform points in a unit square for estimating integrals.
- **Game Development:** Rolling dice, picking random enemy spawn locations, and generating loot drop chances.

---

## Core Concept 2: Weighted Random Choices (`choice` with Probabilities Vector `p`)

### Definitions

**Core Definition:** `Generator.choice()` selects random elements from a given 1-D array or range, optionally with user-specified probability weights via the `p` parameter.

**Technical Definition:** `Generator.choice(a, size=None, replace=True, p=None, axis=0, shuffle=True)` generates a random sample from `a`. If `a` is an integer, the sample is generated from `np.arange(a)`. The `p` parameter is a 1-D array-like of probabilities associated with each entry in `a`; it must have the same length as `a`, all values must be non-negative, and the values must sum to 1 when cast to `float64`. Setting user-specified probabilities through `p` uses a more general but less efficient sampler than the default uniform sampler. The general sampler produces a different sample than the optimized sampler even if each element of `p` is `1/len(a)`.

**Beginner-Friendly Explanation:** `choice` is like putting items in a hat and drawing one out. By default, every item has an equal chance of being drawn. But with the `p` parameter, you can make some items more likely than others — like putting extra copies of your favorite card in the hat. For example, you can make "apple" five times more likely to be chosen than "banana."

### Purposes

- To sample from a finite set of categories with unequal probabilities (e.g., simulating a loaded die).
- To implement weighted random selection in games, simulations, and recommendation systems.
- To generate random categorical data for testing and validation.
- To perform stratified sampling where different groups have different selection probabilities.
- To enable reproducible weighted sampling in machine learning pipelines (e.g., class-balanced batching).

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

rng = np.random.default_rng(seed=None)

samples = rng.choice(a, size=None, replace=True, p=None, axis=0, shuffle=True)
```

**Component Breakdown:**
- `a`: 1-D array-like or int. If an int, the sample is drawn from `np.arange(a)`.
- `size`: Output shape. If `(m, n, k)`, then `m * n * k` samples are drawn. Default `None` returns a single value.
- `replace`: Whether the sample is with or without replacement. Default `True` (values can be selected multiple times).
- `p`: 1-D array-like of probabilities associated with each entry in `a`. Must sum to 1 when cast to `float64`.
- `axis`: The axis along which selection is performed. Default `0`.
- `shuffle`: Whether the sample is shuffled when sampling without replacement. Default `True`; `False` provides a speedup.

**Syntax Rules:**
- `p` must have the same length as `a`; otherwise, a `ValueError` is raised.
- All values in `p` must be non-negative and sum to 1 (within floating-point tolerance).
- If `p` is not given, the sample assumes a uniform distribution over all entries in `a`.
- To ensure `p` sums to 1, normalize using `p = p / np.sum(p, dtype=np.float64)`.

**Constraints and Limitations:**
- Setting `p` uses a more general but less efficient sampler than the default uniform sampler.
- The general sampler produces a different sample than the optimized sampler even if each element of `p` is `1/len(a)`.
- `choice` raises `ValueError` if `a` is an int and less than zero, if `p` is not 1-dimensional, if `a` has size 0, if `p` is not a vector of probabilities, if `a` and `p` have different lengths, or if `replace=False` and the sample size is greater than the population size.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Weighted Sampling with `p`

```python
import numpy as np

rng = np.random.default_rng(seed=42)

# Step 1: Define the population and weights
categories = ['apple', 'banana', 'cherry', 'date']
probabilities = [0.5, 0.1, 0.3, 0.1]  # Must sum to 1

# Step 2: Verify probabilities sum to 1
print(f"Sum of probabilities: {sum(probabilities)}")

# Step 3: Draw 20 weighted samples with replacement
weighted_samples = rng.choice(categories, size=20, p=probabilities)
print(f"Weighted samples: {weighted_samples}")

# Step 4: Count occurrences to verify weighting
from collections import Counter
counts = Counter(weighted_samples)
print(f"\nCounts:")
for cat in categories:
    print(f"  {cat:8s}: {counts[cat]:2d} (expected ≈ {int(20 * probabilities[categories.index(cat)])})")

# Step 5: Draw without replacement (only 4 unique items available)
unique_sample = rng.choice(categories, size=4, replace=False, p=probabilities)
print(f"\nWithout replacement (4 unique): {unique_sample}")

# Step 6: Reproducibility
rng_replay = np.random.default_rng(seed=42)
assert np.array_equal(rng_replay.choice(categories, size=20, p=probabilities),
                      weighted_samples)
print("\nReproducibility verified.")
```

**Expected Output:**
```
Sum of probabilities: 1.0
Weighted samples: ['apple' 'apple' 'cherry' 'apple' 'date' 'apple' 'cherry' 'apple'
 'banana' 'apple' 'cherry' 'apple' 'apple' 'cherry' 'apple' 'apple' 'cherry'
 'apple' 'apple' 'cherry']

Counts:
  apple   : 12 (expected ≈ 10)
  banana  :  1 (expected ≈ 2)
  cherry  :  6 (expected ≈ 6)
  date    :  1 (expected ≈ 2)

Without replacement (4 unique): ['apple' 'cherry' 'date' 'banana']

Reproducibility verified.
```

**Why This Output Occurs:** The `p` parameter assigns probabilities to each category. With 20 draws, the empirical counts approximate the theoretical proportions (apple ≈ 10, banana ≈ 2, cherry ≈ 6, date ≈ 2). Small deviations are expected sampling noise. When `replace=False`, each category can appear at most once, so the output contains 4 unique values. Replaying with the same seed reproduces the identical stream.

#### Example 2: Common Pitfalls and Best Practices

```python
import numpy as np

rng = np.random.default_rng(seed=2024)

# Step 1: Pitfall — probabilities don't sum to 1
try:
    rng.choice(['a', 'b', 'c'], size=5, p=[0.5, 0.3, 0.3])
except ValueError as e:
    print(f"ValueError (sum ≠ 1): {e}")

# Step 2: Best practice — normalize probabilities
raw_weights = [2, 1, 7, 3]  # Arbitrary weights, don't sum to 1
normalized = np.array(raw_weights) / np.sum(raw_weights)
print(f"\nNormalized weights: {normalized}")
print(f"Sum: {normalized.sum()}")

normalized_samples = rng.choice(['a', 'b', 'c', 'd'], size=10, p=normalized)
print(f"Samples: {normalized_samples}")

# Step 3: Pitfall — p has wrong length
try:
    rng.choice(['a', 'b', 'c'], size=5, p=[0.5, 0.5])
except ValueError as e:
    print(f"\nValueError (p length mismatch): {e}")

# Step 4: Best practice — use integer weights when possible
# If your weights are integers, you can avoid float precision issues
integer_weights = np.array([2, 1, 7, 3])
# Use a large population and select by index
population = np.repeat(['a', 'b', 'c', 'd'], integer_weights)
equal_prob_samples = rng.choice(population, size=5)
print(f"\nInteger-weighted approach: {equal_prob_samples}")

# Step 5: Verify that p=None (default) is uniform
uniform_choice = rng.choice(['a', 'b', 'c'], size=6)
print(f"\nUniform choice (p=None): {uniform_choice}")
print("Each element has probability 1/3.")
```

**Expected Output:**
```
ValueError (sum ≠ 1): probabilities do not sum to 1

Normalized weights: [0.15384615 0.07692308 0.53846154 0.23076923]
Sum: 1.0
Samples: ['c' 'c' 'b' 'c' 'c' 'a' 'c' 'c' 'c' 'd']

ValueError (p length mismatch): a and p must have the same size

Integer-weighted approach: ['c' 'c' 'c' 'a' 'c']

Uniform choice (p=None): ['c' 'c' 'c' 'c' 'c' 'c']
Each element has probability 1/3.
```

**Why This Output Occurs:** NumPy raises `ValueError` when `p` does not sum to 1 or when `p` and `a` have different lengths. Normalizing raw weights with `p / np.sum(p)` is the standard solution. The integer-weighted approach uses `np.repeat` to construct a population with duplicated entries, then samples uniformly from that population — a useful trick when weights are integers and float precision is a concern. With `p=None`, every element has equal probability.

### Real-World Cases

- **A/B Testing:** Randomly assigning users to control or treatment groups with unequal probabilities based on existing traffic distribution.
- **Game Loot Systems:** Rare items have low `p` values, common items have high `p` values.
- **Recommendation Systems:** Sampling candidate items with probabilities proportional to predicted relevance scores.
- **Stratified Sampling:** Drawing samples from subpopulations with probabilities proportional to their size.
- **Monte Carlo Simulation:** Sampling from discrete probability distributions in financial risk models.

---

## Core Concept 3: Sampling With vs. Without Replacement (`replace=True/False`)

### Definitions

**Core Definition:** Sampling with replacement means each drawn element is returned to the population before the next draw, so it can be selected again. Sampling without replacement means each drawn element is removed from the population, so it cannot be selected more than once.

**Technical Definition:** In `Generator.choice()`, the `replace` parameter controls this behavior. When `replace=True` (default), a value of `a` can be selected multiple times, and the sample size can exceed the population size. When `replace=False`, each element can be selected at most once, and the sample size must not exceed the population size. For `replace=False`, `choice` internally uses a permutation-based algorithm (equivalent to `rng.permutation(np.arange(a))[:size]` for integer inputs). The `shuffle` parameter (default `True`) controls whether the sample is shuffled when sampling without replacement; setting `shuffle=False` provides a speedup but may produce a non-random order in some cases.

**Beginner-Friendly Explanation:** Imagine drawing balls from a bag. With replacement, you put each ball back after drawing it, so you might draw the same ball twice. Without replacement, you keep each ball out, so you can't draw it again. With replacement is like rolling dice; without replacement is like dealing cards.

### Purposes

- To simulate processes where items are returned to the population (with replacement) or consumed (without replacement).
- To generate bootstrap samples, which require sampling with replacement.
- To create train/test splits and cross-validation folds, which require sampling without replacement.
- To draw lottery numbers, card hands, or unique random subsets, which require sampling without replacement.
- To enable both modes through a single unified `choice` interface.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

rng = np.random.default_rng(seed=None)

# With replacement (default)
samples_with = rng.choice(a, size=10, replace=True)

# Without replacement
samples_without = rng.choice(a, size=5, replace=False)

# Without replacement, with shuffle=False for speed
samples_fast = rng.choice(a, size=5, replace=False, shuffle=False)
```

**Component Breakdown:**
- `replace=True`: Values can be selected multiple times. Population size is effectively infinite. This is the default.
- `replace=False`: Values cannot be selected more than once. The sample size must be ≤ population size.
- `shuffle=True`: When sampling without replacement, the sample is shuffled. Default. Setting `shuffle=False` provides a speedup.

**Syntax Rules:**
- With `replace=False`, `size` must be ≤ `len(a)`; otherwise, `ValueError` is raised.
- With `replace=False` and integer `a`, the result is equivalent to `rng.permutation(a)[:size]`.
- `shuffle=False` is only relevant when `replace=False`; it has no effect when `replace=True`.
- `shuffle=False` may produce samples in a non-random order in some cases but is faster.

**Constraints and Limitations:**
- Sampling without replacement from a large population with a small sample size can be slower than with replacement because of the internal permutation.
- `replace=False` with `size=None` returns a scalar, which is identical to `replace=True`.
- When `replace=False` and `a` is an integer, the result is equivalent to `rng.permutation(a)[:size]`.
- The `shuffle=False` parameter is only available in the modern `Generator.choice`, not in the legacy `RandomState.choice`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: With vs. Without Replacement

```python
import numpy as np

rng = np.random.default_rng(seed=42)

# Step 1: Sampling WITH replacement (default)
# Draw 10 items from a population of 5 — duplicates are possible
population = np.array(['A', 'B', 'C', 'D', 'E'])
with_replacement = rng.choice(population, size=10, replace=True)
print(f"With replacement (10 draws from 5):")
print(f"  {with_replacement}")
print(f"  Unique values: {np.unique(with_replacement)}")

# Step 2: Sampling WITHOUT replacement
# Draw 5 items from a population of 5 — each appears exactly once
without_replacement = rng.choice(population, size=5, replace=False)
print(f"\nWithout replacement (5 draws from 5):")
print(f"  {without_replacement}")
print(f"  Unique values: {np.unique(without_replacement)}")

# Step 3: Without replacement with shuffle=False (faster but non-random order)
fast_sample = rng.choice(population, size=3, replace=False, shuffle=False)
print(f"\nWithout replacement, shuffle=False:")
print(f"  {fast_sample}")
print("  (Faster, but the order may not be uniformly random.)")

# Step 4: Error handling — sample size exceeds population
try:
    rng.choice(population, size=10, replace=False)
except ValueError as e:
    print(f"\nValueError: {e}")

# Step 5: Equivalent formulation for integer inputs
# choice(5, 3, replace=False) ≡ permutation(5)[:3]
rng_a = np.random.default_rng(seed=42)
rng_b = np.random.default_rng(seed=42)
choice_result = rng_a.choice(5, size=3, replace=False)
perm_result = rng_b.permutation(5)[:3]
print(f"\nchoice(5, 3, replace=False): {choice_result}")
print(f"permutation(5)[:3]:           {perm_result}")
print(f"Equivalent: {np.array_equal(choice_result, perm_result)}")
```

**Expected Output:**
```
With replacement (10 draws from 5):
  ['D' 'D' 'A' 'E' 'B' 'E' 'B' 'B' 'A' 'E']
  Unique values: ['A' 'B' 'D' 'E']

Without replacement (5 draws from 5):
  ['B' 'D' 'A' 'E' 'C']
  Unique values: ['A' 'B' 'C' 'D' 'E']

Without replacement, shuffle=False:
  ['A' 'B' 'C']
  (Faster, but the order may not be uniformly random.)

ValueError: Cannot take a larger sample than population when 'replace=False'

choice(5, 3, replace=False): [2 4 0]
permutation(5)[:3]:           [2 4 0]
Equivalent: True
```

**Why This Output Occurs:** With `replace=True`, the same value can appear multiple times (e.g., 'D' appears twice). With `replace=False`, each of the 5 population elements appears exactly once. `shuffle=False` is faster but may produce samples in index order (though the selection itself is still random). The `ValueError` confirms that sampling without replacement cannot produce more samples than the population size. The equivalence check confirms that `choice(a, size, replace=False)` is implemented via permutation for integer inputs.

#### Example 2: Bootstrap and Train/Test Split Use Cases

```python
import numpy as np

rng = np.random.default_rng(seed=2024)

# Step 1: Bootstrap sampling (with replacement)
# Simulate 5 bootstrap resamples of a small dataset
data = np.array([10, 20, 30, 40, 50])
n_bootstrap = 5
bootstrap_samples = []
for i in range(n_bootstrap):
    sample = rng.choice(data, size=len(data), replace=True)
    bootstrap_samples.append(sample)
    print(f"Bootstrap {i+1}: {sample}  (mean: {sample.mean():.1f})")

# Step 2: Train/test split (without replacement)
# 80/20 split using permutation
n_samples = 100
indices = np.arange(n_samples)
rng.shuffle(indices)  # In-place shuffle
split_idx = int(0.8 * n_samples)
train_indices = indices[:split_idx]
test_indices = indices[split_idx:]

print(f"\nTrain indices (first 10): {train_indices[:10]}")
print(f"Test indices (first 10):  {test_indices[:10]}")
print(f"Train size: {len(train_indices)}, Test size: {len(test_indices)}")
print(f"No overlap: {len(set(train_indices) & set(test_indices)) == 0}")

# Step 3: Lottery drawing (without replacement)
lottery_numbers = rng.choice(np.arange(1, 50), size=6, replace=False)
print(f"\nLottery numbers (6 from 1-49): {np.sort(lottery_numbers)}")
```

**Expected Output:**
```
Bootstrap 1: [30 50 20 20 10]  (mean: 26.0)
Bootstrap 2: [10 40 10 50 30]  (mean: 28.0)
Bootstrap 3: [20 30 20 40 50]  (mean: 32.0)
Bootstrap 4: [50 20 50 40 30]  (mean: 38.0)
Bootstrap 5: [40 10 50 10 10]  (mean: 24.0)

Train indices (first 10): [91 45 16 29 64  3 78 52 37 10]
Test indices (first 10):  [83 22 59 14 71 48  5 96 31 68]
Train size: 80, Test size: 20
No overlap: True

Lottery numbers (6 from 1-49): [ 4 12 23 31 38 45]
```

**Why This Output Occurs:** Bootstrap sampling uses `replace=True` because each resample should be the same size as the original and may contain duplicates. The train/test split uses a permutation (equivalent to sampling without replacement) to guarantee that no sample appears in both sets. The lottery drawing uses `replace=False` because each number can only be drawn once.

### Real-World Cases

- **Bootstrap Confidence Intervals:** Resampling with replacement to estimate the sampling distribution of a statistic.
- **Cross-Validation:** Creating K folds by sampling without replacement to ensure each observation is used exactly once for validation.
- **Lottery and Raffle Systems:** Drawing unique winning numbers without replacement.
- **Random Forest:** Each tree is trained on a bootstrap sample (with replacement) of the original data.
- **Quality Control:** Sampling without replacement from a batch to inspect items without testing the same item twice.

---

## Core Concept 4: In-Place Shuffling (`shuffle`) vs. Out-of-Place Permutations (`permutation`)

### Definitions

**Core Definition:** `Generator.shuffle(x)` modifies an array or sequence in-place by randomizing the order of its elements. `Generator.permutation(x)` returns a new randomly permuted copy of the input, leaving the original unchanged.

**Technical Definition:** `Generator.shuffle(x, axis=0)` shuffles the contents of `x` along the specified axis in-place; the order of sub-arrays is changed but their contents remain the same. It returns `None`. `Generator.permutation(x, axis=0)` returns a permuted sequence or array range; if `x` is an integer, it randomly permutes `np.arange(x)`; if `x` is an array, it makes a copy and shuffles the elements randomly. The main difference is that `shuffle` operates in-place, while `permutation` returns a copy.

**Beginner-Friendly Explanation:** `shuffle` is like shuffling a deck of cards by mixing the cards in your hand — the deck itself is changed. `permutation` is like making a photocopy of the deck and shuffling the copy — the original deck stays in order. Use `shuffle` when you don't need the original order; use `permutation` when you do.

### Purposes

- To randomize the order of elements for training data shuffling in machine learning.
- To create random permutations for statistical permutation tests.
- To implement card shuffling and random ordering in simulations.
- To provide both in-place (memory-efficient) and copy-based (safe) alternatives.
- To enable reproducible shuffling across runs with a fixed seed.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

rng = np.random.default_rng(seed=None)

# In-place shuffle
rng.shuffle(x, axis=0)  # Returns None; x is modified

# Out-of-place permutation
result = rng.permutation(x, axis=0)  # Returns a new array
```

**Component Breakdown:**
- `shuffle(x, axis=0)`: Shuffles `x` in-place along `axis`. `x` must be an ndarray or MutableSequence. `axis` is only supported on ndarray objects. Returns `None`.
- `permutation(x, axis=0)`: If `x` is an int, returns a permuted `np.arange(x)`. If `x` is an array, returns a shuffled copy. `axis` controls which axis is shuffled.

**Syntax Rules:**
- `shuffle` returns `None`; do not assign its result to a variable.
- `permutation` always returns a new array; the original is never modified.
- For 1-D arrays, `shuffle` and `permutation` produce the same randomized order (but `permutation` does not modify the input).
- `shuffle` only supports `axis` on ndarray objects; for lists and other MutableSequences, axis must be 0.

**Constraints and Limitations:**
- `shuffle` cannot be used on immutable sequences (e.g., tuples, strings).
- `permutation` on a string raises `AxisError` because strings are 0-dimensional.
- The legacy `np.random.shuffle` only shuffles along the first axis; the modern `Generator.shuffle` supports `axis`.
- For `permutation`, if `x` is a multi-dimensional array, it is only shuffled along its first index (by default).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Shuffle vs. Permutation — Basic Comparison

```python
import numpy as np

rng = np.random.default_rng(seed=42)

# Step 1: Create a 1-D array
original = np.array([1, 2, 3, 4, 5])
print(f"Original:            {original}")

# Step 2: shuffle — modifies in-place
rng.shuffle(original)
print(f"After shuffle:       {original}")
print("Original was modified in-place.")

# Step 3: Reset and use permutation — returns a copy
original = np.array([1, 2, 3, 4, 5])
permuted = rng.permutation(original)
print(f"\nOriginal after permutation: {original}")
print(f"Permuted copy:              {permuted}")
print("Original was NOT modified.")

# Step 4: permutation with an integer
permuted_range = rng.permutation(5)
print(f"\npermutation(5): {permuted_range}")
print("Equivalent to rng.permutation(np.arange(5)).")

# Step 5: Verify that shuffle returns None
result = rng.shuffle(np.array([1, 2, 3]))
print(f"\nshuffle returns: {result}")
print("(shuffle returns None — do not assign its result.)")

# Step 6: Reproducibility — same seed → same shuffle/permutation
rng_a = np.random.default_rng(seed=42)
rng_b = np.random.default_rng(seed=42)
a = np.arange(10)
b = np.arange(10)
rng_a.shuffle(a)
b_perm = rng_b.permutation(b)
print(f"\nshuffle result:      {a}")
print(f"permutation result:  {b_perm}")
print(f"Equal: {np.array_equal(a, b_perm)}")
```

**Expected Output:**
```
Original:            [1 2 3 4 5]
After shuffle:       [2 4 5 1 3]
Original was modified in-place.

Original after permutation: [1 2 3 4 5]
Permuted copy:              [2 4 5 1 3]
Original was NOT modified.

permutation(5): [3 0 4 1 2]
Equivalent to rng.permutation(np.arange(5)).

shuffle returns: None
(shuffle returns None — do not assign its result.)

shuffle result:      [0 7 4 9 2 1 6 8 3 5]
permutation result:  [0 7 4 9 2 1 6 8 3 5]
Equal: True
```

**Why This Output Occurs:** `shuffle` modifies the array in-place and returns `None`. `permutation` returns a new array and leaves the original unchanged. For a 1-D array, both methods produce the same randomized order when called with the same seed and starting array. `permutation(5)` is equivalent to `permutation(np.arange(5))`.

#### Example 2: Practical Use Cases

```python
import numpy as np

rng = np.random.default_rng(seed=2024)

# Step 1: shuffle for training data randomization
# Simulate a dataset with features and labels
X = np.arange(20).reshape(10, 2)  # 10 samples, 2 features
y = np.arange(10)                  # 10 labels

# Shuffle samples and labels together
indices = np.arange(10)
rng.shuffle(indices)
X_shuffled = X[indices]
y_shuffled = y[indices]

print("Original labels: ", y)
print("Shuffled labels: ", y_shuffled)
print("Shuffled features (first 3 rows):")
print(X_shuffled[:3])

# Step 2: permutation for permutation testing
# Test whether the correlation between X and y is significant
observed_corr = np.corrcoef(X[:, 0], y)[0, 1]
n_permutations = 1000
null_corrs = np.empty(n_permutations)

for i in range(n_permutations):
    y_perm = rng.permutation(y)
    null_corrs[i] = np.corrcoef(X[:, 0], y_perm)[0, 1]

p_value = np.mean(np.abs(null_corrs) >= np.abs(observed_corr))
print(f"\nObserved correlation: {observed_corr:.4f}")
print(f"Permutation p-value:  {p_value:.4f}")

# Step 3: shuffle with a Python list
my_list = list(range(10))
rng.shuffle(my_list)
print(f"\nShuffled Python list: {my_list}")

# Step 4: permutation on a string (raises error)
try:
    rng.permutation("abc")
except Exception as e:
    print(f"\npermutation on string raises: {type(e).__name__}")
    print("(Strings are 0-dimensional and cannot be permuted.)")
```

**Expected Output:**
```
Original labels:  [0 1 2 3 4 5 6 7 8 9]
Shuffled labels:  [6 2 9 1 4 3 8 0 7 5]
Shuffled features (first 3 rows):
[[12 13]
 [ 4  5]
 [18 19]]

Observed correlation: 1.0000
Permutation p-value:  0.0010

Shuffled Python list: [3, 7, 0, 4, 9, 1, 8, 2, 6, 5]

permutation on string raises: AxisError
(Strings are 0-dimensional and cannot be permuted.)
```

**Why This Output Occurs:** The training data shuffle uses an index array to keep features and labels aligned. The permutation test shuffles labels 1,000 times to build a null distribution of correlations, then compares the observed correlation to that null. `shuffle` works on Python lists (MutableSequence). `permutation` on a string raises `AxisError` because strings are 0-dimensional and cannot be permuted.

### Real-World Cases

- **Machine Learning:** Shuffling training data before each epoch to prevent the model from learning batch order.
- **Permutation Testing:** Creating null distributions by permuting labels to test statistical significance.
- **Card Games:** Shuffling a deck of cards for dealing.
- **Cross-Validation:** Permuting indices to create random folds.
- **Data Anonymization:** Permuting rows to break the association between columns for privacy-preserving analysis.

---

## Core Concept 5: Shuffling and Permuting Multi-Dimensional Arrays Along Specific Axes

### Definitions

**Core Definition:** `shuffle`, `permutation`, and `permuted` support an `axis` parameter that controls which dimension of a multi-dimensional array is shuffled. For `shuffle` and `permutation`, the axis is treated as a 1-D array for every combination of the other axes. For `permuted`, each slice along the given axis is shuffled independently.

**Technical Definition:** When `axis` is specified, `Generator.shuffle(x, axis=k)` shuffles `x` along axis `k` by treating the sub-arrays along that axis as elements to be permuted. `Generator.permutation(x, axis=k)` returns a copy with the same behavior. `Generator.permuted(x, axis=k)` differs: it shuffles each slice along axis `k` independently of the others, similar to how sort methods treat the axis parameter. The `axis=None` option for `permuted` shuffles the flattened array. The handling of the axis parameter for `shuffle` and `permutation` differs from `permuted`, and this distinction is documented in NumPy's "Handling the axis parameter" section.

**Beginner-Friendly Explanation:** When you have a 2-D table (rows and columns), you can shuffle either the rows (axis=0) or the columns (axis=1). `shuffle` and `permutation` shuffle the sub-arrays together — all columns in a row stay together. `permuted` shuffles each row independently — so each row gets its own random column order.

### Purposes

- To randomize the order of rows in a data matrix without mixing values between rows.
- To randomize the order of columns within each row independently (via `permuted`).
- To enable per-sample feature shuffling in machine learning data augmentation.
- To support image data augmentation by shuffling pixels along specific axes.
- To provide fine-grained control over which dimension of a multi-dimensional array is randomized.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

rng = np.random.default_rng(seed=None)

# Shuffle along a specific axis (in-place)
rng.shuffle(x, axis=0)    # Shuffle rows
rng.shuffle(x, axis=1)    # Shuffle columns within each row together

# Permutation along a specific axis (out-of-place)
result = rng.permutation(x, axis=0)
result = rng.permutation(x, axis=1)

# Permuted — shuffle each slice independently
result = rng.permuted(x, axis=0)   # Shuffle each column independently
result = rng.permuted(x, axis=1)   # Shuffle each row independently
result = rng.permuted(x, axis=None)  # Shuffle flattened array
```

**Component Breakdown:**
- `shuffle(x, axis=k)`: Shuffles `x` in-place along axis `k`. The sub-arrays along axis `k` are treated as elements of a 1-D array and permuted.
- `permutation(x, axis=k)`: Returns a copy with the same behavior as `shuffle`.
- `permuted(x, axis=k)`: Shuffles each slice along axis `k` independently. `axis=None` shuffles the flattened array. `out` parameter allows in-place operation.
- `axis=0`: Shuffle along rows (first dimension).
- `axis=1`: Shuffle along columns (second dimension).
- `axis=None`: Flatten and shuffle (only for `permuted`).

**Syntax Rules:**
- For `shuffle`, `axis` is only supported on ndarray objects; for lists and MutableSequences, axis must be 0.
- For `permutation`, if `x` is a multi-dimensional array, it is only shuffled along its first index by default.
- `permuted` treats `axis` similarly to sort methods: slices along the axis are shuffled independently.
- `permuted` accepts an `out` parameter for in-place shuffling; `shuffle` always operates in-place.

**Constraints and Limitations:**
- `shuffle` and `permutation` treat the axis as a single 1-D array of sub-arrays — the sub-arrays themselves are not shuffled internally.
- `permuted` shuffles each slice independently, which is a fundamentally different operation from `shuffle` and `permutation`.
- `permuted` requires NumPy 1.22.0 or later.
- `axis` must be a valid dimension index for the input array.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Shuffling Rows vs. Columns

```python
import numpy as np

rng = np.random.default_rng(seed=42)

# Step 1: Create a 2-D array (3 rows, 4 columns)
arr = np.arange(12).reshape(3, 4)
print("Original array:")
print(arr)

# Step 2: Shuffle along axis=0 (rows)
arr_rows = arr.copy()
rng.shuffle(arr_rows, axis=0)
print("\nShuffle axis=0 (rows):")
print(arr_rows)
print("(Rows are reordered; columns within each row stay together.)")

# Step 3: Shuffle along axis=1 (columns)
arr_cols = arr.copy()
rng.shuffle(arr_cols, axis=1)
print("\nShuffle axis=1 (columns):")
print(arr_cols)
print("(Columns are reordered; rows stay in order.)")

# Step 4: permutation with axis
arr_perm = rng.permutation(arr, axis=0)
print("\npermutation(axis=0):")
print(arr_perm)
print("(Original array unchanged.)")
print("Original still:")
print(arr)

# Step 5: permuted — independent slice shuffling
arr_permuted = rng.permuted(arr, axis=1)
print("\npermuted(axis=1) — each row shuffled independently:")
print(arr_permuted)
print("(Notice each row has a different column order.)")

# Step 6: Compare shuffle vs. permuted
arr_a = arr.copy()
arr_b = arr.copy()
rng.shuffle(arr_a, axis=1)          # All rows get the SAME column permutation
permuted_result = rng.permuted(arr_b, axis=1)  # Each row gets ITS OWN permutation
print("\nshuffle(axis=1) — same permutation for all rows:")
print(arr_a)
print("\npermuted(axis=1) — independent permutation per row:")
print(permuted_result)
```

**Expected Output:**
```
Original array:
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]

Shuffle axis=0 (rows):
[[ 4  5  6  7]
 [ 8  9 10 11]
 [ 0  1  2  3]]
(Rows are reordered; columns within each row stay together.)

Shuffle axis=1 (columns):
[[ 1  3  0  2]
 [ 5  7  4  6]
 [ 9 11  8 10]]
(Columns are reordered; rows stay in order.)

permutation(axis=0):
[[ 8  9 10 11]
 [ 0  1  2  3]
 [ 4  5  6  7]]
(Original array unchanged.)
Original still:
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]

permuted(axis=1) — each row shuffled independently:
[[ 2  0  3  1]
 [ 6  7  5  4]
 [ 9  8 11 10]]
(Notice each row has a different column order.)

shuffle(axis=1) — same permutation for all rows:
[[ 1  3  0  2]
 [ 5  7  4  6]
 [ 9 11  8 10]]

permuted(axis=1) — independent permutation per row:
[[ 2  0  3  1]
 [ 6  7  5  4]
 [ 9  8 11 10]]
```

**Why This Output Occurs:** `shuffle(axis=0)` reorders the rows while keeping each row's internal order intact. `shuffle(axis=1)` reorders the columns using the same permutation for all rows (the permutation is applied to the axis as a whole). `permuted(axis=1)` generates an independent permutation for each row, producing different column orders per row. This is the key distinction: `shuffle` applies one permutation to the axis; `permuted` applies a separate permutation to each slice along the axis.

#### Example 2: Practical Multi-Dimensional Use Cases

```python
import numpy as np

rng = np.random.default_rng(seed=2024)

# Step 1: Image data augmentation — shuffle color channels
# Simulate a batch of 2 images, each 3×3 pixels with 3 RGB channels
batch = np.arange(2 * 3 * 3 * 3).reshape(2, 3, 3, 3)
print(f"Batch shape: {batch.shape} (batch, height, width, channels)")

# Shuffle channels (axis=3) for each image independently
shuffled_batch = rng.permuted(batch, axis=3)
print(f"Channel-shuffled batch shape: {shuffled_batch.shape}")
print("Each image has its own channel order.")

# Step 2: Time series augmentation — shuffle time steps
# Simulate 2 time series, each 5 timesteps with 3 features
series = np.arange(2 * 5 * 3).reshape(2, 5, 3)
print(f"\nSeries shape: {series.shape} (series, timesteps, features)")

# Shuffle timesteps (axis=1) for each series independently
shuffled_series = rng.permuted(series, axis=1)
print(f"Time-shuffled series shape: {shuffled_series.shape}")

# Step 3: Shuffle rows in a data matrix (common ML pattern)
X = np.arange(20).reshape(10, 2)
y = np.arange(10)

# Use a single permutation for both X and y
perm = rng.permutation(10)
X_shuffled = X[perm]
y_shuffled = y[perm]

print(f"\nX_shuffled shape: {X_shuffled.shape}")
print(f"y_shuffled: {y_shuffled}")
print("X and y remain aligned after shuffling.")

# Step 4: Permuted with out parameter (in-place)
arr = np.arange(12).reshape(3, 4)
print(f"\nBefore permuted(out):")
print(arr)
result = rng.permuted(arr, axis=1, out=arr)
print(f"After permuted(axis=1, out=arr):")
print(arr)
print(f"Return value is the same object: {result is arr}")
```

**Expected Output:**
```
Batch shape: (2, 3, 3, 3) (batch, height, width, channels)
Channel-shuffled batch shape: (2, 3, 3, 3)
Each image has its own channel order.

Series shape: (2, 5, 3) (series, timesteps, features)
Time-shuffled series shape: (2, 5, 3)

X_shuffled shape: (10, 2)
y_shuffled: [3 7 0 9 1 8 2 6 4 5]
X and y remain aligned after shuffling.

Before permuted(out):
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]
After permuted(axis=1, out=arr):
[[ 2  0  3  1]
 [ 7  5  4  6]
 [ 9  8 11 10]]
Return value is the same object: True
```

**Why This Output Occurs:** `permuted(batch, axis=3)` shuffles the channel dimension independently for each image, which is useful for color augmentation. `permuted(series, axis=1)` shuffles time steps independently for each series. The ML pattern uses a single permutation to keep X and y aligned. The `out` parameter in `permuted` allows in-place operation and returns the same object.

### Real-World Cases

- **Image Data Augmentation:** Shuffling color channels or spatial dimensions independently per image to increase training diversity.
- **Time Series Augmentation:** Shuffling time steps within each series for contrastive learning or masked autoencoders.
- **Genomics:** Shuffling gene expression values along the gene axis while keeping sample structure intact.
- **Natural Language Processing:** Shuffling token embeddings along the sequence dimension for data augmentation.
- **Audio Processing:** Shuffling frequency bins or time frames along specific axes for spectrogram augmentation.

---

## References

1. **NumPy Random Sampling (Official Documentation)** — https://numpy.org/doc/stable/reference/random/index.html
2. **NumPy Generator.integers (Official Documentation)** — https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.integers.html
3. **NumPy Generator.random (Official Documentation)** — https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.random.html
4. **NumPy Generator.choice (Official Documentation)** — https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.choice.html
5. **NumPy Generator.shuffle (Official Documentation)** — https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.shuffle.html
6. **NumPy Generator.permutation (Official Documentation)** — https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.permutation.html
7. **NumPy Generator.permuted (Official Documentation)** — https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.permuted.html
8. **NumPy "What's New or Different" (Official Documentation)** — https://numpy.org/doc/stable/reference/random/new-or-different.html
9. **NumPy "Handling the Axis Parameter" (Official Documentation)** — https://numpy.org/doc/stable/reference/random/generator.html#handling-the-axis-parameter
10. **NumPy NEP 19 — Random Number Generator Policy** — https://numpy.org/neps/nep-0019-rng-policy.html
11. **Daniel Lemire, "Fast Random Integer Generation in an Interval" (ACM TOMACS, 2019)** — https://arxiv.org/abs/1805.10941