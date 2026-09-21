---
note_kind: concept
aliases:
  - multilabel classification
  - multi-label classification
  - multilabel
  - multi-label
  - multilabel classifier
  - multilabel classifiers
  - multilabel-indicator
up: "[[Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

A [[Supervised Learning]] task where the goal is to predict multiple non-exclusive categories for a single instance. A single data point can belong to zero, one, or several classes simultaneously.

## Formal statement

The target is not a single value but a vector of $L$ binary indicators, one per label:

$$\mathbf{y} = (y_1, y_2, \dots, y_L) \in \{0, 1\}^{L}, \qquad y_j = \mathbb{1}[\text{instance carries label } j]$$

Each coordinate is an independent yes or no, so the set of admissible targets is the whole hypercube and its size is

$$\left|\{0, 1\}^{L}\right| = 2^{L}$$

one for every subset of the $L$ labels, including the empty set (no label applies) and the full set (all of them do). That is the precise sense in which labels are not mutually exclusive: there is no constraint

$$\sum_{j=1}^{L} y_j = 1$$

Imposing that constraint is exactly what would collapse this back to [[Multiclass Classification]] over $L$ classes, whose one-hot targets are only $L$ of the $2^{L}$ vectors. Dropping it multiplies the output space by $2^{L}/L$.

Because the coordinates are unconstrained, the task decomposes cleanly: fitting $L$ independent [[Binary Classification]] models, one per label, is a correct decomposition, it is the naive one, and it has a standard name in the multilabel literature, **binary relevance**, which turns any multilabel problem into one binary problem per label and trains $L$ classifiers, each responsible for one coordinate's $0$ or $1$. It is the same construction as [[One-versus-Rest]] under a different name, which is why handing `OneVsRestClassifier` a 2D indicator matrix instead of a 1D label fits exactly these $L$ models. What it throws away is correlation between labels, since it models

$$P(\mathbf{y} \mid \mathbf{x}) = \prod_{j=1}^{L} P(y_j \mid \mathbf{x})$$

which is the independence assumption, and labels in real problems are rarely independent. [[Classifier Chain]] is the fix that keeps the decomposition but conditions each label on the ones before it.

The standing [[MNIST]] example has $L = 2$: for each digit, is it large ($\ge 7$) and is it odd. The target is $\mathbf{y} = (\mathbb{1}[d \ge 7], \mathbb{1}[d \bmod 2 = 1])$, and $2^{2} = 4$ label sets are possible. A 5 is $(0, 1)$, a 7 is $(1, 1)$, a 4 is $(0, 0)$.

### Evaluation differs from the single-label case

A prediction is no longer simply right or wrong, it is right on some labels and wrong on others, so a scalar score has to be built by averaging. Per label $j$, compute the ordinary binary score, say $F_{1,j}$. The two averages in standard use are

$$F_1^{\text{macro}} = \frac{1}{L} \sum_{j=1}^{L} F_{1,j}, \qquad F_1^{\text{weighted}} = \frac{\sum_{j=1}^{L} n_j F_{1,j}}{\sum_{j=1}^{L} n_j}$$

where $n_j$ is the support of label $j$, the number of instances that actually carry it. Macro treats every label as equally important regardless of how rare it is. Weighted lets common labels dominate the number. The two coincide when all supports are equal, and diverge exactly to the extent that supports differ, which is why they come out nearly identical on the large-and-odd MNIST target, whose two labels have comparable support, and would not on a target with one rare label.

### In scikit-learn

A multilabel target is `type_of_target(y) == "multilabel-indicator"`: more than one target column, each with cardinality $2$. In practice that is a boolean or $\{0, 1\}$ array of shape `(n_samples, L)`.

Native support is not universal. The scikit-learn 1.6 multiclass and multioutput user guide lists as supporting multilabel directly: `DecisionTreeClassifier`, `ExtraTreeClassifier`, `ExtraTreesClassifier`, `KNeighborsClassifier`, `MLPClassifier`, `RadiusNeighborsClassifier`, `RandomForestClassifier`, `RidgeClassifier`, `RidgeClassifierCV`. `SVC` is absent from that list, which is why a support vector machine on this target has to be wrapped.

scikit-learn 1.6:

```python
import numpy as np
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import cross_val_predict
from sklearn.metrics import f1_score

y_train_large = (y_train >= '7')
y_train_odd = (y_train.astype('int8') % 2 == 1)
y_multilabel = np.c_[y_train_large, y_train_odd]     # shape (n, 2)

knn_clf = KNeighborsClassifier()
knn_clf.fit(X_train, y_multilabel)
knn_clf.predict([some_digit])                        # one row: not large, odd

y_train_knn_pred = cross_val_predict(knn_clf, X_train, y_multilabel, cv=3)
f1_score(y_multilabel, y_train_knn_pred, average="macro")
f1_score(y_multilabel, y_train_knn_pred, average="weighted")
```

`KNeighborsClassifier`, the k-nearest-neighbours classifier fitted above, is the natural choice on this target precisely because it is one of the estimators with native multilabel support.

In 1.6, `f1_score` takes `average` from `{"micro", "macro", "samples", "weighted", "binary"}` or `None`, defaulting to `"binary"`. That default is unusable here, since it reports a single class of a single binary target, so on a multilabel target one of the others must be passed explicitly. `"micro"` pools the counts across all labels before computing one score, `"samples"` scores each instance's label set and averages over instances, and `None` skips averaging and returns the vector of $L$ per-label scores.

## Where it is used

[[Classification]] is the parent task, and this is the branch where the single-label assumption is dropped; it is a [[Supervised Learning]] task, with the full $L$-vector of labels required for every training instance.

It is built out of [[Binary Classification]], one independent binary decision per label, which is both the definition of the target and the naive way to fit it. [[Classifier Chain]] is how a classifier without native support is given it, by fitting one binary model per label in sequence and feeding each model the labels already decided, so that the independence product above is replaced by a chain of conditionals. [[Multioutput Classification]] is its generalization: relax each label from two values to any number and multilabel is the special case that falls out. [[F1 Score]] is the metric it is evaluated with, but only once averaged across labels, which is what the `average` argument of `f1_score` exists to control.
