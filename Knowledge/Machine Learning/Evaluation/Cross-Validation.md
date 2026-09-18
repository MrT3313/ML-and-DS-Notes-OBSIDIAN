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
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## What it does and when

Cross-validation splits the training set into $K$ folds and repeats [[Holdout Validation]] $K$ times, each fold serving once as the validation set. The $K$ scores are averaged. Use it when a single split would be too noisy to trust, at the cost of training $K$ models per candidate.

Its first job is diagnostic, and it catches what a single training score cannot. A decision tree fitted on the housing training data scores an RMSE of exactly $0$ on that same data, which looks like a perfect model and is in fact the signature of [[Overfitting]]; cross-validating the identical estimator returns a mean RMSE of about $66{,}900$, worse than the linear baseline's $69{,}900$ was ever suspected of being. The gap between the two numbers is the finding. A random forest, an [[Ensemble Learning|ensemble]] of many such trees, narrows it without closing it, $17{,}500$ on the training set against $47{,}000$ under cross-validation, so it is still overfitting, merely usefully.

### What it certifies and what it does not

The same call on a classifier, `cross_val_score(sgd_clf, X_train, y_train_5, cv=3, scoring="accuracy")`, shows the limit of what the procedure vouches for. The [[MNIST]] 5-detector comes back at roughly $95$ percent on every one of the three folds, which looks decisive until a [[Baseline Model]] that always answers "not a 5" is put through the identical procedure and scores about $91$ percent. Nothing malfunctioned: cross-validation reported a number that genuinely generalizes, and the number still did not mean what it looked like, because [[Class Imbalance]] had already handed ninety-one of those points to a model that learned nothing at all. Cross-validation checks that a score generalizes. It cannot check that the score is the right score, and picking the right one is the job of the [[Performance Measure]].

### Out-of-fold predictions

`cross_val_predict` performs the same split but returns out-of-fold *predictions* rather than fold scores. Every instance is predicted by a model fitted on the folds that excluded it, so no prediction comes from an estimator that saw the instance during fitting. That is precisely what makes a [[Confusion Matrix]] computed over the training set honest, a measure of [[Generalization]] rather than of memorisation. Passing `method="decision_function"` returns the out-of-fold *scores* instead of the labels, and those scores are what the [[Precision-Recall Tradeoff]] and the [[ROC Curve]] are swept over.

## Algorithm

1. Partition $D_{\text{train}}$ into folds $F_1, \dots, F_K$ of near-equal size.
2. Hold out fold $F_k$ as the validation set for this round.
3. Fit $h_{-k}$ on the remaining folds, $D_{\text{train}} \setminus F_k$, preprocessing included, so nothing is learned from $F_k$.
4. Score $h_{-k}$ on $F_k$, recording $\mathcal{L}(h_{-k}, F_k)$.
5. Repeat steps 2 to 4 for every $k = 1, \dots, K$, so each fold is held out exactly once and each instance is scored exactly once.
6. Aggregate the $K$ fold scores into the estimate, keeping the standard deviation alongside the mean:

   $$\hat{\mathcal{L}}_{\text{CV}} = \frac{1}{K} \sum_{k=1}^{K} \mathcal{L}(h_{-k}, F_k)$$

7. Choose the candidate with the lowest $\hat{\mathcal{L}}_{\text{CV}}$, refit on all of $D_{\text{train}}$, evaluate once on the [[Testing Set]]. Wrapping a loop over candidate settings around this whole procedure is exactly what [[Grid Search]] and [[Randomized Search]] are.

### Reading the output as a distribution

The output is a distribution, not a point. Ten tree scores on the housing data carry a standard deviation of roughly $2{,}100$, and the linear model's roughly $4{,}200$, so reporting only the mean discards the information that would tell you whether a win is real. A candidate that leads on the mean while spreading twice as wide has not clearly beaten anything, and with a small $K$ the spread can easily exceed the gap being argued about.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| number of folds | $K$ | 5 or 10 | each model sees more data, estimate less biased, cost grows linearly, fold scores more correlated | 5 for large $m$, 10 for small, $K = m$ is leave-one-out |

## Failure modes

- Compute: $K$ fits per candidate per hyperparameter setting.
- Leakage across folds when instances are not independent (time series, grouped data, duplicate rows) gives an optimistic estimate.
- Preprocessing fitted outside the folds. The estimator passed in must be the complete [[Pipeline]], preprocessing included, so that the imputer and the scaler are refit on each training fold. Fitting them once on the whole training set before the split lets every validation fold contribute to the transformer that is then evaluated on it, which is [[Data Snooping Bias]] in miniature.
- Folds that do not preserve class balance in [[Classification]] give noisy scores, worst of all under [[Class Imbalance]], where a rare class can come out thin or missing in a fold. In scikit-learn 1.6 this particular case is already handled rather than left to the caller: for an integer or `None` `cv`, `cross_val_score` and `cross_validate` split with `StratifiedKFold` when the estimator is a classifier and `y` is binary or multiclass, and with plain `KFold` in every other case. What they do not do is shuffle. Both splitters are instantiated with `shuffle=False`, so the folds are contiguous blocks of the row order and repeat identically across calls; a file sorted by class, by time, or by any other structure needs an explicit splitter with `shuffle=True` and a [[Random Seed]], because stratification is no defence against ordering.

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

`cross_val_predict` takes the same arguments and the same folds but hands back one prediction per instance rather than one score per fold, which is what every confusion matrix and threshold sweep over the training set is computed from.

scikit-learn 1.6:

```python
from sklearn.model_selection import cross_val_predict

y_train_pred = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3)

y_scores = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3,
                             method="decision_function")
```

`method` names which of the estimator's response methods to call per fold, so `"predict_proba"` is the substitute for an estimator that has no `decision_function`.

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
