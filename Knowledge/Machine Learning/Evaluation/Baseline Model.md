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
  - zero rule
  - zero rule baseline
  - random baseline
  - simple heuristic
  - human baseline
  - existing solution
up: "[[Model Selection]]"
sources:
  - "[[HOML Ch03 Classification]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

An evaluation metric on its own means very little. A number is not good or bad by itself, it is good or bad relative to something, so the first question to ask about any score about to be reported is what baseline it is being evaluated against, and a baseline is whatever answers that question: the figure a candidate's score gets put next to so that it can be read at all.

The cheapest answer, and the one this note's formulas and code are about, is a fitted baseline model. It makes its predictions from simple fixed rules and never looks at the features: it answers from the class distribution, from chance, or from a constant. It exists to give every real candidate a floor to clear, and what it measures is what you get for free. Three other things are baselines too and none of them is fitted, so they carry no strategy and no code, and they are set out below.

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

### Baselines you do not fit

Everything above is `DummyClassifier` and `DummyRegressor`, and two of the five baselines worth naming are exactly those strategies under other names. A **random baseline** predicts at random following a distribution, which is `"stratified"` when the distribution is the empirical one and `"uniform"` when it is even, and its accuracy is the $\sum_k \pi_k^{2}$ already derived. A **zero rule baseline** always predicts the most common class, which is `"prior"` and `"most_frequent"`, and its accuracy is the $\max_k m_k/m$ already derived. Neither of those two adds content here; they add names.

The other three are not fitted, and the difference is structural rather than cosmetic. Nothing about them is estimated from the training labels, so there is no `fit`, no `strategy`, and for two of them no estimator at all.

| baseline | what the number measures | fitted from the training labels |
|---|---|---|
| random baseline | chance, under the empirical class distribution or an even one | yes, as `"stratified"` or `"uniform"` |
| zero rule baseline | the mode of the class distribution | yes, as `"prior"` or `"most_frequent"` |
| simple heuristic | domain knowledge, written down as a rule | no |
| human baseline | what people achieve on the same task | no, it is a measurement of annotators |
| existing solution | whatever is answering the question today | no, it is taken as given |

**A simple heuristic** is a prediction rule written from domain knowledge, with no fitting and no parameters estimated from data: predict the most recent value, predict the most popular item, predict positive when one column clears a threshold somebody chose. The zero rule is its degenerate member, the heuristic that uses no domain knowledge whatsoever, which is why it is the one member whose score can be written down without knowing anything about the problem at all. That makes the heuristic the general case and $\max_k m_k/m$ the floor of its least informative member rather than the floor as such.

The ordering matters because the general case is a much harder floor to clear, and that has been measured (Holte, "Very Simple Classification Rules Perform Well on Most Commonly Used Datasets", *Machine Learning* 11:63-91, 1993). What was measured is narrow and is worth stating narrowly. On 16 datasets taken from the April 1990 Irvine collection, each split at random into two thirds for training and one third for testing and averaged over 25 such splits, a rule reading a single attribute averaged $5.7$ accuracy percentage points below the pruned decision trees of C4 (C4.5 as distributed in May 1990). That average is skewed by two datasets, CH and SO: over the remaining 14 the gap is $3.1$ points, and on half of the 16 it is within $2.6$ points. The system producing the rules is called 1R, and it builds one candidate rule per attribute, discretizing numeric attributes into intervals first, then keeps the candidate with the lowest error on the training set.

Two findings there bear directly on what is above. Holte's own term for $\max_k m_k/m$ is baseline accuracy, the percentage of examples in the most frequently occurring class, and his argument is that it is too low on most of those datasets to be a useful benchmark at all, which is why he proposes the single-attribute rule as the standard a new result should be compared against instead. The zero rule is therefore the weakest floor in this note and not a demanding one. The second is what the result does not say: a one-attribute rule was hard to beat on the datasets the field happened to be using in 1993, which is a claim about those datasets rather than about simple rules in general, and the diagnosis of the two failures shows what makes it dataset-specific, since a single-attribute rule cannot separate more classes than its chosen attribute has values.

