---
note_kind: method
aliases:
  - logit regression
  - logit model
  - binary logistic regression
up: "[[Classification]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
confidence: draft
---

## What it does and when

Logistic regression is a classifier, despite the name. It predicts the probability that an instance belongs to the positive class by passing a linear score through the sigmoid, then thresholds that probability. Use it for binary [[Classification]] when a linear decision boundary is plausible and calibrated probabilities are wanted; the multiclass extension (softmax) is chapter 4.

## Algorithm or formula

$$\hat{p} = h_{\boldsymbol\theta}(\mathbf{x}) = \sigma(\boldsymbol\theta^{T}\mathbf{x}), \qquad \sigma(t) = \frac{1}{1 + e^{-t}}$$

Predict $\hat{y} = 1$ if $\hat{p} \ge 0.5$, else $0$. The [[Cost Function]] is log loss,

$$J(\boldsymbol\theta) = -\frac{1}{m} \sum_{i=1}^{m} \Big[ y^{(i)} \log \hat{p}^{(i)} + (1 - y^{(i)}) \log (1 - \hat{p}^{(i)}) \Big]$$

minimized by gradient descent; there is no closed form.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| inverse regularization strength | $C$ | 1.0 | less regularization, more flexible boundary, more [[Overfitting]] risk | [[Cross-Validation]] over a log grid |
| decision threshold | $t$ | 0.5 | fewer positives predicted, higher precision, lower recall | chapter 3 metrics |

## Failure modes

- Classes not linearly separable in the feature space: underfits; add features or change family.
- Perfectly separable classes: weights diverge without regularization.
- Class imbalance: the 0.5 threshold predicts the majority class; move $t$ or reweight.

## Implementation

scikit-learn 1.6:

```python
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(C=1.0).fit(X_train, y_train)
proba = clf.predict_proba(X_new)[:, 1]
```
