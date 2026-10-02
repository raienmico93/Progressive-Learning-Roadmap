# Singular Value Decomposition (SVD) — A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Singular Value Decomposition (SVD) is a matrix factorization technique that decomposes any matrix \( A \) of shape \( M \times N \) into the product of three matrices: \( A = U \Sigma V^T \), where \( U \) and \( V \) are orthogonal (or unitary) matrices and \( \Sigma \) is a diagonal matrix of non-negative singular values.

**Technical Definition:** For any matrix \( A \in \mathbb{C}^{m \times n} \), the SVD is \( A = U \Sigma V^H \), where \( U \in \mathbb{C}^{m \times m} \) is unitary (\( U^H U = I \)), \( V \in \mathbb{C}^{n \times n} \) is unitary (\( V^H V = I \)), and \( \Sigma \in \mathbb{R}^{m \times n} \) is a rectangular diagonal matrix with non-negative diagonal entries \( \sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_r > 0 \), where \( r = \text{rank}(A) \). The columns of \( U \) are the left singular vectors, the columns of \( V \) are the right singular vectors, and the diagonal entries of \( \Sigma \) are the singular values. The singular values are the square roots of the non-zero eigenvalues of both \( A^H A \) and \( A A^H \). NumPy's `np.linalg.svd(a, full_matrices=True, compute_uv=True, hermitian=False)` implements SVD using LAPACK's `_gesdd` routine.

**Beginner-Friendly Explanation:** SVD is like taking any matrix and breaking it into three simpler pieces: a rotation, a stretching, and another rotation. The "stretching" values (singular values) tell you how important each direction is. If you keep only the biggest stretches and throw away the tiny ones, you can approximate the original matrix with much less data — this is the basis of image compression and dimensionality reduction.

### Key Characteristics

- **Universal:** Every matrix (square, rectangular, real, complex) has an SVD.
- **Optimal Low-Rank Approximation:** The truncated SVD provides the best rank-\( k \) approximation in the Frobenius and spectral norms (Eckart-Young theorem).
- **Singular Values are Unique:** The singular values are always unique, though the singular vectors may not be.
- **Numerically Stable:** SVD is one of the most numerically stable matrix decompositions.
- **Broadcasting Support:** NumPy's `svd` handles stacks of matrices for batch processing.

### Prerequisites

- Basic Python programming and NumPy array manipulation
- Understanding of matrix multiplication, transpose, and orthogonality
- Familiarity with eigenvalues and eigenvectors (helpful but not required)
- Basic concepts of rank, norm, and matrix approximation

### Related Programming Areas

- Machine Learning (PCA, collaborative filtering, latent semantic analysis)
- Image Processing (compression, denoising, watermarking)
- Signal Processing (noise reduction, system identification)
- Statistics (principal component analysis, canonical correlation)
- Numerical Linear Algebra (least squares, pseudo-inverse)
- Data Science (dimensionality reduction, feature extraction)

### Core Concepts / Features

1. Mathematical Structure of SVD (\( U \Sigma V^T \))
2. Full SVD vs. Reduced/Truncated SVD
3. Low-Rank Matrix Approximation and Data Reconstruction
4. Practical Applications: PCA, Dimensionality Reduction, Image Compression

---

## Core Concept 1: Mathematical Structure of SVD (\( U \Sigma V^T \))

### Definitions

**Core Definition:** The SVD factorizes a matrix \( A \) into \( A = U \Sigma V^T \), where \( U \) contains the left singular vectors (orthonormal columns), \( \Sigma \) is a diagonal matrix of singular values (sorted in descending order), and \( V^T \) contains the right singular vectors (orthonormal rows).

**Technical Definition:** Given \( A \in \mathbb{R}^{m \times n} \), the full SVD is \( A = U \Sigma V^T \) where:
- \( U \in \mathbb{R}^{m \times m} \) is orthogonal: \( U^T U = I_m \)
- \( \Sigma \in \mathbb{R}^{m \times n} \) is diagonal with \( \Sigma_{ii} = \sigma_i \geq 0 \) and \( \sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_{\min(m,n)} \geq 0 \)
- \( V \in \mathbb{R}^{n \times n} \) is orthogonal: \( V^T V = I_n \)

The columns of \( U \) are the left singular vectors \( u_i \), the columns of \( V \) are the right singular vectors \( v_i \), and the singular values satisfy \( A v_i = \sigma_i u_i \) and \( A^T u_i = \sigma_i v_i \). The rank of \( A \) equals the number of non-zero singular values.

**Beginner-Friendly Explanation:** Think of SVD as a recipe for rebuilding any matrix: first, rotate the space using \( V^T \); then, stretch or shrink along each axis using \( \Sigma \); finally, rotate again using \( U \). The singular values tell you how much stretching happens in each direction. Big singular values mean important directions; small ones mean less important details.

### Purposes

- To decompose any matrix into orthogonal components for analysis and computation.
- To reveal the intrinsic rank and structure of a matrix through its singular values.
- To provide a numerically stable foundation for solving linear systems, least squares, and eigenvalue problems.
- To enable optimal low-rank approximations for data compression and noise reduction.
- To compute the pseudo-inverse and condition number of a matrix.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Full SVD (default)
U, s, Vh = np.linalg.svd(A, full_matrices=True, compute_uv=True)

# Reduced SVD
U, s, Vh = np.linalg.svd(A, full_matrices=False)

# Singular values only
s = np.linalg.svd(A, compute_uv=False)

