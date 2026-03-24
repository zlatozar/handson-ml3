<!-- You can load it directly into the prompt with `@data-validation` or Copilot will load it based on topics in the description -->
---
name: Data Validation
description: Provides patterns and best practices for input validation, schema checking, and data quality assurance in ML pipelines.
---

# Skill: Data Validation

This skill provides guidelines and code patterns for implementing robust data validation and quality assurance checks within Machine Learning pipelines. Ensuring data quality is crucial for model performance, reproducibility, and preventing unexpected errors in production.

## 1. Input Validation

Input validation focuses on checking the properties of incoming data to ensure it meets basic requirements before further processing. This typically involves type checks, range checks, and format validation.

### 1.1 Basic Python Input Checks

```python
from typing import Any, Dict, List, Union

def validate_numeric_input(value: Any, min_val: Union[int, float], max_val: Union[int, float], name: str) -> Union[int, float]:
    """Validates if a value is numeric and within a specified range."""
    if not isinstance(value, (int, float)):
        raise TypeError(f"Input `{name}` must be a number, got {type(value).__name__}.")
    if not (min_val <= value <= max_val):
        raise ValueError(f"Input `{name}` must be between {min_val} and {max_val}, got {value}.")
    return value

def validate_list_of_strings(data_list: Any, name: str, min_len: int = 1) -> List[str]:
    """Validates if an input is a list of strings with a minimum length."""
    if not isinstance(data_list, list):
        raise TypeError(f"Input `{name}` must be a list, got {type(data_list).__name__}.")
    if len(data_list) < min_len:
        raise ValueError(f"Input `{name}` must contain at least {min_len} elements.")
    for i, item in enumerate(data_list):
        if not isinstance(item, str):
            raise TypeError(f"Element {i} in `{name}` must be a string, got {type(item).__name__}.")
    return data_list

# Example Usage:
# try:
#     age = validate_numeric_input(25, 0, 120, "age")
#     print(f"Age validated: {age}")
#     names = validate_list_of_strings(["Alice", "Bob"], "names")
#     print(f"Names validated: {names}")
# except (TypeError, ValueError) as e:
#     print(f"Validation Error: {e}")
```

## 2. Schema Checking

Schema checking ensures that data conforms to a predefined structure, including column names, data types, and constraints. Libraries like `pandera` are excellent for this with Pandas DataFrames.

### 2.1 Using `pandera` for DataFrame Schema Validation

```python
import pandas as pd
import pandera as pa
from pandera import Column, DataFrameSchema, Check

def validate_dataframe_schema(df: pd.DataFrame) -> pd.DataFrame:
    """Validates a pandas DataFrame against a predefined schema."""
    schema = DataFrameSchema({
        "feature_1": Column(float, Check.in_range(0.0, 1.0), nullable=False),
        "feature_2": Column(int, Check.greater_than_or_equal_to(0)),
        "category": Column(str, Check.isin(["A", "B", "C"])),
        "timestamp": Column(pa.DateTime, nullable=True)
    })
    try:
        validated_df = schema.validate(df)
        print("DataFrame validated successfully!")
        return validated_df
    except pa.errors.SchemaErrors as err:
        print("Schema validation failed:")
        print(err.failure_cases)
        raise

# Example Usage:
# data = {
#     "feature_1": [0.1, 0.5, 0.9],
#     "feature_2": [10, 20, 30],
#     "category": ["A", "B", "C"],
#     "timestamp": pd.to_datetime(["2023-01-01", "2023-01-02", "2023-01-03"])
# }
# df = pd.DataFrame(data)
# try:
#     validated_df = validate_dataframe_schema(df)
# except pa.errors.SchemaErrors:
#     print("Handling schema error...")

# Invalid data example:
# invalid_data = {
#     "feature_1": [0.1, 1.5, 0.9], # 1.5 is out of range
#     "feature_2": [10, -5, 30],
#     "category": ["A", "D", "C"],
#     "timestamp": pd.to_datetime(["2023-01-01", "2023-01-02", "2023-01-03"])
# }
# invalid_df = pd.DataFrame(invalid_data)
# try:
#     validate_dataframe_schema(invalid_df)
# except pa.errors.SchemaErrors:
#     pass # Expected failure
```

## 3. Data Quality Assurance

Beyond basic validation, data quality assurance involves checks for completeness, consistency, uniqueness, and accuracy. This often requires domain-specific rules and statistical analysis.

### 3.1 Common Data Quality Checks

```python
import pandas as pd

def perform_data_quality_checks(df: pd.DataFrame) -> None:
    """Performs a series of data quality checks on a DataFrame."""
    print("\n--- Performing Data Quality Checks ---")

    # Check for missing values
    missing_counts = df.isnull().sum()
    if missing_counts.sum() > 0:
        print("Missing values detected per column:")
        print(missing_counts[missing_counts > 0])
    else:
        print("No missing values detected.")

    # Check for duplicates
    if df.duplicated().any():
        print(f"Duplicate rows detected: {df.duplicated().sum()} rows.")
    else:
        print("No duplicate rows detected.")

    # Check for consistency (example: feature_1 should always be less than feature_2)
    if "feature_1" in df.columns and "feature_2" in df.columns:
        inconsistent_rows = df[df["feature_1"] >= df["feature_2"]]
        if not inconsistent_rows.empty:
            print(f"Inconsistent data where feature_1 >= feature_2: {len(inconsistent_rows)} rows.")
            # print(inconsistent_rows.head())
        else:
            print("Consistency check (feature_1 < feature_2) passed.")

    # Check for outliers (simple example using IQR for a numeric column)
    if "feature_2" in df.columns and pd.api.types.is_numeric_dtype(df["feature_2"]):
        Q1 = df["feature_2"].quantile(0.25)
        Q3 = df["feature_2"].quantile(0.75)
        IQR = Q3 - Q1
        lower_bound = Q1 - 1.5 * IQR
        upper_bound = Q3 + 1.5 * IQR
        outliers = df[(df["feature_2"] < lower_bound) | (df["feature_2"] > upper_bound)]
        if not outliers.empty:
            print(f"Outliers detected in 'feature_2' (IQR method): {len(outliers)} rows.")
            # print(outliers)
        else:
            print("No significant outliers detected in 'feature_2'.")

    print("--- Data Quality Checks Complete ---")

# Example Usage:
# data = {
#     "feature_1": [0.1, 0.5, 0.9, 0.1, None],
#     "feature_2": [10, 20, 30, 10, 1000], # 1000 is an outlier
#     "category": ["A", "B", "C", "A", "B"],
# }
# df_quality = pd.DataFrame(data)
# perform_data_quality_checks(df_quality)
```

## 4. Usage in ML Pipelines

Data validation should be integrated at critical points in the ML pipeline:

- **After Data Ingestion**: Validate raw data immediately after loading from source.
- **After Preprocessing**: Validate processed data before feeding it to the model.
- **Before Model Training/Inference**: Ensure data conforms to the model's expected input schema.

## 5. [TODO] Customization and Extensions

- [ ] Integrate with a data profiling library (e.g., `pandas-profiling`, `great_expectations`).
- [ ] Define project-specific data quality metrics and thresholds.
- [ ] Implement automated alerts for data quality failures.
- [ ] Provide examples for validating data from different sources (e.g., databases, APIs).

---
