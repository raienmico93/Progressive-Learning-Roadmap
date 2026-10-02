# Matrix Operations — A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Matrix operations in NumPy encompass the fundamental mathematical operations performed on two-dimensional arrays, including multiplication, transposition, inversion, decomposition, and extraction of scalar properties like determinant and trace.

**Technical Definition:** NumPy represents matrices as 2-D `ndarray` objects. Matrix operations are implemented through a combination of the `@` operator (`numpy.matmul`), the `numpy.linalg` submodule (which leverages BLAS and LAPACK), and array methods like `.T` and `.conj()`. The `numpy.linalg` submodule provides functions for solving linear systems, computing inverses and pseudo-inverses, calculating norms and condition numbers, and performing matrix decompositions (SVD, QR, Cholesky, eigenvalues). The `@` operator, introduced in NumPy 1.10.0, is the preferred method for matrix multiplication between 2-D arrays.

**Beginner-Friendly Explanation:** A matrix is a rectangular grid of numbers, like a spreadsheet. Matrix operations let you multiply grids together, flip them, find their "inverse" (which undoes multiplication), and extract important numbers like the determinant and trace. NumPy makes these operations fast and easy, using the same optimized libraries that professional scientific software relies on.

### Key Characteristics

- **Dedicated Operator:** The `@` operator provides clean, readable matrix multiplication syntax.
- **BLAS/LAPACK Backed:** `numpy.linalg` functions use highly optimized libraries for speed.
- **Broadcasting:** Matrix operations support stacked arrays and broadcasting for batch processing.
- **Array-Based:** Standard 2-D `ndarray` objects are the recommended representation; `np.matrix` is deprecated.
- **Rich Functionality:** From basic multiplication to SVD-based pseudo-inverses, NumPy covers the full spectrum of linear algebra operations.

### Prerequisites

- Basic Python programming (variables, functions, imports)
- Familiarity with NumPy arrays, dtypes, and indexing
- Basic linear algebra concepts (matrix, vector, determinant, inverse)
- Understanding of the `axis` parameter for multi-dimensional arrays

### Related Programming Areas

- Machine Learning (linear regression, PCA, neural network layers)
- Physics and Engineering (structural analysis, circuit simulation)
- Computer Graphics (transformations, projection matrices)
- Statistics (covariance matrices, multivariate analysis)
- Optimization (linear programming, least squares)
- Signal Processing (filter design, system identification)

### Core Concepts / Features

1. Matrix-Matrix and Matrix-Vector Multiplication
2. Transpose, Conjugate Transpose, and Hermitian Matrices
3. Matrix Inverse, Pseudo-Inverse (Moore-Penrose), and Condition Numbers
4. Determinant, Trace, and Diagonal Extraction/Construction
5. Array vs. Matrix: Standard 2D `ndarray` Behaviors vs. the Deprecated `np.matrix` Class

---

## Core Concept 1: Matrix-Matrix and Matrix-Vector Multiplication

### Definitions

**Core Definition:** Matrix multiplication combines two matrices (or a matrix and a vector) to produce a new matrix or vector, where each element is the dot product of a row from the first operand and a column from the second.

**Technical Definition:** The matrix product of `A` (shape `(n, k)`) and `B` (shape `(k, m)`) is a matrix `C` of shape `(n, m)` where `C[i, j] = sum(A[i, :] * B[:, j])`. NumPy implements this through `numpy.matmul` (and its operator form `@`), which broadcasts stacks of matrices residing in the last two indices. The `@` operator is preferred over `numpy.dot` for 2-D arrays because it implements the `matmul` semantics with stricter shape rules and better broadcasting behavior. `numpy.dot` differs from `matmul` in two important ways: multiplication by scalars is not allowed in `matmul` (use `*` instead), and stacks of matrices are broadcast together as if the matrices were elements.

**Beginner-Friendly Explanation:** Matrix multiplication is like a recipe that combines rows and columns. You take each row of the first matrix, multiply it element-by-element with each column of the second matrix, and add up the results. The `@` symbol is NumPy's way of saying "do matrix multiplication."

### Purposes

- To implement linear transformations in geometry, graphics, and physics.
- To perform forward and backward passes in neural networks.
- To solve systems of linear equations.
- To compute projections and coordinate transformations.
- To provide the foundation for more complex linear algebra operations like SVD and eigenvalue decomposition.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Matrix-matrix multiplication
C = A @ B              # Preferred
C = np.matmul(A, B)    # Equivalent function

# Matrix-vector multiplication
y = A @ x              # Preferred
y = np.matmul(A, x)

# Legacy alternative (not recommended for 2-D)
C = np.dot(A, B)
```

**Component Breakdown:**
- `A @ B`: The `@` operator, introduced in NumPy 1.10.0, implements matrix multiplication for 2-D arrays.
- `np.matmul(A, B)`: The function equivalent of `@`. Requires the last dimension of `A` to match the second-to-last dimension of `B`.
- `np.dot(A, B)`: For 2-D arrays, performs matrix multiplication but with different broadcasting rules for higher dimensions.

**Syntax Rules:**
- For 2-D arrays, `A @ B` requires `A.shape[1] == B.shape[0]`.
- For matrix-vector products, `A @ x` where `x` is 1-D treats `x` as a column vector and returns a 1-D array.
- `matmul` supports stacks of matrices (N-D arrays) with broadcasting.
- `np.dot` for 1-D arrays is the inner product; for 2-D arrays it is matrix multiplication; for N-D it uses the second-to-last axis of `B`.

**Constraints and Limitations:**
- `matmul` does not allow scalar multiplication; use `*` for element-wise scaling.
- The `@` operator requires Python 3.5 or later.
- `np.dot` and `np.matmul` differ in their broadcasting behavior for N-D arrays.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Matrix Multiplication

```python
# Step 1: Import NumPy
import numpy as np

# Step 2: Create two matrices
A = np.array([[1, 2],
              [3, 4]])
B = np.array([[5, 6],
              [7, 8]])
print("Matrix A:")
print(A)
print("\nMatrix B:")
print(B)

# Step 3: Matrix multiplication using @
C = A @ B
print("\nA @ B:")
print(C)
# Manual: [[1*5+2*7, 1*6+2*8], [3*5+4*7, 3*6+4*8]] = [[19, 22], [43, 50]]

# Step 4: Verify with matmul
C_matmul = np.matmul(A, B)
print("\nmatmul result matches @: ", np.array_equal(C, C_matmul))

