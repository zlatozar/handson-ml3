
---
# Training Script Guidelines

This document provides specific guidelines for Python files located in the `training/` directory, focusing on the implementation of training loops, evaluation procedures, and related utilities. These instructions aim to ensure robust, reproducible, and efficient training of Machine Learning models using PyTorch.

## 1. Training Loop Structure

- **Epoch-based Iteration**: Organize the training process into epochs, with each epoch iterating over the entire dataset.
- **Batch Processing**: Within each epoch, iterate over data in mini-batches using `torch.utils.data.DataLoader`.
- **Forward Pass**: Perform the forward pass of the model to get predictions.
- **Loss Calculation**: Compute the loss between predictions and ground truth using appropriate loss functions (e.g., `torch.nn.CrossEntropyLoss`, `torch.nn.MSELoss`).
- **Backward Pass and Optimization**: Perform backpropagation (`loss.backward()`) and update model parameters (`optimizer.step()`, `optimizer.zero_grad()`).

## 2. Optimization and Scheduling

- **Optimizer Selection**: Choose an appropriate optimizer (e.g., `torch.optim.Adam`, `torch.optim.SGD`) and configure its hyperparameters (learning rate, weight decay).
- **Learning Rate Schedulers**: Implement learning rate schedulers (e.g., `torch.optim.lr_scheduler.StepLR`, `torch.optim.lr_scheduler.ReduceLROnPlateau`) to adjust the learning rate during training for better convergence.
- **Gradient Clipping**: Apply gradient clipping (e.g., `torch.nn.utils.clip_grad_norm_`) to prevent exploding gradients, especially in recurrent neural networks or transformers.

## 3. Evaluation and Validation

- **Separate Validation Loop**: Always include a separate validation loop after each training epoch (or at regular intervals) to monitor model performance on unseen data and detect overfitting.
- **Metrics**: Calculate and log relevant evaluation metrics (e.g., accuracy, precision, recall, F1-score, AUC, RMSE) for both training and validation sets.
- **Early Stopping**: Implement early stopping based on validation performance to prevent overfitting and save computational resources.

## 4. Checkpointing and Resumption

- **Model Checkpointing**: Save model checkpoints periodically (e.g., after each epoch, or when validation performance improves). Store `model.state_dict()`, `optimizer.state_dict()`, and `scheduler.state_dict()`.
- **Resuming Training**: Provide functionality to resume training from a saved checkpoint, including loading model weights, optimizer state, and epoch number.
- **Best Model Saving**: Keep track of the best performing model on the validation set and save its state.

## 5. Logging and Visualization

- **Experiment Tracking Integration**: Integrate with experiment tracking tools (e.g., MLflow, Weights & Biases, TensorBoard) to log training and validation metrics, loss curves, and model hyperparameters.
- **Progress Bars**: Use progress bars (e.g., `tqdm`) to visualize training progress per epoch and batch.
- **Informative Output**: Log key information such as epoch number, training loss, validation loss, and metrics to the console or log file.

## 6. Device Management

- **GPU Utilization**: Ensure efficient GPU utilization by moving data and models to the GPU at the beginning of the training process and managing memory effectively.
- **`torch.cuda.empty_cache()`**: Call `torch.cuda.empty_cache()` periodically if memory issues arise, especially after validation loops.

## 7. [TODO] Project-Specific Training Details

- [ ] Define specific loss functions and their configurations.
- [ ] Outline hyperparameter search strategies.
- [ ] Specify data loading workers and pin_memory settings.
- [ ] Detail distributed training setup (if applicable).

---
