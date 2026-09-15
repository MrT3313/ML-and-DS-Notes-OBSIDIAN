---
note_kind: concept
aliases:
  - training data
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition
The [[Training Set]] is the collection of [[Training Instance]]s the system learns from. These are the remaining [[instance]]s after the [[Testing Set]] has been extracted from the overall dataset.  

## Formal statement

The training set is the matrix $\mathbf{X}$ with $m$ rows, one per instance $\mathbf{x}^i$, plus the label vector $\mathbf{y}$ with entries $y^i$ when labels exist:

$$D_{\text{train}} = \{(\mathbf{x}^i, y^i)\}_{i=1}^{m}$$

## Where it is used

Every [[Model]] is fit to a training set. Its size drives [[Insufficient Training Data]] and the [[Unreasonable Effectiveness of Data]]; how it was collected drives [[Nonrepresentative Training Data]]. It is held apart from the [[Testing Set]], and a slice of it is carved off for [[Holdout Validation]]. Each column is a [[Feature]].
