---
note_kind: method
aliases:
  - early stop
  - early stopping
  - early_stopping
  - stop early
up: "[[Regularization]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Early stopping halts an iterative fit at the [[Epoch]] where the validation error bottoms out, and keeps the parameters from that epoch rather than the ones the loop finished with. It is [[Regularization]] with no penalty term: the [[Cost Function]] is untouched, the [[Parameter Space]] is untouched, and the only thing constrained is how far the optimizer is allowed to travel.

Reach for it whenever the fit is iterative and the validation curve is U-shaped, which is the normal shape for [[Stochastic Gradient Descent]] on a model with more capacity than the data supports. It has one property no penalty shares: the strength is discovered inside a single training run instead of being searched over many. Tuning $\alpha$ on a ridge fit means refitting once per candidate value, whereas one early-stopped run tries every value of "how much training" on the way past and reports which was best. That is why it is worth doing even alongside a penalty, and why it is usually the cheapest regularizer available.

It does not apply to a solver that jumps straight to the optimum. There is no trajectory to stop when the answer comes from a closed form, so a [[Normal Equation]] fit gets nothing out of this.

## Algorithm

1. Split the training data into a fit set and a validation set, the usual [[Holdout Validation]] carve. The test set is not involved and must stay untouched.
2. Initialize $\boldsymbol\theta^{(0)}$, set the best score to $\infty$, keep no best parameters yet, set the patience counter to $0$.
3. For each epoch $\tau = 1, 2, \dots, T$: run one pass of the optimizer over the fit set, then evaluate the chosen loss on the validation set, $\mathcal{L}_\tau = \mathcal{L}(h_{\boldsymbol\theta^{(\tau)}}, D_{\text{val}})$.
4. If $\mathcal{L}_\tau$ improves on the best score by more than $\text{tol}$, store a *copy* of the current parameters and reset the patience counter to $0$. Otherwise increment it.
5. Stop when the patience counter reaches $n_{\text{no change}}$, or when $\tau$ reaches the cap $T$, whichever comes first.
6. Return the stored parameters, not the current ones.

Step 6 is the step people skip, and skipping it throws away the entire method. The loop's final parameters are whatever epoch the stopping rule happened to fire on, which is by construction several epochs *past* the minimum.

What the procedure selects is one number:

$$\tau^{*} = \arg\min_{\tau \in \{1, \dots, T\}} \mathcal{L}\big(h_{\boldsymbol\theta^{(\tau)}}, D_{\text{val}}\big)$$

so the number of training steps is being treated as a [[Hyperparameter]] and chosen the same way any other one would be, by validation score. The U shape of $\mathcal{L}_\tau$ against $\tau$ is the same U that every capacity hyperparameter traces out, and the [[Learning Curve]] with training and validation error plotted against epoch is where you read it off: the two curves fall together, then the validation curve turns up while the training curve keeps falling, and the turn is [[Overfitting]] beginning.

### Why stopping early regularizes

A fit stopped at step $\tau$ with step size $\varepsilon$ has travelled at most $\tau\varepsilon$ times the typical gradient away from wherever it started, so the weights it ends on are small for the same reason a penalized model's are: it never had the budget to grow them. That is the sense in which the training-step count is a capacity knob, and it has one consequence worth acting on, that a smaller [[Learning Rate]] is itself mildly regularizing since it shortens the trajectory for the same epoch count. On a linear model with a quadratic error surface the resemblance can be made precise. Started from $\boldsymbol\theta = \mathbf{0}$, after $\tau$ steps gradient descent has recovered a fraction $1 - (1 - \eta\lambda)^{\tau}$ of the least-squares solution in each eigendirection of curvature $\lambda$, where [[Ridge Regression]] recovers $\lambda/(\lambda + \alpha)$ in that direction. Both are shrinkage profiles of the same shape: increasing in $\lambda$, running from nothing to everything, and leaning hardest on the low-curvature directions where the data says least. Identifying $\alpha$ with $1/(\eta\tau)$ lines up their scales, so a long run behaves like a weak $\ell_2$ penalty and a short one like a strong one. The two are not the same function and the fitted weights do not coincide; what carries across is the mechanism and the direction of the effect, and it goes no further than the quadratic surface. None of it changes the procedure above.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| validation fraction | $v$, `validation_fraction` | `0.1` | less noisy stopping signal, fewer rows left to fit on, so a weaker final model | leave at 0.1 for large $m$; below a few thousand rows the held-out slice gets too noisy to stop on and a manual loop with [[Cross-Validation]] is better |
| patience | `n_iter_no_change` | `5` | waits through longer flat stretches, so a later stop, more epochs spent, less risk of stopping on a dip | raise it when the validation curve is jumpy, which it is under [[Stochastic Gradient Descent]] with a large step |
| improvement tolerance | `tol` | `1e-3` | a larger required improvement, so the counter advances sooner and the fit stops earlier, possibly underfit | lower it when `n_iter_` comes back far below `max_iter` while the curve was still falling |
| epoch cap | $T$, `max_iter` | `1000` | more room to run before the cap overrides the stopping rule | set it high enough that the stopping rule fires first, then check `n_iter_` to confirm it did |
| monitored loss | $\mathcal{L}$ | estimator's own `score` | not ordered | match it to the [[Performance Measure]] you will report; a manual loop is the only way to do this with the SGD estimators |

