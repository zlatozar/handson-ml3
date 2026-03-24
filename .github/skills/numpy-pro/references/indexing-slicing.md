# Indexing and Slicing

---

## Overview

NumPy provides powerful indexing beyond Python lists: basic slicing, fancy (advanced) indexing, and boolean masking. Understanding when each returns a view vs a copy is essential for correctness and performance.

---

## Basic Slicing

### 1-D Slicing

```python
import numpy as np

arr = np.arange(10)  # [0, 1, 2, ..., 9]

# Standard slice — returns a VIEW
subset = arr[2:7]       # [2, 3, 4, 5, 6]
stepped = arr[::2]      # [0, 2, 4, 6, 8]
reversed_ = arr[::-1]   # [9, 8, 7, ..., 0]

# Negative indexing
last_three = arr[-3:]   # [7, 8, 9]
```

### Multi-Dimensional Slicing

```python
matrix = np.arange(20).reshape(4, 5)

# Row slice
rows = matrix[1:3, :]   # rows 1-2, all columns

# Column slice
cols = matrix[:, 2:4]   # all rows, columns 2-3

# Single element
val = matrix[2, 3]      # scalar at row 2, col 3

# Mixed: row slice + column index → reduces dimension
col_vector = matrix[1:3, 2]  # shape (2,) not (2, 1)

# Preserve dimensions with slice of length 1
col_2d = matrix[1:3, 2:3]   # shape (2, 1)
```

### Slice Assignment

```python
arr = np.zeros(10)

# Assign scalar to slice (broadcasts)
arr[3:6] = 5.0  # [0, 0, 0, 5, 5, 5, 0, 0, 0, 0]

# Assign array to slice (shapes must match)
arr[0:3] = np.array([10, 20, 30])
```

---

## Fancy (Advanced) Indexing

Fancy indexing uses integer arrays or lists to select arbitrary elements. **Always returns a copy.**

### 1-D Fancy Indexing

```python
arr = np.array([10, 20, 30, 40, 50])

# Select specific indices
indices = np.array([0, 3, 4])
selected = arr[indices]  # [10, 40, 50] — this is a COPY

# Duplicate indices are allowed
repeated = arr[np.array([0, 0, 1, 1])]  # [10, 10, 20, 20]
```

### 2-D Fancy Indexing

```python
matrix = np.arange(12).reshape(3, 4)

# Select specific rows
rows = matrix[np.array([0, 2]), :]  # rows 0 and 2

# Select specific (row, col) pairs — returns 1-D
elements = matrix[np.array([0, 1, 2]), np.array([1, 2, 3])]
# → [matrix[0,1], matrix[1,2], matrix[2,3]]

# Cross-index with np.ix_ — returns sub-matrix
row_idx = np.array([0, 2])
col_idx = np.array([1, 3])
block = matrix[np.ix_(row_idx, col_idx)]  # shape (2, 2)
```

### Fancy Index Assignment

```python
arr = np.zeros(5)
arr[np.array([1, 3])] = np.array([99, 88])
# [0, 99, 0, 88, 0]

# Caution: repeated indices with assignment — last value wins
arr = np.zeros(5)
arr[np.array([1, 1, 1])] = np.array([10, 20, 30])
# arr[1] == 30 (not accumulated)

# Use np.add.at for accumulation
arr = np.zeros(5)
np.add.at(arr, np.array([1, 1, 1]), np.array([10, 20, 30]))
# arr[1] == 60
```

---

## Boolean Masking

Boolean indexing selects elements where a condition is `True`. **Always returns a copy.**

### Basic Masks

```python
arr = np.array([1, -2, 3, -4, 5])

# Single condition
positives = arr[arr > 0]  # [1, 3, 5]

# Combined conditions (use & | ~ with parentheses)
mask = (arr > 0) & (arr < 5)
filtered = arr[mask]  # [1, 3]

# Negation
non_negative = arr[~(arr < 0)]  # [1, 3, 5]
```

### Masks with 2-D Arrays

```python
matrix = np.arange(12).reshape(3, 4)

# Element-wise mask → returns flattened 1-D result
big = matrix[matrix > 6]  # [7, 8, 9, 10, 11]

# Row-wise mask (select entire rows)
row_sums = matrix.sum(axis=1)
high_sum_rows = matrix[row_sums > 10, :]  # rows whose sum > 10

# Column-wise mask
col_mask = np.array([True, False, True, False])
selected_cols = matrix[:, col_mask]  # columns 0 and 2
```

### np.where for Conditional Selection

```python
arr = np.array([1, -2, 3, -4, 5])

# Returns indices where condition is True
indices = np.where(arr > 0)  # (array([0, 2, 4]),)

# Conditional assignment: where(cond, true_val, false_val)
result = np.where(arr > 0, arr, 0)  # [1, 0, 3, 0, 5]

# Multi-condition with np.select
conditions = [arr < 0, arr == 0, arr > 0]
choices = [-1, 0, 1]
signs = np.select(conditions, choices, default=0)
```

---

## Combining Indexing Styles

```python
matrix = np.arange(20).reshape(4, 5)

# Boolean rows + integer column
mask = matrix[:, 0] > 5  # rows where first column > 5
subset = matrix[mask, 2]  # column 2 of those rows

# Fancy index rows + slice columns
rows = np.array([0, 3])
subset = matrix[rows, 1:4]  # specific rows, column slice

# Boolean mask + fancy column index
row_mask = np.array([True, False, True, False])
col_idx = np.array([0, 3])
block = matrix[np.ix_(row_mask, col_idx)]  # cross-indexed sub-matrix
```

---

## Views vs Copies Summary

| Indexing Style | Returns | Modifies Original? |
|---------------|---------|---------------------|
| Basic slice (`arr[1:5]`) | View | Yes |
| Single element (`arr[2]`) | Scalar | N/A |
| Fancy index (`arr[[0,2]]`) | Copy | No |
| Boolean mask (`arr[mask]`) | Copy | No |
| `np.ix_` cross-index | Copy | No |
| Slice assignment (`arr[1:3] = x`) | In-place | Yes |

