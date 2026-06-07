---
title: "Linear Regression from Scratch with Python"
date: 2026-06-07 12:00:00 +0000
categories: [Machine Learning, Projects]
tags: [linear-regression, python, numpy, math]
math: true
---

Linear regression is one of the first algorithms you'll encounter in machine learning — and one of the most important. In this post, I'll explain the math behind it and implement it from scratch using Python and NumPy, no libraries like sklearn involved.

## What is Linear Regression?

Linear regression models the relationship between an input $x$ and an output $y$ by fitting a straight line through the data:

$$\hat{y} = wx + b$$

Where:
- $w$ is the **weight** (slope of the line)
- $b$ is the **bias** (intercept)
- $\hat{y}$ is our **predicted value**

The goal is to find the values of $w$ and $b$ that make our predictions as close as possible to the real values.

---

## The Cost Function

To measure how wrong our model is, we use the **Mean Squared Error (MSE)**:

$$J(w, b) = \frac{1}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i)^2$$

The bigger the error, the higher $J$. We want to **minimize** this.

---

## Gradient Descent

We minimize $J$ using **gradient descent** — an iterative algorithm that updates $w$ and $b$ step by step:

$$w := w - \alpha \frac{\partial J}{\partial w}$$

$$b := b - \alpha \frac{\partial J}{\partial b}$$

Where $\alpha$ is the **learning rate** — it controls how big each step is.

The gradients are:

$$\frac{\partial J}{\partial w} = \frac{2}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i) \cdot x_i$$

$$\frac{\partial J}{\partial b} = \frac{2}{n} \sum_{i=1}^{n} (\hat{y}_i - y_i)$$

---

## The Code

### 1. Generate Random Data

```python
import numpy as np

np.random.seed(42)

n = 100
X = 2 * np.random.rand(n)
y = 3 * X + 1.5 + np.random.randn(n) * 0.5
```

We create 100 data points following the line $y = 3x + 1.5$ with some random noise added. The goal of our model is to recover these true parameters: $w = 3$ and $b = 1.5$.

---

### 2. The Linear Regression Class

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

Let's break down each method:

- **`__init__`** — sets the learning rate, number of epochs, and initializes $w$ and $b$ to zero.
- **`predict`** — applies $\hat{y} = wx + b$ to compute predictions.
- **`compute_loss`** — calculates the MSE between predictions and real values.
- **`fit`** — runs gradient descent for the given number of epochs, updating $w$ and $b$ each time.

---

### 3. Train the Model

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
Epoch  300 | Loss: 0.2478 | w: 2.9518 | b: 1.5177
...
Epoch  900 | Loss: 0.2478 | w: 2.9519 | b: 1.5176

Final Parameters:
  Weight (w): 2.9519  (true: 3.0)
  Bias   (b): 1.5176  (true: 1.5)
```

The model converged to values very close to the true parameters, despite the noise in the data.

---

## Results

| Parameter | True Value | Learned Value |
|-----------|-----------|---------------|
| Weight $w$ | 3.0 | 2.9519 |
| Bias $b$ | 1.5 | 1.5176 |
| Final MSE | — | 0.2478 |

---

## Key Takeaways

- Linear regression finds the best line through data by minimizing the **Mean Squared Error**
- **Gradient descent** is the engine that does the optimization — it nudges $w$ and $b$ in the right direction each epoch
- The **learning rate** $\alpha$ is critical — too large and the model diverges, too small and it trains very slowly
- Even with noisy data, the model recovers parameters very close to the ground truth

---

## What's Next?

- **Multiple Linear Regression** — extend to more than one input feature
- **Regularization (L1 / L2)** — prevent overfitting by penalizing large weights
- **Logistic Regression** — the natural next step for classification problems

The full code is available on my [GitHub](https://github.com/mregrag).
