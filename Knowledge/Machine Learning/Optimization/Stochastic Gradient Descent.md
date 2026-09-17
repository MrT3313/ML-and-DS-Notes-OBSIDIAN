---
note_kind: method
aliases:
  - SGD
  - stochastic GD
  - stochastic gradient descent
up: "[[Gradient Descent]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Stochastic gradient descent picks one random instance from the training set at every step and computes the gradient on that instance alone. Nothing else about [[Gradient Descent]] changes. The consequence is that the cost of a step stops depending on how much data there is, which is the property the whole method exists for.

Reach for it when [[Batch Gradient Descent]] cannot pay for a full pass: a very large $m$, or a training set that does not fit in memory at all. Because a step needs only the current instance, the data can be streamed from disk a chunk at a time and each chunk discarded after use, which makes it an [[Out-of-Core Learning]] algorithm, and because it can equally consume instances that have not arrived yet, it is the basis for [[Online Learning]]. The scikit-learn guide to scaling names `SGDRegressor` in its short list of regressors that train incrementally through `partial_fit`.

What you give up is the orderly descent. Sampling one instance makes the step direction a random variable, so the path is far less regular than the batch path, and less regular than the [[Mini-Batch Gradient Descent]] path that sits between them: instead of decreasing gently until it reaches the minimum, the cost bounces up and down and decreases only on average. Over time the parameters end up very close to the minimum, but they keep bouncing and never settle, so the final values are good rather than optimal. The fix is a [[Learning Schedule]], and the same randomness that causes the problem is what lets it escape a local minimum that batch gradient descent would have been trapped in.

[[Stochastic Gradient Descent Classifier]] is this optimizer with a classification loss bolted on, and the relationship is worth keeping straight: the classifier is a model fitted by this method, this note is the method.

## Algorithm

1. Initialize $\boldsymbol\theta$ randomly.
2. Draw an index $i$ at random from $1, \dots, m$.
3. Compute the gradient of the cost on instance $i$ alone.
4. Step against it, scaled by the current [[Learning Rate]] $\eta_t$.
5. Shrink $\eta_t$ according to the [[Learning Schedule]].
6. Repeat from step 2. One [[Epoch]] is $m$ such steps, which is one instance drawn per instance in the training set, not one step.

For the mean squared error of [[Linear Regression]] the single-instance gradient carries no averaging factor, because there is nothing to average over:

$$\nabla_{\boldsymbol\theta} \, \ell^{(i)}(\boldsymbol\theta) = 2 \, \mathbf{x}^{(i)} \big( (\mathbf{x}^{(i)})^{T} \boldsymbol\theta - y^{(i)} \big)$$

$$\boldsymbol\theta^{\text{next step}} = \boldsymbol\theta - \eta_t \, \nabla_{\boldsymbol\theta} \, \ell^{(i)}(\boldsymbol\theta)$$

The $\frac{2}{m}$ that appears in the batch gradient becomes a bare $2$ here. The division by $m$ belongs to the average over a full pass and has no counterpart when the pass is one row long, so carrying it over by mistake shrinks every step by a factor of $m$ and the fit simply never moves.

### Why one instance is enough

Drawing $i$ uniformly makes the single-instance gradient an unbiased estimate of the full one:

$$\mathbb{E}_i\big[ \nabla_{\boldsymbol\theta} \, \ell^{(i)}(\boldsymbol\theta) \big] = \frac{1}{m}\sum_{i=1}^{m} 2 \, \mathbf{x}^{(i)}\big((\mathbf{x}^{(i)})^{T}\boldsymbol\theta - y^{(i)}\big) = \nabla_{\boldsymbol\theta} \text{MSE}(\boldsymbol\theta)$$

So each step is the correct step plus mean-zero noise. That is the entire justification: individual steps are wrong, their expectation is right, and enough of them average the error away. It also explains the shape of the path, since the noise does not shrink as the optimum is approached. Near the optimum the true gradient goes to zero while the noise does not, so the noise is eventually all that is left.

### Convergence, and what "converges" has to mean here

A fixed $\eta$ does not converge to a point. The iterates reach a stationary distribution around $\boldsymbol\theta^{*}$ rather than settling on it, at a radius [[Learning Schedule]] gives in terms of $\eta$. This is the precise sense in which the answer is good but not optimal, and the precise sense in which the claim that gradient descent converges on the same parameters as the closed form holds for [[Batch Gradient Descent]] and does not hold here without further conditions.

