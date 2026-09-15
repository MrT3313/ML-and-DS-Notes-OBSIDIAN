---
note_kind: concept
aliases:
  - hyperparameters
  - tuning parameter
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

A hyperparameter is a setting of the learning algorithm rather than of the model: it is fixed before training and does not change during it. The amount of regularization and the [[Learning Rate]] are the chapter's two examples.

## Formal statement

Parameters $\boldsymbol\theta$ are chosen by minimizing the [[Cost Function]] on the training set. Hyperparameters $\lambda$ are chosen outside that minimization, by [[Model Selection]] on validation data:

$$\boldsymbol\theta^{*}(\lambda) = \arg\min_{\boldsymbol\theta} J(\boldsymbol\theta; \lambda, D_{\text{fit}}), \qquad \lambda^{*} = \arg\min_{\lambda} \mathcal{L}\big(h_{\boldsymbol\theta^{*}(\lambda)}, D_{\text{val}}\big)$$

## Where it is used

Every `method` note carries a hyperparameter table that links here. Tuning them on the [[Testing Set]] is the mistake [[Holdout Validation]] exists to prevent.
