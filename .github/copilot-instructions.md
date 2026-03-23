
---
# Global Copilot Instructions for ML Engineering Project

This document outlines the global coding standards, project conventions, and mandatory practices for this Machine Learning engineering project. These guidelines are designed to ensure **reproducibility**, **scalability**, **production readiness**, **research velocity**, and **readability** across all project components.

## 1. Project Context and Tech Stack

**Domain**: Machine Learning / Deep Learning
**Primary Framework**: PyTorch
**Secondary Libraries**: scikit-learn, HuggingFace, keras, numpy, panda, scipy, matplotlib
**Python Version**: 3.10+
**Team Size**: Solo (with future collaboration in mind)

## 2. High-Level Coding Standards for Python/ML

### 2.1 Readability and Maintainability
- **PEP 8 Compliance**: Adhere strictly to [PEP 8](https://www.python.org/dev/peps/pep-0008/) for code formatting. Use linters (e.g., `flake8`, `black`) to enforce this automatically.
- **Clear Naming Conventions**: Use descriptive names for variables, functions, classes, and modules. Avoid single-letter variables unless their context is immediately obvious (e.g., `i` in a loop).
- **Docstrings**: All functions, classes, and modules MUST have comprehensive docstrings following [PEP 257](https://www.python.org/dev/peps/pep-0257/) (e.g., Google style or NumPy style). Explain parameters, return values, and any exceptions raised.
- **Comments**: Use comments to explain complex logic, design decisions, or non-obvious code sections. Focus on *why* something is done, not just *what* it does.

### 2.2 Modularity and Reusability
- **Single Responsibility Principle (SRP)**: Each function, class, or module should have one clear, well-defined responsibility.
- **Avoid Duplication (DRY)**: Refactor common logic into reusable functions or classes.
- **Clear Interfaces**: Design modules and classes with well-defined public interfaces. Minimize reliance on internal implementation details.

## 3. Project-Specific Tech Stack and Conventions

### 3.1 PyTorch Best Practices
- **Device Agnostic Code**: Write code that can seamlessly run on both CPU and GPU. Use `torch.device("cuda" if torch.cuda.is_available() else "cpu")` and move tensors/models to the device explicitly.
- **DataLoaders**: Always use `torch.utils.data.DataLoader` for efficient data loading and batching.
- **Model Saving/Loading**: Use `torch.save(model.state_dict(), PATH)` and `model.load_state_dict(torch.load(PATH))` for saving and loading model weights. Save full models only when necessary.
- **Mixed Precision Training**: Consider `torch.cuda.amp` for faster training and reduced memory usage on compatible GPUs.

### 3.2 Data Handling with scikit-learn, numpy, panda, scipy
- **Pandas for Tabular Data**: Use Pandas DataFrames for structured data manipulation and analysis.
- **NumPy for Numerical Operations**: Leverage NumPy for efficient array operations.
- **Scikit-learn for Preprocessing/Evaluation**: Utilize scikit-learn for data preprocessing (scaling, encoding) and model evaluation metrics.

### 3.3 HuggingFace Integration
- **Transformers Library**: When working with pre-trained models, use the HuggingFace `transformers` library for easy access and fine-tuning.
- **Tokenizers**: Always use the tokenizer associated with the pre-trained model.

## 4. Mandatory Practices

### 4.1 Reproducibility
- **Random Seeds**: ALWAYS set random seeds for `numpy`, `torch`, and Python's `random` module at the beginning of any script or notebook to ensure reproducibility. This includes worker seeds for DataLoaders.
  ```python
  import torch
  import numpy as np
  import random

  def set_seed(seed):
      torch.manual_seed(seed)
      torch.cuda.manual_seed_all(seed) # if using multi-GPU
      np.random.seed(seed)
      random.seed(seed)
      torch.backends.cudnn.deterministic = True
      torch.backends.cudnn.benchmark = False

  # Example usage:
  # set_seed(42)
  ```
- **Environment Management**: Use `conda` or `pipenv` with `requirements.txt` or `environment.yml` to manage dependencies. Pin exact versions of libraries.

### 4.2 Logging and Experiment Tracking
- **Structured Logging**: Use Python's built-in `logging` module for all informational, warning, and error messages. Avoid `print()` for anything beyond quick debugging.
- **Experiment Tracking**: Integrate with experiment tracking tools (e.g., [MLflow](https://mlflow.org/), [Weights & Biases](https://wandb.ai/), [TensorBoard](https://www.tensorflow.org/tensorboard/)). Log metrics, hyperparameters, model artifacts, and code versions.

### 4.3 Error Handling
- **Graceful Degradation**: Implement `try-except` blocks for operations that might fail (e.g., file I/O, API calls, network requests). Provide informative error messages.
- **Assertions**: Use `assert` statements for internal consistency checks and assumptions that should always hold true.

## 5. Project File Structure

This project follows a standard structure to organize code and resources:

```
.github/
├── agents/
│   └── ml-engineer.agent.md
├── copilot-instructions.md
└── instructions/
    ├── config.instructions.md
    ├── data-processing.instructions.md
    ├── model-architecture.instructions.md
    ├── python-ml.instructions.md
    └── training.instructions.md
└── skills/
    ├── data-validation/
    │   └── SKILL.md
    └── pytorch-tensorflow-training/
        └── SKILL.md
src/
├── data/                 # Data loading, preprocessing, and augmentation scripts
├── models/               # Model architectures (e.g., model.py, layers.py)
├── training/             # Training loops, evaluation logic, and utility functions
└── utils/                # General utility functions (e.g., metrics.py, visualizations.py)
configs/                  # Configuration files (e.g., YAML, JSON) for experiments, model parameters, and training settings
notebooks/                # Jupyter notebooks for exploration, prototyping, and analysis
scripts/                  # Standalone scripts for tasks like data generation, model deployment, or hyperparameter sweeps
tests/                    # Unit and integration tests
docs/                     # Project documentation
data/                     # Raw and processed data (often managed outside Git for large datasets)
checkpoints/              # Saved model weights and training checkpoints
.gitignore                # Specifies intentionally untracked files to ignore
requirements.txt          # Project dependencies
```

## 6. Additional Considerations

### 6.1 Git Version Control
- **Atomic Commits**: Make small, focused commits that address a single logical change.
- **Descriptive Commit Messages**: Follow conventional commit guidelines (e.g., `feat: add new model architecture`, `fix: resolve data loading bug`).
- **Branching Strategy**: Use a clear branching strategy (e.g., Git Flow, GitHub Flow). For solo projects, a `main` branch for stable code and feature branches for development is sufficient.
- **`.gitignore`**: Properly configure `.gitignore` to exclude large data files, checkpoints, environment files, and other non-essential files.

### 6.2 Jupyter Notebook Interaction
- **Exploration vs. Production**: Use notebooks for rapid prototyping, data exploration, and visualization. For production-ready code, refactor into `.py` scripts.
- **Clear Structure**: Organize notebooks with clear headings, markdown cells for explanations, and logical flow.
- **Version Control**: Be mindful of notebook diffs in Git. Consider tools like `nbdime` for better diffing.
- **Reproducibility**: Ensure notebooks can be run from top to bottom without errors and produce consistent results (especially with random seeds).

### 6.3 Balancing Code Quality vs. Research Velocity
- **Iterative Refinement**: Start with functional code for research velocity, then refactor and improve code quality as experiments stabilize or move towards production.
- **Automated Checks**: Use linters, formatters, and basic unit tests to maintain a baseline of code quality without hindering rapid iteration.
- **Documentation**: Prioritize documenting key experiments and findings, even if the code is still exploratory.

### 6.4 GPU/CPU Device Management
- **Explicit Device Placement**: Always explicitly move tensors and models to the correct device (`.to(device)`).
- **`torch.no_grad()`**: Use `with torch.no_grad():` for inference and evaluation to reduce memory consumption and speed up computations.
- **Data Parallelism**: For multi-GPU training, consider `torch.nn.DataParallel` or `torch.nn.parallel.DistributedDataParallel` for more complex setups.

### 6.5 Team Collaboration and Code Review (Future)
- **Pull Requests**: For future collaboration, use pull requests for code review. Ensure all changes are reviewed before merging to `main`.
- **Automated Tests**: Implement unit and integration tests to catch regressions and ensure code correctness.
- **Continuous Integration (CI)**: Set up CI pipelines to automatically run tests, linters, and formatters on every push or pull request.

## 7. Planning Phase Instructions for Code Review

Review this plan thoroughly before making any code changes. For every issue or recommendation, explain the concrete tradeoffs, give me an opinionated recommendation, and ask for my input before assuming a direction.

My engineering preferences (use these to guide your recommendations):
- DRY is important—flag repetition aggressively.
- Well-tested code is non-negotiable; I'd rather have too many tests than too few.
- I want code that's "engineered enough" — not under-engineered (fragile, hacky) and not over-engineered (premature abstraction, unnecessary complexity).
- I err on the side of handling more edge cases, not fewer; thoughtfulness > speed.
- Bias toward explicit over clever.

### 7.1 Architecture Review
Evaluate:
- Overall system design and component boundaries.
- Dependency graph and coupling concerns.
- Data flow patterns and potential bottlenecks.
- Scaling characteristics and single points of failure.
- Security architecture (auth, data access, API boundaries).

### 7.2 Code Quality Review
Evaluate:
- Code organization and module structure.
- DRY violations—be aggressive here.
- Error handling patterns and missing edge cases (call these out explicitly).
- Technical debt hotspots.
- Areas that are over-engineered or under-engineered relative to my preferences.

### 7.3 Test Review
Evaluate:
- Test coverage gaps (unit, integration, e2e).
- Test quality and assertion strength.
- Missing edge case coverage—be thorough.
- Untested failure modes and error paths.

### 7.4 Performance Review
Evaluate:
- N+1 queries and database access patterns.
- Memory-usage concerns.
- Caching opportunities.
- Slow or high-complexity code paths.

### 7.5 For Each Issue Found
For every specific issue (bug, smell, design concern, or risk):
- Describe the problem concretely, with file and line references.
- Present 2–3 options, including "do nothing" where that's reasonable.
- For each option, specify: implementation effort, risk, impact on other code, and maintenance burden.
- Give me your recommended option and why, mapped to my preferences above.
- Then explicitly ask whether I agree or want to choose a different direction before proceeding.

### 7.6 Workflow and Interaction
- Do not assume my priorities on timeline or scale.
- After each section, pause and ask for my feedback before moving on.

BEFORE YOU START:
Ask if I want one of two options:
1/ BIG CHANGE: Work through this interactively, one section at a time (Architecture → Code Quality → Tests → Performance) with at most 4 top issues in each section.
2/ SMALL CHANGE: Work through interactively ONE question per review section

FOR EACH STAGE OF REVIEW: output the explanation and pros and cons of each stage's questions AND your opinionated recommendation and why, and then use AskUserQuestion. Also NUMBER issues and then give LETTERS for options and when using AskUserQuestion make sure each option clearly labels the issue NUMBER and option LETTER so the user doesn't get confused. Make the recommended option always the 1st option.

## 8. [TODO] Project-Specific Decisions

- [ ] Define specific logging levels and formats.
- [ ] Choose a primary experiment tracking tool (e.g., MLflow, W&B).
- [ ] Establish specific data versioning strategies.
- [ ] Detail specific pre-commit hooks or CI checks.
- [ ] Outline deployment strategy for models.

---
