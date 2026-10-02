# Vector Operations — A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Vector operations in NumPy encompass the fundamental mathematical operations performed on one-dimensional arrays (vectors), including arithmetic operations, products, norms, projections, and orthogonalization.

**Technical Definition:** NumPy represents vectors as 1-D `ndarray` objects. Vector operations are implemented through a combination of universal functions (ufuncs) for element-wise arithmetic, linear algebra routines in `numpy.linalg` for norms and decompositions, and specialized functions (`numpy.dot`, `numpy.inner`, `numpy.outer`, `numpy.cross`) for various products. These operations leverage BLAS and LAPACK for optimized performance and support broadcasting for combining vectors of compatible shapes.

**Beginner-Friendly Explanation:** A vector is just a list of numbers, like `[3, 4, 5]`. Vector operations let you do math with these lists: adding them together, multiplying them, measuring their length, or finding the angle between them. NumPy makes all of these operations fast and easy, whether you're working with a single vector or thousands at once.

### Key Characteristics

- **Element-Wise by Default:** Arithmetic operators (`+`, `-`, `*`, `/`) operate element-by-element on vectors of the same shape.
- **Broadcasting:** NumPy automatically expands smaller arrays to match larger ones during arithmetic operations.
- **BLAS-Backed:** Functions like `dot` and `linalg.norm` use optimized BLAS/LAPACK routines for speed.
- **Axis-Aware:** Many operations (norms, inner products) accept an `axis` parameter for multi-dimensional arrays.
- **Multiple Product Types:** NumPy distinguishes between dot products, inner products, outer products, and cross products, each with specific use cases.

### Prerequisites

- Basic Python programming (variables, functions, imports)
- Familiarity with NumPy arrays, dtypes, and the `Generator` class
- Basic linear algebra concepts (vector, magnitude, angle)
- Understanding of the `axis` parameter for multi-dimensional arrays

### Related Programming Areas

- Machine Learning (feature vectors, similarity measures, embeddings)
- Physics and Engineering (force vectors, projections, cross products)
- Computer Graphics (normal vectors, lighting calculations)
- Signal Processing (inner products, orthogonalization)
- Statistics (correlation, covariance, projections)
- Optimization (gradient vectors, orthogonal projections)

### Core Concepts / Features

1. Vector Scaling and Addition
2. Dot Product vs. General Inner Products
3. Vector Norms (L₁, L₂ Euclidean, Max/Infinity Norms)
4. Vector Projections and Orthogonalization
5. Outer Products and Cross Products

---

## Core Concept 1: Vector Scaling and Addition

### Definitions

**Core Definition:** Vector scaling multiplies each element of a vector by a scalar value. Vector addition combines two vectors of the same shape by adding their corresponding elements.

**Technical Definition:** NumPy implements vector scaling through broadcasting, where a scalar is treated as a 0-D array and applied element-wise to the vector. Vector addition is implemented as an element-wise ufunc (`numpy.add`) that requires operands to have compatible shapes (equal length or broadcastable). When vectors have different lengths, NumPy's broadcasting rules determine whether the operation is valid: the shorter vector is virtually expanded to match the longer one.

**Beginner-Friendly Explanation:** Scaling a vector is like stretching or shrinking an arrow — you multiply all its numbers by the same amount. Adding two vectors is like placing one arrow's tail at the other's head — you add their numbers position by position. NumPy does both operations instantly, even for millions of numbers.

### Purposes

- To combine vectors in force calculations, physics simulations, and engineering models.
- To scale feature vectors during normalization and standardization in machine learning.
- To implement linear combinations of basis vectors in linear algebra.
- To enable broadcasting for efficient batch operations on multiple vectors.
- To provide the foundation for more complex vector operations like projections and orthogonalization.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Vector scaling
scaled = scalar * vector
scaled = np.multiply(scalar, vector)

# Vector addition
sum_vec = vector1 + vector2
sum_vec = np.add(vector1, vector2)

# Vector subtraction
diff_vec = vector1 - vector2

# Element-wise multiplication
product = vector1 * vector2
```

**Component Breakdown:**
- `scalar * vector`: Broadcasting multiplies each element of `vector` by `scalar`.
- `vector1 + vector2`: Element-wise addition; both vectors must have the same shape or be broadcastable.
- `np.add(v1, v2)`: The ufunc equivalent of `+`.
- `vector1 * vector2`: Element-wise (Hadamard) product, not the dot product.

**Syntax Rules:**
- Vectors must have the same shape or be broadcastable for addition and subtraction.
- Scalar multiplication works with any numeric scalar (int, float, complex).
- Broadcasting rules apply: a `(3,)` vector can be added to a `(2, 3)` matrix, resulting in a `(2, 3)` matrix.
- For element-wise multiplication, `*` is the Hadamard product, not the dot product.

**Constraints and Limitations:**
- Adding vectors of incompatible shapes raises `ValueError`.
- Broadcasting silently expands smaller arrays, which can lead to unexpected results if dimensions are misaligned.
- In-place operations (`+=`, `*=`) modify the original array; use with caution.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Vector Scaling and Addition

```python
# Step 1: Import NumPy
import numpy as np

# Step 2: Create two vectors
v1 = np.array([1, 2, 3])
v2 = np.array([4, 5, 6])
print("v1:", v1)
print("v2:", v2)

# Step 3: Vector addition
v_sum = v1 + v2
print("\nv1 + v2:", v_sum)  # [5 7 9]

# Step 4: Vector subtraction
v_diff = v1 - v2
print("v1 - v2:", v_diff)  # [-3 -3 -3]

# Step 5: Scalar multiplication (scaling)
scalar = 3
v_scaled = scalar * v1
print(f"\n{scalar} * v1:", v_scaled)  # [3 6 9]

# Step 6: Element-wise multiplication (Hadamard product)
v_hadamard = v1 * v2
print("v1 * v2 (element-wise):", v_hadamard)  # [4 10 18]

# Step 7: Using ufunc equivalents
v_sum_ufunc = np.add(v1, v2)
assert np.array_equal(v_sum, v_sum_ufunc)
print("\nufunc add matches + operator: True")

# Step 8: Broadcasting — adding a scalar to a vector
v_broadcast = v1 + 10
print("\nv1 + 10 (broadcast):", v_broadcast)  # [11 12 13]

