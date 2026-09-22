---
note_kind: concept
aliases:
  - sampling
  - probability sampling
  - probability-based sampling
  - sampling frame
  - population
up: "[[Training Set]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## Definition

Sampling is drawing a subset of a population according to some rule, so that work done on the subset stands in for work on the whole. The word "sample" has two readings in this vault and this note uses only the first: here a sample is the subset that was drawn, while one row of the design matrix is a [[Training Instance]], which is the note that owns `sample` in that second sense.

Sampling is not one step in the workflow. It happens wherever the whole of something is more than you can or want to handle, which is most places, and the decision of how to draw is usually made once, quietly, and never revisited.

## Formal statement

Let the population be $U = \{1, 2, \dots, N\}$, the finite set of units the answer is wanted about. The **sampling frame** is the list of units the procedure can actually reach, and it is not automatically $U$: a unit that belongs to the population but is missing from the frame cannot be drawn however long the procedure runs. A **sampling design** assigns a probability to each subset $S \subseteq U$, and what that design implies for a single unit is its **inclusion probability**

$$\pi_i = P(i \in S)$$

the chance that unit $i$ ends up in the drawn sample. That one quantity sorts every procedure in this folder into two kinds, and the split is a difference in what can be recovered afterwards rather than a difference in quality.

**Probability sampling** is the case where $\pi_i$ is known for every unit and $\pi_i > 0$ for every unit. Both halves carry weight. Positivity says every unit could have been drawn. Knownness says the draw can be undone arithmetically once it is done, since a unit that was ten times less likely to appear can be counted ten times as heavily to compensate. Under simple random sampling without replacement of $n$ units from $N$,

$$\pi_i = \frac{n}{N} \qquad \text{for every } i$$

which is the flat case that [[Random Sampling]] carries out. [[Stratified Sampling]] and [[Weighted Sampling]] are the same idea with the $\pi_i$ deliberately unequal and still known.

**[[Nonprobability Sampling]]** is the case where $\pi_i$ is unknown, or is zero for some units. The zero is the harsher half, and it is the reason this is a different kind of thing rather than a sloppier version of the same thing. If $\pi_i = 0$ then $P(i \in S) = 0$ at every sample size, so no unit that can never be drawn becomes representable by collecting more: a hundred times more data from the same frame is a hundred times more of the same frame. Sampling noise shrinks as $n$ grows because it is variance. Zero inclusion probability does not shrink, because it is not variance, and this is the distinction that makes the two failure modes in [[Nonrepresentative Training Data]] behave so differently.

## Where it is used

Four distinct uses, and the one everybody names first is only one of them.

**Splitting.** Carving a [[Training Set]] and a [[Testing Set]] out of the data is the use that gets referenced most, and it is where [[Random Sampling]] and [[Stratified Sampling]] live: the first draws uniformly, the second forces the split to reproduce the population mix on a named variable rather than leaving it to a fair draw. This is one application of sampling, not the definition of it.

**Monitoring.** A running system emits far more events than anyone would store or inspect, so what is watched is a sample of them. [[Reservoir Sampling]] is the procedure for exactly this shape, since an event stream has no known length, and [[Stream Processing]] is the setting it reads from.

**Processing cost.** Sometimes all the data is available and processing it is still infeasible on time or money, so a subset is drawn and the subset is what gets processed. [[Weighted Sampling]] is what to reach for here when the units are not equally worth keeping, and [[Importance Sampling]] is what to reach for when the distribution you must answer about is not the one you can draw from.

**Experimentation.** Fit a candidate on a subset first and see whether it is promising at all before committing the time to fit it on everything. The subset is a sample, and if it was drawn badly the promise it shows is about the subset.

Across all four, the choice of procedure is what decides whether the conclusion transfers, and [[Nonprobability Sampling]] is the family where it does not transfer in a way no amount of extra data repairs.
