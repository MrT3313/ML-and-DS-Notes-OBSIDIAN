---
note_kind: concept
aliases:
  - out-of-core
  - out-of-memory training
up: "[[Online Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---
## Definition

Out-of-core learning trains on a dataset too large for main memory by loading it in chunks, running one incremental training step per chunk, and discarding the chunk. It is [[Online Learning]] applied to a fixed dataset rather than a stream.

## Formal statement

Partition $D$ into chunks $C_1, \dots, C_K$ with $|C_k| \cdot \text{bytes per instance} \le \text{RAM}$; for each $k$ run the online update on $C_k$. One pass over all chunks is one epoch.

**Not further quantitative at this depth.**

Out-of-core learning is a training regime, and the chunk partition above is the whole of its mathematics at this depth: the constraint is on memory, and the update run on each chunk belongs to whichever optimizer performs it. What remains to add is the optimizer's, not the regime's: the convergence of [[Stochastic Gradient Descent]] when batches arrive from disk in chunk order, which is a question about shuffling rather than about memory.

## Where it is used

It needs an algorithm with an incremental update, which rules out closed-form fits like the [[Normal Equation]] for [[Linear Regression]], since that one needs the whole design matrix at once. [[Stochastic Gradient Descent]] and [[Mini-Batch Gradient Descent]] are the options that qualify: each step reads one instance or one small batch and touches nothing else, so the chunk on disk can be the batch. [[Batch Gradient Descent]] does not qualify despite being iterative, because every one of its steps sums a gradient over all $m$ instances, and a method that needs the full dataset per step is no better off than a closed form when the dataset does not fit. In scikit-learn the qualifying estimators are the ones exposing `partial_fit`, which the user guide's page on scaling computationally lists; `SGDClassifier` and `SGDRegressor` are both on it. It is the answer when [[Batch Learning]] is impossible for size reasons rather than for freshness reasons.
