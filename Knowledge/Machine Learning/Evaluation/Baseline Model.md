---
note_kind: method
aliases:
  - baseline
  - baselines
  - baseline model
  - baseline classifier
  - dummy classifier
  - Dummy Classifier
  - DummyClassifier
  - DummyRegressor
  - dummy model
  - dummy baseline
  - dummy regressor
  - trivial baseline
up: "[[Model Selection]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## What it does and when

A baseline model makes its predictions from simple fixed rules and never looks at the features: it answers from the class distribution, from chance, or from a constant. It exists to give every real candidate a floor to clear. A score is not good or bad on its own, it is good or bad relative to what you get for free, and the baseline is what "for free" measures.

It is fitted before any real model, as the first entry in [[Model Selection]]: a candidate that cannot beat it is not a candidate, and knowing the floor before you start stops you from shipping a model whose whole score is the class distribution. It is scored exactly the way every other candidate is, through [[Cross-Validation]] on the same folds with the same [[Performance Measure]], because a floor measured differently from the thing standing on it proves nothing. Reach for it whenever a score is about to be reported with nothing to compare it against, which is every first fit on a new target.

Its sharpest use is as the instrument that exposes [[Accuracy]] as misleading. Under [[Class Imbalance]] the constant predictor scores $\max_k m_k/m$, which can be very high, and putting that number next to a real model's is the cheapest way to make the problem visible without yet reaching for a [[Confusion Matrix]]. It also sets the floor for [[Classification]] metrics generally: a `"stratified"` dummy has [[Precision]] equal to the positive class prior and [[Recall]] equal to that same prior, which is the horizontal line a precision-recall curve has to sit above to mean anything.

## Algorithm or formula

Fitting records the training label distribution and nothing else. `predict` then returns, for every row, whatever the chosen `strategy` names, and the input row is never read.

For classification, let the training set hold $m$ instances with $m_k$ in class $k$, and write $\pi_k = m_k/m$ for the empirical class distribution. `DummyClassifier` has five strategies:

| strategy | what `predict` returns |
|---|---|
| `"prior"` (default) | the most frequent class. `predict_proba` returns the empirical class distribution $\pi_k$ |
| `"most_frequent"` | the most frequent class. `predict_proba` returns the matching one-hot vector instead |
| `"stratified"` | a class drawn at random from the empirical distribution $\pi_k$, independently per row |
| `"uniform"` | a class drawn uniformly from the observed classes |
| `"constant"` | the single label you pass as `constant`, which is how you baseline a metric on a non-majority class |

`"prior"` and `"most_frequent"` differ only in `predict_proba`, so any comparison made on accuracy alone cannot tell them apart.

For [[Regression]], `DummyRegressor` returns one number for every row: the training mean $\bar{y}$ under `"mean"` (the default), the training median under `"median"`, the training quantile $q$ you name under `"quantile"`, and the value you pass under `"constant"`.

Each rule sets a floor that can be written down. A constant predictor that always returns the majority class is right exactly when the true label is that class, so its accuracy is the majority class frequency:

$$\text{acc}_{\text{majority}} = \max_k \frac{m_k}{m}$$

This is the formula that explains the MNIST 5-detector. Of the $60{,}000$ MNIST training images, $5{,}421$ are fives and $54{,}579$ are not, so

$$\text{acc}_{\text{majority}} = \frac{54{,}579}{60{,}000} = 0.90965$$

and a model that has learned nothing whatsoever scores above 90%. A [[Stochastic Gradient Descent Classifier]] cross-validates to $[0.95035, 0.96035, 0.9604]$ on the same target, so its real achievement is about five points over a constant, not the ninety-five points the raw number suggests. The general reading: on a two-class problem the informative quantity is not $\text{acc}$ but the gap $\text{acc} - \max_k m_k/m$, and the room available for that gap shrinks to nothing as the majority share approaches $1$.

A predictor that draws class $k$ with probability $\pi_k$, independently of the input, matches the truth with probability