Shrinking $\eta_t$ toward zero is what supplies those conditions, and [[Learning Schedule]] carries the rule for how fast is too fast. Under a schedule that obeys it the iterates converge to $\boldsymbol\theta^{*}$ itself on a convex cost. The practical reading is the same one simulated annealing gives: big steps early to cover ground and shake free of bad regions, small steps late to settle. Move the rate down too quickly and the run freezes before it has arrived; too slowly and it is still bouncing when you stop it.

The bouncing is not purely a cost. On an irregular cost function it lets the run jump out of a local minimum, so it has a better chance of finding the global minimum than the deterministic batch path does.

## Hyperparameters

`SGDRegressor` in scikit-learn 1.6. The defaults differ from `SGDClassifier` in two places worth naming: `learning_rate` defaults to `"invscaling"` here against `"optimal"` there, and `power_t` to $0.25$ against $0.5$.

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| epoch cap | `max_iter` | `1000` | more passes, longer fit, more chance of converging before the cap | the guide's heuristic is `np.ceil(1e6 / m)`, since SGD tends to converge after roughly $10^{6}$ instances seen |
| stopping tolerance | `tol` | `0.001` | stops on a smaller improvement, so an earlier and possibly [[Underfitting\|underfit]] stop | tighten to `1e-5` when the fit stops while the loss is still falling |
| learning rate schedule | `learning_rate` | `"invscaling"` | not ordered; the four schedules and their formulas are in [[Learning Schedule]]. `"constant"` never settles | leave at `"invscaling"`; note that `"optimal"` reads its step size off $\alpha$, so it keeps using the regularization strength as a step size even when `penalty=None` has removed the penalty |
| initial learning rate | $\eta_0$ | `0.01` | longer steps throughout, faster early progress, divergence if it passes the stability ceiling | log grid, then read the loss curve |
| inverse scaling exponent | $p$ | `0.25` | $\eta_t = \eta_0 / t^{\,p}$ decays faster, so less movement late in the fit and an earlier freeze | raise it if the fit is still visibly bouncing at the end, lower it if it stalls early |
| patience | `n_iter_no_change` | `5` | waits through more flat epochs before stopping, so a longer fit and less risk of stopping on noise | raise it sharply, to `100` or so, when `tol` is tight and the loss curve is jumpy |
| early stopping | `early_stopping` | `False` | `True` holds out a slice and stops on the validation score rather than the training loss, guarding [[Overfitting]] at the cost of training data | turn on for large $m$, where the held-out slice is cheap |
| held-out fraction | `validation_fraction` | `0.1` | larger validation slice, less noisy stopping signal, less data to fit on. Read only when `early_stopping=True` | leave at `0.1` unless $m$ is small |
| iterate averaging | `average` | `False` | `True` reports the running average of the weights instead of the last one, which cancels much of the bouncing described above | try `True` when repeated fits disagree with each other |
| shuffle | `shuffle` | `True` | `False` fixes the instance order across epochs, which lets any ordering in the file become a pattern in the fit | leave `True` |

`SGDRegressor` is an estimator as well as an optimizer, so it also takes `loss`, `penalty`, `alpha`, `l1_ratio`, `epsilon` and `fit_intercept`; those pick the model and its [[Regularization]], not the descent, and are tuned by [[Cross-Validation]] rather than read off a loss curve. `epsilon` is the scale-dependent one, so it is set after [[Feature Scaling]] and [[Target Scaling]]. One of them matters here: `penalty` defaults to `"l2"`, so a plain least-squares fit has to ask for `penalty=None`.

`random_state` is deliberately absent. It changes the fitted weights, through the shuffle and through the initialization, but it is not a quantity you tune. Pin it with an integer and see [[Random Seed]].

## Failure modes

