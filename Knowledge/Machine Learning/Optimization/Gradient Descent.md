---
note_kind: method
aliases:
  - GD
  - gradient descent
  - steepest descent
  - gradient-based optimization
up: "[[Cost Function]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Two routes lead to the parameters that fit a model. A closed form computes them directly, solving in one shot for the values that minimize the [[Cost Function]] over the [[Training Set]]. Gradient descent takes the other route: start the parameters anywhere, then tweak them iteratively, each step a little further downhill, until they stop moving. Where they stop depends on which variant is running. On a convex cost with the gradient taken over the whole [[Training Set]] they stop on the same values the closed form would have produced; the variants that compute a gradient from a sample settle into a neighbourhood of that point instead, and [[Stochastic Gradient Descent]] carries the condition under which they close on it. Same destination, different means of travel, and the iterative route is the one that generalizes.

It generalizes because the closed form is a privilege rather than a right. [[Linear Regression]] happens to have one, the [[Normal Equation]], and paying for it means inverting an $(n+1) \times (n+1)$ matrix, which costs on the order of $n^{3}$ and so becomes unaffordable long before $n$ becomes unusual. One gradient step costs $O(mn)$, linear in both. Most models have no closed form at all, so gradient descent is the default and not the fallback.

Gradient descent optimizes; it does not model. What it minimizes is whatever cost you hand it, and the model never enters except through that cost.

The family splits three ways, and the three differ in exactly one respect: how many instances one gradient is computed on. [[Batch Gradient Descent]] uses all $m$ of them, [[Stochastic Gradient Descent]] uses one, [[Mini-Batch Gradient Descent]] uses a small random $b$ in between. Everything else, the update rule, the [[Learning Rate]], the stopping test, is shared.

## Algorithm

1. **Initialize.** Fill the parameter vector $\boldsymbol\theta$ with random values. This is random initialization, and on a cost with more than one minimum it is also the reason two runs can land in different places unless a [[Random Seed]] is pinned.
2. **Measure the local slope.** For each parameter $\theta_j$, compute how much the cost changes when $\theta_j$ alone is nudged: the [[Partial Derivative]] $\partial J / \partial \theta_j$, which is the slope of the line that nudge traces out.
3. **Collect the slopes.** Stacking one partial derivative per parameter gives the [[Gradient]] $\nabla_{\boldsymbol\theta} J(\boldsymbol\theta)$, a vector in [[Parameter Space]] pointing in the direction of steepest increase.
4. **Step the other way.** The gradient points uphill, so go against it, scaled by the [[Learning Rate]] $\eta$, which is the single number setting how far one step travels.
5. **Repeat** from step 2 until the stopping test fires.
6. **Stop** when the gradient is small enough that further steps buy nothing. A gradient of exactly zero means a minimum has been reached; in practice the test is $\lVert \nabla_{\boldsymbol\theta} J \rVert$ or the improvement in $J$ falling below a [[Tolerance]], with a cap on the number of [[Epoch|epochs]] as the backstop.

The update in step 4 is the whole algorithm in one line:

$$\boldsymbol\theta^{\text{next step}} = \boldsymbol\theta - \eta \, \nabla_{\boldsymbol\theta} J(\boldsymbol\theta)$$

There is a ceiling on $\eta$ and it is sharp rather than gradual: outside $0 < \eta < 2/\lambda_{\max}$, where $\lambda_{\max}$ is the largest curvature of the cost, each step overshoots further than the last and the parameters diverge geometrically. [[Learning Rate]] carries the threshold and what sets it.

### The worked case

For [[Linear Regression]] the cost is mean squared error,

$$\text{MSE}(\boldsymbol\theta) = \frac{1}{m} \sum_{i=1}^{m} \big(\boldsymbol\theta^{T}\mathbf{x}^{(i)} - y^{(i)}\big)^{2}$$

Differentiating one squared residual by the chain rule brings the exponent down as a factor of $2$ and leaves $x_j^{(i)}$ from the inner derivative, so the partial with respect to $\theta_j$ is

$$\frac{\partial}{\partial \theta_j} \text{MSE}(\boldsymbol\theta) = \frac{2}{m} \sum_{i=1}^{m} \big(\boldsymbol\theta^{T}\mathbf{x}^{(i)} - y^{(i)}\big) \, x_j^{(i)}$$

The residual is shared by every one of those $n+1$ expressions and only the trailing $x_j^{(i)}$ changes, which is what lets the whole stack collapse into one matrix product:

$$\nabla_{\boldsymbol\theta} \text{MSE}(\boldsymbol\theta) =
\begin{bmatrix}
\frac{\partial}{\partial \theta_0} \text{MSE}(\boldsymbol\theta) \\
\frac{\partial}{\partial \theta_1} \text{MSE}(\boldsymbol\theta) \\
\vdots \\
\frac{\partial}{\partial \theta_n} \text{MSE}(\boldsymbol\theta)
\end{bmatrix}
= \frac{2}{m} \mathbf{X}^{T} (\mathbf{X}\boldsymbol\theta - \mathbf{y})$$

$\mathbf{X}\boldsymbol\theta - \mathbf{y}$ is the vector of residuals, and left-multiplying by $\mathbf{X}^{T}$ weights each residual by each feature and sums, which is precisely the sum written out above.

### Why the terrain matters

