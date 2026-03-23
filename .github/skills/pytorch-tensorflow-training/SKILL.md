
---
name: PyTorch/TensorFlow Training
description: Provides standard training loop patterns with progress bars, checkpointing, and experiment logging for PyTorch and TensorFlow.
---

# Skill: PyTorch/TensorFlow Training

This skill provides standardized patterns for implementing robust and reproducible training loops in both PyTorch and TensorFlow. It emphasizes best practices for progress tracking, model checkpointing, and experiment logging to ensure efficient development and reliable results.

## 1. Core Training Loop Structure

### 1.1 PyTorch Training Loop

```python
import torch
from tqdm import tqdm
import os

def train_pytorch_model(
    model: torch.nn.Module,
    train_loader: torch.utils.data.DataLoader,
    val_loader: torch.utils.data.DataLoader,
    optimizer: torch.optim.Optimizer,
    criterion: torch.nn.Module,
    device: torch.device,
    epochs: int,
    checkpoint_dir: str = "checkpoints",
    log_interval: int = 100,
    experiment_tracker=None, # e.g., MLflow, Weights & Biases client
    seed: int = 42
):
    """Trains a PyTorch model with progress bar, checkpointing, and experiment logging."""
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)

    os.makedirs(checkpoint_dir, exist_ok=True)
    best_val_loss = float("inf")

    model.to(device)

    for epoch in range(epochs):
        model.train()
        running_loss = 0.0
        train_pbar = tqdm(train_loader, desc=f"Epoch {epoch+1}/{epochs} [Train]")
        for batch_idx, (data, target) in enumerate(train_pbar):
            data, target = data.to(device), target.to(device)

            optimizer.zero_grad()
            output = model(data)
            loss = criterion(output, target)
            loss.backward()
            optimizer.step()

            running_loss += loss.item()
            if batch_idx % log_interval == log_interval - 1:
                avg_batch_loss = running_loss / log_interval
                train_pbar.set_postfix(loss=f"{avg_batch_loss:.4f}")
                if experiment_tracker:
                    experiment_tracker.log_metric("train_batch_loss", avg_batch_loss, step=epoch * len(train_loader) + batch_idx)
                running_loss = 0.0

        # Validation phase
        model.eval()
        val_loss = 0
        correct = 0
        with torch.no_grad():
            val_pbar = tqdm(val_loader, desc=f"Epoch {epoch+1}/{epochs} [Val  ]")
            for data, target in val_pbar:
                data, target = data.to(device), target.to(device)
                output = model(data)
                val_loss += criterion(output, target).item() # sum up batch loss
                pred = output.argmax(dim=1, keepdim=True) # get the index of the max log-probability
                correct += pred.eq(target.view_as(pred)).sum().item()

        val_loss /= len(val_loader)
        accuracy = 100. * correct / len(val_loader.dataset)

        print(f"\nEpoch {epoch+1} Avg Val Loss: {val_loss:.4f}, Accuracy: {accuracy:.2f}%\n")

        if experiment_tracker:
            experiment_tracker.log_metric("val_loss", val_loss, step=epoch)
            experiment_tracker.log_metric("val_accuracy", accuracy, step=epoch)

        # Checkpointing
        if val_loss < best_val_loss:
            best_val_loss = val_loss
            checkpoint_path = os.path.join(checkpoint_dir, f"best_model_epoch_{epoch+1}.pt")
            torch.save({
                "epoch": epoch,
                "model_state_dict": model.state_dict(),
                "optimizer_state_dict": optimizer.state_dict(),
                "val_loss": best_val_loss,
            }, checkpoint_path)
            print(f"Saved best model checkpoint to {checkpoint_path}")

    print("Training complete.")

# [TODO] Add TensorFlow Training Loop example
```

### 1.2 TensorFlow Training Loop (Placeholder)

```python
# [TODO] Implement a similar training loop for TensorFlow models.
# It should include:
# - Model compilation with optimizer, loss, and metrics.
# - Iteration over epochs and batches.
# - Progress bars (e.g., Keras callbacks).
# - Checkpointing (e.g., ModelCheckpoint callback).
# - Experiment logging (e.g., TensorBoard callback, custom logger).
# - Device management (e.g., tf.distribute.Strategy).
```

## 2. Key Features and Best Practices

- **Progress Bars (`tqdm`)**: Provides visual feedback on training progress, including estimated time remaining.
- **Checkpointing**: Automatically saves the model's state (weights, optimizer state) based on validation performance. This allows for resuming training or deploying the best model.
- **Experiment Logging**: Integrates with external experiment tracking tools (e.g., MLflow, Weights & Biases, TensorBoard) to log metrics, hyperparameters, and artifacts, crucial for reproducibility and analysis.
- **Device Agnostic**: Code is written to run seamlessly on both CPU and GPU, adapting to the available hardware.
- **Random Seed Management**: Ensures reproducibility of training runs by setting random seeds for all relevant libraries.
- **Validation Loop**: A dedicated validation phase helps monitor for overfitting and guides hyperparameter tuning and early stopping decisions.

## 3. Usage

To use this skill, provide the necessary components (model, data loaders, optimizer, criterion, device, epochs) to the `train_pytorch_model` function. Optionally, pass an `experiment_tracker` instance for integrated logging.

```python
# Example usage (assuming model, loaders, optimizer, criterion, device are defined)
# from my_project.models.my_model import MyModel
# from my_project.data.my_dataset import MyDataset

# model = MyModel(...)
# optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
# criterion = torch.nn.CrossEntropyLoss()
# device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# train_pytorch_model(
#     model=model,
#     train_loader=train_loader,
#     val_loader=val_loader,
#     optimizer=optimizer,
#     criterion=criterion,
#     device=device,
#     epochs=10,
#     checkpoint_dir="./my_checkpoints",
#     experiment_tracker=None # Replace with actual tracker if used
# )
```

## 4. [TODO] Customization and Extensions

- [ ] Add support for learning rate schedulers.
- [ ] Implement early stopping logic.
- [ ] Extend experiment logging to include model graphs, gradients, and more detailed metrics.
- [ ] Integrate distributed training setup.
- [ ] Provide a TensorFlow-specific training loop.

---
