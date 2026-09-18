---
note_kind: concept
aliases:
  - irrelevant feature
  - garbage in garbage out
  - garbage in, garbage out
  - GIGO
up: "[[Feature]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Irrelevant features are input columns that carry no information about the target. They slow training, give the model noise to fit, and worsen [[Generalization]]. The remedy is feature engineering: keep the useful features, combine existing ones into more useful ones, and gather new ones.

## Formal statement

A [[Feature]] $x_j$ is irrelevant when $y \perp x_j \mid \mathbf{x}_{-j}$, so no model gains from it. Feature engineering is a map $\phi: \mathbb{R}^{n} \to \mathbb{R}^{n'}$ chosen so that $y$ is easier to predict from $\phi(\mathbf{x})$.

**Not further quantitative at this depth.**

An irrelevant feature is a data pathology, and the conditional independence above is the whole of its mathematics at this depth: it says when a column is useless, not how to detect that from a finite sample. A later chapter would add the selection criteria that operationalize it, ex a mutual information score or an L1 penalty that drives the column's weight to zero.

## Where it is used

Feature engineering splits into three sub-tasks: feature selection (choose among existing), feature extraction (combine, including [[Dimensionality Reduction]]), and feature creation (collect new data). Interacts with [[Overfitting]]: more irrelevant columns give a flexible model more noise to memorize.
