---
note_kind: concept
aliases:
  - step size
  - eta
  - $\eta$
  - adaptation rate
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

The learning rate is the [[Hyperparameter]] that scales how far the parameters move on each training step. This chapter uses it in the [[Online Learning]] sense: how fast the system adapts to new data. Chapter 4 uses it as the gradient descent step size. In stochastic gradient descent the two are the same number.

## Formal statement

$$\boldsymbol\theta \leftarrow \boldsymbol\theta - \eta \, \nabla_{\boldsymbol\theta} J(\boldsymbol\theta)$$

with $J$ the [[Cost Function]]. Large $\eta$: fast adaptation, fast forgetting, risk of divergence. Small $\eta$: stable, slow, may stall.

## Where it is used

Every gradient-based method carries it in its hyperparameter table. In [[Online Learning]] it governs the plasticity-stability trade-off. Tuning it is [[Model Selection]] work.
