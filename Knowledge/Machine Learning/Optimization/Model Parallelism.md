---
note_kind: method
aliases:
  - model parallelism
  - model parallel
  - pipeline parallelism
  - pipeline parallel
  - GPipe
  - microbatch
  - model sharding
up: "[[Distributed Training]]"
sources:
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Model parallelism splits the model and keeps the data whole. One copy of the parameters is cut across machines, so machine 1 computes the first two layers, machine 2 the next two, and an instance passes through both in order. Nothing is replicated and nothing is reduced, because no parameter exists in more than one place.

Reach for it when the model does not fit, which is growth in the thing being run rather than growth in the traffic, the distinction [[Scalability]] insists on before any claim about coping is made. That is the whole condition, and it follows from the arithmetic in [[Distributed Training]]: this arrangement leaves the total parameter memory at what one whole copy needs and divides the floor under any one machine by the number of pieces, where [[Data Parallelism]] does the exact opposite. So the two are not competing answers to one question. Data parallelism answers "this is too slow" and model parallelism answers "this does not fit", and a run that is both uses both.

**The name misleads, and the misreading is worth stating before anything else.** Splitting a model across machines does not by itself make the machines work at the same time. In the plain layer split above, machine 2 cannot start until machine 1 has produced the activations it consumes, and machine 1 has nothing to do once it has handed them over. The machines are idle in turn rather than busy in parallel, and the arithmetic is unforgiving: with $K$ equal stages and one batch in flight, each stage is busy for $1/K$ of the iteration, so $K$ machines deliver the throughput of one. The total time for a forward and backward pass is the same sum of stage times a single machine would have paid, merely spread across more hardware.

What the plain split buys is therefore capacity and nothing else, which is a perfectly good thing to buy when the alternative is not training at all. Getting the machines genuinely parallel takes a second idea, and that is pipeline parallelism below.

## Algorithm

Write the parameter vector as a partition into $K$ contiguous stages, $\boldsymbol\theta = (\boldsymbol\theta_1, \dots, \boldsymbol\theta_K)$, with stage $k$ placed on one [[Node]]. The partition is of [[Parameter Space]] itself: under this arrangement the coordinates being searched are physically divided, where under data parallelism they are copied.

1. **Cut and place.** Assign each stage a contiguous block of the computation graph and the parameters it owns. Balance by measured stage time rather than by layer count.
2. **Forward, in order.** Stage $k$ receives the activations $\mathbf{a}_{k-1}$ from its predecessor, computes $\mathbf{a}_k = f_k(\mathbf{a}_{k-1}; \boldsymbol\theta_k)$, and sends $\mathbf{a}_k$ onward. Stage 1 reads the instances; stage $K$ produces the prediction.
3. **Loss at the last stage.** Only stage $K$ sees the targets, so only stage $K$ evaluates the [[Cost Function]].
4. **Backward, in reverse.** Stage $k$ receives $\partial J / \partial \mathbf{a}_k$ from its successor, forms its own $\nabla_{\boldsymbol\theta_k} J$ and also $\partial J / \partial \mathbf{a}_{k-1}$, and sends the latter back to its predecessor.
5. **Update locally.** Each stage applies the ordinary [[Gradient Descent]] step to its own $\boldsymbol\theta_k$ and to nothing else. There is no reduction step, because there is no parameter that two machines both hold.

What crosses the wire is the distinctive thing. Data parallelism exchanges gradients with respect to parameters, one gradient's worth per iteration per worker. This exchanges activations and gradients with respect to activations, whose size is set by the batch and the width of the cut rather than by the parameter count. For a model with very large layers that difference is large in this arrangement's favour, and the PipeDream measurements below put it at up to 95 percent less communication than data-parallel training on the models they tried.

The stage sequence is also why the whole run is one failure domain. Under data parallelism a lost worker costs a shard; here a lost stage owns parameters that exist nowhere else, so the iteration cannot complete and the run restarts from a checkpoint, which is the restart-from-a-checkpoint operating model [[High-Performance Computing]] treats as routine rather than as an incident.

### Pipeline parallelism

The idea is to stop feeding the stages one batch at a time. Split the batch into $M$ **microbatches** and push them through the pipeline one after another, so that while stage 2 works on microbatch 1, stage 1 is already working on microbatch 2. That is GPipe, from Huang, Cheng, Bapna, Firat, Chen, Chen, Lee, Ngiam, Le, Wu and Chen (NeurIPS 2019), and it is what makes the machines run at the same time rather than in turn.

It does not make the idleness vanish, it amortizes it. Measuring in stage-times, filling the pipeline takes $K-1$ slots before the last stage has anything to do, and draining it takes $K-1$ at the other end; pushing $M$ microbatches through $K$ stages therefore occupies $M + K - 1$ slots where a perfectly packed schedule would have taken $M$. Each stage is idle for $K-1$ of those slots, so the idle fraction, the **bubble**, is

$$\text{bubble fraction} = \frac{K - 1}{M + K - 1}$$

