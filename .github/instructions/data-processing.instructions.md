
---
name: "Data Processing Guidelines"
description: "Guidelines for data loading, preprocessing, augmentation, and validation."
applyTo: "src/**/data/**/*.py"
---

This document provides specific guidelines for Python files located in the `data/` directory, focusing on data loading, preprocessing, augmentation, and validation. These instructions aim to ensure data integrity, reproducibility, and efficient handling of various data types within the Machine Learning project.

## 1. Data Loading and Access

- **Abstract Data Access**: Encapsulate data loading logic within dedicated classes or functions. Avoid direct file path manipulation within model or training scripts.
- **Efficient Loading**: For large datasets, implement lazy loading or memory-mapped files to minimize memory footprint. Use `torch.utils.data.Dataset` and `torch.utils.data.DataLoader` for PyTorch-specific data handling.
- **Data Versioning**: When applicable, use data versioning tools (e.g., DVC) to track changes in datasets and ensure reproducibility of experiments.

## 2. Data Preprocessing and Transformation

- **Stateless Transformations**: Prefer stateless transformations where possible. If stateful transformations (e.g., normalization based on training data statistics) are necessary, ensure the state is saved and loaded for consistency.
- **Pipelines**: Organize preprocessing steps into clear, sequential pipelines (e.g., using `scikit-learn.pipeline.Pipeline` or custom PyTorch `transforms`).
- **Handling Missing Values**: Explicitly define strategies for handling missing data (e.g., imputation, removal). Document the chosen strategy.
- **Feature Scaling**: Apply appropriate feature scaling (e.g., `StandardScaler`, `MinMaxScaler` from scikit-learn) to numerical features.
- **Categorical Encoding**: Use suitable encoding techniques for categorical features (e.g., one-hot encoding, label encoding).

## 3. Data Augmentation

- **Purposeful Augmentation**: Apply data augmentation techniques relevant to the data type and task (e.g., image transformations for computer vision, text perturbations for NLP).
- **Configurable Augmentation**: Make augmentation parameters configurable to allow for experimentation and reproducibility.
- **Order of Operations**: Be mindful of the order of augmentation operations, as it can significantly impact the resulting data.

## 4. Data Validation and Quality Assurance

- **Input Validation**: Implement checks for data types, ranges, and expected formats immediately after loading data.
- **Schema Enforcement**: Define and enforce data schemas (e.g., using `pandera` for Pandas DataFrames) to ensure consistency across different data sources and processing stages.
- **Outlier Detection**: Implement mechanisms to detect and handle outliers, documenting the chosen approach.
- **Data Splitting**: Ensure proper data splitting (training, validation, test sets) to avoid data leakage. Stratified sampling should be used for imbalanced datasets.

## 5. [TODO] Project-Specific Data Considerations

- [ ] Define specific data sources and their access methods.
- [ ] Outline specific preprocessing steps for each dataset.
- [ ] Establish guidelines for handling sensitive data (if any).
- [ ] Detail data storage and retrieval mechanisms.

---
