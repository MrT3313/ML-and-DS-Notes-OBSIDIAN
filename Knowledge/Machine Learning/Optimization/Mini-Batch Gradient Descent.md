---
note_kind: method
aliases:
  - mini-batch GD
  - minibatch gradient descent
  - mini-batch
  - mini-batches
  - MBGD
up: "[[Gradient Descent]]"
sources:
  - "[[HOML Ch04 Training Models]]"
confidence: draft
---

## What it does and when

Mini-batch gradient descent computes each gradient on a small random set of instances, called a mini-batch, rather than on the whole training set or on a single instance. It is [[Gradient Descent]] with the one free choice in the family, how many instances one gradient is computed on, set somewhere between the two extremes: $b = m$ is [[Batch Gradient Descent]] and $b = 1$ is [[Stochastic Gradient Descent]], and everything in between is this.

Its main advantage over the stochastic variant is not statistical but architectural. A gradient over $b$ instances is one matrix product rather than $b$ separate vector operations, so it lands on the hardware optimizations for matrix arithmetic, which matters most on GPUs, where a batch of $32$ or $256$ rows costs very little more wall-clock time than a batch of one. The step is also less noisy, so the path is more regular and ends up walking closer to the minimum than the stochastic path does. The trade is in the other direction on an irregular cost: less noise means less help escaping a local minimum.

Reach for it as the default whenever the hardware can vectorize and the training set is too large for full batches, which is to say almost always at scale. The two pure forms are best understood as its limiting cases rather than as separate methods.

## Algorithm

1. Initialize $\boldsymbol\theta$ randomly.
2. Shuffle the training set at the start of each [[Epoch]], so no instance keeps company with the same neighbours from one pass to the next.
3. Cut the shuffled data into consecutive blocks of $b$ instances, giving $\lceil m/b \rceil$ mini-batches per epoch.
4. For each mini-batch $B$, compute the gradient averaged over its $b$ instances.
5. Step against it, scaled by the current [[Learning Rate]] $\eta_t$, and shrink $\eta_t$ according to the [[Learning Schedule]].
6. Repeat from step 2 until the epoch cap or the stopping test.

For the mean squared error of [[Linear Regression]], with $\mathbf{X}_B$ the $b \times (n+1)$ block of rows in the current mini-batch:

$$\nabla_{\boldsymbol\theta} \, J_B(\boldsymbol\theta) = \frac{2}{b} \mathbf{X}_B^{T} (\mathbf{X}_B \boldsymbol\theta - \mathbf{y}_B)$$

$$\boldsymbol\theta^{\text{next step}} = \boldsymbol\theta - \eta_t \, \nabla_{\boldsymbol\theta} \, J_B(\boldsymbol\theta)$$

The averaging factor is $\frac{2}{b}$, matching the size of the block actually summed over. Setting $b = m$ recovers the batch gradient exactly and setting $b = 1$ recovers the stochastic one, which is what makes the three variants one algorithm with one knob rather than three algorithms.

### What $b$ buys

Averaging $b$ unbiased gradients leaves the estimate unbiased and, when the mini-batch is drawn with replacement, divides its variance by $b$:

$$\operatorname{Var}\big[\nabla J_B\big] = \frac{1}{b}\operatorname{Var}\big[\nabla \ell^{(i)}\big]$$

Cutting a permutation into consecutive blocks, which is what step 3 of the algorithm above does, samples without replacement instead, and the factor is then $\frac{1}{b}\cdot\frac{m-b}{m-1}$: the same $1/b$ behaviour with a correction that bites only when $b$ is a large fraction of $m$.

That single fact accounts for everything the variant is chosen for and everything it gives up. The noise floor that keeps constant-step stochastic gradient descent wandering in a cloud around the optimum has a radius set by the gradient variance, so dividing the variance by $b$ shrinks that cloud and the path walks closer to the minimum. The same shrinkage removes the kicks that would have carried the run out of a local minimum on a non-convex cost, which is the harder escape. And because the reduction is in $1/b$ while the cost per step is linear in $b$, the returns fall off quickly: noise read as a standard deviation rather than as a variance falls only as $1/\sqrt{b}$, so going from $b = 1$ to $b = 16$ cuts it fourfold, and going from $16$ to $256$ cuts it fourfold again for sixteen times the arithmetic. This is why useful batch sizes are small numbers, and why the choice is usually made by what the hardware processes in one go rather than by the statistics.

