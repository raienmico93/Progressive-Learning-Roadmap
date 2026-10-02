# Introduction to NumPy

NumPy is the foundational library for numerical computing in Python. It provides the core data structures and operations that power nearly every scientific computing, data analysis, and machine learning library in the Python ecosystem.

---

## 1. What NumPy Is

### 1.1 Python Numerical-Computing Library

**NumPy** (Numerical Python) is an **open source Python library** used in almost every field of science and engineering. It is the **universal standard for working with numerical data in Python** and sits at the core of the scientific Python and PyData ecosystems.

### 1.2 Core Array-Processing Library of the Scientific Python Ecosystem

NumPy provides:

- A powerful **N-dimensional array object** (`ndarray`)
- Various **derived objects** (masked arrays, matrices)
- An assortment of **routines for fast operations on arrays**, including mathematical, logical, shape manipulation, sorting, selecting, I/O, discrete Fourier transforms, basic linear algebra, basic statistical operations, random simulation, and much more

The NumPy API is used extensively in **Pandas, SciPy, Matplotlib, scikit-learn, scikit-image, and most other data science and scientific Python packages**.

### 1.3 Importing NumPy

The standard convention is to import NumPy with the alias `np`:

```python
import numpy as np
```

This convention is widely adopted and makes code more readable for everyone working on it.

---

## 2. Why NumPy Exists

### 2.1 Efficient Numerical Computation

Python lists are slow for numerical computation because they store **Python objects** with significant per-element overhead. NumPy arrays store data in **contiguous memory blocks** with **fixed data types**, enabling operations to be executed efficiently with less code than Python's built-in sequences.

### 2.2 Vectorized Operations

**Vectorization** eliminates explicit Python loops — operations are performed on entire arrays at once, with the looping occurring in optimized **C code** instead of Python.

**Python list — explicit loop:**

```python
# Summing 10 million numbers with a Python list
total = 0
for x in my_list:
    total += x
```

**NumPy — vectorized:**

```python
# Summing 10 million numbers with a NumPy array
total = np.sum(my_array)
```

> "NumPy arrays are often an order of magnitude faster than Python lists for numerical computations due to their optimized C implementation and the elimination of Python loops."

### 2.3 Multidimensional Arrays

NumPy's **`ndarray`** is a homogeneous **N-dimensional array object** — it can represent vectors (1D), matrices (2D), and higher-dimensional tensors (3D+).

```python
# 1D array
a = np.array([1, 2, 3])

# 2D array (matrix)
b = np.array([[1, 2, 3],
              [4, 5, 6]])

# 3D array
c = np.array([[[1, 2], [3, 4]],
              [[5, 6], [7, 8]]])
```

### 2.4 Mathematical Functions

NumPy offers **comprehensive mathematical functions**, random number generators, linear algebra routines, Fourier transforms, and more.

```python
np.sin(x)
np.cos(x)
np.exp(x)
np.log(x)
np.sqrt(x)
np.mean(x)
np.std(x)
np.median(x)
```

### 2.5 Linear Algebra

The NumPy linear algebra functions rely on **BLAS and LAPACK** to provide efficient low-level implementations of standard linear algebra algorithms.

```python
from numpy import linalg as LA

# Matrix multiplication
C = A @ B

# Determinant
det = LA.det(A)

# Inverse
A_inv = LA.inv(A)

# Eigenvalues and eigenvectors
eigenvalues, eigenvectors = LA.eig(A)

# Solve linear system Ax = b
x = LA.solve(A, b)
```

---

## 3. NumPy versus Python Lists

### 3.1 Homogeneous Numerical Storage

| Aspect | Python List | NumPy Array |
|---|---|---|
| **Data types** | Can store mixed types | Must be homogeneous (same type) |
| **Size** | Dynamic — can grow/shrink | Fixed at creation |
| **Memory layout** | Pointers to separate objects | Contiguous block of data |
| **Element overhead** | High (Python object per element) | Low (raw C values) |

> "While a Python list can contain different data types within a single list, all of the elements in a NumPy array should be homogeneous. The mathematical operations that are meant to be performed on arrays would be extremely inefficient if the arrays weren't homogeneous."

```python
# Python list — mixed types allowed
my_list = [1, "hello", 3.14, True]

# NumPy array — homogeneous
arr = np.array([1, 2, 3, 4])        # all integers
arr = np.array([1.0, 2.0, 3.0])     # all floats
```

### 3.2 Memory Efficiency

NumPy arrays consume **significantly less memory** than Python lists because they store data in a **contiguous memory block** with fixed data types, while Python lists store each element as a **separate Python object** with a pointer to it.

> "NumPy stores data in a contiguous memory block while Python lists store elements as separate objects. NumPy arrays have fixed data types … this further reduces memory usage compared to Python lists, where each element's size can vary."

