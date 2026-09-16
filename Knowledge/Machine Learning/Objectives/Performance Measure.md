---
note_kind: concept
aliases:
  - performance metric
  - evaluation metric
  - objective
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

A performance measure is the number that says how well a model does its task. It plays two roles that are easy to conflate: the quantity optimized during training, and the quantity reported to judge the trained model. The two need not be the same function.

## Formal statement

It is the $P$ of Mitchell's definition in [[Machine Learning]]. Two sign conventions: a [[Utility Function]] $U(h)$ is maximized, a [[Cost Function]] $J(\boldsymbol\theta)$ is minimized, and $U = -J$ up to a constant. Evaluation metrics such as accuracy or mean squared error are computed on the [[Testing Set]]:

$$\hat{P} = \frac{1}{m_{\text{test}}} \sum_{i=1}^{m_{\text{test}}} \text{score}\big(h(\mathbf{x}^{(i)}), y^{(i)}\big)$$

## Where it is used

Training optimizes one; [[Model Selection]] compares candidates on one; [[Generalization]] is the gap between its training and test values.

On the classification side, every one of these measures is a reading of a single object, the [[Confusion Matrix]] that counts predicted labels against true ones. [[Accuracy]] is its diagonal over its total, so it spends the whole matrix at once and says nothing about which mistake was made. [[Precision]] and [[Recall]] take complementary slices of it, one conditioned on the column the model chose and one on the row the data fixed, which is why they survive an imbalanced target that accuracy cannot read. [[F1 Score]] is the harmonic mean of those two, a single number that refuses to let either be bought with the other. And two of these measures are not numbers at all but sweeps: [[Precision-Recall Tradeoff]] and [[ROC Curve]] each trace a whole family of confusion matrices as the decision threshold moves, which makes the choice of operating point part of the measurement rather than a detail underneath it.
