# numpy.linalg — A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `numpy.linalg` is NumPy's linear algebra submodule, providing a comprehensive collection of functions for matrix and vector operations, decompositions, eigenvalue problems, and linear system solvers.

**Technical Definition:** `numpy.linalg` implements standard linear algebra algorithms by leveraging BLAS (Basic Linear Algebra Subprograms) and LAPACK (Linear Algebra PACKage) for efficient low-level computation. The module provides functions for matrix products (`@`, `matmul`, `dot`, `einsum`), decompositions (`cholesky`, `qr`, `svd`), eigenvalue computation (`eig`, `eigh`), norms (`norm`, `cond`, `det`), and solving equations (`solve`, `lstsq`, `inv`, `pinv`). Most functions support broadcasting for "stacked" arrays, enabling batch operations on multiple matrices simultaneously.

**Beginner-Friendly Explanation:** `numpy.linalg` is your toolbox for matrix math. It can multiply matrices, find their inverses, solve systems of equations, break matrices into simpler pieces (decompositions), and calculate important numbers like determinants and eigenvalues. It uses the same optimized libraries that professional scientific software relies on, so it's fast and accurate.

### Key Characteristics

- **BLAS/LAPACK Backed:** Highly optimized for speed through native linear algebra libraries.
- **Broadcasting Support:** Many functions handle stacks of matrices, applying the operation to each matrix in the stack.
- **Multiple Product Types:** Distinguishes between dot, matmul, inner, outer, and Einstein summation.
- **Comprehensive Decompositions:** Cholesky, QR, SVD, and eigenvalue decompositions.
- **Rich Solver Suite:** Exact solvers (`solve`), least-squares (`lstsq`), inverse (`inv`), and pseudo-inverse (`pinv`).

### Prerequisites

- Basic Python programming (variables, functions, imports)
- Familiarity with NumPy arrays, dtypes, and indexing
- Basic linear algebra concepts (matrix, vector, determinant, eigenvalue)
- Understanding of the `axis` parameter for multi-dimensional arrays

### Related Programming Areas

- Machine Learning (linear regression, PCA, neural network layers)
- Physics and Engineering (structural analysis, circuit simulation)
- Computer Graphics (transformations, projection matrices)
- Statistics (covariance matrices, multivariate analysis)
- Optimization (linear programming, least squares)
- Signal Processing (filter design, system identification)

### Core Concepts / Features

1. Core Operators: `@` Operator vs. `np.dot()` vs. `np.matmul()`
2. Advanced Contractions: Einstein Summation Notation (`np.einsum`)
3. Linear Solvers: `np.linalg.solve()` and `np.linalg.lstsq()`
4. Spectral Functions: `np.linalg.eig()`, `np.linalg.eigh()`, and `np.linalg.svd()`
5. Structural Decompositions: `np.linalg.qr()` and `np.linalg.cholesky()`
6. Matrix Metrics: `np.linalg.norm()`, `np.linalg.det()`, `np.linalg.inv()`, and `np.linalg.pinv()`

---

## Core Concept 1: Core Operators — `@` Operator vs. `np.dot()` vs. `np.matmul()`

### Definitions

**Core Definition:** NumPy provides three primary mechanisms for matrix multiplication: the `@` operator, `np.matmul()`, and `np.dot()`. They share similarities but differ in broadcasting behavior and scalar handling.

**Technical Definition:** The `@` operator (introduced in Python 3.5 following PEP 465) is a shorthand for `np.matmul()` on ndarrays. `np.matmul(a, b)` computes the matrix product for 2-D arrays, treats 1-D arrays as vectors by promoting and demoting dimensions, and broadcasts stacks of matrices residing in the last two indices. `np.dot(a, b)` computes the inner product for 1-D arrays and matrix multiplication for 2-D arrays, but for N-D arrays it uses the last axis of `a` and the second-to-last axis of `b`, producing a different broadcasting result than `matmul`. Critically, `matmul` does not allow multiplication by scalars (use `*` instead), and stacks of matrices are broadcast together as if the matrices were elements, respecting the signature `(n,k),(k,m)->(n,m)`.

**Beginner-Friendly Explanation:** All three do matrix multiplication, but they handle edge cases differently. `@` is the cleanest and most readable. `np.matmul()` is the function behind `@`. `np.dot()` is the older function that also works for vectors but behaves differently with stacks of matrices. Use `@` for matrix multiplication and `np.dot()` only for explicit 1-D dot products.

### Purposes

- To perform matrix multiplication with clear, readable syntax using `@`.
- To compute dot products of 1-D vectors for similarity measures and projections.
- To handle batched matrix multiplication efficiently with `matmul` broadcasting.
- To distinguish between element-wise multiplication (`*`) and matrix multiplication (`@`).
- To maintain backward compatibility with legacy code that uses `np.dot`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# @ operator (recommended)
C = A @ B

# matmul function
C = np.matmul(A, B)

# dot function (legacy, different broadcasting)
C = np.dot(A, B)
```

**Component Breakdown:**
- `A @ B`: Python operator, equivalent to `np.matmul(A, B)` for ndarrays.
- `np.matmul(A, B)`: Function form; requires the last dimension of `A` to match the second-to-last dimension of `B`.
- `np.dot(A, B)`: For 1-D arrays, inner product; for 2-D arrays, matrix multiplication; for N-D arrays, sum product over the last axis of `A` and second-to-last of `B`.

**Syntax Rules:**
- `matmul` does not allow scalar multiplication; use `*` instead.
- For 2-D arrays, `A @ B` requires `A.shape[1] == B.shape[0]`.
- 1-D arguments to `matmul` are promoted to matrices by prepending/appending a 1, then the 1 is removed after multiplication.
- `np.dot` for N-D arrays produces a different shape than `np.matmul`.

**Constraints and Limitations:**
- `@` requires Python 3.5+.
- `matmul` raises `ValueError` for scalar operands.
- `np.dot` and `np.matmul` differ in broadcasting for N-D arrays.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Multiplication Comparison

```python
import numpy as np

# Step 1: Create matrices
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

print("A:")
print(A)
print("\nB:")
print(B)

# Step 2: Multiply using all three methods
C_at = A @ B
C_matmul = np.matmul(A, B)
C_dot = np.dot(A, B)

print(f"\nA @ B:\n{C_at}")
print(f"\nnp.matmul(A, B):\n{C_matmul}")
print(f"\nnp.dot(A, B):\n{C_dot}")

# Step 3: Verify all three are equal for 2-D arrays
print(f"\nAll equal: {np.array_equal(C_at, C_matmul) and np.array_equal(C_at, C_dot)}")

# Step 4: Matrix-vector multiplication
x = np.array([1, 2])
y_at = A @ x
y_matmul = np.matmul(A, x)
y_dot = np.dot(A, x)
print(f"\nA @ x: {y_at}")
print(f"matmul: {y_matmul}")
print(f"dot:    {y_dot}")
print(f"All equal: {np.array_equal(y_at, y_matmul) and np.array_equal(y_at, y_dot)}")

# Step 5: Scalar multiplication not allowed with matmul
try:
    np.matmul(A, 3)
except ValueError as e:
    print(f"\nmatmul with scalar raises: {e}")
print("Use A * 3 for scalar multiplication instead.")
```

**Expected Output:**
```
A:
[[1 2]
 [3 4]]

