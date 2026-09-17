---
note_kind: method
aliases:
  - lasso
  - LASSO
  - Lasso
  - lasso regression
  - least absolute shrinkage and selection operator
  - L1 regularization
  - l1 regularization
  - l1 penalty
  - LassoCV
up: "[[Linear Regression]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Lasso regression is [[Linear Regression]] with an $\ell_1$ term on the weights added to the training [[Cost Function]]. Structurally it is the same move as [[Ridge Regression]], one instance of [[Regularization]] with the penalty applied during training only, and the single difference is the $p$ of the [[Lp Norm]] being charged.

That difference is not cosmetic. Ridge multiplies weights down toward zero and never arrives; lasso subtracts a fixed amount and clips at zero, so weights land on exactly zero and stay there. A fitted lasso therefore names a subset of the features and ignores the rest, which makes it a feature selector as well as a regressor. Reach for it when you believe only a handful of the columns matter and you want the fit to say which, or when a sparse model is worth a little prediction accuracy. When you are not sure, [[Ridge Regression]] is the safer default, and when the surviving features are correlated with each other, [[Elastic Net Regression]] is the repair.

## Algorithm or formula

$$J(\boldsymbol\theta) = \text{MSE}(\boldsymbol\theta) + 2\alpha \sum_{i=1}^{n} \lvert \theta_i \rvert$$

As with ridge, the sum starts at $i = 1$ and the bias $\theta_0$ is not penalized, and scikit-learn's `Lasso` leaves the intercept out of the penalty too.

The factor $2$ is not decoration. `Lasso(alpha=a)` minimizes

$$\frac{1}{2m} \lVert \mathbf{y} - \mathbf{X}\mathbf{w} \rVert_2^2 + a \lVert \mathbf{w} \rVert_1 \;=\; \frac{1}{2}\Big[ \text{MSE} + 2a\lVert \mathbf{w} \rVert_1 \Big]$$

and an overall factor of one half does not move a minimum, so the $\alpha$ above and the `alpha` you pass are the same number. The $2$ in the cost function is exactly what cancels the $\tfrac{1}{2m}$ in the library's data term. The convention table in [[Ridge Regression]] does the same reconciliation for the $\ell_2$ family, and the two are not reconciled the same way, which is why an `alpha` is not portable between `Lasso`, `Ridge` and `SGDRegressor` without checking.

### Why there is no closed form

$\lvert \theta_i \rvert$ has no derivative at $\theta_i = 0$: the slope is $-1$ from the left and $+1$ from the right. So $\nabla J = \mathbf{0}$ cannot be solved for $\boldsymbol\theta$ the way [[Ridge Regression]] solves it, because $\nabla J$ does not exist at precisely the points the solution tends to sit on. The kink can be handled by replacing the gradient with a subgradient, any value in $[-1, 1]$ standing in for the missing slope at zero, which is enough to run [[Gradient Descent]].

The kink is not an obstacle that lasso overcomes; it is the mechanism. A differentiable penalty has zero slope at zero and therefore exerts no force on a weight that is already there, so nothing holds a weight at zero exactly. The $\ell_1$ penalty arrives at zero with slope $\pm 1$ still applied, so a weight is pinned there as long as the data's pull is smaller than the penalty's. The same fact seen geometrically is the corners of the $\ell_1$ unit ball sitting on the axes, which [[Lp Norm]] sets out.

### What the library actually does

scikit-learn does not use subgradient descent. `Lasso` is fitted by **coordinate descent**: hold every weight but one fixed, minimize over that one, move to the next, repeat. The one-dimensional problem has an exact solution, the soft-threshold operator, so each update is a formula rather than a search. On a design normalized so that $\mathbf{X}^{\top}\mathbf{X} = m\mathbf{I}$ the whole fit reduces to one application of it:

$$\hat{\theta}_i = \text{sign}\big(\hat{\theta}_i^{\text{OLS}}\big) \cdot \max\big(\lvert \hat{\theta}_i^{\text{OLS}} \rvert - \alpha,\; 0\big)$$

Subtract $\alpha$ from the magnitude and clip at zero. Any OLS weight smaller than $\alpha$ in magnitude becomes exactly zero; the survivors are shifted toward zero by exactly $\alpha$ and keep their sign. Checked against `Lasso(alpha=0.2, fit_intercept=False)` on such a design, both give $(1.80206, -0.77285, 0, 0)$ from an OLS solution of $(2.00206, -0.97285, 0.14370, 0.00427)$. Compare the ridge factor $m/(m+\alpha)$ in [[Ridge Regression]]: a multiplication that cannot reach zero against a subtraction that does.

