---
note_kind: concept
aliases:
  - hyperparameters
  - tuning parameter
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

A hyperparameter is a setting of the learning algorithm rather than of the model: it is fixed before training and does not change during it. [[Regularization]] strength and the [[Learning Rate]] are the two standard examples.

## Formal statement

Parameters $\boldsymbol\theta$ are chosen by minimizing the [[Cost Function]] on the training set. Hyperparameters $\lambda$ are chosen outside that minimization, by [[Model Selection]] on validation data:

$$\boldsymbol\theta^{*}(\lambda) = \arg\min_{\boldsymbol\theta} J(\boldsymbol\theta; \lambda, D_{\text{fit}}), \qquad \lambda^{*} = \arg\min_{\lambda} \mathcal{L}\big(h_{\boldsymbol\theta^{*}(\lambda)}, D_{\text{val}}\big)$$

## Where it is used

Anything with a tunable setting has some: [[Ridge Regression]], [[Lasso Regression]] and [[Elastic Net Regression]] are searched over $\alpha$, every [[Gradient Descent]] variant over $\eta$ and its [[Learning Schedule]], and [[Polynomial Regression]] over the degree. Tuning them on the [[Testing Set]] is the mistake [[Holdout Validation]] exists to prevent, and enumerating them is what [[Grid Search]] and [[Randomized Search]] do.
