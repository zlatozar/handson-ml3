# Array Creation and Manipulation

---

## Overview

Array creation and reshaping form the foundation of NumPy work. This reference covers constructing, reshaping, stacking, and splitting arrays with NumPy 2.0+ best practices.

---

## Array Creation

### From Python Objects

```python
import numpy as np

# From list — dtype inferred
arr = np.array([1, 2, 3, 4])

# Explicit dtype for numerical stability
arr_f64 = np.array([1.0, 2.0, 3.0], dtype=np.float64)
arr_i32 = np.array([1, 2, 3], dtype=np.int32)

# 2-D from nested lists
matrix = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.float64)

# From tuples (same rules apply)
arr = np.array((10, 20, 30))
```

### Factory Functions

```python
# Zeros and ones
zeros = np.zeros((3, 4), dtype=np.float64)
ones = np.ones((2, 3), dtype=np.int32)

# Full with a constant value
filled = np.full((3, 3), fill_value=7, dtype=np.float64)

# Identity matrix
eye = np.eye(4, dtype=np.float64)

# Empty (uninitialized — faster but unsafe to read before writing)
empty = np.empty((1000, 1000), dtype=np.float64)

# Like variants — match shape and dtype of existing array
zeros_like = np.zeros_like(matrix)
ones_like = np.ones_like(matrix)
```

### Ranges and Sequences

```python
# Integer range (exclusive end)
r = np.arange(0, 10, 2)  # [0, 2, 4, 6, 8]

# Linearly spaced (inclusive both ends)
lin = np.linspace(0.0, 1.0, num=5)  # [0.0, 0.25, 0.5, 0.75, 1.0]

# Logarithmically spaced
log = np.logspace(0, 3, num=4)  # [1, 10, 100, 1000]

# Mesh grids for 2-D evaluation
x = np.linspace(-2, 2, 5)
y = np.linspace(-1, 1, 3)
xx, yy = np.meshgrid(x, y, indexing='xy')
```

### Random Arrays

```python
rng = np.random.default_rng(seed=42)

# Uniform [0, 1)
uniform = rng.random((3, 4))

# Standard normal
normal = rng.standard_normal((3, 4))

# Integers in range [low, high)
integers = rng.integers(low=0, high=100, size=(3, 4))

# Choice from existing array
sample = rng.choice(np.arange(100), size=10, replace=False)
```

---

## Reshaping

### Basic Reshape

```python
arr = np.arange(12)

# Reshape to 2-D (returns a view when possible)
matrix = arr.reshape(3, 4)
assert matrix.base is arr  # confirms it's a view

# Use -1 to infer one dimension
matrix = arr.reshape(3, -1)   # shape (3, 4)
matrix = arr.reshape(-1, 6)   # shape (2, 6)

# Flatten back to 1-D
flat_view = matrix.ravel()    # returns view if possible
flat_copy = matrix.flatten()  # always returns a copy
```

### Adding and Removing Axes

```python
arr = np.array([1, 2, 3])  # shape (3,)

# Add axis — three equivalent ways
col = arr[:, np.newaxis]         # shape (3, 1)
col = np.expand_dims(arr, axis=1) # shape (3, 1)
col = arr.reshape(-1, 1)         # shape (3, 1)

# Remove length-1 axes
squeezed = np.squeeze(col)  # shape (3,)
```

### Transpose and Swapaxes

```python
matrix = np.arange(6).reshape(2, 3)

# Transpose 2-D
transposed = matrix.T  # shape (3, 2)

# For higher-dimensional arrays, specify axis order
cube = np.arange(24).reshape(2, 3, 4)
permuted = np.transpose(cube, axes=(1, 0, 2))  # shape (3, 2, 4)

# Swap two specific axes
swapped = np.swapaxes(cube, 0, 2)  # shape (4, 3, 2)
```

---

## Stacking and Splitting

### Stacking

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Vertical stack (row-wise) — adds rows
v = np.vstack([a, b])  # shape (2, 3)

# Horizontal stack (column-wise) — concatenates along axis 1
h = np.hstack([a, b])  # shape (6,)

# Generic concatenate
c = np.concatenate([a, b], axis=0)  # shape (6,)

# Stack along a new axis
s = np.stack([a, b], axis=0)  # shape (2, 3) — new axis 0
s = np.stack([a, b], axis=1)  # shape (3, 2) — new axis 1

# Column stack (useful for building feature matrices)
col = np.column_stack([a, b])  # shape (3, 2)
```

### Splitting

```python
matrix = np.arange(16).reshape(4, 4)

# Split into equal parts
top, bottom = np.vsplit(matrix, 2)   # two (2, 4) arrays
left, right = np.hsplit(matrix, 2)   # two (4, 2) arrays

# Split at specific indices
parts = np.split(matrix, [1, 3], axis=0)  # rows 0, 1-2, 3
```

---

## Copying and Views

### Views vs Copies

```python
arr = np.arange(10)

# Slicing creates a VIEW (shared memory)
view = arr[2:5]
view[0] = 99
assert arr[2] == 99  # original is modified

# .copy() creates an independent COPY
safe = arr[2:5].copy()
safe[0] = -1
assert arr[2] == 99  # original unchanged

# reshape usually returns a view
reshaped = arr.reshape(2, 5)
assert reshaped.base is arr

# Check if array is a view
print(f"Is view: {reshaped.base is not None}")
```

### When Copies Are Created

| Operation | Returns |
|-----------|---------|
| Basic slicing (`arr[1:5]`) | View |
| `reshape` (contiguous data) | View |
| `ravel` (contiguous data) | View |
| Fancy indexing (`arr[[0,2,4]]`) | Copy |
| Boolean indexing (`arr[mask]`) | Copy |
| `flatten()` | Copy |
| `.copy()` | Copy |
| Arithmetic operations | New array |

