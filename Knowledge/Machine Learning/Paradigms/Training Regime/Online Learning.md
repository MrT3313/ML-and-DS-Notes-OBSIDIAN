---
note_kind: concept
aliases:
  - incremental learning
  - streaming learning
  - mini-batch learning
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
  - "[[DMLS Ch01 Overview of Machine Learning Systems]]"
confidence: draft
---
## Definition

In online learning the system learns incrementally, taking instances one at a time or in small groups called mini-batches, and updating its parameters after each. It suits continuous data streams, rapidly changing environments, and limited compute.

## Formal statement

Each step consumes a mini-batch $B_t$ of size $b$ (often $b = 1$) and updates $\boldsymbol\theta_{t+1} = \boldsymbol\theta_t + \Delta(\boldsymbol\theta_t, B_t)$; the step size is the [[Learning Rate]]. High $\eta$ adapts fast and forgets fast; low $\eta$ is stable and slow. That is the plasticity-stability dilemma.

Step size and adaptation rate are one number read two ways, not two knobs. On a squared-error loss the update is an exponential moving average of the targets with smoothing factor $2\eta$, so an instance seen $k$ steps ago still weighs $(1 - 2\eta)^{k}$ and the effective memory is about $1/(2\eta)$ instances: moving further toward the newest instance is exactly what discarding the older ones means. [[Learning Rate]] carries the derivation.

## Where it is used

The alternative is [[Batch Learning]]. [[Stochastic Gradient Descent]] is the update that makes any of this possible, being the $b = 1$ case of the step above, and [[Mini-Batch Gradient Descent]] is the same thing for $b > 1$; in scikit-learn 1.6 both reach an estimator through `partial_fit`, which keeps its update counter running across calls rather than restarting. That detail has a consequence in production: a decaying [[Learning Schedule]] goes on decaying through the stream, so a system meant to track a changing environment wants a rate that does not decay, in practice `learning_rate="constant"`, or it stops adapting while still appearing to train. The same mechanism run over a dataset too large for memory is [[Out-of-Core Learning]], which is why "online" is a misnomer there. The word is overloaded a second time outside this domain, and not in this sense at all: in [[Online Transaction Processing]] and [[Online Analytical Processing]] "online" means interactive, a request answered while the caller sits waiting for it, which says nothing about whether anything is fitted one instance at a time. The failure mode is bad data arriving live: the model degrades in production, so monitoring and the ability to roll back matter more than in batch settings. Whether a deployed model goes on being updated at all is a separate question again, and it is [[Continual Learning]]: online learning says how one update is computed, from a single instance or a single mini-batch, while continual learning says only that the updating keeps happening after deployment, which is most often done by retraining in batches on a schedule and so need not be online learning at all.
