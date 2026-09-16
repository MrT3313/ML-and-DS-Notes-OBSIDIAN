---
note_kind: concept
aliases:
  - memory-based learning
  - lazy learning
  - nonparametric learning
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

An instance-based system memorizes the training examples and generalizes by comparing a new instance to stored ones through a similarity measure. Nothing is fit; the training set is the model.

## Formal statement

Given similarity $s(\mathbf{x}, \mathbf{x}')$, predict for $\mathbf{x}$ from the labels of its most similar stored instances, for instance the $k$ nearest:

$$\hat{y} = \text{aggregate}\{\, y^{(i)} : \mathbf{x}^{(i)} \in N_k(\mathbf{x}) \,\}$$

Prediction cost grows with $m$ because every stored instance is a candidate.

## Where it is used

The alternative is [[Model-Based Learning]]. The canonical method is [[K-Nearest Neighbours]]. Fails when the similarity measure ignores what matters, and when $m$ is too large to search at prediction time.
