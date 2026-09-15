---
note_kind: concept
aliases:
  - distribution shift
  - dataset shift
  - train-serving skew
  - train-dev set
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## Definition

Data mismatch is when the training data comes from a different distribution than the data the model meets in production, so a model that looks good on a test set drawn from the training source fails on real inputs. It can happen even with a large, clean training set.

## Formal statement

Training draws from $p_{\text{train}}(\mathbf{x}, y)$, production from $p_{\text{prod}}(\mathbf{x}, y)$, and $p_{\text{train}} \ne p_{\text{prod}}$. A test set drawn from $p_{\text{train}}$ then estimates the wrong quantity. Géron's diagnostic: hold out a train-dev set from the training source; if the model does well there but badly on the production-drawn validation set, the problem is mismatch, not [[Overfitting]].

## Where it is used

The [[Testing Set]] and validation set must be drawn from production-like data or their estimates are meaningless. Géron's example is training on web images of flowers and deploying on phone photos. [[Model Rot]] is the same mechanism arising over time after deployment rather than at collection; [[Nonrepresentative Training Data]] is the sampling-side cause.
