---
note_kind: concept
aliases:
  - tol
  - convergence tolerance
  - stopping tolerance
  - convergence criterion
up: "[[Gradient Descent]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

The tolerance $\epsilon$ is the size below which a quantity is declared small enough to stop on: either how flat the cost surface has become, or how little the last few passes improved. It is the number that turns "keep iterating" into a decision an algorithm can make without being told when to quit.

## Formal statement

Two different quantities get compared against $\epsilon$, and they answer different questions.

**The gradient-norm test.** An iterative optimizer is at a stationary point when the gradient vanishes, so stop once it is nearly vanished:

$$\left\| \nabla_{\boldsymbol\theta} J(\boldsymbol\theta) \right\|_2 < \epsilon$$

This is the natural test for [[Batch Gradient Descent]], whose gradient is the true gradient of the [[Cost Function]] and does go to zero at the minimum. It does not transfer to [[Stochastic Gradient Descent]], where the gradient being measured is one instance's and stays noisy no matter how close $\boldsymbol\theta$ is.

What a given $\epsilon$ buys depends on the curvature of $J$ and not on $\epsilon$ alone, so the same `tol` stops a flat, badly conditioned problem much further from the optimum than a well-conditioned one, and on a surface without [[Convexity]] a small gradient norm says stationary, not minimal.

**The improvement test.** Track the training loss $L_e$ achieved on epoch $e$ and the best seen so far, $L^{*}_{e} = \min_{k \le e} L_k$. An epoch counts as no improvement when

$$L_e > L^{*}_{e-1} - \epsilon$$

and the fit stops once that has held for $n$ consecutive epochs. This is the form that works for stochastic methods, because averaging the loss over a whole [[Epoch]] smooths away the per-instance noise that defeats the gradient-norm test, and requiring $n$ consecutive failures rather than one absorbs the epochs that get unlucky.

Note what $\epsilon$ is not, in either form. It is a threshold on an improvement or a slope, not a bound on the distance from $\boldsymbol\theta$ to $\boldsymbol\theta^{*}$, and the conversion between them runs through the curvature. Tightening `tol` by a factor of ten does not put the answer ten times closer to the optimum.

### In scikit-learn

scikit-learn 1.6: `SGDClassifier` and `SGDRegressor` use the improvement test, with `tol` as $\epsilon$ and `n_iter_no_change` as $n$. Stated exactly, and this is the rule that answers what actually stops a fit:

- After each [[Epoch]], the mean training loss over that epoch is compared against the best mean training loss of any earlier epoch. The comparison uses the data loss only; the regularization penalty is not included in it.
- If the epoch failed to beat that best by more than `tol`, a counter increments. If it did beat it, the counter resets to zero.
- When the counter reaches `n_iter_no_change`, the fit stops and reports `n_iter_` as the number of epochs run.
- Independently, the fit stops after `max_iter` epochs whatever the counter says, and raises `ConvergenceWarning` when it ends that way.

The pair is intended to be used with `max_iter` set high and the stopping left to the tolerance: `max_iter` is a budget ceiling, `tol` with `n_iter_no_change` is the convergence test, and whichever fires first ends the fit.

```python
from sklearn.linear_model import SGDRegressor

# converges on tol long before the 1000-epoch ceiling
SGDRegressor(max_iter=1000, tol=1e-5, n_iter_no_change=100, eta0=0.01,
             penalty=None, random_state=42)

# tol=None disables the test entirely: exactly max_iter epochs, every time
SGDRegressor(max_iter=1000, tol=None, penalty="l2", eta0=0.01)
```

Defaults are `tol=1e-3` and `n_iter_no_change=5`. Two settings change what is being measured. `learning_rate="adaptive"` reuses the same counter for a different purpose: reaching `n_iter_no_change` divides $\eta$ by five and resets the counter instead of stopping, and the fit only ends once $\eta$ has reached or fallen below $10^{-6}$. `early_stopping=True` swaps the tracked quantity for the `score` of a held-out `validation_fraction` of the data, and since a score is better when higher the comparison flips to `score < best_score + tol`.

## Where it is used

Every [[Gradient Descent]] variant needs one, because none of them terminates on its own: the loop is infinite until something declares the change small enough. It pairs with [[Epoch]] as the two halves of a stopping rule, `max_iter` bounding how long and `tol` deciding whether that long was needed. [[Early Stopping]] looks superficially identical and is a different question: it watches validation error to choose which epoch generalizes best, whereas tolerance watches training loss to decide when the optimizer has stopped making progress, and a fit can hit its tolerance while already far past the epoch early stopping would have kept. What the number actually buys depends on [[Convexity]]: on a convex surface a small gradient norm bounds the remaining cost through the curvature, and without convexity it certifies only a stationary point that may be a saddle or a local minimum. On a badly scaled problem the same `tol` stops a fit much further from the optimum, which is one more argument for [[Feature Scaling]] before any iterative fit.
