---
note_kind: concept
aliases:
  - Seed
  - random_state
  - seed
  - random seed
  - seeds
  - pseudorandom seed
  - RNG seed
  - np.random.seed
  - default_rng
up: "[[Random Sampling]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

The seed is the starting state handed to a pseudorandom number generator. Because the generator is a deterministic function of its state, fixing the seed fixes the entire stream of numbers that follows, and with it every choice built on that stream: a shuffle, a split, an initialization, a fit.

## Formal statement

A pseudorandom generator is a state $s_t$, a transition $s_{t+1} = f(s_t)$, and an output map $u_t = g(s_t)$ producing values that pass as uniform on $[0, 1)$. The seed sets $s_0$, so the whole sequence $u_0, u_1, u_2, \dots$ is a deterministic function of it. Nothing is random; the numbers are only unpredictable to someone who does not know $s_0$. Two runs with the same seed, the same library version, and the same call order produce identical output.

## Where it is used

Fixing the seed is what makes a [[Testing Set]] hold still across runs. Without it, [[Random Sampling]] draws a different split every execution, and over enough executions the model has trained on every row, which is [[Data Leakage]] reached by accident rather than by carelessness. [[Stratified Sampling]] takes the same argument for the same reason.

The failure mode worth stating plainly: a fixed seed makes a result reproducible, not correct. One seeded split can still be an unlucky split, and reporting its score as though the seed were irrelevant is reading noise as signal. [[Cross-Validation]] is the defence, since averaging over folds estimates how much of the score was the split. If a conclusion changes when the seed changes, the conclusion was never about the [[Model]].

In scikit-learn the argument is `random_state` and appears on every estimator and splitter with a stochastic component. Passing an integer gives a reproducible result; passing `None`, the default, falls back to shared global state, so the same call twice gives different answers. In NumPy the modern interface is a `Generator` object, `np.random.default_rng(seed)`, which keeps the stream local to the object. The legacy `np.random.seed(42)` mutates one process-wide state, so any library that draws from it can perturb your results at a distance.

> [!warning]
> Python's built-in `hash()` is salted per process for strings and bytes, so it is not reproducible across runs no matter what seed you set. `PYTHONHASHSEED` must be fixed before the interpreter starts. This is why a stable split hashes with `crc32` instead.
