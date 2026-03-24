# Linear Algebra

---

## Overview

NumPy's `np.linalg` module provides essential linear algebra operations. This reference covers matrix operations, decompositions, solving linear systems, and eigenvalue computation.

---

## Matrix Operations

### Matrix Multiplication

```python
import numpy as np

A = np.array([[1, 2], [3, 4]], dtype=np.float64)
B = np.array([[5, 6], [7, 8]], dtype=np.float64)

# ✅ Preferred: @ operator (Python 3.5+)
C = A @ B

# Equivalent function call
C = np.matmul(A, B)

# Dot product for 1-D vectors
v1 = np.array([1.0, 2.0, 3.0])
v2 = np.array([4.0, 5.0, 6.0])
dot = v1 @ v2           # scalar: 32.0
dot = np.dot(v1, v2)    # equivalent

# Batch matrix multiply — broadcast over leading dimensions
batch_A = np.random.default_rng(42).standard_normal((10, 3, 4))
batch_B = np.random.default_rng(42).standard_normal((10, 4, 2))
batch_C = batch_A @ batch_B  # shape (10, 3, 2)
```

### Element-wise vs Matrix Operations

```python
# Element-wise multiplication (Hadamard product)
hadamard = A * B

# Matrix multiplication
matmul = A @ B

# Outer product
outer = np.outer(v1, v2)  # shape (3, 3)

# Cross product (3-D vectors)
cross = np.cross(v1, v2)
```

### Transpose and Inverse

```python
# Transpose
A_T = A.T

# Inverse
A_inv = np.linalg.inv(A)
assert np.allclose(A @ A_inv, np.eye(2))

# Pseudo-inverse (for non-square or singular matrices)
A_pinv = np.linalg.pinv(A)

# Determinant
det = np.linalg.det(A)

# Matrix rank
rank = np.linalg.matrix_rank(A)

# Trace
trace = np.trace(A)  # sum of diagonal elements
```

---

## Solving Linear Systems

### Basic Solve: Ax = b

```python
# Solve Ax = b
A = np.array([[3, 1], [1, 2]], dtype=np.float64)
b = np.array([9, 8], dtype=np.float64)

x = np.linalg.solve(A, b)
# Verify
assert np.allclose(A @ x, b)
```

### Least Squares: min ‖Ax − b‖²

```python
# Overdetermined system (more equations than unknowns)
A = np.array([[1, 1], [1, 2], [1, 3]], dtype=np.float64)
b = np.array([1, 2, 2], dtype=np.float64)

# lstsq returns (solution, residuals, rank, singular_values)
x, residuals, rank, sv = np.linalg.lstsq(A, b, rcond=None)
print(f"Solution: {x}, Residuals: {residuals}")
```

---

## Decompositions

### Eigenvalue Decomposition

```python
A = np.array([[4, -2], [1, 1]], dtype=np.float64)

# General eigenvalue decomposition
eigenvalues, eigenvectors = np.linalg.eig(A)

# For symmetric/Hermitian matrices (guaranteed real eigenvalues)
S = np.array([[2, 1], [1, 3]], dtype=np.float64)
eigenvalues, eigenvectors = np.linalg.eigh(S)  # sorted ascending

# Verify: A @ v = λ * v
for i in range(len(eigenvalues)):
    v = eigenvectors[:, i]
    lam = eigenvalues[i]
    assert np.allclose(A @ v, lam * v)
```

### Singular Value Decomposition (SVD)

```python
M = np.array([[1, 0, 0, 0, 2],
              [0, 0, 3, 0, 0],
              [0, 0, 0, 0, 0],
              [0, 2, 0, 0, 0]], dtype=np.float64)

U, s, Vt = np.linalg.svd(M, full_matrices=False)
# U: (4, 4), s: (4,), Vt: (4, 5) — compact form

# Reconstruct
S = np.diag(s)
reconstructed = U @ S @ Vt
assert np.allclose(M, reconstructed)

# Low-rank approximation (keep top k singular values)
k = 2
M_approx = U[:, :k] @ np.diag(s[:k]) @ Vt[:k, :]
```

### Cholesky Decomposition

```python
# For positive-definite matrices
A = np.array([[4, 2], [2, 3]], dtype=np.float64)
L = np.linalg.cholesky(A)

# Verify: A = L @ L.T
assert np.allclose(A, L @ L.T)
```

### QR Decomposition

```python
A = np.array([[1, -1, 4], [1, 4, -2], [1, 4, 2], [1, -1, 0]], dtype=np.float64)
Q, R = np.linalg.qr(A)

# Q is orthogonal: Q.T @ Q ≈ I
assert np.allclose(Q.T @ Q, np.eye(Q.shape[1]))
# A = Q @ R
assert np.allclose(A, Q @ R)
```

---

## Norms

```python
v = np.array([3.0, 4.0])
M = np.array([[1, 2], [3, 4]], dtype=np.float64)

# Vector norms
np.linalg.norm(v)       # L2 (Euclidean): 5.0
np.linalg.norm(v, 1)    # L1 (Manhattan): 7.0
np.linalg.norm(v, np.inf)  # L∞ (max abs): 4.0

# Matrix norms
np.linalg.norm(M, 'fro')  # Frobenius norm
np.linalg.norm(M, 2)      # Spectral norm (largest singular value)

# Row-wise norms for a batch of vectors
points = np.random.default_rng(42).standard_normal((100, 3))
row_norms = np.linalg.norm(points, axis=1)  # shape (100,)
```

---

## Common Patterns for ML

### Covariance Matrix

```python
# Data matrix: rows = samples, columns = features
X = np.random.default_rng(42).standard_normal((100, 5))

# Center the data
X_centered = X - X.mean(axis=0)

# Covariance matrix
cov = (X_centered.T @ X_centered) / (X_centered.shape[0] - 1)
# Equivalent
cov_np = np.cov(X, rowvar=False)
assert np.allclose(cov, cov_np)
```

### PCA via SVD

```python
X_centered = X - X.mean(axis=0)
U, s, Vt = np.linalg.svd(X_centered, full_matrices=False)

# Project onto top k components
k = 2
X_reduced = X_centered @ Vt[:k, :].T  # shape (100, 2)

# Explained variance ratio
explained_var = (s ** 2) / (s ** 2).sum()
print(f"Top {k} components explain {explained_var[:k].sum():.1%} of variance")
```

### Solving Normal Equations (Linear Regression)

```python
# X: (n, p) feature matrix, y: (n,) target
rng = np.random.default_rng(42)
X = rng.standard_normal((100, 3))
y = X @ np.array([1.5, -2.0, 1.0]) + rng.standard_normal(100) * 0.1

# Add intercept
X_b = np.column_stack([np.ones(X.shape[0]), X])

# Normal equation: β = (X'X)^{-1} X'y
beta = np.linalg.solve(X_b.T @ X_b, X_b.T @ y)
# Or via lstsq (numerically more stable)
beta_lstsq, *_ = np.linalg.lstsq(X_b, y, rcond=None)
```