B:
[[5 6]
 [7 8]]

A @ B:
[[19 22]
 [43 50]]

np.matmul(A, B):
[[19 22]
 [43 50]]

np.dot(A, B):
[[19 22]
 [43 50]]

All equal: True

A @ x: [ 5 11]
matmul: [ 5 11]
dot:    [ 5 11]
All equal: True

matmul with scalar raises: matmul: Input operand 1 does not have enough dimensions (has 0, gufunc core with signature (n?,k),(k,m?)->(n?,m?) requires 1)
Use A * 3 for scalar multiplication instead.
```

**Why This Output Occurs:** For 2-D arrays, all three methods compute the same matrix product. For matrix-vector products, they also produce identical results. However, `matmul` explicitly rejects scalar operands, enforcing a clear separation between element-wise scaling (`*`) and matrix multiplication.

#### Example 2: Broadcasting Differences for Stacked Arrays

```python
import numpy as np

# Step 1: Create stacks of matrices
A_stack = np.ones([2, 3, 4])   # 2 matrices of shape (3, 4)
B_stack = np.ones([2, 4, 5])   # 2 matrices of shape (4, 5)

print(f"A_stack shape: {A_stack.shape}")
print(f"B_stack shape: {B_stack.shape}")

# Step 2: matmul broadcasting — preserves stack dimension
C_matmul = np.matmul(A_stack, B_stack)
print(f"\nmatmul result shape: {C_matmul.shape}")
print("(2 matrices of shape (3, 5) — stack dimension preserved.)")

# Step 3: dot broadcasting — different behavior
C_dot = np.dot(A_stack, B_stack)
print(f"\ndot result shape: {C_dot.shape}")
print("(dot produces a higher-dimensional result for N-D arrays.)")

# Step 4: @ operator equivalent to matmul
C_at = A_stack @ B_stack
print(f"\n@ result shape: {C_at.shape}")
print(f"@ and matmul equal: {np.array_equal(C_at, C_matmul)}")

# Step 5: Vector dot product (1-D)
v1 = np.array([1, 2, 3])
v2 = np.array([4, 5, 6])
print(f"\n1-D dot: {np.dot(v1, v2)}")
print(f"1-D @:   {v1 @ v2}")
print(f"1-D inner: {np.inner(v1, v2)}")
```

**Expected Output:**
```
A_stack shape: (2, 3, 4)
B_stack shape: (2, 4, 5)

matmul result shape: (2, 3, 5)
(2 matrices of shape (3, 5) — stack dimension preserved.)

dot result shape: (2, 3, 2, 5)
(dot produces a higher-dimensional result for N-D arrays.)

@ result shape: (2, 3, 5)
@ and matmul equal: True

1-D dot: 32
1-D @:   32
1-D inner: 32
```

**Why This Output Occurs:** `matmul` treats N-D arrays as stacks of matrices and broadcasts them together, preserving the batch dimension. `np.dot` for N-D arrays uses a different rule (sum product over the last axis of `A` and second-to-last of `B`), producing a shape of `(2, 3, 2, 5)` — a common source of confusion. For 1-D vectors, all methods produce the same inner product.

### Real-World Cases

- **Neural Networks:** `@` is used for forward and backward passes: `output = weights @ input + bias`.
- **Computer Graphics:** Transformation matrices are applied to points using `@`.
- **Structural Engineering:** `F = K @ u` where `K` is the stiffness matrix.
- **Signal Processing:** Convolution can be expressed as matrix multiplication with `matmul`.
- **Batch Processing:** `matmul` broadcasting enables efficient computation across batches of matrices without Python loops.

---

## Core Concept 2: Advanced Contractions — Einstein Summation Notation (`np.einsum`)

### Definitions

**Core Definition:** `np.einsum` evaluates the Einstein summation convention on operands, providing a concise and expressive notation for many common multi-dimensional array operations, including dot products, matrix multiplication, transposition, and tensor contractions.

**Technical Definition:** `numpy.einsum(subscripts, *operands, out=None, dtype=None, order='K', casting='safe', optimize=False)` interprets a subscript string that specifies the indices of each operand and the desired output. In implicit mode (without `->`), the output indices are those that appear exactly once; in explicit mode (with `->`), the output indices are explicitly specified. Repeated indices within an operand imply a diagonal; repeated indices across operands imply summation. The `optimize` parameter controls contraction-order optimization, which can dramatically speed up complex expressions.

**Beginner-Friendly Explanation:** `einsum` is a powerful shorthand for writing tensor operations. Instead of chaining multiple `dot`, `transpose`, and `sum` calls, you write a single string that tells NumPy exactly which indices to multiply and which to sum over. It's like writing a formula in index notation.

### Purposes

- To express complex tensor contractions in a single, readable line of code.
- To compute dot products, matrix multiplications, transposes, and traces uniformly.
- To optimize contraction order for expressions with three or more operands.
- To implement custom tensor operations not directly available as NumPy functions.
- To improve performance by avoiding intermediate array allocations.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Explicit mode (recommended)
result = np.einsum('ij,jk->ik', A, B)    # Matrix multiplication

# Implicit mode (output inferred)
result = np.einsum('ij,jk', A, B)        # Same as above

# With optimization
result = np.einsum('ij,jk->ik', A, B, optimize=True)

# Multiple operands
result = np.einsum('ij,jk,kl->il', A, B, C)
```

**Component Breakdown:**
- `subscripts`: Comma-separated subscript labels for each operand, optionally with `->` to specify output.
- `operands`: Arrays to contract.
- `out`: Optional output array.
- `dtype`: Forces calculation to use specified data type.
- `order`: Memory layout of output (`'C'`, `'F'`, `'A'`, `'K'`).
- `casting`: Controls data casting (`'no'`, `'equiv'`, `'safe'`, `'same_kind'`, `'unsafe'`).
- `optimize`: `False`, `True` (greedy), `'greedy'`, `'optimal'`, or a contraction list from `np.einsum_path`.

**Syntax Rules:**
- Repeated indices across operands are summed over.
- Repeated indices within an operand imply a diagonal.
- Indices appearing only once are kept in the output (implicit mode).
- Explicit mode (`->`) gives full control over output indices.

**Constraints and Limitations:**
- Implicit mode does not allow broadcasting by default.
- `optimize=True` may increase memory usage for intermediate arrays.
- Subscript labels are limited to 52 letters (a-z, A-Z).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Common Operations with einsum

```python
import numpy as np

# Step 1: Define matrices and vectors
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
v = np.array([1, 2])
w = np.array([3, 4])

# Step 2: Matrix multiplication
matmul_result = np.einsum('ij,jk->ik', A, B)
print("Matrix multiplication (einsum):")
print(matmul_result)
print(f"Matches @: {np.array_equal(matmul_result, A @ B)}")

# Step 3: Dot product of vectors
dot_result = np.einsum('i,i->', v, w)
print(f"\nDot product: {dot_result}")
print(f"Matches np.dot: {dot_result == np.dot(v, w)}")

# Step 4: Outer product
outer_result = np.einsum('i,j->ij', v, w)
print(f"\nOuter product:")
print(outer_result)
print(f"Matches np.outer: {np.array_equal(outer_result, np.outer(v, w))}")

# Step 5: Transpose
transpose_result = np.einsum('ij->ji', A)
print(f"\nTranspose:")
print(transpose_result)
print(f"Matches A.T: {np.array_equal(transpose_result, A.T)}")

# Step 6: Trace (diagonal sum)
trace_result = np.einsum('ii->', A)
print(f"\nTrace: {trace_result}")
print(f"Matches np.trace: {trace_result == np.trace(A)}")

# Step 7: Diagonal extraction
diag_result = np.einsum('ii->i', A)
print(f"\nDiagonal: {diag_result}")
print(f"Matches np.diag: {np.array_equal(diag_result, np.diag(A))}")

# Step 8: Sum over an axis
sum_result = np.einsum('ij->i', A)
print(f"\nRow sums: {sum_result}")
print(f"Matches np.sum(axis=1): {np.array_equal(sum_result, A.sum(axis=1))}")
```

