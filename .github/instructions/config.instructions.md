
---
name: "Configuration File Guidelines"
description: "Guidelines for configuration files, primarily in YAML format."
applyTo: "src/**/configs/**/*.yaml"
---

This document provides specific guidelines for configuration files located in the `configs/` directory, primarily focusing on YAML format. These instructions aim to ensure consistency, readability, and maintainability of all project configurations, facilitating reproducibility and easy experimentation.

## 1. YAML Structure and Syntax

- **File Extension**: All configuration files MUST use the `.yaml` or `.yml` extension.
- **Indentation**: Use 2 spaces for indentation. Tabs are strictly forbidden.
- **Key-Value Pairs**: Use clear, descriptive keys. Values should be appropriate data types (strings, numbers, booleans, lists, dictionaries).
- **Comments**: Use `#` for comments to explain complex configurations or design choices.

## 2. Configuration Organization

- **Hierarchical Structure**: Organize configurations hierarchically to reflect logical groupings (e.g., `model`, `data`, `training`, `optimizer`).
- **Modularity**: Break down large configurations into smaller, reusable files when appropriate. Use YAML anchors and aliases (`&`, `*`) for common blocks of configuration.
- **Environment-Specific Overrides**: Provide mechanisms for environment-specific overrides (e.g., `base.yaml`, `dev.yaml`, `prod.yaml`) to manage different settings for development, testing, and production.

## 3. Content and Best Practices

- **Single Source of Truth**: Configuration files should be the single source of truth for all hyperparameters, paths, and settings. Avoid hardcoding values in code.
- **Descriptive Naming**: Use clear and consistent naming conventions for all configuration parameters.
- **Data Types**: Ensure values are of the correct data type. For example, learning rates should be floats, batch sizes integers.
- **Paths**: Use relative paths where possible, or define a base path variable that can be easily changed.
- **Avoid Sensitive Information**: NEVER store sensitive information (e.g., API keys, passwords) directly in configuration files. Use environment variables or a secure secrets management system.

## 4. Example Configuration Structure

```yaml
# configs/experiment_a.yaml

experiment_name: "MyFirstExperiment"
seed: 42

model:
  name: "ResNet18"
  num_classes: 10
  pretrained: true

data:
  dataset_name: "CIFAR10"
  batch_size: 64
  num_workers: 4
  train_path: "data/processed/train.pt"
  val_path: "data/processed/val.pt"

training:
  epochs: 50
  learning_rate: 0.001
  optimizer:
    name: "Adam"
    weight_decay: 0.0001
  scheduler:
    name: "ReduceLROnPlateau"
    mode: "min"
    factor: 0.1
    patience: 5
  checkpoint_dir: "checkpoints/experiment_a"
  log_interval: 100

logging:
  tool: "tensorboard"
  project: "image_classification"
```

## 5. [TODO] Project-Specific Configuration Details

- [ ] Define a standard set of required configuration parameters for all experiments.
- [ ] Establish a versioning strategy for configuration files.
- [ ] Guidelines for command-line argument parsing and overriding config values.

---
