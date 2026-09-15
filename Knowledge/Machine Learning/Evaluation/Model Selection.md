---
note_kind: concept
aliases:
  - hyperparameter tuning
  - hyperparameter search
  - model comparison
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Model selection is choosing between candidate models, including the same model family under different [[Hyperparameter]] settings, by comparing their estimated [[Generalization]] error. It has to be done on data the candidates did not train on and that is not the [[Testing Set]].

## Formal statement

Given candidates $c \in \mathcal{C}$ with validation scores $\hat{\mathcal{L}}_{\text{val}}(c)$,

$$c^{*} = \arg\min_{c \in \mathcal{C}} \hat{\mathcal{L}}_{\text{val}}(c)$$

The estimate comes from [[Holdout Validation]] or [[Cross-Validation]]. Because the test set was not touched, its score for $c^{*}$ remains an unbiased generalization estimate.

## Where it is used

It is the loop that balances [[Overfitting]] against [[Underfitting]]. Chapter 2 adds grid and random search over $\mathcal{C}$. The failure Géron warns about: selecting on the test set, which makes the reported score a selection artifact.
