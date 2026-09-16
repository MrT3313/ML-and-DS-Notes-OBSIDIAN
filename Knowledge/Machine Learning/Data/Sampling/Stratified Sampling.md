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
up: "[[Testing Set]]"
sources:
  - "[[HOML Ch02 End-to-End Machine Learning Project]]"
confidence: draft
---

## What it does and when

Split the population into disjoint homogeneous subgroups, the strata, and draw from each in proportion to its size, so the [[Testing Set]] reproduces the population mix by construction instead of by luck. Prefer it to plain [[Random Sampling]] when a variable strongly related to the target is unevenly distributed and a fair draw could still misrepresent it, which is likeliest when $m$ is small or a stratum is rare. The cost is that you must name the variable that matters, a modelling judgement rather than a mechanical step.

## Algorithm or formula

Let strata $S_1, \dots, S_k$ partition the dataset, with shares

$$w_s = \frac{|S_s|}{m}, \qquad \sum_{s=1}^{k} w_s = 1$$

Under proportional allocation, stratum $s$ contributes $w_s \cdot m_{\text{test}}$ instances, drawn at random from within that stratum, so the realised test share equals $w_s$ up to rounding. Random sampling matches it only in expectation, with variance $w_s(1 - w_s)/m_{\text{test}}$ around it.

The stratification key must be categorical, so a continuous variable is binned first. In the [[California Housing|housing dataset]] median income is the variable most correlated with the target, so `pd.cut` slices it into five categories at 1.5, 3.0, 4.5 and 6.0 (income in tens of thousands of dollars). That column exists only to steer the split and is dropped from both halves right afterwards, so it never leaks in as a [[Feature]]. Create a categorical proxy, split on it, drop it: that move is the part worth remembering, and it generalizes to any continuous variable you need represented faithfully.

## Hyperparameters

| name | symbol | default | effect of increasing | how to tune |
|---|---|---|---|---|
| number of strata | $k$ | none, you choose the binning | mix matched more finely, but each stratum holds fewer instances until some are too small to sample from | enough that every stratum is well populated, five income bands on the housing data is the worked case and no stratum should be tiny |
| bin edges | - | none | - | place cuts where the variable's mass actually is, check the category counts before splitting |
| `test_size` | $r$ | 0.25 in `train_test_split`, 0.1 in `StratifiedShuffleSplit` | test estimate less noisy, training set smaller | 0.2 at moderate $m$, less as $m$ grows |
| `n_splits` | - | 10 in `StratifiedShuffleSplit` | more distinct stratified splits produced, linear cost | leave at 1 unless you genuinely need several splits |
| `random_state` | - | `None` | no monotone effect | fix an integer, see [[Random Seed]] |

## Failure modes

- Too many strata. Push $k$ up and some stratum holds a handful of instances, its test allocation rounds to zero or one, and the guarantee evaporates. scikit-learn refuses outright when a class has fewer members than the number of splits.
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