and the shape of that expression is the whole design guidance. At $M = 1$ it is $(K-1)/K$, which is the plain layer split and its $1/K$ utilization restated. It falls toward zero only as $M$ grows, and it grows with $K$, so adding stages to fit a larger model costs efficiency that only more microbatches buy back. GPipe's own rule of thumb is that the bubble is negligible at $M \ge 4K$. This is what turns a qualitative warning about model parallelism not being parallel into a number a planned configuration can be checked against.

Two costs come with a large $M$. For a mini-batch of $b$ instances the microbatch holds $b/M$ of them, so each stage's arithmetic gets less efficient as the microbatch shrinks toward a single row, which is the [[Mini-Batch Gradient Descent]] vectorization argument arriving one level down. And anything computed over a batch, batch normalization above all, now sees a microbatch instead. GPipe also pays for its memory with recomputation: each accelerator stores activations only at its partition boundaries and recomputes the interior during the backward pass, which is the gradient checkpointing of [[Distributed Training]] applied per stage, and it brings the peak from $O(b \cdot L)$ down to $O\big(b + \frac{L}{K}\cdot\frac{b}{M}\big)$ for $L$ total layers.

**PipeDream**, from Narayanan, Harlap, Phanishayee, Seshadri, Devanur, Ganger, Gibbons and Zaharia (SOSP 2019), attacks the same bubble from the other side and pays in a different currency. Instead of running all $M$ forward passes and then all $M$ backward passes and flushing, it interleaves them one for one, the **1F1B** schedule, which keeps every stage busy in a steady state and bounds the number of microbatches in flight. The price is that a stage's parameters may have been updated between a microbatch's forward pass and its backward pass, so the gradient would not be the gradient of the loss that was actually computed. **Weight stashing** is the fix: each stage keeps the version of its weights that a microbatch's forward pass used and loads it again for that microbatch's backward pass. What survives is staleness across stages rather than incorrectness within a microbatch, which makes this an asynchronous pipeline in the same sense that asynchronous SGD is asynchronous, and it is reported at up to about 5 times faster time-to-accuracy than data-parallel training on the models tried. GPipe's flush keeps the update exactly the one a single machine would have made; PipeDream's does not, and buys throughput with that.

### Tensor parallelism, the third axis

Splitting by layer is not the only way to cut a model, and the other way is to cut inside a layer: distribute the rows or columns of a single weight matrix across devices, have each compute its slice of the matrix product, and combine the slices. This is **tensor parallelism**, also called intra-layer model parallelism, and it is the axis that lets a single layer be larger than one device. Shoeybi, Patwary, Puri, LeGresley, Casper and Catanzaro introduced Megatron-LM in September 2019 and trained transformer models up to 8.3 billion parameters on 512 GPUs with it.

Its cost is where it sits in the loop: combining the slices needs a reduction inside every layer, on every forward and every backward pass, rather than once per iteration or once per stage boundary. That is far more communication than either arrangement above, which is why in practice tensor parallelism is kept inside a single machine, across the devices sharing its fast internal links, while pipeline parallelism spans machines and data parallelism spans the whole cluster. The three-axis combination is what large training runs actually run, and it is a design that only makes sense given the interconnect hierarchy [[High-Performance Computing]] describes.

It is worth naming even though the two axes above are the ones this material sets out, because a reader who meets tensor parallelism needs to know it is a third cut of the same kind and not a synonym for either.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| stage count | $K$ | $1$, meaning no split | per-machine parameter memory falls to roughly one $K$th of what one whole copy needs, so a larger model fits; the bubble fraction $(K-1)/(M+K-1)$ grows and one more activation transfer per boundary is added | the smallest $K$ at which one copy of the model fits the machines, and no larger |
| microbatch count | $M$ | $1$, which is the unpipelined split | the bubble falls toward zero; each microbatch holds $b/M$ instances, so per-device arithmetic efficiency falls and any batch-level statistic is computed over fewer rows | start at $M \ge 4K$, then raise until throughput stops improving or the microbatch gets too small to vectorize |
| where the cuts go | the partition | none | not ordered. It decides whether one copy fits at all, and the slowest stage sets the pipeline's rate, so an unbalanced cut wastes the whole pipeline | profile the forward and backward time of each candidate block and equalize measured time, not layer or parameter count |
| schedule | flush against 1F1B | flush, as in GPipe | 1F1B keeps stages busy through the steady state and bounds in-flight microbatches, and it introduces cross-stage staleness, so the parameters it returns are not the ones a single machine would have produced | stay with the flushing schedule while its throughput is acceptable, since it keeps the update exact; move to 1F1B when the bubble has been measured and found to dominate |
| tensor-parallel degree | $T$ | $1$ | each layer's matrices are divided $T$ ways, so a single layer larger than one device becomes possible; a reduction is added inside every layer on every pass | keep $T$ within one machine's internal interconnect, and raise it only when a single layer, not the whole model, is the thing that does not fit |