**Memory comparison:**

| Element Type | Python List (per element) | NumPy Array (per element) |
|---|---|---|
| Integer | ~28 bytes + 8-byte pointer | 8 bytes (int64) |
| Float | ~24 bytes + 8-byte pointer | 8 bytes (float64) |

For a list of 1 million integers, Python lists use roughly **36 MB** while NumPy uses **8 MB** — a **4.5× reduction**.

### 3.3 Vectorized Computation

| Operation | Python List | NumPy Array |
|---|---|---|
| **Add 1 to every element** | `[x + 1 for x in my_list]` | `arr + 1` |
| **Sum all elements** | `sum(my_list)` (slow) | `np.sum(arr)` (fast) |
| **Multiply two arrays** | `[a * b for a, b in zip(x, y)]` | `x * y` |
| **Element-wise square root** | `[math.sqrt(x) for x in my_list]` | `np.sqrt(arr)` |

NumPy's vectorized operations avoid Python loops entirely, executing at **C speed**.

### 3.4 Broadcasting

**Broadcasting** describes how NumPy treats arrays with **different shapes** during arithmetic operations. Subject to certain constraints, the smaller array is "broadcast" across the larger array so that they have compatible shapes.

> "Broadcasting provides a means of vectorizing array operations so that looping occurs in C instead of Python. It does this without making needless copies of data and usually leads to efficient algorithm implementations."

**Broadcasting rules:**

When operating on two arrays, NumPy compares their shapes **element-wise**, starting from the **trailing (rightmost) dimension** and working left. Two dimensions are compatible when:

1. They are **equal**, or
2. One of them is **1**

```python
# Scalar broadcast
a = np.array([1.0, 2.0, 3.0])
b = 2.0
a * b
# array([2., 4., 6.])

# 1D array broadcast across 2D array
a = np.array([[1, 2, 3],
              [4, 5, 6]])       # shape (2, 3)
b = np.array([10, 20, 30])       # shape (3,)
a + b
# array([[11, 22, 33],
#        [14, 25, 36]])
```

---

## 4. NumPy Ecosystem

NumPy is the **foundation** on which most of the scientific Python ecosystem is built. Nearly every data science and machine learning library either **uses NumPy internally** or **mimics its API**.

### 4.1 Pandas

**Pandas** provides **high-performance, easy-to-use data structures** — primarily `DataFrame` (tabular data) and `Series` (1D labeled array) — for data manipulation and analysis.

| Aspect | Relationship to NumPy |
|---|---|
| **Foundation** | DataFrame and Series are built on NumPy arrays |
| **Use case** | Tabular data, time series, data cleaning |
| **API** | NumPy-like operations, but with labels and indexes |

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "name": ["John", "Maria", "Ahmed"],
    "score": np.array([85, 92, 78])
})
```

### 4.2 SciPy

**SciPy** is a Python-based ecosystem of open-source software for mathematics, science, and engineering, built on top of NumPy.

| Aspect | Relationship to NumPy |
|---|---|
| **Foundation** | Built on NumPy arrays |
| **Scope** | Advanced algorithms: optimization, signal processing, statistics, integration, interpolation, linear algebra |
| **Overlap** | `scipy.linalg` extends `numpy.linalg` |

```python
from scipy import optimize, signal, stats
```

### 4.3 Matplotlib

**Matplotlib** is a **2D/3D plotting library** that integrates tightly with NumPy arrays.

| Aspect | Relationship to NumPy |
|---|---|
| **Input** | Plots NumPy arrays directly |
| **Use case** | Data visualization, scientific plots |
| **Output** | PNG, SVG, PDF, interactive backends |

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 2 * np.pi, 100)
plt.plot(x, np.sin(x))
plt.show()
```

### 4.4 Scikit-learn

**Scikit-learn** is a collection of **algorithms and tools for machine learning**, built on NumPy and SciPy.

| Aspect | Relationship to NumPy |
|---|---|
| **Input** | Accepts NumPy arrays as feature matrices |
| **Use case** | Classification, regression, clustering, dimensionality reduction |
| **API** | `fit()`, `predict()`, `transform()` |

```python
from sklearn.linear_model import LinearRegression
import numpy as np

X = np.array([[1], [2], [3], [4]])
y = np.array([2, 4, 6, 8])
model = LinearRegression().fit(X, y)
```

### 4.5 JAX

**JAX** is a **NumPy-compatible library** for high-performance numerical computing, developed by Google. It provides a **familiar NumPy-style API** with composable function transformations for compilation, batching, autodiff, and autoparallel.

| Aspect | Relationship to NumPy |
|---|---|
| **API** | Nearly identical to NumPy |
| **Key additions** | Automatic differentiation, JIT compilation, GPU/TPU support, vectorization |
| **Use case** | Machine learning research, scientific computing at scale |

