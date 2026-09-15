---
note_kind: concept
aliases:
  - self-supervised
  - SSL
  - pretext task
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---
## Definition

Self-supervised learning generates its own labels from unlabeled data, typically by hiding part of each instance and training the model to recover it. The learned model is then used directly or adapted to a downstream task.

## Formal statement

From an unlabeled $\mathbf{x}$, construct a pair $(\tilde{\mathbf{x}}, t(\mathbf{x}))$ where $\tilde{\mathbf{x}}$ is a corrupted or partial view and $t(\mathbf{x})$ is the withheld part; then train $h(\tilde{\mathbf{x}}) \approx t(\mathbf{x})$ as ordinary [[Supervised Learning]]. Not further quantitative at this depth.

## Where it is used

My formulation: it is supervised learning on manufactured labels. It starts from unlabeled data like [[Unsupervised Learning]], but its objective is a [[Classification]] or [[Regression]] loss, and its output is a model rather than a description of the data. [[HOML]] notes that some authors file it under unsupervised learning; I do not.