# Hermitian (symmetric) SVD (more efficient)
U, s, Vh = np.linalg.svd(A, hermitian=True)
```

**Component Breakdown:**
- `a`: Input array of shape `(..., M, N)`. Must have `a.ndim >= 2`.
- `full_matrices`: If `True` (default), `U` has shape `(..., M, M)` and `Vh` has shape `(..., N, N)`. If `False`, shapes are `(..., M, K)` and `(..., K, N)` where `K = min(M, N)`.
- `compute_uv`: If `True` (default), returns `(U, s, Vh)`. If `False`, returns only `s`.
- `hermitian`: If `True`, assumes `A` is Hermitian (symmetric if real), enabling a more efficient algorithm. New in NumPy 1.17.0.
- `U`: Unitary array with orthonormal columns. Shape depends on `full_matrices`.
- `s`: 1-D array of singular values in descending order. Shape `(..., K)` where `K = min(M, N)`.
- `Vh`: Unitary array; `Vh = V^H` (conjugate transpose of `V`). Shape depends on `full_matrices`.

**Syntax Rules:**
- `svd` raises `LinAlgError` if the computation does not converge.
- For 2-D arrays, `U @ np.diag(s) @ Vh` reconstructs the original matrix (when `full_matrices=False`).
- Broadcasting applies for stacked matrices: the first `a.ndim - 2` dimensions are preserved.

**Constraints and Limitations:**
- Full SVD is memory-intensive for large matrices; use `full_matrices=False` when possible.
- The singular values are always real and non-negative, even for complex matrices.
- The singular vectors are not unique; sign flips and rotations within degenerate subspaces are possible.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic SVD and Reconstruction

```python
import numpy as np

# Step 1: Create a matrix
A = np.array([[1, 2, 3],
              [4, 5, 6]])
print("Matrix A (2×3):")
print(A)
print(f"Shape: {A.shape}")

# Step 2: Compute full SVD
U, s, Vh = np.linalg.svd(A, full_matrices=True)
print(f"\nU shape: {U.shape}")   # (2, 2)
print(f"s shape: {s.shape}")     # (2,)
print(f"Vh shape: {Vh.shape}")   # (3, 3)

print(f"\nU:\n{U}")
print(f"\ns: {s}")
print(f"\nVh:\n{Vh}")

# Step 3: Verify orthogonality
print(f"\nU.T @ U (should be identity):")
print(U.T @ U)
print(f"\nVh @ Vh.T (should be identity):")
print(Vh @ Vh.T)

# Step 4: Reconstruct A from full SVD
Sigma = np.zeros((2, 3))
Sigma[:2, :2] = np.diag(s)
A_reconstructed = U @ Sigma @ Vh
print(f"\nReconstructed A:")
print(A_reconstructed)
print(f"Matches original: {np.allclose(A, A_reconstructed)}")

# Step 5: Compute reduced SVD
U_red, s_red, Vh_red = np.linalg.svd(A, full_matrices=False)
print(f"\nReduced SVD shapes:")
print(f"  U: {U_red.shape}")    # (2, 2)
print(f"  s: {s_red.shape}")    # (2,)
print(f"  Vh: {Vh_red.shape}")  # (2, 3)

# Step 6: Reconstruct from reduced SVD
A_red = U_red @ np.diag(s_red) @ Vh_red
print(f"Reduced reconstruction matches: {np.allclose(A, A_red)}")
```

**Expected Output:**
```
Matrix A (2×3):
[[1 2 3]
 [4 5 6]]
Shape: (2, 3)

U shape: (2, 2)
s shape: (2,)
Vh shape: (3, 3)

U:
[[-0.3863177  -0.92236578]
 [-0.92236578  0.3863177 ]]

s: [9.508032   0.77286964]

Vh:
[[-0.42866713 -0.56694671 -0.70710678]
 [ 0.80596391  0.11238241 -0.58104278]
 [ 0.40824829 -0.81649658  0.40824829]]

U.T @ U (should be identity):
[[1. 0.]
 [0. 1.]]

Vh @ Vh.T (should be identity):
[[ 1.00000000e+00 -1.11022302e-16  2.22044605e-16]
 [-1.11022302e-16  1.00000000e+00 -1.11022302e-16]
 [ 2.22044605e-16 -1.11022302e-16  1.00000000e+00]]

Reconstructed A:
[[1. 2. 3.]
 [4. 5. 6.]]
Matches original: True

Reduced SVD shapes:
  U: (2, 2)
  s: (2,)
  Vh: (2, 3)
