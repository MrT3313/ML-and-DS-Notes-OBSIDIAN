---
note_kind: concept
aliases:
  - learning schedules
  - learning rate schedule
  - learning rate decay
  - annealing schedule
up: "[[Learning Rate]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

A learning schedule is the rule that sets the [[Learning Rate]] as a function of how far training has already progressed, instead of holding one number fixed for the whole fit. It exists to start with steps large enough to cover ground and shrink them until the parameters settle rather than keep bouncing.

## Formal statement

The schedule makes $\eta$ a function of the update counter $t$, which counts parameter updates and not passes over the data:

$$\boldsymbol\theta_{t+1} = \boldsymbol\theta_t - \eta_t \, \nabla_{\boldsymbol\theta} J_t(\boldsymbol\theta_t), \qquad \eta_t = f(t)$$

$J_t$ is the [[Cost Function]] evaluated on whatever sample that update saw: one instance, one mini-batch, or the whole [[Training Set]]. The schedule used throughout [[HOML Ch04 Training Models|HOML chapter 4]] is

$$\eta_t = \frac{t_0}{t + t_1}$$

with $t_0$ and $t_1$ fixed in advance, so the schedule itself carries two hyperparameters. That notebook runs it at $t_0 = 5, t_1 = 50$ for [[Stochastic Gradient Descent]] and at $t_0 = 200, t_1 = 1000$ for [[Mini-Batch Gradient Descent]], and indexes it by $t = (\text{epoch}) \cdot U + (\text{iteration})$ where $U$ is the number of updates in one [[Epoch]]. That indexing is the reason a schedule is described per epoch but applied per update.

### Why decay is needed at all

A stochastic update uses the gradient of a sample rather than of the full cost, so it carries noise that does not vanish as $\boldsymbol\theta$ approaches the minimum. With $\eta$ held constant the parameters reach a neighbourhood of the optimum and then keep jittering inside it at a radius set by $\eta$: good, never optimal. Shrinking $\eta$ shrinks that radius.

Shrinking it too fast is the opposite failure: the steps go to zero while $\boldsymbol\theta$ is still far away and the fit freezes short of the minimum. The classical stochastic approximation conditions (Robbins and Monro, 1951) pin both ends at once:

$$\sum_{t=1}^{\infty} \eta_t = \infty \qquad \text{and} \qquad \sum_{t=1}^{\infty} \eta_t^{2} < \infty$$

The first says the total distance the steps could still cover is unbounded, so no finite starting error can strand the search. The second says the accumulated noise is finite, so the jitter dies out. The $t_0/(t + t_1)$ form above satisfies both, its terms falling like $1/t$ and its squares like $1/t^{2}$.

### The annealing analogy

Calling this "similar to simulated annealing" is a claim about one shared idea: both start hot, exploring widely, and cool on a preset schedule until the search stops moving. Simulated annealing also accepts uphill moves by an explicit rule, which a learning schedule never does. It is not simulated annealing, it just cools like one.

### In scikit-learn

scikit-learn 1.6: the schedule is the `learning_rate` string argument on `SGDClassifier` and `SGDRegressor`, and $t$ is the update counter, equal to $1$ on the first update and exposed afterwards as `t_`. The $t_0$ in the `"optimal"` row sits in that schedule's denominator and is a different constant from the $t_0$ in the numerator of the schedule above, spelled the same way by accident.

| `learning_rate` | $\eta_t$ | notes |
|-----------------|----------|-------|
| `"constant"` | $\eta_t = \texttt{eta0}$ | no decay, so the jitter never shrinks |
| `"optimal"` | $\eta_t = \dfrac{1}{\alpha\,(t_0 + t - 1)}$ | $t_0$ is an offset fixed by a heuristic rather than by you, as below |
| `"invscaling"` | $\eta_t = \dfrac{\texttt{eta0}}{t^{\texttt{power\_t}}}$ | $p = 0.25$ by default on the regressor, which is slow decay |
| `"adaptive"` | $\eta_t = \texttt{eta0}$, held and divided by 5 | each time `n_iter_no_change` consecutive epochs fail to improve by the [[Tolerance]] `tol` the rate drops fivefold; the fit ends once $\eta$ has reached or fallen below $10^{-6}$ |

Two indexing conventions for `"optimal"` are in circulation and differ by one in the denominator: the shipped code computes $1/(\alpha(t_0 + t - 1))$ off a counter that is $1$ on the first update, and the user guide writes $1/(\alpha(t_0 + t))$ off one that starts at $0$. Same schedule, both putting the first update at $1/(\alpha t_0)$; check where $t$ starts before concluding either is wrong.

$t_0$ is not free either: a heuristic due to Léon Bottou reads `alpha` and the loss and picks the offset that places the first update's rate where it wants it. Two consequences. `eta0` is never read under this schedule, which is why `SGDClassifier` can ship `eta0=0.0`. And $\alpha = 0$ leaves the rate undefined, so the schedule needs a nonzero regularization strength even where the penalty itself is switched off.

The two estimators ship different defaults, which is a real difference and not a documentation quirk:

```python
from sklearn.linear_model import SGDClassifier, SGDRegressor

SGDClassifier()   # learning_rate="optimal",    eta0=0.0,  power_t=0.5,  alpha=1e-4
SGDRegressor()    # learning_rate="invscaling", eta0=0.01, power_t=0.25
```

Switching a classifier to `"constant"`, `"invscaling"` or `"adaptive"` without also setting `eta0` is therefore rejected outright: the fit raises `ValueError: eta0 must be > 0` rather than running with a rate of zero. The same guard means `alpha=0` under `"optimal"` raises too.

An online system that has to keep tracking drift is the case where decay is actively wrong: every decaying schedule makes the model less responsive the longer it has been running, and `t_` keeps climbing across `partial_fit` calls, so `learning_rate="constant"` is the setting that preserves adaptivity.

## Where it is used

[[Stochastic Gradient Descent]] is the method that needs one to converge at all, since with a fixed rate it reaches the neighbourhood of the optimum and then refuses to settle; [[Mini-Batch Gradient Descent]] inherits the same problem in milder form, its jitter shrinking with batch size but never reaching zero. [[Batch Gradient Descent]] is the exception that needs no schedule, because its gradient is the true gradient and goes to zero on its own as the minimum is approached. The schedule is quoted per [[Epoch]] and applied per update, which is what the $t = \text{epoch} \cdot U + \text{iteration}$ indexing above converts between. Choosing $t_0$ and $t_1$, or `eta0` and `power_t`, is [[Model Selection]] work like any other hyperparameter, with the twist that a schedule that decays too fast looks exactly like [[Underfitting]] on a [[Learning Curve]].
