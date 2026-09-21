---
note_kind: concept
aliases:
  - model drift
  - concept drift
  - data drift
  - model decay
  - performance decay
up: "[[MLOps]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[DMLS Ch01 Overview of Machine Learning Systems]]"
confidence: draft
---

## Definition

Model rot is the slow decline of a deployed model's performance because the world it was trained on has moved: the data distribution, or the relation between features and label, is no longer what it was at training time. The model has not changed; its assumptions have gone stale.

## Formal statement

Let $p_t(\mathbf{x}, y)$ be the production distribution at time $t$ and $p_0$ the training distribution. Rot is $p_t$ drifting from $p_0$, so that

$$\mathcal{L}(h, p_t) \text{ rises with } t \text{ while } h \text{ is fixed}$$

Drift in $p(\mathbf{x})$ alone is data drift; drift in $p(y \mid \mathbf{x})$ is concept drift.

## Where it is used

It is the standing cost of [[Batch Learning]]: a fixed model must be retrained on a schedule. [[Online Learning]] tracks drift automatically at the price of the plasticity-stability trade-off in [[Learning Rate]]. [[Continual Learning]] is what that retraining becomes when it is run as a standing practice with a cadence and a rollback path rather than as a repair, and it is the response this note describes the need for. It is the same mechanism as [[Data Mismatch]], arising over time rather than at collection. Detecting it requires monitoring in production, which [[HOML Ch02 End-to-End Machine Learning Project|HOML chapter 2]] covers.

The same moving world is also a reason to have chosen a learned solution in the first place, which is the symmetry worth holding onto. [[Machine Learning Applicability]] counts constantly changing patterns among the conditions that make a problem worth learning at all, on the grounds that hard-coded solutions such as hand-written rules go out of date quickly when the patterns underneath them keep moving. Rot is that same staleness arriving on the other side of the deployment. What differs is who has to notice. A rule set goes stale only as fast as somebody works out that it has and writes a new one, and a fitted model goes stale with nobody touching it and is brought back by refitting on newer data rather than by re-deriving anything. Rot is therefore the price of the convenience rather than an argument against it, and it is why choosing machine learning for a moving problem commits you to the maintenance above.
