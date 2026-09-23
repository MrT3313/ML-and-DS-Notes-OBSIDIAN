---
note_kind: method
aliases:
  - data parallelism
  - data parallel
  - data-parallel training
  - all-reduce
  - DistributedDataParallel
  - parameter server
  - straggler
  - stragglers
  - straggler problem
  - synchronous SGD
  - asynchronous SGD
up: "[[Distributed Training]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Data parallelism splits the batch and replicates the model. Every worker holds a whole copy of the parameters, computes a gradient on its own share of the instances, and the shares are combined into one gradient that every copy then applies, so all copies stay identical from one iteration to the next. It is the default arrangement in every framework that has one, and it is the arrangement to reach for when the model fits on one machine and the run is too slow or the data too large for one machine to finish it.

The condition is the whole of the choice. Data parallelism replicates the per-copy memory figure $M_1$ of [[Distributed Training]] once per worker and never lowers the floor under any single machine, so it buys throughput and no room at all. When the model itself does not fit, [[Model Parallelism]] is the arrangement that applies, and a run large enough to need both uses both.

A worker is a [[Node]] in the sense that note fixes, a participant holding its own state and reached only by messages, which is why one machine with eight accelerators is conventionally run as eight workers rather than as one: the count $W$ below is a count of participants and never of boxes.

What it does not buy is fewer gradient computations. The arithmetic per instance is unchanged; what changes is that $W$ machines do it at once, and that the batch one update is computed from is $W$ times larger than the batch one machine held. That second consequence is not free, and the two sections of mathematics below are the two places it comes due.

## Algorithm

Write the batch as $B$, the worker count as $W$, and each worker's share as $B_w$, with $|B_w| = b$.

1. **Shard the batch.** Cut the shuffled batch into $W$ disjoint shards of equal size, one per worker. Disjoint and equal are both load bearing, and the first subsection below says why.
2. **Hold one whole copy per worker.** Every worker starts the iteration with the same parameter vector $\boldsymbol\theta$. This is established once by broadcasting the initialization and maintained thereafter by step 5, not re-established each iteration.
3. **Compute a local gradient.** Worker $w$ evaluates $\nabla J_{B_w}(\boldsymbol\theta)$ on its own shard, against its own copy. This is the only step that is genuinely parallel, and it is where all the arithmetic is.
4. **Reduce.** Average the $W$ local gradients into one. Every worker must have the result before any of them steps.
5. **Apply the same update on every copy.** Each worker runs the ordinary [[Gradient Descent]] update with the reduced gradient, so all $W$ copies remain bit-identical and the next iteration's step 2 holds for free.

### Why the order is the algorithm

For disjoint shards of equal size, the average of the local gradients is exactly the gradient on their union:

$$\frac{1}{W} \sum_{w=1}^{W} \nabla J_{B_w}(\boldsymbol\theta) \;=\; \nabla J_{B}(\boldsymbol\theta), \qquad |B| = Wb$$

That identity is the licence for the whole arrangement. It says $W$ workers at per-worker batch $b$ are running [[Mini-Batch Gradient Descent]] at batch $Wb$ and nothing else, which is why the arrangement needs no new convergence argument of its own: it inherits the one the batch size already had. Both ends of that family reappear here. At $b = 1$ each worker does exactly the work of one [[Stochastic Gradient Descent]] step while the update the cluster applies is a mini-batch of $W$ instances, so the arrangement is not $W$ stochastic runs but one less noisy run. At the other end, when the shards between them exhaust the training set, the average is the full-data gradient and the arrangement is [[Batch Gradient Descent]] with the sum computed in parallel, which is the case the identity was always describing.

The identity holds only because every term is evaluated at the same $\boldsymbol\theta$. A worker that applies its own gradient before the reduce evaluates its next one at $\boldsymbol\theta_w \ne \boldsymbol\theta$, and an average of gradients taken at different points in [[Parameter Space]] is not the gradient of anything at any single point. That is not a faster version of this method, it is **asynchronous SGD**, whose steps carry gradient staleness (a gradient computed at a parameter vector that has since been updated some number of times) and which therefore has a separate and weaker convergence story. Both are legitimate; they are different algorithms, and the reduce is the line between them.