Both names are established and they come from different places. `OneR` is 1R carried into Weka as `weka.classifiers.rules.OneR`, with that paper as the cited reference and a minimum bucket size of 6 as the discretization default. `zero rule` is not Holte's: the word "zero" does not occur in that paper, and the name is a back-formation by analogy with 1R, established through Weka's `weka.classifiers.rules.ZeroR`, which predicts the mode for a nominal target and the mean for a numeric one and is therefore `DummyClassifier(strategy="most_frequent")` and `DummyRegressor(strategy="mean")` under an older name.

**A human baseline** is the score people get on the same task, and it is the only baseline here whose number comes from running an experiment on people rather than from a fit. It matters most where the point of the system is to automate something a person is doing now, because that is the case where a model losing to the person it would replace has no use whatever its margin over a dummy. It has no estimator, no hyperparameters and no code, and what stands in place of an implementation is a measurement protocol. [[Hand Labeling]] is that protocol, and the two steps in it that decide whether the number means anything are qualifying the annotators against a gold set and fixing the replication count in advance.

Fixing the replication is what makes the sharper point available, which is that a human baseline is a distribution and not a constant. [[Label Multiplicity]] is the reason: several people reading one unchanged instance return different answers, so the score has a spread over annotators, and a single quoted figure hides it. The measured case is the ImageNet challenge (Russakovsky and colleagues, "ImageNet Large Scale Visual Recognition Challenge", *International Journal of Computer Vision* 115, 2015), which used two expert annotators. A1 practised on 500 images, labelled 1500 test images, and reached $5.1\%$ top-5 error. A2 practised on 100 images, labelled 258, and reached $12.0\%$. On the 204 images both of them labelled, counting an image correct when either one got it right gives $2.4\%$, against GoogLeNet's $4.9\%$ on that same sample. So human top-5 error on ImageNet is anywhere from $2.4\%$ to $12.0\%$ depending on which person was asked and which rule combines them, and the $5.1\%$ that gets quoted is one annotator. What separates A1 from A2 is how much each had practised, which is the qualification step above, so the baseline moves with the protocol that produced it.

**An existing solution** is whatever answers the question today: a rule engine, a manual process, an older model. Its score is the only baseline here that is also a constraint rather than a reading, because that system exists and has users, so a candidate that beats a random draw and loses to the incumbent cannot ship however large its margin over the dummy is. [[Business Objective]] is where that bites, since the decision is made on a business metric for which the incumbent already has a measured value. Clearing it is also a comparison between vectors rather than between numbers, the structure [[Production Machine Learning]] derives: beating the incumbent on [[Accuracy]] while costing more to serve or taking longer to answer has not beaten it, because the requirements it already satisfies are part of what a replacement is required to satisfy.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `strategy` | | `"prior"` for the classifier, `"mean"` for the regressor | Categorical, so no direction. Each value is a different fixed rule and therefore a different floor: the majority rule scores $\max_k \pi_k$ on accuracy, the stratified rule $\sum_k \pi_k^{2}$, and the mean and median each minimize a different regression loss | Not tuned. Chosen to match the metric being floored: majority or mean for accuracy and squared error, `"median"` for absolute error, `"stratified"` for precision and recall, `"constant"` to floor a metric on a class the majority rule never predicts |
| `constant` | $c$ | none | Every prediction moves to the new value, so the floor becomes whatever the metric awards that one label or number | Set to the class or value whose floor you want. Read only when `strategy="constant"` |
| `quantile` | $q$ | none | Slides the returned value up through the training distribution, from the minimum at $q = 0$ to the maximum at $q = 1$, with $q = 0.5$ reproducing `"median"` | Set to the quantile the loss is asymmetric about. Read only when `strategy="quantile"` |

`random_state` is left out on purpose: under `"stratified"` and `"uniform"`, the two strategies that draw, it fixes which class each prediction lands on, so it changes the predictions that come back, but it is pinned rather than tuned. Fix an integer, see [[Random Seed]].

