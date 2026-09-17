---
note_kind: concept
aliases:
  - overfit
  - overfitting the training data
  - high variance
up: "[[Generalization]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

A model overfits when it learns the training data too closely, including its noise and quirks, so it performs well on the training set and poorly on new data. The model is more complex than the data can justify.

## Formal statement

$$\mathcal{L}_{\text{train}} \ll \mathcal{L}_{\text{test}}$$

The generalization gap of [[Generalization]] is large. It appears when model capacity is high relative to $m$, so the remedies are: a simpler model or fewer parameters, more training data, or less noise in the data (fix errors, remove outliers). Constraining the model is [[Regularization]], and what it does to the inequality is close it from both sides: the training loss is allowed to get worse on purpose, which is the price, and the test loss falls, which is the point. The generalization gap narrows even though the fit to the training data is now deliberately imperfect.

Overfitting is the high-variance end of the [[Bias-Variance Tradeoff]], so the same dial that closes this gap opens the [[Underfitting]] one at the other end, and there is no setting that escapes both. A [[Learning Curve]] is the diagnostic that says which end you are currently on.

## Where it is used

It is the failure mode nearly every model family has to be defended against, whatever shape that family's capacity takes. Detected by comparing training and [[Testing Set]] performance, and controlled through [[Model Selection]] with [[Holdout Validation]] or [[Cross-Validation]]. The opposite failure is [[Underfitting]].