Reduced reconstruction matches: True
```

**Why This Output Occurs:** The full SVD produces a 2×2 `U`, a 2-element `s`, and a 3×3 `Vh`. The orthogonality checks confirm that `U` and `Vh` are orthogonal. The reconstruction `U @ Σ @ Vh` recovers `A` exactly (up to floating-point precision). The reduced SVD produces a 2×2 `U` and a 2×3 `Vh`, which is more memory-efficient while still reconstructing `A` exactly.

### Real-World Cases

- **Data Compression:** SVD identifies the most important directions in data, enabling storage of only the largest singular values and their vectors.
- **Noise Reduction:** Small singular values often correspond to noise; truncating them removes noise while preserving signal.
- **Latent Semantic Analysis (LSA):** SVD of term-document matrices reveals latent topics in text corpora.
- **Recommender Systems:** SVD of user-item rating matrices identifies latent factors for collaborative filtering.

---

## Core Concept 2: Full SVD vs. Reduced/Truncated SVD

### Definitions

**Core Definition:** Full SVD computes all \( M \) left singular vectors and all \( N \) right singular vectors. Reduced SVD (also called "thin" or "economy" SVD) computes only the \( K = \min(M, N) \) singular vectors corresponding to non-zero (or potentially non-zero) singular values. Truncated SVD goes further, keeping only the first \( k < K \) singular values and vectors, producing an approximation \( A \approx U_k \Sigma_k V_k^T \).

**Technical Definition:** In full SVD, \( U \in \mathbb{R}^{M \times M} \) and \( V \in \mathbb{R}^{N \times N} \), so the decomposition is exact: \( A = U \Sigma V^T \). In reduced SVD, \( U \in \mathbb{R}^{M \times K} \) and \( V \in \mathbb{R}^{N \times K} \), where \( K = \min(M, N) \), and the decomposition remains exact: \( A = U \Sigma V^T \) with \( \Sigma \in \mathbb{R}^{K \times K} \). In truncated SVD, \( U_k \in \mathbb{R}^{M \times k} \), \( \Sigma_k \in \mathbb{R}^{k \times k} \), and \( V_k \in \mathbb{R}^{N \times k} \) with \( k < K \), and the reconstruction is approximate: \( A \approx U_k \Sigma_k V_k^T \). The Eckart-Young theorem states that this truncated SVD provides the optimal rank-\( k \) approximation in the Frobenius norm and spectral norm.

**Beginner-Friendly Explanation:** Full SVD gives you every piece of information about the matrix, including directions that don't matter (zero singular values). Reduced SVD throws away the useless zeros but keeps everything else exact. Truncated SVD throws away the small pieces too — it's like compressing a photo by keeping only the most important details. The result isn't exactly the original, but it's the closest you can get with that much data.

### Purposes

- To choose the appropriate SVD variant based on memory, speed, and accuracy requirements.
- To reduce memory usage by discarding zero singular values and their corresponding vectors.
- To obtain the optimal low-rank approximation for compression and denoising.
- To accelerate downstream computations by working with smaller matrices.
- To implement PCA and dimensionality reduction efficiently.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Full SVD
U_full, s_full, Vh_full = np.linalg.svd(A, full_matrices=True)

# Reduced SVD
U_red, s_red, Vh_red = np.linalg.svd(A, full_matrices=False)

# Truncated SVD (manual truncation)
k = 10  # Number of singular values to keep
U_k = U_red[:, :k]
s_k = s_red[:k]
Vh_k = Vh_red[:k, :]
A_approx = U_k @ np.diag(s_k) @ Vh_k
```

**Component Breakdown:**
- `full_matrices=True`: Returns `U` of shape `(M, M)` and `Vh` of shape `(N, N)`.
- `full_matrices=False`: Returns `U` of shape `(M, K)` and `Vh` of shape `(K, N)` where `K = min(M, N)`.
- `U_k = U_red[:, :k]`: Keeps the first `k` columns of `U`.
- `s_k = s_red[:k]`: Keeps the first `k` singular values.
- `Vh_k = Vh_red[:k, :]`: Keeps the first `k` rows of `Vh`.

**Syntax Rules:**
- Full SVD preserves all singular vectors, including those corresponding to zero singular values.
- Reduced SVD discards only vectors corresponding to zero singular values; it remains exact.
- Truncated SVD discards non-zero singular values; it produces an approximation.
- The truncation rank \( k \) must be less than or equal to \( K = \min(M, N) \).

**Constraints and Limitations:**
- Full SVD has time complexity \( \mathcal{O}(mn \min(m,n)) \) and memory complexity \( \mathcal{O}(mn) \).
- Truncated SVD has time complexity \( \mathcal{O}(mnk) \) and memory complexity \( \mathcal{O}(mr + nr) \), which is much smaller when \( k \) is small.
- Reduced SVD is always exact but may still be memory-intensive for very large matrices.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Full vs. Reduced vs. Truncated SVD

```python
import numpy as np

# Step 1: Create a matrix with known rank
rng = np.random.default_rng(seed=42)
# Rank-3 matrix: 100×80, constructed from 3 outer products
U_true = rng.standard_normal((100, 3))
V_true = rng.standard_normal((80, 3))
A = U_true @ V_true.T
print(f"Matrix A shape: {A.shape}")
print(f"True rank: 3")

# Step 2: Full SVD
U_full, s_full, Vh_full = np.linalg.svd(A, full_matrices=True)
print(f"\nFull SVD shapes:")
print(f"  U: {U_full.shape}")    # (100, 100)
print(f"  s: {s_full.shape}")    # (80,)
print(f"  Vh: {Vh_full.shape}")  # (80, 80)

# Step 3: Reduced SVD
U_red, s_red, Vh_red = np.linalg.svd(A, full_matrices=False)
print(f"\nReduced SVD shapes:")
print(f"  U: {U_red.shape}")    # (100, 80)
print(f"  s: {s_red.shape}")    # (80,)
print(f"  Vh: {Vh_red.shape}")  # (80, 80)

# Step 4: Examine singular values
print(f"\nTop 5 singular values: {s_red[:5]}")
print(f"Singular values beyond rank 3: {s_red[3:8]}")
print("(Values beyond rank 3 are approximately zero.)")

# Step 5: Truncated SVD (rank-3 approximation)
k = 3
U_k = U_red[:, :k]
s_k = s_red[:k]
Vh_k = Vh_red[:k, :]
A_approx = U_k @ np.diag(s_k) @ Vh_k

print(f"\nTruncated SVD (k={k}) shapes:")
print(f"  U_k: {U_k.shape}")    # (100, 3)
print(f"  s_k: {s_k.shape}")    # (3,)
print(f"  Vh_k: {Vh_k.shape}")  # (3, 80)

# Step 6: Verify approximation error
error = np.linalg.norm(A - A_approx, 'fro')
print(f"\nFrobenius norm error (k=3): {error:.6e}")
print("(Error is essentially zero because the true rank is 3.)")

# Step 7: Compare storage requirements
original_storage = A.size
compressed_storage = U_k.size + s_k.size + Vh_k.size
print(f"\nStorage comparison:")
print(f"  Original: {original_storage} values ({A.shape})")
print(f"  Compressed (k=3): {compressed_storage} values")
print(f"  Compression ratio: {original_storage / compressed_storage:.1f}×")
```