# Step 9: Broadcasting — adding a row vector to a matrix
matrix = np.array([[1, 2, 3], [4, 5, 6]])
row = np.array([10, 20, 30])
result = matrix + row
print("\nMatrix + row vector (broadcast):")
print(result)
```

**Expected Output:**
```
v1: [1 2 3]
v2: [4 5 6]

v1 + v2: [5 7 9]
v1 - v2: [-3 -3 -3]

3 * v1: [3 6 9]
v1 * v2 (element-wise): [ 4 10 18]

ufunc add matches + operator: True

v1 + 10 (broadcast): [11 12 13]

Matrix + row vector (broadcast):
[[11 22 33]
 [14 25 36]]
```

**Why This Output Occurs:** 
- Element-wise operations apply the operator to each corresponding pair of elements. 
- Scalar multiplication broadcasts the scalar to every element. 
- The matrix + row vector example demonstrates broadcasting: the row vector `(3,)` is virtually expanded to `(2, 3)` to match the matrix shape.

#### Example 2: Linear Combinations of Vectors

```python
import numpy as np

# Step 1: Define basis vectors
e1 = np.array([1, 0, 0])
e2 = np.array([0, 1, 0])
e3 = np.array([0, 0, 1])

# Step 2: Create a linear combination: 3*e1 + 2*e2 + 4*e3
coefficients = np.array([3, 2, 4])
basis = np.array([e1, e2, e3])  # Shape (3, 3)

# Step 3: Compute the linear combination
result = coefficients @ basis  # Matrix-vector product
print("Linear combination (3,2,4):", result)

# Step 4: Alternative — manual accumulation
result_manual = 3*e1 + 2*e2 + 4*e3
print("Manual result:", result_manual)
print("Match:", np.array_equal(result, result_manual))

# Step 5: Scaling a vector to unit length (normalization)
v = np.array([3.0, 4.0])
norm = np.linalg.norm(v)  # L2 norm = 5.0
v_unit = v / norm
print(f"\nOriginal vector: {v}")
print(f"Norm: {norm}")
print(f"Unit vector: {v_unit}")
print(f"Unit vector norm: {np.linalg.norm(v_unit):.6f}")

# Step 6: Scaling to a target magnitude
target_magnitude = 10.0
v_scaled_to_target = (v / norm) * target_magnitude
print(f"\nScaled to magnitude {target_magnitude}: {v_scaled_to_target}")
print(f"New magnitude: {np.linalg.norm(v_scaled_to_target):.6f}")
```

**Expected Output:**
```
Linear combination (3,2,4): [3 2 4]
Manual result: [3 2 4]
Match: True

Original vector: [3. 4.]
Norm: 5.0
Unit vector: [0.6 0.8]
Unit vector norm: 1.000000

Scaled to magnitude 10.0: [6. 8.]
New magnitude: 10.000000
```

**Why This Output Occurs:** 
- The matrix-vector product `coefficients @ basis` computes the linear combination by multiplying each basis vector by its coefficient and summing. 
- The normalization step divides the vector by its L2 norm, producing a unit vector. 
- Scaling by the target magnitude then produces a vector of the desired length.

### Real-World Cases

- **Physics Simulations:** Summing force vectors acting on an object to compute the net force.
- **Machine Learning:** Computing weighted sums of feature vectors in linear models and neural networks.
- **Computer Graphics:** Combining translation, rotation, and scaling vectors to transform 3D objects.
- **Signal Processing:** Adding multiple signal components (e.g., harmonics) to synthesize a complex waveform.
- **Economics:** Computing weighted portfolios where asset returns are vectors and weights are scalars.

---

## Core Concept 2: Dot Product vs. General Inner Products

### Definitions

**Core Definition:** The dot product (or inner product) of two vectors is the sum of the products of their corresponding elements, yielding a scalar. NumPy provides multiple functions for inner products: `numpy.dot`, `numpy.inner`, `numpy.vdot`, and `numpy.vecdot`.

**Technical Definition:** `numpy.dot(a, b)` computes the inner product of vectors for 1-D arrays (without complex conjugation), matrix multiplication for 2-D arrays, and sum products over the last axis of `a` and the second-to-last axis of `b` for higher dimensions. `numpy.inner(a, b)` computes the ordinary inner product for 1-D arrays and a sum product over the last axes for higher dimensions. `numpy.vdot(a, b)` takes the complex conjugate of the first argument if it is complex, and flattens multidimensional arrays before computing the dot product. `numpy.vecdot` (NumPy 2.0+) computes the vector dot product with broadcasting support.

**Beginner-Friendly Explanation:** The dot product takes two vectors and returns a single number. It's calculated by multiplying matching elements and adding the results. For example, `[1, 2, 3]` dot `[4, 5, 6]` = 1×4 + 2×5 + 3×6 = 32. The different NumPy functions (`dot`, `inner`, `vdot`) handle edge cases like complex numbers and multi-dimensional arrays differently.

### Purposes

- To measure similarity between vectors (cosine similarity in machine learning).
- To compute projections and orthogonal components in linear algebra.
- To implement matrix multiplication for 2-D arrays via `dot`.
- To calculate work in physics (force · displacement).
- To provide the foundation for computing vector norms and angles.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Dot product (1-D vectors)
result = np.dot(a, b)

# Inner product (1-D vectors)
result = np.inner(a, b)

# Vector dot product (flattens inputs)
result = np.vdot(a, b)

# Vector dot product (NumPy 2.0+)
result = np.vecdot(a, b)
```

**Component Breakdown:**
- `np.dot(a, b)`: For 1-D arrays, returns the inner product. For 2-D arrays, returns matrix multiplication. For N-D arrays, sum product over the last axis of `a` and second-to-last of `b`.
- `np.inner(a, b)`: For 1-D arrays, returns the inner product. For higher dimensions, sum product over the last axes of `a` and `b`.
- `np.vdot(a, b)`: Flattens both inputs to 1-D, takes the complex conjugate of `a` if complex, then computes the dot product.
- `np.vecdot(a, b)`: Vector dot product of two arrays with broadcasting support (NumPy 2.0+).