# Step 5: Verify with dot
C_dot = np.dot(A, B)
print("dot result matches @:     ", np.array_equal(C, C_dot))

# Step 6: Matrix-vector multiplication
x = np.array([1, 2])
y = A @ x
print(f"\nA @ x: {y}")  # [1*1+2*2, 3*1+4*2] = [5, 11]

# Step 7: Non-commutativity — A @ B ≠ B @ A
AB = A @ B
BA = B @ A
print(f"\nA @ B:\n{AB}")
print(f"B @ A:\n{BA}")
print(f"Equal: {np.array_equal(AB, BA)}")
```

**Expected Output:**
```
Matrix A:
[[1 2]
 [3 4]]

Matrix B:
[[5 6]
 [7 8]]

A @ B:
[[19 22]
 [43 50]]

matmul result matches @:  True
dot result matches @:      True

A @ x: [ 5 11]

A @ B:
[[19 22]
 [43 50]]
B @ A:
[[23 34]
 [31 46]]
Equal: False
```

**Why This Output Occurs:** `A @ B` computes the matrix product by taking dot products of rows of `A` with columns of `B`. The result `[[19, 22], [43, 50]]` follows from `1*5+2*7=19`, `1*6+2*8=22`, etc. Matrix multiplication is not commutative, so `A @ B` differs from `B @ A`.

#### Example 2: Stacked Matrix Multiplication

```python
import numpy as np

# Step 1: Create stacks of matrices
# 3 matrices of shape (2, 3)
A_stack = np.random.default_rng(42).random((3, 2, 3))
# 3 matrices of shape (3, 2)
B_stack = np.random.default_rng(43).random((3, 3, 2))

print(f"A_stack shape: {A_stack.shape}  (3 matrices of 2×3)")
print(f"B_stack shape: {B_stack.shape}  (3 matrices of 3×2)")

# Step 2: Batch matrix multiplication
C_stack = A_stack @ B_stack
print(f"\nC_stack shape: {C_stack.shape}  (3 matrices of 2×2)")

# Step 3: Verify each pair individually
for i in range(3):
    individual = A_stack[i] @ B_stack[i]
    assert np.allclose(C_stack[i], individual)
print("All batch results match individual computations.")

# Step 4: Compare with np.dot (different broadcasting)
dot_result = np.dot(A_stack, B_stack)
print(f"\nnp.dot result shape: {dot_result.shape}")
print("(np.dot produces a different shape for stacked arrays.)")

# Step 5: Matrix-vector batch
x_stack = np.random.default_rng(44).random((3, 3))  # 3 vectors of length 3
y_stack = A_stack @ x_stack
print(f"\nA_stack @ x_stack shape: {y_stack.shape}  (3 vectors of length 2)")
```

**Expected Output:**
```
A_stack shape: (3, 2, 3)  (3 matrices of 2×3)
B_stack shape: (3, 3, 2)  (3 matrices of 3×2)

C_stack shape: (3, 2, 2)  (3 matrices of 2×2)
All batch results match individual computations.

np.dot result shape: (3, 2, 3, 2)
(np.dot produces a different shape for stacked arrays.)

A_stack @ x_stack shape: (3, 2)  (3 vectors of length 2)
```

**Why This Output Occurs:** `matmul` treats N-D arrays as stacks of matrices residing in the last two indices and broadcasts them together. `np.dot` for N-D arrays uses a different rule (sum product over the last axis of `A` and second-to-last of `B`), producing a higher-dimensional result.

### Real-World Cases

- **Neural Networks:** Forward pass: `output = activation(weights @ input + bias)`.
- **Computer Graphics:** Transforming 3D points: `transformed = transformation_matrix @ point_vector`.
- **Structural Engineering:** Computing forces: `F = K @ u` where `K` is the stiffness matrix.
- **Economics:** Input-output models: `x = (I - A)^(-1) @ d`.
- **Recommendation Systems:** Matrix factorization: `R ≈ U @ V.T`.

---

## Core Concept 2: Transpose, Conjugate Transpose, and Hermitian Matrices

### Definitions

**Core Definition:** The transpose of a matrix flips it over its diagonal, swapping rows and columns. The conjugate transpose (Hermitian transpose) additionally takes the complex conjugate of each element. A Hermitian matrix equals its own conjugate transpose.

**Technical Definition:** For a matrix `A` of shape `(m, n)`, the transpose `A.T` has shape `(n, m)` with `A.T[i, j] = A[j, i]`. The conjugate transpose `A.conj().T` (or `A.H` for `np.matrix`) has elements `A.H[i, j] = conj(A[j, i])`. A matrix `A` is Hermitian if `A == A.H`, which requires `A` to be square. For real-valued matrices, the conjugate transpose equals the ordinary transpose.

**Beginner-Friendly Explanation:** Transposing a matrix is like rotating it 90 degrees and flipping it — rows become columns and columns become rows. The conjugate transpose does the same but also flips the sign of the imaginary part of complex numbers. A Hermitian matrix is a special symmetric matrix that equals its own conjugate transpose.

### Purposes

- To compute inner products and projections in linear algebra.
- To check symmetry and Hermitian properties in physics and quantum mechanics.
- To implement backpropagation in neural networks (gradient transposes).
- To solve linear systems using QR decomposition.
- To transform covariance matrices and correlation structures.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Transpose
A_T = A.T                    # Property
A_T = np.transpose(A)        # Function
A_T = A.transpose()          # Method

# Conjugate (complex conjugate element-wise)
A_conj = np.conj(A)
A_conj = A.conj()

# Conjugate transpose
A_H = A.conj().T
A_H = np.conj(A).T

# For np.matrix (deprecated)
A_H = A.H                    # Conjugate transpose property
A_T = A.T                    # Ordinary transpose (does not conjugate)
```

**Component Breakdown:**
- `A.T`: Returns a view of the array with axes transposed. For 2-D arrays, swaps rows and columns.
- `np.transpose(A)`: Function equivalent of `A.T`.
- `np.conj(A)`: Returns the complex conjugate of each element.
- `A.conj().T`: The conjugate transpose (Hermitian transpose).
- `np.matrix.H`: The conjugate transpose property for the deprecated matrix class.