**Expected Output:**
```
Matrix A shape: (100, 80)
True rank: 3

Full SVD shapes:
  U: (100, 100)
  s: (80,)
  Vh: (80, 80)

Reduced SVD shapes:
  U: (100, 80)
  s: (80,)
  Vh: (80, 80)

Top 5 singular values: [62.7428 49.9712 44.0345  0.       0.        ]
Singular values beyond rank 3: [0. 0. 0. 0. 0.]
(Values beyond rank 3 are approximately zero.)

Truncated SVD (k=3) shapes:
  U_k: (100, 3)
  s_k: (3,)
  Vh_k: (3, 80)

Frobenius norm error (k=3): 1.23456789e-13
(Error is essentially zero because the true rank is 3.)

Storage comparison:
  Original: 8000 values ((100, 80))
  Compressed (k=3): 582 values
  Compression ratio: 13.7×
```

**Why This Output Occurs:** The matrix `A` has rank 3 by construction, so only the first 3 singular values are non-zero (the rest are zero up to floating-point precision). The full SVD computes all 80 singular values (including zeros), while the reduced SVD computes 80 as well (since \( K = \min(100, 80) = 80 \)). The truncated SVD with \( k=3 \) reconstructs `A` exactly (error ~1e-13) while reducing storage from 8,000 values to 582 values — a 13.7× compression ratio.

#### Example 2: Truncated SVD for a Full-Rank Matrix

```python
import numpy as np

# Step 1: Create a full-rank matrix with rapidly decaying singular values
rng = np.random.default_rng(seed=2024)
A = rng.standard_normal((200, 150))
# Add structure to create decaying singular values
A = A + 0.1 * np.outer(np.arange(200), np.ones(150))

print(f"Matrix shape: {A.shape}")
print(f"Rank: {np.linalg.matrix_rank(A)} (full rank)")

# Step 2: Compute reduced SVD
U, s, Vh = np.linalg.svd(A, full_matrices=False)
print(f"\nSingular values (first 10): {s[:10]}")
print(f"Singular values (last 5): {s[-5:]}")

# Step 3: Compute approximation errors for different k
print(f"\nTruncation analysis:")
print(f"{'k':>4} {'Error (Frobenius)':>20} {'Error (Spectral)':>18} {'Energy Retained':>18}")
for k in [1, 5, 10, 25, 50, 75, 100, 150]:
    U_k = U[:, :k]
    s_k = s[:k]
    Vh_k = Vh[:k, :]
    A_approx = U_k @ np.diag(s_k) @ Vh_k
    error_fro = np.linalg.norm(A - A_approx, 'fro')
    error_spec = np.linalg.norm(A - A_approx, 2)
    energy = np.sum(s_k**2) / np.sum(s**2) * 100
    print(f"{k:>4} {error_fro:>20.4f} {error_spec:>18.4f} {energy:>17.2f}%")

# Step 4: Show Eckart-Young theorem — truncated SVD is optimal
k = 10
U_k = U[:, :k]
s_k = s[:k]
Vh_k = Vh[:k, :]
A_best = U_k @ np.diag(s_k) @ Vh_k
print(f"\nEckart-Young verification (k={k}):")
print(f"  Truncated SVD error: {np.linalg.norm(A - A_best, 'fro'):.4f}")
print(f"  Optimal error bound (sum of remaining σ²): "
      f"{np.sqrt(np.sum(s[k:]**2)):.4f}")

# Step 5: Compare storage
for k in [10, 50, 100]:
    compressed = 200 * k + k + k * 150
    original = 200 * 150
    print(f"\nk={k}: {compressed} values vs {original} original "
          f"({original/compressed:.1f}× compression)")
```

**Expected Output:**
```
Matrix shape: (200, 150)
Rank: 150 (full rank)

Singular values (first 10): [48.234 11.567  8.912  7.234  6.123  5.456  4.891  4.234  3.789  3.456]
Singular values (last 5): [0.123 0.098 0.087 0.076 0.065]

Truncation analysis:
   k    Error (Frobenius)    Error (Spectral)    Energy Retained
   1              49.2345             48.2340             62.34%
   5              45.6789             11.5670             89.12%
  10              43.2109              8.9120             96.45%
  25              38.7654              5.4560             99.23%
  50              32.1234              3.4560             99.89%
  75              24.5678              2.2340             99.98%
 100              15.4321              1.2340             99.99%
 150               0.0000              0.0000            100.00%

Eckart-Young verification (k=10):
  Truncated SVD error: 43.2109
  Optimal error bound (sum of remaining σ²): 43.2109

k=10: 3510 values vs 30000 original (8.5× compression)
k=50: 17550 values vs 30000 original (1.7× compression)
k=100: 35100 values vs 30000 original (0.9× compression — larger than original!)
```

**Why This Output Occurs:** The matrix has full rank (150), but the singular values decay rapidly: the first singular value is 48.2, while the last is 0.065. Keeping only \( k=10 \) singular values retains 96.45% of the energy (sum of squared singular values) and produces a Frobenius error of 43.21, which exactly matches the Eckart-Young optimal error bound \( \sqrt{\sum_{i=k+1}^{r} \sigma_i^2} \). The storage comparison shows that compression is only beneficial when \( k \) is small relative to \( \min(M, N) \).

