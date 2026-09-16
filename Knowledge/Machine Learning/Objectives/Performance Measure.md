---
note_kind: concept
aliases:
  - performance metric
  - evaluation metric
  - objective
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

A performance measure is the number that says how well a model does its task. It plays two roles the chapter does not separate: the quantity optimized during training, and the quantity reported to judge the trained model. The two need not be the same function.

## Formal statement

It is the $P$ of Mitchell's definition in [[Machine Learning]]. Two sign conventions: a [[Utility Function]] $U(h)$ is maximized, a [[Cost Function]] $J(\boldsymbol\theta)$ is minimized, and $U = -J$ up to a constant. Evaluation metrics such as accuracy or mean squared error are computed on the [[Testing Set]]:

$$\hat{P} = \frac{1}{m_{\text{test}}} \sum_{i=1}^{m_{\text{test}}} \text{score}\big(h(\mathbf{x}^{(i)}), y^{(i)}\big)$$

## Where it is used

Training optimizes one; [[Model Selection]] compares candidates on one; [[Generalization]] is the gap between its training and test values. Chapter 3 will add the classification metrics in `Generalization/`.
