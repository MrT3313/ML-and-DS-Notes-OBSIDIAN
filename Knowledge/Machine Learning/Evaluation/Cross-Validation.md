---
note_kind: method
aliases:
  - CV
  - k-fold cross-validation
  - K-fold
  - cross validation
up: "[[Holdout Validation]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Cross-validation splits the training set into $K$ folds and repeats [[Holdout Validation]] $K$ times, each fold serving once as the validation set. The $K$ scores are averaged. Use it when a single split would be too noisy to trust, at the cost of training $K$ models per candidate.

### From Chapter 2

Chapter 2 puts cross-validation to a diagnostic use that a single training score cannot serve. A decision tree fitted on the housing training data scores an RMSE of exactly $0$ on that same data, which looks like a perfect model and is in fact the signature of [[Overfitting]]; cross-validating the identical estimator returns a mean RMSE of about $66{,}900$, worse than the linear baseline's $69{,}900$ was ever suspected of being. The gap between the two numbers is the finding. A random forest, an [[Ensemble Learning|ensemble]] of many such trees, narrows it without closing it, $17{,}500$ on the training set against $47{,}000$ under cross-validation, so it is still overfitting, merely usefully.

The output is a distribution, not a point. Those ten tree scores carry a standard deviation of roughly $2{,}100$, and the linear model's roughly $4{,}200$, so reporting only the mean discards the information that would tell you whether a win is real. A candidate that leads on the mean while spreading twice as wide has not clearly beaten anything, and with a small $K$ the spread can easily exceed the gap being argued about.

The estimator passed in must be the complete [[Pipeline]], preprocessing included, so that the imputer and the scaler are refit on each training fold. Fitting them once on the whole training set before the split lets every validation fold contribute to the transformer that is then evaluated on it, which is [[Data Snooping Bias]] in miniature. Wrapping a loop over candidate settings around this whole procedure is exactly what [[Grid Search]] and [[Randomized Search]] are.

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

Every scikit-learn scorer obeys "higher is better", so error metrics are exposed negated. `scoring="neg_root_mean_squared_error"` returns minus the [[Root Mean Squared Error]], and a leading minus sign on the call turns the scores back into errors you can read:

```python
tree_rmses = -cross_val_score(tree_reg, housing, housing_labels,
                              scoring="neg_root_mean_squared_error", cv=10)
pd.Series(tree_rmses).describe()   # mean and std, not just the mean
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
