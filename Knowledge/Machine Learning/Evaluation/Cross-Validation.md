---
note_kind: method
aliases:
  - CV
  - k-fold cross-validation
  - K-fold
  - cross validation
up:
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## What it does and when

Cross-validation splits the training set into $K$ folds and repeats [[Holdout Validation]] $K$ times, each fold serving once as the validation set. The $K$ scores are averaged. Use it when a single split would be too noisy to trust, at the cost of training $K$ models per candidate.

## Algorithm

Partition $D_{\text{train}}$ into folds $F_1, \dots, F_K$ of near-equal size. For each $k$, fit $h_{-k}$ on $D_{\text{train}} \setminus F_k$ and score it on $F_k$:

$$\hat{\mathcal{L}}_{\text{CV}} = \frac{1}{K} \sum_{k=1}^{K} \mathcal{L}(h_{-k}, F_k)$$

Choose the candidate with the lowest $\hat{\mathcal{L}}_{\text{CV}}$, refit on all of $D_{\text{train}}$, evaluate once on the [[Testing Set]].

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| number of folds | $K$ | 5 or 10 | each model sees more data, estimate less biased, cost grows linearly, fold scores more correlated | 5 for large $m$, 10 for small, $K = m$ is leave-one-out |

## Failure modes

- Compute: $K$ fits per candidate per hyperparameter setting.
- Leakage across folds when instances are not independent (time series, grouped data, duplicate rows) gives an optimistic estimate.
- Folds that do not preserve class balance in [[Classification]] give noisy scores; stratify.

## Implementation

scikit-learn 1.6:

```python
from sklearn.model_selection import cross_val_score
scores = cross_val_score(model, X_train, y_train, cv=5, scoring="neg_mean_squared_error")
```

```base
filters:
  and:
    - file.hasLink(this.file)
    - file.inFolder("Knowledge")
views:
  - type: table
    name: Linked here
    order:
      - file.name
      - note_kind
      - confidence
```