> "JAX is numpy + autograd + JIT + other functionalities. Advantage: Numerical computation-wise, JAX is considered faster than NumPy/PyTorch thanks to JIT's ability to optimize computing for different hardware."

```python
import jax.numpy as jnp

x = jnp.array([1.0, 2.0, 3.0])
y = jnp.sin(x)
```

### 4.6 PyTorch

**PyTorch** is a deep learning framework whose **Tensor** object is essentially a **multidimensional array** with GPU support and automatic differentiation.

| Aspect | Relationship to NumPy |
|---|---|
| **Tensor** | Multidimensional array, similar to `ndarray` |
| **Key additions** | GPU acceleration, autograd, neural network modules |
| **Interop** | Converts to/from NumPy seamlessly |

```python
import torch

tensor = torch.tensor([1.0, 2.0, 3.0])
numpy_array = tensor.numpy()
```

### 4.7 TensorFlow

**TensorFlow** is a deep learning framework that defines **Tensor** objects — multidimensional arrays — with computation graphs and GPU/TPU support.

| Aspect | Relationship to NumPy |
|---|---|
| **Tensor** | Multidimensional array |
| **Key additions** | Computation graphs, distributed training, deployment |
| **Interop** | `tf.numpy_function`, `tf.convert_to_tensor` |

```python
import tensorflow as tf

tensor = tf.constant([1.0, 2.0, 3.0])
numpy_array = tensor.numpy()
```

### 4.8 Ecosystem Summary

```
                          ┌─────────────────┐
                          │      JAX        │
                          │  PyTorch        │
                          │  TensorFlow     │
                          └────────┬────────┘
                                   │
                          ┌────────▼────────┐
                          │  Scikit-learn   │
                          └────────┬────────┘
                                   │
                          ┌────────▼────────┐
                          │     Pandas      │
                          │     SciPy       │
                          │   Matplotlib    │
                          └────────┬────────┘
                                   │
                          ┌────────▼────────┐
                          │     NumPy       │
                          │  (ndarray)      │
                          └─────────────────┘
```

| Library | Layer | Relationship to NumPy |
|---|---|---|
| **NumPy** | Foundation | Core array object and operations |
| **Pandas** | Data manipulation | Built on NumPy arrays |
| **SciPy** | Scientific computing | Built on NumPy; extends `linalg` |
| **Matplotlib** | Visualization | Plots NumPy arrays |
| **Scikit-learn** | Machine learning | Accepts NumPy arrays |
| **JAX** | Accelerated computing | NumPy-compatible API + autodiff + JIT |
| **PyTorch** | Deep learning | Tensor ≈ ndarray + GPU + autograd |
| **TensorFlow** | Deep learning | Tensor ≈ ndarray + graphs + GPU |

---

## Summary Table

| Topic | Key Points |
|---|---|
| **What NumPy is** | Open source Python library for numerical computing |
| **Core object** | `ndarray` — homogeneous N-dimensional array |
| **Why it exists** | Efficiency, vectorization, multidimensional arrays, math functions, linear algebra |
| **vs. Python lists** | Homogeneous, fixed-size, contiguous memory, vectorized |
| **Broadcasting** | Operations on arrays of different shapes without copying |
| **Ecosystem** | Pandas, SciPy, Matplotlib, scikit-learn, JAX, PyTorch, TensorFlow |
| **Import convention** | `import numpy as np` |

---

## Key Takeaways

1. **NumPy** is the foundational library for numerical computing in Python — the **universal standard** for working with numerical data.
2. Its core object is the **`ndarray`** — a homogeneous, fixed-size, N-dimensional array stored in contiguous memory.
3. NumPy exists because Python lists are **slow, memory-heavy, and lack vectorized operations**.
4. **Vectorization** eliminates Python loops, performing operations in optimized C code — often **an order of magnitude faster** than Python lists.
5. **Broadcasting** allows arithmetic between arrays of different shapes without copying data.
6. NumPy provides **comprehensive mathematical functions, linear algebra routines (BLAS/LAPACK), Fourier transforms, and random number generators**.
7. The **scientific Python ecosystem** — Pandas, SciPy, Matplotlib, scikit-learn — is built on NumPy.
8. Modern machine learning libraries — **JAX, PyTorch, TensorFlow** — adopt NumPy-compatible APIs or use NumPy internally.
9. **JAX** is essentially "NumPy + autodiff + JIT + GPU/TPU."
10. Learning NumPy is a prerequisite for virtually every data science and scientific computing workflow in Python.

---

Would you like me to continue with the next topic — **NumPy Arrays**, **NumPy Array Creation**, or **NumPy Indexing and Slicing**? I can format the next section in the same style.