**Syntax Rules:**
- For 1-D arrays, `np.dot` and `np.inner` produce identical results.
- `np.vdot` flattens multidimensional arrays before computing the dot product.
- `np.dot` uses the second-to-last axis of `b` for N-D arrays, while `np.inner` uses the last axis.
- For complex numbers, `np.dot` does NOT conjugate; `np.vdot` DOES conjugate.

**Constraints and Limitations:**
- `np.dot` raises `ValueError` if the last dimension of `a` is not the same size as the second-to-last dimension of `b`.
- `np.inner` raises `ValueError` if both arrays are nonscalar and their last dimensions have different sizes.
- `np.vdot` flattens inputs, which may be inefficient for very large multidimensional arrays.
- `np.vecdot` requires NumPy 2.0 or later.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Dot, Inner, and Vdot

```python
import numpy as np

# Step 1: Define vectors
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Step 2: Dot product
dot_result = np.dot(a, b)
print(f"np.dot:   {dot_result}  (1*4 + 2*5 + 3*6 = {1*4 + 2*5 + 3*6})")

# Step 3: Inner product (identical for 1-D)
inner_result = np.inner(a, b)
print(f"np.inner: {inner_result}  (same for 1-D)")

# Step 4: Vdot (identical for real 1-D)
vdot_result = np.vdot(a, b)
print(f"np.vdot:  {vdot_result}  (same for real 1-D)")

# Step 5: Verify all three are equal for real 1-D vectors
assert dot_result == inner_result == vdot_result
print("\nAll three identical for real 1-D vectors: True")

# Step 6: Complex numbers — dot does NOT conjugate, vdot DOES
c = np.array([1+2j, 3+4j])
d = np.array([5+6j, 7+8j])

dot_complex = np.dot(c, d)
vdot_complex = np.vdot(c, d)
print(f"\nComplex dot:  {dot_complex}")
print(f"Complex vdot: {vdot_complex}")
print("(vdot conjugates the first argument.)")

# Step 7: Multi-dimensional arrays — dot vs. inner
A = np.arange(6).reshape(2, 3)
B = np.arange(6).reshape(3, 2)

dot_2d = np.dot(A, B)   # Matrix multiplication
inner_2d = np.inner(A, B)  # Sum product over last axes
print(f"\nA shape: {A.shape}, B shape: {B.shape}")
print(f"np.dot(A, B) shape: {dot_2d.shape}")
print(f"np.inner(A, B) shape: {inner_2d.shape}")

# Step 8: vdot flattens multidimensional arrays
vdot_2d = np.vdot(A, B)
manual = np.sum(A.ravel() * B.ravel())
print(f"\nnp.vdot(A, B): {vdot_2d}")
print(f"Manual (flattened): {manual}")
print(f"Match: {vdot_2d == manual}")
```

**Expected Output:**
```
np.dot:   32  (1*4 + 2*5 + 3*6 = 32)
np.inner: 32  (same for 1-D)
np.vdot:  32  (same for real 1-D)

All three identical for real 1-D vectors: True

Complex dot:  (-13+0j)
Complex vdot: (70-8j)
(vdot conjugates the first argument.)

A shape: (2, 3), B shape: (3, 2)
np.dot(A, B) shape: (2, 2)
np.inner(A, B) shape: (2, 3)

np.vdot(A, B): 55
Manual (flattened): 55
Match: True
```

**Why This Output Occurs:** For 1-D real vectors, `dot`, `inner`, and `vdot` all compute the same sum of products. For complex vectors, `vdot` conjugates the first argument (`1-2j` and `3-4j`), producing a different result than `dot`. For 2-D arrays, `dot` performs matrix multiplication (shape `(2,2)`), while `inner` computes a sum product over the last axes (shape `(2,3)`). `vdot` flattens both arrays before computing the dot product.

#### Example 2: Practical Applications of Dot Products

```python
import numpy as np

# Step 1: Cosine similarity between vectors
def cosine_similarity(u, v):
    """Compute cosine similarity: (u · v) / (||u|| * ||v||)."""
    dot = np.dot(u, v)
    norm_u = np.linalg.norm(u)
    norm_v = np.linalg.norm(v)
    return dot / (norm_u * norm_v)

v1 = np.array([1, 2, 3])
v2 = np.array([4, 5, 6])
v3 = np.array([-1, -2, -3])  # Opposite direction to v1

print(f"cos(v1, v2): {cosine_similarity(v1, v2):.4f}")
print(f"cos(v1, v3): {cosine_similarity(v1, v3):.4f}")
print(f"cos(v2, v3): {cosine_similarity(v2, v3):.4f}")

# Step 2: Work calculation (physics)
force = np.array([10, 0, 0])       # 10 N in x-direction
displacement = np.array([5, 3, 0]) # 5 m in x, 3 m in y
work = np.dot(force, displacement)
print(f"\nWork = force · displacement = {work} J")
print("(Only the x-component of displacement contributes.)")

# Step 3: Orthogonality check
u = np.array([1, 0])
v = np.array([0, 1])
dot_uv = np.dot(u, v)
print(f"\nu · v = {dot_uv}")
print(f"Orthogonal: {dot_uv == 0}")

# Step 4: Projection using dot product
a = np.array([3.0, 4.0])
b = np.array([1.0, 0.0])
# Scalar projection of a onto b
scalar_proj = np.dot(a, b) / np.linalg.norm(b)
print(f"\nScalar projection of {a} onto {b}: {scalar_proj}")

# Step 5: Batch dot products using np.einsum
vectors = np.array([[1, 2], [3, 4], [5, 6]])  # 3 vectors of length 2
query = np.array([1, 1])
# Dot each vector with the query
dots = np.einsum('ij,j->i', vectors, query)
print(f"\nBatch dot products: {dots}")
print(f"Manual: {[np.dot(v, query) for v in vectors]}")
```

**Expected Output:**
```
cos(v1, v2): 0.9746
cos(v1, v3): -1.0000
cos(v2, v3): -0.9746

Work = force · displacement = 50 J
(Only the x-component of displacement contributes.)

u · v = 0
Orthogonal: True

Scalar projection of [3. 4.] onto [1. 0.]: 3.0

Batch dot products: [ 3  7 11]
Manual: [3, 7, 11]
```

**Why This Output Occurs:** Cosine similarity measures the angle between vectors: 1 means identical direction, -1 means opposite, 0 means orthogonal. The work calculation shows that only the force component along the displacement contributes. The orthogonality check confirms that perpendicular vectors have a dot product of zero. The batch dot product uses `einsum` for efficient computation across multiple vectors.

