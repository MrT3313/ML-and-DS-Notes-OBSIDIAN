---
note_kind: concept
aliases:
  - epochs
  - training epoch
  - pass over the training set
  - n_epochs
up: "[[Gradient Descent]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## Definition

An epoch is one complete pass over the [[Training Set]]: every instance has been used exactly once. It is the unit training length is counted in, the unit a [[Learning Schedule]] is quoted in, and the unit a stopping rule is checked on.

## Formal statement

One epoch is a fixed amount of gradient computation and a variable number of parameter updates. With $m$ instances and a batch size $b$, the number of updates in one epoch is

$$U = \left\lceil \frac{m}{b} \right\rceil$$

which gives the three standard cases directly:

| variant | $b$ | updates per epoch |
|---------|-----|-------------------|
| [[Batch Gradient Descent]] | $m$ | $1$ |
| [[Mini-Batch Gradient Descent]] | $1 < b < m$ | $\lceil m/b \rceil$ |
| [[Stochastic Gradient Descent]] | $1$ | $m$ |

After $E$ epochs the total update count is $T = E \cdot U$, and that $T$ is the $t$ a [[Learning Schedule]] is indexed by: $t = e \cdot U + i$ for iteration $i$ within epoch $e$.

Every variant touches each instance once per epoch, so the per-epoch cost of computing gradients is the same, $O(mn)$ for $n$ features; what differs by a factor of $m$ is how many times $\boldsymbol\theta$ moves in exchange for it, which is the argument [[Gradient Descent]] sets out between the three. The consequence visible in the epoch count is that a stochastic fit is quoted in tens of epochs while a batch fit is quoted in hundreds or thousands: the notebook for [[HOML Ch04 Training Models|HOML chapter 4]] runs batch gradient descent for 1000 epochs and stochastic gradient descent for 50 on the same data, and the 50 is the larger number of updates.

An epoch shuffles or samples rather than marching in order. Mini-batch permutes the instance indices at the start of each epoch and cuts consecutive slices; stochastic gradient descent as written in that notebook draws a uniform random index $m$ times, which means an epoch there is $m$ draws with replacement rather than a strict permutation, and some instances are seen twice while others are skipped. Both are called an epoch.

### In scikit-learn

scikit-learn 1.6: the estimators count epochs, never individual updates, and the argument is named for iterations even though an iteration here means one epoch.

```python
from sklearn.linear_model import SGDRegressor

sgd_reg = SGDRegressor(max_iter=1000, tol=1e-5, penalty=None, eta0=0.01,
                       n_iter_no_change=100, random_state=42)
sgd_reg.fit(X, y.ravel())

sgd_reg.n_iter_   # epochs actually run before the stopping rule fired
sgd_reg.t_        # updates performed, equal to n_iter_ * n_samples + 1
```

`max_iter` is the ceiling on epochs and defaults to 1000. `n_iter_` reports how many were actually run, which is the number to read when you want to know whether the fit converged or merely ran out of budget. `t_` converts back to updates and confirms $U = m$ for these estimators, since they are pure stochastic gradient descent with $b = 1$.

`partial_fit` runs exactly one epoch per call, with `max_iter` forced to 1 internally, and neither `tol` nor `max_iter` applies. That makes the epoch the natural loop body when you want to inspect the model between passes, which is exactly how [[Early Stopping]] is hand-rolled: call `partial_fit` once, score the validation set, keep the best copy, repeat.

## Where it is used

Epochs are the horizontal axis of the [[Learning Curve]] that diagnoses whether more training would still help. They are one half of the stopping rule, with [[Tolerance]] the other: `max_iter` caps how many epochs may run and `tol` decides whether they were needed, and whichever fires first ends the fit. [[Early Stopping]] asks the different question of which epoch to keep, scoring a held-out set after each pass and reverting to the best. [[Stochastic Gradient Descent]] and [[Mini-Batch Gradient Descent]] both make many updates per epoch through the $\lceil m/b \rceil$ count above, which is also what a [[Learning Schedule]] converts between when it is quoted per epoch but applied per update.
