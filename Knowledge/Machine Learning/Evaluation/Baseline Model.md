---
note_kind: concept
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

## Definition

A baseline model makes its predictions from simple fixed rules and never looks at the features: it answers from the class distribution, from chance, or from a constant. It exists to give every real candidate a floor to clear. A score is not good or bad on its own, it is good or bad relative to what you get for free, and the baseline is what "for free" measures.

## Formal statement

Let the training set hold $m$ instances with $m_k$ in class $k$. A constant predictor that always returns the majority class is right exactly when the true label is that class, so its accuracy is the majority class frequency:

$$\text{acc}_{\text{majority}} = \max_k \frac{m_k}{m}$$

This is the formula that explains the MNIST 5-detector. Of the $60{,}000$ MNIST training images, $5{,}421$ are fives and $54{,}579$ are not, so

$$\text{acc}_{\text{majority}} = \frac{54{,}579}{60{,}000} = 0.90965$$

and a model that has learned nothing whatsoever scores above 90%. A [[Stochastic Gradient Descent Classifier]] cross-validates to $[0.95035, 0.96035, 0.9604]$ on the same target, so its real achievement is about five points over a constant, not the ninety-five points the raw number suggests. The general reading: on a two-class problem the informative quantity is not $\text{acc}$ but the gap $\text{acc} - \max_k m_k/m$, and the room available for that gap shrinks to nothing as the majority share approaches $1$.

Other fixed rules give other floors. A predictor that draws class $k$ with probability $\pi_k = m_k/m$, independently of the input, matches the truth with probability

$$\text{acc}_{\text{stratified}} = \sum_k \pi_k^{2}$$

which is lower than the majority rule whenever more than one class has mass. For [[Regression]], the constant that minimizes squared error is the training mean $\bar{y}$, and its mean squared error is the target's variance $\frac{1}{m}\sum_i (y^{(i)} - \bar{y})^{2}$, which is why an $R^{2}$ of $0$ means "no better than predicting the mean".

### In scikit-learn

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

The default `strategy` is `"prior"`, and its `predict` returns the most frequent training label, which is what makes `any(...)` come back `False` here. The five allowed strategies:

| strategy | what `predict` returns |
|---|---|
| `"prior"` (default) | the most frequent class. `predict_proba` returns the empirical class distribution $\pi_k$ |
| `"most_frequent"` | the most frequent class. `predict_proba` returns the matching one-hot vector instead |
| `"stratified"` | a class drawn at random from the empirical distribution $\pi_k$, independently per row |
| `"uniform"` | a class drawn uniformly from the observed classes |
| `"constant"` | the single label you pass as `constant`, which is how you baseline a metric on a non-majority class |

`"prior"` and `"most_frequent"` differ only in `predict_proba`, so any comparison made on accuracy alone cannot tell them apart. `random_state` matters for `"stratified"` and `"uniform"` only, the two that actually draw; see [[Random Seed]].

`DummyRegressor` is the regression counterpart, with `strategy="mean"` by default and `"median"`, `"quantile"` and `"constant"` available.

## Where it is used

A baseline is the first entry in [[Model Selection]]: a candidate that cannot beat it is not a candidate, and knowing the floor before you start stops you from shipping a model whose whole score is the class distribution. It is scored exactly the way every other candidate is, through [[Cross-Validation]] on the same folds with the same [[Performance Measure]], because a floor measured differently from the thing standing on it proves nothing.

Its sharpest use is as the instrument that exposes [[Accuracy]] as misleading. Under [[Class Imbalance]] the constant predictor scores $\max_k m_k/m$, which can be very high, and putting that number next to a real model's is the cheapest way to make the problem visible without yet reaching for a [[Confusion Matrix]]. It also sets the floor for [[Classification]] metrics generally: a `"stratified"` dummy has [[Precision]] equal to the positive class prior and [[Recall]] equal to that same prior, which is the horizontal line a precision-recall curve has to sit above to mean anything.