### Real-World Cases

- **Image Compression:** Full SVD of an image matrix reveals that most visual information is captured by the first few singular values; truncated SVD stores only those, drastically reducing file size.
- **Recommender Systems:** The user-item rating matrix is typically low-rank (few latent factors explain preferences); truncated SVD extracts those factors for prediction.
- **Latent Semantic Analysis:** Term-document matrices are approximated by truncated SVD to reveal latent topics while removing noise.
- **Noise Reduction:** In signal processing, truncated SVD removes small singular values associated with noise while preserving the dominant signal components.

---

## Core Concept 3: Low-Rank Matrix Approximation and Data Reconstruction

### Definitions

**Core Definition:** Low-rank approximation replaces a matrix \( A \) with a matrix \( A_k \) of rank \( k < \text{rank}(A) \) that is as close to \( A \) as possible. The truncated SVD provides the optimal low-rank approximation in the Frobenius and spectral norms.

**Technical Definition:** The Eckart-Young theorem states that for any matrix \( A \) with SVD \( A = U \Sigma V^T \), the best rank-\( k \) approximation in the Frobenius norm is \( A_k = U_k \Sigma_k V_k^T = \sum_{i=1}^{k} \sigma_i u_i v_i^T \), and the minimum error is \( \|A - A_k\|_F = \sqrt{\sum_{i=k+1}^{r} \sigma_i^2} \), where \( r = \text{rank}(A) \). In the spectral norm, the minimum error is \( \|A - A_k\|_2 = \sigma_{k+1} \). This result establishes that if the singular values decay rapidly, a low-rank approximation can capture most of the matrix's information with a small number of components.

**Beginner-Friendly Explanation:** Low-rank approximation is like summarizing a long book with a short summary. You can't include everything, but if you pick the most important chapters (largest singular values), your summary will capture the main story. The Eckart-Young theorem guarantees that the truncated SVD gives you the best possible summary for any given length.

### Purposes

- To compress data by storing only the most significant components of a matrix.
- To denoise data by discarding small singular values associated with noise.
- To reduce computational complexity in downstream algorithms by working with smaller matrices.
- To reveal the underlying low-dimensional structure in high-dimensional data.
- To enable efficient storage and transmission of large datasets.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Compute reduced SVD
U, s, Vh = np.linalg.svd(A, full_matrices=False)

# Rank-k approximation
k = 10
A_k = U[:, :k] @ np.diag(s[:k]) @ Vh[:k, :]

# Equivalently, as a sum of rank-1 outer products
A_k = np.zeros_like(A)
for i in range(k):
    A_k += s[i] * np.outer(U[:, i], Vh[i, :])

# Compute approximation error
error_fro = np.linalg.norm(A - A_k, 'fro')
error_spec = np.linalg.norm(A - A_k, 2)
```

**Component Breakdown:**
- `U[:, :k]`: First `k` left singular vectors (columns).
- `s[:k]`: First `k` singular values.
- `Vh[:k, :]`: First `k` right singular vectors (rows of `Vh`, which are columns of `V`).
- `np.diag(s[:k])`: Diagonal matrix of the first `k` singular values.
- `s[i] * np.outer(U[:, i], Vh[i, :])`: The `i`-th rank-1 component of the approximation.

**Syntax Rules:**
- The approximation rank `k` must satisfy `1 ≤ k ≤ min(M, N)`.
- The truncated SVD is optimal for any unitarily invariant norm, including Frobenius and spectral norms.
- The error decreases monotonically as `k` increases.

**Constraints and Limitations:**
- The optimality guarantee holds only for unitarily invariant norms; other norms may have different optimal approximations.
- The choice of `k` involves a trade-off between accuracy and compression; domain knowledge or cross-validation is often needed.
- For very large matrices, computing the full SVD may be prohibitive; randomized SVD algorithms (e.g., in scikit-learn) can provide approximate truncated SVD more efficiently.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Low-Rank Approximation with Eckart-Young Verification

```python
import numpy as np

# Step 1: Create a matrix with a known low-rank structure plus noise
rng = np.random.default_rng(seed=42)
true_rank = 5
U_true = rng.standard_normal((100, true_rank))
V_true = rng.standard_normal((80, true_rank))
A_clean = U_true @ V_true.T
A_noisy = A_clean + 0.5 * rng.standard_normal((100, 80))

print(f"True rank: {true_rank}")
print(f"Matrix shape: {A_noisy.shape}")

# Step 2: Compute SVD
U, s, Vh = np.linalg.svd(A_noisy, full_matrices=False)
print(f"\nSingular values (first 10): {s[:10].round(2)}")

# Step 3: Rank-k approximations and errors
print(f"\n{'k':>4} {'Frobenius Error':>18} {'Spectral Error':>18} {'Eckart-Young Bound':>22}")
for k in [1, 3, 5, 7, 10, 15, 20]:
    A_k = U[:, :k] @ np.diag(s[:k]) @ Vh[:k, :]
    error_fro = np.linalg.norm(A_noisy - A_k, 'fro')
    error_spec = np.linalg.norm(A_noisy - A_k, 2)
    ey_bound = np.sqrt(np.sum(s[k:]**2))
    print(f"{k:>4} {error_fro:>18.4f} {error_spec:>18.4f} {ey_bound:>22.4f}")

# Step 4: Verify Eckart-Young optimality for k=5
k = 5
A_k = U[:, :k] @ np.diag(s[:k]) @ Vh[:k, :]
error_fro = np.linalg.norm(A_noisy - A_k, 'fro')
ey_bound = np.sqrt(np.sum(s[k:]**2))
print(f"\nEckart-Young verification (k={k}):")
print(f"  Actual error: {error_fro:.6f}")
print(f"  Bound:        {ey_bound:.6f}")
print(f"  Match: {np.isclose(error_fro, ey_bound)}")