The caveat about local minima only bites where there are local minima to be trapped in. On MSE and on [[Log Loss]] the cost is convex, so every minimum is the minimum, and on the linear models the only thing $b$ changes is how tightly the path converges. The warning bites only on non-convex costs, which is also the setting where mini-batching is standard practice, neural network training above all.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| mini-batch size | $b$ | no library default; $20$ in the worked case, and `batch_size="auto"` on scikit-learn's MLP estimators resolves to $\min(200, m)$ | gradient variance falls as $1/b$, so a steadier path that settles closer to the optimum, at a cost per step linear in $b$ and fewer updates per epoch | powers of two that fit the hardware, $32$ to $512$; raise it only until the wall-clock time per epoch starts rising |
| initial learning rate | $\eta_0$ | no library default; $t_0/t_1 = 0.2$ at the first step in the worked case | longer steps, faster early progress, divergence past the stability ceiling | raise it alongside $b$, since a steadier gradient tolerates a longer step |
| learning schedule | $t_0, t_1$ | no library default; $t_0 = 200$, $t_1 = 1000$ in the worked case | a larger $t_1$ holds the rate near $\eta_0$ for longer before the decay bites | index the schedule by the update count, then check the rate at the last step is small but not zero |
| epoch cap | `n_epochs` | no library default; $50$ in the worked case | more passes, proportionally longer fit | set generously and stop on a [[Tolerance]] |
| shuffling between epochs | `shuffle` | on | fixed order means the same instances share a mini-batch every epoch, so their gradient noise is correlated across passes rather than averaged away | leave on, and pin a [[Random Seed]] |

## Failure modes

- **A mini-batch that is not random.** Cutting an unshuffled file into consecutive blocks makes each mini-batch a slice of whatever order the file was in, so a file sorted by the target hands the model a batch that is entirely one class or one range. The gradient is then biased, not merely noisy, and no amount of averaging fixes a bias. Reshuffling at the start of each epoch is step 2 for this reason.
- **Reading the loss curve per step.** The curve is noisy in proportion to $1/b$, so at small $b$ individual steps move the loss the wrong way constantly. Early stopping applied to that curve fires almost immediately. Average the loss over an epoch, or use a held-out score, before deciding the fit has stopped improving.
- **A learning rate carried over from a different batch size.** The gradient's magnitude is preserved by the $\frac{2}{b}$ averaging but its variance is not, so a rate tuned at $b = 1$ is needlessly cautious at $b = 256$ and a rate tuned at $b = 256$ diverges at $b = 1$. The two knobs have to be retuned together.
- **Expecting a constant rate to converge.** Mini-batches reduce the noise but do not remove it, so a fixed $\eta$ leaves the parameters in the same cloud around the optimum, smaller than the stochastic one by a factor of $\sqrt{b}$ but not smaller than nothing. Only a decaying [[Learning Schedule]] or weight averaging closes that gap.
- **A last mini-batch of the wrong size.** With $\lceil m/b \rceil$ blocks the final one holds $m \bmod b$ instances, which can be a single row. Dividing that step's gradient by $b$ rather than by the number of rows actually present scales it wrongly, and hard-coding the divisor is the usual way this gets written.

## Implementation

scikit-learn 1.6 exposes no mini-batch knob on the linear models. `SGDRegressor` updates once per instance, and `partial_fit` performs one full epoch over the chunk it is given rather than one mini-batch step, so feeding it chunks gives out-of-core stochastic gradient descent and not this. Where the library does expose $b$ is on estimators built around it: `MLPRegressor` and `MLPClassifier` take `batch_size`, defaulting to `"auto"`, which resolves to $\min(200, m)$, and `MiniBatchKMeans` takes a `batch_size` of its own that defaults to $1024$ rather than to `"auto"`. For the linear case, write the loop.

NumPy, on the worked case of $m = 100$ instances with $b = 20$, which is five mini-batches per epoch over $50$ epochs:

```python
import numpy as np
from math import ceil

n_epochs = 50
minibatch_size = 20
n_batches_per_epoch = ceil(m / minibatch_size)

np.random.seed(42)
theta = np.random.randn(2, 1)      # random initialization

t0, t1 = 200, 1000                 # learning schedule hyperparameters

def learning_schedule(t):
    return t0 / (t + t1)

for epoch in range(n_epochs):
    shuffled_indices = np.random.permutation(m)
    X_b_shuffled = X_b[shuffled_indices]
    y_shuffled = y[shuffled_indices]
    for iteration in range(n_batches_per_epoch):
        idx = iteration * minibatch_size
        xi = X_b_shuffled[idx : idx + minibatch_size]
        yi = y_shuffled[idx : idx + minibatch_size]
        gradients = 2 / minibatch_size * xi.T @ (xi @ theta - yi)
        eta = learning_schedule(epoch * n_batches_per_epoch + iteration)
        theta = theta - eta * gradients
```

Three details carry the method. The permutation is redrawn inside the epoch loop, which is what keeps the batches random across passes. The schedule is indexed by `epoch * n_batches_per_epoch + iteration`, the running count of updates, so it decays once per step rather than once per epoch. And $t_0$ and $t_1$ are much larger here than the $5$ and $50$ used for single-instance steps, because there are only $\lceil m/b \rceil$ updates per epoch instead of $m$ of them, so a schedule tuned for the stochastic update count would decay $b$ times too fast.

Plotting the three paths through [[Parameter Space]] on one pair of axes shows what the variance argument predicts: the batch path is a smooth curve straight into the optimum, the stochastic path is a jagged scatter around it, and the mini-batch path runs between them, less erratic than the stochastic one and stopping short of the exact point the batch one reaches.
