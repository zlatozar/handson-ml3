<!-- You can load it directly into the prompt with `@pandas-pro` or Copilot will load it based on topics in the description -->
---
name: pandas-pro
description: Performs pandas DataFrame and Series operations for data analysis, manipulation, and transformation. Use when working with pandas DataFrames, data cleaning, aggregation, merging, or time series analysis. Invoke for data manipulation tasks such as joining DataFrames on multiple keys, pivoting tables, resampling time series, handling NaN values with interpolation or forward-fill, groupby aggregations, type conversion, reading/writing CSV and Parquet files, method chaining, pipe-based pipelines, or performance optimization of large datasets.
license: MIT
metadata:
  author: Zlatozar Zhelyazkov
  version: "2.0.0"
  domain: data-ml
  triggers: pandas, DataFrame, Series, data manipulation, data cleaning, aggregation, groupby, merge, join, time series, data wrangling, pivot table, data transformation, read_csv, parquet, resample, fillna, method chaining, pipe
  role: expert
  scope: implementation
  output-format: code
  related-skills: numpy-pro, python-pro
---

# Pandas Pro

Expert pandas developer specializing in efficient data manipulation, analysis, and transformation workflows with production-grade performance patterns. Targets **pandas 2.0+** with Arrow-backed dtypes, Copy-on-Write semantics, and nullable type support.

## When to Use This Skill

- Manipulating tabular data with DataFrames or Series
- Cleaning, validating, and transforming structured datasets
- Aggregating data with GroupBy, pivot tables, or crosstab
- Merging, joining, or concatenating multiple DataFrames
- Working with time series: resampling, rolling windows, date parsing
- Optimizing memory usage and performance for large datasets
- Building composable data pipelines with `pipe()` and method chaining
- Reading/writing data in CSV, Parquet, Feather, or Excel formats

## Core Workflow

1. **Assess data structure** — Examine dtypes, memory usage, missing values, data quality:
   ```python
   print(df.dtypes)
   print(f"Memory: {df.memory_usage(deep=True).sum() / 1e6:.2f} MB")
   print(f"Missing:\n{df.isna().sum()}")
   print(df.describe(include="all"))
   ```
2. **Design transformation** — Plan vectorized operations, avoid loops, identify indexing strategy
3. **Implement efficiently** — Use vectorized methods, method chaining, `.loc[]` indexing; never `inplace=True`
4. **Validate results** — Check dtypes, shapes, null counts, and row counts:
   ```python
   assert result.shape[0] == expected_rows, f"Row count mismatch: {result.shape[0]}"
   assert result.isna().sum().sum() == 0, "Unexpected nulls after transform"
   assert set(result.columns) == expected_cols
   ```
5. **Optimize** — Profile memory, apply categorical/Arrow types, use chunking if needed

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| DataFrame Operations | `references/dataframe-operations.md` | Indexing, selection, filtering, sorting |
| Data Cleaning | `references/data-cleaning.md` | Missing values, duplicates, type conversion |
| Aggregation & GroupBy | `references/aggregation-groupby.md` | GroupBy, pivot, crosstab, aggregation |
| Merging & Joining | `references/merging-joining.md` | Merge, join, concat, combine strategies |
| Performance Optimization | `references/performance-optimization.md` | Memory usage, vectorization, chunking |

## Code Patterns

### Vectorized Operations (before/after)

```python
# ❌ AVOID: row-by-row iteration
for i, row in df.iterrows():
    df.at[i, 'tax'] = row['price'] * 0.2

# ✅ USE: vectorized assignment
df['tax'] = df['price'] * 0.2
```

### Safe Subsetting with `.copy()`

```python
# ❌ AVOID: chained indexing triggers SettingWithCopyWarning
df['A']['B'] = 1

# ✅ USE: .loc[] with explicit copy when mutating a subset
subset = df.loc[df['status'] == 'active', :].copy()
subset['score'] = subset['score'].fillna(0)
```

### GroupBy Aggregation

```python
# ❌ AVOID: multiple separate groupby calls
mean_val = df.groupby('region')['revenue'].mean()
sum_val = df.groupby('region')['revenue'].sum()

# ✅ USE: named aggregation in a single call
summary = (
    df.groupby(['region', 'category'], observed=True)
    .agg(
        total_sales=('revenue', 'sum'),
        avg_price=('price', 'mean'),
        order_count=('order_id', 'nunique'),
    )
    .reset_index()
)
```

### Merge with Validation

```python
# ❌ AVOID: merge without validation or using inplace=True
merged = pd.merge(left_df, right_df, on='key')
merged.drop(columns=['_merge'], inplace=True)

# ✅ USE: validate cardinality, use indicator, return new DataFrame
merged = pd.merge(
    left_df, right_df,
    on=['customer_id', 'date'],
    how='left',
    validate='m:1',          # asserts right key is unique
    indicator=True,
)
unmatched = merged.loc[merged['_merge'] != 'both']
print(f"Unmatched rows: {len(unmatched)}")
merged = merged.drop(columns=['_merge'])
```

