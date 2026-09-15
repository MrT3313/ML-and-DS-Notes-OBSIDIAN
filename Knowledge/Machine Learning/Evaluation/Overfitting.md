---
note_kind: concept
aliases:
  - overfit
  - overfitting the training data
  - high variance
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

A model overfits when it learns the training data too closely, including its noise and quirks, so it performs well on the training set and poorly on new data. The model is more complex than the data can justify.

## Formal statement

$$\mathcal{L}_{\text{train}} \ll \mathcal{L}_{\text{test}}$$

The generalization gap of [[Generalization]] is large. It appears when model capacity is high relative to $m$, so the remedies are: a simpler model or fewer parameters, more training data, or less noise in the data (fix errors, remove outliers). Constraining the model is regularization, which the chapter covers and this vault does not yet have a note for.

## Where it is used

Every `method` note names it under Failure modes. Detected by comparing training and [[Testing Set]] performance, and controlled through [[Model Selection]] with [[Holdout Validation]] or [[Cross-Validation]]. The opposite failure is [[Underfitting]].
