---
note_kind: concept
aliases:
  - multiclass classification
  - multi-class classification
  - multiclass
  - multinomial classification
  - multiclass classifier
  - multiclass classifiers
  - multi-class classifier
  - multiclass classification task
up: "[[Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
  - "[[HOML Ch04 Training Models]]"
  - "[[DMLS Ch02 Introduction to Machine Learning Systems Design]]"
confidence: draft
---

## Definition

A [[Supervised Learning]] task where an algorithm classifies an input data point into one, and only one, category out of three or more possible mutually exclusive classes. Also called multinomial classification.

## Formal statement

The label space is finite with $K \ge 3$ elements,

$$\mathcal{Y} = \{1, 2, \dots, K\}, \qquad h : \mathcal{X} \to \mathcal{Y}$$

Mutual exclusivity means the classes partition the outcome space: each instance carries exactly one label, so $P(y = j \wedge y = k \mid \mathbf{x}) = 0$ for $j \ne k$, and exhaustiveness means the class probabilities sum to one,

$$\sum_{k=1}^{K} P(y = k \mid \mathbf{x}) = 1$$

That single constraint is the whole difference from [[Multilabel Classification]], where no such sum rule holds. Setting $K = 2$ recovers [[Binary Classification]], so this is the generalization along the number of classes while the number of targets stays at one.

A multiclass model produces one score per class, $s_1(\mathbf{x}), \dots, s_K(\mathbf{x})$, and predicts the largest:

$$\hat{y} = \arg\max_{k \in \{1, \dots, K\}} s_k(\mathbf{x})$$

The scores become probabilities obeying the sum rule through the softmax,

$$P(y = k \mid \mathbf{x}) = \frac{e^{s_k(\mathbf{x})}}{\sum_{j=1}^{K} e^{s_j(\mathbf{x})}}$$

which sums to $1$ by construction and is strictly increasing in $s_k$, so softmax never changes which class wins the $\arg\max$. It supplies calibrated-looking probabilities, not a different decision. Nothing here says where the scores come from; [[Softmax Regression]] is the model that fits them, one linear score per class, and it is named for this function because the function is the only part of it that is not already [[Logistic Regression]].

### Counting the binary sub-problems

When the underlying algorithm only knows how to separate two classes, the $K$-class problem is decomposed. [[One-versus-Rest]] trains $K$ classifiers, one per class against the union of the others. [[One-versus-One]] trains one classifier per unordered pair of distinct classes, and the count is the number of such pairs:

$$\binom{K}{2} = \frac{K!}{2!\,(K-2)!} = \frac{K(K-1)}{2}$$

Read the arithmetic directly: there are $K$ choices for the first class of a pair and $K - 1$ remaining for the second, giving $K(K-1)$ ordered pairs, and each unordered pair is counted twice, so divide by $2$. For the ten MNIST digits this is $\frac{10 \times 9}{2} = 45$, which is exactly the number of scores `SVC` computes internally on that dataset.

### When the class count is large

A label space with many classes is a **high cardinality** target, and it is a condition on $\mathcal{Y}$ rather than on the data or the algorithm: it is $K$ that is large, and both counts above grow in it, $K$ linearly and $\frac{K(K-1)}{2}$ quadratically.

The word is overloaded in this vault and the two senses are worth holding apart. In [[One-Hot Encoding]] and [[Feature Engineering]], cardinality counts the distinct values a categorical **feature** takes, and high cardinality is a problem about how wide the encoded input becomes. Here it counts the values the **label** takes, and high cardinality is a problem about how many decisions the model has to make. Neither note claims the bare word as a name for itself.

[[Hierarchical Classification]] is the remedy on this side of the ambiguity: give the $K$ classes a tree and put a classifier at each internal node over its own children, so the one $K$-way decision becomes a short chain of narrow ones. It is available only when the problem domain supplies the tree, and it carries its own arithmetic, in classifier count and in how accuracy decays along the path.

### In scikit-learn