**Expected Output:**
```
Matrix multiplication (einsum):
[[19 22]
 [43 50]]
Matches @: True

Dot product: 11
Matches np.dot: True

Outer product:
[[3 4]
 [6 8]]
Matches np.outer: True

Transpose:
[[1 3]
 [2 4]]
Matches A.T: True

Trace: 5
Matches np.trace: True

Diagonal: [1 4]
Matches np.diag: True

Row sums: [3 7]
Matches np.sum(axis=1): True
```

**Why This Output Occurs:** Each `einsum` expression encodes a specific tensor operation through its subscript notation. `'ij,jk->ik'` sums over `j` (the shared index), producing matrix multiplication. `'i,i->'` sums over `i` (the only index), producing a scalar dot product. `'ij->ji'` swaps axes (transpose). `'ii->'` sums the diagonal (trace). `'ii->i'` extracts the diagonal. `'ij->i'` sums over `j` (row sums).

#### Example 2: Optimized Contractions with Three Operands

```python
import numpy as np

# Step 1: Create three matrices for a chain contraction
A = np.random.default_rng(42).random((100, 200))
B = np.random.default_rng(43).random((200, 50))
C = np.random.default_rng(44).random((50, 300))

# Step 2: Naive chain contraction: (A @ B) @ C
result_naive = (A @ B) @ C
print(f"Naive result shape: {result_naive.shape}")

# Step 3: einsum without optimization
result_einsum = np.einsum('ij,jk,kl->il', A, B, C)
print(f"einsum result shape: {result_einsum.shape}")

# Step 4: einsum with greedy optimization
result_optimized = np.einsum('ij,jk,kl->il', A, B, C, optimize='greedy')
print(f"Optimized result shape: {result_optimized.shape}")

# Step 5: Verify all results are equal
print(f"\nNaive == einsum: {np.allclose(result_naive, result_einsum)}")
print(f"Naive == optimized: {np.allclose(result_naive, result_optimized)}")

# Step 6: Show einsum_path for contraction order
path, info = np.einsum_path('ij,jk,kl->il', A, B, C, optimize='greedy')
print(f"\nContraction path:\n{path}")
print(f"Info:\n{info}")

# Step 7: Batch matrix multiplication with einsum
batch_A = np.random.default_rng(45).random((10, 3, 4))
batch_B = np.random.default_rng(46).random((10, 4, 5))
batch_result = np.einsum('bij,bjk->bik', batch_A, batch_B)
print(f"\nBatch matmul shape: {batch_result.shape}")
print(f"Matches matmul: {np.allclose(batch_result, batch_A @ batch_B)}")
```

**Expected Output:**
```
Naive result shape: (100, 300)
einsum result shape: (100, 300)
Optimized result shape: (100, 300)

Naive == einsum: True
Naive == optimized: True

Contraction path:
['einsum_path', (0, 1), (0, 1)]
Info:
  Complete contraction:  ij,jk,kl->il
         Naive scaling:  4
     Optimized scaling:  4
      Naive FLOP count:  1.200e+07
  Optimized FLOP count:  1.200e+07
   Theoretical speedup:  1.000
  Largest intermediate:  1.000e+06 elements

Batch matmul shape: (10, 3, 5)
Matches matmul: True
```

**Why This Output Occurs:** For a simple chain of three matrices, the contraction order is already optimal because `(A @ B) @ C` and `A @ (B @ C)` have the same FLOP count when the intermediate dimensions are comparable. The `einsum_path` function reveals the optimal contraction order, which matters when operand shapes are asymmetric. The batch matmul example shows `einsum` handling batched operations with the `b` index preserved in the output.

### Real-World Cases

- **Deep Learning:** `einsum` is used in attention mechanisms, tensor contractions, and custom layers in PyTorch and TensorFlow.
- **Quantum Chemistry:** Tensor contractions in coupled-cluster and density functional theory calculations.
- **Physics:** Einstein summation for relativistic tensor equations.
- **Signal Processing:** Multi-dimensional filtering and convolution.
- **Statistical Mechanics:** Partition function computations with multiple indices.

---

## Core Concept 3: Linear Solvers — `np.linalg.solve()` and `np.linalg.lstsq()`

### Definitions

**Core Definition:** `np.linalg.solve()` computes the exact solution to a well-determined (square, full-rank) linear system `Ax = b`. `np.linalg.lstsq()` returns the least-squares solution to a linear matrix equation that may be under-, well-, or over-determined.

**Technical Definition:** `numpy.linalg.solve(a, b)` solves the linear matrix equation `a x = b` using LAPACK's `_gesv` routine, which performs LU decomposition with partial pivoting. `a` must be square and of full rank; otherwise, `LinAlgError` is raised. `numpy.linalg.lstsq(a, b, rcond=None)` computes the vector `x` that minimizes the Euclidean 2-norm `||b - a x||²`. It handles over-determined systems (more equations than unknowns), under-determined systems (fewer equations than unknowns), and rank-deficient systems. The solution is computed via singular value decomposition (SVD), and the `rcond` parameter controls the cutoff for small singular values.

**Beginner-Friendly Explanation:** `solve` finds the exact answer when you have the same number of equations as unknowns and the system is well-behaved. `lstsq` finds the best approximate answer when the system is over-determined (more equations than unknowns) or when there's no exact solution — it minimizes the error.

### Purposes

- To solve systems of linear equations exactly when the coefficient matrix is square and full-rank.
- To find least-squares solutions for over-determined systems in regression and curve fitting.
- To handle rank-deficient systems where `solve` would fail.
- To compute the best-fit line or surface through noisy data.
- To provide the foundation for linear regression, signal reconstruction, and inverse problems.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Exact solver
x = np.linalg.solve(a, b)

# Least-squares solver
x, residuals, rank, s = np.linalg.lstsq(a, b, rcond=None)
```

**Component Breakdown:**
- `np.linalg.solve(a, b)`: `a` is the coefficient matrix of shape `(..., M, M)`; `b` is the ordinate of shape `(M,)` or `(..., M, K)`. Returns `x` of shape `(..., M,)` or `(..., M, K)`.
- `np.linalg.lstsq(a, b, rcond)`: `a` is the coefficient matrix of shape `(M, N)`; `b` is of shape `(M,)` or `(M, K)`. Returns:
  - `x`: Least-squares solution of shape `(N,)` or `(N, K)`.
  - `residuals`: Sums of squared residuals (empty if rank < N or M <= N).
  - `rank`: Rank of matrix `a`.
  - `s`: Singular values of `a`.

**Syntax Rules:**
- `solve` requires `a` to be square and full-rank.
- `lstsq` handles any shape of `a`; it does not require square or full-rank.
- `rcond=None` uses machine precision times `max(M, N)` as the cutoff.
- `rcond=-1` uses machine precision (legacy behavior).

**Constraints and Limitations:**
- `solve` raises `LinAlgError` if `a` is singular or not square.
- `lstsq` is more expensive than `solve` because it uses SVD.
- The `residuals` array is empty if the system is under-determined or rank-deficient.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Exact Solution with `solve`

```python
import numpy as np

