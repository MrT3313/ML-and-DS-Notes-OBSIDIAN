---
note_kind: concept
aliases:
  - data quality
  - data cleaning
  - noisy data
  - dirty data
  - poor quality data
up: "[[Training Set]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Poor-quality data contains errors, outliers, noise, and missing values that hide the real patterns and pull the model toward artefacts. Géron's claim, which I accept: most of a practitioner's time goes to cleaning it.

## Formal statement

The observed value is $\tilde{\mathbf{x}}^{(i)} = \mathbf{x}^{(i)} + \boldsymbol\epsilon^{(i)}$ or has components missing; a model that fits $\tilde{\mathbf{x}}$ closely fits $\boldsymbol\epsilon$, which is one route to [[Overfitting]].

## Where it is used

Remedies are per-defect: drop or fix outliers; for a [[Feature]] with missing values, drop the feature, drop the instances, or impute (median fill, or train with and without). Chapter 2 gives the pipeline. Closely related challenges are [[Irrelevant Features]] and [[Nonrepresentative Training Data]].
