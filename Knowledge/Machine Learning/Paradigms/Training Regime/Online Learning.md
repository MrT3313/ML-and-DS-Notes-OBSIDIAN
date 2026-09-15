---
note_kind: concept
aliases:
  - incremental learning
  - streaming learning
  - mini-batch learning
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---
## Definition

In online learning the system learns incrementally, taking instances one at a time or in small groups called mini-batches, and updating its parameters after each. It suits continuous data streams, rapidly changing environments, and limited compute.

## Formal statement

Each step consumes a mini-batch $B_t$ of size $b$ (often $b = 1$) and updates $\boldsymbol\theta_{t+1} = \boldsymbol\theta_t + \Delta(\boldsymbol\theta_t, B_t)$; the step size is the [[Learning Rate]]. High $\eta$ adapts fast and forgets fast; low $\eta$ is stable and slow. That is the plasticity-stability dilemma.

## Where it is used

The alternative is [[Batch Learning]]. The same mechanism run over a dataset too large for memory is [[Out-of-Core Learning]], which is why "online" is a misnomer there. The failure mode is bad data arriving live: the model degrades in production, so monitoring and the ability to roll back matter more than in batch settings.
