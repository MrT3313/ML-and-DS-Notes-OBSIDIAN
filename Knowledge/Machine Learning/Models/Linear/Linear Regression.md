---
note_kind: method
aliases:
  - ordinary least squares
  - OLS
  - linear model
  - multiple regression
up: "[[Regression]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## What it does and when

Linear regression predicts a continuous target as a weighted sum of the features plus a bias, with the weights chosen to minimize squared error on the training set. Use it when the relation is close to linear, when interpretability matters, or as the baseline every other regressor must beat. Belongs to the [[Regression]] map and is the worked example of [[Model-Based Learning]].

## Algorithm or formula

$$\hat{y} = h_{\boldsymbol\theta}(\mathbf{x}) = \theta_0 + \theta_1 x_1 + \dots + \theta_n x_n = \boldsymbol\theta^{T} \mathbf{x}$$

with $x_0 = 1$. The [[Cost Function]] is mean squared error,

$$J(\boldsymbol\theta) = \frac{1}{m} \sum_{i=1}^{m} \big(\boldsymbol\theta^{T}\mathbf{x}^{(i)} - y^{(i)}\big)^2$$

and training sets $\boldsymbol\theta^{*} = \arg\min J$. The closed form and the gradient descent route arrive with HOML chapter 4.

## Hyperparameters

None for the plain fit. Regularized variants (ridge, lasso) add a penalty weight; chapter 4.

## Failure modes

- Nonlinear relation: the model underfits ([[Underfitting]]) and no amount of data helps.
- Outliers: squared error lets a few far points dominate the fit.
- Highly correlated features: weights become unstable and uninterpretable.

## Implementation

scikit-learn 1.6:

```python
from sklearn.linear_model import LinearRegression
model = LinearRegression().fit(X_train, y_train)
y_pred = model.predict(X_new)
```