**Syntax Rules:**
- `A.T` is a view, not a copy; modifying the transpose modifies the original.
- For complex matrices, `A.T` does NOT conjugate; use `A.conj().T` for the Hermitian transpose.
- `np.matrix.T` returns the transpose without conjugation; `np.matrix.H` returns the conjugate transpose.

**Constraints and Limitations:**
- The transpose of a 1-D array is the same array (no effect).
- For `np.matrix`, `*` is matrix multiplication, not element-wise multiplication.
- The `.H` property is only available on `np.matrix`, not on standard `ndarray`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Transpose and Conjugate Transpose

```python
import numpy as np

# Step 1: Real matrix transpose
A = np.array([[1, 2, 3],
              [4, 5, 6]])
print("Matrix A (2×3):")
print(A)

A_T = A.T
print(f"\nA.T (3×2):")
print(A_T)
print(f"Shape: {A_T.shape}")

# Step 2: Verify transpose property
assert A_T[0, 1] == A[1, 0]
assert A_T[2, 0] == A[0, 2]
print("\nTranspose property verified: A.T[i,j] == A[j,i]")

# Step 3: Complex matrix
B = np.array([[1+2j, 3-1j],
              [0+1j, 2+0j]])
print("\nComplex matrix B:")
print(B)

# Step 4: Ordinary transpose (does NOT conjugate)
B_T = B.T
print(f"\nB.T (ordinary transpose):")
print(B_T)
print("(Note: imaginary parts are NOT conjugated.)")

# Step 5: Conjugate transpose (Hermitian transpose)
B_H = B.conj().T
print(f"\nB.conj().T (Hermitian transpose):")
print(B_H)
print("(Imaginary parts ARE conjugated.)")

# Step 6: Verify Hermitian property
# A matrix is Hermitian if A == A.conj().T
H = np.array([[2, 1+1j],
              [1-1j, 3]])
H_H = H.conj().T
print(f"\nHermitian matrix H:")
print(H)
print(f"\nH.conj().T:")
print(H_H)
print(f"\nH is Hermitian: {np.allclose(H, H_H)}")

# Step 7: Non-Hermitian check
print(f"\nB is Hermitian: {np.allclose(B, B.conj().T)}")
```

**Expected Output:**
```
Matrix A (2×3):
[[1 2 3]
 [4 5 6]]

A.T (3×2):
[[1 4]
 [2 5]
 [3 6]]
Shape: (3, 2)

Transpose property verified: A.T[i,j] == A[j,i]

Complex matrix B:
[[1.+2.j 3.-1.j]
 [0.+1.j 2.+0.j]]

B.T (ordinary transpose):
[[1.+2.j 0.+1.j]
 [3.-1.j 2.+0.j]]
(Note: imaginary parts are NOT conjugated.)

B.conj().T (Hermitian transpose):
[[1.-2.j 0.-1.j]
 [3.+1.j 2.-0.j]]
(Imaginary parts ARE conjugated.)

Hermitian matrix H:
[[2.+0.j 1.+1.j]
 [1.-1.j 3.+0.j]]

H.conj().T:
[[2.-0.j 1.-1.j]
 [1.+1.j 3.-0.j]]

H is Hermitian: True

B is Hermitian: False
```

**Why This Output Occurs:** `B.T` swaps rows and columns but preserves the imaginary parts. `B.conj().T` first conjugates each element (flipping the sign of imaginary parts) and then transposes. `H` is Hermitian because `H[0,1] = 1+1j` and `H[1,0] = 1-1j`, which are complex conjugates of each other, satisfying `H == H.conj().T`.

#### Example 2: Symmetric Matrices and Properties

```python
import numpy as np

# Step 1: Create a symmetric matrix
S = np.array([[4, 1, 2],
              [1, 5, 3],
              [2, 3, 6]])
print("Symmetric matrix S:")
print(S)

# Step 2: Verify symmetry
print(f"\nS is symmetric: {np.allclose(S, S.T)}")

# Step 3: Properties of symmetric matrices
print(f"Trace of S: {np.trace(S)}")
print(f"Determinant of S: {np.linalg.det(S):.4f}")

# Step 4: Eigenvalues of symmetric matrix are real
eigenvalues = np.linalg.eigvalsh(S)  # Use eigvalsh for symmetric
print(f"\nEigenvalues (real): {eigenvalues}")

# Step 5: Transpose of matrix product
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
AB_T = (A @ B).T
BT_AT = B.T @ A.T
print(f"\n(A @ B).T:\n{AB_T}")
print(f"\nB.T @ A.T:\n{BT_AT}")
print(f"\n(A @ B).T == B.T @ A.T: {np.array_equal(AB_T, BT_AT)}")

# Step 6: Transpose of inverse
A_inv = np.linalg.inv(A)
A_inv_T = A_inv.T
A_T_inv = np.linalg.inv(A.T)
print(f"\ninv(A).T == inv(A.T): {np.allclose(A_inv_T, A_T_inv)}")
```

**Expected Output:**
```
Symmetric matrix S:
[[4 1 2]
 [1 5 3]
 [2 3 6]]

S is symmetric: True
Trace of S: 15
Determinant of S: 79.0000

Eigenvalues (real): [2.272 4.262 8.466]

(A @ B).T:
[[23 31]
 [34 46]]

B.T @ A.T:
[[23 31]
 [34 46]]

(A @ B).T == B.T @ A.T: True

inv(A).T == inv(A.T): True
```

**Why This Output Occurs:** Symmetric matrices satisfy `S == S.T` and have real eigenvalues. The identity `(AB)^T = B^T A^T` is a fundamental property of transposes. The identity `inv(A)^T = inv(A^T)` follows from the fact that the inverse of the transpose equals the transpose of the inverse.

### Real-World Cases

- **Quantum Mechanics:** Hermitian operators represent physical observables; their eigenvalues are real.
- **Machine Learning:** Gradient computation requires transposing weight matrices during backpropagation.
- **Statistics:** Covariance matrices are symmetric positive semi-definite.
- **Signal Processing:** Transpose operations in adaptive filter updates.
- **Robotics:** Jacobian transposes for force/torque transformations.

---

## Core Concept 3: Matrix Inverse, Pseudo-Inverse (Moore-Penrose), and Condition Numbers

### Definitions

**Core Definition:** The inverse of a square matrix `A` is a matrix `A⁻¹` such that `A @ A⁻¹ = A⁻¹ @ A = I`. The pseudo-inverse (Moore-Penrose) generalizes the inverse to non-square and singular matrices. The condition number measures how sensitive a matrix is to numerical errors when solving linear systems.

