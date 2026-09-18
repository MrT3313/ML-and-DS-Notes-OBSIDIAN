---
note_kind: concept
aliases:
  - fitness function
  - reward function
  - objective function (maximized)
up: "[[Performance Measure]]"
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

Rare in supervised learning, where the [[Cost Function]] form dominates. It is the native form in reinforcement learning and evolutionary search. It is the maximized one of the two sign conventions a [[Performance Measure]] can take, and [[Model Selection]] compares candidates on a validation score of either sign, ranking them by $\arg\max$ under a utility where it ranks by $\arg\min$ under a cost. [[Accuracy]] is a utility in this sense, higher being better, though it is reported rather than optimized, since the indicator inside it has no gradient to follow.
