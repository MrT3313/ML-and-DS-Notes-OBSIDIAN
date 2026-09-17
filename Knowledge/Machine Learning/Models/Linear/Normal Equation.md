---
note_kind: method
aliases:
  - normal equations
  - closed form solution
  - closed-form solution
  - pseudoinverse
  - Moore-Penrose pseudoinverse
  - least squares closed form
up: "[[Linear Regression]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

The normal equation is the closed form for [[Linear Regression]]: it computes the parameter vector that minimizes squared error directly, by solving for the point where the gradient vanishes, rather than walking toward it. One matrix solve and the fit is exact, with no step size, no stopping rule, and no [[Feature Scaling]] needed, since nothing here follows a slope whose shape the column units could distort.

Reach for it when the feature count $n$ is modest, in the hundreds or low thousands, and the training data fits in memory. The cost is linear in the instance count $m$ and superlinear in $n$, so it stays comfortable on tall data and becomes the wrong tool on wide data. Reach for [[Gradient Descent]] when $n$ is large, and for [[Stochastic Gradient Descent]] when the data must be streamed, because a closed form has no incremental update and therefore no [[Out-of-Core Learning]] story at all: it needs every row present before it can compute anything.

## Algorithm or formula

Stack the training set into the design matrix $\mathbf{X}$ of shape $m \times (n+1)$, one row per instance, with a leading column of ones carrying $x_0 = 1$ so the bias term rides along as $\theta_0$. Collect the targets into $\mathbf{y} \in \mathbb{R}^{m}$ and the parameters into $\boldsymbol\theta \in \mathbb{R}^{n+1}$. The [[Cost Function]] is mean squared error, written in matrix form,

$$J(\boldsymbol\theta) = \frac{1}{m} \lVert \mathbf{X}\boldsymbol\theta - \mathbf{y} \rVert_2^2 = \frac{1}{m}\big(\boldsymbol\theta^{T}\mathbf{X}^{T}\mathbf{X}\boldsymbol\theta - 2\,\mathbf{y}^{T}\mathbf{X}\boldsymbol\theta + \mathbf{y}^{T}\mathbf{y}\big)$$

which is the square of [[Root Mean Squared Error]]; the root is strictly increasing on $[0,\infty)$, so both are minimized by the same $\boldsymbol\theta$.

### Deriving it

Differentiate with respect to $\boldsymbol\theta$. The quadratic term contributes $2\mathbf{X}^{T}\mathbf{X}\boldsymbol\theta$, the linear term contributes $-2\mathbf{X}^{T}\mathbf{y}$, and the constant drops:

$$\nabla_{\boldsymbol\theta} J = \frac{2}{m}\,\mathbf{X}^{T}\big(\mathbf{X}\boldsymbol\theta - \mathbf{y}\big)$$

Set it to zero. The factor $2/m$ divides out, which is why minimizing the summed squared error and the mean squared error give the identical answer, and what is left is the system that gives the method its name:

$$\mathbf{X}^{T}\mathbf{X}\,\boldsymbol\theta = \mathbf{X}^{T}\mathbf{y}$$

When $\mathbf{X}^{T}\mathbf{X}$ is invertible, solve it:

$$\hat{\boldsymbol\theta} = \big(\mathbf{X}^{T}\mathbf{X}\big)^{-1}\mathbf{X}^{T}\mathbf{y}$$

Two things finish the derivation. First, a vanishing gradient only marks a stationary point, but $J$ is a convex quadratic (see [[Convexity]]), so the stationary point is the global minimum. Second, the name is geometric. Rearranged, the condition reads $\mathbf{X}^{T}(\mathbf{X}\hat{\boldsymbol\theta} - \mathbf{y}) = \mathbf{0}$: the residual vector is orthogonal, normal, to every column of $\mathbf{X}$, so $\mathbf{X}\hat{\boldsymbol\theta}$ is the orthogonal projection of $\mathbf{y}$ onto the column space.

Shapes, since they are what makes the formula readable: $\mathbf{X}^{T}\mathbf{X}$ is $(n+1) \times (n+1)$, $\mathbf{X}^{T}\mathbf{y}$ is $(n+1) \times 1$, and $\hat{\boldsymbol\theta}$ is $(n+1) \times 1$. Note that $m$ appears nowhere in the shape of the object being inverted; it only sets how long the products take to form. Transpose, inverse and rank are the [[Mathematics]] here, and each of them is a statement about those shapes.

### The pseudoinverse, which is what actually runs

scikit-learn never forms $(\mathbf{X}^{T}\mathbf{X})^{-1}$. `LinearRegression` computes

$$\hat{\boldsymbol\theta} = \mathbf{X}^{+}\mathbf{y}$$

where $\mathbf{X}^{+}$ is the Moore-Penrose pseudoinverse, obtained from the singular value decomposition $\mathbf{X} = \mathbf{U}\boldsymbol\Sigma\mathbf{V}^{T}$ as $\mathbf{X}^{+} = \mathbf{V}\boldsymbol\Sigma^{+}\mathbf{U}^{T}$. Building $\boldsymbol\Sigma^{+}$ is where the robustness comes from: every singular value above a cutoff is replaced by its reciprocal, and every singular value at or below the cutoff is replaced by zero rather than by an enormous number.