**Technical Definition:** `numpy.linalg.inv(a)` computes the multiplicative inverse of a square matrix using LU factorization. `numpy.linalg.pinv(a, rcond, hermitian)` computes the Moore-Penrose pseudo-inverse using singular-value decomposition (SVD), where singular values less than `rcond * largest_singular_value` are set to zero. `numpy.linalg.cond(x, p)` computes the condition number as the norm of `x` times the norm of the inverse of `x`, using one of seven different norms depending on `p`.

**Beginner-Friendly Explanation:** The inverse of a matrix undoes its multiplication, just like dividing by a number. But some matrices can't be inverted (they're "singular"). The pseudo-inverse is a fallback that works for any matrix. The condition number tells you how "dangerous" it is to invert a matrix — a high condition number means small errors in the input can cause huge errors in the output.

### Purposes

- To solve systems of linear equations `Ax = b` as `x = A⁻¹b`.
- To compute least-squares solutions for overdetermined systems via the pseudo-inverse.
- To assess numerical stability before performing matrix inversion.
- To implement ridge regression and other regularized linear models.
- To compute covariance matrices and their inverses in multivariate statistics.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Matrix inverse (square, non-singular)
A_inv = np.linalg.inv(A)

# Pseudo-inverse (any shape)
A_pinv = np.linalg.pinv(A, rcond=None, hermitian=False)

# Condition number
c = np.linalg.cond(A, p=None)
```

**Component Breakdown:**
- `np.linalg.inv(A)`: Computes the multiplicative inverse. Raises `LinAlgError` if `A` is singular or not square.
- `np.linalg.pinv(A, rcond, hermitian)`: Computes the Moore-Penrose pseudo-inverse via SVD. `rcond` sets the cutoff for small singular values (default `1e-15`). `hermitian=True` assumes `A` is Hermitian for efficiency.
- `np.linalg.cond(A, p)`: Computes the condition number. `p=None` uses the 2-norm (default), `p='fro'` uses the Frobenius norm, `p=np.inf` uses the infinity norm, and `p=1` uses the 1-norm.

**Syntax Rules:**
- `inv` requires a square matrix; `pinv` works for any shape.
- `pinv` is the recommended tool for solving least-squares problems.
- `cond` can return `np.inf` for singular matrices.
- The default `rcond` for `pinv` is `1e-15` in NumPy, but newer versions support the Array API `rtol` parameter.

**Constraints and Limitations:**
- Inverting a matrix is numerically less stable than solving the system directly with `np.linalg.solve`.
- `pinv` is more expensive than `inv` because it uses SVD.
- A high condition number (> 10¹⁰) indicates the matrix is ill-conditioned for inversion.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inverse and Pseudo-Inverse

```python
import numpy as np

# Step 1: Square invertible matrix
A = np.array([[4, 7],
              [2, 6]])
print("Matrix A:")
print(A)

# Step 2: Compute inverse
A_inv = np.linalg.inv(A)
print(f"\nInverse A⁻¹:")
print(A_inv)

# Step 3: Verify A @ A⁻¹ = I
identity = A @ A_inv
print(f"\nA @ A⁻¹:")
print(identity)
print(f"Is identity: {np.allclose(identity, np.eye(2))}")

# Step 4: Verify A⁻¹ @ A = I
identity2 = A_inv @ A
print(f"\nA⁻¹ @ A is identity: {np.allclose(identity2, np.eye(2))}")

# Step 5: Singular matrix — inv fails
S = np.array([[1, 2],
              [2, 4]])  # Row 2 = 2 × Row 1
print(f"\nSingular matrix S (det = {np.linalg.det(S):.1f}):")
print(S)
try:
    np.linalg.inv(S)
except np.linalg.LinAlgError as e:
    print(f"inv(S) raises: {e}")

# Step 6: Pseudo-inverse for singular matrix
S_pinv = np.linalg.pinv(S)
print(f"\nPseudo-inverse of S:")
print(S_pinv)
print(f"S @ S_pinv @ S ≈ S: {np.allclose(S @ S_pinv @ S, S)}")
print(f"S_pinv @ S @ S_pinv ≈ S_pinv: {np.allclose(S_pinv @ S @ S_pinv, S_pinv)}")

# Step 7: Pseudo-inverse for non-square matrix
B = np.array([[1, 2, 3],
              [4, 5, 6]])  # 2×3
B_pinv = np.linalg.pinv(B)  # 3×2
print(f"\nNon-square B shape: {B.shape}")
print(f"Pseudo-inverse shape: {B_pinv.shape}")
print(f"B @ B_pinv @ B ≈ B: {np.allclose(B @ B_pinv @ B, B)}")
```

**Expected Output:**
```
Matrix A:
[[4 7]
 [2 6]]

Inverse A⁻¹:
[[ 0.6 -0.7]
 [-0.2  0.4]]

A @ A⁻¹:
[[1. 0.]
 [0. 1.]]
Is identity: True

A⁻¹ @ A is identity: True

Singular matrix S (det = 0.0):
[[1 2]
 [2 4]]
inv(S) raises: Singular matrix

Pseudo-inverse of S:
[[0.04 0.08]
 [0.08 0.16]]
S @ S_pinv @ S ≈ S: True
S_pinv @ S @ S_pinv ≈ S_pinv: True

Non-square B shape: (2, 3)
Pseudo-inverse shape: (3, 2)
B @ B_pinv @ B ≈ B: True
```

**Why This Output Occurs:** `inv(A)` computes the multiplicative inverse such that `A @ A_inv = I`. For the singular matrix `S` (whose determinant is zero), `inv` raises `LinAlgError`. The pseudo-inverse `S_pinv` satisfies the four Moore-Penrose conditions: `A @ A⁺ @ A = A`, `A⁺ @ A @ A⁺ = A⁺`, `(A @ A⁺)ᴴ = A @ A⁺`, and `(A⁺ @ A)ᴴ = A⁺ @ A`.

#### Example 2: Condition Numbers and Numerical Stability

```python
import numpy as np

# Step 1: Well-conditioned matrix
A_good = np.array([[1, 0],
                   [0, 1]])
cond_good = np.linalg.cond(A_good)
print(f"Identity matrix condition number: {cond_good}")

# Step 2: Moderately conditioned matrix
A_mod = np.array([[1, 2],
                  [3, 4]])