Not every cost is a well-behaved bowl. Some carry holes, ridges, plateaus and other irregular terrain, and each of those defeats the procedure differently: a plateau makes the gradient so small that progress stops without a minimum having been reached, and a hole makes it stop at a minimum that is not the minimum. [[Convexity]] is the property that rules all of this out at once. A convex cost has no local minimum that is not global, so "a minimum" and "the minimum" become the same thing and where you initialize stops mattering. Both MSE and [[Log Loss]] are convex, which is why the linear models can be fitted this way without anxiety. The caution about irregular terrain applies wherever convexity fails and nowhere else.

Conditioning is the second property of the terrain and it survives convexity. When the features are on wildly different scales the eigenvalues of $\mathbf{X}^{T}\mathbf{X}$ spread out, the level sets of the cost stretch from circles into a long thin ravine, and the steepest direction points across the ravine rather than along it and the path zigzags. Putting all the features on a similar scale with [[Feature Scaling]] is the fix, and it is cheap: one pass over the columns buys back iterations that would otherwise be spent on the geometry rather than on the fit.

The search itself runs over [[Parameter Space]], one axis per parameter being optimized. Linear regression on $n$ features searches $n+1$ dimensions. A model with a million parameters searches a million, and the difficulty of the search grows with them, which is the practical reason the cheap-per-step variants win as models get large.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| learning rate | $\eta$ | no library default; $0.1$ in the worked linear regression case | longer steps, faster early progress, and above $2/\lambda_{\max}$ the iteration diverges instead of converging | grid over powers of ten, plot the cost against the step count, take the largest value whose curve falls smoothly |
| batch size | $b$ | $b = m$ is [[Batch Gradient Descent]], $b = 1$ is [[Stochastic Gradient Descent]] | steadier gradients and a path that converges to a point rather than to a neighbourhood, at a cost per step that is linear in $b$ | $b$ between $32$ and a few hundred when the hardware can vectorize it, $b = m$ when $m$ is small |
| epoch cap | `n_epochs` | no library default; $1000$ in the worked case | more passes over the data, longer fit, and no effect at all once the [[Tolerance]] test has already fired | set it generously and let the tolerance do the stopping, then check the fit stopped for the reason you intended |
| stopping tolerance | $\epsilon$ | no library default | stops on a smaller improvement, so an earlier and possibly [[Underfitting\|underfit]] stop | lower it while the cost is still visibly falling at the stop |
| initial parameters | $\boldsymbol\theta^{(0)}$ | random draw | not ordered. Irrelevant on a convex cost, which has one minimum to find; decisive on a cost with several | pin a [[Random Seed]], and on a non-convex cost restart from several draws and keep the best |

## Failure modes

- **A learning rate above the stability ceiling.** With $\eta > 2/\lambda_{\max}$ each step overshoots the valley and lands further up the opposite wall than it started, so the cost rises monotonically and the parameters overflow to `inf` within a few dozen steps. A cost that grows rather than falls is this and nothing else, and it is diagnosed by plotting the cost per epoch, not by waiting.
- **A learning rate far below it.** At $\eta = 0.02$ on the worked case the path is still visibly short of the line after the twenty steps the figure draws. It does arrive by the thousandth, so the rate is slow rather than fatal; what makes it a failure mode is that any tighter epoch cap stops the fit before it gets there. Nothing looks broken, the cost falls the whole way, and the result reads as [[Underfitting]] and gets misdiagnosed as a modelling problem.
- **Unscaled features.** One $\eta$ multiplies the gradient in every coordinate at once, so a column measured in hundreds of thousands and a column in $[0,1]$ cannot both be given a sensible step length. The ceiling $2/\lambda_{\max}$ is set by the largest column and the progress along the smallest is then glacial, which is the ravine described above. This is a preprocessing bug, and [[Feature Scaling]] is the whole fix.
- **A plateau read as convergence.** The stopping test fires on a small gradient, and a long flat stretch of a non-convex cost produces a small gradient without being anywhere near a minimum. The test cannot tell the two apart.
- **Local minima and saddle points on a non-convex cost.** Where [[Convexity]] fails, the point reached depends on where the run started, so the model is a property of the random initialization as much as of the data. Two seeded runs that disagree is the symptom.
- **Counting epochs and counting steps as if they were the same.** One [[Epoch]] is one pass over the training set, which is a single update for batch gradient descent and $m$ of them for stochastic. A learning schedule or a patience setting written against the wrong one of those is off by a factor of $m$.

## Implementation

scikit-learn 1.6 has no library call for gradient descent as such, because it is the machinery inside the estimators rather than an estimator itself. Written out directly it is five lines, and the worked linear regression case runs with $\eta = 0.1$ for $1000$ epochs on $m = 100$ instances. NumPy:

```python
import numpy as np
from sklearn.preprocessing import add_dummy_feature

X_b = add_dummy_feature(X)     # prepend the x0 = 1 column
eta = 0.1                      # learning rate
n_epochs = 1000
m = len(X_b)

np.random.seed(42)
theta = np.random.randn(2, 1)  # random initialization

for epoch in range(n_epochs):
    gradients = 2 / m * X_b.T @ (X_b @ theta - y)
    theta = theta - eta * gradients
```

Swap the learning rate for $0.5$ and the path bounces across the valley and away; swap it for $0.02$ and the first twenty steps are still well short of the optimum, though the full thousand do get there. Neither change touches any other line.

scikit-learn 1.6 exposes the gradient-based route through `SGDRegressor` and `SGDClassifier`, which take one instance per update rather than the full batch, and through the iterative `solver` choices on `Ridge` and `LogisticRegression`. [[Stochastic Gradient Descent]] carries the estimator call and its arguments. `LinearRegression` does not iterate at all; it solves the least squares problem directly, which is the [[Normal Equation]] route.
