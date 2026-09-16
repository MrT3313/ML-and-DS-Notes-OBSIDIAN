---
note_kind: concept
aliases:
  - accuracy
  - accuracy score
  - classification accuracy
  - accuracy_score
  - error rate
  - misclassification rate
  - subset accuracy
up: "[[Confusion Matrix]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

Accuracy is the share of predictions a classifier got right: correct predictions over total predictions, counting every class and every kind of mistake the same. It is the first number anyone reaches for and the first one to stop trusting, because a single ratio cannot distinguish a model that has learned something from a model that has learned which answer is common.

## Formal statement

In terms of the four binary cells of a [[Confusion Matrix]],

$$\text{Accuracy} = \frac{\text{TP} + \text{TN}}{\text{TP} + \text{TN} + \text{FP} + \text{FN}}$$

The numerator is the correct predictions of both classes and the denominator is every instance, so the denominator is $m$ and does not depend on the model at all. The general $K$-class form is the mean of an indicator over the evaluation set,

$$\text{Accuracy} = \frac{1}{m} \sum_{n=1}^{m} \mathbb{1}\!\left[\hat{y}^{(n)} = y^{(n)}\right] = \frac{\operatorname{tr}(C)}{\sum_{i}\sum_{j} C_{ij}} = \frac{\operatorname{tr}(C)}{m}$$

which ties it to the confusion matrix exactly: accuracy is the diagonal mass divided by the total mass, and it throws away the entire off-diagonal structure, keeping only how much of it there is. The complement is the error rate,

$$\text{error rate} = 1 - \text{Accuracy} = \frac{m - \operatorname{tr}(C)}{m} = \frac{\text{FP} + \text{FN}}{m}$$

Accuracy is symmetric under relabelling which class is positive, since swapping exchanges TP with TN and FP with FN and leaves both sums alone. That symmetry is the source of both its appeal and its blindness: it cannot express a preference between the two kinds of error, so it is the wrong measure exactly when the two kinds cost different amounts.

### Why a high accuracy can carry almost no information

Let $p_k = m_k / m$ be the frequency of class $k$ in the evaluation set. A constant predictor, one that ignores its input and always answers $c$, scores

$$\text{Accuracy}(\text{always } c) = p_c, \qquad \max_{c} \text{Accuracy}(\text{always } c) = \max_{k} p_k$$

because its confusion matrix is a single non-zero column, and the only diagonal cell in that column is $C_{cc} = m_c$. So the floor for accuracy is not $1/K$ and it is not $0.5$: it is the majority class frequency, which under [[Class Imbalance]] is close to $1$. Any accuracy below $\max_k p_k$ is worse than answering the same thing every time, and any accuracy above it has to be judged by how far above, not by how close to $1$.

The numbers on the [[MNIST]] 5-detector make the point. Relabelling the ten-class MNIST target into a 5-versus-rest detector gives $5421$ fives among the $60000$ training instances, so

$$p_{5} = \frac{5421}{60000} = 0.09035, \qquad p_{\text{not }5} = \frac{54579}{60000} = 0.90965$$

A `DummyClassifier` that never predicts a five scores exactly $0.90965$ in all three cross-validation folds, which is $p_{\text{not }5}$ reproduced to five decimal places and not a coincidence: it is the formula above. The [[Stochastic Gradient Descent Classifier]] scores $0.95035$, $0.96035$ and $0.96040$ on the same three folds, roughly $0.957$ on average. Read as a raw number that is an excellent score. Read against the floor it is about four and a half points of accuracy bought by an actual model, and the confusion matrix says where they went: of the $5421$ fives, $3530$ were caught and $1891$ were missed, so the detector overlooks more than a third of the thing it exists to detect while reporting $95\%$ accuracy. That gap between the score and the behaviour is the whole argument for reading [[Precision]] and [[Recall]] instead.

The condition is the target's class frequencies, not the feature distribution. [[Skewed Data]] in a feature is a different problem with a different fix; what breaks accuracy is a skewed *label*.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.metrics import accuracy_score
from sklearn.model_selection import cross_val_score
from sklearn.dummy import DummyClassifier

accuracy_score(y_train_5, y_train_pred)

cross_val_score(sgd_clf, X_train, y_train_5, cv=3, scoring="accuracy")

dummy_clf = DummyClassifier()
cross_val_score(dummy_clf, X_train, y_train_5, cv=3, scoring="accuracy")
```

The signature is `accuracy_score(y_true, y_pred, *, normalize=True, sample_weight=None)`. Accuracy is the default scorer for classifiers, so `scoring="accuracy"` and omitting `scoring` entirely give the same thing, which is part of why it gets reported by default whether or not it is the right measure. `DummyClassifier()` defaults to `strategy="prior"`, whose `predict` returns the most frequent class for every input, so running it is the cheapest way to compute the floor $\max_k p_k$ for the actual folds rather than estimating it; see [[Baseline Model]]. For a multilabel target `accuracy_score` computes subset accuracy, scoring an instance correct only when its entire predicted label set matches exactly, which is a stricter quantity that happens to share the name.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| `normalize` | switches between $\operatorname{tr}(C)/m$ and $\operatorname{tr}(C)$ | `True` | `True` returns the fraction of correct predictions; setting it to `False` returns the raw count instead, so the value stops being bounded by $1$ and scales with $m$ | leave it `True` for anything comparable across sets; `False` only when you want the instance count itself |
| `sample_weight` | per-instance weights $w_i$ | `None` | each instance contributes $w_i$ rather than $1$, so the score becomes $\sum_i w_i \mathbb{1}[\hat{y}_i = y_i] / \sum_i w_i$ and instances with larger weights dominate | leave unset unless the instances genuinely are not equally important, ex a sample drawn with unequal probabilities |

Nothing else `accuracy_score` accepts affects the value.

## Where it is used

It is the simplest [[Performance Measure]] for [[Classification]] and the default scorer in scikit-learn, which is why it is the number every model reports before anyone asks for a better one. It is the trace of a [[Confusion Matrix]] over that matrix's total, so it is a strict summary of the matrix and can always be recovered from it while the reverse is false.

[[Class Imbalance]] is the condition that makes it misleading, since the floor rises to the majority class frequency $\max_k p_k$ rather than staying at chance. [[Baseline Model]] is how you find that floor in practice: a constant predictor that learns nothing scores $p_{\text{not }5} = 0.90965$ on the 5-versus-rest MNIST target, and any accuracy is read as a distance above that, not as a distance below $1$. [[Precision]] and [[Recall]] are what you report instead, because each is computed about the positive class alone and therefore cannot be inflated by a large, easy negative class.
