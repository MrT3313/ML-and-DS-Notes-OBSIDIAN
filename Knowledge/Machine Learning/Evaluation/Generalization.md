---
note_kind: concept
aliases:
  - generalization error
  - generalization gap
  - out-of-sample error
  - generalize
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Generalization is a model's ability to perform well on instances it did not train on. It is the actual goal of training; low error on the training set is only a means to it.

## Formal statement

Generalization error is the expected loss on the data distribution, estimated on a held-out [[Testing Set]]:

$$\mathcal{L}_{\text{gen}} = \mathbb{E}_{(\mathbf{x}, y) \sim p}\big[\ell(h(\mathbf{x}), y)\big] \approx \frac{1}{m_{\text{test}}} \sum_{i=1}^{m_{\text{test}}} \ell\big(h(\mathbf{x}^{(i)}), y^{(i)}\big)$$

The generalization gap is $\mathcal{L}_{\text{gen}} - \mathcal{L}_{\text{train}}$. A large gap is [[Overfitting]]; both terms high is [[Underfitting]].

## Where it is used

This is the folder's anchor. Everything here is about estimating it ([[Testing Set]], [[Holdout Validation]], [[Cross-Validation]]), choosing models by it ([[Model Selection]]), or the ways it fails ([[Insufficient Training Data]], [[Data Mismatch]]). The two strategies for achieving it are [[Instance-Based Learning]] and [[Model-Based Learning]].
