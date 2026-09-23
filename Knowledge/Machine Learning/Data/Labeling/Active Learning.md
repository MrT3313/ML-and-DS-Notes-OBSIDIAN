---
note_kind: method
aliases:
  - active learning
  - query learning
  - uncertainty sampling
  - query-by-committee
  - query by committee
  - pool-based sampling
up: "[[Training Set]]"
sources:
  - "[[DMLS Ch04 Training Data]]"
  - "[[DMLS Ch06 Model Development and Offline Evaluation]]"
confidence: draft
---

## What it does and when

Active learning chooses which instances to pay to have labelled, instead of labelling whatever a random draw returns. The premise is that a model can reach a given accuracy from fewer labels if it gets to pick what it learns from, because most instances in a large pool are ones the model already handles and labelling them buys nothing. Reach for it when unlabelled data is cheap and plentiful, annotation is the binding cost, and the budget is a small fraction of the pool. It is also called query learning, and the two names describe the same loop from the two ends: the learner is active rather than passive, and what it does is ask.

A query here is a request sent to a person: one unlabelled instance, handed over for one label. It is not the database sense of [[Point Query]], which is an equality lookup that returns records already stored. Nothing is looked up here, because the thing being asked for does not exist yet.

The contrast with the other answers to a label shortage is about where the human goes. [[Weak Supervision]] removes the annotator, replacing per-instance judgement with heuristics applied at scale. Active learning keeps the annotator and spends them better, which makes it the right choice when the label genuinely requires a judgement no rule captures, a radiologist's read or a lawyer's classification, and the wrong one when the labels are cheap enough that [[Random Sampling]] over the pool costs less than the machinery.

## Algorithm

Write the labelled pool as $\mathcal{L}$, the unlabelled pool as $\mathcal{U}$, and the budget as $B$ labels.

1. **Seed.** Label a small initial $\mathcal{L}$, drawn at random. It has to be random, and it has to be large enough and balanced enough to fit a model whose probabilities mean something, because everything downstream is that model's opinion.
2. **Fit** the current model on $\mathcal{L}$.
3. **Score** every instance in $\mathcal{U}$ by the query strategy, a function of the fitted model's output on that instance alone.
4. **Select** the top $b$ instances by that score.
5. **Query.** Send them to the annotator and get labels back.
6. **Move** them from $\mathcal{U}$ to $\mathcal{L}$.
7. **Refit and repeat** from step 2, until the budget $B$ is spent or the score on a held-out set stops moving.

The ordering carries the content. Steps 2 and 3 cannot be swapped or merged: the scores are properties of a model fitted on everything labelled so far, so they are stale the instant a label arrives, which is why the loop refits rather than scoring the pool once and working down the list. And the held-out set in step 7 is not carved out of $\mathcal{L}$, for the reason given in the failure modes.

### Three scenarios

The loop above is one of three arrangements, and they differ in what the learner is allowed to ask about.

- **Pool-based sampling** is the arrangement written above and the usual case: a fixed collection of unlabelled instances is available, the learner scores all of it and picks. It assumes the whole pool can be scored each round, which is the cost that grows.
- **Stream-based selective sampling** sees one instance at a time and must decide, then and there, to query it or discard it, with no ability to compare it against instances not yet arrived. It is the right shape when data arrives as a stream or when the pool is too large to score.
- **Membership query synthesis** lets the learner construct an instance from scratch and ask about that, rather than picking one that occurred. It is strictly more powerful and often unusable: Settles reports Lang and Baum's 1992 experiment in which a network generating handwritten character queries produced hybrid shapes that were not any character, so the human oracle had nothing to answer. It works where the oracle is an experiment rather than a person, as in the robot scientist that synthesises growth media and reads off whether the yeast thrived.

### Uncertainty sampling

Query the instance the model is least sure about. There are three standard measures of "least sure", they are not the same function, and they disagree once there are more than two classes. Writing $P_{\theta}(y \mid \mathbf{x})$ for the fitted model's posterior:

**Least confident**, the model's own belief that it will get this one wrong, which is its expected 0/1 loss on the instance:

$$\mathbf{x}^{*}_{LC} = \arg\max_{\mathbf{x}} \Big(1 - P_{\theta}(\hat{y}_1 \mid \mathbf{x})\Big), \qquad \hat{y}_1 = \arg\max_{y} P_{\theta}(y \mid \mathbf{x})$$

**Margin**, the gap between the two most probable labels, smallest first:

$$\mathbf{x}^{*}_{M} = \arg\min_{\mathbf{x}} \Big(P_{\theta}(\hat{y}_1 \mid \mathbf{x}) - P_{\theta}(\hat{y}_2 \mid \mathbf{x})\Big)$$

**Entropy**, over the whole posterior:

$$\mathbf{x}^{*}_{H} = \arg\max_{\mathbf{x}} \Big(-\sum_{k} P_{\theta}(y_k \mid \mathbf{x}) \log P_{\theta}(y_k \mid \mathbf{x})\Big)$$

Least confident reads one number off the posterior and margin reads two, so both throw away everything below the top two classes; entropy reads all of them. The consequence is specific rather than vague. Take an instance where one class is ruled out and the other two are tied: margin and least confident both rate it highly informative, since the decision between the live pair is genuinely open, while entropy rates it below an instance whose mass is spread evenly over all three, because there is less total uncertainty to remove. Which behaviour you want follows from the objective, entropy when the loss being minimised is log loss, margin when it is classification error, and empirical comparisons across the three have come out mixed. In the binary case the question does not arise: all three reduce to querying the instance whose posterior is closest to $0.5$.

### Query-by-committee

The second strategy is disagreement, and its precise name is query-by-committee, introduced by Seung, Opper and Sompolinsky in 1992. Maintain a committee $\theta^{(1)}, \dots, \theta^{(C)}$ of models all consistent with $\mathcal{L}$ so far, and query the instance they disagree on most. The committee is an [[Ensemble Learning|ensemble]] used backwards: an ensemble pools its members' predictions and treats their disagreement as the noise it is averaging away, while query-by-committee keeps the disagreement and throws the prediction away, which is why query-by-bagging and query-by-boosting build committees with exactly the machinery bagging and boosting already provide.

Two measures of disagreement are standard. **Vote entropy**, where $V(y_k)$ counts the committee members predicting class $k$:

$$\mathbf{x}^{*}_{VE} = \arg\max_{\mathbf{x}} \Big(-\sum_{k} \frac{V(y_k)}{C} \log \frac{V(y_k)}{C}\Big)$$

**Average Kullback-Leibler divergence to the consensus**, which uses each member's full posterior rather than its hard vote:

$$\mathbf{x}^{*}_{KL} = \arg\max_{\mathbf{x}} \frac{1}{C}\sum_{c=1}^{C} D\big(P_{\theta^{(c)}} \,\|\, P_{\mathcal{C}}\big), \qquad D\big(P_{\theta^{(c)}} \,\|\, P_{\mathcal{C}}\big) = \sum_{k} P_{\theta^{(c)}}(y_k \mid \mathbf{x}) \log \frac{P_{\theta^{(c)}}(y_k \mid \mathbf{x})}{P_{\mathcal{C}}(y_k \mid \mathbf{x})}$$

with the consensus $P_{\mathcal{C}}(y_k \mid \mathbf{x}) = \frac{1}{C}\sum_{c} P_{\theta^{(c)}}(y_k \mid \mathbf{x})$. Vote entropy is the committee's version of entropy-based uncertainty sampling; average KL divergence prefers the instance on which some member departs furthest from what the rest believe.