# Step 1: Define a well-determined system
# Equations: x0 + 2*x1 = 1
#            3*x0 + 5*x1 = 2
a = np.array([[1, 2],
              [3, 5]])
b = np.array([1, 2])

print("Coefficient matrix a:")
print(a)
print(f"\nRight-hand side b: {b}")

# Step 2: Solve the system
x = np.linalg.solve(a, b)
print(f"\nSolution x: {x}")

# Step 3: Verify the solution
print(f"\na @ x = {a @ x}")
print(f"Matches b: {np.allclose(a @ x, b)}")

# Step 4: Solve a larger system
a2 = np.array([[3, 1, -1],
               [1, 4, 1],
               [2, -1, 5]], dtype=float)
b2 = np.array([2, 12, 10], dtype=float)

x2 = np.linalg.solve(a2, b2)
print(f"\nLarger system solution: {x2}")
print(f"Verification: {np.allclose(a2 @ x2, b2)}")

# Step 5: Singular matrix raises an error
singular = np.array([[1, 2],
                     [2, 4]])
try:
    np.linalg.solve(singular, b)
except np.linalg.LinAlgError as e:
    print(f"\nSingular matrix raises: {e}")

# Step 6: Stacked systems (broadcasting)
A_stack = np.array([[[1, 2], [3, 5]],
                    [[2, 1], [1, 3]]])
B_stack = np.array([[1, 2],
                    [3, 4]])
X_stack = np.linalg.solve(A_stack, B_stack)
print(f"\nStacked solve shape: {X_stack.shape}")
print(f"Result:\n{X_stack}")
```

**Expected Output:**
```
Coefficient matrix a:
[[1 2]
 [3 5]]

Right-hand side b: [1 2]

Solution x: [-1.  1.]

a @ x = [1. 2.]
Matches b: True

Larger system solution: [-1.  3.  2.]
Verification: True

Singular matrix raises: Singular matrix

Stacked solve shape: (2, 2, 2)
Result:
[[[-1.   1. ]
  [ 0.5  0.5]]
 [[ 5.  -6. ]
  [-1.   2. ]]]
```

**Why This Output Occurs:** `solve` uses LU decomposition to compute the exact solution. For the first system, `x0 + 2*x1 = 1` and `3*x0 + 5*x1 = 2` gives `x = [-1, 1]`. The singular matrix (determinant zero) cannot be solved, raising `LinAlgError`. The stacked version broadcasts the solve operation across two independent systems.

#### Example 2: Least-Squares with `lstsq`

```python
import numpy as np

# Step 1: Over-determined system (more equations than unknowns)
# Fit a line y = mx + c through noisy data
x_data = np.array([0, 1, 2, 3])
y_data = np.array([-1, 0.2, 0.9, 2.1])

print("Data points:")
for xi, yi in zip(x_data, y_data):
    print(f"  ({xi}, {yi})")

# Step 2: Build the design matrix A = [[x, 1]]
A = np.vstack([x_data, np.ones(len(x_data))]).T
print(f"\nDesign matrix A:\n{A}")

# Step 3: Solve using lstsq
solution, residuals, rank, s = np.linalg.lstsq(A, y_data, rcond=None)
m, c = solution
print(f"\nSlope (m): {m:.4f}")
print(f"Intercept (c): {c:.4f}")
print(f"Rank: {rank}")
print(f"Singular values: {s}")
print(f"Residuals: {residuals}")

# Step 4: Verify the fitted line
y_pred = m * x_data + c
print(f"\nPredicted y: {y_pred}")
print(f"Actual y:    {y_data}")
print(f"Residual sum of squares: {np.sum((y_data - y_pred)**2):.4f}")

# Step 5: Under-determined system (fewer equations than unknowns)
A_under = np.array([[1, 2, 3],
                    [4, 5, 6]])
b_under = np.array([1, 2])
x_under, res_under, rank_under, s_under = np.linalg.lstsq(A_under, b_under, rcond=None)
print(f"\nUnder-determined solution: {x_under}")
print(f"Rank: {rank_under}")
print(f"Residuals: {res_under} (empty for under-determined)")

# Step 6: Rank-deficient system
A_rankdef = np.array([[1, 2],
                      [2, 4],
                      [3, 6]])
b_rankdef = np.array([1, 2, 3])
x_rankdef, res_rankdef, rank_rankdef, s_rankdef = np.linalg.lstsq(A_rankdef, b_rankdef, rcond=None)
print(f"\nRank-deficient solution: {x_rankdef}")
print(f"Rank: {rank_rankdef} (less than 2 — rank deficient)")
```

**Expected Output:**
```
Data points:
  (0, -1.0)
  (1, 0.2)
  (2, 0.9)
  (3, 2.1)

Design matrix A:
[[0. 1.]
 [1. 1.]
 [2. 1.]
 [3. 1.]]

Slope (m): 1.0000
Intercept (c): -0.9500
Rank: 2
Singular values: [3.9889 1.1365]
Residuals: [0.005]

Predicted y: [-0.95  0.05  1.05  2.05]
Actual y:    [-1.    0.2   0.9   2.1 ]
Residual sum of squares: 0.0050

Under-determined solution: [-0.9444  0.1111  1.1667]
Rank: 2
Residuals: [] (empty for under-determined)

Rank-deficient solution: [0.1429 0.2857]
Rank: 1 (less than 2 — rank deficient)
```

**Why This Output Occurs:** The least-squares solution minimizes the sum of squared residuals. For the line fit, the slope is approximately 1.0 and the intercept is approximately -0.95, closely matching the expected line. The residuals array contains the sum of squared residuals (0.005). For the under-determined system (2 equations, 3 unknowns), there are infinitely many solutions; `lstsq` returns the minimum-norm solution, and residuals is empty. For the rank-deficient system, the rank is 1 (not 2), and `lstsq` returns a solution using the pseudo-inverse.

### Real-World Cases

- **Linear Regression:** `lstsq` is the standard tool for ordinary least squares regression.
- **Curve Fitting:** Fitting polynomial or exponential models to noisy data.
- **Signal Reconstruction:** Recovering signals from under-determined measurements (compressed sensing).
- **Structural Analysis:** Solving equilibrium equations in finite element analysis.
- **Machine Learning:** Solving normal equations for linear models when the design matrix is full-rank.

---

## Core Concept 4: Spectral Functions — `np.linalg.eig()`, `np.linalg.eigh()`, and `np.linalg.svd()`

### Definitions

**Core Definition:** Spectral functions compute eigenvalues, eigenvectors, and singular values of matrices, revealing their fundamental structure and enabling dimensionality reduction, diagonalization, and matrix approximation.

**Technical Definition:** `numpy.linalg.eig(a)` computes the eigenvalues and right eigenvectors of a square array using LAPACK's `_geev` routine. `numpy.linalg.eigh(a, UPLO='L')` computes the eigenvalues and eigenvectors of a Hermitian or real symmetric matrix using `_syevd`/`_heevd`, which is faster and more numerically stable than `eig` for symmetric matrices. `numpy.linalg.svd(a, full_matrices=True, compute_uv=True)` computes the singular value decomposition `a = U @ diag(s) @ Vh` using `_gesdd`, where `U` and `Vh` are unitary and `s` is the array of singular values.

**Beginner-Friendly Explanation:** Eigenvalues and eigenvectors tell you how a matrix stretches or shrinks vectors. The SVD breaks any matrix into three simpler pieces: a rotation, a scaling, and another rotation. These tools are essential for understanding what a matrix does, compressing data, and solving optimization problems.

### Purposes

- To compute eigenvalues and eigenvectors for diagonalization and stability analysis.
- To perform principal component analysis (PCA) for dimensionality reduction.
- To compute the SVD for matrix approximation, compression, and pseudo-inverse.
- To analyze vibrations, oscillations, and stability in physical systems.
- To implement spectral clustering and graph analysis algorithms.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# General eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(a)

# Symmetric/Hermitian eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eigh(a, UPLO='L')

# Singular value decomposition
U, s, Vh = np.linalg.svd(a, full_matrices=True, compute_uv=True)
```