cond_mod = np.linalg.cond(A_mod)
print(f"Moderate matrix condition number: {cond_mod:.4f}")

# Step 3: Ill-conditioned matrix
A_bad = np.array([[1, 1],
                  [1, 1.0001]])
cond_bad = np.linalg.cond(A_bad)
print(f"Ill-conditioned matrix condition number: {cond_bad:.4f}")

# Step 4: Singular matrix — infinite condition number
A_sing = np.array([[1, 2],
                   [2, 4]])
cond_sing = np.linalg.cond(A_sing)
print(f"Singular matrix condition number: {cond_sing}")

# Step 5: Demonstrate numerical instability
# Solve Ax = b for well-conditioned and ill-conditioned systems
b = np.array([1, 2])

x_good = np.linalg.solve(A_good, b)
x_bad = np.linalg.solve(A_bad, b)

# Add a tiny perturbation to b
b_perturbed = b + np.array([1e-10, 1e-10])
x_good_pert = np.linalg.solve(A_good, b_perturbed)
x_bad_pert = np.linalg.solve(A_bad, b_perturbed)

print(f"\nWell-conditioned solution change: {np.linalg.norm(x_good_pert - x_good):.2e}")
print(f"Ill-conditioned solution change:  {np.linalg.norm(x_bad_pert - x_bad):.2e}")

# Step 6: Condition number with different norms
print(f"\nCondition numbers for A_bad:")
print(f"  2-norm (default): {np.linalg.cond(A_bad):.4f}")
print(f"  Frobenius norm:   {np.linalg.cond(A_bad, 'fro'):.4f}")
print(f"  Infinity norm:    {np.linalg.cond(A_bad, np.inf):.4f}")
print(f"  1-norm:           {np.linalg.cond(A_bad, 1):.4f}")
```

**Expected Output:**
```
Identity matrix condition number: 1.0
Moderate matrix condition number: 14.9330
Ill-conditioned matrix condition number: 40002.0002
Singular matrix condition number: inf

Well-conditioned solution change: 1.00e-10
Ill-conditioned solution change:  1.00e-06

Condition numbers for A_bad:
  2-norm (default): 40002.0002
  Frobenius norm:   40002.0002
  Infinity norm:    40002.0002
  1-norm:           40002.0002
```

**Why This Output Occurs:** The condition number of the identity matrix is 1 (perfectly conditioned). The ill-conditioned matrix `A_bad` has a condition number of ~40,000, meaning a tiny perturbation (1e-10) in the input causes a 1e-6 change in the solution — a 10,000× amplification. The singular matrix has an infinite condition number because it cannot be inverted.

### Real-World Cases

- **Linear Regression:** The pseudo-inverse provides the least-squares solution when the design matrix is rank-deficient.
- **Numerical Weather Prediction:** Condition numbers assess the stability of covariance matrices in data assimilation.
- **Structural Engineering:** Ill-conditioned stiffness matrices indicate near-singular structures that may be unstable.
- **Machine Learning:** Ridge regression adds a regularization term to improve the condition number of `XᵀX`.
- **Signal Processing:** The pseudo-inverse is used in beamforming and MIMO channel estimation.

---

## Core Concept 4: Determinant, Trace, and Diagonal Extraction/Construction

### Definitions

**Core Definition:** The determinant is a scalar value that encodes whether a matrix is invertible and how it scales volumes. The trace is the sum of the diagonal elements. Diagonal extraction retrieves the diagonal elements; diagonal construction creates a matrix with specified diagonal values.

**Technical Definition:** `numpy.linalg.det(a)` computes the determinant via LU factorization using LAPACK's `z/dgetrf` routine. `numpy.trace(a, offset, axis1, axis2)` returns the sum along diagonals, with support for offsets and multi-dimensional arrays. `numpy.diag(v, k)` extracts a diagonal if `v` is 2-D, or constructs a diagonal matrix if `v` is 1-D. `numpy.diagonal(a, offset, axis1, axis2)` returns specified diagonals as a view (NumPy 1.9+).

**Beginner-Friendly Explanation:** The determinant tells you if a matrix can be inverted (non-zero means yes). The trace is just the sum of the diagonal numbers. `diag` is a dual-purpose function: give it a matrix and it extracts the diagonal; give it a list and it builds a diagonal matrix.

### Purposes

- To check matrix invertibility (non-zero determinant).
- To compute the volume scaling factor of a linear transformation.
- To calculate eigenvalues (product of eigenvalues = determinant; sum = trace).
- To extract and construct diagonal matrices for covariance and correlation analysis.
- To implement Jacobi and Gauss-Seidel iterative solvers.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Determinant
det = np.linalg.det(A)

# Log-determinant (for large matrices)
sign, logdet = np.linalg.slogdet(A)

# Trace
tr = np.trace(A)
tr = np.trace(A, offset=1)        # Upper diagonal
tr = np.trace(A, offset=-1)       # Lower diagonal

# Diagonal extraction
d = np.diag(A)                    # Extract main diagonal
d = np.diag(A, k=1)              # Extract k-th diagonal
d = np.diagonal(A)                # Alternative extraction

# Diagonal construction
D = np.diag([1, 2, 3])           # Create 3×3 diagonal matrix
D = np.diag([1, 2], k=1)         # Create matrix with diagonal offset
```

**Component Breakdown:**
- `np.linalg.det(A)`: Returns the determinant as a float. Supports broadcasting for stacks of matrices.
- `np.linalg.slogdet(A)`: Returns `(sign, logabsdet)` for numerical stability with large matrices.
- `np.trace(A, offset, axis1, axis2)`: Sums along diagonals. `offset` shifts the diagonal.
- `np.diag(v, k)`: If `v` is 2-D, extracts the k-th diagonal; if `v` is 1-D, constructs a matrix.
- `np.diagonal(a, offset, axis1, axis2)`: Returns a view of the specified diagonal.

**Syntax Rules:**
- `det` requires square matrices (or stacks of square matrices).
- `trace` works on any 2-D (or N-D) array; `offset` can be positive or negative.
- `diag` extraction returns a copy (NumPy 1.9+), while `diagonal` returns a read-only view.
- `diag` construction from a 1-D array produces a square matrix.

**Constraints and Limitations:**
- `det` can underflow/overflow for large matrices; use `slogdet` instead.
- `trace` with `offset` beyond the matrix dimensions returns 0.
- `diagonal` returns a read-only view in NumPy 1.9+.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Determinant and Trace