Unequal shards break the identity in a quieter way. With $|B_w|$ varying, the unweighted average is $\sum_w \frac{1}{W} \nabla J_{B_w}$ rather than $\sum_w \frac{|B_w|}{|B|} \nabla J_{B_w}$, so every instance on a small shard is given more weight than every instance on a large one. Nothing errors; the objective being minimized is simply not the one intended.

### The straggler problem, and why it grows with $W$

Under synchronous updates every worker waits at step 4, so the wall-clock time of an iteration is not an average over workers but a maximum over them. With $T_i$ the time worker $i$ takes on its shard and $T_{\text{reduce}}$ the cost of the reduction,

$$T_{\text{iter}} = \max_{1 \le i \le W} T_i \;+\; T_{\text{reduce}}$$

The wait exists because the reduce is a synchronisation point inside a [[Distributed System]], which is also why a straggler cannot simply be declared finished: that note's system model makes a slow peer and a crashed one indistinguishable from outside, so no participant can tell which case it is waiting on.

$\mathbb{E}\big[\max_i T_i\big]$ is nondecreasing in $W$ for any distribution and strictly increasing for any non-degenerate one, even when the $T_i$ are independent and identically distributed, so no amount of identical hardware removes the effect. For a worked case, take $T_i$ exponential with mean $\mu$ and the expectation is the harmonic number, $\mu H_W \approx \mu(\ln W + \gamma)$: ten workers wait about $2.9\mu$ and a thousand about $7.5\mu$ for work that averages $\mu$. Tails heavier than exponential are worse. This is what makes "the straggler problem grows with the number of machines" a derivable claim rather than an observation.

The tail form is the one to keep, because it is what gets measured. If each worker independently exceeds a threshold $t$ with probability $p$, then

$$\Pr\big[\max_i T_i > t\big] = 1 - (1-p)^{W}$$

Dean and Barroso ("The Tail at Scale", *CACM* 56(2), 2013) work the instance that makes this concrete: a server typically answering in 10 ms with a 99th-percentile latency of one second makes one request in 100 slow on its own, and a request that must collect responses from 100 such servers in parallel is slow 63 percent of the time. The same paper notes that at one slow response in 10,000, a fan-out to 2,000 servers still leaves almost one user request in five above a second. Their subject is an interactive service and a synchronous training iteration is the same fan-out with the same arithmetic, which is why the number transfers.

The standard answer is **backup workers**, from Chen, Pan, Monga, Bengio and Jozefowicz ("Revisiting Distributed Synchronous SGD", 2016). Run $W + W_{\text{backup}}$ workers, take the first $W$ gradients to arrive and drop the rest, so the iteration waits on an order statistic rather than on the maximum. Their point is that this keeps the exactness of the synchronous update while cutting the worst of the wait, and that it beats asynchronous updates, which buy the same speed with gradient noise instead.

### Effective batch size, and what it costs

Because the reduce averages over every shard, the batch that one update is computed from is the sum of the shards:

$$b_{\text{eff}} = W \cdot b$$

and with $A$ accumulation steps before each reduce, $b_{\text{eff}} = W b A$. This is the mechanism behind spreading a model over many machines making the batch very large: the per-worker batch is chosen by what one accelerator holds, so the effective batch is decided by the worker count whether or not anyone decided it.

Whether that is a problem is a measured question, and the paper that measured it is Shallue, Lee, Antognini, Sohl-Dickstein, Frostig and Dahl (*JMLR* 20, 2019), across seven data sets, several model families and three optimizers, tuning the learning rate, momentum and schedule anew at every batch size. The steps-to-a-target-error curve has the same three-region shape everywhere they looked: **perfect scaling**, where each doubling of the batch halves the steps needed; **diminishing returns**, where it does not; and **maximal data parallelism**, where further batch size changes the step count not at all. So the diminishing returns are real and they are a property of the workload rather than of the optimizer's tuning, which earlier disagreements in the literature had confounded. Their protocol is also the answer to the [[Scalability]] question asked properly: the load parameter is the worker count and the performance measure is steps or wall-clock time to a fixed target error, and a claim that adding workers stops helping is checkable only once both are named.