### Real-World Cases

- **Machine Learning:** Cosine similarity in recommendation systems and NLP embeddings (word2vec, BERT).
- **Physics:** Work = force · displacement; power = force · velocity.
- **Computer Graphics:** Lambertian shading uses the dot product of the surface normal and light direction.
- **Signal Processing:** Correlation and convolution are implemented via dot products.
- **Statistics:** Covariance and correlation are normalized dot products of centered vectors.

---

## Core Concept 3: Vector Norms (L₁, L₂ Euclidean, Max/Infinity Norms)

### Definitions

**Core Definition:** A vector norm is a function that assigns a non-negative length or magnitude to a vector. Common norms include the L₁ (Manhattan) norm, L₂ (Euclidean) norm, and L∞ (max/infinity) norm.

**Technical Definition:** The `numpy.linalg.norm` function computes vector and matrix norms. For vector `x` of length n: the L₁ norm is `sum(|x_i|)`, the L₂ norm is `sqrt(sum(x_i²))`, and the L∞ norm is `max(|x_i|)`. The `ord` parameter selects the norm type: `ord=1` for L₁, `ord=2` or `None` for L₂ (default), `ord=np.inf` for L∞, and `ord=0` for the count of non-zero elements.

**Beginner-Friendly Explanation:** A norm is a way to measure how "big" a vector is. The L₂ norm (Euclidean) is the straight-line distance from the origin — like measuring with a ruler. The L₁ norm (Manhattan) is the distance if you can only walk along grid lines — like navigating city blocks. The L∞ norm just takes the largest absolute value.

### Purposes

- To measure vector magnitude for normalization and unit vector construction.
- To compute distances between vectors for clustering and nearest-neighbor search.
- To regularize machine learning models (L₁/L₂ regularization).
- To assess numerical stability and condition numbers in linear algebra.
- To provide the foundation for projections, orthogonalization, and convergence criteria.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# L2 norm (default)
l2_norm = np.linalg.norm(x)

# L1 norm
l1_norm = np.linalg.norm(x, ord=1)

# L-infinity norm
linf_norm = np.linalg.norm(x, ord=np.inf)

# L0 "norm" (count of non-zeros)
l0_norm = np.linalg.norm(x, ord=0)

# Frobenius norm (for matrices)
fro_norm = np.linalg.norm(matrix, ord='fro')

# Norm along a specific axis
row_norms = np.linalg.norm(matrix, axis=1)

# Keep dimensions for broadcasting
norm_keep = np.linalg.norm(x, keepdims=True)
```

**Component Breakdown:**
- `x`: Input array (1-D or 2-D).
- `ord`: Order of the norm. `None` (default) gives L₂ for vectors. `1` gives L₁. `np.inf` gives L∞. `0` gives the count of non-zeros. `'fro'` gives the Frobenius norm for matrices.
- `axis`: If `None`, computes the norm of the flattened array. If an integer, computes norms along that axis. If a 2-tuple, computes matrix norms.
- `keepdims`: If `True`, retains the reduced dimensions as size-1, enabling broadcasting.

**Syntax Rules:**
- For vectors, `ord=None` or `ord=2` gives the Euclidean norm.
- For matrices, `ord=None` or `ord='fro'` gives the Frobenius norm.
- `ord=0` is not a mathematical norm but counts non-zero elements.
- `axis` must be `None`, an integer, or a 2-tuple of integers.

**Constraints and Limitations:**
- `ord='fro'` and `ord='nuc'` raise `ValueError` if the input is not 2-D.
- For `ord < 1`, the result is not a true mathematical norm.
- The L∞ norm is not differentiable, which matters in optimization contexts.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Computing All Common Norms

```python
import numpy as np

# Step 1: Create a vector
v = np.array([3.0, -4.0, 0.0, 5.0, -1.0])
print("Vector:", v)

# Step 2: L1 norm (Manhattan) — sum of absolute values
l1 = np.linalg.norm(v, ord=1)
print(f"\nL1 norm: {l1}  (|3| + |-4| + |0| + |5| + |-1| = {abs(3)+abs(-4)+abs(0)+abs(5)+abs(-1)})")

# Step 3: L2 norm (Euclidean) — sqrt of sum of squares
l2 = np.linalg.norm(v, ord=2)  # or np.linalg.norm(v)
print(f"L2 norm: {l2:.4f}  (sqrt(9+16+0+25+1) = {np.sqrt(9+16+0+25+1):.4f})")

# Step 4: L-infinity norm — maximum absolute value
linf = np.linalg.norm(v, ord=np.inf)
print(f"L∞ norm: {linf}  (max(|3|, |-4|, |0|, |5|, |-1|) = 5)")

# Step 5: L0 "norm" — count of non-zero elements
l0 = np.linalg.norm(v, ord=0)
print(f"L0 norm: {l0}  (count of non-zeros = 4)")

# Step 6: Verify with manual calculations
assert l1 == 13.0
assert np.isclose(l2, np.sqrt(51))
assert linf == 5.0
assert l0 == 4.0
print("\nAll norms verified.")

# Step 7: Norms along axes for a matrix
M = np.array([[1, 2, 3], [4, 5, 6]])
print(f"\nMatrix M:\n{M}")
print(f"L2 norm (flattened): {np.linalg.norm(M):.4f}")
print(f"L2 norm along axis=0 (columns): {np.linalg.norm(M, axis=0)}")
print(f"L2 norm along axis=1 (rows):    {np.linalg.norm(M, axis=1)}")

# Step 8: keepdims for broadcasting
row_norms = np.linalg.norm(M, axis=1, keepdims=True)
normalized = M / row_norms
print(f"\nNormalized rows:\n{normalized}")
print(f"Row norms after normalization: {np.linalg.norm(normalized, axis=1)}")
```

**Expected Output:**
```
Vector: [ 3. -4.  0.  5. -1.]

L1 norm: 13.0  (|3| + |-4| + |0| + |5| + |-1| = 13.0)
L2 norm: 7.1414  (sqrt(9+16+0+25+1) = 7.1414)
L∞ norm: 5.0  (max(|3|, |-4|, |0|, |5|, |-1|) = 5)
L0 norm: 4.0  (count of non-zeros = 4)

