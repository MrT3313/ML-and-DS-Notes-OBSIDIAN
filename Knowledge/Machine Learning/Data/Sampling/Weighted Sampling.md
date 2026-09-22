---
note_kind: method
aliases:
  - weighted sampling
  - weighted random sampling
  - probability proportional to size
up: "[[Sampling]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Give each unit a weight, and let the weight set how likely that unit is to be drawn. Reach for it in two situations. The first is domain knowledge: you know some units are worth more to the model than others, recent transactions over year-old ones, hard cases over easy ones, and you want the drawn set to reflect that instead of pretending every row is interchangeable. The second is correction: the data you hold comes from a different distribution than the one the model will meet, you know roughly what both look like, and upweighting the underrepresented groups pulls the drawn set back toward the deployment mix.

One distinction decides whether this note is the one you want, and it is easy to get wrong because both things are called weights. **A weight here decides how likely an instance is to be drawn. A weight that decides how much a drawn instance counts once it is in the fit is [[Loss Reweighting]].** Weighted sampling changes which rows the model sees; loss reweighting changes what each row contributes to the [[Cost Function]] and therefore how much it can move the decision boundary. They can be used together, they can substitute for each other in some settings, and they are not the same operation. If the answer changes when you ask "does the row appear twice, or once with twice the influence", you are in the right place to care.

## Algorithm or formula

Weights are relative, not probabilities, so the first thing that happens to them is normalization. With non-negative weights $w_1, \dots, w_N$, at least one of them positive, unit $i$ is drawn with probability

$$p_i = \frac{w_i}{\sum_{j=1}^{N} w_j}$$

So the weights $(0.5, 0.3, 0.2)$ and the weights $(5, 3, 2)$ are the same design, and units A, B, C are drawn 50, 30 and 20 percent of the time under both.

**Correcting a distribution mismatch.** This is the use with an actual formula behind it, rather than a judgement call. Let the data hold group $g$ at share $q_g$ and let the population hold it at share $p_g$. To make the drawn set look like the population, set

$$w_g \;\propto\; \frac{p_g}{q_g}$$

which is the inverse of the sampling design's distortion, the same cancellation that makes the Horvitz-Thompson estimator in [[Nonprobability Sampling]] work. Groups the data oversamples get weights below one, groups it undersamples get weights above one, and only the ratios matter because the normalization above rescales everything anyway.

Worked: the data is 25 percent red and 75 percent blue, and the real world is 50 percent each. Then

$$w_{\text{red}} \propto \frac{0.5}{0.25} = 2, \qquad w_{\text{blue}} \propto \frac{0.5}{0.75} = \frac{2}{3}, \qquad \frac{w_{\text{red}}}{w_{\text{blue}}} = 3$$

so red units are given three times the weight of blue ones. The ratio, not the pair of numbers, is the answer: any weights in a 3 to 1 ratio produce the same draw.

**What unequal weights cost.** Every draw is still one row, but rows that repeat or dominate carry less information than their count suggests. The usual measure is the effective sample size,

$$n_{\text{eff}} = \frac{\left(\sum_i w_i\right)^2}{\sum_i w_i^2}$$

which equals $n$ when all weights are equal and collapses toward 1 as one weight swamps the rest (Owen, *Monte Carlo theory, methods and examples*, chapter 9). Two thousand rows drawn under weights concentrated on a handful of units can be worth a few dozen, and nothing in the row count says so.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| the weights | $w_i$ | uniform, every $w_i = 1$ | raising one unit's weight raises its draw probability roughly in proportion and lowers every other unit's, since the denominator grows too | for distribution correction, compute $w_g = p_g / q_g$ from counted group shares; for value weighting, set them from domain knowledge and check $n_{\text{eff}}$ afterwards |
| sample size | $n$ | none, you choose it | the drawn set matches the weighted distribution more closely, and the variance of anything estimated from it falls | large enough that $n_{\text{eff}}$, not $n$, is the number you would have been happy with |
| `replace` | - | `True` in `numpy.random.Generator.choice`, `False` in `pandas.DataFrame.sample` | with replacement, $p_i$ holds exactly on every draw and duplicates appear; without replacement, the weights govern each successive draw rather than the final inclusion probabilities, so $\pi_i$ is no longer proportional to $w_i$ | with replacement when the weights are the point and duplicate rows are harmless, without when a repeated row would distort the fit |

