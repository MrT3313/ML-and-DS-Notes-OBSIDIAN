---
note_kind: concept
aliases:
  - hyperparameter tuning
  - hyperparameter search
  - model comparison
up: "[[Generalization]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

Model selection is choosing between candidate models, including the same model family under different [[Hyperparameter]] settings, by comparing their estimated [[Generalization]] error. It has to be done on data the candidates did not train on and that is not the [[Testing Set]].

## Formal statement

Given candidates $c \in \mathcal{C}$ with validation scores $\hat{\mathcal{L}}_{\text{val}}(c)$,

$$c^{*} = \arg\min_{c \in \mathcal{C}} \hat{\mathcal{L}}_{\text{val}}(c)$$

The estimate comes from [[Holdout Validation]] or [[Cross-Validation]]. Because the test set was not touched, its score for $c^{*}$ remains an unbiased generalization estimate.

## Where it is used

It is the loop that balances [[Overfitting]] against [[Underfitting]]. [[Grid Search]] and [[Randomized Search]] are the standard ways of enumerating $\mathcal{C}$. The failure Géron warns about: selecting on the test set, which makes the reported score a selection artifact.

$\arg\min$ over $\mathcal{C}$ answers only which candidate in the set is ahead, and it is silent about whether the set is the right set. The complementary diagnostic is a [[Learning Curve]], run on a single candidate rather than across them: it says whether the next move is more data or a different model family, which is the question you have to settle before enlarging $\mathcal{C}$ is worth the fits.