Two of their findings matter for how the knob is set. They found no evidence that a larger batch degrades out-of-sample error once the optimization parameters are retuned, which weakens the sharper "large batches generalize worse" claim considerably. And they found no reliable support for any published heuristic that sets the learning rate from the batch size, so their recommendation is to retune everything at every batch size, and their honest verdict on prediction is that the transition points depend on non-obvious properties of the model, the optimizer and the data set together.

McCandlish, Kaplan and Amodei ("An Empirical Model of Large-Batch Training", 2018) supply the quantity the transition is supposed to be set by. The **gradient noise scale** is the trace of the per-instance gradient covariance over the squared norm of the gradient,

$$B_{\text{simple}} = \frac{\operatorname{tr}(\Sigma)}{|G|^{2}}$$

which is a signal-to-noise ratio: the batch size at which the estimate's variance is on the scale of the estimate. Averaging their local model over a run gives the tradeoff between wall-clock steps and total instances processed as a hyperbola,

$$\left(\frac{S}{S_{\min}} - 1\right)\left(\frac{E}{E_{\min}} - 1\right) = 1, \qquad B_{\text{crit}} = \frac{E_{\min}}{S_{\min}}$$

with $S$ the steps and $E$ the instances needed to reach a fixed level of performance. Training at $B_{\text{crit}}$ sets both factors to 1, so it costs twice the steps of the most time-efficient run and twice the instances of the most data-efficient one: that is the definition of the knee rather than a defect of it. Their claim is that $B_{\text{crit}} \approx B_{\text{noise}}$ and that measuring it predicts the largest useful batch to within an order of magnitude. Whether the largest useful batch is predictable in advance is therefore genuinely unsettled between these two papers, and the honest reading is that the noise scale is a cheap order-of-magnitude estimate and not a substitute for measuring the curve.

Keeping a large effective batch trainable is the contribution of Goyal, Dollár, Girshick, Noordhuis, Wesolowski, Kyrola, Tulloch, Jia and He (2017), and it is two rules. The **linear scaling rule** multiplies the [[Learning Rate]] by the same factor the batch was multiplied by, $\eta \to k\eta$ when $b_{\text{eff}} \to k \, b_{\text{eff}}$, which follows from wanting $k$ small steps and one large step to move the parameters by about the same amount. The rule fails early in training, where the parameters move fast enough that the approximation behind it does not hold, so it is paired with a **gradual warmup** that starts at the small-batch rate and ramps to $k\eta$ over the first few epochs. With both, they trained ResNet-50 on ImageNet at a batch of 8,192 across 256 GPUs in one hour with no loss of accuracy against the small-batch baseline, at about 90 percent scaling efficiency from 8 to 256 GPUs.

### Where the asymmetry between workers comes from

Workers running the same model setup are not expected to consume the same resources, and the reason is the arrangement of the reduce rather than anything about the model.

**Under a parameter server**, from Li, Andersen, Park, Smola, Ahmed, Josifovski, Long, Shekita and Su ("Scaling Distributed Machine Learning with the Parameter Server", OSDI 2014), the parameters live on server nodes and the workers push gradients to them and pull parameters back. Every worker exchanges one gradient's worth of bytes $S$ per iteration; a server handling $W$ workers exchanges $O(W \cdot S)$. Its network, its memory for the master copy and its CPU for applying the update all scale with the worker count while a worker's do not, which is the whole of the observed asymmetry and why that paper's design is about sharding the server role rather than about the learning.