What both strategies are really doing is buying labels near the [[Decision Boundary]]. An instance the model is confident about sits deep inside a decision region, and its label, whatever it turns out to be, moves the fitted boundary hardly at all; an instance whose posterior is near a tie sits on the boundary, where the label decides which side the surface passes. That is the whole intuition behind labelling what the model is most confused about, and it is also the source of half the failure modes, since "near the boundary" and "unrepresentative" are easy to confuse.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| query strategy | | none, you pick one | not ordered, so there is no "increasing": moving from least confident to margin to entropy widens how much of the posterior is read, and moving to query-by-committee replaces the posterior with disagreement among models | pick by objective, entropy for log loss and margin for classification error, and by whether a single calibrated posterior is available at all. Measure the alternatives against a random-selection baseline on the same budget, because that baseline is beaten less often than the literature suggests |
| batch size per round | $b$ | 1, one query at a time | fewer refits and parallel annotation, but the top $b$ instances are scored by one stale model and are typically near-duplicates of each other, so the marginal label in a batch buys much less than the first | keep $b$ as small as the annotation pipeline tolerates. Above roughly one query per refit, stop taking the top $b$ by score and add an explicit diversity term, otherwise the batch is one query repeated |
| committee size | $C$ | 2 in the original formulation | more distinct disagreement patterns and a finer-grained vote entropy, at $C$ fits per round; the gain flattens quickly and the members become correlated | small values work. Two or three have been reported as effective, there is no agreed figure, and correlated members buy nothing for the same reason they buy nothing in [[Ensemble Learning]] |
| labelling budget and stopping rule | $B$ | none, set by money | more labels and a better model, up to the point where the held-out score plateaus and further labels are spent on nothing | the honest stopping rule is external: stop when the cost of the next label exceeds the cost of the errors it would prevent. Self-stopping rules that watch the learner's own confidence plateau exist, and in practice the budget runs out first |
| seed set size | $m_0$ | none | a larger random seed set gives a first model whose uncertainty is worth acting on, at the price of spending budget before any selection happens | large enough to cover every class and fit a stable model. Below that the loop is selecting on noise, which is the cold start in the failure modes |

Pinned rather than tuned: the random seed governing the initial draw and any tie-breaking among equally scored instances, which changes which instances are queried and so which model comes back, but has no value to search over ([[Random Seed]]).

## Failure modes

- **The labelled pool stops being a random sample of anything.** This is the failure most easily missed, because nothing about it looks wrong. After the first round, $\mathcal{L}$ is a deliberately biased set, over-representing the boundary and implicitly shaped by the model class that did the selecting. Two consequences follow. Any estimate computed on a slice of $\mathcal{L}$, a held-out split or a [[Cross-Validation]] fold over it, estimates performance on the biased distribution rather than on the deployment one, so the number is measuring the wrong quantity and usually flattering: a set concentrated near the boundary is harder, and a set concentrated on whatever the strategy favoured may be easier, and which it is cannot be read off the number. The held-out set therefore has to be drawn at random before the loop starts and never touched. The other consequence is that the training set is tied to the model class that built it, so it may transfer badly when the model is later replaced, which is a cost paid long after the labelling is done.
- **Batch redundancy.** Take the $b$ most uncertain instances and they are frequently near-duplicates, all sitting in the same ambiguous corner, because uncertainty is a smooth function of the input and the top of the ranking is a neighbourhood rather than a spread. Labelling all $b$ buys roughly what labelling one would have bought. Selecting the top $b$ by an instance-level score ignores the overlap in information content between them by construction, and off-the-shelf strategies used this way have been found worse than random selection.
- **Cold start.** The first model is fitted on a handful of labels and its posterior is close to meaningless, so the first rounds of "uncertainty" are noise, and the labels they spend are wasted at exactly the point in the budget when they were most valuable. The loop then compounds the error, because the next model is fitted on the instances the noise picked.
- **Outliers score as maximally uncertain.** The model is unsure about a corrupted record, a mislabelled-in-the-source instance, or a one-off that no future instance resembles, precisely because nothing like it has been seen. It goes to the top of the ranking, consumes budget, and teaches the model about a region of the input space nothing depends on. Strategies that weight uncertainty by how dense the surrounding region is exist for this reason.
- **A miscalibrated model.** Every uncertainty measure above is a function of $P_{\theta}(y \mid \mathbf{x})$ and assumes those numbers track how often the model is actually right, which is exactly the property [[Model Calibration]] measures and the one a recalibration map repairs. A model whose scores are monotone but not calibrated ranks instances in an order that has nothing to do with their informativeness, and the loop is then reading noise with great precision. This bites hardest at the start, where the cold start guarantees poor calibration.
- **Membership query synthesis producing instances nobody can label.** Given freedom to construct its query, a learner will construct whatever maximises its criterion, which is generally not a thing that occurs. The annotator is handed an input with no answer, and the round is lost.

