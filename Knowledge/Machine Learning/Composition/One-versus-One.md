---
note_kind: method
aliases:
  - OvO
  - one-versus-one
  - one-vs-one
  - one versus one
  - OneVsOneClassifier
  - sklearn.multiclass.OneVsOneClassifier
  - pairwise classification
up: "[[Multiclass Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## What it does and when

One-versus-one is the other way to give a binary-only algorithm [[Multiclass Classification]]. Instead of one classifier per class it trains one per *pair* of classes, $\frac{N(N-1)}{2}$ of them for $N$ classes, and each of those [[Binary Classification]] problems is fitted only on the instances that actually belong to its two classes. At prediction time every pairwise classifier votes for one of its two classes and the class with the most votes wins.

The main advantage is that property of the training sets: each classifier only needs the part of the training set containing the two classes in question. That matters exactly when the base algorithm's cost grows faster than linearly in the number of instances. `SVC` is the standing case, since its fit time "scales at least quadratically with the number of samples", and it is why `SVC` and `NuSVC` decompose pairwise by default in scikit-learn 1.6 while [[Stochastic Gradient Descent Classifier|SGDClassifier]] and `LinearSVC` decompose against the rest. For anything whose cost is linear in $m$, [[One-versus-Rest]] is cheaper and is the fair default.

You rarely have to ask for it: scikit-learn applies a decomposition on its own when a binary-only estimator is handed a multiclass target, and the user guide says "You don't need to use the `sklearn.multiclass` module unless you want to experiment with different multiclass strategies." The explicit `OneVsOneClassifier` is therefore an override, useful for forcing pairwise training on an estimator whose built-in choice is one-versus-rest. Like its sibling it is an estimator wrapping an estimator under the [[Scikit-Learn Estimator API]], with the wrapped object's settings reachable as `estimator__param`.

A confusion worth heading off: `SVC` exposes `decision_function_shape`, which defaults to `"ovr"`. That controls only whether the $\frac{N(N-1)}{2}$ pairwise scores are aggregated into $N$ per-class scores before being returned. `SVC` still trains pairwise either way.

## Algorithm

1. Enumerate the unordered pairs of classes. Order does not matter, because the classifier separating $C_i$ from $C_j$ is the same object as the one separating $C_j$ from $C_i$, and a class is never paired with itself. There are $N$ choices for the first member and $N-1$ for the second, giving $N(N-1)$ ordered pairs, and each unordered pair is counted twice, so

$$\binom{N}{2} = \frac{N(N-1)}{2}$$

  pairs. For the ten MNIST digits that is $\frac{10 \cdot 9}{2} = 45$ classifiers.
2. For each pair $(i, j)$, take the subset $D_{ij} = \{(\mathbf{x}^{(t)}, y^{(t)}) : y^{(t)} \in \{C_i, C_j\}\}$ and fit a binary classifier $f_{ij}$ on it with $C_i$ negative and $C_j$ positive. Instances of the other $N - 2$ classes are dropped, not relabelled.
3. To predict, run all $\frac{N(N-1)}{2}$ classifiers on $\mathbf{x}$ and count votes:

$$v_k(\mathbf{x}) = \sum_{\{i,j\} \ni k} \mathbb{1}\big[f_{ij}(\mathbf{x}) \text{ picks } C_k\big]$$

4. Return $\hat{y}(\mathbf{x}) = \arg\max_k \big( v_k(\mathbf{x}) + c_k(\mathbf{x}) \big)$, where $c_k$ is the tie-break term of the next paragraph.

The raw vote count on its own leaves ties unresolved, so scikit-learn adds a small continuous term. It sums the signed pairwise confidences into $s_k(\mathbf{x})$, one per class, then squashes that sum before adding it to the votes:

$$c_k = \frac{s_k}{3\,(|s_k| + 1)} \in \left(-\tfrac{1}{3}, \tfrac{1}{3}\right)$$

Because $|c_k| < \frac{1}{3} < \frac{1}{2}$, the term can never reorder two classes that differ by a whole vote. It only separates classes that are level on votes. So the precise rule is: **most votes wins, and among classes tied on votes the one with the larger normalized sum of pairwise confidences wins.** `predict` is literally `argmax(decision_function(X), axis=1)` over that combined quantity.

