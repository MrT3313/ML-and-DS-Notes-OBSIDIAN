---
note_kind: concept
aliases:
  - multioutput classification
  - multi-output classification
  - multioutput
  - multi-output
  - multioutput-multiclass classification
  - multiclass-multioutput classification
  - multioutput classifier
  - multioutput classifiers
up: "[[Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

A [[Supervised Learning]] task where the goal is to predict multiple target variables for a single instance, where each target variable is a categorical feature with two or more possible classes. It is a generalization of [[Multilabel Classification]] where each label can be multiclass.

## Formal statement

There are $L$ targets, and target $j$ draws from its own finite class set $C_j$ with $|C_j| \ge 2$. The prediction is a point in the product of those sets:

$$\mathbf{y} = (y_1, \dots, y_L) \in C_1 \times C_2 \times \dots \times C_L, \qquad y_j \in C_j$$

The class sets need not agree with one another, in size or in content. The number of distinct outputs the model can emit is the product of their sizes,

$$\left|C_1 \times \dots \times C_L\right| = \prod_{j=1}^{L} |C_j|$$

Every other classification task in the vault is a constrained case of this one:

| task | constraint | output space size |
|---|---|---|
| [[Binary Classification]] | $L = 1$ and $\lvert C_1 \rvert = 2$ | $2$ |
| [[Multiclass Classification]] | $L = 1$ and $\lvert C_1 \rvert = K \ge 3$ | $K$ |
| [[Multilabel Classification]] | $\lvert C_j \rvert = 2$ for every $j$ | $2^{L}$ |
| multioutput | none beyond $\lvert C_j \rvert \ge 2$ | $\prod_j \lvert C_j \rvert$ |

Setting every $|C_j| = 2$ in the product gives $\prod_{j=1}^{L} 2 = 2^{L}$, which is exactly the multilabel hypercube. That is the generalization stated as arithmetic: multilabel is the corner of this task where every output happens to be binary, and each individual output of a multioutput problem is on its own a multiclass problem.

Each output is still one-of-$C_j$, so within a single target the classes remain mutually exclusive and the per-target probabilities sum to one,

$$\sum_{c \in C_j} P(y_j = c \mid \mathbf{x}) = 1 \quad \text{for each } j$$

Non-exclusivity lives between targets, not inside one. It is also worth separating this from multioutput regression, which has the same $L > 1$ shape but continuous targets rather than finite $C_j$; the distinguishing property is that $C_j$ is a finite set of categories.

Image denoising pushes $L$ hard. The task is removing noise from an [[MNIST]] image: the input is a noisy $28 \times 28$ image, and the target is the clean image, so there are $L = 784$ outputs, one per pixel, each taking an intensity in $C_j = \{0, 1, \dots, 255\}$. The output space is $256^{784}$, and the line between classification and regression gets thin here, since pixel intensity is ordered in a way ordinary class labels are not.

### In scikit-learn

The target type is `type_of_target(y) == "multiclass-multioutput"`: more than one target column, each with cardinality greater than $2$. The scikit-learn 1.6 multiclass and multioutput user guide lists as supporting it natively: `DecisionTreeClassifier`, `ExtraTreeClassifier`, `ExtraTreesClassifier`, `KNeighborsClassifier`, `RadiusNeighborsClassifier`, `RandomForestClassifier`. Anything else has to be wrapped in `MultiOutputClassifier`, which fits one independent estimator per target column.

scikit-learn 1.6:

```python
import numpy as np
from sklearn.neighbors import KNeighborsClassifier

np.random.seed(42)
noise = np.random.randint(0, 100, (len(X_train), 784))
X_train_mod = X_train + noise                 # noisy input
y_train_mod = X_train                         # clean image as the 784-column target

knn_clf = KNeighborsClassifier()
knn_clf.fit(X_train_mod, y_train_mod)
clean_digit = knn_clf.predict([X_test_mod[0]])
```

`KNeighborsClassifier`, the k-nearest-neighbours classifier fitted above, is on the native-support list, so the denoising fit needs no wrapper. Note that the metrics do not follow the estimators: `f1_score` and the rest of the binary and multilabel scorers reject a `multiclass-multioutput` target, so evaluation here is done on the reconstruction itself rather than with a classification metric.

## Where it is used

[[Classification]] is the parent task, and this is its most general form, every other shape of classification target being a constrained case of the product space above. It is a [[Supervised Learning]] task, since all $L$ target columns have to be supplied for every training instance.

[[Multilabel Classification]] is the special case where every output is binary, $|C_j| = 2$ for all $j$, which is why the $2^{L}$ count there is just $\prod_j |C_j|$ with every factor equal to two. [[Multiclass Classification]] is what each individual output is: fix an index $j$ and the sub-problem "predict $y_j$" is a one-of-$C_j$ multiclass problem, which is how the wrapper `MultiOutputClassifier` decomposes the task and how a per-target [[Confusion Matrix]] would be read.