The table is about the fitted baselines, and the three that are not fitted have no rows in it and none anywhere. A human baseline and an existing solution have no hyperparameters at all, because nothing is fitted and there is nothing to set; what corresponds to a knob for the first is the labelling protocol that produced the number, the annotator qualification and the replication $r$ of [[Hand Labeling]], and those are that note's parameters and not this one's. A simple heuristic does carry numbers, the column, the threshold or the window its author picked, and they are set from domain knowledge rather than tuned, which is why they are named here instead of tabulated: the columns above ask how to tune them and the answer is that you do not. Tuning them against the data is how a heuristic stops being a baseline, since once its numbers were chosen by watching the score it is a candidate and needs a floor of its own.

## Failure modes

- A majority-class baseline that scores 90 percent accuracy on a 90/10 split and looks like a model. The MNIST 5-detector's dummy scores $0.90965$, and a report that shows the candidate's $0.96$ without that number beside it, or that number without the candidate's, reads as success either way. The floor has to be reported next to the candidate, as the gap $\text{acc} - \max_k m_k/m$, or it has done nothing.
- A `"stratified"` or `"uniform"` baseline whose score varies run to run because `random_state` was left at `None`. The floor moves under the candidate, and a candidate that clears it in one run and misses it in the next says nothing about the candidate. Fix the seed, or use `"prior"` or `"most_frequent"`, which do not draw.
- A regression baseline compared on RMSE against a model tuned on MAE. The default `"mean"` is the constant that minimizes squared error, and the median is the one that minimizes absolute error, so a model can beat the mean floor on MAE merely because the floor was aimed at a different loss. The baseline and the candidate share one [[Performance Measure]], and the strategy is chosen to be the optimal constant for that measure.
- A `"prior"` baseline used to floor precision or recall on the minority class. It never predicts the positive label, so its recall is $0$ and its precision is undefined, and a floor of zero is cleared by any model whatsoever. Floor those metrics with `"stratified"`, whose precision and recall both equal the positive class prior, or with `"constant"` set to the positive class.
- A human baseline quoted as a single number. A claim that people reach 95 percent on the task reads as a constant, and it is an average over however many annotators were asked, under whatever qualification they had and whatever rule combined them. Two expert annotators on ImageNet came in at $5.1\%$ and $12.0\%$ top-5 error on the same task, so a model reported as beating the human baseline may only be beating the less practised of two people. Report which protocol produced the figure and what the spread across annotators was, which is [[Label Multiplicity]], or report it as the one annotator it is.
- A model shipped because it cleared the dummy while losing to the system it was built to replace. The gap over $\max_k m_k/m$ can be wide and still be beside the point, because the number the project is judged on is the incumbent's and nothing in a [[Cross-Validation]] score against a `DummyClassifier` measures it. Score the existing solution on the same split and the same [[Performance Measure]] before the comparison against the dummy is reported, and where it cannot be scored that way say so, rather than letting the dummy stand in for it.

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

Three of the baselines have no such call. A simple heuristic is ordinary code, and the only reason it acquires an estimator wrapper at all is that scoring it on the same folds as everything else means handing it to the same scorer, which is what a floor is held to:

```python
import numpy as np
from sklearn.base import BaseEstimator, ClassifierMixin

class ThresholdHeuristic(BaseEstimator, ClassifierMixin):
    """Flag a row when one chosen column clears a threshold fixed from domain knowledge."""

    def __init__(self, col=0, threshold=0.0):
        self.col = col
        self.threshold = threshold

    def fit(self, X, y):
        self.classes_ = np.array([0, 1])   # fixed in advance, not learned
        return self

    def predict(self, X):
        return (np.asarray(X)[:, self.col] > self.threshold).astype(int)

cross_val_score(ThresholdHeuristic(col=3, threshold=2.5), X_train, y_train, cv=3)
```

`fit` reads neither `X` nor `y` and exists only because the scoring API insists on it, which is the whole difference between this and a `DummyClassifier`. A human baseline and an existing solution have no implementation at all: nothing is fitted and nothing is called, and what replaces code for them is a measurement protocol. For the first that is [[Hand Labeling]], qualification against a gold set and a replication count fixed in advance. For the second it is a scored run of the incumbent on the same split and the same [[Performance Measure]] as every candidate.
