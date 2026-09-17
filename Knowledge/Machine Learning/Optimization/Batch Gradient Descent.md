---
note_kind: method
aliases:
  - batch GD
  - BGD
  - full-batch gradient descent
  - full batch gradient descent
up: "[[Gradient Descent]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Batch gradient descent is [[Gradient Descent]] with the gradient computed over the whole training set at every step. The name is literal: each step uses the entire batch of training data $\mathbf{X}$, which is also its single biggest liability, since nothing is updated until all $m$ instances have been read.

Reach for it when the training set fits in memory and you want a path you can reason about. It is the only member of the family that actually converges to a point: on a convex cost with a step size under the stability ceiling, it settles on the optimum and stays there, which [[Stochastic Gradient Descent]] and [[Mini-Batch Gradient Descent]] do not do unless their step size is shrunk on a schedule. The price is that every step costs a full pass, so on large $m$ it is very slow.

Where it earns its place against the closed form is feature count. It scales well with the number of features, and fitting [[Linear Regression]] with hundreds of thousands of features is far faster this way than through the [[Normal Equation]] or the SVD, both of which grow faster than quadratically in $n$.

## Algorithm

1. Initialize $\boldsymbol\theta$ randomly.
2. Compute the residual vector $\mathbf{X}\boldsymbol\theta - \mathbf{y}$ over all $m$ instances at once.
3. Turn it into the gradient by weighting each residual by each feature and averaging.
4. Step against the gradient, scaled by the [[Learning Rate]] $\eta$.
5. Repeat from step 2 until the gradient is near zero, the improvement falls below the [[Tolerance]], or the epoch cap is reached.

For the mean squared error cost of linear regression, steps 2 and 3 are one matrix product,

$$\nabla_{\boldsymbol\theta} \text{MSE}(\boldsymbol\theta) = \frac{2}{m} \mathbf{X}^{T} (\mathbf{X}\boldsymbol\theta - \mathbf{y})$$

and step 4 is

$$\boldsymbol\theta^{\text{next step}} = \boldsymbol\theta - \eta \, \nabla_{\boldsymbol\theta} \text{MSE}(\boldsymbol\theta)$$

The gradient vector points uphill, so the minus sign is what turns it into a descent. Multiplying by $\eta$ is what decides how big the downhill step is. Every step here is also one [[Epoch]], since one pass over the data produces exactly one update, which makes the two counters interchangeable for this variant alone.

### What one step costs

$\mathbf{X}\boldsymbol\theta$ is $m$ dot products of length $n+1$ and $\mathbf{X}^{T}(\cdot)$ is the same work again, so one step is $O(mn)$ and $k$ epochs are $O(kmn)$: linear in the instances, linear in the features, linear in the epochs. That single expression accounts for both of the properties this variant is known by. Linear in $m$ and paid $k$ times over is what makes it slow on large training sets. Linear in $n$ is what makes it scale well with features, because the closed-form alternatives are not: their cost grows faster than quadratically in $n$, which [[Normal Equation]] sets out, where doubling $n$ merely doubles the $O(mn)$ gradient step.

### Where it ends up

On a convex cost whose curvature does not change abruptly, which is exactly the situation for MSE, batch gradient descent with a fixed step size under $2/\lambda_{\max}$ converges to the global optimum, and that optimum is the same $\boldsymbol\theta$ the [[Normal Equation]] returns in closed form. The two routes are not approximations of each other; they are two ways of solving one problem that has one answer. [[Convexity]] is what guarantees this. Without it the procedure still converges, but to whichever minimum the random initialization happened to sit above.

The convergence is asymptotic, so the iteration never reports that it has arrived. It is stopped instead, by a [[Tolerance]] on the gradient norm and an epoch cap behind it, and both of those are choices about how much precision is worth paying for.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| learning rate | $\eta$ | no library default; $0.1$ in the worked case | longer steps and faster descent, until $\eta$ passes $2/\lambda_{\max}$ and the iteration diverges instead | plot the cost per epoch over a grid of powers of ten and take the largest value whose curve falls smoothly |
| epoch cap | `n_epochs` | no library default; $1000$ in the worked case | more full passes, proportionally longer fit, no effect once the tolerance test has fired | set it high, stop on tolerance, then verify the stop was not the cap |
| stopping tolerance | $\epsilon$ | no library default | stops on a smaller gradient norm, so an earlier and cruder fit | lower it while the cost is still visibly falling at the stop |
| initial parameters | $\boldsymbol\theta^{(0)}$ | random draw | not ordered, and on a convex cost it changes only the path taken, never the destination | pin a [[Random Seed]] so the path is reproducible |

## Failure modes

- **Large $m$.** Every step reads all $m$ rows, so a fit that needs $1000$ epochs on ten million instances performs ten billion row-gradient evaluations before it stops. [[Stochastic Gradient Descent]] gets within sight of the same optimum after a fraction of one pass.
- **A training set that does not fit in memory.** The update is defined over the full $\mathbf{X}$, so there is no version of this variant that streams. That rules it out of [[Out-of-Core Learning]] entirely, which is a structural limit rather than a performance one.
- **Unscaled columns.** The epoch count is driven by the ratio of the largest curvature to the smallest, for the reason [[Gradient Descent]] sets out, and the path visibly zigzags across a stretched valley instead of running down it. [[Feature Scaling]] is the fix.
- **Reading a smooth falling curve as success.** The cost curve for this variant falls monotonically whenever $\eta$ is stable, including when $\eta$ is a hundred times too small. The curve alone cannot distinguish a converged fit from one that was cut off, so `n_epochs` has to be checked against whether the tolerance actually fired.
- **A non-convex cost.** The determinism that makes the path easy to reason about also means there is no noise to shake it out of a local minimum. A stochastic variant sometimes escapes one; this one never does.

## Implementation

scikit-learn 1.6 ships no plain full-batch gradient descent estimator. `LinearRegression` solves least squares directly, and `SGDRegressor` updates one instance at a time, so the full-batch loop is written out. NumPy, on the worked case of $m = 100$ instances and one feature plus a bias:

```python
import numpy as np
from sklearn.preprocessing import add_dummy_feature

X_b = add_dummy_feature(X)     # x0 = 1 column, so theta_0 is the bias
eta = 0.1                      # learning rate
n_epochs = 1000
m = len(X_b)

np.random.seed(42)
theta = np.random.randn(2, 1)  # random initialization

for epoch in range(n_epochs):
    gradients = 2 / m * X_b.T @ (X_b @ theta - y)
    theta = theta - eta * gradients
```

The `2 / m` factor is the one that makes this batch rather than stochastic: the gradient is averaged over every instance in $\mathbf{X}$ before a single step is taken. With $\eta = 0.1$ the resulting `theta` matches what `np.linalg.inv(X_b.T @ X_b) @ X_b.T @ y` returns to several decimals, which is the [[Normal Equation]] and this loop agreeing, as they must on a convex problem.

Adding a stopping test rather than a fixed epoch count:

```python
tol = 1e-5
for epoch in range(n_epochs):
    gradients = 2 / m * X_b.T @ (X_b @ theta - y)
    if np.linalg.norm(gradients) < tol:
        break
    theta = theta - eta * gradients
```
