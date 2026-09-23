---
note_kind: concept
aliases:
  - distributed training
  - training at scale
  - distributed machine learning
  - gradient checkpointing
  - activation checkpointing
up: "[[Gradient Descent]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## Definition

Distributed training is fitting one model on more than one machine, by splitting either the data the gradient is computed on or the parameters the gradient is computed for. It is the same search over [[Parameter Space]] that [[Gradient Descent]] describes, with the work of one iteration cut into pieces that separate machines do at the same time and then reconcile, so that each iteration still moves one agreed parameter vector.

Two axes, and a run can use both at once:

- **What gets split is the data.** Every machine holds a whole copy of the parameters and computes a gradient on its own share of the batch, and the shares are then combined. This is [[Data Parallelism]].
- **What gets split is the model.** One copy of the parameters is cut across the machines, each holding some layers or some slice of each layer, and an instance passes through all of them. This is [[Model Parallelism]].

The distinction is not a matter of taste. The two relax different constraints, and which one a given model forces is settled by arithmetic rather than by preference, which is what the next section is for.

## Formal statement

The quantity that decides the arrangement is the memory one copy of the model needs, and [[Scalability]] already carries it: for $P$ parameters at $\beta$ bytes each with $k$ copies of every parameter held at once,

$$M_{1} = P \cdot \beta \cdot k$$

That note writes the bytes per parameter as $b$. Here it is $\beta$, because $b$ is already the per-worker batch size that [[Mini-Batch Gradient Descent]] owns and this whole block turns on it. The $k$ is the interesting factor: serving holds the weights alone, $k = 1$, while a gradient-based fit holds the weights, the gradient and whatever state the update rule carries, so $k = 4$ for Adam in single precision and 16 bytes per parameter in total for mixed-precision Adam by the accounting of Rajbhandari, Rasley, Ruwase and He (ZeRO, SC 2020).

Each arrangement multiplies that figure by something different, and the two numbers that matter are the total across the cluster and the floor under any one machine.

| arrangement | total parameter memory | floor on one machine |
|---|---|---|
| one machine | $M_{1}$ | $M_{1}$ |
| [[Data Parallelism]] over $W$ workers | $W \cdot M_{1}$ | $M_{1}$, unchanged |
| [[Model Parallelism]] over $K$ stages | $M_{1}$ | about $M_{1}/K$ |

Read the right-hand column and the choice makes itself. **Data parallelism replicates $M_1$ once per worker and never lowers the floor**, so a model whose single copy does not fit on one machine cannot be trained this way at any worker count: adding workers multiplies the quantity that was already too large. **Model parallelism divides the floor and leaves the total alone**, so it is the arrangement a large model forces, and it buys room rather than throughput. Throughput is what data parallelism buys, which is why a run large enough to need both uses both.

The middle case is worth naming because it is where production systems sit. ZeRO shards the $k$ copies themselves across the $W$ data-parallel workers instead of replicating them, so the per-worker figure becomes about $P \beta k / W$ while every worker still computes a gradient on its own shard: the paper reports a factor of 4 from partitioning the optimizer state alone, 8 from also partitioning the gradients, and a reduction linear in the device count from also partitioning the parameters. That is data parallelism in its arithmetic of gradients and model parallelism in its arithmetic of memory.

### The term the parameter count leaves out

$M_1$ is not the whole training figure. The forward pass computes intermediate values at every layer, and backpropagation needs them again to form the gradient, so they are held rather than recomputed. That is **activation memory**, and it scales with the per-worker batch size $b$, the depth of the network and the width of each layer, not with $P$. A training memory figure quoted without a batch size beside it is therefore not a figure about anything.

**Gradient checkpointing**, more accurately called activation checkpointing, is the one knob on that term. It stores activations at a subset of layers and recomputes the rest during the backward pass from the nearest stored one, trading arithmetic for memory. Chen, Xu, Zhang and Guestrin (2016) give the scaling: an $n$-layer network can be trained in $O(\sqrt{n})$ activation memory at the cost of one extra forward pass per mini-batch, and in $O(\log n)$ memory at a cost of $O(n \log n)$ extra forward computation. Their measured case is a 1,000-layer residual network on ImageNet going from 48 GB to 7 GB for about 30 percent more running time. It addresses the depth, the width and the batch size, and it is what [[Model Parallelism]]'s pipelines lean on to keep a stage's activations bounded while several batches are in flight.

## Where it is used

Four notes carry the machines, the interconnect and the failure model this one assumes and does not restate. [[Distributed System]] holds the system model every arrangement on this branch is proved against, no shared memory, no shared clock, messages that can be delayed or lost, and a crashed peer indistinguishable from a slow one, which is exactly why an averaged gradient needs a synchronisation point and why a straggler cannot simply be declared dead. [[Node]] is the unit the worker count $W$ and the stage count $K$ are counts of, and its process reading matters here because one machine with eight accelerators is conventionally run as eight workers rather than one. [[High-Performance Computing]] is the tradition this work belongs to, one enormous calculation split across many machines rather than a service answering requests, and it supplies both the restart-from-a-checkpoint operating model that a multi-week run depends on and the fast private interconnect that makes a per-iteration gradient exchange affordable at all. [[Scalability]] supplies the $M_1$ above and the discipline of naming a load parameter before claiming anything scales.

What is owned here is the other half: the gradient, the parameter copy, the batch and the update. [[Mini-Batch Gradient Descent]] owns the batch size that data parallelism turns into an effective batch, [[Batch Gradient Descent]] and [[Stochastic Gradient Descent]] are its two limits, and [[Learning Rate]] is the knob that has to move with the effective batch rather than staying where it was tuned.

### Preprocessing goes out of core too

The reason for distributing rarely arrives alone. A training set too large for one machine's memory is also too large for the preprocessing to be done in one pass in memory, so zero-centering, normalizing, whitening, shuffling and batching all have to run over chunks and across machines before the first gradient is computed. [[Out-of-Core Learning]] is the regime that names the problem, and [[Feature Scaling]] and [[Standardization]] are the transforms most affected, because both need statistics over the whole training set and a chunked pass computes them incrementally rather than in one reduction. Shuffling is the step that suffers most: a shuffle inside a chunk is not a shuffle of the file, and a file in a systematic order handed to a chunked reader produces batches that are biased rather than merely noisy, which is the failure [[Mini-Batch Gradient Descent]] warns about arriving through the storage layer. Pin a [[Random Seed]] for the shard assignment so a run can be repeated.

The other reason a run ends up distributed is the model rather than the data, and [[Machine Learning Systems Design]] is where the growth of both is counted as a requirement rather than as an accident.
