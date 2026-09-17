---
note_kind: method
aliases:
  - ridge
  - Ridge
  - ridge regression
  - Tikhonov regularization
  - Tikhonov
  - L2 regularization
  - l2 regularization
  - l2 penalty
  - ridge regularization
  - RidgeCV
up: "[[Linear Regression]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Ridge regression is [[Linear Regression]] with an $\ell_2$ term on the weights added to the training [[Cost Function]], so the fit has to explain the data while keeping the weights as small as it can. It is one instance of the argument in [[Regularization]], and the penalty exists only during training: the number you report on held-out data is the plain unpenalized error.

Reach for it as the default whenever least squares has more freedom than the data justifies, which is the usual case for a polynomial expansion, for many features, or for correlated features. It shrinks every weight toward zero and sets none of them to exactly zero, so it constrains the model without removing any [[Feature]]. When you want features removed outright that is [[Lasso Regression]], and when you want both shrinkage and removal it is [[Elastic Net Regression]]. The choice between the three is a choice of $p$ in the [[Lp Norm]] used for the penalty, and nothing else.

## Algorithm or formula

The model is unchanged, $\hat{y} = \boldsymbol\theta^{\top}\mathbf{x}$. What changes is what training minimizes:

$$J(\boldsymbol\theta) = \text{MSE}(\boldsymbol\theta) + \frac{\alpha}{m} \sum_{i=1}^{n} \theta_i^{2}$$

Two details of that sum carry weight. It runs from $i = 1$, so the bias $\theta_0$ is not in it, and it is a sum over the $n$ features rather than the $m$ instances. Leaving $\theta_0$ out is not an oversight: the bias sets the level of the prediction, so penalizing it would tie the fit to where the origin of $y$ happens to sit, and adding a constant to every target would change the model in a way that is not a property of the data. scikit-learn does the same, which is why `intercept_` is reported separately from `coef_` and why `fit_intercept=False` on uncentered data is a different model rather than a cosmetic change.

$\alpha$ is the dial. At $\alpha = 0$ the penalty vanishes and this is plain [[Linear Regression]]. As $\alpha$ grows the weights are driven toward zero and the fit flattens toward [[Underfitting]]; see the Failure modes below for what "flattens" actually means, since it is not what it sounds like.

### Closed form

Ridge extends the [[Normal Equation]] by one additive term:

$$\hat{\boldsymbol\theta} = \left( \mathbf{X}^{\top}\mathbf{X} + \alpha \mathbf{A} \right)^{-1} \mathbf{X}^{\top} \mathbf{y}$$

$\mathbf{A}$ is the $(n+1) \times (n+1)$ identity matrix with its top left cell set to zero, which is exactly how the bias is kept out of the penalty. For a single feature,

$$\mathbf{A} = \begin{pmatrix} 0 & 0 \\ 0 & 1 \end{pmatrix}$$

The inverse is taken of the whole bracket. Writing $\mathbf{A}^{-1}$ instead is impossible as well as wrong: a matrix with a zero row is singular and has no inverse.

Adding $\alpha\mathbf{A}$ lifts the penalized diagonal entries of $\mathbf{X}^{\top}\mathbf{X}$, and that is what makes the bracket invertible when the columns of $\mathbf{X}$ are collinear and $\mathbf{X}^{\top}\mathbf{X}$ on its own is singular. Ridge therefore has a unique solution in cases where ordinary least squares has a whole subspace of them, which is a second reason to prefer it over a bare fit.

Under an orthonormal design scaled so that $\mathbf{X}^{\top}\mathbf{X} = m\mathbf{I}$, the closed form collapses to $\hat{\theta}_i = \hat{\theta}_i^{\text{OLS}} \cdot \frac{m}{m + \alpha}$: every weight is multiplied by the same factor strictly between $0$ and $1$, so no weight ever reaches zero. [[Lasso Regression]] draws the contrast against its own operator.

### What alpha means in each estimator

The same symbol names different constants across the library, and the difference is silent. Writing $\text{SSE} = \lVert \mathbf{y} - \mathbf{X}\mathbf{w} \rVert_2^2 = m \cdot \text{MSE}$:

| objective | what is minimized | equivalent to the form above with |
|---|---|---|
| the cost above | $\text{MSE} + \frac{\alpha}{m}\lVert\mathbf{w}\rVert_2^2$ | $\alpha$ |
| `Ridge(alpha=a)` | $\text{SSE} + a\lVert\mathbf{w}\rVert_2^2$ | $\alpha = a$ |
| `SGDRegressor(penalty="l2", alpha=a)` | $\frac{1}{2}\text{MSE} + \frac{a}{2}\lVert\mathbf{w}\rVert_2^2$ | $\alpha = a \cdot m$ |
| `ElasticNet(alpha=a, l1_ratio=0)` | $\frac{1}{2}\text{MSE} + \frac{a}{2}\lVert\mathbf{w}\rVert_2^2$ | $\alpha = a \cdot m$ |

The first two rows differ by an overall factor of $m$, which does not move the minimum, so the $\alpha$ in the cost function and the `alpha` you hand to `Ridge` are the same number. The last two rows divide the data term by $m$ and leave the penalty alone, which makes the penalty $m$ times heavier for the same `alpha`.

That is the arithmetic behind `SGDRegressor(penalty="l2", alpha=0.1 / m)` being the correct partner for `Ridge(alpha=0.1)`, and it is worth stating plainly because getting it wrong produces a different model rather than an error. On $m = 100$ points, `Ridge(alpha=0.1)` fits $\theta_0 = 1.1441$, $\theta_1 = 0.3973$; the SGD route with `alpha=0.1 / m` lands on $1.1444$ and $0.3980$, and the same call with `alpha=0.1` lands on $1.2068$ and $0.3537$, which is the fit you would get from `Ridge(alpha=10)`.

### Gradient descent route

The penalty contributes $\frac{2\alpha}{m}\boldsymbol\theta$ to the gradient (with the $\theta_0$ entry held at zero), so the [[Gradient Descent]] update becomes

$$\boldsymbol\theta \leftarrow \boldsymbol\theta - \eta\left[ \nabla_{\boldsymbol\theta}\text{MSE}(\boldsymbol\theta) + \frac{2\alpha}{m}\boldsymbol\theta \right]$$

Every step scales the current weights down before moving on the data gradient, which is why the $\ell_2$ penalty is also called weight decay. [[Stochastic Gradient Descent]] does the same thing one instance at a time, which is what `SGDRegressor(penalty="l2")` is.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| penalty strength | $\alpha$ | `1.0` | weights shrink further, variance falls, bias rises, [[Overfitting]] gives way to [[Underfitting]] | log grid under [[Cross-Validation]], with `RidgeCV` as the built-in route. Supply your own `alphas`; the three-point default is there to make the class runnable, not to be used |
| solver | `solver` | `"auto"` | not ordered. `"auto"` picks in this order: `"lbfgs"` if `positive=True`, `"sag"` if the solver returns the intercept, `"cholesky"` for dense, `"sparse_cg"` for sparse | leave at `"auto"`. Move to `"svd"`, the most stable of them on a singular or near-singular $\mathbf{X}$, if the fit is numerically unstable, and to the iterative `"sag"` or `"saga"` when both $m$ and $n$ are large. The solver changes the numerical path, and on a well-conditioned problem all of them should agree to several digits; a disagreement is a conditioning warning, not a choice |
| fit intercept | `fit_intercept` | `True` | `False` forces $\theta_0 = 0$, which is a different model unless the data is already centered. It also removes the one weight the penalty was deliberately not charging | leave `True` unless you have centered $\mathbf{X}$ and $\mathbf{y}$ yourself |
| non-negativity | `positive` | `False` | `True` constrains every weight to be at least zero, which is a real constraint on the solution and forces `solver="lbfgs"` | set it only when negative weights are meaningless in the domain |
| iteration cap | `max_iter` | `None` | more iterations for the iterative solvers, so a solution closer to the true minimum. Inert for `"svd"` and `"cholesky"`, which are direct | raise it if an iterative solver stops short. `None` means each solver's own default: `1000` for `"sag"` and `"saga"`, `15000` for `"lbfgs"`, and whatever scipy decides for `"sparse_cg"` and `"lsqr"` |
| stopping tolerance | `tol` | `1e-4` | stops on a smaller improvement, so an earlier and less exact solution. Inert for the direct solvers | lower it when successive fits disagree more than the effect you are measuring |