$$\text{acc}_{\text{stratified}} = \sum_k \pi_k^{2}$$

which is lower than the majority rule whenever more than one class has mass. For regression, the constant that minimizes squared error is the training mean $\bar{y}$, and its mean squared error is the target's variance $\frac{1}{m}\sum_i (y^{(i)} - \bar{y})^{2}$, which is why an $R^{2}$ of $0$ means "no better than predicting the mean". The constant that minimizes absolute error is the training median, which is why `"median"` is the floor that matches a mean absolute error comparison.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `strategy` | | `"prior"` for the classifier, `"mean"` for the regressor | Categorical, so no direction. Each value is a different fixed rule and therefore a different floor: the majority rule scores $\max_k \pi_k$ on accuracy, the stratified rule $\sum_k \pi_k^{2}$, and the mean and median each minimize a different regression loss | Not tuned. Chosen to match the metric being floored: majority or mean for accuracy and squared error, `"median"` for absolute error, `"stratified"` for precision and recall, `"constant"` to floor a metric on a class the majority rule never predicts |
| `constant` | $c$ | none | Every prediction moves to the new value, so the floor becomes whatever the metric awards that one label or number | Set to the class or value whose floor you want. Read only when `strategy="constant"` |
| `quantile` | $q$ | none | Slides the returned value up through the training distribution, from the minimum at $q = 0$ to the maximum at $q = 1$, with $q = 0.5$ reproducing `"median"` | Set to the quantile the loss is asymmetric about. Read only when `strategy="quantile"` |

`random_state` is left out on purpose: under `"stratified"` and `"uniform"`, the two strategies that draw, it fixes which class each prediction lands on, so it changes the predictions that come back, but it is pinned rather than tuned. Fix an integer, see [[Random Seed]].

## Failure modes

- A majority-class baseline that scores 90 percent accuracy on a 90/10 split and looks like a model. The MNIST 5-detector's dummy scores $0.90965$, and a report that shows the candidate's $0.96$ without that number beside it, or that number without the candidate's, reads as success either way. The floor has to be reported next to the candidate, as the gap $\text{acc} - \max_k m_k/m$, or it has done nothing.
- A `"stratified"` or `"uniform"` baseline whose score varies run to run because `random_state` was left at `None`. The floor moves under the candidate, and a candidate that clears it in one run and misses it in the next says nothing about the candidate. Fix the seed, or use `"prior"` or `"most_frequent"`, which do not draw.
- A regression baseline compared on RMSE against a model tuned on MAE. The default `"mean"` is the constant that minimizes squared error, and the median is the one that minimizes absolute error, so a model can beat the mean floor on MAE merely because the floor was aimed at a different loss. The baseline and the candidate share one [[Performance Measure]], and the strategy is chosen to be the optimal constant for that measure.
- A `"prior"` baseline used to floor precision or recall on the minority class. It never predicts the positive label, so its recall is $0$ and its precision is undefined, and a floor of zero is cleared by any model whatsoever. Floor those metrics with `"stratified"`, whose precision and recall both equal the positive class prior, or with `"constant"` set to the positive class.

## Implementation

scikit-learn 1.6:

```python
from sklearn.dummy import DummyClassifier
from sklearn.model_selection import cross_val_score

dummy_clf = DummyClassifier()
dummy_clf.fit(X_train, y_train_5)
print(any(dummy_clf.predict(X_train)))   # False: it never predicts a 5

cross_val_score(dummy_clf, X_train, y_train_5, cv=3, scoring="accuracy")
# array([0.90965, 0.90965, 0.90965])
```

The default `strategy` is `"prior"`, and its `predict` returns the most frequent training label, which is what makes `any(...)` come back `False` here. `DummyRegressor` is the regression counterpart, with `strategy="mean"` by default and `"median"`, `"quantile"` and `"constant"` available:

```python
from sklearn.dummy import DummyRegressor

dummy_reg = DummyRegressor(strategy="median")
dummy_reg.fit(X_train, y_train)
dummy_reg.predict(X_train[:3])   # the training median, three times
```