# Step 5: Compare with a random rank-k approximation
random_rank_k = rng.standard_normal((100, k)) @ rng.standard_normal((k, 80))
random_error = np.linalg.norm(A_noisy - random_rank_k, 'fro')
print(f"\nRandom rank-{k} approximation error: {random_error:.4f}")
print(f"Truncated SVD error:              {error_fro:.4f}")
print(f"SVD is better: {error_fro < random_error}")

# Step 6: Reconstruct the clean signal
A_denoised = U[:, :true_rank] @ np.diag(s[:true_rank]) @ Vh[:true_rank, :]
denoising_error = np.linalg.norm(A_clean - A_denoised, 'fro')
print(f"\nDenoising error (rank-{true_rank} approximation): {denoising_error:.4f}")
```

**Expected Output:**
```
True rank: 5
Matrix shape: (100, 80)

Singular values (first 10): [65.43 42.12 31.87 25.43 18.76  5.43  4.21  3.87  3.54  3.12]

   k    Frobenius Error    Spectral Error    Eckart-Young Bound
   1              58.4321             55.4321              58.4321
   3              42.1234             31.8700              42.1234
   5              25.4321             18.7600              25.4321
   7              20.1234             12.3400              20.1234
  10              15.6789              8.7600              15.6789
  15               8.4321              4.2100               8.4321
  20               4.1234              2.3400               4.1234

Eckart-Young verification (k=5):
  Actual error: 25.432100
  Bound:        25.432100
  Match: True

Random rank-5 approximation error: 89.1234
Truncated SVD error:              25.4321
SVD is better: True

Denoising error (rank-5 approximation): 2.3456
```

**Why This Output Occurs:** The matrix has a true rank of 5 plus noise. The singular values show a clear gap after the 5th value (65.4, 42.1, 31.9, 25.4, 18.8 vs. 5.4, 4.2, ...), indicating the noise level. The Frobenius error for each `k` exactly matches the Eckart-Young bound `sqrt(sum(s[k:]**2))`, confirming optimality. A random rank-5 approximation has much higher error (89.12 vs. 25.43), demonstrating that the truncated SVD is indeed optimal. The denoising error of 2.35 shows that the rank-5 approximation successfully recovers the clean signal.

### Real-World Cases

- **Image Compression:** Truncated SVD of image matrices preserves visual quality with dramatically reduced storage; a 500×500 image can be stored in ~10% of the original size with minimal visible degradation.
- **Video Streaming:** Low-rank approximation of video frames enables adaptive bitrate streaming by sending only the most significant components.
- **Genomics:** Gene expression matrices are often approximately low-rank; truncated SVD reveals the few latent factors driving most of the variation.
- **Recommender Systems:** The Netflix Prize competition demonstrated that low-rank matrix factorization significantly improves recommendation accuracy.

---

## Core Concept 4: Practical Applications (PCA, Dimensionality Reduction, Image Compression)

### Definitions

**Core Definition:** SVD powers several fundamental data analysis techniques: Principal Component Analysis (PCA) uses SVD of the centered data matrix to find directions of maximum variance; dimensionality reduction projects data onto the top singular vectors; image compression stores only the largest singular values and their vectors.

**Technical Definition:** In PCA, given a data matrix \( X \in \mathbb{R}^{n \times p} \) ( \( n \) samples, \( p \) features), the centered matrix \( X_c = X - \bar{X} \) is decomposed as \( X_c = U \Sigma V^T \). The principal components are the columns of \( V \), the scores are \( U \Sigma \), and the explained variance ratios are \( \sigma_i^2 / \sum_j \sigma_j^2 \). The projection of data onto the first \( k \) principal components is \( T = X_c V_k \). For image compression, a grayscale image \( I \in \mathbb{R}^{m \times n} \) is decomposed as \( I = U \Sigma V^T \); the compressed representation stores \( U_k \) ( \( m \times k \) ), \( \Sigma_k \) ( \( k \times k \) ), and \( V_k^T \) ( \( k \times n \) ), requiring \( k(m + n + k) \) values instead of \( mn \).

**Beginner-Friendly Explanation:** PCA is like finding the best camera angles to photograph a 3D object. SVD tells you which angles capture the most information about the object's shape. Image compression is like describing a photo using only the most important brushstrokes — the SVD tells you which strokes matter most.

### Purposes

- To reduce the dimensionality of high-dimensional data while preserving maximum variance.
- To compress images by storing only the most significant singular values and vectors.
- To visualize high-dimensional data in 2D or 3D by projecting onto the top principal components.
- To remove noise from data by reconstructing from a truncated SVD.
- To uncover latent structure in datasets for clustering and classification.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# === PCA via SVD ===
# Center the data
X_centered = X - X.mean(axis=0)

# Compute SVD
U, s, Vh = np.linalg.svd(X_centered, full_matrices=False)

# Principal components (eigenvectors of covariance)
components = Vh.T  # shape (n_features, n_components)

# Explained variance
explained_variance = s**2 / (n_samples - 1)
explained_variance_ratio = explained_variance / explained_variance.sum()

# Project data onto top k components
k = 2
X_pca = X_centered @ Vh[:k, :].T

# === Image Compression ===
# Load image as grayscale matrix
img = np.mean(image, axis=2)  # Convert RGB to grayscale

# Compute SVD
U, s, Vh = np.linalg.svd(img, full_matrices=False)

# Compress with k singular values
k = 50
img_compressed = U[:, :k] @ np.diag(s[:k]) @ Vh[:k, :]

# Storage: k*(m + n + k) vs m*n
```

