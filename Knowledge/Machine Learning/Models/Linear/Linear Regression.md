---
note_kind: method
aliases:
  - ordinary least squares
  - OLS
  - linear model
  - multiple regression
up: "[[Regression]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Linear regression predicts a continuous target as a weighted sum of the features plus a bias, with the weights chosen to minimize squared error on the training set. Use it when the relation is close to linear, when interpretability matters, or as the baseline every other regressor must beat. Belongs to the [[Regression]] map and is the worked example of [[Model-Based Learning]].

## Algorithm or formula

$$\hat{y} = h_{\boldsymbol\theta}(\mathbf{x}) = \theta_0 + \theta_1 x_1 + \dots + \theta_n x_n = \boldsymbol\theta^{T} \mathbf{x}$$

with $x_0 = 1$. Reading the symbols: $\hat{y}$ is the predicted value, $n$ the number of features, $x_j$ the $j$th feature value of the instance, and $\theta_j$ the $j$th parameter, where $\theta_0$ is the bias term (the intercept) and $\theta_1$ through $\theta_n$ are the feature weights. The vectorized form collects the parameters into $\boldsymbol\theta$ and the instance's features into $\mathbf{x}$, with $x_0 = 1$ prepended so the bias is carried as an ordinary weight, a convention [[Notation]] records along with the column-vector shapes; $h_{\boldsymbol\theta}$ is the hypothesis, and $\boldsymbol\theta^{T}\mathbf{x}$ is the dot product $\boldsymbol\theta \cdot \mathbf{x} = \theta_0 x_0 + \theta_1 x_1 + \dots + \theta_n x_n$.

The vectorized form takes one instance's feature vector $\mathbf{x}$, a single row. The design matrix $\mathbf{X}$ is the whole training set stacked, and the corresponding statement for all of it at once is $\hat{\mathbf{y}} = \mathbf{X}\boldsymbol\theta$, a vector in $\mathbb{R}^{m}$ rather than a scalar. Substituting $\mathbf{X}$ for $\mathbf{x}$ in the single-instance formula is a shape error, not a shorthand.

The [[Cost Function]] is mean squared error,

$$J(\boldsymbol\theta) = \frac{1}{m} \sum_{i=1}^{m} \big(\boldsymbol\theta^{T}\mathbf{x}^{(i)} - y^{(i)}\big)^2$$

and training sets $\boldsymbol\theta^{*} = \arg\min J$.

Two routes reach that minimum, and they minimize the same $J$. [[Normal Equation]] solves $\nabla J = \mathbf{0}$ analytically and lands on the exact minimizer in one matrix solve. [[Gradient Descent]] starts from an arbitrary $\boldsymbol\theta$ and steps against $\nabla J$ until it stops improving. Because $J$ is a convex quadratic in $\boldsymbol\theta$ it has one minimum and no local ones, so the two agree, with one qualification worth stating precisely: [[Batch Gradient Descent]] converges to that minimum given a small enough step size and enough iterations, whereas [[Stochastic Gradient Descent]] and [[Mini-Batch Gradient Descent]] at a fixed step size only reach its neighbourhood and then keep bouncing around it, never settling. They settle only if the step size is decayed on a [[Learning Schedule]]. So "the same parameters" is exact for the closed form and for batch gradient descent, and approximate for the noisy variants.

Plain least squares is rarely the model you should ship. A little [[Regularization]] is almost always better than none, so [[Ridge Regression]] is the sensible default and an unpenalized fit is the option you should have to argue for; this note is the family the penalized variants are penalizing.

### Choosing a fitting route

| Algorithm | Large $m$ | Out-of-core support | Large $n$ | Tunable parameters | Scaling required | scikit-learn |
|---|---|---|---|---|---|---|
| [[Normal Equation]] | fast | no | slow | 0 | no | none exposed |
| SVD pseudoinverse | fast | no | slow | 0 | no | `LinearRegression` |
| [[Batch Gradient Descent]] | slow | no | fast | 2 | yes | none exposed |
| [[Stochastic Gradient Descent]] | fast | yes | fast | 2 or more | yes | `SGDRegressor` |
| [[Mini-Batch Gradient Descent]] | fast | yes | fast | 2 or more | yes | none exposed |

The pattern falls out of the arithmetic. The two direct routes cost $O(m n^{2})$ to form the products and then between $O(n^{2})$ and $O(n^{3})$ to solve, so they are linear in $m$ and superlinear in $n$: comfortable on tall data, unusable on wide data. One gradient step costs $O(mn)$ with no $n^{2}$ term anywhere, which reverses both columns: fast in $n$, and slow in $m$ for the batch variant because every step reads the whole training set, while the stochastic and mini-batch variants read one row or one block per step and so stop caring about $m$ altogether.

Out-of-core support is the same distinction stated as memory: only an algorithm with an incremental update can be fed the data in chunks, which is what `SGDRegressor.partial_fit` provides and what a closed form cannot. Mini-batch gradient descent is marked as unexposed despite that method, because `partial_fit` performs one per-instance update for every row of the block handed to it rather than one averaged update for the block; it is stochastic gradient descent fed in chunks.

Scaling required is the ravine argument from [[Feature Scaling]]: columns of very different scale stretch the contours of $J$ and force a small step size, which costs the gradient routes iterations. The direct routes solve the system outright and are indifferent to it.

The two tunable parameters counted for batch gradient descent are the [[Learning Rate]] and the iteration budget, the latter usually expressed as a [[Tolerance]] plus a cap on the number of epochs. The stochastic and mini-batch rows read "2 or more" because they add at least the [[Learning Schedule]], and mini-batch adds the batch size on top of that.

## Hyperparameters

None for the plain fit. Regularized variants add a penalty weight $\alpha$: [[Ridge Regression]], [[Lasso Regression]] and [[Elastic Net Regression]], which also mixes the two penalties with a ratio. [[Polynomial Regression]] adds a degree, though it does so by changing the features rather than the model.

Two switches on `LinearRegression` do change the fitted output without being tunable dials. `fit_intercept=False` forces the fit through the origin, which is a claim about the data rather than a setting to search over, and `positive=True` constrains every coefficient to be non-negative, which changes the solver to non-negative least squares and is a modelling decision made on domain grounds.

## Failure modes

- Nonlinear relation: the model underfits ([[Underfitting]]) and no amount of data helps.
- Outliers: squared error lets a few far points dominate the fit.
- Highly correlated features: weights become unstable and uninterpretable.

## Implementation

scikit-learn 1.6. The direct route, which solves the least squares problem through the SVD-based pseudoinverse:

```python
from sklearn.linear_model import LinearRegression
model = LinearRegression().fit(X_train, y_train)
y_pred = model.predict(X_new)
```

scikit-learn 1.6. The iterative route, for when $n$ is large or the data arrives in pieces. `penalty=None` is what makes this plain linear regression rather than a regularized variant, since the default is `"l2"`:

```python
from sklearn.linear_model import SGDRegressor

sgd_reg = SGDRegressor(max_iter=1000, tol=1e-5, penalty=None, eta0=0.01,
                       n_iter_no_change=100, random_state=42)
sgd_reg.fit(X, y.ravel())  # y.ravel() because fit() expects 1D targets
```

`tol` and `n_iter_no_change` are the stopping rule together: training halts when the loss fails to improve by more than `tol` for that many consecutive epochs. Both are set well away from their defaults of `1e-3` and `5` here, to make the run keep going until it has genuinely stopped improving. Put a scaler in front of it in a [[Pipeline]]; unlike `LinearRegression`, this one is sensitive to column scale.
