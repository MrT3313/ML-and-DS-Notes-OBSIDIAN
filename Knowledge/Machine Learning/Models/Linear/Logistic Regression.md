---
note_kind: method
aliases:
  - logit regression
  - logit model
  - binary logistic regression
up: "[[Classification]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch03 Classification]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Logistic regression is a classifier, despite the name. It predicts the probability that an instance belongs to the positive class by passing a linear score through the sigmoid, then thresholds that probability. Use it for binary [[Classification]] when a linear [[Decision Boundary]] is plausible and calibrated probabilities are wanted.

[[Softmax Regression]] is the multiclass generalization, one linear score per class instead of one score total, and nothing has to be switched on to reach it: in scikit-learn 1.6 a target with three or more classes is already fitted multinomially by default under every solver but `liblinear`, so [[Multiclass Classification]] works out of the box. `newton-cholesky` was the second exception through 1.5 and stopped being one in 1.6, which extended that solver to the full multinomial loss.

## Algorithm or formula

$$\hat{p} = h_{\boldsymbol\theta}(\mathbf{x}) = \sigma\big(s(\mathbf{x})\big), \qquad s(\mathbf{x}) = \boldsymbol\theta^{T}\mathbf{x}, \qquad \sigma(s) = \frac{1}{1 + e^{-s}}$$

Predict $\hat{y} = 1$ if $\hat{p} \ge 0.5$, else $0$.

The linear score $s(\mathbf{x})$ handed to the sigmoid is called the **logit**, and the name comes from the inverse function rather than from the forward one:

$$\sigma^{-1}(p) = \log\frac{p}{1 - p}$$

which is the log-odds of the positive class. So the thing this model is linear in is not the probability, it is the log-odds, and a one-unit change in a feature adds its weight to the log-odds no matter where on the curve the instance sits.

### The boundary is a hyperplane

$\sigma$ is strictly increasing and $\sigma(0) = 0.5$, so the prediction rule can be read off the raw score without ever computing the probability:

$$\hat{p} \ge 0.5 \iff \boldsymbol\theta^{T}\mathbf{x} \ge 0$$

The set where the prediction changes is therefore the hyperplane $\boldsymbol\theta^{T}\mathbf{x} = 0$, with the fitted $\boldsymbol\theta$ as its normal vector. Thresholding at some other $t$ replaces the $0$ with $\log\frac{t}{1-t}$, which slides the plane along that normal without turning it. [[Decision Boundary]] carries the geometry and what it implies for tuning the threshold.

### What is minimized, and what convexity does and does not promise

The [[Cost Function]] is [[Log Loss]], the average over the training set of minus the log of the probability the model gave to the class that actually occurred. Per instance it is a two-branch expression, $-\log(\hat{p})$ when $y = 1$ and $-\log(1 - \hat{p})$ when $y = 0$, and the condition selects the branch from outside: it is a case label attached to the line, not an argument inside the logarithm, so $-\log(1 - \hat{p} \text{ if } y = 0)$ is not a compressed spelling of it but a formula with no meaning. [[Log Loss]] carries both branches and the single expression they collapse into.

There is no closed form for the minimizer. Setting the gradient $\frac{1}{m}\mathbf{X}^{T}(\sigma(\mathbf{X}\boldsymbol\theta) - \mathbf{y})$ to zero gives an equation in which $\boldsymbol\theta$ appears inside the sigmoid as well as outside it, and no rearrangement isolates it, so the fit is iterative: [[Gradient Descent]] or a second-order solver.

What makes that acceptable is convexity in $\boldsymbol\theta$, which is a property of the composition rather than of the loss alone. The Hessian is

$$\nabla^{2}_{\boldsymbol\theta} J(\boldsymbol\theta) = \frac{1}{m} \mathbf{X}^{T} \operatorname{diag}\!\big(\hat{p}^{(i)}(1 - \hat{p}^{(i)})\big) \mathbf{X}$$