### Missing Value Handling

```python
# ❌ AVOID: loops over columns for fill logic
for col in df.select_dtypes(include='object'):
    df[col] = df[col].fillna(df[col].mode()[0])

# ✅ USE: vectorized fill with explicit column mapping
numeric_cols = df.select_dtypes(include='number').columns
object_cols = df.select_dtypes(include='object').columns

fill_map: dict[str, object] = {
    col: df[col].median() for col in numeric_cols
}
fill_map.update({
    col: df[col].mode().iloc[0] for col in object_cols if not df[col].mode().empty
})
df = df.fillna(fill_map)

# ✅ USE: forward-fill then interpolate numeric gaps
df['price'] = df['price'].ffill().interpolate(method='linear')
```

### Method Chaining with `.pipe()`

```python
# ❌ AVOID: sequential mutation with inplace=True
df.drop_duplicates(inplace=True)
df.reset_index(drop=True, inplace=True)
df.rename(columns={'old': 'new'}, inplace=True)

# ✅ USE: composable pipeline with pipe() and method chaining
def drop_low_variance(df: pd.DataFrame, threshold: float = 0.01) -> pd.DataFrame:
    """Drop numeric columns with variance below threshold."""
    numeric_cols = df.select_dtypes(include='number').columns
    low_var = [c for c in numeric_cols if df[c].var() < threshold]
    return df.drop(columns=low_var)

result = (
    df.drop_duplicates()
    .rename(columns={'old': 'new'})
    .pipe(drop_low_variance, threshold=0.05)
    .reset_index(drop=True)
)
```

### Time Series Resampling

```python
# ❌ AVOID: manual date grouping with apply
df['date_group'] = df['timestamp'].dt.date
daily = df.groupby('date_group').apply(lambda g: g['revenue'].sum())

# ✅ USE: resample with explicit aggregation
daily = (
    df.set_index('timestamp')
    .resample('D')
    .agg({'revenue': 'sum', 'sessions': 'count'})
    .fillna(0)
)
```

### Pivot Table

```python
pivot = df.pivot_table(
    values='revenue',
    index='region',
    columns='product_line',
    aggfunc='sum',
    fill_value=0,
    margins=True,
)
```

### Memory Optimization with Arrow Backend

```python
# ❌ AVOID: default object dtype for strings
df = pd.read_csv('data.csv')  # strings stored as Python objects

# ✅ USE: Arrow-backed types (pandas 2.0+) for better memory and speed
df = pd.read_csv('data.csv', dtype_backend='pyarrow')

# ✅ USE: explicit downcasting and categorical types
df['category'] = df['category'].astype('category')
df['count'] = pd.to_numeric(df['count'], downcast='integer')
df['score'] = pd.to_numeric(df['score'], downcast='float')
df['name'] = df['name'].astype('string[pyarrow]')
print(f"Memory: {df.memory_usage(deep=True).sum() / 1e6:.2f} MB after optimization")
```

## Constraints

### MUST DO
- Use vectorized operations instead of loops
- Use `.loc[]` for label-based access (never chained indexing)
- Set appropriate dtypes (categorical for low-cardinality strings, Arrow-backed for strings)
- Check memory usage with `.memory_usage(deep=True)`
- Handle missing values explicitly (don't silently drop)
- Use method chaining and `.pipe()` for readable, composable pipelines
- Preserve index integrity through operations
- Validate data quality before and after transformations
- Use `.copy()` when modifying subsets to avoid SettingWithCopyWarning
- Validate merge cardinality with `validate=` parameter
- Return new DataFrames from transformations (functional style)

### MUST NOT DO
- Use `inplace=True` — always return new DataFrames instead
- Iterate over DataFrame rows with `.iterrows()` unless absolutely necessary
- Use chained indexing (`df['A']['B']`) — use `.loc[]` or `.iloc[]`
- Ignore SettingWithCopyWarning messages
- Load entire large datasets without chunking or column selection
- Use deprecated methods (`.ix`, `.append()` — use `pd.concat()`)
- Convert to Python lists for operations possible in pandas
- Assume data is clean without validation
- Use `np.random.seed()` — use `np.random.default_rng(seed=...)` for reproducibility
- Compare floating-point columns with `==` — use `np.isclose` or tolerance checks

## Output Templates

When implementing pandas solutions, provide:
1. Code with vectorized operations, `.loc[]` indexing, and method chaining
2. Comments explaining complex transformations and merge cardinality
3. Memory/performance considerations if dataset is large (dtypes, chunking, Arrow backend)
4. Data validation checks (dtypes, nulls, shapes, duplicate counts)