**Under all-reduce** the roles are symmetric and the volume is not. A ring reduce-scatter followed by an all-gather moves $2\frac{W-1}{W}S$ bytes into and out of every worker, which approaches $2S$ and stops growing, so no participant is a bottleneck by construction (the bandwidth-optimal construction is Patarasuk and Yuan, *JPDC* 2009). That per-iteration exchange is affordable only on a network engineered for it, which is what [[High-Performance Computing]] supplies and an ordinary service network does not: a dedicated interconnect with remote direct memory access, on the trusted-tenant terms that note sets out. Asymmetry then survives only in the duties nobody sharded: one rank is usually the one that writes checkpoints, runs the evaluation pass, logs and holds the data-sharding bookkeeping, and those are real memory and real disk on one machine. See [[Experiment Tracking]] for the logging half of that.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| worker count | $W$ | $1$ | effective batch grows as $Wb$, so past the critical batch size the compute buys no reduction in steps; expected iteration time grows through $\mathbb{E}[\max_i T_i]$ | raise it while wall-clock time to a fixed target error keeps falling, and stop when it flattens; that flattening is the diminishing-returns region, not a configuration error |
| per-worker batch size | $b$ | framework-dependent; the per-device figure the accelerator holds | more arithmetic per worker per step, less gradient noise, fewer steps per [[Epoch]]; interacts with $W$ only through the product | set it to what one device holds without spilling, then treat $W$ as the knob |
| gradient accumulation steps | $A$ | $1$ | effective batch grows as $WbA$ with no extra machines, at proportionally longer wall clock per update | use it to reach a target effective batch when $W$ is fixed by the hardware you have |
| update discipline | synchronous or asynchronous | synchronous | asynchronous removes the wait at the reduce and replaces exactness with gradient staleness, so the answer changes rather than arriving sooner | stay synchronous unless the straggler cost has been measured; prefer backup workers to asynchrony |
| backup workers | $W_{\text{backup}}$ | $0$ | iteration waits on the $W$th of $W + W_{\text{backup}}$ arrivals rather than on the last, so the tail shortens; the batch becomes a random subset of the shards, so which instances contribute varies per step | raise from zero while iteration time falls; a few percent of $W$ is the reported range |
| warmup length | epochs of ramp | none | more epochs spent climbing from the small-batch rate to the scaled one, so less chance of the early divergence the linear scaling rule invites, at the cost of those epochs running slower than they could | lengthen it while the loss still spikes in the first few epochs; five epochs is the ResNet-50 figure, not a constant |

Three settings pass the test but are derived or pinned rather than searched. **The linear scaling factor $k$ on the [[Learning Rate]]** is not tuned at all once the effective batch is known: it is the factor the effective batch grew by, and what is tuned around it is the warmup above and a verification against the small-batch baseline. **The reduction mode**, mean against sum, changes the gradient's magnitude by a factor of $W$ and is therefore exactly equivalent to scaling the [[Learning Rate]] by $W$; pick mean, keep it, and let $\eta$ carry the scaling. And the **[[Random Seed]]** has to be handled in two directions at once here: the same seed everywhere for the parameter initialization, so that step 2 of the algorithm holds, and a different one per worker for the data order and for any stochastic layer, since identical seeds everywhere make every worker draw the same shard and quietly reduce the effective batch back to $b$.

## Failure modes

- **A straggler setting the pace of every iteration.** One worker on a throttled device, sharing a machine with another job, or holding a shard that happens to be slower, sets $T_{\text{iter}}$ for all $W$ of them. The symptom is aggregate utilization falling as workers are added while per-worker utilization stays high, and the arithmetic above says it gets worse rather than better with scale. Backup workers are the answer that keeps the update exact.
- **Uneven or overlapping shards.** A final shard with fewer instances, or a sampler that hands the same instance to two workers, makes the unweighted average of local gradients something other than the gradient on the batch. Every framework's answer is to drop the incomplete last shard, and the reason is this rather than tidiness: a silently reweighted objective produces a fit nobody can reconstruct.
- **An effective batch pushed past the critical batch size.** Past that point the steps to a target error stop falling, so the extra machines buy nothing but a larger bill, and the measured three-region curve says the point exists for every workload. The failure is invisible from inside the run, because loss per step still looks fine; it shows only in time or instances to a target error against a smaller $W$.
- **A learning rate carried across a change in $W$.** The effective batch changed by a factor of $W$ and the rate did not, so the run either crawls or, after linear scaling with no warmup, diverges in the first few epochs when the parameters are still moving fast. Both directions are the [[Mini-Batch Gradient Descent]] failure arriving through the worker count, where it is easier to miss because no line of the training script mentions the batch size.
- **Copies drifting apart.** Anything that updates one copy and not the others, a worker that skips the reduce after a recoverable error, a parameter not registered for reduction, a buffer updated locally, leaves $W$ slightly different models. The loss curve looks normal, because each worker reports its own; only the rank that writes checkpoints decides what actually ships.
- **Statistics computed per shard rather than per batch.** Any operation whose result depends on the whole batch, batch normalization being the standard case, sees $b$ instances rather than $Wb$ when it runs locally, so the objective itself changes with the worker count and two runs at different $W$ are not comparable. This is a genuine objective change rather than a numerical one, which is what makes it hard to find.

