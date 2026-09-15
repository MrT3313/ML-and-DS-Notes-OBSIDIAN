---
note_kind: concept
aliases:
  - model drift
  - concept drift
  - data drift
  - model decay
  - performance decay
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Model rot is the slow decline of a deployed model's performance because the world it was trained on has moved: the data distribution, or the relation between features and label, is no longer what it was at training time. The model has not changed; its assumptions have gone stale.

## Formal statement

Let $p_t(\mathbf{x}, y)$ be the production distribution at time $t$ and $p_0$ the training distribution. Rot is $p_t$ drifting from $p_0$, so that

$$\mathcal{L}(h, p_t) \text{ rises with } t \text{ while } h \text{ is fixed}$$

Drift in $p(\mathbf{x})$ alone is data drift; drift in $p(y \mid \mathbf{x})$ is concept drift.

## Where it is used

It is the standing cost of [[Batch Learning]]: a fixed model must be retrained on a schedule. [[Online Learning]] tracks drift automatically at the price of the plasticity-stability trade-off in [[Learning Rate]]. It is the same mechanism as [[Data Mismatch]], arising over time rather than at collection. Detecting it requires monitoring in production, which chapter 2 covers.