and for any direction $\mathbf{v}$ the quadratic form is $\frac{1}{m} \sum_i \hat{p}^{(i)}(1 - \hat{p}^{(i)}) (\mathbf{x}^{(i)} \cdot \mathbf{v})^{2} \ge 0$, since every $\hat{p}^{(i)}(1 - \hat{p}^{(i)})$ is strictly positive. A positive semidefinite Hessian everywhere is [[Convexity]], which rules out spurious local minima: every local minimum is a global one. It is *semi*definite rather than definite, and the form vanishes in any direction with $\mathbf{X}\mathbf{v} = \mathbf{0}$, so on a rank-deficient design the minimizer is a flat set rather than a point and which member comes back depends on the solver. The $K$-class form is convex for the same reason, which is what [[Softmax Regression]] inherits. That is a real and unusual guarantee, and it is weaker than "gradient descent finds the global minimum". Three things still have to hold.

- **The step size has to be suitable.** [[Convexity]] constrains the shape of the surface, not the size of the steps taken across it. Too large a [[Learning Rate]] oscillates or diverges on a convex bowl just as it does anywhere else.
- **The run has to be long enough.** Gradient descent approaches the minimum asymptotically rather than landing on it, so a fit that stops on `max_iter` or on `tol` returns a point near the minimum, not the minimum. A `ConvergenceWarning` is the solver saying exactly this.
- **The minimum has to exist.** On linearly separable classes it does not. Scaling any separating $\boldsymbol\theta$ to $c\boldsymbol\theta$ sends every $\hat{p}^{(i)}$ toward the correct extreme and the loss monotonically toward $0$ as $c \to \infty$, so the infimum is approached and never attained, and the unpenalized maximum likelihood estimate does not exist. What a solver returns in that case is whatever the iteration cap happened to stop at. Any penalty restores a finite minimizer, and scikit-learn applies one by default.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| inverse regularization strength | $C$ | `1.0` | less regularization, more flexible boundary, more [[Overfitting]] risk | [[Cross-Validation]] over a log grid |
| penalty | none | `"l2"` | `"l1"` and `"elasticnet"` change which weights survive rather than only how large they are, so the fitted model differs in kind; `None` removes the penalty and with it the guarantee that a minimizer exists on separable data | leave `"l2"` unless you want the selection behaviour of [[Lasso Regression]] |
| solver | none | `"lbfgs"` | not ordered. It decides which penalties are legal and, on a rank-deficient design, which member of the flat minimizing set comes back | `"lbfgs"` by default, `"liblinear"` only when you want the one-versus-rest fit, `"saga"` for `"l1"` or `"elasticnet"` at scale |
| iteration cap | `max_iter` | `100` | a longer run and a closer approach to the minimum. The default is low enough that unscaled features routinely hit it and raise `ConvergenceWarning` | raise it until the warning stops, and scale the features rather than only raising it |
| class weighting | `class_weight` | `None` | `"balanced"` reweights each class by $m/(K m_k)$, which moves the boundary toward the rare class and changes the fitted model | set it under [[Class Imbalance]], as an alternative to moving $t$ |
| decision threshold | $t$ | `0.5` | fewer positives predicted, so [[Recall]] falls monotonically. [[Precision]] usually rises but not always: losing one true positive can drop it | sweep $t$ and pick the operating point off the curve rather than accepting $0.5$, see [[Precision-Recall Tradeoff]] |

$t$ is the one row here that is not a constructor argument. `predict` thresholds at $0.5$ and exposes no way to change it; moving the cut means comparing `predict_proba` yourself or wrapping the fitted estimator, which is what `FixedThresholdClassifier` and `TunedThresholdClassifierCV` are for. It earns a row because changing it changes what the model predicts.

## Failure modes

- Classes not linearly separable in the feature space: underfits; add features or change family.
- Perfectly separable classes: weights diverge without [[Regularization]], since no finite minimizer exists.
- [[Class Imbalance]]: the 0.5 threshold predicts the majority class; move $t$ or reweight, `class_weight="balanced"` being the built-in form of the second.

## Implementation

scikit-learn 1.6:

```python
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(C=1.0).fit(X_train, y_train)
proba = clf.predict_proba(X_new)[:, 1]
```