## Implementation

**scikit-learn 1.6 has no distributed training.** There is no estimator in it that computes gradients on several machines and combines them, and there is no API to add one: the library's own page on parallelism describes `n_jobs` as joblib-managed parallelism, processes under the default `loky` backend or threads under another, with no mention of more than one machine anywhere.

`n_jobs=-1` is therefore not this method, and reading it as this method is the substantive error rather than a loose analogy. What joblib parallelizes in scikit-learn is **independent tasks on one machine**: the folds of a [[Cross-Validation]], the candidates of a [[Grid Search]] or a [[Randomized Search]], the trees of a forest under [[Bagging]]. Nothing is reduced, because nothing needs to be: each task owns its own fitted object and they never have to agree. The gradient-accumulating loop above has a synchronisation point in the middle of every iteration, and that is exactly the thing joblib's model has no place for.

Multi-machine scikit-learn work runs through a joblib backend supplied by another project, and it parallelizes candidates rather than gradients:

```python
# scikit-learn 1.6 with joblib: candidates across a cluster, NOT gradients.
import joblib
from dask.distributed import Client

client = Client("scheduler-address:8786")        # or Client() for a local cluster
with joblib.parallel_backend("dask"):
    search.fit(X, y)                             # each candidate fitted whole, on one worker
```

```python
# the same shape with Ray as the backend
import joblib
from ray.util.joblib import register_ray

register_ray()
with joblib.parallel_backend("ray"):
    search.fit(X, y)
```

Both dispatch whole `fit` calls to cluster workers. Dask's own documentation is explicit that this route is for workloads whose data fits in memory and whose parallelism is in the number of independent operations, and that larger-than-memory data needs its incremental estimators instead. Those incremental wrappers call `partial_fit` over chunks, which is one parameter copy walking through the data sequentially: that is [[Out-of-Core Learning]], on one machine, and it is not this either.

The method as described lives in the deep learning frameworks. What differs between them is which part of $M_1$ they replicate and where the reduce happens:

- **`torch.distributed` with `DistributedDataParallel`.** One process per device, each with a full copy of the parameters and the optimizer state. Gradients are grouped into buckets and each bucket is all-reduced as soon as its parameters' gradients are ready, so the communication overlaps the backward pass instead of following it. The reduce is an average, so the update is the identity above with no rescaling. `DataParallel`, the older single-process multi-device class, is a different mechanism, and PyTorch's own documentation directs users away from it toward this one.
- **Horovod**, from Sergeev and Del Balso (2018), which took the ring all-reduce out of the HPC world and made it the default gradient exchange for framework-agnostic training over MPI or NCCL. Its contribution is the communication pattern rather than the algorithm, which is why the arithmetic of the reduce above is stated in ring terms.
- **DeepSpeed**, which implements ZeRO. Data-parallel in its gradients and not in its memory: the optimizer state, then the gradients, then the parameters themselves are sharded across the $W$ workers instead of replicated, so the per-worker floor falls by up to a factor linear in $W$ and the arrangement stops being purely this method.
- **A parameter server**, the OSDI 2014 design above, still the arrangement behind TensorFlow's `ParameterServerStrategy` and behind most large recommender training, where the parameters are a sparse embedding table far too large to replicate. This is the asymmetric arrangement, and it is chosen for the memory rather than for the speed.
- **Megatron-LM** for the combination, where data parallelism is one of several axes rather than the whole story. See [[Model Parallelism]].

The paper that carries both parallelisms and the asynchronous parameter server in one place is Dean, Corrado, Monga, Chen, Devin, Le, Mao, Ranzato, Senior, Tucker, Yang and Ng ("Large Scale Distributed Deep Networks", NeurIPS 2012), whose DistBelief is where Downpour SGD and the sharded parameter server come from.