`random_state` is left out on purpose: it seeds the shuffling in `"sag"` and `"saga"` and so changes the answer run to run without being a quantity you tune. Pin it with an integer, see [[Random Seed]].

## Failure modes

- **Unscaled features make $\alpha$ mean something different per column.** One $\alpha$ charges every weight at the same rate, so a feature recorded in small units needs a large weight to contribute at all, and that large weight is exactly what the penalty destroys. Two features contributing equally to the target, $y = a_1 + a_2 + \varepsilon$, fitted at $\alpha = 10$ on $m = 300$ points: on comparable scales the weights come back $0.974$ and $0.960$. Record the second column in units one thousand times smaller and the weights become $0.991$ and $0.0288$, an effective contribution of $0.0000288$ per unit of the original quantity. The second feature has been silently deleted by a change of units. Put [[Standardization]] in front of the model, inside a [[Pipeline]]; this is the "penalty" mechanism in [[Feature Scaling]].
- **A large $\alpha$ sends the weights to zero but the prediction to the mean of $y$, not to zero.** Because $\theta_0$ is not penalized, it absorbs the level of the target. `Ridge(alpha=1e12)` on data with $\bar{y} = 1.70455087$ returns $\theta_1 = 3 \times 10^{-11}$ and $\theta_0 = 1.70455087$: a flat line at the mean, which is the [[Baseline Model]] a regressor has to beat, not a line at zero.
- **An $\alpha$ carried between estimators quietly refits a different model.** `Ridge`, `SGDRegressor` and `ElasticNet` normalize the data term differently, per the table above, so the same `alpha` is up to $m$ times stronger in one than in another. Nothing raises an error; you get a worse model and no signal that a constant is wrong.
- **No feature selection, by construction.** Ridge multiplies weights by a factor short of one and never reaches zero, so a fit with a thousand irrelevant columns still uses all thousand of them at prediction time. If sparsity is the goal, ridge cannot supply it at any $\alpha$; see [[Lasso Regression]].
- **Correlated features get their weight split rather than resolved.** The penalty prefers two weights of $3$ to one of $6$, since $3^2 + 3^2 < 6^2$, so a group of near-duplicate columns shares the coefficient between them. That is stable and good for prediction, but it means a single coefficient cannot be read as the effect of its feature.

## Implementation

scikit-learn 1.6, the direct route:

```python
from sklearn.linear_model import Ridge

ridge_reg = Ridge(alpha=0.1, solver="cholesky")
ridge_reg.fit(X, y)
ridge_reg.predict([[1.5]])
```

scikit-learn 1.6, the same model fitted by stochastic gradient descent. Note `alpha=0.1 / m`, which is the constant that makes the two objectives agree:

```python
from sklearn.linear_model import SGDRegressor

sgd_reg = SGDRegressor(penalty="l2", alpha=0.1 / m, tol=None,
                       max_iter=1000, eta0=0.01, random_state=42)
sgd_reg.fit(X, y.ravel())   # ravel() because fit() expects a 1D target
sgd_reg.predict([[1.5]])
```

scikit-learn 1.6, the closed form written out, which reproduces the first block to every digit printed:

```python
import numpy as np

alpha = 0.1
A = np.array([[0., 0.], [0., 1.]])       # identity with the bias cell zeroed
X_b = np.c_[np.ones(m), X]               # prepend the x0 = 1 column
np.linalg.inv(X_b.T @ X_b + alpha * A) @ X_b.T @ y
```

scikit-learn 1.6, how you should actually fit it: scale inside a pipeline and let cross-validation pick $\alpha$.

```python
import numpy as np
from sklearn.linear_model import RidgeCV
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

model = make_pipeline(
    StandardScaler(),
    RidgeCV(alphas=np.logspace(-4, 4, 50)),
)
model.fit(X_train, y_train)
model[-1].alpha_        # the alpha chosen by leave-one-out cross-validation
```
