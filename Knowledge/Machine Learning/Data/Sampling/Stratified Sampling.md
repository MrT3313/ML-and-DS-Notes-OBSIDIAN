---
note_kind: method
aliases:
  - stratified sampling
  - stratified split
  - stratification
  - strata
  - stratum
  - StratifiedShuffleSplit
  - proportional allocation
up: "[[Sampling]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
  - "[[DMLS Ch04 Training Data]]"
confidence: draft
---

## What it does and when

Divide the population into the groups you care about, the strata, and sample from each group separately rather than from the pool as a whole. Take 1% from class A and 1% from class B and both come back at 1%, however lopsided the ratio between them, because the fraction is applied inside each group instead of across all of them. That is the whole promise: a group is represented at its own rate by construction rather than in expectation, so nothing about it is left to the draw. Reproducing the population mix in a [[Testing Set]] is the application reached for most often, and the technique is the same wherever else [[Sampling]] must not leave a group to chance. Prefer it to plain [[Random Sampling]] when a variable strongly related to the target is unevenly distributed and a fair draw could still misrepresent it, which is likeliest when $m$ is small or a stratum is rare. The cost is that you must name the variable that matters, a modelling judgement rather than a mechanical step.

## Algorithm

1. Name the stratification key, the variable the split must represent faithfully. It must be categorical, so a continuous variable is binned first. In the [[California Housing|housing dataset]] median income is the variable most correlated with the target, so `pd.cut` slices it into five categories at 1.5, 3.0, 4.5 and 6.0 (income in tens of thousands of dollars).
2. Let the strata $S_1, \dots, S_k$ partition the dataset, with shares

   $$w_s = \frac{|S_s|}{m}, \qquad \sum_{s=1}^{k} w_s = 1$$

3. Under proportional allocation, draw $w_s \cdot m_{\text{test}}$ instances at random from within each stratum $s$, so the realised test share equals $w_s$ up to rounding. [[Random Sampling]] matches it only in expectation: the share it returns scatters around $w_s$ with variance $w_s(1 - w_s)/m_{\text{test}}$, and the far end of that scatter, for a small enough $w_s$, is a stratum that comes back empty, at the miss probability derived in [[Random Sampling]]. Allocating from within each stratum removes the scatter and the empty draw together, since a draw taken inside a group cannot return none of it.
4. Drop the proxy column from both halves right afterwards. It existed only to steer the split, and left in place it leaks in as a [[Feature]].

Create a categorical proxy, split on it, drop it: that move is the part worth remembering, and it generalizes to any continuous variable you need represented faithfully.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| number of strata | $k$ | none, you choose the binning | mix matched more finely, but each stratum holds fewer instances until some are too small to sample from | enough that every stratum is well populated, five income bands on the housing data is the worked case and no stratum should be tiny |
| bin edges | - | none | - | place cuts where the variable's mass actually is, check the category counts before splitting |
| `test_size` | $r$ | 0.25 in `train_test_split`, 0.1 in `StratifiedShuffleSplit` | test estimate less noisy, training set smaller | 0.2 at moderate $m$, less as $m$ grows |
| `n_splits` | - | 10 in `StratifiedShuffleSplit` | more distinct stratified splits produced, linear cost | leave at 1 unless you genuinely need several splits |

`random_state` is left out on purpose: it fixes which rows are drawn from each stratum, so it changes the split that comes back, but it is pinned rather than tuned. Fix one integer, see [[Random Seed]].

## Failure modes

- Strata that are not a partition. Everything above assumes each unit lands in exactly one $S_s$, and that assumption fails whenever the variable you care about is multi-valued per unit: an instance carrying several labels at once ([[Multilabel Classification]]), overlapping customer segments, a customer trading in two regions. The group shares then sum past one and the per-stratum allocations double-count every unit in more than one group, so there is no partition to allocate over and stratification on that variable is not defined. Three ways out, each with a price: impose a membership rule that picks one group per unit, and accept that the rule is now a modelling choice; cross the variables into a finer partition, one stratum per observed combination, and accept that $k$ grows while the strata shrink; or leave that variable unstratified and stratify on something else. scikit-learn 1.6 takes the middle route silently. Hand `stratify` a 2D indicator matrix and `StratifiedShuffleSplit` joins each row into one string before counting classes, so what it stratifies on is the label combination and not the labels, and because most rows of real multilabel data are unique it then refuses with `The least populated class in y has only 1 member`. `StratifiedKFold` rejects the same input outright, with `Supported target types are: ('binary', 'multiclass')`, so the two routes through the library do not agree with each other.
- Too many strata. Push $k$ up and some stratum holds a handful of instances, its test allocation rounds to zero or one, and the guarantee evaporates. `StratifiedShuffleSplit`, which is what `train_test_split` reaches for whenever `stratify` is set, refuses when any stratum holds fewer than two units, and refuses again when the absolute train or test count falls below $k$. `StratifiedKFold` draws its line elsewhere, warning when the smallest stratum has fewer units than `n_splits` and raising only when every stratum does.
- Stratifying on a variable unrelated to the target. You pay the complexity, get none of the variance reduction, and the variable that actually drives the target is still left to chance.
- Stratifying on the target itself, or on a binning of it. Forcing the label distribution to match makes the test set easier than a fresh draw from the deployment distribution, so the [[Generalization]] estimate flatters exactly the quantity being measured.
- Leaving the proxy column in place. `income_cat` is engineered from a feature the model already sees, so if it survives the split the model trains on a hand-built summary of the target's best predictor.
- Reading `StratifiedShuffleSplit`'s splits as folds. Unlike [[Cross-Validation]], its test sets are independent draws and are not guaranteed mutually exclusive across iterations.

## Implementation

pandas 2.x, building the categorical proxy:

```python
import numpy as np
import pandas as pd

housing["income_cat"] = pd.cut(housing["median_income"],
                               bins=[0., 1.5, 3.0, 4.5, 6., np.inf],
                               labels=[1, 2, 3, 4, 5])
```

scikit-learn 1.6, one stratified split:

```python
from sklearn.model_selection import train_test_split

strat_train_set, strat_test_set = train_test_split(
    housing, test_size=0.2, stratify=housing["income_cat"], random_state=42)

for set_ in (strat_train_set, strat_test_set):
    set_.drop("income_cat", axis=1, inplace=True)
```

scikit-learn 1.6, several distinct stratified splits, the only reason to prefer the longer route:

```python
from sklearn.model_selection import StratifiedShuffleSplit

splitter = StratifiedShuffleSplit(n_splits=10, test_size=0.2, random_state=42)
strat_splits = []
for train_index, test_index in splitter.split(housing, housing["income_cat"]):
    strat_splits.append((housing.iloc[train_index], housing.iloc[test_index]))

strat_train_set, strat_test_set = strat_splits[0]
```