The scikit-learn user guide states the mechanism plainly: the least squares solution is computed using the singular value decomposition of $\mathbf{X}$. In 1.6 the dense path calls `scipy.linalg.lstsq`, which is SVD based, passing `cond = max(X.shape) * eps` as that cutoff. Two other paths exist and are worth knowing about, because neither is this one: a sparse `X` is solved by the iterative `scipy.sparse.linalg.lsqr`, and `positive=True` routes to `scipy.optimize.nnls`. The fitted estimator exposes `rank_` and `singular_`, which is the direct read on whether the decomposition found a deficiency.

Avoiding $\mathbf{X}^{T}\mathbf{X}$ is the point of that route rather than an implementation detail. Forming the product squares the condition number, $\kappa(\mathbf{X}^{T}\mathbf{X}) = \kappa(\mathbf{X})^{2}$, so a design that is merely badly scaled becomes numerically singular in the Gram matrix while the SVD of $\mathbf{X}$ itself is untroubled. Writing the closed form as an inverse is how the equation is stated; it is not how it should be computed.

This buys two things that the inverse cannot give.

- $\mathbf{X}^{+}$ is defined for every matrix, including ones where $\mathbf{X}^{T}\mathbf{X}$ is singular and the inverse simply does not exist. Where infinitely many $\boldsymbol\theta$ minimize the error equally well, the pseudoinverse returns the one of smallest $\lVert \boldsymbol\theta \rVert_2$, so a definite answer comes back instead of an exception.
- Singularity is **not** only the wide case $m < n$. That case guarantees it, since $\mathbf{X}$ cannot have more than $m$ independent columns. But rank deficiency arrives at any $m$ from duplicated columns, from a feature that is a linear combination of others, or from a full one-hot block, whose levels sum to a constant: centered under `fit_intercept=True` they sum to the zero vector, and alongside a hand-added column of ones they sum to exactly that column. Short of exact deficiency, the user guide's multicollinearity warning applies: near dependence leaves the estimate highly sensitive to noise in $\mathbf{y}$, which shows up as large-variance, uninterpretable weights.

On "more efficient", keep two claims apart. The complexity gap between $O(n^{2})$ and $O(n^{3})$ is real but modest, and it is not the reason the library takes this route. The stronger and more useful claim is numerical: the SVD returns a sensible answer on rank-deficient and ill-conditioned matrices where the explicit inverse either fails outright or returns weights inflated by the reciprocal of a near-zero pivot. Robustness is the argument; speed is a side benefit.

## Hyperparameters

None for the plain fit. The two `LinearRegression` switches that change the fitted output without being tunable dials, `fit_intercept` and `positive`, are in [[Linear Regression]]. (The singular value cutoff named above is computed internally and is not exposed as an argument either.) [[Ridge Regression]] is this same closed form with one term added, $\hat{\boldsymbol\theta} = (\mathbf{X}^{T}\mathbf{X} + \alpha\mathbf{A})^{-1}\mathbf{X}^{T}\mathbf{y}$, and its $\alpha$ is a real hyperparameter; [[Lasso Regression]] and [[Elastic Net Regression]] add a penalty weight too but have no closed form to add it to.

## Failure modes

- Wide data. At $n = 10{,}000$ features the $(n+1) \times (n+1)$ matrix alone is a hundred million entries, and the solve is superlinear on top of that. A [[Batch Gradient Descent]] step costs $O(mn)$ and does not care.
- Rank-deficient or near-deficient $\mathbf{X}$. The textbook formula raises on the singular case; the pseudoinverse survives it but silently returns the minimum-norm member of an infinite solution set, so weights on the offending columns are an arbitrary split and must not be read as feature importances. Check `rank_` against `n_features_in_` before interpreting any coefficient: under `fit_intercept=True` the estimator centers $\mathbf{X}$ and $\mathbf{y}$ and decomposes the centered matrix with no constant column attached, so a healthy fit reports `rank_ == n_features_in_` and anything lower is a deficiency.
- Streaming or growing data. There is no incremental update, so every new batch means refitting from scratch over the whole dataset. This is why [[Out-of-Core Learning]] rules the closed form out and reaches for `partial_fit` on [[Stochastic Gradient Descent]] instead.
- Hand-rolling `np.linalg.inv(X.T @ X) @ X.T @ y` in production. It is the right thing to write once, to see the formula work, and the wrong thing to ship: it is both slower and less stable than the `lstsq` call the library already makes.

## Implementation

scikit-learn 1.6. The literal formula first, which is worth running once to confirm it reproduces what the library returns:

```python
import numpy as np
from sklearn.preprocessing import add_dummy_feature

X_b = add_dummy_feature(X)  # prepend a column of ones, so x0 = 1 on every row
theta_best = np.linalg.inv(X_b.T @ X_b) @ X_b.T @ y

X_new = np.array([[0], [2]])
X_new_b = add_dummy_feature(X_new)
y_predict = X_new_b @ theta_best
```

`add_dummy_feature(X, value=1.0)` lives in `sklearn.preprocessing` and returns a copy with the constant column inserted in front, turning an $m \times n$ array into $m \times (n+1)$.

The same fit through the pseudoinverse, and then through the estimator that wraps it:

```python
theta_best_svd = np.linalg.pinv(X_b) @ y          # X^+ y, computed by SVD

from sklearn.linear_model import LinearRegression
lin_reg = LinearRegression().fit(X, y)            # no dummy column needed here
lin_reg.intercept_, lin_reg.coef_                 # bias and weights, kept apart
lin_reg.rank_, lin_reg.singular_                  # rank check and spectrum
```

`LinearRegression` adds and removes the constant column itself under `fit_intercept=True`, which is why `X` goes in raw and the bias comes back separately in `intercept_` rather than as the first entry of `coef_`.
