---
note_kind: concept
aliases:
  - test data
  - holdout test set
  - train-test split
  - test-set
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

The test set is a slice of the data, set aside before any training, used exactly once to estimate how the final model will perform on data it has never seen. Its only job is to estimate [[Generalization]] error honestly.

## Formal statement

Split $D$ into $D_{\text{train}}$ of size $m_{\text{train}}$ and $D_{\text{test}}$ of size $m_{\text{test}}$ with $D_{\text{train}} \cap D_{\text{test}} = \emptyset$; Géron's default is 80/20, with a smaller test fraction when $m$ is large. The estimate is

$$\hat{\mathcal{L}}_{\text{gen}} = \frac{1}{m_{\text{test}}} \sum_{i \in D_{\text{test}}} \ell\big(h(\mathbf{x}^{(i)}), y^{(i)}\big)$$

## Where it is used

Comparing $\hat{\mathcal{L}}_{\text{gen}}$ with training error diagnoses [[Overfitting]]. It must not be used to choose between models or tune a [[Hyperparameter]]; that is what [[Holdout Validation]] and [[Cross-Validation]] are for, and using the test set for it produces a model tuned to the test set and an optimistic estimate. If the test set does not match production data, see [[Data Mismatch]].
