---
note_kind: concept
aliases:
  - underfit
  - underfitting the training data
  - high bias
up: "[[Generalization]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

A model underfits when it is too simple to capture the structure in the data, so it performs poorly on the training set and on new data alike. No amount of extra data fixes it; the model family is the problem.

## Formal statement

$$\mathcal{L}_{\text{train}} \approx \mathcal{L}_{\text{test}}, \quad \text{both high}$$

Remedies: a more powerful model, better [[Feature]]s, or fewer constraints (less regularization).

## Where it is used

The opposite failure to [[Overfitting]]; the two bound the [[Model Selection]] problem from below and above. Diagnosed from training error alone, which is what makes it the cheaper failure to catch.
