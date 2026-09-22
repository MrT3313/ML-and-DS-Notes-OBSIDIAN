---
note_kind: method
aliases:
  - reservoir sampling
  - algorithm R
  - Vitter's algorithm
up: "[[Sampling]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Keep a uniform sample of fixed size $k$ from a sequence whose length is not known in advance and may never end, in a single pass, holding $k$ items and a counter and nothing else. Reach for it when both of the usual routes are closed: you cannot shuffle, because there is no end to shuffle from, and you cannot compute a selection probability up front, because that needs $N$ and $N$ is exactly what you do not have. A live event stream is the standard case, which is what [[Stream Processing]] supplies, but so is any file too large to hold or count twice.

Two guarantees make it worth the trouble, and they are the ones to check any proposed variant against. Every element the sequence has produced has the same probability of being in the reservoir. And the algorithm can be stopped at any moment, with the reservoir already a correct uniform sample of everything seen up to that moment, rather than becoming correct only once the sequence ends.

The result is about sequences rather than about models, and it holds whether or not anything is ever fitted on the reservoir. Nothing in the argument below mentions features, labels or a loss.

## Algorithm

Algorithm R, due to Alan Waterman and given this name in Vitter's 1985 paper *Random Sampling with a Reservoir* (ACM Transactions on Mathematical Software 11(1)), where it is the baseline the paper's own faster algorithms are measured against.

1. Put the first $k$ elements of the sequence into the reservoir, filling slots $1$ through $k$.
2. For each subsequent arrival, the $n$-th element of the sequence, draw an integer $i$ uniformly from $1, 2, \dots, n$.
3. If $i \le k$, replace the element in slot $i$ of the reservoir with the $n$-th element.
4. Otherwise discard the $n$-th element and keep the reservoir as it is.
5. When the sequence ends, or whenever you choose to stop, the reservoir is the sample.

Step 2 is where $N$ would have been needed and is not: the draw is over $1..n$, the count so far, which is available, rather than over $1..N$.

### Why it is uniform

The invariant is stronger than the statement about the incoming element that the steps make obvious, and it is the stronger version that has to be proved, because the weaker one would be satisfied by procedures that are not uniform at all.

**Invariant.** After $n \ge k$ arrivals, every one of the $n$ elements seen so far is in the reservoir with probability $k/n$.

**Base case, $n = k$.** The first $k$ elements are all in the reservoir, and $k/k = 1$.

**Inductive step.** Assume the invariant at $n$. Arrival $n+1$ is admitted exactly when $i \le k$, which happens with probability

$$P(\text{admit } n{+}1) = \frac{k}{n+1}$$

so the new element already has the right probability. Now take an element already resident. It leaves only if the new element is admitted **and** the slot drawn is that resident's, which is one slot out of $k$. So it survives with probability

$$\left(1 - \frac{k}{n+1}\right) + \frac{k}{n+1}\cdot\frac{k-1}{k} \;=\; 1 - \frac{k}{n+1} + \frac{k-1}{n+1} \;=\; \frac{n}{n+1}$$

and its unconditional probability of being in the reservoir after arrival $n+1$ is

$$\frac{k}{n}\cdot\frac{n}{n+1} \;=\; \frac{k}{n+1}$$

which is the invariant at $n+1$ for old and new elements alike. The invariant holds after every arrival, which is the stopping guarantee: there is no moment at which the reservoir is only partly correct.

### Faster variants

Algorithm R draws one uniform variate per arrival, $N - k$ of them in total, and so runs in $O(N)$ time no matter how small $k$ is. Vitter's contribution in the same paper is algorithms X, Y and Z, which replace the per-element test with a random **skip**: draw how many arrivals to discard before the next one is admitted, then jump past them without touching them. Algorithm Z reaches $O(k(1 + \log(N/k)))$ expected time, which the paper shows is optimal up to a constant factor. The sample they produce is distributed identically to R's; what changes is the work, which stops scaling with $N$ and starts scaling with how many admissions actually happen.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| reservoir size | $k$ | none, you choose it | memory and per-element cost both scale with $k$, the sample is more precise, and every element's inclusion probability $k/n$ rises in proportion | set it from the memory you are willing to spend and the precision the downstream estimate needs, not from the stream, since $N$ is unknown by assumption and cannot enter the choice |

The random seed is pinned rather than tuned: it fixes which elements survive, so it changes the reservoir that comes back, but there is no better or worse value. Fix one integer, see [[Random Seed]].

## Failure modes

- Reading the reservoir before $k$ elements have arrived. Until the fill phase completes the reservoir is not a sample of anything, it is a prefix, and a hand-rolled loop returns it happily with no error. Any consumer has to check the length against $k$ before treating it as a draw.
- Using the reservoir as a [[Training Set]] when recency matters. Uniformity over all time is precisely what you do not want under a drifting distribution: a stream running for a year gives a record from January the same weight as one from this morning, so the sample is a faithful picture of a world that has moved on, and the fit is stale by construction.
- Assuming a weighted version falls out of the same loop. It does not. Weighting by $w_i$ needs a different algorithm, A-Res or A-ExpJ (Efraimidis and Spirakis, *Weighted random sampling with a reservoir*, Information Processing Letters 97(5), 2006), which give each arrival the key $u_i^{1/w_i}$ for a uniform $u_i \in (0,1)$ and keep the $k$ largest keys, rather than testing each arrival against $k/n$. Bolting weights onto the $i \le k$ test of algorithm R produces a draw whose inclusion probabilities are nothing in particular, and there is no diagnostic in the output that says so.
- Splitting the stream across several workers, each keeping its own reservoir of size $k$. The union of $m$ such reservoirs is not a uniform sample of the whole stream unless the per-worker counts are equal, because each worker's elements were drawn at $k/n_j$ for its own $n_j$. Combining them correctly means carrying each worker's count and merging with those counts as weights, which is what Spark's `reservoirSampleAndCount` returns the input size for.
- Adversarial or merely non-uniform arrival order is handled correctly, and this is worth stating because it is often assumed to be a problem. The invariant makes no assumption about the order of the sequence. What order does affect is the reservoir's contents at a given instant, not the probability with which any element is in it.

## Implementation

There is no library call for this in the machine learning stack. scikit-learn 1.6 has no reservoir sampling API, and `sklearn.utils.resample` needs arrays with a known first dimension, which is the assumption reservoir sampling exists to remove. Python's `random.sample(population, k)` requires a sequence, and since Python 3.11 will not even accept a set, so it does not apply to a stream either. The loop is short enough to write, and writing it is the honest option.

Python standard library, `random`, algorithm R exactly as stated above with zero-based slots:

```python
import random

def reservoir_sample(stream, k, seed=None):
    """Uniform sample of size k from an iterable of unknown length."""
    rng = random.Random(seed)
    reservoir = []
    for n, item in enumerate(stream, start=1):
        if n <= k:
            reservoir.append(item)          # fill phase
        else:
            i = rng.randrange(n)            # uniform on 0 .. n-1
            if i < k:
                reservoir[i] = item         # admit, evicting slot i
    return reservoir     # shorter than k if the stream held fewer than k
```

Where it genuinely runs in production: Apache Spark implements it as `reservoirSampleAndCount` in `org.apache.spark.util.random.SamplingUtils`, and `RangePartitioner` calls it on every input partition to sketch the key distribution before choosing range boundaries. That is the unknown-length shape in its natural habitat, since the partitioner would otherwise have to count the data in a pass of its own before it could sample it. The returned input size is what lets the per-partition reservoirs be combined correctly, which is the merge problem named among the failure modes above.
