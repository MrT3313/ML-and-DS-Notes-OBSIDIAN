---
note_kind: index
aliases:
  - ML
up: "[[Home]]"
---

The root of the machine learning domain. Everything under `Knowledge/Machine Learning/` hangs off this note. Its job is to hold the definitions the rest of the vault leans on, and to name the areas so nothing gets filed by accident.

> [!quote] Definition(s)
>Machine learning is the science (and art) of programming computers so they can learn from data. ~[[HOML Ch01 The Machine Learning Landscape|HOML Chapter 1]]
>
>Machine Learning is the field of study that gives computers the ability to learn without being explicitly programmed. ~Arthur Samuel, 1959
> 
>A computer program is said to learn from experience $E$ with respect to some task $T$ and some performance measure $P$, if its performance on $T$, as measured by $P$, improves with experience $E$. ~Tom Mitchell, 1977

Mitchell's $E$, $T$ and $P$ are the three hooks the rest of the domain hangs on: $E$ is the [[Training Set]], $T$ is one of the tasks listed under Areas below, and $P$ is the [[Performance Measure]].

## Areas

- **`_Foundations/`** the vocabulary every other note assumes: [[Model]], [[Feature]], [[Hyperparameter]], [[Training Set]], [[Training Instance]].
- **`Paradigms/`** the three independent axes a system can be placed on. Supervision ([[Supervised Learning]] through [[Unsupervised Learning]]), training regime ([[Batch Learning]] versus [[Online Learning]]), and representation ([[Instance-Based Learning]] versus [[Model-Based Learning]]).
- **`Tasks/`** what the system is asked to do: [[Classification]], [[Regression]], [[Clustering]], [[Dimensionality Reduction]], [[Anomaly Detection]], [[Novelty Detection]], [[Association Rule Learning]].
- **`Models/`** the algorithm families, one folder per family. Currently `Linear/` only.
- **`Objectives/`** what training optimizes: [[Performance Measure]] and its two sign conventions, [[Cost Function]] and [[Utility Function]].
- **`Optimization/`** the mechanics of fitting. [[Learning Rate]] today, gradient descent and regularization from chapter 4.
- **`Evaluation/`** estimating [[Generalization]] honestly: [[Testing Set]], [[Holdout Validation]], [[Cross-Validation]], [[Model Selection]], and the two failures, [[Overfitting]] and [[Underfitting]].
- **`Data/`** everything about the input. `Challenges/` holds the six ways data defeats a model.

Operational concerns sit outside this domain, in `Knowledge/MLOps/`. [[Model Rot]] is the first of them.

## What is missing

- Reinforcement learning has no note and no folder; HOML chapter 1 covers it.
- Regularization and the no free lunch theorem are named in chapter 1 and written nowhere.
- `Models/` holds one family. Trees, ensembles, kernel methods, neighbours, and neural networks all arrive in later chapters.
- No metrics notes yet. Chapter 3 fills `Evaluation/`.
