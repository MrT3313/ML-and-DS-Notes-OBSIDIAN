---
note_kind: concept
aliases:
  - precision recall tradeoff
  - precision/recall tradeoff
  - precision-recall trade-off
  - precision/recall trade-off
  - decision threshold
  - classification threshold
  - score threshold
  - precision-recall curve
  - precision/recall curve
  - PR curve
  - precision_recall_curve
  - decision_function
up: "[[Performance Measure]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

Precision and recall are not two independent properties of a trained classifier. They are two readings taken at one setting of a single decision threshold, and moving that threshold moves both of them, in opposite directions on average. Neither number means anything quoted on its own, because the model was never asked to produce it: you produced it by choosing where to cut.

The vault owner's rule for hearing such a claim, and it is his own line: anytime someone says "Let's reach 99% precision", you should ask "At what recall?"

## Formal statement

A binary classifier does its work in two stages. A scoring function $s: \mathcal{X} \to \mathbb{R}$ assigns each instance a number, and a threshold $t$ turns that number into a label:

$$\hat{y}(\mathbf{x}; t) = \mathbb{1}\!\left[\, s(\mathbf{x}) > t \,\right]$$

Everything the model learned lives in $s$. The threshold $t$ is not fitted and carries no information about the data, and the whole point is that it is yours to set after training. A [[Stochastic Gradient Descent Classifier]] cuts at $t = 0$ on a signed distance to the hyperplane, and [[Logistic Regression]] cuts at $t = 0.5$ on a probability, but neither of those values is privileged by anything except convention.

### Recall is monotone in the threshold, precision is not

Let $\Pi(t) = \{\, i : s(\mathbf{x}^{(i)}) > t \,\}$ be the set of instances predicted positive. For $t' > t$ the condition is strictly harder, so

