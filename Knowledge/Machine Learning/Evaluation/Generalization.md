---
note_kind: concept
aliases:
  - generalization error
  - generalization gap
  - out-of-sample error
  - generalize
up: "[[Machine Learning]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

Generalization is a model's ability to perform well on instances it did not train on. It is the actual goal of training; low error on the training set is only a means to it.

## Formal statement

Generalization error is the expected loss on the data distribution, estimated on a held-out [[Testing Set]]:

$$\mathcal{L}_{\text{gen}} = \mathbb{E}_{(\mathbf{x}, y) \sim p}\big[\ell(h(\mathbf{x}), y)\big] \approx \frac{1}{m_{\text{test}}} \sum_{i=1}^{m_{\text{test}}} \ell\big(h(\mathbf{x}^{(i)}), y^{(i)}\big)$$

The generalization gap is $\mathcal{L}_{\text{gen}} - \mathcal{L}_{\text{train}}$. A large gap is [[Overfitting]]; both terms high is [[Underfitting]].

Under squared-error loss $\mathcal{L}_{\text{gen}}$ itself splits into three terms with different causes, squared bias, variance and an irreducible noise floor. That is the [[Bias-Variance Tradeoff]], and it is what turns the two failure names above from symptoms into diagnoses, since the terms respond to different remedies. The gap is watched empirically with a [[Learning Curve]], which plots both errors against training set size and so shows whether the gap is still closing.

## Where it is used

This is the folder's anchor. Everything here is about estimating it ([[Testing Set]], [[Holdout Validation]], [[Cross-Validation]]), choosing models by it ([[Model Selection]]), or the ways it fails ([[Insufficient Training Data]], [[Data Mismatch]]). The two strategies for achieving it are [[Instance-Based Learning]] and [[Model-Based Learning]].