The [[Random Seed]] behaves differently here than under data parallelism. A stochastic layer lives on exactly one stage, so its draws come from that stage's generator alone and reproducibility is a property of the partition as well as of the seed: repartition the same model across a different $K$ and the run is no longer bit-comparable with the old one even at the same seed. Pin the seed per stage and record the partition alongside it.

## Failure modes

- **Expecting parallelism from the split alone.** At $M = 1$ the bubble fraction is $(K-1)/K$, so eight machines run at about one eighth utilization each and the wall clock is no better than a single machine that could have held the model. Nothing errors and no log line complains; the run is simply as slow as it was, on eight times the hardware.
- **An unbalanced partition.** The pipeline runs at the rate of its slowest stage, so a cut that gives one machine 40 percent of the compute caps every other machine at that rate and the bubble arithmetic above understates the waste. Cutting by layer count or by parameter count rather than by measured time is the usual way this happens, since a layer's parameter count and its forward time are only loosely related.
- **Activation memory concentrated at the early stages.** Under a flushing schedule stage 1 must hold the stored activations of every microbatch still in flight while the later stages hold fewer, so peak memory is not uniform across machines and the partition that balances compute can be the one that runs stage 1 out of memory. Raising $M$ to shrink the bubble makes this worse, which is the direct tension between the two knobs.
- **Staleness taken for a free lunch.** An asynchronous pipeline without weight stashing computes a microbatch's backward pass against weights its forward pass never saw, so the gradient is not the gradient of anything that was evaluated and the run can diverge or converge to something else. Weight stashing removes that particular incorrectness and leaves cross-stage staleness in place, so the parameters returned still differ from the single-machine ones and a comparison run has to allow for it.
- **Activation traffic on the wrong network.** The tensors crossing each boundary scale with the batch and the width of the cut, and under tensor parallelism a reduction crosses on every layer of every pass. Placed across machines on an ordinary network rather than inside one machine, that traffic dominates the iteration and the arrangement is slower than not splitting at all, which is the general warning [[Distributed System]] makes about a cluster losing to one computer, arriving through the interconnect.
- **One stage lost taking the run with it.** Every stage owns parameters that exist in no other place, so there is nothing to fail over to and the iteration cannot complete. Checkpoint frequency is therefore a real hyperparameter of the operating procedure even though it is not one of the method, and a long run without one loses everything since the last.

## Implementation

**scikit-learn 1.6 has no model parallelism, and no place to put it.** Its estimators fit one object in one process, and the [[Scikit-Learn Estimator API]] contract says nothing about parameters that live in several address spaces: `fit` returns `self`, with the learned attributes on that instance. `n_jobs` does not approach this either, in a way worth being exact about, since it is the knob that gets misread as both parallelisms. Under [[Bagging]] it fits separate whole estimators in separate processes on one machine, and under [[Grid Search]] it fits separate whole candidates; in both cases each process holds a complete model, which is the opposite of splitting one. Nothing in the library cuts a single model across processes, let alone across machines.

The method lives in the deep learning frameworks, and what differs between them is which cut they make and whether the update stays exact:

- **`torch.distributed.pipelining`**, PyTorch's pipeline parallelism, which splits a module into stages and drives them with a schedule you choose, the flushing and 1F1B families both included. It succeeded the earlier in-tree port of torchgpipe, and the schedule choice is exposed precisely because it is the exactness-against-throughput decision above.
- **GPipe** itself, from the TensorFlow and Lingvo side, and `torchgpipe` as the independent PyTorch implementation. Synchronous and flushing, so the update is the one a single machine would have made, with recomputation at stage boundaries for the memory.
- **PipeDream** and its successors, asynchronous and 1F1B with weight stashing, plus PipeDream-2BW (Narayanan and colleagues, 2021) which cuts the stashing memory down to two weight versions.
- **Megatron-LM** for tensor parallelism, splitting the attention heads and the feed-forward matrices of a transformer across the devices inside a machine.
- **DeepSpeed** for the combination, where tensor parallelism runs inside a machine, pipeline parallelism across machines and [[Data Parallelism]] across the resulting groups, with ZeRO sharding what the data-parallel axis would otherwise replicate.

The planning arithmetic is worth having as code rather than as a rule of thumb, because the two knobs pull against each other and the configuration is chosen before anything is run. Plain Python:

```python
def bubble_fraction(stages, microbatches):
    """Fraction of stage-time each pipeline stage spends idle."""
    return (stages - 1) / (microbatches + stages - 1)

k = 8                                  # stages, set by what fits
for m in (1, 8, 32, 64):
    print(m, round(bubble_fraction(k, m), 3))
# 1  0.875   the unpipelined split: 1/K utilization
# 8  0.467
# 32 0.184   M = 4K, GPipe's negligible-bubble rule of thumb
# 64 0.099
```

Read down that column and the tradeoff is explicit. Eight stages with one batch in flight waste seven eighths of the hardware; the same eight stages at 32 microbatches waste under a fifth, and the microbatch is one thirty-second of the batch, which is where the arithmetic efficiency and the batch-level statistics start to complain instead.
