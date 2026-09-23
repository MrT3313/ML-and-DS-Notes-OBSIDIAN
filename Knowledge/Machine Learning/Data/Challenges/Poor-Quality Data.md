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
  - "[[DMLS Ch03 Data Engineering Fundamentals]]"
  - "[[DMLS Ch05 Feature Engineering]]"
confidence: draft
---

## Definition

Poor-quality data contains errors, outliers, noise, and missing values that hide the real patterns and pull the model toward artefacts. Géron's claim, which I accept: most of a practitioner's time goes to cleaning it.

## Formal statement

The observed value is $\tilde{\mathbf{x}}^{(i)} = \mathbf{x}^{(i)} + \boldsymbol\epsilon^{(i)}$ or has components missing; a model that fits $\tilde{\mathbf{x}}$ closely fits $\boldsymbol\epsilon$, which is one route to [[Overfitting]].

## Where it is used

Remedies are per-defect: drop or fix outliers; for a [[Feature]] with missing values, drop the feature, drop the instances, or impute (median fill, or train with and without); for duplicates and near-duplicates, which arrive both from the collection itself and from merging several sources that overlap, deduplicate before the split rather than after, since what they cost is a contaminated split rather than a distorted column ([[Data Leakage]]). [[HOML Ch02 End-to-End Machine Learning Project|HOML chapter 2]] gives the pipeline. Closely related challenges are [[Irrelevant Features]] and [[Nonrepresentative Training Data]]. [[Data Source]] is the upstream account of where the defects come from, since the origins a record can arrive from differ in how likely they are to arrive wrong and in who is answerable for it.