All norms verified.

Matrix M:
[[1 2 3]
 [4 5 6]]
L2 norm (flattened): 9.5394
L2 norm along axis=0 (columns): [4.1231 5.3852 6.7082]
L2 norm along axis=1 (rows):    [3.7417 8.7750]

Normalized rows:
[[0.2673 0.5345 0.8018]
 [0.4558 0.5698 0.6838]]
Row norms after normalization: [1. 1.]
```

**Why This Output Occurs:** The L₁ norm sums absolute values (13), the L₂ norm is the Euclidean distance (√51 ≈ 7.1414), the L∞ norm is the largest absolute value (5), and the L₀ "norm" counts non-zeros (4). For matrices, `axis=0` computes norms down columns, `axis=1` computes norms across rows. `keepdims=True` preserves the dimension for broadcasting during normalization.

#### Example 2: Distance Metrics and Regularization

```python
import numpy as np

# Step 1: Euclidean distance between two vectors
u = np.array([1.0, 2.0, 3.0])
v = np.array([4.0, 5.0, 6.0])

# L2 distance = norm of difference
l2_distance = np.linalg.norm(u - v)
print(f"L2 distance (Euclidean): {l2_distance:.4f}")

# Step 2: Manhattan distance
l1_distance = np.linalg.norm(u - v, ord=1)
print(f"L1 distance (Manhattan): {l1_distance:.4f}")

# Step 3: Chebyshev distance
linf_distance = np.linalg.norm(u - v, ord=np.inf)
print(f"L∞ distance (Chebyshev): {linf_distance:.4f}")

# Step 4: Machine learning — L2 regularization
weights = np.array([0.5, -1.2, 0.8, 0.3])
l2_reg = 0.01 * np.sum(weights ** 2)  # L2 penalty
l1_reg = 0.01 * np.sum(np.abs(weights))  # L1 penalty
print(f"\nL2 regularization penalty: {l2_reg:.6f}")
print(f"L1 regularization penalty: {l1_reg:.6f}")

# Step 5: L1 regularization encourages sparsity
# Demonstrate by comparing gradient magnitudes
grad_l2 = 2 * weights
grad_l1 = np.sign(weights)
print(f"\nL2 gradient: {grad_l2}")
print(f"L1 gradient: {grad_l1}")
print("(L1 gradient has constant magnitude, driving weights to exactly zero.)")

# Step 6: Nearest neighbor search using norms
candidates = np.array([[1, 1], [2, 2], [10, 10], [0, 5]])
query = np.array([3, 3])
distances = np.linalg.norm(candidates - query, axis=1)
nearest_idx = np.argmin(distances)
print(f"\nQuery: {query}")
print(f"Distances to candidates: {distances}")
print(f"Nearest candidate: {candidates[nearest_idx]} (index {nearest_idx})")
```

**Expected Output:**
```
L2 distance (Euclidean): 5.1962
L1 distance (Manhattan): 9.0000
L∞ distance (Chebyshev): 3.0000

L2 regularization penalty: 0.024200
L1 regularization penalty: 0.028000

L2 gradient: [ 1.  -2.4  1.6  0.6]
L1 gradient: [ 1. -1.  1.  1.]
(L1 gradient has constant magnitude, driving weights to exactly zero.)

Query: [3 3]
Distances to candidates: [2.8284 1.4142 9.8995 3.6056]
Nearest candidate: [2 2] (index 1)
```

**Why This Output Occurs:** The L₂ distance is the straight-line distance; the L₁ distance is the sum of absolute differences; the L∞ distance is the maximum coordinate difference. The regularization penalties show how L₂ pushes all weights toward zero proportionally, while L₁ applies constant force that drives some weights to exactly zero (sparsity). The nearest-neighbor search finds the candidate closest to the query using L₂ distance.

### Real-World Cases

- **Machine Learning:** L₁ regularization (Lasso) for feature selection; L₂ regularization (Ridge) for weight decay.
- **Clustering:** K-means uses L₂ distance; K-medoids can use L₁ distance.
- **Recommender Systems:** Cosine similarity (based on L₂ norms) for item-item similarity.
- **Numerical Analysis:** Condition numbers use matrix norms to assess stability.
- **Image Processing:** Total variation denoising uses L₁ norm of gradients.

---

## Core Concept 4: Vector Projections and Orthogonalization

### Definitions

**Core Definition:** A vector projection is the orthogonal projection of one vector onto another, representing the component of the first vector that lies along the direction of the second. Orthogonalization is the process of transforming a set of vectors into a set of mutually orthogonal (perpendicular) vectors.

**Technical Definition:** The projection of vector `a` onto vector `b` is given by `proj_b(a) = (a · b / b · b) * b`. The orthogonal component of `a` relative to `b` is `a - proj_b(a)`. The Gram-Schmidt process orthogonalizes a set of vectors `{v₁, v₂, ..., vₙ}` by iteratively subtracting projections onto previously computed orthogonal vectors: `u₁ = v₁`, `u₂ = v₂ - proj_{u₁}(v₂)`, `u₃ = v₃ - proj_{u₁}(v₃) - proj_{u₂}(v₃)`, and so on. Normalizing each orthogonal vector yields an orthonormal basis.

**Beginner-Friendly Explanation:** Projecting a vector onto another is like shining a light perpendicular to the second vector and seeing the shadow of the first vector on it. Orthogonalization is like straightening a set of arrows so they all point in completely independent directions (at right angles to each other).

### Purposes

- To decompose vectors into components parallel and perpendicular to a given direction.
- To construct orthonormal bases for vector spaces (QR decomposition, Gram-Schmidt).
- To solve least squares problems and linear regression.
- To implement dimensionality reduction (PCA uses orthogonal projections).
- To compute residuals and errors in numerical linear algebra.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Scalar projection of a onto b
scalar_proj = np.dot(a, b) / np.linalg.norm(b)

# Vector projection of a onto b
vector_proj = (np.dot(a, b) / np.dot(b, b)) * b

# Orthogonal component of a relative to b
orthogonal = a - vector_proj

# Gram-Schmidt orthogonalization (manual)
def gram_schmidt(vectors):
    basis = []
    for v in vectors:
        w = v.copy().astype(float)
        for u in basis:
            w -= np.dot(w, u) / np.dot(u, u) * u
        if np.linalg.norm(w) > 1e-10:
            basis.append(w)
    return basis
```