$$\Pi(t') \subseteq \Pi(t)$$

Raising the threshold can only remove instances from the predicted-positive set, never add any. Splitting that set by true label, both $TP(t) = |\Pi(t) \cap \mathcal{P}|$ and $FP(t) = |\Pi(t) \setminus \mathcal{P}|$ are therefore non-increasing functions of $t$, where $\mathcal{P}$ is the set of actual positives.

Recall inherits that directly. Its denominator is the count of actual positives, which is a property of the labels and does not depend on $t$ at all:

$$R(t) = \frac{TP(t)}{TP(t) + FN(t)} = \frac{TP(t)}{|\mathcal{P}|}$$

A non-increasing numerator over a constant denominator gives a monotonically non-increasing function. Raise the threshold and recall falls, or stays put, every single time.

Precision does not inherit it, and this is the part that is easy to miss:

$$P(t) = \frac{TP(t)}{TP(t) + FP(t)} = \frac{TP(t)}{|\Pi(t)|}$$

Numerator and denominator both shrink as $t$ rises, so the ratio is free to move either way. Raising $t$ past exactly one instance's score removes that instance from $\Pi$. If it was a false positive, precision goes up. If it was a true positive, precision goes *down*, since the numerator lost $1$ while the denominator lost $1$ and $\frac{a-1}{b-1} < \frac{a}{b}$ whenever $a < b$. So precision only trends upward. Locally it dips, once for every true positive that happens to sit above a false positive in the score ordering, and those dips are visible as the jagged precision line in the threshold plot below. The dips get larger toward the high end of the threshold range, where $|\Pi(t)|$ is small enough that removing one instance is a large relative change.

### The tradeoff is within one model, not across models

The rule is that precision and recall trade off *along one classifier's threshold sweep*. It is not a statement about classifiers in general, and reading it as one is a mistake the compact phrasing invites. A better model moves the whole curve outward and can beat a worse model on both numbers at once. The comparison on [[MNIST]] shows exactly this: the SGD 5-detector at its default threshold scores $P = 0.8371$ and $R = 0.6512$, while the random forest at its default threshold scores $P = 0.9897$ and $R = 0.8725$. Both higher. No tradeoff was violated, because the two numbers came from two different scoring functions, and the tradeoff only constrains pairs read off the same one.

### Targeting a precision

Because the threshold is free, "reach 90% precision" is a solvable request rather than a modelling problem: sweep $t$, find the smallest one whose precision clears the target, and read off what recall that costs. On the MNIST 5-detector the answer is $t = 3370.02$, which delivers $P = 0.9000$ and $R = 0.4800$. That is the honest form of the claim. The 5-detector can be made 90% precise, and doing so means it now misses slightly more than half of all real fives. Quoting the $0.90$ without the $0.48$ is not a summary, it is a selection.

### In scikit-learn

The scores have to come from out-of-fold predictions, not from the training fit, or the curve is measuring memorization. `cross_val_predict` with `method="decision_function"` returns one honest score per training instance by scoring each instance with a model that never saw it, which is what makes the sweep trustworthy. This is the same argument [[Cross-Validation]] makes for scores, applied to raw decision values.

scikit-learn 1.6:

```python
import matplotlib.pyplot as plt
from sklearn.model_selection import cross_val_predict
from sklearn.metrics import precision_recall_curve

y_scores = cross_val_predict(sgd_clf, X_train, y_train_5, cv=3,
                             method="decision_function")

precisions, recalls, thresholds = precision_recall_curve(y_train_5, y_scores)

# precision and recall against the threshold, the two curves that cross
plt.plot(thresholds, precisions[:-1], label="Precision")
plt.plot(thresholds, recalls[:-1], label="Recall")

# the same data plotted against each other, which is the PR curve itself
plt.plot(recalls, precisions, label="Precision/Recall curve")

idx_for_90_precision = (precisions >= 0.90).argmax()
threshold_for_90_precision = thresholds[idx_for_90_precision]   # 3370.0194991439557

y_train_pred_90 = (y_scores >= threshold_for_90_precision)
```

The 1.6 signature is `precision_recall_curve(y_true, y_score=None, *, pos_label=None, sample_weight=None, drop_intermediate=False, probas_pred='deprecated')`. The second positional parameter is `y_score`. The older name `probas_pred` still exists in 1.6 but is deprecated as of 1.5 and scheduled for removal in 1.7, so code written against it will warn now and break later; pass the array positionally or as `y_score`.

**Why `precisions[:-1]` is needed.** The return is `(precision, recall, thresholds)` with `precision` and `recall` of length `n_thresholds + 1` and `thresholds` of length `n_thresholds`, increasing. The arrays are one longer because a final sentinel point is appended that corresponds to no threshold at all: the degenerate operating point where nothing is predicted positive, at which the docs fix `precision[-1] = 1` and `recall[-1] = 0` by convention. That point exists so the PR curve closes at the left edge, and it is exactly why plotting precision or recall *against* `thresholds` requires dropping the last element, while plotting `precisions` against `recalls` uses both arrays whole.

**The `argmax` idiom, and where it bites.** `(precisions >= 0.90)` is a boolean array, and NumPy's `argmax` returns the index of the maximum value, breaking ties toward the first occurrence. Since `True` is the maximum of a boolean array, this returns the index of the first `True`, which is the owner's "returns the first index of the maximum value" stated for this case. Two failure modes follow from that, and neither raises a warning:

- On a boolean array with no `True` at all, `argmax` returns $0$ silently, because every element ties for maximum and the first index wins. Nothing distinguishes that from a genuine hit at index $0$. Any use of this idiom on an array that might not contain the target needs an explicit `.any()` check first.
- On the output of `precision_recall_curve` specifically, the all-`False` case cannot arise, because the appended sentinel guarantees `precisions[-1] == 1.0`, which clears any target below $1$. So the search always succeeds, and the way it fails instead is worse: if the only element clearing the target is that sentinel, the returned index is `len(precisions) - 1 == len(thresholds)`, and `thresholds[idx]` then raises `IndexError` on an array one element shorter. The idiom is safe for a reachable target and throws for an unreachable one.

Since scikit-learn 1.5 the library has its own answer to this whole manoeuvre, which the hand-rolled sweep above predates: [[HOML Ch03 Classification|HOML chapter 3]] dates from 2022 and searches the threshold by hand. `TunedThresholdClassifierCV` wraps a fitted or unfitted estimator and searches the threshold that maximizes a scoring function under cross-validation, exposing the winner as `best_threshold_`. `FixedThresholdClassifier` wraps an estimator with a threshold you already decided, so that `predict` applies it, which is what turns a chosen operating point into something a [[Pipeline]] can carry.

scikit-learn 1.6:

```python
from sklearn.model_selection import FixedThresholdClassifier, TunedThresholdClassifierCV

tuned = TunedThresholdClassifierCV(sgd_clf, scoring="f1", cv=3).fit(X_train, y_train_5)
tuned.best_threshold_

pinned = FixedThresholdClassifier(sgd_clf, threshold=3370.02,
                                  response_method="decision_function").fit(X_train, y_train_5)
pinned.predict(X_train)
```

`TunedThresholdClassifierCV` defaults to `scoring="balanced_accuracy"`, `thresholds=100` candidate cut points and `cv=None` for stratified 5-fold. `FixedThresholdClassifier` defaults to `threshold="auto"`, which resolves to $0.5$ when it is reading `predict_proba` and $0$ when it is reading `decision_function`, matching the two conventions above. Its full 1.6 signature is `FixedThresholdClassifier(estimator, *, threshold='auto', pos_label=None, response_method='auto')`, and `TunedThresholdClassifierCV`'s is `TunedThresholdClassifierCV(estimator, *, scoring='balanced_accuracy', response_method='auto', thresholds=100, cv=None, refit=True, n_jobs=None, random_state=None, store_cv_results=False)`.

Both routes are valid and neither retires the other, because they answer different questions. Take the manual sweep when you want the whole curve in front of you and intend to choose the operating point by eye or by an explicit rule, since it is the only route that shows you what every threshold you did not pick would have cost. Take the estimator when you want the chosen threshold to travel with the model rather than live as a loose number in a notebook cell: a `FixedThresholdClassifier` or a `TunedThresholdClassifierCV` is a fitted object obeying the same estimator contract as anything else, so it drops into a [[Pipeline]] as a step and is cross-validated as one unit, threshold included.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| the decision threshold | $t$ | $0$ on a signed score, $0.5$ on a probability | fewer instances are predicted positive, so recall falls monotonically while precision trends up without being monotone; at the top of the range nothing is predicted positive and recall reaches $0$ | sweep it over the out-of-fold scores, pick the point that meets the requirement the task actually states, and report the other number alongside it |
| `pos_label` (`precision_recall_curve`) | the class treated as positive | `None` | names which class the curve is drawn about; switching it computes a completely different curve from the other class's perspective rather than a reflection of this one | required when the labels are not $\{0, 1\}$ or $\{-1, 1\}$, since the function raises rather than guess |
| `drop_intermediate` (`precision_recall_curve`) | whether collinear vertices are discarded | `False` | turning it on removes suboptimal points that would not appear on a plotted curve, shrinking all three returned arrays and changing which index a given `argmax` lands on | leave it `False` whenever you are going to index `thresholds` by a position found in `precisions`, since dropping points breaks that correspondence |
| `scoring` (`TunedThresholdClassifierCV`) | the objective the threshold search maximizes | `"balanced_accuracy"` | decides which point on the sweep is declared best, so switching to `"f1"` or a cost-weighted `make_scorer` moves `best_threshold_` and therefore every number the wrapped model reports | set it to the metric the application is judged on, never left at the default by accident |

`sample_weight` also changes the curve, by replacing each cell count with a weighted sum before the ratios are taken.

## Where it is used

It is the reason [[Precision]] and [[Recall]] are reported together and never separately: they are two readings of one threshold, and either one alone can be driven to an arbitrary value by moving that threshold without improving the model at all. It is also why [[F1 Score]] exists, and why F1 is not always what you want, since collapsing the pair into one balanced number discards the operating point that produced it.

The sweep is run on the scores of a [[Stochastic Gradient Descent Classifier]], which exposes its decision scores through `decision_function` while refusing to let you set the threshold on the estimator itself, and those scores are made honest by [[Cross-Validation]], which supplies an out-of-fold score for every training instance so that the curve is not read off data the model was fitted on. The same sweep viewed through a different pair of axes is the [[ROC Curve]], which plots true positive rate against false positive rate as the identical threshold moves.

[[Logistic Regression]] is the contrast that shows the threshold is a convention rather than a property: its cut sits at $0.5$ because its score is a probability, against the SGD classifier's $0$ on a signed score, and the same reasoning applies unchanged to both. [[Class Imbalance]] is the condition that makes all of this matter, because on a balanced target the default threshold is usually defensible and on a skewed one it rarely is.
