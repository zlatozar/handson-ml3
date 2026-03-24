# Broadcasting and Vectorization

---

## Overview

Broadcasting and vectorized operations are the keys to writing fast, readable NumPy code. This reference covers broadcasting rules, universal functions (ufuncs), and common vectorization patterns.

---

## Broadcasting Rules

NumPy compares shapes element-wise from the **trailing dimensions** forward. Two dimensions are compatible when:
1. They are equal, **or**
2. One of them is 1

```
Shape (4, 3) + Shape (3,)    → (4, 3)   ✅  trailing dims match
Shape (4, 3) + Shape (4, 1)  → (4, 3)   ✅  dim 1 broadcasts
Shape (4, 3) + Shape (4,)    → ERROR    ❌  trailing dims 3 ≠ 4
Shape (2, 1, 3) + Shape (4, 3) → (2, 4, 3) ✅
```

### Verifying Broadcast Compatibility

```python
import numpy as np

def check_broadcast(*shapes: tuple[int, ...]) -> tuple[int, ...]:
    """Return the result shape or raise ValueError."""
    return np.broadcast_shapes(*shapes)

# Examples
print(check_broadcast((4, 3), (3,)))       # (4, 3)
print(check_broadcast((2, 1, 3), (4, 3)))  # (2, 4, 3)
```

### Common Broadcasting Patterns

```python
matrix = np.arange(12, dtype=np.float64).reshape(3, 4)

# Subtract column means — center each column
col_means = matrix.mean(axis=0)  # shape (4,)
centered = matrix - col_means    # (3, 4) - (4,) → (3, 4)

# Subtract row means — center each row
row_means = matrix.mean(axis=1, keepdims=True)  # shape (3, 1)
centered = matrix - row_means  # (3, 4) - (3, 1) → (3, 4)

# Outer product via broadcasting
a = np.array([1, 2, 3])[:, np.newaxis]  # shape (3, 1)
b = np.array([10, 20, 30, 40])          # shape (4,)
outer = a * b  # shape (3, 4)

# Pairwise distances via broadcasting
rng = np.random.default_rng(seed=42)
points = rng.standard_normal((100, 3))  # 100 points in 3-D
diff = points[:, np.newaxis, :] - points[np.newaxis, :, :]  # (100, 100, 3)
distances = np.linalg.norm(diff, axis=2)  # (100, 100)
```

---

## Universal Functions (ufuncs)

Ufuncs operate element-wise and are implemented in C for speed.

### Common Ufuncs

```python
arr = np.array([1.0, 4.0, 9.0, 16.0])

# Arithmetic
np.add(arr, 1)          # arr + 1
np.multiply(arr, 2)     # arr * 2
np.power(arr, 0.5)      # arr ** 0.5

# Math
np.sqrt(arr)            # [1, 2, 3, 4]
np.exp(arr)             # e^x
np.log(arr)             # natural log
np.log10(arr)           # log base 10

# Trigonometry
angles = np.linspace(0, np.pi, 5)
np.sin(angles)
np.cos(angles)

# Comparison (return boolean arrays)
np.greater(arr, 5)      # arr > 5
np.equal(arr, 4)        # arr == 4
```

### Reduce, Accumulate, Outer

```python
arr = np.array([1, 2, 3, 4, 5])

# Reduce — apply ufunc across entire array
total = np.add.reduce(arr)         # 15 (same as np.sum)
product = np.multiply.reduce(arr)  # 120 (same as np.prod)

# Accumulate — running result
running_sum = np.add.accumulate(arr)        # [1, 3, 6, 10, 15]
running_prod = np.multiply.accumulate(arr)  # [1, 2, 6, 24, 120]

# Outer — all pairwise combinations
outer = np.multiply.outer(arr, arr)  # shape (5, 5)
```

### Using `out` Parameter to Avoid Temporaries

```python
a = np.arange(1_000_000, dtype=np.float64)
result = np.empty_like(a)

# Writes directly into pre-allocated array — no temporary
np.multiply(a, 2.0, out=result)
np.add(result, 1.0, out=result)
```

---

## Vectorization Patterns

### Replace Python Loops

```python
# ❌ AVOID: Python loop
result = np.empty(len(arr))
for i in range(len(arr)):
    result[i] = arr[i] ** 2 + 2 * arr[i] + 1

# ✅ USE: vectorized expression
result = arr ** 2 + 2 * arr + 1
```

### Conditional Logic

```python
# ❌ AVOID: if/else in a loop
for i in range(len(arr)):
    if arr[i] > 0:
        result[i] = arr[i]
    else:
        result[i] = 0

# ✅ USE: np.where
result = np.where(arr > 0, arr, 0)

# ✅ USE: np.clip for bounding
clipped = np.clip(arr, a_min=0, a_max=100)
```

### Multiple Conditions

```python
# ❌ AVOID: nested if/elif in apply
# ✅ USE: np.select
conditions = [
    arr < 0,
    (arr >= 0) & (arr < 10),
    arr >= 10,
]
choices = [
    -1,    # negative → -1
    arr,   # 0-9 → keep
    10,    # 10+ → cap at 10
]
result = np.select(conditions, choices, default=0)
```

### Custom Vectorized Functions

```python
# Wrap a scalar Python function for element-wise use
def custom_fn(x: float) -> float:
    """Example scalar function."""
    if x > 0:
        return x ** 0.5
    return 0.0

vec_fn = np.vectorize(custom_fn, otypes=[np.float64])
result = vec_fn(arr)
# NOTE: np.vectorize is syntactic sugar — NOT a performance tool.
# For speed, rewrite with ufuncs and np.where instead.
```

---

## Aggregation with Axis Control

```python
matrix = np.arange(12, dtype=np.float64).reshape(3, 4)

# Flatten aggregation (default)
np.sum(matrix)          # scalar: sum of all elements

# Along axis 0 — collapse rows → one value per column
np.sum(matrix, axis=0)  # shape (4,)
np.mean(matrix, axis=0)

# Along axis 1 — collapse columns → one value per row
np.sum(matrix, axis=1)  # shape (3,)
np.std(matrix, axis=1)

# Keep dimensions for broadcasting
row_means = np.mean(matrix, axis=1, keepdims=True)  # shape (3, 1)
normalized = matrix - row_means  # broadcasts correctly

# Multiple axes at once
np.sum(matrix, axis=(0, 1))  # same as np.sum(matrix)
```