**Component Breakdown:**
- `np.dot(a, b) / np.linalg.norm(b)`: The scalar projection (length of the shadow) of `a` onto `b`.
- `(np.dot(a, b) / np.dot(b, b)) * b`: The vector projection of `a` onto `b`.
- `a - vector_proj`: The component of `a` orthogonal to `b`.
- `gram_schmidt(vectors)`: Iteratively orthogonalizes a list of vectors.

**Syntax Rules:**
- The denominator `np.dot(b, b)` equals `||b||²`; avoid dividing by zero for zero vectors.
- For orthonormalization, divide each orthogonal vector by its L₂ norm.
- The Gram-Schmidt process can be numerically unstable for nearly dependent vectors; modified Gram-Schmidt or Householder reflections are more stable.

**Constraints and Limitations:**
- Gram-Schmidt in floating-point arithmetic can lose orthogonality for ill-conditioned inputs.
- The projection formula assumes `b` is non-zero.
- For large-scale problems, `numpy.linalg.qr` is preferred over manual Gram-Schmidt for numerical stability.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Vector Projection

```python
import numpy as np

# Step 1: Define vectors
a = np.array([3.0, 4.0])
b = np.array([1.0, 0.0])
print(f"a = {a}")
print(f"b = {b}")

# Step 2: Scalar projection of a onto b
scalar_proj = np.dot(a, b) / np.linalg.norm(b)
print(f"\nScalar projection of a onto b: {scalar_proj}")

# Step 3: Vector projection of a onto b
vector_proj = (np.dot(a, b) / np.dot(b, b)) * b
print(f"Vector projection: {vector_proj}")

# Step 4: Orthogonal component
orthogonal = a - vector_proj
print(f"Orthogonal component: {orthogonal}")

# Step 5: Verify orthogonality
dot_check = np.dot(vector_proj, orthogonal)
print(f"\nDot product of projection and orthogonal component: {dot_check}")
print(f"Orthogonal: {np.isclose(dot_check, 0)}")

# Step 6: Verify reconstruction
reconstructed = vector_proj + orthogonal
print(f"\nReconstructed a: {reconstructed}")
print(f"Matches original: {np.allclose(reconstructed, a)}")

# Step 7: Projection onto a non-axis-aligned vector
c = np.array([1.0, 1.0])
proj_c = (np.dot(a, c) / np.dot(c, c)) * c
print(f"\nProjection of {a} onto {c}: {proj_c}")

# Step 8: Using projection to find distance from a point to a line
point = np.array([1.0, 5.0])
line_dir = np.array([1.0, 0.0])  # Line along x-axis
point_proj = (np.dot(point, line_dir) / np.dot(line_dir, line_dir)) * line_dir
distance = np.linalg.norm(point - point_proj)
print(f"\nPoint {point} projected onto line: {point_proj}")
print(f"Distance from point to line: {distance}")
```

**Expected Output:**
```
a = [3. 4.]
b = [1. 0.]

Scalar projection of a onto b: 3.0
Vector projection: [3. 0.]
Orthogonal component: [0. 4.]

Dot product of projection and orthogonal component: 0.0
Orthogonal: True

Reconstructed a: [3. 4.]
Matches original: True

Projection of [3. 4.] onto [1. 1.]: [3.5 3.5]

Point [1. 5.] projected onto line: [1. 0.]
Distance from point to line: 5.0
```

**Why This Output Occurs:** The projection of `[3, 4]` onto the x-axis `[1, 0]` is `[3, 0]` — the x-component of the original vector. The orthogonal component `[0, 4]` is perpendicular to the projection. Their dot product is zero, confirming orthogonality. The distance from `[1, 5]` to the x-axis is 5.0, which is the magnitude of the orthogonal component.

#### Example 2: Gram-Schmidt Orthogonalization

```python
import numpy as np

# Step 1: Define a set of vectors (not orthogonal)
v1 = np.array([1.0, 1.0, 0.0])
v2 = np.array([1.0, 0.0, 1.0])
v3 = np.array([0.0, 1.0, 1.0])

vectors = [v1, v2, v3]
print("Original vectors:")
for i, v in enumerate(vectors):
    print(f"  v{i+1} = {v}")

# Step 2: Gram-Schmidt orthogonalization
def gram_schmidt(vectors):
    basis = []
    for v in vectors:
        w = v.copy().astype(float)
        for u in basis:
            # Subtract projection of w onto u
            w -= (np.dot(w, u) / np.dot(u, u)) * u
        if np.linalg.norm(w) > 1e-10:
            basis.append(w)
    return basis

ortho_basis = gram_schmidt(vectors)
print("\nOrthogonal basis:")
for i, u in enumerate(ortho_basis):
    print(f"  u{i+1} = {u}")

# Step 3: Verify orthogonality
print("\nPairwise dot products (should all be 0):")
for i in range(len(ortho_basis)):
    for j in range(i+1, len(ortho_basis)):
        dot = np.dot(ortho_basis[i], ortho_basis[j])
        print(f"  u{i+1} · u{j+1} = {dot:.2e}")

# Step 4: Orthonormalize (normalize each vector)
orthonormal_basis = [u / np.linalg.norm(u) for u in ortho_basis]
print("\nOrthonormal basis:")
for i, q in enumerate(orthonormal_basis):
    print(f"  q{i+1} = {q}")
    print(f"    ||q{i+1}|| = {np.linalg.norm(q):.6f}")

# Step 5: Verify orthonormality
print("\nPairwise dot products (should all be 0):")
for i in range(len(orthonormal_basis)):
    for j in range(i+1, len(orthonormal_basis)):
        dot = np.dot(orthonormal_basis[i], orthonormal_basis[j])
        print(f"  q{i+1} · q{j+1} = {dot:.2e}")

# Step 6: Compare with numpy.linalg.qr
Q, R = np.linalg.qr(np.column_stack(vectors))
print("\nnumpy.linalg.qr orthonormal basis:")
print(Q)
print("(The QR decomposition provides a numerically stable orthonormal basis.)")
```