```python
import numpy as np

# Step 1: Create matrices
A = np.array([[1, 2],
              [3, 4]])
print("Matrix A:")
print(A)

# Step 2: Determinant
det_A = np.linalg.det(A)
print(f"\nDeterminant: {det_A}")  # 1*4 - 2*3 = -2

# Step 3: Trace
tr_A = np.trace(A)
print(f"Trace: {tr_A}")  # 1 + 4 = 5

# Step 4: Verify determinant = product of eigenvalues
eigenvalues = np.linalg.eigvals(A)
print(f"\nEigenvalues: {eigenvalues}")
print(f"Product of eigenvalues: {np.prod(eigenvalues):.4f}")
print(f"Matches determinant: {np.isclose(np.prod(eigenvalues), det_A)}")

# Step 5: Verify trace = sum of eigenvalues
print(f"Sum of eigenvalues: {np.sum(eigenvalues):.4f}")
print(f"Matches trace: {np.isclose(np.sum(eigenvalues), tr_A)}")

# Step 6: slogdet for numerical stability
sign, logdet = np.linalg.slogdet(A)
print(f"\nslogdet: sign={sign}, logdet={logdet:.4f}")
print(f"exp(logdet) = {np.exp(logdet):.4f} (matches det: {np.isclose(np.exp(logdet), det_A)})")

# Step 7: Trace with offset
B = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])
print(f"\nMatrix B:")
print(B)
print(f"Trace (main):     {np.trace(B)}")        # 1+5+9 = 15
print(f"Trace (offset=1): {np.trace(B, offset=1)}")  # 2+6 = 8
print(f"Trace (offset=-1): {np.trace(B, offset=-1)}") # 4+8 = 12

# Step 8: Determinant of singular matrix
S = np.array([[1, 2],
              [2, 4]])
print(f"\nSingular matrix determinant: {np.linalg.det(S):.1f}")
print("(Zero determinant means the matrix is singular.)")
```

**Expected Output:**
```
Matrix A:
[[1 2]
 [3 4]]

Determinant: -2.0
Trace: 5

Eigenvalues: [-0.3723  5.3723]
Product of eigenvalues: -2.0000
Matches determinant: True
Sum of eigenvalues: 5.0000
Matches trace: True

slogdet: sign=-1.0, logdet=0.6931
exp(logdet) = -2.0000 (matches det: True)

Matrix B:
[[1 2 3]
 [4 5 6]
 [7 8 9]]
Trace (main):     15
Trace (offset=1): 8
Trace (offset=-1): 12

Singular matrix determinant: 0.0
(Zero determinant means the matrix is singular.)
```

**Why This Output Occurs:** The determinant of `[[1,2],[3,4]]` is `1*4 - 2*3 = -2`. The trace is `1+4=5`. The product of eigenvalues equals the determinant, and the sum equals the trace — these are fundamental linear algebra identities. `slogdet` returns the sign and log of the absolute determinant for numerical stability.

#### Example 2: Diagonal Extraction and Construction

```python
import numpy as np

# Step 1: Extract diagonal from a matrix
A = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])
print("Matrix A:")
print(A)

# Step 2: Extract main diagonal
diag_main = np.diag(A)
print(f"\nMain diagonal: {diag_main}")  # [1, 5, 9]

# Step 3: Extract off-diagonals
diag_upper = np.diag(A, k=1)
diag_lower = np.diag(A, k=-1)
print(f"Upper diagonal (k=1): {diag_upper}")  # [2, 6]
print(f"Lower diagonal (k=-1): {diag_lower}")  # [4, 8]

# Step 4: Construct diagonal matrix from vector
v = np.array([10, 20, 30])
D = np.diag(v)
print(f"\nDiagonal matrix from {v}:")
print(D)

# Step 5: Construct matrix with off-diagonal
D_upper = np.diag([1, 2], k=1)
print(f"\nMatrix with diagonal at k=1:")
print(D_upper)

# Step 6: diagonal() function returns a view
A_diag_view = np.diagonal(A)
print(f"\ndiagonal(A): {A_diag_view}")
# Modify the view (may be read-only in newer NumPy)
try:
    A_diag_view[0] = 100
    print(f"After modifying view: {A_diag_view}")
    print(f"Original matrix diagonal: {np.diag(A)}")
except ValueError as e:
    print(f"View is read-only: {e}")

# Step 7: Use diag for covariance matrix visualization
cov = np.array([[4.0, 1.5, 0.5],
                [1.5, 3.0, 0.8],
                [0.5, 0.8, 2.0]])
print(f"\nCovariance matrix:")
print(cov)
print(f"Variances (diagonal): {np.diag(cov)}")
print(f"Total variance (trace): {np.trace(cov)}")

# Step 8: Create identity-like matrix with custom diagonal
custom_diag = np.diag([1, 2, 3]) + np.diag([0.5, 0.5], k=1) + np.diag([0.5, 0.5], k=-1)
print(f"\nTridiagonal matrix:")
print(custom_diag)
```

**Expected Output:**
```
Matrix A:
[[1 2 3]
 [4 5 6]
 [7 8 9]]

Main diagonal: [1 5 9]
Upper diagonal (k=1): [2 6]
Lower diagonal (k=-1): [4 8]

Diagonal matrix from [10 20 30]:
[[10  0  0]
 [ 0 20  0]
 [ 0  0 30]]

Matrix with diagonal at k=1:
[[0 1 0]
 [0 0 2]
 [0 0 0]]

diagonal(A): [1 5 9]
View is read-only: assignment destination is read-only

Covariance matrix:
[[4.  1.5 0.5]
 [1.5 3.  0.8]
 [0.5 0.8 2. ]]
Variances (diagonal): [4. 3. 2.]
Total variance (trace): 9.0

Tridiagonal matrix:
[[1.  0.5 0.  ]
 [0.5 2.  0.5]
 [0.  0.5 3. ]]
```

**Why This Output Occurs:** `np.diag(A)` extracts the main diagonal `[1, 5, 9]`. `np.diag(A, k=1)` extracts the first super-diagonal `[2, 6]`. `np.diag([10, 20, 30])` constructs a 3×3 diagonal matrix. `np.diagonal` returns a read-only view in NumPy 1.9+, so attempting to modify it raises an error. The tridiagonal construction combines three `diag` calls to build a matrix with non-zero elements on three diagonals.