`random_state` is pinned rather than tuned: it fixes which units the draw returns, so it changes the output, but there is no answer to what value is better. Fix one integer for the life of the project, see [[Random Seed]].

## Failure modes

- Weights built from a guessed population, not a measured one. $w_g = p_g/q_g$ is only as good as $p_g$, and $p_g$ is a claim about the deployment distribution. Guess it and the correction moves the data toward a distribution that is also wrong, with the added cost that the mismatch is now deliberate and documented as a fix.
- Weight concentration, where a few units hold most of the mass. $n_{\text{eff}} = (\sum w_i)^2 / \sum w_i^2$ falls far below $n$, every downstream estimate is noisier than its sample size suggests, and the row count in the logs reports the wrong number with no warning.
- With-replacement draws duplicating a high-weight unit many times. The model sees the same row repeatedly, which is not more evidence, and a duplicate that lands on both sides of a later split leaks the [[Testing Set]] into the [[Training Set]].
- Weights derived from the target. Upweighting rows because of what their label is, or because a model got them wrong, puts label information into the selection rule. On a training draw that is a form of leakage, and on an evaluation draw it makes the estimate meaningless.
- Weighting instead of reweighting, or the reverse, without deciding which one the problem calls for. Drawing the rare class more often changes which rows are seen, which interacts with [[Class Imbalance]] differently from leaving the rows alone and raising their contribution to the loss, which is [[Loss Reweighting]]. Doing both by accident applies the correction twice.
- Assuming `replace=False` with weights gives inclusion probabilities proportional to the weights. It does not: successive draws renormalize over what is left, so the marginal $\pi_i$ are not $\propto w_i$, and any estimator that divides by $w_i$ to undo the draw is then wrong.

## Implementation

NumPy 2.x, using the modern `Generator`. `choice` takes `p`, which must be a probability vector summing to 1, so the weights are normalized first:

```python
import numpy as np

rng = np.random.default_rng(42)

weights = np.array([0.5, 0.3, 0.2])
p = weights / weights.sum()

draws = rng.choice(["A", "B", "C"], size=1000, replace=True, p=p)
```

pandas 2.2, which takes raw `weights` instead and normalizes them for you. A column name works when sampling rows, missing values in it are treated as zero, and infinite values are rejected:

```python
import numpy as np

data["sampling_weight"] = np.where(data["colour"] == "red", 2.0, 2 / 3)

subsample = data.sample(n=1000, replace=True,
                        weights="sampling_weight", random_state=42)
```

Python standard library, for the case with no array in hand. `random.choices` is the weighted, with-replacement draw; `random.sample` is the unweighted, without-replacement one and takes no weights at all:

```python
import random

random.choices(["A", "B", "C"], weights=[0.5, 0.3, 0.2], k=1000)
```

scikit-learn is the trap. Through 1.6 it has no weighted-sampling utility at all: `sklearn.utils.resample` is `resample(*arrays, replace=True, n_samples=None, random_state=None, stratify=None)` and there is nowhere to put a weight. The `sample_weight` argument that appears on estimator `fit` methods all over the library is [[Loss Reweighting]], a different operation entirely, and reaching for it because the name matched is the commonest way this distinction gets lost.

From 1.7 onward `resample` does gain a `sample_weight` argument, and in that one function it genuinely means this note's thing: the documentation says the values are normalized to sum to one and interpreted as the probability of sampling each data point, and it requires `replace=True`. So the same identifier means selection probability in `sklearn.utils.resample` and loss contribution in every estimator's `fit`, in the same library, and which one you get depends on where you typed it.