**Expected Output:**
```
Original vectors:
  v1 = [1. 1. 0.]
  v2 = [1. 0. 1.]
  v3 = [0. 1. 1.]

Orthogonal basis:
  u1 = [1. 1. 0.]
  u2 = [ 0.5 -0.5  1. ]
  u3 = [-0.6667  0.6667  0.6667]

Pairwise dot products (should all be 0):
  u1 · u2 = 0.00e+00
  u1 · u3 = 0.00e+00
  u2 · u3 = 0.00e+00

Orthonormal basis:
  q1 = [0.70710678 0.70710678 0.        ]
    ||q1|| = 1.000000
  q2 = [ 0.40824829 -0.40824829  0.81649658]
    ||q2|| = 1.000000
  q3 = [-0.57735027  0.57735027  0.57735027]
    ||q3|| = 1.000000

Pairwise dot products (should all be 0):
  q1 · q2 = 0.00e+00
  q1 · q3 = 0.00e+00
  q2 · q3 = 0.00e+00

numpy.linalg.qr orthonormal basis:
[[-0.70710678  0.40824829 -0.57735027]
 [-0.70710678 -0.40824829  0.57735027]
 [ 0.         -0.81649658 -0.57735027]]
(The QR decomposition provides a numerically stable orthonormal basis.)
```

**Why This Output Occurs:** Gram-Schmidt iteratively removes the components of each vector that lie along previously computed orthogonal vectors. The result is a set of mutually orthogonal vectors. Normalizing them produces an orthonormal basis. The QR decomposition provides the same orthonormal basis (up to sign) but is numerically more stable.

### Real-World Cases

- **Linear Regression:** Orthogonal projection computes the least-squares solution and residuals.
- **PCA:** Principal components are orthonormal vectors obtained via eigen-decomposition or SVD.
- **QR Decomposition:** Used in solving linear systems and eigenvalue problems.
- **Signal Processing:** Orthogonalization separates signals into independent components.
- **Computer Graphics:** Gram-Schmidt constructs orthonormal camera bases.

---

## Core Concept 5: Outer Products and Cross Products

### Definitions

**Core Definition:** The outer product of two vectors produces a matrix where each element is the product of a component from the first vector and a component from the second. The cross product of two 3D vectors produces a new vector perpendicular to both.

**Technical Definition:** For vectors `a` of length M and `b` of length N, the outer product is an M×N matrix `C` where `C[i, j] = a[i] * b[j]`. The cross product of vectors `a` and `b` in R³ is a vector `c` perpendicular to both, with magnitude `||a|| ||b|| sin(θ)` and direction given by the right-hand rule. NumPy's `np.cross` supports 2D and 3D vectors and full broadcasting for arrays of vectors.

**Beginner-Friendly Explanation:** The outer product takes two vectors and builds a table where every combination of their elements is multiplied. The cross product takes two arrows in 3D space and produces a third arrow that points straight up from the plane they form — like the axis of rotation.

### Purposes

- To construct rank-1 matrices and dyadic products in linear algebra.
- To compute torque, angular momentum, and magnetic force in physics.
- To generate grids and meshes for scientific computing and visualization.
- To implement the tensor product in multilinear algebra.
- To compute surface normals in computer graphics and geometry.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Outer product
outer_result = np.outer(a, b)

# Cross product (3D vectors)
cross_result = np.cross(a, b)

# Cross product with axis control
cross_result = np.cross(a, b, axisa=-1, axisb=-1, axisc=-1)

# Multiple cross products (arrays of vectors)
cross_result = np.cross(A, B)  # A, B shape (..., 3)
```

**Component Breakdown:**
- `np.outer(a, b)`: Flattens inputs to 1-D, returns an M×N array where `out[i, j] = a[i] * b[j]`.
- `np.cross(a, b)`: Returns the cross product of two 3D vectors. For 2D vectors, the z-component is assumed zero and the result is a scalar (the z-component of the 3D cross product).
- `axisa`, `axisb`, `axisc`: Control which axis of the input arrays defines the vector components.
- `axis`: Overrides all three axis parameters.

**Syntax Rules:**
- `np.outer` flattens inputs if they are not already 1-D.
- `np.cross` requires vectors of dimension 2 or 3; higher dimensions raise `ValueError`.
- For arrays of vectors, the vector dimension is the last axis by default.
- Full broadcasting is supported for `np.cross` with array inputs.

**Constraints and Limitations:**
- `np.outer` always returns a 2-D array, even if inputs are scalars.
- `np.cross` raises `ValueError` if the vector dimension is not 2 or 3.
- `np.cross` for 2D vectors returns a scalar (the z-component), not a 3D vector.
- `np.linalg.outer` (NumPy 2.0+) accepts only 1-D inputs.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Outer Products

```python
import numpy as np

# Step 1: Basic outer product of two vectors
a = np.array([1, 2, 3])
b = np.array([4, 5])
print(f"a = {a} (shape {a.shape})")
print(f"b = {b} (shape {b.shape})")

outer = np.outer(a, b)
print(f"\nOuter product (shape {outer.shape}):")
print(outer)

# Step 2: Verify element-wise
print("\nManual verification:")
for i in range(len(a)):
    for j in range(len(b)):
        print(f"  outer[{i},{j}] = {a[i]} * {b[j]} = {a[i]*b[j]}")

# Step 3: Outer product as rank-1 matrix
u = np.array([1.0, 2.0, 3.0])
v = np.array([4.0, 5.0, 6.0])
rank1 = np.outer(u, v)
print(f"\nRank of outer product: {np.linalg.matrix_rank(rank1)}")

# Step 4: Outer product for grid generation
x = np.linspace(0, 1, 5)
y = np.linspace(0, 1, 4)
X, Y = np.meshgrid(x, y)  # Equivalent to outer products
print(f"\nGrid X shape: {X.shape}")
print(f"Grid Y shape: {Y.shape}")

# Step 5: Outer product with broadcasting
result = np.outer(a, b)
manual_broadcast = a[:, np.newaxis] * b[np.newaxis, :]
print(f"\nOuter product matches broadcasting: {np.array_equal(result, manual_broadcast)}")

# Step 6: Outer product for multiple vectors (batched)
vectors_a = np.array([[1, 2], [3, 4]])  # 2 vectors of length 2
vectors_b = np.array([[5, 6], [7, 8]])  # 2 vectors of length 2