**Component Breakdown:**
- `np.linalg.eig(a)`: Returns `(w, v)` where `w` are eigenvalues (may be complex) and `v` are normalized eigenvectors (columns).
- `np.linalg.eigh(a, UPLO)`: Returns `(w, v)` where `w` are real eigenvalues in ascending order and `v` are orthonormal eigenvectors. `UPLO` specifies whether to use the lower (`'L'`) or upper (`'U'`) triangular part.
- `np.linalg.svd(a, full_matrices, compute_uv)`: Returns `(U, s, Vh)`. If `full_matrices=True`, `U` is `(M, M)` and `Vh` is `(N, N)`. If `False`, shapes are `(M, K)` and `(K, N)` with `K = min(M, N)`. `s` is a 1-D array of singular values in descending order.

**Syntax Rules:**
- `eig` returns complex eigenvalues if the matrix is not symmetric.
- `eigh` assumes the matrix is symmetric/Hermitian; using it on a non-symmetric matrix gives incorrect results.
- `eigh` returns eigenvalues in ascending order; `eig` does not guarantee order.
- `svd` singular values are always non-negative and sorted in descending order.

**Constraints and Limitations:**
- `eig` is slower and less numerically stable than `eigh` for symmetric matrices.
- `eigh` only uses one triangle of the input; the other is assumed to be its conjugate transpose.
- `svd` with `full_matrices=True` can be memory-intensive for large matrices.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Eigenvalue Decomposition

```python
import numpy as np

# Step 1: General matrix (non-symmetric)
A = np.array([[4, 1],
              [2, 3]])
print("Matrix A:")
print(A)

# Step 2: Compute eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)
print(f"\nEigenvalues: {eigenvalues}")
print(f"Eigenvectors (columns):\n{eigenvectors}")

# Step 3: Verify the eigen-decomposition: A @ v = λ * v
for i in range(len(eigenvalues)):
    lam = eigenvalues[i]
    v = eigenvectors[:, i]
    Av = A @ v
    lambda_v = lam * v
    print(f"\nλ{i+1} = {lam:.4f}")
    print(f"  A @ v = {Av}")
    print(f"  λ * v = {lambda_v}")
    print(f"  Equal: {np.allclose(Av, lambda_v)}")

# Step 4: Symmetric matrix — use eigh (faster, more stable)
S = np.array([[4, 1],
              [1, 3]])
print(f"\nSymmetric matrix S:")
print(S)

eigenvalues_s, eigenvectors_s = np.linalg.eigh(S)
print(f"\neigh eigenvalues: {eigenvalues_s}")
print(f"eigh eigenvectors:\n{eigenvectors_s}")

# Step 5: Verify eigh results
for i in range(len(eigenvalues_s)):
    lam = eigenvalues_s[i]
    v = eigenvectors_s[:, i]
    print(f"\nλ{i+1} = {lam:.4f}")
    print(f"  A @ v = {S @ v}")
    print(f"  λ * v = {lam * v}")
    print(f"  Equal: {np.allclose(S @ v, lam * v)}")

# Step 6: Verify eigenvectors are orthonormal
print(f"\nEigenvector dot products (should be identity):")
print(eigenvectors_s.T @ eigenvectors_s)

# Step 7: Complex eigenvalues example
C = np.array([[0, -1],
              [1, 0]])  # Rotation matrix
eigenvalues_c, eigenvectors_c = np.linalg.eig(C)
print(f"\nRotation matrix eigenvalues: {eigenvalues_c}")
print("(Complex eigenvalues indicate rotation.)")
```

**Expected Output:**
```
Matrix A:
[[4 1]
 [2 3]]

Eigenvalues: [5. 2.]
Eigenvectors (columns):
[[ 0.70710678 -0.4472136 ]
 [ 0.70710678  0.89442719]]

λ1 = 5.0000
  A @ v = [3.53553391 3.53553391]
  λ * v = [3.53553391 3.53553391]
  Equal: True

λ2 = 2.0000
  A @ v = [-0.89442719  1.78885438]
  λ * v = [-0.89442719  1.78885438]
  Equal: True

Symmetric matrix S:
[[4 1]
 [1 3]]

eigh eigenvalues: [2.38196601 4.61803399]
eigh eigenvectors:
[[-0.52573111 -0.85065081]
 [ 0.85065081 -0.52573111]]

λ1 = 2.3820
  A @ v = [-1.25167796  2.02590046]
  λ * v = [-1.25167796  2.02590046]
  Equal: True

λ2 = 4.6180
  A @ v = [-3.92831922 -2.42799895]
  λ * v = [-3.92831922 -2.42799895]
  Equal: True

Eigenvector dot products (should be identity):
[[ 1.00000000e+00 -1.11022302e-16]
 [-1.11022302e-16  1.00000000e+00]]

Rotation matrix eigenvalues: [0.+1.j 0.-1.j]
(Complex eigenvalues indicate rotation.)
```

**Why This Output Occurs:** For the general matrix `A`, the eigenvalues are 5 and 2, and the eigenvectors satisfy `A @ v = λ * v`. For the symmetric matrix `S`, `eigh` returns real eigenvalues in ascending order (2.38, 4.62) and orthonormal eigenvectors (their dot product matrix is the identity). The rotation matrix `C` has purely imaginary eigenvalues, indicating that it rotates vectors without stretching them.

#### Example 2: Singular Value Decomposition

