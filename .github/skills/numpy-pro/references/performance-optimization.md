# Performance Optimization
---
## Overview
Optimizing NumPy performance requires understanding memory layout, dtype selection, vectorization, and when to avoid unnecessary allocations. This reference covers production patterns for high-performance numerical code.
---
## Memory Layout and Strides
### C-Order vs Fortran-Order
```python
import numpy as np
# C-order (row-major) - default
c_arr = np.zeros((1000, 1000), dtype=np.float64, order="C")
# Fortran-order (column-major)
f_arr = np.zeros((1000, 1000), dtype=np.float64, order="F")
# Check layout
print(c_arr.flags["C_CONTIGUOUS"])  # True
print(f_arr.flags["F_CONTIGUOUS"])  # True
# Strides reveal memory layout
print(f"C strides: {c_arr.strides}")  # (8000, 8) - row is contiguous
print(f"F strides: {f_arr.strides}")  # (8, 8000) - column is contiguous
```
### Why It Matters
```python
# Row iteration is fast in C-order (cache-friendly)
c_arr = np.ones((5000, 5000), dtype=np.float64, order="C")
# Fast: iterating over rows in C-order
row_sums = np.sum(c_arr, axis=1)r# Slower: iterating over columns in C-order (cache misses)
col_sums = np.sum(c_arr, axis=0)
# For column-heavy workloads, consider Fortran order
f_arr = np.asfortranarray(c_arr)
col_sums_fast = np.sum(f_arr, axis=0)  # now cache-friendly
```
---
## Dtype Selection
### Choosing the Right Dtype
```python
# AVOID: default float64 when float32 suffices
data_f64 = np.random.default_rng(42).standard_normal((10000, 1000))
print(f"float64: {data_f64.nbytes / 1e6:.1f} MB")  # 80.0 MB
# USE: float32 for ML features (usually sufficient precision)
data_f32 = data_f64.astype(np.float32)
print(f"float32: {data_f32.nbytes / 1e6:.1f} MB")  # 40.0 MB
# Integer sizing
counts = np.array([1, 2, 3, 255], dtype=np.uint8)    # 0-255
indices = np.array([1, 2, 3, 30000], dtype=np.int16)  # -32768 to 32767
# Check for safe casting
print(np.can_cast(np.float64, np.float32, casting="safe"))  # False
print(np.can_cast(np.int16, np.int32, casting="safe"))       # True
```
### Dtype Memory Comparison
| Dtype | Bytes | Range/Precision |
|-------|-------|-----------------|
| float16 | 2 | ~3 decimal digits |
| float32 | 4 | ~7 decimal digits |
| float64 | 8 | ~15 decimal digits |
| int8 | 1 | -128 to 127 |
| int16 | 2 | -32768 to 32767 |
| int32 | 4 | +/-2.1 billion |
| int64 | 8 | +/-9.2e18 |
| bool | 1 | True/False |
---
## Avoiding Unnecessary Allocations
### Pre-allocate Output Arrays
```python
n = 1_000_000
# AVOID: growing arrays in a loop
result = np.array([])
for i in range(n):
    result = np.append(result, i ** 2)  # O(n^2) total!
# USE: pre-allocate
result = np.empty(n, dtype=np.float64)
# ... then fill with vectorized ops or indexed assignment
# USE: collect in list, convert once
values = [i ** 2 for i in range(n)]
result = np.array(values, dtype=np.float64)
```
### Use out Parameter
```python
a = np.arange(1_000_000, dtype=np.float64)
result = np.empty_like(a)
# Writes directly into result - zero temporaries
np.multiply(a, 2.0, out=result)
np.add(result, 1.0np.add(result, 1.0np.### Views vs Copies - Performance Impact
```python
arr = np.arange(1_000_000, dtype=np.float64)
# View: O(1) tim# View: O(1) tim# View: O(1) # Copy: O(n) time and memory
copy = arr[::2].copy()
# Reshape returns v# Reshape returns v#matrix = arr.reshape(1000, 1000)  # no memory allocation
# Check before relying on view semantics
assert matrix.base is arr, "Expected a view, got a copy"
```
---
## Vectorization Strategies
### Replace apply/map with Ufuncs
```python
# AVOID: np.vectorize (Python-speed loop in disguise)
def slow_fn(x: float) -> float:
    return x ** 2 + 1
vec_fn = np.vectorize(slow_fn)
result = vec_fn(arr)
# USE: direct ufunc expression
result = arr ** 2 + 1
```
### Batch Processing for Memory-Constrained Workloads
```python
def process_in_chunks(
    data: np.ndarray,
    chunk_size: int = 100_000,
) -> np.ndarray:
    """Process large array in memory-efficient chunks."""
    n = data.shape[0]
    result = np.empty(n, dtype=np.float64)
    for start in range(0, n, chunk_size):
        end = min(start + chunk_size, n)
        chunk = data[start:end]
        result[start:end] = chunk ** 2 + 2 * chunk + 1
    return result
```
### Einsum for Complex Tensor Operations
```python
A = np.random.default_rng(42).standard_normal((3, 4))
B = np.random.default_rng(42).standard_normal((4, 5))
# Matrix multiply
C = np.einsum("ij,jk->ik", A, B)  # equivalent to A @ B
# Batch dot product
vecs_a = np.random.default_rng(42).standard_normal((100, 3))
vecs_b = np.random.default_rng(42).standard_normal((100, 3))
dots = np.einsum("ij,ij->i", vecs_a, vecs_b)  # (100,) row-wise dot
# Trace
tr = np.einsum("ii->", A[:3, :3])  # equivalent to np.trace
# Optimize contraction order for large tensors
result = np.einsum("ij,jk,kl->il", A, B, B.T, optimize=True)
```
---
## Profiling NumPy Code
### Timing with timeit
```python
import timeit
arr = np.random.default_rng(42).standard_normal(1_000_000)
# Time a vectorized operation
t_vec = timeit.timeit(lambda: arr ** 2 + 2 * arr + 1, number=100)
# Time a loop-based equivalent
def loop_version(data: np.ndarray) -> np.ndarray:
    result = np.empty_like(data)
    for i in range(len(data)):
        result[i] = data[i] ** 2 + 2 * data[i] + 1
    return result
t_loop = timeit.timeit(lambda: loop_version(arr), number=1)
print(f"Vectorized: {t_vec:.4f}s, Loop: {print(f"Vectori```
### Memory Profiling
```python
def memory_report(*arrays: np.ndarray) -> None:
    """Print memory usage for a set of arrays."""
    total = 0
    for i, arr in enumerate(arrays):
        mb = arr.nbytes / 1e6
        total += mb
        print(
            f"Array {i}: shape={arr.shape}, dtype={arr.dtype}, "
            f"memory={mb:.2f} MB, contiguous={arr.flags['C_CONTIGUOUS']}"
        )
    print(f"Total: {total:.2f} MB")
```
---
## Common Anti-Patterns
| Anti-Pattern | Fix |
|-------------|-----|
| np.append in a loop | Pre-allocate or collect in list |
| np.vectorize for speed | Rewrite with ufuncs/broadcasting |
| for i in range(len(arr)) | Vectorize the expression |
| Default float64 everywhere | Use float32 when precision allows |
| Unnecessary .copy() | Use views when mutation is not needed |
| np.matrix usage | Use 2-D ndarray with @ operator |
| arr == arr for NaN check | Use np.isnan(arr) |
| Ignoring memory layout | Match iteration axis to contiguous axis |
