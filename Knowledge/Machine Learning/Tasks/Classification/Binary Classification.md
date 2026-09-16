---
note_kind: concept
aliases:
  - binary classification
  - binary classifier
  - binary classifiers
  - binary classification task
  - two-class classification
  - two-class problem
  - binomial classification
up: "[[Classification]]"
sources:
  - "[[HOML Ch03 Classification]]"
confidence: draft
---

## Definition

A [[Supervised Learning]] task where the goal is to categorize input data points into one of **two mutually exclusive classes**. Every instance gets exactly one of the two, and the two exhaust the possibilities, so answering "is it the first class?" answers the whole question.

## Formal statement

The label space has two elements. Write it $\mathcal{Y} = \{0, 1\}$, or $\{-1, +1\}$ when the sign of a score is doing the work, and the learned hypothesis is a map

$$h : \mathcal{X} \to \mathcal{Y}, \qquad \mathcal{Y} = \{0, 1\}$$

Mutual exclusivity and exhaustiveness are the same statement about the conditional distribution:

$$P(y = 1 \mid \mathbf{x}) + P(y = 0 \mid \mathbf{x}) = 1$$

so one number determines both. That is why nearly every binary classifier is really a real-valued scoring function plus a cut point: a score $s(\mathbf{x}) \in \mathbb{R}$ and a threshold $t$, with

$$\hat{y} = \begin{cases} 1 & s(\mathbf{x}) \ge t \\ 0 & s(\mathbf{x}) < t \end{cases}$$

[[Logistic Regression]] takes $s(\mathbf{x}) = \sigma(\boldsymbol\theta^{T}\mathbf{x})$ and $t = 0.5$; a [[Stochastic Gradient Descent Classifier]] takes the raw linear score $s(\mathbf{x}) = \boldsymbol\theta^{T}\mathbf{x}$ and $t = 0$.

### Which class is positive is a choice, not a fact

Nothing in the data says which of the two values is "positive". That assignment is made by whoever frames the problem, and it is what makes the evaluation asymmetric. Accuracy is invariant under swapping the labels,

$$\text{Accuracy} = \frac{\text{TP} + \text{TN}}{\text{TP} + \text{TN} + \text{FP} + \text{FN}}$$

because the swap exchanges TP with TN and FP with FN, leaving both sums alone. Precision and recall are not:

$$\text{Precision} = \frac{\text{TP}}{\text{TP} + \text{FP}}, \qquad \text{Recall} = \frac{\text{TP}}{\text{TP} + \text{FN}}$$

Swapping the positive class turns these into the negative-class versions, which are different numbers. Both are computed about the positive class only, so declaring the rare class positive and declaring the common class positive give two different reports on the same predictions.

### Binary problems built by relabelling

A $K$-class target becomes a binary one by collapsing it, picking a class $c$ and using the indicator

$$y_c^{(i)} = \mathbb{1}\left[y^{(i)} = c\right]$$

[[MNIST]] supplies the standing case: the ten-class digit label is relabelled into a 5-versus-rest detector, true for every image of a 5 and false for everything else. Collapsing costs balance. The positive rate of the new target is $P(y = c)$, roughly $1/10$ across the ten digits, so a model that always predicts negative already scores about $1 - P(y = c) \approx 0.90$ accuracy while detecting nothing. That single number is the argument for reading a [[Confusion Matrix]] instead of an accuracy score.

### In scikit-learn

scikit-learn 1.6:

```python
from sklearn.linear_model import SGDClassifier

y_train_5 = (y_train == '5')   # True for every 5, False for every other digit
y_test_5 = (y_test == '5')

sgd_clf = SGDClassifier(random_state=42)
sgd_clf.fit(X_train, y_train_5)
sgd_clf.predict([some_digit])
```

A boolean target, or any target with exactly two distinct values, is detected as `type_of_target(y) == "binary"`, and the metrics follow: `precision_score`, `recall_score` and `f1_score` all default to `average="binary"`, which reports the single class named by `pos_label`. For a boolean or $\{0, 1\}$ target `pos_label` defaults to `1`, which is the library making the modeller's choice on their behalf when they do not state it.

## Where it is used

[[Classification]] is the parent task, and binary is its smallest case, the one every other case is built out of. It is a [[Supervised Learning]] task, since the two-valued target has to be supplied for every training instance before anything can be fit.

The vault's two binary classifiers are [[Logistic Regression]], which thresholds a sigmoid probability at $0.5$, and the [[Stochastic Gradient Descent Classifier]], which thresholds a raw decision score at $0$ and exposes that score through `decision_function` so the cut point can be moved. [[Confusion Matrix]] is where binary earns its special status: the $2 \times 2$ case is the only one whose four cells have names, true negative, false positive, false negative and true positive, and every rate read off it is a ratio of two of them. [[Precision-Recall Tradeoff]] follows directly from the asymmetry above, since moving $t$ trades one class's errors for the other's and there is no threshold that improves both.

[[Multiclass Classification]] reduces to this note twice over: [[One-versus-Rest]] builds $K$ binary problems by the indicator relabelling above, and [[One-versus-One]] builds one binary problem per pair of classes.
