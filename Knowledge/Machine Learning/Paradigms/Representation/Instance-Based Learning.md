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

The alternative is [[Model-Based Learning]], which fits parameters once and never consults the stored instances again, so its prediction cost is independent of $m$ where this paradigm's grows with it. The canonical method is k-nearest neighbours, which has no note yet and which no HOML chapter takes as its subject. Every method here measures distance between instances, so it depends on [[Feature Scaling]]: a column measured in tens of thousands swamps a column in $[0, 1]$ in every similarity computed, and the nearest neighbours are then chosen by that one column. Fails when the similarity measure ignores what matters, and when $m$ is too large to search at prediction time.
