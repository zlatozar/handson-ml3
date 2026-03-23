
---
# Model Architecture Guidelines

This document provides specific guidelines for Python files located in the `models/` directory, focusing on defining and implementing neural network architectures using PyTorch. These instructions aim to promote modularity, readability, and efficient design of machine learning models.

## 1. Module and Class Structure

- **`torch.nn.Module` Inheritance**: All neural network components (layers, blocks, full models) MUST inherit from `torch.nn.Module`. This provides essential functionality like parameter tracking and device management.
- **`__init__` Method**: The `__init__` method should define all sub-modules and parameters. Avoid performing heavy computations or defining non-`nn.Module` attributes that change during `forward` pass here.
- **`forward` Method**: The `forward` method defines the computational graph of the model. It should only contain operations on tensors and calls to defined sub-modules.

## 2. Layer Design and Reusability

- **Modular Layers**: Break down complex architectures into smaller, reusable `nn.Module` blocks (e.g., `ConvBlock`, `ResidualBlock`). This improves readability and facilitates experimentation.
- **Parameter Initialization**: Explicitly initialize weights and biases of layers where custom initialization is beneficial (e.g., Kaiming, Xavier). Otherwise, rely on PyTorch's default initialization.
- **Activation Functions**: Choose appropriate activation functions based on the layer and task. Use `torch.nn.ReLU`, `torch.nn.GELU`, etc., as modules rather than functional calls when they have learnable parameters or need to be part of the `state_dict`.

## 3. Input and Output Handling

- **Clear Input/Output Shapes**: Document the expected input and output tensor shapes for each `nn.Module` or model. This is crucial for debugging and understanding data flow.
- **Device Agnostic**: Ensure models are designed to be device-agnostic. Parameters and buffers should be moved to the appropriate device using `.to(device)`.

## 4. Model Complexity and Efficiency

- **Computational Graph Optimization**: Be mindful of operations that can break the computational graph or lead to inefficient memory usage. Use `torch.jit.script` or `torch.compile` for potential performance gains.
- **Regularization**: Incorporate regularization techniques (e.g., Dropout, BatchNorm, Weight Decay) as needed to prevent overfitting. Place them appropriately within the architecture.

## 5. [TODO] Project-Specific Model Conventions

- [ ] Define a base model class or interface for all models in the project.
- [ ] Establish conventions for model versioning and naming.
- [ ] Guidelines for integrating pre-trained models (e.g., from HuggingFace).
- [ ] Specific requirements for model output formats (e.g., logits, probabilities).

---