### Real-World Cases

- **Statistics:** The determinant of a covariance matrix measures the generalized variance; the trace measures total variance.
- **Machine Learning:** Determinants appear in Gaussian process regression and Bayesian model selection.
- **Quantum Mechanics:** The trace of a density matrix is always 1 (normalization).
- **Structural Engineering:** The determinant of the stiffness matrix indicates structural stability.
- **Image Processing:** Diagonal extraction for color channel manipulation.

---

## Core Concept 5: Array vs. Matrix — Standard 2D `ndarray` Behaviors vs. the Deprecated `np.matrix` Class

### Definitions

**Core Definition:** NumPy provides two representations for matrices: standard 2-D `ndarray` objects (recommended) and the deprecated `np.matrix` class (not recommended). The key difference is that `np.matrix` overrides `*` to mean matrix multiplication, while `ndarray` uses `*` for element-wise multiplication.

**Technical Definition:** `np.matrix` is a subclass of `ndarray` that forces 2-D shape and overrides binary operations so that `*` performs matrix multiplication and `**` performs matrix power. The class is officially deprecated: "It is no longer recommended to use this class, even for linear algebra. Instead use regular arrays. The class may be removed in the future". Standard `ndarray` uses `*` for element-wise multiplication and `@` for matrix multiplication. The `np.matrix` class also provides special properties: `.T` (transpose), `.H` (conjugate transpose), `.I` (inverse), and `.A` (return as ndarray).

**Beginner-Friendly Explanation:** The old `np.matrix` class was designed to make matrix math feel like MATLAB. But it caused confusion because `*` meant matrix multiplication instead of element-wise multiplication. The modern `ndarray` uses `@` for matrix multiplication and `*` for element-wise multiplication, which is clearer and consistent with the rest of Python.

### Purposes

- To understand legacy code that uses `np.matrix`.
- To migrate existing MATLAB-style code to modern NumPy.
- To avoid the pitfalls of the deprecated class in new code.
- To recognize the behavioral differences between `*` and `@`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# === DEPRECATED np.matrix ===
M = np.matrix([[1, 2], [3, 4]])   # Matrix multiplication with *
product = M * M                    # Matrix multiplication (NOT element-wise)
power = M ** 2                     # Matrix power
inv = M.I                          # Inverse
transpose = M.T                    # Transpose
conj_transpose = M.H               # Conjugate transpose
as_array = M.A                     # Convert to ndarray

# === RECOMMENDED ndarray ===
A = np.array([[1, 2], [3, 4]])
product = A @ A                    # Matrix multiplication
elementwise = A * A                # Element-wise multiplication
inv = np.linalg.inv(A)             # Inverse
transpose = A.T                    # Transpose
conj_transpose = A.conj().T        # Conjugate transpose
```

**Component Breakdown:**
- `np.matrix(data)`: Constructs a matrix from an array-like or string.
- `M * M`: For `np.matrix`, performs matrix multiplication (overridden `__mul__`).
- `M ** 2`: For `np.matrix`, performs matrix power.
- `M.I`: Returns the multiplicative inverse.
- `M.H`: Returns the conjugate transpose.
- `M.A`: Returns the matrix as an ndarray.

**Syntax Rules:**
- `np.matrix` always produces 2-D objects, even from 1-D input.
- `np.matrix` overrides `*` and `**` for matrix operations.
- Standard `ndarray` uses `@` for matrix multiplication and `*` for element-wise.
- `np.matrix` is deprecated and may be removed in future NumPy versions.

**Constraints and Limitations:**
- The deprecation has no announced removal date, but new code should avoid `np.matrix`.
- `np.matrix` does not support N-D arrays (only 2-D).
- Mixing `np.matrix` with `ndarray` can produce unexpected results.
- The `np.matlib` module is also pending deprecation.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Key Differences Between Matrix and Array

```python
import numpy as np

# Step 1: Create the same data in both forms
A_array = np.array([[1, 2],
                    [3, 4]])
A_matrix = np.matrix([[1, 2],
                      [3, 4]])

print("Array:")
print(A_array)
print(f"Type: {type(A_array)}")

print("\nMatrix:")
print(A_matrix)
print(f"Type: {type(A_matrix)}")

# Step 2: The critical difference — * operator
array_mult = A_array * A_array
matrix_mult = A_matrix * A_matrix

print(f"\nArray * Array (element-wise):")
print(array_mult)

print(f"\nMatrix * Matrix (matrix multiplication):")
print(matrix_mult)

# Step 3: Matrix multiplication with @ works for both
array_at = A_array @ A_array
matrix_at = A_matrix @ A_matrix
print(f"\nArray @ Array:\n{array_at}")
print(f"Matrix @ Matrix:\n{matrix_at}")
print(f"Both @ results equal: {np.array_equal(array_at, np.asarray(matrix_at))}")

# Step 4: The ** operator difference
array_power = A_array ** 2      # Element-wise square
matrix_power = A_matrix ** 2    # Matrix power
print(f"\nArray ** 2 (element-wise):\n{array_power}")
print(f"Matrix ** 2 (matrix power):\n{matrix_power}")

# Step 5: Shape preservation
row_array = np.array([1, 2, 3])
row_matrix = np.matrix([1, 2, 3])
print(f"\n1-D array shape: {row_array.shape}")
print(f"Matrix from 1-D shape: {row_matrix.shape}")
print("(np.matrix always forces 2-D shape.)")

# Step 6: Converting between them
array_from_matrix = np.asarray(A_matrix)
matrix_from_array = np.matrix(A_array)
print(f"\nArray from matrix type: {type(array_from_matrix)}")
print(f"Matrix from array type: {type(matrix_from_array)}")
```

**Expected Output:**
```
Array:
[[1 2]
 [3 4]]
Type: <class 'numpy.ndarray'>

Matrix:
[[1 2]
 [3 4]]
Type: <class 'numpy.matrix'>

Array * Array (element-wise):
[[ 1  4]
 [ 9 16]]

Matrix * Matrix (matrix multiplication):
[[ 7 10]
 [15 22]]

Array @ Array:
[[ 7 10]
 [15 22]]
Matrix @ Matrix:
[[ 7 10]
 [15 22]]
Both @ results equal: True

Array ** 2 (element-wise):
[[ 1  4]
 [ 9 16]]
Matrix ** 2 (matrix power):
[[ 7 10]
 [15 22]]

