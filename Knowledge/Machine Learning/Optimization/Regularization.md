---
note_kind: concept
aliases:
  - regularize
  - regularized
  - regularisation
  - regularization term
  - regularization strength
  - penalty
  - penalty term
  - constraining the model
up: "[[Overfitting]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

Regularization is any constraint put on a learning algorithm on purpose so that it fits the training data less freely, in exchange for a smaller generalization gap. It is a category rather than a technique: adding a penalty term to the [[Cost Function]], cutting the number of parameters outright, and stopping the fit before it converges are all instances of it, and each one accepts a worse training score to buy a better score on data the model has not seen.

The test that keeps the category from being a grab bag is whether the move is meant to cost training accuracy. Collecting more data or cleaning label noise improves the training score too, so neither is regularization.

## Formal statement

Every penalized method has one shape. Take the unregularized [[Cost Function]] $J$ and add a term that charges for the size of the parameter vector:

$$J_{\text{reg}}(\boldsymbol\theta) = J(\boldsymbol\theta) + \alpha \, R(\boldsymbol\theta)$$

$R$ is the penalty, and $\alpha \ge 0$ is the regularization strength. $\alpha$ is a [[Hyperparameter]] in the strict sense: it is fixed before the fit, it does not move during it, and it is chosen outside the minimization by [[Cross-Validation]]. Both ends of the dial are known in advance. At $\alpha = 0$ the term vanishes and the fit is the unregularized one. As $\alpha$ grows the penalty dominates and the minimizer pushes every weight toward $0$. Because the intercept is left unpenalized, what is left is not the zero function but a flat line at the mean of $y$, which is [[Underfitting]] in its purest form and is exactly the [[Baseline Model]] a regressor has to beat. Everything useful sits between the two ends, spread over orders of magnitude rather than over an interval, which is why $\alpha$ is searched on a log grid.

The same thing can be written as a constraint instead of a price. Minimizing $J + \alpha R$ is the Lagrangian form of

$$\min_{\boldsymbol\theta} \; J(\boldsymbol\theta) \quad \text{subject to} \quad R(\boldsymbol\theta) \le t$$

and for every $\alpha > 0$ there is a budget $t$ that gives the identical solution, with $t$ shrinking as $\alpha$ grows. The constrained picture is the one that explains behaviour rather than just naming it: the solution sits where the contours of $J$ first touch the feasible region $\{\boldsymbol\theta : R(\boldsymbol\theta) \le t\}$, so the *shape* of that region decides what the answer looks like. That is where [[Lp Norm]] does the work, since $p$ is exactly what sets the shape, and that note carries what a corner on an axis does to the point of contact.

### The penalty is a training-time term only

$R$ belongs to the objective that is minimized and to nothing else:

$$\boldsymbol\theta^{*} = \arg\min_{\boldsymbol\theta} \Big[ J(\boldsymbol\theta, D_{\text{train}}) + \alpha R(\boldsymbol\theta) \Big], \qquad \text{reported} \;=\; \mathcal{L}\big(h_{\boldsymbol\theta^{*}}, D_{\text{test}}\big)$$

Notice that $\mathcal{L}$ on the right carries no $\alpha R$ term. The function you optimize and the number you report are therefore two different functions of the same fitted model, and that is by design: the penalty exists to bend $\boldsymbol\theta^{*}$ toward smaller weights, and once the fit has happened it has done its whole job. A ridge model's reported [[Root Mean Squared Error]] is the plain unregularized RMSE, computed exactly as it would be for an unpenalized model, and comparing a penalized model against an unpenalized one is a fair comparison for that reason.

scikit-learn draws the same line in its naming and in its API. The quantity the stochastic gradient descent solver minimizes is called the *regularized training error*, $E(\boldsymbol\theta, b) = \frac{1}{m}\sum_{i} L(y^{(i)}, f(\mathbf{x}^{(i)})) + \alpha R(\boldsymbol\theta)$, while `score()` on the same fitted estimator returns $R^2$ for a regressor and accuracy for a classifier, neither of which has a penalty term in it. The rule generalizes past linear models: anything you read off a fitted estimator, a cross-validated score or a test-set metric is unpenalized.

### Which penalty to reach for

$R$ is a norm of the weight vector, and the norm is the entire difference between the three penalized linear models. [[Ridge Regression]] penalizes the squared $\ell_2$ norm, [[Lasso Regression]] the $\ell_1$ norm, and [[Elastic Net Regression]] a weighted sum of both, mixed by a ratio $r$ that recovers ridge at $r = 0$ and lasso at $r = 1$. Each of those notes carries its own objective, with the constant factors and the fact that the bias term $\theta_0$ is left unpenalized.

At the level of which one to pick:

- **Some regularization beats none.** A little of it is almost always better than plain [[Linear Regression]], so ridge is the sensible default and unpenalized least squares is the option you should have to argue for. The mechanism is visible in the least-squares estimate itself: when the columns of $\mathbf{X}$ are close to linearly dependent the design matrix is near singular, and the estimate then becomes highly sensitive to random error in the observed target, producing a large variance. A penalty on $R(\boldsymbol\theta)$ is what bounds that. The exception is when you want the unbiased estimate rather than the accurate prediction, since a penalty biases every coefficient toward zero on purpose, and a coefficient read as an effect size is no longer reading what it claims to.
- **Lasso or elastic net when you suspect only a few features matter.** Both drive the weights of unhelpful features to exactly zero, so the fit performs feature selection as a side effect.
- **Elastic net over lasso as the general rule**, for two named circumstances. When several features are strongly correlated, lasso is likely to pick one of the group essentially at random while elastic net tends to keep both, so the selection is unstable under resampling in a way elastic net's is not. And when the number of features exceeds the number of training instances, $n > m$, lasso can select at most $m$ of them, a ceiling the elastic net does not have (Zou and Hastie 2005, the paper that introduced elastic net for exactly this reason).

### Regularization without a penalty term

Not every instance of the category adds a term to $J$. Two others matter here and both are named in the same argument.

**Cut the capacity directly.** Fewer degrees of freedom means less room to fit noise, and in a polynomial fit the degrees of freedom are the degree: dropping [[Polynomial Regression]] from degree $d$ to degree $d'$ removes $d - d'$ weights from the model outright rather than shrinking them. A weight penalty is the softer version of the same move, leaving the parameters in place and making them expensive instead of impossible, which is usually preferable because the data gets to decide which ones survive.

**Stop the fit early.** [[Early Stopping]] never touches the objective at all; it stops the optimizer while the validation error is at its minimum and keeps those parameters, so nothing is ever charged for the size of a weight. The weights stay small anyway, because a fit that is stopped short never had the budget to grow them, and on a linear model with a quadratic error surface that resemblance sharpens into an exact correspondence with an $\ell_2$ penalty. [[Early Stopping]] carries it.

### In scikit-learn

scikit-learn 1.6: the advice to avoid an unregularized fit is, for most of the library, a default it already enforces. `LogisticRegression` ships with `penalty="l2"` and `C=1.0`, and its reference page says so outright: regularization is applied by default. `SGDClassifier` and `SGDRegressor` both default to `penalty="l2"` with `alpha=1e-4`. `LinearSVC` defaults to `C=1.0`. `LinearRegression` is the odd one out, having no penalty argument at all, which is why the penalized variants are separate classes rather than a keyword.

Two arguments name the strength and they run in opposite directions, which is the single easiest thing to get backwards:

| argument | estimators | direction | default |
|---|---|---|---|
| `alpha` | `Ridge`, `Lasso`, `ElasticNet`, `SGDRegressor`, `SGDClassifier` | larger means more regularization | `1.0` for the first three, `1e-4` for the SGD pair |
| `C` | `LogisticRegression`, `LinearSVC`, `SVC` | inverse of the strength, so smaller means more regularization | `1.0` |

```python
from sklearn.linear_model import LogisticRegression, SGDRegressor, RidgeCV
import numpy as np

LogisticRegression()                # penalty="l2", C=1.0: already regularized
LogisticRegression(penalty=None)    # the unpenalized fit, which you have to ask for

SGDRegressor()                      # penalty="l2", alpha=1e-4
SGDRegressor(penalty=None)          # no penalty at all

# Pick the strength by cross-validation rather than by hand.
ridge_cv = RidgeCV(alphas=np.logspace(-4, 2, 30)).fit(X_train, y_train)
ridge_cv.alpha_
```

`RidgeCV` defaults to `alphas=(0.1, 1.0, 10.0)`, which is three points and almost never the grid you want. Its `cv=None` default runs an efficient leave-one-out cross-validation scored by negative mean squared error, so it is cheap enough that widening `alphas` costs little. `LassoCV` and `ElasticNetCV` are the corresponding classes for the other two penalties.

One precondition applies to every penalized fit and to none of the capacity-cutting ones. A single $\alpha$ charges every weight at one rate, which is only a fair price if a unit of each feature means a comparable amount, so [[Feature Scaling]] comes first and belongs in the same [[Pipeline]] as the estimator. Skipping it does not merely slow the fit, it changes which coefficients get shrunk.

## Where it is used

This is the remedy [[Overfitting]] exists to be remedied by, and the failure at the other end, [[Underfitting]], is what too much of it produces; the dial between the two is the [[Bias-Variance Tradeoff]], with more regularization buying lower variance at the price of higher bias. A [[Learning Curve]] is the plot that says which end you are currently on and therefore which way to turn $\alpha$. In the [[Cost Function]] it appears as an added term, which is the reason the training objective and the reported [[Performance Measure]] stop being the same function. Its strength is a [[Hyperparameter]], so choosing it is [[Model Selection]] work done by [[Cross-Validation]] on validation folds and never on the [[Testing Set]]. [[Ridge Regression]], [[Lasso Regression]] and [[Elastic Net Regression]] are the three standard penalties on a linear model, distinguished only by which [[Lp Norm]] they charge, and [[Early Stopping]] is the member of the family that reaches the same effect with no penalty term at all.