The [[Learning Rate]] is not in the table because it belongs to the optimizer, but a shorter trajectory and a smaller step are the same lever, so changing one changes what the other is worth.

## Failure modes

- **Returning the last model instead of the best.** A loop that stops and uses whatever is in `sgd_reg` has kept the parameters from $n_{\text{no change}}$ epochs past the minimum, which is strictly worse than the model it was supposed to select. The fix is `deepcopy` at each improvement, and the copy is why the method costs memory as well as time.
- **Stopping on a noisy dip.** Stochastic gradient descent makes the per-epoch validation score jump around, so a single accidental improvement resets the patience counter and a single accidental flat run trips it. `n_iter_no_change=5` with `tol=1e-3` is an aggressive rule on a slowly improving problem; check `n_iter_` against `max_iter` before believing the fit converged, and raise the patience rather than lower the tolerance when the curve is merely noisy.
- **Spending the validation set twice.** The epoch was chosen by minimizing error on $D_{\text{val}}$, so the score at that epoch is a minimum over many candidates and is biased low as an estimate of [[Generalization]]. It is a selection score, not a test score. An honest number needs a [[Testing Set]] that played no part in the stopping rule.
- **Monitoring a different metric than the one you report.** With `early_stopping=True` the scikit-learn SGD estimators monitor whatever their own `score()` returns, which is $R^2$ for `SGDRegressor` and accuracy for [[Stochastic Gradient Descent Classifier|SGDClassifier]]. If the decision you actually care about is [[Root Mean Squared Error]] or [[F1 Score]], the stopping rule is optimizing something else, and under [[Class Imbalance]] the accuracy it watches can be near-flat while the metric you report is still moving.
- **Losing the held-out rows permanently.** `early_stopping=True` carves out `validation_fraction` of the training data and never refits on the full set afterwards, so the deployed model saw ten percent less data than it could have. On a small training set that cost can exceed the benefit.
- **Confusing it with a convergence check.** `tol` with `early_stopping=False` watches the *training* objective stop improving, which is [[Tolerance]] doing convergence detection, not regularization. Training loss plateauing and validation loss turning upward are different events, and only the second one is what this method is about.

## Implementation

Two routes exist in scikit-learn 1.6 and they are not the same thing.

**The built-in flag.** scikit-learn 1.6: `SGDRegressor` and `SGDClassifier` both take `early_stopping`, default `False`. Setting it `True` switches the stopping criterion from the training objective to a validation score.

```python
from sklearn.linear_model import SGDRegressor

sgd_reg = SGDRegressor(early_stopping=True, validation_fraction=0.1,
                       n_iter_no_change=5, tol=1e-3, max_iter=1000,
                       random_state=42)
sgd_reg.fit(X_train_prep, y_train)
sgd_reg.n_iter_          # epochs actually run; compare against max_iter
```

The estimator sets aside `validation_fraction` of the data, fits on the rest, and stops when the score returned by its own `score` method fails to improve by at least `tol` for `n_iter_no_change` consecutive epochs. With `early_stopping=False` the same two arguments still apply, but to the training objective instead.

The held-out split is **not** stratified for a regressor, and should not be, since a continuous target has no classes to stratify on. Internally the splitter is chosen by `is_classifier`: `StratifiedShuffleSplit` for a classifier, plain `ShuffleSplit` for a regressor. Both draw on `random_state`, so pin it (see [[Random Seed]]) or the validation slice, and therefore the stopping epoch, changes between runs.

**The manual loop.** scikit-learn 1.6: nothing built in lets you monitor a metric of your own choosing or keep the best model, so a loop does both. This is the route the [[HOML Ch04 Training Models|HOML chapter 4]] notebook takes, on a degree-90 [[Polynomial Regression]] of a quadratic dataset with $m = 100$ split evenly into fit and validation, deliberately unpenalized so the only regularization present is the stopping itself.

```python
from copy import deepcopy
from sklearn.linear_model import SGDRegressor
from sklearn.metrics import root_mean_squared_error

sgd_reg = SGDRegressor(penalty=None, eta0=0.002, random_state=42)
best_valid_rmse = float("inf")
best_model = None

for epoch in range(500):
    sgd_reg.partial_fit(X_train_prep, y_train)
    valid_rmse = root_mean_squared_error(y_valid, sgd_reg.predict(X_valid_prep))
    if valid_rmse < best_valid_rmse:
        best_valid_rmse = valid_rmse
        best_model = deepcopy(sgd_reg)
```

`partial_fit` runs exactly one epoch per call and carries the fit forward, which is what makes the loop incremental rather than 500 fits from scratch. Note that `early_stopping=True` is rejected outright in combination with `partial_fit`, which is consistent: the flag and the loop are two implementations of one idea and you pick one.

`warm_start=True` with repeated `fit` calls is the other way to run the loop, and it differs in one respect that matters. Both reuse the previous coefficients as the starting point, but `fit` resets the internal count of samples seen while `partial_fit` keeps incrementing it, so under a decaying learning-rate schedule a `warm_start` loop restarts the schedule at full step size on every call and a `partial_fit` loop does not. With the default `learning_rate="invscaling"` on `SGDRegressor` that is a real difference in trajectory, not a detail.

Use the flag when you want a fit that stops itself, the default metric is acceptable, and the training set is large enough to spare the slice. Use the loop when the metric matters, when you want the best model rather than the stopping model, or when you want the validation curve itself for a [[Learning Curve]] plot.