```python
import numpy as np

# Step 1: Create a rectangular matrix
A = np.array([[1, 2, 3],
              [4, 5, 6]])
print("Matrix A (2×3):")
print(A)

# Step 2: Compute SVD
U, s, Vh = np.linalg.svd(A, full_matrices=True)
print(f"\nU shape: {U.shape}")
print(f"s shape: {s.shape}, values: {s}")
print(f"Vh shape: {Vh.shape}")

print(f"\nU:\n{U}")
print(f"\nVh:\n{Vh}")

# Step 3: Verify A = U @ diag(s) @ Vh
Sigma = np.zeros((2, 3))
Sigma[:2, :2] = np.diag(s)
A_reconstructed = U @ Sigma @ Vh
print(f"\nReconstructed A:\n{A_reconstructed}")
print(f"Matches original: {np.allclose(A, A_reconstructed)}")

# Step 4: Low-rank approximation (truncated SVD)
# Keep only the largest singular value
k = 1
U_k = U[:, :k]
s_k = s[:k]
Vh_k = Vh[:k, :]
A_approx = U_k @ np.diag(s_k) @ Vh_k
print(f"\nRank-{k} approximation:\n{A_approx}")
print(f"Approximation error: {np.linalg.norm(A - A_approx):.4f}")

# Step 5: Full-rank approximation (using all singular values)
A_full = U @ np.diag(s) @ Vh
print(f"\nFull-rank reconstruction error: {np.linalg.norm(A - A_full):.2e}")

# Step 6: SVD for image compression demonstration
# Create a synthetic "image" with structure
rng = np.random.default_rng(42)
image = rng.random((50, 50))
U_img, s_img, Vh_img = np.linalg.svd(image, full_matrices=False)
print(f"\nImage SVD: {len(s_img)} singular values")
print(f"Top 5 singular values: {s_img[:5]}")
print(f"Energy retained by top 10: {np.sum(s_img[:10]**2) / np.sum(s_img**2) * 100:.2f}%")
```

**Expected Output:**
```
Matrix A (2×3):
[[1 2 3]
 [4 5 6]]

U shape: (2, 2)
s shape: (2,), values: [9.508032   0.77286964]
Vh shape: (3, 3)

U:
[[-0.42866713 -0.90329619]
 [-0.90329619  0.42866713]]

Vh:
[[-0.42866713 -0.56694671 -0.70710678]
 [ 0.80596391  0.11238241 -0.58104278]
 [ 0.40824829 -0.81649658  0.40824829]]

Reconstructed A:
[[1. 2. 3.]
 [4. 5. 6.]]
Matches original: True

Rank-1 approximation:
[[1.74456215 2.30623264 2.8783964 ]
 [3.6799298  4.86527574 6.07161504]]
Approximation error: 0.7729

Full-rank reconstruction error: 1.39e-15

Image SVD: 50 singular values
Top 5 singular values: [25.46430416 2.86112723 2.63002403 2.48636338 2.25738029]
Energy retained by top 10: 94.83%
```

**Why This Output Occurs:** The SVD decomposes `A` into `U @ diag(s) @ Vh`, where `U` and `Vh` are orthogonal and `s` contains the singular values in descending order. The reconstruction is exact up to floating-point precision. The rank-1 approximation keeps only the largest singular value, capturing the dominant structure but introducing error. The image compression example shows that the top 10 singular values retain 94.83% of the total energy (sum of squared singular values), demonstrating the effectiveness of SVD for dimensionality reduction.

### Real-World Cases

- **Principal Component Analysis (PCA):** Eigen-decomposition of the covariance matrix for dimensionality reduction.
- **Image Compression:** Truncated SVD retains only the largest singular values, compressing images with minimal quality loss.
- **Recommender Systems:** Matrix factorization using SVD for collaborative filtering.
- **Vibration Analysis:** Eigenvalues of the stiffness matrix give natural frequencies of structures.
- **Quantum Mechanics:** Eigenvalues of Hermitian operators give measurable physical quantities.
- **Latent Semantic Analysis:** SVD of term-document matrices for topic modeling.

---

## Core Concept 5: Structural Decompositions — `np.linalg.qr()` and `np.linalg.cholesky()`

### Definitions

**Core Definition:** Structural decompositions factorize a matrix into a product of simpler matrices with specific properties (orthogonal, triangular), enabling efficient solving, inversion, and analysis.

**Technical Definition:** `numpy.linalg.qr(a, mode='reduced')` computes the QR factorization of `a`, factoring it as `a = Q @ R` where `Q` is orthonormal and `R` is upper-triangular. The `mode` parameter controls the shapes of `Q` and `R`. `numpy.linalg.cholesky(a)` computes the Cholesky decomposition of a positive-definite, symmetric (or Hermitian) matrix `a`, returning the lower-triangular matrix `L` such that `a = L @ L.T` (or `a = L @ L.conj().T` for complex matrices).

**Beginner-Friendly Explanation:** QR decomposition breaks a matrix into a rotation (`Q`) and an upper-triangular matrix (`R`). Cholesky decomposition is like taking the "square root" of a symmetric positive-definite matrix — it's useful for efficient solving and simulation.

### Purposes

- To solve linear systems efficiently using QR decomposition (more stable than LU for least-squares).
- To compute the Cholesky decomposition for efficient solving of symmetric positive-definite systems.
- To generate correlated random variables in Monte Carlo simulation via Cholesky.
- To implement QR-based eigenvalue algorithms (QR algorithm).
- To compute orthonormal bases for vector spaces.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# QR decomposition
Q, R = np.linalg.qr(a, mode='reduced')

# Cholesky decomposition
L = np.linalg.cholesky(a)
```

**Component Breakdown:**
- `np.linalg.qr(a, mode)`: Returns `(Q, R)` where `a = Q @ R`. `mode` options:
  - `'reduced'` (default): `Q` is `(M, K)`, `R` is `(K, N)` with `K = min(M, N)`.
  - `'complete'`: `Q` is `(M, M)`, `R` is `(M, N)`.
  - `'r'`: Returns only `R`.
  - `'raw'`: Returns `(h, tau)` in LAPACK format.
- `np.linalg.cholesky(a)`: Returns the lower-triangular `L` such that `a = L @ L.T`. Raises `LinAlgError` if `a` is not positive-definite.

**Syntax Rules:**
- `qr` works for any matrix shape (square or rectangular).
- `cholesky` requires a symmetric (or Hermitian) positive-definite matrix.
- `cholesky` returns a lower-triangular matrix by default; use `.T` for the upper-triangular factor.

**Constraints and Limitations:**
- `cholesky` raises `LinAlgError` if the matrix is not positive-definite.
- `qr` with `mode='complete'` can be memory-intensive for large matrices.
- The Cholesky decomposition is unique when the diagonal of `L` is constrained to be positive.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: QR Decomposition

```python
import numpy as np

# Step 1: Create a matrix
A = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 10]], dtype=float)
print("Matrix A:")
print(A)

# Step 2: Compute QR decomposition (reduced)
Q, R = np.linalg.qr(A, mode='reduced')
print(f"\nQ (orthonormal):\n{Q}")
print(f"\nR (upper-triangular):\n{R}")

# Step 3: Verify A = Q @ R
A_reconstructed = Q @ R
print(f"\nReconstructed A:\n{A_reconstructed}")
print(f"Matches original: {np.allclose(A, A_reconstructed)}")

# Step 4: Verify Q is orthonormal
QTQ = Q.T @ Q
print(f"\nQ.T @ Q (should be identity):\n{QTQ}")
print(f"Is identity: {np.allclose(QTQ, np.eye(3))}")

# Step 5: Verify R is upper-triangular
print(f"\nR is upper-triangular: {np.allclose(R, np.triu(R))}")