A multiclass target is the case `type_of_target(y) == "multiclass"`: one target column, more than two distinct values.

scikit-learn 1.6:

```python
from sklearn.svm import SVC
from sklearn.linear_model import SGDClassifier
from sklearn.multiclass import OneVsRestClassifier

svm_clf = SVC(random_state=42)
svm_clf.fit(X_train[:2000], y_train[:2000])    # the 10-class target, not a 5-vs-rest one
svm_clf.decision_function([some_digit])        # 10 aggregated scores by default

svm_clf.decision_function_shape = "ovo"        # 45 raw pairwise scores instead
svm_clf.decision_function([some_digit])

ovr_clf = OneVsRestClassifier(SVC(random_state=42))   # override the default strategy
ovr_clf.fit(X_train[:2000], y_train[:2000])
len(ovr_clf.estimators_)                       # 10

sgd_clf = SGDClassifier(random_state=42)
sgd_clf.fit(X_train, y_train)                  # also accepts the 10-class target directly
```

`decision_function_shape` controls only whether the $45$ pairwise scores are aggregated into $10$ per-class scores before being returned. `SVC` trains one-versus-one either way.

## Where it is used

[[Classification]] is the parent task and this is its general single-target case; it is a [[Supervised Learning]] task because the one-of-$K$ label has to be supplied for every training instance.

It rests on [[Binary Classification]] whenever the base algorithm is two-class only, through the two decompositions above: [[One-versus-Rest]] is the $K$-classifier strategy where the winner is the highest confidence score, and [[One-versus-One]] is the $\frac{K(K-1)}{2}$-classifier strategy where the winner is the class with the most pairwise votes, each of those classifiers trained only on the two classes it separates. [[Confusion Matrix]] is how multiclass errors are actually read: the $K \times K$ matrix has no named cells, so the diagonal is the correct predictions and every off-diagonal entry $(i, j)$ is the count of true class $i$ predicted as class $j$, which is what turns "the model is 5% wrong" into "the model confuses 8s for 5s".

### Which estimators handle it natively

It is tempting to say some algorithms are binary only, and from the caller's side that is false. Quoting the scikit-learn 1.6 multiclass user guide: "All classifiers in scikit-learn do multiclass classification out-of-the-box. You don't need to use the `sklearn.multiclass` module unless you want to experiment with different multiclass strategies." Hand any of them a $K$-class `y` and it fits. What differs is what happens underneath, and there are three cases:

- **Inherently multiclass.** The algorithm itself optimizes over $K$ classes, with no decomposition: `LogisticRegression` (with most solvers), `RandomForestClassifier`, `GaussianNB`, `DecisionTreeClassifier`, `KNeighborsClassifier`, `MLPClassifier`, `LinearDiscriminantAnalysis`, `RidgeClassifier`.
- **Binary algorithms that scikit-learn wraps automatically.** The caller passes a multiclass `y` and gets a fitted model, but a decomposition happened inside. One-versus-rest is used by `SGDClassifier`, `Perceptron`, `PassiveAggressiveClassifier` and `GradientBoostingClassifier`. One-versus-one is used by `SVC` and `NuSVC`.
- **Explicit wrapping.** `OneVsRestClassifier`, `OneVsOneClassifier` and `OutputCodeClassifier` in `sklearn.multiclass` are needed only to override the default strategy, not to obtain multiclass support that is missing.

[[Logistic Regression]] is the case where the strategy changed recently. Its `multi_class` argument was deprecated in scikit-learn 1.5 and is scheduled for removal in 1.7; the 1.6 default `multi_class="auto"` already selects multinomial softmax for $K \ge 3$, falling back to one-versus-rest only for binary targets or `solver="liblinear"`. After removal, multinomial is used for $K \ge 3$ unconditionally and one-versus-rest requires wrapping the estimator in `OneVsRestClassifier` by hand. So the softmax in the formula above is what a plain `LogisticRegression` fits on a multiclass target in 1.6, not an opt-in, and [[Softmax Regression]] is the name for the model it thereby fits.