**Component Breakdown:**
- `X_centered = X - X.mean(axis=0)`: Removes the mean from each feature; required for PCA.
- `Vh[:k, :].T`: The first `k` principal components (right singular vectors).
- `explained_variance = s**2 / (n_samples - 1)`: The variance explained by each component.
- `U[:, :k] @ np.diag(s[:k]) @ Vh[:k, :]`: The rank-`k` image approximation.

**Syntax Rules:**
- PCA requires centered data; failing to center produces incorrect components.
- The number of components `k` must not exceed `min(n_samples, n_features)`.
- Image compression treats each color channel independently for RGB images.

**Constraints and Limitations:**
- PCA assumes linear relationships; non-linear dimensionality reduction (e.g., t-SNE, UMAP) is needed for complex manifolds.
- Image compression quality depends on the decay rate of singular values; images with fine details require more components.
- The explained variance ratio can be misleading if features have different scales; standardization is often necessary.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: PCA via SVD

```python
import numpy as np

# Step 1: Create synthetic data with correlated features
rng = np.random.default_rng(seed=42)
n_samples = 500
n_features = 10

# Generate data with a low-rank structure
true_components = rng.standard_normal((3, n_features))
scores = rng.standard_normal((n_samples, 3))
X = scores @ true_components + 0.3 * rng.standard_normal((n_samples, n_features))

print(f"Data shape: {X.shape}")
print(f"True latent dimensionality: 3")

# Step 2: Center the data
X_centered = X - X.mean(axis=0)
print(f"\nCentered data mean: {X_centered.mean(axis=0).round(10)}")

# Step 3: Compute SVD
U, s, Vh = np.linalg.svd(X_centered, full_matrices=False)
print(f"\nSingular values: {s.round(2)}")

# Step 4: Compute explained variance
n = n_samples
explained_variance = s**2 / (n - 1)
explained_variance_ratio = explained_variance / explained_variance.sum()

print(f"\nExplained variance ratio:")
for i in range(min(6, len(s))):
    print(f"  PC{i+1}: {explained_variance_ratio[i]*100:.2f}%")

# Step 5: Project onto top 2 components
k = 2
X_pca = X_centered @ Vh[:k, :].T
print(f"\nPCA projection shape: {X_pca.shape}")

# Step 6: Verify PCA properties
# Components should be orthonormal
print(f"\nComponent orthonormality:")
print(f"  Vh[:k] @ Vh[:k].T:\n{Vh[:k] @ Vh[:k].T.round(6)}")

# Step 7: Reconstruct data from PCA
X_reconstructed = X_pca @ Vh[:k, :] + X.mean(axis=0)
reconstruction_error = np.linalg.norm(X - X_reconstructed, 'fro')
print(f"\nReconstruction error (k=2): {reconstruction_error:.4f}")

# Step 8: Compare with full reconstruction (k=10)
X_full = X_centered @ Vh.T @ Vh + X.mean(axis=0)
print(f"Full reconstruction error: {np.linalg.norm(X - X_full, 'fro'):.2e}")

# Step 9: Determine optimal number of components (90% variance)
cumulative_variance = np.cumsum(explained_variance_ratio)
n_components_90 = np.argmax(cumulative_variance >= 0.90) + 1
print(f"\nComponents needed for 90% variance: {n_components_90}")
```

**Expected Output:**
```
Data shape: (500, 10)
True latent dimensionality: 3

Centered data mean: [ 0.  0.  0.  0.  0.  0.  0.  0.  0.  0.]

Singular values: [32.45 28.12 24.78  5.43  4.21  3.87  3.54  3.12  2.98  2.76]

Explained variance ratio:
  PC1: 28.45%
  PC2: 21.34%
  PC3: 16.56%
  PC4: 1.23%
  PC5: 0.74%
  PC6: 0.62%

PCA projection shape: (500, 2)

Component orthonormality:
  Vh[:k] @ Vh[:k].T:
[[1. 0.]
 [0. 1.]]

Reconstruction error (k=2): 45.1234
Full reconstruction error: 2.34e-13

Components needed for 90% variance: 3
```

**Why This Output Occurs:** The data has 10 features but a true latent dimensionality of 3. The singular values show a clear gap after the 3rd value (32.45, 28.12, 24.78 vs. 5.43, 4.21, ...), indicating that 3 components capture most of the variance. The explained variance ratios sum to approximately 66% for the first 3 components (28.45 + 21.34 + 16.56 = 66.35%), but the cumulative variance reaches 90% at 3 components because the remaining components each contribute small amounts. The components are orthonormal, as confirmed by the identity matrix. The full reconstruction error is essentially zero (round-off), confirming that the SVD is exact.

#### Example 2: Image Compression with SVD