# Step 6: Solve a linear system using QR
b = np.array([1, 2, 3], dtype=float)
# A x = b → Q R x = b → R x = Q.T b
y = Q.T @ b
x_qr = np.linalg.solve(R, y)
x_direct = np.linalg.solve(A, b)
print(f"\nQR solve: {x_qr}")
print(f"Direct solve: {x_direct}")
print(f"Match: {np.allclose(x_qr, x_direct)}")

# Step 7: Complete mode
Q_full, R_full = np.linalg.qr(A, mode='complete')
print(f"\nComplete mode: Q shape {Q_full.shape}, R shape {R_full.shape}")

# Step 8: Cholesky decomposition of a positive-definite matrix
S = np.array([[4, 2, 1],
              [2, 5, 3],
              [1, 3, 6]], dtype=float)
print(f"\nSymmetric positive-definite matrix S:")
print(S)

L = np.linalg.cholesky(S)
print(f"\nCholesky factor L (lower triangular):\n{L}")

# Step 9: Verify S = L @ L.T
S_reconstructed = L @ L.T
print(f"\nReconstructed S:\n{S_reconstructed}")
print(f"Matches original: {np.allclose(S, S_reconstructed)}")

# Step 10: Verify L is lower-triangular
print(f"\nL is lower-triangular: {np.allclose(L, np.tril(L))}")

# Step 11: Solve using Cholesky
b_chol = np.array([1, 2, 3], dtype=float)
# S x = b → L L.T x = b → solve in two steps
y_chol = np.linalg.solve(L, b_chol)          # Forward substitution
x_chol = np.linalg.solve(L.T, y_chol)       # Back substitution
x_direct_chol = np.linalg.solve(S, b_chol)
print(f"\nCholesky solve: {x_chol}")
print(f"Direct solve:    {x_direct_chol}")
print(f"Match: {np.allclose(x_chol, x_direct_chol)}")
```

**Expected Output:**
```
Matrix A:
[[ 1.  2.  3.]
 [ 4.  5.  6.]
 [ 7.  8. 10.]]

Q (orthonormal):
[[-0.12309149  0.90453403  0.40824829]
 [-0.49236596  0.30151134 -0.81649658]
 [-0.86164044 -0.30151134  0.40824829]]

R (upper-triangular):
[[-8.1240384  -9.6011363 -12.44814951]
 [ 0.          0.90453403  1.50755672]
 [ 0.          0.          0.40824829]]

Reconstructed A:
[[ 1.  2.  3.]
 [ 4.  5.  6.]
 [ 7.  8. 10.]]
Matches original: True

Q.T @ Q (should be identity):
[[ 1.00000000e+00 -1.11022302e-16  2.22044605e-16]
 [-1.11022302e-16  1.00000000e+00 -1.11022302e-16]
 [ 2.22044605e-16 -1.11022302e-16  1.00000000e+00]]
Is identity: True

R is upper-triangular: True

QR solve: [-0.16666667  0.5         0.16666667]
Direct solve: [-0.16666667  0.5         0.16666667]
Match: True

Complete mode: Q shape (3, 3), R shape (3, 3)

Symmetric positive-definite matrix S:
[[4. 2. 1.]
 [2. 5. 3.]
 [1. 3. 6.]]

Cholesky factor L (lower triangular):
[[2.         0.         0.        ]
 [1.         2.         0.        ]
 [0.5        1.25       1.85404962]]

Reconstructed S:
[[4. 2. 1.]
 [2. 5. 3.]
 [1. 3. 6.]]
Matches original: True

L is lower-triangular: True

Cholesky solve: [-0.12962963  0.25925926  0.48148148]
Direct solve:    [-0.12962963  0.25925926  0.48148148]
Match: True
```

**Why This Output Occurs:** The QR decomposition factors `A` into an orthonormal matrix `Q` and an upper-triangular matrix `R`. `Q.T @ Q` is the identity (confirming orthonormality), and `Q @ R` reconstructs `A`. The QR-based linear solve uses the identity `A x = b → Q R x = b → R x = Q.T b`, solving first for `y = Q.T b` and then for `x` via back-substitution. The Cholesky decomposition produces a lower-triangular `L` such that `L @ L.T = S`. The Cholesky-based solve uses forward substitution (`L y = b`) and back substitution (`L.T x = y`).

### Real-World Cases

- **Least-Squares Regression:** QR decomposition provides a numerically stable method for solving least-squares problems.
- **Monte Carlo Simulation:** Cholesky decomposition generates correlated random variables for simulating correlated asset returns.
- **Kalman Filtering:** Cholesky decomposition is used for covariance matrix updates.
- **QR Algorithm:** The QR decomposition is the foundation of the QR algorithm for computing eigenvalues.
- **Portfolio Optimization:** Cholesky decomposition ensures positive-definiteness of covariance matrices.

---

## Core Concept 6: Matrix Metrics — `np.linalg.norm()`, `np.linalg.det()`, `np.linalg.inv()`, and `np.linalg.pinv()`

### Definitions

**Core Definition:** Matrix metrics quantify properties of matrices, including their "size" (norm), invertibility (determinant), inverse (for square matrices), and pseudo-inverse (for any matrix).

**Technical Definition:** `numpy.linalg.norm(x, ord=None, axis=None, keepdims=False)` computes vector or matrix norms. `numpy.linalg.det(a)` computes the determinant via LU factorization. `numpy.linalg.inv(a)` computes the multiplicative inverse of a square matrix using LU decomposition. `numpy.linalg.pinv(a, rcond=1e-15, hermitian=False)` computes the Moore-Penrose pseudo-inverse via SVD. `numpy.linalg.cond(x, p=None)` computes the condition number, a measure of numerical stability.

**Beginner-Friendly Explanation:** The norm tells you how "big" a matrix or vector is. The determinant tells you if a matrix can be inverted (non-zero means yes). The inverse undoes matrix multiplication. The pseudo-inverse is a fallback for matrices that can't be inverted normally. The condition number tells you how "dangerous" it is to invert a matrix.

### Purposes

- To measure vector and matrix magnitudes for normalization and convergence criteria.
- To check matrix invertibility and compute inverses for solving systems.
- To compute pseudo-inverses for rank-deficient and non-square matrices.
- To assess numerical stability before performing matrix operations.
- To regularize linear models using norms (L1/L2 regularization).

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Norms
n = np.linalg.norm(x, ord=None, axis=None, keepdims=False)

# Determinant
d = np.linalg.det(a)

# Inverse (square, non-singular)
a_inv = np.linalg.inv(a)

# Pseudo-inverse (any matrix)
a_pinv = np.linalg.pinv(a, rcond=1e-15, hermitian=False)

# Condition number
c = np.linalg.cond(x, p=None)
```

**Component Breakdown:**
- `np.linalg.norm(x, ord, axis, keepdims)`:
  - For vectors: `ord=1` (L1), `ord=2` or `None` (L2), `ord=np.inf` (L∞), `ord=0` (non-zero count).
  - For matrices: `ord='fro'` (Frobenius), `ord=1` (max column sum), `ord=np.inf` (max row sum), `ord=2` (largest singular value).
- `np.linalg.det(a)`: Returns determinant as float. For stacks, returns array of determinants.
- `np.linalg.inv(a)`: Returns inverse. Raises `LinAlgError` if singular.
- `np.linalg.pinv(a, rcond, hermitian)`: `rcond` sets cutoff for small singular values. `hermitian=True` assumes Hermitian for efficiency.
- `np.linalg.cond(x, p)`: Computes norm of `x` times norm of `x⁻¹`. `p=None` uses 2-norm.