1-D array shape: (3,)
Matrix from 1-D shape: (1, 3)
(np.matrix always forces 2-D shape.)

Array from matrix type: <class 'numpy.ndarray'>
Matrix from array type: <class 'numpy.matrix'>
```

**Why This Output Occurs:** For `ndarray`, `*` is element-wise multiplication (`[[1*1, 2*2], [3*3, 4*4]] = [[1,4],[9,16]]`). For `np.matrix`, `*` is matrix multiplication (`[[1*1+2*3, 1*2+2*4], [3*1+4*3, 3*2+4*4]] = [[7,10],[15,22]]`). The `@` operator produces the same matrix product for both types. `np.matrix` from a 1-D array creates a 1×3 matrix, while `ndarray` preserves the 1-D shape.

#### Example 2: Migration from np.matrix to ndarray

```python
import numpy as np

# === OLD WAY (DEPRECATED) ===
print("=== Old np.matrix code ===")
M = np.matrix([[1, 2], [3, 4]])
result_old = M * M          # Matrix multiplication
inv_old = M.I               # Inverse
trans_old = M.T             # Transpose
det_old = np.linalg.det(M)  # Determinant
print(f"M * M:\n{result_old}")
print(f"M.I:\n{inv_old}")
print(f"M.T:\n{trans_old}")
print(f"det(M): {det_old}")

# === NEW WAY (RECOMMENDED) ===
print("\n=== Modern ndarray code ===")
A = np.array([[1, 2], [3, 4]])
result_new = A @ A            # Matrix multiplication
inv_new = np.linalg.inv(A)    # Inverse
trans_new = A.T               # Transpose
det_new = np.linalg.det(A)    # Determinant
print(f"A @ A:\n{result_new}")
print(f"inv(A):\n{inv_new}")
print(f"A.T:\n{trans_new}")
print(f"det(A): {det_new}")

# Step 3: Verify equivalence
print(f"\nResults identical: {np.allclose(result_old, result_new)}")
print(f"Inverses identical: {np.allclose(inv_old, inv_new)}")

# Step 4: Show the pitfalls of np.matrix
print("\n=== Pitfalls of np.matrix ===")
M1 = np.matrix([[1, 2], [3, 4]])
M2 = np.matrix([[5, 6], [7, 8]])

# Accidentally using * thinking it's element-wise
wrong = M1 * M2  # Matrix multiplication (surprise!)
print(f"M1 * M2 (matrix multiplication, not element-wise):")
print(wrong)

# Correct element-wise with np.multiply
correct = np.multiply(M1, M2)
print(f"\nnp.multiply(M1, M2) (element-wise):")
print(correct)

# Step 5: Converting old code
print("\n=== Converting ===")
old_matrix = np.matrix([[1, 2], [3, 4]])
new_array = np.asarray(old_matrix)
print(f"Old type: {type(old_matrix)}")
print(f"New type: {type(new_array)}")
print(f"Values equal: {np.array_equal(old_matrix, new_array)}")
```

**Expected Output:**
```
=== Old np.matrix code ===
M * M:
[[ 7 10]
 [15 22]]
M.I:
[[-2.   1. ]
 [ 1.5 -0.5]]
M.T:
[[1 3]
 [2 4]]
det(M): -2.0

=== Modern ndarray code ===
A @ A:
[[ 7 10]
 [15 22]]
inv(A):
[[-2.   1. ]
 [ 1.5 -0.5]]
A.T:
[[1 3]
 [2 4]]
det(A): -2.0

Results identical: True
Inverses identical: True

=== Pitfalls of np.matrix ===
M1 * M2 (matrix multiplication, not element-wise):
[[19 22]
 [43 50]]

np.multiply(M1, M2) (element-wise):
[[ 5 12]
 [21 32]]

=== Converting ===
Old type: <class 'numpy.matrix'>
New type: <class 'numpy.ndarray'>
Values equal: True
```

**Why This Output Occurs:** The old and new approaches produce identical numerical results for matrix multiplication, inverse, transpose, and determinant. The pitfall is that `M1 * M2` for `np.matrix` performs matrix multiplication, not element-wise multiplication — a common source of bugs. `np.multiply` provides the correct element-wise behavior. Converting from `np.matrix` to `ndarray` with `np.asarray` preserves values while removing the deprecated class's behavior.

### Real-World Cases

- **Legacy Codebases:** Scientific code written before NumPy 1.10 often uses `np.matrix`; understanding the differences is essential for maintenance.
- **MATLAB Migration:** Researchers migrating from MATLAB to Python need to understand that `*` in MATLAB means matrix multiplication, while in NumPy it means element-wise.
- **Tutorials and Textbooks:** Older educational materials may use `np.matrix`; new materials should use `ndarray` with `@`.
- **Library Compatibility:** Some third-party libraries still return `np.matrix`; knowing how to convert is important for interoperability.
- **Code Review:** Recognizing `np.matrix` usage helps reviewers flag deprecated patterns.

---

## References

1. **NumPy Linear Algebra (Official Documentation)** — https://numpy.org/doc/stable/reference/routines.linalg.html
2. **numpy.matmul (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.matmul.html
3. **numpy.dot (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.dot.html
4. **numpy.linalg.inv (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.inv.html
5. **numpy.linalg.pinv (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.pinv.html
6. **numpy.linalg.cond (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.cond.html
7. **numpy.linalg.det (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.det.html
8. **numpy.linalg.slogdet (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.slogdet.html
9. **numpy.trace (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.trace.html
10. **numpy.diag (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.diag.html
11. **numpy.diagonal (Official Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.diagonal.html
12. **numpy.matrix (Deprecated Class Documentation)** — https://numpy.org/doc/stable/reference/generated/numpy.matrix.html
13. **NumPy "Matrix Objects" (Array Classes Documentation)** — https://numpy.org/doc/stable/reference/arrays.classes.html#matrix-objects
14. **G. H. Golub and C. F. Van Loan, Matrix Computations, 4th ed., Johns Hopkins University Press, 2013** — Standard reference for matrix algorithms.
15. **G. Strang, Linear Algebra and Its Applications, 2nd Ed., Academic Press, 1980** — Referenced by NumPy's `pinv` and `cond` documentation.
16. **LAPACK — Linear Algebra PACKage** — https://www.netlib.org/lapack/
17. **BLAS — Basic Linear Algebra Subprograms** — https://www.netlib.org/blas/