`selection="random"` updates a randomly chosen coordinate instead of cycling in order. The solver is built around a nonzero threshold, which is why scikit-learn's own documentation says the `Lasso` object should not be used with `alpha=0` and directs you to `LinearRegression` instead.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| penalty strength | $\alpha$ | `1.0` | more weights hit exactly zero and the survivors shrink further, so a sparser and more biased model, moving from [[Overfitting]] toward [[Underfitting]] | log grid under [[Cross-Validation]]; `LassoCV` builds its own path of `n_alphas=100` values spanning `eps=1e-3` of the range and picks by k-fold. The user guide prefers `LassoCV` for many collinear features and `LassoLarsCV` when $m$ is very small against $n$ |
| iteration cap | `max_iter` | `1000` | coordinate descent runs longer and gets closer to the true minimum, which changes which features survive, not only how large they are | raise it whenever a `ConvergenceWarning` appears. The default is not enough on badly scaled or strongly collinear data |
| stopping tolerance | `tol` | `1e-4` | stops on a smaller improvement, so an earlier and less exact solution and a less trustworthy support | tighten it together with `max_iter` when successive fits disagree about which features are in |
| fit intercept | `fit_intercept` | `True` | `False` forces $\theta_0 = 0$, a different model unless $\mathbf{X}$ and $\mathbf{y}$ are already centered | leave `True` |
| non-negativity | `positive` | `False` | `True` constrains every weight to be at least zero, a genuine restriction on the solution | set it only when a negative weight is meaningless in the domain |
| coordinate order | `selection` | `"cyclic"` | not ordered. `"random"` picks a coordinate at random each iteration, which usually converges faster and, since convergence is only approximate, can land on a slightly different solution | leave `"cyclic"` unless convergence is slow; if you switch, pin `random_state` |

## Failure modes

- **It picks one of a group of correlated features, more or less arbitrarily.** Three columns where $x_2$ is $x_1$ plus a small amount of noise and both genuinely drive the target: `Lasso(alpha=0.5)` returns weights $(4.930,\; 0.557,\; 0)$, loading almost everything onto one of the pair. `ElasticNet(alpha=0.5, l1_ratio=0.5)` on the same data returns $(2.536,\; 2.536,\; 0)$, splitting it evenly. Read as feature selection, the lasso fit says $x_2$ barely matters, which is false. This is the case [[Elastic Net Regression]] exists for.
- **It cannot select more than $m$ features.** With $n = 200$ columns and $m = 20$ instances, a converged lasso returns at most $19$ nonzero weights at every $\alpha$ tried, no matter how many columns actually carry signal; the ceiling is $m$ less the one degree of freedom that centering under `fit_intercept=True` spends, and elastic net on the same data keeps over forty. If $n > m$ and you expect more than $m$ relevant features, lasso structurally cannot report them.
- **A non-converged fit reports the wrong features, not merely imprecise ones.** On the correlated data above, `Lasso(alpha=0.05, max_iter=1)`, a deliberately extreme cap, returns $(5.973,\; 0.0009,\; 0)$ while the converged fit returns $(0,\; 5.977,\; 0)$. The two disagree about which feature was selected. Coordinate descent raises `ConvergenceWarning` when it stops on `max_iter`; treat that warning as invalidating the support, not as a speed note.
- **Unscaled features corrupt the selection.** One $\alpha$ is subtracted from every weight, so a column recorded in small units needs a large weight and is penalized hardest for it, and a column in large units is barely charged at all. Lasso then zeroes features for having inconvenient units. Same mechanism as the ridge case, where the effective contribution of a rescaled feature collapsed from $1.0$ to $0.0000288$; put [[Standardization]] in front of the model. See [[Feature Scaling]].
- **A large $\alpha$ gives a flat line, not a zero line.** With the bias unpenalized, `Lasso(alpha=1e6)` returns every weight at exactly $0$ and an intercept equal to $\bar{y}$ to the last printed digit, which is the [[Baseline Model]] rather than a broken model.

## Implementation

scikit-learn 1.6:

```python
from sklearn.linear_model import Lasso

lasso_reg = Lasso(alpha=0.1)
lasso_reg.fit(X, y)
lasso_reg.predict([[1.5]])
```

scikit-learn 1.6, reading the selection back out. Zeroed weights are the answer to "which features does this model use", which is [[Feature Engineering]] done by the fit rather than by hand, and a direct attack on [[Irrelevant Features]]:

```python
import numpy as np

kept = np.flatnonzero(lasso_reg.coef_)
feature_names[kept]
```

scikit-learn 1.6, how you should actually fit it: scale inside a pipeline, let the path pick $\alpha$, and check that it converged.

```python
from sklearn.linear_model import LassoCV
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

model = make_pipeline(
    StandardScaler(),
    LassoCV(cv=5, max_iter=100_000, random_state=42),
)
model.fit(X_train, y_train)
model[-1].alpha_                     # alpha chosen by cross-validation
model[-1].n_iter_                    # compare against max_iter before trusting the support
```

The equivalent fit by stochastic gradient descent is `SGDRegressor(penalty="l1")`, which normalizes its data term by $m$ and so wants a different `alpha`; see the convention table in [[Ridge Regression]]. It reaches exact zeros far less reliably than coordinate descent, because a stochastic step rarely lands on the kink and stays.