# Manual batched outer product
batched_outer = vectors_a[:, :, np.newaxis] * vectors_b[:, np.newaxis, :]
print(f"\nBatched outer product shape: {batched_outer.shape}")
print(batched_outer)
```

**Expected Output:**
```
a = [1 2 3] (shape (3,))
b = [4 5] (shape (2,))

Outer product (shape (3, 2)):
[[ 4  5]
 [ 8 10]
 [12 15]]

Manual verification:
  outer[0,0] = 1 * 4 = 4
  outer[0,1] = 1 * 5 = 5
  outer[1,0] = 2 * 4 = 8
  outer[1,1] = 2 * 5 = 10
  outer[2,0] = 3 * 4 = 12
  outer[2,1] = 3 * 5 = 15

Rank of outer product: 1

Grid X shape: (4, 5)
Grid Y shape: (4, 5)

Outer product matches broadcasting: True

Batched outer product shape: (2, 2, 2)
[[[ 5  6]
  [10 12]]
 [[21 24]
  [28 32]]]
```

**Why This Output Occurs:** The outer product of a length-3 vector and a length-2 vector produces a 3×2 matrix. Each element is the product of the corresponding components. The outer product of two non-zero vectors always has rank 1 because every row is a scalar multiple of the first row. The batched version uses broadcasting to compute multiple outer products simultaneously.

#### Example 2: Cross Products

```python
import numpy as np

# Step 1: Basic 3D cross product
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
cross = np.cross(a, b)
print(f"a = {a}")
print(f"b = {b}")
print(f"a × b = {cross}")

# Step 2: Verify perpendicularity
dot_a = np.dot(cross, a)
dot_b = np.dot(cross, b)
print(f"\ncross · a = {dot_a} (should be 0)")
print(f"cross · b = {dot_b} (should be 0)")
print(f"Perpendicular to both: {dot_a == 0 and dot_b == 0}")

# Step 3: Cross product magnitude equals area of parallelogram
magnitude = np.linalg.norm(cross)
print(f"\n||a × b|| = {magnitude:.4f}")
print("This equals the area of the parallelogram formed by a and b.")

# Step 4: 2D cross product (returns z-component as scalar)
a_2d = np.array([1, 2])
b_2d = np.array([3, 4])
cross_2d = np.cross(a_2d, b_2d)
print(f"\n2D cross product: {cross_2d}")
print("(This is the z-component of the 3D cross product with z=0.)")

# Step 5: Multiple cross products (arrays of vectors)
A = np.array([[1, 2, 3], [4, 5, 6]])
B = np.array([[4, 5, 6], [1, 2, 3]])
cross_batch = np.cross(A, B)
print(f"\nBatch cross products (shape {cross_batch.shape}):")
print(cross_batch)

# Step 6: Axis control — compute cross product along a different axis
A_axis = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])  # Vectors along axis=0
B_axis = np.array([[7, 8, 9], [4, 5, 6], [1, 2, 3]])
cross_axis = np.cross(A_axis, B_axis, axisa=0, axisb=0, axisc=0)
print(f"\nCross product along axis=0:\n{cross_axis}")

# Step 7: Physical application — torque calculation
r = np.array([1.0, 0.0, 0.0])  # Position vector (1 m in x)
F = np.array([0.0, 10.0, 0.0]) # Force vector (10 N in y)
torque = np.cross(r, F)
print(f"\nPosition: {r}")
print(f"Force: {F}")
print(f"Torque (r × F): {torque} N·m")
print("(Torque is perpendicular to both r and F, pointing in +z.)")
```

**Expected Output:**
```
a = [1 2 3]
b = [4 5 6]
a × b = [-3  6 -3]

cross · a = 0 (should be 0)
cross · b = 0 (should be 0)
Perpendicular to both: True

||a × b|| = 7.3485
This equals the area of the parallelogram formed by a and b.

2D cross product: -2
(This is the z-component of the 3D cross product with z=0.)

Batch cross products (shape (2, 3)):
[[-3  6 -3]
 [ 3 -6  3]]

Cross product along axis=0:
[[-3  6 -3]
 [ 3 -6  3]
 [-3  6 -3]]

Position: [1. 0. 0.]
Force: [0. 10. 0.]
Torque (r × F): [ 0.  0. 10.] N·m
(Torque is perpendicular to both r and F, pointing in +z.)
```

**Why This Output Occurs:** The cross product `[1,2,3] × [4,5,6]` = `[2*6-3*5, 3*4-1*6, 1*5-2*4]` = `[-3, 6, -3]`. It is perpendicular to both input vectors, confirmed by zero dot products. The 2D cross product returns a scalar (the z-component). The torque calculation shows that a force applied at a distance produces a torque perpendicular to both the position and force vectors.

### Real-World Cases

- **Physics:** Torque = r × F; angular momentum = r × p; Lorentz force = q(v × B).
- **Computer Graphics:** Surface normals via cross products; back-face culling; reflection vectors.
- **Robotics:** Jacobian matrices for velocity and force transformations.
- **Electromagnetism:** Poynting vector = E × H.
- **Linear Algebra:** Rank-1 updates via outer products; covariance matrix = outer product of centered data.
- **Scientific Computing:** Outer products for constructing finite difference operators and grid interpolation.

---

## References

1. **NumPy Linear Algebra (Official Documentation)** — https://numpy.org/doc/stable/reference/routines.linalg.html
2. **numpy.linalg.norm (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.norm.html
3. **numpy.dot (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.dot.html
4. **numpy.inner (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.inner.html
5. **numpy.vdot (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.vdot.html
6. **numpy.vecdot (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.vecdot.html
7. **numpy.outer (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.outer.html
8. **numpy.linalg.outer (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.outer.html
9. **numpy.cross (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.cross.html
10. **numpy.linalg.cross (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.cross.html
11. **G. H. Golub and C. F. Van Loan, Matrix Computations, 3rd ed., Johns Hopkins University Press, 1996** — Standard reference for norms, projections, and orthogonalization.
12. **NumPy Broadcasting (Official Documentation)** — https://numpy.org/doc/stable/user/basics.broadcasting.html
13. **NumPy "Linear Algebra" User Guide** — https://numpy.org/doc/stable/user/absolute_beginners.html#linear-algebra
14. **SciPy Linear Algebra (Official Documentation)** — https://docs.scipy.org/doc/scipy/reference/linalg.html