---
note_kind: concept
aliases:
  - fitness function
  - reward function
  - objective function (maximized)
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

A utility function is a [[Performance Measure]] that says how good a model is; higher is better, and training maximizes it. Genetic algorithms call it fitness; reinforcement learning calls it reward.

## Formal statement

$$\boldsymbol\theta^{*} = \arg\max_{\boldsymbol\theta} U(\boldsymbol\theta)$$

Any utility can be written as a [[Cost Function]] by negation, $J = -U$, so the choice between them is convention, not substance.

## Where it is used

Rare in supervised learning, where the [[Cost Function]] form dominates. It is the native form in reinforcement learning and evolutionary search.
