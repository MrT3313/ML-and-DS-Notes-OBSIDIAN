---
note_kind: concept
aliases:
  - loss function
  - objective function
  - error function
  - J
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

A cost function is a [[Performance Measure]] that says how bad a model's predictions are; lower is better, and training minimizes it. Géron's example: linear regression's cost is the distance between the line and the training points.

## Formal statement

A per-instance loss $\ell(\hat{y}, y)$ averaged over the [[Training Set]]:

$$J(\boldsymbol\theta) = \frac{1}{m} \sum_{i=1}^{m} \ell\big(h_{\boldsymbol\theta}(\mathbf{x}^{(i)}), y^{(i)}\big)$$

Mean squared error is $\ell = (\hat{y} - y)^2$. Training is $\boldsymbol\theta^{*} = \arg\min_{\boldsymbol\theta} J(\boldsymbol\theta)$.

## Where it is used

Every [[Model-Based Learning]] method defines one; [[Linear Regression]] uses squared error, [[Logistic Regression]] uses log loss. Gradient descent moves $\boldsymbol\theta$ against $\nabla J$ scaled by the [[Learning Rate]]. Its value on training data versus the [[Testing Set]] is the diagnostic for [[Overfitting]] and [[Underfitting]]. The sign-flipped form is the [[Utility Function]].
