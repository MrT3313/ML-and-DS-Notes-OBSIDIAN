---
note_kind: concept
aliases:
  - unsupervised
  - unlabeled learning
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

In unsupervised learning the training data has no labels. The system has to find structure on its own: groups, low-dimensional descriptions, unusual points, or co-occurrence rules.

## VS

It is one end of the supervision axis with [[Supervised Learning]] at the other and [[Semi-Supervised Learning]] and [[Self-Supervised Learning]] between. Whether self-supervised learning counts as unsupervised is disputed; see that note.

## Formal statement

Given $D = \{\mathbf{x}^{(i)}\}_{i=1}^{m}$ with no $y$, learn a description of the data distribution $p(\mathbf{x})$ or a function of it.

**Not further quantitative at this depth.**

Each task has its own objective.

## Where it is used

### Tasks

- [[Clustering]] groups the instances by similarity, with no label saying what the groups are.
- [[Dimensionality Reduction]] finds a low-dimensional description of the data that keeps most of its variation.
- [[Anomaly Detection]] flags instances far from the bulk of a training set that may itself contain some outliers.
- [[Novelty Detection]] flags the same way but trains on a reference set assumed clean, so anything unlike it is novel.
- [[Association Rule Learning]] finds attribute values that co-occur more often than chance and states them as rules.