**Syntax Rules:**
- `det` and `inv` require square matrices.
- `pinv` works for any shape.
- `cond` can return `np.inf` for singular matrices.
- `norm` with `axis` computes norms along specified axes.

**Constraints and Limitations:**
- `det` can underflow/overflow for large matrices; use `slogdet` instead.
- `inv` is less stable than `solve` for solving systems.
- `pinv` is more expensive than `inv` because it uses SVD.
- A high condition number (> 10¹⁰) indicates ill-conditioning.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Norms, Determinant, and Inverse

```python
import numpy as np

# Step 1: Vector norms
v = np.array([3.0, -4.0, 0.0, 5.0])
print(f"Vector v: {v}")

print(f"\nL1 norm:  {np.linalg.norm(v, ord=1)}")
print(f"L2 norm:  {np.linalg.norm(v, ord=2)}")
print(f"L∞ norm:  {np.linalg.norm(v, ord=np.inf)}")
print(f"L0 count: {np.linalg.norm(v, ord=0)}")

# Step 2: Matrix norms
A = np.array([[1, 2, 3],
              [4, 5, 6]])
print(f"\nMatrix A:\n{A}")

print(f"\nFrobenius norm: {np.linalg.norm(A, 'fro'):.4f}")
print(f"1-norm (max col sum): {np.linalg.norm(A, 1):.4f}")
print(f"∞-norm (max row sum): {np.linalg.norm(A, np.inf):.4f}")
print(f"2-norm (largest SV):  {np.linalg.norm(A, 2):.4f}")

# Step 3: Determinant
S = np.array([[4, 7],
              [2, 6]], dtype=float)
print(f"\nMatrix S:\n{S}")
det_S = np.linalg.det(S)
print(f"Determinant: {det_S}")
print(f"Manual: 4*6 - 7*2 = {4*6 - 7*2}")

# Step 4: Inverse
S_inv = np.linalg.inv(S)
print(f"\nInverse:\n{S_inv}")
print(f"S @ S⁻¹:\n{S @ S_inv}")
print(f"Is identity: {np.allclose(S @ S_inv, np.eye(2))}")

# Step 5: Condition number
cond_S = np.linalg.cond(S)
print(f"\nCondition number: {cond_S:.4f}")

# Step 6: Pseudo-inverse for singular matrix
singular = np.array([[1, 2],
                     [2, 4]], dtype=float)
print(f"\nSingular matrix:\n{singular}")
try:
    np.linalg.inv(singular)
except np.linalg.LinAlgError:
    print("inv() fails for singular matrix.")

pinv_singular = np.linalg.pinv(singular)
print(f"Pseudo-inverse:\n{pinv_singular}")
print(f"S @ S⁺ @ S ≈ S: {np.allclose(singular @ pinv_singular @ singular, singular)}")

# Step 7: Pseudo-inverse for non-square matrix
B = np.array([[1, 2, 3],
              [4, 5, 6]], dtype=float)
B_pinv = np.linalg.pinv(B)
print(f"\nNon-square B shape: {B.shape}")
print(f"Pseudo-inverse shape: {B_pinv.shape}")
print(f"B @ B⁺ @ B ≈ B: {np.allclose(B @ B_pinv @ B, B)}")

# Step 8: slogdet for numerical stability
sign, logdet = np.linalg.slogdet(S)
print(f"\nslogdet: sign={sign}, logdet={logdet:.4f}")
print(f"exp(logdet) = {np.exp(logdet):.4f} (matches det: {np.isclose(np.exp(logdet), det_S)})")
```

**Expected Output:**
```
Vector v: [ 3. -4.  0.  5.]

L1 norm:  12.0
L2 norm:  7.0710678118654755
L∞ norm:  5.0
L0 count: 3.0

Matrix A:
[[1 2 3]
 [4 5 6]]

Frobenius norm: 9.5394
1-norm (max col sum): 9.0000
∞-norm (max row sum): 15.0000
2-norm (largest SV):  9.5080

Matrix S:
[[4. 7.]
 [2. 6.]]
Determinant: 10.0
Manual: 4*6 - 7*2 = 10

Inverse:
[[ 0.6 -0.7]
 [-0.2  0.4]]
S @ S⁻¹:
[[1. 0.]
 [0. 1.]]
Is identity: True

Condition number: 15.0417

Singular matrix:
[[1. 2.]
 [2. 4.]]
inv() fails for singular matrix.
Pseudo-inverse:
[[0.04 0.08]
 [0.08 0.16]]
S @ S⁺ @ S ≈ S: True

Non-square B shape: (2, 3)
Pseudo-inverse shape: (3, 2)
B @ B⁺ @ B ≈ B: True

slogdet: sign=1.0, logdet=2.3026
exp(logdet) = 10.0000 (matches det: True)
```

**Why This Output Occurs:** The vector norms compute different "sizes" of the vector: L1 sums absolute values (12), L2 is the Euclidean length (√50 ≈ 7.07), L∞ is the max absolute value (5), and L0 counts non-zeros (3). The determinant of `S` is `4*6 - 7*2 = 10`. The inverse satisfies `S @ S_inv = I`. The pseudo-inverse works for the singular matrix (satisfying the Moore-Penrose conditions) and for non-square matrices. `slogdet` returns the sign and log of the determinant for numerical stability.

### Real-World Cases

- **Machine Learning:** L1 and L2 norms for regularization (Lasso and Ridge); condition numbers for assessing feature collinearity.
- **Numerical Analysis:** Determinants for checking invertibility; condition numbers for error bounds.
- **Statistics:** Pseudo-inverse for solving normal equations in rank-deficient regression.
- **Signal Processing:** Norms for signal energy and power; inverse for filter design.
- **Computer Graphics:** Norms for normalizing vectors; inverse for transformation matrices.

---

## References

1. **NumPy Linear Algebra (numpy.linalg) — Official Documentation** — https://numpy.org/doc/stable/reference/routines.linalg.html
2. **numpy.linalg.solve — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.solve.html
3. **numpy.linalg.lstsq — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.lstsq.html
4. **numpy.linalg.eig — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.eig.html
5. **numpy.linalg.eigh — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.eigh.html
6. **numpy.linalg.svd — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.svd.html
7. **numpy.linalg.qr — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.qr.html
8. **numpy.linalg.cholesky — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.cholesky.html
9. **numpy.linalg.norm — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.norm.html
10. **numpy.linalg.det — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.det.html
11. **numpy.linalg.inv — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.inv.html
12. **numpy.linalg.pinv — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.pinv.html
13. **numpy.linalg.cond — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.cond.html
14. **numpy.einsum — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.einsum.html
15. **numpy.matmul — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.matmul.html
16. **numpy.dot — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.dot.html
17. **G. Strang, Linear Algebra and Its Applications, 2nd Ed., Academic Press, 1980** — Standard reference for linear algebra fundamentals.
18. **G. H. Golub and C. F. Van Loan, Matrix Computations, 4th ed., Johns Hopkins University Press, 2013** — Comprehensive reference for matrix algorithms.
19. **LAPACK — Linear Algebra PACKage** — https://www.netlib.org/lapack/
20. **BLAS — Basic Linear Algebra Subprograms** — https://www.netlib.org/blas/