The cost argument that makes the subset property real. With roughly balanced classes each class holds about $\frac{m}{N}$ instances, so each pairwise problem sees about

$$m_{ij} \approx \frac{2m}{N}$$

instances. If the base algorithm costs $T(m) = O(m^a)$, the two strategies total

$$\text{one-versus-one: } \frac{N(N-1)}{2} \left(\frac{2m}{N}\right)^{a} \qquad \text{one-versus-rest: } N m^{a}$$

and their ratio is $\dfrac{(N-1)\,2^{a-1}}{N^{a}}$. For $a = 1$ this is about $1$, so there is nothing to gain. For $a = 2$ and $N = 10$ it is $\frac{9 \cdot 2}{100} = 0.18$, so pairwise training does roughly a fifth of the work despite fitting 45 classifiers instead of 10. That is the whole case for the strategy.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| wrapped base estimator | $f$ | required, positional, no default | a higher-capacity base classifier gives each of the $\frac{N(N-1)}{2}$ pairwise problems a more flexible boundary, changes every fitted $f_{ij}$, and sets the total cost $\frac{N(N-1)}{2} \cdot T\!\left(\frac{2m}{N}\right)$; it also decides the tie-break, since a base classifier with `decision_function` supplies margins and one with only `predict_proba` supplies probabilities | [[Cross-Validation]] over candidate base classifiers, with each candidate's own settings reached as `estimator__C`, `estimator__gamma` and so on |

`n_jobs` is the only other constructor argument in 1.6 and it does not change the fitted output, so it is not a hyperparameter.

## Failure modes

- **The classifier count grows quadratically in $N$, and prediction pays all of it.** $\frac{N(N-1)}{2}$ is $45$ calls at $N = 10$ but $4{,}950$ at $N = 100$ and $499{,}500$ at $N = 1000$. The training saving comes from small subsets, so it does not help here: every one of those classifiers has to be evaluated on every instance you predict, and the trained model has that many sets of parameters to store.
- **Votes tie, and the tie-break is a different kind of quantity.** Nothing in the voting scheme prevents $v_i = v_j$; with $N = 3$ a cyclic disagreement gives every class exactly one vote. The winner is then decided by $c_k$, which is built from the base classifier's raw confidences, so the outcome depends on the scale of margins that were never calibrated against each other. The classification is real but the evidence behind it is one vote wide.
- **Each pairwise classifier is interrogated about classes it never saw.** $f_{ij}$ was fitted on $D_{ij}$ alone, so an instance of class $C_k$ with $k \notin \{i, j\}$ is outside its training distribution entirely, and it still returns a confident vote for $C_i$ or $C_j$. Every prediction therefore carries $\binom{N-1}{2}$ such uninformed votes, $36$ of the $45$ on MNIST, and the scheme only works because those votes are expected to scatter rather than concentrate on one wrong class.
- **Rare classes get pairwise subsets too small to fit.** A class with a handful of instances appears in $N - 1$ of the pairs, and in each it contributes only that handful against a full-sized opponent, so those classifiers are both tiny and extremely skewed. One-versus-rest at least gives every fit the whole training set.

## Implementation

scikit-learn 1.6:

```python
from sklearn.multiclass import OneVsOneClassifier
from sklearn.linear_model import SGDClassifier

# SGDClassifier decomposes against the rest on its own; this overrides that
ovo_clf = OneVsOneClassifier(SGDClassifier(random_state=42))
ovo_clf.fit(X_train, y_train)

ovo_clf.predict([some_digit])
len(ovo_clf.estimators_)          # 45 for the ten digits
ovo_clf.decision_function([some_digit])   # shape (1, 10): votes plus tie-break
```

The full signature in 1.6 is `OneVsOneClassifier(estimator, *, n_jobs=None)`. There is no `random_state` on the wrapper; it is set on the wrapped estimator, as `SGDClassifier(random_state=42)` above, and [[Random Seed]] covers why leaving it at `None` makes the fit unreproducible.

To see the pairwise scores from an estimator that decomposes this way internally rather than through the wrapper:

```python
from sklearn.svm import SVC

svm_clf = SVC(random_state=42)
svm_clf.fit(X_train[:2000], y_train[:2000])

svm_clf.decision_function_shape = "ovo"
svm_clf.decision_function([some_digit])   # shape (1, 45), the raw pairwise scores
```
