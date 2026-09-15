---
note_kind: concept
aliases:
  - feature engineering
  - feature selection
  - feature extraction
  - garbage in garbage out
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Irrelevant features are input columns that carry no information about the target. They slow training, give the model noise to fit, and worsen [[Generalization]]. The remedy is feature engineering: keep the useful features, combine existing ones into more useful ones, and gather new ones.

## Formal statement

A [[Feature]] $x_j$ is irrelevant when $y \perp x_j \mid \mathbf{x}_{-j}$, so no model gains from it. Feature engineering is a map $\phi: \mathbb{R}^{n} \to \mathbb{R}^{n'}$ chosen so that $y$ is easier to predict from $\phi(\mathbf{x})$. Not further quantitative at this depth.

## Where it is used

Three sub-tasks from the chapter: feature selection (choose among existing), feature extraction (combine, including [[Dimensionality Reduction]]), and feature creation (collect new data). Interacts with [[Overfitting]]: more irrelevant columns give a flexible model more noise to memorize.
