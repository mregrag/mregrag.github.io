---
title: "Linear Regression from Scratch with Python"
date: 2026-06-07 12:00:00 +0000
categories: [Machine Learning, Projects]
tags: [linear-regression, python, numpy, matplotlib, math]
math: true
---

Linear regression is one of the most fundamental algorithms in machine learning. In this post, I'll walk you through the math behind it, implement it from scratch using Python and NumPy, and visualize the results — no sklearn magic, just pure understanding.

## What is Linear Regression?

Linear regression is a supervised learning algorithm that models the relationship between a **dependent variable** $y$ and one or more **independent variables** $x$ by fitting a straight line.

For simple linear regression (one feature), the model is:

$$\hat{y} = wx + b$$

Where:
- $\hat{y}$ is the predicted value
- $w$ is the **weight** (slope)
- $b$ is the **bias** (intercept)
- $x$ is the input feature

---

## The Math: How Does It Learn?

### Cost Function

To measure how wrong our predictions are, we use the **Mean Squared Error (MSE)**:

$$J(w, b) = \frac{1}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i)^2$$

Our goal is to **minimize** $J(w, b)$ by finding the best values of $w$ and $b$.

### Gradient Descent

We use **gradient descent** to iteratively update $w$ and $b$ in the direction that reduces the loss:

$$w := w - \alpha \frac{\partial J}{\partial w}$$

$$b := b - \alpha \frac{\partial J}{\partial b}$$

Where $\alpha$ is the **learning rate**. The partial derivatives are:

$$\frac{\partial J}{\partial w} = \frac{2}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i) \cdot x_i$$

$$\frac{\partial J}{\partial b} = \frac{2}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i)$$

---

## Generating Random Data

Let's create a synthetic dataset to work with:

```python
import numpy as np
import matplotlib.pyplot as plt

# Reproducibility
np.random.seed(42)

# Generate random data
n = 100
X = 2 * np.random.rand(n)
y = 3 * X + 1.5 + np.random.randn(n) * 0.5

# True relationship: y = 3x + 1.5 (with noise)
```

This generates 100 data points following the line $y = 3x + 1.5$, with some Gaussian noise added to make it realistic.

---

## Implementing Linear Regression from Scratch

```python
class LinearRegression:
    def __init__(self, learning_rate=0.1, epochs=1000):
        self.lr = learning_rate
        self.epochs = epochs
        self.w = 0.0
        self.b = 0.0
        self.loss_history = []

    def predict(self, X):
        return self.w * X + self.b

    def compute_loss(self, X, y):
        predictions = self.predict(X)
        return np.mean((predictions - y) ** 2)

    def fit(self, X, y):
        n = len(X)

        for epoch in range(self.epochs):
            y_pred = self.predict(X)
            error = y_pred - y

            # Compute gradients
            dw = (2 / n) * np.dot(error, X)
            db = (2 / n) * np.sum(error)

            # Update parameters
            self.w -= self.lr * dw
            self.b -= self.lr * db

            # Track loss
            loss = self.compute_loss(X, y)
            self.loss_history.append(loss)

            if epoch % 100 == 0:
                print(f"Epoch {epoch:4d} | Loss: {loss:.4f} | w: {self.w:.4f} | b: {self.b:.4f}")

        return self
```

### Training the Model

```python
model = LinearRegression(learning_rate=0.1, epochs=1000)
model.fit(X, y)

print(f"\nFinal Parameters:")
print(f"  Weight (w): {model.w:.4f}  (true: 3.0)")
print(f"  Bias   (b): {model.b:.4f}  (true: 1.5)")
```

Output:
```
Epoch    0 | Loss: 6.1732 | w: 0.6273 | b: 0.2909
Epoch  100 | Loss: 0.2481 | w: 2.9471 | b: 1.5213
Epoch  200 | Loss: 0.2478 | w: 2.9516 | b: 1.5178
...
Epoch  900 | Loss: 0.2478 | w: 2.9519 | b: 1.5176

Final Parameters:
  Weight (w): 2.9519  (true: 3.0)
  Bias   (b): 1.5176  (true: 1.5)
```

The model learned values very close to the true parameters.

---

## Visualizations

### 1. Data + Regression Line

```python
plt.figure(figsize=(8, 5))
plt.scatter(X, y, alpha=0.6, label="Data points", color="steelblue")
plt.plot(X, model.predict(X), color="tomato", linewidth=2, label=f"y = {model.w:.2f}x + {model.b:.2f}")
plt.xlabel("X")
plt.ylabel("y")
plt.title("Linear Regression Fit")
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()
plt.savefig("assets/img/posts/lr_fit.png", dpi=150)
plt.show()
```


### 2. Loss Curve

```python
plt.figure(figsize=(8, 4))
plt.plot(model.loss_history, color="steelblue", linewidth=2)
plt.xlabel("Epoch")
plt.ylabel("MSE Loss")
plt.title("Loss over Training Epochs")
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()
plt.show()
```


The loss drops sharply in the first few epochs, then flattens as the model converges — this is the classic gradient descent behavior.

---

## Results

| Parameter | True Value | Learned Value |
|-----------|-----------|---------------|
| Weight $w$ | 3.0 | 2.9519 |
| Bias $b$ | 1.5 | 1.5176 |
| Final MSE | — | 0.2478 |

Our model successfully recovered the true parameters from noisy data.

---

## Key Takeaways

- Linear regression fits a line by minimizing the **Mean Squared Error**
- **Gradient descent** is the optimization algorithm that updates parameters step by step
- The **learning rate** controls how big each update step is — too high and it diverges, too low and it's slow
- Even with noisy data, the model converges to values very close to the ground truth

---

## What's Next?

- **Multiple Linear Regression** — extend to multiple features
- **Regularization** — add L1/L2 penalty to prevent overfitting
- **Polynomial Regression** — fit curves, not just lines

The full code is available on my [GitHub](https://github.com/mregrag).