- **A constant learning rate, then treating the result as converged.** With `learning_rate="constant"` the parameters settle into a cloud around the optimum rather than onto a point, so two fits that differ only in where they were stopped disagree, and neither equals what the [[Normal Equation]] returns on the same data. Anything that requires the actual optimum needs a decaying schedule or `average=True`.
- **Unscaled features.** One $\eta_t$ multiplies every coordinate of a gradient computed from a single row, so a feature measured in the thousands produces a step that overshoots in that direction while a feature in the unit interval barely moves. Noisy single-instance gradients make this worse than it is for the batch variant, not better, and the divergence can happen inside the first few hundred updates. Put a scaler in front, inside a [[Pipeline]].
- **Stopping early on a flat stretch.** The default `tol=0.001` with `n_iter_no_change=5` ends the fit as soon as five epochs in a row fail to improve the loss by that much, and a noisy loss curve produces such runs routinely while still descending. The worked linear regression case needs `tol=1e-5` and `n_iter_no_change=100` before the fit runs long enough to match the closed form; check `n_iter_` against `max_iter` before believing anything converged.
- **Data that arrives sorted.** Sampling is what makes the gradient unbiased. A file ordered by the target, by time, or by any other structure, fed in order with `shuffle=False` or streamed through `partial_fit` in file order, breaks the assumption behind the unbiasedness above, and the model tracks the ordering instead of the data.
- **Run-to-run drift with no seed.** Shuffling is on by default, so the instance order and the weights it produces change on every fit. A difference between two candidates smaller than that drift is not a difference.
- **Forgetting that `fit` resets the schedule and `partial_fit` does not.** The decaying rate is a function of the update counter `t_`. Calling `fit` again restarts it, so the model relearns from large steps; calling `partial_fit` continues it, so each successive chunk moves the model less than the last. Neither behaviour is wrong, and choosing the wrong one silently gives a model that either forgets everything or stops adapting.

## Implementation

scikit-learn 1.6 for the library route, NumPy for the hand-written one. Taking the hand-written one first, since the learning schedule that makes it converge is invisible from outside the estimator. The worked case runs $50$ epochs over $m = 100$ instances, which is $5000$ updates, on the schedule $\eta_t = t_0/(t + t_1)$ with $t_0 = 5$ and $t_1 = 50$. NumPy:

```python
import numpy as np

n_epochs = 50
t0, t1 = 5, 50           # learning schedule hyperparameters

def learning_schedule(t):
    return t0 / (t + t1)

np.random.seed(42)
theta = np.random.randn(2, 1)          # random initialization

for epoch in range(n_epochs):
    for iteration in range(m):
        random_index = np.random.randint(m)
        xi = X_b[random_index : random_index + 1]
        yi = y[random_index : random_index + 1]
        gradients = 2 * xi.T @ (xi @ theta - yi)   # no 1/m: one instance
        eta = learning_schedule(epoch * m + iteration)
        theta = theta - eta * gradients
```

The schedule is indexed by `epoch * m + iteration`, the running count of updates, not by the epoch. Indexing it by the epoch instead would hold $\eta$ fixed for $m$ steps at a time and decay it $m$ times too slowly.

scikit-learn 1.6 packages the same thing as `SGDRegressor`. The arguments below override defaults in three places: `tol` is tightened from `0.001` and `n_iter_no_change` raised from `5` so the fit is not stopped by a flat stretch, and `penalty=None` removes the `"l2"` penalty that is on by default, which is what makes this plain linear regression rather than [[Ridge Regression]]. `eta0=0.01` restates the default explicitly.

```python
from sklearn.linear_model import SGDRegressor

sgd_reg = SGDRegressor(max_iter=1000, tol=1e-5, penalty=None, eta0=0.01,
                       n_iter_no_change=100, random_state=42)
sgd_reg.fit(X, y.ravel())  # y.ravel() because fit() expects 1D targets
```

`fit` expects a one-dimensional target, so a column vector of shape $(m, 1)$ has to be flattened. `sgd_reg.intercept_` and `sgd_reg.coef_` hold $\theta_0$ and the rest, and they land within a few thousandths of the closed-form solution rather than on it, $(4.2128, 2.7727)$ against the exact $(4.2151, 2.7701)$, which is the bouncing showing up in the output.

scikit-learn 1.6, the out-of-core form. `partial_fit` performs one epoch over whatever it is handed and keeps the update counter, so the learning rate keeps decaying across calls instead of restarting:

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
sgd_reg = SGDRegressor(penalty=None, eta0=0.01, random_state=42)

for X_chunk, y_chunk in stream_chunks():       # each chunk fits in memory
    sgd_reg.partial_fit(scaler.partial_fit(X_chunk).transform(X_chunk),
                        y_chunk.ravel())
```

`StandardScaler` exposes `partial_fit` for the same reason the estimator does, so the running mean and variance can be updated chunk by chunk without ever holding the whole file. Note that the scaler is therefore still changing while the model is being fitted against it, which is a real cost of streaming and an argument for computing the scaling constants on a first pass when you can afford two.
