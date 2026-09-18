---
note_kind: concept
aliases:
  - supervised
  - labeled learning
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---
## Definition

In [[Supervised Learning]] the [[Training Set]] carries the desired output for each instance, called the label. The model learns a mapping from features to labels so it can produce labels for instances it has not seen.

## VS

[[Semi-Supervised Learning]] relaxes the requirement that every instance be labeled; [[Self-Supervised Learning]] manufactures the labels; [[Unsupervised Learning]] drops them. Label cost is what makes the alternatives attractive.

## Formal statement

Given $D = \{(\mathbf{x}^{(i)}, y^{(i)})\}_{i=1}^{m}$, find $h$ such that $h(\mathbf{x}) \approx y$ on new $(\mathbf{x}, y)$ drawn from the same distribution. When $y$ is discrete the task is [[Classification]]; when $y$ is continuous it is [[Regression]].

## Where it is used

### Tasks

- [[Classification]] is the supervised task whose label is discrete, so the model learns to assign each instance a class.
- [[Regression]] is the supervised task whose label is continuous, so the model learns to predict a number.

### Models

- [[Linear Regression]] is the base supervised model for regression, fitting a weighted sum of the features to the label.
- [[Logistic Regression]] is the base supervised model for classification, squashing that same weighted sum into a class probability.