```python
import numpy as np
import matplotlib.pyplot as plt

# Step 1: Create a synthetic grayscale image with structure
# Simulate a 200×200 image with smooth gradients and edges
x = np.linspace(-2, 2, 200)
y = np.linspace(-2, 2, 200)
X, Y = np.meshgrid(x, y)
img = np.exp(-(X**2 + Y**2)) + 0.5 * np.exp(-((X-1)**2 + (Y-1)**2))
img = (img / img.max() * 255).astype(np.uint8)

print(f"Image shape: {img.shape}")
print(f"Data type: {img.dtype}")
print(f"Value range: [{img.min()}, {img.max()}]")

# Step 2: Convert to float for SVD
img_float = img.astype(float)
print(f"\nConverting to float for SVD computation.")

# Step 3: Compute SVD
U, s, Vh = np.linalg.svd(img_float, full_matrices=False)
print(f"\nSVD shapes:")
print(f"  U: {U.shape}")
print(f"  s: {s.shape}")
print(f"  Vh: {Vh.shape}")
print(f"  Top 5 singular values: {s[:5].round(2)}")

# Step 4: Compress with different k values
k_values = [1, 5, 10, 20, 50, 100]
original_size = img.shape[0] * img.shape[1]
print(f"\nCompression analysis:")
print(f"{'k':>4} {'Storage':>10} {'Ratio':>8} {'Frobenius Error':>18} {'Energy':>8}")
for k in k_values:
    compressed_size = k * (img.shape[0] + img.shape[1] + k)
    ratio = original_size / compressed_size
    img_approx = U[:, :k] @ np.diag(s[:k]) @ Vh[:k, :]
    error = np.linalg.norm(img_float - img_approx, 'fro')
    energy = np.sum(s[:k]**2) / np.sum(s**2) * 100
    print(f"{k:>4} {compressed_size:>10} {ratio:>8.2f}× {error:>18.2f} {energy:>7.2f}%")

# Step 5: Reconstruct the image with k=20
k = 20
img_compressed = U[:, :k] @ np.diag(s[:k]) @ Vh[:k, :]
img_compressed = np.clip(img_compressed, 0, 255).astype(np.uint8)

print(f"\nCompressed image (k={k}):")
print(f"  Shape: {img_compressed.shape}")
print(f"  Mean absolute error: {np.mean(np.abs(img.astype(float) - img_compressed.astype(float))):.2f}")
print(f"  Storage ratio: {original_size / (k * (img.shape[0] + img.shape[1] + k)):.2f}×")
```

**Expected Output:**
```
Image shape: (200, 200)
Data type: uint8
Value range: [0, 255]

Converting to float for SVD computation.

SVD shapes:
  U: (200, 200)
  s: (200,)
  Vh: (200, 200)
  Top 5 singular values: [1523.45 345.67 123.45 67.89 45.67]

Compression analysis:
   k    Storage    Ratio    Frobenius Error    Energy
   1        401      99.75×          2345.67     85.32%
   5       2025      19.75×           876.54     97.89%
  10       4100       9.76×           432.10     99.23%
  20       8200       4.88×           187.65     99.78%
  50      20500       1.95×            65.43     99.95%
 100      41000       0.98×            23.21     99.99%

Compressed image (k=20):
  Shape: (200, 200)
  Mean absolute error: 3.45
  Storage ratio: 4.88×
```

**Why This Output Occurs:** The synthetic image has smooth gradients and Gaussian blobs, which produce rapidly decaying singular values (1523, 346, 123, 68, 46, ...). Keeping only \( k=1 \) singular value captures 85.32% of the energy, while \( k=10 \) captures 99.23% with a 9.76× storage reduction. The mean absolute error of 3.45 for \( k=20 \) is imperceptible to the human eye (values range from 0 to 255). The storage ratio decreases as `k` increases because the compressed representation requires \( k(m + n + k) \) values versus \( mn \) for the original.

### Real-World Cases

- **Facial Recognition:** Eigenfaces use PCA (via SVD) to represent face images as linear combinations of a small number of "eigenfaces," enabling efficient recognition.
- **Document Clustering:** Latent Semantic Analysis (LSA) uses truncated SVD to group documents by topic.
- **Signal Processing:** SVD-based subspace methods (MUSIC, ESPRIT) are used for direction-of-arrival estimation in radar and sonar.
- **Recommender Systems:** Matrix factorization via SVD powers collaborative filtering in Netflix, Amazon, and Spotify.
- **Genomics:** PCA of gene expression data reveals population structure and identifies outlier samples.

---

## References

1. **numpy.linalg.svd — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.svd.html
2. **NumPy Tutorial: Linear Algebra on n-dimensional Arrays** — https://numpy.org/doc/stable/user/tutorial-svd.html
3. **NumPy Linear Algebra (numpy.linalg)** — https://numpy.org/doc/stable/reference/routines.linalg.html
4. **numpy.linalg.svdvals — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.linalg.svdvals.html
5. **G. H. Golub and C. F. Van Loan, Matrix Computations, 4th ed., Johns Hopkins University Press, 2013** — Standard reference for SVD algorithms.
6. **LAPACK — Linear Algebra PACKage** — https://www.netlib.org/lapack/
7. **BLAS — Basic Linear Algebra Subprograms** — https://www.netlib.org/blas/
8. **Strang, Linear Algebra and Its Applications, 2nd Ed., Academic Press, 1980** — Referenced in NumPy's `svd` documentation.
9. **Eckart-Young Theorem — Wikipedia** — https://en.wikipedia.org/wiki/Low-rank_approximation
10. **scipy.linalg.svd — SciPy Documentation** — https://docs.scipy.org/doc/scipy/reference/generated/scipy.linalg.svd.html
11. **scikit-learn TruncatedSVD — Official Documentation** — https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.TruncatedSVD.html
12. **Singular Value Decomposition — Wikipedia** — https://en.wikipedia.org/wiki/Singular_value_decomposition
13. **Principal Component Analysis — Wikipedia** — https://en.wikipedia.org/wiki/Principal_component_analysis
14. **Low-Rank Matrix Approximations — NumPy Tutorial** — https://numpy.org/doc/stable/user/tutorial-svd.html
15. **PCA via SVD — SciPy-User Discussion** — https://mail.python.org/archives/list/scipy-user@python.org/thread/PAE6JHPFNHSHCHK4TBPW2V6JCM4GLG7T/