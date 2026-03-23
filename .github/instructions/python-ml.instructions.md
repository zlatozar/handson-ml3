
---
# Python ML Coding Guidelines

This document provides specific coding guidelines for general Python files within the Machine Learning project, particularly those not covered by more specific path-based instructions. These guidelines build upon the global instructions and emphasize best practices for Python development in an ML context.

## 1. File Structure and Imports

- **Standard Library First**: Imports should be grouped in the following order:
  1. Standard library imports (e.g., `os`, `sys`, `math`)
  2. Third-party library imports (e.g., `torch`, `numpy`, `pandas`)
  3. Local application/library-specific imports
- **Absolute Imports**: Prefer absolute imports over relative imports for clarity and to avoid issues with complex package structures.
- **Wildcard Imports**: Avoid wildcard imports (`from module import *`) as they can lead to name collisions and make code harder to read and debug.

## 2. Type Hinting

- **Mandatory Type Hints**: All function signatures, class attributes, and complex variable assignments MUST include type hints ([PEP 484](https://www.python.org/dev/peps/pep-0484/)). This improves code readability, enables static analysis, and reduces bugs.
  ```python
  from typing import List, Dict, Any

  def process_data(data: List[Dict[str, Any]]) -> Dict[str, float]:
      # ... implementation ...
      return result
  ```
- **Use `typing` Module**: Leverage the `typing` module for more complex types (e.g., `List`, `Dict`, `Optional`, `Union`, `Callable`).

## 3. Error Handling and Validation

- **Specific Exceptions**: Raise specific exceptions rather than general `Exception` to provide clearer error messages and allow for more precise error handling by callers.
- **Input Validation**: Validate function inputs at the beginning of functions, especially for public APIs, to ensure correct usage and prevent unexpected behavior.

## 4. Performance Considerations

- **NumPy/PyTorch Vectorization**: Whenever possible, use vectorized operations provided by NumPy or PyTorch instead of explicit Python loops for performance-critical sections.
- **Avoid Unnecessary Object Creation**: Be mindful of creating large temporary objects in loops or frequently called functions.

## 5. Logging

- **Consistent Logging**: Use the `logging` module consistently throughout the codebase. Define a logger for each module.
  ```python
  import logging

  logger = logging.getLogger(__name__)

  def my_function():
      logger.info("Starting my_function")
      # ...
      logger.debug("Intermediate step completed")
  ```
- **Informative Messages**: Log messages should be clear, concise, and provide enough context to diagnose issues without needing to inspect the code.

## 6. [TODO] Project-Specific Python Conventions

- [ ] Define specific utility functions to be used across the project.
- [ ] Establish conventions for handling configuration parameters within Python scripts.
- [ ] Guidelines for using abstract base classes or interfaces.

---
