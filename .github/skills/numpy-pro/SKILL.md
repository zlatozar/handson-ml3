<!-- You can load it directly into the prompt with `@numpy-pro` or Copilot will load it based on topics in the description -->
---
name: numpy-pro
description: Performs NumPy array operations for numerical computing, scientific data processing, and linear algebra. Use when working with NumPy ndarrays, array creation, reshaping, broadcasting, vectorized math, random number generation, or memory-efficient numerical computation. Invoke for tasks such as element-wise operations, matrix multiplication, fancy indexing, boolean masking, FFT, statistical aggregations, structured arrays, or optimizing array memory layout.
license: MIT
metadata:
  author: Zlatozar Zhelyazkov
  version: "1.0.0"
  domain: data-ml
  triggers: numpy, ndarray, array, vectorization, broadcasting, linear algebra, matrix, numerical computing, ufunc, random, scientific computing, array manipulation
  role: expert
  scope: implementation
  output-format: code
  related-skills: pandas-pro, python-pro
---

# NumPy Pro

Expert NumPy developer specializing in efficient numerical computing, vectorized operations, and production-grade array workflows.

## Core Workflow

1. **Assess data structure** — Examine shape, dtype, memory layout, and strides:
   ```python
   print(f"Shape: {arr.shape}, Dtype: {arr.dtype}")
   print(f"Strides: {arr.strides}, C-contiguous: {arr.flags['C_CONTIGUOUS']}")
   print(f"Memory: {arr.nbytes / 1e6:.2f} MB")
   print(f"NaN count: {np.isnan(arr).sum()}")
   ```
2. **Design computation** — Plan vectorized operations, identify broadcasting rules, choose appropriate dtype
3. **Implement efficiently** — Use ufuncs, broadcasting, and fancy indexing; avoid Python loops
4. **Validate results** — Check shapes, dtypes, value ranges, and NaN propagation:
   ```python
   assert result.shape == expected_shape, f"Shape mismatch: {result.shape}"
   assert result.dtype == expected_dtype, f"Dtype mismatch: {result.dtype}"
   assert not np.isnan(result).any(), "Unexpected NaN in result"
   np.testing.assert_allclose(result, expected, rtol=1e-7)
   ```
5. **Optimize** — Profile memory layout, prefer views over copies, use in-place operations where safe

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Array Creation & Manipulation | `references/array-creation-manipulation.md` | Creating, reshaping, stacking, splitting arrays |
| Indexing & Slicing | `references/indexing-slicing.md` | Fancy indexing, boolean masks, views vs copies |
| Broadcasting & Vectorization | `references/broadcasting-vectorization.md` | Element-wise ops, ufuncs, broadcasting rules |
| Linear Algebra | `references/linear-algebra.md` | Matrix ops, decompositions, solving systems |
| Performance Optimization | `references/performance-optimization.md` | Memory layout, dtype selection, profiling |

## Code Patterns

### Vectorized Operations (before/after)

```python
# ❌ AVOID: Python loop over elements
result = []
for x in data:
    result.append(x ** 2 + 2 * x + 1)
result = np.array(result)

# ✅ USE: vectorized expression
result = data ** 2 + 2 * data + 1
```

### Broadcasting (before/after)

```python
# ❌ AVOID: manual tiling to match shapes
col_means = np.mean(matrix, axis=0)
tiled = np.tile(col_means, (matrix.shape[0], 1))
centered = matrix - tiled

# ✅ USE: broadcasting handles shape alignment automatically
col_means = np.mean(matrix, axis=0)  # shape (n_cols,)
centered = matrix - col_means        # broadcasts (n_rows, n_cols) - (n_cols,)
```

### Boolean Masking

```python
# ❌ AVOID: filtering with a loop
filtered = []
for val in arr:
    if val > 0:
        filtered.append(val)
filtered = np.array(filtered)

# ✅ USE: boolean mask
filtered = arr[arr > 0]

# ✅ USE: np.where for conditional assignment
result = np.where(arr > 0, arr, 0)
```

### Fancy Indexing

```python
# Select specific rows and columns
indices = np.array([0, 3, 7])
subset = matrix[indices, :]

# Advanced: np.ix_ for cross-indexing
row_idx = np.array([0, 2])
col_idx = np.array([1, 3])
block = matrix[np.ix_(row_idx, col_idx)]
```

### Aggregation with Axis Control

```python
# ❌ AVOID: ambiguous aggregation
total = np.sum(matrix)  # flattens to scalar — often unintentional

# ✅ USE: explicit axis
col_sums = np.sum(matrix, axis=0)   # sum down rows → one value per column
row_sums = np.sum(matrix, axis=1)   # sum across columns → one value per row
```

### Safe Random Number Generation

```python
# ❌ AVOID: legacy global random state
np.random.seed(42)
data = np.random.randn(100)

# ✅ USE: Generator API (NumPy 1.17+)
rng = np.random.default_rng(seed=42)
data = rng.standard_normal(100)
samples = rng.choice(data, size=10, replace=False)
```

### Memory-Efficient Operations

```python
# ❌ AVOID: creating unnecessary intermediate arrays
result = np.sqrt(np.sum(np.square(matrix), axis=1))

# ✅ USE: in-place and out parameter to reduce allocations
squared = np.square(matrix)              # intermediate
row_sums = np.sum(squared, axis=1)       # intermediate
result = np.sqrt(row_sums, out=row_sums) # in-place sqrt

# ✅ USE: np.linalg.norm directly
result = np.linalg.norm(matrix, axis=1)
```

### Structured Arrays

```python
# Define a structured dtype for heterogeneous tabular data
dt = np.dtype([
    ('name', 'U20'),
    ('age', 'i4'),
    ('salary', 'f8'),
])
records = np.array([
    ('Alice', 30, 75000.0),
    ('Bob', 25, 62000.0),
], dtype=dt)

# Access fields by name
print(records['salary'].mean())
```

## Constraints

### MUST DO
- Use vectorized operations and ufuncs instead of Python loops
- Specify `dtype` explicitly when creating arrays for numerical stability
- Use the `np.random.Generator` API (not legacy `np.random.seed`)
- Check array shapes before operations to catch broadcasting errors early
- Use `np.testing.assert_allclose` for floating-point comparisons in tests
- Prefer views over copies when mutation is not needed
- Document axis semantics in comments for multi-dimensional operations
- Set random seeds via `np.random.default_rng(seed=...)` for reproducibility

### MUST NOT DO
- Iterate over array elements with Python `for` loops
- Use `np.matrix` — it is deprecated; use 2-D `ndarray` with `@` operator
- Ignore dtype overflow (e.g., int8 overflow on large values)
- Rely on implicit broadcasting without verifying shape compatibility
- Use `np.random.seed()` global state in production code
- Mutate arrays that are views of other arrays without explicit `.copy()`
- Compare floating-point arrays with `==`; use `np.isclose` or `np.allclose`
- Use `np.append` in a loop — pre-allocate or collect in a list first

## Output Templates

When implementing NumPy solutions, provide:
1. Code with vectorized operations and explicit dtype/axis parameters
2. Comments explaining broadcasting rules and shape transformations
3. Memory considerations (views vs copies, dtype sizing)
4. Validation checks (shapes, dtypes, value ranges, NaN handling)