## Implementation

**scikit-learn 1.6 has no active learning API.** There is no query strategy, no selector and no loop anywhere in it; the API index has `sklearn.semi_supervised` and nothing adjacent. The pieces are all there, though, and the pool-based loop is short, because a query strategy is a function of `predict_proba` and nothing else.

scikit-learn 1.6 and NumPy 2.x, one round of margin sampling:

```python
import numpy as np
from sklearn.calibration import CalibratedClassifierCV
from sklearn.linear_model import LogisticRegression

def margin(probs):                                  # smaller is more informative
    top2 = np.sort(probs, axis=1)[:, -2:]
    return top2[:, 1] - top2[:, 0]

labelled = list(seed_idx)                           # random, from step 1
pool = [i for i in range(len(X)) if i not in set(labelled)]

for _ in range(n_rounds):
    clf = CalibratedClassifierCV(LogisticRegression(max_iter=1000), cv=5)
    clf.fit(X[labelled], y[labelled])

    probs = clf.predict_proba(X[pool])              # (len(pool), n_classes)
    chosen = [pool[j] for j in np.argsort(margin(probs))[:batch_size]]

    y[chosen] = annotate(X[chosen])                 # the human, step 5
    labelled += chosen
    pool = [i for i in pool if i not in set(chosen)]
```

`CalibratedClassifierCV` is doing real work here rather than decorating the example, since the failure mode above is that uncalibrated scores make every measure meaningless, and [[Model Calibration]] is where the wrapper's arguments, the maps behind them and the held-out requirement they all rest on are set out. Its `method` defaults to `"sigmoid"`, Platt scaling, which is the right default in this loop: `"isotonic"` is non-parametric and needs far more data than a seed set has, and it overfits badly when it does not get it. Swapping `margin` for `-probs.max(axis=1)` gives least confident and for `-(probs * np.log(probs)).sum(axis=1)` gives entropy, which is the whole difference between the three measures in code. A committee is a list of estimators fitted on bootstrap resamples of `X[labelled]`, with the scoring function replaced by vote entropy over their `predict` outputs or by average KL divergence over their `predict_proba` outputs.

Where the method lives as a library, checked on 22 September 2026: `scikit-activeml` is current and built directly on scikit-learn, at version 1.0.0 released December 2025, requiring scikit-learn 1.6 or later, and it supplies each pool-based strategy as an object with a `query` method: `UncertaintySampling` takes the three measures above as a `method` argument, and `QueryByCommittee` takes vote entropy or KL divergence the same way. Note a trap in the older recommendation: the active learning library commonly cited as modAL installs from PyPI as `modAL-python`, whose last release is 0.4.2.1 from June 2023, while the PyPI name `modAL` now resolves to the Modal cloud platform's client and is an unrelated package. `libact` (last release 2019) and `ALiPy` (2021) are likewise no longer maintained.

The survey the strategies above are drawn from is Settles, "Active Learning Literature Survey", University of Wisconsin-Madison Computer Sciences Technical Report 1648 (2009), which defines the three scenarios, the three uncertainty measures and both query-by-committee disagreement measures in one place, and is still the reference for the vocabulary.
