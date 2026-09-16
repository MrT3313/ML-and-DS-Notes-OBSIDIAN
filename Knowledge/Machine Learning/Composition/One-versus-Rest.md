---
note_kind: method
aliases:
  - OvR
  - OvA
  - one-versus-rest
  - one-vs-rest
  - one versus rest
  - one-versus-the-rest
  - one-vs-the-rest
  - one-versus-all
  - one-vs-all
  - one versus all
  - OneVsRestClassifier
  - sklearn.multiclass.OneVsRestClassifier
  - binary relevance
up: "[[Multiclass Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## What it does and when

One-versus-rest turns an algorithm that can only draw one boundary into a [[Multiclass Classification]] classifier. For $N$ classes it trains $N$ separate [[Binary Classification]] problems, each asking "class $k$, or anything else?", and at prediction time it keeps whichever of the $N$ answers came back with the highest confidence score. "One-versus-all" and "OvA" are the same method under a different name, not a variant of it: scikit-learn's own user guide introduces the strategy as "the one-vs-rest strategy, also known as one-vs-all", and [[HOML Ch03 Classification|HOML chapter 3]] states the same equivalence outright: "This is called one-versus-the-rest (OvR) strategy, or sometimes one-versus-all (OvA)". That spelling adds a definite article, putting a third surface form of the name into circulation, which is why all of them are aliases here.

Reach for it as the default decomposition. It is the cheaper of the two in classifier count, $N$ against the $\frac{N(N-1)}{2}$ of [[One-versus-One]], and each class ends up owned by exactly one classifier, so inspecting that classifier tells you what the model learned about that class. Prefer one-versus-one instead when the base algorithm's training cost grows faster than linearly in the number of instances, since one-versus-rest hands the *whole* training set to every one of its $N$ fits.

The meta-estimator is rarely needed to *get* multiclass behaviour. scikit-learn already applies a strategy on its own when a binary-only algorithm is handed a multiclass target, and the user guide is blunt about it: "All classifiers in scikit-learn do multiclass classification out-of-the-box. You don't need to use the `sklearn.multiclass` module unless you want to experiment with different multiclass strategies." In 1.6 the built-in choice is one-versus-rest for [[Stochastic Gradient Descent Classifier|SGDClassifier]], `Perceptron`, `PassiveAggressiveClassifier`, `GradientBoostingClassifier` and `LinearSVC`, and one-versus-one for `SVC` and `NuSVC`. So the explicit wrapper earns its place when you want to *override* that default: `SVC` would train pairwise on its own, and wrapping it in `OneVsRestClassifier` forces the other decomposition.

It is an estimator that wraps an estimator, so it obeys the [[Scikit-Learn Estimator API]] in both directions, the same relationship [[Pipeline]] has to its steps: it exposes `fit`, `predict` and `decision_function`, it clones the estimator it was given, and the wrapped object's own settings are reachable through the `estimator__param` path.

## Algorithm

Let the training set be $\{(\mathbf{x}^{(i)}, y^{(i)})\}_{i=1}^{m}$ with $y^{(i)} \in \{C_1, \dots, C_N\}$.

1. For each class $k = 1, \dots, N$, relabel every instance: $z^{(i)}_k = 1$ if $y^{(i)} = C_k$, else $z^{(i)}_k = 0$. This is what `LabelBinarizer` produces internally.
2. Fit a binary classifier $f_k$ on the **entire** training set with those relabelled targets. Class $k$ is positive, the union of the other $N - 1$ classes is negative.
3. Repeat until all $N$ classifiers are fitted. Nothing is shared between them; each is an independent fit.
4. To predict, evaluate every $f_k$ on $\mathbf{x}$ and take the class whose classifier scored highest:

$$\hat{y}(\mathbf{x}) = \arg\max_{k \in \{1, \dots, N\}} f_k(\mathbf{x})$$

$f_k$ here is the raw score, not a hard label. scikit-learn asks the wrapped classifier for `decision_function` first and falls back to `predict_proba` when there is none.

The cost is the point of contrast with one-versus-one. One-versus-rest fits $N$ classifiers, each on all $m$ instances, so the total training work is

$$N \cdot T(m)$$

where $T$ is the base algorithm's cost on $m$ instances. Prediction costs $N$ calls to the base classifier.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| wrapped base estimator | $f$ | required, positional, no default | a higher-capacity base classifier gives each of the $N$ binary problems a more flexible boundary and changes every fitted $f_k$; it also sets the total cost $N \cdot T(m)$ | [[Cross-Validation]] over candidate base classifiers, with each candidate's own settings reached as `estimator__C`, `estimator__gamma` and so on |

`n_jobs` and `verbose` are the only other constructor arguments in 1.6 and neither changes the fitted output, so neither is a hyperparameter.

## Failure modes

- **Every binary problem is imbalanced by construction.** With roughly equal class sizes, each $f_k$ sees a positive rate of $\frac{m/N}{m} = \frac{1}{N}$. On the MNIST digits, $N = 10$, so each of the ten classifiers trains on 10% positives against 90% negatives, and a base classifier that reacts badly to skew, or a threshold left at its default, will under-predict its own class before any multiclass logic runs.
- **The $N$ scores are not on a common scale.** Each $f_k$ was fitted independently on a different problem, so nothing forces its margins or probabilities to be comparable with $f_j$'s. The $\arg\max$ in step 4 nonetheless compares them directly. A base classifier whose decision function is systematically larger for one class wins arguments it should have lost, and the fix is calibration, not a different $\arg\max$.
- **The cost is the full training set $N$ times over.** For an algorithm with $T(m) \in O(m^2)$, such as `SVC`, whose documentation states the fit time "scales at least quadratically with the number of samples", the total is $O(N m^2)$. This is why an `SVC` wrapped this way is fitted on `X_train[:2000]` rather than all 60,000 rows, and why one-versus-one is the strategy `SVC` picks for itself.
- **Ties in the $\arg\max$ are resolved by position, not by evidence.** scikit-learn's implementation scans the estimators in order and keeps the last index attaining the running maximum, so two classes with identical scores resolve to whichever class sorts later in `classes_`. It is rare with continuous scores and silent when it happens.

## Implementation

scikit-learn 1.6:

```python
from sklearn.multiclass import OneVsRestClassifier
from sklearn.svm import SVC

# SVC would decompose pairwise on its own; the wrapper overrides that choice
ovr_clf = OneVsRestClassifier(SVC(random_state=42))
ovr_clf.fit(X_train[:2000], y_train[:2000])

ovr_clf.predict([some_digit])
len(ovr_clf.estimators_)   # 10, one per digit
```

The full signature in 1.6 is `OneVsRestClassifier(estimator, *, n_jobs=None, verbose=0)`. There is no `random_state` on the wrapper itself; reproducibility is set on the wrapped estimator, `SVC(random_state=42)` above, and the [[Random Seed]] note covers why that argument has to be pinned rather than left at `None`.

Passing a 2D binary indicator matrix as `y` makes the same class do multilabel work instead, one independent classifier per label. That is the binary relevance method, and it models no dependence between labels, which is the gap [[Classifier Chain]] exists to close.
