---
note_kind: concept
aliases:
  - step size
  - eta
  - adaptation rate
  - eta0
up: "[[Hyperparameter]]"
sources:
  - "[[HOML Ch01 The Machine Learning Landscape]]"
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

The learning rate $\eta$ is the [[Hyperparameter]] that scales how far the parameters move on each training step: it is the number the gradient gets multiplied by before it is subtracted. Everything else said about it, that it sets how fast a fit converges and how fast an online system adapts to new data, is a consequence of that one multiplication.

## Formal statement

$$\boldsymbol\theta \leftarrow \boldsymbol\theta - \eta \, \nabla_{\boldsymbol\theta} J(\boldsymbol\theta)$$

with $J$ the [[Cost Function]]. The gradient supplies the direction and the relative magnitudes; $\eta$ supplies the scale, and it is the only thing standing between a correct direction and a step of the wrong size.

### Too large and too small

Too large overshoots. The failure is sharp rather than gradual, and for a quadratic cost it has an exact threshold. Write the cost's Hessian as $\mathbf{H}$; one step multiplies the error component along each eigenvector of $\mathbf{H}$ by $|1 - \eta\lambda|$, so the iteration contracts on every axis, and therefore converges at all, exactly when

$$0 < \eta < \frac{2}{\lambda_{\max}(\mathbf{H})}$$

For [[Linear Regression]] with $J(\boldsymbol\theta) = \frac{1}{m}\lVert \mathbf{X}\boldsymbol\theta - \mathbf{y} \rVert_2^2$ the Hessian is $\mathbf{H} = \frac{2}{m}\mathbf{X}^{\mathsf{T}}\mathbf{X}$, so the bound is $\eta < m / \lambda_{\max}(\mathbf{X}^{\mathsf{T}}\mathbf{X})$. Cross that line and the error grows by a constant factor every step: the cost does not wander upward, it diverges geometrically, which is why overshoot shows up as overflow rather than as a poor fit. The threshold is set by curvature, so the same $\eta$ that is safe on scaled features can diverge on unscaled ones, and this is one of the concrete reasons [[Feature Scaling]] comes before an iterative fit. How much room there is under the threshold is set by how badly conditioned $\mathbf{X}$ is, which is the second.

Too small converges slowly, and the practical consequence is not that the answer is worse but that the fit stops before reaching it: the [[Epoch]] budget runs out, or the [[Tolerance]] test reads the tiny per-step improvement as convergence and halts on a plateau that is not a minimum.

One correction to the usual pairing. A small rate does not cause the optimizer to get trapped in a local minimum. Two things are true instead. On a convex cost there are no local minima to be trapped in, and the squared-error cost of linear regression is convex, so on that surface the question does not arise (see [[Convexity]]). On a cost that is not convex, a small rate does not create the basin the search fell into and does not put it there; what it removes is the only escape a first-order method has, since leaving a basin requires either a step long enough to clear its wall or the sampling noise of [[Stochastic Gradient Descent]]. The accurate statement is that a small rate fails to escape a local minimum rather than causing entrapment in one. There is a separate way a small rate really does strand a fit: a [[Learning Schedule]] that decays too fast drives $\eta$ to zero while $\boldsymbol\theta$ is still far from the optimum, and the parameters freeze in place.

### Step size and adaptation rate

These are one knob, not two, and the mechanism says why rather than merely asserting it.

In [[Stochastic Gradient Descent]] each update uses a single instance's loss, $\boldsymbol\theta \leftarrow \boldsymbol\theta - \eta \nabla_{\boldsymbol\theta} \ell(h_{\boldsymbol\theta}(\mathbf{x}^{(i)}), y^{(i)})$. Take squared error and a single constant feature, so that $\ell = (\theta - y_i)^2$ and $\nabla \ell = 2(\theta - y_i)$. The update becomes

$$\theta \leftarrow (1 - 2\eta)\,\theta + 2\eta \, y_i$$

which is an exponential moving average of the targets with smoothing factor $2\eta$. The influence of an instance seen $k$ steps ago is down by $(1 - 2\eta)^{k}$, so the model's effective memory is about $1/(2\eta)$ instances. "How far the parameters move per step" and "how fast the system forgets old data" are therefore two readings of the same coefficient, because moving further toward the newest instance is exactly what discarding the older ones means.

Two riders keep this from being glib. In [[Batch Gradient Descent]] the adaptation reading has no content, because there is no stream and every step already sees all the data; there $\eta$ is a step size and nothing else. And a decaying [[Learning Schedule]] makes the two readings diverge over time in a way that matters operationally: as $\eta \to 0$ the model stops adapting, so a system meant to track drift needs a rate that does not decay, which in practice means `learning_rate="constant"`.

$\eta$ is set by hand in every variant, batch gradient descent included; it is never read off the data. The notebook for [[HOML Ch04 Training Models|HOML chapter 4]] hardcodes `eta = 0.1` for its batch fit and `eta = 0.5` for its softmax fit. The exceptions worth knowing are that scikit-learn's `"optimal"` schedule computes its starting rate from the regularization strength by a heuristic, and that second-order and line-search methods derive a step from curvature instead of taking one on faith, neither of which is in play for plain gradient descent.

### In scikit-learn

scikit-learn 1.6: the rate is set through `eta0` and the decay through `learning_rate`, on `SGDClassifier` and `SGDRegressor`.

```python
from sklearn.linear_model import SGDRegressor

# fixed rate, no decay: the choice for an online system that must keep adapting
SGDRegressor(learning_rate="constant", eta0=0.01, max_iter=1000, tol=1e-5)

# the regressor default: eta0 / t ** power_t, with power_t=0.25
SGDRegressor()   # learning_rate="invscaling", eta0=0.01
```

`eta0` is the initial rate and is read by `"constant"`, `"invscaling"` and `"adaptive"`, but not by `"optimal"`, which derives its own from `alpha`. `SGDClassifier` defaults to `learning_rate="optimal"` and therefore ships `eta0=0.0`, which is harmless until you switch its schedule and forget to supply a rate, at which point every step is zero and the model never moves. `partial_fit` keeps the update counter `t_` running across calls, so a decaying schedule keeps decaying through a stream rather than restarting per call. The four schedules and their formulas are in [[Learning Schedule]].

## Where it is used

Every gradient-based method carries it in its hyperparameter table: [[Batch Gradient Descent]], [[Stochastic Gradient Descent]] and [[Mini-Batch Gradient Descent]] differ in what they compute the gradient from and agree entirely on what $\eta$ does to it. [[Gradient Descent]] is where the update rule itself lives; this note is only about the coefficient in front of it. Varying $\eta$ over the course of a fit rather than fixing it is a [[Learning Schedule]], which is what [[Stochastic Gradient Descent]] needs before it will settle instead of bouncing. In [[Online Learning]] this same number governs the plasticity-stability trade-off, by the exponential-forgetting mechanism above. Tuning it is [[Model Selection]] work, and it is the first thing to lower when a training curve rises instead of falling, since a rising training error points at divergence rather than at [[Overfitting]].
